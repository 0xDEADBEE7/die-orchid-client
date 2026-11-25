# die-orchid-client

Chat with your [orchid](https://github.com/0xDEADBEE7/orchid) LLM sessions from inside [DIE](https://github.com/0xDEADBEE7/die).

An extension for DIE: a session browser and a chat buffer per session, ported from an Emacs client and keyed the same way.

```text
Workspace: ~/projects/notes
Agent: default
───────────────────────────────────  notes  ───────────────────────────────────

[user]    [Summarise the open TODOs in this folder]    [ 12k] [10:02:11]
[read]    [todo.md]                                    [ 13k] [10:02:13]
[agnt]    [There are four open items: ...]             [ 15k] [10:02:19]

[session restored]                                                     [  15k]
───────────────────────────────────  notes  ───────────────────────────────────

Now group them by priority█
```

## What it does

- **Browse sessions**: a searchable list with status, last change, and live `RUN` marks.
- **Chat**: send messages, watch each turn's events arrive, expand any of them with `TAB`.
- **Stays out of your way**: new events never move your cursor or your half-typed input.
- **Survives restarts**: chats come back after `:rebuild`, expanded sections and input intact.
- **Manage sessions**: start, stop and delete them, all through orchid itself.
- **Light**: nothing runs until you open it; one timer for all its buffers.

## Install

In your DIE config's `Cargo.toml`:

```toml
die-orchid-client = { git = "https://github.com/0xDEADBEE7/die-orchid-client", branch = "main" }
```

Then add the extension and a key:

```rust
Box::new(die_orchid_client::Orchid::default()),
keys.bind(State::Normal, "<leader>o", "orchid");
```

You need `orchid` on your PATH.

## Docs

- [Usage](docs/usage.md): the browser, chats and every command
- [Keys](docs/keys.md): what each key does
- [Install](docs/install.md): adding it to your config
- [Development](docs/development.md): building and testing beside DIE

## License

MIT or Apache-2.0, at your option: [LICENSE-MIT](LICENSE-MIT), [LICENSE-APACHE](LICENSE-APACHE).
