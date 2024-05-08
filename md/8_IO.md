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