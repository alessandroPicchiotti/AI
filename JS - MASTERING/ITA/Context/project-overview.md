# [Nome del Progetto]

## Panoramica

Applicazione enterprise basata su ecosistema .NET 8, con persistenza su SQL Server e autenticazione centralizzata tramite Duende IdentityServer, progettata per garantire elevate performance, scalabilità e manutenibilità.

## Obiettivi

1. Garantire una separazione netta tra i livelli applicativi sfruttando le convenzioni standard di ASP.NET Core.
2. Centralizzare la sicurezza e l'identità degli utenti tramite Duende IdentityServer a supporto di più applicazioni.
3. Ottimizzare le performance delle query e delle elaborazioni dati complesse mediante l'uso mirato di SQL Server e T-SQL.

## Flusso Utente Principale

1. L'utente accede all'applicazione e viene reindirizzato a Duende IdentityServer per l'autenticazione.
2. A seguito dell'autenticazione riuscita, il token di sicurezza viene validato dall'applicazione client/web.
3. L'utente naviga nell'interfaccia (Blazor / viste MVC) interagendo con i servizi backend protetti.
4. Le richieste eseguono le operazioni sul database SQL Server tramite Entity Framework Core o Stored Procedure dedicate.

## Funzionalità

### [Categoria di Funzionalità Uno]
- Gestione e visualizzazione dei dati tramite interfaccia web reattiva.
- Integrazione con i servizi di autenticazione centralizzati.

### [Categoria di Funzionalità Due]
- Elaborazione dati lato database tramite logiche T-SQL ottimizzate.

## Ambito (Scope)

### Incluso nell'Ambito
- Sviluppo di API REST e/o applicazioni Web .NET 8.
- Configurazione e integrazione di Duende IdentityServer.
- Progettazione schema database SQL Server e implementazione logiche T-SQL.

### Escluso dall'Ambito
- Sistemi di autenticazione legacy esterni non standard.

## Criteri di Successo

1. Un utente autenticato tramite Duende IdentityServer può accedere correttamente alle risorse protette dell'applicazione.
2. Le interazioni con SQL Server (tramite EF Core e T-SQL) restituiscono i dati in modo corretto ed efficiente.
3. La compilazione della soluzione .NET 8 risulta pulita e priva di warning bloccanti.