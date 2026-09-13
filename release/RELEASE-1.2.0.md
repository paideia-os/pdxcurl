# pdxcurl v1.2.0 -- release note (Wave V drain, unsigned source-tag)

**Repo:** github.com/paideia-os/pdxcurl
**Wave:** R100 user-space networking tools -- Wave V foundation drain
(`design/networking/r100-user-tools-plan.md` §7 + §13.3 +
`design/networking/pdxcurl-design.md`, in the paideia-os monorepo).
**Version at this release:** 1.2.0 (unsigned source-tag).
**Follows:** the pdxsock v1.1.0 -> v1.2.0 unsigned-source-tag pattern
(a second unsigned drain layer before the dual-signed release-source
landing at v1.3.0, per the pdxsock precedent slid one minor).

This document is the release note for what v1.2.0 ships and the
operator runbook for cutting the tag. The actual `git tag v1.2.0` +
`git push` is a manual step main performs separately from this
milestone -- see section 5.

## 1. What v1.2.0 ships

v1.2.0 is the **Wave V drain** landing across pdxcurl's earliest
foundation work -- five open issues from M1 and M3 close. Nothing in
this release touches the _start body: v1.2.0 is a rodata-only
foundation drain that unblocks parallel parser + emit-site work
without churning the wire the STUB tool already emits at v1.1-B.

Concretely:

- **Error taxonomy rodata namespace** (pdxcurl#12, M3-005). New
  `src/error_taxonomy.pdx` module `ErrorTaxonomy` publishes 13
  `RC_*` result_code constants (0=OK..12=INTENT) and 11 `EXIT_*`
  exit-code constants as `pub let ... : u64` .rodata. Every
  failure mode maps to a distinct pair per design-doc §7. Wire-in
  at each emit-site lands per-milestone as pdxcurl#5..#11 close.
- **Argv-surface rodata namespace** (pdxcurl#2, M1-002 partial).
  New `src/argv_surface.pdx` module `ArgvSurface` publishes the
  long-form and short-form flag literals + argc bounds
  (`ARGV_MIN=2`, `ARGV_MAX=32`) + canonical HTTP method value
  strings, each with a sibling `_LEN` u64 wire byte count. The
  parser BODY is deferred behind libpdx-argv M1 or a hand-rolled
  scanner.
- **`--dry-run` diagnostic rodata reservation** (pdxcurl#3,
  M1-003 partial). New `curl_msg_dry_run_stub` (`pdxcurl:
  --dry-run, no network I/O run\n`, 39 wire bytes) in
  `src/main.pdx`. Referenced by nothing at v1.2.0; landed so the
  future --dry-run body is a call-site edit rather than a rodata
  edit.
- **M3-003 honest-witness note** (pdxcurl#10). Comment-only edit
  to `src/main.pdx` documenting which fields of the v1.1-B emit
  do NOT yet fill and which upstream landings unblock each.
- **M1-001 formal close** (pdxcurl#1). The scaffold shipped at
  v1.1.0; Wave V reconciles the tracker with the shipped state.
- **Manifest + PDX_TOOL_VERSION bumps.** `manifest.pdxproj`
  `version = 1.2.0` + new sources listed. `PDX_TOOL_VERSION` in
  `src/tool_ident.pdx` -> `"1.2.0\0"`.
- **`release/RELEASE-1.2.0.md`** (this document) +
  **`release/manifest.pdxsig.txt`** refreshed to v1.2.0 UNSIGNED
  source-tag shape.

## 2. What v1.2.0 does NOT ship (deferred, honest scope)

- Argv parser BODY (pdxcurl#2 parser-side -- rodata half landed
  here; scan loop needs libpdx-argv M1 or hand-rolled).
- Real HTTP GET / HTTPS TLS / DNS resolve / audit INTENT/RESULT
  wiring -- pdxcurl#5..#9, #11, #13. Every one blocked on a
  satellite-lib landing (libpdx-net M2/M3/M4/M5, libpdx-url M1,
  libpdx-audit call-site at M3-002, pdxtrust M1).
- Redirect-chain smoke -- pdxcurl#17 (needs M4-002 real body).
- Dual-signed release manifest -- pdxcurl#18. Target slid from
  v1.2.0 to v1.3.0 after the Wave V drain (pdxsock precedent:
  collect drain before minting hybrid signatures).
- Mirror push -- pdxcurl#19 (needs M5-001).

## 3. Compiled artifact posture

At v1.2.0 the tool ELF (`build-out/pdxcurl.elf`) is:
- exit 2 on every invocation (no argv parse; STUB refusal),
- with one `HttpRequestRecord@0.1` semantic-pipe record emitted
  per invocation before the refusal (v1.1-B wire witness).

This is unchanged from v1.1.0. v1.2.0's additions are all
`pub let` .rodata symbols that are linked into the ELF but
referenced by nothing in _start; they exist for future call-site
edits to bind against without rodata churn.

## 4. Release manifest posture

`release/manifest.pdxsig.txt` remains the source form of the
eventual binary `manifest.pdxsig`. v1.2.0 is still UNSIGNED --
the `[artifacts.source]` set grows to include the two new
`.pdx` files, and every `<BLAKE3-*>` slot remains a
placeholder-until-fill the v1.3.0 dual-signed landing
(pdxcurl#18, M5-001) will populate.

## 5. Cutting the v1.2.0 tag (manual, main-only)

After this drain lands on `main`:

```sh
cd pdxcurl
git tag -a v1.2.0 -m "pdxcurl v1.2.0: Wave V foundation drain (unsigned source-tag)"
git push origin v1.2.0
```

Closes pdxcurl issues #1, #2, #3, #10, #12 (see CHANGELOG
`[1.2.0]` stanza for the per-issue delta and STATUS.md for the
overall status of every open pdxcurl milestone).
