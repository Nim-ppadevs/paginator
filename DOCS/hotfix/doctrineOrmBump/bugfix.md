# Hotfix — ppadevs/paginator: doctrine/orm constraint bump for ORM 3

> Companion hotfix: `C:\WWW\ppadevs\ddd\DOCS\hotfix\doctrineOrmBump\` — the same
> constraint conflict exists there; both packages must be released for any PHP 8.5 /
> Symfony 7.4 / ORM 3 consumer to install them together.

## Problem

New PHP projects run on **PHP 8.5 + Symfony 7.4 + Doctrine ORM 3** (`doctrine/orm
^3.0`; ORM 3 is the current Doctrine generation — the ORM YAML driver was removed
there). `ppadevs/paginator` pins `doctrine/orm: ^2.11` in its composer constraint. The
intersection of `^2.11` and `^3.0` is empty, so **any consumer requiring this package
alongside ORM 3 fails dependency resolution**.

## Current behavior

Reproduce with a minimal test project (verified 2026-09-30 against the same three
requires):

```powershell
mkdir C:\WWW\ppadevs\_orm3-bump-verify; cd C:\WWW\ppadevs\_orm3-bump-verify
Set-Content composer.json '{"require": {"php": ">=8.2", "doctrine/orm": "^3.0", "ppadevs/ddd": "^2.0", "ppadevs/paginator": "^2.1"}}'
& composer85.bat install --no-interaction
```

Output:

```
Your requirements could not be resolved to an installable set of packages.
  Problem 1
    - Root composer.json requires ppadevs/paginator ^2.1 -> satisfiable by ppadevs/paginator[2.1.0, ..., v2.2.5].
    - ppadevs/paginator[2.1.0, ..., v2.2.5] require doctrine/orm ^2.11 -> found doctrine/orm[2.11.0, ..., 2.20.13] but
it conflicts with your root composer.json require (^3.0).
```

(the companion ddd produces an identical problem block).

Repo facts (checked 2026-09-30):

- Published release on Packagist is `v2.2.5`
  (source `https://github.com/ppadevs/paginator.git`), and it requires
  `doctrine/orm: ^2.11`.
- Local clone `C:\WWW\ppadevs\paginator` is in sync with its origin; its origin is a
  **testing fork, not used for production** — production publishes via the ppadevs org
  repo that Packagist reads.

Code compatibility is **not** the problem — only the constraint:

- The paginator uses `Doctrine\ORM\QueryBuilder` / `Query` /
  `NonUniqueResultException` APIs that are unchanged in ORM 3 (getParameters/
  setParameter/addOrderBy/select/getSingleScalarResult/setFirstResult/setMaxResults).
- All 24 `*.php` files in `src/` lint clean on PHP 8.5 (`C:\PHP85\php.exe -l` →
  0 failures, reference run 2026-09-30).

## Expected behavior

1. `composer.json` requires `"doctrine/orm": "^2.11 || ^3.0"` (everything else
   unchanged, including the 2-space indent style).
2. Tag `v26.41.0` is pushed to the production repo
   (`https://github.com/ppadevs/paginator.git`) so Packagist serves it:
   `https://repo.packagist.org/p2/ppadevs/paginator.json` lists `v26.41.0` with the
   relaxed constraint (webhook or owner-triggered update).
3. A consumer project requiring `doctrine/orm ^3.0` + `ppadevs/paginator ^26.0`
   resolves and installs; BC: existing `doctrine/orm ^2.11` + `ppadevs/paginator ^2.x`
   consumers still resolve — and by the major jump they **cannot** pick up 26.x, which
   is intentional (legacy stacks stay on 2.x).

## Reproduction steps

1. Create the minimal test project shown under *Current behavior* (test project in a
   temp folder — `php >= 8.2` + the three requires; no composer.lock).
2. Run `& composer85.bat install --no-interaction`.
3. Observe the unsolvable resolution error quoted above.

## Fix constraints

- Constraint-only change; no code edits, no BC break (`^2.11` installs still resolve).
- Do the change on the hotfix branch, not directly on master.
- **Do not also tag a 2.x line** (e.g. v2.2.6): the major jump to 26.x is intentional —
  `^2.x` consumers (legacy stacks) must never auto-pick the new release.
- No commits/pushes without the owner's explicit request; pushing to GitHub uses the
  owner's credentials/permissions.

## Scope

- **In**: constraint relax + release `v26.41.0` + publish; verification with a throwaway
  ORM 3 test project (see tasks.md — shared final gate with the companion ddd hotfix).
- **Out**: package code changes; Packagist account automation (the owner
  triggers/verifies packagist updates); any specific consumer projects (out of scope
  here).
