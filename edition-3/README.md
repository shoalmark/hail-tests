# Third-edition cases

These are the current draft's cases. The top-level corpus remains at its earlier text pin for compatibility;
its single-fault outcomes are not current-edition filing outcomes. No implementation is included here.

Each folder has `case.toml` and `expect.toml`. An `input.hail`, when present, is read byte for byte.
`checks` explicitly selects the part of the text exercised: a form case proves no authority or filing.
Other cases state their input event and context. A `context.toml` holds supplied facts, never a network lookup.
The reader must use the specification's procedure, not an existing implementation's fault precedence.

A malformed section stops only checks whose inputs cannot be recovered. Independent faults are all printed.
Form faults use the header name, body line (one-based), ask, or source number. Context and event cases use
the corresponding settings or record location. The case's `rule` maps it to the edition round.

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

Case files contain only `rule`, `checks` and `about`. Projected context supplies facts, including
`source-number` for carried hails and decider publication. Signature booleans stand for supplied
verification results, not executed cryptography. `verification-performed = false` selects an
unchecked display; an omitted verification result must never imply success. In roster-flag cases,
`roster-signing-required` projects `teams.<team>.signing-required`; omission selects §9's true default.
Expected step, verification status, effect, selection, actions, owed state and clocks live only in
`expect.toml`, not beside those inputs.
