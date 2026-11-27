# Hand run

Apply hail/1 sections 3 and 11 and Appendix A to the supplied frame and lookup facts.
The required headers, source count, labels and body citations pass unless a fault below says otherwise.
A NUL byte makes this file non-text for a line range. File lookup succeeds; range-end checking skips.

Complete fault stream, in order:

```text
source 1: result.txt is not UTF-8 text without NUL; write name:path@commit for the whole file, or name:path@commit:first-last for a text file.
```

Verdict: `refuse`; fault count: 1.
