# ttf-atkinson-hyperlegible-nerd — Claude Guide

Arch Linux packaging (a `PKGBUILD`) that builds a **Nerd Fonts**-patched build of the
**Atkinson Hyperlegible Next** font. `makepkg` clones the upstream Google Fonts source,
runs each TTF through the Nerd Fonts `font-patcher` (FontForge) to inject the Nerd Fonts
icon glyphs, and installs the patched TTFs into `/usr/share/fonts/TTF`.

This is a tiny repo: essentially one `PKGBUILD` plus a generated `.SRCINFO`. There is no
application code, no test suite, and no build system beyond `makepkg`.

## ⚡ superpowers — use whenever applicable

Always prefer **superpowers** skills over ad-hoc approaches. If there's even a small chance a skill applies to the task, invoke it via the `Skill` tool before acting (including before clarifying questions).

- **Process skills first** — `brainstorming` before creative/feature work, `systematic-debugging` before fixing bugs, `test-driven-development` before writing implementation.
- **Then implementation skills** — domain-specific skills guide execution.
- **Verify before claiming done** — `verification-before-completion` / `requesting-code-review` before merging.

User instructions always take precedence over skills; skills override default behavior.

### Mode switch

- **"lite mode"** — fully disables superpowers: no skill is invoked, not even the applicability check, until **"normal mode"** is said.
- **"normal mode"** (default) — standard superpowers behavior, plus: when delegating coding work, dispatch at most 1 agent at a time, and never use a model above Sonnet (no Opus).
- **"modo desatendido"** (unattended mode) — the user is away and delegates autonomy: work without waiting for confirmations and make reasonable decisions yourself instead of asking. In this mode you MAY **`git push` the feature branches you create** and **open PRs via `gh`** on your own, so the work is ready for review when the user returns. The hard limits still hold and are NOT lifted: **never merge anything** (no `git merge`, no fast-forward integration, no `gh pr merge`), **never push to `main`** or any protected/default branch directly, and **never** `git push --force` / `--force-with-lease`. Deliver everything as pushed branches + PRs for the user to merge. Reverts to defaults on **"normal mode"**.

Confirm the switch briefly when it happens.

## Stack

- **Arch Linux packaging** — a single `PKGBUILD` (`-git` VCS package) built and installed with `makepkg`. Metadata mirror in `.SRCINFO`.
- **Nerd Fonts `font-patcher`** — the `font-patcher` package (`/usr/share/font-patcher/font-patcher`), run under **FontForge** (`fontforge -script`), patches each source TTF with Nerd Fonts glyphs. Declared in `makedepends`.
- **Upstream source** — `git+https://github.com/googlefonts/atkinson-hyperlegible-next.git` (Atkinson Hyperlegible Next, OFL-1.1). TTFs live in `fonts/ttf/` upstream.
- **Patch flags** — `--complete --careful --makegroups 5 --metrics TYPO`: `--complete` adds all Nerd Fonts glyph sets, `--careful` won't overwrite existing glyphs, `--makegroups 5` names the output family, `--metrics TYPO` uses the OS/2 typo metrics. Patching is fanned out across cores with `xargs -P $(nproc)`.

## Commands

```bash
# Build + install the package locally (clones upstream, patches every TTF, installs)
makepkg -si

# Build only, without installing
makepkg

# Force a clean rebuild (re-clone source, wipe previous build dirs)
makepkg -sCf

# Refresh source checksums in the PKGBUILD after changing sources
updpkgsums

# Regenerate .SRCINFO after ANY PKGBUILD edit (keep it in sync!)
makepkg --printsrcinfo > .SRCINFO

# Lint the PKGBUILD and/or the built package for common mistakes
namcap PKGBUILD
namcap ttf-*-nerd-git-*.pkg.tar.zst
```

Note: `pkgver()` derives the version from the upstream git history
(`r<commit-count>.<short-sha>`), so `pkgver` updates automatically on each build — never
hand-edit it.

## Quality — font & packaging checks

There are no unit tests here. "Quality" means the package builds reproducibly, the patched
fonts are valid, and the AUR metadata is consistent. Before considering a change done, verify:

- **Patched fonts contain the expected glyphs/ranges** — after a build, inspect an output
  TTF and confirm the Nerd Fonts private-use ranges are present, e.g.
  `fc-scan output/*.ttf` for family/style, and check codepoints with
  `ttx -t cmap -o - output/SomeFont.ttf | grep -i 'code=0xe'` (Nerd Fonts icons live in the
  PUA, roughly `U+E000`+, `U+F000`+, plus higher planes). The whole point of the package is
  that terminal/editor icons render — an empty or partial patch is the main failure mode.
- **TTFs are structurally valid** — run `fontlint output/*.ttf` (part of FontForge) and/or a
  `ttx` round-trip (`ttx -q output/Font.ttf` produces a `.ttx` without errors). A build that
  finishes but yields a corrupt font is worse than a failed build.
- **The package installs cleanly and the font is discoverable** — `makepkg -si`, then
  `fc-list | grep -i atkinson` and `fc-cache -f`; the patched family should appear.
  A real smoke test is rendering: open a terminal/editor configured with the family and
  confirm Nerd Font icons (e.g. a Powerline segment or a git branch glyph) show, not tofu.
- **PKGBUILD ↔ .SRCINFO in sync** — `.SRCINFO` is generated, never hand-edited. After any
  `PKGBUILD` change run `makepkg --printsrcinfo > .SRCINFO` and commit both together;
  `pkgname`, `pkgdesc`, `source`, `makedepends`, and checksums must match. (Heads-up: the
  metadata file in this repo is currently named `.SCRINFO` and its contents drift from the
  `PKGBUILD` — fix the filename to `.SRCINFO` and regenerate it as part of any cleanup.)
- **Checksums current** — for the `-git` source `sha256sums=('SKIP')` is correct (VCS
  sources aren't checksummed). If a non-VCS source is ever added, run `updpkgsums` so the
  sums match; a stale checksum fails the build.
- **`namcap` is clean** — run `namcap` on both the `PKGBUILD` and the built package and
  resolve warnings (missing deps, wrong licenses, bad file placement) rather than ignoring
  them.

**Process rule:** decide what "correct" means for the change yourself — which glyph ranges
must survive the patch, which families must install — and verify against that, rather than
trusting that a green `makepkg` run implies a good font.

## Real-system verification — what no green build can prove

The section above lists the right gates. This one names the **kinds** of check they are, so one can
be asked for by name, and states the rules for writing one worth trusting.

The framing that matters for a font package: **`fontlint` clean, `ttx` round-tripping and `fc-list`
finding the family do not mean a single icon renders.** They are structural checks on a file. The
product is glyphs on a screen. The failure mode this package exists to prevent — tofu where a
Powerline arrow or a git-branch glyph should be — is *visual*, and no structural check reports it.

What "real system" means here, concretely:

- **A real display, at a real size.** Install, `fc-cache -f`, then open a terminal and an editor
  configured with the patched family and look. Rasterisation depends on the FreeType version,
  fontconfig rules, subpixel settings and DPI — a font that reads well on a 4K panel can be muddy at
  1080p, and this is a *legibility* typeface, so that is the whole point.
- **A clean chroot for the build**, not your dev box: `extra-x86_64-build` or `makechrootpkg`. A
  build that works only because `font-patcher` or FontForge happens to be installed globally is a
  missing `makedepends` that fails for everyone else — silently, for you.
- **A throwaway system for the install.** `makepkg -si` installs fonts on your daily machine and
  rewrites your fontconfig cache; a bad build there is a real problem, not a failed test.

### The names, so you can ask for them by name

| Name | What it means here |
| --- | --- |
| **E2E / on-system acceptance test** | Build in a clean chroot, install on a throwaway system, and assert on observable results — `pacman -Ql` lists the TTFs under `/usr/share/fonts/TTF` and the license under `/usr/share/licenses/`, `fc-list` reports the family, and **the icons actually render in a terminal**. Never on the build log. |
| **Contract test** | Checks that assumptions about **things you do not control** still hold, which for a `-git` package is most of them. The `source` floats to upstream's default branch, so the input fonts change under you between builds. `font-patcher` is a `makedepends`, so its flag semantics move with the Nerd Fonts release — `--complete`, `--careful`, `--makegroups 5` and `--metrics TYPO` have each changed meaning or naming output across versions. FontForge's own version changes what comes out. Record which upstream commit and which patcher version produced a build, or a regression is unattributable. |
| **Mutation testing** (here: by hand) | Drop a patcher flag or a `makedepends`, rebuild in the chroot, confirm the check you rely on goes red, restore. **A check that has never failed has not been tested** — a glyph-coverage check that has never seen an unpatched font proves nothing. |
| **State-invariant test** | Asserts a relationship **between two things** neither one alone can prove: `PKGBUILD` against the generated metadata file; the family name baked into the TTF against what `fc-list` reports against what a user writes in their terminal config; the patched output against the upstream input it came from. Each side can be individually valid while the pair is wrong. |
| **Test pollution / isolation leak** | State that outlives a run and changes the next one. The sharp one here is the **fontconfig cache**: an older installed version can keep rendering after a bad rebuild, so the icons look fine and the package is broken — always `fc-cache -f` and re-check, and prefer verifying on a machine that never had a previous build installed. Building outside a chroot is the same class of problem. |

### Rules that came out of real bugs, not theory

- **Prove every check can fail before you trust it green.** Patch a font with `--complete` removed,
  confirm your coverage check reports the missing ranges, restore. An unexercised check is decoration.
- **Never assert on a count you cannot predict.** This is the specific trap for this package: the
  number of Private Use Area codepoints in a patched font tracks the **Nerd Fonts release**, not your
  build — so `ttx -t cmap … | grep -c 'code=0xe'` will happily report a healthy-looking number for a
  half-patched font, and will "fail" a perfectly good one after an upstream glyph-set change. Assert
  **named codepoints you actually use** instead: `U+E0B0` (Powerline arrow), `U+F09B` (git branch),
  and whichever glyphs your prompt and editor really draw. Same for file size and glyph totals.
- **A checksum, generated metadata file or `pkgrel` must die with the source it describes.** VCS
  sources legitimately use `SKIP`, so the *only* thing tying a build to its input is what you record
  about it — stale metadata beside a rebuilt font produces a package that installs happily and is
  wrong, with nothing failing.
- **Never test destructively on your own machine.** Chroot to build, container or VM to install. If
  you do install locally, know `pacman -Rns` and `fc-cache -f` before you start.
- **Run the build the way that actually works here.** Patching fans out with `xargs -P $(nproc)` and
  FontForge is memory-hungry, so on a many-core box cap the fan-out (or run it under a memory ceiling)
  rather than discovering the limit by taking the machine down. And **claim exactly what you
  verified**: "`makepkg` succeeded" is not "the icons render".

## Working rules

- **Use superpowers skills whenever they apply** — invoke via `Skill` before acting; process skills before implementation skills.
- **Don't add packages / build deps without asking** — the toolchain (`font-patcher`, FontForge) is intentional; adding `makedepends` or `depends` changes what users must install.
- **Never hand-edit generated files** — `pkgver` (set by `pkgver()`) and `.SRCINFO` (produced by `makepkg --printsrcinfo`) are derived; regenerate, don't type.
- **Regenerate `.SRCINFO` with every `PKGBUILD` change** and commit them together.
- **Don't change patch flags casually** — `--complete --careful --makegroups --metrics` affect which glyphs land and the resulting font metrics; a change here can silently drop icons or shift line height. Rebuild and re-verify glyphs afterward.
- **Verify by building** — this repo has no tests; the acceptance check is an actual `makepkg -si` plus a glyph/render check (see "Quality" above).

## Git & GitHub

- **Commits and branches OK** — create commits and new branches whenever it makes sense, without asking first.
- **Never push** *(default)* — no `git push` under any circumstance, and absolutely never `git push --force` / `--force-with-lease`. Leave pushing to the user. **Exception:** when **"modo desatendido"** is active, you may push the feature branches you create (never `main`/protected branches, never force) so PRs are ready for review.
- **Never merge — no permission** — you do NOT have permission to merge anything into any branch, nor to merge any pull request. No `git merge`, no fast-forward integration, no `gh pr merge`. Leave every merge (branches and PRs alike) to the user. This holds in every mode, **including "modo desatendido"**.
- **GitHub via `gh`** — if the `gh` CLI is available, you may open pull requests, issues, and similar (comments, labels, etc.). These don't require pushing on your part beyond what `gh` itself does for an already-pushed branch.
- **Every PR must include a manual test plan** — when opening a PR, add a **How to test manually** section describing the exact steps to exercise the change by hand. Here that means building and installing the package and verifying the font renders with icons: e.g. `makepkg -sCf` (clean build) → `makepkg -si` (install) → `fc-cache -f && fc-list | grep -i atkinson` (font is registered) → open a terminal/editor set to the patched family and confirm Nerd Font icons render instead of tofu. Include any setup (which upstream commit, extra sources) and edge cases (missing `font-patcher`, a font whose patch drops glyphs).
