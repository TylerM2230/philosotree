# philoso-tree 🌳📖
![philoso-tree](https://github.com/user-attachments/assets/076a054d-69e0-4dce-99a5-4884401ae494)

## 1. Overview

Three.js project that renders a mystical 3D scene featuring a low-poly floating island, a grand tree, and a wise reader figure. The scene is set against a dynamic, procedurally generated starry sky with a flowing nebula. Orbiting the tree are several glowing orbs, each representing a link to a website dedicated to exploring a specific philosopher and their ideas. Users can interact with these orbs by clicking on them to open the corresponding link and by hovering over them to see a visual effect. The project utilizes custom shaders for the sky background and employs various Three.js techniques for procedural geometry generation, lighting, and user interaction.

**Key Features:**

* **Procedurally Generated Island and Tree:** Creates unique, organic-looking central elements.
* **Dynamic Sky Background:** A custom GLSL shader generates an animated starry sky with a flowing nebula using Simplex noise and Fractional Brownian Motion (FBM).
* **Interactive Philosophical Orbs:** Glowing orbs that link to external websites about philosophers, with hover and click interactions.
* **Wise Reader Character:** A simple, low-poly character adding to the scene's narrative.
* **Standard 3D Scene Components:** Includes camera, lighting (ambient and directional), shadows, fog, and orbit controls for navigation.

## 2. Technical Explanation

The project is structured within a single HTML file, utilizing ES6 modules for Three.js and its add-ons.

### 2.1 Core Three.js Setup (`init` function)

The `init()` function is the main entry point for setting up the entire 3D environment.

* **Scene (`THREE.Scene`):**
    * The root container for all 3D objects, lights, and cameras.
    * `scene.fog = new THREE.Fog(0x181028, 140, 680);` is applied to create atmospheric perspective, making distant objects fade into a rich dark purple color.

* **Sky Background:**
    * A large `THREE.SphereGeometry` is created to act as the skybox.
    * `THREE.ShaderMaterial` is used with custom `skyVertexShader` and `nightSkyFragmentShader` to render the dynamic sky.
        * `uniforms: { iTime: { value: 0.0 } }`: An `iTime` uniform is passed to the shaders, which is updated in the `animate` loop to drive the animations within the sky shader.
        * `side: THREE.BackSide`: Renders the material on the inside of the sphere.
        * `depthWrite: false`: Ensures the sky is always rendered behind other objects.

* **Camera (`THREE.PerspectiveCamera`):**
    * `new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 2200);`
    * Configured with a 75-degree field of view, aspect ratio based on window size, and near/far clipping planes.
    * Positioned at `(60, 55, 110)` to provide an initial overview of the scene.

* **Renderer (`THREE.WebGLRenderer`):**
    * `new THREE.WebGLRenderer({ antialias: true });`
    * Manages the drawing of the scene to the HTML `<canvas>` element.
    * `antialias: true` enables smoother edges.
    * `renderer.setPixelRatio(window.devicePixelRatio);` adjusts for high-DPI screens.
    * `renderer.shadowMap.enabled = true;` and `renderer.shadowMap.type = THREE.PCFSoftShadowMap;` enable soft shadows.
    * `renderer.toneMapping = THREE.ACESFilmicToneMapping;` and `renderer.toneMappingExposure = 0.8;` are used for more realistic lighting and color grading.

* **Controls (`OrbitControls`):**
    * `new OrbitControls(camera, renderer.domElement);`
    * Allows users to orbit, pan, and zoom the camera around the scene using the mouse.
    * `enableDamping = true` provides smoother camera movement.

* **Lighting:**
    * **Ambient Light (`THREE.AmbientLight(0x9090B0, 1.25);`):** Provides a base level of illumination to all objects in the scene with a cool, slightly purple hue.
    * **Directional Light (`THREE.DirectionalLight(0xFFE0C0, 1.55);`):** Simulates a distant light source (like the sun or moon) casting shadows.
        * `castShadow = true;` enables shadow casting from this light.
        * Shadow properties (`shadow.mapSize`, `shadow.camera`, `shadow.bias`) are configured to control shadow quality and prevent artifacts.

* **Element Creation:**
    * Calls `createFloatingIsland()`, `createOldTree()`, `createWiseReader()`, and `createLinkOrbs()` to generate and add the main scene objects.
    * The island is added directly to the scene. The tree and reader are added as children of the island, so they move with it if the island's position changes.

* **Raycasting Setup (`THREE.Raycaster`, `mouse` vector):**
    * Initialized for detecting mouse interactions with the link orbs.
    * Event listeners for `mousedown` and `mousemove` are added to the renderer's DOM element.

* **Event Listeners:**
    * `mousedown`: Calls `onDocumentMouseDown` for orb clicks.
    * `mousemove`: Calls `onDocumentMouseMove` for orb hover effects.
    * `resize`: Calls `onWindowResize` to adapt the scene to window size changes.

* **Animation Loop:**
    * Calls `animate()` to start the rendering and animation loop.

* **Initial Message:**
    * Calls `showMessage()` to display a welcome message.

* **Error Handling:**
    * A `try...catch` block wraps the initialization to catch and display errors.

### 2.2 Helper Functions

* **`showMessage(message, duration = 3000)`:**
    * Selects the HTML element with `id="messageBox"`.
    * Sets its `textContent` to the provided `message` and makes it visible.
    * Uses `setTimeout` to hide the message box after the specified `duration`.

### 2.3 Procedural Geometry Functions

* **`createFloatingIsland()`:**
    * Creates a `THREE.Group` to hold all parts of the island.
    * Uses `THREE.CylinderGeometry` for the main earth base and a thinner cylinder for the grass cap.
    * **Organic Deformation:** Iterates through the vertices of the earth and grass geometries.
        * Calculates a `warpFactor` based on trigonometric functions of the vertex's angle and y-position, creating undulating patterns.
        * Applies `warpOffsetX` and `warpOffsetZ` to displace vertices horizontally.
        * Adds random displacement (`randomFactor`) influenced by the vertex's distance from the center and its y-position to make the shape less uniform and more organic.
        * `baseEarthGeom.computeVertexNormals()` is called after deformation to recalculate normals for correct lighting.
    * **Materials:** Uses `THREE.MeshStandardMaterial` for `earthMaterial` (brown) and `grassMaterial` (green), both with `flatShading: true` for a low-poly aesthetic.
    * **Floating Rocks:** Creates several smaller `THREE.IcosahedronGeometry` meshes with the `earthMaterial`, randomly positioned and rotated around and below the main island to enhance the "floating" effect.
    * All meshes are configured to `castShadow` and `receiveShadow`.

* **`createOldTree()`:**
    * Creates a `THREE.Group` for the tree.
    * **Trunk:**
        * Constructed from multiple `THREE.CylinderGeometry` segments stacked vertically.
        * Each segment's top and bottom radii are interpolated to create a tapering effect from `baseTrunkRadius` to `topTrunkRadius`.
        * Segments (except the first) are slightly offset and rotated randomly to give the trunk a gnarled, old appearance.
    * **Roots:**
        * Several `THREE.CylinderGeometry` meshes are created, thinner at the tip and wider at the base.
        * Positioned radially around the tree base, angled downwards and outwards.
    * **Main Branch:**
        * A `THREE.CylinderGeometry` is used, positioned and rotated to extend from the upper part of the trunk.
    * **Foliage:**
        * `createFoliageClump(size, numLeaves)`: A helper function that creates a cluster of "leaves."
            * Each leaf is an `THREE.IcosahedronGeometry` with a random size and rotation.
            * Leaves are randomly positioned within a spherical volume defined by `size`.
        * Foliage clumps are created and positioned at the top of the trunk and at the tip and mid-section of the main branch. `getWorldPosition()` and `worldToLocal()` are used to correctly place foliage on the moving branch.
    * **Materials:** Uses `trunkMaterial` (dark brown) and `foliageMaterial` (dark green), both with `flatShading: true`.
    * All tree parts `castShadow` and `receiveShadow`.

* **`createWiseReader()`:**
    * Creates a `THREE.Group` for the character.
    * The reader is assembled from basic geometries:
        * **Lower Body:** `THREE.BoxGeometry` (robe).
        * **Torso:** `THREE.CylinderGeometry` (robe), slightly tilted.
        * **Head:** `THREE.SphereGeometry` (skin).
        * **Beard:** `THREE.ConeGeometry`.
        * **Hands:** `THREE.SphereGeometry`.
        * **Book:** A `THREE.Group` containing two `THREE.BoxGeometry` meshes (cover and pages).
    * **Materials:** Uses distinct `THREE.MeshStandardMaterial` instances for robe, skin, book cover, pages, and beard, all with `flatShading: true`.
    * The character is scaled down using `readerGroup.scale.set()`.
    * Positioning relative to the tree is calculated using trigonometric functions (`Math.cos`, `Math.sin`) based on an angle and distance from the trunk, and the reader is rotated to face outwards.

### 2.4 Link Orb Functions

* **`createLinkOrb(url, name, originalColorUnused, size = 2.0)`:**
    * Creates a `THREE.Group` for each orb to allow independent scaling for hover effects without affecting the child mesh's local scale if it were also animated.
    * `orbGroup.userData.originalScale` stores the initial scale for resetting hover.
    * `orbGroup.userData.baseEmissiveIntensity` stores a slightly randomized base intensity for the shimmer effect.
    * **Orb Mesh:**
        * `THREE.SphereGeometry` with increased segments (`24, 18`) for a smoother appearance.
        * `orbColor = new THREE.Color(0xFFD700);` - All orbs are set to a regal golden color.
        * `THREE.MeshStandardMaterial` is used with:
            * `color: orbColor`
            * `emissive: orbColor` (to make it glow)
            * `emissiveIntensity`: Set to `baseEmissiveIntensity`.
            * `roughness: 0.1` and `metalness: 0.8` for a shiny, regal golden look.
        * `orbMesh.userData` stores `isLinkOrb` (boolean), `url`, and `name`.
    * **Point Light:** A `THREE.PointLight` with a warm golden color (`0xFFE580`) is added as a child of the orb mesh, making the orb appear to illuminate its surroundings.
    * The `orbMesh` is added to the `linkOrbs` array (for raycasting and shimmer animation), and the `orbGroup` is added to the `linkOrbGroups` array (for bobbing/rotation).

* **`createLinkOrbs()`:**
    * Defines an array `linkData` containing information for each orb (name, URL, size). The `color` property in `linkData` is currently ignored as all orbs are made golden by `createLinkOrb`.
    * Calculates a `orbBaseHeight` relative to the tree's foliage.
    * Iterates through `linkData`:
        * Calls `createLinkOrb()` for each item.
        * Positions each orb group in a circular pattern around the tree using `Math.cos(angle)` and `Math.sin(angle)`, with randomized distance and vertical scatter.
        * Adds the orb group to the `island` group, so they are part of the main scene structure.

### 2.5 Mouse Interaction Functions

* **`onDocumentMouseDown(event)`:**
    * Triggered on mouse click.
    * Updates `mouse` vector with normalized device coordinates.
    * `raycaster.setFromCamera(mouse, camera);` updates the raycaster based on the mouse position and camera.
    * `raycaster.intersectObjects(linkOrbs, false);` checks for intersections with the orb meshes.
    * If an intersection occurs with an object marked as `isLinkOrb` and having a `url`:
        * `window.open(firstIntersectedObject.userData.url, "_blank");` opens the link in a new tab.
        * `showMessage()` displays which link is being opened.

* **`onDocumentMouseMove(event)`:**
    * Triggered on mouse movement.
    * Updates `mouse` vector and `raycaster` similarly to `onDocumentMouseDown`.
    * Checks for intersections with `linkOrbs`.
    * **Hover Effect:**
        * If an orb is intersected (`intersectedMesh.userData.isLinkOrb`):
            * Its parent group (`targetOrbGroup`) is identified.
            * If this is a new orb being hovered (not `currentHoveredOrbGroup`):
                * The previously hovered orb (if any) is scaled back to its `originalScale`.
                * `currentHoveredOrbGroup` is updated.
                * The new `currentHoveredOrbGroup` is scaled up by `hoverScaleFactor`.
                * The mouse cursor is set to `pointer`.
        * If the mouse moves off an orb or intersects a non-orb object:
            * The `currentHoveredOrbGroup` (if any) is scaled back to its `originalScale`.
            * `currentHoveredOrbGroup` is set to `null`.
            * The mouse cursor is set to `default`.

### 2.6 Window Resize

* **`onWindowResize()`:**
    * Updates the camera's `aspect` ratio.
    * Calls `camera.updateProjectionMatrix()` to apply the aspect ratio change.
    * Resizes the `renderer` to match the new window dimensions.

### 2.7 Animation Loop (`animate` function)

* **`requestAnimationFrame(animate);`:** Creates a continuous loop for rendering and animation.
* **Delta Time (`clock.getDelta()`):** Gets the time elapsed since the last frame, useful for frame-rate independent animations (though `elapsedTime` is used more here).
* **Elapsed Time (`clock.getElapsedTime()`):** Gets the total time elapsed since the `THREE.Clock` started.
* **Sky Animation:**
    * `skyBackground.material.uniforms.iTime.value = elapsedTime * 0.4;` updates the `iTime` uniform in the sky shader, driving the nebula flow and star twinkling. The `0.4` multiplier controls the overall speed of the sky animation.
* **Orb Animations:**
    * Iterates through `linkOrbGroups`:
        * **Bobbing:** `orbGroup.position.y` is animated using `Math.sin()` based on `elapsedTime` and a unique `bobbleFactor` (derived from `orbGroup.id` or index) to make each orb bob up and down slightly differently.
        * **Rotation:** `orbGroup.rotation.y` is animated to make the orbs slowly spin.
    * Iterates through `linkOrbs` (the meshes themselves):
        * **Shimmer:** `orbMesh.material.emissiveIntensity` is animated using `Math.sin()` based on `elapsedTime` and a unique `shimmerSpeed` to create a pulsing glow effect around its `baseEmissiveIntensity`.
* **Controls Update:** `controls.update();` is called if `enableDamping` is true on the `OrbitControls` to smoothly continue camera movement.
* **Render:** `renderer.render(scene, camera);` draws the scene from the camera's perspective.

### 2.8 Function Interactions

* `init()` is the orchestrator, calling creation functions and setting up event listeners that trigger interaction functions.
* `createLinkOrbs()` calls `createLinkOrb()` multiple times.
* `onDocumentMouseDown()` and `onDocumentMouseMove()` use the `raycaster` and `camera` (set up in `init`) to determine which orb is being interacted with. They also call `showMessage()`.
* `animate()` continuously updates the `iTime` uniform used by the `skyMaterial` (created in `init`) and modifies the properties of orbs (created by `createLinkOrbs`).
* Procedural generation functions (`createFloatingIsland`, `createOldTree`, `createWiseReader`) create complex geometries and materials that are then added to the scene by `init`.

## 3. Sky Background Noise Generation Algorithm

The dynamic sky background utilizes a custom GLSL fragment shader (`nightSkyFragmentShader`) to generate its appearance. The core of the flowing nebula effect is achieved using **2D Simplex Noise** combined through **Fractional Brownian Motion (FBM)**.

### 3.1 Simplex Noise (`snoise` function in GLSL)

* **What it is:** Simplex noise is a gradient noise function, similar to Perlin noise but designed to have fewer directional artifacts and better performance in higher dimensions (though here we use its 2D version). It produces natural-looking, smooth, pseudo-random patterns.
* **Implementation:** The shader includes a common GLSL implementation of 2D Simplex noise (often attributed to Stefan Gustavson, based on Ken Perlin's work).
    * It involves dividing the 2D space into a grid of skewed simplices (triangles in 2D).
    * For a given input point, it determines which simplex it falls into.
    * It calculates contributions from the vertices of that simplex using pseudo-random gradients and a falloff function.
    * These contributions are summed to produce the noise value at that point.
* **Characteristics:**
    * Smooth and continuous.
    * Band-limited (contains a limited range of frequencies).
    * Produces values typically in the range of -1.0 to 1.0.

### 3.2 Fractional Brownian Motion (`fbm_simplex` function in GLSL)

* **What it is:** FBM is a technique used to create more detailed and natural-looking textures by summing multiple "octaves" of a noise function (in this case, `snoise`). Each octave has a higher frequency and lower amplitude than the previous one.
* **Implementation:**
    ```glsl
    float fbm_simplex(vec2 p, int octaves, float persistence, float lacunarity, float initial_scale, float time_offset) {
        float total = 0.0;
        float frequency = initial_scale;
        float amplitude = 1.0;
        float maxValue = 0.0; // Used for normalizing result
        for (int i = 0; i < octaves; i++) {
            total += snoise(vec2(p.x * frequency + time_offset, p.y * frequency - time_offset * 0.3)) * amplitude; 
            maxValue += amplitude;
            amplitude *= persistence; 
            frequency *= lacunarity;  
        }
        return (total / maxValue) * 0.5 + 0.5; // Normalize to approx 0-1 range
    }
    ```
    * **`octaves`**: The number of noise layers to sum. More octaves add finer detail but increase computation. (Value: `5`)
    * **`persistence`**: Determines how much the amplitude of each successive octave decreases. A value less than 1 means higher frequencies contribute less. (Value: `0.48`)
    * **`lacunarity`**: Determines how much the frequency of each successive octave increases. A value greater than 1 means each octave adds finer details. (Value: `2.1`)
    * **`initial_scale`**: The base frequency (or zoom level) for the first octave of noise.
    * **`time_offset`**: This value, derived from `iTime`, is added to the input coordinates of the `snoise` function. By changing `time_offset` over time, the entire FBM pattern appears to move or "flow." The `x` and `y` components of the noise input are offset differently by `time_offset` to create a more dynamic, less linear flow.
* **Nebula Generation:**
    * Two layers of FBM (`fbmVal1`, `fbmVal2`) are generated with different `initial_scale` and `time_offset` values, and slightly different UV coordinates. This creates a base layer of broader, slower structures and a second layer of more detailed, slightly faster-moving wisps, adding depth.
    * `smoothstep` is applied to the FBM results to control the contrast and coverage of the nebula clouds.
    * The two FBM layers are combined to form `nebulaDensity`.
    * This `nebulaDensity` then drives the mixing of several predefined nebula colors (`nebulaColorDeepPurple`, `nebulaColorMagenta`, etc.) to create the final colored nebula effect. Brighter parts of the noise result in more contribution from the highlight colors.

### 3.3 Star Generation

The stars are generated separately using the simpler `random` hash function (not Simplex noise) on a grid. Their size, brightness, and twinkling are modulated by `iTime` and other random values to create a dynamic starfield overlaid on the nebula.

## 4. Philosophical Orbs

The glowing orbs surrounding the tree are not just decorative. Each orb is intended to be a visual hyperlink. When clicked, an orb will open a new browser tab leading to a website or resource dedicated to the exploration of a particular philosopher or school of thought. This adds an educational and contemplative layer to the visual experience, inviting users to explore profound ideas through the interactive elements of the scene.
