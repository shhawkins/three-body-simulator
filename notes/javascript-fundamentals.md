# JavaScript Fundamentals for Three Body Simulator

This document covers the core JavaScript concepts used in our Three Body Simulator project.

## Variables and Data Types

### Variable Declarations

```javascript
// Constants (cannot be reassigned)
const BOUNDARY_RADIUS = 60;
const G = 0.3;

// Variables that can change
let simulationSpeed = 0.3;
let freePlayMode = false;
```

### Common Data Types

```javascript
// Numbers
const mass = 1.5;

// Strings
const statusMessage = 'Running';

// Booleans
let isAnimating = false;

// Arrays
const masses = [1, 1, 1];

// Objects
const body = {
  position: new THREE.Vector3(0, 0, 0),
  velocity: new THREE.Vector3(0, 0, 0.5),
  mass: 1
};
```

## Functions

### Function Declarations

```javascript
// Regular function
function updatePhysics(deltaTime) {
  // Function body
}

// Arrow function
const updateFPS = () => {
  frameCount++;
  // More code...
};
```

### Function Parameters

```javascript
// Parameters with default values
function sliderValueToMass(sliderValue) {
  if (sliderValue <= 0) return 0.1; // Minimum mass
  
  // Function logic...
}
```

## DOM Manipulation

### Selecting Elements

```javascript
// Get element by ID
const startButton = document.getElementById('startButton');

// Query selector (CSS-style selection)
const buttons = document.querySelectorAll('.speed-button');
```

### Modifying Elements

```javascript
// Change text content
document.getElementById('pauseButton').textContent = 'Play';

// Change CSS styles
document.body.style.cursor = 'grab';

// Change CSS classes
freePlayButton.classList.toggle('active');

// Show/hide elements
document.getElementById('resetButtonRow').style.display = 'flex';
```

## Event Handling

```javascript
// Add event listener
document.getElementById('startButton').addEventListener('click', startSimulation);

// Event handler with event object
currentDragControls.addEventListener('dragstart', function(event) {
  controls.enabled = false;
  document.body.style.cursor = 'grabbing';
  hideTooltip();
});
```

## Loops and Iterations

```javascript
// For loop
for (let i = 0; i < bodies.length; i++) {
  // Do something with bodies[i]
}

// For...of loop (iterating over array values)
for (let body of bodies) {
  // Do something with body
}

// forEach (array method with callback)
bodies.forEach(body => {
  body.position.add(body.velocity);
});
```

## Conditionals

```javascript
// If-else statement
if (isAnimating && !isPaused) {
  updatePhysics(simulationSpeed);
} else {
  // Do something else
}

// Ternary operator (compact if-else)
const message = isPaused ? 'Paused' : 'Running';
```

## Object-Oriented Techniques

### Working with Objects

```javascript
// Object creation
const body = {
  position: new THREE.Vector3(-5, 0, 0),
  velocity: new THREE.Vector3(0, 0, 0.5),
  mass: 1,
  trail: []
};

// Adding methods to objects
body.updatePosition = function(deltaTime) {
  this.position.add(this.velocity.clone().multiplyScalar(deltaTime));
};

// Accessing properties
body.position.x = 5;
```

## Asynchronous JavaScript

### requestAnimationFrame

Used for smooth animations:

```javascript
function animate() {
  requestAnimationFrame(animate);
  
  // Update and render code here
  
  renderer.render(scene, camera);
}

// Start the animation loop
animate();
```

### setTimeout

Used for delayed actions:

```javascript
setTimeout(() => {
  boundaryPlane.material.color.copy(originalColor);
}, 500);
```

## Modules and Imports

The project uses ES modules for organizing code:

```javascript
// Import statements
import * as THREE from "three";
import { OrbitControls } from "three/addons/controls/OrbitControls.js";
import { DragControls } from "three/addons/controls/DragControls.js";
import _ from "lodash";
import { gsap } from "gsap";
```

## Working with Math

```javascript
// Math operations
const distance = Math.sqrt(
  position.x * position.x + position.z * position.z
);

// Using Math object for constants and functions
const angle = Math.atan2(position.z, position.x);
position.x = BOUNDARY_RADIUS * Math.cos(angle);

// Converting radians to degrees
const angleDegrees = angleRadians * (180 / Math.PI);
```

## Tips for JavaScript in Simulation Projects

1. **Use descriptive variable names** - Especially important in physics simulations
2. **Separate logic from presentation** - Keep physics calculations separate from UI updates
3. **Optimize performance-critical code** - Physics calculations need to be efficient
4. **Use constants for configuration** - Makes it easier to adjust parameters
5. **Add comments for complex math** - Physics formulas often need explanation

Understanding these JavaScript fundamentals will help you navigate and modify the Three Body Simulator codebase. 