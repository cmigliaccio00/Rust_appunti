<style> 
    body{
        text-align: justify; 
    } 

    h2{ margin: 0px; }
</style>

# `Rust::chiusure`
<div style='font-family: Arial; 
            text-align: right; margin-top: -20px; 
            font-size: 14px;
            border-top: 1px solid black'>
    &copy2024 <i>Carlo MIGLIACCIO</i>
</div>

## Puntatori a funzione

> **Funzione di ordine superiore**: funzione che può ricevere come parametro una funzione o che può ritornare una funzione.

Il concetto di funzione di ordine superiore implica che una funzione dovrebbe essere assegnata ad una variabile. In linguaggi come il C questo concetto viene modellato tramite un *puntatore a funzione*, in Rust similmente grazie ad una variabile di tipo `fn(<tipo1>, <tipo2>, ..., <tipon>) -><tipo_ret>` o attraverso l'utilizzo dei *tratti funzionali* `Fn`, `FnOnce` e `FnMut`.
Facciamo un primo esempio

```rust
fn add(a:i32, b:i32)->i32{
    a+b
}
fn main(){
    //Assegno ad una variabile una funzione
    let ptr_fun: fn(i32,i32)->i32 = add; 
}
```

Il codice (di un qualsiasi linguaggio di programmazione) che fa uso degli oggetti funzionali è molto verboso. Per questo motivo nei linguaggi moderni (Java, C++, JavaScript...) tra cui Rust, sono state introdotte le **funzioni lambda**. 

> **Funzione lambda**: funzione anonima costituita da un blocco di codice espresso in *forma letterale*.

In Rust per esprimere una chiusura si ricorre alla notazione
 `|<par1>, ..., <parN>| { <codice> }`.

Una volta definita una funzione lambda, essa può essere assegnata ad una variabile o essere chiamata tramite il nome della variabile, passando però gli argomenti come se la variabile fosse una funzione. Facciamo un esempio

```rust
fn main(){
    let mut fn_ptr = |a, b| {
        a+1; b+1; 
        println!("Nuovi valori: {:?} {:?}", a, b);
    }; 
    //Chiamo la funzione lambda  (passando i parametri)
    fn_ptr(4,5); 
}
```

## Chiusure e variabili libere
In Rust  il corpo di una funzione lambda pu fare riferimento alle variabili che sono visibili nel *contesto in cui è definita*, acquisendone:
* un riferimento
* un riferimento mutabile
* una copia
* il possesso

Tali variabili sono dette *variabili libere* e la funzione lambda così ottenuta viene detta **chiusura**, in quanto *racchiude* i valori catturati rendendoli disponibili laddove la chiusura venga chiamata nuovamente. Mentre ad esempio in Java e C++ le funzioni lambda vengono trasformate in **oggetti funzionali**, in Rust una funzione lambda viene trasformata in una *tupla* che ha **tanti campi quante sono le variabili libere**. Questa tupla implementa uno dei *tratti funzionali* citati in precedenza.

### Cattura delle variabili in Rust
In Rust, per default, tutte le variabili libere che si trovano nel corpo della funzione vengono 
**<u><center>catturate per riferimento (&)</center></u>**
Se occorre invece modificare le variabili catturate, occorre dichiarare la chiusura come mutabile in modo che possano essere passata come 
<center><u><b>riferimento mutabile (&mut)</b></u></center>

Quando necessario è possibile specificare che la funzione lambda prenda possesso delle variabili libere. Questo lo si deve indicare esplicitamente anteponendo la parola chiave `move` alla definizione della chiusura.

```RUST
...         ...         ...
let my_string = String::from("Carlo e' un nome maschile"); 

let clos_mia = move |s| {
    println!("Tua stringa {:?}",my_string); 
    println!("Mia stringa {:?}", s); 
}
//qui non posso utilizzare piu' my_string perché è stata mossa
...         
```
Quando esplito il movimento e la variabile che catturo è copiabile, di questa viene tenuta **una copia** all'interno della chiusura.

## Tratti funzionali `Fn`, `FnOnce`, `FnMut`
Questi tre tratti funzionali definiti da Rust sono implementabili  SOLO tramite chiusure, quale tratto viene implementato dipende da 
<b><center> cosa e come</center></b>
viene catturato all'interno della chiusura. La differenza nei diversi tratti sta quindi nel come si trattano le variabili libere catturate.

1. Una chiusura che implementa il tratto `Fn<Args>` **cattura per riferimento** le variabili libere che sono in *prestito condiviso* e non possono essere cambiate.
2. Una chiusura che implementa il tratto `FnMut<Args>` prende i parametri liberi attraverso un *riferimento mutabile* `&mut`. Se chiamata più volte, dal momento che vengono modificate le variabili catturate, potrebbe produrre un risultato diverso.
3. Quando una chiusura *consuma* uno o più valori come parr della sua esecuzione allora implementa il tratto `FnOnce<Args>`; come stesso il nome dice questa può essere chiamata <u>una sola volta</u>.

Tra le chiusure che implementano questi tratti funzionali, c'è una relazione d'ordine che è espressa da questo Diagramma di Venn:

<center>
    <img src='/img/tratti_funzionali.png' width='60%'>
</center>

## Funzioni di ordine superiore e chiusure
E' possibile realizzare una funzione di ordine superiore (funzione che accetta come parametro una funzione) ricorrendo alla programmazione generica. 

Il seguente codice esemplifica ciò che è appena stato esposto:

```rust
fn funz_sup<F>(num: i32, f: F)->i32 where F: Fn(i32)->i32{
    f(num)
}
fn main(){
    let mut num=4; 
    num = funz_sup(5, |a| {
        println!("numero passato: {}", a); 
        a
    }); 
    println!("Risultato chiusura: {}", num); 
}
```
Qui il parametro fun_clos può essere una *qualsiasi*  funzione o chiusura che sia chiamabile più volte e che cattura le variabili per riferimento.

Come caso duale è possibile scrivere delle funzioni che **ritornano** delle chiusure. Se la chiusura cattura delle variabili ci potrebbero essere dei problemi con i tempi di vita dei riferimenti; per questo motivo spesso si usa ritornare le chiusure anteponendo `move` alla chiusura stessa.

Facciamo un esempio:

```rust
fn ritorna_closure()->impl FnMut(i32)->i32{
    let mut i=0; 
    move |a| {
        i=i+1;
        println!("{}", i);
        a+i
    }
}
fn main(){
    let mut a = ritorna_closure(); 
    a(3);
    a(4); 
    a(5);
}
```






