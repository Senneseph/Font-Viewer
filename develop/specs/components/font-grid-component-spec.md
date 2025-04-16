# FontGridComponent Specification

## Overview

The FontGridComponent is responsible for displaying a grid or list of available fonts. It provides a visual interface for browsing, searching, and selecting fonts, and supports drag-and-drop interactions.

## Responsibilities

1. Display a grid or list of available fonts
2. Support different view modes (grid or list)
3. Provide search and filter functionality
4. Enable font selection and drag-and-drop
5. Support sorting and pagination

## Interface

```typescript
interface FontGridOptions {
  viewMode: 'grid' | 'list';
  sortBy: 'name' | 'family' | 'lastModified';
  sortDirection: 'asc' | 'desc';
  filterBy?: FontFilterCriteria;
  searchQuery?: string;
  pageSize?: number;
  pageIndex?: number;
}

interface FontFilterCriteria {
  family?: string;
  style?: string;
  weight?: string;
  format?: string;
}

class FontGridComponent extends HTMLElement {
  /**
   * Set the fonts to display
   * @param fonts Array of font metadata
   */
  setFonts(fonts: FontMetadata[]): void;
  
  /**
   * Update display options
   * @param options The grid display options
   */
  setOptions(options: FontGridOptions): void;
  
  /**
   * Get the currently selected font
   * @returns The selected font or undefined if none selected
   */
  getSelectedFont(): FontMetadata | undefined;
  
  /**
   * Select a font by ID
   * @param id The font ID to select
   * @returns True if the font was found and selected
   */
  selectFontById(id: string): boolean;
  
  /**
   * Clear the current selection
   */
  clearSelection(): void;
  
  /**
   * Refresh the grid display
   */
  refresh(): void;
  
  /**
   * Add an event listener for font grid events
   * @param event The event name
   * @param callback The callback function
   */
  addEventListener(event: 'font-selected' | 'font-drag-start' | 'view-mode-changed' | 'sort-changed' | 'filter-changed' | 'search-changed' | 'page-changed', callback: Function): void;
  
  /**
   * Remove an event listener
   * @param event The event name
   * @param callback The callback function
   */
  removeEventListener(event: 'font-selected' | 'font-drag-start' | 'view-mode-changed' | 'sort-changed' | 'filter-changed' | 'search-changed' | 'page-changed', callback: Function): void;
}
```

## HTML Structure

```html
<font-grid>
  <div class="font-grid-toolbar">
    <div class="view-mode-controls">
      <button class="view-mode-button" data-mode="grid">Grid</button>
      <button class="view-mode-button" data-mode="list">List</button>
    </div>
    
    <div class="search-controls">
      <input type="text" class="search-input" placeholder="Search fonts...">
      <button class="search-button">Search</button>
    </div>
    
    <div class="sort-controls">
      <select class="sort-select">
        <option value="name">Name</option>
        <option value="family">Family</option>
        <option value="lastModified">Date Added</option>
      </select>
      <button class="sort-direction-button" data-direction="asc">↑</button>
    </div>
    
    <div class="filter-controls">
      <button class="filter-button">Filter</button>
      <div class="filter-dropdown">
        <!-- Filter options will be dynamically generated -->
      </div>
    </div>
  </div>
  
  <div class="font-grid-container">
    <!-- Font items will be dynamically generated -->
  </div>
  
  <div class="font-grid-pagination">
    <button class="pagination-prev">Previous</button>
    <span class="pagination-info">Page 1 of 10</span>
    <button class="pagination-next">Next</button>
  </div>
</font-grid>
```

## CSS Structure

```css
:host {
  display: block;
  width: 100%;
  height: 100%;
  overflow: hidden;
}

.font-grid-toolbar {
  display: flex;
  align-items: center;
  padding: 8px;
  border-bottom: 1px solid var(--border-color);
}

.font-grid-container {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 16px;
  padding: 16px;
  overflow-y: auto;
  height: calc(100% - 100px);
}

.font-grid-container.list-view {
  display: flex;
  flex-direction: column;
}

.font-item {
  display: flex;
  flex-direction: column;
  padding: 16px;
  border: 1px solid var(--border-color);
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.font-item:hover {
  border-color: var(--primary-color);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.font-item.selected {
  border-color: var(--primary-color);
  background-color: var(--primary-color-light);
}

.font-preview {
  height: 80px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 24px;
  overflow: hidden;
}

.font-name {
  margin-top: 8px;
  font-size: 14px;
  text-align: center;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.font-family {
  font-size: 12px;
  color: var(--text-secondary-color);
  text-align: center;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.font-grid-pagination {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 8px;
  border-top: 1px solid var(--border-color);
}

.pagination-info {
  margin: 0 16px;
}

/* List view styles */
.font-grid-container.list-view .font-item {
  flex-direction: row;
  align-items: center;
  margin-bottom: 8px;
}

.font-grid-container.list-view .font-preview {
  width: 100px;
  height: 40px;
  margin-right: 16px;
}

.font-grid-container.list-view .font-info {
  flex: 1;
}

.font-grid-container.list-view .font-name {
  text-align: left;
  margin-top: 0;
}

.font-grid-container.list-view .font-family {
  text-align: left;
}
```

## Implementation Details

### Font Item Generation

The FontGridComponent will generate font items using the following approach:

1. For each font in the provided array, create a font item element
2. Set the font preview text using the font itself
3. Display the font name and family
4. Make the font item draggable
5. Add event listeners for selection and drag start

### View Modes

The FontGridComponent will support two view modes:

1. **Grid View**: Display fonts in a grid layout with preview and name
2. **List View**: Display fonts in a list layout with preview, name, and additional information

### Search and Filter

The FontGridComponent will provide search and filter functionality:

1. **Search**: Filter fonts by name, family, or style based on the search query
2. **Filter**: Filter fonts by family, style, weight, or format
3. **Sort**: Sort fonts by name, family, or date added

### Pagination

The FontGridComponent will support pagination for large font collections:

1. Display a configurable number of fonts per page
2. Provide navigation controls for moving between pages
3. Show the current page and total pages

### Drag and Drop

The FontGridComponent will support drag and drop functionality:

1. Make font items draggable
2. Set the drag data to include the font ID and metadata
3. Provide visual feedback during drag operations
4. Emit events when drag operations start

### Event System

The FontGridComponent will emit the following events:

1. **font-selected**: Emitted when a font is selected
2. **font-drag-start**: Emitted when a font drag operation starts
3. **view-mode-changed**: Emitted when the view mode changes
4. **sort-changed**: Emitted when the sort order changes
5. **filter-changed**: Emitted when the filter criteria change
6. **search-changed**: Emitted when the search query changes
7. **page-changed**: Emitted when the current page changes

## Usage Examples

### Basic Usage

```html
<font-grid id="fontGrid"></font-grid>

<script>
  // Get the font grid element
  const fontGrid = document.getElementById('fontGrid');
  
  // Set the fonts to display
  fontGrid.setFonts(fonts);
  
  // Set display options
  fontGrid.setOptions({
    viewMode: 'grid',
    sortBy: 'name',
    sortDirection: 'asc',
    pageSize: 20
  });
  
  // Listen for font selection events
  fontGrid.addEventListener('font-selected', (event) => {
    const selectedFont = event.detail;
    console.log('Selected font:', selectedFont.name);
  });
</script>
```

### Search and Filter

```javascript
// Set search query
fontGrid.setOptions({
  ...fontGrid.getOptions(),
  searchQuery: 'Helvetica'
});

// Set filter criteria
fontGrid.setOptions({
  ...fontGrid.getOptions(),
  filterBy: {
    family: 'Helvetica',
    style: 'italic'
  }
});
```

### Drag and Drop

```javascript
// Listen for drag start events
fontGrid.addEventListener('font-drag-start', (event) => {
  const font = event.detail.font;
  const dragEvent = event.detail.dragEvent;
  
  console.log('Drag started for font:', font.name);
});
```

## Dependencies

The FontGridComponent will depend on the following:

1. **FontIndexService**: For retrieving font metadata
2. **DragDropManager**: For handling drag and drop operations
3. **EventEmitter**: For implementing the event system

## Testing Strategy

The FontGridComponent will be tested using the following approach:

1. **Unit Tests**: Test individual methods with mock data
2. **Integration Tests**: Test the component with the FontIndexService
3. **Visual Tests**: Test the component's appearance in different view modes
4. **Interaction Tests**: Test selection, drag and drop, and pagination

## Accessibility Considerations

The FontGridComponent will implement the following accessibility features:

1. **Keyboard Navigation**: Support for navigating the grid using keyboard
2. **ARIA Attributes**: Proper ARIA attributes for screen readers
3. **Focus Management**: Proper focus handling for interactive elements
4. **Color Contrast**: Sufficient color contrast for text and UI elements

## Performance Considerations

The FontGridComponent will implement the following performance optimizations:

1. **Virtualization**: Only render visible items for large font collections
2. **Lazy Loading**: Load font previews on demand
3. **Efficient Updates**: Only update changed elements when refreshing
4. **Debouncing**: Debounce search and filter operations to avoid excessive updates

## Future Enhancements

1. **Font Categories**: Group fonts by category
2. **Font Tags**: Support for tagging and filtering by tags
3. **Font Favorites**: Mark fonts as favorites
4. **Custom Preview Text**: Allow customizing the preview text for all fonts
