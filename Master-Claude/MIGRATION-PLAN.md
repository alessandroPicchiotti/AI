# Piano di Migrazione — NTSPJ.EShop → ASP.NET Core + BUS API

**Approccio:** Side-by-side  
**Stack attuale:** ASP.NET MVC 5 / .NET Framework 4.8  
**Target:** ASP.NET Core / .NET 10

Spunta ogni task con `[x]` quando completato.

---

## Architettura target

```
┌─────────────────────────────────────┐
│  NTSPJ.EShop.WebUI_CORE  (.NET 10)  │  ← nuovo progetto
│  (MVC + Razor + Identity)           │
└────────────────┬────────────────────┘
                 │ HTTP (HttpClient tipizzato)
┌────────────────▼────────────────────┐
│  NTSPJ.Eshop.BUS_EXP  (.NET Fw 4.8)│  ← resta su Framework
│  (ASP.NET Web API 2 — nuovo layer)  │
└────────────────┬────────────────────┘
                 │ assembly reference
         DLL ERP Business (Framework)
```

> `NTSPJ.EShop.Domain` — valutare dual-target `.NET Standard 2.0` per condividerlo
> tra i due progetti senza duplicare le entità.  
> `NTSPJ.EShop.WebUI_CUBE` — rimane attivo durante tutta la migrazione, smontato solo a fine percorso.

---

## FASE 0 — Preparazione

- [ ] Confermare versione target: **.NET 10**
- [ ] Creare branch git dedicato alla migrazione
- [ ] Inventario endpoint: route, HTTP method, auth rule, response shape per ogni controller → `baseline.md`
- [ ] Inventario pipeline: HttpModules, HttpHandlers, Global.asax events
- [ ] Inventario feature: quali satellite skill caricare (vedi `.claude/commands/migrating-aspnet-framework-to-core.md`)
- [ ] Backup del `Web.config` corrente
- [ ] Valutare se `NTSPJ.EShop.Domain` può diventare `.NET Standard 2.0` (nessuna dipendenza da `System.Web` o DLL ERP)

---

## FASE 1 — BUS_EXP → Web API (rimane .NET Framework)

> BUS_EXP non migra a .NET perché dipende dalle DLL ERP Business che rimangono su Framework.
> Diventa invece un Web API 2 auto-ospitato che espone la business logic via HTTP.

- [ ] Aggiungere pacchetto `Microsoft.AspNet.WebApi.SelfHost` o convertire BUS_EXP in un progetto `ASP.NET Web API 2` separato (`NTSPJ.Eshop.BUS_API`)
- [ ] Decidere porta di ascolto dell'API (es. `http://localhost:5100`)
- [ ] Definire contratti API (DTO): un DTO per ogni entità/operazione esposta al WebUI
- [ ] Implementare i controller API per ogni servizio business attualmente usato dal WebUI:
  - [ ] Endpoint autenticazione / gestione utenti
  - [ ] Endpoint articoli / categorie / famiglie-gruppi-sottogruppi
  - [ ] Endpoint carrello / ordini / nuovo documento
  - [ ] Endpoint clienti / destinazioni
  - [ ] Endpoint fatture / estratto conto / distinte incasso
  - [ ] Endpoint news / newsletter
  - [ ] Endpoint download / business file
  - [ ] Endpoint lead / GDPR
  - [ ] Endpoint report (Admin)
  - [ ] Endpoint gestione utenti / ruoli (Admin)
- [ ] Aggiungere autenticazione API (API Key o JWT) per proteggere gli endpoint da accessi esterni
- [ ] Swagger / OpenAPI per documentare i contratti (Swashbuckle per Web API 2)
- [ ] `dotnet build` (o MSBuild) BUS_API → 0 errori
- [ ] Smoke test: chiamata HTTP a un endpoint di test risponde 200
- [ ] **Gate:** BUS_API è raggiungibile e risponde correttamente a tutte le operazioni business

---

## FASE 2 — Scaffold Nuovo Progetto Core (.NET 10)

- [ ] Creare `NTSPJ.EShop.WebUI_CORE` con SDK-style `.csproj` target `net10.0`
- [ ] Configurare YARP proxy verso il vecchio `WebUI_CUBE` (fallback durante la migrazione)
- [ ] Aggiungere project reference a `NTSPJ.EShop.Domain` (se dual-target Standard 2.0, altrimenti duplicare solo i DTO necessari)
- [ ] Registrare `HttpClient` tipizzato verso `BUS_API` in `Program.cs`
- [ ] `dotnet build` → 0 errori
- [ ] `dotnet run` → smoke test `/health` risponde 200
- [ ] Aggiungere `app.MapControllers()` al pipeline
- [ ] **Gate:** WebUI_CORE si avvia, fa proxy via YARP e può chiamare BUS_API

---

## FASE 3 — Configurazione

- [ ] Migrare 140+ chiavi `<appSettings>` → `appsettings.json`
- [ ] Migrare 3 connection strings (ConnString, ConnStringArcProc, ConnStringBF) → `appsettings.json`
- [ ] Creare `appsettings.Development.json` dai transform di `Web.Debug.config`
- [ ] Spostare credenziali (`NtsPj_PasswordBusiness`, ecc.) → User Secrets (dev) o variabili d'ambiente
- [ ] Configurare `BaseAddress` di BUS_API in `appsettings.json`
- [ ] Wire `IConfiguration` in `Program.cs`
- [ ] **Gate:** tutti i valori di configurazione leggibili via `IConfiguration`

---

## FASE 4 — Dependency Injection (Ninject → built-in)

- [ ] Rimuovere Ninject e `NinjectControllerFactory` dal nuovo progetto
- [ ] Registrare `IHttpContextAccessor`
- [ ] Registrare `HttpClient` tipizzato per BUS_API con `builder.Services.AddHttpClient<IBusApiClient>()`
- [ ] Registrare model binders custom: `NTSPJSettBinder`, `IBuzEntityManagerBinder`, `SessioneAcquistoBinder`
- [ ] Registrare validator providers custom: `DestinazioneValidator`, `ClienteValidator`, `DistIncTestaValidator`, `LeadValidator`
- [ ] **Gate:** app si avvia, DI container risolve senza errori

---

## FASE 5 — HttpModules → Middleware

- [ ] Convertire `CultureModule` (`CultureInfoHttpModule.cs`) → middleware ASP.NET Core
- [ ] Convertire `StoricizUltimiAcquistiHttpModule` → middleware ASP.NET Core
- [ ] Registrare i middleware nel pipeline nell'ordine corretto
- [ ] Scegliere alternativa a ELMAH (Serilog + Seq, NLog, o elmah.io)
- [ ] Configurare error logging nel nuovo progetto
- [ ] **Gate:** pipeline si comporta come baseline per richieste non autenticate

---

## FASE 6 — Migrazione Controller

> Ogni controller del WebUI_CORE chiama BUS_API via `IBusApiClient` invece di dipendere direttamente da BUS_EXP.  
> Ordine: più semplici prima, auth-dipendenti dopo la Fase 7.

### Area FAQ (2 controller — senza auth)

- [ ] `FAQ/ArchiviController`
- [ ] `FAQ/GestFaqController`

### Controller semplici root (senza `[Authorize]`)

- [ ] `ErrorController`
- [ ] `HomeController`
- [ ] `DownloadController`
- [ ] `DownloadTuttiController`
- [ ] `NewsController`
- [ ] `RecuperaPasswordController`
- [ ] `RegistrazioneUtenteController`
- [ ] `GDPRController`

### Controller business (media complessità)

- [ ] `ArticoliController`
- [ ] `CategorieProdottiController`
- [ ] `FamGruSubgruController`
- [ ] `NewsletterController`
- [ ] `LeadController`
- [ ] `BusinessFileController`

### Controller business (alta complessità)

- [ ] `CarrelloController`
- [ ] `ClientiController`
- [ ] `NuovoDocController`
- [ ] `AccettazioneController`
- [ ] `FattureController`
- [ ] `EstrattoContoController`
- [ ] `DistinteIncassoController`

### Controller auth-dipendenti root _(DOPO Fase 7)_

- [ ] `BaseController` ← migrare prima degli altri derivati
- [ ] `AccountController`

### Area ADMIN _(DOPO Fase 7 — richiedono `[Authorize(Roles="Admin")]`)_

- [ ] `Admin/MenuController`
- [ ] `Admin/RuoliController`
- [ ] `Admin/UtentiController`
- [ ] `Admin/GestioneNewsController`
- [ ] `Admin/NewsletterGestioneController`
- [ ] `Admin/NewsletterIscrittiController`
- [ ] `Admin/TipiDocumentiController`
- [ ] `Admin/ArticoGowController`
- [ ] `Admin/AssArtiTipiArtController`
- [ ] `Admin/ReportsController`

---

## FASE 7 — Autenticazione (Forms Auth + SqlMembership → ASP.NET Core Identity)

> ⚠️ Caricare satellite skill `migrating-mvc-authentication` prima di iniziare questa fase.

- [ ] Analizzare schema database Membership attuale (`aspnet_Users`, `aspnet_Roles`, `aspnet_UsersInRoles`, ecc.)
- [ ] Creare migration EF per schema ASP.NET Core Identity
- [ ] Script di migrazione dati utenti/ruoli dal vecchio schema al nuovo
- [ ] Configurare `AddAuthentication().AddCookie()` in `Program.cs`
- [ ] Configurare `AddAuthorization()` con policy equivalenti ai ruoli attuali
- [ ] Migrare `AccountController` (Login, Logout, Register, RecuperaPassword)
- [ ] Testare: login e logout funzionano end-to-end
- [ ] Testare: controller con `[Authorize]` proteggono correttamente le route
- [ ] **Gate:** flussi auth completi, inventario auth dalla baseline verificato

---

## FASE 8 — Views e Asset Statici

> ⚠️ Caricare satellite skill `migrating-mvc-razor-views` prima di iniziare questa fase.

- [ ] Convertire Master pages → Layout Razor: `Site.Master` → `_Layout.cshtml`, `SiteNoDoc.Master` → `_LayoutNoDoc.cshtml`, `PopUp.Master` → `_LayoutPopup.cshtml`
- [ ] Creare `_ViewImports.cshtml` con namespace Tag Helpers
- [ ] Creare `_ViewStart.cshtml`
- [ ] Convertire le 97 view ASPX → Razor (`.cshtml`) — una area alla volta
- [ ] Convertire User Controls (`.ascx`): `HeadLogOn`, `LogOnUserControl`, `UC_HiddenAcquisto`, `UC_InsManualeArticolo`, `UC_RicercaArticoli` → Partial Views o ViewComponents
- [ ] Sostituire `Html.ActionLink`, `Html.BeginForm` e altri HtmlHelpers → Tag Helpers (`<a asp-*>`, `<form asp-*>`)
- [ ] Spostare `Content/` e `Scripts/` → `wwwroot/`
- [ ] Aggiungere `app.UseStaticFiles()` a `Program.cs`
- [ ] Scegliere strategia bundling (LibMan, npm pipeline, o riferimenti diretti)
- [ ] **Gate:** tutte le view rendono senza errori, asset statici si caricano

---

## FASE 9 — Handler Speciali

- [ ] Telerik Reporting: verificare disponibilità versione ASP.NET Core (`Telerik.Reporting.AspNetCore`)
- [ ] Se disponibile: migrare `Telerik.ReportViewer.axd` nel nuovo progetto Core
- [ ] Se non disponibile: esporre i report via BUS_API e servirli via proxy da WebUI_CORE
- [ ] ELMAH `elmah.axd`: sostituito con la soluzione scelta in Fase 5

---

## FASE 10 — Cleanup e Verifica Finale

- [ ] Rimuovere tutti i `#if NETFRAMEWORK` rimasti nel nuovo progetto
- [ ] Verificare 0 riferimenti `System.Web` nel nuovo progetto (`dotnet list package`)
- [ ] Soluzione compila con 0 errori e 0 warning legati alla migrazione
- [ ] Tutti gli endpoint dell'inventario baseline (`baseline.md`) rispondono correttamente
- [ ] Flussi auth funzionano end-to-end (login, logout, register, recupera password)
- [ ] Tutti i valori di configurazione leggibili via `IConfiguration`
- [ ] Asset statici si caricano, nessun `404` su CSS/JS
- [ ] Chiamate a BUS_API funzionano per tutti i flussi business (carrello, ordini, clienti, fatture)
- [ ] Test di regressione sulle funzionalità principali
- [ ] **Post-migrazione (manuale):** rimuovere `WebUI_CUBE` dalla solution dopo conferma in produzione

---

## Note

- `WebUI_CUBE` e `BUS_EXP` rimangono attivi durante tutta la migrazione — nessuna rimozione anticipata.
- `BUS_EXP` (ora BUS_API) **non migrerà mai a .NET** — dipende da DLL ERP Framework-only.
- Per ogni controller, verificare che il corrispondente endpoint su BUS_API esista prima di portare il controller in WebUI_CORE.
- I satellite skill sono in `.claude/commands/` — caricarli nel momento indicato, non tutti in anticipo.
