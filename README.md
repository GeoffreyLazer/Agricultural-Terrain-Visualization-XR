# Agricultural Terrain Visualization in XR

A Unity and Meta Quest 3 prototype for exploring agricultural terrain, inspecting land-plot information, and comparing operation-centre placement in an immersive environment.

Developed during **September–December 2024**, the project connects a QGIS-based geospatial preparation workflow with a Unity/C# interaction layer. This repository contains the **exported Unity asset package**, including the scene, scripts, terrain and field meshes, materials, and field metadata.

**[Watch the demonstration](https://www.youtube.com/shorts/IbXerF5edYM)** · **[View the portfolio](https://geoff-portfolio-github-io.vercel.app/xr/)** · **[Download the Unity package](./MM806package.unitypackage)**

## Project overview

- **Situation:** Agricultural terrain, plot boundaries, and field attributes are difficult to relate spatially when viewed as separate map layers and tables.
- **Task:** Build a Quest 3 prototype that brings terrain and plot metadata into one immersive view, with interactions for field inspection and operation-centre placement.
- **Implementation:** Combine prepared meshes with GeoJSON-style metadata; use controller rays for selection; display field attributes in a world-space interface; track assigned fields and operation centres; calculate a distance-based planning score.
- **Outcome:** An interactive geospatial XR prototype, supported by the demonstration and exported package. No measured productivity improvement, performance benchmark, or production deployment is claimed.

## Features in the published package

- **Terrain and land plots:** separate FBX assets for the overall terrain and individual agricultural fields, plus a satellite raster asset.
- **Field inspection:** ray-based selection, material highlighting, and a panel showing area, longitude, latitude, average temperature, snowfall, precipitation, and a computed field-difficulty value.
- **Operation centres:** place centres on the ray-interactable surface and remove a selected centre.
- **Field assignment:** add and remove fields from the active planning selection, called owned fields in the code.
- **Planning feedback:** update a score using the distance from each assigned field to its closest centre, weighted by the field-difficulty calculation.
- **Controller navigation:** a separate zoom script moves the camera toward or away from a raycast target using `OVRInput`.

`fielddata.txt` contains a GeoJSON `FeatureCollection` with **238 features**. This is a dataset count, not a performance or user-study result.

## Technology and data flow

| Area | Technology / artifact |
| --- | --- |
| Engine and implementation | Unity 6, C# |
| Target device | Meta Quest 3 |
| GIS preparation | QGIS; prepared terrain and field geometry |
| Metadata | GeoJSON-style JSON parsed with Newtonsoft.Json |
| XR interaction | Meta/Oculus interaction APIs, `OVRInput`, Unity XR Core Utilities |
| Interface | TextMesh Pro |
| Rendering assets | FBX meshes, materials, Shader Graph asset, satellite raster |

```text
Prepared GIS assets                         Quest controller ray interaction
  ├─ terrain / field FBX meshes                          │
  └─ fielddata.txt (FeatureCollection)                   ▼
              │                                  SelectManager
              ▼                                  ├─ selection / highlighting
          JSONReader                             ├─ centre placement / removal
          ├─ field attributes                    ├─ field assignment
          └─ difficulty calculation              └─ planning score
              │                                        │
              └────────────────────────────────────────┤
                                                       ▼
                           CentreManager lists + DisplayManager UI
```

Geometry is already prepared in the package. `JSONReader` reads feature properties; it does not generate terrain meshes or reproject arbitrary GIS datasets at runtime. The planning score uses direct Unity-space distances. A road-network or obstacle-aware route solver is not documented in this export.

## Repository and asset map

```text
Agricultural-Terrain-Visualization-XR/
├── README.md
└── MM806package.unitypackage
```

The original package filename is retained so the exported artifact remains unchanged. After importing it into Unity, the main assets include:

| Path inside the package | Purpose |
| --- | --- |
| `Assets/Scenes/ZoomedScene.unity` | Exported interactive terrain scene |
| `Assets/Resources/fbx/full_terrain/full_terrain.fbx` | Overall terrain mesh |
| `Assets/Resources/fbx/fields/fields.fbx` | Field geometry |
| `Assets/Resources/fbx/full_terrain/GOOGLE_SAT_WM.tif` | Satellite raster asset |
| `Assets/Resources/fielddata.txt` | Area, coordinates, climate attributes, and other field metadata |
| `Assets/Prefabs/CommandCentre.prefab` | Operation-centre object |
| `Assets/Scripts/SelectManager.cs` | Selection, highlighting, centre placement, field assignment, scoring |
| `Assets/Scripts/JSONReader.cs` | JSON parsing and field-property/difficulty access |
| `Assets/Scripts/CentreManager.cs` | Runtime lists of centres and assigned fields |
| `Assets/Scripts/DisplayManager.cs` | Title, description, statistics, score, and state labels |
| `Assets/Scripts/TriggerZoom.cs` | Controller-driven camera movement |

The package also includes TextMesh Pro example/support assets and a QuickOutline script. These are separate from the project-specific scripts above. Some small scripts, including `Glow` and `OnHoverHighlighter`, are placeholders in this export.

## Setup and import

### Requirements

The original repository records **Unity 6 (6000.0.31f1)**, **Android Build Support**, and **Windows Build Support**. Start with that editor version when reproducing the project.

This export does **not** include `Packages/manifest.json`, a package lockfile, or `ProjectSettings`. Exact dependency versions, rendering configuration, and XR settings must be supplied by the receiving project.

Dependencies referenced by the scripts and scene include:

- Meta XR packages providing `Oculus.Interaction`, `Oculus.Interaction.Input`, `OVRInput`, and `Meta.XR.ImmersiveDebugger`. Scene references also include Meta interaction sample components.
- Unity XR Core Utilities (`Unity.XR.CoreUtils`).
- Unity's Newtonsoft Json package (`com.unity.nuget.newtonsoft-json`).
- TextMesh Pro/UI and suitable render-pipeline/Shader Graph support for the materials.
- Unity Test Framework/NUnit, referenced by an unused `NUnit.Framework.Constraints` import in `SelectManager.cs`.

Use dependencies compatible with Unity 6000.0.31f1; the latest SDK versions have not been verified against this export. See Meta's [XR package documentation](https://developers.meta.com/vr/documentation/unity/unity-package-manager/) for obtaining its Unity packages.

### Import steps

1. Clone the repository or download `MM806package.unitypackage`:

   ```bash
   git clone https://github.com/GeoffreyLazer/Agricultural-Terrain-Visualization-XR.git
   ```

2. Create or open a Unity **6000.0.31f1** project with the dependencies above. For Quest builds, install Android Build Support and its SDK/NDK/OpenJDK modules through Unity Hub.
3. Choose **Assets → Import Package → Custom Package**, select `MM806package.unitypackage`, review the asset list, and import it. Unity documents this [local package import workflow](https://docs.unity3d.com/6000.0/Documentation/Manual/AssetPackagesImport.html).
4. Open **`Assets/Scenes/ZoomedScene.unity`**.
5. Resolve Console errors and missing SDK/sample references before Play mode. Check the XR rig, ray interactors, UI callbacks, field parent, centre prefab, materials, and `JSONReader.jsonRaw` reference.
6. For headset testing, configure Android and Quest-compatible XR settings, include `ZoomedScene` in the build scene list, connect a developer-mode Quest 3, and build/run through Unity.

Windows Build Support was recorded in the original setup, but the interaction code depends on Meta controller APIs. A standalone mouse/keyboard workflow is not documented.

## Interaction walkthrough

Once the scene and dependencies are configured:

1. Point at the terrain and select a field to inspect its attributes and highlight it.
2. Assign fields to the active planning selection using the ownership controls.
3. Choose the place-centre action and select a terrain point to create an operation centre.
4. Compare the score as fields and centres change. Select a centre to remove it, or remove a field from the selection.
5. Inspect camera navigation through `TriggerZoom`: it reads the primary index trigger to move closer and the primary hand-trigger button to move away. Left/right mapping depends on the controller configuration.

The relevant `SelectManager` handlers are `HandlePointerEvent`, `SetModePlaceCentre`, `ownField`, `unownField`, and `destroyCentre`. Persistent scene callbacks connect to these methods.

## Validation and limitations

The demonstration provides visual evidence of the prototype. The asset map, data count, dependencies, and behavior above were checked against the package contents. A clean Unity import, APK build, and worn-headset test have **not** been reproduced as part of this documentation update.

For reproduction, check selection/highlighting, displayed metadata, centre creation/removal, field assignment, score updates, controller navigation, and Console errors. Compare behavior with the demonstration.

- **Export completeness:** this is an asset export, not a complete Unity project or installable APK. Some scene objects reference external SDK/sample assets.
- **Data alignment:** field lookup assumes object ordering matches feature ordering. Preserve or rebuild that mapping when changing data.
- **Score interpretation:** the score is a prototype heuristic. Its current loop indexes difficulty by selected-list position; review that mapping before treating the score as an analytical result.
- **Navigation:** `SetModeZoom` has no action in the selection-event switch; camera movement is in the separate `TriggerZoom` component. Some other interaction modes are placeholders.
- **Robustness:** the code assumes populated lists, valid metadata, nonzero scoring distances, and assigned scene references. Arbitrary datasets would require additional validation.
- **Performance evidence:** no FPS, latency, memory, or comparative planning measurements are published here.

## Credits

Project portfolio: **Geoffrey Lazer** — [GitHub](https://github.com/GeoffreyLazer) · [Portfolio](https://geoff-portfolio-github-io.vercel.app/xr/).

Supporting Unity/Meta, TextMesh Pro, QuickOutline, and geospatial/satellite assets retain their respective terms. A project-wide reuse license and complete dataset attribution are not included in this export.
