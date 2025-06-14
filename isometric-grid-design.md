# Isometric Grid System - High Level Design

## Overview

A minimal isometric grid system with pan and zoom capabilities. Each class has a single, clear responsibility.

## Core Classes

### 1. IsometricGrid
**Responsibility**: Convert between grid coordinates and screen coordinates

```typescript
interface Point {
  x: number;
  y: number;
}

class IsometricGrid {
  constructor(tileWidth: number, tileHeight: number) {}
  
  // Convert grid position to screen position
  gridToScreen(gridX: number, gridY: number): Point
  
  // Convert screen position to grid position
  screenToGrid(screenX: number, screenY: number): Point
  
  // Get the grid cell at a screen position
  getCellAt(screenX: number, screenY: number): Point
}
```

### 2. Camera
**Responsibility**: Manage viewport position and zoom level

```typescript
class Camera {
  constructor(viewportWidth: number, viewportHeight: number) {}
  
  // Pan the camera by delta amounts
  pan(deltaX: number, deltaY: number): void
  
  // Set zoom level (1.0 = 100%)
  setZoom(zoom: number): void
  
  // Get current zoom level
  getZoom(): number
  
  // Get camera position
  getPosition(): Point
  
  // Set camera position
  setPosition(x: number, y: number): void
  
  // Convert world coordinates to viewport coordinates
  worldToViewport(worldX: number, worldY: number): Point
  
  // Convert viewport coordinates to world coordinates
  viewportToWorld(viewportX: number, viewportY: number): Point
}
```

### 3. GridRenderer
**Responsibility**: Draw the grid tiles to a canvas

```typescript
class GridRenderer {
  constructor(canvas: HTMLCanvasElement) {}
  
  // Clear the canvas
  clear(): void
  
  // Draw a single tile
  drawTile(x: number, y: number, color: string): void
  
  // Draw grid lines
  drawGridLines(startX: number, startY: number, endX: number, endY: number): void
}
```

### 4. IsometricViewport
**Responsibility**: Orchestrate the grid, camera, and renderer

```typescript
class IsometricViewport {
  constructor(canvas: HTMLCanvasElement, tileWidth: number, tileHeight: number) {}
  
  // Render the visible portion of the grid
  render(): void
  
  // Handle mouse/touch input for panning
  startPan(x: number, y: number): void
  updatePan(x: number, y: number): void
  endPan(): void
  
  // Handle zoom input
  zoom(delta: number, centerX: number, centerY: number): void
  
  // Get grid cell at viewport position
  getGridCellAt(viewportX: number, viewportY: number): Point
}
```

## Data Flow

```
User Input
    ↓
IsometricViewport
    ↓
Camera (transforms coordinates)
    ↓
IsometricGrid (converts grid ↔ screen)
    ↓
GridRenderer (draws to canvas)
```

## Usage Example

```typescript
// Initialize
const canvas = document.getElementById('canvas') as HTMLCanvasElement;
const viewport = new IsometricViewport(canvas, 64, 32);

// Handle mouse events
canvas.addEventListener('mousedown', (e) => {
  viewport.startPan(e.clientX, e.clientY);
});

canvas.addEventListener('mousemove', (e) => {
  viewport.updatePan(e.clientX, e.clientY);
});

canvas.addEventListener('mouseup', () => {
  viewport.endPan();
});

canvas.addEventListener('wheel', (e) => {
  viewport.zoom(e.deltaY, e.clientX, e.clientY);
});

// Render loop
function animate() {
  viewport.render();
  requestAnimationFrame(animate);
}
animate();
```

## Key Design Decisions

1. **Separation of Concerns**
   - Grid math is isolated in `IsometricGrid`
   - Camera state is isolated in `Camera`
   - Drawing logic is isolated in `GridRenderer`
   - User interaction is handled by `IsometricViewport`

2. **Immutable Coordinates**
   - All coordinate transformations return new Point objects
   - No side effects in transformation methods

3. **Canvas-Only Rendering**
   - Simple 2D canvas API for initial implementation
   - No WebGL complexity

4. **No Optimization**
   - No culling
   - No batching
   - No caching
   - Redraw everything each frame

5. **Fixed Grid**
   - No dynamic tile loading
   - Infinite grid assumed
   - All tiles are the same

## Minimal Implementation Size

- ~50 lines for IsometricGrid
- ~40 lines for Camera  
- ~30 lines for GridRenderer
- ~60 lines for IsometricViewport
- Total: ~180 lines of code