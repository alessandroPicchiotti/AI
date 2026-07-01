# Ricognizione: migrazione Bootstrap 3.4 -> Bootstrap 5 via npm

Nota: questo file e' un report di ricognizione (read-only), non un elenco di step di modifica automatica. Contiene i dati necessari per pianificare la migrazione incrementale insieme all'utente.

## 1. Struttura generale del progetto

Solution Gow-4.sln con 3 progetti:
- NTSPJ.EShop.Domain -- dominio/entita
- NTSPJ.Eshop.BUS_EXP -- business layer
- NTSPJ.EShop.WebUI_CUBE -- progetto web, ibrido ASP.NET MVC 5.2.9 + Web Forms, .NET Framework 4.8 (confermato anche da NTSPJ.EShop.WebUI_CUBE/CLAUDE.md, presente nel repo)

Il progetto e' un classico ASP.NET MVC "Web Application" (non .NET Core, non usa wwwroot). Cartelle principali dentro NTSPJ.EShop.WebUI_CUBE/:
- Views/ -- viste MVC, contiene sia .cshtml (Razor) sia .aspx/.ascx (Web Forms View Engine, ancora supportato da MVC) sia .Master (Web Forms master page)
- Content/ -- CSS, immagini, font (equivalente del vecchio "static assets", nessun wwwroot)
- Scripts/ -- tutti i file JS (jQuery, Bootstrap, Telerik, plugin vari), gestiti come Content nel .csproj, non via npm
- Areas/Admin/Views/, Areas/Faq/Views/ -- due Aree MVC, ciascuna con proprio _ViewStart.cshtml
- TNET/ -- pagine .aspx di utilita tecnica (fuori da Views)
- Nessuna cartella wwwroot (conferma: non e' ASP.NET Core)

E' confermato ibrido: 112 file .aspx referenziano MasterPageFile="~/Views/Shared/Site.Master", 1 referenzia SiteNoDoc.Master, 1 referenzia PopUp.Master. Esistono in parallelo _Layout.cshtml e _LayoutNts.cshtml per le viste Razor. Il file CLAUDE.md del progetto conferma: "Migrazione ASPX -> Razor (in corso)" -- la conversione .aspx -> .cshtml e' un lavoro incrementale gia in atto, quindi la migrazione Bootstrap si sovrappone a quella.

## 2. Come viene referenziato Bootstrap oggi

Versione: Bootstrap 3.4.1 ovunque (locale via NuGet + CDN in un solo file).

Package NuGet (packages.config del progetto WebUI):
- package id="bootstrap" version="3.4.1" targetFramework="net48"
- package id="Bootstrap.Datepicker" version="1.7.1" targetFramework="net48"

Cartelle NuGet corrispondenti: packages/bootstrap.3.4.1/, packages/Bootstrap.Datepicker.1.7.1/.

File fisici copiati in Content/ e Scripts/ (referenziati singolarmente nel .csproj come Content Include, righe 532-543, 616-632, 655-658, 707+):
- Content/bootstrap.css, bootstrap.min.css (+ .map)
- Content/bootstrap-theme.css, bootstrap-theme.min.css (+ .map)
- Content/bootstrap-datepicker.css, bootstrap-datepicker3.css e varianti standalone/min (+ .map) -- plugin datepicker Bootstrap 3
- Scripts/bootstrap.js, bootstrap.min.js
- Scripts/bootstrap-datepicker.js, bootstrap-datepicker.min.js (+ localizzazioni in Scripts/locales/bootstrap-datepicker.*.min.js, decine di file)

Riferimenti nei layout (i file "entry point" da modificare per la migrazione):

| File | CSS bootstrap | JS bootstrap |
|---|---|---|
| Views/Shared/_Layout.cshtml (layout Razor principale, MVC) | ~/Content/bootstrap.min.css (locale) | ~/Scripts/bootstrap.js (locale) |
| Views/Shared/_LayoutNts.cshtml (layout Razor, area FAQ / pagine Nts) | TRIPLO riferimento: CDN cdnjs twitter-bootstrap 3.4.1 css + locale ~/Content/bootstrap.css + locale ~/Content/bootstrap.min.css (ridondanza/bug esistente) | CDN cdnjs twitter-bootstrap 3.4.1 js |
| Views/Shared/Site.Master (master page Web Forms, usato da 112 view .aspx) | ~/Content/bootstrap.min.css (locale) | ~/Scripts/bootstrap.js (locale) |
| Views/Shared/SiteNoDoc.Master (Web Forms, 1 uso) | ~/Content/bootstrap.min.css (locale) | ~/Scripts/bootstrap.js (locale) |
| Views/Shared/PopUp.Master (Web Forms, 1 uso, popup) | ~/Content/bootstrap.min.css (locale) | nessun bootstrap.js incluso |

Tutti i layout includono anche bootstrap-datepicker.css / bootstrap-datepicker3.css (locali), tranne PopUp.Master.

Nessun BundleConfig.cs / System.Web.Optimization in uso (il CLAUDE.md del progetto lo conferma esplicitamente: "Non aggiungere System.Web.Optimization a Views/Web.config -- la DLL non e referenziata nel progetto"). Tutti i CSS/JS sono inclusi con link/script singoli, non bundle.

libman.json esiste ma e' vuoto ("libraries": []) -- non gestisce Bootstrap.

## 3. Build tool npm/node esistente

Nessuno. Verificato:
- Nessun package.json in tutta la repo
- Nessun webpack.config.js, gulpfile.js, .esproj, node_modules/
- libman.json presente ma vuoto/inutilizzato

Tutte le librerie front-end sono gestite tramite:
1. NuGet (packages.config): bootstrap 3.4.1, Bootstrap.Datepicker 1.7.1, jQuery 2.2.0/3.5.0.1 (due versioni installate), jQuery.UI.Combined (due versioni: 1.12.1 e 1.13.2), jQuery.Validation (due versioni: 1.11.1 e 1.19.3), Modernizr 2.6.2, Respond 1.4.2, html5-shiv 3.7.3, popper.js 1.14.3, FontAwesome 4.7.0, Microsoft.jQuery.Unobtrusive.Validation (due versioni), Microsoft.AspNet.Web.Optimization 1.1.3 (installato ma non referenziato/usato)
2. File JS/CSS copiati "a mano" in Scripts/Content/ (Telerik 2011.1.315, summernote, tiny_mce, datatables, ecc. -- molti senza corrispondente pacchetto NuGet)

Introdurre npm significa aggiungere un package.json ex novo nella root di NTSPJ.EShop.WebUI_CUBE/ (o alla root solution) e un build step (es. semplice script npm/copy o bundler leggero) che copi node_modules/bootstrap/dist/{css,js} dentro Content/Scripts/, oppure referenziare direttamente i file da node_modules nel .csproj con un target di copia in fase di build. Da rimuovere: pacchetto NuGet bootstrap (3.4.1) e i relativi Content Include nel .csproj; da valutare se rimuovere anche Bootstrap.Datepicker (plugin CSS/JS Bootstrap-3-only, non ha una release ufficiale per Bootstrap 5 -- vedi punto 6).

## 4. Sottocartelle Views/ -- inventario per pianificare la migrazione incrementale

Conteggio file .cshtml + .aspx + .ascx + .Master per sottocartella (Views principali):

| Cartella | File view |
|---|---|
| Views/Shared | 24 (mix .cshtml/.aspx/.ascx/.Master -- contiene i layout condivisi) |
| Views/Articoli | 15 |
| Views/CategorieProdotti | 7 |
| Views/Ordini | 6 |
| Views/Account | 5 |
| Views/Carrello | 5 |
| Views/Newsletter | 5 |
| Views/Clienti | 4 |
| Views/News | 4 |
| Views/Supporto | 4 |
| Views/Download | 3 |
| Views/EstrattoConto | 3 |
| Views/Home | 3 |
| Views/Lead | 3 |
| Views/NuovoDoc | 3 |
| Views/RegistrazioneUtente | 3 |
| Views/Ricambi | 3 |
| Views/Schede | 3 |
| Views/Stampe | 3 |
| Views/RecuperaCarrello | 2 |
| Views/RecuperaPassword | 2 |
| Views/SessioneAcquistiRiassunto | 2 |
| Views/Fatture | 2 |
| Views/Accettazione | 1 |
| Views/DistinteIncasso | 1 |
| Views/DownloadTutti | 1 |
| Views/FamGruSubgru | 1 |
| Views/GDPR | 1 |
| Views/Licenza | 1 |
| Views/SMTP | 1 |
| Views/TipiPagClaSconto | 1 |

Aree MVC (Areas/NomeArea/Views/):

| Cartella | File view |
|---|---|
| Areas/Admin/Views/Utenti | 10 |
| Areas/Admin/Views/Reports | 8 |
| Areas/Admin/Views/TipiDocumenti | 8 |
| Areas/Faq/Views/GestFaq | 7 |
| Areas/Admin/Views/Ruoli | 5 |
| Areas/Faq/Views/Archivi | 5 |
| Areas/Admin/Views/ArticoGow | 3 |
| Areas/Admin/Views/AssArtiTipiArt | 1 |
| Areas/Admin/Views/GestioneNews | 2 |
| Areas/Admin/Views/Menu | 2 |
| Areas/Admin/Views/NewsletterGestione | 2 |
| Areas/Admin/Views/NewsletterIscritti | 2 |
| Areas/Faq/Views/Shared | 1 |

Totali per estensione, su Views/ + Areas/:
- .cshtml: 27
- .aspx: 131
- .ascx: 19
- .Master: 3 (Site.Master, SiteNoDoc.Master, PopUp.Master -- tutti in Views/Shared)

Da notare anche TNET/ (10 file .aspx, pagine tecniche/utility, fuori da Views) che referenziano bootstrap.min.css solo indirettamente tramite il proprio markup (non hanno master page comune con Bootstrap -- verificare singolarmente se serve migrarle).

La netta prevalenza di .aspx/.ascx (150 file) su .cshtml (27) conferma che la conversione ASPX->Razor e' ancora agli inizi: la migrazione Bootstrap toccherà quindi soprattutto markup Web Forms, non Razor.

## 5. Uso di classi Bootstrap 3 (portata della migrazione)

Conteggi (occorrenze totali / file coinvolti) su Views/ + Areas/ (estensioni .cshtml/.aspx/.ascx/.Master):

| Pattern Bootstrap 3 | Occorrenze | File |
|---|---|---|
| form-group | 234 | 69 |
| btn-default | 162 | 41 |
| data-toggle= | 54 | 27 |
| col-xs- | 115 (Views) + 119 (Areas) = 234 | 31 + 29 = 60 |
| data-target= | 19 | 9 |
| panel-default/heading/body/title | 13 | 10 |
| data-dismiss= | 5 | 5 |
| img-responsive | 5 | 5 |
| well (classe, non testo) | 4 | 4 |
| col-md-offset | 0 | 0 |
| hidden-xs | 0 | 0 |
| visible-xs | 0 | 0 |
| pull-left | 0 | 0 |
| pull-right | 0 | 0 |
| navbar-toggle | 3 | 2 |
| glyphicon | 10 | 7 (uso concentrato: menu/navbar/dropdown nei layout) |

Note:
- form-group (234) e col-xs-/btn-default (234 e 162) sono di gran lunga i pattern piu diffusi: interessano quasi ogni view con form o griglia. In Bootstrap 5, form-group e' rimosso (sostituito da margini utility mb-3 ecc.), col-xs- diventa semplicemente col- (nessun breakpoint xs esplicito), btn-default diventa btn-secondary.
- glyphicon (10 occorrenze) e' concentrato praticamente solo nei layout condivisi (_Layout.cshtml, _LayoutNts.cshtml, Site.Master, SiteNoDoc.Master, piu Areas/Admin/Views/AssArtiTipiArt/GestioneAssociazione.aspx e Areas/Admin/Views/ArticoGow/ArticoGow.aspx) -- Font Awesome (gia presente in progetto, Content/font-awesome.css) puo sostituirlo con relativa facilita.
- navbar-toggle, data-toggle=/data-target=/data-dismiss= sono concentrati nei 4 layout principali (navbar collapse, dropdown menu utente, modal) -- punto di attacco naturale per iniziare la migrazione (basso numero di file, alto impatto visivo).
- Plugin bootstrap-datepicker (Bootstrap-3-only, vedi Content/bootstrap-datepicker*.css, Scripts/bootstrap-datepicker*.js): usato via classi CSS proprie, non intercettato dai pattern sopra -- va cercato/valutato separatamente (nessuna release ufficiale per Bootstrap 5; alternative: flatpickr, air-datepicker, o passare a input type=date).
- Nessun uso di select2 rilevato.

## 6. Plugin JS Bootstrap inizializzati via jQuery

Ricerca mirata di .modal(, .tooltip(, .popover(, .dropdown(, .collapse( in file view (esclusi i file di libreria in Scripts/, dove i match erano falsi positivi da tiny_mce/jquery-ui/summernote):

- Areas/Admin/Views/AssArtiTipiArt/GestioneAssociazione.aspx:275 -- selettore [data-toggle=tooltip] con .tooltip()
- Areas/Admin/Views/ArticoGow/ArticoGow.aspx:417 -- selettore [data-toggle=tooltip] con .tooltip()
- Views/SMTP/Index.aspx:190 -- riga commentata con .modal("hide") (dead code, da verificare)
- Views/Shared/_LayoutNts.cshtml:209 -- #modalTextarea con .modal({ backdrop: static })

Impatto limitato (4 punti), ma in Bootstrap 5 tutta l'API jQuery-based e' rimossa: vanno riscritti con l'API JS nativa (bootstrap.Modal.getOrCreateInstance / new bootstrap.Tooltip, ecc.) oppure mantenuti tramite gli attributi data-bs-* (rinominati da data-*) se si usa solo l'attivazione dichiarativa via markup (che e' il caso predominante: data-toggle=collapse|dropdown|tooltip, data-target=, data-dismiss= diffusi nei layout, vedi punto 5) -- questi vanno tutti rinominati in data-bs-toggle, data-bs-target, data-bs-dismiss.

Il file JS applicativo Scripts/Nts-pj.js (174KB, "centrale" secondo CLAUDE.md) non risulta usare le API jQuery di Bootstrap nei pattern cercati -- va comunque riletto per sicurezza durante la migrazione, dato il monito nel CLAUDE.md di non modificarne le closures senza contesto completo.

## 7. Form Web Forms (.aspx/.ascx) e site.master

Confermato: Site.Master e' una vera Web Forms Master Page (direttiva Master Language=C# Inherits=System.Web.Mvc.ViewMasterPage), usata dal View Engine WebFormViewEngine di ASP.NET MVC (non e' un'app Web Forms "pura" -- e' l'integrazione ibrida standard di MVC 5 con .aspx come view engine alternativo a Razor).

- 112 file .aspx dichiarano MasterPageFile="~/Views/Shared/Site.Master"
- 1 file usa SiteNoDoc.Master
- 1 file usa PopUp.Master
- I file .ascx (19 totali) sono Web Forms User Controls, richiamati sia da .aspx che, per compatibilita, referenziati anche da alcune .cshtml tramite Html.Action verso controller che a loro volta possono restituire .ascx.

Questo conferma che i 3 file .Master sono la fonte "Web Forms" di Bootstrap (styles/script identici, duplicati rispetto a _Layout.cshtml/_LayoutNts.cshtml) e vanno migrati in coppia con essi per mantenere coerenza visiva tra le view non ancora convertite in Razor e quelle gia convertite.

## Riepilogo per la pianificazione incrementale

Punti di ingresso Bootstrap da aggiornare per primi (pochi file, alto impatto, sbloccano tutto il resto):
1. Views/Shared/_Layout.cshtml, Views/Shared/_LayoutNts.cshtml, Views/Shared/Site.Master, Views/Shared/SiteNoDoc.Master, Views/Shared/PopUp.Master -- sostituire i link/script bootstrap con i file da npm (node_modules/bootstrap/dist/... copiati o referenziati), rimuovere il duplicato CDN/bootstrap.css in _LayoutNts.cshtml, rinominare data-toggle/target/dismiss in data-bs-*, sostituire glyphicon con Font Awesome, navbar-toggle con markup navbar BS5.
2. Rimuovere pacchetto NuGet bootstrap (3.4.1) da packages.config e dal .csproj (righe elencate al punto 2), una volta introdotto npm.
3. Valutare sostituto per Bootstrap.Datepicker (nessuna versione BS5 ufficiale).
4. Migrazione view-by-view: ordine suggerito dalle cartelle piu piccole (1 file: Accettazione, DistinteIncasso, DownloadTutti, FamGruSubgru, GDPR, Licenza, SMTP, TipiPagClaSconto, AssArtiTipiArt) verso le piu grandi (Articoli 15, Views/Shared 24, Areas/Admin/Utenti 10), sostituendo form-group con mb-3/rimosso, col-xs- con col-, btn-default con btn-secondary, panel-* con card-*, img-responsive con img-fluid, well con card o p-3 bg-light in ciascun file.
5. In parallelo alla skill convert-view gia presente nel progetto (.claude/skills/convert-view) per la conversione ASPX->Razor, che puo essere l'occasione naturale per applicare anche le classi Bootstrap 5 durante la conversione delle 131 view .aspx + 19 .ascx ancora da convertire.

## Revisione richiesta dall'utente (2026-07-02)

Richiesta: sostituire il datepicker con il tag nativo input type="date", POI applicare le classi Bootstrap 5 (ordine: prima datepicker, poi classi).

### Verifica approfondita: cosa c'e' davvero da sostituire

Ho cercato tutti gli usi reali (non solo gli include nei layout) di "datepicker" nel codice. Risultato: nel progetto convivono DUE componenti distinti chiamati "datepicker", solo uno dei due e' legato a Bootstrap:

1. bootstrap-datepicker (il plugin jQuery/Bootstrap-3 discusso al punto 2 e 5 del piano originale: Content/bootstrap-datepicker*.css, Scripts/bootstrap-datepicker*.js, package NuGet Bootstrap.Datepicker 1.7.1). E' incluso nei 3 layout Web Forms/Razor (_Layout.cshtml, Site.Master, SiteNoDoc.Master) ma RISULTA DI FATTO NON UTILIZZATO nelle view: le uniche chiamate jQuery .datepicker() trovate sono commentate (dead code) in:
   - Views/Download/Download.aspx righe 18, 22
   - Views/DownloadTutti/DownloadTutti.aspx righe 14, 18
   - Areas/Admin/Views/NewsletterGestione/Nuovo_Modifica.aspx riga 22
   - Areas/Admin/Views/GestioneNews/Modifica_Nuovo.aspx riga 26
   - Areas/Admin/Views/GestioneNews/GestioneNews.aspx righe 21-22
   Nessuna classe CSS tipo class="datepicker" o data-provide="datepicker" e' stata trovata attiva in nessuna view.

2. Html.Telerik().DatePicker() -- un widget MVC del componente Telerik (2011.1.315, gia in uso nel progetto per grid/editor/ecc.), COMPLETAMENTE SEPARATO da Bootstrap e da bootstrap-datepicker. E' quello REALMENTE usato oggi per i campi data, negli stessi 5 file sopra elencati (dove le chiamate .datepicker() commentate sono probabilmente residui di un vecchio tentativo mai completato):
   - Views/Download/Download.aspx righe 40, 44 -- campi FilDataIni / FilDataFin
   - Views/DownloadTutti/DownloadTutti.aspx righe 38, 42 -- stessi campi
   - Areas/Admin/Views/GestioneNews/Modifica_Nuovo.aspx riga 48 -- campo DataNews
   - Areas/Admin/Views/GestioneNews/GestioneNews.aspx righe 46, 49 -- campi FindDaData / FindAData
   (Nuovo_Modifica.aspx di NewsletterGestione ha solo la riga commentata, da verificare se il campo DataNewsletter usa un altro controllo non "datepicker"-named)

### Interpretazione e impatto sul piano

Poiche' la richiesta e' nel contesto della migrazione Bootstrap, il "datepicker da sostituire con input type=date" e' presumibilmente il bootstrap-datepicker (punto 1) -- che pero' e' gia' di fatto morto/non referenziato in nessuna view attiva. Se e' questo il caso, l'azione e' semplice:
- Rimuovere gli include CSS/JS di bootstrap-datepicker dai 3 layout (_Layout.cshtml, Site.Master, SiteNoDoc.Master) -- PopUp.Master e _LayoutNts.cshtml non li includono gia'
- Rimuovere il pacchetto NuGet Bootstrap.Datepicker (1.7.1) da packages.config e i relativi Content Include dal .csproj (Content/bootstrap-datepicker*.css/.map, Scripts/bootstrap-datepicker*.js, Scripts/locales/bootstrap-datepicker.*.min.js)
- Nessuna modifica alle view necessaria per questo componente, dato che non e' in uso attivo

Se invece l'intenzione e' sostituire anche i campi che oggi usano Html.Telerik().DatePicker() con <input type="date"> nativo (i 5 file sopra elencati, 7 campi data totali), si tratta di un lavoro diverso e piu ampio: tocca il rendering server-side Telerik (non CSS/JS Bootstrap), il binding del Model (formati data, culture), e possibile styling con classi Bootstrap 5 (form-control) sull'input nativo al posto del widget Telerik.

### Domanda da chiarire con l'utente prima di eseguire

Quale datepicker intende sostituire l'utente?
- (A) Solo bootstrap-datepicker (il plugin Bootstrap 3, gia' inutilizzato) -- pulizia rapida, nessun impatto su UI/funzionalita' esistenti
- (B) Anche/soltanto i campi Html.Telerik().DatePicker() nei 5 file sopra elencati, sostituendoli con <input type="date"> reale -- richiede modifiche a controller/model/binding oltre che al markup, e va valutato caso per caso (formato data, localizzazione IT, eventuali JS collegati)

### Ordine di esecuzione rivisto (in attesa di conferma punto precedente)

1. Chiarire con l'utente la portata esatta della sostituzione datepicker (A, B, o entrambi)
2. Sostituire/rimuovere il datepicker secondo la portata concordata (prima di toccare le classi Bootstrap, come richiesto dall'utente)
3. Migrare i layout condivisi (_Layout.cshtml, _LayoutNts.cshtml, Site.Master, SiteNoDoc.Master, PopUp.Master) a Bootstrap 5 via npm, incluse le classi/attributi (data-bs-*, glyphicon->Font Awesome, navbar-toggle, form-group, col-xs-, btn-default, panel-*, img-responsive, well) come da punto 5 del piano originale
4. Rimuovere il pacchetto NuGet bootstrap 3.4.1 (e Bootstrap.Datepicker se rientra nella portata concordata) da packages.config/.csproj
5. Migrazione view-by-view delle classi Bootstrap 5, dalle cartelle piu piccole alle piu grandi, come da Riepilogo del piano originale (punto 4)

### Conferma utente

L'utente ha scelto l'opzione (C): sostituire ENTRAMBI i datepicker.
- Rimuovere completamente bootstrap-datepicker (CSS/JS nei 3 layout, package NuGet Bootstrap.Datepicker, Content Include nel .csproj) -- gia' inutilizzato, nessun impatto su UI/funzionalita'.
- Sostituire anche i campi Html.Telerik().DatePicker() con <input type="date"> nativo nei 5 file/7 campi individuati:
  - Views/Download/Download.aspx -- FilDataIni, FilDataFin
  - Views/DownloadTutti/DownloadTutti.aspx -- FilDataIni, FilDataFin
  - Areas/Admin/Views/GestioneNews/Modifica_Nuovo.aspx -- DataNews
  - Areas/Admin/Views/GestioneNews/GestioneNews.aspx -- FindDaData, FindAData
  - Areas/Admin/Views/NewsletterGestione/Nuovo_Modifica.aspx -- DataNewsletter (verificare nome/binding esatto del campo, la riga trovata era solo il residuo commentato $("#DataNewsletter").datepicker())

Questa sostituzione (B) richiede, per ciascuno dei 5 file, di verificare: nome esatto della property nel Model, formato data atteso da controller/binder (probabilmente dd/MM/yyyy per cultura IT vs formato ISO yyyy-MM-dd richiesto da input type=date), eventuale JS collegato (validazione, submit), e styling con classe Bootstrap 5 form-control sull'input nativo al posto del markup generato da Telerik.

### Ordine di esecuzione definitivo

1. Sostituzione datepicker (entrambi):
   a. Rimuovere bootstrap-datepicker da _Layout.cshtml, Site.Master, SiteNoDoc.Master (CSS/JS), da packages.config e dal .csproj (Content Include relativi)
   b. Sostituire Html.Telerik().DatePicker() con <input type="date"> nei 5 file/7 campi elencati, verificando model/binding/formato data per ciascuno
2. Migrazione Bootstrap 5 dei layout condivisi (_Layout.cshtml, _LayoutNts.cshtml, Site.Master, SiteNoDoc.Master, PopUp.Master): introduzione npm, sostituzione CSS/JS, rinomina data-toggle/target/dismiss -> data-bs-*, glyphicon -> Font Awesome, navbar-toggle -> markup navbar BS5
3. Rimozione pacchetto NuGet bootstrap 3.4.1 da packages.config/.csproj una volta introdotto npm
4. Migrazione view-by-view delle classi Bootstrap 5 (form-group, col-xs-, btn-default, panel-*, img-responsive, well), dalle cartelle piu piccole alle piu grandi, come da Riepilogo del piano originale (punto 4), applicando anche form-control ai nuovi input type=date introdotti al passo 1b

### Vincolo di processo richiesto dall'utente

L'utente richiede che l'esecuzione avvenga UNA VIEW/FILE ALLA VOLTA, con STOP obbligatorio dopo ogni singola modifica in attesa di conferma esplicita dell'utente, che effettuera' un test visivo nel browser prima di dare il via libera al file successivo. Questo vincolo si applica a tutte le fasi che toccano piu file (sostituzione datepicker sui 5 file, migrazione dei 5 layout, migrazione view-by-view delle classi Bootstrap 5 su tutte le cartelle Views/Areas).

Flusso operativo per ciascun file/step, valido per tutte le fasi del piano:
1. Modificare UN SOLO file (o master/layout)
2. Segnalare all'utente che la modifica e' pronta per il test
3. Attendere che l'utente esegua il test visivo su browser e dia conferma esplicita
4. Solo dopo conferma, passare al file successivo secondo l'ordine gia' definito (datepicker -> layout condivisi -> rimozione NuGet bootstrap -> view-by-view dalle cartelle piu piccole alle piu grandi)
5. Se l'utente segnala un problema visivo, correggere il file corrente prima di procedere oltre

Questo vincolo sostituisce/integra il punto "Ordine di esecuzione definitivo" della sezione precedente: l'ordine dei file resta quello li' definito, ma l'esecuzione avviene sempre in modalita' sequenziale con conferma singola, mai in batch su piu file contemporaneamente.

## Domande di chiarimento aperte (prima di iniziare l'esecuzione)

1. Setup npm: dove va creato il package.json (dentro NTSPJ.EShop.WebUI_CUBE/ o alla root della solution Gow-4.sln) e come deve avvenire il trasferimento dei file da node_modules/bootstrap/dist verso Content/Scripts -- copia manuale one-shot, script npm (es. postinstall/copyfiles), o target MSBuild custom nel .csproj che copia ad ogni build?
2. Popper.js: il progetto ha gia' popper.js 1.14.3 via NuGet, ma Bootstrap 5 richiede Popper 2.x (bundle bootstrap.bundle.js lo include gia'). Va aggiornato/installato Popper 2.x via npm insieme a Bootstrap, rimuovendo la versione 1.14.3 esistente?
3. Il campo DataNewsletter in Areas/Admin/Views/NewsletterGestione/Nuovo_Modifica.aspx ha solo una riga commentata trovata ($("#DataNewsletter").datepicker()), nessun controllo Telerik attivo individuato per quel campo in quel file -- va verificato con l'utente se il campo esiste davvero con quel nome/id o e' gestito diversamente, prima di procedere alla sostituzione.
4. Formato data: i campi Html.Telerik().DatePicker() lavorano probabilmente in formato cultura IT (dd/MM/yyyy), mentre input type=date richiede formato ISO (yyyy-MM-dd) in HTML e restituisce comunque un DateTime nel binding MVC -- va confermato se e' sufficiente il binder standard di MVC (di norma gestisce la conversione) o serve un adattamento esplicito nei controller/model per ciascuno dei 5 file.
5. Ordine concreto di partenza tra i 5 file del datepicker (Download.aspx, DownloadTutti.aspx, Modifica_Nuovo.aspx GestioneNews, GestioneNews.aspx, Nuovo_Modifica.aspx NewsletterGestione) -- da quale iniziare?
6. Compatibilita' browser: il progetto include html5shiv, respond.js e Modernizr 2.6.2 (supporto IE legacy), che Bootstrap 5 non supporta piu' (droppa IE11). Vanno rimossi questi pacchetti/script insieme alla migrazione, o mantenuti per compatibilita' con eventuali utenti su browser vecchi?

## Risposte utente alle domande di chiarimento (2026-07-02)

1. Setup npm: CONFERMATO -- package.json va creato dentro NTSPJ.EShop.WebUI_CUBE/ (non alla root della solution).
2. Popper.js: l'utente ha risposto "aggiorna popper.js alla versione nuget per bootstrap 5" -- risposta ambigua da chiarire ulteriormente, vedi nota sotto.
3. Campo DataNewsletter in Nuovo_Modifica.aspx: CONFERMATO -- procedere comunque con la modifica di quel file (l'utente ha detto "modifica pure"), quindi va incluso nell'elenco dei 5 file da trattare per la sostituzione datepicker, verificando durante l'esecuzione il binding effettivo del campo.
4. Formato data (binder standard MVC vs adattamento esplicito): NON ANCORA RISPOSTA -- rimane aperta.
5. Ordine di partenza: CONFERMATO -- si parte da Views/Download/Download.aspx.
6. Compatibilita' browser legacy: CONFERMATO -- rimuovere html5shiv, respond.js e Modernizr 2.6.2 (script include nei layout + pacchetti NuGet html5-shiv 3.7.3, Respond 1.4.2, Modernizr 2.6.2, oltre ai file fisici in Scripts/).

### Nota da chiarire: ambiguita' sulla risposta 2 (Popper.js)

L'utente ha scritto "aggiorna popper.js alla versione nuget per bootstrap 5", ma questo e' ambiguo nel contesto della migrazione a npm:
- Se si intende "aggiorna il pacchetto NuGet popper.js alla versione compatibile con Bootstrap 5" -- non esiste un pacchetto NuGet ufficiale Popper 2.x mantenuto in modo standard per questo scopo; il pacchetto NuGet popper.js attuale e' fermo a 1.14.3 (Bootstrap 3/4 friendly, non compatibile con Bootstrap 5 che richiede Popper 2.x)
- Se invece si intende "aggiorna Popper alla versione che serve per Bootstrap 5" (cioe' Popper 2.x, il concetto generale di "versione giusta per BS5" senza legarsi specificamente a NuGet) -- dato che l'intero piano prevede di spostare Bootstrap su npm, l'opzione piu coerente e' installare Popper 2.x via npm (incluso automaticamente nel bundle bootstrap.bundle.min.js di Bootstrap 5, che gia' contiene Popper) e rimuovere sia il pacchetto NuGet popper.js 1.14.3 sia i file Scripts/popper.js, Scripts/popper.min.js, Scripts/popper-utils.js e varianti attualmente copiati a mano

Verra' richiesta conferma esplicita su questo punto prima di procedere alla fase 2 (migrazione layout), che e' quella che coinvolge Popper.

### Decisione su Popper.js (2026-07-02)

L'utente ha lasciato la scelta a discrezione: "procedi come ritieni opportuno, non fare caso alla mia risposta".

DECISIONE: Popper verra' gestito interamente via npm come dipendenza di Bootstrap 5, usando il bundle bootstrap.bundle.min.js (che include gia' Popper 2.x al suo interno, nessuna dipendenza npm separata da aggiungere esplicitamente per Popper). Di conseguenza:
- Rimuovere il pacchetto NuGet popper.js 1.14.3 da packages.config e dal .csproj
- Rimuovere i file fisici Scripts/popper.js, Scripts/popper.min.js, Scripts/popper-utils.js, Scripts/popper-utils.min.js e relativi .map, e i rispettivi Content Include nel .csproj
- Nei layout, usare bootstrap.bundle.min.js (Bootstrap + Popper) al posto dei riferimenti separati a bootstrap.js e popper.js

Questa decisione verra' applicata durante la fase 2 del piano (migrazione dei layout condivisi a Bootstrap 5 via npm).

### Decisione su formato data (2026-07-02)

L'utente ha scelto l'OPZIONE B: verificare/adattare esplicitamente il binding di ciascun controller/model per ognuno dei 5 file, invece di affidarsi al model binder standard di MVC senza controllo.

DECISIONE: per ciascuno dei 5 file trattati nella fase 1 (sostituzione datepicker), prima di sostituire Html.Telerik().DatePicker() con input type="date", verificare esplicitamente:
- la property del Model associata al campo (tipo DateTime/DateTime?/string, eventuali attributi DataType/DisplayFormat)
- il Controller/Action che riceve il valore in POST/GET (parametri, eventuale ModelBinder custom, filtri di culture)
- il formato effettivo atteso (probabile dd/MM/yyyy per cultura IT) e la necessita' di conversione esplicita verso/da ISO 8601 (yyyy-MM-dd) richiesto da input type=date
- se serve, aggiungere conversione esplicita nel controller/model (es. parsing con CultureInfo.InvariantCulture per il valore ISO in ingresso, formattazione ToString("yyyy-MM-dd", CultureInfo.InvariantCulture) per il valore in uscita verso la view)

Questo controllo va fatto singolarmente per ciascuno dei 5 file, nell'ordine gia' concordato (si parte da Download.aspx), sempre nel rispetto del vincolo di processo "una view alla volta, con conferma dell'utente dopo test visivo prima di procedere al file successivo".

## Piano DEFINITIVO -- pronto per esecuzione

Tutte le domande di chiarimento sono state risposte. Il piano e' considerato completo e pronto per passare alla fase di esecuzione, in attesa del via libera esplicito dell'utente per uscire dalla Plan Mode.

Riepilogo decisioni finali:
- npm: package.json dentro NTSPJ.EShop.WebUI_CUBE/
- Popper: gestito via npm nel bundle bootstrap.bundle.min.js di Bootstrap 5, rimozione NuGet popper.js 1.14.3 e file manuali
- Datepicker: sostituzione di ENTRAMBI i componenti (bootstrap-datepicker inutilizzato + Telerik DatePicker -> input type=date) sui 5 file (Download.aspx, DownloadTutti.aspx, Modifica_Nuovo.aspx GestioneNews, GestioneNews.aspx, Nuovo_Modifica.aspx NewsletterGestione), con verifica esplicita di binding/formato data per ciascuno (Opzione B)
- Ordine di partenza: Download.aspx
- Browser legacy: rimozione di html5shiv, respond.js, Modernizr 2.6.2 (script, NuGet, file fisici)
- Processo: esecuzione rigorosamente sequenziale, un file alla volta, con stop e attesa di conferma dell'utente dopo test visivo su browser prima di ogni file successivo
- Ordine complessivo delle fasi: (1) sostituzione datepicker sui 5 file con verifica binding, (2) migrazione Bootstrap 5 dei 5 layout condivisi via npm (incluso bundle Popper, rimozione bootstrap.js/popper.js separati, data-bs-*, glyphicon->Font Awesome, navbar-toggle, rimozione html5shiv/respond/modernizr), (3) rimozione pacchetto NuGet bootstrap 3.4.1, (4) migrazione view-by-view delle classi Bootstrap 5 (form-group, col-xs-, btn-default, panel-*, img-responsive, well) dalle cartelle piu piccole alle piu grandi, sempre una alla volta con conferma

### Richiesta utente: spostare il file di piano nella root del progetto

L'utente ha chiesto di spostare questo markdown di piano dalla cartella .claude/plans/ alla root del progetto NTSPJ.EShop.WebUI_CUBE/, dato che riguarda solo quel progetto.

NOTA IMPORTANTE: siamo in Plan Mode. In questa modalita' l'unico file che e' consentito modificare e' il file di piano stesso, nella sua posizione originale (.claude/plans/); non e' consentito creare/spostare file nel progetto reale finche' non si esce dalla Plan Mode. Ho quindi rimandato l'operazione di spostamento fisico del file (es. verso NTSPJ.EShop.WebUI_CUBE/PIANO_MIGRAZIONE_BOOTSTRAP5.md o nome equivalente) al momento dell'uscita dalla Plan Mode, come primissima azione da eseguire prima di iniziare la fase 1 (sostituzione datepicker su Download.aspx).

AZIONE DA ESEGUIRE ALL'AVVIO DELL'ESECUZIONE (fuori Plan Mode):
- Copiare/spostare il contenuto di questo file di piano dentro NTSPJ.EShop.WebUI_CUBE/ (nome suggerito: PIANO_MIGRAZIONE_BOOTSTRAP5.md, salvo diversa indicazione dell'utente)

