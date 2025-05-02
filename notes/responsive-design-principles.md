# Responsive Design Principles

This document covers key responsive design principles that we use in the Three Body Simulator to ensure it works well on all devices.

## Core Responsive Design Concepts

### 1. Fluid Layouts

Instead of fixed pixel widths, we use relative units like percentages, viewport units, and flexible sizing:

```css
#container {
  width: 100%;
  max-width: 177.78vh; /* 16:9 aspect ratio */
  height: 100%;
  max-height: 56.25vw; /* 16:9 aspect ratio */
  margin: auto;
}
```

### 2. Media Queries

We use media queries to apply different styles based on screen size, device type, or orientation:

```css
/* Mobile and touch styles */
@media (pointer: coarse) {
  .button {
    padding: 10px 14px !important;
    margin: 5px !important;
    min-width: 70px;
    font-size: 14px !important;
  }
}

/* iPhone specific adjustments */
@media only screen and (max-device-width: 480px) {
  .button {
    padding: 8px 10px !important;
    margin: 3px !important;
    min-width: 60px;
    font-size: 12px !important;
  }
}

/* Landscape orientation adjustments */
@media (orientation: landscape) {
  body {
    height: 100dvh; /* Dynamic viewport height */
  }
  
  #controls {
    max-height: 60dvh;
    max-width: 35% !important;
  }
}
```

### 3. Device Detection

We detect device capabilities and adjust functionality accordingly:

```javascript
// Detect if we're on a touch device
const isTouchDevice = 'ontouchstart' in window || navigator.maxTouchPoints > 0;

if (isTouchDevice) {
  // Apply touch-specific settings
  controls.rotateSpeed = 0.7; // Slower rotation for more precision
  controls.zoomSpeed = 1.2;   // Slightly faster zoom
}
```

## Touch-Friendly UI Elements

### 1. Appropriate Touch Target Sizes

Touch targets should be large enough for fingers (minimum 44×44px):

```css
@media (pointer: coarse) {
  .button {
    padding: 10px 14px !important;
    min-width: 70px;
  }
  
  .mass-slider::-webkit-slider-thumb {
    width: 22px !important;
    height: 22px !important;
  }
}
```

### 2. Touch Feedback

Provide visual feedback for touch interactions:

```css
.button:active {
  transform: scale(0.95);
  background-color: #34495e;
}

.mass-slider::-webkit-slider-thumb:active {
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.5);
}
```

### 3. Simplified Gestures

We implement intuitive touch gestures:

```javascript
// Special handling for multi-touch
controls.touchStart = function(event) {
  // Two-finger touch - force zoom instead of rotate
  if (event.touches.length === 2) {
    this.handleTouchStartDolly(event);
  } else {
    // Single-finger touch - normal behavior
    this.handleTouchStartRotate(event);
  }
};
```

## Viewport Configuration

Proper viewport settings are crucial for responsive design:

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
<meta name="apple-mobile-web-app-capable" content="yes" />
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent" />
<meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover, maximum-scale=1.0, user-scalable=no" />
```

## Handling Different Screen Orientations

Different orientations require different layouts:

```javascript
// Orientation change handler
window.addEventListener('orientationchange', function() {
  setTimeout(() => {
    // Delay execution to allow the browser to complete the orientation change
    handleWindowResize();
    
    // Additional iPad-specific adjustments
    const isLandscape = window.innerWidth > window.innerHeight;
    
    // Adjust UI elements based on orientation
    if (isLandscape) {
      // Landscape orientation - spread controls horizontally
      document.getElementById('mass-controls').style.maxHeight = '80vh';
    } else {
      // Portrait orientation - stack controls vertically
      document.getElementById('mass-controls').style.maxHeight = '40vh';
    }
    
    // Force a render to update the scene
    renderScene();
  }, 300); // Short delay to ensure screen dimensions have updated
});
```

## Performance Considerations for Mobile

Mobile devices have limited resources, so we optimize for performance:

```javascript
function reduceSimulationComplexity() {
  // Reduce trail length very gradually
  if (trailLength > 100) {
    trailLength = Math.max(100, trailLength - 10);
    updateTrails();
  }
  
  // Reduce number of trail segments very gradually
  if (trailSegments > 50) {
    trailSegments = Math.max(50, trailSegments - 2);
    updateTrails();
  }
}

// Monitor FPS and adapt
function updateFPS() {
  frameCount++;
  const now = performance.now();
  if (now - lastFPSUpdate >= 1000) {
    currentFPS = Math.round((frameCount * 1000) / (now - lastFPSUpdate));
    frameCount = 0;
    lastFPSUpdate = now;
    
    // Only reduce complexity if FPS drops below 15
    if (currentFPS < 15 && !isPaused) {
      reduceSimulationComplexity();
    }
  }
}
```

## Device-Specific Fixes

Some devices need special handling:

```javascript
// Fix iOS/iPad slider handling
function fixIOSSliders() {
  // Check if this is an iOS device
  const isIOS = /iPad|iPhone|iPod/.test(navigator.userAgent) || 
               (navigator.platform === 'MacIntel' && navigator.maxTouchPoints > 1);
  
  if (isIOS) {
    const sliders = document.querySelectorAll('input[type="range"]');
    sliders.forEach(slider => {
      // Add explicit initialization touch handler
      slider.addEventListener('touchstart', function() {
        // This is intentionally empty to ensure touch behavior is initialized properly
      }, {passive: true});
    });
  }
}
```

## Adaptive Content

We adapt content based on available space:

```css
/* Ensure all buttons are visible */
#controls {
  max-width: 90% !important;
  overflow-y: auto;
  max-height: 70vh;
}

/* iPad-specific adjustments */
@media only screen and (min-device-width: 768px) and (max-device-width: 1024px) {
  #controls {
    top: 20px;
    left: 20px;
    max-width: 360px !important;
  }
  
  #mass-controls {
    top: 20px;
    right: 20px;
    max-width: 360px !important;
    max-height: 90vh;
  }
}
```

## UI Controls for Mobile

We provide appropriately sized controls:

```css
/* UI toggle positioned for easy access on mobile */
#ui-toggle {
  position: fixed;
  bottom: 15px;
  right: 15px;
  padding: 6px 10px;
  font-size: 12px;
}

#cinematic-toggle {
  bottom: 15px;
  right: 95px;
}
```

## Testing Responsive Design

When developing responsive designs:

1. **Test on real devices** whenever possible, not just emulators
2. **Test both orientations** (portrait and landscape)
3. **Try different interaction models** (mouse, touchpad, touch)
4. **Test at various zoom levels** to ensure the layout adapts appropriately
5. **Verify performance** on lower-end mobile devices

By applying these responsive design principles, our Three Body Simulator provides a great experience across desktop computers, tablets, and mobile phones. 