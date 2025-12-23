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
Use AR Foundation! Add AR Camera Manager, AR Camera Background to the main camera in XR Origin; Set the main camera's camera component: Solid Color (0,0,0,0). Add an AR Plane Manager. Add an AR Session with Scripts: AR Session and AR Input Manager. However the plane tracking would not work if it is PC streaming and not being built.

---
### Vive Tracker
To use Vive Tracker while the headset is not HTC or Vive Elite, SteamVR will be needed. Use Vive Hub and SteamVR at the same time. Set the tracker correctly and connect it with the controller in SteamVR. However, during the development, it is found that the tracker's data would be captured in Unity ONLY IF SteamVR is set to OpenXR Runtime except Meta Horizon Link, which will cause a serious problem, because other functions like Passthrough is relying on Meta Horizon Link (_OR Meta Quest REQUIRES Meta Horizon Link to be set as the Runtime if it is linked to PC using USB_). So Meta Link would reopen again and again for the Runtime and SteamVR will need it as well, ending up with Unity crashes.

### Some Useful Online Tutorials
[Getting Data from VIVE Tracker (SteamVR)](https://developer.vive.com/resources/openxr/unity/tutorials/tracker-overview/getting-data-from-vive-tracker/)<br>
[Unity VR Basics - XR Hands](https://www.youtube.com/watch?v=f_jHGNxwN2g)<br>
[Unity VR Basics - Poke Interactor](https://youtu.be/MaA76IHXMOM?si=LEVnxSUZvP_y_wA1)<br>
[Hand Tracking with Unity XR Interaction Toolkit](https://youtu.be/mJ3fygb9Aw0?si=npuDh1UBi6DRD1Hu)<br>
[Mixed Reality with Unity XR Toolkit](https://youtu.be/nyFRuN1CYAI?si=-5Kq-9i3BR2QsIUG)<br>
[Vive Tracker Implementation using OpenXR in Unity](https://youtu.be/Tw5LbNLW9Q4?si=BZi8A86JWeV9qGT7)<br>
[AR Foudation Image Tracking](https://youtu.be/stKzVqvRwOQ?si=HVulbMQEGqivKrwy)<br>
[Concurrent Controller and Hand Tracking with Meta SDK](https://youtu.be/ShIeLRZbUEA?si=iAs70ueK-nL4w7Aa)<br>
