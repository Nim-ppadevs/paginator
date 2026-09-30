# Tasks — hotfix-orm3ApiCompat (ppadevs/paginator)

Branch: `hotfix-orm3ApiCompat` created from `master` in this repo
(`C:\WWW\ppadevs\paginator`) by the implementing agent (may ride the
`hotfix-doctrineOrmBump` branch if both hotfixes are done together — owner decides).
Conventions: no staging/committing/pushing without the owner's explicit request —
present results and ask per task.

- [x] 1. Remove the dead `setCacheable` call
  - `src/Paginator/Paginator.php`: in `private function cloneQuery(Query $query)`
    (line ~302) remove `$cloneQuery->setCacheable(false);` — `Query::setCacheable()`
    was removed in ORM 3; the helper has no call sites (dead code), keep the rest
  - Verify: PHP 8.5 lint (`C:\PHP85\php.exe -l`); single-hunk diff
  - Observable: no removed-API call remains
  - _Evidence: ORM 3.7.2 `Doctrine\ORM\Query` has no `setCacheable`; reproduced
    against the testing line install at `C:\WWW\ppadevs\_orm3-bump-verify`_

- [ ] 2. Release (shared train with the `doctrineOrmBump` companion)
  - Commit on the hotfix branch (owner approval); the production tag carries
    constraint bump (companion spec) + this fix in one release — proposed next number
    on top of the testing-line head (v26.41.1); owner decides
  - Push to the testing line first (`nimdev/paginator` on Packagist) and re-verify;
    then to the production repo `https://github.com/ppadevs/paginator.git` —
    Packagist reads it (the local clone's origin is a testing fork, not used for
    production)
  - **Do not also tag a 2.x line** (e.g. v2.2.6): the major jump is intentional —
    `^2.x` consumers must never auto-pick the new release
  - Observable: `https://repo.packagist.org/p2/ppadevs/paginator.json` lists the new
    tag with `doctrine/orm ^2.11 || ^3.0`

- [ ] 3. Verification (shared final gate with the companion ddd hotfix)
  - Test project (temp folder; interim against the testing line, final against
    `ppadevs/paginator ^26.0`): `{"require": {"php": ">=8.2", "doctrine/orm": "^3.0",
    "<vendor>/ddd": "^26.0", "<vendor>/paginator": "^26.0"}}` — resolves, installs,
    autoloads (`Paginator`, `PaginatableQueryInterface`)
  - Runtime spot check for this fix: the installed `Paginator.php` contains no
    `setCacheable` reference
  - BC check: a second test project requiring `doctrine/orm ^2.11` +
    `ppadevs/paginator ^2.1` still resolves v2.2.5 (26.x stays invisible to it)
  - Report installed versions back to the owner

## Notes

- Minimal diff; keep the file's existing style.
- Companion specs: ddd runtime fix
  (`C:\WWW\ppadevs\ddd\DOCS\hotfix\orm3ApiCompat\`) and the constraint bump
  (`C:\WWW\ppadevs\paginator\DOCS\hotfix\doctrineOrmBump\`).
- Packagist update may need the owner to trigger it if the webhook does not fire
  within minutes of the push.
- Clean up the throwaway test project folder after the production releases land.
