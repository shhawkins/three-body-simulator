# Performance Optimization

This document outlines techniques for optimizing performance in the Three Body Simulator, focusing on addressing the memory leak and improving overall efficiency.

## Memory Management

### Identifying Memory Leaks

Memory leaks often occur when:
1. Objects are created but not properly disposed of
2. References to objects are maintained after they're no longer needed
3. Geometry and materials continue to consume memory even when not visible

### Proper Object Disposal

```javascript
function cleanupMemory() {
  // Clear old trails
  for (let i = 0; i < 3; i++) {
    if (trailMeshes[i]) {
      scene.remove(trailMeshes[i]);
      trailMeshes[i].geometry.dispose();
      trailMeshes[i].material.dispose();
      trailMeshes[i] = null;
    }
  }
  
  // Force garbage collection if available
  if (window.gc) {
    window.gc();
  }
  
  // Recreate trails with reduced complexity
  trailLength = Math.max(100, trailLength - 10);
  trailSegments = Math.max(50, trailSegments - 2);
  updateTrails();
}
```

### Limiting Trail History

```javascript
// Limit trail length to prevent excessive memory usage
if (bodies[i].trail.length > MAX_TRAIL_LENGTH) {
  bodies[i].trail = bodies[i].trail.slice(-MAX_TRAIL_LENGTH);
}
```

## Rendering Optimizations

### Dynamic Level of Detail (LOD)

```javascript
function reduceSimulationComplexity() {
  // Reduce trail length gradually
  if (trailLength > 100) {
    trailLength = Math.max(100, trailLength - 10);
    updateTrails();
  }
  
  // Reduce number of trail segments
  if (trailSegments > 50) {
    trailSegments = Math.max(50, trailSegments - 2);
    updateTrails();
  }
}
```

### Adaptive Graphics

```javascript
// Monitor FPS and adapt
function updateFPS() {
  frameCount++;
  const now = performance.now();
  if (now - lastFPSUpdate >= 1000) {
    currentFPS = Math.round((frameCount * 1000) / (now - lastFPSUpdate));
    frameCount = 0;
    lastFPSUpdate = now;
    
    // Only reduce complexity if FPS drops below threshold
    if (currentFPS < 15 && !isPaused) {
      reduceSimulationComplexity();
    }
  }
}
```

### Efficient Geometry Creation

```javascript
// Create geometry with appropriate detail level based on importance
const tubeGeometry = new THREE.TubeGeometry(
  curve,
  Math.min(points.length * 2, 200), // Limit segments for performance
  trailRadius,
  8, // radial segments (reduce for better performance)
  false // closed
);
```

## Physics Calculation Optimizations

### Optimized Force Calculations

```javascript
// Only calculate forces between unique pairs of bodies
for (let i = 0; i < bodies.length; i++) {
  for (let j = i + 1; j < bodies.length; j++) {
    // Calculate forces between bodies i and j
    // This avoids redundant calculations
  }
}
```

### Limit Physics Steps for Complex Scenarios

```javascript
// Scale down force for very large mass products to prevent extreme calculations
if (massProduct > 100) {
  scaleFactor = 1.0 / Math.log10(massProduct / 10);
}
```

## Mobile-Specific Optimizations

### Reduce Visual Complexity on Mobile

```javascript
// Detect if on mobile and reduce initial complexity
const isMobile = /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(navigator.userAgent);

if (isMobile) {
  // Reduce particle count for starfield
  starCount = 5000; // Instead of 15000 for desktop
  
  // Use simpler materials
  bodyMaterial = new THREE.MeshBasicMaterial({ color: BODY_COLORS[i] });
  
  // Disable or simplify effects
  trailSegments = 50; // Reduced from 100
}
```

### Touch Event Optimization

```javascript
// Use passive event listeners for better scroll performance
element.addEventListener('touchstart', handler, { passive: true });

// Throttle or debounce touch move events
let lastTouchMove = 0;
element.addEventListener('touchmove', function(event) {
  const now = Date.now();
  if (now - lastTouchMove > 16) { // ~60fps
    // Handle touch move
    lastTouchMove = now;
  }
});
```

## General JavaScript Optimizations

### Use Efficient Data Structures

```javascript
// Use a fixed-size array for trails instead of continuously growing
// This avoids frequent reallocations
const trailArray = new Array(MAX_TRAIL_LENGTH);
```

### Avoid Reflows and Repaints

```javascript
// Batch DOM updates
function updateUI() {
  // Start batched updates
  document.body.style.display = 'none';
  
  // Make multiple DOM changes
  element1.style.width = '100px';
  element2.textContent = 'Updated';
  element3.classList.add('active');
  
  // End batched updates (forces single reflow)
  void document.body.offsetHeight; // Force reflow
  document.body.style.display = '';
}
```

### Use Object Pooling

```javascript
// Reuse vector objects instead of creating new ones
const tempVector = new THREE.Vector3();

function calculateDirection(sourceObj, targetObj) {
  // Reuse the tempVector instead of creating a new vector
  return tempVector.subVectors(targetObj.position, sourceObj.position).normalize();
}
```

## Three.js Specific Optimizations

### Texture Management

```javascript
// Use the Three.js texture loader with caching
const textureLoader = new THREE.TextureLoader();
const textureCache = {};

function loadTexture(url) {
  if (!textureCache[url]) {
    textureCache[url] = textureLoader.load(url);
  }
  return textureCache[url];
}
```

### Share Materials When Possible

```javascript
// Create a single material for multiple objects of the same type
const sharedMaterial = new THREE.MeshPhongMaterial({
  color: 0xffffff,
  shininess: 30
});

// Reuse for multiple objects
const mesh1 = new THREE.Mesh(geometry1, sharedMaterial);
const mesh2 = new THREE.Mesh(geometry2, sharedMaterial);
```

### Use Instanced Meshes for Repeated Objects

```javascript
// For many identical objects like stars
const instancedGeometry = new THREE.InstancedBufferGeometry().copy(starGeometry);
const instancedMaterial = new THREE.MeshPhongMaterial({ color: 0xffffff });
const instancedMesh = new THREE.InstancedMesh(
  instancedGeometry, instancedMaterial, starCount
);

// Set instance positions
for (let i = 0; i < starCount; i++) {
  const position = new THREE.Vector3(
    (Math.random() - 0.5) * 200,
    (Math.random() - 0.5) * 200,
    (Math.random() - 0.5) * 200
  );
  const matrix = new THREE.Matrix4().setPosition(position);
  instancedMesh.setMatrixAt(i, matrix);
}
```

## Monitoring and Profiling

### Use Performance Monitoring

```javascript
// Add performance monitoring
let lastFrameTime = performance.now();
let frameCount = 0;
let lastFPSUpdate = performance.now();
let currentFPS = 0;

function updateFPS() {
  const now = performance.now();
  const deltaTime = now - lastFrameTime;
  lastFrameTime = now;
  
  frameCount++;
  if (now - lastFPSUpdate >= 1000) {
    currentFPS = Math.round((frameCount * 1000) / (now - lastFPSUpdate));
    console.log(`FPS: ${currentFPS}, Frame time: ${deltaTime.toFixed(2)}ms`);
    frameCount = 0;
    lastFPSUpdate = now;
  }
}
```

### Check Memory Usage

```javascript
// Monitor memory usage if available
function checkMemoryUsage() {
  if (performance.memory) {
    const usedHeap = performance.memory.usedJSHeapSize / (1024 * 1024);
    console.log(`Memory usage: ${usedHeap.toFixed(2)} MB`);
    
    if (usedHeap > 500) { // 500MB threshold
      cleanupMemory();
    }
  }
}
```

## Implementation Strategy

When implementing these optimizations, follow this approach:

1. **Measure first** - Identify specific performance bottlenecks before optimizing
2. **Tackle memory leaks** - Fix memory leaks and disposals first
3. **Optimize rendering** - Reduce geometric complexity and material complexity
4. **Optimize physics calculations** - Streamline the most expensive calculations
5. **Add adaptive quality** - Implement systems that adjust detail based on performance
6. **Mobile-specific optimizations** - Apply special optimizations for mobile devices

By applying these performance optimization techniques, you can create a smoother, more responsive Three Body Simulator that works well across all devices. 