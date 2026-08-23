fluffytw is a base for Teeworlds/DDNet hacking, updated here to build against
DDNet 19.9.

The original project (`fluffysnaff/fluffytw`) was made for older DDNet versions
that used a different client build setup. 19.9 collects client sources with
`GLOB_RECURSE`, so there's no `add_subdirectory` for fluffytw anymore and this
folder's `CMakeLists.txt` isn't pulled in by the DDNet build itself.

## Features
- **Aimbot** (hook-assisted): aims your hook at other players.
- **ESP / visuals**: draws the aimbot FOV.

Everything is configurable as `cl_fluffy_*` console variables.

## Building it into DDNet 19.9
1. Put this folder at `ddnet-19.9/src/game/client/fluffytw`
   (so that `src/game/client/fluffytw/f_helper.h` exists).
2. Apply the DDNet-side changes below. Easiest is to just grab a DDNet 19.9
   fork that already has them.

You need the following changes on the DDNet side:

- `src/game/collision.h`: add a getter
  ```cpp
  CTile *GetTiles() const { return m_pTiles; }
  ```
- `src/engine/shared/config_variables.h`: add the config variables (also listed
  in the table at the bottom):
  ```cpp
  MACRO_CONFIG_INT(FluffyAimbot, cl_fluffy_aimbot, 0, 0, 1, CFGFLAG_SAVE | CFGFLAG_CLIENT, "fluffytw: enable aimbot")
  MACRO_CONFIG_INT(FluffyAimbotFov, cl_fluffy_aimbot_fov, 360, 0, 360, CFGFLAG_SAVE | CFGFLAG_CLIENT, "fluffytw: aimbot field of view")
  MACRO_CONFIG_INT(FluffyAimbotSilent, cl_fluffy_aimbot_silent, 0, 0, 1, CFGFLAG_SAVE | CFGFLAG_CLIENT, "fluffytw: silent aimbot (only hooks, no mouse move)")
  MACRO_CONFIG_INT(FluffyAimbotHookVisible, cl_fluffy_aimbot_hookvisible, 0, 0, 1, CFGFLAG_SAVE | CFGFLAG_CLIENT, "fluffytw: only hook when target is visible")
  MACRO_CONFIG_INT(FluffyAimbotEdge, cl_fluffy_aimbot_edge, 0, 0, 1, CFGFLAG_SAVE | CFGFLAG_CLIENT, "fluffytw: enable edge/hitpoint scan")
  MACRO_CONFIG_INT(FluffyAimbotAccuracy, cl_fluffy_aimbot_accuracy, 1, 1, 100, CFGFLAG_SAVE | CFGFLAG_CLIENT, "fluffytw: edge scan accuracy (higher = more hitpoints)")
  MACRO_CONFIG_INT(FluffyEsp, cl_fluffy_esp, 0, 0, 1, CFGFLAG_SAVE | CFGFLAG_CLIENT, "fluffytw: enable esp/visuals")
  MACRO_CONFIG_INT(FluffyEspFov, cl_fluffy_esp_fov, 0, 0, 1, CFGFLAG_SAVE | CFGFLAG_CLIENT, "fluffytw: draw aimbot fov")
  ```
- root `CMakeLists.txt`: add the fluffytw files to `GAME_CLIENT` (inside the
  `set_src(GAME_CLIENT GLOB_RECURSE ...)` block), in alphabetical order:
  ```
  fluffytw/aimbot/aimbot.cpp
  fluffytw/aimbot/aimbot.h
  fluffytw/aimbot/aimbot_scans.cpp
  fluffytw/f_bots.cpp
  fluffytw/f_bots.h
  fluffytw/f_component.h
  fluffytw/f_config.h
  fluffytw/f_helper.cpp
  fluffytw/f_helper.h
  fluffytw/f_visuals.cpp
  fluffytw/f_visuals.h
  fluffytw/f_visuals-base.cpp
  ```
- `src/game/client/gameclient.cpp`:
  ```cpp
  #include <memory>
  #include "game/client/fluffytw/f_helper.h"
  ...
  std::unique_ptr<FHelper> fHelper; // near the other globals
  ...
  // inside CGameClient::OnConsoleInit():
  fHelper = std::make_unique<FHelper>(this);
  ```
- `src/game/client/components/controls.cpp`: include the header, then in
  `SnapInput` (after the direction block) feed the config and run the bots:
  ```cpp
  #include "game/client/fluffytw/f_helper.h"
  ...
  fHelper->m_pConfig->aimbotCfg.enabled = g_Config.m_FluffyAimbot != 0;
  fHelper->m_pConfig->aimbotCfg.fov = static_cast<float>(g_Config.m_FluffyAimbotFov);
  fHelper->m_pConfig->aimbotCfg.silent = g_Config.m_FluffyAimbotSilent != 0;
  fHelper->m_pConfig->aimbotCfg.hookVisible = g_Config.m_FluffyAimbotHookVisible != 0;
  fHelper->m_pConfig->aimbotCfg.edge = g_Config.m_FluffyAimbotEdge != 0;
  fHelper->m_pConfig->aimbotCfg.accuracy = static_cast<float>(g_Config.m_FluffyAimbotAccuracy);
  fHelper->m_pBots->Run();
  ```
- `src/game/client/components/players.cpp`: include the header, then in
  `RenderPlayer` (once `Position` is known) run the visuals:
  ```cpp
  #include "game/client/fluffytw/f_helper.h"
  ...
  fHelper->m_pConfig->espCfg.enabled = g_Config.m_FluffyEsp != 0;
  fHelper->m_pConfig->espCfg.drawFov = g_Config.m_FluffyEspFov != 0;
  fHelper->m_pVisuals->Run(ClientId, Angle, Position);
  ```

## Enabling
```
cl_fluffy_aimbot 1
cl_fluffy_aimbot_hookvisible 1
cl_fluffy_aimbot_edge 1
```
The aimbot is **hook-assisted**. For it to aim on its own, enable
**Aimbot + Auto Hook + Edge Scan**. With only **Aimbot** on, it just aims while
you hold hook and there's a clear line to the target.

## Config variables
| Variable | Default | Description |
| --- | --- | --- |
| `cl_fluffy_aimbot` | 0 | Master enable |
| `cl_fluffy_aimbot_fov` | 360 | Aim FOV (0–360) |
| `cl_fluffy_aimbot_silent` | 0 | Silent (hook only, no mouse move) |
| `cl_fluffy_aimbot_hookvisible` | 0 | Auto-press hook when target visible |
| `cl_fluffy_aimbot_edge` | 0 | Edge/hitpoint scan |
| `cl_fluffy_aimbot_accuracy` | 1 | Edge scan accuracy (1–100) |
| `cl_fluffy_esp` | 0 | Enable ESP |
| `cl_fluffy_esp_fov` | 0 | Draw aimbot FOV |
