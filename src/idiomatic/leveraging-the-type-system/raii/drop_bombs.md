# Drop Bombs: Enforcing API Correctness

Use `Drop` to enforce invariants and detect incorrect API usage.
A "drop bomb" panics if not defused.

```rust
struct Transaction {
    committed: bool,
}

impl Transaction {
    fn new() -> Self {
        println!("BEGIN TRANSACTION");
        Self { committed: false }
    }

    fn commit(mut self) {
        println!("COMMIT");
        self.committed = true;
        // Dropped after this point, no panic
    }
}

impl Drop for Transaction {
    fn drop(&mut self) {
        if !self.committed {
            panic!("Transaction dropped without commit! Rolling back.");
        }
    }
}
```

<details>

- This technique ensures a transaction is not forgotten.
  If you fail to call `.commit()`, it panics.

- This is useful when forgetting to finalize is *worse* than failing loudly.

- In most scenarios you can design this more safely without the use of a drop bomb:
  - Hide internal state (`committed`) behind encapsulation.
  - Consume `self` in `commit()` so it's impossible to call anything after.

- Don’t overuse this: panics in `Drop` are dangerous
  in foundational types or inside panicking code.

- If cleanup needs to happen regardless of success/failure,
  prefer `Drop` + side effects over a panic.

- As always, document clearly when `Drop` enforces something non-obvious.

- More advanced designs may use types like
  [`ManuallyDrop`](https://doc.rust-lang.org/std/mem/struct.ManuallyDrop.html) or
  [`Option<T>` + `.take()`](https://doc.rust-lang.org/std/option/enum.Option.html#method.take)
  to manage partial state or defusing.

- Rust does not currently support full *linear types* or *typestate programming*
  in the core language, meaning the compiler cannot always enforce that a resource
  has been used exactly once or finalized before being dropped.

  Drop bombs are a runtime fallback used to enforce these usage invariants manually.
  Typically via a `Drop` implementation that panics when an expected method
  (like `.commit()`) was not called.

  Open RFC and discussion on linear types in Rust: <https://github.com/rust-lang/rfcs/issues/814>.


</details>
