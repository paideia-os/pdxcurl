# pdxcurl -- mirror-push runbook (BLOCKED ON R32)

**Repo:** github.com/paideia-os/pdxcurl
**Wave:** R100 user-space networking tools
**Milestone:** M5-002 (`pdxcurl#19`)
**Upstream design:** `design/networking/r100-user-tools-plan.md` §13.3
(the `pdxcurl.M5-002 mirror push` line) + §2.6; mirror tree shape from
`design/tooling/plan.md` §6.3, in the paideia-os monorepo.
**Status at this landing (v1.4.1, 2026-09-13):** documentation only.
**This runbook has never been executed and cannot be** -- see §6.

---

## 0. What this document pins

The URLs, the file tree, and the ordered steps that move a pdxcurl
release from a tagged commit on `main` into the `pkgs.paideia-os`
package mirror. The mirror does not exist at HEAD and the signing
substrate it depends on has not landed, so this document is the
**input contract** the eventual push is built against -- written now,
while the release shape is fresh, so the first real push is not a
schema negotiation against a moving target.

Nothing in this document is code. `pdxcurl#19` ships this file and
nothing else; no `src/`, `tests/`, or `tools/` file changes at
v1.4.1.

**Read §6 before running any command below.** Every step from §3.4
onward is gated on R32 and will fail closed today.

---

## 1. Mirror URL convention

### 1.1 There is exactly one mirror, and it is not a git remote

pdxcurl has a single git remote:

    origin    https://github.com/paideia-os/pdxcurl

**This wave defines no second git remote.** "Mirror push" in the R100
milestone vocabulary (`r100-user-tools-plan.md` §2.6, §13.3) means
publishing a signed *package* to `pkgs.paideia-os`, not mirroring the
git history to a second forge. Commits and tags go to `origin` and
only to `origin`. A future maintainer reading the M5-002 issue title
should not add a `mirror` git remote on the strength of the word
"mirror" -- if a source-forge mirror is ever wanted, it is a separate
design question with its own doc.

### 1.2 Artifact mirror URL pattern

The mirror is a static HTTPS-served tree (`design/tooling/plan.md`
§6.3). Every path below is canonical for pdxcurl; `$VERSION` is a
semver triple that MUST equal `manifest.pdxproj` `version`, the
`package-version` line in `release/manifest.pdxsig.txt`, the
`## [$VERSION]` CHANGELOG stanza, and the `v$VERSION` git tag --
all five or none.

| Role | URL |
|------|-----|
| Mirror root | `https://pkgs.paideia-os/` |
| Signed index | `https://pkgs.paideia-os/main/index.pdxsig` |
| Upload target (author-signed) | `https://pkgs.paideia-os/staging/pdxcurl/$VERSION/` |
| Published target (dual-signed) | `https://pkgs.paideia-os/main/pdxcurl/$VERSION/` |

After a successful push the published directory holds exactly three
files:

```
pkgs.paideia-os/
  main/
    index.pdxsig                       <- re-signed: gains one pdxcurl/$VERSION row
    pdxcurl/
      $VERSION/
        pkg.tar                        <- the pdxcurl package archive
        manifest.pdxsig                <- dual-signed; byte-identical to the copy inside pkg.tar
        mirror.entry                   <- admission record (§2)
```

The manifest is duplicated deliberately -- inside the tar and
alongside it -- so a local `pkg verify /pkgs/pdxcurl-$VERSION/` can
re-check both signatures with no network round trip.

### 1.3 `pkg.tar` layout

Canonical, lexicographically sorted so the archive hash is
reproducible across build hosts:

```
pkg.tar contents:
  CHANGELOG.md
  LICENSE
  caps.decl
  doc/pdxcurl.pdxdoc
  manifest.pdxsig
  src/argv_surface.pdx
  src/audit_emit.pdx
  src/error_taxonomy.pdx
  src/main.pdx
  src/tls_wire.pdx
  src/tool_ident.pdx
  src/url_parse.pdx
```

pdxcurl ships **source-first**, matching `libpdx-argv`'s precedent:
`pkg install` lowers `src/*.pdx` on the target, where the paideia-as
toolchain lives. `tests/` is NOT shipped -- it stays in-repo. The
compiled `build-out/pdxcurl.elf` is NOT shipped; it is a build
product of the target, not a mirror artifact.

`doc/pdxcurl.pdxdoc` does not exist in this repo yet (`pdxcurl#18`
landed the manifest half of M5-001 unsigned; the `.pdxdoc` half is
still outstanding). It is listed here because the tar layout is the
contract; §3.1's checklist catches its absence.

### 1.4 Hash discipline

**BLAKE3-256** (upper 32 bytes, 64 lowercase hex chars), matching
`release/manifest.pdxsig.txt`'s existing `<BLAKE3-*>` slots and
`rm`'s `.release/mirror.pdxmeta` `tar_hash_algo`.

Known divergence, recorded rather than silently resolved: `shell`'s
`design/mirror-push.md` specifies **sha3-256** for its index rows and
`libpdx-argv`'s `pkgs/mirror.entry` specifies **sha256**. Three
sibling repos naming three algorithms for the same index is a real
ecosystem inconsistency that the `pkgs.paideia-os` scaffolding round
must reconcile before any of these runbooks fires. pdxcurl commits to
BLAKE3-256 because that is what its own signed manifest already
commits to; if the mirror standardises elsewhere, this section and
`release/manifest.pdxsig.txt` change together, in one commit.

---

## 2. `mirror.entry` and the index row

`mirror.entry` is the admission record the mirror runner reads.
Format follows the `index.pdxsig`-inner-record shape
(`design/tooling/plan.md` §6.3):

```
name:              pdxcurl
version:           $VERSION
wave:              R100
kind:              cli-tool
license:           MIT
source_repo:       https://github.com/paideia-os/pdxcurl
source_tag:        v$VERSION
source_commit:     <git rev-parse HEAD at tag time>
source_tree_root:  <BLAKE3-256 of the canonical source tree>
pkg_tar_blake3:    PENDING:release-runner
manifest_blake3:   PENDING:release-runner
mirror_path:       main/pdxcurl/$VERSION/
manifest_path:     main/pdxcurl/$VERSION/manifest.pdxsig
pkg_tar_path:      main/pdxcurl/$VERSION/pkg.tar
declared_output_schemas:
  - HttpRequestRecord@0.1
deps:              libpdx-net, libpdx-url, libpdx-audit, pdxtrust
toolchain_floor:   paideia-as >= 0.33-crypto-kdf
```

The two `PENDING:release-runner` sentinels are filled by the mirror
runner at admission time -- they carry the runner's **post-verify**
hashes, not the pre-sign hashes. A pushed `mirror.entry` that already
has them filled is a red flag: it means someone computed them locally
and the runner has nothing independent to check.

Adding a row re-signs the **whole** `index.pdxsig` under
`paideia_root_pk`. There is no partial-update path, by design: one
signature covers the entire index, so a mirror operator who adds a
package without the root key cannot inject something the index does
not attest to.

Every `deps:` entry above is a repo that does not exist yet or has
not shipped to the mirror. pdxcurl pushes **after** all four; see
§3.1 item 5.

---

## 3. The push workflow

Set `$VERSION` once per session and never hardcode it. `libpdx-elevate`'s
runbook was written against a hardcoded `1.0.0`, drifted three minor
versions behind `main`, and would have re-pushed the wrong tree the
day it fired. Do not repeat that.

    VERSION=1.4.1          # or whatever main HEAD's version is

### 3.1 Pre-flight checklist

Confirm all of these before touching anything. Each has drifted at
least once somewhere in this ecosystem:

1. **`manifest.pdxproj`** `version = $VERSION`.
2. **`release/manifest.pdxsig.txt`** `package-version` = `$VERSION`,
   and its `[artifacts.source]` / `[artifacts.tests]` sets list every
   file the tree actually has -- no stale entries, no missing ones.
3. **`CHANGELOG.md`** has a real dated `## [$VERSION]` stanza.
4. **`STATUS.md`** top-of-file `**Version:**` line names `$VERSION`.
5. **`release/RELEASE-$VERSION.md`** exists and its §1 matches the
   CHANGELOG stanza.
6. **`doc/pdxcurl.pdxdoc`** exists (§1.3). It does not today.
7. **Dependencies present at `main/`.** `libpdx-url`, `libpdx-net`,
   `libpdx-audit`, and `pdxtrust` must each already be published:

       pkg list --repo=pkgs.paideia-os/main libpdx-url libpdx-net \
                                            libpdx-audit pdxtrust

   pdxcurl is a leaf consumer of all four and pushes last.
8. **`v$VERSION` does not already exist as a real signed tag.** The
   existing `v1.1.0`..`v1.4.0` tags are annotated, GPG-**un**signed
   landing markers -- they record coherent points on `main`, not
   executed releases. Reusing one of those names for a real signed
   release means deleting the marker
   (`git push origin :refs/tags/vX && git tag -d vX`) and recreating
   it signed. Never leave two different objects claiming one tag name
   on the remote, and never silently drop `-s` because a same-named
   unsigned tag exists.

If any of 1--6 is stale, fix it in a separate prose commit first.
This runbook signs whatever the working tree says; it does not
update prose.

### 3.2 Freeze the source tree

    git checkout main && git pull
    git log --oneline -1        # note HEAD; this is source-commit

The `source-commit` written into `manifest.pdxsig` and
`mirror.entry` MUST equal `git rev-parse HEAD` here. If HEAD moves,
re-run the witnesses and re-hash -- a changed source-tree hash
changes the manifest body and requires a fresh signing pass.

### 3.3 Build the artifact

    bash tools/build.sh

Confirms the tree assembles. The ELF is **not** a mirror artifact
(§1.3); this step is a gate, not a product.

### 3.4 Populate the manifest hashes -- *R32-gated*

Replace every `<BLAKE3-*>` placeholder in
`release/manifest.pdxsig.txt` with the real digest, and fill
`source-commit` / `source-tag` from §3.2. Commit the hash-populated
manifest on a `release/v$VERSION` branch. This is the body both
signatures will cover.

### 3.5 Author-side sign -- *R32-gated*

    paideia-as release --sign \
        --manifest manifest.pdxsig \
        --key author_pk \
        --output manifest.pdxsig

Fills `[signature.author]`. `[signature.paideia-root]` is left for
the runner.

### 3.6 Pack -- *R32-gated*

    paideia-as release --pack \
        --manifest manifest.pdxsig \
        --output pdxcurl-$VERSION.pkg.tar

Deterministic ordering per §1.3.

### 3.7 Tag and push to origin

    git tag -a -s v$VERSION -m "pdxcurl $VERSION -- <one line from CHANGELOG>"
    git push origin main
    git push origin v$VERSION

`-s` is a git/OpenPGP attestation only. The ML-DSA-65 signature that
gates `pkg install` lives in `manifest.pdxsig`, not in the git tag --
a valid git tag proves nothing about the package.

**Origin push precedes the mirror push**, always. The mirror's
`source_commit` must be publicly resolvable at the moment the runner
verifies it; pushing the package first creates a window where the
mirror attests to a commit nobody outside the release machine can
fetch.

### 3.8 Push to staging -- *R32-gated*

    pkg push --repo=pkgs.paideia-os/staging \
             --pkg pdxcurl-$VERSION.pkg.tar

Lands under `staging/pdxcurl/$VERSION/`. Upload order within the
directory is `pkg.tar`, then `manifest.pdxsig`, then `mirror.entry`
-- the entry last, because it is what the runner polls for.

### 3.9 Runner promotion (not a maintainer step)

The signing runner, on picking up staging:

  a. Verifies the author signature against the `author_pk`
     fingerprint registered for the paideia-os org.
  b. Unpacks and recomputes the source-tree hash independently.
  c. Countersigns the same canonical body under `paideia_root_pk`,
     filling `[signature.paideia-root]`.
  d. Fills the two `PENDING:release-runner` sentinels in
     `mirror.entry`.
  e. Copies the tuple to `main/pdxcurl/$VERSION/`.
  f. Appends the row to `main/index.pdxsig` and re-signs the whole
     index under `paideia_root_pk`.

`index.pdxsig` is overwritten **last** and atomically (rename over
the old file). A client fetching mid-promotion sees either the old
index or the new one, never an index pointing at bytes that are not
there yet.

### 3.10 Verify from a clean machine

On a host holding neither `author_pk` nor `paideia_root_pk`:

    pkg install pdxcurl
    #   resolved:  pdxcurl-$VERSION (pkgs.paideia-os/main)
    #   verified:  author=paideia-os-team (ML-DSA-65)
    #   verified:  paideia-manifest (ML-DSA-65 root)
    #   cap-audit: KIND_USER, KIND_TLS_TRUST (optional), KIND_IPC_ENDPOINT
    #   installed: /pkgs/pdxcurl-$VERSION

Then exercise the tool itself -- an install that verifies but does
not run is a half-verified release:

    /pkgs/pdxcurl-$VERSION/bin/pdxcurl --audit-only http://10.0.2.2/
    echo $?          # expect 200 (EXIT_SUCCESS_AUDIT_ONLY)

A verification failure at this step means half-pushed state. It is
detectable from any client: the tar hash in `index.pdxsig` will not
match the tar bytes on the mirror.

### 3.11 Close out

Flip the M5-002 row in `STATUS.md` to LANDED, note the push in the
CHANGELOG stanza, and commit to `main` referencing this runbook.
Only after §3.10 passes.

---

## 4. Rollback

The mirror is **append-only for signed releases**. There is no
retraction.

1. To pull a bad archive: remove the path and rebuild
   `index.pdxsig` under `paideia_root_pk` without the row.
2. Publish a superseding patch release whose CHANGELOG names the
   withdrawn version and the reason.
3. **Never reuse a version number for different bytes.** Semver is
   immutable at the mirror.

Machines that already installed the withdrawn version keep working --
the `manifest.pdxsig` they hold still verifies against the root key
they already trust. Removal takes away *availability*, not validity.
`pkg upgrade` moves them to the patch release.

---

## 5. Escalations

- **Author key compromise.** Rotate `author_pk`, re-sign every extant
  pdxcurl release under the new key, re-push in dependency order
  (`libpdx-url` -> `libpdx-net` -> `libpdx-audit` -> `pdxtrust` ->
  `pdxcurl`). Coordinate with the root re-sign gate to invalidate
  old-key manifests at `main/`.
- **Paideia root compromise.** A `design/user/model.md` §8 event.
  Halt all mirror-push activity and follow the R32 root-rotation
  playbook. Out of scope for this file.

---

## 6. BLOCKED ON R32

**Every step from §3.4 onward is unexecutable today.** This is the
whole reason `pdxcurl#19` ships documentation and no code.

`pdxcurl#19` is blocked on **R32**, the round that lands the
post-quantum signing substrate and `paideia_root_pk` (the R32 root
key) that this entire workflow signs and verifies against. This
document does not scope R32, does not restate its contents, and does
not predict its landing date -- it records R32 only as the open
prerequisite. R32's own design owns everything else about it.

What the blocker concretely withholds from this runbook:

| Blocked step | What R32 must first provide |
|---|---|
| §3.4 hash fill | The canonical hashing + manifest-signing tool path |
| §3.5 author sign | `paideia-as release --sign` with a real ML-DSA-65 sign |
| §3.6 pack | `paideia-as release --pack` |
| §3.7 `-s` tag | (git-side only; unblocked, but pointless alone) |
| §3.8 push | `pkg push` against a mirror that admits signed packages |
| §3.9 promotion | `paideia_root_pk` and the runner that holds it |
| §3.10 verify | Two verifiable signatures to check |

Secondary blockers, named so a future reader does not mistake R32 for
the only one -- clearing R32 alone does not make this runbook fire:

- **The `pkgs.paideia-os` mirror does not exist.** It is scaffolded
  in a separate paideia-os infrastructure round. No host, no runner,
  no staging area.
- **`release/manifest.pdxsig.txt` is unsigned source form.** Per its
  own header and `pdxcurl#18`, every `<BLAKE3-*>` slot is a
  placeholder and there is no `[signatures]` block at all. M5-001 is
  LANDED-UNSIGNED, not LANDED.
- **`doc/pdxcurl.pdxdoc` is not written** (§1.3, §3.1 item 6).
- **Four dependency repos have not shipped** (§3.1 item 7).

Until R32 lands, the correct state of this milestone is: this file
exists, it is accurate, and `STATUS.md` says `M5-002 mirror push --
DOCUMENTED, BLOCKED ON R32`. Do not mark it LANDED. Do not
approximate the signing steps with a local stub to "make progress" --
an unsigned artifact in a directory named `main/` is worse than no
artifact, because `pkg` clients treat that path as attested.
