# Hotfix — ppadevs/paginator: remove removed-API setCacheable() call (ORM 3)

> Companion hotfixes: `C:\WWW\ppadevs\ddd\DOCS\hotfix\orm3ApiCompat\` (same runtime
> concern) and `C:\WWW\ppadevs\paginator\DOCS\hotfix\doctrineOrmBump\` (the
> constraint bump — ships on the same release train).

## Problem

Doctrine ORM 3 **removed** `Doctrine\ORM\Query::setCacheable()` (verified against
ORM 3.7.2 — no such method in `src/Query.php`). `ppadevs\Paginator\Paginator` has a
`private function cloneQuery(Query $query)` helper (line ~302 in the testing-line
install) that calls `$cloneQuery->setCacheable(false);`. The helper is **dead code**
(no call sites in the package), so nothing fatals today — the cleanup makes the
package truly ORM-3-safe and prevents a future fatal if the helper is ever used.

## Current behavior

Verified 2026-09-30 against the testing line install
(`C:\WWW\ppadevs\_orm3-bump-verify` — kept as evidence):

```powershell
& 'C:\PHP85\php.exe' -r "require 'C:\WWW\ppadevs\_orm3-bump-verify\vendor\autoload.php'; var_dump(method_exists('Doctrine\ORM\Query', 'setCacheable'), class_exists('Paginator\Paginator'));"
```

→ `bool(false)`, `bool(true)`.

- `vendor/nimdev/paginator/src/Paginator/Paginator.php:302` (testing line v26.41.1)
  still calls the removed method inside `cloneQuery()`.
- Install itself works fine: `nimdev/paginator v26.41.1` + `doctrine/orm 3.7.2`
  resolve and autoload (constraint already relaxed by the companion
  `doctrineOrmBump` hotfix).

Everything else the paginator uses is unchanged in ORM 3: `QueryBuilder`
(getParameters/setParameter/addOrderBy/select/getQuery/getSingleScalarResult/
getFirstResult/setMaxResults), `Query\Parameter` (getName/getValue/getType),
`NonUniqueResultException`.

## Expected behavior

`Paginator.php` no longer references `setCacheable()` — remove the line inside
`cloneQuery()` (minimal diff; removing the whole dead helper is the owner's
alternative). Pagination behavior is unchanged.

## Reproduction steps

1. Install the testing line into a test project (companion `doctrineOrmBump`
   reproduction; evidence at `C:\WWW\ppadevs\_orm3-bump-verify`).
2. Run the `method_exists` check above → `setCacheable` false on ORM 3.
3. Read the installed `Paginator.php` — the removed method is still called (in dead
   code).

## Fix constraints

- Minimal code change: drop the `setCacheable` call only; no other edits, no BC
  break for `^2.11`/`^2.x` installs.
- Do the change on the hotfix branch, not directly on master.
- No commits/pushes without the owner's explicit request; pushing to GitHub uses the
  owner's credentials/permissions.
- Release rides the same `v26.41.x` train as the companion `doctrineOrmBump` hotfix
  (one production tag carrying constraint + this fix).

## Scope

- **In**: the `cloneQuery()` cleanup; release on the shared train; verification.
- **Out**: behavioural changes to the pagination logic; other package code; the
  constraint bump itself (companion spec); any specific consumer projects.
