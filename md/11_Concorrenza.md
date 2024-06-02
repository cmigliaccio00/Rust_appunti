<style> 
    body{
        text-align: justify; 
    } 

    h2{ margin: 0px; }
</style>

# `Rust::concorrenza`
<div style='font-family: Arial; 
            text-align: right; margin-top: -20px; 
            font-size: 14px;
            border-top: 1px solid black'>
    &copy2024 <i>Carlo MIGLIACCIO</i>
</div>

## Programmazione concorrente
Un **programma concorrente** è caratterizzato da due o più flussi di esecuzione contemporanei, questi possono essere eseguiti su _core indipendenti_ nel caso in cui si disponga di più core sullo stesso elaboratore altrimenti sono sotto il controllo di uno schedulatore. 

All'atto della sua creazione un processo dispone di un **unico thread** cioè di un _unico flusso di esecuzione_ questo può richiedere di creare altri thread. 
La cosa fondamentale è che ad ogni thread è associata una  *computazione indipendente*.
Thread nativo $\iff$ coinvolgimento del sistema operativo + schedulazione
"Green thread" $\iff$ schedulazione fatta a **livello utente**

### Thread nativi

Interazione programma $\leftrightarrow$ insieme dei thread. Operazioni di base:
* _Creazione di un thread_ un processo all'atto della sua creazione coincide con il thread principale, la funzione di creazione invece mi permette di ricevere un _handler_ ad un thread creato (nello specifico mi viene dato il riferimento ad una struttura che contiene le informazioni del thread appena creato).
* _Identificazione del thread_ (simile a PID) (Thread Identificator) $\iff$ TID
* _Attesa_ terminazione del thread, corretta? con errore? 

>**Nota che:** non esiste la _cancellazione_ del thread. Esiste la kill del processo, che comporterebbe la  terminazione di TUTTI i thread ad esso associati. Cosa fare? $\Longrightarrow$ serve un modo per realizzare questa funzione tra i thread stessi **in modo cooperativo**.

## Implicazioni della concorrenza
1. Complessità computazionale aggiunta
2. Complicazioni a livello software aggiunte

Sono Ok, quando si introducono effettivamente dei benefici. Quanto realmente ci guadagno introducendo nel mio software la concorrenza?

##### Regola generale
1. **Thread** $\to$ bisogna fare delle cose insieme
2. **Programmazione asincrona** $\to$ bisogna attendere insieme

### Modello di memoria
<center>
    <img src='/img/concurrency_1.png' style='width: 80%; border: 3px dotted red; border-radius: 15px; padding: 10px'>
</center>

<center>
    <img src='/img/concurrency_2.png' style='width: 80%; border: 3px dotted red; border-radius: 15px; padding: 10px'>
</center>

## Problemi aperti e risposta dell'HW
1. Atomicità
2. Ordinamento
3. Visibilità

##### Soluzioni
1. _Istruzioni ad-hoc_ (x86) $\to$ invalido le linee di cache corrispondenti alle modifiche effettuate della memoria condivisa
2. _Memory barrier_ (ARM) istruzioni hardware che garantiscono determinati comportamenti

## Thread e memoria
<center>
    <img src='/img/concurrency_3.png' style='width: 80%; border: 3px dotted red; border-radius: 15px; padding: 10px'>
</center>

```rust
//-------------CREAZIONE THREAD IN RUST-----------
use std::thread; 
//Funzione di creazione del thread
let t1 = thread::spawn(|| { funzione();}); 
let t2 = thread::spawn(|| { funzione();}); 
//Attesa/Terminazione nel thread principale
t1.join().unwrap();     //Restituisce Result 
t2.join().unwrap();     //Restituisce Result 
```

**Qual è il risultato di questo codice?** Non è possibile fare delle ipotesi in assenza di sincronizzazione! $\Longrightarrow$ **Race condition**. 

```c++
#include <iostream>
#include <thread>
int a=0;

void run(){
    while(a>=0){
        int before=a;       //Lettura
        a++;                //Update/Modify  
        int after=a;        //Scrittura
    }
}    
int main(){

    //Creazione e attesa dei thread

    return 0; 
}
```

## Problemi aperti
Quando si parla di concorrenza bisogna prestare attenzione a certi aspetti, soprattutto:
* Atomicità: quali operazioni vanno eseguite in modo indivisibile
* Visibilità: quando le scritture di un thread sono visibili da un secondo thread?
* Ordinamento: quando le scritture possono avvenire in ordine diffrente?

> **Morale**: il mio programma si deve arricchire di porzioni di codice in cui devo comunicare con gli altri thread. Ogni processore ha comunque il suo modo di far fronte a  questi problemi.

## Sincronizzazione 
Cosa succede se invece non faccio nulla per coordinarmi con gli altri thread in esecuzione? Espongo il mio programma ad una serie di eventi casuali.
Alcune delle possibili soluzioni ai problemi dell'accesso condiviso sono particolari strutture dati che godono di certe proprietà:
* **Atomic**: tutti i processori implementano come base le operazioni di Read/Modify/Write in modo **atomico**, cioè non interrompibili e non osservabili nei loro stati intermedi. Tuttavia queste sono limitate ai tipi semplici (booleani, interi, puntatori). Tali istruzioni utilizzano delle barriere di memoria.

* **Mutex** Per estendere le garanzie di atomicità a dei tipi più complessi bisogna introdurre il concetto di **Mutex**. Un mutex può essere o libero o posseduto da un singolo thread, se cerca di utilizzarlo, rimane in attesa senza sprecare cicli di CPU.

* **Condition variable** in alcuni casi occorre attendere e senza consumare inutilmente cicli di CPU che una certa condizione (_più complessa della semplice attesa su un mutex_) si verifichi. Questo a patto che un certo thread  segnali il verificarsi di tale eventi, tramite l'utilizzo di opportuni metodi. Una condition variable può essere utilizzata solo **in coppia** con un Mutex.

## Thread in Rust
In Rust, un *thread nativo* viene creato con la funzione `std::thread::spawn(...)` a cui viene passata una funzione lambda associata alla computazione che il thread dovrà eseguire. La funzione ritorna una struct di tipo `std::thread::JoinHandle<T>`, dove `T` rappresenta il tipo restituito dalla computazione del thread, ovvero dalla funzione lambda passata a spawn.

La terminazione di un thread e il recupero del valore prodotto è possibile tramite l'invocazione sulla struct `JoinHandle` del metodo `join()` che ritorna:
*  `std::Result::Ok(T)` con il valore finale
* `std::Result::Error` nel caso in cui il metodo sia invocato durante l'esecuzione della computazione.

La cosa importante da notare è che non c'è nessuna relazione di parentela tra il thread creatore e il thread creato, tuttavia spesso nel _main thread_ si attende la terminazione dei thread creati.

Al fine di garantire **accessi in modo corretto in memoria** e **assenza di comportamenti non definiti**  in Rust esistono due _tratti marker_ Send e Synch.
* Send è applicato automaticamente a tutti i tipi per cui è possibile che l'accesso al loro contenuto non avviene **contemporaneamente**. 
* Synch si applica a tutti i tipi T tale che &T risulta avere il tratto Send. Questi possono essere condivisi tra più thread senza creare dei comportamenti indefiniti.

I puntatori e i riferimenti non hanno il tratto send, quindi non possono essere catturati dalla chiusura passata a `spawn()` perché il thread e quindi la computazione associata possono avere una durata indefinita rispetto alle variabili cui si fa riferimento.

Si noti infine il tipo `Rc` (Reference Counter) NON gode del tratto Send in quanto il suo utilizzo prevede che venga fatto l'incremento o il decremento di un contatore, e questo **non viene fatto in modo atomico**.

## Modelli di concorrenza

### Primo modello: Condivisione dello stato
E' un modello basato sulla condivisione di una **struttura dati condivisa**, a cui tutti i thread interessati possono accedere in lettura e in scrittura. E' il meccanismo di base su cui si basano i tipi `std::sync::Mutex` e `std::sync::Condvar`.

### Secondo modello: Condivisione di messaggi
E' un modello basato sullo scambio di messaggi tra più mittenti e un solo destinatario. Questo modello è implementato dal tipo `std::sync::mpsc::channel`.

## Condivisione dello stato

### `std::sync::Mutex<T>` (uno alla volta)
Un oggetto di tipo mutex incapsula un dato di tipo T oltre  al riferimento ad un mutex nativo del sistema operativo. 
Può avere un singolo possessore, questa è la ragion e principale per cui lo si incapsula all'interno di un `Arc`, questo è simile a `Rc` con l'unica differenza che le operazioni di incremento e decremento dei contatori `strong` e `weak` vengono fatti in modo **atomico**. 

L'unico modo per accedere al dato contenuto in un Mutex è utilizzando il metodo `lock()`, questo metodo restituisce a sua volta un oggetto di tipo `LockResult<MutexGuard<T>>` e resta bloccato  finché non è stato possibile acquisire il mutex nativo a cui punta il campo `inner`.
L'operazione di lock() ritorna un result perché nel caso in cui questo viene acquisito da un thread che panica, viene avvelenato (il lock, campo poison) ritornando un errore.

Quando il `lock()` ha successo e quindi riesco ad avere accesso esclusivo al mutex nativo, mi viene ritornato uno smart pointer di tipo `MutexGuard<T>`, dereferenziandolo si ottiene un *riferimento mutabile* al dato T incapsulato dal Mutex.
Quando il `MutexGuard` esce dallo scope, il mutex nativo viene rilasciato permettendo ad  altri thread di chiederne il possesso.

### `std::sync::RwLock<T>`
Il fatto di tenere bloccato il dato incapsulato nel mutex va bene quando questo deve essere modificato, perché altrimenti si potrebbero verificare dei comportamenti indesiderati (eg. non determinismo). Si potrebbe però pensare di permettere a più thread contemporaneamente in lettura del dato di tipo T incapsulato nel lock.
`RwLock` implementa quindi la politica **multiple reader-single writer**.

### Attese condizionate
Spesso in diversi casi si vogliono fare delle operazioni su dei dati in modo esclusivo ma in modo subordinato al verificarsi di una certa condizione.
Questo richiede come minimo la presenza di un Mutex che magari incapsula un valore booleano. Su questo Mutex poi viene fatto in polling il controllo di *'condizione verificata'* oppure si manda in sleep il thread per un certo periodo di tempo, ma entrambe queste soluzioni sono inefficienti.

Per gestire in modo efficiente queste operazione, i moderni sistemi operativi offrono il concetto di **condition variable** che non sono altro che strutture dati che permettono di *bloccare un thread* finché non succeda qualcosa. su una condition variable si possono effettuare principalmente le operazioni di:
* `wait()`: mediante questa operazione vengo spostato dalla coda dei runnable alla coda dei *not-runnable*, inoltre mi viene messa un'etichetta relativa al fatto che sono in attesa di notifica su una certa condition variable.

* `notify()`: è l'operazione che mi permette di spostare il thread dalla coda dei not-runnable alla coda dei *runnable*. La `notify()` in generale può essere:
    * `notify_one()` in cui viene rimosso dalla coda dei not-runnable un thread a caso in attesa su quella condition variable. 
    * `notify_all()` vengono rimossi tutti i thread etichettati con quella condition variable.

In Rust, le condition variable sono implementate nella libreria standard dal tipo `std::sync::Condvar`, essa è dotata dei metodi:
* `notify_one()`, `notify_all()`
* `wait()` che fa continuare i processi uno alla volta dopo l'acquisizione del lock, questo è il motivo per cui le condition variable sono utilizzate sempre in coppia con un Mutex.

#### Notifiche spurie
Si è osservato sperimentalmente che anche se un thread viene spostato nella coda dei not-runnable, può essere ugualmente tolto dal sistema operativo da questa coda. Questo comportamento risulta in una notifica spuria. Occorre quindi in qualche modo verificare che la condizione attesa sia verificata.

#### Notifiche perse
Altre volte è probabile che si abilita la prosecuzione di un altro thread prima che questo chiami il metodo `wait()`. In questo caso la notifica viene persa perché la notify agisce SOLO se trova un thread nella coda; a questo punto il thread che eseguirà la wait aspetterà una notifica di un qualcosa che già si è verificato.

L'utilizzo del metodo `wait_while()` evita sia il caso di notifica spuria che quello di notifica persa, accettando una chiusura. Quando il thread viene notificato, si controlla il predicato implementato nella chiusura, se questo risulta essere vero allora il thread resta nella coda dei not-runnable altrimenti viene svegliato. Questo risparmia il programmatore dal dover fare questo lavoro a mano.

Esistono inoltre altri metodi che permettono di limitare l'attesa, questi sono: wait_timeout() che accetta un parametro di tipo `Duration` e wait_timeout_while() che previene anche le situazioni di notifiche perse e notifiche spurie.

## Condivisione di messaggi (Multiple producer, single reader)
E' il modello implementato da channel(), questo produce una tupla costituita da due struct:
* Una di tipo `Sender<T>` che può essere clonata. Su questa viene chiamata il metodo `send()` che va a buon fine solo nel caso in cui ci sia almeno un receiver. Operazione non bloccante.

* Una di tipo `Receiver<T>` che non può essere clonata su cui può essere chiamato il metodo `recv()` che va a buon fine solo nel caso in cui ci sia almeno un sender.


## Altri modelli di concorrenza

### La libreria Crossbeam
##### 1) Modello Fan-in/Fan-out
##### 2) Modello Pipeline
##### 3) Modello Produttore-Consumatore

### Il modello degli attori


















