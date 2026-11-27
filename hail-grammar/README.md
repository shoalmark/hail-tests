# Grammar cases

Each case is data, not checker code. `case.toml` identifies a pinned starting grammar by its path in
the text repository and gives either a body projection or a grammar-file edit. A body projection
has `marker`, `re`, `body`, optional `ask` and `kind`. It is the result of hail/1 parsing; sources and
citations are hail/1 checks, outside this grammar-only run. Meaning cases additionally give the
received hail or original text needed for comparison. No fact's truth is assumed checked.

A grammar-file case states edits to a copy of the named starting grammar; that precise edited data
is the input, and all unmentioned fields stay unchanged. Edit paths name TOML tables/keys, not code.
The case’s `expect.toml` holds the expected result and complete diagnostic stream.
`trace.md` records the hand run in draft procedure order and explains every emitted fault. A fresh
reader needs only this procedure, the starting grammar and case data. No prototype is used.

Edit operations are `set` (replace or add the value at the listed key path) and `delete` (remove that
key). Apply edits in order. `remove_from_all_allow` additionally removes that opener from every
marker's allow array; it changes no require list. `grammar-raw` uses its `raw` string as the whole
file and no base data. `measure` cases run the draft's arithmetic, not its body procedure.
`hail_kind` supplies the Owner-ask kind for a parsed body. `received_sources`, `received_body_lines`
and `received_headers` give the part lookup of the hail named by re. `original` supplies the text
before shortening. A source's existence and the truth of a Fact are not inferred from these fields.

The expected stream is faults only. A full tool also reports the checks not made. For these hand
runs, all bodies lack external fact verification (which the draft never claims); preservation is
not checked when `original` is absent, and reply-part existence is not checked unless the received
projection supplies it. The traces record the applicable meaning checks, not a claim of proof of truth
or body sufficiency. A PASS means no fault in the checks whose inputs this case supplies.

## Expected-result contract

`expect.toml` format 2 is the expected result, kept out of checker inputs. Compare the verdict,
ordered fault codes and locations, and fault count exactly. Each fault's location field must
match the location printed in its one-line diagnostic. No fault can be silently merged or dropped.

For `match = "exact"`, `line` is the exact UTF-8 printed line, followed by one LF. Its problem and
repair words come from a fixed template in the governing text. Every hail-grammar fault uses this
mode. For `match = "content"`, no literal template is fixed by hail/1: `problem-content` and
`repair-content` state the required meaning and permitted replacement text or form. A reader checks
that meaning and the location/problem/repair parts; `example` is illustrative, never a byte oracle.
The content mode is available only for hail/1 faults, never hail-grammar faults.

`reports` contains selected informational notices. Compare their codes, including multiplicity and
order, never their example words. Reports are separate from `faults` and do not increase fault-count.
The selected checks bound these expectations: a full tool must additionally disclose omitted authority,
lookup and meaning checks. Those unselected not-checked notices are not compared by this suite.
Other expected result fields are compared exactly. Passing selected checks authorizes no filing.

Every grammar fault entry carries `code`, `location`, `match = "exact"` and `line`, including
shape and meaning-review faults. No synonym or punctuation normalization is allowed. An empty
fault list has `fault-count = 0` and verdict pass. Measurement totals and an explicitly asserted
schema result also live in `expect.toml`. Traces describe the hand run and refer to these exact lines;
they are evidence, not checker inputs. The starting files and schema remain unchanged from the
coordinator’s executable validation; semantic review is still required beside their shape validation.
