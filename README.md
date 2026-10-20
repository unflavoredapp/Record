# BattleCam

iPhone-only, landscape, bodycam-style tactical FPS. SwiftUI front end over a custom Metal engine ("MX"). Plays offline against bots. Built unsigned via XcodeGen + GitHub Actions.

Built by MRzefV.

## Build

1. Push to `main` (or run **Build IPA** from the Actions tab).
2. Download the `BattleCam-ipa` artifact: an unsigned `BattleCam.ipa`.
3. Sign and install it with your usual signer.

On failure the run summary lists the compiler errors. The full log is in the `build-log` artifact and in `build-logs/errors.txt` on the `build-logs` branch.

## Layout

- `project.yml`: XcodeGen spec (bundle `city.battlecam.game`, iOS 16+, iPhone)
- `.github/workflows/build.yml`: CI. Its `keep=` list prunes stale Swift files, so **add every new `.swift` file to it**.
- `Sources/`: game and engine
  - `MX*.swift`: Metal engine (loader, renderer, shaders, materials)
  - `Tactical*.swift`, `Tac*.swift`: game logic, maps, HUD, weapons, lobby
  - `TacAnnex.swift`: ANNEX, the CQC two-floor map with continuous stairs (default)
  - `TacFX.swift`: bullet decals, dust, sparks, blood, glass
  - `TacMuzzle.swift`: muzzle flash
- `Resources/`: gun models (`gun_*.glb`, `gunmeta.json`), Annex baked maps (`annex_f1.glb`, `annex_f2.glb`), fonts, fx
- `Assets.xcassets/`: icons and images

## Maps

ANNEX (default): compact CQC house, 1000x800 lot, 10 rooms per floor, 2-3 doors on every room, one continuous staircase. BLOCK 9: large. CUSTOM: editor map. Pick one in the lobby's OPERATION card.

## Engine notes

- The Annex is modelled in Blender and baked in Cycles (albedo, light and AO, height). Materials load as baked, so the engine skips its own wall and floor lighting tricks on them.
- Post: ACES tone mapping, bloom, adaptive exposure, lens dirt, bodycam grade.
- Dynamic: 8 nearest point lights, muzzle-flash light, sun shadow map.
- Not implemented: SSAO (needs a stored depth buffer).

## Controls

Fixed move stick (bottom-left). Drag the right side to free-look (up, down, left, right). FIRE button for manual shooting (no auto-fire, no aim assist). ADS toggle raises the weapon so its sight anchor sits on the camera axis with the barrel levelled, slows look sensitivity and tightens spread. No crosshair. Reload, crouch, jump.

## Bots

Every bot has its own traits (aggression, patience, reflex, nerve) and picks its own posts, so they spread out instead of moving as squads.

## Status

The latest changes have not been compiled on device. If CI fails, send the `errors.txt`.

## Operators
Every operator is the Special Task Force character, a real skinned model. Presets live in `Sources/TacGear.swift` (Assault, Breacher, Recon, Marksman, Medic, Support, Spectre, Urban). Customize → Operators lets you change face (7), headgear (bare / helmet / NVG), helmet colour (5), eyewear (5 lenses), uniform (7), vest (5 or none) and gloves per slot; picks are stored as overrides on top of the preset. Bots, the lobby and every preview use the same model: arms pose on each gun's real grips and the legs walk. First-person sleeves and gloves follow the selected kit. Admin / staff ranks keep the code-built soldier.

## Engine: skinning
`MXGLTF` loads glTF skins (JOINTS_0 / WEIGHTS_0 / inverse bind matrices). `MXSkin` deforms the bind pose on the CPU each frame (linear blend, 4 influences) into three rotating GPU buffers, only for meshes that are on screen. `MXRig` finds bones by name and applies a pose (model-space rotation per bone, optional bone-head position). Files: `Sources/MXSkin.swift`, `MXLoader.swift`, `MXRender.swift`.

## Editors
- Map builder: real 3D viewport (orbit / pan / pinch zoom, tap to paint), blueprint views, and a turntable asset viewer with size, height and behaviour for every tile.
- Customize → Weapons → Camo & Attachment Lab: design camos (pattern + 3 colours) and attachments (stats under a power cap), save, equip, copy a share code, import a code.
- Tap your name in the lobby for rank, league, W/L and recent matches.

## Blender art
`Resources/prop_*.glb` (13 modelled props) and `Resources/ic_*.png` (142 rendered asset icons) are built with Blender scripts (props_build.py / icons_build.py). The map builder shows them in its asset tree, preview and 3D view.

## Lobby look and Blender operators

- UI face is Avenir Next Condensed (ships with iOS); numerals in HUDs are monospaced. Palette is gunmetal with one restrained orange.
- Lobby backdrop is a desaturated night scene with film grain, scanlines, vignette and a live bodycam timestamp (`BodycamBackdrop`, `BodycamGrain`, `BodycamOSD`). Panels use viewfinder corner brackets (`.tacBrackets()`).
- Operator assets: `stf_rig.glb` (one 68-bone skeleton, six skinned parts: body, head, vest, helmet, NVG, lenses) and `stfpose.json` (per gun: solved arm / torso bone pose and where the gun sits in the hands). Textures are swapped at runtime from `stf_body_*`, `stf_gear_*`, `stf_helm_*`, `stf_head_*` jpgs. `TacOperator.bot(style:)` builds a live operator; `TacOperator.node(style:gun:)` a posed preview. Every use falls back to the code-built soldier if a piece is missing.
- `op_<kit>.png` are the card portraits; `stfsw_*.png` are the swatches in Customize.
- In-game HUD uses the same language: square gunmetal panels (`tuiGlass`), corner brackets, condensed type, monospaced numerals, no capsules.
- Rebuild: `blender-scripts/stf_rig.py` (needs `stf_lib.py`, `stf_pose.py`, `stf_export.py` beside it) writes `stf_rig.glb` + `stfpose.json`; `stf_rigcheck2.py` checks the runtime pose maths against the Blender solver. Python 3 with bpy.

## Credits

- "Makarov PM" by Martin Trafas (https://sketchfab.com/TinoD2) is licensed under CC BY 4.0 (https://creativecommons.org/licenses/by/4.0/). Source: https://sketchfab.com/3d-models/makarov-pm-dbf03ce95c834cb7b07edba799dcd46a. Changes: cleaned up (centered, grounded, renamed), converted to GLB/FBX/OBJ/BLEND; re-oriented, split into slide / magazine / frame for animation, and exported to `Resources/gun_pm.glb`. Distributed by AiZity (https://aizity.com).
- Special Task Force rigged character: supplied with no licence file. Confirm its licence before any public release.

## CI
The build workflow only builds. It no longer prunes or commits anything to the repository.
