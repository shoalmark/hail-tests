# Hand run

Load `grammar.toml` beside `context.toml`, at the same vector commit. Its name matches
`grammar = "case-budget"`. F1–F3 pass: the closed file has every required field and marker row,
positive characters = 240, and exactly the five unchanged common meaning strings. F4–F7 pass:
REQUEST allows the defined Fact opener and requires none; all other rows allow and require
none with zero lines; all five meanings differ. F8 reads nothing because glossary = none.

The complete input has four canonical REQUEST body lines, each `Fact: The observed count is 8.`.
Sources and the ask do not count. B1 compares four with three and emits the single budget
fault at body line 4. B2–B5 pass for every line; B6 has no check, B7 requires none, B8 is below
240 on every line and B9 has no glossary. The grammar diagnostic is exactly:

```text
body line 4: [g-lines] REQUEST allows at most 3 body lines; remove or combine 1 line without dropping meaning.
```

This is a new filing. Under hail/1 section 11 the result is refuse, effect is not-filed,
and the diagnostic is the one exact fault. There are no informational reports.
