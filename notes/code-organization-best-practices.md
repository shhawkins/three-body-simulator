# Code Organization Best Practices

This document outlines best practices for organizing JavaScript code in web applications like our Three Body Simulator.

## Project Structure Principles

### 1. Separation of Concerns

Separate different aspects of your application:

- **HTML**: Structure and content
- **CSS**: Presentation and styling
- **JavaScript**: Behavior and logic

```
project/
├── index.html          # Minimal HTML structure
├── css/                # Styling
├── js/                 # JavaScript code
└── assets/             # Images, fonts, etc.
```

### 2. Modular File Organization

Group related files together based on functionality:

```
js/
├── core/               # Application core
├── features/           # Feature-specific code
├── components/         # Reusable components
├── utils/              # Utility functions
└── vendor/             # Third-party libraries
```

### 3. Consistent Naming Conventions

- Use **camelCase** for variables, functions, and methods
- Use **PascalCase** for classes and constructors
- Use **kebab-case** for file names and CSS classes
- Use **UPPER_SNAKE_CASE** for constants

```javascript
// Constants
const MAX_PARTICLES = 1000;

// Classes
class PhysicsEngine { ... }

// Functions and variables
function calculateGravity() { ... }
const bodyPosition = new Vector3();

// File names
// physics-engine.js
// drag-controls.js
```

## Code Organization Within Files

### 1. File Structure Template

Organize each file with a consistent pattern:

```javascript
/**
 * @fileoverview Description of file purpose
 * @author Your Name
 */

// Imports
import { Something } from './somewhere.js';

// Constants
const DEFAULT_VALUE = 42;

// Helper functions
function helperFunction() { ... }

// Main class or functionality
export class MyClass {
  // Properties
  // Constructor
  // Methods
}

// Exports
export { helperFunction };
```

### 2. Grouping Related Code

Group related code together within a file:

```javascript
// Group 1: Initialization
function initialize() { ... }
function setupEvents() { ... }

// Group 2: Event handlers
function handleClick() { ... }
function handleHover() { ... }

// Group 3: Core functionality
function performCalculation() { ... }
function updateState() { ... }

// Group 4: Utilities
function formatData() { ... }
function validateInput() { ... }
```

## Commenting and Documentation

### 1. File Header Comments

Include a description at the top of each file:

```javascript
/**
 * Physics engine for Three Body Simulator
 * Handles gravitational calculations and collision detection
 * between celestial bodies.
 */
```

### 2. Function Comments

Document functions with JSDoc-style comments:

```javascript
/**
 * Calculates gravitational force between two bodies
 * @param {Object} body1 - First celestial body
 * @param {Object} body2 - Second celestial body
 * @param {Number} G - Gravitational constant
 * @returns {Vector3} - Force vector
 */
function calculateGravitationalForce(body1, body2, G) {
  // Implementation
}
```

### 3. Inline Comments

Use inline comments for complex logic:

```javascript
// Scale force for very large mass products to prevent instability
if (massProduct > 100) {
  scaleFactor = 1.0 / Math.log10(massProduct / 10);
}
```

## State Management

### 1. Centralized State

Keep application state centralized and well-defined:

```javascript
// Define application state object
const state = {
  simulation: {
    isRunning: false,
    isPaused: false,
    time: 0,
    speed: 1
  },
  bodies: [],
  ui: {
    activeTab: 'controls',
    showTrails: true
  }
};

// Functions to update state
function updateSimulationState(changes) {
  state.simulation = { ...state.simulation, ...changes };
  notifyStateListeners('simulation', state.simulation);
}
```

### 2. State Change Notifications

Implement a simple observer pattern for state changes:

```javascript
// State change listeners
const listeners = {};

// Register listener
function onStateChange(key, callback) {
  if (!listeners[key]) {
    listeners[key] = [];
  }
  listeners[key].push(callback);
}

// Notify listeners
function notifyStateListeners(key, newValue) {
  if (listeners[key]) {
    listeners[key].forEach(callback => callback(newValue));
  }
}
```

## Error Handling

### 1. Centralized Error Handling

Implement consistent error handling:

```javascript
// Centralized error handler
function handleError(error, context) {
  console.error(`Error in ${context}:`, error);
  
  // Log to monitoring service in production
  if (isProduction) {
    logErrorToService(error, context);
  }
  
  // Display user-friendly message if appropriate
  if (shouldShowUserError(error)) {
    showErrorMessage(getErrorMessage(error));
  }
}

// Usage
try {
  performRiskyOperation();
} catch (error) {
  handleError(error, 'performRiskyOperation');
}
```

### 2. Graceful Degradation

Allow your application to function even when parts fail:

```javascript
function initializeFeature() {
  try {
    // Initialize complex feature
    setupAdvancedGraphics();
  } catch (error) {
    // Fall back to basic version if initialization fails
    handleError(error, 'advancedGraphics');
    setupBasicGraphics();
  }
}
```

## Performance Considerations

### 1. Code Splitting

Split code into logical chunks:

```javascript
// main.js - Core functionality only
import { initCore } from './core.js';

// Dynamically import other features when needed
document.getElementById('advancedButton').addEventListener('click', async () => {
  const { initAdvancedFeatures } = await import('./advanced-features.js');
  initAdvancedFeatures();
});
```

### 2. Lazy Initialization

Initialize components only when needed:

```javascript
// Initialize on first use
let expensiveResource;

function getExpensiveResource() {
  if (!expensiveResource) {
    expensiveResource = createExpensiveResource();
  }
  return expensiveResource;
}
```

## Configuration Management

### 1. Centralized Configuration

Keep configuration values in one place:

```javascript
// config.js
export const config = {
  simulation: {
    gravityConstant: 0.3,
    boundaryRadius: 60,
    defaultSpeed: 0.3
  },
  rendering: {
    trailLength: 200,
    starCount: 15000
  },
  ui: {
    colors: {
      body1: 0xd8a0ff,
      body2: 0xffb6c1,
      body3: 0x87cefa
    }
  }
};
```

### 2. Environment-specific Configuration

Adjust configuration based on environment:

```javascript
// Detect environment
const isDevelopment = process.env.NODE_ENV === 'development';

// Adjust configuration
const config = {
  ...baseConfig,
  simulation: {
    ...baseConfig.simulation,
    // Faster simulation speed in development for testing
    defaultSpeed: isDevelopment ? 1.0 : 0.3
  },
  debug: isDevelopment ? {
    showFPS: true,
    logPhysics: true
  } : {
    showFPS: false,
    logPhysics: false
  }
};
```

## Testing Considerations

### 1. Testable Code Structure

Organize code to make it testable:

```javascript
// BEFORE: Hard to test
function doSomethingComplex() {
  const data = fetchDataFromGlobalState();
  const result = complexCalculation(data);
  updateUIWithResult(result);
}

// AFTER: Testable structure
function complexCalculation(data) {
  // Pure function that can be tested in isolation
  return processedResult;
}

function doSomethingComplex() {
  const data = fetchDataFromGlobalState();
  const result = complexCalculation(data); // Easily tested separately
  updateUIWithResult(result);
}
```

### 2. Separation of Core Logic and Side Effects

Separate pure logic from side effects:

```javascript
// Pure function (no side effects, same output for same input)
export function calculateTrajectory(position, velocity, mass, deltaTime) {
  // Pure physics calculations
  return newPosition;
}

// Function with side effects
export function updateBodyPosition(body, deltaTime) {
  // Calculate new position using pure function
  const newPosition = calculateTrajectory(
    body.position, 
    body.velocity, 
    body.mass, 
    deltaTime
  );
  
  // Side effect: Update the body object
  body.position = newPosition;
  
  // Side effect: Update the UI
  updateBodyMeshPosition(body);
}
```

## Browser Compatibility

### 1. Feature Detection

Detect feature support rather than checking browser:

```javascript
// Check for WebGL support
function isWebGLSupported() {
  try {
    const canvas = document.createElement('canvas');
    return !!(
      window.WebGLRenderingContext && 
      (canvas.getContext('webgl') || canvas.getContext('experimental-webgl'))
    );
  } catch (e) {
    return false;
  }
}

// Use the feature if available
if (isWebGLSupported()) {
  initializeWebGL();
} else {
  showFallbackMessage();
}
```

### Polyfills for Missing Features

Add polyfills only when needed:

```javascript
// Check if the feature exists
if (!Object.fromEntries) {
  // Polyfill if it doesn't
  Object.fromEntries = entries => 
    entries.reduce((obj, [key, val]) => {
      obj[key] = val;
      return obj;
    }, {});
}
```

## Security Practices

### 1. Input Validation

Always validate user input:

```javascript
function updateMass(bodyIndex, massValue) {
  // Validate input
  if (typeof bodyIndex !== 'number' || bodyIndex < 0 || bodyIndex >= bodies.length) {
    throw new Error(`Invalid body index: ${bodyIndex}`);
  }
  
  if (typeof massValue !== 'number' || massValue <= 0 || !isFinite(massValue)) {
    throw new Error(`Invalid mass value: ${massValue}`);
  }
  
  // Proceed with valid input
  bodies[bodyIndex].mass = massValue;
}
```

### 2. Safe DOM Manipulation

Avoid innerHTML for user content:

```javascript
// UNSAFE - vulnerable to XSS
element.innerHTML = userProvidedContent;

// SAFE - use textContent for text
element.textContent = userProvidedContent;

// For HTML content, use sanitization
import { sanitizeHtml } from './sanitize.js';
element.innerHTML = sanitizeHtml(userProvidedContent);
```

## Documentation Resources

Include links to documentation resources:

- Your application's README
- Inline code documentation
- Architecture diagrams
- API documentation

By following these best practices for code organization, you'll create a more maintainable, testable, and robust application that's easier to extend and debug. 