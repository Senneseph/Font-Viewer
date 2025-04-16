# FontIndexService Specification

## Overview

The FontIndexService is responsible for scanning, indexing, and managing fonts available in the system. It provides methods for refreshing the font index, retrieving font metadata, and searching/filtering fonts.

## Responsibilities

1. Scan the font directory and extract font metadata
2. Create and maintain an index of available fonts
3. Provide methods for retrieving and searching fonts
4. Support refreshing the font index on demand
5. Store font metadata for efficient retrieval

## Interface

```typescript
interface FontMetadata {
  id: string;           // Unique identifier (generated from name and path)
  name: string;         // Font name
  family: string;       // Font family
  style: string;        // Font style (normal, italic, etc.)
  weight: string;       // Font weight (normal, bold, etc.)
  format: string;       // Font format (ttf, otf, woff, woff2)
  path: string;         // Path to font file
  lastModified: number; // Last modified timestamp
  previewUrl?: string;  // Data URL for font preview (optional)
}

interface FontFilterCriteria {
  family?: string;      // Filter by font family
  style?: string;       // Filter by font style
  weight?: string;      // Filter by font weight
  format?: string;      // Filter by font format
}

interface FontSortOptions {
  field: 'name' | 'family' | 'lastModified';
  direction: 'asc' | 'desc';
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
  filterFonts(criteria: FontFilterCriteria): FontMetadata[];
  
  /**
   * Sort fonts by field
   * @param fonts The fonts to sort
   * @param options The sort options
   * @returns Sorted array of fonts
   */
  sortFonts(fonts: FontMetadata[], options: FontSortOptions): FontMetadata[];
  
  /**
   * Generate a preview for a font
   * @param font The font metadata
   * @returns Promise resolving to the preview URL
   */
  generateFontPreview(font: FontMetadata): Promise<string>;
  
  /**
   * Add an event listener for font index events
   * @param event The event name
   * @param callback The callback function
   */
  addEventListener(event: 'fonts-refreshed' | 'font-added' | 'font-removed', callback: Function): void;
  
  /**
   * Remove an event listener
   * @param event The event name
   * @param callback The callback function
   */
  removeEventListener(event: 'fonts-refreshed' | 'font-added' | 'font-removed', callback: Function): void;
}
```

## Implementation Details

### Font Scanning

The FontIndexService will scan the font directory using the following approach:

1. Identify the font directory (jellicule/app/assets/fonts by default)
2. Recursively scan for font files with supported extensions (.ttf, .otf, .woff, .woff2)
3. Extract metadata from each font file
4. Generate a unique ID for each font based on its name and path
5. Create a FontMetadata object for each font

### Metadata Extraction

The service will extract the following metadata from font files:

1. **Name**: The font name from the font file
2. **Family**: The font family from the font file
3. **Style**: The font style from the font file (normal, italic, etc.)
4. **Weight**: The font weight from the font file (normal, bold, etc.)
5. **Format**: The font format based on the file extension
6. **Path**: The path to the font file
7. **Last Modified**: The last modified timestamp of the file

### Storage Strategy

The FontIndexService will use the following storage strategy:

1. **In-Memory Cache**: Keep the current font index in memory for fast access
2. **LocalStorage**: Cache the font index in localStorage for persistence across sessions
3. **IndexedDB**: Store detailed font metadata in IndexedDB for efficient querying

### Event System

The FontIndexService will emit the following events:

1. **fonts-refreshed**: Emitted when the font index is refreshed
2. **font-added**: Emitted when a new font is added to the index
3. **font-removed**: Emitted when a font is removed from the index

### Error Handling

The FontIndexService will handle the following error conditions:

1. **Directory Not Found**: If the font directory is not found, return an empty array
2. **Invalid Font File**: If a font file cannot be parsed, skip it and continue
3. **Storage Errors**: If localStorage or IndexedDB is not available, fall back to in-memory storage

## Usage Examples

### Refreshing the Font Index

```typescript
const fontIndexService = new FontIndexService();

// Refresh the font index
fontIndexService.refreshFontIndex()
  .then(fonts => {
    console.log(`Found ${fonts.length} fonts`);
  })
  .catch(error => {
    console.error('Error refreshing font index:', error);
  });
```

### Searching and Filtering Fonts

```typescript
// Search for fonts by name
const searchResults = fontIndexService.searchFonts('Helvetica');

// Filter fonts by criteria
const filterResults = fontIndexService.filterFonts({
  family: 'Helvetica',
  style: 'italic'
});

// Sort fonts by name
const sortedFonts = fontIndexService.sortFonts(fonts, {
  field: 'name',
  direction: 'asc'
});
```

### Listening for Events

```typescript
// Listen for font index refresh events
fontIndexService.addEventListener('fonts-refreshed', (fonts) => {
  console.log(`Font index refreshed with ${fonts.length} fonts`);
});

// Listen for font added events
fontIndexService.addEventListener('font-added', (font) => {
  console.log(`New font added: ${font.name}`);
});
```

## Dependencies

The FontIndexService will depend on the following:

1. **FontDetect**: A library for detecting and extracting metadata from font files
2. **StorageManager**: A utility for managing localStorage and IndexedDB
3. **EventEmitter**: A utility for implementing the event system

## Testing Strategy

The FontIndexService will be tested using the following approach:

1. **Unit Tests**: Test individual methods with mock data
2. **Integration Tests**: Test the service with a test font directory
3. **Performance Tests**: Test the service with a large number of fonts
4. **Error Handling Tests**: Test the service with various error conditions

## Future Enhancements

1. **Font Preview Generation**: Generate previews for fonts on demand
2. **Font Categorization**: Automatically categorize fonts by style and usage
3. **Font Similarity**: Identify similar fonts based on visual characteristics
4. **Font Usage Tracking**: Track which fonts are used most frequently
