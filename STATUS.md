# pdxcurl -- status

**Version:** v1.4.1 (documentation-only patch tag, 2026-09-13; M5-002
mirror-push runbook). Compiled artifact is byte-identical to v1.4.0.
**Wave:** R100 (paideia-os design/networking/r100-user-tools-plan.md §7 + §13.3).
**Design doc:** paideia-os design/networking/pdxcurl-design.md.

## Overall status

pdxcurl at v1.4.0 has a REAL HTTP-only request/response driver
(landed at v1.3.0, Wave CC) plus, at this Wave HH landing: an
https:// path gated behind `--trust=cap:<n>` and a WEAK-stub
`net_tls_wrap` call (`src/tls_wire.pdx`, pdxcurl#8 -- the request
still travels in PLAINTEXT since the stub always succeeds without a
real handshake); an `--audit-only` INTENT-only terminal path that
performs no network I/O and exits a distinct `200`
(`EXIT_SUCCESS_AUDIT_ONLY`, pdxcurl#11); and two new self-contained
smoke witnesses (`tests/curl_get_smoke.pdx`, pdxcurl#13; `tests/
curl_redirect_matrix.pdx`, pdxcurl#17). A real TLS handshake still
blocks on libpdx-net M5 + pdxtrust M1 cap minting; DNS-name hosts,
`-H`/`--header`, `-X` method override, and a real redirect-follow
loop remain deferred. The M1-002 argv parser BODY is still deferred
to libpdx-argv M1 (or a hand-rolled scanner), decoupled from the
flag-name namespace which landed at v1.2.0 -- src/main.pdx's own
hand-rolled Phase A scanner (not libpdx-argv) is what recognises
`--trust=`/`--audit-only` at this landing.

## Milestone status

| Milestone | Issue | Status |
|-----------|-------|--------|
| **M1-001 scaffold + caps.decl**                             | **pdxcurl#1**  | **LANDED (v1.1.0; formally closed at v1.2.0 -- caps.decl, link.ld, tools/build.sh, src/main.pdx STUB, tests/, manifest.pdxproj)** |
| **M1-002 argv surface (rodata half)**                       | **pdxcurl#2**  | **LANDED-PARTIAL (v1.2.0 -- src/argv_surface.pdx rodata namespace: flag literals + _LEN + argc bounds + method values; parser BODY deferred to libpdx-argv M1 or hand-rolled scanner)** |
| **M1-003 --dry-run first-runnable**                         | **pdxcurl#3**  | **LANDED-PARTIAL (v1.2.0 -- src/main.pdx curl_msg_dry_run_stub reserved; body follows M1-002 parser)** |
| **M2-001 libpdx-url integration**                           | **pdxcurl#4**  | **LANDED (v1.3.0 -- inline url_parse in src/main.pdx Phase B; src/url_parse.pdx holds the frozen result-shape .bss slots)** |
| **M2-002 HTTP-only GET (libpdx-net.http_get)**               | **pdxcurl#5**  | **LANDED (v1.3.0 -- real sys_socket/connect/send/recv path in src/main.pdx)** |
| **M2-003 --output FILE**                                     | **pdxcurl#6**  | **LANDED (v1.3.0 -- sys_open O_CREAT\|O_WRONLY\|O_TRUNC + sys_write + sys_close)** |
| **M2-004 --data / POST**                                     | **pdxcurl#7**  | **LANDED (v1.3.0 -- POST request line + decimal-encoded Content-Length)** |
| **M3-001 HTTPS path (net_tls_wrap + --trust)**                | **pdxcurl#8**  | **LANDED-STUB (v1.4.0 Wave HH -- --trust=cap:<n> argv gate + src/tls_wire.pdx WEAK-stub net_tls_wrap; request travels in PLAINTEXT since the stub always succeeds -- real handshake still blocks on libpdx-net M5 + pdxtrust M1 cap minting)** |
| **M3-002 libpdx-audit INTENT/RESULT**                        | **pdxcurl#9**  | **LANDED (v1.3.0 -- local audit surface in src/audit_emit.pdx + inline sys_semantic_send emits in src/main.pdx; libpdx-audit itself still does not exist in-repo, a future landing swaps the emit for a real audit_append_leaf call)** |
| **M3-003 semantic-pipe HttpRequestRecord@0.1 real fill**    | **pdxcurl#10** | **LANDED-WITNESS (v1.1-B stub emit landed at v1.1.0; v1.2.0 documents honest-witness gaps at src/main.pdx top comment; real per-request fill needs pdxcurl#5/#8/#9)** |
| **M3-004 --audit-only**                                      | **pdxcurl#11** | **LANDED (v1.4.0 Wave HH -- curl_audit_only_path in src/main.pdx: INTENT record emit + fd-2 fingerprint + sys_exit(EXIT_SUCCESS_AUDIT_ONLY=200), no network I/O)** |
| **M3-005 structured error taxonomy**                        | **pdxcurl#12** | **LANDED (v1.2.0 -- src/error_taxonomy.pdx module ErrorTaxonomy: RC_* result_code enum + EXIT_* exit code, distinct pair per failure mode per design-doc §7)** |
| **M4-001 HTTP GET happy-path smoke**                         | **pdxcurl#13** | **LANDED (v1.4.0 -- tests/curl_get_smoke.pdx: local WEAK-stub net_mock_connect/send/recv feed a canned 200 OK through a self-contained curl_run_request driver; self-contained per tools/build.sh's per-file compile-only posture)** |
| **M4-002 HTTPS happy-path smoke witness**                   | **pdxcurl#14** | **LANDED (v1.1.0 -- honest-blockage witness at tests/m4_002_https_smoke.pdx; real body EDIT at libpdx-net M5 unblock)** |
| **M4-003 failure-matrix smoke witness**                     | **pdxcurl#15** | **LANDED (v1.1.0 -- honest-blockage witness at tests/m4_003_failure_matrix.pdx; three roles r/n/k)** |
| **M4-004 audit-trail assertion witness**                    | **pdxcurl#16** | **LANDED (v1.1.0 -- honest-blockage witness at tests/m4_004_audit_trail.pdx; row-shape contract documented)** |
| **M4-005 redirect-chain smoke**                              | **pdxcurl#17** | **LANDED (v1.4.0 -- tests/curl_redirect_matrix.pdx: local redirect-decision engine asserts 301/302/303 downgrade + 307/308 preserve + 10-hop cap; src/main.pdx itself does not follow redirects yet)** |
| **M5-001 dual-signed manifest.pdxsig + .pdxdoc**             | **pdxcurl#18** | **LANDED-UNSIGNED (v1.4.0 -- CHANGELOG + README + release/RELEASE-1.4.0.md + release/manifest.pdxsig.txt refreshed; manifest STAYS an unsigned source-form placeholder pending the v0.33 PQ-signing crypto landing)** |
| **M5-002 mirror push**                                      | **pdxcurl#19** | **DOCUMENTED, BLOCKED ON R32 (v1.4.1 -- `release/mirror-push.md` pins the `pkgs.paideia-os/{staging,main}/pdxcurl/$VERSION/` URL convention, the pkg.tar layout, the 11-step push workflow, rollback + escalation. No code: every step from §3.4 onward needs R32's PQ signing substrate + `paideia_root_pk`, and the mirror host itself is not scaffolded. Do NOT mark LANDED until a real push executes)** |
| **v1.1-B semantic-pipe emission wire**                      | **pdxcurl#21** | **LANDED (v1.1.0 -- src/main.pdx marshals + emits HttpRequestRecord@0.1 via sys_semantic_send)** |
| **v1.1-C release closer + tag v1.1.0**                      | **pdxcurl#22** | **LANDED (v1.1.0 -- manifest.pdxproj version=1.1.0, CHANGELOG [1.1.0] stanza, release/manifest.pdxsig.txt source form, release/RELEASE-1.1.0.md note)** |

## Substrate readiness

| Substrate | Status |
|-----------|--------|
| paideia-as >= 0.36.0                    | AVAILABLE (Wave L baseline) |
| sys_semantic_send (SC+ ID 115)          | AVAILABLE (paideia-os R107-M0-001 #2350) |
| libpdx-audit audit_append_leaf          | AVAILABLE (libpdx-audit#31; pdxcurl uses a local sys_semantic_send audit surface, src/audit_emit.pdx, landed at v1.3.0 -- swap to a real audit_append_leaf IPC send is a future call-site edit) |
| libpdx-net net_connect / net_resolve    | UNAVAILABLE as a library (pdxcurl's own inline sys_socket/sys_connect since v1.3.0 covers IPv4-literal GET/POST without it; hostname DNS resolve still blocks on it) |
| libpdx-net net_tls_wrap                 | UNAVAILABLE (WEAK-stub passthrough in src/tls_wire.pdx since v1.4.0 fills the call-site shape; a real handshake blocks on libpdx-net M5) |
| pdxtrust KIND_TLS_TRUST cap mint        | UNAVAILABLE (blocked on pdxtrust M1) |
| libpdx-url url_parse                    | UNAVAILABLE (blocked on libpdx-url M1) |
| paideia-os smoke-runner fixture-seq     | UNAVAILABLE (paideia-os monorepo work) |
| R32 PQ signing substrate + `paideia_root_pk` | UNAVAILABLE (blocks M5-001's real signatures and every signing/push step of `release/mirror-push.md`) |
| `pkgs.paideia-os` mirror host + signing runner | UNAVAILABLE (not scaffolded; separate paideia-os infrastructure round. Independent of R32 -- clearing R32 alone does not unblock M5-002) |
