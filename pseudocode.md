# Three-Body Problem Simulator - How It Works

## Overview
This document explains how the three-body simulator works in simple terms. The simulator allows you to create and observe three celestial bodies (like planets or stars) interacting through gravity.

## Core Components

### 1. The Scene Setup
```pseudocode
When the program starts:
    Create a 3D space (scene)
    Add a camera to view the scene
    Add lighting to see the bodies
    Create a flat surface (plane) where bodies move
    Set up controls for:
        - Moving the camera
        - Dragging bodies
        - Adjusting settings
```

### 2. The Three Bodies
```pseudocode
Each body has:
    - Position (where it is in space)
    - Velocity (how fast and in what direction it's moving)
    - Mass (how heavy it is)
    - Color (to distinguish it from others)
    - Trail (showing where it's been)
```

### 3. Physics Simulation
```pseudocode
For each frame of animation:
    For each body:
        Calculate gravitational force from other bodies:
            Force = (G * mass1 * mass2) / (distance²)
            Direction = towards the other body
        
        Update velocity:
            New velocity = Current velocity + (Force / mass) * time
        
        Update position:
            New position = Current position + velocity * time
        
        Keep body within boundary if enabled
```

### 4. User Controls

#### Camera Controls
```pseudocode
Camera can:
    - Rotate around the scene
    - Zoom in and out
    - Follow specific bodies
    - Switch between preset views (top, side, etc.)
    - Automatically adjust to show all bodies
```

#### Body Controls
```pseudocode
Each body can be:
    - Dragged to new positions
    - Given initial velocity by dragging arrows
    - Have its mass adjusted with sliders
```

#### Simulation Controls
```pseudocode
Simulation can be:
    - Started/stopped
    - Paused/resumed
    - Reset to initial positions
    - Speed adjusted (slow to super fast)
```

### 5. Visual Effects

#### Trails
```pseudocode
For each body:
    Remember its last positions
    Draw a colored tube following its path
    Make the trail fade out over time
    Update trail as body moves
```

#### Velocity Arrows
```pseudocode
For each body:
    Show an arrow indicating:
        - Direction of movement
        - Speed (longer arrow = faster)
    Update arrow as velocity changes
```

### 6. Special Features

#### Boundary Mode
```pseudocode
When boundary mode is on:
    Keep all bodies within a circular area
    If a body hits the boundary:
        Stop the simulation
        Show reset options
```

#### Cinematic Mode
```pseudocode
When cinematic mode is on:
    Automatically move camera to show interesting views
    Switch between different camera angles
    Focus on bodies when they're close together
```

#### Free Play Mode
```pseudocode
When free play mode is on:
    Remove boundary restrictions
    Allow bodies to move anywhere
    Show larger grid for reference
```

## How Everything Works Together

1. **Initialization**
   - Set up the 3D environment
   - Create three bodies with default positions
   - Initialize all controls and UI elements

2. **User Interaction**
   - User can position bodies and set velocities
   - User can adjust simulation settings
   - User can control the camera view

3. **Simulation Loop**
   - Calculate gravitational forces
   - Update body positions and velocities
   - Update visual elements (trails, arrows)
   - Check for collisions or boundary violations
   - Render the updated scene

4. **Visual Feedback**
   - Show body movements in real-time
   - Display trails showing past positions
   - Update velocity arrows
   - Show status messages for important events

## Technical Details

### Physics Calculations
```pseudocode
Gravitational Force:
    F = G * (m1 * m2) / r²
    where:
        F = force
        G = gravitational constant
        m1, m2 = masses of the bodies
        r = distance between bodies

Velocity Update:
    v = v0 + (F/m) * t
    where:
        v = new velocity
        v0 = current velocity
        F = force
        m = mass
        t = time step

Position Update:
    p = p0 + v * t
    where:
        p = new position
        p0 = current position
        v = velocity
        t = time step
```

### Performance Optimizations
```pseudocode
To keep the simulation smooth:
    - Limit trail length
    - Adjust time step based on speed
    - Clean up old trail data
    - Optimize rendering for mobile devices
```

## Common Scenarios

### Setting Up a Simulation
```pseudocode
1. Position the bodies where you want them
2. Set initial velocities by dragging arrows
3. Adjust masses if desired
4. Click "Start" to begin simulation
```

### Observing Interesting Patterns
```pseudocode
1. Try different initial positions
2. Experiment with different velocities
3. Adjust masses to create different effects
4. Use cinematic mode to automatically find good views
```

### Troubleshooting
```pseudocode
If simulation stops unexpectedly:
    - Check if bodies hit the boundary
    - Look for collision messages
    - Use reset buttons to start over
```

This simulator provides a hands-on way to explore the complex and fascinating behavior of gravitational systems with three bodies, demonstrating how small changes in initial conditions can lead to dramatically different outcomes. 