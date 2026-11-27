> **A draft, unsigned, and it may change.** This branch, `draft/edition-3`, holds the working test vectors of hail/1's
> third edition. They are not signed and not in force, and any of them may still change. A signed edition goes on
> `main`; there is none yet.

# hail/1 conformance vectors

One folder per vector, named by its id. The runner is `crates/hail/tests/conformance.rs`: `cargo test -p hail`. The
vectors are for four of edition three's §11 rows: "frame, header, conditional keys, duplicates, marker, body, options,
ask and pointer form"; "sender, reach, return grant, signing flag, keys, edition and pin"; "future written time"; and
"pointers and stale context". The format holds the inputs of all four: the hail; where "future written time" reads
one, a reference time; and where "pointers and stale context" or the authority row is checked, that row's inputs.

The CLI's vector-count test holds this count to the suite's folders:

| Vectors | Count |
|---|---|
| the suite's vectors | 519 |

| File | Required | What |
|---|---|---|
| `input.hail` | yes | the message's bytes, exactly as received: one hail or several, canonical or not, as the vector needs |
| `reference-time` | no | the reference time of "future written time", in `written-against`'s form (§3): one line, ending in one LF. A vector without it is checked as a run given no reference |
| `context.toml` | no | the inputs of "pointers and stale context" (§3, §11), below. A vector without it is checked as a run given none of them |
| `bytes/` | with `context.toml` | a folder of the files `context.toml` names by file name, each byte for byte |
| `authority.toml` | no | the inputs of "sender, reach, return grant, signing flag, keys, edition and pin" (§8, §9, §11), below. A vector without it is checked as a run given none of them |
| `roster/` | with `authority.toml` | the committed tree at the counted state, each file at its path in the roster's repository, byte for byte |
| `received/` | no | the hails a reply answers, one file each, byte for byte; the folder given is the input given |
| `organisation/` | with `[resolving]` | for a resolving reader, `pinned/` and `in-force/`: the organisation's tree at the pin and at its state in force; and `lock/`, a bot's tree holding a lock, where `roster/` holds none, to compute the standing from |
| `expect.toml` | yes | `verdict` (`pass`, `refuse` or `not-judged`); `rule`, for `refuse`: the id of the rule that refuses it; `reports`: the ids of the rules the check reports on, each once; `about`: what the vector exercises and the section of edition three it rests on, in words, never empty |

A vector's folder holds these files and nothing else; the suite's folder holds the vectors, this README and
`PENDING`. Anything else fails the run, as does an expectation that isn't well formed, a `reference-time` that isn't
a time in §3's form, a `context.toml` that isn't well formed, a file in `bytes/` that it doesn't name, or a name it
gives that `bytes/` doesn't hold; so does an `authority.toml` that isn't well formed, a roster folder without it, or
`[resolving]` without `organisation/`.

**The context row's inputs.** `hail check` takes the bytes, tree entries, destination and scan policy as explicit
inputs, and fetches nothing (§11). `context.toml` gives them as data, every key below required unless marked optional. An input left out is reported
as not checked, never read as given and empty:

- `destination`, optional: `public` or `private`, the hail's destination as the team's extension classes it. Without
  it the private-id scan is not made, and the run reports `private-ids-not-checked`.
- `private-id-prefixes`: the configured prefixes, as written, malformed ones included; `[]` for none, which finds none.
  A string outside `[A-Za-z]+-` refuses a public-bound hail with `private-id-prefix`; a pass reports it as
  `private-id-prefix-malformed`, including when no destination is given (the key table; §11).
- `home`, optional: the sending user's own home directory, in each way it is written; `[]` for none. Without it, or
  where each entry is empty, the run reports `own-home-not-checked`. The listed forms of §11 are checked either way.
- `[secret-patterns]`, optional: the versioned secret-pattern set, its `version` and its `patterns`. Without it, or
  with no patterns in it, the bytes are not checked for secrets, and the run reports `secrets-not-checked`. Each
  pattern is a regular expression in RE2 syntax, unanchored, matched over the bytes of each line of every scanned
  location: a line ends at LF or at the end of the bytes, and one CR that ends a line, straight before its LF or as the
  last byte, is not part of the line. A match refuses with `secret`. A pattern that doesn't compile, or whose compiled form exceeds 1 MiB or nests deeper than 64, refuses every hail the
  set would scan with `secret-pattern`, and so does a set of more than 512 patterns, or one whose patterns compiled
  together exceed 8 MiB. The lazy DFA's cache keeps the regex crate's own limit, which bounds the memory it takes while
  matching and refuses no pattern. This implementation reads RE2 syntax with Unicode mode off: `\d`, `\w`, `\b`
  and the POSIX classes are ASCII, as in RE2, and a non-ASCII character written outside a bracket class matches
  its UTF-8 bytes, as in RE2. Where it differs from RE2, which reads the pattern and the text as UTF-8 (the list is of
  the forms found and does not claim to be complete):
  - **One byte where RE2 takes one character:** `.`, a negated class (`[^a]`, `[[:^alpha:]]`), `\D`, `\W` and `\S`
    each take one byte, unless `(?u)` turns Unicode mode on (`^[^a]{3}$` matches `éx`, `^\W{2}$` matches `é`), and `\B`
    matches between the bytes of one character. `\x80` to `\xff`, with two hex digits, are bytes where RE2's are the
    characters U+0080 to U+00FF (`\xff` matches the byte 0xFF, not `ÿ`), so `[\x80-\xff]` is a range of bytes;
    `\x{FF}` is the character U+00FF, as in RE2.
  - **`\s` takes VT:** `\s` is `[\t\n\v\f\r ]`, where RE2's is `[\t\n\f\r ]`, so `\s` matches the byte 0x0B and `\S`
    does not; `[[:space:]]` takes it in both.
  - **Case:** `(?i)` folds ASCII only: `(?i)é` does not match `É`, nor `(?i)k` the Kelvin sign.
  - **Refused as not compiling, where RE2 accepts:** Unicode classes (`\pL`, `\p{Greek}`), `\C`, `\Q..\E`, an octal
    escape (`\123`, `\0`), a non-ASCII character inside a bracket class, as itself (`[é]`) or as a braced hex escape
    (`[\x{FF}]`), a `{` that begins no repetition (`a{,3}`, `a{`), which RE2 reads as itself, `\E` without `\Q`
    (`a\E`), a `-` after a class escape inside a bracket class (`[\d-z]`), which RE2 reads as itself, and a flag
    repeated in one group (`(?ii)a`).
  - **Accepted, where RE2 refuses:** a count over 1000 (`a{1001}`), and nested counts whose product is over 1000
    (`(?:a{100}){11}`); a repetition of a repetition (`a**`, `a*+`, `a*??`, `a?*`, `a+*`, `a{2}{3}`); a POSIX class
    name RE2 doesn't define (`[[:alfa:]]`, read as a class of its characters); the flags `(?x)`, `(?u)` and `(?R)`;
    the escapes `\u0041`, `\u{41}` and `\U00000041`; and a `.` in a group's name (`(?P<a.b>x)`).
  - **Read differently:** inside a bracket class, `[` opens a nested class and `&&`, `--` and `~~` are set operations,
    where RE2 reads each character as itself (and may refuse the range `--` makes); `\b{start}`, `\b{end}`,
    `\b{start-half}` and `\b{end-half}` are word-boundary assertions, where RE2 reads `\b` and then the braces as
    themselves; `\<` and `\>` are the start and end of a word, where RE2 reads `<` and `>`; and a count with spaces
    inside its braces (`a{1, 2}`, `a{ 1}`, `a{1 }`) is a repetition, where RE2 reads the braces as themselves.
- `[limits]`, optional, and each of its keys optional: the scan's limits. Edition three bounds none of the scanned
  bytes, so the limits and their refusals are `hail check`'s own, as the secret-pattern set's bounds are.
  `location-bytes` bounds the bytes of one scanned location: a hail, from `hail/1` through `end hail/1` in canonical
  form with each line's LF, or one file a pointer or a hash list reaches. `line-bytes` bounds one line the scan reads,
  in the header, body or ask or in such a file, a line read as the secret scan reads it (above). `listed-files` bounds
  the files the hash lists reach, a file counted each time a list's line names it, a nested list and a file named
  twice included. `message-bytes` bounds the bytes the scan reads in one message: every hail in it and every file,
  counted each time a pointer or a list's line reaches it, so one file that several hails point at counts once for
  each. A location over its limit is refused `location-limit` before it is hashed or scanned: the hail before any file
  is reached, and each file as it is reached. A location that would take the message past its limit is refused
  `message-limit` at the same point. The walk refuses `files-limit` at the first line past its limit, before that
  line's file is hashed. Each line of a location is measured before any scan runs on that location, and a line over
  its limit is refused `line-limit`. So `hail check` never hashes, scans or matches past a limit. A limit not given is
  an omitted input: the scan is not bounded by it, and the run reports `limits-not-checked`, one report however many
  are left out, so it never gives a full pass.
- `[[repository]]`, any number: a declared repository's `origin` as sealanes/1 §11.3 defines it, from which its token
  is derived; optionally its `head`, the lookup result of the reader's head, and beside it `ancestors`, the lookup
  result of the commits that head descends from; and its `[[repository.commit]]`
  entries, each the commits available: its full `id` and its `tree`, the tree entries given. An entry is `{ path,
  kind }`, `kind` one of `file`, `tree`, `link` or `submodule`; a `file` entry may name its bytes in `bytes/` with
  `bytes`, and without it the file is proven but its bytes are not given.
- `[[workbench]]`, any number: a workbench's `name` and its `files`, each `{ path, bytes }`.

**The authority row's inputs.** `hail` reads no roster: the runner reads `roster/` with sealanes/1's reader, as the
default branch's chain started at a counted state (`ChainStart::Counted`, no commits after it), and gives `hail` the
outcome as plain data. `authority.toml` holds:

- `repository`: the URL of the repository the roster is read in; `commit`: the counted state's commit.
- `default-branch`, optional: the default branch's name. Without it the chain reaches no counted state, and every right
  is refused (§8).
- `mode`: `check`, the whole roster read in its repository, or `bot`, a bot reader reading its lock (sealanes/1 §2.3).
- `[[edition]]`, optional: the lookup of which edition a `hail@<commit>` text is, each `{ commit, edition }`. Without
  it a `hail@<commit>` is reported `edition-not-compared`; a commit the lookup doesn't name, or names twice, is too.
- `[resolving]`, optional: a resolving reader's pin into the organisation's repository: its `origin`, the `pin`, the
  organisation's `in-force` commit and its default branch's `chain`. The runner reads `organisation/in-force/` as the
  organisation's chain, calls sealanes/1's `Lock::recompute` against `organisation/pinned/` for this repository,
  then asks `Lock::standing` for the pin's standing; `hail` is given the chain and standing. Recomputing before
  reading the standing is the runner's order; no vector holds it, because the texts do not order a slice difference
  with a pin behind. The runner decides the slice before `hail` reads the message: a slice difference
  (`Reason::SliceDifference`) refuses `roster-unreadable` (§8; sealanes/1 §2.3) before every form, time and pointer
  refusal. Other `roster-unreadable` refusals fall where `hail` itself finds their causes. The runner's other
  resolution errors, including a pin off the organisation's chain, an origin or pinned-roster hash that does not
  match the lock, a missing lock, or an organisation chain it cannot read, stop the run; no vector holds them.

The reader's kind is read from these inputs: a roster read in place is the whole roster; a lock is a slice; a lock
with a resolution resolves. A resolution given beside a roster read in place contradicts it, and is refused
`resolution-without-lock`; `organisation/lock/` exists only to write that input. A slice checks a hail inside the teams it
holds, and reports a hail across teams `slice-cross-team` and the row not checked (§8; test 96). A resolving reader
checks against the organisation's state in force, unless the pin's standing refuses every right or names `hail` among
the extensions that changed: then `pin-behind` (§8). The key table of `ext.hail` is checked whole, except the encoding
of an oversight key's recipient, which is the sealed-content row's. `received/` serves a bot's reply to a person:
without it such a reply reports `reply-not-checked` and the row not checked. The reference time, where given, is the
moment grants with `until` are read at. A grant bounded by a time with no reference time, or by a tracker, whose
history is no input of this row, is neither held nor refused: the span reports `grant-not-checked` and the row not
checked, and the return-grant configuration check reports the same for every span where it can't decide. Every grant
of the actor's for the act is judged together, in any order: one that grants passes; otherwise one that can't be
judged for want of an input is reported; otherwise the act is refused. A reply whose received hail can't be verified
for want of an input, such as an Owner key line bounded in time with no reference time, reports that input not
checked, and is not refused for it.

**The Owner's keys.** sealanes/1 §3.5 makes the Owner's signing-keys file git's allowed-signers file, and the check
honours each option of a key's line as that format defines it: `namespaces` is a pattern list (`*`, `?`, and `!` to
negate); `valid-after` and `valid-before` bound the key at the reference time, inclusive, and a time without `Z`,
which the format reads in the system's zone, or no reference time, leaves the key unchecked (`owner-key-not-checked`
and the row not checked, the hail never verified); `cert-authority`, or any option the format doesn't define, keeps
the line from verifying the hail's signature.

A token derived from no declared origin is a workbench name (§3). Names that start with a dot, such as the Finder's `.DS_Store`, are ignored in every folder.

- **`pass`:** every hail in the message conforms to the rows checked. A pass authorises nothing: its `reports` name
  each row of §11 the run did not check (§11; test 100).
- **`refuse`:** a hail is refused, under the rule `rule` names. The check stops at the first failure and reports that
  rule (§11), so a refusal's `reports` are empty.
- **`not-judged`:** the edition has the check judge nothing here. It is the text's own outcome, never a placeholder
  for a case the text leaves open: a message with no span from `hail/1` through `end hail/1`, or its sealed form, holds
  no hail, and everything outside a span is ignored (§2). Its `reports` are empty.
- **A report** is something the edition has `hail check` say about a hail that passes without refusing it for that,
  such as an id whose minute differs from its written time (§3, §11).

**The rows a run did not check.** A form-only run reports the checks it did not make (§11). A run over a message and
a reference time reports, as one rule each, every row of §11's table but the form row and the time row, each plain
span the rows it was not given the inputs of:
`not-checked-authority`, `not-checked-live-record`, `not-checked-pointers`, `not-checked-duplicates`,
`not-checked-decision`, `not-checked-act`, `not-checked-sessions`, `not-checked-forced-end`, `not-checked-away` and
`not-checked-decider`. Without a reference time it also reports `time-form-only`. A run given the authority row's
inputs checks that row for each plain span and doesn't report `not-checked-authority` for it; a sealed span keeps it. A run given the context row's
inputs checks that row for each plain span and doesn't report `not-checked-pointers` for it; a sealed span keeps
`not-checked-pointers`, and reports each scan input left out as a plain span does. The run reports what of the row it could not check:
`secrets-not-checked`, `private-ids-not-checked`, `own-home-not-checked` and `limits-not-checked` for an input left out; `head-unknown`
for a repository `written-against` names that is not declared or has no head given; and `ancestry-unknown` where
the head is another commit and no ancestry is given. `moved` is reported only where the ancestry given has the
head descend from the named commit: a head behind it or beside it has not moved past it (§3). A sealed span given alone is
checked for its form, and reports `contents-not-checked` (§9a). The row "a roster or keys change" checks a roster
change, not a hail, and is not reported on one.

A rule's id is the one `hail::Refusal` or `hail::Report` gives it, with its section of the text; the two share one
namespace of ids, `rule` names a refusal and `reports` reports, and the section is never written in the
vector. The files are byte-exact: `.gitattributes` keeps git from converting them on any system.

**Canonical form and Unicode.** The check rebuilds each message's canonical form before it finds frame lines (§2,
§9): UTF-8 in NFC as Unicode 13.0.0 defines it, LF line endings and no trailing spaces. Its NFC tables are
generated by `tools/hail/generate_unicode.py` from Unicode 13.0.0's data, pinned by sha256 in
`crates/hail/unicode-13.0.0/`: `CompositionExclusions.txt`, `DerivedNormalizationProps.txt` and
`NormalizationTest.txt` there, and `UnicodeData.txt` read from `conformance/sealanes/unicode-13.0.0/`, its hash
checked equal. A unit test runs every line of `NormalizationTest.txt`.

**Two rules no edition holds yet.** `body-frame-line` and `control-character` are an Owner's answer until an edition
takes them. The section each names is the one where the next edition would write it, and its vectors' `about` says so.
- **`body-frame-line`** reads a plain span's body, once the header is read and before any other rule of the body. A
  body line that is a line of a frame's form, `hail/<n>` or `end hail/<n>`, plain or with ` sealed`, `<n>` digits, or
  one of the four armor lines of a signature block (§9) or a sealed span (§9a), is refused. A line is read as the
  rebuild gives it, so `hail/12` with trailing spaces is one. A line that only looks like one, such as `end hail/1x`,
  `hail/`, `Hail/2` or an armor line with more text on it, is body text. A plain span runs from `hail/1` through the
  next `end hail/1` (§2): `hail/1`, `hail/1 sealed`, `end hail/1 sealed` and every line of another form are lines
  of it, read as body after the pointer line and as header lines before it; `end hail/1` ends it, and a line after it
  is outside every span. A sealed span holds no body: a line other than its `to-mailbox:` line and its armor is
  refused `sealed-span`.
- **`control-character`** reads every line of a hail once it is rebuilt: from its opening line through its end line,
  plain or sealed, and its signature block's lines. A control character is one of Unicode's class Cc, and a direction
  control is U+202A to U+202E or U+2066 to U+2069. It reads a span's lines as the span is found, and a signature
  block's once its end line is found, before the form of either or any rule of the span's content is read. A line's LF
  is not part of it, nor is the one CR before the LF that the rebuild removes; any other CR is. U+200E, U+200F and
  U+061C are neither, and stay body text. A line outside every span is no hail's, and neither rule reads it.

**The rules.** Each rule the vectors name, as `hail::Refusal` and `hail::Report` hold them, each in the order the text
gives them: a refusal is the first failure a check finds, except that the runner decides a resolving reader's slice
before `hail` reads the message, as `[resolving]` above describes. A report is said of a message that passes. A test
holds this table to the code.

| Rule | Section | Kind | What |
|---|---|---|---|
| `unknown-form` | §2 | refusal | A line `hail/<n>` or `hail/<n> sealed`, with `<n>` other than 1, opens a span the reader can't read. |
| `second-signature` | §2 | refusal | A second signature block straight after the first is not part of this edition's form. |
| `body-lines` | §2 | refusal | The body holds more than five lines. |
| `after-ask` | §2 | refusal | A line stands between `ask:` and `end hail/1`. |
| `body-frame-line` | §2 | refusal | A body line is a line of a frame's form, `hail/<n>` or `end hail/<n>`, plain or with ` sealed`, or an armor line of a signature block or a sealed span. |
| `header-line` | §3 | refusal | A header line is not `key: value` pairs: a key of lower-case letters and hyphens, a colon and exactly one space, then a value, each further pair or continuation after ` · `. |
| `key-twice` | §3 | refusal | A key is given twice. |
| `retired` | §3 | refusal | A retired header key (`roster`, `keys`, `acting-for`, `handover`), `kind: grant-change` or `from: owner pair`. |
| `unknown-key` | §3 | refusal | A DECISION, AWAY or BACK carries a key this edition doesn't define. |
| `key-not-allowed` | §3 | refusal | A key scoped to one kind of hail or marker is on another, or `ask` is written as a header pair. |
| `missing-key` | §3 | refusal | A key required on every hail, or on this kind of hail or marker, is missing. |
| `from-form` | §3 | refusal | `from` is not a bot with its session, `person.<id>` or `owner desktop`. |
| `to-form` | §3 | refusal | `to` is not one or more addresses separated by `, `, or `owner` alone. |
| `id-form` | §3 | refusal | `id` or `supersedes` is not an id: a sender, a calendar stamp `<yyyymmdd>T<hhmm>Z` and six characters of the random key's alphabet; or an id-shaped `re` part has a stamp that is no calendar time. |
| `id-sender` | §3 | refusal | The id's sender prefix doesn't match `from`. |
| `re-form` | §3 | refusal | `re` is not `none`, an id, or ids separated by `, `, or a free description: one id-shaped part makes it an id list, and a free description is never `none` in another case. On a DECISION it is exactly one id. |
| `written-against-form` | §3 | refusal | An item of `written-against` names a repository commit that is not 7 to 64 lower-case hex digits. |
| `written-time` | §3 | refusal | `written-against` doesn't hold exactly one time in §3's time form. |
| `model-form` | §3 | refusal | `model` is not two non-space words. |
| `time-form` | §3 | refusal | `decide-by`, `harm-by` or the time of `answer: later` is not in §3's time form. |
| `time-offset` | §3 | refusal | A time's offset has an hour outside 00–23 or a minute outside 00–59: an offset is bounded as the time is, in each of its forms. |
| `keyword-form` | §3 | refusal | A value that is a keyword (`for`, `disposition`, `kind`, `scope`, `outcome`) is not one of its keywords, or a marker this edition defines or reserves is written in another case. |
| `name-form` | §3 | refusal | `clarify`, `raised-by`, `bot`, `session` or `for-team` is not the address, id or name its row gives. |
| `clarify-order` | §3 | refusal | `clarify` doesn't name the raising bot first, then the gate, or only the sender for its own need. |
| `answer-form` | §3 | refusal | `answer` is not `yes`, `no`, `option A` to `option E`, `words`, `later <time>` or `talk`. |
| `brief-form` | §3 | refusal | `brief` is not a brief id followed by a sha256. |
| `stop-subject` | §3 | refusal | A STOP's `subject` names more than one item. |
| `decision-form` | §3 | refusal | `decision` is not an Owner's id followed by a sha256. |
| `session-distinct` | §3 | refusal | A START's or FORCED END's `session` is the sender's own session. |
| `matter-form` | §3 | refusal | `matter` is not a matter reference: `tracker:`, `ask:`, `thread:` or `plan:`. |
| `pointer-form` | §3 | refusal | `pointer` is not a repository, head or workbench pointer with its sha256, or `none`; or a workbench pointer names a commit. |
| `ask-missing` | §3 | refusal | A REQUEST or QUESTION has no `ask:` line. |
| `ask-forbidden` | §3 | refusal | A marker other than REQUEST and QUESTION has an `ask:` line. |
| `ask-twice` | §3 | refusal | A second `ask:` line. |
| `ask-form` | §3 | refusal | The `ask:` line is not `ask: ` and one non-empty line. |
| `list-workbench` | §3 | refusal | A hash list is named by a workbench pointer. |
| `list-revision` | §3 | refusal | A hash list's repository doesn't occur exactly once in `written-against`, or the pointer's commit doesn't resolve to that revision. |
| `list-empty` | §3 | refusal | A hash list is empty. |
| `list-line` | §3 | refusal | A hash list is not UTF-8 lines, each `<64 lower-case hex>`, two spaces and a path with no CR or NUL, followed by LF. |
| `list-path` | §3 | refusal | A listed path is absolute or a drive path, or holds an empty, `.` or `..` segment. |
| `list-missing` | §3 | refusal | A hash list, a file it names or a parent folder of either has no tree entry at the list's revision. |
| `list-entry` | §3 | refusal | A hash list, a file it names or a parent folder of either is a symbolic link or a submodule, or the file is not a regular file. |
| `list-hash` | §3 | refusal | A listed file's sha256 is not the hash its line gives. |
| `list-cycle` | §3 | refusal | A hash list names itself or a list that names it. |
| `pointer-hash` | §3 | refusal | The sha256 of the bytes a pointer names, as committed or in the workbench, is not the pointer's hash. |
| `reserved-marker` | §4 | refusal | A reserved marker without its use: DECISION, AWAY or BACK not from the Owner, or INSTRUCTION. |
| `receipt-re` | §4 | refusal | A receipt (RECEIVED, WILCO, DECLINED or DONE) must quote an id or ids separated by `, ` in `re`. |
| `roster-unreadable` | §8 | refusal | The chain to a counted state can't be read, or its state in force holds no roster the reader accepts: every right is refused. |
| `pin-behind` | §8 | refusal | A resolving reader's pin is behind a state in force that refuses every right, or that changed hail's extension: hail acts are refused until the pin moves. |
| `resolution-without-lock` | §8 | refusal | A resolution is given beside a roster read in place: a reader resolves only from its lock, so the inputs contradict each other. |
| `edition` | §8 | refusal | The state in force doesn't adopt this edition: `ext.hail` is missing, or names another spec or edition. |
| `extension` | §8 | refusal | `ext.hail` breaks its key table: an unknown key, a wrong type, a value out of range, a malformed name, a team, bot or destination the whole roster doesn't declare, or `mode = "enforce"` while §13's six cases are open. |
| `return-grant` | §8 | refusal | A permitted bot-to-bot message, STOP or WARNING that §4 answers has no reverse `hail.answer`: the configuration refuses every hail act. |
| `not-in-roster` | §8 | refusal | The sender is no bot of the roster in force, nor a decider with a signing key. |
| `reach` | §8 | refusal | No right in effect reaches a recipient for this hail: `hail.message` with its `only`, `hail.answer`, `hail.stop`, `hail.warn` or `hail.owner`, as §8's table selects, or the reply rule for a person's verified hail. |
| `not-utf8` | §9 | refusal | The bytes are not UTF-8, so no canonical form can be rebuilt. |
| `control-character` | §9 | refusal | Once the hail is rebuilt in canonical form, a line of it holds a control character of Unicode's class Cc or a direction control, U+202A to U+202E or U+2066 to U+2069: a line from its opening line through its end line, plain or sealed, or of its signature block. |
| `signature-block` | §9 | refusal | A signature block straight after `end hail/1` is not an armored SSH signature. |
| `unsigned` | §9 | refusal | A hail its team's `signing-required` requires to be signed, or either team's on a route across teams, carries no signature. |
| `not-signed` | §9 | refusal | A person's hail carries no signature. |
| `signature` | §9 | refusal | A signature block doesn't verify over the hail's canonical bytes. |
| `namespace` | §9 | refusal | A signature is not in its hail's namespace, `hail-decision` on a DECISION and `hail` on any other, or the pattern list of the Owner key line's `namespaces` doesn't match it. |
| `key-window` | §9 | refusal | The Owner's key line bounds it with `valid-after` or `valid-before`, and the reference time lies outside that window. |
| `signer` | §9 | refusal | The signing key is not the sender's: the bot's own key or its team's, the decider's `keys.signing`, or an Owner's key on a line whose options let it sign a hail: not `cert-authority`, nor one the allowed-signers format doesn't define, nor a `namespaces` value longer than 256 bytes. |
| `sealed-span` | §9a | refusal | A sealed span is not `to-mailbox: <opaque id>`, then the age armor: padded base64 wrapped at 64 columns, as the age v1 format §9a names defines its armor. |
| `options` | §10b | refusal | Option lines other than two to five, sequential from `option A:` and each used once, on `kind: decide`; or an option line on another kind. |
| `options-only` | §10b | refusal | A body line other than an option line on `kind: decide`: its body lines are its options only, its reason in `why` and its detail behind the pointer. |
| `reason-required` | §10b | refusal | A DECISION answering `no` or `words` gives no reason in its body. |
| `future-time` | §11 | refusal | The written time is more than two minutes after the reference time. |
| `pointer-unavailable` | §11 | refusal | Bytes a pointer or a hash list names are not among the inputs: no commit given resolves its commit uniquely, or no bytes are given for its file. |
| `home-path` | §11 | refusal | An absolute home path in a listed form, or the sending user's own home as given, is in the header, body or ask, or in a file the pointer or a hash list reaches. |
| `private-id-prefix` | §11 | refusal | A public-bound hail is checked against a private-id prefix that is not ASCII letters ending `-`. |
| `private-id` | §11 | refusal | A bounded private id is in a public-bound hail's header, body or ask, or in a file the pointer or a hash list reaches. |
| `secret-pattern` | §11 | refusal | A pattern of the secret-pattern set given doesn't compile as a regular expression in RE2 syntax, or exceeds the bounds on its compiled size or nesting, or the set exceeds the bounds on its number of patterns or their compiled size together: every hail the set would scan is refused until it is fixed. |
| `secret` | §11 | refusal | The versioned secret-pattern set finds a secret in the header, body or ask, or in a file the pointer or a hash list reaches. |
| `location-limit` | §11 | refusal | A scanned location holds more bytes than the scan policy's limit: a hail, from `hail/1` through `end hail/1` in canonical form with each line's LF, or a file the pointer or a hash list reaches, measured before it is hashed or scanned. |
| `line-limit` | §11 | refusal | A line the scan reads, in the header, body or ask or in a file the pointer or a hash list reaches, holds more bytes than the scan policy's limit, a line read as the secret scan reads it; each line of a location is measured before any scan runs on that location. |
| `files-limit` | §11 | refusal | The hash lists reach more files than the scan policy's limit, a file counted each time a list's line names it; the walk stops at the first line past the limit, before its file is hashed. |
| `message-limit` | §11 | refusal | The bytes the scan reads in one message exceed the scan policy's limit, counted over every hail and every file each time a pointer or a list's line reaches it: the location that would pass the limit is refused before it is hashed or scanned. |
| `unknown-marker` | §2 | report | A marker this edition doesn't define, and doesn't reserve, is read as INFORMATION and never acted on. |
| `id-minute` | §3 | report | The id's minute differs from the written time's minute in UTC. |
| `moved` | §3 | report | A repository `written-against` names has moved past the commit it names: the lookup given says its head descends from that commit. |
| `head-unknown` | §3 | report | No head was given for a repository `written-against` names, or it is not declared, so whether it moved was not checked. |
| `ancestry-unknown` | §3 | report | A repository's head is not the commit `written-against` names, and no ancestry was given, so whether it moved past it was not checked. |
| `slice-cross-team` | §8 | report | The roster was read from a slice, which can't check a hail across teams: the row "sender, reach, return grant, signing flag, keys, edition and pin" was not checked for it. |
| `grant-not-checked` | §8 | report | A grant the row needs is bounded by a time or a tracker, and no reference time or tracker history was given, so whether it holds was not checked. |
| `reply-not-checked` | §8 | report | A bot's response to a person was not checked against the hail it answers: none was given. |
| `edition-differs` | §8 | report | A `hail@<commit>` names text of another edition than the roster adopted, as the lookup given says; it grants and refuses nothing. |
| `edition-not-compared` | §8 | report | A `hail@<commit>` names the writer's text, and no lookup given says which edition it is, so it was not compared with the adopted edition. |
| `owner-key-not-checked` | §9 | report | The Owner's key line bounds it in time, and no reference time was given, or the line's time names no zone, so the key was not checked and the hail not verified. |
| `team-key` | §9 | report | The signature verifies against the sender's team key, which proves the team, not the bot. |
| `contents-not-checked` | §9a | report | A sealed span was checked for its form only; its contents were not checked. |
| `time-form-only` | §11 | report | No reference time was given, so the written time was checked for its form only. |
| `secrets-not-checked` | §11 | report | The scanned bytes were not checked against a secret-pattern set: none was given, or the set given holds no patterns. |
| `private-ids-not-checked` | §11 | report | No destination was given, so the scanned bytes were not checked for private ids. |
| `private-id-prefix-malformed` | §11 | report | A configured private-id prefix is not ASCII letters ending `-`. |
| `own-home-not-checked` | §11 | report | The sending user's own home was not given, or each entry given is empty, so the scanned bytes were checked for the listed home-path forms only. |
| `limits-not-checked` | §11 | report | One or more of the scan policy's limits was not given, on a location's bytes, a line's bytes, the files the hash lists reach or the bytes the scan reads in one message, so the scan was not bounded by it. |
| `not-checked-authority` | §11 | report | The row "sender, reach, return grant, signing flag, keys, edition and pin" was not checked. |
| `not-checked-live-record` | §11 | report | The row "live versus record" was not checked. |
| `not-checked-pointers` | §11 | report | The row "pointers and stale context" was not checked. |
| `not-checked-duplicates` | §11 | report | The row "duplicate or reused id" was not checked. |
| `not-checked-decision` | §11 | report | The row "DECISION, expiry, change of mind, standing evidence, one use" was not checked. |
| `not-checked-act` | §11 | report | The row "act check" was not checked. |
| `not-checked-sessions` | §11 | report | The row "session state, cap and waiting work" was not checked. |
| `not-checked-forced-end` | §11 | report | The row "FORCED END" was not checked. |
| `not-checked-away` | §11 | report | The row "AWAY/BACK and lead's `hail.owner`" was not checked. |
| `not-checked-decider` | §11 | report | The row "a decider or disclosure" was not checked. |

**`PENDING`** is this implementation's state, not part of the suite, and stays out of the public export. It lists the
vectors the code doesn't meet yet, one id per line and nothing else. A vector listed that passes fails the run, so the
list only shrinks. Every vector's expectation names only rules `hail` holds: a vector whose expectation waits on the
Owner's answer joins the suite once the answer settles it.

hail/1's third edition is signed, and it is the text in `hail/spec/`. The vectors are built from its text: Appendix
C's frames byte for byte, §11's tests of these rows, and the advisor's set re-checked against it. Where the text and
a vector differ, the vector changes, never the text.
