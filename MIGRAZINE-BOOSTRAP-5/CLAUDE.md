# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Stack

- ASP.NET MVC 5.2.9 / .NET Framework 4.8
- Bootstrap 3.4.1, jQuery 3.5.0
- Telerik Grid 2011 (EOL — in sostituzione progressiva con DataTables.js v1.13.8)

## Build

```
"C:\Program Files\Microsoft Visual Studio\2022\Community\MSBuild\Current\Bin\MSBuild.exe" NTSPJ.EShop.WebUI_CUBE.csproj /p:Configuration=Debug /nologo /v:minimal
```

Avvio locale: Visual Studio 2022 Community → F5 (IIS Express).

Warning da ignorare sempre: `MSB3270` per `ChilkatDotNet48` — mismatch architettura MSIL/AMD64 preesistente, non causato da modifiche al codice.

## Migrazione ASPX → Razor (in corso)

`.aspx`, `.ascx` e `.Master` coesistono con `.cshtml`. La conversione è incrementale. Per le view nuove o convertite:

- `Views/_ViewStart.cshtml` imposta automaticamente `_Layout.cshtml` — non serve `@{ Layout = ... }` nelle view normali.
- Le view nelle aree (es. `Areas/Admin/Views/`) usano `Areas/Admin/Views/_ViewStart.cshtml`.
- `Views/Web.config` importa globalmente: `System.Web.Mvc`, `System.Web.Mvc.Ajax`, `System.Web.Mvc.Html`, `System.Web.Routing`, `NTSPJ`. I namespace `NTSPJ.EShop.Domain.Concrete` e `NTSPJ.EShop.Domain.Infrastructure` vanno aggiunti esplicitamente con `@using` nelle view che li richiedono.
- **Non aggiungere** `System.Web.Optimization` a `Views/Web.config` — la DLL non è referenziata nel progetto.
- I Telerik Grid nelle view Admin si sostituiscono con `<table id="...">` + DataTables init (`$('#id').DataTable({ language: { url: '/Scripts/datatables/it.json' } })`).

## JavaScript — Nts-pj.js

`Scripts/Nts-pj.js` è il file JS centrale dell'applicazione. Regole critiche:

- Il bottone `.addCarrello` usa `data-origine`: `"G"` = riga griglia tabella (`<td>`), `"C"` = card Bootstrap (`.articolo-card`), assente = `<tr>` normale.
- I selettori che cercano la riga padre devono usare `closest('tr, .articolo-card')`, non solo `closest('tr')`.
- Non modificare closures o funzioni parzialmente visibili senza leggere l'intero blocco circostante.

## Web.config — Regole strutturali

Tutti i `<dependentAssembly>` devono essere figli di `<assemblyBinding>` (dentro `<runtime>`), non figli diretti di `<runtime>`. Inserire nuovi binding redirect prima di `</assemblyBinding>`.

Chiavi `appSettings` che richiedono impostazione manuale per ogni ambiente (marcate "DA IMPOSTARE" nei commenti):
- `NtsPj_ApplicationName`, `NtsPj_UtenteBusiness`, `NtsPj_PasswordBusiness`, `NtsPj_ProfiloBusiness`
- `NtsPj_CartellaFileArchivia`, `NtsPj_CartellaFileTmp`
- SMTP: `<network>` in `<system.net><mailSettings>`

## Aree MVC

- `Areas/Admin` — back-office: Utenti, TipiDocumenti, Articoli, News, Newsletter, Reports, Ruoli
- `Areas/Faq` — FAQ pubbliche

## ArticoliController

`Controllers/ArticoliController.cs` (2000+ righe). Le azioni che servivano `View("ListaArticoli")` / `View("GrigliaArticoli")` ora tornano `View("Index")` con `ViewBag.ViewMode` impostato da `GetTipoVisArticoli(sesAcquisto)`. La partial view card è `Views/Shared/_ArticoloCard.cshtml`.
