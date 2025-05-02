# Three.js Fundamentals

Three.js is a JavaScript library that makes it easier to create 3D graphics in the browser using WebGL. It provides a simpler API for working with 3D scenes, cameras, geometries, and materials.

## Key Three.js Concepts Used in the Project

### 1. Scene
The scene is the container that holds all of your 3D objects, lights, and cameras.

```javascript
const scene = new THREE.Scene();
```

### 2. Camera
The camera defines what is visible in the scene. In this project, we use a perspective camera, which mimics how human eyes see.

```javascript
const camera = new THREE.PerspectiveCamera(75, aspectRatio, 0.1, 1000);
// 75 = field of view in degrees
// aspectRatio = width/height ratio
// 0.1, 1000 = near and far clipping planes
```

### 3. Renderer
The renderer takes the scene and camera and draws the 3D content onto a canvas element.

```javascript
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(width, height);
container.appendChild(renderer.domElement);
```

### 4. Geometries and Materials
Geometries define the shape of 3D objects, while materials define their appearance.

```javascript
// Create a sphere geometry
const bodyGeometry = new THREE.SphereGeometry(size, 32, 32);
// Create a material with a specific color
const bodyMaterial = new THREE.MeshPhongMaterial({
  color: BODY_COLORS[i],
  shininess: 30,
  emissive: new THREE.Color(BODY_COLORS[i]).multiplyScalar(0.2)
});
// Create a mesh by combining the geometry and material
const bodyMesh = new THREE.Mesh(bodyGeometry, bodyMaterial);
```

### 5. Meshes
Meshes are objects that combine a geometry and a material. They can be positioned, rotated, and scaled.

```javascript
bodyMesh.position.copy(initialPositions[i]);
scene.add(bodyMesh); // Add to the scene
```

### 6. Lighting
Lights illuminate the objects in the scene. The project uses:

```javascript
// Ambient light provides global illumination
const ambientLight = new THREE.AmbientLight(0x404040);

// Directional light comes from a specific direction
const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8);
directionalLight.position.set(0, 10, 10);
```

### 7. Controls
The project uses OrbitControls to allow camera manipulation with mouse/touch:

```javascript
const controls = new OrbitControls(camera, renderer.domElement);
```

### 8. Animation Loop
Three.js uses a loop to continuously render frames:

```javascript
function animate() {
  requestAnimationFrame(animate);
  
  // Update physics, move objects, etc.
  
  // Render the scene
  renderer.render(scene, camera);
}
```

### 9. Vector3
The `Vector3` class is used extensively to represent positions, directions, and velocities in 3D space:

```javascript
const direction = new THREE.Vector3().subVectors(
  body2.position, body1.position
).normalize();
```

## How Three.js Relates to the Three-Body Problem

In our simulation:
- The three bodies are represented as meshes with sphere geometries
- Their trails are represented using tube geometries
- Gravitational forces are calculated in the physics update loop
- The renderer visualizes the resulting positions and movements

This allows us to create an interactive 3D visualization of a complex physics problem that would be difficult to understand through equations alone. 