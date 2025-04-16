# Font-Viewer Specification

## Overview

Font-Viewer is a Progressive Web Application (PWA) within the Cyberdeck ecosystem that allows users to browse, preview, and interact with available fonts through a drag-and-drop interface. It serves as a visual catalog of fonts available in the system and provides tools for comparing and selecting fonts for use in other applications.

## Core Objectives

1. Provide an intuitive interface for browsing and previewing fonts
2. Enable drag-and-drop interaction for font selection and comparison
3. Support dynamic refresh of the font index
4. Expose an API for other Cyberdeck applications to query available fonts
5. Follow WebAX standards and Cyberdeck ecosystem design patterns

## Project Structure

```text
Font-Viewer/
├── app/                  # Application code
│   ├── assets/           # Static assets
│   │   ├── icons/        # App icons
│   │   └── styles/       # CSS files
│   ├── src/              # Source code
│   │   ├── components/   # Web components
│   │   ├── services/     # Service modules
│   │   └── utils/        # Utility functions
│   └── index.html        # Entry point
├── cicd/                 # CI/CD configuration
├── develop/              # Development files
│   ├── specs/            # Specifications
│   └── tests/            # Test files
├── dist/                 # Distribution files
└── docs/                 # Documentation
```

## Functional Requirements

### Font Discovery

1. **Font Scanning**
   - Automatically scan and index fonts from the jellicule/app/assets/fonts directory
   - Support for TTF, OTF, WOFF, and WOFF2 font formats
   - Extract and display metadata from font files when available

2. **Font Indexing**
   - Create and maintain an index of available fonts
   - Store font metadata for efficient retrieval
   - Support for refreshing the index on demand

### User Interface

1. **Font Grid/List**
   - Display available fonts in a grid or list view
   - Show font name and preview using the font itself
   - Support for sorting and filtering

2. **Font Preview**
   - Display sample text using the selected font
   - Allow customization of sample text
   - Provide controls for font size, weight, and style
   - Support for color and background customization

3. **Font Details**
   - Display detailed metadata for selected fonts
   - Show font family, style, weight, and format
   - Provide information about character sets and features

### Interaction

1. **Drag and Drop**
   - Drag fonts to preview area to change sample text font
   - Drag to create collections or favorites
   - Drag to compare multiple fonts side-by-side

2. **Refresh Capability**
   - Provide a refresh button to update the font index
   - Show visual indication of refresh status
   - Notify user of newly added fonts

3. **Search and Filter**
   - Search fonts by name, family, or style
   - Filter fonts by format, style, or weight
   - Sort fonts by name, family, or date added

### Integration

1. **API**
   - Expose an API for other applications to query available fonts
   - Provide endpoints for font selection and retrieval
   - Support for event-based communication

2. **Cyberdeck Integration**
   - Follow Cyberdeck UI guidelines and patterns
   - Support for Cyberdeck themes
   - Integration with Cyberdeck navigation and window management

## Technical Specifications

### Components

#### FontIndexService

```typescript
interface FontMetadata {
  id: string;           // Unique identifier
  name: string;         // Font name
  family: string;       // Font family
  style: string;        // Font style (normal, italic, etc.)
  weight: string;       // Font weight (normal, bold, etc.)
  format: string;       // Font format (ttf, otf, woff, woff2)
  path: string;         // Path to font file
  lastModified: number; // Last modified timestamp
}

class FontIndexService {
  /**
   * Scan the font directory and update the index
   * @returns Promise resolving to the updated font list
   */
  refreshFontIndex(): Promise<FontMetadata[]>;

  /**
   * Get the current font index
   * @returns Array of font metadata
   */
  getFontIndex(): FontMetadata[];

  /**
   * Get a specific font by ID
   * @param id The font ID to find
   * @returns The font metadata or undefined if not found
   */
  getFontById(id: string): FontMetadata | undefined;

  /**
   * Search fonts by query string
   * @param query The search query
   * @returns Array of matching fonts
   */
  searchFonts(query: string): FontMetadata[];

  /**
   * Filter fonts by criteria
   * @param criteria The filter criteria
   * @returns Array of matching fonts
   */
  filterFonts(criteria: object): FontMetadata[];
}
```

#### FontGridComponent

```typescript
interface FontGridOptions {
  viewMode: 'grid' | 'list';
  sortBy: 'name' | 'family' | 'added';
  filterBy?: object;
  searchQuery?: string;
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
   * Handle font selection
   * @param callback Function to call when a font is selected
   */
  onFontSelect(callback: (font: FontMetadata) => void): void;

  /**
   * Handle font drag start
   * @param callback Function to call when a font drag starts
   */
  onFontDragStart(callback: (font: FontMetadata, event: DragEvent) => void): void;
}
```

#### FontPreviewComponent

```typescript
interface PreviewOptions {
  text: string;
  size: number;
  weight: string;
  style: string;
  color: string;
  backgroundColor: string;
}

class FontPreviewComponent extends HTMLElement {
  /**
   * Set the font to preview
   * @param font The font metadata
   */
  setFont(font: FontMetadata): void;

  /**
   * Set multiple fonts for comparison
   * @param fonts Array of font metadata
   */
  setFontsForComparison(fonts: FontMetadata[]): void;

  /**
   * Update preview options
   * @param options The preview options
   */
  setOptions(options: PreviewOptions): void;

  /**
   * Handle drop events
   * @param callback Function to call when a font is dropped
   */
  onFontDrop(callback: (font: FontMetadata, event: DragEvent) => void): void;
}
```

#### DragDropManager

```typescript
interface DropZone {
  element: HTMLElement;
  onDrop: (font: FontMetadata, event: DragEvent) => void;
  onDragOver?: (event: DragEvent) => void;
  onDragLeave?: (event: DragEvent) => void;
}

interface Draggable {
  element: HTMLElement;
  data: any;
  onDragStart?: (event: DragEvent) => void;
  onDragEnd?: (event: DragEvent) => void;
}

class DragDropManager {
  /**
   * Register an element as draggable
   * @param draggable The draggable configuration
   */
  registerDraggable(draggable: Draggable): void;

  /**
   * Register a drop zone
   * @param zone The drop zone configuration
   */
  registerDropZone(zone: DropZone): void;

  /**
   * Enable drag and drop functionality
   */
  enable(): void;

  /**
   * Disable drag and drop functionality
   */
  disable(): void;
}
```

#### FontCollectionService

```typescript
interface FontCollection {
  id: string;
  name: string;
  fonts: FontMetadata[];
  createdAt: number;
  updatedAt: number;
}

class FontCollectionService {
  /**
   * Get all collections
   * @returns Array of collections
   */
  getCollections(): Promise<FontCollection[]>;

  /**
   * Get a collection by ID
   * @param id The collection ID
   * @returns The collection or undefined if not found
   */
  getCollectionById(id: string): Promise<FontCollection | undefined>;

  /**
   * Create a new collection
   * @param name The collection name
   * @returns The created collection
   */
  createCollection(name: string): Promise<FontCollection>;

  /**
   * Add a font to a collection
   * @param collectionId The collection ID
   * @param fontId The font ID
   * @returns The updated collection
   */
  addFontToCollection(collectionId: string, fontId: string): Promise<FontCollection>;

  /**
   * Remove a font from a collection
   * @param collectionId The collection ID
   * @param fontId The font ID
   * @returns The updated collection
   */
  removeFontFromCollection(collectionId: string, fontId: string): Promise<FontCollection>;

  /**
   * Delete a collection
   * @param id The collection ID
   * @returns True if successful
   */
  deleteCollection(id: string): Promise<boolean>;
}
```

#### FontViewerApp

```typescript
class FontViewerApp extends HTMLElement {
  /**
   * Initialize the application
   */
  initialize(): Promise<void>;

  /**
   * Refresh the font index
   */
  refreshFonts(): Promise<void>;

  /**
   * Handle font selection
   * @param font The selected font
   */
  selectFont(font: FontMetadata): void;

  /**
   * Add a font to favorites
   * @param font The font to add to favorites
   */
  addToFavorites(font: FontMetadata): void;

  /**
   * Compare multiple fonts
   * @param fonts The fonts to compare
   */
  compareFonts(fonts: FontMetadata[]): void;
}
```

### API Endpoints with Interlacid Compliance

The Font-Viewer will expose a RESTful API for other applications to interact with, implementing the interlacid compliance framework for robust API passthroughs with automatic fallback policies:

```text
GET /api/fonts
Returns a list of all available fonts
Interlacid Policy: Cache fallback - Returns cached font list if scan fails

GET /api/fonts/:id
Returns details for a specific font
Interlacid Policy: Default font fallback - Returns system default if font not found

GET /api/collections
Returns a list of user-created collections
Interlacid Policy: Empty list fallback - Returns empty array if storage fails

POST /api/collections
Creates a new collection
Interlacid Policy: Full rollback - No partial collection creation

GET /api/collections/:id
Returns details for a specific collection
Interlacid Policy: Empty collection fallback - Returns empty collection if not found

PUT /api/collections/:id
Updates a collection
Interlacid Policy: Snapshot rollback - Reverts to previous state if update fails

DELETE /api/collections/:id
Deletes a collection
Interlacid Policy: Soft delete - Marks as deleted but preserves data for recovery
```

### Interlacid Transaction Flow

Each API operation follows this transaction flow:

1. **Snapshot**: Capture the current state before modification
2. **Attempt**: Perform the requested operation
3. **Verify**: Confirm the operation completed successfully
4. **Commit/Rollback**: Either commit the change or roll back to the snapshot
5. **Notify**: Inform the client of the outcome and any fallback actions taken

### Event System

The Font-Viewer will use a custom event system for component communication:

```typescript
// Events emitted by the application
interface FontViewerEvents {
  'font-selected': FontMetadata;
  'fonts-refreshed': FontMetadata[];
  'collection-created': FontCollection;
  'collection-updated': FontCollection;
  'collection-deleted': string; // Collection ID
}
```

### Storage Strategy

The Font-Viewer will use the following storage strategy:

1. **Font Index**: Cached in localStorage for quick startup
2. **Font Metadata**: Stored in IndexedDB for efficient querying
3. **User Preferences**: Stored in localStorage
4. **Collections**: Stored in IndexedDB with sync capability

## User Interface Design

### Main Layout

The Font-Viewer will have a three-panel layout:

1. **Left Panel**: Font list/grid with search and filter controls
2. **Center Panel**: Font preview with customization options
3. **Right Panel**: Font details and collections

### Responsive Design

The layout will adapt to different screen sizes:

1. **Desktop**: Three-panel layout as described above
2. **Tablet**: Two-panel layout with font list and preview/details
3. **Mobile**: Single-panel layout with tabs for switching between views

### Theme Support

The Font-Viewer will support Cyberdeck themes:

1. **Light Theme**: Light background with dark text
2. **Dark Theme**: Dark background with light text
3. **High Contrast**: High contrast colors for accessibility
4. **Custom Themes**: Support for custom Cyberdeck themes

## Implementation Plan

### Phase 1: Core Functionality (Week 1)

1. Set up project structure
2. Implement FontIndexService
3. Create basic UI components
4. Implement font scanning and indexing
5. Create basic font grid and preview

### Phase 2: Drag and Drop (Week 2)

1. Implement DragDropManager
2. Add drag and drop support to font grid
3. Add drop zones to preview area
4. Implement font comparison via drag and drop

### Phase 3: Collections and API (Week 3)

1. Implement FontCollectionService
2. Create collections UI
3. Implement favorites functionality
4. Create API endpoints
5. Add event system

### Phase 4: Polish and Integration (Week 4)

1. Implement responsive design
2. Add theme support
3. Optimize performance
4. Add offline support
5. Integrate with Cyberdeck ecosystem
6. Create documentation

## Testing Strategy

1. **Unit Tests**: Test individual components and services
2. **Integration Tests**: Test component interactions
3. **End-to-End Tests**: Test complete user flows
4. **Performance Tests**: Test loading and rendering performance
5. **Accessibility Tests**: Ensure accessibility compliance

## Deployment Strategy

1. **Development**: Local development environment
2. **Staging**: Deployment to staging environment for testing
3. **Production**: Deployment to production environment
4. **Distribution**: Packaging as a WebAXD for distribution

## Conclusion

The Font-Viewer will provide a powerful and intuitive interface for browsing, previewing, and selecting fonts within the Cyberdeck ecosystem. By following WebAX standards and Cyberdeck design patterns, it will integrate seamlessly with other applications and provide a consistent user experience.
