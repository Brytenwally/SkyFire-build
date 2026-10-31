# CLAUDE.md

Notes for Claude sessions working on SkyFire-build. **Keep this file up to
date** with anything future sessions need, in the same commit as the change.

## What this is

A GitHub Actions workflow (`.github/workflows/build-windows.yml`) that builds
**SkyFire 5.4.8** Windows x64 binaries from `Brytenwally/SkyFire_548` with
`Brytenwally/SkyFire-playerbots` checked out into `modules/mod-playerbots` and
`Brytenwally/SkyFire-DestinyWeaver` into `modules/mod-destiny-weaver` (input
`with_destiny_weaver`, on by default; the module itself is disabled in its
conf), linked against the **MySQL 8.0.44** client. Usage is in `README.md`.

Project-wide notes (repositories, porting rules) live in
SkyFire-playerbots' `CLAUDE.md`. In short: `Brytenwally/mod-playerbots` is a
3.3.5 reference only and is never built here.

## Git rules

- Commit and push straight to **`main`**. No feature branches or PRs unless the
  owner asks for one.

## Workflow facts

- Triggers: `workflow_dispatch` (inputs: core/module repo and ref, module
  on/off, tools on/off, MySQL version) and `v*` tags, which also publish a
  GitHub Release. Tag builds use the defaults.
- Dependencies match the core's own `windows.yml`: Boost 1.91 (built from
  source, cached), OpenSSL 4.0.3 via choco, CMake via choco `--pre`.
- MySQL: the official `mysql-<ver>-winx64.zip` from cdn.mysql.com (Downloads,
  then archives); only include/, `libmysql.lib`/`.dll` and its OpenSSL 3 DLLs
  are cached. The build fails if the headers are not the requested version or
  CMake picks the runner's preinstalled MySQL; MySQL paths are passed with
  forward slashes and checked in `CMakeCache.txt`.
- Changelog: the "Changelog since the previous build" step writes `commits.json`
  (full SHAs) and `CHANGELOG.md` into the package, the job summary and the release
  notes (`RELEASE-NOTES.md` = BUILD-INFO + changelog). The previous build is the
  last successful run of this workflow (same branch/tag first, then any) whose
  package artifact can still be downloaded (`gh run download`, needs
  `actions: read`); its commits come from `commits.json`, or are parsed from
  `BUILD-INFO.txt` for older builds. Commits between the two come from the GitHub
  compare API (max 250 listed), so shallow checkouts are fine.
- No `-G`: CMake uses whatever Visual Studio `windows-latest` has, `-A x64`.
- Package = `cmake --install` output + DLLs/`.conf.dist` from `build/bin/Release`
  + MySQL DLLs + core `sql/{base,create,updates}` as `sql\` + `BUILD-INFO.txt`.
- The servers log in as `skyfire`/`skyfire` to `auth`/`world`/`characters`
  (conf defaults). Database setup is done by the core itself:
  `<Login|World|Character>Database.AutoSetup = 1`, `SqlPath = "sql"` (relative
  to the working directory). World also needs `WorldDatabase.BaseSqlFile`, a
  dump from github.com/ProjectSkyfire/SkyFire_548/releases (not in git).
  `.account create` takes `<email> <username> <password>`.
- The owner's local setup: a portable MySQL 8.0.44 in "LoA Pandaria Repack\mysql"
  bound to 127.0.0.1. Its root account has a password (set by the repack, not
  the empty one init_mysql.bat's `--initialize-insecure` gives); never write
  that password into this repo. Their confs log in as root
  (`"127.0.0.1;3306;root;<password>;auth"`) with AutoSetup/AutoCreate = 1 and
  `WorldDatabase.BaseSqlFile` = the release dump
  `sql/SFDB_full_548_26.002_2026_008_18_Release.sql`.
- That release dump is world-only, has stored procedures with
  `DEFINER=root@localhost` / `SkyFire@%` (importing it needs root or
  SET_USER_ID; the core importer does not strip DEFINERs) and an empty
  `skyfire_db_updates`, so every file in `updates/world` is applied after it.
- The cloud container cannot reach mysql.com and has no Windows: verify changes
  by running the workflow (GitHub MCP `actions_run_trigger`) and reading logs.
