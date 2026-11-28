# Hand run

Apply hail/1 sections 3 and 11 and Appendix A to the supplied frame and settings.
The required headers, source count, source label and body citation pass.
The source value contains an actual U+0009 tab after `work:a`, then `b`, a literal
backslash followed by `u0009`, a double quote, `é`, and `@aaaaaaa`.
The path is malformed, so the incomplete-reference row does not apply. Emit one
h-source-form fault at source 1, using the general all-forms repair. Skip dependent
lookup and hash checks; their absent inputs produce no not-checked report.

For the substitution, the tab becomes the six bytes `\u0009`; the literal backslash
becomes two backslashes and its following `u0009` stays literal. The double quote
gets one preceding backslash. The UTF-8 bytes of `é` remain unchanged. Add no enclosing
quotes and do not escape the generated escapes or the fixed template again.

Complete fault stream, in order:

```text
source 1: work:a\u0009b\\u0009\"é@aaaaaaa is not a well-formed source value; write name@commit or name:path@commit or name:path@commit:first-last or name:path@commit:first-last label or name@from..to or name:path@from..to; use a listed name, a relative path of nonempty /-separated ASCII letter, digit, ., _ or - segments (not . or ..), and unique 7 to 64 lower-case hex commit ids; range numbers are positive decimal without leading zeros; label is one word without whitespace or controls; only name:path@commit may append sha256 H with H equal to the whole file's 64 lower-case hex hash.
```

The printed line has one terminating LF and no embedded tab or LF.
Verdict: `refuse`; fault count: 1; selected informational reports: none.
