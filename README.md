# ProcelianVRDevelopment
## Log: Quest3 + OpenXR/MetaXR
Tried to use Meta Quest 3 and VIVE tracker at the same time but failed. There would be 2 possible tracks later: HTC + VIVE tracker + OpenXR, and Quest 3 Hands + Controller + MetaXR.
### Phase 1: OpenXR
#### UI Interaction
<li>To make a 2D Canvas interactive, add <b>Tracked Device Graphic Raycaster</b> as one component to the canvas. </li>
<li>If XR Simulation is checked in the project settings, simply add a default Graphic Raycaster on the canvas so that it would be interacitve while testing in simulation on screen.</li>

#### Click Button Visualizer
It looks like that the default button component would not be loaded correctly based on the settings; Maybe the reason is some buttons' img is unchecked (or None). In that case, add a visualizer script as a component.

#### Passthrough in OpenXR
Use AR Foundation! Add AR Camera Manager, AR Camera Background to the main camera in XR Origin; Set the main camera's camera component: Solid Color (0,0,0,0). Add an AR Plane Manager. Add a GameObject as AR Session with scripts: AR Session and AR Input Manager. However, the plane tracking would not work if it is PC streaming and not being built.

---
### Vive Tracker
To use Vive Tracker while the headset is not HTC or Vive Elite, SteamVR will be needed. Use Vive Hub and SteamVR at the same time. Set the tracker correctly and connect it with the controller in SteamVR. However, during the development, it is found that the tracker's data would be captured in Unity ONLY IF SteamVR is set to OpenXR Runtime other than Meta Horizon Link, which will cause a **serious** problem, because other functions like Passthrough is relying on Meta Horizon Link (_OR Meta Quest REQUIRES Meta Horizon Link to be set as the Runtime if it is linked to PC using USB_). So Meta Link would reopen again and again for the Runtime and SteamVR will need it as well, ending up with Unity crashes.

### Some Useful Online Tutorials
[Getting Data from VIVE Tracker (SteamVR)](https://developer.vive.com/resources/openxr/unity/tutorials/tracker-overview/getting-data-from-vive-tracker/)<br>
[Unity VR Basics - XR Hands](https://www.youtube.com/watch?v=f_jHGNxwN2g)<br>
[Unity VR Basics - Poke Interactor](https://youtu.be/MaA76IHXMOM?si=LEVnxSUZvP_y_wA1)<br>
[Hand Tracking with Unity XR Interaction Toolkit](https://youtu.be/mJ3fygb9Aw0?si=npuDh1UBi6DRD1Hu)<br>
[Mixed Reality with Unity XR Toolkit](https://youtu.be/nyFRuN1CYAI?si=-5Kq-9i3BR2QsIUG)<br>
[Vive Tracker Implementation using OpenXR in Unity](https://youtu.be/Tw5LbNLW9Q4?si=BZi8A86JWeV9qGT7)<br>
[AR Foundation Image Tracking](https://youtu.be/stKzVqvRwOQ?si=HVulbMQEGqivKrwy)<br>
[Concurrent Controller and Hand Tracking with Meta SDK](https://youtu.be/ShIeLRZbUEA?si=iAs70ueK-nL4w7Aa)<br>

---
### Phase 2: Meta XR
Get All-in-One SDK<br>
Interaction SDK - Add Poke Interaction to the canvas<br>
OVRCameraRig<br>
<img width="500" height="600" alt="image" src="https://github.com/user-attachments/assets/ead62f59-a21a-4295-a4aa-62ac9ed3a5d4" /><br>
Hand Tracking Support: Controllers and Hands<br>
<img width="500" height="400" alt="image" src="https://github.com/user-attachments/assets/511f53ac-82de-4d7b-90de-4a71ea300ff6" /><br>
<img width="500" height="400" alt="image" src="https://github.com/user-attachments/assets/1c34cfef-1039-4987-83f8-327549884d93" /><br>
<img width="800" height="400" alt="image" src="https://github.com/user-attachments/assets/f6f3d944-c7a8-4afc-a1f4-3bb226222e92" /><br>
<img width="500" height="400" alt="image" src="https://github.com/user-attachments/assets/79d2abcf-a1fc-4bf4-a6d7-6205ac3b0a0a" />


## Other things updated
## Topic: Meta XR SDK Multimodal Interaction (Hands + Controllers)
### Date: January 6, 2026
### 1. The Conflict: Hand vs. Controller Anchors
**The Problem:** When attempting to attach a UI Canvas directly as a child of the OVRLeftControllerPrefab (specifically the Left Controller), two major issues occurred:<br>
**Flickering/Disappearing:** The OVRControllerHelper script automatically disables the controller model (and its children) when Hand Tracking is active or when the controller is idle. This caused the UI to vanish unexpectedly.<br>
**The "Physical Paradox":** When the Canvas is a child of the controller, and the system uses Multimodal tracking, the "Controller Anchor" sometimes snaps to the Hand's position.<br>
*(Same problem occurred when trying to use the anchor to relocate model based on the location of RightController.)* <br><br>
**The Solution:**
The Canvas must be decoupled from the OVRControllerPrefab hierarchy. It should exist independently in the scene but mathematically follow the controller's hardware position.<br>
*Instead of following through targetAnchor, directly request the position of LTouch.* <br><br>
**Implement UIFollowController.cs** <br>
Instead of parenting, use a script to update the Canvas position.<br>
It seems that we cannot directly rely on scene GameObjects like LeftControllerAnchor as their transforms are manipulated by the SDK's high-level logic (which prioritizes Hands in some states).<br><br>
*Tracking Space* <br>
**Principle:** The raw data of the controller is relative to the origin when the headset starts. We need it to convert the data into the world coordinates of the current scene.<br>
Controller Type: L Touch.<br>
*LeftControllerAnchor is a Unity GameObject. Meta's scripts (such as OVRCameraRig) will update its position based on the current strategy. In Multimodal mode, it may sometimes prioritize following gestures. The Meta SDK seems to lack detailed user manual explanations for this aspect, so just resort to using scripts to get the hardware position.<br>
OVRInput.GetLocalControllerPosition(OVRInput.Controller.LTouch) is directly asking the underlying driver: "Tell me where the left controller hardware is?".<br>
It won't change just because you extend your left hand (Hand Tracking); it only recognizes that physical controller.*
Use OVRInput.GetLocalControllerPosition(OVRInput.Controller.LTouch). This bypasses the scene graph and queries the raw hardware driver directly.<br>
The UI now stays firmly attached to the physical controller (acting as a stand/anchor), allowing the user's actual hands to move freely and interact with it without the UI "running away" or flickering.<br>

            // Obtain the local coordinates and rotation of the controller in the Tracking Space.
            Vector3 localPos = OVRInput.GetLocalControllerPosition(controllerType);
            Quaternion localRot = OVRInput.GetLocalControllerRotation(controllerType);

            // Obtain the local coordinates and rotation of the controller in the Tracking Space.
            // Since OVRInput provides coordinates relative to the TrackingSpace, we need to make a conversion.
            Vector3 worldPos = trackingSpace.TransformPoint(localPos);
            Quaternion worldRot = trackingSpace.rotation * localRot;

            // Calculate the target position after offset.
            // The logic here is the same as before: push forward based on the current rotation of the controller.
            Vector3 targetPos = worldPos + (worldRot * positionOffset);
            Quaternion targetRot = worldRot * Quaternion.Euler(rotationOffset);<br>
### Interaction Architecture: "Hands as Input, Controllers as Anchors"
To achieve a clean UX where the user holds the UI with a controller but clicks it with their hands:<br>
Disabled the ControllerRayInteractor and ControllerPokeInteractor (and the HoverInteractorsGate) in the OVRInteractionComprehensive rig.<br>
This prevents the controller from emitting rays that conflict with hand gestures. The controller is downgraded to a "dumb" physical prop, while the Hands remain the sole source of input events.<br><br>
### 2.ToggleToShowModel
Logic: When a UI Toggle is checked, the model snaps to the Right Controller's hardware position (using the same OVRInput method as the canvas).<br>
Scaling: The joystick (Thumbstick Y-axis) controls the scale of the object.<br>
Constraints: Implemented Mathf.Clamp setting reasonable minScaleLimit and maxScaleLimit to prevent the object from becoming microscopic or excessively large.<br>

#### Current Known Issues
**Canvas Jitter:** When moving the joystick aggressively to scale the model, the UI Canvas (on the left hand) or the model itself exhibits jitter.<br>
Attempted Fix: Switched from Update to LateUpdate to sync with the render loop, but the issue persisted.<br>
Status: Reverted to Update for now.<br>
