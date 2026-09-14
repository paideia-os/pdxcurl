# pdxcurl

Improved-curl CLI for PaideiaOS. Capability-native trust (`KIND_TLS_TRUST` caps minted by `pdxtrust`, no ambient CA store, no `-k`/`--insecure` escape hatch), audit-first (`libpdx-audit` INTENT + RESULT records), semantic-pipe typed output, PQ-preferring (Ed25519 v1; MLDSA65 reserved), explicit `--dry-run` / `--audit-only`, structured error taxonomy — every failure mode has a distinct exit code and result_code. See `design/networking/pdxcurl-design.md` (in paideia-os) for the CLI's full design.

## Status

**v1.4.1** (documentation-only patch tag over v1.4.0's unsigned
source-tag; compiled artifact unchanged — adds
[`release/mirror-push.md`](release/mirror-push.md), the M5-002
mirror-push runbook, which is **blocked on R32**). CLI surface: `pdxcurl [-o|--output FILE] [-d|--data BODY] [--trust=cap:<n>] [--audit-only] <url>`. Real HTTP-only GET/POST against an IPv4-literal host; `https://` is refused unless `--trust=cap:<n>` is given, in which case the request is wrapped through a WEAK-stub `net_tls_wrap` (plaintext — a real TLS handshake still blocks on `libpdx-net` M5 + `pdxtrust` M1 cap minting). `--audit-only` emits an INTENT audit record and exits `200` without performing any network I/O. See `STATUS.md` for the full per-milestone breakdown.

## Spec

Full design lives in the paideia-os monorepo at
[`design/networking/r100-user-tools-plan.md`](https://github.com/paideia-os/paideia-os/blob/main/design/networking/r100-user-tools-plan.md)
(softarch's R100 user-tools plan). Section references in issues point into that document.

This repository is one of seven satellite repos that together deliver the
R100 wave: `libpdx-net`, `libpdx-url`, `pdxcurl`, `pdxping`, `pdxdig`,
`pdxsock`, `pdxtrust`.

## License

MIT.