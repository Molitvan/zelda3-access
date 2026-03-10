# AGENTS.md

## Project Purpose
This branch/workstream is focused on making the `zelda3` project accessible to blind players via screen reader narration (Tolk-based on Windows).

## Accessibility Work Completed

### 1) Tolk integration at startup
- Implemented Windows-side Tolk initialization in `src/main.c`.
- Added dynamic loading via `LoadLibraryA("Tolk.dll")` + `GetProcAddress`:
  - `Tolk_Load`
  - `Tolk_Output`
  - `Tolk_Unload`
  - `Tolk_TrySAPI`
- Startup announcement now speaks:
  - `"Zelda3 accessibility patch loaded"`
- Uses interrupt mode for startup line (`interrupt = true`).
- Added clean shutdown (`Tolk_Unload` + `FreeLibrary`) on app exit.

### 2) Reusable speech API for gameplay/menu code
- Added `Accessibility_Speak(const char *text, bool interrupt)` declaration in:
  - `src/zelda_rtl.h`
- Implemented in:
  - `src/main.c`
- Behavior:
  - Windows: routes to Tolk (UTF-8 -> wide conversion with ACP fallback).
  - Non-Windows: no-op implementation.

### 3) Pre-game menu narration (file select flows)
- Added narration state machine and dedupe logic in:
  - `src/select_file.c`
- Narration added for:
  - File select main screen (slot 1/2/3, Copy, Delete)
  - Copy file flow:
    - choose source
    - choose target
    - confirm (Yes/No)
  - Delete file flow:
    - choose slot
    - confirm (Yes/No)
  - Name entry flow:
    - cursor/key narration as you move
    - spoken confirmation on character entry
    - spoken confirmation when name accepted
- Narration is event-driven (selection/focus change), not every frame.

### 4) Name-entry key speech improvements
- Name entry now speaks explicit labels for:
  - letters (`A`..`Z`, including `I`)
  - symbols:
    - `Dash`
    - `Period`
    - `Apostrophe`
  - controls:
    - `Jump to A row button`
    - `Jump to K row button`
    - `Jump to U row button`
    - `Move left button`
    - `Move right button`
    - `End button`
  - non-interactive cells:
    - `Separator`
- Entering a key says `Entered <label>`.

## Runtime Files / DLL Notes
- Current expected runtime DLLs in repo root:
  - `Tolk.dll`
  - `nvdaControllerClient64.dll`
  - `SAAPI64.dll` (kept as original filename per user request)
- `Sapi64.dll` compatibility copy was created during debugging and then removed.

## Build / Validation Notes
- `run_with_tcc.bat` has been used repeatedly to validate changes.
- Builds and runs successfully with current changes.
- Known non-blocking warnings from TCC build:
  - `M_PI redefined`
  - `FORCEINLINE redefined`

## Files Modified for Accessibility
- `src/main.c`
- `src/zelda_rtl.h`
- `src/select_file.c`
- `zelda3.vcxproj` (content copy settings for accessibility DLLs)

## Useful Technical Details for Future Work

### Name entry token mapping source
- Name-entry keyboard token matrix is currently hardcoded in:
  - `src/select_file.c` (`kNameEntryTokens` in narration helper)
- It mirrors the game’s own name-entry token table used by logic.

### Narration design guidance already used
- Speak only on state changes (focus/selection changes), not per frame.
- Keep short, deterministic phrases.
- Use interrupt mode for menu focus movement.
- Centralize speech output through `Accessibility_Speak`.

## Caveats / Follow-up
- Symbol naming is currently based on known visible keys from the name-entry screen screenshot and token mapping.
- Unknown/unmapped tokens still fall back to `Symbol key <token>`.
- If more symbol keys appear in alternate language/layout modes, extend `NameEntry_GetTokenLabel()`.
