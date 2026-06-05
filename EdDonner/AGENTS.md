# Kanban Project

## Requisiti di Business (Business Requirements)

- Un MVP di un'applicazione di Project Management in stile Kanban sotto forma di web app.
- La web app deve avere una sola lavagna (board).
- La lavagna ha 5 colonne fisse che possono essere rinominate.
- Ogni scheda (card) ha esclusivamente un titolo e dei dettagli.
- Interfaccia Drag and drop per spostare le schede tra le colonne.
- Possibilità di aggiungere una nuova scheda a una colonna e di eliminare una scheda esistente.
- Nessuna funzionalità aggiuntiva: nessun archivio, nessuna funzione di ricerca o filtro. Mantieni il tutto semplice.
- La priorità è una UI/UX fantastica, professionale e curata, con funzionalità estremamente semplici.
- L'applicazione deve aprirsi con dati fittizi (dummy data) già popolati per l'unica lavagna esistente dal backend.

## Dettagli Tecnici (Technical Details)

- Implementata con un backend in .NET 8 (Web API RESTful basate sui controller) per la gestione dello stato in memoria. NO MINIMAL API.
- Il backend deve essere creato in una sottodirectory chiamata `backend`.
- L'interfaccia utente deve essere sviluppata come una Single Page Application (SPA) in Vue.js 3 (Composition API con TypeScript/Vite).
- L'app Vue.js deve essere creata in una sottodirectory chiamata `frontend`.
- Nessuna persistenza dei dati (no persistence): lo stato della lavagna è mantenuto in memoria (in-memory store) sul server .NET e si resetta al riavvio.
- Nessuna gestione degli utenti (user management) per questo MVP.
- Utilizza librerie popolari e stili CSS leggeri.
- Il più semplice possibile, ma con una UI elegante.

## Schema Cromatico (Color Scheme)

- Giallo di Accento (Accent Yellow): `#ecad0a` - linee di accento, evidenziatori
- Blu Principale (Blue Primary): `#209dd7` - link, sezioni chiave
- Viola Secondario (Purple Secondary): `#753991` - pulsanti di invio, azioni importanti
- Blu Notte Scuro (Dark Navy): `#032147` - intestazioni principali
- Testo Grigio (Gray Text): `#888888` - testo di supporto, etichette

## Strategia (Strategy)

1. Scrivi un piano con criteri di successo per ogni fase, da spuntare man mano. Includi l'impalcatura del progetto (scaffolding) per la soluzione .NET e l'app Vue.js, compreso il file .gitignore unificato, e test unitari rigorosi in xUnit per i modelli C#.
2. Esegui il piano assicurandoti che tutti i criteri siano soddisfatti, esponendo gli endpoint REST necessari dal backend .NET ed effettuando il binding dei dati reattivi in Vue.js.
3. Effettua test di integrazione approfonditi con Playwright o simili per simulare il drag and drop nel browser, correggendo i difetti.
4. Considera il lavoro completato solo quando l'MVP è finito e testato, con il server Kestrel di .NET avviato, l'app Vite pronta e responsiva per l'utente.

## Standard di Codifica (Coding standards)

1. Utilizza le versioni più recenti delle librerie e approcci idiomatici aggiornati a oggi (.NET 8 C# e Vue.js 3 Script Setup).
2. Mantieni la semplicità: NON sovraprogettare (NEVER over-engineer), SEMPLIFICA SEMPRE, EVITA programmazioni difensive non necessarie. Nessuna funzionalità extra - focus sulla semplicità.
3. Sii conciso. Mantieni il file README minimale. IMPORTANTE: non usare mai emoji (no emojis ever).
