<style> 
    body{
        text-align: justify; 
    } 

    h2{ margin: 0px; }
</style>

# `Rust::iteratori`
<div style='font-family: Arial; 
            text-align: right; margin-top: -20px; 
            font-size: 14px;
            border-top: 1px solid black'>
    &copy2024 <i>Carlo MIGLIACCIO</i>
</div>

In generale, un **iteratore** è una qualsiasi struttura dati in grado di generare una *sequenza* di valori che possono essere:
1. Estratti da una *collezione* (contenitore)
2. Essere generati in programmazione, come nel caso di un Range.

[Iterator come Behavioural Design Pattern](https://refactoring.guru/design-patterns/iterator)
Un iterator è in realtà un **design pattern** comportamentale che prevede l'esistenza di due metodi:
* `hasNext(): bool` che ritorna un booleano associato al fatto che ci sia ancora un elemento da generare o meno.
* `next(): Item` che genera il prossimo elemento e lo restituisce, tale elemento è di tipo `Item`.

Inoltre esso può contenere *ulteriori metodi* che servono per accorpare, eliminare, trasformare, aggiungere elementi all'iteratore di partenza. Quasi tutti i linguaggi di programmazione moderni offrono gli iteratori come strumento della loro libreria standard.

Gli iteratori offrono una gestione dei dati che è completamente **indipendente** dalla loro rappresentazione interna, operano inoltre in modalità <u>lazy</u> (generano il prossimo elemento solo se richiesto).
Gli iteratori si sovrappongono in modo concettuale a quello che si trova nei cicli for/while, ma - **soprattutto nelle situazioni più complesse**, quando ad esempio bisogna considerare degli elementi generati e scartarne altri, offrono maggiore *leggibilità* e *compattezza*.
Questa flessibilità è garantita dal fatto che da un iteratore, spesso, c'è la possibilità di ricavare un altro iteratore.
Negli ambienti in cui sono disponibili *più core di elaborazione* l'uso di iteratori consente una **elaborazione parallela** dei dati. 

Attraverso l'utilizzo di iteratori è possibile creare delle vere e proprie *catene di elaborazione* (quello che in Java è espresso dal concetto di `Stream`) in cui ad ogni passo si filtrano i dati, li si trasforma, li si elabora nei più disparati modi.

## Iteratori in `Rust`
In Rust un Iteratore è un qualsiasi tipo o struttura dati che implementa il tratto `std::iter::Iterator`

```rust
    trait Iterator{
        type Item; 
        fn next(&mut self)->Option<Self::Item>;
    }
```
`Item` è il tipo associato ai valori che l'iteratore produce  Il metodo principale offerto dal tratto `Iterator` è `next()` che accorpa tutte e due le funzionalità di `hasNext()` e `next()`, in quanto:
* Nel caso in cui sia possibile farlo restituisce un `Some(Item)`, poi con un `unwrap()` si può passare ad estrarre il valore; 
* Nel caso in cui non sia possibilile ritorna invece un `None`.

Un tipo si dice **iterabile** se può essere esplorato tramite un iteratore, questa caratteristica la si soddisfa tramite l'implementazione di un tratto specifico (`std::iter::IntoIterator`). Se si vuole implementare `IntoIterator` per un tipo personalizzato, bisogna implementare la  funzione `into_iter()` che restituisca una struttura dati che implementa `Iterator` che ha la  possibilità di essere scandita utilizzando il metodo `next()`.

## Iteratori e ownership
Tutte le **collezioni** (capitolo prossimo) hanno la possibilità di generare in modi diversi un **iteratore** ai suoi elementi. Esistono principalmente tre modi per farlo:
1. `iter()`, estrae dalla collezione un iteratore che restituisce oggetti di tipo `&Item` che non consumano quindi i dati generati;  
2. `iter_mut()`, estrae invece degli oggetti di tipo `&mut Item` che permettono di modificare il dato generato.
3. `into_iter()`, consuma la collezione e crea un iteratore che restituisce oggetti di tipo `Item` estraendoli dal contenitore.

## Adattatori e Consumatori
![Immagine](/img/iteratori.png)

La porzione di codice che, ricevendo gli item di un iteratore ne genera un altro è detto **Adattatore**, quello che invece consuma un iteratore e produce un risultato prende il nome di **Consumatore**.
In particolare possono essere combinati insieme più adattatori in *catene più o meno lunghe* al cui termine occorre mettere un consumatore che calcoli un aggregato sui dati trasformati. Tutti gli iteratori generati nei tratti intermedi della catena, operano ugualmente in modalità **lazy**, nel senso che non invocano il metodo `next()` fino a quando non è un consumatore a farlo.

#### Adattatori usati di frequente
1. `map()` esegue la chiusura ricevuta come argomento su ogni valore della collezione 
2. `filter()` esegue la chiusura ricevuta come parametro sugli elementi dell'iteratore di ingresso, genera un iteratore di uscita in cui ci sono solo gli elementi per cui applicata la chiusura il risultato è `true`.

#### Consumatori usati di frequente
1. `forEach()` esegue la chiusura passata come parametro su ogni elemento consumandone il valore; 
2. `collect()` trasforma l'iteratore in una collezione i cui elementi sono quelli generati dall'iteratore.