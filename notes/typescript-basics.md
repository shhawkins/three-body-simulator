# TypeScript Basics for Three Body Simulator

This document introduces TypeScript fundamentals and how they could be applied to enhance our Three Body Simulator project.

## What is TypeScript?

TypeScript is a superset of JavaScript that adds static typing and other features to help build more robust applications. It compiles down to regular JavaScript but provides better tooling, error checking, and code organization.

## Key TypeScript Concepts

### 1. Type Annotations

The core feature of TypeScript is being able to specify types for variables, parameters, and return values:

```typescript
// Basic type annotations
let mass: number = 1.5;
let isRunning: boolean = false;
let bodyName: string = "Earth";

// Function with typed parameters and return value
function calculateGravity(mass1: number, mass2: number, distance: number): number {
  return G * mass1 * mass2 / (distance * distance);
}
```

### 2. Interfaces

Interfaces define the shape of objects:

```typescript
// Define an interface for a celestial body
interface CelestialBody {
  position: Vector3;
  velocity: Vector3;
  mass: number;
  radius: number;
  color: number; // Three.js color as a hex number
  trail: Vector3[];
}

// Use the interface
function updateBody(body: CelestialBody, deltaTime: number): void {
  // Update body properties
  body.position.add(body.velocity.clone().multiplyScalar(deltaTime));
}
```

### 3. Classes

TypeScript enhances JavaScript classes with visibility modifiers and typed properties:

```typescript
class Body implements CelestialBody {
  // Public properties
  public color: number;
  
  // Private properties
  private _mass: number;
  private _trail: Vector3[] = [];
  
  // Constructor with parameter types
  constructor(
    public position: Vector3,
    public velocity: Vector3,
    mass: number,
    public radius: number = 1.0
  ) {
    this._mass = mass;
    this.color = 0xffffff; // Default color
  }
  
  // Getter with type annotation
  get mass(): number {
    return this._mass;
  }
  
  // Setter with validation
  set mass(value: number) {
    if (value <= 0) {
      throw new Error("Mass must be positive");
    }
    this._mass = value;
  }
  
  // Method with return type
  public getKineticEnergy(): number {
    return 0.5 * this._mass * this.velocity.lengthSq();
  }
  
  // Method with void return type (returns nothing)
  public addTrailPoint(): void {
    this._trail.push(this.position.clone());
    
    // Limit the trail length
    if (this._trail.length > 1000) {
      this._trail.shift();
    }
  }
  
  // Getter for the trail (returns readonly array)
  get trail(): ReadonlyArray<Vector3> {
    return this._trail;
  }
}
```

### 4. Type Definitions for Libraries

TypeScript provides type definitions for many libraries, including Three.js:

```typescript
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';

// Now you get type checking and autocomplete for Three.js
const scene: THREE.Scene = new THREE.Scene();
const camera: THREE.PerspectiveCamera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
const renderer: THREE.WebGLRenderer = new THREE.WebGLRenderer({ antialias: true });
```

### 5. Enums

Enums provide a way to define named constants:

```typescript
// Define simulation states
enum SimulationState {
  Stopped,
  Running,
  Paused
}

// Use the enum
let currentState: SimulationState = SimulationState.Stopped;

function startSimulation(): void {
  if (currentState === SimulationState.Stopped) {
    currentState = SimulationState.Running;
    // Start logic...
  }
}
```

### 6. Union Types

Union types allow a variable to be one of several types:

```typescript
// A function that accepts either a number or a string
function setMass(body: Body, mass: number | string): void {
  if (typeof mass === 'string') {
    // Convert string to number
    body.mass = parseFloat(mass);
  } else {
    body.mass = mass;
  }
}
```

### 7. Type Aliases

Create your own custom types with type aliases:

```typescript
// Create a type alias for 3D coordinates
type Point3D = {
  x: number;
  y: number;
  z: number;
};

// Or for a color in different formats
type Color = number | string | THREE.Color;

// Function that accepts any color format
function setBodyColor(body: Body, color: Color): void {
  if (typeof color === 'number') {
    body.material.color.setHex(color);
  } else if (typeof color === 'string') {
    body.material.color.set(color);
  } else {
    body.material.color.copy(color);
  }
}
```

## Applying TypeScript to the Three Body Simulator

### Refactoring Plan

1. **Setup TypeScript**

   First, we need to set up TypeScript in the project:

   ```bash
   npm init -y
   npm install --save-dev typescript ts-loader webpack webpack-cli
   ```

   Create a `tsconfig.json` file:

   ```json
   {
     "compilerOptions": {
       "target": "es2015",
       "module": "es2015",
       "moduleResolution": "node",
       "strict": true,
       "sourceMap": true,
       "outDir": "./dist",
       "baseUrl": "./src",
       "esModuleInterop": true,
       "lib": ["dom", "es2015", "es2016", "es2017"]
     },
     "include": ["src/**/*"]
   }
   ```

2. **Define Core Interfaces**

   Start by defining interfaces for the main components:

   ```typescript
   // src/types/index.ts
   
   import * as THREE from 'three';
   
   export interface SimulationOptions {
     gravityConstant: number;
     boundaryRadius: number;
     simulationSpeed: number;
     trailLength: number;
     showTrails: boolean;
   }
   
   export interface CelestialBody {
     position: THREE.Vector3;
     velocity: THREE.Vector3;
     mass: number;
     trail: THREE.Vector3[];
     mesh?: THREE.Mesh;
     trailMesh?: THREE.Mesh;
     addTrailPoint(): void;
     update(): void;
     reset(): void;
   }
   
   export interface PhysicsEngine {
     bodies: CelestialBody[];
     options: SimulationOptions;
     update(deltaTime: number): void;
     addBody(body: CelestialBody): void;
     checkCollisions(): boolean;
     reset(): void;
   }
   
   export interface SceneManager {
     scene: THREE.Scene;
     camera: THREE.PerspectiveCamera;
     renderer: THREE.WebGLRenderer;
     controls: any; // OrbitControls
     render(): void;
     resize(): void;
     add(object: THREE.Object3D): void;
     remove(object: THREE.Object3D): void;
   }
   ```

3. **Implement Core Classes**

   ```typescript
   // src/entities/Body.ts
   
   import * as THREE from 'three';
   import { CelestialBody } from '../types';
   
   export class Body implements CelestialBody {
     public trail: THREE.Vector3[] = [];
     public mesh: THREE.Mesh;
     public trailMesh?: THREE.Mesh;
     
     private initialPosition: THREE.Vector3;
     private initialVelocity: THREE.Vector3;
     
     constructor(
       public position: THREE.Vector3,
       public velocity: THREE.Vector3,
       public mass: number,
       public color: number,
       private maxTrailLength: number = 5000
     ) {
       this.initialPosition = position.clone();
       this.initialVelocity = velocity.clone();
       
       // Create the mesh
       const radius = this.getRadiusFromMass();
       const geometry = new THREE.SphereGeometry(radius, 32, 32);
       const material = new THREE.MeshPhongMaterial({
         color: this.color,
         shininess: 30,
         emissive: new THREE.Color(this.color).multiplyScalar(0.2)
       });
       
       this.mesh = new THREE.Mesh(geometry, material);
       this.mesh.position.copy(this.position);
     }
     
     public getRadiusFromMass(): number {
       if (this.mass <= 1) {
         return Math.max(0.5, Math.pow(this.mass, 1/3) * 0.45);
       } else {
         return 0.45 + 0.35 * Math.log10(this.mass);
       }
     }
     
     public addTrailPoint(): void {
       this.trail.push(this.position.clone());
       
       if (this.trail.length > this.maxTrailLength) {
         this.trail = this.trail.slice(-this.maxTrailLength);
       }
     }
     
     public update(): void {
       // Update mesh position to match physics position
       this.mesh.position.copy(this.position);
     }
     
     public reset(): void {
       this.position.copy(this.initialPosition);
       this.velocity.copy(this.initialVelocity);
       this.trail = [];
       this.update();
     }
   }
   ```

4. **Physics Engine Implementation**

   ```typescript
   // src/core/PhysicsEngine.ts
   
   import * as THREE from 'three';
   import { PhysicsEngine, CelestialBody, SimulationOptions } from '../types';
   
   export class Physics implements PhysicsEngine {
     public bodies: CelestialBody[] = [];
     public collisionOccurred: boolean = false;
     
     constructor(public options: SimulationOptions) {}
     
     public addBody(body: CelestialBody): void {
       this.bodies.push(body);
     }
     
     public update(deltaTime: number): void {
       if (this.collisionOccurred) return;
       
       // Calculate forces between all unique pairs of bodies
       for (let i = 0; i < this.bodies.length; i++) {
         for (let j = i + 1; j < this.bodies.length; j++) {
           // Check for collision
           const body1 = this.bodies[i];
           const body2 = this.bodies[j];
           
           const distance = body1.position.distanceTo(body2.position);
           // Check collision based on visual size
           // (this would be better determined from the Body class)
           const radius1 = this.getRadiusFromMass(body1.mass);
           const radius2 = this.getRadiusFromMass(body2.mass);
           
           if (distance < (radius1 + radius2)) {
             this.collisionOccurred = true;
             return;
           }
           
           // Calculate and apply gravitational forces
           this.applyGravitationalForce(body1, body2, deltaTime);
         }
       }
       
       // Update positions based on velocities
       for (const body of this.bodies) {
         // Update position
         body.position.add(body.velocity.clone().multiplyScalar(deltaTime));
         
         // Check boundary if not in free play mode
         if (!this.options.freePlayMode) {
           const distanceFromCenter = Math.sqrt(
             body.position.x * body.position.x + 
             body.position.z * body.position.z
           );
           const radius = this.getRadiusFromMass(body.mass);
           
           if (distanceFromCenter + radius > this.options.boundaryRadius) {
             this.collisionOccurred = true;
             return;
           }
         }
         
         // Record trail point
         body.addTrailPoint();
       }
     }
     
     private applyGravitationalForce(body1: CelestialBody, body2: CelestialBody, deltaTime: number): void {
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
         
         const forceMagnitude = this.options.gravityConstant * massProduct / distanceSquared * scaleFactor;
         const force = direction.normalize().multiplyScalar(forceMagnitude);
         
         // Apply forces
         const acc1 = force.clone().divideScalar(body1.mass);
         const acc2 = force.clone().negate().divideScalar(body2.mass);
         
         body1.velocity.add(acc1.multiplyScalar(deltaTime));
         body2.velocity.add(acc2.multiplyScalar(deltaTime));
       }
     }
     
     // Helper method to determine visual radius from mass
     private getRadiusFromMass(mass: number): number {
       if (mass <= 1) {
         return Math.max(0.5, Math.pow(mass, 1/3) * 0.45);
       } else {
         return 0.45 + 0.35 * Math.log10(mass);
       }
     }
     
     public checkCollisions(): boolean {
       return this.collisionOccurred;
     }
     
     public reset(): void {
       this.collisionOccurred = false;
       for (const body of this.bodies) {
         body.reset();
       }
     }
   }
   ```

### Benefits of Using TypeScript for the Three Body Simulator

1. **Improved Code Quality**
   - Detect errors at compile time rather than runtime
   - Reduce bugs related to incorrect types or object shapes

2. **Better Developer Experience**
   - Autocomplete for methods and properties
   - Documentation appears in tooltips
   - Easier refactoring with confidence

3. **Clearer Code Organization**
   - Interfaces define the contract between components
   - Type definitions document expectations
   - Class hierarchies become more explicit

4. **Safer Refactoring**
   - The compiler will catch many errors when you change code
   - Better confidence when modularizing the codebase

5. **Integration with Three.js**
   - Three.js has excellent TypeScript definitions
   - Get autocomplete and error checking for Three.js API calls

## Performance Considerations

TypeScript itself doesn't impact runtime performance as it compiles to JavaScript. The resulting code is as efficient as hand-written JavaScript. However, well-typed code can lead to better optimization decisions when designing the simulation.

## TypeScript with Modern Tools

TypeScript integrates well with:

- **Webpack**: For module bundling
- **ESLint**: For code linting with TypeScript-specific rules
- **Jest**: For unit testing with TypeScript support
- **VS Code**: For excellent TypeScript editing and debugging

## Conclusion

Adopting TypeScript for the Three Body Simulator would improve code quality, maintainability, and developer experience. The initial setup requires some effort, but the benefits become apparent quickly, especially when working with a complex system like a physics simulation. 