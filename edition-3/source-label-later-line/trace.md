# Hand run

Apply hail/1 sections 3 and 11 and Appendix A to the supplied frame and lookup facts.
The required headers, source count, labels and body citations pass unless a fault below says otherwise.
The range is valid, but key is only on line 2, not the first selected line 1. It therefore refuses.

Complete fault stream, in order:

```text
source 1: label key is absent from first line 1 of the range; write name:path@commit:first-last label with the label copied from that first line, or name:path@commit:first-last without a label.
```

Verdict: `refuse`; fault count: 1.
