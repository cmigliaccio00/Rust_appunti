<style> 
    body{
        text-align: justify; 
    } 

    h2{ margin: 0px; }
</style>

# `Rust::collezioni`
<div style='font-family: Arial; 
            text-align: right; margin-top: -20px; 
            font-size: 14px;
            border-top: 1px solid black'>
    &copy2024 <i>Carlo MIGLIACCIO</i>
</div>

La maggior parte dei linguaggi di programmazione offrono nella propria libreria standard un insieme di strutture dati che sono volte a semplificare il lavoro del programmatore, offrendo per ognuna delle strutture gli algoritmi più comuni realizzate utilizzando anche metodi diversi. Le strutture dati comuni sono principalmente:
* Liste ordinate
* Insiemi (di elementi univoci)
* Mappe chiave valore

La seguente figura è un sommario di tutte le strutture dati più comuni nei linguaggi di programmazione moderni:

![](/img/collections.png)

Tutte le collezioni, della libreria standard di Rust, hanno in comune dei comportamenti (metodi):
* `new()` per allocare una nuova collezione (qualunque essa sia). Qui in assenza di specificazione dei tipi generici, questi vengono inferiti dal compilatore all'atto del primo inserimento;
* `len()` permette di conoscer il numero di elementi di un certo tipo allocati in una certa collezione;
* `clear()` elimina tutti gli elementi della collezione; 
* `is_empty()` ritorna un valore booleano associato al fatto che la collezione possa o meno essere vuota
* `iter()` estrae un iteratore per gli items contenuti nella collezione 
* `extend()` permette di concatenare ad una prima collezione una seconda

Oltre a tutti questi metodi di cui abbbiamo parlato, TUTTE le collezioni implementano poi i tratti `IntoIterator` e `FromIterator` che forniscono rispettivamente i metodi:
1. `into_iter()` che permette di convertire qualsiasi collezione in un iteratore
2. `collect()` permette invece, al contrario, di raccogliere gli elementi generati da un iteratore in una certa collezione.

Nei prossimi paragrafi si darà una descrizione breve degli aspetti principali legati ai tipi comuni di collezione. 

## Code generalizzate 
### `Vec<T>`
Il tipo `Vec<T>` è una collezione dati che rappresenta un array ridimensionabile con elementi di tipo `T` che vengono allocati nello heap. La creazione di tale collezione può essere fatta:
* Ricorrendo al metodo statico `Vec::new()`
* Utilizzando la *macro* `vec![el1, ..., eln]`

Una variabile che ha tipo **`Vec<T>`** è una tupla formata da tre valori:
* Un puntatore al buffer di elementi allocato sullo heap
* Un intero unsigned riferito al numero di elementi effettivamente presenti nella collezione (len)
* Un intero senza segno riferito alla capacità massima attuale dell'array.

Un **nuovo elemento** può essere inserito con il metodo **`push()`**, nel caso in cui ci sia ancora spazio nel buffer, l'elemento viene collocato nella *prima posizione libera*, altrimenti viene allocato uno spazio maggiore e quello allocato in precedenza viene *rilasciato*.
Un **riferimento** ad un elemento della struttura si può ottenere in uno dei seguenti modi:
1. Utilizzando la notazione slice-fashioned `&v[indice]`
2. Utilizzando il metodo `get(indice)` che scatena un `panic!()` nel caso in cui siano superati i limiti del vettore allocato
3. Utilizzando il metodo `get_mut(indice)` che restituisce un `Option::None` nel caso in cui non si riesca altrimenti un `Option::Some(ref)`.

Per maggiori dettagli, si veda: [Rust Documentation: Vec\<T\>](https://doc.rust-lang.org/std/vec/index.html)

### `VecDeque<T>`
Il tipo `VecDeque<T>` rappresenta una coda a doppia entrata in cui gli elementi possono essere inseriti ed estratti sia dalla testa che dalla coda ricorrendo ai metodi `push_back()`, `pop_back()`, `push_front()`, `pop_front()`. 
Può essere indicizzato tramite la notazione `MioDeque[i]`, non garantisce che gli elementi siano contigui, ma la libreria standard mette a disposizione un metodo a tal proposito (`make_contiguous()`). 
La figura che segue offre una possibile rappresentazione in memoria della struttura dati appena descritta.

![](/img/Deque.png)
Tra i metodi messi a disposizioni per questo particolare tipo, degno di nota è `retain(f: F)`, che *trattiene* nella collezione gli elementi che soddisfano un certo predicato (passato tramite una chiusura).

##### Esempio (`retain()`)

```rust
let V = VecDeque::new(); 
V.push(3); V.push(4); V.push(8); 
//Trattieni solo  gli elementi pari
V.retain(|&x| x%2==0);
```

Per maggiori dettagli, si veda: [Rust Documentation: VecDeque\<T\>]()

### `LinkedList<T>`
E' una struttura dati che modella una **lista doppiamente concatenata**, similmente a `VecDeque<T>` per la presenza del doppio puntatore (successore e predecessore) permette di inserire ed estrarre elementi con costo costante.
Tuttavia si preferisce l'uso di `Vec` e `VecDeque` per prestazioni migliori e uso migliore della memoria.

Non c'è un metodo per ordinare una lista. Quello che invece si può fare è trasformare una `LinkedList<T>` in un `Vec<T>` passando per l'iteratore e poi ricostruire la linked list dopo aver ordinato il vettore.

Per maggiori dettagli, si veda: [Rust Documentation: LinkedList\<T\>]()

## Mappe
Una **mappa** in generale èuna collezione di coppie con chiave di tipo `K` e valore di tipo `V`, sia che la realizzo con una Tabella di Hash che con un albero bilanciato.

Per maggiori dettagli, si veda: [Rust Documentation: HashMap\<K,V\>]()
Per maggiori dettagli, si veda: [Rust Documentation: BTreeMap\<K,V\>]()

### `HashMap<K,V>`
In questo caso i valori sono salvati in memoria come una singola **tabella di hash**, si preferisce l'utilizzo di una tabella di hash all'albero quando le chiavi non hanno un ordinamento. Per ovvi motivi il tipo `K` della chiave deve implementare i tratti `Eq` (per poter gestire `==` o `!=`) e `Hash` per poter essere trasformate in un "digest" che faccia da indice alla tabella di hash, la chiave in quanto tale DEVE essere **univoca**.
L'allocazione di una nuova entry può causare la riallocazione dello spazio.

![](/img/hash_map.png)

#### Metodi utili
* `insert(key, value)` inserimento di una entry
* `get(&key)` restituisce un riferimento all'item con chiave specificata come parametro (se presente)
* `keys()` restituisce un iteratore alle chiavi della mappa
* `values()` restituisce un iteratore ai valori della mappa
* `remove(key)` rimuove dalla collezione un elemento con una certa chiave `key`
* `contains_key(key)`, restituisce un booleano elemento con una certa chiave presente/non presente.

#### `Entry<'a, K, V>`
Il lunguaggio Rust offre la possibilità di ottimizzare l'utilizzo delle mappe, mettendo a disposizione il metodo `entry(key)` che ritorna un enumerazione che permette di gestire in modo sicuro ulteriori operazioni sulla entry trovata. In particolare l'enum `Entry<'a, K, V>` offre i metodi:
* `and_modify<F>(f:F)` nel caso in cui l'elemento venga trovato viene modificato utilizzando la chiusura passata come parametro.
* `or_insert(valore: V)` nel caso in cui non ci sia, questo viene inserito nella mappa; 
* `or_insert_with(f:F)` nel caso in cui non venga trovato,viene inserito con un comportamento passato come parametro.

### `BTreeMap<K,V>`
Le entry della mappa sono salvate all'interno di un albero che viene mantenuto bilanciato in base all'ordine imposto dalla chiave. Si preferisce utilizzare un BTree al posto di un Hash quando le chiavi hanno un ordine.
Anche qui un nuovo elememento inserito può provocare la riallocazione dello spazio associato. La chiave  `K` deve essere univoca e deve implementare il tratto `Ord`.

![](/img/btree.png)

I metodi forniti per il `BTreeMap` sono più o meno gli stessi con l'aggiunta di questi altri interessanti metodi:
1. `range(range)` ritorna un iteratore agli elementi della mappa con chiave compresa in un certo range.
2. `range_mut(range)` restituisce un iteratore mutabile agli elementi della mappa con chiavbe compresa in un certo range.

## Insiemi
L'insieme è una collezione costituita da elementi univoci di tipo `V`. 

Per maggiori dettagli, si veda: [Rust Documentation: HashSet\<T\>]()
Per maggiori dettagli, si veda: [Rust Documentation: BTreeSet\<T\>]()

### `HashSet<T>`
Gli elementi dell'insieme sono salvati in un'unica *Tabella di Hash*. E' implementato come un wrapeer intorno all'implementazione di `HashSet<T,()>`.

### `BTreeSet<T>`
Gli elementi univoci sono salvati all'interno di un unico albero binario bilanciato salvato sullo heap.

In entrambe le collezioni ci soni i metodi classici e quelli che realizzano le più comuni operazioni insiemistiche (unione, intersezione, differenza, sottoinsieme, insieme disgiunto...).

## Altre collezioni
### `BinaryHeap<T>`
Questa struttura dati è la realizzazione efficiente di una coda a priorità in cui l'elemento più grande si trova sempre nel fronte della struttura dati. Dovendo stabilire una relazione d'ordine, il tipo `T` deve implementare il tratto `Ord`. Il metodo `peek()` permette di ritornare l'elemento più grande con complessità O(1).

Per maggiori dettagli, si veda: [Rust Documentation: BinaryHeap\<T\>]()