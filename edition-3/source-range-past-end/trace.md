# Hand run

Apply hail/1 sections 3 and 11 and Appendix A to the supplied frame and lookup facts.
The required headers, source count, labels and body citations pass unless a fault below says otherwise.
There are three lines, not four; the terminal LF creates no fourth empty line. Range ordering passes, then the end check refuses.

Complete fault stream, in order:

```text
source 1: range ends at 4 but result.txt has 3 lines; write name:path@commit:first-last within those lines, or name:path@commit for the whole file.
```

Verdict: `refuse`; fault count: 1.
