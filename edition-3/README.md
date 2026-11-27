# Third-edition cases

These are the current draft's cases. The top-level corpus remains at its earlier text pin for compatibility;
its single-fault outcomes are not current-edition filing outcomes. No implementation is included here.

Each folder has `case.toml` and `expect.toml`. An `input.hail`, when present, is read byte for byte.
`checks` explicitly selects the part of the text exercised: a form case proves no authority or filing.
Other cases state their input event and context. A `context.toml` holds supplied facts, never a network lookup.
The reader must use the specification's procedure, not an existing implementation's fault precedence.
`faults` is the complete ordered list of expected lines, with one LF after each at output. Empty means no
fault in the selected checks. `reports` holds informational lines and is separate from refusal.

A malformed section stops only checks whose inputs cannot be recovered. Independent faults are all printed.
Form faults use the header name, body line (one-based), ask, or source number. Context and event cases use
the corresponding settings or record location. The case's `rule` maps it to the edition round.
