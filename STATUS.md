# pdxcurl -- status

**Version:** v1.2.0 (unsigned source-tag release, 2026-09-13; Wave V drain).
**Wave:** R100 (paideia-os design/networking/r100-user-tools-plan.md §7 + §13.3).
**Design doc:** paideia-os design/networking/pdxcurl-design.md.

## Overall status

pdxcurl at v1.2.0 is a **STUB scaffold** with the v1.1-B semantic-pipe
emission wire landed (v1.1.0) plus a Wave V rodata-only foundation
drain (this release): error taxonomy (`src/error_taxonomy.pdx`), argv
flag-name surface (`src/argv_surface.pdx`), and a `--dry-run`
diagnostic slot reserved in `src/main.pdx`. The _start body is
unchanged from v1.1.0 -- it compiles, links, and emits one
`HttpRequestRecord@0.1` per invocation (result_code=INTENT, other
fields zero) before refusing argv with exit 2 -- honest witness of
the wire without a fabricated request. Every real request-path
milestone (M2-001 libpdx-url, M2-002 http_get, M3-001 net_tls_wrap,
M3-002 audit INTENT/RESULT, M3-003 HRR real fill, M3-004 --audit-only)
is deferred to satellite-lib unblocks tracked in the pdxcurl issue
list. The M1-002 parser BODY is deferred to libpdx-argv M1 (or a
hand-rolled scanner), decoupled from the flag-name namespace which
landed at v1.2.0.

## Milestone status

| Milestone | Issue | Status |
|-----------|-------|--------|
| **M1-001 scaffold + caps.decl**                             | **pdxcurl#1**  | **LANDED (v1.1.0; formally closed at v1.2.0 -- caps.decl, link.ld, tools/build.sh, src/main.pdx STUB, tests/, manifest.pdxproj)** |
| **M1-002 argv surface (rodata half)**                       | **pdxcurl#2**  | **LANDED-PARTIAL (v1.2.0 -- src/argv_surface.pdx rodata namespace: flag literals + _LEN + argc bounds + method values; parser BODY deferred to libpdx-argv M1 or hand-rolled scanner)** |
| **M1-003 --dry-run first-runnable**                         | **pdxcurl#3**  | **LANDED-PARTIAL (v1.2.0 -- src/main.pdx curl_msg_dry_run_stub reserved; body follows M1-002 parser)** |
| M2-001 libpdx-url integration                               | pdxcurl#4  | DEFERRED (blocked on libpdx-url M1) |
| M2-002 HTTP-only GET (libpdx-net.http_get)                  | pdxcurl#5  | DEFERRED (blocked on libpdx-net M2) |
| M2-003 --output FILE                                        | pdxcurl#6  | DEFERRED (needs M2-002) |
| M2-004 --data / POST                                        | pdxcurl#7  | DEFERRED (needs M2-002) |
| M3-001 HTTPS path (net_tls_wrap + --trust)                  | pdxcurl#8  | DEFERRED (blocked on libpdx-net M5 + pdxtrust M1) |
| M3-002 libpdx-audit INTENT/RESULT                           | pdxcurl#9  | DEFERRED (audit_append_leaf landed at libpdx-audit#31; pdxcurl call-site not wired) |
| **M3-003 semantic-pipe HttpRequestRecord@0.1 real fill**    | **pdxcurl#10** | **LANDED-WITNESS (v1.1-B stub emit landed at v1.1.0; v1.2.0 documents honest-witness gaps at src/main.pdx top comment; real per-request fill needs pdxcurl#5/#8/#9)** |
| M3-004 --audit-only                                         | pdxcurl#11 | DEFERRED (needs M3-002) |
| **M3-005 structured error taxonomy**                        | **pdxcurl#12** | **LANDED (v1.2.0 -- src/error_taxonomy.pdx module ErrorTaxonomy: RC_* result_code enum + EXIT_* exit code, distinct pair per failure mode per design-doc §7)** |
| M4-001 HTTP GET happy-path smoke                            | pdxcurl#13 | DEFERRED (needs M2-002) |
| **M4-002 HTTPS happy-path smoke witness**                   | **pdxcurl#14** | **LANDED (v1.1.0 -- honest-blockage witness at tests/m4_002_https_smoke.pdx; real body EDIT at libpdx-net M5 unblock)** |
| **M4-003 failure-matrix smoke witness**                     | **pdxcurl#15** | **LANDED (v1.1.0 -- honest-blockage witness at tests/m4_003_failure_matrix.pdx; three roles r/n/k)** |
| **M4-004 audit-trail assertion witness**                    | **pdxcurl#16** | **LANDED (v1.1.0 -- honest-blockage witness at tests/m4_004_audit_trail.pdx; row-shape contract documented)** |
| M4-005 redirect-chain smoke                                 | pdxcurl#17 | DEFERRED (needs M4-002) |
| M5-001 dual-signed manifest.pdxsig + .pdxdoc                | pdxcurl#18 | DEFERRED (target slid v1.2.0 -> v1.3.0 after Wave V drain) |
| M5-002 mirror push                                          | pdxcurl#19 | DEFERRED (needs M5-001) |
| **v1.1-B semantic-pipe emission wire**                      | **pdxcurl#21** | **LANDED (v1.1.0 -- src/main.pdx marshals + emits HttpRequestRecord@0.1 via sys_semantic_send)** |
| **v1.1-C release closer + tag v1.1.0**                      | **pdxcurl#22** | **LANDED (v1.1.0 -- manifest.pdxproj version=1.1.0, CHANGELOG [1.1.0] stanza, release/manifest.pdxsig.txt source form, release/RELEASE-1.1.0.md note)** |

## Substrate readiness

| Substrate | Status |
|-----------|--------|
| paideia-as >= 0.36.0                    | AVAILABLE (Wave L baseline) |
| sys_semantic_send (SC+ ID 115)          | AVAILABLE (paideia-os R107-M0-001 #2350) |
| libpdx-audit audit_append_leaf          | AVAILABLE (libpdx-audit#31; pdxcurl wire pending M3-002) |
| libpdx-net net_connect / net_resolve    | UNAVAILABLE (blocked on libpdx-net M2/M3/M4) |
| libpdx-net net_tls_wrap                 | UNAVAILABLE (blocked on libpdx-net M5) |
| pdxtrust KIND_TLS_TRUST cap mint        | UNAVAILABLE (blocked on pdxtrust M1) |
| libpdx-url url_parse                    | UNAVAILABLE (blocked on libpdx-url M1) |
| paideia-os smoke-runner fixture-seq     | UNAVAILABLE (paideia-os monorepo work) |
