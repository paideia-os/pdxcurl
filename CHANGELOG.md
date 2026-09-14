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
  v1.2.0 -- unsigned source-tag release (2026-09-13). Wave V
             foundation drain: M1-001 scaffold reconciled, M1-002
             argv-surface rodata half, M1-003 --dry-run rodata
             reservation, M3-003 semantic-pipe honest witness,
             M3-005 error-taxonomy rodata namespace.
  v1.3.0 -- unsigned source-tag release (2026-09-13). Wave CC drain:
             M2/M3 real bodies -- inline url_parse (pdxcurl#4), real
             HTTP-only GET (pdxcurl#5), --output file (pdxcurl#6),
             --data / POST body (pdxcurl#7), libpdx-audit INTENT +
             RESULT record pair (pdxcurl#9). M5-001 dual-signed
             release-source landing slides one more minor.
  v1.4.0 -- unsigned source-tag release (2026-09-13). Wave HH drain:
             --trust=cap:<n> HTTPS gate + WEAK-stub net_tls_wrap
             (pdxcurl#8), --audit-only INTENT-only path (pdxcurl#11),
             GET happy-path smoke vs mocked fixture (pdxcurl#13),
             redirect-chain method-preservation + 10-hop-cap smoke
             (pdxcurl#17), release closer (pdxcurl#18 -- manifest
             stays UNSIGNED pending v0.33 PQ-signing crypto).
  v1.4.1 -- documentation-only patch tag (2026-09-13). M5-002
             mirror-push runbook (pdxcurl#19). No source, test, or
             build change: the compiled artifact is byte-identical
             to v1.4.0, so `manifest.pdxproj` `version`,
             `release/manifest.pdxsig.txt` `package-version`, and
             `PDX_TOOL_VERSION` in src/tool_ident.pdx all remain
             1.4.0 deliberately. The five-way version agreement
             `release/mirror-push.md` §3.1 requires applies to a
             real mirror push; v1.4.1 is a docs tag, not a push.
-->

## [1.4.1] -- 2026-09-13 -- M5-002 mirror-push runbook (documentation-only)

### Added

- **`release/mirror-push.md` (pdxcurl#19, M5-002).** The input
  contract for publishing a pdxcurl release to the `pkgs.paideia-os`
  package mirror, written while the release shape is fresh so the
  first real push is not a schema negotiation against a moving
  target. Pins:
  - **Mirror URL convention.** `https://pkgs.paideia-os/staging/pdxcurl/$VERSION/`
    for the author-signed upload, `https://pkgs.paideia-os/main/pdxcurl/$VERSION/`
    for the dual-signed publication, `main/index.pdxsig` for the
    signed index. Three files per published version: `pkg.tar`,
    `manifest.pdxsig` (byte-identical to the copy inside the tar, so
    `pkg verify` needs no network), `mirror.entry`.
  - **"Mirror" is not a git remote.** pdxcurl has exactly one remote
    (`origin`); this wave defines no second forge. The doc says so
    explicitly so a future reader does not add one on the strength of
    the M5-002 issue title.
  - **`pkg.tar` layout**, source-first per `libpdx-argv`'s precedent
    (`src/*.pdx` lowered on the target; `tests/` and `build-out/`
    not shipped), lexicographically ordered for a reproducible hash.
  - **Hash discipline: BLAKE3-256**, matching this repo's existing
    `<BLAKE3-*>` manifest slots -- with the three-way ecosystem
    divergence recorded rather than silently resolved (`shell`
    specifies sha3-256, `libpdx-argv` sha256, `rm` blake3-256 for
    the same index).
  - **The push workflow**, §3.1 pre-flight checklist through §3.11
    close-out: freeze tree -> build -> populate manifest hashes ->
    author-sign -> pack -> tag + push origin -> push staging ->
    runner promotion -> verify from a clean machine -> STATUS
    flip. Genericized on `$VERSION` from the start, explicitly to
    avoid `libpdx-elevate`'s hardcoded-`1.0.0` drift bug.
    Origin push always precedes mirror push, so the mirror never
    attests to a commit nobody can fetch.
  - **Rollback + escalation.** The mirror is append-only for signed
    releases; version numbers are immutable; withdrawal removes
    availability, not validity.
- **`STATUS.md`** M5-002 row flipped from `DEFERRED (needs M5-001)`
  to `DOCUMENTED, BLOCKED ON R32`, plus two new substrate-readiness
  rows (R32 signing substrate; the mirror host itself).

### Blocked

- **M5-002 cannot execute: BLOCKED ON R32.** Every step from
  `release/mirror-push.md` §3.4 onward depends on R32's
  post-quantum signing substrate and the `paideia_root_pk` root key
  this workflow signs and verifies against. This repo does not scope
  R32 and states no date for it -- it is recorded only as the open
  prerequisite. Secondary blockers, named so R32 is not mistaken for
  the only one: the `pkgs.paideia-os` mirror host and signing runner
  are not scaffolded, `release/manifest.pdxsig.txt` is still an
  unsigned source-form placeholder (pdxcurl#18, LANDED-UNSIGNED),
  `doc/pdxcurl.pdxdoc` is unwritten, and all four dependency repos
  (`libpdx-url`, `libpdx-net`, `libpdx-audit`, `pdxtrust`) have yet
  to ship to the mirror.

## [1.4.0] -- 2026-09-13 -- Wave HH TLS gate + audit-only + smoke matrix (unsigned source-tag)

### Added

- **--trust=cap:<n> HTTPS gate + WEAK-stub net_tls_wrap (pdxcurl#8,
  M3-001).** New `src/tls_wire.pdx` module `TlsWire` publishes
  `tls_wire_parse_cap_uri` (decodes the `cap:<n>` half of a
  `--trust=cap:<n>` argv token into a u64, or an all-ones sentinel on
  malformed input) and `tls_wire_net_tls_wrap(sock_fd, trust_cap,
  hostname_ptr, hostname_len) -> u64`. `src/main.pdx`'s Phase A
  scanner gains `curl_flag_trust_long` / `curl_take_trust` (single-
  token flag, no following argv slot consumed) storing
  `trust_present` + `trust_cap_n` into two new `curl_argv_slots`
  fields (+32/+40, widening the slot block from 32 to 56 bytes).
  Phase B.4's https gate now requires `trust_present != 0` before
  letting an https:// URL past `curl_url_parse_ok` (still refuses at
  `curl_tls_not_yet` / `EXIT_TLS_HANDSHAKE_FAILED=9` without a trust
  token, unchanged from v1.3.0); Phase C.1.5, right after a successful
  `sys_connect`, calls `tls_wire_net_tls_wrap`. libpdx-net does not
  exist in this repo and this toolchain has no STB_WEAK linker
  binding (see `src/tls_wire.pdx` header for the full rationale
  mirroring `tools/user/cat/src/schema_wire.pdx`'s precedent), so the
  stub is a real, locally-defined function that always returns 0
  (success-passthrough) -- the request proceeds in **plaintext** over
  the raw socket rather than performing a real TLS handshake. The new
  `curl_tls_wrap_fail` branch (RC_TLS_HANDSHAKE_FAILED=4 /
  EXIT_TLS_HANDSHAKE_FAILED=9) stays unreachable until a real
  `net_tls_wrap` lands at a future libpdx-net M5 unblock. `caps.decl`
  keeps `KIND_TLS_TRUST` optional -- promoting it to mandatory ahead
  of a real enforcement point (pdxtrust M1) would refuse every
  https:// invocation for a claim this landing cannot back up.

- **--audit-only INTENT-only path (pdxcurl#11, M3-004).** `src/main.
  pdx` gains `curl_flag_audit_long` / `curl_take_audit` (single-token
  flag storing `audit_only` at `curl_argv_slots+48`) and
  `curl_audit_only_path`: reached from `curl_url_parse_ok` (Phase
  B.4, checked BEFORE the https/--trust= gate, since an audit-only
  run performs no network I/O regardless of scheme) it determines
  method, marshals + emits an INTENT `HttpRequestRecord@0.1`
  (`result_code = RC_AUDIT_ONLY = 11`) to the audit sink via
  `sys_semantic_send`, writes `pdxcurl: --audit-only, INTENT emitted,
  no I/O\n` to fd 2, then `sys_exit(EXIT_SUCCESS_AUDIT_ONLY = 200)` --
  a new taxonomy constant in `src/error_taxonomy.pdx`, deliberately
  DISTINCT from the ordinary success exit (0) so a caller can tell an
  audit-only run apart from a completed request without parsing the
  audit stream. No `sys_socket`/`connect`/`send`/`recv` and no
  semantic-pipe `HttpRequestRecord@0.1` emit run on this path (that
  stream stays reserved for completed requests per M3-003's
  contract).

- **GET happy-path smoke vs mocked fixture (pdxcurl#13, M4-001).**
  New `tests/curl_get_smoke.pdx` (module `CurlGetSmoke`). Since
  `src/main.pdx` issues raw `syscall` instructions rather than named
  `net_*` wrappers (nothing to intercept) and this toolchain has no
  STB_WEAK binding, the "WEAK stub" is three real local functions
  (`net_mock_connect` / `net_mock_send` / `net_mock_recv`, the latter
  copying a canned `HTTP/1.1 200 OK ... hello world\n` response into
  the caller's buffer) feeding a self-contained `curl_run_request`
  driver that parses status + finds the body via the same
  `\r\n\r\n`-scan idiom `src/main.pdx` uses, writes the body to fd 1,
  and asserts a 12-byte body match. Exit 0 + `gets_msg_ok` on fd 2 on
  pass. Fully self-contained (tools/build.sh compiles every
  `tests/*.pdx` standalone and never links them against `src/*.pdx`
  or each other).

- **Redirect-chain method-preservation + 10-hop-cap smoke (pdxcurl#17,
  M4-005).** New `tests/curl_redirect_matrix.pdx` (module
  `CurlRedirectMatrix`). `src/main.pdx` does not follow redirects at
  v1.4.0 (`redirect_count` stays 0), so this witness carries its own
  local redirect-decision engine: `crm_redirect_method` asserts
  301/302/303 downgrade `POST`->`GET` and 307/308 preserve `POST`
  across five canned-response rows (each also stages a canned final
  200, parsed through the same bytes-`[9..12]` status-parse idiom),
  and `crm_follow_chain` simulates an all-redirect chain that must
  refuse at the 11th hop (`CRM_MAX_HOPS = 10`) rather than loop
  forever. A future landing that gives `src/main.pdx` a real
  Location-header follow loop replaces this file's local engine with
  calls into that shipped one, preserving the row shapes.

- **Release closer (pdxcurl#18, M5-001).** `manifest.pdxproj`
  `version = 1.4.0`; `sources:` gains `src/tls_wire.pdx`; `tests:`
  gains the two new smoke files. `PDX_TOOL_VERSION` bumped to
  `"1.4.0\0"` in `src/tool_ident.pdx`. `release/RELEASE-1.4.0.md`
  (operator note) and `release/manifest.pdxsig.txt` refreshed to
  v1.4.0 -- STILL an UNSIGNED source-form placeholder (every
  `<BLAKE3-*>` slot remains a fill-in): the v0.33 PQ-signing crypto
  landing (`paideia-pq-sign::sign_release_artifact`, Ed25519 +
  ML-DSA-65 hybrid) is not yet invocable from this repo's tooling, so
  "dual-signed" in the issue title names the target shape rather than
  v1.4.0's actual content. `README.md` gains a version/CLI-surface
  status line.

### Not shipped (still deferred)

- A real TLS handshake (`net_tls_wrap` stays a passthrough stub; see
  pdxcurl#8 above) -- blocked on libpdx-net M5 + pdxtrust M1.
- DNS-name hosts, `-H`/`--header`, `-X`/`--method` override,
  chunked-encoding, recv-loop for bodies > 4 KiB -- unchanged from
  v1.3.0.
- `--dry-run` body (pdxcurl#3 -- rodata reservation only).
- A real redirect-follow loop in `src/main.pdx` (pdxcurl#17's smoke
  exercises a local decision engine, not a shipped chain-follow).
- Actual Ed25519 + ML-DSA-65 signatures over the release manifest
  (blocked on the v0.33 PQ-signing crypto landing).
- Mirror push (pdxcurl#19, M5-002).

### Encoder discipline

Every new `.pdx` in this release honours the paideia-as 0.36+
pitfalls per user memory `pdx encoder pitfalls`: no `test rN, rN`; no
2-op `imul r, imm` (decimal decode/encode uses the shl-3/add/add *10
idiom throughout); no `and reg, imm64`; byte loads via `xor rN, rN;
mov_b rN, [ptr]`; module basename PascalCase (`TlsWire`,
`CurlGetSmoke`, `CurlRedirectMatrix`); every label prefixed `tlsw_` /
`gets_` / `crm_` per file (reserved-word discipline: `loop`, `if`,
etc. are keywords); single-line string literals with `_len` sibling
constants; no push/pop anywhere in the three new files, so every
`call` site lands with `rsp % 16 == 0` inherited from process entry
without needing alignment padding.

## [1.3.0] -- 2026-09-13 -- Wave CC M2/M3 real bodies (unsigned source-tag)

### Added

- **Inline url_parse (pdxcurl#4, M2-001).** New `src/url_parse.pdx`
  module `UrlParse` exposes the frozen parser result-shape as .bss
  slots (`curl_url_scheme`, `curl_url_ip`, `curl_url_port`,
  `curl_url_host_ptr`, `curl_url_host_len`, `curl_url_path_ptr`,
  `curl_url_path_len`), scheme prefix rodata (`CURL_PREFIX_HTTP` /
  `CURL_PREFIX_HTTPS` + `_LEN` siblings), default port constants
  (`CURL_PORT_HTTP_DEFAULT` = 80, `CURL_PORT_HTTPS_DEFAULT` = 443),
  and the `CURL_PATH_ROOT` = "/" default-path literal. The parser
  BODY lives inline in `src/main.pdx`'s `_start` block (Phase B --
  http:// prefix + IPv4 dotted-decimal host [a.b.c.d, 0..255 each,
  max 3 digits per octet] + optional :PORT slug + optional /path
  slug). libpdx-url does not exist in-repo; a future landing swaps
  the inline parser for `libpdx_url::parse` without touching the
  result-slot names or widths. https:// parses to scheme=1 but the
  request-path branch jumps to `curl_tls_not_yet` (exit
  `EXIT_TLS_HANDSHAKE_FAILED` = 9) since M3-001 / pdxcurl#8 is not
  yet wired.

- **HTTP-only GET (pdxcurl#5, M2-002).** `src/main.pdx`'s `_start`
  Phase C wires the real socket-family call chain: `sys_socket`
  (SC+ 87, AF_INET/SOCK_STREAM) + `sys_connect` (SC+ 91, ip_u32 +
  port) + `sys_send` (SC+ 92) of the assembled request +
  `sys_recv` (SC+ 93) of the response into a 4 KiB scratch. Every
  syscall failure has its own diagnostic on fd 2 and a distinct
  nonzero exit status (`EXIT_CONN_REFUSED` = 7, etc.). Response
  parse extracts HTTP status from bytes [9..12] of the first line
  and finds the body via a linear `\r\n\r\n` scan. Response body
  cap = 4 KiB single-recv MVP; a recv-loop landing is tracked as
  a follow-up.

- **--output FILE (pdxcurl#6, M2-003).** When `-o` / `--output` is
  present in argv, Phase D.1 calls `sys_open` (SC+ 2) with flags
  `O_CREAT | O_WRONLY | O_TRUNC` (0xC1) + mode 0644 (0x1A4) for
  the output path, writes the response body via `sys_write`
  (SC+ 1) to the returned fd, then `sys_close` (SC+ 3) it. Absent
  `--output` writes the body to fd 1 (stdout). `sys_open` failure
  emits `pdxcurl: open output fail\n` on fd 2 and exits
  `EXIT_CAPS_REFUSED` = 3.

- **--data / POST body (pdxcurl#7, M2-004).** When `-d` / `--data`
  is present in argv, the scanner captures the body pointer +
  computed strlen (`curl_argv_slots+16` / `+24`; body cap = 1024
  bytes) and Phase C.2 assembles a POST request instead of GET:
  request-line `POST <path> HTTP/1.0\r\n`, `Host: <host>\r\n`,
  `Content-Length: <decimal>\r\n\r\n<body>`. The decimal encode
  for Content-Length uses `div r64` (one-op form; two-op `imul` is
  banned per user memory `pdx encoder pitfalls`) with an LSD-first
  scratch buffer (`curl_clen_buf`, 24 B) that's reversed on emit
  into `curl_req_buf`.

- **libpdx-audit INTENT + RESULT (pdxcurl#9, M3-002).** New
  `src/audit_emit.pdx` module `AuditEmit` exposes the audit schema
  tag (`curl_schema_audit` = 0x0100_7475_4172_7543 = "CurAut"+01)
  + a 48-byte `curl_audit_record_buf` .bss slot distinct from the
  pipe-emit record. Phase C.0 emits an INTENT
  `HttpRequestRecord@0.1` with `result_code = RC_INTENT` (12)
  BEFORE `sys_connect`; Phase D.2 emits a RESULT record AFTER the
  response parse with `status` + `body_bytes` filled and
  `result_code = RC_OK` (2xx) / `RC_HTTP_4XX` (5) / `RC_HTTP_5XX`
  (6). Failure branches (`curl_connect_fail`,
  `curl_tls_not_yet`) inline-emit a RESULT record with
  `RC_CONN_REFUSED` (2) / `RC_TLS_HANDSHAKE_FAILED` (4). All
  audit records travel under a distinct schema tag from the pipe
  emit so a semantic-pipe consumer dispatches cleanly by schema.
  libpdx-audit does not exist in-repo; a future landing swaps
  `sys_semantic_send` for a real `audit_append_leaf` IPC send
  without changing the record shape.

### Changed

- **`src/main.pdx` FULL BODY REWRITE.** Retires the v1.2.0 argv-
  refusing STUB (which emitted a single HRR record then exited 2)
  with a real HTTP driver. The v1.1-B semantic-pipe emit is
  preserved (schema 0x4874747052657101 = "HttpReq"+01, 48-byte
  record) with all numeric fields now real (`method_and_scheme`
  from method + scheme, `status` from response parse, `body_bytes`
  from the write-count, `result_code` from the branch outcome)
  instead of zero.

- **`PDX_TOOL_VERSION` bumped `1.2.0` -> `1.3.0`** in
  `src/tool_ident.pdx`.

- **`manifest.pdxproj` version -> 1.3.0**; `sources:` list gains
  `src/url_parse.pdx` + `src/audit_emit.pdx`.

### Not shipped (still deferred)

- pdxcurl#8 M3-001 net_tls_wrap / https:// -- blocks on libpdx-net
  M5 landing.
- pdxcurl#11 M3-004 --audit-only body -- flag literal in
  `src/argv_surface.pdx` honoured by scanner (rejects unknown
  flags, exits usage) but the branch-body is not wired.
- Response body > 4 KiB (single sys_recv MVP; recv-loop follow-up).
- Chunked transfer-encoding parse (HTTP/1.0 in the request-line
  suppresses in most stacks).
- -H / --header repeated flag (namespace landed at v1.2.0 in
  `src/argv_surface.pdx`; scanner does not consume yet).
- -X / --method override to force HEAD / PUT / DELETE (v1.3.0
  picks GET when --data absent, POST when present).
- Redirect follow chain (pdxcurl scope: `redirect_count` field in
  HRR stays zero at v1.3.0).

## [1.2.0] -- 2026-09-13 -- Wave V drain (unsigned source-tag)

### Added

- **Error taxonomy rodata namespace (pdxcurl#12, M3-005).** New
  `src/error_taxonomy.pdx` module `ErrorTaxonomy` publishes the
  `RC_*` result_code enum (13 values, RC_OK..RC_INTENT) and the
  `EXIT_*` exit-code namespace (11 values) as `pub let ... : u64`
  .rodata constants. Every failure mode maps to a distinct pair
  per design-doc §7 -- DNS_FAILED (RC=1, EXIT=6), CONN_REFUSED
  (RC=2, EXIT=7), TLS_KEY_MISMATCH (RC=3, EXIT=8), TLS_HANDSHAKE_
  FAILED (RC=4, EXIT=9), HTTP_4XX (RC=5, EXIT=4), HTTP_5XX (RC=6,
  EXIT=5), TIMEOUT (RC=7, EXIT=11), TOO_MANY_REDIRECTS (RC=8,
  EXIT=12), NO_PERMISSION (RC=9, EXIT=3), DRY_RUN (RC=10, EXIT=0),
  AUDIT_ONLY (RC=11, EXIT=0), INTENT (RC=12, v1.1-B witness only).
  u64 storage even though wire fields are u32/u8 so a call-site
  load is one aligned `mov r64, [rip + LABEL]` (proven encoder
  pattern at src/main.pdx line 176 for the schema tag). Wire-in
  at each emit site lands per-milestone as pdxcurl#5..#11 close.

- **Argv-surface rodata namespace (pdxcurl#2, M1-002 partial).**
  New `src/argv_surface.pdx` module `ArgvSurface` publishes the
  flag-name literals (`--method`, `--output`, `--data`, `--header`,
  `--trust`, `--dry-run`, `--audit-only`) plus their short-form
  aliases (`-X`, `-o`, `-d`, `-H`, `-t`) plus the canonical HTTP
  method value strings (GET / POST / HEAD / PUT / DELETE) plus
  argc bounds (ARGV_MIN=2, ARGV_MAX=32) as .rodata. Every flag
  literal carries a sibling `_LEN` u64 with the wire byte count
  (excludes trailing NUL) so a length-first argv-slot comparison
  short-circuits fast. This is the rodata HALF of pdxcurl#2 --
  the parser BODY (the scan loop that binds flags to state)
  requires libpdx-argv M1 scan_options() to land or a hand-rolled
  scanner; landing the namespace now unblocks parallel parser
  work without churning the flag names at parser-landing time.

- **--dry-run diagnostic rodata reservation (pdxcurl#3, M1-003
  partial).** `src/main.pdx` now declares `curl_msg_dry_run_stub`
  (`pdxcurl: --dry-run, no network I/O run\n`, 39 wire bytes) as
  a `pub let` rodata slot. Referenced by no _start path at
  v1.2.0; landed so the future --dry-run body (blocked on
  pdxcurl#2 parser) is a single call-site edit rather than a
  rodata edit here. Emission contract documented at the
  declaration: parser detects `--dry-run`, marshals the HRR as
  it would emit it (`result_code = RC_DRY_RUN = 10`), writes
  this diagnostic to fd 2, exits `EXIT_OK = 0`.

- **M3-003 semantic-pipe emit honest-witness note (pdxcurl#10).**
  Comment-only edit to `src/main.pdx` documenting what the v1.1-B
  emission wire does NOT yet fill (method_and_scheme, status,
  body_bytes, header_count, redirect_count, real result_code,
  paired RESULT record) and which upstream landings unblock each
  field. The v1.2.0 emit is deliberately UNCHANGED from v1.1.0 so
  downstream consumers keep parsing HRR records against the
  v1.1-B fingerprint; the real per-request emit lands as an EDIT
  of this file once pdxcurl#5 / #8 / #9 clear.

- **M1-001 scaffold formally closed (pdxcurl#1).** Reconciliation
  entry -- the scaffold already landed at v1.1.0 (caps.decl,
  link.ld, tools/build.sh, src/main.pdx STUB, tests/, manifest.
  pdxproj). Wave V closes pdxcurl#1 in the tracker to match the
  shipped state.

- **Manifest bump.** `version = 1.2.0`. `sources:` extended with
  `src/error_taxonomy.pdx` + `src/argv_surface.pdx`. Scope
  comment rewritten to name the Wave V drain contents +
  everything v1.2.0 deliberately does not ship (v1.3.0 remains
  the target for the M5-001 dual-signed release-source landing;
  slid one minor from v1.2.0).

- **PDX_TOOL_VERSION bumped.** `src/tool_ident.pdx` -->
  `"1.2.0\0"` (same 6-byte array size; libpdx-argv resolves the
  UND ref against the new value at whatever future point the
  argv scanner lands).

- **Release-note + manifest source form.** `release/RELEASE-
  1.2.0.md` (operator note for cutting the `v1.2.0` tag) and
  `release/manifest.pdxsig.txt` refreshed to v1.2.0 UNSIGNED
  source-tag shape (new source files added to
  `[artifacts.source]`; every hash slot still placeholder).

### Deferred (not shipped at v1.2.0)

- Argv parser BODY (pdxcurl#2 parser-side); real HTTP/HTTPS
  bodies (pdxcurl#5..#9, #11, #13); redirect chain (pdxcurl#17);
  dual-signed release (pdxcurl#18, slid to v1.3.0); mirror push
  (pdxcurl#19).

### Encoder discipline

Every new `.pdx` in this release honours the paideia-as 0.36+
pitfalls per user memory `pdx encoder pitfalls`: module name =
PascalCase file basename (`ErrorTaxonomy`, `ArgvSurface`); no
function bodies -> no register plan needed; array size =
literal byte count including NUL, `_LEN` sibling = wire byte
count excluding NUL; single-line string literals throughout;
u64 storage for every enum / length constant.

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
