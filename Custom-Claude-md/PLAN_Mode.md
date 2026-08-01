Aggiungi una sezione 'Protocollo Pianificazione' a CLAUDE.md e seguilo da adesso in poi:

PRIMA di scrivere qualunque codice per una feature o refactor:
1. Esplora, poi scrivi un piano a /docs/plans/<FEATURE>-plan.md contenente: (a) il layer architettura esatto che cambierai e PERCHÉ, più i layer alternativi che hai rigettato; (b) una tabella file-by-file di ogni cambio; (c) DB/SQL script changes; (d) come verificherai (build + Chrome flow); (e) una lista numerata di domande aperte.
2. MAI sovrascrivere o riscrivere un documento di piano/analisi esistente. Se l'approccio cambia, crea <FEATURE>-plan-v2.md e lascia v1 intatto.
3. Batch TUTTE le domande chiarificatrici in quella lista numerata alla volta. Non chiedere una alla volta durante l'implementazione. Se ti ho già dato un file path o numero linea, usalo — non chiedere di nuovo.
4. Aspetta il mio approvazione esplicito, poi crea una lista task ed esegui il PIANO INTERO autonomamente: tutti i layer, SQL script, build, e verifica browser. Fermati presto solo se colpisci un blocker vero.
5. Finisci con un riassunto diff mappato al tavolo piano, marcando ogni riga FATTO / DEVIATO (con ragione) / SKIPPED.

Conferma che hai aggiunto questo, poi applicalo alla prossima feature che descrivo.
