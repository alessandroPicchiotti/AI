Aggiungi vicino al top di CLAUDE.md sotto una nuova sezione 
'## Convenzioni Progetto' o '## Posizionamento File Documentazione'.\n\n## Posizionamento File Documentazione
Crea sempre CLAUDE.md, CLAUDE.local.md, e qualunque documento generato di piano/catalogo (es. PIANO_*.md, PERSONALIZZAZIONI_*.md) nella RADICE DEL PROGETTO (la cartella contenente il .csproj/.sln), 
mai in una cartella parent, soluzione, o home. Se incerto quale cartella è la radice del progetto, chiedi prima di scrivere.
Aggiungi sotto una sezione 
'## Standard Codifica' o nuova sezione '## Encoding File' in CLAUDE.md.\n\n## Encoding File
Tutti i file .cs, .cshtml, .aspx e .sql devono essere scritti come UTF-8 CON BOM. Dopo creare o editare qualunque file contenente caratteri italiani accentati (à è ì ò ù, 'Sì'), 
verifica l'encoding e ri-controlla il testo renderizzato per mojibake.
Aggiungi sotto una sezione 
'## Frontend / JavaScript' in CLAUDE.md.\n\n## Regole Legacy JavaScript
Questo codebase sovrascrive il globale `$` su alcune pagine. Non dare per scontato che `$` sia jQuery: usa `jQuery(...)` esplicitamente o cattura un alias locale dentro un IIFE. 
Inoltre nota che DataTables è v2.x — usa selettori/API v2, non quelli 1.x (`dataTables_wrapper`, `fnXxx`). Verifica che i percorsi script bundle referenziati effettivamente esistano prima di aggiungere un bundle.
Aggiungi sotto una sezione 
'## Stile di Lavoro' o '## Pianificazione' in CLAUDE.md.\n\n## Pianificazione e Edit Distruttivi
Quando ti è chiesto di pianificare, NON modificare documenti di piano/spec esistenti — scrivi la nuova proposta in un file separato e mantieni l'originale intatto. 
Quando l'utente ha già indicato un file/linea/controller specifico, usalo invece di chiedere di nuovo o ri-esplorare.
Aggiungi sotto una sezione 
'## Comandi Build & Test' in CLAUDE.md.\n\n## Verifica Build
Dopo qualunque cambio multi-file a codice C#/Razor, esegui la build soluzione (MSBuild/dotnet build) 
e riporta il risultato prima di dichiarare il task fatto. Non rivendicare mai completamento su codice che non ha compilato.
