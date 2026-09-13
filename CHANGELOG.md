# Changelog

All notable changes to `pdxcurl` are recorded here. Format:
keep-a-changelog style, semver-ordered, newest first.

<!--
Version discipline:
  v1.0.x -- reserved for a future 1.0-shape signed release plan.
             Not yet cut.
  v1.1.0 -- unsigned source-tag release (2026-09-13). STUB body +
             v1.1-B semantic-pipe emission wire + M4-002/003/004
             honest-blockage witnesses + PDX_TOOL_NAME extern.
  v1.2.0 -- reserved for the M5-001 dual-signed release-source
             landing (pdxcurl#18), following the pdxsock precedent.
-->

## [1.1.0] -- 2026-09-13 -- Wave M drain (unsigned source-tag)

### Added

- **Scaffold (pdxcurl#1, M1-001).** Repo bootstrap: `caps.decl`
  (KIND_USER mandatory, KIND_TLS_TRUST + KIND_IPC_ENDPOINT optional
  per design-doc §8), `link.ld` (mirrors pdxsock/mkfs.pdxfs R64v2
  shape, .text @ 0x00400000, .data @ 0x00600000), `tools/build.sh`
  (paideia-as >= 0.36.0 floor, iterates `src/*.pdx` + `tests/*.pdx`,
  links via `ld -T link.ld` to `build-out/pdxcurl.elf`),
  `manifest.pdxproj` at v1.1.0, `STATUS.md`, this file, and
  `.gitignore`. `src/main.pdx` STUB entry emits the v1.1-B semantic-
  pipe record then refuses argv with exit 2.

- **PDX_TOOL_NAME + PDX_TOOL_VERSION externs** (Wave 6 hotfix
  contract, same discipline rm/cp/cat/mv/mkdir follow). New
  `src/tool_ident.pdx` module `ToolIdent` declares `PDX_TOOL_NAME
  = "pdxcurl\0"` (8 bytes) and `PDX_TOOL_VERSION = "1.1.0\0"` (6
  bytes) as `pub let` .rodata symbols so a future libpdx-argv
  1.1.3+ bump resolves its UND refs without a source edit here.

- **v1.1-B semantic-pipe emission wire (pdxcurl#21).**
  `src/main.pdx` marshals a 48-byte `HttpRequestRecord@0.1` into
  `curl_record_buf` (@align(8) .bss) and emits it via
  `sys_semantic_send` (SC+ ID 115, R107-M0-001 #2350). Record shape:
  `magic:[u8;8] = PDXKCURL` (little-endian u64 0x4C5255434B584450),
  `version:u32 = 1`, `flags:u32 = 0`, `method_and_scheme:u32 = 0`,
  `status:u32 = 0`, `body_bytes:u64 = 0`, `redirect_count:u32 = 0`,
  `header_count:u32 = 0`, `result_code:u32 = 12 (INTENT)`,
  `reserved:u32 = 0`. Schema tag `0x4874747052657101` = "HttpReq"
  + version 0x01, grep-friendly ASCII mnemonic stable pre-
  paideia-os#2000 (schema registry). At v1.1.0 the emission is a
  wire-shape witness -- every numeric field is zero except the
  INTENT result_code -- and precedes the argv refusal so an
  operator sees one record per invocation. Real per-request emit
  (post-response fill of status / body_bytes / method / scheme /
  header_count) lands at M3-003 (pdxcurl#10). Emission ordering
  is inverted from the design-doc-final pipeline (INTENT before
  DNS, RESULT after response); the INTENT/RESULT pair lands with
  M3-002 (pdxcurl#9).

- **M4-002 HTTPS happy-path smoke witness (pdxcurl#14).** New
  `tests/m4_002_https_smoke.pdx` (module `M4002HttpsSmoke`).
  BLOCKED on libpdx-net M5 (net_tls_wrap / net_connect / net_resolve)
  + pdxtrust M1 (KIND_TLS_TRUST cap mint) per issue text (R100-PREP-
  001 + R100-PREP-005). At v1.1.0 emits `pdxcurl https-smoke
  BLOCKED on libpdx-net\n` on fd 2 + sys_exit(0) -- honest
  blockage witness, not fabricated pass. Real body (TCP connect
  127.0.0.1:8443 + TLS handshake against pinned KIND_TLS_TRUST
  cap in slot 1 + HTTP GET / + response validation + `pdxcurl
  https-smoke ok\n` on fd 2 + sys_shutdown + sys_exit(0)) lands
  as an EDIT of this file at M5 unblock. Fingerprint contract:
  `pdxcurl https-smoke ok\n` = 23 wire bytes, `pdxcurl https-smoke
  setup fail\n` = 31 wire bytes, `pdxcurl https-smoke trust FAIL\n`
  = 31 wire bytes, exit 0/5/8 taxonomy per pdxcurl-design.md §7.

- **M4-003 failure-matrix smoke witness (pdxcurl#15).** New
  `tests/m4_003_failure_matrix.pdx` (module `M4003FailureMatrix`).
  Three-role ELF driven by argv[1] first byte: `r`=connect-refused
  (127.0.0.1:9, expect exit 7 + `pdxcurl fail conn-refused\n`),
  `n`=nxdomain (RFC 6761 `invalid` TLD, expect exit 6 + `pdxcurl
  fail dns\n`), `k`=tls-key-mismatch (fixture TLS server + trust
  cap over different key, expect exit 8 + `pdxcurl fail tls-key\n`).
  BLOCKED on libpdx-net M3/M4/M5 + pdxtrust M1. At v1.1.0 every
  role emits `pdxcurl fail-matrix BLOCKED role=X\n` and exits 0.
  Co-witness of M4-004 (pdxcurl#16): every failure role emits a
  RESULT audit record whose `result_code` matches the design-doc
  §7 enum, and M4-004 asserts one row per M4-003 role.

- **M4-004 audit-trail assertion witness (pdxcurl#16).** New
  `tests/m4_004_audit_trail.pdx` (module `M4004AuditTrail`).
  Opens `/system/audit/pdxcurl.log` and asserts a matching
  INTENT/RESULT row-pair (shared audit_id) for every earlier
  smoke run in the same session (M4-001/002/003). Row shape
  (libpdx-audit @0.2 wire): 48 bytes @align(8), 16-byte magic
  (`PDXAUDIT`) + version + flags header, 8B audit_id, 8B
  timestamp_ns, 4B tool_id (0xC01AC071 = "curl"), 4B kind
  (0=INTENT/1=RESULT), 4B result_code (§7 enum), 4B reserved.
  BLOCKED on pdxcurl.M3-002 audit wire not yet landed (libpdx-
  audit#31 audit_append_leaf HAS landed; pdxcurl call-site is
  the pending piece) + /system/audit tmpfs mount at smoke time.
  At v1.1.0 emits `pdxcurl audit-trail BLOCKED\n` + sys_exit(0).

- **v1.1-C release closer (pdxcurl#22).** `manifest.pdxproj` at
  `version = 1.1.0`, `release/manifest.pdxsig.txt` (source form
  of manifest.pdxsig, following the pdxsock v1.2.0 template but
  as UNSIGNED source-tag -- no signature blocks filled in at
  v1.1.0), `release/RELEASE-1.1.0.md` (operator note for cutting
  the `v1.1.0` tag; documents what v1.1.0 ships and what it
  deliberately does not). Signed v1.2.0 landing scheduled at
  M5-001 (pdxcurl#18) following the pdxsock precedent.

### Deferred (not shipped at v1.1.0)

- Argv scanner (pdxcurl#2, M1-002); --dry-run (pdxcurl#3, M1-003);
  libpdx-url integration (pdxcurl#4, M2-001); real HTTP/HTTPS
  bodies (pdxcurl#5..#12, M2-M3); redirect chain (pdxcurl#17,
  M4-005); dual-signed release (pdxcurl#18, M5-001); mirror push
  (pdxcurl#19, M5-002).

### Encoder discipline

Every `.pdx` in this release honours the paideia-as 0.36+ pitfalls
per user memory `pdx encoder pitfalls`: no `test rN, rN`; no 2-op
`imul r, imm`; byte loads via `xor rN, rN; mov_b rN, [ptr]`; module
name PascalCase file basename; every label prefixed `curl_` (src)
or `hts_`/`fmx_`/`aud_` (tests); no `and reg, imm64`; single-line
string literals with `_len` sibling constants naming wire byte
counts (exclude trailing NUL).
