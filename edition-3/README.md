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
order. For a report with `match = "exact"`, also compare its `location` exactly and its `line`
byte for byte as UTF-8 followed by one LF, as for exact faults. Its location field must match the
location printed in the line. For reports without that mode, compare only their codes, multiplicity
and order, never their example words. Reports are separate from `faults` and do not increase fault-count.
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

## Earlier-edition read

`first-edition-pointer-read` selects `earlier-edition-read`. Its context projects the edition and
text settings at the original filing; `current-edition` describes the reader's current record.
These are supplied settings facts, not a serialized roster or proof of a historical filing.

1. Select `filing-settings.edition` and its text pin before reading the frame. Do not select the
   current edition or try parsers until one accepts.
2. Load `text-path` at `text-commit` from `text-repository` (`hail` names the companion text
   repository), and check its bytes against `text-sha256`. That pinned first-edition text is the
   source for the frame, header and body-form check in its sections 1–3.
3. Read `input.hail` under that text. Retain the input bytes, including its pointer line and final
   LF. Report the selected edition and text commit, and the retained bytes' sha256 as
   `preserved-sha256`; compare those fields with `expect.toml` under the contract above.

The selected check covers edition selection, form and byte preservation. Authority, signing,
receipt obligations and time freshness remain outside it. The paired `sources-legacy-header`
case supplies identical bytes to the current third-edition form check as a new filing and expects
refusal. Reading the old record grants no right to file those bytes under the new edition.

## Source object inputs

Source cases keep the suite's supplied-fact convention: `context.toml` projects git objects
and `bytes/` carries the raw blobs. No network or programming-language parser is involved.
These are object projections, not a serialized repository or an assertion that repeated example
hex ids hash to these bytes. The reader may consume supplied lookup results, as hail/1 §11 allows.

For the extended cases, `lookup.object-inventory-complete` distinguishes a complete supplied
object-name inventory from an unavailable lookup; `lookup.parent-graph-complete` states whether
all parent edges needed to decide ancestry are supplied. Each `[[commit]]` names its repository,
full `id`, ordered `parents` and `tree-complete`. A complete tree's only leaf entries are the
matching `[[file]]` rows; absent rows prove absence only for a complete tree. Intermediate
directories are ordinary trees. `kind = "file"` is a regular blob at mode 100644; `tree`,
`symlink` and `submodule` are modes 040000, 120000 and 160000 respectively. `bytes` names the raw
blob file relative to the case. `[[object]]` instead supplies a non-commit object's name, id
and type. These fields are input facts, not expected verdicts. Empty parent arrays are roots.

Older file cases without the lookup/commit extension keep their original convention: a file
row supplies its commit and tree lookup; `settings.files = []` supplies a complete lookup
showing that the requested path is absent. This backward-compatible shorthand does not infer
success from an explicitly unavailable lookup in an extended case.
