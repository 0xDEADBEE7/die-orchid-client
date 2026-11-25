# Developing it beside DIE

DIE's crates come from [github.com/0xDEADBEE7/die](https://github.com/0xDEADBEE7/die) at `main`. To work on DIE and this together, build against a checkout of DIE beside this one instead.

Copy `dev-config.example.toml` to `.cargo/config.toml`, which is gitignored. Its paths are relative, for a DIE checkout beside this one (`../die`), and cargo reads them from this repository's root; change them if your checkout is elsewhere. It patches die-core, die-vim and die-test-cap to the checkout's `crates/` and `tools/`, and sets `DIE_FONT_DIR`.

```sh
mkdir -p .cargo
cp dev-config.example.toml .cargo/config.toml   # for a DIE checkout beside this one
cargo fmt --check
cargo clippy --all-targets
DIE_UNSEEN=1 cargo test
```

## Notes

- **`DIE_FONT_DIR`** is the folder holding DIE's fonts, as a DIE checkout's `fonts/` (`cargo xtask fonts` there fills it), which DIE's renderer and die-draw read when a test draws text: this repository has no `fonts/` of its own. Its tests draw no text today, and pass with it pointing nowhere, but the dev config sets it, as every extension's does, so a test that draws finds them.
- **`DIE_UNSEEN=1`** keeps any DIE a test starts off your screen. These tests start none and open no window, so today it changes nothing here.
- **The real orchid.** `tests/orchid.rs` and `tests/real_chat.rs` run the real orchid, offline, and are ignored without it: `ORCHID_BIN=~/.local/bin/orchid cargo test --test orchid -- --ignored`.
- **Speed.** `tests/perf.rs` times a long chat: `cargo test --test perf -- --ignored --nocapture`.
- **Toolchain.** `rust-toolchain.toml` pins Rust 1.96.0 with rustfmt and clippy, for the checks; it builds with any Rust from `rust-version`, 1.95, up.
