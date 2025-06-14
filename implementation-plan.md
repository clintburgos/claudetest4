# Isometric Grid Implementation Plan

## Overview
Test-driven implementation with small, focused commits. Each commit represents a single testable unit of functionality.

## Implementation Order

### Phase 1: Project Setup
1. **Set up TypeScript project with Jest testing framework**
   - Initialize npm project with TypeScript and Jest configs
   - Add .gitignore, tsconfig.json, jest.config.js

2. **Create Point interface and basic type definitions**
   - Define Point interface with x, y properties
   - Create types.ts file for shared interfaces

### Phase 2: IsometricGrid (Core Math)
3. **Add IsometricGrid constructor with tile dimensions**
   - Create IsometricGrid class with tileWidth/tileHeight
   - Write tests for constructor validation

4. **Implement IsometricGrid.gridToScreen with tests**
   - Add gridToScreen method converting grid coords to screen
   - Test with multiple grid positions including negatives

5. **Implement IsometricGrid.screenToGrid with tests**
   - Add screenToGrid method (inverse transformation)
   - Test that screenToGrid(gridToScreen(x,y)) ≈ (x,y)

6. **Implement IsometricGrid.getCellAt with tests**
   - Add getCellAt method that floors screenToGrid result
   - Test cell boundaries and edge cases

### Phase 3: Camera (Viewport Management)
7. **Create Camera constructor with viewport dimensions**
   - Initialize Camera with width, height, position (0,0), zoom (1)
   - Test initial state

8. **Implement Camera pan method with tests**
   - Add pan(deltaX, deltaY) method
   - Test cumulative panning

9. **Implement Camera zoom methods with tests**
   - Add setZoom(zoom) and getZoom() methods
   - Test zoom limits and validation

10. **Implement Camera position getters/setters with tests**
    - Add getPosition() and setPosition(x, y)
    - Test position updates

11. **Implement Camera.worldToViewport with tests**
    - Transform world coordinates to viewport space
    - Test with different camera positions and zoom levels

12. **Implement Camera.viewportToWorld with tests**
    - Transform viewport coordinates to world space
    - Test inverse relationship with worldToViewport

### Phase 4: GridRenderer (Drawing)
13. **Create GridRenderer constructor with canvas**
    - Initialize with HTMLCanvasElement and get 2D context
    - Mock canvas for testing

14. **Implement GridRenderer.clear with tests**
    - Add clear() method to clear entire canvas
    - Test that canvas clear is called

15. **Implement GridRenderer.drawTile with tests**
    - Draw diamond-shaped tile at screen coordinates
    - Test path drawing calls

16. **Implement GridRenderer.drawGridLines with tests**
    - Draw grid lines for debugging
    - Test line drawing between grid cells

### Phase 5: IsometricViewport (Integration)
17. **Create IsometricViewport constructor**
    - Compose IsometricGrid, Camera, and GridRenderer
    - Test component initialization

18. **Implement IsometricViewport pan methods with tests**
    - Add startPan, updatePan, endPan for drag handling
    - Test pan state management

19. **Implement IsometricViewport.zoom with tests**
    - Add zoom method with focal point
    - Test zoom around mouse position

20. **Implement IsometricViewport.getGridCellAt with tests**
    - Convert viewport coords to grid cell
    - Test coordinate transformation chain

21. **Implement IsometricViewport.render with tests**
    - Calculate visible tiles and render them
    - Test render calls with mocked dependencies

### Phase 6: Demo
22. **Create basic HTML demo page**
    - Add index.html with canvas element
    - Include compiled JavaScript

23. **Add mouse event handlers to demo**
    - Wire up mouse events for pan and zoom
    - Test interaction in browser

24. **Add animation loop to demo**
    - Add requestAnimationFrame loop
    - Display working isometric grid

## Testing Strategy

### Unit Test Structure
```typescript
describe('IsometricGrid', () => {
  describe('gridToScreen', () => {
    it('should convert origin (0,0) to screen center', () => {
      // Arrange
      const grid = new IsometricGrid(64, 32);
      
      // Act
      const result = grid.gridToScreen(0, 0);
      
      // Assert
      expect(result).toEqual({ x: 0, y: 0 });
    });
  });
});
```

### Mock Strategy
- Mock HTMLCanvasElement for GridRenderer tests
- Use test doubles for dependencies in IsometricViewport
- Verify method calls rather than visual output

## Commit Message Format
```
<type>: <description>

[optional body]
```

Types: feat, test, refactor, docs, chore

Examples:
- `feat: add IsometricGrid.gridToScreen method`
- `test: add tests for Camera pan behavior`
- `chore: set up TypeScript and Jest configuration`