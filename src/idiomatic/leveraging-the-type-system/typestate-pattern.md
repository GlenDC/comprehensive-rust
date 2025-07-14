---
minutes: 5
---

# The Typestate Pattern

The typestate pattern uses the type system to make **invalid states unrepresentable**.

```rust
use std::marker::PhantomData;
# pub struct Provider;
# pub struct Verifier;
# pub struct ClientConfig;

pub struct WantsVersions;
pub struct WantsVerifier { versions: Vec<&'static str> }
pub struct Ready { versions: Vec<&'static str>, verifier: Verifier }

pub struct ConfigBuilder<State> {
    provider: Provider,
    state: State,
    _marker: PhantomData<State>,
}

impl ConfigBuilder<WantsVersions> {
    fn with_default_versions(self) -> ConfigBuilder<WantsVerifier> {
        self.with_versions(vec!["1.2", "1.3"])
    }

    fn with_versions(self, versions: Vec<&'static str>) -> ConfigBuilder<WantsVerifier> {
        ConfigBuilder {
            provider: self.provider,
            state: WantsVerifier { versions },
            _marker: PhantomData,
        }
    }
}

impl ConfigBuilder<WantsVerifier> {
    pub fn with_verifier(self, verifier: Verifier) -> ConfigBuilder<Ready> {
        ConfigBuilder {
            provider: self.provider,
            state: Ready {
                versions: self.state.versions,
                verifier,
            },
            _marker: PhantomData,
        }
    }
}

impl ConfigBuilder<Ready> {
    pub fn build(self) -> ClientConfig {
        // Use self.provider, self.state.versions, self.state.verifier...
        ClientConfig
    }
}
```

<details>

- This example is inspired by
  [Rustls's client config builder](https://docs.rs/rustls/latest/rustls/struct.ConfigBuilder.html).

- Rust's type system acts as the foundation for the builder's state machine.
  This ensures that the user is guided through each step of the build process,
  and that it is impossible to construct an invalid `ClientConfig`.

- The typestate pattern does not require generics. It can also be implemented
  by defining distinct types for each state,
  rather than relying on generic parameters and monomorphization.

- The typestate pattern shifts error handling for invalid configurations to compile time.
  This approach is clearer and safer than returning runtime `Result` types,
  as it prevents an entire class of mistakes and simplifies correct usage.


- The wrong code simply won’t compile.

  This is the essence of typestate:
  **using types to encode valid states and transitions**,
  and making invalid transitions impossible.

- Besides the builder pattern there are plenty of other use cases:
  - File I/O or network APIs where setup (e.g., `.connect()` or `.open()`) must happen before use.
  - Locking APIs that transition from unlocked to locked states.
  - Embedded systems where safe hardware access requires correct init sequences.
  - Authentication and privilege tokens.

- [The Embedded Rust Book also dedicated a chapter on this pattern.](https://docs.rust-embedded.org/book/static-guarantees/typestate-programming.html).

</details>
