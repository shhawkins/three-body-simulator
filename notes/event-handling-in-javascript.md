# Event Handling in JavaScript

This document explains how event handling works in JavaScript, with examples from our Three Body Simulator project.

## Basic Event Handling Concepts

Events are actions or occurrences that happen in the browser, like clicks, key presses, or mouse movements. JavaScript allows us to detect and respond to these events.

### The Event Listener Pattern

```javascript
// Basic pattern for adding an event listener
element.addEventListener(eventType, eventHandlerFunction);

// Example from our project
document.getElementById('startButton').addEventListener('click', startSimulation);
```

## Common Events Used in Our Project

### Mouse Events

```javascript
// Click events for buttons
document.getElementById('stopButton').addEventListener('click', stopSimulation);

// Mouse movement for tooltips
slider.addEventListener('mousemove', function(event) {
  if (this.matches(':hover')) {
    showSliderTooltip(tooltipText, event, this);
  }
});

// Mouse down/up for dragging
slider.addEventListener('mousedown', function() {
  this.isActive = true;
});

slider.addEventListener('mouseup', function() {
  this.isActive = false;
});
```

### Touch Events for Mobile

```javascript
// Touch start for handling first touch
slider.addEventListener('touchstart', function(event) {
  this.isActive = true;
  showSliderTooltip(tooltipText, event, this);
});

// Touch move for tracking finger movement
slider.addEventListener('touchmove', function(event) {
  showSliderTooltip(tooltipText, event, this);
});

// Touch end for completing the interaction
slider.addEventListener('touchend', function() {
  this.isActive = false;
  hideTooltip();
});
```

### Input Events

```javascript
// Triggered when input value changes (e.g., sliders)
slider.addEventListener('input', function() {
  const sliderValue = parseInt(this.value);
  const massValue = sliderValueToMass(sliderValue);
  document.getElementById(`body${bodyIndex+1}MassValue`).textContent = formatMassValue(massValue);
  updateBodyMass(bodyIndex, massValue);
});
```

### Window Events

```javascript
// Resize events to handle window size changes
window.addEventListener('resize', handleWindowResize);

// Orientation change for mobile devices
window.addEventListener('orientationchange', function() {
  setTimeout(() => {
    handleWindowResize();
    // Additional iPad-specific adjustments
    // ...
  }, 300);
});
```

## Three.js-Specific Events

### OrbitControls Events

OrbitControls, which handles camera movement, has its own events:

```javascript
// Enable or disable controls
controls.enabled = true;

// Control properties affect how events are handled
controls.enableZoom = true;
controls.enableRotate = true;
controls.enablePan = false;
```

### DragControls Events

DragControls, which allows dragging 3D objects, provides these events:

```javascript
// Setup drag controls
const dragControls = new DragControls(draggableObjects, camera, renderer.domElement);

// Hovering over draggable object
dragControls.addEventListener('hoveron', function(event) {
  document.body.style.cursor = 'grab';
  showTooltip('Drag to position the body', event);
});

// Starting to drag an object
dragControls.addEventListener('dragstart', function(event) {
  controls.enabled = false; // Disable orbit controls during drag
  document.body.style.cursor = 'grabbing';
});

// During the drag operation
dragControls.addEventListener('drag', function(event) {
  const objectType = event.object.userData.type;
  const bodyIndex = event.object.userData.bodyIndex;
  
  // Custom logic for handling different object types
  if (objectType === 'body') {
    // Body dragging logic
  } else if (objectType === 'velocityArrow') {
    // Velocity arrow dragging logic
  }
});

// Ending the drag operation
dragControls.addEventListener('dragend', function(event) {
  controls.enabled = true; // Re-enable orbit controls
  document.body.style.cursor = 'auto';
});
```

## The Event Object

Many event handlers receive an event object that contains information about the event:

```javascript
element.addEventListener('click', function(event) {
  // event.target: The element that triggered the event
  console.log('Clicked element:', event.target);
  
  // event.clientX and event.clientY: Mouse coordinates
  console.log('Mouse position:', event.clientX, event.clientY);
  
  // Prevent default behavior (e.g., form submission)
  event.preventDefault();
  
  // Stop event propagation to parent elements
  event.stopPropagation();
});
```

## Event Delegation

Rather than adding event listeners to many elements, you can use event delegation by adding a listener to a parent element:

```javascript
// Add one listener to the parent instead of many child listeners
document.querySelector('.button-row').addEventListener('click', function(event) {
  // Check if the clicked element is a button
  if (event.target.matches('.button')) {
    // Handle the button click
    const buttonId = event.target.id;
    // Do something based on which button was clicked
  }
});
```

## Handling Touch vs. Mouse Events

For a responsive application that works on both desktop and mobile:

```javascript
// Detect if we're on a touch device
const isTouchDevice = 'ontouchstart' in window || navigator.maxTouchPoints > 0;

// Adjust controls based on device type
if (isTouchDevice) {
  controls.rotateSpeed = 0.7; // Slower rotation for more precision on touch
  controls.zoomSpeed = 1.2;   // Slightly faster zoom on touch
  
  // Special handling for multi-touch
  controls.touchStart = function(event) {
    if (event.touches.length === 2) {
      // Two-finger touch - force zoom instead of rotate
      this.handleTouchStartDolly(event);
    } else {
      // Single-finger touch - normal behavior
      this.handleTouchStartRotate(event);
    }
  };
}
```

## Animation Frame Events

For smooth animations, the project uses requestAnimationFrame:

```javascript
function animate() {
  // Request the next frame
  requestAnimationFrame(animate);
  
  // Only update physics if animation is running and not paused
  if (isAnimating && !isPaused) {
    updatePhysics(simulationSpeed);
  }
  
  // Always update controls and render
  controls.update();
  renderer.render(scene, camera);
}

// Start the animation loop
animate();
```

## Best Practices for Event Handling

1. **Clean up event listeners** when they're no longer needed
2. **Debounce or throttle** events that fire rapidly (like resize or scroll)
3. **Use event delegation** where appropriate
4. **Maintain UI state** separate from event handlers
5. **Provide feedback** to users when events are processed

Understanding event handling is crucial for creating interactive web applications like our Three Body Simulator. 