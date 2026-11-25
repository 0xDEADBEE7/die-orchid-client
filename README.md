# die-orchid-client

A client for [orchid](https://github.com/0xDEADBEE7/orchid), a headless command-line tool for LLM conversations, as an extension for [DIE](https://github.com/0xDEADBEE7/die): orchid's session browser and a chat per session. It's a port of emacs-orchid-client, the owner's Emacs client for orchid, which isn't published, and it's laid out and keyed as that was. It depends on die-core alone, so it's platform-neutral, and it's not part of DIE: a repository of its own that your config installs if you want it.

## Use

`\o`, or `M-x orchid`, is emacs-orchid-client, ported as die-orchid-client: an extension outside DIE, in this repository, which a config registers and binds (see [installing it](#installing-it-in-your-config)). It drives orchid 0.2.2 as the Emacs client does: `orchid` on the PATH your login shell sets (so `~/.local/bin/orchid` from DIE.app too), on orchid's own root, `$ORCHID_CONFIG` or else `~/.config/orchid`, passed to every command as `--config`. Its commands never inherit `ORCHID_SESSION_ID`, `ORCHID_CONFIG` or `ORCHID_HOOK_DEPTH`, so a DIE started from inside an orchid tool doesn't make its sessions children of that one. Emacs and DIE can be open on the same sessions at once: control goes through orchid, and its files are read without a lock, as orchid's own readers read them.

**The session browser**, `\o` or `M-x orchid`, shows `*orchid-sessions*` in this window, never a split, laid out as seek, the owner's list browser for Emacs (also unpublished), laid it out: the title centred in `─`, the search line, the columns `LABEL`, `STATUS` and `UPDATED`, and a row per session, the one changed last first. It's 78 columns wide, or the window's width if that's less, and a row wider than the window wraps at a word, as the owner's Emacs, with `global-visual-line-mode`, wrapped it there. LABEL is the session's label, or else its agent and workspace (`default-die`), or else its id. STATUS is `ACTIVE` while a chat is open on it and `IDLE` otherwise, with `•RUN` while it runs, wherever the turn was started, and `DEL` or `KILL` when it's marked. UPDATED says how long ago it last changed. The selected row is highlighted, and no cursor is drawn. A window shows as many rows as fit under the six lines above them, 20 at most, and `No results` when there are none.

It reads each session's `metadata.json` rather than running `orchid list`, so it opens at once, with times and order Emacs's never had. A status that changes redraws it, and while a window shows it, it reads the sessions again every 2 s, so a turn you started from Emacs shows too. When the list changes under it, the selection stays on its session. It's read-only, and `:rebuild` brings it back with its query, selection and marks. In normal state, as the owner's Evil has its keys:

- `j` and `k` move down and up, `p` up too (in Emacs, Evil's `p` pasted there), and `J` and `K` a page.
- `RET` opens the selected session's chat in this window, or goes to the one open on it, and kills the browser. `n` asks `Agent: ` among the valid agents and starts a chat on a new session in the browser's directory, the one of the buffer you opened it from; the browser stays. `q` kills it.
- `r` and `g` read the sessions again.
- `D` marks the session to delete (orchid archives it) and `S` to stop its turn, which only a running one has; each moves down. `u` takes the mark off. `x` asks `Execute N marked operation(s)? (yes or no)`, which wants `yes` or `no` typed out, in any case but with nothing around it, as `yes-or-no-p` does, so a bare `RET` asks again. Then it deletes and stops each in turn, in the order you marked them, keeps the marks that failed (`*Orchid Log*` says why), reads the sessions again and says what it did: `Deleted 1 session, Killed 1 process, 1 failed`.
- `/` and `a` search: what you type filters the rows, its characters in that order, in any case, in the label, the status (`run` finds the running ones) or the time it changed. Every printable key goes into the query, `j` and `q` too. `DEL` takes one back, `RET` and `Esc` stop with the query kept, and `C-g` clears it. `d` clears it from normal state too, and goes back to the first row.

**A chat**, `*orchid-chat-<agent>-<wsdir>-<id5>*`, opens in this window from the browser's `RET` or `M-x orchid-open-session`, laid out as the Emacs client's: `Workspace:` and `Agent:`, a separator labelled with the workspace, the session's last 250 events, oldest first, each a collapsed 80-column stub with its tokens and local time (`[user]`, `[agnt]`, `[bash]`, `[read]`, `[edit]`, `[rslt]`, and `[fail]` and `[term]`, which Emacs showed as raw JSON), a separator before each of your messages, then `[session restored]` with the session's cumulative input, and your input under the last separator. It starts in normal state at the end, in the directory you opened it from (the browser's, for its `RET`), as Emacs's did, so `find-file` and `orchid-new-session` start there; `/new` goes by the session's. From there it follows the session, every 0.5 s while it runs and every 2 s otherwise, putting each new event in above the line that says how the turn is going. Its keys:

| Key | State | What it does |
|---|---|---|
| `RET` | insert | In your input, send it, trimmed, as `orchid send --id ID -- MSG`; an empty input sends nothing. Above the input, a newline. In normal state `RET` is Evil's, as in the owner's Emacs. |
| `S-RET` | normal, insert | A newline. |
| `TAB` | normal, insert | On a stub or its detail, expand or collapse it; a reply's or a tool call's detail is made the first time, and a result's files are sections of their own inside it. In your input, a tab, which in insert state is part of what `.` puts in again. Anywhere else it says what it's for. |
| `/` | insert | Alone at the start of an input line, the slash menu, `/ `, which wants one of `config` (the find-file prompt at orchid's root), `exit  (quit)` (kill the chat), `kill` (`orchid stop`), `logs` and `metadata` (the session's files), or `new  (clear)` (a new session with this one's agent and directory). Anywhere else, a `/`. |
| `C-c C-l`, `S-Tab` | normal, insert, visual | The session browser, in this window. |
| `C-c C-q` | normal, insert, visual | Kill the chat. The session goes on. |

- **Sending** puts your message's stub and a separator above the input, and under the stub `[Processing: Ns]`, with the cumulative input at column 80, drawn again each second. The turn's events go in above it, collapsed, and it reads `[Finished: Ns]` once the session stops, however it stops (idle, failed, cancelled or terminated). A send that fails says `[SYSTEM] Send failed:` and orchid's reason, framed, above your input.
- **What you type stays put.** Events go in above the input as edits undo never sees, so neither your text nor your cursor moves, in insert state too, and `u` takes back only your typing. A visual selection stays on what it selected, both ends and its shape, and so does one on the ticking indicator. A window reading the transcript is taken to the end when a turn finishes, as Emacs took every window there, a selection there stretching to the end; one in your input stays.
- **Your input** starts at the blank line under the last separator, so `G`, `j` or a click there type into it too (die-vim's motions never go to the empty line after the last newline, where Emacs's input line was). What you type there comes back as it was after `:rebuild`.
- A session already running when you open it shows its indicator, counting from your last message, with its events going in above it. A turn started elsewhere, from Emacs say, shows its message and indicator as one of yours does.
- **Read-only**, as in Emacs: the `[session restored]` line and the separator under it, and the separator under a running session's indicator. The rest of the transcript is yours to edit.
- **Search** (`/ ? n N`) opens a collapsed section it lands in, and `j` and `k` step over hidden text.
- **A new session** (the browser's `n`, `M-x orchid-new-session`, or `/new`) opens `*orchid-chat-<agent>-<wsdir>-new*` at once. Type in it while orchid makes the session, and the session's chat takes its place in the same window with what you typed, still in insert state, labelled `<agent>-<wsdir>` as Emacs labelled it. A create that fails says why there.
- An archived or deleted session's chat says so once, stops following it, and sends nothing more: orchid would refuse it, and its lock would make the session's directory again. A directory left holding only orchid's lock is a session gone too.
- **After `:rebuild`** each chat comes back as a chat, in the windows it was in, rebuilt from `events.jsonl` with what came while DIE was down: the sections you'd expanded expanded again, each window's cursor and scroll on the same text, what you'd typed and your cursor's place in it. It follows again, a chat no window shows too, and if its turn still runs, its indicator ticks again, counting from your last message. One still being made makes its session, or takes the one orchid made while DIE restarted, so there's never a second; one whose session has gone says so and keeps your input. History you'd loaded with `orchid-chat-load-more-history` comes back as the last 250 events.
- `M-x orchid-chat-load-more-history` puts the 250 events before the oldest shown above them, in order; it's unbound, as in Emacs. `M-x orchid-collapsible-report-stats` says how many lazy details were made, and how long that took, in `*orchid-collapsible-stats*`.

What else there is:

- `M-x orchid-check-cli` says whether orchid is on the PATH and its version, and warns below 0.2, which this client is written for.
- `M-x orchid-list-sessions` lists each session as `  label (id)`, oldest first, in `*orchid-list-sessions*`, below, with the focus staying, as Emacs's echo area showed them.
- `M-x orchid-open-session` asks `Session ID or label: ` (or takes it after the name, `:orchid-open-session ID`), finds it by its label too, which Emacs's didn't, and says `Opened session: …`, or `Failed to open session: Session '…' not found`, as Emacs's did.
- `M-x orchid-session-browser-show` is `orchid`'s other name, as in Emacs, and `M-x orchid-collapsible-toggle-at-point` is `TAB` on a section, doing nothing elsewhere.
- `M-x orchid-new-session` asks `Agent: ` among the agents `orchid agent` says are valid, and starts in the buffer's directory. A prompt you open while orchid answers stays yours, and `Agent: ` comes when it closes, as Emacs's never took one from you.
- `M-x orchid-log-show` asks `Session ID: ` (or takes it after the name) and shows ` *orchid-log:<id>*`, beside, as `display-buffer` does: the session's `events.jsonl` as its chat has read it, the last 5 MB at first, then each line as the chat reads it, as Emacs's monitor kept it. It goes with the chat. With no chat following the session, it's Emacs's `Not monitoring session <id>`.
- `*Orchid Log*`, the client's own log, keeps its last 2,000 lines and isn't kept over a restart. You go to it as to any buffer, as in Emacs.
- An orchid command that hasn't answered in 5 minutes is stopped, with whatever it started that's still in its process group; orchid's own turn worker is in a group of its own, so it goes on.
- Nothing of orchid's runs or is read until you ask. While no orchid buffer is open, the client sets no timer. While one is, a single timer looks at the sessions for all of them: every 0.5 s while a turn runs or a send is out, every 2 s otherwise, to see a turn started elsewhere. A turn's indicator ticks each second only while the turn runs. That 2 s look is all an idle DIE does for orchid: it doesn't keep DIE's own looks at your files and its session awake, which rest at one every 5 s after 30 s of nothing from you.

### What isn't there yet

- In the session browser, insert and visual state have die-vim's keys, where the Emacs client's browser keymap worked in insert state too.
- In a chat, Emacs's hover hints ("Press TAB to toggle"), and `M-p` and `M-n`, which did nothing there.
- DIE's session file keeps a chat's whole text, as it keeps any buffer's that has no file, though a restart rebuilds the chat from `events.jsonl`: about 250 KB for 250 events, written about every 5 s while a turn runs.

## Installing it in your config

Your config is a binary crate that depends on DIE and on the extensions you choose ([die-config](https://github.com/0xDEADBEE7/die-config) is the owner's). Installing this one is a dependency line, its entry in your list of extensions, and its key. In your config's `Cargo.toml`:

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

## Developing it beside DIE

DIE's crates come from [github.com/0xDEADBEE7/die](https://github.com/0xDEADBEE7/die) at `main`. To work on DIE and this together, build against a checkout of DIE beside this one instead. Copy `dev-config.example.toml` to `.cargo/config.toml`, which is gitignored, and write its `/abs/path/to/...` paths out in full, since cargo doesn't expand `~`. It patches die-core, die-vim and die-test-cap to the checkout's `crates/` and `tools/`, and sets `DIE_FONT_DIR`.

```sh
mkdir -p .cargo
cp dev-config.example.toml .cargo/config.toml   # then write its paths out
cargo fmt --check
cargo clippy --all-targets
DIE_UNSEEN=1 cargo test
```

- **`DIE_FONT_DIR`** is the folder holding DIE's fonts, as a DIE checkout's `fonts/` (`cargo xtask fonts` there fills it), which DIE's renderer and die-draw read when a test draws text: this repository has no `fonts/` of its own. Its tests draw no text today, and pass with it pointing nowhere, but the dev config sets it, as every extension's does, so a test that draws finds them.
- `DIE_UNSEEN=1` keeps any DIE a test starts off your screen. These tests start none and open no window, so today it changes nothing here.
- `tests/orchid.rs` and `tests/real_chat.rs` run the real orchid, offline, and are ignored without it: `ORCHID_BIN=~/.local/bin/orchid cargo test --test orchid -- --ignored`. `tests/perf.rs` times a long chat: `cargo test --test perf -- --ignored --nocapture`.
- `rust-toolchain.toml` pins Rust 1.96.0 with rustfmt and clippy, for the checks; it builds with any Rust from `rust-version`, 1.95, up.

## License

MIT or Apache-2.0, at your option: [LICENSE-MIT](LICENSE-MIT), [LICENSE-APACHE](LICENSE-APACHE).
