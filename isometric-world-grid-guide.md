# Isometric World Grid: Complete Implementation Guide

## Overview

This document provides a comprehensive guide for implementing an isometric world grid system, including coordinate transformations, camera positioning, and depth sorting algorithms.

## 1. Coordinate System Fundamentals

### Understanding Isometric Projection

Isometric projection creates a 3D-like view using 2D graphics by rotating the viewpoint 45° horizontally and approximately 30° vertically. This creates the characteristic diamond-shaped tiles where:
- Moving right in grid space moves diagonally down-right on screen
- Moving down in grid space moves diagonally down-left on screen

### Coordinate Transformation Formulas

#### Grid to Screen Transformation

```javascript
function gridToScreen(gridX, gridY, tileWidth, tileHeight) {
    const screenX = (gridX - gridY) * (tileWidth / 2);
    const screenY = (gridX + gridY) * (tileHeight / 2);
    return { x: screenX, y: screenY };
}
```

#### Screen to Grid Transformation

```javascript
function screenToGrid(screenX, screenY, tileWidth, tileHeight) {
    const gridX = (screenX / (tileWidth / 2) + screenY / (tileHeight / 2)) / 2;
    const gridY = (screenY / (tileHeight / 2) - screenX / (tileWidth / 2)) / 2;
    return { x: gridX, y: gridY };
}
```

#### Matrix Representation

The transformation can be represented as matrix multiplication:

**Grid to Screen Matrix:**
```
⎡ Tw/2  -Tw/2 ⎤
⎣ Th/2   Th/2 ⎦
```

**Screen to Grid Matrix (Inverse):**
```
⎡  1/Tw    1/Th ⎤
⎣ -1/Tw    1/Th ⎦
```

### Important Implementation Notes

1. **Use Floating-Point Arithmetic**: Always use floats for calculations to maintain precision
2. **Standard Tile Ratio**: Most isometric games use a 2:1 width-to-height ratio for tiles
3. **Sub-tile Positioning**: These formulas work for fractional grid positions (e.g., 2.5, 1.5)
4. **Origin Point**: (0,0) in grid space typically maps to the top corner of the isometric diamond

## 2. Camera Positioning and Management

### Camera Angle Configuration

The standard isometric camera setup uses:
- **Horizontal Rotation**: 45 degrees
- **Vertical Rotation**: 30 degrees (for 2:1 tiles) or 35.264 degrees (true isometric)
- **Projection Type**: Orthographic (no perspective distortion)

### Camera Centering

To center the camera on a specific grid position:

```javascript
function centerCameraOnTile(gridX, gridY, tileWidth, tileHeight, viewportWidth, viewportHeight) {
    // Convert grid position to screen coordinates
    const tileScreen = gridToScreen(gridX, gridY, tileWidth, tileHeight);
    
    // Calculate camera offset to center the tile
    const cameraX = tileScreen.x - viewportWidth / 2;
    const cameraY = tileScreen.y - viewportHeight / 2;
    
    return { x: cameraX, y: cameraY };
}
```

### Finding the Starting Tile Position

To determine which tile should be drawn first (top-left of viewport):

```javascript
function getViewportStartTile(cameraX, cameraY, tileWidth, tileHeight) {
    // Convert camera position to approximate grid coordinates
    const startGrid = screenToGrid(cameraX, cameraY, tileWidth, tileHeight);
    
    // Round down to get the tile index
    return {
        x: Math.floor(startGrid.x),
        y: Math.floor(startGrid.y)
    };
}
```

### Camera Best Practices

1. **Maintain Consistent Height**: Keep camera height constant to avoid disorienting players
2. **Smooth Movement**: Implement camera smoothing/lerping for better feel
3. **Boundary Constraints**: Prevent camera from showing areas outside the game world
4. **Focus Points**: Use a "look at" system for dynamic camera targets

## 3. Depth Sorting and Rendering Order

### Basic Z-Order Calculation

The simplest depth sorting formula:

```javascript
function calculateZOrder(gridX, gridY) {
    return gridX + gridY;
}
```

This works well for flat tiles and simple objects occupying single tiles.

### Advanced Sorting for Complex Scenes

#### Painter's Algorithm Implementation

```javascript
function sortIsometricObjects(objects) {
    return objects.sort((a, b) => {
        // Primary sort by combined coordinates
        const zOrderA = a.gridX + a.gridY;
        const zOrderB = b.gridX + b.gridY;
        
        if (zOrderA !== zOrderB) {
            return zOrderA - zOrderB;
        }
        
        // Secondary sort by height (for stacked objects)
        return a.z - b.z;
    });
}
```

#### Topological Sorting for Complex Dependencies

For objects spanning multiple tiles or with complex occlusion relationships:

```javascript
class IsometricDepthSorter {
    constructor() {
        this.dependencies = new Map();
    }
    
    addDependency(objectA, objectB) {
        if (!this.dependencies.has(objectA)) {
            this.dependencies.set(objectA, new Set());
        }
        this.dependencies.get(objectA).add(objectB);
    }
    
    // Check if objectA should be drawn before objectB
    shouldDrawBefore(objectA, objectB) {
        // Check spatial relationship
        if (objectA.gridX + objectA.width <= objectB.gridX ||
            objectA.gridY + objectA.height <= objectB.gridY) {
            return true;
        }
        return false;
    }
    
    topologicalSort(objects) {
        // Build dependency graph
        for (let i = 0; i < objects.length; i++) {
            for (let j = 0; j < objects.length; j++) {
                if (i !== j && this.shouldDrawBefore(objects[i], objects[j])) {
                    this.addDependency(objects[i], objects[j]);
                }
            }
        }
        
        // Perform topological sort (simplified version)
        const sorted = [];
        const visited = new Set();
        
        const visit = (object) => {
            if (visited.has(object)) return;
            visited.add(object);
            
            const deps = this.dependencies.get(object) || new Set();
            deps.forEach(dep => visit(dep));
            
            sorted.push(object);
        };
        
        objects.forEach(obj => visit(obj));
        return sorted;
    }
}
```

### Handling Multi-Tile Objects

For objects spanning multiple tiles, consider these approaches:

1. **Tile Slicing**: Break large sprites into 1x1 tile pieces
2. **Anchor Points**: Use a single reference tile for sorting
3. **Bounding Box Sorting**: Sort based on the furthest corner

```javascript
function getMultiTileZOrder(object) {
    // Use the furthest tile position for sorting
    const farX = object.gridX + object.tilesWide - 1;
    const farY = object.gridY + object.tilesHigh - 1;
    return farX + farY;
}
```

## 4. Practical Implementation Example

Here's a complete example combining all concepts:

```javascript
class IsometricWorld {
    constructor(tileWidth, tileHeight) {
        this.tileWidth = tileWidth;
        this.tileHeight = tileHeight;
        this.tileWidthHalf = tileWidth / 2;
        this.tileHeightHalf = tileHeight / 2;
    }
    
    // Transform grid to screen coordinates
    gridToScreen(gridX, gridY) {
        return {
            x: (gridX - gridY) * this.tileWidthHalf,
            y: (gridX + gridY) * this.tileHeightHalf
        };
    }
    
    // Transform screen to grid coordinates
    screenToGrid(screenX, screenY) {
        return {
            x: (screenX / this.tileWidthHalf + screenY / this.tileHeightHalf) / 2,
            y: (screenY / this.tileHeightHalf - screenX / this.tileWidthHalf) / 2
        };
    }
    
    // Get tiles visible in viewport
    getVisibleTiles(cameraX, cameraY, viewportWidth, viewportHeight) {
        // Calculate viewport corners in grid space
        const topLeft = this.screenToGrid(cameraX, cameraY);
        const bottomRight = this.screenToGrid(
            cameraX + viewportWidth, 
            cameraY + viewportHeight
        );
        
        // Expand bounds to ensure we don't miss edge tiles
        const startX = Math.floor(topLeft.x) - 1;
        const startY = Math.floor(topLeft.y) - 1;
        const endX = Math.ceil(bottomRight.x) + 1;
        const endY = Math.ceil(bottomRight.y) + 1;
        
        const visibleTiles = [];
        
        // Collect tiles in drawing order
        for (let y = startY; y <= endY; y++) {
            for (let x = startX; x <= endX; x++) {
                const screen = this.gridToScreen(x, y);
                visibleTiles.push({
                    gridX: x,
                    gridY: y,
                    screenX: screen.x - cameraX,
                    screenY: screen.y - cameraY,
                    zOrder: x + y
                });
            }
        }
        
        // Sort by depth
        return visibleTiles.sort((a, b) => a.zOrder - b.zOrder);
    }
}
```

## 5. Performance Optimization Tips

### Culling and Visibility

1. **Frustum Culling**: Only process tiles within the viewport
2. **Dirty Rectangle**: Track changed areas to minimize redraws
3. **Tile Caching**: Pre-render static tile combinations

### Batch Rendering

```javascript
// Group tiles by texture/sprite sheet
function batchTilesByTexture(tiles) {
    const batches = new Map();
    
    tiles.forEach(tile => {
        const texture = tile.texture;
        if (!batches.has(texture)) {
            batches.set(texture, []);
        }
        batches.get(texture).push(tile);
    });
    
    return batches;
}
```

### Spatial Indexing

For large worlds, implement spatial data structures:

```javascript
class SpatialHash {
    constructor(cellSize) {
        this.cellSize = cellSize;
        this.cells = new Map();
    }
    
    getCell(x, y) {
        const cellX = Math.floor(x / this.cellSize);
        const cellY = Math.floor(y / this.cellSize);
        return `${cellX},${cellY}`;
    }
    
    insert(object) {
        const cell = this.getCell(object.gridX, object.gridY);
        if (!this.cells.has(cell)) {
            this.cells.set(cell, []);
        }
        this.cells.get(cell).push(object);
    }
    
    query(minX, minY, maxX, maxY) {
        const objects = [];
        
        for (let x = minX; x <= maxX; x += this.cellSize) {
            for (let y = minY; y <= maxY; y += this.cellSize) {
                const cell = this.getCell(x, y);
                if (this.cells.has(cell)) {
                    objects.push(...this.cells.get(cell));
                }
            }
        }
        
        return objects;
    }
}
```

## 6. Common Pitfalls and Solutions

### Issue: Depth Sorting Artifacts
**Solution**: Use sub-pixel precision and consistent rounding rules

### Issue: Performance with Large Worlds
**Solution**: Implement level-of-detail (LOD) and chunk-based loading

### Issue: Mouse Picking Inaccuracy
**Solution**: Account for tile overlap and use precise hit testing

```javascript
function getTileAtScreenPos(screenX, screenY, cameraX, cameraY, tileWidth, tileHeight) {
    // Convert screen to world coordinates
    const worldX = screenX + cameraX;
    const worldY = screenY + cameraY;
    
    // Convert to grid coordinates
    const grid = screenToGrid(worldX, worldY, tileWidth, tileHeight);
    
    // Return the tile coordinates
    return {
        x: Math.floor(grid.x),
        y: Math.floor(grid.y)
    };
}
```

## Conclusion

Implementing an isometric world grid requires careful attention to:
1. Mathematical precision in coordinate transformations
2. Proper camera setup and management
3. Robust depth sorting algorithms
4. Performance optimization strategies

By following these guidelines and understanding the underlying mathematics, you can create efficient and visually correct isometric game worlds.