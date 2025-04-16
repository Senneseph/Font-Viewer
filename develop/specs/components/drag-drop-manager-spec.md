# DragDropManager Specification

## Overview

The DragDropManager is responsible for handling drag and drop operations in the Font-Viewer application. It provides a unified interface for making elements draggable and defining drop zones, and handles the transfer of data between them.

## Responsibilities

1. Make elements draggable with associated data
2. Define drop zones and handle drop events
3. Provide visual feedback during drag operations
4. Manage the transfer of data between draggable elements and drop zones
5. Support custom drag and drop behaviors

## Interface

```typescript
interface DragOptions {
  /**
   * The data to transfer during drag operations
   */
  data: any;
  
  /**
   * The drag image to use (optional)
   */
  dragImage?: HTMLElement;
  
  /**
   * The offset of the drag image (optional)
   */
  dragImageOffset?: { x: number, y: number };
  
  /**
   * Callback when drag starts
   */
  onDragStart?: (event: DragEvent) => void;
  
  /**
   * Callback when drag ends
   */
  onDragEnd?: (event: DragEvent) => void;
}

interface DropZoneOptions {
  /**
   * Callback when an item is dropped
   */
  onDrop: (data: any, event: DragEvent) => void;
  
  /**
   * Callback when an item is dragged over
   */
  onDragOver?: (event: DragEvent) => void;
  
  /**
   * Callback when an item is dragged into the zone
   */
  onDragEnter?: (event: DragEvent) => void;
  
  /**
   * Callback when an item is dragged out of the zone
   */
  onDragLeave?: (event: DragEvent) => void;
  
  /**
   * Function to determine if the drop is allowed
   */
  isDropAllowed?: (data: any, event: DragEvent) => boolean;
}

class DragDropManager {
  /**
   * Make an element draggable
   * @param element The element to make draggable
   * @param options The drag options
   * @returns A function to remove the draggable behavior
   */
  makeDraggable(element: HTMLElement, options: DragOptions): () => void;
  
  /**
   * Make an element a drop zone
   * @param element The element to make a drop zone
   * @param options The drop zone options
   * @returns A function to remove the drop zone behavior
   */
  makeDropZone(element: HTMLElement, options: DropZoneOptions): () => void;
  
  /**
   * Get the data being dragged
   * @returns The data being dragged or null if no drag is in progress
   */
  getDragData(): any | null;
  
  /**
   * Check if a drag operation is in progress
   * @returns True if a drag operation is in progress
   */
  isDragging(): boolean;
  
  /**
   * Add a global drag event listener
   * @param event The event name
   * @param callback The callback function
   */
  addEventListener(event: 'drag-start' | 'drag-end' | 'drop', callback: Function): void;
  
  /**
   * Remove a global drag event listener
   * @param event The event name
   * @param callback The callback function
   */
  removeEventListener(event: 'drag-start' | 'drag-end' | 'drop', callback: Function): void;
}
```

## Implementation Details

### Making Elements Draggable

The DragDropManager will make elements draggable using the following approach:

1. Add the `draggable` attribute to the element
2. Add event listeners for `dragstart` and `dragend` events
3. Set the drag data using `dataTransfer.setData`
4. Optionally set a custom drag image using `dataTransfer.setDragImage`
5. Store the drag data in the manager for access during the drag operation

```javascript
makeDraggable(element, options) {
  // Make the element draggable
  element.setAttribute('draggable', 'true');
  
  // Add dragstart event listener
  const dragStartHandler = (event) => {
    // Set the drag data
    event.dataTransfer.setData('application/json', JSON.stringify(options.data));
    
    // Set the drag effect
    event.dataTransfer.effectAllowed = 'copy';
    
    // Set a custom drag image if provided
    if (options.dragImage) {
      event.dataTransfer.setDragImage(
        options.dragImage,
        options.dragImageOffset?.x || 0,
        options.dragImageOffset?.y || 0
      );
    }
    
    // Store the drag data
    this._currentDragData = options.data;
    
    // Add dragging class to the element
    element.classList.add('dragging');
    
    // Call the onDragStart callback if provided
    if (options.onDragStart) {
      options.onDragStart(event);
    }
    
    // Emit the drag-start event
    this._emitEvent('drag-start', { data: options.data, event });
  };
  
  // Add dragend event listener
  const dragEndHandler = (event) => {
    // Clear the drag data
    this._currentDragData = null;
    
    // Remove dragging class from the element
    element.classList.remove('dragging');
    
    // Call the onDragEnd callback if provided
    if (options.onDragEnd) {
      options.onDragEnd(event);
    }
    
    // Emit the drag-end event
    this._emitEvent('drag-end', { event });
  };
  
  // Add the event listeners
  element.addEventListener('dragstart', dragStartHandler);
  element.addEventListener('dragend', dragEndHandler);
  
  // Return a function to remove the draggable behavior
  return () => {
    element.removeAttribute('draggable');
    element.removeEventListener('dragstart', dragStartHandler);
    element.removeEventListener('dragend', dragEndHandler);
    element.classList.remove('dragging');
  };
}
```

### Creating Drop Zones

The DragDropManager will create drop zones using the following approach:

1. Add event listeners for `dragover`, `dragenter`, `dragleave`, and `drop` events
2. Prevent the default behavior to allow drops
3. Add visual feedback during drag operations
4. Extract the drag data from the `dataTransfer` object
5. Call the appropriate callbacks when events occur

```javascript
makeDropZone(element, options) {
  // Add dragover event listener
  const dragOverHandler = (event) => {
    // Prevent default to allow drop
    event.preventDefault();
    
    // Get the drag data
    const data = this.getDragData();
    
    // Check if drop is allowed
    if (options.isDropAllowed && !options.isDropAllowed(data, event)) {
      event.dataTransfer.dropEffect = 'none';
      return;
    }
    
    // Set the drop effect
    event.dataTransfer.dropEffect = 'copy';
    
    // Call the onDragOver callback if provided
    if (options.onDragOver) {
      options.onDragOver(event);
    }
  };
  
  // Add dragenter event listener
  const dragEnterHandler = (event) => {
    // Prevent default to allow drop
    event.preventDefault();
    
    // Get the drag data
    const data = this.getDragData();
    
    // Check if drop is allowed
    if (options.isDropAllowed && !options.isDropAllowed(data, event)) {
      return;
    }
    
    // Add drag-over class to the element
    element.classList.add('drag-over');
    
    // Call the onDragEnter callback if provided
    if (options.onDragEnter) {
      options.onDragEnter(event);
    }
  };
  
  // Add dragleave event listener
  const dragLeaveHandler = (event) => {
    // Remove drag-over class from the element
    element.classList.remove('drag-over');
    
    // Call the onDragLeave callback if provided
    if (options.onDragLeave) {
      options.onDragLeave(event);
    }
  };
  
  // Add drop event listener
  const dropHandler = (event) => {
    // Prevent default to handle drop
    event.preventDefault();
    
    // Remove drag-over class from the element
    element.classList.remove('drag-over');
    
    // Get the drag data
    let data;
    try {
      const jsonData = event.dataTransfer.getData('application/json');
      data = JSON.parse(jsonData);
    } catch (error) {
      console.error('Error parsing drag data:', error);
      return;
    }
    
    // Check if drop is allowed
    if (options.isDropAllowed && !options.isDropAllowed(data, event)) {
      return;
    }
    
    // Call the onDrop callback
    options.onDrop(data, event);
    
    // Emit the drop event
    this._emitEvent('drop', { data, event });
  };
  
  // Add the event listeners
  element.addEventListener('dragover', dragOverHandler);
  element.addEventListener('dragenter', dragEnterHandler);
  element.addEventListener('dragleave', dragLeaveHandler);
  element.addEventListener('drop', dropHandler);
  
  // Return a function to remove the drop zone behavior
  return () => {
    element.removeEventListener('dragover', dragOverHandler);
    element.removeEventListener('dragenter', dragEnterHandler);
    element.removeEventListener('dragleave', dragLeaveHandler);
    element.removeEventListener('drop', dropHandler);
    element.classList.remove('drag-over');
  };
}
```

### Event System

The DragDropManager will implement an event system for global drag and drop events:

1. **drag-start**: Emitted when a drag operation starts
2. **drag-end**: Emitted when a drag operation ends
3. **drop**: Emitted when an item is dropped

```javascript
_eventListeners = {
  'drag-start': [],
  'drag-end': [],
  'drop': []
};

addEventListener(event, callback) {
  if (!this._eventListeners[event]) {
    this._eventListeners[event] = [];
  }
  
  this._eventListeners[event].push(callback);
}

removeEventListener(event, callback) {
  if (!this._eventListeners[event]) {
    return;
  }
  
  this._eventListeners[event] = this._eventListeners[event].filter(
    (cb) => cb !== callback
  );
}

_emitEvent(event, data) {
  if (!this._eventListeners[event]) {
    return;
  }
  
  for (const callback of this._eventListeners[event]) {
    callback(data);
  }
}
```

## Usage Examples

### Making an Element Draggable

```javascript
const dragDropManager = new DragDropManager();

// Get the element to make draggable
const fontItem = document.getElementById('font-item');

// Make the element draggable
const removeDraggable = dragDropManager.makeDraggable(fontItem, {
  data: { id: 'font-1', name: 'Helvetica' },
  onDragStart: (event) => {
    console.log('Drag started');
  },
  onDragEnd: (event) => {
    console.log('Drag ended');
  }
});

// Later, to remove the draggable behavior
removeDraggable();
```

### Creating a Drop Zone

```javascript
// Get the element to make a drop zone
const previewArea = document.getElementById('preview-area');

// Make the element a drop zone
const removeDropZone = dragDropManager.makeDropZone(previewArea, {
  onDrop: (data, event) => {
    console.log('Font dropped:', data.name);
    // Update the preview with the dropped font
    updatePreview(data.id);
  },
  onDragEnter: (event) => {
    console.log('Drag entered preview area');
  },
  onDragLeave: (event) => {
    console.log('Drag left preview area');
  },
  isDropAllowed: (data, event) => {
    // Only allow dropping fonts
    return data && data.id && data.name;
  }
});

// Later, to remove the drop zone behavior
removeDropZone();
```

### Listening for Global Events

```javascript
// Listen for drag start events
dragDropManager.addEventListener('drag-start', (data) => {
  console.log('Drag started with data:', data);
});

// Listen for drop events
dragDropManager.addEventListener('drop', (data) => {
  console.log('Item dropped with data:', data);
});
```

## Dependencies

The DragDropManager has no external dependencies and can be used as a standalone utility.

## Testing Strategy

The DragDropManager will be tested using the following approach:

1. **Unit Tests**: Test individual methods with mock events
2. **Integration Tests**: Test with actual DOM elements
3. **Event Tests**: Test the event system
4. **Edge Cases**: Test error handling and edge cases

## Browser Compatibility

The DragDropManager will be compatible with the following browsers:

1. **Chrome**: Version 58+
2. **Firefox**: Version 52+
3. **Safari**: Version 11+
4. **Edge**: Version 16+

## Performance Considerations

The DragDropManager will implement the following performance optimizations:

1. **Event Delegation**: Use event delegation where appropriate
2. **Cleanup**: Properly clean up event listeners when removing behaviors
3. **Throttling**: Throttle event handlers for dragover events

## Future Enhancements

1. **Touch Support**: Add support for touch devices
2. **Multi-Item Drag**: Support dragging multiple items at once
3. **Drag Preview Customization**: More options for customizing drag previews
4. **Drop Effects**: Support for different drop effects (copy, move, link)
