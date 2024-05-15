<style> 
    body{
        text-align: justify; 
    } 

    h2{ margin: 0px; }
</style>

# `Rust::gestione_errori`
<div style='font-family: Arial; 
            text-align: right; margin-top: -20px; 
            font-size: 14px;
            border-top: 1px solid black'>
    &copy2024 <i>Carlo MIGLIACCIO</i>
</div>

Tutti i programmi che siano applicativi (lato utente) o di sistema fanno delle *computazioni* che possono essere sicuramente invalide nel senso che possono fallire. Questi fallimenti possono avere delle caratteristiche che li rende più o meno risolvibili. In particolare:
* Ci sono errori che possono verificarsi in maniera prevedibile (esempio: conversione Stringa $\to$ Intero); 
* Tipologie di errori legate a limiti del sistema (es: non riesco ad allocare più di una certa quantità di memoria).

Si può fare inoltre un'ulteriore classificazione degli errori in:
1. Errori **recuperabili**: sono quelli per cui esistono delle strategie di ripristino
2. Errori **irrecuperabili** per cui non ci sono strategie di ripristino e ci si deve per forza fermare per non andare incontro a comportamenti imprevedibili.

#### Quali possono essere le strategie da mettere in atto?
* Posso riprovare a fare l'operazione che mi ha portato all'errore dopo un po', questa strategia può funzionare nel caso in cui abbia ad esempio interazione con l'utente o se è scaduto un TimeOut di rete.
* Posso fermarmi temporaneamente; 
* Provo a fare la stessa cosa in modo diverso magari in un modo meno agile di quello che mi ha portato all'errore.

Cosa fare quando mi accorgo che c'è stato un errore? 
1. Caso 1: lancio un messaggio d'errore numerico. Esempio: associo al numero 27 il messaggio `FileNotFound`, non sapendo però qual è il file non trovato.
2. Caso 2: lancio un messaggio d'errore, ad esempio: "Non ho scritto tutto", ma anche in questa situazione non saprei cosa non ho scritto.

**Morale: servono delle informazioni aggiuntive** Sono utilizzabili in questo caso diverse strategie, ma qualunque cosa faccia le cose si complicano e non poco. Pensiamo ad esempio alla gestione di apertura e chiusura di due file. Per gestire apertura e chiusura mi servono quattro coppie di else-if per una gestione granulare. Si provi ad immaginare cosa capita se queste aperture/chiusure stessero dentro un ciclo.

Ad  un certo punto si è cominciato a pensare che un semplice codice d'errore non basta.

