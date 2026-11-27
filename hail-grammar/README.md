# Grammar cases

Each case is data, not checker code. `case.toml` identifies a pinned starting grammar by its path in
the text repository and gives either a body projection or a grammar-file edit. A body projection
has `marker`, `re`, `body`, optional `ask` and `kind`. It is the result of hail/1 parsing; sources and
citations are hail/1 checks, outside this grammar-only run. Meaning cases additionally give the
received hail or original text needed for comparison. No fact's truth is assumed checked.

A grammar-file case states edits to a copy of the named starting grammar; that precise edited data
is the input, and all unmentioned fields stay unchanged. Edit paths name TOML tables/keys, not code.
The case's `expect.txt` is the complete expected diagnostic stream, one LF per fault; empty is PASS.
`trace.md` records the hand run in draft procedure order and explains every emitted fault. A fresh
reader needs only this procedure, the starting grammar and case data. No prototype is used.
