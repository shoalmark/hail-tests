# Hand run

Apply hail/1 sections 3 and 11 and Appendix A to the supplied frame and lookup facts.
The required headers, source count, labels and body citations pass unless a fault below says otherwise.
Positive endpoint syntax passes, but 3 exceeds 2. Emit only the order fault; end and label checks skip.

Complete fault stream, in order:

```text
source 1: range 3-2 starts after it ends; write name:path@commit:first-last with 1 <= first <= last.
```

Verdict: `refuse`; fault count: 1.
