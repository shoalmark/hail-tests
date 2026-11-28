# Hand run

Under hail/1 section 3, the frame has all required headers, sources: 1 and the consecutive
label #1. The single body citation resolves to that label. The source value is otherwise
well-formed name:path but lacks @commit, so section 11 selects h-source-commit alone.
Substitute source ordinal 1 and R = work:result.txt in both R positions of the fixed line.
The form name:path@commit and sha256 H stay literal. The optional hash is not required
by these settings. Lookup and hash checks for the malformed value skip; no missing-file
or hash fault is guessed. Independent form and citation checks pass.

One fault; refuse the selected checks:

```text
source 1: committed file reference work:result.txt is incomplete; write name:path@commit using work:result.txt as name:path and 7 to 64 lower-case hex commit digits; optionally append sha256 H with H equal to the committed file's 64 lower-case hex hash.
```
