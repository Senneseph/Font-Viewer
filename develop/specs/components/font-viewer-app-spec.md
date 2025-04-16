# FontViewerApp Specification

## Overview

The FontViewerApp is the main application component that orchestrates all other components in the Font-Viewer application. It serves as the entry point and provides the overall structure and layout of the application.

## Responsibilities

1. Initialize and coordinate all application components
2. Manage the application state and data flow
3. Handle user interactions and events
4. Provide the overall layout and structure
5. Integrate with the Cyberdeck ecosystem

## Interface

```typescript
class FontViewerApp extends HTMLElement {
  /**
   * Initialize the application
   * @returns Promise that resolves when initialization is complete
   */
  initialize(): Promise<void>;
  
  /**
   * Refresh the font index
   * @returns Promise that resolves when refresh is complete
   */
  refreshFonts(): Promise<void>;
  
  /**
   * Select a font for preview
   * @param fontId The ID of the font to select
   */
  selectFont(fontId: string): void;
  
  /**
   * Compare multiple fonts
   * @param fontIds Array of font IDs to compare
   */
  compareFonts(fontIds: string[]): void;
  
  /**
   * Add a font to favorites
   * @param fontId The ID of the font to add to favorites
   */
  addToFavorites(fontId: string): void;
  
  /**
   * Remove a font from favorites
   * @param fontId The ID of the font to remove from favorites
   */
  removeFromFavorites(fontId: string): void;
  
  /**
   * Add an event listener for application events
   * @param event The event name
   * @param callback The callback function
   */
  addEventListener(event: 'fonts-refreshed' | 'font-selected' | 'fonts-compared' | 'favorite-added' | 'favorite-removed', callback: Function): void;
  
  /**
   * Remove an event listener
   * @param event The event name
   * @param callback The callback function
   */
  removeEventListener(event: 'fonts-refreshed' | 'font-selected' | 'fonts-compared' | 'favorite-added' | 'favorite-removed', callback: Function): void;
}
```

## HTML Structure

```html
<font-viewer-app>
  <header class="app-header">
    <div class="app-title">
      <h1>Font Viewer</h1>
    </div>
    
    <div class="app-actions">
      <button id="refresh-button" class="action-button">
        <span class="icon">↻</span> Refresh Fonts
      </button>
    </div>
  </header>
  
  <main class="app-content">
    <div class="app-sidebar">
      <font-grid id="font-grid"></font-grid>
    </div>
    
    <div class="app-main">
      <font-preview id="font-preview"></font-preview>
    </div>
    
    <div class="app-sidebar-right">
      <div class="collections-panel">
        <h2>Collections</h2>
        <div id="collections-container"></div>
      </div>
      
      <div class="font-details-panel">
        <h2>Font Details</h2>
        <div id="font-details-container"></div>
      </div>
    </div>
  </main>
  
  <footer class="app-footer">
    <p>Font Viewer - Part of the Cyberdeck Ecosystem</p>
  </footer>
</font-viewer-app>
```

## CSS Structure

```css
:host {
  display: flex;
  flex-direction: column;
  width: 100%;
  height: 100%;
  overflow: hidden;
  --primary-color: #3498db;
  --primary-color-light: #ebf5fb;
  --secondary-color: #2ecc71;
  --text-color: #333333;
  --text-secondary-color: #7f8c8d;
  --background-color: #ffffff;
  --border-color: #e0e0e0;
  --panel-background: #f5f5f5;
}

.app-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 16px;
  height: 60px;
  background-color: var(--primary-color);
  color: white;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.app-title h1 {
  margin: 0;
  font-size: 24px;
  font-weight: 500;
}

.app-actions {
  display: flex;
  align-items: center;
}

.action-button {
  display: flex;
  align-items: center;
  padding: 8px 16px;
  background-color: rgba(255, 255, 255, 0.2);
  border: none;
  border-radius: 4px;
  color: white;
  font-size: 14px;
  cursor: pointer;
  transition: background-color 0.2s ease;
}

.action-button:hover {
  background-color: rgba(255, 255, 255, 0.3);
}

.action-button .icon {
  margin-right: 8px;
}

.app-content {
  display: flex;
  flex: 1;
  overflow: hidden;
}

.app-sidebar {
  width: 300px;
  border-right: 1px solid var(--border-color);
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

.app-main {
  flex: 1;
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

.app-sidebar-right {
  width: 300px;
  border-left: 1px solid var(--border-color);
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

.collections-panel, .font-details-panel {
  padding: 16px;
  border-bottom: 1px solid var(--border-color);
}

.collections-panel h2, .font-details-panel h2 {
  margin: 0 0 16px 0;
  font-size: 18px;
  font-weight: 500;
}

.app-footer {
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: var(--panel-background);
  border-top: 1px solid var(--border-color);
  color: var(--text-secondary-color);
  font-size: 12px;
}

/* Responsive layout */
@media (max-width: 1200px) {
  .app-content {
    flex-direction: column;
  }
  
  .app-sidebar, .app-sidebar-right {
    width: 100%;
    height: 300px;
    border-right: none;
    border-left: none;
    border-bottom: 1px solid var(--border-color);
  }
}

/* Dark theme */
:host(.dark-theme) {
  --primary-color: #2980b9;
  --primary-color-light: #1c4966;
  --secondary-color: #27ae60;
  --text-color: #f5f5f5;
  --text-secondary-color: #bdc3c7;
  --background-color: #2c3e50;
  --border-color: #34495e;
  --panel-background: #1a2530;
  
  background-color: var(--background-color);
  color: var(--text-color);
}
```

## Implementation Details

### Component Initialization

The FontViewerApp will initialize all required components and services:

1. **FontIndexService**: For scanning and indexing fonts
2. **FontCollectionService**: For managing font collections
3. **DragDropManager**: For handling drag and drop operations
4. **FontGridComponent**: For displaying the font list/grid
5. **FontPreviewComponent**: For previewing selected fonts
6. **CollectionsComponent**: For displaying and managing collections

```javascript
async initialize() {
  try {
    // Initialize services
    this.fontIndexService = new FontIndexService();
    this.fontCollectionService = new FontCollectionService();
    this.dragDropManager = new DragDropManager();
    
    // Initialize components
    this.fontGrid = this.querySelector('#font-grid');
    this.fontPreview = this.querySelector('#font-preview');
    
    // Initialize collections container
    this.collectionsContainer = this.querySelector('#collections-container');
    this.fontDetailsContainer = this.querySelector('#font-details-container');
    
    // Set up event listeners
    this._setupEventListeners();
    
    // Refresh fonts
    await this.refreshFonts();
    
    // Load collections
    await this._loadCollections();
    
    // Emit initialized event
    this._emitEvent('app-initialized', {});
    
  } catch (error) {
    console.error('Error initializing FontViewerApp:', error);
    throw error;
  }
}
```

### Event Handling

The FontViewerApp will set up event listeners for all components:

```javascript
_setupEventListeners() {
  // Refresh button
  const refreshButton = this.querySelector('#refresh-button');
  refreshButton.addEventListener('click', () => {
    this.refreshFonts();
  });
  
  // Font grid events
  this.fontGrid.addEventListener('font-selected', (event) => {
    const fontId = event.detail.id;
    this.selectFont(fontId);
  });
  
  this.fontGrid.addEventListener('font-drag-start', (event) => {
    // Handle font drag start
  });
  
  // Font preview events
  this.fontPreview.addEventListener('font-dropped', (event) => {
    const fontId = event.detail.id;
    this.selectFont(fontId);
  });
  
  this.fontPreview.addEventListener('options-changed', (event) => {
    // Handle preview options changed
  });
  
  // Font collection events
  this.fontCollectionService.addEventListener('collection-created', (event) => {
    this._loadCollections();
  });
  
  this.fontCollectionService.addEventListener('collection-updated', (event) => {
    this._loadCollections();
  });
  
  this.fontCollectionService.addEventListener('collection-deleted', (event) => {
    this._loadCollections();
  });
}
```

### Font Refresh

The FontViewerApp will handle refreshing the font index:

```javascript
async refreshFonts() {
  try {
    // Show loading indicator
    this._showLoading(true);
    
    // Refresh the font index
    const fonts = await this.fontIndexService.refreshFontIndex();
    
    // Update the font grid
    this.fontGrid.setFonts(fonts);
    
    // Hide loading indicator
    this._showLoading(false);
    
    // Emit fonts refreshed event
    this._emitEvent('fonts-refreshed', { fonts });
    
    return fonts;
  } catch (error) {
    console.error('Error refreshing fonts:', error);
    
    // Hide loading indicator
    this._showLoading(false);
    
    throw error;
  }
}
```

### Font Selection

The FontViewerApp will handle selecting fonts for preview:

```javascript
async selectFont(fontId) {
  try {
    // Get the font from the index
    const font = this.fontIndexService.getFontById(fontId);
    
    if (!font) {
      throw new Error(`Font with ID ${fontId} not found`);
    }
    
    // Update the font grid selection
    this.fontGrid.selectFontById(fontId);
    
    // Update the font preview
    this.fontPreview.setFont(font);
    
    // Update the font details
    this._updateFontDetails(font);
    
    // Check if the font is in favorites
    const isInFavorites = await this.fontCollectionService.isInFavorites(fontId);
    
    // Update the favorites button
    this._updateFavoritesButton(fontId, isInFavorites);
    
    // Emit font selected event
    this._emitEvent('font-selected', { font });
  } catch (error) {
    console.error('Error selecting font:', error);
    throw error;
  }
}
```

### Font Comparison

The FontViewerApp will handle comparing multiple fonts:

```javascript
async compareFonts(fontIds) {
  try {
    // Get the fonts from the index
    const fonts = fontIds.map(id => this.fontIndexService.getFontById(id)).filter(Boolean);
    
    if (fonts.length === 0) {
      throw new Error('No valid fonts to compare');
    }
    
    // Update the font preview
    this.fontPreview.setFontsForComparison(fonts);
    
    // Set the preview mode to comparison
    this.fontPreview.setPreviewMode({ mode: 'comparison' });
    
    // Emit fonts compared event
    this._emitEvent('fonts-compared', { fonts });
  } catch (error) {
    console.error('Error comparing fonts:', error);
    throw error;
  }
}
```

### Favorites Management

The FontViewerApp will handle adding and removing fonts from favorites:

```javascript
async addToFavorites(fontId) {
  try {
    // Add the font to favorites
    const favorites = await this.fontCollectionService.addToFavorites(fontId);
    
    // Update the favorites button
    this._updateFavoritesButton(fontId, true);
    
    // Emit favorite added event
    this._emitEvent('favorite-added', { fontId, favorites });
  } catch (error) {
    console.error('Error adding font to favorites:', error);
    throw error;
  }
}

async removeFromFavorites(fontId) {
  try {
    // Remove the font from favorites
    const favorites = await this.fontCollectionService.removeFromFavorites(fontId);
    
    // Update the favorites button
    this._updateFavoritesButton(fontId, false);
    
    // Emit favorite removed event
    this._emitEvent('favorite-removed', { fontId, favorites });
  } catch (error) {
    console.error('Error removing font from favorites:', error);
    throw error;
  }
}
```

### Collections Management

The FontViewerApp will handle loading and displaying collections:

```javascript
async _loadCollections() {
  try {
    // Get all collections
    const collections = await this.fontCollectionService.getCollections();
    
    // Clear the collections container
    this.collectionsContainer.innerHTML = '';
    
    // Create a collection element for each collection
    for (const collection of collections) {
      const collectionElement = document.createElement('div');
      collectionElement.classList.add('collection-item');
      
      const nameElement = document.createElement('div');
      nameElement.classList.add('collection-name');
      nameElement.textContent = collection.name;
      
      const countElement = document.createElement('div');
      countElement.classList.add('collection-count');
      countElement.textContent = `${collection.fonts.length} fonts`;
      
      collectionElement.appendChild(nameElement);
      collectionElement.appendChild(countElement);
      
      // Add click event listener
      collectionElement.addEventListener('click', () => {
        this._selectCollection(collection.id);
      });
      
      // Make the collection a drop zone
      this.dragDropManager.makeDropZone(collectionElement, {
        onDrop: (data) => {
          if (data && data.fontId) {
            this.fontCollectionService.addFontToCollection(collection.id, data.fontId);
          }
        }
      });
      
      this.collectionsContainer.appendChild(collectionElement);
    }
  } catch (error) {
    console.error('Error loading collections:', error);
  }
}
```

### Font Details

The FontViewerApp will handle displaying font details:

```javascript
_updateFontDetails(font) {
  // Clear the font details container
  this.fontDetailsContainer.innerHTML = '';
  
  // Create font details elements
  const nameElement = document.createElement('div');
  nameElement.classList.add('font-detail-item');
  nameElement.innerHTML = `<strong>Name:</strong> ${font.name}`;
  
  const familyElement = document.createElement('div');
  familyElement.classList.add('font-detail-item');
  familyElement.innerHTML = `<strong>Family:</strong> ${font.family}`;
  
  const styleElement = document.createElement('div');
  styleElement.classList.add('font-detail-item');
  styleElement.innerHTML = `<strong>Style:</strong> ${font.style}`;
  
  const weightElement = document.createElement('div');
  weightElement.classList.add('font-detail-item');
  weightElement.innerHTML = `<strong>Weight:</strong> ${font.weight}`;
  
  const formatElement = document.createElement('div');
  formatElement.classList.add('font-detail-item');
  formatElement.innerHTML = `<strong>Format:</strong> ${font.format}`;
  
  // Add elements to the container
  this.fontDetailsContainer.appendChild(nameElement);
  this.fontDetailsContainer.appendChild(familyElement);
  this.fontDetailsContainer.appendChild(styleElement);
  this.fontDetailsContainer.appendChild(weightElement);
  this.fontDetailsContainer.appendChild(formatElement);
  
  // Add favorites button
  const favoritesButton = document.createElement('button');
  favoritesButton.classList.add('favorites-button');
  favoritesButton.id = 'favorites-button';
  favoritesButton.innerHTML = '<span class="icon">★</span> Add to Favorites';
  
  favoritesButton.addEventListener('click', () => {
    this._toggleFavorite(font.id);
  });
  
  this.fontDetailsContainer.appendChild(favoritesButton);
}
```

### Theme Support

The FontViewerApp will support light and dark themes:

```javascript
_initTheme() {
  // Check for system preference
  const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
  
  // Check for stored preference
  const storedTheme = localStorage.getItem('font-viewer-theme');
  
  // Set initial theme
  if (storedTheme === 'dark' || (storedTheme === null && prefersDark)) {
    this.classList.add('dark-theme');
  }
  
  // Add theme toggle button
  const themeButton = document.createElement('button');
  themeButton.classList.add('theme-button', 'action-button');
  themeButton.innerHTML = '<span class="icon">🌙</span> Theme';
  
  themeButton.addEventListener('click', () => {
    this._toggleTheme();
  });
  
  this.querySelector('.app-actions').appendChild(themeButton);
}

_toggleTheme() {
  const isDark = this.classList.contains('dark-theme');
  
  if (isDark) {
    this.classList.remove('dark-theme');
    localStorage.setItem('font-viewer-theme', 'light');
  } else {
    this.classList.add('dark-theme');
    localStorage.setItem('font-viewer-theme', 'dark');
  }
}
```

### Loading Indicator

The FontViewerApp will provide a loading indicator for asynchronous operations:

```javascript
_showLoading(show) {
  // Get or create loading element
  let loadingElement = this.querySelector('.loading-indicator');
  
  if (!loadingElement && show) {
    loadingElement = document.createElement('div');
    loadingElement.classList.add('loading-indicator');
    loadingElement.innerHTML = '<div class="loading-spinner"></div><div class="loading-text">Loading...</div>';
    this.appendChild(loadingElement);
  }
  
  if (loadingElement) {
    if (show) {
      loadingElement.style.display = 'flex';
    } else {
      loadingElement.style.display = 'none';
    }
  }
}
```

### Event System

The FontViewerApp will implement an event system for application events:

```javascript
_eventListeners = {
  'fonts-refreshed': [],
  'font-selected': [],
  'fonts-compared': [],
  'favorite-added': [],
  'favorite-removed': [],
  'app-initialized': []
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

### Basic Usage

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Font Viewer</title>
  <link rel="stylesheet" href="styles.css">
  <script type="module" src="app.js"></script>
</head>
<body>
  <font-viewer-app></font-viewer-app>
  
  <script>
    document.addEventListener('DOMContentLoaded', () => {
      const app = document.querySelector('font-viewer-app');
      
      app.addEventListener('app-initialized', () => {
        console.log('Font Viewer initialized');
      });
      
      app.initialize().catch(error => {
        console.error('Error initializing Font Viewer:', error);
      });
    });
  </script>
</body>
</html>
```

### Listening for Events

```javascript
const app = document.querySelector('font-viewer-app');

// Listen for font selection events
app.addEventListener('font-selected', (data) => {
  console.log('Font selected:', data.font.name);
});

// Listen for favorites events
app.addEventListener('favorite-added', (data) => {
  console.log('Font added to favorites:', data.fontId);
});
```

### Programmatic Control

```javascript
const app = document.querySelector('font-viewer-app');

// Refresh fonts
app.refreshFonts().then(fonts => {
  console.log(`Found ${fonts.length} fonts`);
});

// Select a font
app.selectFont('helvetica');

// Compare fonts
app.compareFonts(['helvetica', 'arial', 'times-new-roman']);

// Add to favorites
app.addToFavorites('helvetica');
```

## Dependencies

The FontViewerApp will depend on the following components and services:

1. **FontIndexService**: For scanning and indexing fonts
2. **FontCollectionService**: For managing font collections
3. **DragDropManager**: For handling drag and drop operations
4. **FontGridComponent**: For displaying the font list/grid
5. **FontPreviewComponent**: For previewing selected fonts

## Integration with Cyberdeck

The FontViewerApp will integrate with the Cyberdeck ecosystem:

1. **Theme Support**: Support for Cyberdeck themes
2. **Navigation**: Integration with Cyberdeck navigation
3. **Window Management**: Support for Cyberdeck window management
4. **API**: Expose an API for other Cyberdeck applications

## Testing Strategy

The FontViewerApp will be tested using the following approach:

1. **Unit Tests**: Test individual methods with mock dependencies
2. **Integration Tests**: Test with actual components and services
3. **End-to-End Tests**: Test complete user flows
4. **Accessibility Tests**: Ensure accessibility compliance

## Accessibility Considerations

The FontViewerApp will implement the following accessibility features:

1. **Keyboard Navigation**: Support for navigating the application using keyboard
2. **ARIA Attributes**: Proper ARIA attributes for screen readers
3. **Focus Management**: Proper focus handling for interactive elements
4. **Color Contrast**: Sufficient color contrast for text and UI elements

## Performance Considerations

The FontViewerApp will implement the following performance optimizations:

1. **Lazy Loading**: Load components and data on demand
2. **Efficient Updates**: Only update changed elements
3. **Resource Management**: Clean up resources when components are removed
4. **Caching**: Cache data for faster access

## Future Enhancements

1. **Font Categories**: Group fonts by category
2. **Font Search**: Advanced search capabilities
3. **Font Export**: Export font information and previews
4. **Font Installation**: Support for installing fonts
5. **Font Analytics**: Track font usage and popularity
