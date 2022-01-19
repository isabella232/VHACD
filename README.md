# VHACD

[![License](https://img.shields.io/badge/license-BSD--3-green.svg)](LICENSE.md)
![Unity](https://img.shields.io/badge/unity-2020.2+-brightgreen)

---

We're currently working on lots of things! Please take a short moment fill out our [survey](https://unitysoftware.co1.qualtrics.com/jfe/form/SV_0ojVkDVW0nNrHkW) to help us identify what products and packages to build next.

---

## The V-HACD library decomposes a 3D surface into a set of "near" convex parts.

![Approximate convex decomposition of "Camel"](https://github.com/kmammou/v-hacd/raw/master/doc/acd.png)

## Installation

1. Using Unity 2020.2 or later, open the Package Manager from `Window` -> `Package Manager`.
2. In the Package Manager window, find and click the + button in the upper lefthand corner of the window. Select `Add package from git URL....`

    ![image](https://user-images.githubusercontent.com/29758400/110989310-8ea36180-8326-11eb-8318-f67ee200a23d.png)

3. Enter the git URL for the desired package. Note: you can append a version tag to the end of the git url, like `#v0.4.0` or `#v0.5.0`, to declare a specific package version, or exclude the tag to get the latest from the package's `main` branch.

    ```
    https://github.com/Unity-Technologies/VHACD.git?path=/com.unity.robotics.vhacd#generator/vhacd-child
    ```

4. Click `Add`.

To install from a local clone of the repository, see [installing a local package](https://docs.unity3d.com/Manual/upm-ui-local.html) in the Unity manual.

---

---

## Mesh Decomposer Tool Usage

1. If you are in Play mode, exit Play mode.
2. To start, go to the top-level menu bar and select `VHACD > Generate Collider Meshes`.
3. In the new `VHACD Generation Settings` window that appears, select a `Generation Mode`.
   1. `Single Mode` allows you to select one prefab or FBX file to import into the scene and fine-tune the parameters before saving the file.
      1. If in `Single Mode`, it is suggested that you open an empty scene as the mesh will temporarily import into the currently open scene.
   2. `Batch Mode` allows you to select a directory and convert all prefabs or FBX files found inside to have newly generated colliders.
4. Select a file or directory to convert.
   1. In `Batch Mode`, select if you want to apply changes to `Prefabs` or `FBX` files.
5. The `Mesh save path` will print below the Selected Filename in Single Mode. In Batch Mode, this path will default to the chosen Asset Directory, but you can change this location.
6. Where applicable, update the following:
   1. `Overwrite any existing collider components?`: If this is *checked*, any existing mesh collider will be removed and replaced with the newly generated ones.
   2. `Overwrite existing assets?`: In case a prefab is selected, when this is *checked*, this will automatically overwrite the same prefab asset. If this is *unchecked*, you can select a different directory to save the new prefabs to.
7. You can modify the VHACD parameters for the generation process as desired. View the [parameters](#parameters) section to review these parameter definitions.
8. If `Add to new child object` is *checked*, a new GameObject will be added to your imported mesh called `VHACD_Colliders`, leaving the rest of your hierarchy of GameObjects unmodified. This object will contain a copied hierarchy, replacing the Mesh Renderers with Colliders.
   1. If `Child defaults to off` is *checked*, the saved Prefab will deactivate the `VHACD_Colliders` child object. If it is *unchecked*, the `VHACD_Colliders` child object will be active.
   
   If `Add to new child object` is *unchecked*, the newly generated colliders will add directly to the objects in the hierarchy containing renderers.
9.  Generation step:
   1. In `Single Mode`:
      1. You can now `Import Mesh` to add the mesh into your scene for preview purposes.
      2. Once imported, you can select `Generate!` to generate mesh colliders for the imported object with the given VHACD parameters. Note the Scene View updates as the colliders are added.
      3. You can now modify the parameters and Re`Generate!`, `Save` the new prefab (which will open a save file panel), `Clear Object`, which will undo your recent generation and remove the preview object from the scene, or `Reset VHACD Parameters`, which will reset the VHACD Parameters to the default without regenerating.
   2. In `Batch Mode`:
      1. You can preview how many assets are found below the VHACD parameters, next to `Assets found in directory`. You can now click `Generate!` to iterate through the assets and generate new collider meshes. Note the Scene View updates as the colliders are added for each object. Also note that a progress bar in the Editor Window will display how many assets remain to be converted.
      2. When the generation is done, the progress bar will disappear and the preview objects will be removed from the scene.

---

## Why do we need approximate convex decomposition?

Collision detection is essential for [realistic physical interactions](https://www.youtube.com/watch?v=oyjE5L4-1lQ) in video games and computer animation. In order to ensure real-time interactivity with the player/user, video game and 3D modeling software developers usually approximate the 3D models composing the scene (e.g. animated characters, static objects...) by a set of simple convex shapes such as ellipsoids, capsules or convex-hulls. In practice, these simple shapes provide poor approximations for concave surfaces and generate false collision detection.

![Convex-hull vs. ACD](https://raw.githubusercontent.com/kmammou/v-hacd/master/doc/chvsacd.png)

A second approach consists in computing an exact convex decomposition of a surface S, which consists in partitioning it into a minimal set of convex sub-surfaces. Exact convex decomposition algorithms are NP-hard and non-practical since they produce a high number of clusters. To overcome these limitations, the exact convexity constraint is relaxed and an approximate convex decomposition of S is instead computed. Here, the goal is to determine a partition of the mesh triangles with a minimal number of clusters, while ensuring that each cluster has a concavity lower than a user defined threshold.

![ACD vs. ECD](https://raw.githubusercontent.com/kmammou/v-hacd/master/doc/ecdvsacd.png)


## Parameters
| Parameter name | Description | Default value | Range |
| ------------- | ------------- | ------------- | ---- |
| resolution | maximum number of voxels generated during the voxelization stage	| 100,000 | 10,000-64,000,000 |
| depth |	maximum number of clipping stages. During each split stage, all the model parts (with a concavity higher than the user defined threshold) are clipped according the "best" clipping plane | 20 | 1-32 |
| concavity |	maximum concavity |	0.0025 | 0.0-1.0 |
| planeDownsampling |	controls the granularity of the search for the "best" clipping plane | 4 | 1-16 |
| convexhullDownsampling | controls the precision of the convex-hull generation process during the clipping plane selection stage | 4 | 1-16 |
| alpha | controls the bias toward clipping along symmetry planes | 0.05 | 0.0-1.0 |
| beta | controls the bias toward clipping along revolution axes | 0.05 | 0.0-1.0 |
| gamma |	maximum allowed concavity during the merge stage | 0.00125 | 0.0-1.0 |
| pca |	enable/disable normalizing the mesh before applying the convex decomposition | 0 | 0-1 |
| mode | 0: voxel-based approximate convex decomposition, 1: tetrahedron-based approximate convex decomposition | 0 | 0-1 |
| maxNumVerticesPerCH |	controls the maximum number of triangles per convex-hull | 64 | 4-1024 |
| minVolumePerCH | controls the adaptive sampling of the generated convex-hulls | 0.0001 | 0.0-0.01 |

---

## Support
For questions or discussions about Unity Robotics package installations or how to best set up and integrate your robotics projects, please create a new thread on the [Unity Robotics forum](https://forum.unity.com/forums/robotics.623/) and make sure to include as much detail as possible.

For feature requests, bugs, or other issues, please file a [GitHub issue](https://github.com/Unity-Technologies/v-hacd-unity/issues) using the provided templates and the Robotics team will investigate as soon as possible.

## More from Unity Robotics
Visit the [Robotics Hub](https://github.com/Unity-Technologies/Unity-Robotics-Hub) for more tutorials, tools, and information on robotics simulation in Unity!

## License
[BSD 3-Clause](LICENSE)