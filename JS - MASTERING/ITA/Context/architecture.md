# Contesto Architetturale

## Stack

| Livello | Tecnologia | Ruolo |
| ------- | ---------- | ----- |
| Framework | .NET 8 / ASP.NET Core (MVC / Blazor) | Framework di sviluppo backend e web |
| UI | MudBlazor / Bootstrap 5.x | Libreria componenti e interfaccia utente |
| Auth | Duende IdentityServer | Gestione centralizzata dell'autenticazione e OpenID Connect |
| Database | SQL Server (Entity Framework Core / T-SQL) | Persistenza dati, Stored Procedure e gestione relazionale |
| [Livello] | [Tecnologia] | [Ruolo] |

## Limiti del Sistema

- `[cartella]` — [Cosa possiede questa cartella e di cosa è responsabile]
- `[cartella]` — [Cosa possiede questa cartella e di cosa è responsabile]
- `[cartella]` — [Cosa possiede questa cartella e di cosa è responsabile]
- `[cartella]` — [Cosa possiede questa cartella e di cosa è responsabile]

## Modello di Memorizzazione

- **SQL Server Database**: Contiene i metadati applicativi, le tabelle relazionali, la gestione tramite Entity Framework Core e le logiche eseguite tramite T-SQL (Stored Procedure, Function).
- **File / Blob Storage**: Memorizzazione di file generati, allegati, media e artefatti di grandi dimensioni esterni al database.

## Modello di Autenticazione e Accesso

- **Autenticazione centralizzata**: Ogni applicazione si autentica tramite Duende IdentityServer (gestione unificata di utenti, ruoli e token JWT/OpenID Connect).
- **Proprietà**: Ogni progetto o entità principale fa capo a un singolo proprietario identificato tramite i claim di sicurezza.
- **Controllo degli accessi**: Solo il proprietario o gli utenti autorizzati/collaboratori tramite ruoli possono eseguire operazioni di modifica sulle risorse.

## Invarianti

1. Le logiche di business complesse o performanti a livello di database devono privilegiare l'uso di Stored Procedure o Entity Framework Core in modo pulito.
2. I controller e gli endpoint API non devono eseguire operazioni di background di lunga durata direttamente nel flusso di risposta.
3. La gestione della sicurezza e dei token deve passare obbligatoriamente attraverso Duende IdentityServer.
4. Nessun accesso diretto al database non validato o non protetto da parametrizzazione T-SQL (prevenzione SQL Injection).