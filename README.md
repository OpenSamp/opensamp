# OpenSAMP

A clean-room, open-source reimplementation of the **SA-MP**
(San Andreas Multiplayer) client for GTA: San Andreas (US 1.0).
Modern C++ with an ImGui overlay over D3D9.

OpenSAMP targets [**open.mp**](https://www.open.mp/) — the open-source,
backward-compatible SAMP server. Because open.mp documents the wire
protocol, the client is written from scratch and shares no code with the
original SA-MP client.

> **Status:** early alpha / work in progress.
> Server connect works, but in-game synchronization is not fully implemented yet.

---

## What's in the box

- **`launcher/`** — small launcher that starts `gta_sa.exe` and injects
  the native DLL into it.
- **`native/`** — the DLL that lives inside the game process:
  - memory hooks and patches for GTA SA US 1.0 (via MinHook);
  - ImGui overlay on top of D3D9: chat, dialogs, debug windows;
  - networking layer on top of `sampraknet` (a RakNet fork used for
    SAMP wire-protocol compatibility);
  - connection state machine;
  - built-in crash dumper (minidump into the `crash/` folder).

## Requirements

- Windows 10/11, Visual Studio 2022 (toolset v143, x86).
- Submodules: [`imgui`](https://github.com/ocornut/imgui),
  [`sampraknet`](https://github.com/mishpro-programm/sampraknet),
  [`MinHook`](https://github.com/TsudaKageyu/minhook).
- Target game: **GTA: San Andreas, US 1.0** — other versions are not
  supported.

## Building and running

> Build instructions will land together with the first public release.
> Right now this repo is an active work-in-progress; building from
> source still requires manual setup of game paths and dependencies.

## License

TODO - will be decided before the first public release. Because OpenSAMP
is a clean-room reimplementation (not a SAMP fork), we are not bound by
SAMP's license.

Note that vendored third-party code (`vendor/sampraknet`, `vendor/imgui`,
MinHook) keeps its own upstream license.

## Roadmap and open work

See [TODO.md](TODO.md).

## Contributing

The project is still on the workbench. Pull requests are not accepted
until LICENSE, CONTRIBUTING and out-of-the-box builds are in place.
Bug reports and feature suggestions via issues are welcome.
