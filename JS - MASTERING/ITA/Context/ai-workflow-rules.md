# Regole del Flusso di Lavoro IA

## Approccio

Costruire questo progetto in modo incrementale utilizzando un flusso di lavoro basato su specifiche. I file di contesto definiscono cosa costruire, come costruirlo e lo stato attuale dei progressi. Implementare sempre seguendo queste specifiche — non inferire o inventare comportamenti da zero.

## Regole di Scoping

- Lavorare su un'unità di funzionalità alla volta (es. una singola entity, una Stored Procedure o un componente UI).
- Preferire incrementi piccoli e verificabili a modifiche speculative di grandi dimensioni.
- Non combinare confini di sistema non correlati in un singolo passaggio di implementazione.

## Quando Suddividere il Lavoro

Suddividere un passaggio di implementazione se combina:
- Modifiche all'interfaccia utente (Razor/Blazor) e modifiche alla logica di database T-SQL.
- Più endpoint API o controller non correlati.
- Comportamenti di business non chiaramente definiti nei file di contesto.

Se una modifica non può essere verificata end-to-end rapidamente, l'ambito è troppo ampio — suddividerla.

## Gestione dei Requisiti Mancanti

- Non inventare comportamenti di prodotto non definiti nei file di contesto.
- Se un requisito è ambiguo, risolverlo nel file di contesto pertinente prima di procedere all'implementazione.
- Se un requisito è mancante, aggiungerlo come domanda aperta in `progress-tracker.md` prima di continuare.

## File Protetti

Non modificare i seguenti elementi se non espressamente indicato:
- File di configurazione interna di Duende IdentityServer generati o librerie di terze parti protette.

## Mantenere la Documentazione Sincronizzata

Aggiornare il file di contesto pertinente ogni volta che l'implementazione modifica:
- L'architettura o i confini del sistema.
- Le decisioni sul modello di storage o script SQL Server.
- Le convenzioni o gli standard del codice .NET.
- L'ambito delle funzionalità.

## Prima di Passare all'Unità Successiva

1. L'unità corrente funziona end-to-end all'interno del suo ambito definito.
2. Nessuna invariante definita in `architecture.md` è stata violata.
3. `progress-tracker.md` riflette il lavoro completato.
4. La compilazione della soluzione (`dotnet build`) viene completata senza errori.