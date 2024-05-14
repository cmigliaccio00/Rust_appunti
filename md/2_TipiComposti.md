<style> 
    body{
        text-align: justify; 
    } 
</style>

# `Rust::tipi_composti`
<div style='font-family: Arial; 
            text-align: right; margin-top: -20px; 
            font-size: 14px;
            border-top: 1px solid black'>
    &copy2024 <i>Carlo MIGLIACCIO</i>
</div>


## Sommario
1. [Struct](#struct)
    1.1 [Dichiarazione e uso di una struct](#dichiarazione-e-uso-di-una-struct)
    1.2 [Visibilità](#visibilità)
2. [Moduli](#moduli)
    2.1 [Definizione di un modulo](#definizione-di-un-modulo)
    2.2  [Utilizzo di un modulo](#utilizzo-di-un-modulo)
3. [Metodi](#metodi)
    3.1 [Costruttori](#costruttori)
    3.2 [Distruttori](#distruttori)
        3.2.1 [Il paradigma RAII](#il-paradigma-raii)
    3.3 [Metodi statici](#metodi-statici)
4. [Enum](#enum)
    4.1 [Clausola Match](#enum-e-clusola-match)
    4.2 [Enumerazioni generiche](#enumerazioni-generiche-optiont-e-resultte)


## Struct
Spesso e nei contesti più svariati occorre mantenere uniti insiemi di informazioni eterogenee. A questo punto ci si chiede perché non utilizzare le *tuple*. Queste seppure permettono ugualmente di racchiudere informazioni eterogenee, nascondono la semantica dei campi di cui sono costituite. 
Una `struct` invece è costituita da un **insieme di campi** il cui *nome* e il cui *tipo* sono definiti dal programmatore. Il seguente è un esempio:

```rust
struct MioTipo{
    nome_alunno: String, 
    eta: i32, 
    matricola: usize 
}
```
Si usano per le struct delle convenzioni sui nomi e in particolare: 
* Il nome del tipo (nome della struct) viene espresso in `CamelCase`
* Il **nome dei campi** viene espresso invece in `snake_case`.

##### Dichiarazione e uso di una struct
```rust
#[derive(Debug)]
struct MioTipo{
    nome: String, 
    eta: usize
}
fn main{
    let my_struct = MioTipo{nome: "Carlo".to_string, eta: 34}; 
    println!("Nome {:?} Eta {:?}", my_struct.nome, my_struct.eta);
}
```
Il codice sopra mostra come **dichiarare** e **inizializzare** una struct, i suoi campi sono accessibili tramite la *dot-notation* (`variabile.campo`). 
E' importante notare che <u>ogni `struct` introduce un **nuovo tipo**</u> il cui nome coincide con il nome della struct stessa.
Il Rust consente di avere una certa flessibilità permettendo di:
- Dichiarare delle struct simili a delle tuple nel modo seguente
```rust
struct Data(i32, i32, i32); 
let my_Data=Data(24,7,2000); 
``` 
- Dichiarare **struct vuote** che hanno un tipo analogo a `()` nel modo seguente
```rust
struct StructVuota; 
let my_vec:Vec<StructVuota> = Vec::new();
```

#### Visibilità
Poiché una struct (un Tipo) potrebbe essere utilizzata in una porzione di codice diversa dal modulo in cui essa è stata dichiarata, il linguaggio Rust fornisce un *modificatore di visibilità* per regolare questo aspetto.
Nello specifico, di base tutti i campi di una struct sono **privati** cioè accessibili solo al codice contenuto nel modulo in cui la struttura è stata definita e ai suoi sottomoduli. Per rendere invece accessibile all'esterno la *struct definita* e i *suoi campi* bisogna anteporre ad ognuno la parola chiave **`pub`**. In altri linguaggi orientati agli oggetti come il `Java` la parola 'privato' assume un significato diverso! In Rust non esistono le classi, per cui le <u>regole di visibilità</u> sono riferite *solo* ai moduli.

## Moduli
Abbiamo citato più volte il concetto di **modulo** ne introduciamo l'uso in questo paragrafo. Di base i moduli sono utilizzabili per dividere il codice in *porzioni indipendenti*. I moduli per questa loro caratteristica permettono di implementare il meccanismo di *information hiding*. Per essere però efficace, è utile associare dei *comportamenti* alla struct.
Mentre in altri linguaggi, quali `Java` e `C++` definizione della struttura dati e implementazione risiedono in un unica struttura che è la *classe*, in `Rust`, l'implementazione risiede in un blocco separato introdotto da `impl`.

Il segmento di codice seguente riporta la definizione e l'utilizzo di un modulo.

##### Definizione di un modulo
```rust
module MioModulo{
    pub struct MiaStruct{
        campo1: i32
    }; 
    pub fn funzione_stampa(){
        println!("sono una funzione");
    }
}
```
Per essere utilizzate, la struct e la funzione, hanno bisogno del modificatore `pub` a cui prima abbiamo fatto cenno. Questo le rende pubbliche per l'utilizzo  in porzioni di codice che non si trovano nello stesso modulo  in cui la struttura è stata definita.

##### Utilizzo di un modulo
```rust
use MioModulo::MiaStruct; 
use MioModulo::funzione_stampa;

fn main(){
    let a = MiaStruct(campo1: 15);      //Dichiarazione
    //Utilizzo della funzione del modulo
    funzione_stampa(); 
}
```

## Metodi
I metodi collegati ad un certo tipo si possono definire all'interno di un blocco di codice racchiuso tra parentesi graffe e introdotto da `impl <NomeTipo>`. 
Se all'interno di questo blocco le funzioni hanno come primo parametro uno tra `self`, `&self`, `&mut self` quella funzione diventa un **metodo** per le istanze di quel particolare tipo. 
Anche in `Rust` si è soliti utilizzare la notazione *instance.method()*, dove:
* *instance* è un'istanza del tipo in questione, chiamata anche **ricevitore** del metodo;
* *method* costituisce il nome della funzione.

L'implementazione del metodo ha accesso ai dati della struttura tramite `self` e le sue varianti che mi permettono di specificare **come** il metodo tratta l'istanza della struttura che riceve come parametro.

Il **primo parametro** di un metodo definisce il *livello di accesso* che il  codice del metodo ha sul ricevitore, in particolare:
* `self` indica che il ricevitore viene passato per **movimento** consumando il valore della variabile. E' la forma compatta di `self: Self`;
* `&self`, in questo caso il ricevitore viene passato per **riferimento condiviso**, il dato della struttura NON viene consumato; è una forma compatta per indicare `&self: &Self`;
* `&mut self`, ricevitore passato per **riferimento mutabile**, anche in questo caso è la forma compatta per `&mut self: &mut Self`. 
A questo punto uno si potrebbe domandare: *qual è la differenza tra `self` e `Self`?* E' abbastanza semplice. Il primo rappresenta l'istanza (o il ricevitore) il secondo identifica invece il **tipo** per cui si sta implementando il metodo.

Quando invece un metodo non ha come primo parametro `self`, un suo riferimento o riferimento mutabile, si cotruisce quello che in linguaggi come Java, è rivestito dal ruolo di metodi statici e costruttori.

Riassumiamo ora con un esempio tutti i concetti che abbiamo introdotto.

```rust
//Definizione struttura
#[derive(Debug)]
struct Punto{
    x: 32, 
    y: i32,
    z: i32
}
//Implementazione struttura (comportamenti associati)
impl Punto{
    fn get_x(&self)->i32{
        self.x;
    }
    fn add_one(&mut self){
        self.x+1; 
        self.y+1; 
        self.z+1;
    }
    fn transform(self)->Self{
        Self{x:self.x+3, y:self.y+3, z:self.z+3}
    }
}
//Utilizzo del tipo nuovo
fn main{
    let A = Punto{x:0, y:0, z:0}; 
    //meotodo che usa A come riferimento condiviso
    let x = A.get_x;    
    //metodo che usa A come riferimento mutabile
    A.add_one(); 
    //metodo che usa il ricevitore per movimento
    let B = A.transform();
    //da qui in poi A risulta 'uninitialized'
}
```

#### Costruttori
In `Rust` non esiste il concetto di **costruttore**, un'istanza di un certo tipo può  essere creata in qualsiasi punto del codice. Tuttavia per evitare *duplicazioni del codice*, nelle implementazioni dei tipi vengono definite delle *funzioni di inizializzazione* del tipo `fn new()->Self{...}`. 
Inoltre, dato che in `Rust` non esiste il concetto di overloading delle funzioni, spesso si ricorre a più funzioni di inizializzazione diverse del tipo `fn with_details(...)->Self`

```rust
impl Punto{
    ...     
    //Funzione di inizializzazione
    fn new()->Self{
        Self{x:0,y:0,z:0}
    }
    //Inizializzazione alternativa
    fn with_value(X: i32, Y:i32, Z:i32)->Self{
        Self{x:X,Y:y,Z:z}
    }
    ...
}
//Utilizzo
fn main(){
    let var=Punto::new();m
    let var_b=Punto::with_value(10,10,10);
    ...
}
```

#### Distruttori
Il Rust gestisce il rilascio di risorse acquisite da un'istanza utilizzando il tratto `Drop`, questo ha  associato una funzione `drop(&mut self)->()` che viene chiamato quando una variabile di un certo tipo **esce dallo scope sintattico**. Si può inoltre **forzare il rilascio** delle risorse utilizzando la funzione `drop(mio_oggetto)`, che rilasciando le risorse associate, determina l'uscita della variabile dallo scope sintattico. 

##### Il Paradigma RAII
Nel linguaggio C++ esiste una funzione chiamata **distruttore** che viene introdotta da `~<NomeTipo>`, similmente il compilatore chiama questo metodo quando l'istanza della classe esce dallo scope sintattico o quando si chiama `delete`. 
La presenza del distruttore *abilita* un particolare paradigma detto **RAII**(Resource Acquisition is initialization): *poiché il distruttore viene chiamato nel momento in cui una variabile esce dallo scope sintattico, allora la coppia **costruttore-distruttore** può essere utilizzata affinché determinate operazioni siano fatte in una porzione di codice in cui ci sia una variabile appositamente dichiarata*. 
Il **paradigma RAII** può essere espresso in sintesi nel modo seguente:
* Le **risorse** sono incapsulate all'interno di una struttura in cui:
    * il <u>costruttore</u> acquisisce le risorse e lancia un'eccezione nel caso in cui questo non sia possibile;
    * il <u>distruttore</u> ha il compito di rilasciare solo le risorse senza sollevare alcuna eccezione.
* Una classe che è RAII compatibile:
    * Ha una **gestione automatica** del tempo di vita, oppure
    * E' legato al tempo di vita di un altro oggetto (ad esempio è una parte di esso).

> In realtà il nome del paradigma RAII sarebbe dovuto essere **Resource Finalization is Destruction**(RFID) utilizzato però già in altri ambiti.

In ambiente `Rust` per implementare correttamente il paradigma ci deve essere la garanzia che ogni volta che venga invocato `Drop` ci sia stata una sola costruzione dell'oggetto.
Proprio per questo motivo, il tratto `Copy` è <u>mutuamente esclusivo</u> con il tratto `Drop`.

#### Metodi statici
Sono definite anche *funzioni associate* e si possono introdurre nell'implementazione del tipo semplicemente non mettendo come primo parametro `self`. Come in tutti i linguaggi di programmazione, sono funzione **legate al tipo** più che all'istanza del tipo.
Mostriamo un esempio di metodo statico per il tipo ```Punto``` prima definito

```rust
impl Punto{
    ...
    fn origin()->Self{
        Self{x:0, y:0, z:0}
    }
    ...
}

fn main(){
    let origine=Punto::origin(); 
}
```

## Enum
In Rust è possibile introdurre tipi enumerativi composti da un semplice valore scalare (come in C/C++), ma anche incapsulare **più tuple o più struct** insieme, queste di solite sono volte a specificare ulteriori informazioni su quel valore specifico. 
Come anche il Java permette di fare, è possibile legare dei **metodi** ad un tipo enumerativo con la dicitura `impl` seguito dal nome del tipo. 

Seguono due esempi, uno in cui si fa uso delle struct alla streua del C, l'altro invece al modo di  intendere di Rust.

#### C-like style
```rust
enum HttpResponse{
    Ok=200,
    NotFound=404,
    InternalError=500
}
``` 
In questo caso si stanno assegnando dei valori (scalari) ai singoli 'casi, se questo non venisse fatto i valori assegnati partirebbero dallo  0.

#### Enumerazioni di Rust
```rust
enum HttpResponse{
    Ok, 
    NotFound(String), 
    InternalError{
        desc: String, 
        much: i32
    }
}
```
In questo caso invece una singola struct è formata da un valore vuoto da una stringa e da una struct. I nomi Ok, NotFound e InternalError sono i nomi delle singole varianti della struct. La variante `NotFound` è una `String`, mentre la variante `InternalError` è una struct costituita dai campi `desc`e `much`. Descriviamo qui inoltre la sintassi per inizializzare ogni variante: 

```rust
/*******************Inizializzazione**************************/
let var1 = HttpResponse::Ok; 
let var2 = HttpResponse::NotFound("Non trovato".to_string());
let var3 = HttpResponse::InternalError{
    desc: "Descrizione".to_string(), 
    much: 32
}; 

/*********************Uso nel costrutto match****************/
let mia_var = HttpResponse::Ok;

let ris_comp = match mia_var{
    HttpResponse::Ok => println!("Tutto a posto"), 
    //tra parentesi nome var temporanea
    HttpResponse::NotFound(str) =>{
        println!("Desc_errore: {}", str); 
        str
    } 
    HttpResponse::InternalError{desc: D, much: M} => {
        println!("Info ricavate {} {}", D, M);
        (D, M)
    }
};

/***********************Uso con if-let***********************/
//Analisi di una variante specifica

if let HttResponse::NotFound(ris_str) = mia_var{
    println!("Beccata la variante HttpResponse_NotFound"); 
}
```


> **Nota che:**  
Enum è definito come **tipo somma**, in quanto l'insieme dei valori che può contenere è dato dall'unione dei valori che possonono contenere le singole varianti. Le struct al contrario sono un **tipo prodotto**, in quanto l'insieme dei valori che possono rappresentare è dato dal *prodotto cartesiano* dei domini dei singoli campi. 

### Rappresentazioni in memoria degli `enum`
La *rappresentazione in memoria* di un enum in Rust dipende dalla natura delle sue varianti.
Rust utilizza un implementazione dei tipi enumerativi basata su **tag**: ad ogni variante è legato un byte che la identifica, nel caso di enum definite in questo modo
```rust
enum Colore{Red, Green, Blue}
```
Viene allocato in memoria quindi, lo spazio per il tag e quello per un intero. Nel caso invece di un tipo enumerativo in cui sono presenti delle varianti con delle dimensioni eterogenee, in memoria viene riservato:
* Lo spazio per il tag
* Lo spazio per la variante più grande. Nel caso in cui la somma `1 Byte + Dim. variante più grande` non sia una potenza di due, vengono aggiunti dei byte di *padding* per una questione di allineamento di memoria.

Di seguito un esempio di rappresentazione per la seguente enum
```rust
enum Shape{
    Square(u32),
    Point{x:u8, y:u8}, 
    Empty
}
```
Qui la prima variante occupa un solo byte, la seconda variante occupa 4 byte, mentre la terza 0 byte.
Viene allocato quindi lo spazio per la variante più grande che è così composto:
* **1 byte** per il Tag identificativo
* **4 byte** per rappreseentare l'intero
* **3 byte** di *padding* affinché la somma (8) sia una potenza di due. Di seguito la figura della rappresentazione in memoria

![](/img/enums.png)

> **Utility**
Per poter stampare il valore associato ad una struct del tipo
`enum Prova{A, B, C}` bisogna utilizzare un cast del tipo `Prova::A as i32`.

#### Tipi enumerativi e clausola `match`
I tipi enumerativi si prestano bene ad essere usati insieme al costrutto `match`, la possibilità di differenziare i diversi casi e di utilizzare delle variabili temporanee offre una maggiore compattezza del codice. 

##### Esempio
```rust
#[allow(dead_code)]
enum Errore{
    Connessione(String),
    Hardware{which: String, messaggio: String}
}
impl Errore{
    fn explain_error(&self){
       match self{
            Errore::Connessione(mia_str)=>{
                println!("Messaggio estratto: {}", mia_str); 
            }, 
            Errore::Hardware{which: quale, messaggio: mess} => {
                println!("Valori estratti: {}, {}", quale, mess); 
            }
        } 
    }
}
fn main(){
    let a=Errore::Connessione("Errore generico di connessione".to_string()); 
    let b=Errore::Hardware{
        which: "Scheda Madre".to_string(), 
        messaggio:"Fusione di alcuni circuiti integrati".to_string()
    }; 
    a.explain_error();
    b.explain_error();
}
```
Nell'esempio appena mostrato:
1. Viene introdotto il tipo enumerativo `Errore` costituito da due varianti (Connessione e Messaggio), uno associato ad una stringa, l'altro ad una struct con due campi; 
2. Viene data l'implementazione del tipo che include un singolo metodo `explain_error()`
3. L'unico metodo associato al tipo analizza le diverse varianti del dato con il costrutto `match` che come ricordiamo deve essere esaustivo, cioè deve analizzare tutti i valori del dominio di una certa variabile.
4. Le variabili utilizzate nella differenziazione dei diversi casi sono *variabili temporanee* introdotte per estrarre il dato specifico associato ad una certa variante.

#### Destrutturazione di un tipo enumerativo
L'uso del costrutto `match` non è l'unico per esplorare le diverse varianti di un tipo enumerativo. Possono essere utilizzati in particolare i seguenti costrutti:
1. `if let <pattern> = <valore> {...}`
2. `while let <pattern> = <valore> {...}` 

Il seguente esempio utilizza questa alternativa:

```rust
#[allow(dead_code, non_snake_case)]
enum Color{
    Red(usize), 
    Green(usize), 
    Blue(usize)
}

fn main(){
    let my_color = Color::Red(50); 

    if let Color::Red(intensita) = my_color{
        println!("Colore: Rosso, Intensita: {}", intensita);
    }
}
```

La riga di codice `if let Color::Red(intensita) = my_color` va letta in questo modo:
* Se la variabile `my_color` è la variante `Color::Red`
* Metti il valore dell'intero associato nella variabile temporanea denominata `intensita`.

#### Enumerazioni generiche: `Option<T>` e `Result<T,E>`
Rust offre alcune enumerazioni che sono parametrizzate da tipi generici (*prossimo capitolo*) e che sono alla base della libreria standard. Questi sono: `Option` e `Result`.

#### `Option<T>`
Permette di gestire i casi in cui c'è presenza o assenza di un certo tipo di valore. Ha due possibili varianti:
* `Some(T)` è il caso in cui un valore è presente e il tipo associato è T; 
* `None` variante vuota, denota un fallimento dell'estrapolazione del dato.

#### `Result<T,E>`
Si usa per notificare il risultato di una computazione, la si può trovare in una delle seguenti varianti:
* `Ok(T)`, in questo caso il valore restituito ha tipo T(tipo generico) 
* `Err(E)`, nel caso in cui la computazione fallisca il valore ritornato è di tipo E che viene utilizzatpo per la descrizione dell'errore.

## Esempio ulteriore...

```rust
#[derive(Debug)]
enum Error_Yogurt{
    Confezione(String), 
    Contenuto(String)
}
impl Error_Yogurt{
    //decapsula l'errore dalla variante enumerativa
    fn unwrap_error(self)->String{
        match self{
            Self::Confezione(s)=>{s},
            Self::Contenuto(s)=>{s}
        }
    }
    
}
#[allow(non_snake_case)]
fn Valuta_Yogurt(msg: String) -> Result<i32, Error_Yogurt>{
    if msg=="Tutto ok"{
        Ok(27)
    }
    else if msg=="err_conf"{
        Err(Error_Yogurt::Confezione("Errore nella confezione".to_string()))
    }
    else {
        Err(Error_Yogurt::Contenuto("Errore nel contenuto".to_string()))
    }
}
fn main(){
    let mio_msg1 = "Tutto ok".to_string();
    let mio_msg2 = "err_conf".to_string(); 
    let mio_msg3 = "altro_msg".to_string();
    let R1 = Valuta_Yogurt(mio_msg1); 
    let R2 = Valuta_Yogurt(mio_msg2); 
    let R3 = Valuta_Yogurt(mio_msg3);
    
    if let Ok(var) = R1 {
        println!("Computazione corretta, valore ritornato: {}", var);
    }
    if let Err(errore) = R2 {
        println!("Errore: {} ", errore.unwrap_error()); 
    }
    if let Err(errore) = R3 {
        println!("Errore:  {} ", errore.unwrap_error());
    }
}
```




