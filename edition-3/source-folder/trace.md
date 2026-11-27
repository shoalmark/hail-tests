# Hand run

Apply hail/1 sections 3 and 11 and Appendix A to the supplied frame and lookup facts.
The required headers, source count, labels and body citations pass unless a fault below says otherwise.
The named tree entry is a directory, so it cannot be a file source. A directory component would be permitted, but the final entry must be a regular blob.

Complete fault stream, in order:

```text
source 1: result.txt is not a regular file path at the named commit; write name:path@commit naming a regular file through tree directories only. For a file change use name:path@from..to with a regular file at every endpoint where the path exists.
```

Verdict: `refuse`; fault count: 1.
