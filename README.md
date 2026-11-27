# hail/1 conformance data

This branch holds the vectors of hail/1's third edition, signed by the Owner's merge into `main`.

The current third-edition cases are in [edition-3/](edition-3/README.md); the testable grammar cases
are on `draft/hail-grammar-1` in `shoalmark/hail-tests`. Each new case names the rule it holds. Current expected
faults have separate codes and locations, with exact lines or required repair content as each suite's README defines. The changed-case
manifest selects exactly the cases for the second implementation; there is no claim that a historical
runner already understands these data files.

The 519 original top-level case folders remain byte for byte as the compatibility corpus at vector
commit `ec89972a0e06909b0da2de946b72ae4ade82dfee`, interpreted against text commit
`c22b52e104d52dc8dcc9ee362a872b844dbe9ecd`. They preserve the earlier draft's pointer form, signature
and sealing fixtures, and single-fault expectations. They are not current third-edition filing tests.
Their original README and data contract remain available at that same vector pin. Select that text
explicitly when running them; do not feed them to the new-edition parser by guessing from their bytes.
A stage-B runner must select the suite rather than recursively treating every directory as an old case.

The third edition changes neither the historical fixture bytes nor their signatures. Current form
examples and new expectations are in the locations above. No current expectation is derived
from what the old implementation happens to accept. Both repositories disable line-end conversion.
