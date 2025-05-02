# Physics Simulation Basics

This document explains the physics concepts and implementation techniques used in our Three Body Simulator.

## Gravitational Physics

### Newton's Law of Universal Gravitation

The simulation is based on Newton's Law of Universal Gravitation, which states that every mass attracts every other mass with a force that is:
- Directly proportional to the product of their masses
- Inversely proportional to the square of the distance between them

The formula is:

```
F = G * (m1 * m2) / r²
```

Where:
- F is the gravitational force between two objects
- G is the gravitational constant
- m1 and m2 are the masses of the objects
- r is the distance between the centers of the masses

In our code, this is implemented as:

```javascript
const forceMagnitude = G * massProduct / distanceSquared * scaleFactor;
```

### Forces and Acceleration

Newton's Second Law (F = ma) relates force to acceleration. In our simulation, we calculate acceleration as:

```javascript
const acc1 = force.clone().divideScalar(body1.mass);
const acc2 = force.clone().negate().divideScalar(body2.mass);
```

This means heavier bodies accelerate less under the same force.

## Physics Integration

### Time Steps

The simulation advances in discrete time steps. The `updatePhysics` function advances the simulation by a small time delta with each call:

```javascript
updatePhysics(simulationSpeed);
```

### Velocity Verlet Integration (Simplified)

Although not explicitly named in the code, the simulation uses a simplified form of velocity Verlet integration:

1. Calculate forces/accelerations at the current position
2. Update velocities based on these accelerations
3. Update positions based on the new velocities

```javascript
// Update velocities based on forces
body1.velocity.add(acc1.multiplyScalar(deltaTime));
body2.velocity.add(acc2.multiplyScalar(deltaTime));

// Update positions based on velocities
body.position.add(body.velocity.clone().multiplyScalar(deltaTime));
```

## Collision Detection

The simulation detects two types of events:

### Body-Body Collisions
When two bodies get close enough that their radii overlap:

```javascript
const distance = body1.position.distanceTo(body2.position);
const body1Radius = getBodyRadius(body1.mass);
const body2Radius = getBodyRadius(body2.mass);

if (distance < (body1Radius + body2Radius)) {
  handleCollision(i, j);
  return;
}
```

### Boundary Violations
When a body moves outside the allowed boundary:

```javascript
const distanceFromCenter = Math.sqrt(
  body.position.x * body.position.x + 
  body.position.z * body.position.z
);

if (distanceFromCenter + bodyRadius > BOUNDARY_RADIUS) {
  handleBoundaryViolation(i);
  return;
}
```

## Scale Factors and Constants

The simulation uses arbitrary units and scale factors to make the visualization more intuitive:

1. The gravitational constant (G) is set to 0.3, not the actual physical value
2. Masses are visualized with non-linear scaling to make differences more apparent
3. Time progression is adjustable with simulation speed settings

## Performance Considerations

### Fixed Time Step vs. Variable Time Step

The simulation uses a fixed time step approach (controlled by simulationSpeed) rather than a variable time step based on actual frame time. This provides more predictable and stable physics.

### Optimization Techniques

- Force calculations only occur between unique pairs of bodies (no redundant calculations)
- Boundary checks are skipped in "free play" mode
- Scaling factors are applied to extremely large mass products to prevent numerical instability

## Understanding Simulation Parameters

### Mass

- Affects the gravitational pull of objects
- Affects the visual size of objects (using a logarithmic scale)
- Ranges from 0.1 to 100 in the simulation

### Velocity

- Determines the initial trajectory of objects
- Visualized with arrows in the initial setup
- Updated continuously due to gravitational forces

### Position

- Defines where objects are located in 3D space
- Affects the gravitational force calculation
- Limited by boundary constraints (when enabled)

By understanding these physics principles, you can create more interesting orbital configurations and better predict how the simulation will behave. 