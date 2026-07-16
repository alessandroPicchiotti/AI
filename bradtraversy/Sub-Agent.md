
**Sotto-agente Personalizzato**

Crea un sotto-agente a livello di progetto chiamato code-reviewer. Dovrebbe esaminare le attuali modifiche non committate (non salvate) in t... *[il testo originale qui è troncato]*

* Codice morto o importazioni (*imports*) non utilizzate
* Istruzioni `console.log` lasciate nel codice
* Proprietà `key` mancanti nelle liste in C#
* Mancanze di accessibilità (testo `alt` mancante, etichette `aria` mancanti sui pulsanti con icona)
* Valori scritti fissi nel codice (*hardcoded*) che dovrebbero essere variabili d'ambiente o costanti
* Qualsiasi cosa che violi i pattern descritti nel file `CLAUDE.md`

Dovrebbe produrre un report in formato markdown con i risultati, raggruppati per gravità. NON dovrebbe apportare alcuna modifica - solo... *[anche qui il testo è troncato]*

Attivalo quando dico "review my code", "run the reviewer", o `/code-reviewer`.
