</style>

# `Rust::processi`
<div style='font-family: Arial; 
            text-align: right; margin-top: -20px; 
            font-size: 14px;
            border-top: 1px solid black'>
    &copy2024 <i>Carlo MIGLIACCIO</i>
</div>

## Processi
Un **processo**, nell'ambito di un sistema operativo, costituisce l'unità base di esecuzione. Esso, praticamente sempre, viene identificato da un _PID_ (Process Identifier) che, in quanto tale è unico tra i processi in esecuzione. Un processo, a differenza di un thread, è dotato di uno **spazio di indirizzamento** all'interno del quale possono "alloggiare" uno o più thread che come abbiamo visto sono dei <u> flussi di esecuzione </u> che sono schedulabili in modo indipendente. 
Il **livello di isolamento** offerto da un processo è già ad un livello più alto rispetto ai thread (che condividono invece lo spazio di indirizzamento con altri thread dello stesso processo), ma comunque questo isolamento è di tipo **parziale**, in quanto ci sono degli oggetti che quando sono **in contesa** possono generare delle *interferenze*. 
Come vedremo certe volte il fatto che ci sia 'meno isolamento' tra più processi è desiderabile in quanto ad esempio voglio che due processi in esecuzione possano scambiarsi delle informazioni.

> Un **esempio pratico**: supponiamo di voler gestire tramite le scorciatoie di sistema un semplice copia e incolla tra i programmi Word ed Excel del pacchetto Microsoft Office. Poiché questi sono sicuramente **due processi diversi** in qualche modo devono passarsi queste informazioni...

I processi sono intrinsecamente legati al concetto di thread per il semplice fatto che un processo è costituito da almeno un flusso di esecuzione, in base a quanti flussi di esecuzione possiede un processo lo si può identificare come **processo single thread** o come **processo multi-thread**. 

In ogni caso non sempre è desiderabile che ci sia un unico spazio di indirizzamento condiviso, come avviene per i thread questo perché ci si espone a dei *rischi sul piano della sicurezza*. Prove di questa osservazione possono essere le seguenti:
- Il browser *Google Chrome* quando si tengono aperte più finestre crea per ognuna di quelle un processo diverso, questo per motivi di sicurezza; 
- Nell'ambito di ambienti di sviluppo (IDE) è desiderabile che ci siano più moduli adibiti ognuno ad una fase che porta il codice sorgente a diventare un programma. Ad esempio il **compilatore** e il **linker**, che essendo moduli diversi, saranno associati loro due processi distinti e la comunicazione tra questi due può avvenire ad esempio tramite un file di appoggio su cui l'uno trasferisce le informazioni che necessita l'altro.

Le operazioni di base che si possono fare con i processi (creazione, terminazione, attesa) hanno una semantica molto diversa coerentemente con il sistema operativo utilizzato (Windows, Unix-Like...). Il concetto è che
$ \text{Sistemi Operativi distinti} \to \text{Filosofia distinta}$.


### Processi in Windows
Partiamo analizzando il caso di Windows. In questo contesto, per quanto la creazione del processo possa essere più articolata, il processo risultante ha caratteristiche da certi punti di vista 'migliori' rispetto ai sistemi Unix-Like.
In primis, in ambiente Windows il processo che crea e il processo creato non hanno nessuna relazione di parentela esplicita; la creazione di un processo può avvenire con la funzione  `CreateProcess(...)` o `CreateProcessExec(...)` che: 
1. Crea un **nuovo spazio di indirizzamento**
2. Lo inizializza con l'immagine dell'eseguibile
3. Ne attiva il thread primario

Il processo creato può vedere chiaramente le **variabili d'ambiente** e altre strutture come i **file-handler**, mentre non può condividere handle a thread, processi o altre regioni di memoria.

### Processi in Linux
Come anticipavamo qui la filosofia che soggiace la creazione e gestione di un processo è molto diversa da quella che vale in ambiente Windows.
Un nuovo processo lo si crea con la primitiva `fork()` che:
0. Stabilisce in primis una relazione padre-figlio tra il processo che crea e il processo creato. Infatti la funzione viene chiamata una sola volta, ma ha due uscite: (i) ritorna 0 se si trova nel processo figlio, (ii) il pid del processo creato in caso si tratti del processo padre, (iii) -1 in caso d'errore;
1. Crea un nuovo spazio di indirizzamento che costituisce però un **duplicato** ripsetto a quello del processo genitore;
2. Il processo child inizia la sua computazione in cui si trova uno **stack già popolato** con le chiamate del padre, stesso discorso vale per lo **heap**, per il **codice** e le variabili globali.

La duplicazione avviene in modo molto efficiente seguendo la filosofia **Copy-On-Write** (CoW) secondo cui: all'inizio parent e child condividono il riferimento alle stesse pagine fisiche, però nel momento in cui uno dei due effettua delle modifiche, le pagine fisiche modificate corrispondenti vengono copiate e su una delle due copie viene effettuata la modifica. Questa tecnica risulta molto efficiente tutte le volte che i processi non devono effettuare delle modifiche.
Questo è quanto accade all'invocazione della chiamata di sistema `fork()`. Nel momento in cui però nel processo figlio viene eseguita una chiamata ad una delle funzioni `exec*()` (l'asterisco raggruppa tutte le versioni che sono differenti per il modo in cui ricevono i parametri). Una generica funzione `exec*()` rimpiazza nello spazio di indirizzamento un eseguibile passato come parametro.

Fin qui abbiamo presupposto che il processo padre avesse un unico flusso di esecuzione, ma nella maggior parte dei casi NON è così! Infatti il processo padre può avere più thread mentre il processo figlio ne conterrà uno solo, inoltre gli strumenti di sincronizzazione utilizzati potrebbero trovarsi poi in uno stato inconsistente! A questo scopo c'è la funzione 
```c
int pthread_atfork(
    void (*prepare)(void),      //Eseguita prima della fork()
    void (*parent)(void),       //Eseguita solo in parent
    void (*child)(void)         //Eseguita solo in child
)
```

Queste tre funzioni chiamate (in ordine), prima della chiamata a `fork()` nel processo padre, solo nel processo padre e solo nel processo figlio hanno proprio la funzione di preparare le entità coinvolte nella chiamata della fork.

## Terminazione di un processo 
Un processo può terminare dopo la chiamata a `ExitProcess(int status)` (Windows) o `_exit(int status)` enrtrambi causano la **terminazione immediata** di tutti i thread associati al processo chiudendo tutti i file aperti, in Windows vengono inoltre sganciate tutte le DLL precedentemente caricate, queste due funzioni <u> terminano il processo e basta </u>. La libreria standard del C/C++ offre delle possibilità più interessanti con la funzione `void exit(int status)` che hanno la possibilità di avere registrate alcune **callback()** tramite la funzione `int atexit(void (*callback()))`, il valore restituito permette di stabilire se l'esecuzione di questa è andata a buon fine o meno.

Un **altro motivo** per cui un processo termina è quando la **fnzione principale** (main) ritorna. In questo particolare caso  <u> si utilizza il valore ritornato dal main per chiamare la funzione `exit(status)` e passarglielo come parametro </u>. (Il valore del codice di ritorno è arbitrario, tuttavia il valore 0 indica di solito una terminazione senza problemi).

## Processi in Rust
Dopo questa lunga introduzione possiamo vedere finalmente come si creano e si gestiscono i processi in Rust. Qui la standard library mette a disposizione il modulo denominato `std::process`.
La struct `Command`, e i metodi ad essa associati, permette la creazione di un nuovo processo, in particolare si utilizza il (design) pattern *builder* per **creare** il processo figlio ed interagirvi. 
I metodi `arg()` e `args()` permettono di passare uno o più argomenti al processo che si sta creando. 
Esistono poi diversi modi, dopo averne definito le caratteristiche di creare effettivamente un processo, il primo che vediamo usa il metodo `output(&self)` applicato a un oggetto creato con `Command::new("Nuovo_Processo")` permette di **generare il processo** e **aspettarne la terminazione** raccogliendo il risultato nella monade `Result<Output>` che è fatta nel modo seguente:
```rust
pub struct Output{
    pub status: ExitStatus,
    pub stdout: Vec<u8>, 
    pub stderr: Vec<u8>
}
```

E' possibile stabilire la ridirezione dei flussi di I/O tramite i metodi `stdin()`, `stdout()` e `stderr()`, a ognuno di questi è possibile passarvi le seguenti opzioni:
- `ineherit()` $\to$ il processo figlio eredita il descrittore corrispondente **usato nel processo padre**.
- `piped()` $\to$ in questo caso viene creata una **pipe monodireziaonale** (dal figlio verso il padre) in cui un'estremità è posseduta dal figlio un'altra estremità invece è memorizzata nella struct del comando che 'avvia' il processo.
- `null()` con tale opzione il flusso corrispondente (di input o di output) sarà ignorato.

Gli altri due modi per generare un processo sono: 
- tramite il metodo `status(&mut self)` che avvia il processo e ne attende la terminazione raccogliendo il risultato in una monade di tipo `Result<ExitStatus>` che nel caso di sistemi Unix-Like contiene delle informazioni su COME quel processo è terminato. 
- tramite il metodo `spawn(&mut self)` che avvia il processo ma non ne attende la terminazione. Esso restituisce un risulstato di tipo `Result<Child>` dove `Child` è la struttura tramite la quale è possibile interagire con il processo creato.

La `struct Child` offre vari meccanismi per controllare lo svolgimento del processo figlio:
1. Qualora questi siano stati dichiarati non nulli gli attributi `stdin, stdout, stderr` rappresentano gli **handler ai relativi flussi**; 
2. Il metodo  `id(&self)` restituisce l'identificativo univoco assegnato dal sistema operativo.
3. Il metodo `wait(&mut self)` permette di aspettare la terminazione del processo figlio; 
4. Il metodo `wait_with_output(&mut self)` ha delle conseguenze particolari in quanto: chiude il flusso d'ingresso del processo figlio ne attende la terminazione racccogliendo in una struttura di tipo `Output` definita come prima quanto non letto dei  flussi di uscita ed errore.
5. Il metodo `kill()` permette di terminare il processo figlio.

Il seguente segmento di codice permette di creare un nuovo processo con flussi piped di ingresso e di uscita.
```rust
use std::process::Command; 

let mut child = Command::new("rev")
        .stdin(Stdio::piped())
        .stdin(Stdio::piped())
        .spawn()
        .expect("Failed to spawn process"); 
```

### Terminazione di un processo in Rust
La *terminazione di un processo* in Rust può avvenire in modi diversi: 
1. Usando la funzione `std::process::exit(code: i32)->!`, in questo caso si termina immediatamente il processo e tutti i thread associati ad esso, dato che non viene eseguito nessun distruttore, è responsabilità del programmatore chiamare questa funzione quando si ha la certezza di avere liberato tutto quello che era necessario. Come si può notare, la funzione riceve come parametro un codice a 32 bit, ma in ambiente Linux, solo gli 8 bit meno significativi vengono **passati al sistema operativo**.
2. Usando la funzione `std::process::abort()->!` che termina immediatamente il processo con i thread di cui è costituito rendendo esplicito il fatto che si sia verificata una **interruzione anomala** del processo tramite un *codice di default*.
3. La macro `panic!()` a differenza delle altre due causa la **contrazione dello stack** del thread corrente e l'esecuzione dei distruttori posti al suo interno senza la terminazione del processo. C'è una solaa eccezione a questo:  quando la chiamata di `panic!()` viene fatta nel thread principale.


## Gestione di altri processi: `wait()/waitpid()`
Tutti i sistemi operativi offrono dei meccanismi per *aspettare la terminazione di processi* di cui si possiede una handle. La libreria standard mette a disposizione le due funzioni:
1. `pid_t wait(int *status)` che:
a.  Se il processo non ha processi figli, termina con il codice -1; 
b. Se il processo ha dei figli nessuno dei quali è terminato, la chiamata si blocca in attesa che **uno dei suoi figli** temrini. 
c. Quando vi è un **figlio terminato** se il parametro `status` non è nullo, viene utilizzato per raccoglierne il motivo di terminazione, si aggiornano i **contatori di utilizzo del sistema** aggiungendo quelli del processo figlio, il valore di ritorno è quello del processo terminato, chiaramente una ulteriore chiamata a `wait()` del processo padre  non fornirà più le indicazioni relative al processo appena analizzato. 

2. `pid_t waitpid(pid_t pid, int *status, int options)` permette di indicare che l'attesa deve essere per uno specifico  figlio il cui pid è passato come parametro, il puntatore ad intero `status` se non nullo punta ad un intero a 16 bit formato da una serie di sotto-campi che danno ulteriori informazioni sul motivo per cui un certo processo è terminato.

La terminazione prematura del processo padre o del processo figlio determina:
- Un **Processo Orfano** nel caso in cui il processo padre termina prima dei processi figli senza attenderne quindi la terminazione. Nei moderni sistemi operativi in questo caso si assegna come padre di tale processo il processo `init` che ha PID=1; 
- Un processo si dice invece **Zombie** se termina prima che il padre abbia invocato la `wait/waitpid`. In questi casi questa situazione viene gestita nei sistemi operativi in modo che venga liberato lo spazio di indirizzamento di quei processi e reso disponibile ad altri, mentre le strutture del kernel (handle vari, PCB...) vengono allocati nel momento  in cui il processo padre ne effettua la wait.

A questo punto, una volta che ci siamo occupati della creazione, distruzione e attesa di processi, non ci resta che esplorare i **meccanismi attraverso i quali si può superare il livello di isolamento tra i processi** parlando nello specifico di tecniche note come di **IPC** (Inter-Process communication). Di fatto **qualunque sia il tipo di meccanismo adottato** occorre adattare le informazioni scambiate in un modo che siano comprensibili al destinatario in formati basati su testo (CSV, JSON...) o formati binari. Chiaramente in questo contesto il crate `serde` (di cui abbiamo parlato nel capitolo dedicato all'I/O) può essere utilizzato.

I dati vengono scambiati quindi nel formato esterno e l'operazione che un processo fa per esportare i dati verso un processo esterno viene anche detto *marshalling*, mentre l'operazione inversa che fa il processo ricevente di 'decompattare' i dati e utilizzarli viene detta *unmarshalling*. 

Analizziamo ora brevemente due tecniche di IPC: (i) **coda di messaggi**, (ii) **pipe**.

## 1) Coda di messaggi 
E' nota con il nome di `MailSlot` (Windows) o `FIFO` (Linux). Permette a **molti processi di origine** di scambiare i messaggi con **uno specifico destinatario**. Il tipo di comunicazione che avviene tramite l'utilizzo di questo strumento è di tipo: **asincrono** (il SO si occupa di tenere il messaggio finché non è stato letto dal destinatario) e **mono-direzionale** (il processo destinatario non invia delle risposte).
*Come identificare una coda?* Avviene tramite un identificativo univoco (stringa) che il destinatario ha il compito di inizializzarla, i destinatari devono conoscere il nome di questa coda anche facendo uso di altre tecniche di IPC (esempio pipe)

## 2) Pipe
Costituisce un meccanismo abbastanza diverso rispetto al precedente. Una pipe è un "tubo" che permette il trasferimento di **sequenze di byte** di lunghezza arbitraria, realizza un tipo di *comunicazione sincrona* e *1-1* (uno inietta l'altro preleva, ecco perché pipe). Cosa importante è che i messaggi abbiano dei marcatori che consentano di delimitarli.

Sia in Windows, sia in sistemi Unix-Like ci sono le primitive per la realizzazione di entrambe queste tecniche di IPC. Rust mette a disposizione il crate **interprocess** che consente di gestire in modo multipiattaforma il concetto di *comunicazione tra processi* garantendo che ci sia ugualmente il supporto alle funzionalità dei singoli sistemi operativi.



