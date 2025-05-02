# UI Components Overview

This document provides an overview of the UI components in our Three Body Simulator, explaining their purpose and functionality.

## Main UI Layout

The UI is organized into several panels, each with specific controls:

```
+-------------------------------+      +------------------------+
|                               |      |                        |
| Three Body Problem Simulator  |      |  Body Controls         |
| +-------------------------+   |      |  +------------------+  |
| | Start | Stop | Pause    |   |      |  | Body 1 Mass      |  |
| +-------------------------+   |      |  | [Slider]         |  |
| | Reset | Reset Previous  |   |      |  | Velocity Arrows  |  |
| +-------------------------+   |      |  +------------------+  |
| | Toggle | Hide Trails    |   |      |  | Body 2 Mass      |  |
| +-------------------------+   |      |  | [Slider]         |  |
| | Speed Controls          |   |      |  | Velocity Arrows  |  |
| +-------------------------+   |      |  +------------------+  |
|                               |      |  | Body 3 Mass      |  |
+-------------------------------+      |  | [Slider]         |  |
                                       |  | Velocity Arrows  |  |
+-------------------------------+      |  +------------------+  |
|                               |      |                        |
| Camera Controls               |      +------------------------+
| +-------------------------+   |
| | Top | Side | Angled     |   |      +------------------------+
| +-------------------------+   |      |                        |
| | Wide | Super Wide       |   |      |  Instructions         |
| +-------------------------+   |      |                        |
| | Follow Body Controls    |   |      |  1. Drag bodies       |
| +-------------------------+   |      |  2. Drag arrows       |
| | Cinematic Mode          |   |      |  3. Adjust sliders    |
| +-------------------------+   |      |  4. Click Start       |
|                               |      |                        |
+-------------------------------+      +------------------------+
```

## Core Control Panels

### 1. Simulation Controls Panel
- **Location**: Top left of the screen
- **Purpose**: Primary controls for the simulation
- **Key Elements**:
  - Start/Stop/Pause buttons
  - Reset buttons (default and previous state)
  - Toggle Borderless Mode and Trail visibility
  - Speed control buttons

### 2. Body Controls Panel
- **Location**: Top right of the screen
- **Purpose**: Adjust properties of the three bodies
- **Key Elements**:
  - Mass sliders for each body
  - Real-time velocity displays
  - Velocity arrows (visualized in the 3D scene)

### 3. Camera Controls Panel
- **Location**: Bottom left of the screen
- **Purpose**: Change viewing angles and perspectives
- **Key Elements**:
  - Preset view buttons (Top, Side, Angled)
  - Distance options (Wide, Super Wide)
  - Follow body buttons
  - Cinematic Mode toggle

### 4. Instructions Panel
- **Location**: Bottom right of the screen
- **Purpose**: Guide users on how to use the simulator
- **Key Elements**:
  - Step-by-step instructions
  - Warning about collisions
  - Usage tips

## Interactive 3D Elements

The UI includes interactive elements within the 3D scene:

1. **Draggable Bodies**: The three spheres can be dragged to position them
2. **Velocity Arrows**: Arrows extending from each body can be dragged to set velocity
3. **Grid and Boundary Circle**: Visual indicators of the simulation space

## Button Controls Explained

### Simulation Flow Controls
- **Start**: Begins the simulation from current positions
- **Stop**: Halts the simulation and allows repositioning
- **Pause**: Temporarily freezes the simulation

### Reset Controls
- **Reset (Default)**: Returns to the original stable configuration
- **Reset (Previous)**: Reverts to the last saved state (pre-simulation)

### Visualization Controls
- **Toggle Borderless**: Removes boundary constraints
- **Hide/Show Trails**: Toggles the visibility of motion trails

### Camera Perspective Controls
- **Top/Side/Angled View**: Preset camera positions
- **Wide/Super Wide**: Zoomed out perspectives
- **Follow Body**: Camera tracks a specific body
- **View to Fit**: Automatically frames all bodies
- **Cinematic Mode**: Automatic camera movement

## Mobile-Specific Adaptations

The UI adapts for mobile devices:
- Larger touch targets
- Simplified layouts for smaller screens
- Touch-optimized dragging mechanics
- Orientation-specific adjustments

## UI State Management

The interface has several states:
1. **Setup Mode**: Before simulation starts, allows positioning
2. **Running Mode**: During simulation, certain controls are disabled
3. **Paused Mode**: Simulation frozen but can be resumed
4. **Post-Collision**: After bodies collide, special options appear

## CSS Styling Strategy

The UI uses:
- Semi-transparent panels with backdrop blur
- Consistent color scheme with high contrast for readability
- Collapsible panels to reduce visual clutter
- Distinct color coding for each body
- Responsive sizing based on device

## Tooltips and Feedback

- Hover tooltips provide guidance
- Visual indicators show active states
- Real-time velocity displays update continuously
- Error states (boundary violations, collisions) have visual feedback

## UI Toggle Controls

For a cleaner view:
- **Hide UI** button removes all controls
- **Cinematic Mode** toggle available when UI is hidden
- Collapsible panels can be minimized individually

Understanding these UI components helps you navigate the interface and make the most of the simulation's features. 