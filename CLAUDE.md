# CLAUDE.md

Notes for Claude sessions working on SkyFire-build. **Keep this file up to
date** with anything future sessions need, in the same commit as the change.

## What this is

A GitHub Actions workflow (`.github/workflows/build-windows.yml`) that builds
**SkyFire 5.4.8** Windows x64 binaries from `Brytenwally/SkyFire_548` with
`Brytenwally/SkyFire-playerbots` checked out into `modules/mod-playerbots`,
linked against the **MySQL 8.0.44** client. Usage is in `README.md`.

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
- No `-G`: CMake uses whatever Visual Studio `windows-latest` has, `-A x64`.
- Package = `cmake --install` output + DLLs/`.conf.dist` from `build/bin/Release`
  + MySQL DLLs + `BUILD-INFO.txt`.
- The cloud container cannot reach mysql.com and has no Windows: verify changes
  by running the workflow (GitHub MCP `actions_run_trigger`) and reading logs.
