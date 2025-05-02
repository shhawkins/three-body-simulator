# React Integration Basics

This document explains the fundamentals of React and how we could integrate it with our Three Body Simulator to create a more maintainable and feature-rich UI.

## What is React?

React is a JavaScript library for building user interfaces. It uses a component-based architecture that allows you to build encapsulated, reusable pieces of UI that manage their own state.

Key React concepts:
- **Component-based**: Break down UIs into independent, reusable pieces
- **Declarative**: Define what you want the UI to look like based on state
- **Virtual DOM**: Efficiently updates the real DOM by only changing what needs to be changed
- **Unidirectional data flow**: Data flows from parent to child components

## Why Use React for the Three Body Simulator?

1. **Organized UI Code**: Separate UI from simulation logic
2. **Reusable Components**: Create control panels, sliders, and buttons as reusable components
3. **State Management**: Handle UI state more effectively
4. **Responsive Design**: Simplify creating layouts that work on different devices

## Basic React Concepts

### 1. Components

Components are the building blocks of React applications:

```jsx
// A functional component for a button
function SimulationButton({ label, onClick, disabled }) {
  return (
    <button 
      className="simulation-button" 
      onClick={onClick} 
      disabled={disabled}
    >
      {label}
    </button>
  );
}

// Usage
<SimulationButton 
  label="Start" 
  onClick={startSimulation} 
  disabled={isRunning} 
/>
```

### 2. Props

Props (short for properties) are how you pass data to components:

```jsx
// Component that receives props
function MassControls({ body, onMassChange }) {
  return (
    <div className="mass-control">
      <label>{body.name} Mass:</label>
      <input 
        type="range" 
        min="0" 
        max="100" 
        value={body.massSliderValue} 
        onChange={(e) => onMassChange(body.id, parseInt(e.target.value))} 
      />
      <span>{body.mass.toFixed(1)}</span>
    </div>
  );
}
```

### 3. State

State is used to manage data that changes over time:

```jsx
import { useState } from 'react';

function SimulationControls() {
  // State hooks for UI state
  const [isRunning, setIsRunning] = useState(false);
  const [isPaused, setIsPaused] = useState(false);
  const [showTrails, setShowTrails] = useState(true);
  
  // Handler functions
  const handleStart = () => {
    setIsRunning(true);
    startSimulation(); // Call the Three.js function
  };
  
  const handleStop = () => {
    setIsRunning(false);
    setIsPaused(false);
    stopSimulation(); // Call the Three.js function
  };
  
  const handleToggleTrails = () => {
    const newShowTrails = !showTrails;
    setShowTrails(newShowTrails);
    toggleTrails(newShowTrails); // Call the Three.js function
  };
  
  return (
    <div className="controls-panel">
      <h3>Simulation Controls</h3>
      <div className="button-row">
        <SimulationButton 
          label="Start" 
          onClick={handleStart} 
          disabled={isRunning && !isPaused} 
        />
        <SimulationButton 
          label="Stop" 
          onClick={handleStop} 
          disabled={!isRunning} 
        />
        <SimulationButton 
          label={isPaused ? "Resume" : "Pause"} 
          onClick={() => setIsPaused(!isPaused)} 
          disabled={!isRunning} 
        />
      </div>
      <div className="button-row">
        <SimulationButton 
          label={showTrails ? "Hide Trails" : "Show Trails"} 
          onClick={handleToggleTrails} 
          disabled={false} 
        />
      </div>
    </div>
  );
}
```

### 4. Effects

Effects let you perform side effects in components:

```jsx
import { useEffect } from 'react';

function SimulationStats({ bodies }) {
  useEffect(() => {
    // Update stats when component mounts or bodies change
    const updateStats = () => {
      // Update statistics display
      if (bodies.length === 0) return;
      
      // Calculate total energy, momentum, etc.
      // ...
    };
    
    // Call initially
    updateStats();
    
    // Set up an interval to update regularly
    const intervalId = setInterval(updateStats, 500);
    
    // Clean up when component unmounts
    return () => clearInterval(intervalId);
  }, [bodies]); // Re-run effect if bodies array changes
  
  return (
    <div className="stats-panel">
      {/* Display statistics */}
    </div>
  );
}
```

## Integrating React with Three.js

### Approach 1: React for UI only, Three.js for Simulation

This approach keeps the simulation core in Three.js and uses React just for the UI controls.

```jsx
import { useRef, useEffect, useState } from 'react';
import { initSimulation, updateSimulationParameters } from './threeBodySimulator';

function ThreeBodyApp() {
  const containerRef = useRef(null);
  const [bodies, setBodies] = useState([
    { id: 1, name: 'Body 1', mass: 1.0, color: '#d8a0ff', position: [-5, 0, 0], velocity: [0, 0, 0.42] },
    { id: 2, name: 'Body 2', mass: 1.0, color: '#ffb6c1', position: [5, 0, 0], velocity: [0, 0, -0.42] },
    { id: 3, name: 'Body 3', mass: 1.5, color: '#87cefa', position: [0, 0, 0], velocity: [0, 0, 0] }
  ]);
  
  // Set up Three.js on mount
  useEffect(() => {
    if (!containerRef.current) return;
    
    // Initialize Three.js simulation
    const simulation = initSimulation(containerRef.current, {
      bodies,
      onBodyUpdate: (updatedBodies) => {
        // Update React state with new positions/velocities
        setBodies(prevBodies => 
          prevBodies.map((body, i) => ({
            ...body,
            position: updatedBodies[i].position,
            velocity: updatedBodies[i].velocity
          }))
        );
      }
    });
    
    // Clean up on unmount
    return () => {
      simulation.dispose();
    };
  }, []);
  
  // Update Three.js when body parameters change
  useEffect(() => {
    updateSimulationParameters({ bodies });
  }, [bodies]);
  
  const handleMassChange = (bodyId, newMass) => {
    setBodies(prevBodies => 
      prevBodies.map(body => 
        body.id === bodyId ? { ...body, mass: newMass } : body
      )
    );
  };
  
  return (
    <div className="app-container">
      {/* Three.js container */}
      <div className="simulation-container" ref={containerRef}></div>
      
      {/* React UI components */}
      <SimulationControls />
      <BodyControlsPanel bodies={bodies} onMassChange={handleMassChange} />
      <CameraControlsPanel />
    </div>
  );
}
```

### Approach 2: React for UI, Three.js via react-three-fiber

This approach uses `react-three-fiber`, a React renderer for Three.js:

```jsx
import { Canvas, useFrame } from '@react-three/fiber';
import { OrbitControls } from '@react-three/drei';
import { useState, useRef } from 'react';

// Physics logic in a custom hook
function usePhysics(initialBodies, G = 0.3) {
  const [bodies, setBodies] = useState(initialBodies);
  
  useFrame((state, delta) => {
    // Update physics with delta time
    const deltaTime = delta * 0.3; // simulation speed
    
    // Copy current bodies for calculation
    const newBodies = bodies.map(body => ({
      ...body,
      position: [...body.position],
      velocity: [...body.velocity]
    }));
    
    // Calculate forces between bodies
    for (let i = 0; i < newBodies.length; i++) {
      for (let j = i + 1; j < newBodies.length; j++) {
        // Physics calculations
        // ...
      }
    }
    
    // Update positions
    newBodies.forEach(body => {
      // Update position based on velocity
      body.position[0] += body.velocity[0] * deltaTime;
      body.position[1] += body.velocity[1] * deltaTime;
      body.position[2] += body.velocity[2] * deltaTime;
    });
    
    setBodies(newBodies);
  });
  
  return [bodies, setBodies];
}

// A celestial body component
function CelestialBody({ position, mass, color }) {
  // Calculate radius from mass
  const radius = mass <= 1 
    ? Math.max(0.5, Math.pow(mass, 1/3) * 0.45)
    : 0.45 + 0.35 * Math.log10(mass);
    
  return (
    <mesh position={position}>
      <sphereGeometry args={[radius, 32, 32]} />
      <meshPhongMaterial 
        color={color} 
        shininess={30} 
        emissive={color} 
        emissiveIntensity={0.2} 
      />
    </mesh>
  );
}

// Trail component
function Trail({ points, color }) {
  return (
    <line>
      <bufferGeometry>
        <bufferAttribute 
          attachObject={['attributes', 'position']}
          count={points.length}
          array={new Float32Array(points.flat())}
          itemSize={3}
        />
      </bufferGeometry>
      <lineBasicMaterial color={color} />
    </line>
  );
}

// The main simulation component
function Simulation() {
  const initialBodies = [
    { id: 1, position: [-5, 0, 0], velocity: [0, 0, 0.42], mass: 1.0, color: '#d8a0ff', trail: [] },
    { id: 2, position: [5, 0, 0], velocity: [0, 0, -0.42], mass: 1.0, color: '#ffb6c1', trail: [] },
    { id: 3, position: [0, 0, 0], velocity: [0, 0, 0], mass: 1.5, color: '#87cefa', trail: [] }
  ];
  
  const [bodies, setBodies] = usePhysics(initialBodies);
  
  return (
    <>
      {/* Grid and environment */}
      <gridHelper args={[120, 20]} />
      <ambientLight intensity={0.4} />
      <directionalLight position={[10, 10, 10]} intensity={0.8} />
      
      {/* Bodies */}
      {bodies.map(body => (
        <CelestialBody 
          key={body.id}
          position={body.position}
          mass={body.mass}
          color={body.color}
        />
      ))}
      
      {/* Trails */}
      {bodies.map(body => (
        <Trail 
          key={`trail-${body.id}`}
          points={body.trail}
          color={body.color}
        />
      ))}
      
      {/* Camera controls */}
      <OrbitControls />
    </>
  );
}

// App component
function App() {
  const [showUI, setShowUI] = useState(true);
  
  return (
    <div className="app">
      <Canvas>
        <Simulation />
      </Canvas>
      
      {showUI && (
        <div className="ui-container">
          <SimulationControls />
          <BodyControls />
          <CameraControls />
        </div>
      )}
      
      <button 
        className="ui-toggle" 
        onClick={() => setShowUI(!showUI)}
      >
        {showUI ? 'Hide UI' : 'Show UI'}
      </button>
    </div>
  );
}
```

## State Management Options

For a complex application like the Three Body Simulator, you might need more advanced state management:

### 1. React Context API

For sharing state between components without prop drilling:

```jsx
import { createContext, useContext, useState } from 'react';

// Create context
const SimulationContext = createContext();

// Context provider component
function SimulationProvider({ children }) {
  const [simulationState, setSimulationState] = useState({
    isRunning: false,
    isPaused: false,
    showTrails: true,
    simulationSpeed: 0.3,
    freePlayMode: false,
    bodies: [/* initial bodies */]
  });
  
  // Functions to update state
  const startSimulation = () => {
    setSimulationState(prev => ({ ...prev, isRunning: true, isPaused: false }));
    // Call Three.js function
  };
  
  const setBodyMass = (bodyId, mass) => {
    setSimulationState(prev => ({
      ...prev,
      bodies: prev.bodies.map(body => 
        body.id === bodyId ? { ...body, mass } : body
      )
    }));
    // Update Three.js
  };
  
  // Provide state and functions to children
  return (
    <SimulationContext.Provider value={{
      state: simulationState,
      startSimulation,
      setBodyMass,
      // Other functions...
    }}>
      {children}
    </SimulationContext.Provider>
  );
}

// Custom hook to use the context
function useSimulation() {
  const context = useContext(SimulationContext);
  if (!context) {
    throw new Error('useSimulation must be used within a SimulationProvider');
  }
  return context;
}

// Usage in components
function MassControls() {
  const { state, setBodyMass } = useSimulation();
  
  return (
    <div>
      {state.bodies.map(body => (
        <div key={body.id}>
          <input 
            type="range" 
            value={body.mass * 10} 
            onChange={(e) => setBodyMass(body.id, e.target.value / 10)} 
          />
        </div>
      ))}
    </div>
  );
}
```

### 2. Redux

For more complex state management:

```jsx
import { configureStore, createSlice } from '@reduxjs/toolkit';
import { Provider, useSelector, useDispatch } from 'react-redux';

// Create a slice for simulation state
const simulationSlice = createSlice({
  name: 'simulation',
  initialState: {
    isRunning: false,
    isPaused: false,
    showTrails: true,
    simulationSpeed: 0.3,
    bodies: [/* initial bodies */]
  },
  reducers: {
    startSimulation: (state) => {
      state.isRunning = true;
      state.isPaused = false;
    },
    pauseSimulation: (state) => {
      state.isPaused = !state.isPaused;
    },
    setBodyMass: (state, action) => {
      const { bodyId, mass } = action.payload;
      const body = state.bodies.find(b => b.id === bodyId);
      if (body) {
        body.mass = mass;
      }
    }
    // Other actions...
  }
});

// Extract actions and reducer
export const { startSimulation, pauseSimulation, setBodyMass } = simulationSlice.actions;

// Create store
const store = configureStore({
  reducer: {
    simulation: simulationSlice.reducer
  }
});

// Usage in components
function SimulationControls() {
  const dispatch = useDispatch();
  const { isRunning, isPaused } = useSelector(state => state.simulation);
  
  return (
    <div>
      <button 
        onClick={() => dispatch(startSimulation())}
        disabled={isRunning && !isPaused}
      >
        Start
      </button>
      <button 
        onClick={() => dispatch(pauseSimulation())}
        disabled={!isRunning}
      >
        {isPaused ? 'Resume' : 'Pause'}
      </button>
    </div>
  );
}

// Wrap app with provider
function App() {
  return (
    <Provider store={store}>
      <ThreeBodySimulator />
    </Provider>
  );
}
```

## Responsive UI with React

React makes it easier to build responsive UIs:

```jsx
import { useState, useEffect } from 'react';

// Custom hook for responsive design
function useResponsiveLayout() {
  const [isMobile, setIsMobile] = useState(window.innerWidth < 768);
  const [isLandscape, setIsLandscape] = useState(window.innerWidth > window.innerHeight);
  
  useEffect(() => {
    const handleResize = () => {
      setIsMobile(window.innerWidth < 768);
      setIsLandscape(window.innerWidth > window.innerHeight);
    };
    
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);
  
  return { isMobile, isLandscape };
}

// Responsive control panel
function ControlPanel() {
  const { isMobile, isLandscape } = useResponsiveLayout();
  
  return (
    <div className={`
      control-panel 
      ${isMobile ? 'mobile' : 'desktop'}
      ${isLandscape ? 'landscape' : 'portrait'}
    `}>
      {isMobile ? (
        // Mobile layout with tabs for controls
        <TabLayout />
      ) : (
        // Desktop layout with all controls visible
        <DesktopLayout />
      )}
    </div>
  );
}
```

## Setting Up the Project

To set up a React + Three.js project:

1. **Create React App with TypeScript**

```bash
npx create-react-app three-body-simulator --template typescript
cd three-body-simulator
```

2. **Install Dependencies**

```bash
npm install three @types/three @react-three/fiber @react-three/drei
```

3. **Project Structure**

```
three-body-simulator/
├── public/
│   └── index.html
├── src/
│   ├── components/
│   │   ├── ui/           # React UI components
│   │   │   ├── ControlPanel.tsx
│   │   │   ├── BodyControls.tsx
│   │   │   └── CameraControls.tsx
│   │   │
│   │   └── simulation/   # Three.js + React components
│   │       ├── Scene.tsx
│   │       ├── Body.tsx
│   │       └── Trail.tsx
│   │
│   ├── hooks/            # Custom React hooks
│   │   ├── usePhysics.ts
│   │   └── useResponsiveLayout.ts
│   │
│   ├── models/           # TypeScript interfaces
│   │   └── types.ts
│   │
│   ├── utils/            # Utility functions
│   │   └── physics.ts
│   │
│   ├── App.tsx           # Main app component
│   └── index.tsx         # Entry point
└── package.json
```

## Conclusion

Integrating React with Three.js for the Three Body Simulator provides several advantages:

1. **Better UI Organization**: Separate UI from simulation logic
2. **Component Reusability**: Create a library of reusable UI components
3. **Declarative UI**: Define UI based on application state
4. **State Management**: Handle complex UI state more effectively
5. **Responsive Design**: Easily adapt to different devices
6. **Developer Experience**: Use modern tooling and patterns

Whether you choose the "React for UI only" approach or the fully integrated approach with react-three-fiber depends on:

- How much of the existing Three.js code you want to preserve
- How comfortable you are with React concepts
- Whether you prefer imperative (traditional Three.js) or declarative (React) programming

For a gradual migration, starting with "React for UI only" and slowly transitioning more components might be the best approach. 