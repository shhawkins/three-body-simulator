# Performance Optimization Guide for Three Body Simulator

This document provides specific techniques to optimize the Three Body Simulator for smooth performance across devices.

## Current Performance Issues

Based on code review, the simulator suffers from several performance bottlenecks:

1. **Inefficient Trail Rendering**: Continuously growing trail arrays without limits
2. **Unnecessary Geometry Recreation**: Creating new geometries in animation loops
3. **Unoptimized Physics Calculations**: Computing forces between all bodies regardless of distance
4. **DOM Manipulation During Animation**: UI updates during the render loop
5. **Memory Leaks**: Failure to dispose of Three.js objects properly

## Optimization Strategies

### 1. Rendering Optimizations

#### Trail Rendering

**Current Implementation:**
```javascript
// Example of problematic code
function updateTrail(body) {
  body.trail.push(body.position.clone());
  
  // Create new line geometry on each update
  const geometry = new THREE.BufferGeometry();
  geometry.setFromPoints(body.trail);
  
  // Replace old line with new one
  scene.remove(body.trailLine);
  body.trailLine = new THREE.Line(geometry, body.trailMaterial);
  scene.add(body.trailLine);
}
```

**Optimized Approach:**
```javascript
// Initialize once
function initTrail(body, maxPoints = 5000) {
  // Pre-allocate array of positions
  const positions = new Float32Array(maxPoints * 3);
  
  // Create buffer geometry with allocated buffer
  const geometry = new THREE.BufferGeometry();
  geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
  
  // Set initial draw range
  geometry.setDrawRange(0, 0);
  
  body.trailLine = new THREE.Line(geometry, body.trailMaterial);
  body.trailPositions = positions;
  body.trailPointCount = 0;
  body.maxTrailPoints = maxPoints;
  
  scene.add(body.trailLine);
}

// Update efficiently
function updateTrail(body) {
  const positions = body.trailPositions;
  const geometry = body.trailLine.geometry;
  
  // Add new point - handle wrapping in circular buffer
  const index = body.trailPointCount % body.maxTrailPoints;
  const posIndex = index * 3;
  
  positions[posIndex] = body.position.x;
  positions[posIndex + 1] = body.position.y;
  positions[posIndex + 2] = body.position.z;
  
  body.trailPointCount++;
  
  // Update the part of the buffer that has data
  const drawCount = Math.min(body.trailPointCount, body.maxTrailPoints);
  geometry.setDrawRange(0, drawCount);
  
  // Flag the attribute for update
  geometry.attributes.position.needsUpdate = true;
}
```

#### Instancing for Multiple Bodies

If displaying many similar bodies:

```javascript
function createInstancedBodies(count, radius) {
  // Create shared geometry
  const geometry = new THREE.SphereGeometry(radius, 16, 16);
  
  // Create instanced mesh
  const material = new THREE.MeshPhongMaterial();
  const instancedMesh = new THREE.InstancedMesh(geometry, material, count);
  
  // Initial positions and colors
  const dummy = new THREE.Object3D();
  for (let i = 0; i < count; i++) {
    // Set position and orientation
    dummy.position.set(positions[i].x, positions[i].y, positions[i].z);
    dummy.updateMatrix();
    instancedMesh.setMatrixAt(i, dummy.matrix);
    
    // Set color
    instancedMesh.setColorAt(i, new THREE.Color(colors[i]));
  }
  
  scene.add(instancedMesh);
  return instancedMesh;
}

// Update positions
function updateInstancedBodies(instancedMesh, positions) {
  const dummy = new THREE.Object3D();
  
  for (let i = 0; i < positions.length; i++) {
    dummy.position.copy(positions[i]);
    dummy.updateMatrix();
    instancedMesh.setMatrixAt(i, dummy.matrix);
  }
  
  instancedMesh.instanceMatrix.needsUpdate = true;
}
```

### 2. Physics Calculation Optimizations

#### Space Partitioning

For systems with many bodies, implement spatial partitioning:

```javascript
function calculateForces(bodies, deltaTime) {
  const sectors = {};
  
  // Assign bodies to sectors
  bodies.forEach(body => {
    const sectorKey = getSectorKey(body.position, sectorSize);
    if (!sectors[sectorKey]) sectors[sectorKey] = [];
    sectors[sectorKey].push(body);
  });
  
  // Calculate forces only between potentially interacting bodies
  bodies.forEach(body1 => {
    const nearbySectors = getNearbySectorKeys(body1.position, sectorSize);
    
    nearbySectors.forEach(sectorKey => {
      if (!sectors[sectorKey]) return;
      
      sectors[sectorKey].forEach(body2 => {
        if (body1 === body2) return;
        
        // Calculate and apply gravitational force
        applyGravitationalForce(body1, body2, deltaTime);
      });
    });
  });
}
```

#### Web Workers for Physics Calculations

Move physics calculations off the main thread:

```javascript
// In main thread
const physicsWorker = new Worker('physics-worker.js');

// Send body data to worker
physicsWorker.postMessage({
  type: 'update',
  bodies: bodies.map(b => ({
    id: b.id,
    mass: b.mass,
    position: [b.position.x, b.position.y, b.position.z],
    velocity: [b.velocity.x, b.velocity.y, b.velocity.z]
  })),
  deltaTime: deltaTime,
  gravityConstant: G
});

// Receive updated positions
physicsWorker.onmessage = function(e) {
  if (e.data.type === 'updated') {
    // Update positions in the scene
    e.data.bodies.forEach(bodyData => {
      const body = bodiesById[bodyData.id];
      body.position.set(bodyData.position[0], bodyData.position[1], bodyData.position[2]);
      body.velocity.set(bodyData.velocity[0], bodyData.velocity[1], bodyData.velocity[2]);
    });
    
    // Render scene with updated positions
    renderer.render(scene, camera);
  }
};

// In physics-worker.js
self.onmessage = function(e) {
  if (e.data.type === 'update') {
    const { bodies, deltaTime, gravityConstant } = e.data;
    
    // Perform physics calculations
    updatePhysics(bodies, deltaTime, gravityConstant);
    
    // Send updated bodies back to main thread
    self.postMessage({
      type: 'updated',
      bodies: bodies
    });
  }
};
```

#### Adaptive Time Stepping

```javascript
function simulateWithAdaptiveTimeStep(totalDeltaTime, maxSubSteps = 10) {
  // Don't let simulation get too far behind
  const maxDeltaTime = 1/30; // Max 1/30th of a second per frame
  const clampedDeltaTime = Math.min(totalDeltaTime, maxDeltaTime);
  
  // Physics timestep (fixed for stability)
  const fixedTimestep = 1/120; 
  
  // Number of substeps needed
  const numSubSteps = Math.floor(clampedDeltaTime / fixedTimestep);
  
  // Don't do too many steps at once
  const steps = Math.min(numSubSteps, maxSubSteps);
  
  // Remaining time after substeps
  let remainingTime = clampedDeltaTime - (steps * fixedTimestep);
  
  // Perform fixed timestep updates
  for (let i = 0; i < steps; i++) {
    updatePhysics(fixedTimestep);
  }
  
  // Handle remaining time with a smaller step
  if (remainingTime > 1e-6) {
    updatePhysics(remainingTime);
  }
}
```

### 3. Memory Optimization

#### Proper Disposal of Three.js Objects

```javascript
function cleanupObject(object) {
  if (!object) return;
  
  // Dispose of geometries and materials
  if (object.geometry) {
    object.geometry.dispose();
  }
  
  if (object.material) {
    if (Array.isArray(object.material)) {
      object.material.forEach(material => material.dispose());
    } else {
      object.material.dispose();
    }
  }
  
  // Remove from parent
  if (object.parent) {
    object.parent.remove(object);
  }
}

function resetSimulation() {
  // Clean up existing objects
  bodies.forEach(body => {
    cleanupObject(body.mesh);
    cleanupObject(body.trailLine);
  });
  
  // Create new bodies
  initBodies();
}
```

#### Object Pooling

```javascript
class ObjectPool {
  constructor(createFn, initialSize = 10) {
    this.createObject = createFn;
    this.pool = [];
    
    // Pre-fill pool
    for (let i = 0; i < initialSize; i++) {
      this.pool.push(this.createObject());
    }
  }
  
  get() {
    if (this.pool.length > 0) {
      return this.pool.pop();
    }
    return this.createObject();
  }
  
  release(object) {
    this.pool.push(object);
  }
}

// Example usage for particle effects
const particlePool = new ObjectPool(() => {
  const geometry = new THREE.SphereGeometry(0.1, 8, 8);
  const material = new THREE.MeshBasicMaterial({ color: 0xffffff });
  return new THREE.Mesh(geometry, material);
}, 50);

function createExplosion(position) {
  const particles = [];
  
  for (let i = 0; i < 20; i++) {
    const particle = particlePool.get();
    particle.position.copy(position);
    scene.add(particle);
    particles.push(particle);
  }
  
  // After animation completes
  particles.forEach(particle => {
    scene.remove(particle);
    particlePool.release(particle);
  });
}
```

### 4. Rendering Pipeline Optimizations

#### Use RequestAnimationFrame Properly

```javascript
let lastTime = 0;

function animate(timestamp) {
  requestAnimationFrame(animate);
  
  // Calculate actual elapsed time
  const deltaTime = lastTime === 0 ? 0 : (timestamp - lastTime) / 1000;
  lastTime = timestamp;
  
  // Skip if tab is inactive or delta is too large
  if (deltaTime === 0 || deltaTime > 1) return;
  
  // Update physics with proper timestep
  if (isSimulationRunning && !isPaused) {
    updatePhysics(deltaTime * simulationSpeed);
  }
  
  // Update camera controls
  controls.update();
  
  // Render scene
  renderer.render(scene, camera);
  
  // Performance monitoring
  updatePerformanceStats();
}
```

#### Level of Detail (LOD)

```javascript
function createBodyWithLOD(position, mass, color) {
  // High detail geometry
  const highDetailGeometry = new THREE.SphereGeometry(radius, 32, 24);
  const mediumDetailGeometry = new THREE.SphereGeometry(radius, 16, 12);
  const lowDetailGeometry = new THREE.SphereGeometry(radius, 8, 6);
  
  const material = new THREE.MeshPhongMaterial({ color });
  
  // Create LOD object
  const lod = new THREE.LOD();
  
  // Add detail levels
  lod.addLevel(new THREE.Mesh(highDetailGeometry, material), 0);     // Up close
  lod.addLevel(new THREE.Mesh(mediumDetailGeometry, material), 10);  // Medium distance
  lod.addLevel(new THREE.Mesh(lowDetailGeometry, material), 50);     // Far away
  
  lod.position.copy(position);
  
  return lod;
}
```

### 5. Performance Monitoring

```javascript
// Add Stats.js for performance monitoring
import Stats from 'stats.js';

const stats = new Stats();
stats.showPanel(0); // 0: fps, 1: ms, 2: mb
document.body.appendChild(stats.dom);

function animate() {
  stats.begin();
  
  // Simulation and rendering code
  
  stats.end();
  requestAnimationFrame(animate);
}
```

### 6. Mobile-Specific Optimizations

```javascript
function detectDeviceCapabilities() {
  // Check for low-end devices
  const isLowEnd = window.navigator.hardwareConcurrency <= 2 || 
                  /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(navigator.userAgent);
                  
  if (isLowEnd) {
    // Reduce quality settings
    renderer.setPixelRatio(1);
    renderer.setSize(window.innerWidth * 0.8, window.innerHeight * 0.8);
    
    // Simplify geometries
    bodyGeometryDetail = { segments: 16 };
    maxTrailPoints = 200;
    
    // Disable effects
    enableParticles = false;
    enablePostProcessing = false;
  } else {
    // Higher quality for powerful devices
    renderer.setPixelRatio(window.devicePixelRatio);
    bodyGeometryDetail = { segments: 32 };
    maxTrailPoints = 2000;
  }
}
```

## Implementation Priority

1. **High Priority (Immediate Wins)**
   - Fixed-size trail buffers
   - Proper object disposal
   - Request animation frame optimization
   - Performance monitoring

2. **Medium Priority**
   - Web Workers for physics
   - Adaptive time stepping
   - Mobile optimizations

3. **Lower Priority / Advanced Techniques**
   - Instanced rendering
   - Space partitioning
   - Level of Detail

## Benchmarking Strategy

To ensure optimizations actually improve performance:

1. Define baseline metrics:
   - Average FPS
   - Frame time (ms)
   - Memory usage
   - CPU usage

2. Create test scenarios:
   - Few bodies (3-10)
   - Many bodies (50+)
   - Long trails vs short trails
   - With and without effects

3. Run benchmarks before and after each optimization

4. Document improvements and any regressions

## Conclusion

By implementing these optimizations incrementally and measuring their impact, the Three Body Simulator can achieve smooth performance across a wide range of devices while maintaining visual quality. Start with the high-priority optimizations that address the most critical performance bottlenecks, then move to more advanced techniques as needed. 