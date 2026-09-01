# pdxcurl

Improved-curl CLI for PaideiaOS. Capability-native trust (`KIND_TLS_TRUST` caps minted by `pdxtrust`, no ambient CA store, no `-k`/`--insecure` escape hatch), audit-first (`libpdx-audit` INTENT + RESULT records), semantic-pipe typed output, PQ-preferring (Ed25519 v1; MLDSA65 reserved), explicit `--dry-run` / `--audit-only`, structured error taxonomy — every failure mode has a distinct exit code and result_code. See `design/networking/pdxcurl-design.md` (in paideia-os) for the CLI's full design.

## Spec

Full design lives in the paideia-os monorepo at
[`design/networking/r100-user-tools-plan.md`](https://github.com/paideia-os/paideia-os/blob/main/design/networking/r100-user-tools-plan.md)
(softarch's R100 user-tools plan). Section references in issues point into that document.

This repository is one of seven satellite repos that together deliver the
R100 wave: `libpdx-net`, `libpdx-url`, `pdxcurl`, `pdxping`, `pdxdig`,
`pdxsock`, `pdxtrust`.

## License

MIT.