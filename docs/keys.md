# Keys

What each orchid buffer binds. For what the buffers show, see [usage.md](usage.md).

## Session browser

In normal state, as Evil has its keys:

| Key | What it does |
|---|---|
| `j`, `k` | Down and up. `p` goes up too (in Emacs, Evil's `p` pasted there). |
| `J`, `K` | A page down and up. |
| `RET` | Open the selected session's chat in this window, or go to the one open on it, and kill the browser. |
| `n` | Ask `Agent: ` among the valid agents and start a chat on a new session in the browser's directory, the one of the buffer you opened it from. The browser stays. |
| `q` | Kill the browser. |
| `r`, `g` | Read the sessions again. |
| `D` | Mark the session to delete (orchid archives it), and move down. |
| `S` | Mark the session to stop its turn (only a running one has one), and move down. |
| `u` | Take the mark off. |
| `x` | Run the marks (below). |
| `/`, `a` | Search (below). |
| `d` | Clear the search and go back to the first row. |

**Running the marks.** `x` asks `Execute N marked operation(s)? (yes or no)`, which wants `yes` or `no` typed out, in any case but with nothing around it, as `yes-or-no-p` does, so a bare `RET` asks again. Then it deletes and stops each in turn, in the order you marked them, keeps the marks that failed (`*Orchid Log*` says why), reads the sessions again and says what it did: `Deleted 1 session, Killed 1 process, 1 failed`.

**Searching.** What you type filters the rows, its characters in that order, in any case, in the label, the status (`run` finds the running ones) or the time it changed. Every printable key goes into the query, `j` and `q` too. `DEL` takes one back, `RET` and `Esc` stop with the query kept, and `C-g` clears it.

## Chat

| Key | State | What it does |
|---|---|---|
| `RET` | insert | In your input, send it, trimmed, as `orchid send --id ID -- MSG`; an empty input sends nothing. Above the input, a newline. In normal state `RET` is Evil's, as in Emacs. |
| `S-RET` | normal, insert | A newline. |
| `TAB` | normal, insert | On a stub or its detail, expand or collapse it; a reply's or a tool call's detail is made the first time, and a result's files are sections of their own inside it. In your input, a tab, which in insert state is part of what `.` puts in again. Anywhere else it says what it's for. |
| `/` | insert | Alone at the start of an input line, the slash menu (below). Anywhere else, a `/`. |
| `C-c C-l`, `S-Tab` | normal, insert, visual | The session browser, in this window. |
| `C-c C-q` | normal, insert, visual | Kill the chat. The session goes on. |

**The slash menu**, `/ `, wants one of:

- `config`: the find-file prompt at orchid's root;
- `exit  (quit)`: kill the chat;
- `kill`: `orchid stop`;
- `logs` and `metadata`: the session's files;
- `new  (clear)`: a new session with this one's agent and directory.

`M-x orchid-chat-load-more-history` is unbound, as in Emacs.
