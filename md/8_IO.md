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

## `Path` e `PathBuf`

Path $\Longrightarrow$ si lega al concetto di `str`
PathBuf $\Longrightarrow$ si lega al concetto di `String`

```rust
std::fs::read_dir(dir: &Path)->Result<()>
std::fs::create_dir(dir: &Path)->Result<()>
std::fs::remove_dir(dir: &Path)->Result<()>
```

## Operazioni con i file

```rust
struct File; 

open(path:P)->Result<File> where P: AsRef<Path>
//N.B. non c'è close() --> tratto Drop
create(path: P)->Result<File> where P: AsRef<Path>

read_to_string(&mut str)

//---------------------------

let nome_file="/mioPath"; 
let mut mio_file = File::open(nome_file); 
let mut cont_file = String::new(); 
mio_file.read_to_string(&mut cont_file); 
            ...         ....
```

### `std::fs::OpenOption`
Contiene la descrizione delle opzioni con cui posso aprire un file
* lettura/scrittura
* append
* ...

```rust
    let mut mio_file = OpenOption::new()
                        .<Opzione1>(<par>)
                        .<Opzione2>(<par>)
                        . ...
                        .open("/mioPath"); 
```

## Leggere e scrivere un file 

1. File di moderate dimensioni
    * `read_to_string()`
    * `write()`

## Tratti relativi ad Input/Output

### `std::io::Read`

```rust
trait Read{
    fn read(buf: &[u8])->Result<usize>; 
    fn read_to_end(buf: &mut Vec<u8>)->Result<usize>; 
    fn bytes()->Bytes<Self>; //ritorna iteratore ai byte del file

    let mut lim_reader = file.take(10); 

    lim_reader.read_to_end()...
}
```

### `std::io::BufRead`: ottimizzazione della lettura

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

## `std::io::Seek`
```rust
trait Seek{
    fn seek()
     
}
```

## Serializzazione e Deserializzazione: `serde`

Da una struttura dati --> File (Serializzazione)
Da file --> struttura dati (Deserializzazione)

## Appendice: `write!()`
...simile alla `sprintf()` del C




