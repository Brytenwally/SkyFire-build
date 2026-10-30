# SkyFire-build

GitHub Actions build of **SkyFire 5.4.8** (Mists of Pandaria) Windows x64
binaries, linked against the **MySQL 8.0.44** client, with the
[SkyFire-playerbots](https://github.com/Brytenwally/SkyFire-playerbots) module
compiled in.

| Source | Default |
| --- | --- |
| Core | [Brytenwally/SkyFire_548](https://github.com/Brytenwally/SkyFire_548) `main` |
| Module | [Brytenwally/SkyFire-playerbots](https://github.com/Brytenwally/SkyFire-playerbots) `main` (in `modules/mod-playerbots`) |
| MySQL client | 8.0.44 (`libmysql.dll` + its OpenSSL 3 DLLs are shipped) |
| Boost / OpenSSL | 1.91.0 / 4.0.3, same as the core's own Windows CI |
| Compiler | Visual Studio on `windows-latest`, x64, Release |

## Building

**From the GitHub website:** Actions → *Build Windows binaries* → **Run
workflow**. Every input is optional: pick other core/module branches, commits or
forks, leave the module out, skip the extractor tools, or link another MySQL
version. When the run finishes (about an hour cold, faster once Boost and
MySQL are cached), download the zip from the run's **Artifacts** section.

**From git:** push a tag starting with `v` and the same build runs, then the
zip is published as a GitHub Release:

```sh
git tag v2026.10.30
git push origin v2026.10.30
```

Tag builds use the defaults above (core and module `main`).

**From the command line** with the [GitHub CLI](https://cli.github.com/):

```sh
gh workflow run build-windows.yml -R Brytenwally/SkyFire-build \
  -f core_ref=main -f playerbots_ref=main
gh run watch -R Brytenwally/SkyFire-build
gh run download -R Brytenwally/SkyFire-build   # fetch the artifact
```

## What is in the zip

- `worldserver.exe`, `authserver.exe` and, unless turned off, the map / vmap /
  mmap extractors and assembler.
- `worldserver.conf.dist`, `authserver.conf.dist`, `playerbots.conf.dist`.
- `libmysql.dll` (MySQL 8.0.44), the OpenSSL 4 DLLs the core uses and the
  OpenSSL 3 DLLs `libmysql.dll` needs.
- `BUILD-INFO.txt`: the exact core/module commits and dependency versions.

The binaries need the [Visual C++ 2015-2022 x64 redistributable](https://aka.ms/vs/17/release/vc_redist.x64.exe).
The database SQL and setup steps come from the core repository; see the
[SkyFire Windows install guide](https://wiki.projectskyfire.org/index.php?title=Installation_(Windows_5xx)).

## MySQL version

The workflow downloads the official `mysql-<version>-winx64.zip`, keeps only the
client headers, `libmysql.lib`/`libmysql.dll` and its DLLs, and fails the build if the
headers are not the requested version or CMake picks up another MySQL (the
runner has its own preinstalled). Run your server against MySQL 8.0.x so the
client and server match.
