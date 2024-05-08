<style> 
    body{
        text-align: justify; 
    } 

    h2{ margin: 0px; }
</style>

# `Rust::lifetime`
<div style='font-family: Arial; 
            text-align: right; margin-top: -20px; 
            font-size: 14px;
            border-top: 1px solid black'>
    &copy2024 <i>Carlo MIGLIACCIO</i>
</div>

> **_Tempo di vita di un riferimento_**
Intervallo minimo (insieme di righe di codice) per il quale il valore prestato (borrowed) deve essere bloccato (in scrittura) affinché non vengano violati i vincoli imposti dalla funzione.
Un tentativo di accesso (in scrittura) al dato prestato prima che sia concluso il tempo di vita del  riferimento porta alla generazione di un errore da parte del compilatore. Questo costituisce la base della **robustezza** garantita dal linguaggio Rust.

## Riferimenti e funzioni
Quando ad una **funzione** viene passato come parametro un riferimento, il suo **tempo di vita** (lifetime) diventa parte integrante della firma della funzione stessa.
Quando non ci sono ambiguità il compilatore facente parte della toolchain di `Rust` prevede a gestire autonomamente i tempi di vita senza che ci debba essere l'intervento del programmatore (come vedremo questo è un concetto noto come *lifetime elision*).

La situazione però cambia nel momento in cui una funzione riceve **due o più** riferimenti in ingresso e restituisce un riferimento in uscita. Il compilatore in questo caso NON riesce ad assegnare autonomamente i tempi di vita: è richiesta  in questo caso **annotazione esplicita** da parte del programmatore.

Nel caso di singolo riferimento in ingresso e singolo riferimento in uscita (situazione in cui si può  evitare l'annotazione esplicita) la firma del metodo
```rust
fn funzione_rif (rif_a:&str) -> &str;
```
diventa:
```rust
fn funzione_rif<'a>(rif_a:&'a str) -> &'a str; 
```
Nel caso in cui invece i riferimenti siano due o più si possono avere le seguenti due situazioni:

##### Primo caso (tempi di vita uguali)
```rust
fn funzione<'a> (rif_a:&'a str, rif_b:&'a str)->&'a str; 
```
##### Secondo caso (tempi di vita differenti)
```rust
fn funzione<'a, 'b> (rif_a:&'a str, rif_b:&'b str)->&'a str; 
```
Alcune osservazioni:
* C'è la necessità di introdurre le annotazioni solo nella firma dei metodi non nel corpo della funzione;
* Se il **riferimento restituito** dipende da un solo parametro tra quelli della funzione, occorre annotare solo quel parametro.
* Nel caso di funzione generica, le *metavariabili* vengono indicate <u>dopo</u> le annotazioni di lifetime.
Lo scopo degli identificatori legati al lifetime è duplice:
1. Per il codice chiamante, che invoca la funzione, essi indicano su quale, tra gli indirizzi in ingresso è basato il risultato di uscita
2. Per il codice chiamato, all'interno della funzione essi garantiscono che vengono restituiti valori per cui è lecito accedere almeno per il tempo di vita indicato.

## Riferimenti e strutture dati
Nel caso in cui un riferimento viene memorizzato all'interno di una **struttura dati**, il compilatore deduce che il tempo di vita della struttura deve essere incluso (minore) o coincidente(uguale) al tempo di vita del riferimento.

Un ragionamento analogo vale per le strutture dati che vengono definite e che al loro interno hanno un riferimento: è  richiesta annotazione esplicita di lifetime. Un esempio è il seguente

```rust
struct Utente<'a>{
    nome: &'a str, 
    eta: usize
}
```
Questa annotazione esplicita serve per permettere al compilatore di verificare che il valore a cui si punta abbia un tempo di vita che sia **maggiore o uguale** della struttura dati in cui è contenuto.

Se una tipo che viene definito contiene a sua volta un campo che richiede che sia specificato il tempo di vita, bisogna anche per questo specificare il tempo di vita. Un esempio:

```rust
struct Profilo<'a>{
    Who: Utente<'a>, 
    data: SystemTime    
}
```
Per indicare che si ha un riferimento ad una stringa costante il riferimento deve avere l'annotazione `'static`, questo denota che il valore prestato ha durata di vita pari alla durata dell'intero programma.
### Regole del compilatore
E' utile a questo punto poter definire delle regole che il compilatore utilizza nel momento in cui una funzione ha come parametri dei riferimenti che quindi richiedono delle annotazioni esplicite di tempo di vita.

1. Per ogni parametro in ingresso che sia un riferimento il compilatore assegna un tempo di vita differente.
2. Se la funzione restituisce più riferimenti e ha come parametro formale un solo riferimento, il tempo di vita di quel riferimento viene propagato su tutti i parametri d'uscita; 
3. Se ci sono più riferimenti in ingresso, e quindi più parametri diversi di lifetime, ma uno di loro è `&self` o `&mut self`, il parametro di lifetime di self è assegnato a tutti i parametri di output. <small>Questo parte dall'assunzione che se una funzione è un metodo di un certo tipo e restituisce un riferimento, probabilmente quello punta a qualcosa che l'oggetto stesso possiede.</small>

## Tempi di vita anonimi
Nel caso di riferimenti inseriti all'interno di strutture dati, nel caso in cui il tempo di vita si possa inferire dal metodo, si può utilizzare l'annotazione di tempo di vita anonimo , che si indica con `<'_>`.