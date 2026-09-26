# Contesto UI

## Tema

Il linguaggio visivo è strutturato per un'applicazione enterprise moderna, con supporto a temi coerenti con l'uso di MudBlazor o Bootstrap 5.x, garantendo un'esperienza utente pulita, responsive e professionale.

## Colori

I colori e i token di stile sono gestiti tramite le classi e le proprietà dei framework UI scelti (MudBlazor / Bootstrap), evitando valori esadecimali sparsi nel codice.

| Ruolo | Variabile / Classe di Riferimento | Valore / Token |
| ----- | --------------------------------- | -------------- |
| Sfondo pagina | `--bg-base` / classi Bootstrap/MudBlazor | Standard |
| Superficie | `--bg-surface` | Standard |
| Testo primario | `--text-primary` | Standard |
| Testo disattivato | `--text-muted` | Standard |
| Accento primario | `--accent-primary` (Primary Color) | Standard |
| Bordo | `--border-default` | Standard |
| Errore | `--state-error` (Error Color) | Standard |
| Successo | `--state-success` (Success Color) | Standard |

## Tipografia

| Ruolo | Font | Variabile |
| ----- | ---- | --------- |
| Testo UI | Roboto / Inter / Segoe UI | `--font-sans` |
| Codice/mono | Cascadia Code / Consolas | `--font-mono` |

## Raggio dei Bordi (Border Radius)

Gestito nativamente dai temi dei componenti MudBlazor o dalle classi di utilità di Bootstrap (`rounded`).

## Componenti e Librerie

- **MudBlazor** o **Bootstrap 5.x** combinati con viste Razor (`.cshtml`) o componenti Blazor in base al sotto-modulo applicativo.
- Utilizzo dei componenti nativi della libreria scelta per mantenere coerenza visiva ed evitare componenti scritti da zero senza standard.

## Pattern di Layout

- Layout responsivo con barra di navigazione superiore e/o menu laterale fisso.
- Utilizzo di tabelle dati avanzate, form validati e finestre modali integrate nei rispettivi framework UI.

## Icone

- **MudBlazor Icons** o **Font Awesome** / **Bootstrap Icons** per una coerenza visiva basata su tratti puliti e dimensioni standardizzate.