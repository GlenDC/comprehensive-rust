```rust
use std::fs::File;
use std::io::{self, Write};

fn write_log() -> io::Result<()> {
    println!("Entering log scope");

    {
        println!("Opening log.txt");
        let mut file = File::create("log.txt")?; // ownership starts here

        writeln!(file, "Logging a message...")?;
        println!("Finished writing to file");
    } // file is dropped here, log.txt is flushed and closed

    println!("Exited log scope");
    Ok(())
}
```
