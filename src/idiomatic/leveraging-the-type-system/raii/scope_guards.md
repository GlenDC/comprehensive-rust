# Scope Guards with `Mutex`

In Rust, critical sections are automatically released via scope guards.

```rust
use std::sync::Mutex;

fn main() {
    let mux = Mutex::new(vec![1, 2, 3]);

    {
        let mut data = mux.lock().unwrap();
        data.push(4); // lock held here
    } // lock automatically released here
}
```

<details>

- [The `Mutex`](https://doc.rust-lang.org/std/sync/struct.Mutex.html)
  owns its data: you can’t access the value inside without first acquiring the lock.

- `mux.lock()` returns a
  [`MutexGuard`](https://doc.rust-lang.org/std/sync/struct.MutexGuard.html),
  which [dereferences](https://doc.rust-lang.org/std/ops/trait.DerefMut.html)
  to the data and implements [`Drop`](https://doc.rust-lang.org/std/ops/trait.Drop.html).

- When the guard goes out of scope,
  [`Drop::drop`](https://doc.rust-lang.org/std/ops/trait.Drop.html#tymethod.drop)
  runs and unlocks the mutex. This is a scope guard.

- Compare with C++ or Java: often you must manually unlock
  or use a separate `lock/unlock` pattern.

- In Rust, the compiler ensures the lock *cannot* be forgotten.
  There’s no way to bypass the guard unless you go into `unsafe`.

- This applies not just to `Mutex`. `RwLock`, file locks,
  and other owners of shared or external resources follow this pattern.

- You *can* accidentally hold a lock too long
  if you let the guard escape its intended scope, so be mindful of lifetimes.

- For more advanced needs, see crates like
  [`scopeguard`](https://docs.rs/scopeguard) for custom cleanup logic.

  This crate allows you to easily write code similar to `defer` in languages such as Golang.

- While Rust's ownership system prevents data races, you can still experience deadlocks.

  This happens in case two threads are each waiting for the other to release a resource.

- Consider reading [*Rust Atomics and Locks* by Mara Bos](https://marabos.nl/atomics/)
  to learn more about implementing synchronization primitives. The book demonstrates
  how `Drop` and scope guards work together in practice.

</details>
