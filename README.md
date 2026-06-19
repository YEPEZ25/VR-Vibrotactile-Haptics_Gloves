# VR-Vibrotactile-Haptics_Gloves

Unity-based vibrotactile texture rendering system for portable haptic gloves, designed to enhance immersive robotic teleoperation through independent finger-level tactile feedback.

This project extends the original **VR-Vibrotactile-Haptics** repository by adapting the concept of vibrotactile roughness rendering from VR controllers to a wearable haptic glove based on vibrating thimbles. Instead of using the vibration motors of commercial VR controllers, this version aims to generate independent PWM values for each finger, allowing a vibrotactile glove to reproduce different virtual texture sensations during object manipulation.

The system is intended for immersive robotic teleoperation scenarios, where the user interacts with virtual or remote objects and receives tactile information about contact, roughness, smoothness, and surface relief.

![Immersive robotic teleoperation context](Images/teleoperation_context.jpg)

---

## Project Overview

The main goal of this repository is to implement a vibrotactile texture rendering model in Unity for integration with a portable haptic glove.

Unlike simple binary haptic feedback, which only indicates whether contact has occurred, the proposed approach generates variable vibrotactile feedback according to:

* finger-object contact detection,
* tangential finger motion over the surface,
* surface normal variation,
* virtual material height map variation,
* material-dependent haptic parameters.

The final output of the model is a set of independent PWM values, one for each finger, within the range:

```text
D1, D2, D3, D4, D5 ∈ [0, 255]
```

These values can be sent to a microcontroller or wireless RF module connected to a vibrotactile haptic glove.

---

## Unity Version

This project has been tested and adapted using:

```text
Unity 2022.3.62f3 LTS
```

The original repository was developed for an older Unity and SteamVR workflow. Therefore, some SteamVR/OpenVR files may require manual adjustment when the project is opened in Unity 2022 or newer.

---

## VR Runtime Requirement

This project uses SteamVR/OpenVR components from the original implementation. Therefore, when running the VR scene from a Windows PC, the following software is required:

1. **Steam**
2. **SteamVR**
3. A compatible VR headset
4. The corresponding PC VR connection software, if required by the headset

For example, when using a **Meta Quest headset through USB**, the recommended execution order is:

```text
1. Connect the Meta Quest headset to the PC using USB.
2. Open the Meta Quest Link / Oculus PC application.
3. Enable Quest Link from the headset.
4. Open Steam.
5. Open SteamVR.
6. Verify that SteamVR detects the headset.
7. Open Unity.
8. Load the HapticRoughness scene.
9. Press Play.
```

The recommended execution order is:

```text
Steam opened
→ SteamVR opened
→ headset detected
→ Unity opened
→ Play the scene
```

If SteamVR is not installed or has never been opened, OpenVR may fail to initialize.

---

## OpenVR Native Plugin Requirement

The file:

```text
openvr_api.cs
```

is only a C# wrapper. It does not replace the native OpenVR library.

The native Windows library is also required:

```text
openvr_api.dll
```

For Windows 64-bit execution in Unity, the DLL should be located in one of the following paths:

```text
Assets/SteamVR/Plugins/x86_64/openvr_api.dll
```

or:

```text
Assets/Plugins/x86_64/openvr_api.dll
```

If the DLL is missing, Unity may show the following error:

```text
DllNotFoundException: openvr_api
```

This means that Unity found the C# wrapper but could not find or load the native OpenVR library.

---

## openvr_api.dll Import Settings in Unity

After adding `openvr_api.dll`, select it in the Unity Project window and check its Import Settings.

Recommended configuration:

```text
Any Platform: OFF
Editor: ON
Standalone: ON
Android: OFF
```

For Windows PC execution, the plugin should be configured for:

```text
CPU: x86_64
OS: Windows
```

If the `OS: Windows` option does not appear, make sure that the DLL is inside a `x86_64` folder and that the project is configured for Windows Standalone.

Example Unity configuration:

![openvr\_api.dll Unity import settings](Images/openvr_api.png)

After modifying the import settings, press:

```text
Apply
```

Then close and reopen Unity. Native plugins are not fully unloaded while the Unity Editor remains open.

---

## SteamVR Path Registry Error

If Unity shows the following error:

```text
[SteamVR] Error during OpenVR Init: Init_PathRegistryNotFound
```

it usually means that OpenVR cannot find the SteamVR runtime path in Windows.

This can happen when:

* Steam is not installed,
* SteamVR is not installed,
* SteamVR has never been opened,
* the OpenVR path registry is missing or corrupted.

To fix it:

```text
1. Install Steam.
2. Install SteamVR from Steam.
3. Open SteamVR at least once.
4. Verify that the headset is detected.
5. Close and reopen Unity.
6. Run the scene again.
```

If the error persists, SteamVR may need to be repaired from Steam.

---

## Build Settings for PC VR

For PC-based VR execution, use the following Unity Build Settings:

```text
File > Build Settings
Platform: PC, Mac & Linux Standalone
Target Platform: Windows
Architecture: x86_64
```

The project should not be tested as an Android build when using SteamVR/OpenVR from the Unity Editor.

For Android standalone execution, such as a native Meta Quest APK, the project would require a different XR configuration and should not rely on the Windows `openvr_api.dll`.

---

## Temporary Execution Without VR

If the haptic model, material response, or communication system needs to be tested without a VR headset, the SteamVR-related objects can be temporarily disabled in the Unity scene.

Common objects/components to disable include:

```text
[CameraRig]
Controller left
Controller right
SteamVR_PlayArea
SteamVR_RenderModel
```

This allows the user to test the virtual objects, texture rendering scripts, and haptic communication logic without initializing SteamVR.

However, for full VR interaction, Steam, SteamVR, and a detected headset are required.

---

## Conceptual Model

The vibrotactile rendering model follows a finger-object interaction pipeline implemented in Unity.

First, Unity detects the contact between each virtual finger and the object surface. Then, the system captures the contact point, surface normal, texture coordinates, and finger motion. Based on this information, the model estimates the perceived texture intensity and converts it into PWM values for the glove actuators.

![General model flow](Images/model_flow.jpg)

The model considers each finger independently. For each finger `i`, the following variables are used:

```text
Cᵢ(t)   → contact state
pᵢ(t)   → contact point
nᵢ(t)   → surface normal
uvᵢ(t)  → texture coordinates
vₜ,ᵢ(t) → tangential velocity
PWMᵢ(t) → output vibration value
```

The contact state is used as an activation condition. If the finger is not touching the object, the PWM output is set to zero. If contact exists, the vibration intensity is computed according to the motion and texture properties.

---

## Finger-Surface Interaction

The system estimates the interaction between the virtual finger and the object surface using contact detection methods such as raycasting or hand collider-based detection.

The contact point and local surface normal are used to determine how the finger moves over the object. This is important because texture perception is not only related to touching a surface, but also to sliding the finger across it.

![Finger-surface interaction](Images/finger_surface_interaction.png)

The real tactile exploration figure represents the physical reference behind the proposed model. In real interaction, texture perception is not produced only by touching an object, but also by sliding the finger over its surface. This real contact situation helps explain why the proposed Unity model considers the previous position, current position, and movement direction when generating vibrotactile feedback.

![Finger-surface interaction real](Images/finger_surface_interaction_real.png)

The finger movement is decomposed into normal and tangential components. The tangential component is especially relevant for texture rendering, since sliding over rough or irregular surfaces produces stronger tactile sensations.

![Motion decomposition](Images/motion_decomposition.png)

---

## Vibrotactile Texture Rendering

The model estimates texture intensity from two main sources:

### 1. Geometric Roughness

Geometric roughness is obtained from the variation of surface normals between consecutive contact points. Smooth surfaces produce small normal variations, while rough or irregular surfaces produce larger changes.

### 2. Height Map Variation

The texture map or height map of the virtual material is also used to estimate local surface variation. When the finger moves across the object, changes in the height map are interpreted as additional texture information.

The resulting vibrotactile intensity is calculated as a normalized value between 0 and 1. This value is then smoothed over time and converted into a PWM signal.

```text
PWMᵢ(t) = round(255 · Âᵢ(t))
```

where:

```text
Âᵢ(t)   → smoothed vibrotactile intensity for finger i
PWMᵢ(t) → final PWM value sent to the actuator
```

---

## Material Profiles

Different virtual materials can be represented using different haptic parameters. Each material profile modifies the contribution of geometric roughness, texture map variation, sliding motion, and base vibration.

Example material profiles:

| Virtual Material | Expected Vibrotactile Response               |
| ---------------- | -------------------------------------------- |
| Smooth           | Minimal and soft vibration                   |
| Fabric           | Light and continuous variation               |
| Wood             | Medium roughness with directional perception |
| Rubber           | Drag-dependent sensation                     |
| Sandpaper        | High and clear roughness                     |

These profiles can be adjusted experimentally according to the vibration motors, the glove design, and user perception.

---

## Haptic Glove Architecture

The proposed architecture is organized into three main stages:

1. **Unity model**
   Computes the vibrotactile intensity for each finger based on contact, motion, and texture information.

2. **Serial / RF communication**
   Sends the generated PWM values from the computer to the haptic glove communication module.

3. **Haptic glove**
   Receives the PWM values and applies independent vibration intensity to each finger actuator.

![Unity and haptic glove architecture](Images/system_architecture_rf.png)

An example communication frame is:

```text
H=L;D1=35;D2=120;D3=80;D4=0;D5=0
```

where:

```text
H=L   → left hand
D1    → thumb actuator PWM
D2    → index finger actuator PWM
D3    → middle finger actuator PWM
D4    → ring finger actuator PWM
D5    → little finger actuator PWM
```

This structure allows each finger to receive a different vibration intensity according to the simulated texture and the user’s movement.

---

## Prerequisites

To use or modify this project, the following elements are recommended:

1. Unity Engine 2022.3.62f3 LTS or compatible.
2. Steam installed on the PC.
3. SteamVR installed and launched before running the Unity scene.
4. A VR headset compatible with SteamVR/OpenVR.
5. A virtual hand, controller, or stylus interaction system.
6. A vibrotactile haptic glove with independent actuators.
7. A microcontroller or wireless communication module capable of receiving PWM values.
8. Serial or RF communication between Unity and the glove hardware.
9. The native OpenVR library `openvr_api.dll` correctly placed and configured for Windows x86_64.

---

## How to Use

1. Open Steam.
2. Open SteamVR.
3. Verify that the VR headset is detected.
4. Open the project in Unity.
5. Load the main scene included in the project.
6. Verify that the VR headset and controller bindings are correctly configured.
7. Attach the texture rendering scripts to the corresponding Manager object or interaction controller.
8. Assign the virtual objects with mesh colliders and material properties.
9. Configure the haptic material parameters.
10. Run the scene and interact with the virtual objects.
11. Send the generated PWM values to the haptic glove through the selected communication interface.

---

## How to Modify the Haptic Response

The vibrotactile response can be modified by adjusting the parameters associated with each virtual material.

The most relevant parameters are:

1. **Geometric roughness weight**
   Controls how much the variation of surface normals affects the vibration.

2. **Height map weight**
   Controls how much the texture or height map variation affects the vibration.

3. **Sliding contribution**
   Controls the vibration component associated with tangential finger movement.

4. **Base vibration**
   Defines a minimum vibration level when contact exists.

5. **Smoothing factor**
   Controls how fast or gradual the vibration changes over time.

6. **PWM limits**
   Allows the maximum vibration intensity to be limited for comfort or actuator protection.

---

## Common Errors and Fixes

### DllNotFoundException: openvr_api

Cause:

```text
Unity cannot find the native OpenVR DLL.
```

Fix:

```text
1. Verify that openvr_api.dll exists.
2. Place it in Assets/SteamVR/Plugins/x86_64/.
3. Enable it for Editor and Standalone.
4. Use Windows x86_64 as the build architecture.
5. Restart Unity.
```

---

### Init_PathRegistryNotFound

Cause:

```text
OpenVR cannot find the SteamVR runtime path.
```

Fix:

```text
1. Install Steam.
2. Install SteamVR.
3. Open SteamVR before Unity.
4. Verify that the headset is detected.
5. Restart Unity and run the scene again.
```

---

### VREditor.GetVREnabledDevicesOnTargetGroup Error

Cause:

```text
The original SteamVR plugin uses old Unity VR APIs that are not available in Unity 2022.
```

Fix:

```text
Update or patch the SteamVR editor scripts, or use a more recent SteamVR Unity Plugin version.
```

---

### SteamVR Opens but Unity Still Shows Errors

Possible fixes:

```text
1. Close Unity.
2. Open Steam.
3. Open SteamVR.
4. Wait until the headset is detected.
5. Open Unity again.
6. Press Play.
```

Native plugin changes often require a full Unity restart.

---

## Relation to the Original Repository

This repository is based on the original project:

**VR-Vibrotactile-Haptics**
Original repository:
https://github.com/IvanNik17/VR-Vibrotactile-Haptics

The original project was designed to test the use of vibrotactile motors in HTC Vive controllers to reproduce surface roughness sensations for in-air haptics. It was based on the paper:

**Preliminary Study on the Use of Off-the-Shelf VR Controllers for Vibrotactile Differentiation of Levels of Roughness on Meshes**
VISIGRAPP 2020
https://www.scitepress.org/Link.aspx?doi=10.5220%2f0009101303340340

This adapted version keeps the general idea of vibrotactile texture differentiation but redirects the output toward a portable haptic glove with independent finger-level PWM control.

---

## Research Context

This repository is related to the development of vibrotactile feedback systems for immersive robotic teleoperation. The approach aims to improve the operator’s tactile perception during remote manipulation by providing additional information about contact and virtual surface properties.

The system is especially oriented toward applications such as:

* immersive robotic teleoperation,
* virtual object manipulation,
* haptic interaction in VR,
* texture rendering,
* robotic training environments,
* serious games,
* motor rehabilitation scenarios.

---

## Citation

If you use this repository or the concepts developed in this work, please cite the related research work:

```bibtex
@inproceedings{yepezfigueroa2026vibrotactile,
  title        = {Vibrotactile Rendering of Virtual Textures in Unity for Immersive Teleoperation},
  author       = {Yepez-Figueroa, Johnny J. and O{\~n}a, Edwin D. and Balaguer, Carlos and Jard{\'o}n, Alberto},
  booktitle    = {Jornadas de Autom{\'a}tica},
  volume       = {47},
  year         = {2026}
}
```

The original reference work should also be cited when using the roughness rendering approach based on VR controller vibration:

```bibtex
@inproceedings{nikolov2020preliminary,
  title     = {Preliminary Study on the Use of Off-the-Shelf VR Controllers for Vibrotactile Differentiation of Levels of Roughness on Meshes},
  author    = {Nikolov, Ivan and H{\o}ngaard, Jens Stokholm and Kraus, Martin and Madsen, Claus B.},
  booktitle = {Proceedings of the 15th International Joint Conference on Computer Vision, Imaging and Computer Graphics Theory and Applications},
  pages     = {334--340},
  year      = {2020},
  publisher = {SciTePress},
  doi       = {10.5220/0009101303340340}
}
```

---

## Acknowledgements

This project is based on the original VR-Vibrotactile-Haptics repository and the research work by Nikolov et al. on vibrotactile roughness differentiation using commercial VR controllers.

This adapted version is focused on the integration of vibrotactile texture rendering with a portable haptic glove for immersive robotic teleoperation.

---

## License

This repository keeps the license terms of the original project. Please review the `LICENSE` file before using, modifying, or redistributing the code.
