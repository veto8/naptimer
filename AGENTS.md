# AGENTS.md — naptimer

## What this is

Cross-platform desktop time tracker, ported from `github/gtimer` (GTK 4/Libadwaita C) to NAppGUI. Keeps gtimer's daily-total model and core workflow: projects, tasks, tags, annotations, timers, reports, CLI, and preferences.

## Stack

- C (C99)
- [NAppGUI](https://nappgui.com) SDK at `/usr/local/nappgui`
- SQLite3 for local storage
- No network code

## Build / run

The user builds and runs (`./build.sh`, or `./ask.sh` task 4). The agent should not assume the NAppGUI SDK or sqlite3 headers are present in every environment.

Task 1 installs the Windows dev environment: cmake and git via winget/Chocolatey/Scoop when missing, a C compiler (Visual Studio Build Tools with the C++ workload via winget `--override`/Chocolatey, or an existing MinGW `gcc`), vcpkg at `C:/vcpkg` with `sqlite3` (`x64-windows-static` for MSVC, `x64-mingw-static` for MinGW), and NAppGUI at `C:/nappgui`. Task 4 selects the CMake generator explicitly on Windows (Visual Studio via `vswhere`, or MinGW Makefiles) instead of relying on CMake's default, which can pick NMake with no `nmake` available. It then passes the vcpkg toolchain, the matching triplet, and `C:/nappgui/cmake` to CMake. `src/CMakeLists.txt` feeds NAppGUI an explicit source list instead of relying on its `file(GLOB)`, which can produce "No SOURCES given to target: naptimer" on Windows. The Windows CI workflow uses the hosted runner's default generator (do not force Visual Studio there; hosted images can reject an explicit generator) and ships `src/res/logo256.ico` and `src/res/logo48.ico` for the Win32 resource script; `src/CMakeLists.txt` also copies the NAppGUI template icons into `src/res` if an older working copy is missing them. The FreeBSD workflow is disabled with `if: false` and shows as skipped: NAppGUI 1.7 rejects `CMAKE_SYSTEM_NAME=FreeBSD` in `prj/NAppCompilers.cmake` ("Unknown Platform"), so a real FreeBSD build requires an upstream-supported target or a patched SDK fork. `control.yml` runs all reusable workflows for both repository owners `myridia` and `veto8`.

## Layout

- `src/main.c` — entry point, DB path resolution, CLI/GUI dispatch, `osmain_sync`
- `src/gui.c/.h` — NAppGUI window, table, filter, quick entry, day navigation, menus, dialogs, hotkeys
- `src/cli.c/.h` — gtimer-compatible CLI options and output
- `src/model.c/.h` — in-memory model backed by JSON docs; CRUD, tags, annotations, day totals, import/export
- `src/store.c/.h` — SQLite `docs(id,_rev,doc)` storage, CouchDB-style `_rev`, backup/restore/vacuum
- `src/json.c/.h` — minimal JSON parser/writer used for docs and exports
- `src/timer.c/.h` — timer start/stop/toggle, autosave, midnight splitting, pause/resume
- `src/idle.c/.h` — portable idle detection
- `src/report.c/.h` — text/HTML report generation
- `src/util.c/.h` — strings, dates, paths, file helpers

## Data conventions

- SQLite table is exactly `docs(id TEXT PRIMARY KEY NOT NULL, _rev TEXT NOT NULL, doc TEXT NOT NULL)`.
- Every persisted app object is a JSON document in `doc` with a `type` field.
- `_rev` is CouchDB-style `N-hash` and increments on every save.
- Do not add relational app tables; keep data CouchDB-sync friendly.
- No network access, ever.

## Porting notes

- gtimer source is authoritative where old gtimer docs conflict.
- Preserve gtimer behavior where practical, but document intentional differences.
- `ask.d/` contains install/build helper scripts; `ask.sh` is the task runner.
