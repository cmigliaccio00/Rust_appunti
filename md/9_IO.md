<style> 
    body{
        text-align: justify; 
    } 

    h2{ margin: 0px; }
</style>

# `Rust::io`
<div style='font-family: Arial; 
            text-align: right; margin-top: -20px; 
            font-size: 14px;
            border-top: 1px solid black'>
    &copy2024 <i>Carlo MIGLIACCIO</i>
</div>

## File e file system
Dall'Avvento dei Sistemi Operativi si è avuta da sempre l'esigenza di mantenere nel tempo delle informazioni, quindi delle metodologie per la gestione della persistenza. I file sono lo strumento che asseriscono a questo compito, essi non sono altri che "pacchi di byte" a cui viene associato un *Path* che ne costituisce il suo **nome**. 
Da un'analisi preliminare dei sistemi operativi più conosciuti, risulta che la gestione dei file è un qualcosa di molto più complicato di quanto sembri.

In C non c'erano funzioni integrate nella libreria standard che permettessero la navigazione del file system e la manipolazione di file, neanche il C++ nelle sue primissime versioni aveva tale supporto che invece è stato introdotto con la versione *C++ 17*. Il linguaggio Java invece sin dalla prima versione ha offerto delle classi e dei metodi standard per la gestione dei file. 

Per quanto riguarda il Rust invece, esiste un *crate* della libreria standard `std::fs` che permette di disporre di una serie di astrazioni legate al concetto di file, prima fra tutte quella offerta dal tipo (struct) `std::File`.

## `Path` e `PathBuf`
Il Rust pone tra i suoi obiettivi l'*indipendenza dalla piattaforma* per cui dispone di due particolari tipi, simili alle stringhe che sono però specializzati per la gestione e la manipolazione delle informazioni e metadati associati ai file. 

Il principio di base è che ogni sistema operativo ha un modo proprio per definire un file, le struct `std::path::Path` e `std::path::PathBuf` **nascondono tali differenze** fornendo dei meccanismi portabili per segmentare un certo path e ricavare le informazioni. 

* `Path` $\Longrightarrow$ si lega al concetto di `str`, è un tipo *unsized* che NON può essere modificato, ma solo referenziato in lettura; 
* `PathBuf` $\Longrightarrow$ si lega al concetto di `String` in quanto possedendo il suo dato, può anche modificarlo.

I metodi forniti da questi due tipi risultano *molto utili* per la realizzazione della *System Integration* (di seguito la definizione dalla pagina di *Wikipedia*).

>**Definition. System integration** is defined in engineering as the process of bringing together the component sub-systems into one system (an aggregation of subsystems cooperating so that the system is able to deliver the overarching functionality) and ensuring that the subsystems function together as a system, and in information technology as the process of linking together different computing systems and software applications physically or functionally, to act as a coordinated whole.

## Navigare il File System 
Il crate `std::fs` delle funzioni di base per la navigazione, creazione e rimozione delle directory. Queste sono: 

```rust
std::fs::read_dir(dir: &Path)->Result<ReadDir>
std::fs::create_dir(dir: &Path)->Result<()>
std::fs::remove_dir(dir: &Path)->Result<()>
```

1. **`std::fs::read_dir(dir: &Path)->Result<ReadDir>`**, in caso di successo ritorna un *iteratore* agli elementi della directory sottoforma di elementi di tipo `std::fs::DirEntry` che hanno una funzione descrittiva dei file contenuti nella cartella.
2. **`std::fs::create_dir(dir: &Path)->Result<()>`** crea una cartella dato il nome, restituisce un `Result` per gestire le situazioni in cui non si disponga delle autorizzazioni necessarie o se la cartella già esiste.
3. **`std::fs::remove_dir(dir: &Path)->Result<()>`** rimuove una cartella dal file system dato il path a patto che: 
    * La cartella esista
    * La cartella sia vuota
    * Si disponga dei permessi necessari per fare l'operazione di rimozione.

## Manipolazione dei file
Il crate `std::fs` mette a disposizione inoltre delle funzioni per la manipolazione dei file e in particolare per copiarli, spostarli o rimuoverli, similmente a quanto descritto per le directory, i metodi associati a queste operazioni ritornano dei Result per la corretta gestione di *situazioni di errore*.

1. La funzione `std::fs::copy(from: &Path, to: &Path)->Result<i64>` copia il contenuto di un file di Path pari a `from` in un file di path pari a `to`, nel caso l'operazione vada a buon fine viene ritornato il numero di byte letti.

2. La funzione `std::fs::rename(from: &Path, to: &Path)->Result<()>` permette di spostare un file da un path all'altro. Nel caso in cui i due percorsi facciano riferimento ad uno stesso volume del file system, allora vengono modificate poche informazioni e il file precedente viene eliminato. Nel caso in cui invece i due percorsi si trovino su volumi differenti, viene effettuata una copia del file originale e quello nuovo avrà nome specificato nel parametro `to`. 

3. La funzione `std::fs::remove_file(path: &Path)->Result<()>` rimuove un file. L'operazione può essere rimandato ritorndo un `Result::Err` nel caso in cui il file sia ancora in utilizzo.

## Operazioni con i file
Con questi primi paragrafi abbiamo visto in generale quali sono le funzioni del crate std::fs che ci permettono di interagire con il file system permettendo la gestione di percorsi, la manipolazione di cartelle e manipolazione di file (intesa come posizione che questi occupano all'interno del file system).  Più frequentemente però si è interessati alla lettura e manipolazione del **contenuto** di un file in termini di byte in esso contenuti.

La manipolazione dei byte contenuti in un file è completamente mediata dal Sistema Operativo. Tramite una chiamata di sistema  si fa una richiesta di *aprire* (open) un certo file, il sistema operativo, se possibile, mette a disposizione della funzione chiamante un *handler* o un *file descriptor* come riferimento opaco per agire sul contenuto del file stesso. 

Il tipo (struct) `File` mette a disposizione due metodi di base per aprire un file: `open()` e `create()` che agiscono in modo leggermente diverso, sono entrambe delle funzioni generiche che in caso di successo restituiscono un'istanza di tipo `File`.

1. **`open(path: P)->Result<File> where P: AsRef<Path>`** a condizione che il file indicato esista, lo apre restituendone l'handler.
2. **`create(path: P)->Result<File> where P: AsRef<Path>`**, fa un'operazione leggermente più complicata rispetto a open e in particolare se il file indicato dal path già esiste lo tronca, altrimenti lo crea e lo apre in scrittura.

La struct `std::fs::OpenOption` permette inoltre di specificare delle opzioni sull'apertura del file (lettura, scrittura, append...), queste opzioni una volta creata un'istanza con `OpenOptions::new()`, possono essere concatenate con la dot notation, l'apertura del file con la funzione open, a cui si passa il path termina questa catena. 

```rust
    let mut mio_file = OpenOption::new()
                        .<Opzione1>(<par>)
                        .<Opzione2>(<par>)
                        . ...
                        .open("/mioPath"); 
```

> Nel paragrafo che segue vengono mostrate le funzioni di base per leggere/scrivere un file dopodiché sono presentati in generale i **tratti** che possono essere utilizzati per le operazioni di I/O. 

## Leggere e scrivere un file 

Le seguenti due funzioni permettono di leggere e scrivere il contenuto di un file di moderate dimensioni, cioè che abbia delle dimensioni che siano compatibili con lo  spazio allocabile al processo, per questo motivo per utilizzare tali funzioni occorre essere certi che il buffer da leggere/da scrivere su file possa essere ospitato nello spazio indirizzabile al processo.

* `read_to_string(path: &Path)` scarica il contenuto dell'intero file in una stringa
* `write(path: &Path, contents: &[u8])` scarica lo slice `contents` sul file avente come path  il path indicato come parametro.

## Tratti relativi ad Input/Output
In Rust le principali operazioni di I/O sono gestite tramite l'utilizzo di alcuni tratti che offrono *metodi di base* per le operazioni di lettura e scrittura. 
Questi tratti sono essenzialmente quattro e sono: `Read`, `BufRead`,`Write` e `Seek`.
In generale se le operazioni di base abilitate dall'implementazione di questi tratti  producono degli errori, questi sono del tipo enumerativo `ErrorKind` che contiene molte varianti ognuna delle quali afferisce alla descrizione di un certo tipo di errore.

<center>
    <img src='/img/IO_Traits.png' width='60%'>
</center>


### `std::io::Read`: leggere un flusso di byte
Per l'implementazione di questo tratto basta fornire l'implementazione del metodo `read(buf: &mut [u8]) ->Result<usize>` tutti gli altri metodi sono implementati in base all'implementazione di questo metodo. Chiaramente ogni chiamata di `read()` può scatenare una chiamata di sistema con successivo cambio di contesto e questo può portare a dei cali di performance inaccettabili.
Altri metodi messi a disposizione da questo tratto sono ad esempio `read_to_end()`, `read_to_string()` (utilizzato in precedenza), `read_exact()` che permette di leggere un numero preciso di byte da un file e così via. Un altro utile metodo è costituito da `bytes()` che permette di estrarre un iteratore sugli elementi del file stesso.

### `std::io::BufRead`: ottimizzazione della lettura
Abbiamo visto che in generale la chiamata del metodo `read` del tratto `Read` può causare ogni volta una chiamata di sistema con rispettivo calo delle performance. Il tratto `BufRead` per sopperire a questa carenza offre dei metodi che tendono a migliorare le operazioni di input output che si servono di un buffer di memoria, che quando saturo, provvederà a scatenare la system call necessaria.
L'implementazione del tratto prevede l'implementazione dei metodi `fill_buf()` e `consume()`, il primo che ritorna il contenuto del buffer in memoria, il secondo ritorna il numero specificato di bytes. 
Questi metodi non sono chiamati esplicitamente di solito, più utilizzati sono invece i metodi `read_line()` e `lines` che permettono di leggere il contenuto di un file riga per riga.

```rust
trait BufRead{
    fn fill_buf();
    fn consume();
    fn read_line(&mut self, buf: &mut String);
    fn lines(self)
}

let input = File::open("mioPath"); 
let buffer_reader = BufReader::new(input); 
buffer_reader.read_line()
```

## `std::io::Write`
Permette di scrivere un flusso di dati, richiede l'implementazione dei metodi `write()` e `flush()`. Il primo prova a scriver l'intero buffer passato come parametro e ritorna il numero di byte di cui è riuscito effettivamente ad affettuare la scrittura, il secondo serve per **finalizzare l'output** garantendo che eventuali buffer transitori siano correttamente svuotati. Infine citiamo `write_all()`, che chiama ricorsivamente `write()` finché tutti i byte del buffer passato come parametro non sono stati scritti, altrimenti ritorna un errore fatale.

## `std::io::Seek`
E' il tratto che offre i metodi di base utili a riposizionare il **cursore** di lettura/scrittura di un *flusso di byte*. 

Quando il flusso attinge ad un *dato di dimensione nota* è possibile riposizionare il cursore in modo relativo rispetto:
* All'**inizio del flusso**: esempio `SeekFrom::Start(n: u64)`
* Alla **fine del flusso** esempio `SeekFrom::End(n: u64)`
* Alla **posizione corrente del flusso** ad esempio `SeekFrom::Current(n: 64)`.
Queste tre varianti possono essere passate al metodo seek.

I metodi offerti dal tratto sono: 
* `seek()` per posizionare il cursore alla posizione `pos` passata come parameetro
* `stream_position()` restituisce la posizione corrente rispetto all'inizio del flusso.

## Serializzazione e Deserializzazione: il framework `Serde`
Sebbene sia possibile agire sul contenuto di un file a livello di singoli byte, a volte può essere sconveniente farlo. 
Rust mette a disposizione, tramite il framework `Serde` (sta per *Serialization-Deserialization*), una serie di funzioni efficienti per <u>serializzare</u>, <u>deserializzare</u> verso e da file di *formati standard* (quali JSON, Avro, CSV...). In particolare mette a disposizione i tratti `serde::Serialize` e `serde::Deserialize` che possono essere implementati o derivati dai tipi standard al fine di poter essere serializzati/deserializzati in modo opportuno.

Per poter utilizzare tale framework c'è bisogno di introdurre nella sezione `[dependences]` le dipendenze opportune con le indicazioni su versioni e features. Bisogna utilizzare invece la direttiva `#[derive()]` per fare in modo che un tipo *user-defined* sia serializzabile.
Di seguito un esempio:

```rust
#[derive(Serialize, Deserialize, Debug)]
struct Test{
    alfa: String, 
    beta: String
}
```


