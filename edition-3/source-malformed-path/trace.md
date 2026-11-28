# Hand run

Apply hail/1 sections 3 and 11 and Appendix A to the supplied frame and lookup facts.
The required headers, source count, labels and body citations pass unless a fault below says otherwise.
This value does not match a permitted source form. Section 11 uses the general all-forms repair, without nearest-form guessing. Lookup and hash comparison skip; independent citations pass.

Complete fault stream, in order:

```text
source 1: work:…/result.txt@aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa is not a well-formed source value; write name@commit or name:path@commit or name:path@commit:first-last or name:path@commit:first-last label or name@from..to or name:path@from..to; use a listed name, a relative path of nonempty /-separated ASCII letter, digit, ., _ or - segments (not . or ..), and unique 7 to 64 lower-case hex commit ids; range numbers are positive decimal without leading zeros; label is one word without whitespace or controls; only name:path@commit may append sha256 H with H equal to the whole file's 64 lower-case hex hash.
```

Verdict: `refuse`; fault count: 1.
