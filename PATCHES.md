# Fork notes

This is a fork of [graydwarf/godot-gdscript-linter](https://github.com/graydwarf/godot-gdscript-linter)
(MIT, by Poplava). It exists so our projects can consume a fixed and, later, extended linter without
waiting on upstream.

**Base:** upstream `v3.3.0` (`e45b640`).
**Divergence:** two commits, both in `addons/gdscript-linter/analyzer/checkers/function-checker.gd`.
Nothing else differs from upstream.

## Our commits

| Commit | Fix |
|---|---|
| `9b79783` | `static func` is now a function boundary |
| `2e02e26` | a function body ends at class-scope code, not just the next `func` |

Both are **generic bug fixes with no project-specific content**, deliberately kept as separate,
self-contained commits so either can be cherry-picked into an upstream PR later. Keep it that way:
put project-specific rules in their own commits, never mixed into these two.

### Why they matter

Upstream measured a function body as "everything until the next `func `". Two consequences:

1. **Static functions were never analyzed** — and each one was absorbed into the preceding
   non-static function. A 3-line `_process` followed by static helpers reported as **236 lines,
   complexity 24**. Codebases that keep pure logic in static functions were effectively invisible
   to the linter while appearing clean.
2. **Trailing class-scope code counted as body** — a 3-line function followed by a `const` table
   reported as **166 lines**.

Both produce *confidently wrong* numbers rather than obvious failures, which is what makes them
worth carrying a fork for: findings get filed against functions that are fine, and real problems go
unreported.

## Upgrading from upstream

```bash
git fetch upstream
git rebase upstream/main        # our two commits replay on top
```

If a rebase conflicts, the conflict is almost certainly in `analyze_functions` — re-read the two
commits above rather than resolving mechanically, since both fixes are about *where a body ends*.

After rebasing, re-vendor into the consuming projects and re-baseline:

```bash
cd ~/code/par52 && las lint-upgrade && las lint
```

Expect the issue counts to move; that is the point. Compare against the previous baseline before
assuming a regression.

## Consuming projects

Godot needs the addon present in the project tree, so each project vendors a copy of
`addons/gdscript-linter/` rather than using a submodule. `las lint-upgrade` (in par52) copies from
this repo and records the source commit in `addons/gdscript-linter/FORK_VERSION`.

- `~/code/par52` — tuned via `gdlint.json`; see its `AGENTS.md` → "Linting / code quality"
- `~/code/gizzard`

## Publishing

This repo has no `origin` yet — only `upstream`. To publish:

```bash
gh repo create godot-gdscript-linter --public --source=. --remote=origin
git push -u origin main
```

Upstream attribution and the MIT `LICENSE` are unchanged and must stay that way.
