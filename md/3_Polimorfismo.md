# `Rust::polimorfismo`

## Polimorfismo
Si parla di **polimorfismo** in informatica quando si associano dei comportamenti comuni a più tipi distinti.

$ 
\Large{
    \text{Comportamenti comuni} \to \text{Tipi distinti}
}
$

Nonostante il **processo di analisi** di un dominio applicativo tenda a dividere il codice sorgente in più categorie, d'altro canto si tende spesso a cercare di minimizzare il codice scritto  utilizzando dei **pattern comuni**.
Il *polimorfismo* può essere implementato attraverso:
* L'utilizzo di **tipi generici** nell'implementazione di alcuni metodi permette di utilizzare delle variabili senza il tipo, ma solo come simboli astratti, nel momento in cui vengono poi istanziati i tipi vengono definiti.
* L'utilizzo di **interfacce** che permettano l'implementazione specifica per ogni tipo

## Tratti

> **Definizione (Tratto)**
Un *tratto* si definisce come una **collezione di metodi** che definiscono dei *comportamenti* che un tipo <u>può</u> implementare, eventualmente come in altri linguaggi di programmazione, si può definire un'implementazione di default.

A differenza di altri linguaggi di programmazione, in Rust, quando si usa un tratto e lo si implementa quindi, non c'è un costo aggiuntivo che si presenta invece come vedremo, solo quando si crea un riferimento dinamico `&dyn Tratto`.

Un tratto si definisce con la seguente sintassi: 

```rust
trait MioTratto{
    fn metodo1(&self)->i32;
    fn metodo2(self)->u8;
    .....//altri metodi e/o implementazioni di default
}
```

Una struttura dati, sia essa una `struct` o una `enum`, che voglia implementare il tratto lo invece con la seguente sintassi:

```rust
impl MioTratto for MioTipo{
    fn metodo1(&self)->i32{
        //implementazione del metodo 1
        unimplemented!(); 
    }
    ....
}
```
Dato un tipo che implementa un certo tratto, i metodi ad esso associato possono essere chiamati sempre alla stregua delle struct con la notazione `tipo.metodo()` a patto che il tratto sia stato dichiarato nello stesso crate o che sia stato importato con la notazione `use <Namespace>::<Tratto>`.

## Definire ed usare un tratto
Quando si utilizza la parola chiave `Self` all'interno della firma dei metodi ci si riferisce al tipo che implementerà quello specifico tratto.

Se all'interno di una funzione definita nel tratto, viene utilizzato come primo parametro il tipo **`self`** o una delle sue varianti (`&self`, `&mut self`), questo prende il nome di **metodo** in quanto sarà caratteristico del tipo che implementerà il tratto. [In realtà il primo parametro può essere anche uno smart pointer a self, per esempio  `Box<self>`, `Rc<self>`, `Arc<self>`].

### Tipi associati
Un tratto può avere uno o più tipi associati, questo consente alle funzioni del tratto di far riferimento, **in  modo astratto** a tali tipi che poi ovviamente dovranno essere specificati nel momento in cui il tratto stesso sarà implementato.

```rust
trait MioTratto2{
    type Tipo_Assoc; 
    fn somma (par1: Self::TipoAssoc, par2: Self::TipoAssoc)->Self::TipoAssoc;
}
```

Nell'implementazione, poi, si specificherà il tipo nel seguente modo

```rust
impl MioTratto2 for MioTipo2{
    type Tipo_Assoc=i32; 
    fn somma(par1: Self::TipoAssoc, par2: Self::TipoAssoc)->Self::TipoAssoc{
        par1 + par2
    }
}
```
[In realtà a questo punto dovrebbe essere specificato un altro aspetto del tratto, che per il momento trascuriamo per poi riprendere. La sintassi dell'esempio appena mostrato quindi è solo parzialmente corretta].

### Implementazione di default
Come già accennato all'interno di un tratto, come per le interfacce di Java, ci possono essere delle implementazioni di default, questa poi potrà essere sovrascritta o utilizzata come tale dal tipo specifico che implementerà il tratto.

## Dipendenza tra tratti: Sottotratti e Supertratti
In Rust NON esiste il concetto di **ereditarietà** c'è però una relazione di dipendenza tra tratti del tipo Supertratto $\iff$ Sottotratto. Nello specifico un codice come `trait MioTratto1: MioTratto2` sta ad indicare che i tipi che implementano `MioTratto1`, <u>devono</u> implementare anche il tratto `MioTratto2`.

### Invocare un tratto
L'invocazione dei metodi di un tratto può avvenire in due modi distinti: 
#### Invocazione statica
In questo caso il tipo è noto, il compilatore può identificare quindi l'indirizzo della funzione da chiamare e quindi può generare il codice senza nessuna penalità. E' il modo più comune per chiamare i metodi che prevede un certo tratto.

#### Invocazione dinamica
E' il caso in cui si ha a disposizione un puntatore di cui il compilatore sa solo che sarà ad una variabile di un tipo che implementa un dato tratto, questo è il caso in cui occorre effettuare una **chiamata indiretta** passando per una VTABLE. I riferimenti o puntatori ai tipi tratto vengono detti **oggetti-tratto**, in quanto riferimenti possono essere condivisi o mutabili e devono rispettare i tempi di vita dei dati a cui fanno riferimento.
Gli oggetti tratto vengono implementati come dei *fat-pointer* costituito da:
* un puntatore al dato
* un puntatore ad una struttura VTABLE

<!--### <div style='color: red'> (...) continuare </style>-->

## Tratti della libreria standard
La libreria standard di Rust definisce un **insieme di tratti  base** la cui eventuale implementazione da parte dei tipi che vengono definiti dall'utente, abilita tutta una serie di scorciatoie da utilizzare nel codice che facilitano di molto la vita del programmatore. E' un meccanismo ben noto come *operator overloading* già presente in C++. 
Un esempio: è possibile confrontare due istanze di un certo tipo con gli operatori `== `e `!=` se tale tipo implementa i tratti `Eq` o `PartialEq`, è possibile invece usare gli operatori di ordine se i tipi implementano i tratti `Ord` e/o `PartialOrd` e così via. 

> *Nota che*: sebbene sia possibile fornire un implementazione di certi tratti della libreria standard, molte volte è meglio affidarsi al compilatore lasciando che **derivi automaticamente** i tratti della libreria standard. Si consideri il seguente esempio:
```rust
#[derive(Debug)]
struct MioTipo{
    campo: i32
}
fn main(){
    let a = MioTipo{campo:2}; 
    println!("Struttura: {:?}", a);
}
```
>In questo caso la stampa con `{:?}` è abilitata da `#[derive(Debug)]` che aggiunge automaticamente al tipo l'implementazione del tratto per il campo interno `campo`.

#### Gestire i confronti di uguaglianza: `Eq, PartialEq`
Affinché si possa implementare il confronto di uguaglianza per un certo tipo, c'è bisogno che siano rispettate delle proprietà logiche che sono essenziali: si parla della proprietà riflessiva, simmetrica e transitiva. In Rust queste proprietà devono essere espletate tramite l'implementazione di due differenti tratti: 
* `PartialEq` che richiede l'implementazione dei metoodi `eq()` ed `ne()` $\to$ asserisce al fatto che siano rispettate le proprietà **simmetrica** e **transitiva**
* `Eq` (sottotratto e quindi specializzazione di `PartialEq`) non ha metodi, ma la sua implementazione è solo un modo per esplicitare l'uguaglianza tra elementi di quel tipo gode anche della proprietà **riflessiva**.

I metodi associati ai tratti sono:
```rust
eq(&self, other: &Rhs)->bool; 
ne(&self, other: &Rhs)->bool;
```
Il tipo `Rhs` di other è per default `Self`, in rari casi potrebbe essere necessario dover fare dei confronti tra tipi diversi, in quel caso va specificato con la notazione `PartialEq<i32>` (per esempio se l'altro tipo associato è i32).

#### `PartialOrd`, `Ord`: Gestire i confronti d'ordine
Per permettere di gestire relazioni d'ordine totali e parziali i tratti `Ord` e `PartialOrd` forniscono i metodi necessari per farlo.
La funzione da implementare se si sceglie di implementare il tratto `PartialOrd` è 

```rust
enum Ordering{
    Less, 
    Equal, 
    Greater
}
partial_cmp(&self, other: &Rhs)->Option<Ordering>; 
```
Questo è adatto per le situazioni in cui non si riescono a confrontare due istanze  di un dato tipo.
Se poi la relazione d'ordine che vogliamo imporre sulle istanze di un dato tipo è di tipo *totale*, allora bisogna implementare il tratto `Ord` che ha come vincoli che si implementino sia `Eq` che `PartialOrd` (chiaramente dato che `Eq` è un sotto tratto di `PartialEq` anche questo deve essere implementato).

Il metodo da implementare (di cui quindi NON è fornita un'implementazione di default) è il seguente: 

```rust
cmp(&self, other: &Rhs)->Ordering; 
```

Entrambi i tratti forniti per implementare le relazione d'ordine e ordine totale, forniscono dei metodi con implementazione di default quali `le(), lt(), ge(), gt()`.


#### `Display`: Visualizzazione dei contenuti di un tipo
Alcune macro fornite da Rust come `println!` e `format!` consentono di stampare un valore associato al segnaposto `{}` a condizione che tale tipo implementi il tratto `Display` (che prevede l'implementazione del metodo `fmt()`). 
La firma del tratto `Debug` è la stessa, il segnaposto che attiva questo tratto è però `{:?}`. Uno si potrebbe quindi chiedere: *Qual è la differenza tra questi due tratti?* Il primo serve per visualizzare il contenuto di un tipo per l'**utente finale**, l'altro invece dovrebbe essere implementato per una visualizzazione facilitata per il programmatore.
Come accennato prima, può essere generato automaticamente utilizzando la direttiva `#[derive(Debug)]`.

#### `Copy` e `Clone`: gestire la copia e la duplicazione
Il tratto `Clone` permettono ad un'istanza di un dato tipo di realizzare un **duplicato** di un valore dato. Questo può essere utilizzato ad esempio per trasformare un riferimento in un valore posseduto. E' una funzionalità che richiedendo una *deep-copy* può essere anche molto costosa e di fatto non viene attivata mai in automatico.

Il tratto `Copy` permette ad un dato tipo di creare una vera copia *bit per bit* del suo valore, al contrario di clone questa operazione viene svolta in modo molto veloce tramite la chiamata di sistema di `memcopy()`. 
Può essere implementato solo da strutture i cui campi implementino il tratto stesso. 

>**Importante**: la presenza del tratto copy $\textsf{Trasforma la semantica delle operazioni di assegnazione}$ perché quello che determinerebbe un **movimento** rendendo il dato precedente inaccessibile, diventa come negli altri linguaggi di programmazione una **copia**.

#### `Drop`: rilasciare le risorse
L'implementazione di tale tratto tramite il metodo `drop()` permette di realizzare in Rust quello che in C++ viene realizzato con il distruttore di una classe. Il compilatore assicura di chiamare tale metodo nel momento in cui una variabile esce dal suo *scope sintattico* in modo che non ci sia in nessun modo *memory leakage*. 

Inoltre affinché non ci sia un doppio rilascio questo tratto e `Copy` sono mutuamente esclusivi.
A questo tratto inoltre è accompagnata la **funzione globale** `drop()` che *forzando un passaggio di possesso* ad una nuova variabile ne provoca l'uscita di scena provocandone la **distruzione definitiva**.

Di seguito  è stata definita una funzione che fa una cosa simile. Essendo che la variabile `a` è di tipo `Vec` e non implementa `Copy` un semplice passaggio alla funzione ne provoca il movimento con un conseguente 'passaggio di proprietà'. La variabile passata quindi viene distrutta.

```rust
fn mia_drop<T> (mia_var: T) where T: Drop{}
fn main(){
    let a = Box::new(5); 
    mia_drop(a);
}
```

#### `Index` e `IndexMut`: indicizzare una struttura dati
Per fare in modo che su una certa struttura dati si possa utilizzare la notazione `t[i]`, bisogna che questa implementi i tratti `Index` e `IndexMut`, questi sono dotati di un solo metodo rispettivamente si chiamano `index()` e `index_mut()`, avvengono quindi le seguenti trasformazioni: 

* Un accesso in lettura con la notazione `t[i]` viene trasformato in `*t.index(i)`;
* Un accesso in scrittura a `t[i]` viene trasformato invece in `*t.index_mut()`.

Di seguito riportiamo un esempio in cui si implementano i due tratti per un tipo definito  dall'utente. Si sceglie una stringa come tipo per indicizzare:

```rust
use core::ops::IndexMut;
use core::ops::Index;
struct Studente<'a>{
    nome: &'a str,
    giorno: usize, 
    mese: usize, 
    anno: usize
}
impl<'a> Studente<'a>
{
    fn new(nome: &'a str)->Self{
        Studente{nome, giorno:0, mese:0, anno:0}
    }
    fn set_data(&mut self, g:usize, m:usize, a:usize){
        self.giorno=g; 
        self.mese=m; 
        self.anno=a; 
    }
}
impl<'a> Index<String> for Studente<'a>{
    type Output=usize; 
    fn index(&self, index: String)->&Self::Output{
        if index=="giorno".to_string(){
            &self.giorno
        }
        else if index=="mese".to_string(){
            &self.mese
        }
        else if index=="anno".to_string(){
            &self.anno
        }
        else {
            panic!("Caso non previsto!"); 
        }
    }
}
impl<'a> IndexMut<String> for Studente<'a>{
    fn index_mut(&mut self, index: String)->&mut Self::Output{
        if index=="giorno".to_string(){
            &mut self.giorno
        }
        else if index=="mese".to_string(){
            &mut self.mese
        }
        else if index=="anno".to_string(){
            &mut self.anno
        }
        else {
            panic!("Caso non previsto!"); 
        }
    }
}
fn main(){
    let mut a = Studente::new("Carlo"); 
    a.set_data(24,7,2000);  
    
    println!("Nome: {}", a.nome);
    println!("Giorno: {}, Mese: {}, Anno: {}", a["giorno".to_string()], 
    a["mese".to_string()], 
    a["anno".to_string()]);
    
    //Modifica del valore
    a["giorno".to_string()]=27; 
    println!("Giorno: {}, Mese: {}, Anno: {}", a["giorno".to_string()], 
    a["mese".to_string()], 
    a["anno".to_string()]);
}
```

#### `Deref` e `DerefMut`: dereferenziare un valore
Se un tipo implementa questi due tratti, è possibile utilizzarlo come se fosse un puntatore. Praticamente parlando, la sintassi `*t` (se  un tipo implementa Deref) su un tipo che non sia un puntatore nativo equivale a `*(t.deref())` e restituisce un dato immutabile di tipo `Self::Target`. Se  invece implementa `DerefMut`, la sintassi `*t` equivale a `*(t.deref_mut())` e ritorna un dato mutabile di tipo `Self::Target`. 

La cosa importante è che il tipo `Self::Target`, DEVE essere qualcosa contenuto, posseduto o referenziato dal tipo Self e che abbia dimensione nota a tempo di compilazione. I due tratti permettono inoltre la **deref coercion**, cioè la traduzione automatica di &Self  in &Self::Target e di &mut Self in &mut  Self::Target.

Mostriamo di seguito un esempio che utilizzi queste proprietà:

```rust
use core::ops::DerefMut;
use core::ops::Deref;
#[derive(Debug)]
#[allow(dead_code)]
struct Studente{
    matricola: usize, 
    nome: String
}

impl Studente{
    fn with_attributes(matricola: usize, nome: String)->Self{
        Self{matricola, nome}
    }
}
impl Deref for Studente{
    type Target=usize; 
    fn deref(&self)->&Self::Target{
        &self.matricola
    }
}
impl DerefMut for Studente{
    fn deref_mut(&mut self)->&mut Self::Target{
        &mut self.matricola
    }
}
fn main(){
    let mut stud = Studente::with_attributes(2724, "Carlo".to_string()); 
    println!("Prima: {:?}", stud);
    *stud = 3329; 
    println!("Dopo: {:?}", stud);
    *(stud.deref_mut())=2444;
    println!("Dopo: {:?}", stud);
}
```

Abbiamo voluto in questo caso che le istanze di tipo `Studente` nel momento in cui fossero dereferenziate, si accedesse come campo target alla matricola (in lettura o in scrittura). 

#### `From, Into`: Conversione tra tipi
I tratti `From` e `Into` permettono di fare le conversioni di tipo sono perfettamente duali e sono definiti nel modo seguente: 
```rust
trait From<T>: Sized{
    fn from(other: T)->Self; 
}
trait Into<T>:Sized{
    fn into(self)->T;
}
```
I due tratti sono **perfettamente duali** tanto che se viene implementato uno dei due l'altro viene generato automaticamente.

Ci sono inoltre i tratti `TryFrom` e `TryInto` che permettono di fare la stessa cosa, a differenza del fatto che i metodi `try_from()` e `try_into()` restituiscono un `Result<T,E>` utile per la gestione degli errori. Per ottenere un tipo `T` a partire da una Stringa esiste il tratto `FromStr`, che restituisce come gli altri un  `Result` per permettere la gestione degli errori. 

Nel seguente esempio abbiamo implementato per il tipo `Studente` il tratto `From` per la conversione Stringa $\to$ Studente.

```rust
#[derive(Debug)]
#[allow(dead_code)]
struct Studente{
    nome: String, 
    matricola: String
}
impl From<String> for Studente{
    fn from(other: String)->Self{
        let ar:Vec<&str> = other.as_str().split(",").collect(); 
        Self{nome: ar[0].to_string(), matricola: ar[1].to_string()}
    }
}
fn main(){
    let b = Studente::from("Carlo,272489".to_string()); 
    
    println!("{:?}", b);
}
```

## Tipi generici
L'uso in programmazione di un sistema di tipi stringente offre la possibilità di creare del codice che sia robusto e snello. Sia il C++ (con la template programming) che il Rust (con l'utilizzo di *generics*) permettono di fare uso di funzioni, strutture dati, metodi e tratti che manipolini delle variabili il cui tipo è rappresentato da una **meta-variabile** indicata genericamente con il tipo `T`. 
In questo modo è possibile descrivere dei *comportamenti generali*  concentrandosi più sull'algoritmo e meno sui tipi che questi algoritmi manipolano.

Quando poi una funzione generica viene invocata, il compilatore provvede a dedurre cosa debba essere sostituito con il segnaposto `T` per specializzare il codice per tale tipo. Questo meccanismo è noto come **monomorfizzazione**.

In Rust una struttura di tipo generica viene definita con un nome seguito da un certo numero di metavariabili racchiuse  tra parentesi angolari, accompagnati da una serie di vincoli espressi come *tratti da implementare* o *tempi di vita che deve garantire*. 

Si riporta un esempio: 
```rust
//Definizione della struttura
struct MyStruct<T> where T:SomeTrait{
    campo: T
}

//Implementazione della struttura
impl<T> MyStruct<T> where T: SomeTrait{
    fn metodo1(){...}
    ...
}
```

Se vengono utilizzate delle meta-variabili in un metodo o in generale una funzione, le metavariabili devono essere dichiarate (sempre tra parentesi angolari) prima dei parametri formali e dopo il nome della funzione, nel seguente modo: 

```rust
fn funz_generica<T>(miavar: T){
    unimplemented!(); 
}
```

##### Trait Bounds
Sono un modo per esprimere dei vincoli sui tipi rappresentati dalle meta-variabili. Il linguaggio Rust offre due possibilità per rappresentare i vincoli sui tipi:
* Una *notazione compatta* del tipo `<T: Trait1+Trait2>`
* Una *notazione estesa* che fa uso della clausola `where` dopo il nome della struttura o dopo la firma della funzione/metodo, con una sintassi del tipo `where T: Trait1+Trait2.

> Come si può intuire, il + viene utilizzato quando su un tipo ci sono più vincoli dettati da più tratti diversi.

```rust
use core::fmt::Debug;
struct Item<T> where T: Debug{
    item: T
}

impl<T> Item<T> where T: Debug{
    fn new(i: T)->Self{
        Self{item:i}
    }
    fn debugga(&self){
        println!("Debuggo il tipo: {:?}", self.item); 
    }
}

fn main(){
    //T=i32
    let primo = Item::new(34); 
    primo.debugga(); 
    //T=Box<usize>
    let secondo = Item::new(Box::new(2)); 
    secondo.debugga(); 
}
```

##### Tratti e Tipi generici
Nonostante sia i *tratti* che i *tipi generici* sono due modi per implementare il polimorfismo, i due hanno un legame profondo in quanto:
* Uno o più tratti possono essere utilizzati per vincolare l'utilizzo di un tipo generico (come abbiamo visto)
* Si può definire un **tratto generico** ovvero un tratto i cui metodi *ricevono e/o restituiscono* delle variabili di tipi generici che eventualmente sono vincolate da altri tratti.








