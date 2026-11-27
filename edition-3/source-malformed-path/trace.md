# Hand run

Under hail/1 section 3, the frame, required headers, source count and label pass.
The body cites #1, so citation matching remains available and there is no uncited source.
The path begins with an ellipsis scalar, outside the permitted ASCII path characters, despite having a commit.
Section 11 selects h-source-form, not h-source-commit. Substitute source ordinal 1 and
the entire supplied value for V. The form name:path@commit and sha256 H stay literal.
Lookup and hash checks skip for this malformed value; independent form and citation
checks pass. The supplied file facts do not make malformed source syntax valid.

One fault; refuse the selected checks:

```text
source 1: work:…/result.txt@aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa is not a well-formed source value; write name:path@commit using a listed name, a relative path of nonempty /-separated segments containing only ASCII letters, digits, ., _ or - (no . or .. segments), and 7 to 64 lower-case hex commit digits; optionally append sha256 H with H equal to the committed file's 64 lower-case hex hash.
```
