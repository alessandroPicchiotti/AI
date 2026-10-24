# Controllo delle skill: le regole di Anthropic

Questo file serve a Claude per controllare le tue skill con le regole della guida ufficiale di Anthropic e sistemarle. Allegalo in Claude Code o in Claude, scrivi dove stanno le tue skill e segui quello che ti propone. Non cambia niente finché non approvi.

## Istruzioni per Claude

Controlla le skill nella cartella che ti indico seguendo le regole qui sotto: prima le 7 principali, poi le 5 aggiuntive, poi i controlli veloci. Per ogni skill guarda il file SKILL.md e tutti i file a cui rimanda.

Prima di cambiare qualsiasi cosa, fammi vedere cosa cambieresti. Non toccare nessun file finché non approvo le correzioni.

## Le 7 regole principali

### Regola 1. Un indice in cima ai file lunghi
Ogni file di supporto che supera le 100 righe apre con un indice delle sue sezioni, nello stesso ordine in cui compaiono. Serve perché a volte Claude legge solo l'inizio dei file lunghi: con l'indice sa cosa c'è più giù.
Come controlli: conta le righe di ogni file; per quelli oltre le 100, guarda se nelle prime righe c'è un indice.
Come sistemi: aggiungi in cima un titolo "Indice" con una riga per ogni sezione.

### Regola 2. La libertà giusta, e le regole che non si rompono
Il dettaglio di ogni passaggio dipende da quanto costa sbagliarlo. Dove un errore costa (soldi, dati cancellati, messaggi ai clienti) serve un comando o uno script preciso. Dove ci sono tanti modi giusti basta l'obiettivo. Nella stessa skill possono convivere passaggi liberi e passaggi bloccati.
Come controlli: per ogni passaggio chiediti "se Claude lo fa in un altro modo, cosa succede?". Segnala i passaggi rischiosi scritti in modo vago e quelli semplici bloccati senza motivo.
Poi cerca le regole scritte come "mai", "sempre" o "obbligatorio" che devono valere ogni singola volta. Per ognuna dimmi cosa costerebbe sbagliarla una volta e, se sto usando Claude Code, se andrebbe trasformata in un hook: un controllo che parte da solo prima di un'azione. L'hook si può dichiarare dentro la skill stessa (campo `hooks` nell'intestazione) e resta attivo per tutta la sessione da quando la skill viene usata. Dimmi prima di quale azione partirebbe. Le altre regole restano scritte, con il motivo accanto.

### Regola 3. Scritta per il modello che usi, senza istruzioni da modello vecchio
Una skill funziona bene quanto il modello che la usa. Le skill scritte per i modelli precedenti sono spesso troppo rigide e con i modelli nuovi peggiorano il risultato.
Come controlli: segnala maiuscole insistite, elenchi rigidi dove l'ordine non conta e paragrafi che spiegano cose che Claude sa già (per esempio cos'è una fattura).
Come sistemi: proponi i tagli come prova, non come cancellazione definitiva: lo stesso lavoro va provato con e senza, e si tiene la versione che va meglio. Nella skill restano le informazioni che Claude non può sapere da solo: prezzi, clienti, regole interne.
La descrizione: deve dire cosa fa la skill e quando usarla, scritta in terza persona ("Prepara i preventivi. Usala quando...", mai "Ti aiuto con..."). È la parte che Claude legge di tutte le skill per scegliere quale aprire: segnala le descrizioni vaghe o senza il "quando".
Se uso la skill con più modelli, ricordami di provarla su ognuno: Haiku ha bisogno di più indicazioni, Sonnet di istruzioni chiare e senza giri, Opus di non sentirsi spiegare troppo. Se Haiku salta un passaggio, quel passaggio va scritto più chiaro o trasformato in uno script.

### Regola 4. Il file principale corto, tutto a un passo
SKILL.md resta sotto le 500 righe e fa da indice. I dettagli stanno in file separati, ognuno collegato direttamente da SKILL.md. Se la skill copre più aree, un file per area. Le istruzioni più importanti stanno in cima a SKILL.md, perché dopo una conversazione lunga Claude Code conserva solo l'inizio di ogni skill.
Come controlli: conta le righe di SKILL.md (senza l'intestazione), elenca i file a cui rimanda e segnala i file raggiungibili solo passando da un altro file.
Come sistemi: sposta in un file a parte ciò che serve solo ad alcuni lavori; collega direttamente da SKILL.md ogni file che oggi si raggiunge solo a catena; porta in cima le regole che non devono mai sfuggire.

### Regola 5. Una lista di controllo con il "torna indietro"
I lavori con molti passaggi in un ordine preciso hanno i passaggi numerati, una lista che Claude copia nella risposta e spunta man mano e, sotto ogni controllo, l'indicazione di dove tornare se il controllo non passa. Se l'ordine non conta, la lista non serve.
Come controlli: trova i lavori con più di tre o quattro passaggi e guarda se c'è la lista e cosa succede quando un controllo fallisce.
Come sistemi: numera i passaggi, aggiungi la lista da copiare e una riga tipo "se un totale non torna, torna al passo 2".

### Regola 6. Claude ricontrolla il suo lavoro
Dove la qualità conta (testi per i clienti, documenti, dati) la skill finisce con un controllo: si confronta il risultato con una guida o una lista, si correggono gli errori e si ricontrolla finché tutto passa. Se lo stile conta, la skill contiene due o tre esempi di lavoro fatto bene, con la richiesta e il risultato.
Come controlli: guarda se c'è un controllo finale e se un errore rimanda Claude a correggere. Guarda se, dove lo stile conta, ci sono esempi.
Come sistemi: aggiungi una breve lista di controllo, il passaggio di correzione e la frase "finisci solo quando tutti i controlli passano"; dove serve, aggiungi due o tre coppie richiesta e risultato. Quando un lavoro sbaglia per un motivo che la guida non prevede, proponimi una regola nuova da aggiungere: la approvo io.

### Regola 7. Pensa a chi la userà
Ogni pacchetto o programma che la skill usa è scritto in SKILL.md con il comando per installarlo. Niente percorsi che esistono solo sul mio computer, e sempre la barra / nei percorsi.
Come controlli: elenca pacchetti, programmi e script usati dalla skill e cerca percorsi personali o con la barra rovesciata.
Come sistemi: aggiungi in SKILL.md una sezione "Serve" con il comando di installazione di ciascuno e sostituisci i percorsi personali con percorsi dentro la cartella della skill.

## Le 5 regole aggiuntive

### Regola 8. Modelli che dicono quanto sono rigidi
Quando la skill dà un modello da seguire (un report, un formato di risposta), deve dire se va usato identico ("usa sempre esattamente questa struttura") o se è un punto di partenza ("questa è la forma di base, adattala se serve").
Come controlli e sistemi: trova i modelli senza questa indicazione e proponi la frase giusta per ciascuno.

### Regola 9. Il bivio scritto
Se un lavoro cambia strada a seconda dei casi (creare una cosa nuova oppure modificarne una esistente), la skill lo dice in chiaro: "se devi creare, segui questi passaggi; se devi modificare, segui questi".
Come controlli e sistemi: trova i lavori con più casi possibili e scrivi il bivio all'inizio.

### Regola 10. Prima le prove, poi la skill
Ogni skill importante ha almeno tre prove: tre richieste reali con il risultato che ti aspetti. Si provano senza la skill (per vedere dove Claude sbaglia da solo) e con la skill, in una conversazione nuova, perché quello che resta in memoria da quando hai scritto la skill nasconde i buchi delle istruzioni.
Come controlli e sistemi: segnala le skill senza prove e proponi tre richieste di prova per ciascuna, prese dal lavoro vero.

### Regola 11. Non chiedere a Claude di scrivere il suo ragionamento
Sui modelli nuovi, una skill che chiede a Claude di trascrivere o spiegare il proprio ragionamento nella risposta può far rifiutare la richiesta. Si può chiedere una breve spiegazione della risposta o un riassunto di cosa ha fatto.
Come controlli e sistemi: cerca frasi come "ragiona passo passo e scrivi il tuo ragionamento" o "mostra il tuo pensiero" e trasformale in "dammi la risposta e una breve spiegazione".

### Regola 12. Errori che dicono cosa non va
Se la skill usa degli script, i loro messaggi di errore devono dire cosa manca e cosa c'è invece (per esempio "Campo 'data_firma' non trovato. Campi disponibili: ..."), non un generico "Errore".
Come controlli e sistemi: leggi i messaggi di errore degli script e proponi la versione che nomina il problema.

## I controlli veloci

- Niente informazioni che scadono (date, "dopo agosto usa il nuovo sistema"): il metodo attuale in chiaro, quello vecchio in una sezione a parte.
- Sempre lo stesso termine per la stessa cosa, dall'inizio alla fine.
- Una soluzione predefinita invece di cinque alternative; al massimo una seconda per il caso particolare.
- File con nomi che dicono cosa contengono ("regole-sconti.md", non "doc2.md").
- Se la skill usa strumenti collegati (MCP), il nome completo dello strumento, con il nome del servizio davanti.
- Script: deve essere chiaro se Claude li deve eseguire o leggere.

## Come mi rispondi

1. Non cambiare ancora niente.
2. Parti da una tabella: una riga per skill, quante regole non rispetta, il problema più grave.
3. Poi skill per skill, regola per regola: rispettata, non rispettata o non applicabile, con il file e la riga come prova e la correzione esatta.
4. Chiudi con un elenco numerato di tutte le correzioni, dalla più importante.

Se le skill sono tante, parti dalle cinque che uso di più o da quelle che toccano soldi, clienti o dati da cancellare.

Poi aspetta me. Applica solo le correzioni che approvo indicando i numeri. Dopo averle applicate ricontrolla le skill modificate con le stesse regole e, se qualcosa ancora non passa, torna indietro e sistemalo. Finisci con l'elenco di cosa è cambiato, file per file.

## Fonti

- Guida ufficiale di Anthropic per scrivere le skill: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices
- Documentazione delle skill in Claude Code (hook, skill dopo una conversazione lunga, prove con e senza la skill): https://code.claude.com/docs/en/skills
- Guida di Anthropic per Claude Fable 5 (skill troppo rigide, ragionamento nella risposta): https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5
