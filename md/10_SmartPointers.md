<style> 
    body{
        text-align: justify; 
    } 

    h2{ margin: 0px; }
</style>

# `Rust::smart_pointers`
<div style='font-family: Arial; 
            text-align: right; margin-top: -20px; 
            font-size: 14px;
            border-top: 1px solid black'>
    &copy2024 <i>Carlo MIGLIACCIO</i>
</div>

Gli **smart pointers** in Rust sono un concetto e uno strumento soprattutto che risulta essere un'estensione, ancora una volta, di qualcosa nato 'in casa'  C++. 
>Uno *smart pointer* in sintesi è una struttura che può essere utilizzata a tutti gli effetti come un <u>puntatore nativo </u>, ma in realtà non lo è perché, in generale, contiene tutta una serie di informazioni accessorie che permettono di utilizzare (accedere) alla memoria **in modo controllato**.

Per poter realizzare questo tipo di strutture, sia il C++ sia Rust, offrono la possibilità di *ridefinire* i comportamenti degli operatori, permettendo quindi la definizioni di **tipi che sembrano puntatori**, ma che al loro interno hanno delle informazioni accessorie  usate per:
* Realizzare il paradigma RAII che prevede acquisizione insieme a rilascio delle risorse di memoria ottenute; 
* Tenere dei conteggi sui riferimenti ad una certa locazione di memoria
* Accesso con attesa...
Tutta questo insieme di ipotesi ha portato all'introduzione del concetto di "smart pointer": un insieme di puntatori che **possiedono** i dati a cui puntano, a differenza dei riferimenti che prendono il dato solo in <u>prestito condiviso</u> o <u> prestito esclusivo </u>. 

Certamente, l'uso di puntatori non è semplice, ma questi sono il **solo modo** di realizzare strutture dati complesse: liste concatenate, alberi, grafi di diverse tipologie e così via.

Di base, il borrow checker di Rust, impedisce di avere più riferimenti allo stesso valore impedendo di fatto la realizzazione di **strutture cicliche**.
Con l'utilizzo degli smart pointers, invece, si permette:
* Sia di fare in modo che un valore abbia più possessori (tramite l'uso di `Rc<T>` e `Arc<T>`); 
* Sia di realizzare delle strutture che siano cicliche tramite il puntatore di tipo `Weak<T>`.

In C++, i tre smart pointer disponibili sono `unique_ptr<T>`, `shared_ptr<T>` e `weak_ptr<T>`. Vedremo come Rust parte di base da questi concetti e ne *estende la semantica*.

## Smart pointer in Rust
Rust offre rispetto a C++ una maggiore varietà di smart pointer per permetterne l'uso sia in *programmi sequenziali* sia in *programmi concorrenti*. In generale sono definiti, nella libreria standard, tramite tipi che implementano i tratti `Deref` e `DerefMut`.


### `std::Box<T>`: puntatore con possesso del dato

<center>
    <img src='/img/box.png' style='width: 50%'>
</center>

Il tipo `Box<T>` incapsula al suo interno un puntatore ad un dato che viene creato con la chiamata del metodo statico (costruttore) `b=Box::new(t)` dove `t` è un'istanza di una variabile di tipo `T`.
Il dato a cui punta è **posseduto** dal Box, quando la struttura stessa esce dallo scope sintattico l'area di memoria viene rilasciata (dato che Box implementa il tratto Drop). [Tuttavia c'è la possibilità di anticipare il rilascio, questo lo si fa richiamando il metodo `drop(b)`].

Se la struttura viene mossa in un'altra variabile, questa ne acquisisce il possesso e quindi *ne è responsabile del rilascio*. Questo, come abbiamo avuto già occasione di vedere, offre la possibilità di estendere il tempo di vita di variabili create all'interno di un certo scope sintattico.

Il tipo a cui fa riferimento il puntatore incapsulato in `Box` può anche essere <u>non noto</u> a tempo di compilazione (ergo: non implementa il tratto `Sized`), in questo caso la Box al suo interno avrà un **fat pointer**, stessa cosa capita se viene fatto riferimento ad un oggetto in *modo astratto* utilizzando oggetti-tratto (`dyn Trait`).

Questo semplice esempio riassume i concetti espressi:
```rust
let mut b = Box::new(5); 
*b=4;               //Modifica del valore contenuto in Box
let mut a=b;        //Movimento
println!("Valore del box: {}", a); 

//Se utilizzo qui `b` ci sarà un errore di compilazione
```
> **Nota:** se ad un certo punto chiamo il metodo `drop()` su un dato di tipo `Box`, la variabile in cui era stato allocato il puntatore non viene ancora tolta dallo stack che naturalmente si comprime ed espande, viene solo quindi rilasciata la porzione di memoria allocata per il valore a cui si puntava.

### `std::Rc<T>`: più possessori, dato immutabile
<center>
    <img src='/img/rc.png' style='width: 50%'>
</center>

Lo smart pointer `Rc<T>` si utilizza quando si vogliono avere **più possessori di uno stesso dato immutabile**. Viene allocato, come nel caso di Box, sullo Heap ma conserva - oltre il dato puntato - *due contatori* che sono:
* `strong`: indica quante copie(cloni) ci sono del puntatore; 
* `weak`: indica quanti riferimenti deboli sono presenti.

Ogni volta che un dato di tipo `Rc` viene clonato il contatore `strong` viene incrementato. Quando uno dei puntatori clonati esce dallo scope, `strong` viene decrementato di un'unità, quando arriva a 0 l'area di memoria allocata viene rilasciata.
`Rc<T>` è un tipo di puntatore che si presta alla realizzazione di alberi e in generali di grafi aciclici (anche non orientati).
Il tipo `Rc<T>` presenta un problema nel momento in cui si provano a realizzare dei collegamenti **ciclici**: se venisse usato lo stesso meccanismo di clonazione, ci sarebbe un problema di rilascio. Se ad esempio un dato $A$ punta ad un dato $B$ e quaesto punta a sua volta al dato $A$, entrambi i puntatori resteranno allocati **"in eterno"** perché ognuno dei due con il conteggio `strong=1` tiene in vita l'altro, questa è la ragione per cui si sono introdotti i puntatori di tipo `Weak<T>`.

> **Importante**: il tipo di puntatore `Rc<T>` non si presta ad essere utilizzato in ambienti *multi-thread* in quando la semplice operazione di incremento del contatore può trasformarsi in un problema. Ad esempio cosa succede se due thread fanno contemporaneamente su questo contatore le operazioni di *Read, Update & Write*? In assenza di sincronizzazione ci sarebbero seri problemi. Questa è  la ragione per cui in ambiente multi-thread viene introdotto un altro tipo di puntatore (`Arc<T>`), che preserva le caratteristiche di `Rc<T>` e ha l'unica importante differenza che le operazioni di aggiornamento dei contatori vengono fatte in modo **atomico**. In pratica l'unica cosa che cambia tra i due è l'istruzione macchina che viene utilizzata che in un caso è una semplice `INC/DEC` (in x86), altrimenti un'operazione di tipo atomico implementata in vari modi.

I metodi `Rc::<T>::strong_count(&a)` e `Rc::<T>::weak_count(&a)` permettono di ottenere il valore dei due contatori associati allo smart pointer.

Il dato puntato da `Rc` in realtà è mutabile SOLO se non esistono altri riferimenti al dato (`strong=1`), in questo caso tramite il metodo `Rc::<T>::get_mut(&mut rc)` posso risalire ad un riferimento mutabile che mi permette di modificare il dato. Dato che l'operazione di estrazione di riferimento mutabile da tale smart pointer potrebbe non essere possibile, il metodo `get_mut()` restituisce un `Option<T>`.

### `std::Weak<T>`: riferimento senza possesso
E' stato presentato il problema dei riferimenti circolari nel caso in cui si facesse utilizzo soltanto di `Rc` per la realizzazione di strutture cicliche. Questo è il motivo per cui sono stati introdotti i riferimenti di tipo `Weak<T>`, questo si può vedere come una versione particolare di `Rc` in cui non c'è il possesso del dato. Per questo motivo i riferimenti di tipo <u>weak</u> NON tengono in vita il dato e nello specifico: 
* Se ciò a cui punto è ancora vivo (`strong`>1), ci arrivo
* Se ciò a cui punto invece è stato rilasciato (`strong`=0), non ci arrivo.

Un riferimento debole di tipo `Weak<T>` può essere creato a partire da un puntatore di tipo `Rc` con il metodo `Rc::downgrade(&rif)` (dove `rif` è di tipo `Rc`). Il metodo 
`upgrade()` consente di creare un riferimento di tipo `strong` e quindi di creare un nuovo `Rc`. Dato che questa operazione è possibile solo in presenza di almeno un riferimento `strong`, viene ritornato dal metodo un `Option<T>`.

### `std::cell::Cell<T>`: *interior mutability*
<center>
    <img src='/img/cell.png' style='width: 50%'>
</center>

Sappiamo bene a questo punto della trattazione di Rust che il borrow checker controlla di volta in volta il corretto utilizzo di riferimenti e riferimenti mutabili effettuando un'accurata **analisi statica** del codice  sorgente.
Tuttavia certe volte le regole del compilatore risultano troppo restrittive, a questo proposito `std::cell` offre delle strutture che consentono una mutabilità condivisa, nel senso che è possibile modificare il dato puntato anche se ci sono in giro dei riferimenti ad esso. [Nota che: come `Rc` questi possono essere utilizzati solo in ambiente **single thread**].

In particolare la struct `std::cell::Cell<T>` implementa una **interior mutability** nell'ambito di una **exterior immutability**, le `Cell` consentono quindi si bypassare le regole imposte dal compilatore permettendo di modificare il dato posseduto (**allocato sullo stack**) tramite un meccanismo di swap (leggo il vecchio, rimpiazzo il nuovo).

Facciamo un esempio:

```rust
use std::cell::Cell;

#[allow(dead_code)]
#[derive(Debug)]
struct Tipo{
    a: i32, 
    b: Cell<i32>
}

fn main(){
    let struttura = Tipo{a: 5, b:Cell::new(4)}; 
    println!("Struttura prima: {:?}", struttura); 
    struttura.b.set(8);
    println!("Struttura dopo: {:?}", struttura);
}
```

### `std::cell::RefCell<T>`
<center>
    <img src='/img/refcell.png' style='width: 50%'>
</center>

Questo tipo di smart pointer permette invece di ottenere un riferimento al dato posseduto (allocato sullo stack), `Cell` non consente di fare questa operazione.
In particolare, una struct di tipo `std::cell::RefCell<T>` è associato ad un'area di memoria a cui si può accedere tramite particolari smart pointer (un'altro paio) `Ref<'_,T>` e `RefMut<'_,T>` che *simulano* il comportamento di riferimenti e riferimenti mutabili. L'unica **differenza** è che la compatibilità viene verificata a *tempo di esecuzione* e NON a tempo di compilazione. 

Per poter avere questo tipo di comportamento questa struttura contiene, oltre al dato, un campo `borrow` che funge da contatore dei riferimenti al dato T che è incapsulato al suo interno. Valgono le seguenti regole:
* Se viene chiesto un riferimento in lettura ed esiste già un riferimento in lettura questo viene concesso; viene inoltre aggiornato il counter `borrow`.
* Se viene richiesto un riferimento in lettura ed esiste invece un riferimento in scrittura (di tipo `mut`) al dato stesso, viene generato `panic`
* Allo stesso modo se si chiede un riferimento in scrittura ed esiste un riferimento in lettura/scrittura, viene mandato in `panic` il programma.

I metodi `borrow()` e `borrow_mut()` restituiscono uno smart pointer, rispettivamente, di tipo `Ref` e `RefMut`, questi generano panic nel caso non siano rispettate a tempo di esecuzione le regole elencate prima.
Esistono inoltre delle versioni di questi metodi che restituendo un `Result` non mandano il programma in panic. Essi sono `try_borrow()` e `try_borrow_mut()`. 

Ulteriori metodi forniti da `RefCell`  sono:
* `into_inner()` consuma il dato di tipo `T` posseduto ritornandolo.
* `get_mut()` ritorna un vero e proprio `&mut` nel caso in cui non ci siano in essere dei riferimenti al dato (`borrow=0`). 

> **Nota implementativa**: se creo un riferimento mutabile tramite il metodo `borrow_mut()` o `try_borrow_mut()` e si cerca di fare una stampa sul valore originale, verrà stampato il segnaposto `<borrowed>`. Al contrario se viene estratto un riferimento in sola lettura, viene stampato il valore originale. Valgono sempre però le regole che vengono controllate in fase di esecuzione.

### `std::borrow::Cow<'a, B>`
E' un tipo di Smart Pointer che implementa il meccanismo **clone on write (COW)**. Funziona in questo modo:
* Se si aveva a disposizione un riferimento e cerco di modificare il dato, questo viene clonato, si effettua la modifica, se ne prende possesso mentre l'originale viene lasciato invariato.
* Se invece si cerca di effettuare una modifica di un qualcosa di già posseduto, non viene effettuata la clonazione, ma viene modificata direttamente la copia originale.

Un istanza di questo particolare smart pointer viene creata con l'istruzione `Cow::from()` e internamente (nella libreria standard quindi) viene realizzata tramite un tipo enumerativo che ha due varienti: `Borrowed` e `Owned`, il primo si riferisce ad un *Cow-smartpointer* a partire da un riferimento, l'altro si riferisce ad uno smart pointer creato su un dato già posseduto.

### Smart pointer e metodi 
Sappiamo dal capitolo sui tipi di dato composti e sul polimorfismo che un metodo di un certo tipo si riconosce dal fatto che, *all'interno del blocco `impl`*, ci siano delle funzioni che come primo parametro hanno `self` o un suo riferimento. Questi hanno quindi tipo `Self`, `&Self` o `&mut Self`. 
Dato che Rust internamente conosce molto bene gli smart pointer, questi possono essere utilizzati nei metodi come tipo del parametro self, inoltre dato che `Rc, Arc, Box` implementano il tratto `AsRef`, questi possono essere utilizzati 'normalmente' utilizzando la notazione `ricevitore.metodo()` anche se ricevitore è stato istanziato come uno smart pointer.

Se vogliamo che invece certi metodi siano legati ad istanze di un certo tipo dichiarate come tipi specifici di smart pointer, questo dobbiamo specificarlo nella firma del metodo (ergo: il compilatore non può inferirlo in nessun modo). 
Facciamo un esempio:

```rust
impl Nodo{
    fn appendi(self: Arc<Self>, el: i32){
        unimplemented!(); 
    }
}
```
In questo caso stiamo specificando che il metodo `appendi()` è 'dedicato' solo a quelle istanze del tipo `Nodo` che sono state dichiarate come tipo puntato da uno smart pointer di tipo `Arc`. 

Qui di seguito viene fatto un esempio in cui vengono chiariti questi aspetti: 
```rust
use std::rc::Rc; 
#[derive(Debug)]
struct MioTipo{
    campo1: i32, 
    campo2: i32,
}
impl MioTipo{
    fn new()->Self{
        Self{campo1:0, campo2:0}
    }
    //Ricorda Rc non può avere il dato puntato modificato
    fn metodo1(self: Rc<Self>){
        println!("Stampa speciale della struct: {:?}", self);
    }
    fn metodo2(&mut self, inc: i32){
        self.campo1+=inc; 
        self.campo2+=inc; 
    }
    fn print_debug(&self){
        println!("Stampa di Debug: {:?}", self);
    }
}
fn main(){
    let mut mys = MioTipo::new(); 
    mys.metodo2(3);
    mys.print_debug();
    //Tipo istanziato utilizzando un reference counter
    let mut mys1 = Rc::new(MioTipo::new()); 
    //Questo me lo consente di fare: ho strong=1, altrimenti 
    //mi darebbe None perché ci sono più copie del puntatore
    //ovvero ci sono più possessori dello stesso dato (immutabile)
    println!("strong_count: {}", Rc::<MioTipo>::strong_count(&mys1));
    if let Some(r) = Rc::<MioTipo>::get_mut(&mut mys1){
        //(*r).metodo2(3);          <== crazy form!
        r.metodo2(3);
    }
    mys1.metodo1();
    //mys.metodo1();        <== Questo genera errore 
                            //perché mys NON e' incapsulato in un ARC
}
```
