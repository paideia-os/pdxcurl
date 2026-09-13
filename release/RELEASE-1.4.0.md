# pdxcurl v1.4.0 -- release note (Wave HH drain, unsigned source-tag)

**Repo:** github.com/paideia-os/pdxcurl
**Wave:** R100 user-space networking tools -- Wave HH drain
(`design/networking/r100-user-tools-plan.md` §7 + §13.3 +
`design/networking/pdxcurl-design.md`, in the paideia-os monorepo).
**Version at this release:** 1.4.0 (unsigned source-tag).
**Follows:** v1.3.0's Wave CC real-body landing; this release closes
a five-issue cohort (pdxcurl#8, #11, #13, #17, #18).

This document is the release note for what v1.4.0 ships and the
operator runbook for cutting the tag. The actual `git tag v1.4.0` +
`git push` is a manual step main performs separately -- see section 5.

## 1. What v1.4.0 ships

- **--trust=cap:<n> HTTPS gate + WEAK-stub net_tls_wrap (pdxcurl#8,
  M3-001).** New `src/tls_wire.pdx` module `TlsWire`. `src/main.pdx`'s
  argv scanner recognises `--trust=cap:<n>` as a single-token flag;
  an https:// URL is refused (`curl_tls_not_yet`, exit 9) UNLESS a
  trust token was captured, in which case `TlsWire::
  tls_wire_net_tls_wrap` is called right after `sys_connect`
  succeeds. This toolchain has no linker-weak (STB_WEAK) mechanism
  and libpdx-net does not exist in this repo, so the "WEAK stub" is a
  real local function that always returns success -- the request
  proceeds in **plaintext**, not over a real TLS channel. `caps.decl`
  keeps `KIND_TLS_TRUST` optional pending pdxtrust M1 + a real
  libpdx-net M5 handshake.
- **--audit-only INTENT-only path (pdxcurl#11, M3-004).** A bare
  `--audit-only` flag short-circuits to an INTENT-only terminal
  branch: emits the audit record (`result_code = RC_AUDIT_ONLY = 11`)
  and a fd-2 fingerprint, then exits a new, distinct
  `EXIT_SUCCESS_AUDIT_ONLY = 200` -- no TCP connect, no HTTP
  send/recv.
- **GET happy-path smoke vs mocked fixture (pdxcurl#13, M4-001).**
  New `tests/curl_get_smoke.pdx`.
- **Redirect-chain method-preservation + 10-hop-cap smoke
  (pdxcurl#17, M4-005).** New `tests/curl_redirect_matrix.pdx`.
- **Manifest + PDX_TOOL_VERSION bumps.** `manifest.pdxproj`
  `version = 1.4.0` + new sources/tests listed. `PDX_TOOL_VERSION` in
  `src/tool_ident.pdx` -> `"1.4.0\0"`.
- **`release/RELEASE-1.4.0.md`** (this document) +
  **`release/manifest.pdxsig.txt`** refreshed to v1.4.0 + **README.md**
  status line.

## 2. What v1.4.0 does NOT ship (deferred, honest scope)

- A real TLS handshake -- `net_tls_wrap` stays a passthrough stub;
  blocked on libpdx-net M5 + pdxtrust M1 cap minting.
- DNS-name hosts, `-H`/`--header`, `-X`/`--method` override, chunked
  transfer-encoding, recv-loop for bodies > 4 KiB -- unchanged from
  v1.3.0.
- `--dry-run` body (pdxcurl#3 -- rodata reservation only).
- A real redirect-follow loop in `src/main.pdx` (`redirect_count`
  stays 0; pdxcurl#17's smoke exercises a local decision engine).
- **Actual dual signatures.** `release/manifest.pdxsig.txt` is
  refreshed but STAYS an UNSIGNED source-form placeholder -- the
  v0.33 PQ-signing crypto landing
  (`paideia-pq-sign::sign_release_artifact`, Ed25519 + ML-DSA-65
  hybrid) is not yet invocable from this repo's tooling. "Dual-
  signed" in the pdxcurl#18 issue title names the eventual target
  shape, not v1.4.0's actual content.
- Mirror push (pdxcurl#19, M5-002).

## 3. Compiled artifact posture

At v1.4.0 the tool ELF (`build-out/pdxcurl.elf`):
- runs a real HTTP GET/POST against an IPv4-literal host (unchanged
  from v1.3.0);
- refuses `https://` (exit 9) unless `--trust=cap:<n>` is given, in
  which case it proceeds in plaintext through the WEAK-stub TLS wrap;
- with `--audit-only`, emits one audit INTENT record and exits `200`
  without any socket-family syscall.

## 4. Release manifest posture

`release/manifest.pdxsig.txt` remains the source form of the
eventual binary `manifest.pdxsig`. v1.4.0 is still UNSIGNED -- the
`[artifacts.source]` / `[artifacts.tests]` sets grow to include every
file this repo has actually accumulated through v1.3.0 + v1.4.0
(`src/url_parse.pdx`, `src/audit_emit.pdx`, `src/tls_wire.pdx`, and
the two new `tests/*.pdx` smoke files), and every `<BLAKE3-*>` slot
remains a placeholder-until-fill the eventual real PQ-signing landing
will populate.

## 5. Cutting the v1.4.0 tag (manual, main-only)

After this drain lands on `main`:

```sh
cd pdxcurl
git tag -a v1.4.0 -m "pdxcurl v1.4.0: Wave HH TLS gate + audit-only + smoke matrix (unsigned source-tag)"
git push origin v1.4.0
```

Closes pdxcurl issues #8, #11, #13, #17, #18 (see CHANGELOG `[1.4.0]`
stanza for the per-issue delta and STATUS.md for the overall status
of every open pdxcurl milestone).
