# Using die-orchid-client

die-orchid-client is a client for [orchid](https://github.com/0xDEADBEE7/orchid), a headless command-line tool for LLM conversations, as an extension for [DIE](https://github.com/0xDEADBEE7/die): orchid's session browser and a chat per session. It's a port of emacs-orchid-client, an Emacs client for orchid that isn't published, and it's laid out and keyed as that was. It depends on die-core alone, so it's platform-neutral, and it's not part of DIE: a repository of its own that your config installs if you want it ([install](install.md)).

The keys are in [keys.md](keys.md).

## How it runs orchid

`\o`, or `M-x orchid`, opens it once your config binds it. It drives orchid 0.2.2 as the Emacs client does: `orchid` on the PATH your login shell sets (so `~/.local/bin/orchid` from DIE.app too), on orchid's own root, `$ORCHID_CONFIG` or else `~/.config/orchid`, passed to every command as `--config`. Its commands never inherit `ORCHID_SESSION_ID`, `ORCHID_CONFIG` or `ORCHID_HOOK_DEPTH`, so a DIE started from inside an orchid tool doesn't make its sessions children of that one. Emacs and DIE can be open on the same sessions at once: control goes through orchid, and its files are read without a lock, as orchid's own readers read them.

## The session browser

`\o` or `M-x orchid` shows `*orchid-sessions*` in this window, never a split, laid out as seek (a list browser for Emacs, also unpublished) lays it out: the title centred in `─`, the search line, the columns `LABEL`, `STATUS` and `UPDATED`, and a row per session, the one changed last first. It's 78 columns wide, or the window's width if that's less, and a row wider than the window wraps at a word, as Emacs with `global-visual-line-mode` wraps it.

- **LABEL** is the session's label, or else its agent and workspace (`default-die`), or else its id.
- **STATUS** is `ACTIVE` while a chat is open on it and `IDLE` otherwise, with `•RUN` while it runs, wherever the turn was started, and `DEL` or `KILL` when it's marked.
- **UPDATED** says how long ago it last changed.

The selected row is highlighted, and no cursor is drawn. A window shows as many rows as fit under the six lines above them, 20 at most, and `No results` when there are none.

It reads each session's `metadata.json` rather than running `orchid list`, so it opens at once, with times and order the Emacs client never had. A status that changes redraws it, and while a window shows it, it reads the sessions again every 2 s, so a turn you started from Emacs shows too. When the list changes under it, the selection stays on its session. It's read-only, and `:rebuild` brings it back with its query, selection and marks.

## A chat

A chat, `*orchid-chat-<agent>-<wsdir>-<id5>*`, opens in this window from the browser's `RET` or `M-x orchid-open-session`, laid out as the Emacs client's:

- `Workspace:` and `Agent:`, and a separator labelled with the workspace;
- the session's last 250 events, oldest first, each a collapsed 80-column stub with its tokens and local time (`[user]`, `[agnt]`, `[bash]`, `[read]`, `[edit]`, `[rslt]`, and `[fail]` and `[term]`, which Emacs showed as raw JSON), with a separator before each of your messages;
- `[session restored]` with the session's cumulative input;
- your input under the last separator.

It starts in normal state at the end, in the directory you opened it from (the browser's, for its `RET`), as Emacs's did, so `find-file` and `orchid-new-session` start there; `/new` goes by the session's. From there it follows the session, every 0.5 s while it runs and every 2 s otherwise, putting each new event in above the line that says how the turn is going.

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

## Other commands

- `M-x orchid-check-cli` says whether orchid is on the PATH and its version, and warns below 0.2, which this client is written for.
- `M-x orchid-list-sessions` lists each session as `  label (id)`, oldest first, in `*orchid-list-sessions*`, below, with the focus staying, as Emacs's echo area showed them.
- `M-x orchid-open-session` asks `Session ID or label: ` (or takes it after the name, `:orchid-open-session ID`), finds it by its label too, which Emacs's didn't, and says `Opened session: …`, or `Failed to open session: Session '…' not found`, as Emacs's did.
- `M-x orchid-session-browser-show` is `orchid`'s other name, as in Emacs, and `M-x orchid-collapsible-toggle-at-point` is `TAB` on a section, doing nothing elsewhere.
- `M-x orchid-new-session` asks `Agent: ` among the agents `orchid agent` says are valid, and starts in the buffer's directory. A prompt you open while orchid answers stays yours, and `Agent: ` comes when it closes, as Emacs's never took one from you.
- `M-x orchid-log-show` asks `Session ID: ` (or takes it after the name) and shows ` *orchid-log:<id>*`, beside, as `display-buffer` does: the session's `events.jsonl` as its chat has read it, the last 5 MB at first, then each line as the chat reads it, as Emacs's monitor kept it. It goes with the chat. With no chat following the session, it's Emacs's `Not monitoring session <id>`.
- `*Orchid Log*`, the client's own log, keeps its last 2,000 lines and isn't kept over a restart. You go to it as to any buffer, as in Emacs.

## Timers and processes

- An orchid command that hasn't answered in 5 minutes is stopped, with whatever it started that's still in its process group; orchid's own turn worker is in a group of its own, so it goes on.
- Nothing of orchid's runs or is read until you ask. While no orchid buffer is open, the client sets no timer. While one is, a single timer looks at the sessions for all of them: every 0.5 s while a turn runs or a send is out, every 2 s otherwise, to see a turn started elsewhere. A turn's indicator ticks each second only while the turn runs. That 2 s look is all an idle DIE does for orchid: it doesn't keep DIE's own looks at your files and its session awake, which rest at one every 5 s after 30 s of nothing from you.

## What isn't there yet

- In the session browser, insert and visual state have die-vim's keys, where the Emacs client's browser keymap worked in insert state too.
- In a chat, Emacs's hover hints ("Press TAB to toggle"), and `M-p` and `M-n`, which did nothing there.
- DIE's session file keeps a chat's whole text, as it keeps any buffer's that has no file, though a restart rebuilds the chat from `events.jsonl`: about 250 KB for 250 events, written about every 5 s while a turn runs.
