Voglio un riferimento architettura duraturo per questa soluzione legacy .NET MVC così le future sessioni non necessitano di esplorazione lunga.

Lancia 6 subagent IN PARALLELO in un singolo messaggio. Ognuno è READ-ONLY (Read/Grep/Glob solo, no edit) e deve scrivere esattamente un file markdown in /docs/architecture/:

1. DATA-LAYER.md — pattern repository/entity manager, come query sono scritte, convenzioni primary-key handling, dove SQL raw vive.
2. GRIDS.md — ogni grid nell'app: Telerik vs versione DataTables, comportamento pager/query-string, pitfall noti (Html.CheckBox hidden fields, page reset su page-size change).
3. VIEWS.md — split .aspx vs .cshtml, layout/bundle wiring, quali bundle script esistono e quali percorsi sono effettivamente validi su disco.
4. SESSION-STATE.md — ogni session flag/key, chi lo setta, chi lo legge, e quali view rendono condizionalmente su esso.
5. LOCALIZATION.md — come label/Guids funzionano end to end, più gli step esatti per aggiungere una nuova string localizzata.
6. JS-CONVENTIONS.md — uso jQuery, dove il globale $ viene oscurato, pattern event-binding, pattern validation.

Regole: NON generare qualunque agent il cui job sia aspettare o polling. Ogni doc deve citare path file concreti e numeri linea. Quando tutti e 6 finiscono, scrivi /docs/architecture/INDEX.md riassumendoli e aggiungi un pointer a esso da CLAUDE.md.
