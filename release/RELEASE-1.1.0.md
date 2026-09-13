# pdxcurl v1.1.0 -- release note (Wave M drain, unsigned source-tag)

**Repo:** github.com/paideia-os/pdxcurl
**Wave:** R100 user-space networking tools
(`design/networking/r100-user-tools-plan.md` §7 + §13.3 +
`design/networking/pdxcurl-design.md`, in the paideia-os monorepo).
**Version at this release:** 1.1.0 (unsigned source-tag).
**Follows:** the pdxsock v1.1.0 → v1.2.0 pattern (unsigned source-tag
first, dual-signed release-source landing second).

This document is the release note for what v1.1.0 ships and the
operator runbook for cutting the tag. The actual `git tag v1.1.0` +
`git push` is a manual step main performs separately from this
milestone -- see §5.

## 1. What v1.1.0 ships

v1.1.0 is the **Wave M drain** landing across pdxcurl's earliest
scaffold work. Concretely:

- **Repo scaffold** (pdxcurl#1): `caps.decl`, `link.ld`,
  `tools/build.sh`, `manifest.pdxproj`, `STATUS.md`, `CHANGELOG.md`,
  `.gitignore`, and the initial `src/` + `tests/` tree.
- **STUB entry body** (`src/main.pdx`): argv-refusing exit-2 with
  the v1.1-B semantic-pipe emission running before the refusal.
- **v1.1-B semantic-pipe emission wire** (pdxcurl#21): 48-byte
  `HttpRequestRecord@0.1` marshalled + emitted via
  `sys_semantic_send` (SC+ ID 115). Schema tag `0x4874747052657101`
  = `HttpReq` + version 0x01. Every numeric field zero except
  `result_code = INTENT (12)` at v1.1.0.
- **PDX_TOOL_NAME + PDX_TOOL_VERSION externs** (`src/tool_ident.
  pdx`): pre-emptive UND-extern hooks for the future libpdx-argv
  1.1.3+ bump.
- **M4-002 / M4-003 / M4-004 witness suite** (pdxcurl#14/#15/#16):
  three compile-only witness `.pdx` files under `tests/` documenting
  the exact runtime contract (fingerprint bytes, exit codes,
  fixture-harness dependencies) each will implement once its
  upstream blockage clears.
- **v1.1-C release closer** (pdxcurl#22): this document,
  `release/manifest.pdxsig.txt` (source form, unsigned at v1.1.0),
  the `[1.1.0]` CHANGELOG stanza, and the manifest.pdxproj
  version bump.

## 2. What v1.1.0 does NOT ship (deferred, honest scope)

- Real HTTP GET / HTTPS TLS / DNS resolve / audit INTENT/RESULT
  wiring -- pdxcurl#5..#12, all blocked on satellite-lib landings
  (libpdx-net M2/M3/M4/M5, libpdx-url M1, libpdx-audit
  pdxcurl-side wire at M3-002, pdxtrust M1).
- Argv scanner + `--dry-run` -- pdxcurl#2 and pdxcurl#3.
- Redirect-chain smoke -- pdxcurl#17 (needs M4-002 real body).
- Dual-signed release manifest -- pdxcurl#18, lands at v1.2.0
  following the pdxsock precedent.
- Mirror push -- pdxcurl#19 (needs M5-001).

Every deferred item has a tracking issue and a documented substrate
blocker; see STATUS.md "Substrate readiness" table.

## 3. Compiled artifact posture

At v1.1.0 the tool ELF (`build-out/pdxcurl.elf`) is:
- exit 2 on every invocation (no argv parse; STUB refusal),
- with one `HttpRequestRecord@0.1` semantic-pipe record emitted
  per invocation before the refusal (v1.1-B wire witness).

It is not (yet) a usable HTTP client. It is a wire-shape witness
that consumers can start parsing HRR records against BEFORE
pdxcurl.M3-003 (pdxcurl#10) lands the real per-request emission.

## 4. Release manifest posture

`release/manifest.pdxsig.txt` is the source form of the eventual
binary `manifest.pdxsig`. At v1.1.0 it is UNSIGNED -- the
`[artifacts.source]` / `[artifacts.tests]` / `[artifacts.meta]`
BLAKE3 hash slots are placeholder-until-fill, and the `[signatures]`
block is not present. Dual-sign (hybrid Ed25519 + ML-DSA-65 via
`paideia-pq-sign::sign_release_artifact`) lands at v1.2.0 following
the pdxsock precedent.

## 5. Cutting the v1.1.0 tag (manual, main-only)

After this drain lands on `main`:

```sh
cd pdxcurl
git tag -a v1.1.0 -m "pdxcurl v1.1.0: Wave M drain (unsigned source-tag)"
git push origin v1.1.0
```

Closes pdxcurl issues #14, #15, #16, #21, #22 (see CHANGELOG
[1.1.0] stanza for the per-issue delta and STATUS.md for the
overall status of every open pdxcurl milestone).
