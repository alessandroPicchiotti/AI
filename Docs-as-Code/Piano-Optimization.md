# Piano di ottimizzazione — GOW (NTSPJ.EShop)

## Contesto

Domanda di partenza: *"come conviene ottimizzare questa codebase?"*. Ricerca condotta con fan-out/fan-in — 5 ricercatori paralleli in sola lettura su cinque domini indipendenti (accesso dati, livello web, asset frontend, build/config, manutenibilità del codice C#), seguiti da verifica diretta delle affermazioni ad alto impatto e da riconciliazione dei punti in cui i ricercatori si contraddicevano.

Il codice si è rivelato **in condizioni migliori di quanto la sua età suggerisca**: il DI con Ninject è reale e rispettato (0 occorrenze di `new XxxRepository(` nei controller), le stored procedure sono parametrizzate, `TnetSql` gestisce correttamente le connessioni nei suoi metodi centrali, non ci sono `throw ex;` che perdono lo stack trace. Il debito vero è concentrato in pochi punti identificabili, non diffuso ovunque.

Il repo è confermato come **sorgente dei deploy in produzione**, il che sposta in testa alla lista alcune impostazioni di configurazione.

Due correzioni ai report dei ricercatori, verificate:
- Il livello web **non** è del tutto privo di ADO.NET raw: `Controllers\DownloadController.cs` (6 occorrenze) e `Controllers\BaseController.cs` (2) costruiscono `SqlCommand` direttamente. Isolato, ma reale.
- Il `CLAUDE.md` del progetto web non "vieta il bundling": constata solo (riga 28) che `System.Web.Optimization` non è referenziata nel progetto.

Esito voluto: ordine di esecuzione per rapporto impatto/costo, con un criterio esplicito su cosa **non** toccare — in una codebase senza test la maggior parte del debito non va ripagata.

---

## Fase 0 — Esposizione in produzione (urgente, ~1-2 ore)

Un solo file da modificare (`NTSPJ.EShop.WebUI_CUBE\Web.Release.config`) più due decisioni di configurazione. Nessun C#, nessuna view.

### Il meccanismo

`Web.config` è la configurazione di sviluppo; `Web.Release.config` contiene le trasformazioni XDT applicate al publish in configurazione Release. Oggi ha **una sola trasformazione attiva**, `<compilation xdt:Transform="RemoveAttributes(debug)" />` — tutto il resto sono esempi commentati generati da Visual Studio. Quindi in produzione arriva `Web.config` tale e quale, tranne l'attributo `debug`.

### 0.1 — Verifica preliminare (bloccante)

Confermare che la pubblicazione avvenga **in configurazione Release**. Se si pubblica in Debug le trasformazioni non vengono applicate e `debug="true"` (`Web.config:172`) è già oggi in produzione: niente ottimizzazioni JIT, `executionTimeout` di fatto infinito, batch compilation disattivata. Senza questa conferma il resto della fase non produce alcun effetto.

### 0.2 — Trasformazioni XDT mancanti

| `Web.config` | Effetto in produzione | Trasformazione |
|---|---|---|
| `customErrors mode="Off"` (riga 227) | Yellow Screen of Death all'utente: stack trace, percorsi fisici del server, spesso frammenti di query | `SetAttributes` → `mode="RemoteOnly"` |
| `directoryBrowse enabled="true"` (riga 260) | Directory senza documento di default navigabili come elenco file | `SetAttributes` → `enabled="false"` |
| `runAllManagedModulesForAllRequests="true"` (riga 255) | `CultureModule` ed Elmah girano su ogni CSS/JS/immagine — overhead, non sicurezza | `SetAttributes` → `false` |

Per l'ultima: verificare prima che `CultureModule` ed `ErrorLog` (`Web.config:257-258`) reggano `preCondition="managedHandler"`. Il `Web.config` di sviluppo resta intatto, quindi IIS Express su :61270 continua a mostrare gli stack trace come adesso.

### 0.3 — Membership provider senza difese contro il brute force

`Web.config:190` — `AspNetSqlMembershipProvider` con `minRequiredPasswordLength="1"`, `maxInvalidPasswordAttempts="100"`, `minRequiredNonalphanumericCharacters="0"`, `requiresQuestionAndAnswer="false"`. Password di un carattere ammesse e cento tentativi falliti prima del lockout: l'autenticazione è di fatto aperta a un attacco a dizionario. Le password sono `Hashed`, quindi il DB non espone testo in chiaro.

**Pesa più delle tre voci di 0.2.** Portare `maxInvalidPasswordAttempts` a un valore realistico (5) e `minRequiredPasswordLength` ad almeno 8. Attenzione: irrigidire la lunghezza minima **non invalida le password esistenti** (sono già hashate e la validazione si applica alla creazione/cambio), ma va comunicato prima di applicarlo, e `passwordAttemptWindow="10"` significa che utenti legittimi disattenti verranno bloccati per 10 minuti — decisione da concordare, non da imporre.

### 0.4 — Credenziali di dominio nei commenti

`Web.config:234-237` — quattro righe `<identity impersonate>` commentate, ognuna con username e password reali di utenti Windows. Inerti a runtime, ma diventano un problema nel momento del `git init` della fase 3.3, dove finirebbero nella storia del repo. Rimuoverle e, se quegli account esistono ancora, considerarli compromessi.

### 0.5 — Elmah: verifica a costo zero, non un buco confermato

`elmah.axd` è registrato come handler (`Web.config:220` e `:249`), nessuna `<location>` lo protegge e l'unico blocco `<authorization>` è commentato (righe 182-186). Il default di Elmah è però `allowRemoteAccess="false"` e nel file non esiste una sezione `<elmah><security>` che lo sovrascriva — **allo stato attuale la pagina è raggiungibile solo da localhost**. Vale comunque una richiesta esplicita a `https://<produzione>/elmah.axd` da fuori: costo nullo, e la posta in gioco è l'intero log degli errori applicativi.

---

## Fase 1 — Quick win a rischio zero (~2 ore, effetto su ogni pagina)

Nessuna di queste modifiche cambia comportamento applicativo.

**1.1 jQuery non minificato su ogni pagina del sito.** I tre layout caricano `Scripts/jquery-3.5.0.js` (292 KB) mentre `Scripts/jquery-3.5.0.min.js` (87 KB) è già nel repo, stessa versione, mai referenziato. Sostituire in:
- `Views\Shared\_Layout.cshtml:179`
- `Views\Shared\Site.Master:52`
- `Views\Shared\SiteNoDoc.Master:49`

**≈ −205 KB su ogni page load**, il singolo intervento col miglior rapporto impatto/costo dell'intero piano.

**1.2 CSS inesistente referenziato da tutti e quattro i layout.** `~/Content/GOW/css/personalizzazioni.css` non esiste (`Content\GOW\css\` contiene solo `Site.css`, `SiteNts.css`, `TelerikLight2.css`, `catalogo.css`). Genera un 404 per page load. Rimuovere il `<link>` da `_Layout.cshtml:28`, `Site.Master:45`, `SiteNoDoc.Master:46`, `PopUp.Master:23` — oppure creare il file vuoto se l'intento era un hook di personalizzazione per cliente. **Decidere quale delle due**, non lasciarlo com'è.

**1.3 Cache busting fuori sincrono tra i layout.** Lo stesso asset ha versioni diverse a seconda del layout che serve la pagina:
- `Content/GOW/css/Site.css`: `?v=14` in `_Layout.cshtml:24` e `Site.Master:40`, `?v=4` in `SiteNoDoc.Master:41`, **nessuna** in `PopUp.Master:24`
- `Scripts/Nts-pj.js`: `?v=7` (`_Layout.cshtml:189`), `?v=6` (`Site.Master:63`), `?v=5` (`SiteNoDoc.Master:60`), **nessuna** (`PopUp.Master:35`)

Chi passa da una pagina SiteNoDoc/PopUp si porta in cache la versione vecchia e non vede più gli aggiornamenti. Allineare tutte le occorrenze al valore più alto. Questo è lo scenario di cache stale che `CLAUDE.local.md` invita a sospettare per primo — qui è già in atto strutturalmente, non è un'ipotesi.

**1.4 `TnetRdl.cs` è vuoto.** `TNET\TnetRdl.cs`: 9.846 righe totali, di cui 8.330 commentate e 1.516 vuote — **zero righe di codice vivo** (verificato per conteggio diretto). È il file più grande della solution e inquina ogni ricerca full-text. Rimuoverlo dal progetto e dal disco, poi build di controllo con `/build-check`.

---

## Fase 2 — Correttezza a runtime (~1-2 giorni)

**2.1 Connection leak.** `NTSPJ.EShop.Domain\Concrete\anagraRepository.m.cs:41-55` — `SearchInDescrizioneFromInizio` apre `new SqlConnection(_connectionString)` (riga 48), la apre (riga 53) e la passa a `ToClienteList(cmd)` (righe 1093-1106), che dispone solo il `SqlDataReader`. La connessione non viene mai chiusa né disposta. È un metodo di ricerca cliente/fornitore, quindi verosimilmente dietro un campo di ricerca digitando: sotto carico esaurisce il connection pool. Fix: `using` sulla connessione, oppure `CommandBehavior.CloseConnection` sul reader in `ToClienteList`. **Cercare lo stesso pattern negli altri `*.m.cs`** prima di considerarlo chiuso.

**2.2 TinyMCE caricato su ~93 pagine, usato in 6.** `Site.Master:56`, `SiteNoDoc.Master:53`, `PopUp.Master:31` caricano incondizionatamente `Scripts/tiny_mce/tiny_mce.js` (197 KB). L'editor è inizializzato solo in `Areas\Admin\Views\GestioneNews\Modifica_Nuovo.aspx`, `NewsletterGestione\Nuovo_Modifica.aspx` e 4 view sotto `Views\Supporto\`. Spostare l'include in un placeholder opzionale (`ContentPlaceHolder` per i `.Master`, `@RenderSection("scripts", required: false)` per `_Layout.cshtml`) popolato solo dalle 6 view che lo usano. **−197 KB su ~87 pagine.** Costo medio perché va verificata pagina per pagina, ma il risparmio è il secondo per entità dopo jQuery.

**2.3 56 `catch` vuoti che silenziano gli errori.** Su 226 `catch (Exception` totali, 56 sono completamente vuoti (es. `CarrelloController.cs:2425`, `RecuperaPasswordController.cs:261`, `Areas\Faq\Models\MostraFaqModel.cs:56`). In più, i controller mostrano `ex.Message` all'utente senza loggare nulla (9 occorrenze in `DownloadController.cs`, 7 in `StampeController.cs`, 6 in `CarrelloController.cs`); in tutta la cartella `Controllers\` ci sono solo 11 chiamate di logging. Le uniche eccezioni tracciate sono quelle non gestite che arrivano a `Global.asax.cs:57-67` (Windows EventLog, source "Gow").

Approccio: aggiungere un helper di log centralizzato in `Controllers\BaseController.cs` e chiamarlo nei blocchi `catch`, **senza cambiare la logica di flusso**. Rivedere i 56 vuoti uno a uno — alcuni sono legittimi (parsing opzionale) e vanno lasciati con un commento che lo dichiari.

**2.4 Cache delle tabelle di decodifica.** L'unica cache in tutta l'app è `HttpContext.Cache` in `Controllers\CategorieProdottiController.cs:79,86,133,139` per l'albero delle categorie. Tabelle quasi immutabili (IVA/`tabciva`, `TipiPagamento`, `Mesi`, `Lingue`, Province, `TipiDocumenti`) vengono rilette a ogni richiesta. Partire **solo** da quelle davvero immutabili, con `HttpRuntime.Cache` e scadenza assoluta, senza costruire un'infrastruttura di invalidazione: se una decodifica cambia una volta all'anno, una scadenza di qualche ora è sufficiente.

---

## Fase 3 — Riproducibilità della build (~mezza giornata)

**3.1 28 view Razor su 54 non sono incluse nel `.csproj`** — tra cui `Views\_ViewStart.cshtml`, `Areas\Admin\Views\_ViewStart.cshtml`, `Views\Account\LogOn.cshtml`. In IIS Express funzionano (il view engine legge da disco), ma **un Web Deploy basato sugli item di progetto le esclude**. Dato che si pubblica da qui e la migrazione a Razor è in corso, questo è un guasto di produzione in attesa di accadere. Aggiungerle come `<Content Include="...">`.

**3.2 Tre `HintPath` verso cartelle inesistenti fuori dal repo:**
- `NTSPJ.EShop.WebUI_CUBE.csproj:161` → `..\..\..\..\BERTI_CUBE\...\NTSPJ.EShop.Reports.dll`
- `:171` → `..\..\..\..\_PRODOTTI_NTSPJ\GOW\Rel 04.00__pub\...\RouteDebug.dll`
- `:125` → `..\NTSPJ.Eshop.BUS_CUBE\packages\...\MailKit.dll` (cartella sibling inesistente; il progetto reale si chiama `BUS_EXP`)

Nessuno dei tre percorsi esiste su questo filesystem. La build oggi funziona solo perché le DLL sono già in `bin\`. Su una macchina pulita non funziona. Determinare per ciascuna se serve davvero, e nel caso portarla in `packages\` o in una cartella `lib\` versionata dentro il repo.

**3.3 `.gitignore` da 1 riga.** Contiene solo `CLAUDE.local.md`. Il repo non è ancora sotto git; un `git init && git add .` oggi porterebbe dentro `bin\` (324 MB), `packages\` (199 MB), `node_modules\` (12 MB), `.vs\` (32 MB) **e `Web.config` con credenziali SQL/SMTP in chiaro**. Scrivere un `.gitignore` completo **prima** del `git init`: `bin/`, `obj/`, `packages/`, `.vs/`, `node_modules/`, `log/`, `FILES_TMP/`, `*.user`, `*.suo`. Per `Web.config` serve la strategia usuale: committare un `Web.config` con placeholder e tenere i valori reali fuori dal repo (`configSource` su un file ignorato).

**3.4 `node_modules` incluso nel `.csproj`.** ~500 righe `<Content Include="node_modules\...">` (da `NTSPJ.EShop.WebUI_CUBE.csproj:631` in poi), 12 MB. `package.json` dichiara solo `bootstrap: ^5.3.3`, da cui i minificati vengono copiati **a mano** in `Content\lib\bootstrap5\` e `Scripts\lib\bootstrap5\`; non esiste pipeline gulp/webpack. Rimuovere quegli item dal progetto: un publish che li rispetta pubblica 12 MB di sorgenti npm mai serviti.

---

## Fase 4 — Pulizia (~2 ore, opportunistica)

- **~1,1 MB di JS morto**: `Scripts\jquery-ui-1.13.2.js` (548 KB), `jquery-ui-1.13.2.min.js` (255 KB), `MicrosoftAjax.debug.js` (316 KB) — zero riferimenti in `.cshtml/.aspx/.Master/.ascx/.js`. Confermare con una ricerca testuale finale prima di cancellare.
- **25 item `Content`/`None` orfani** nel `.csproj` che puntano a `.aspx`/`.ascx` già convertiti (`Views\Account\LogOn.aspx`, `Views\Shared\Error.aspx`, `Views\Shared\UC_HiddenAcquisto.ascx`) — residui della migrazione in corso.
- **Duplicazione verbatim**: `GetMenuProdottiConcatenato(string codiceDitta)` è identico in `ArticoliController.cs:467-474` e `CategorieProdottiController.cs:337-344`, entrambe le classi ereditano da `Controller` e non da una base comune. Estrarlo in `BaseController` o in un service.
- **`BouncyCastle.Cryptography 2.6.2` e `Portable.BouncyCastle 1.9.0` coesistono** in `NTSPJ.EShop.WebUI_CUBE\packages.config` (righe 5 e 30): stessa libreria, due lineage NuGet. Rimuovere quella non usata.
- **`bindingRedirect` di MimeKit** disallineato: `Web.config` reindirizza a `4.15.0.0`, `packages.config` installa `4.15.1`.

---

## Cosa deliberatamente NON fare

Il criterio: senza test di regressione, ogni refactoring strutturale è rischio puro.

- **Non migrare il data layer a EF/Dapper.** ~400 file di repository generati, 4.976 occorrenze ADO.NET su 216 file. Costo enorme, zero rete di sicurezza.
- **Non spezzare i controller monolitici** (`ArticoliController.cs` 206 KB/25 action, `CarrelloController.cs` 186 KB/29 action, `OrdiniController.cs` 111 KB/23 action). Sono grandi ma fanno orchestrazione, non accesso dati. Il refactoring big-bang costa più di quanto rende. Ridurli **opportunisticamente** quando si tocca una singola action.
- **Non spezzare i repository generati** (`anagraRepository.cs` e simili): scaffolding ripetitivo, non spaghetti code.
- **Non convertire async l'intera catena SOAP** verso Business File (`Proxy\WsBusinessFile.cs`, 2.651 righe, pattern APM legacy). Il problema è reale — nessuna action è `async`, ogni chiamata lenta blocca un thread IIS per l'intero timeout — ma va affrontato solo se e quando si manifesta thread starvation misurata, non preventivamente.
- **Non introdurre bundling ora.** `System.Web.Optimization` non è referenziata (`NTSPJ.EShop.WebUI_CUBE\CLAUDE.md:28`). I punti 1.1 e 2.2 recuperano già ~400 KB con rischio zero; il bundling su `Site.Master`, che è deprecato e in via di sostituzione, ha ROI basso rispetto al finire la migrazione Razor.
- **Non rinominare `NTSPJ.Eshop.BUS_EXP` in `EShop`.** 553 file usano `NTSPJ.EShop`, nessun file mescola le due casing al proprio interno: confusione reale ma confinata al nome del progetto. Tocca 3 `.csproj`, binding degli assembly e riferimenti IIS.
- **Non convertire le 10 pagine `TNET\*.aspx`** (1.477 righe, strumenti interni a basso traffico).
- **Non toccare i 1.599 `#region`** né i warning già filtrati da `/build-check` (`MSB3270`, `MSB3247`, `CS0436`).
- **Sessione InProc con `SessioneAcquisto` non serializzabile** (`Domain\Entities\SessioneAcquisto.cs`, contiene carrello + un riferimento a `IMemoCarrelloTestaRepository`): impedisce StateServer/SQL session mode, quindi un riavvio dell'app pool perde il carrello. Da segnalare come **limite architetturale noto**, non da risolvere in questo giro — esiste già `RecuperaCarrelloController.cs` che mitiga il caso.

---

## Verifica

Dopo ogni fase, non alla fine.

1. **Build**: `/build-check` sul progetto web — nessun errore, solo i warning noti (`MSB3270`, `MSB3247`, `CS0436`).
2. **Browser** su `http://localhost:61270/` con `/gow-verify-page`, hard reload:
   - una pagina servita da `_Layout.cshtml`, una da `Site.Master`, una da `SiteNoDoc.Master`, una popup da `PopUp.Master` — sono quattro percorsi di asset distinti e le fasi 1 e 2 li toccano tutti;
   - DevTools → Network: confermare che `jquery-3.5.0.min.js` sia servito, che **non** ci sia più il 404 su `personalizzazioni.css`, e misurare il peso totale trasferito prima/dopo;
   - Console: zero errori JS nuovi (il rischio concreto del punto 2.2 è una pagina che usa TinyMCE senza dichiarare la sezione).
3. **Punto 2.1**: verificare il fix del connection leak con `sp_who2` o `SELECT * FROM sys.dm_exec_connections` durante ricerche cliente ripetute, non solo a occhio sul codice.
4. **Punto 3.1**: dopo aver aggiunto le view al `.csproj`, un publish di prova su cartella locale e conferma che tutti i 54 `.cshtml` siano presenti nell'output.
5. **Fase 0**: verificabile solo in un ambiente di staging pubblicato in Release — provocare un'eccezione e confermare che non appaia lo stack trace.

Le fasi 1, 3 e 4 sono indipendenti fra loro e parallelizzabili. La fase 2 va fatta in sequenza, un punto alla volta con verifica in browser fra uno e l'altro.
