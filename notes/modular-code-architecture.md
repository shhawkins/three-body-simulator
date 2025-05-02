# Modular Code Architecture

This document outlines principles and techniques for creating a modular code architecture for the Three Body Simulator, which will make the code more maintainable, testable, and extensible.

## Why Modularize?

The current codebase places all functionality in a single HTML file with inline JavaScript. Modularizing offers several benefits:

1. **Improved readability** - Smaller, focused files are easier to understand
2. **Better maintainability** - Changes to one module won't break others
3. **Easier collaboration** - Different team members can work on different modules
4. **Reusability** - Modules can be reused in other projects
5. **Testability** - Isolated modules are easier to test individually

## Proposed Module Structure

Here's a proposed structure for modularizing the Three Body Simulator:

```
three-body-simulator/
│
├── index.html              # Main HTML file (minimal)
├── styles/                 # CSS styles
│   ├── main.css            # Core styles
│   ├── ui-components.css   # UI component styles
│   └── responsive.css      # Responsive design styles
│
├── js/
│   ├── main.js             # Entry point for the application
│   │
│   ├── core/               # Core simulation functionality
│   │   ├── scene.js        # Three.js scene setup
│   │   ├── physics.js      # Physics calculations
│   │   └── animation.js    # Animation loop
│   │
│   ├── entities/           # Simulation objects
│   │   ├── body.js         # Celestial body class
│   │   ├── trail.js        # Trail visualization
│   │   └── boundary.js     # Boundary system
│   │
│   ├── ui/                 # User interface components
│   │   ├── controls.js     # Main control panel
│   │   ├── body-panel.js   # Body control panel
│   │   ├── camera-panel.js # Camera control panel
│   │   └── help-panel.js   # Help/instructions panel
│   │
│   ├── controls/           # User interaction
│   │   ├── drag.js         # Drag controls for bodies
│   │   ├── camera.js       # Camera controls
│   │   └── touch.js        # Touch-specific controls
│   │
│   └── utils/              # Utility functions
│       ├── math.js         # Math utilities
│       ├── ui-helpers.js   # UI helper functions
│       └── performance.js  # Performance monitoring
```

## Module Design Principles

### 1. Single Responsibility Principle

Each module should have one specific responsibility:

```javascript
// BEFORE: Mixed responsibilities
function updatePhysics(deltaTime) {
  // Physics calculations
  // ...
  
  // UI updates
  document.getElementById('simulation-time').textContent = 'Time: ' + simulationTime.toFixed(2) + 's';
  
  // Trail visualization
  updateTrails();
}

// AFTER: Separated responsibilities
// physics.js
function updatePhysics(deltaTime, bodies) {
  // Only physics calculations
  return updatedBodies;
}

// ui.js
function updateUI(simulationTime) {
  document.getElementById('simulation-time').textContent = 'Time: ' + simulationTime.toFixed(2) + 's';
}

// trails.js
function updateTrails(bodies) {
  // Only trail visualization
}
```

### 2. Dependency Injection

Pass dependencies to functions rather than accessing globals:

```javascript
// BEFORE: Using globals
function createBodies() {
  for (let i = 0; i < 3; i++) {
    const size = getBodyRadius(masses[i]);
    // Uses global variables: scene, BODY_COLORS, etc.
    // ...
  }
}

// AFTER: Using dependency injection
function createBodies(options) {
  const { scene, bodyColors, initialPositions, masses } = options;
  
  const bodies = [];
  for (let i = 0; i < masses.length; i++) {
    const size = getBodyRadius(masses[i]);
    // Create bodies using passed parameters
    // ...
  }
  return bodies;
}
```

### 3. ES Modules

Use ES modules for clean imports/exports:

```javascript
// physics.js
export function calculateGravity(body1, body2, G) {
  // Gravity calculation
  return force;
}

// body.js
import { calculateGravity } from './physics.js';

export class CelestialBody {
  // Body implementation
  applyGravityFrom(otherBody) {
    const force = calculateGravity(this, otherBody, this.G);
    // Apply force
  }
}
```

### 4. Event-based Communication

Use custom events for communication between modules:

```javascript
// In UI module
document.getElementById('startButton').addEventListener('click', () => {
  const event = new CustomEvent('simulation:start');
  document.dispatchEvent(event);
});

// In physics module
document.addEventListener('simulation:start', () => {
  startPhysicsSimulation();
});
```

## Example Module Implementation

### Scene Module (scene.js)

```javascript
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

export class SceneManager {
  constructor(container, options = {}) {
    this.container = container;
    this.options = Object.assign({
      backgroundColor: 0x000000,
      aspectRatio: 16 / 9
    }, options);
    
    this.scene = new THREE.Scene();
    this.setupCamera();
    this.setupRenderer();
    this.setupLights();
    this.setupControls();
    
    // Handle window resize
    window.addEventListener('resize', this.handleResize.bind(this));
    
    // Initial resize
    this.handleResize();
  }
  
  setupCamera() {
    this.camera = new THREE.PerspectiveCamera(
      75, this.options.aspectRatio, 0.1, 1000
    );
    this.camera.position.set(0, 42, 50);
    this.camera.lookAt(0, 0, 0);
  }
  
  setupRenderer() {
    this.renderer = new THREE.WebGLRenderer({ antialias: true });
    this.renderer.setPixelRatio(window.devicePixelRatio);
    this.container.appendChild(this.renderer.domElement);
  }
  
  setupLights() {
    const ambientLight = new THREE.AmbientLight(0x404040);
    this.scene.add(ambientLight);
    
    const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8);
    directionalLight.position.set(0, 10, 10);
    this.scene.add(directionalLight);
  }
  
  setupControls() {
    this.controls = new OrbitControls(this.camera, this.renderer.domElement);
    this.controls.enableDamping = true;
    this.controls.dampingFactor = 0.05;
    this.controls.enabled = true;
  }
  
  handleResize() {
    const width = window.innerWidth;
    const height = window.innerHeight;
    
    // Maintain aspect ratio
    let newWidth, newHeight;
    if (width / height > this.options.aspectRatio) {
      newHeight = height;
      newWidth = height * this.options.aspectRatio;
    } else {
      newWidth = width;
      newHeight = width / this.options.aspectRatio;
    }
    
    this.camera.aspect = this.options.aspectRatio;
    this.camera.updateProjectionMatrix();
    this.renderer.setSize(newWidth, newHeight);
  }
  
  add(object) {
    this.scene.add(object);
  }
  
  remove(object) {
    this.scene.remove(object);
  }
  
  render() {
    this.controls.update();
    this.renderer.render(this.scene, this.camera);
  }
}
```

### Physics Module (physics.js)

```javascript
import * as THREE from 'three';

export class PhysicsEngine {
  constructor(options = {}) {
    this.options = Object.assign({
      G: 0.3,
      boundaryRadius: 60,
      simulationSpeed: 0.3,
      freePlayMode: false
    }, options);
    
    this.simulationTime = 0;
    this.bodies = [];
    this.collisionOccurred = false;
  }
  
  addBody(body) {
    this.bodies.push(body);
    return this;
  }
  
  setFreePlayMode(enabled) {
    this.options.freePlayMode = enabled;
  }
  
  update(deltaTime) {
    if (this.collisionOccurred) return;
    
    // Update simulation time
    this.simulationTime += deltaTime;
    
    // Calculate forces between all unique pairs of bodies
    for (let i = 0; i < this.bodies.length; i++) {
      for (let j = i + 1; j < this.bodies.length; j++) {
        // Check for collision
        const body1 = this.bodies[i];
        const body2 = this.bodies[j];
        
        if (this.checkCollision(body1, body2)) {
          return;
        }
        
        // Calculate and apply gravitational forces
        this.applyGravitationalForce(body1, body2, deltaTime);
      }
    }
    
    // Update positions based on velocities
    for (let i = 0; i < this.bodies.length; i++) {
      const body = this.bodies[i];
      
      // Update position
      body.position.add(body.velocity.clone().multiplyScalar(deltaTime));
      
      // Check boundary only if free play mode is not active
      if (!this.options.freePlayMode) {
        if (this.checkBoundaryViolation(body)) {
          return;
        }
      }
      
      // Record trail point
      body.addTrailPoint();
    }
    
    return {
      bodies: this.bodies,
      simulationTime: this.simulationTime,
      collisionOccurred: this.collisionOccurred
    };
  }
  
  applyGravitationalForce(body1, body2, deltaTime) {
    // Calculate direction vector
    const direction = new THREE.Vector3().subVectors(
      body2.position, body1.position
    );
    const distanceSquared = direction.lengthSq();
    
    if (distanceSquared > 0) {
      // Calculate force magnitude
      const massProduct = body1.mass * body2.mass;
      let scaleFactor = 1.0;
      
      // Scale down force for very large mass products
      if (massProduct > 100) {
        scaleFactor = 1.0 / Math.log10(massProduct / 10);
      }
      
      const forceMagnitude = this.options.G * massProduct / distanceSquared * scaleFactor;
      const force = direction.normalize().multiplyScalar(forceMagnitude);
      
      // Apply forces (F = ma, so a = F/m)
      const acc1 = force.clone().divideScalar(body1.mass);
      const acc2 = force.clone().negate().divideScalar(body2.mass);
      
      body1.velocity.add(acc1.multiplyScalar(deltaTime));
      body2.velocity.add(acc2.multiplyScalar(deltaTime));
    }
  }
  
  checkCollision(body1, body2) {
    const distance = body1.position.distanceTo(body2.position);
    if (distance < (body1.radius + body2.radius)) {
      this.collisionOccurred = true;
      this.collisionPoint = new THREE.Vector3().addVectors(
        body1.position, body2.position
      ).multiplyScalar(0.5);
      return true;
    }
    return false;
  }
  
  checkBoundaryViolation(body) {
    const distanceFromCenter = Math.sqrt(
      body.position.x * body.position.x + 
      body.position.z * body.position.z
    );
    
    if (distanceFromCenter + body.radius > this.options.boundaryRadius) {
      this.collisionOccurred = true;
      this.boundaryViolationPoint = body.position.clone();
      return true;
    }
    return false;
  }
  
  reset() {
    this.simulationTime = 0;
    this.collisionOccurred = false;
    
    // Reset all bodies to initial state
    this.bodies.forEach(body => body.reset());
  }
}
```

### Body Module (body.js)

```javascript
import * as THREE from 'three';

export class CelestialBody {
  constructor(options = {}) {
    this.options = Object.assign({
      mass: 1,
      position: new THREE.Vector3(0, 0, 0),
      velocity: new THREE.Vector3(0, 0, 0),
      color: 0xffffff,
      trailColor: 0xffffff,
      maxTrailLength: 5000
    }, options);
    
    this.mass = this.options.mass;
    this.position = this.options.position.clone();
    this.velocity = this.options.velocity.clone();
    this.color = this.options.color;
    this.trailColor = this.options.trailColor;
    
    this.initialPosition = this.options.position.clone();
    this.initialVelocity = this.options.velocity.clone();
    
    this.trail = [];
    this.maxTrailLength = this.options.maxTrailLength;
    
    // Create the visual representation
    this.createMesh();
  }
  
  createMesh() {
    this.radius = this.getRadiusFromMass();
    
    // Create sphere geometry
    const geometry = new THREE.SphereGeometry(this.radius, 32, 32);
    const material = new THREE.MeshPhongMaterial({
      color: this.color,
      shininess: 30,
      emissive: new THREE.Color(this.color).multiplyScalar(0.2)
    });
    
    this.mesh = new THREE.Mesh(geometry, material);
    this.mesh.position.copy(this.position);
  }
  
  getRadiusFromMass() {
    // Logarithmic scaling for visual size
    if (this.mass <= 1) {
      return Math.max(0.5, Math.pow(this.mass, 1/3) * 0.45);
    } else {
      return 0.45 + 0.35 * Math.log10(this.mass);
    }
  }
  
  updateMass(mass) {
    this.mass = mass;
    
    // Update visual size
    const newRadius = this.getRadiusFromMass();
    
    // Remove old mesh
    const parent = this.mesh.parent;
    if (parent) parent.remove(this.mesh);
    
    // Create new mesh with updated size
    this.createMesh();
    
    // Re-add to parent if it existed
    if (parent) parent.add(this.mesh);
  }
  
  addTrailPoint() {
    this.trail.push(this.position.clone());
    
    // Limit trail length
    if (this.trail.length > this.maxTrailLength) {
      this.trail = this.trail.slice(-this.maxTrailLength);
    }
  }
  
  clearTrail() {
    this.trail = [this.position.clone()];
  }
  
  update() {
    // Update mesh position to match physics position
    this.mesh.position.copy(this.position);
  }
  
  reset() {
    this.position.copy(this.initialPosition);
    this.velocity.copy(this.initialVelocity);
    this.clearTrail();
    this.update();
  }
}
```

## Integrating with Main Application

The main application ties all modules together:

```javascript
// main.js
import { SceneManager } from './core/scene.js';
import { PhysicsEngine } from './core/physics.js';
import { CelestialBody } from './entities/body.js';
import { TrailManager } from './entities/trail.js';
import { BoundarySystem } from './entities/boundary.js';
import { ControlPanel } from './ui/controls.js';
import { BodyPanel } from './ui/body-panel.js';
import { CameraPanel } from './ui/camera-panel.js';

class ThreeBodySimulator {
  constructor() {
    // Setup scene
    this.scene = new SceneManager(
      document.getElementById('container'),
      { aspectRatio: 16/9 }
    );
    
    // Setup physics
    this.physics = new PhysicsEngine({
      G: 0.3,
      boundaryRadius: 60
    });
    
    // Setup trails
    this.trails = new TrailManager(this.scene);
    
    // Setup boundary
    this.boundary = new BoundarySystem(this.scene, {
      radius: 60
    });
    
    // Create celestial bodies
    this.createBodies();
    
    // Setup UI
    this.setupUI();
    
    // Setup animation loop
    this.animating = false;
    this.paused = false;
    this.setupAnimationLoop();
  }
  
  createBodies() {
    // Body 1
    const body1 = new CelestialBody({
      mass: 1.0,
      position: new THREE.Vector3(-5, 0, 0),
      velocity: new THREE.Vector3(0, 0, 0.42),
      color: 0xd8a0ff,
      trailColor: 0xd8a0ff
    });
    
    // Body 2
    const body2 = new CelestialBody({
      mass: 1.0,
      position: new THREE.Vector3(5, 0, 0),
      velocity: new THREE.Vector3(0, 0, -0.42),
      color: 0xffb6c1,
      trailColor: 0xffb6c1
    });
    
    // Body 3
    const body3 = new CelestialBody({
      mass: 1.5,
      position: new THREE.Vector3(0, 0, 0),
      velocity: new THREE.Vector3(0, 0, 0),
      color: 0x87cefa,
      trailColor: 0x87cefa
    });
    
    // Add bodies to physics engine
    this.physics.addBody(body1).addBody(body2).addBody(body3);
    
    // Add meshes to scene
    this.scene.add(body1.mesh);
    this.scene.add(body2.mesh);
    this.scene.add(body3.mesh);
    
    // Store bodies array
    this.bodies = [body1, body2, body3];
  }
  
  setupUI() {
    // Main control panel
    this.controlPanel = new ControlPanel({
      onStart: () => this.startSimulation(),
      onStop: () => this.stopSimulation(),
      onPause: () => this.pauseSimulation(),
      onReset: () => this.resetSimulation(),
      onToggleTrails: (enabled) => this.toggleTrails(enabled),
      onToggleBoundary: (enabled) => this.toggleBoundary(enabled),
      onSpeedChange: (speed) => this.setSimulationSpeed(speed)
    });
    
    // Body control panel
    this.bodyPanel = new BodyPanel({
      bodies: this.bodies,
      onMassChange: (bodyIndex, mass) => this.updateBodyMass(bodyIndex, mass),
      onVelocityChange: (bodyIndex, velocity) => this.updateBodyVelocity(bodyIndex, velocity)
    });
    
    // Camera control panel
    this.cameraPanel = new CameraPanel({
      scene: this.scene,
      onViewChange: (viewType) => this.changeView(viewType),
      onFollowBody: (bodyIndex) => this.followBody(bodyIndex)
    });
  }
  
  setupAnimationLoop() {
    const animate = () => {
      requestAnimationFrame(animate);
      
      if (this.animating && !this.paused) {
        // Update physics
        const result = this.physics.update(this.currentSpeed);
        
        // Update visual positions
        this.bodies.forEach(body => body.update());
        
        // Update trails
        this.trails.update(this.bodies);
        
        // Check for collisions
        if (result && result.collisionOccurred) {
          this.handleCollision();
        }
      }
      
      // Always render scene
      this.scene.render();
    };
    
    animate();
  }
  
  startSimulation() {
    if (this.animating) return;
    
    this.animating = true;
    this.paused = false;
    
    // Additional start logic...
    
    // Save initial positions for reset
    this.bodies.forEach(body => {
      body.initialPosition.copy(body.position);
      body.initialVelocity.copy(body.velocity);
    });
  }
  
  stopSimulation() {
    this.animating = false;
    this.paused = false;
    
    // Additional stop logic...
  }
  
  // Other methods...
}

// Initialize application
document.addEventListener('DOMContentLoaded', () => {
  window.simulator = new ThreeBodySimulator();
});
```

## Migration Strategy

To migrate from the current monolithic structure to this modular architecture:

1. **Create the folder structure** first
2. **Extract one module at a time**, starting with the most independent ones
3. **Test each module** as you extract it
4. **Refactor as you go**, improving code quality
5. **Update the main application** to use the new modules
6. **Remove code from the original file** as it's migrated

By following these modular architecture principles, the Three Body Simulator will become more maintainable, extensible, and easier to develop further. 