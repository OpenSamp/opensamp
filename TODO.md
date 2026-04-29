# TODO

Roadmap and outstanding work that needs to be closed before the repository
can go public and accept third-party contributions.

Priorities:
- **P0** — must-do before going public. Either embarrassing or legally
  risky to ship without.
- **P1** — needs to land before the first public alpha build.
- **P2** — quality-of-life and code-health improvements; can be done
  incrementally.

---

## P0 — Repository hygiene and legal

- [ ] **LICENSE.** Pick a license and commit the file.
- [ ] **`.gitignore` is too small.** Today it lists only `bin/`, `obj/`,
      `packages/`, `riderModule.iml`, `_ReSharper.Caches/`. Missing:
  - [ ] `Debug/`, `Release/`, `x64/` (VS x86/x64 build outputs)
  - [ ] `*.obj`, `*.pdb`, `*.ilk`, `*.idb`, `*.tlog`, `*.lastbuildstate`,
        `*.log`, `*.exp`, `*.lib` (`.lib` carefully — we have `vendor/lib/`)
  - [ ] `*.user`, `*.vcxproj.user`, `*.suo`
  - [ ] `.vs/`, `.idea/` (unless we keep curated per-user configs)
  - [ ] `.run/` (Rider run configurations)
  - [ ] `Backup/` (a stale backup of `opensamp.sln` — not history-worthy)
- [ ] **Purge build junk from the index.** ~216 files under
      `native/Debug/`, `Backup/`, `redist/` etc. are tracked. Run
      `git rm -r --cached <paths>` and commit a clean state.
- [ ] **`redist/`.** VC_redist (~25 MB) and the .NET runtime installer
      (~70 MB) are committed straight into git. Move them to release
      assets or have the launcher download them on demand.
- [ ] **Confirm `bin/` is never committable.** `bin/` currently contains
      a real GTA SA install (`gta_sa.exe`, `models/`, `audio/`, `data/`,
      `scripts/`, `anim/`). It is `.gitignore`d today, but any regression
      in the ignore rules would publish **Rockstar's proprietary assets
      in a public repo**. Add a pre-commit hook / CI check that rejects
      commits containing `gta_sa.exe`, `*.img`, `*.dff`, `*.txd`,
      `*.ide`, `*.ipl`.
- [ ] **README.md** — present, but lacks build instructions. Fill it in
      once builds are reproducible.
- [ ] **CONTRIBUTING.md** — code style, PR workflow, how to discuss
      additions to the offset map.

## P0 — Security and reputation

- [ ] **Remove the hardcoded test server** in
      [native/gui/chat.cpp:147](native/gui/chat.cpp:147) — `// @todo
      temporary dev shortcut`. We don't want public builds to ship a
      shortcut that connects to someone's private dev server.
- [ ] **Audit history for leaked nicknames / IPs / passwords.**
      Run `git log -p | grep -iE "(password|192\.168|samp\.|open\.mp)"`
      before going public. Rewrite history if anything sensitive is
      found, or accept the leak and rotate.

## P1 — Builds out of the box

- [ ] **CMake alongside `.sln`.** Right now the project only builds via
      Visual Studio 2022 on Windows. That's fine for the target platform
      but CMake makes CI and clang-cl experiments much cheaper.
- [ ] **GitHub Actions CI.** At minimum: build on push. Ideally with
      clang-tidy and cppcheck.
- [ ] **`docs/build.md`** — step-by-step: required toolchain, where to
      put the SDKs, how to fetch submodules, how to point the build at
      a game install.
- [ ] **Update the `imgui` submodule.** Pinned at `v1.62` (2018) — about
      5000+ commits behind upstream. Bump to a recent stable tag.
- [ ] **Move log files out of `CWD`.** `GameReady.log` and `OpenSamp.log`
      are written next to `gta_sa.exe`. Move them to
      `%LOCALAPPDATA%\OpenSAMP\logs\` (or alongside `crash/`) so we
      don't pollute the game directory.
- [ ] **Game install path in config**, not hardcoded inside the injector.

## P1 — Architectural debt

- [ ] **Break up `native/sampraknet_bridge.cpp`** (3204 lines!).
      Suggested split:
  - `bridge/init.cpp` — `Initialize`, `Shutdown`, `Connect`, `Disconnect`.
  - `bridge/sync_onfoot.cpp` — `Send/HandleOnFootSync`.
  - `bridge/sync_incar.cpp` — `Send/HandleInCarSync`, passenger.
  - `bridge/sync_aim.cpp`, `sync_unoccupied.cpp`, `sync_trailer.cpp`.
  - `bridge/rpc/*.cpp` — one file per RPC family (server, world,
    scr_player, scr_vehicle, scr_object, dialog, textdraw…).
  - `bridge/rpc_table.h` — all IDs in a single `constexpr` namespace
    instead of ~140 global `static int RPC_*`.
- [ ] **Drop the ~340 lines of commented-out code** in the bridge —
      git history has it; in-source it's just noise.
- [ ] **`addresses.h`** — a single offset map for GTA SA US 1.0. The
      codebase currently has ~422 magic addresses (`0x53C095`,
      `0xBA677B`, `0x732E30`, …) scattered across 23 files. Without a
      central offset table no contributor can tell what is being
      hooked or patched. Reference public reverse-engineering work
      (e.g. [plugin-sdk](https://github.com/DK22Pac/plugin-sdk)) but
      list **every address we use**.
- [ ] **One naming style.**
- [ ] **`using namespace std`** at
      [native/gui/chat.cpp:14](native/gui/chat.cpp:14)
      and `using namespace std::chrono` next to it — drop them.
- [ ] **Hardcoded server netcode version.** `#define NETGAME_VERSION 4057`
      at [native/sampraknet_bridge.cpp:29](native/sampraknet_bridge.cpp:29)
      means we support exactly one server version. Move to a config /
      per-server selection.
- [ ] **`int g_myPlayerID = -1;`** plus dozens of other globals in the
      bridge — fold them into a client state struct.

## P1 — Functional gaps

These are what makes the client **not yet playable**.

- [ ] **`OnEnter` / `OnExit`** in the FSM — empty. Transitions only
      log; no side effects.
- [ ] **`CONNECTION_FAILED` / timeout in `TickConnecting`** — no
      handling. Bridge needs to surface those events.
- [ ] **`@todo` patches** in [native/patches.cpp](native/patches.cpp):
  - `0x609C08` NOP 39 — CPlayerPed::ProcessControl crash fix.
  - `0x47BF54` — SCM events processor hook.
  - `0x6884C4` NOP 6 — "don't rotate ped from camera".
  - `0x57BA57` — disable the original main menu (we draw our own).
  - `0x705331..0x7053AF` — disable the in-game photo camera.
- [ ] **`m_maxLines = 200` / `m_visibleLines = 15`** in
      [native/gui/chat.hpp](native/gui/chat.hpp:65) marked `@todo
      use/change` — the chat buffer isn't actually capped to that limit.

## P2 — Code quality and tooling

- [ ] **`.editorconfig`** — indent, line endings, encodings.
- [ ] **`.clang-format`** — single formatting style.
- [ ] **`.clang-tidy`** — start with `modernize-`, `bugprone-`, `cert-`.
- [ ] **`docs/architecture.md`** — diagram of
      `launcher → inject → DllMain → MainThread → patches/hooks/D3D
      → ImGui overlay → CNetGame FSM → bridge → RakClient`.
- [ ] **`docs/offsets.md`** — for each address, where it came from
      (own reverse engineering / classic SAMP / plugin-sdk).
- [ ] **Document `parse_dmp.py` and `parse_stack.py`.** They aren't
      even tracked in git, and how to apply them to a `crash/*.dmp`
      is not documented anywhere.
- [ ] **Tests.** At least unit tests for `util/encoding.h` and the
      chat color-tag parser (`ImGuiChat::ParseHexColorTag`). The RPC
      bitstream parsers are great regression-test candidates.
- [ ] **Replace `using PLAYERID = unsigned short;`** globally with a
      strong typedef / enum class — prevents accidental confusion with
      `VEHICLEID`.
- [ ] **`ExitProcess(1)`** at
      [native/hooks.cpp:119](native/hooks.cpp:119) on init failure is
      brutal. Replace with a graceful shutdown + user-visible message.
- [ ] **MinHook via `#pragma comment(lib, ...)`** in
      [native/dllmain.cpp](native/dllmain.cpp) — implicit dependencies.
      Move to project linker settings.
- [ ] **Singletons via `static T instance;` in `Get()`** — fine for a
      DLL, but verify destruction order at `DLL_PROCESS_DETACH`.

## Explicit non-goals for this list

- Implementing the full SA-MP feature surface in one go (objects,
  gangzones, textdraws, menus, custom skins/objects). These are
  separate epics that come **after** basic player and vehicle sync
  are stable.
- Supporting other `gta_sa.exe` builds (EU 1.0, US/EU 1.01, Steam,
  re3/reVC). Separate epic, after the first public alpha.
