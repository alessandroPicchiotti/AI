# Standard del Codice

## Generale

- Mantenere i moduli piccoli, riutilizzabili e con un unico scopo (Single Responsibility Principle).
- Risolvere le cause radice dei problemi a livello di codice, evitando stratificazioni di soluzioni temporanee (workaround).
- Separare rigorosamente la logica di business, l'accesso ai dati (Repository pattern o EF Core) e i componenti di presentazione.

## C# / .NET

- Rispettare rigorosamente le linee guida di stile e le convenzioni di naming di .NET (PascalCase per classi e metodi, camelCase per variabili locali).
- Sfruttare le funzionalità moderne di C# (.NET 8) come i nullable reference types abilitati per prevenire i riferimenti nulli non gestiti.
- Validare sempre gli input esterni ai confini del sistema (validazione dei model ASP.NET Core) prima di processarli.

## ASP.NET Core & UI

- Mantenere i Controller MVC focalizzati e leggeri, delegando la logica ai servizi applicativi.
- Strutturare i componenti Blazor e le viste Razor (.cshtml) suddividendoli in componenti atomici e manutenibili.
- Utilizzare una gestione coerente degli stati e dei layout basata su MudBlazor o Bootstrap 5.x.

## Stile e UI

- Utilizzare classi CSS utility-first o i token nativi dei framework UI scelti — evitare valori hardcoded sparsi.
- Seguire le scale di spaziatura e raggio dei bordi definite in `ui-context.md`.

## API Routes e Controller

- Validare e analizzare l'input delle richieste in ingresso prima di eseguire qualsiasi logica di business.
- Applicare i filtri di autorizzazione (`[Authorize]`) e verificare la proprietà della risorsa prima di qualsiasi operazione di modifica (Command).
- Restituire formati di risposta HTTP coerenti e prevedibili (es. DTO tipizzati).

## Dati e Storage (SQL Server & T-SQL)

- I metadati e le entità relazionali appartengono al database SQL Server.
- Utilizzare Entity Framework Core per le operazioni CRUD standard e T-SQL (Stored Procedure / Function) per elaborazioni dati complesse o ad alte prestazioni.
- Non memorizzare file binari o contenuti pesanti direttamente nelle tabelle del database; utilizzare il file/blob storage salvando nel DB solo i riferimenti/path.

## Organizzazione dei File

- `Controllers/` o `Endpoints/` — [Gestione delle richieste HTTP e rotte API]
- `Services/` o `BLL/` — [Logica di business e servizi applicativi]
- `Models/` o `Entities/` — [Modelli di dominio, DTO e configurazioni EF Core]
- `Views/` o `Components/` — [Viste Razor .cshtml o componenti UI Blazor]