---
minutes: 10
---

# RAII and `Drop` in Practice

RAII (*Resource Acquisition Is Initialization*)
means tying the lifetime of a resource to the lifetime of a value.

Rust applies RAII automatically for memory management.
The [Drop] trait lets you extend this pattern to resources outside
Rust’s ownership system, such as files, sockets, or locks.

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

<details>

- Recall from [the Memory Management chapter](../../memory-management/drop.md)
  that [`Drop::drop`](https://doc.rust-lang.org/std/ops/trait.Drop.html#tymethod.drop)
  is automatically called when a value goes out of scope.

- This is that exact mechanism in action, and is a particular implementation of the RAII concept.
  We're using it here to ensure the file is closed when the file is dropped at the end of its scope.

- RAII ensures resources are automatically cleaned up when they go out of scope.
  No `close()` required here.

- The `File` type implements
  [the `Drop` trait](https://doc.rust-lang.org/std/ops/trait.Drop.html),
  which flushes and closes the file when it's dropped.

- This is guaranteed even if `?` short-circuits or the function panics.

- It's important to know that unless the program aborts,
  the `Drop` always runs *once* and *immediately at scope exit*.

  This happens deterministically as part of unwinding the stack, even in the case of a panic.
  It will not happen in case the project is compiled with a strategy such as `panic=abort`.

- This pattern generalizes to anything that needs cleanup:
  sockets, temporary files, memory buffers, database handles.

  Whereas in languages with manual memory management such as C and C++
  you also need to use this pattern for memory allocations, in Rust it is
  only required for resources that live outside Rust's ownership system.

- Prefer this approach over manual `.close()` methods or tracking "open" state.
  It’s safer, simpler, and less error-prone.

- If you ever need to drop something *before* the end of scope, use `std::mem::drop(val)`.

- [The Rust Book also has a chapter dedicated to it in the context of smart pointers.](https://doc.rust-lang.org/book/ch15-03-drop.html).

</details>
