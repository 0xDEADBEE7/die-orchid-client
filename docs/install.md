# Installing it in your config

Your config is a binary crate that depends on DIE and on the extensions you choose ([die-config](https://github.com/0xDEADBEE7/die-config) is one). Installing this one is a dependency line, its entry in your list of extensions, and its key. In your config's `Cargo.toml`:

```toml
[dependencies]
die-orchid-client = { git = "https://github.com/0xDEADBEE7/die-orchid-client", branch = "main" }
```

DIE's crates come from git at `branch = "main"` here as in your config, so one build has one die-core and the `Extension` trait lines up. Then, in your list of extensions and your bindings:

```rust
Box::new(die_orchid_client::Orchid::default()),
```

```rust
// The session browser, in normal state, as die-config binds it.
keys.bind(State::Normal, "<leader>o", "orchid");
```

`Orchid::default()` takes the client's defaults; `Orchid::new(Options { .. })` sets its options. Removing it is deleting those lines.

orchid itself must be on the PATH your login shell sets; `M-x orchid-check-cli` says whether DIE finds it. See [usage.md](usage.md#how-it-runs-orchid).
