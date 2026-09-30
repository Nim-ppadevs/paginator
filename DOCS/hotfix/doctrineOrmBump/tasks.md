# Tasks — hotfix-doctrineOrmBump (ppadevs/paginator)

Branch: `hotfix-doctrineOrmBump` created from `master` in this repo
(`C:\WWW\ppadevs\paginator`) by the implementing agent.
Conventions: no staging/committing/pushing without the owner's explicit request —
present results and ask per task.

- [ ] 1. Relax the doctrine/orm constraint
  - `composer.json`: `"doctrine/orm": "^2.11"` → `"doctrine/orm": "^2.11 || ^3.0"`
    (keep the 2-space indent style; single-line diff)
  - Verify: `composer validate` (composer85.bat validate); PHP 8.5 lint pass over
    `src/` (`C:\PHP85\php.exe -l` — reference run 2026-09-30: 24 files, 0 failures)
  - Observable: composer.json parses, validator passes, no other diff hunks

- [ ] 2. Release `v26.41.0`
  - Commit on `hotfix-doctrineOrmBump` (owner approval), merge to `master`, tag
    `v26.41.0`, push `master` + tag to the production repo
    `https://github.com/ppadevs/paginator.git` — Packagist reads it (the local
    clone's origin is a testing fork, not used for production)
  - **Do not also tag a 2.x line** (e.g. v2.2.6): the major jump is intentional —
    `^2.x` consumers must never auto-pick the new release
  - Observable: `https://repo.packagist.org/p2/ppadevs/paginator.json` lists
    `v26.41.0` with `doctrine/orm ^2.11 || ^3.0`
- [ ] 3. Verification with a fresh ORM 3 consumer project (shared final gate — only
      after BOTH this release and the companion ddd hotfix `v26.41.0` are
      published)
  - Create a throwaway test project (temp folder):
    `{"require": {"php": ">=8.2", "doctrine/orm": "^3.0", "ppadevs/ddd": "^26.0",
    "ppadevs/paginator": "^26.0"}}` and run
    `& composer85.bat install --no-interaction`
  - Observable: resolution succeeds — `ppadevs/ddd v26.41.0` +
    `ppadevs/paginator v26.41.0` install alongside `doctrine/orm ^3.0`; and with the
    vendor autoload loaded, `class_exists('Paginator\Paginator')` and
    `class_exists('ppadevs\Ddd\Domain\Exception\DomainException')` are true (quick
    `php -r` sanity)
  - BC check: a second test project requiring `doctrine/orm ^2.11` +
    `ppadevs/paginator ^2.1` still resolves v2.2.5 (constraint-only change, no break;
    26.x stays invisible to it)
  - Report installed versions back to the owner

## Notes

- Constraint-only change; no BC break — `^2.11` consumers still resolve.
- Companion spec (same fix, ddd repo):
  `C:\WWW\ppadevs\ddd\DOCS\hotfix\doctrineOrmBump\`.
- Packagist update may need the owner to trigger it if the webhook does not fire
  within minutes of the push.
- Clean up the throwaway test project folders afterwards.
