# Hand run

Apply hail/1 sections 3 and 11 and Appendix A to the supplied frame and lookup facts.
The required headers, source count, labels and body citations pass unless a fault below says otherwise.
The suffix is well formed but forbidden on a range. Its target and range independently pass; no hash comparison is run.

Complete fault stream, in order:

```text
source 1: sha256 is allowed only on name:path@commit; remove the sha256 suffix, or write name:path@commit sha256 H for the whole file with its 64 lower-case hex hash.
```

Verdict: `refuse`; fault count: 1.
