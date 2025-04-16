# FontPreviewComponent Specification

## Overview

The FontPreviewComponent is responsible for displaying a preview of selected fonts with customization options. It allows users to see how fonts look with different text, sizes, weights, and styles, and supports drag-and-drop for font selection.

## Responsibilities

1. Display a preview of selected fonts
2. Provide controls for customizing the preview text
3. Allow adjusting font size, weight, style, and color
4. Support comparing multiple fonts side-by-side
5. Accept font drops for quick preview

## Interface

```typescript
interface PreviewOptions {
  text: string;
  size: number;
  weight: string;
  style: string;
  color: string;
  backgroundColor: string;
  alignment: 'left' | 'center' | 'right';
  lineHeight: number;
  letterSpacing: number;
  textTransform: 'none' | 'uppercase' | 'lowercase' | 'capitalize';
}

interface FontPreviewMode {
  mode: 'single' | 'comparison' | 'paragraph';
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
  setOptions(options: Partial<PreviewOptions>): void;
  
  /**
   * Get current preview options
   * @returns The current preview options
   */
  getOptions(): PreviewOptions;
  
  /**
   * Set the preview mode
   * @param mode The preview mode
   */
  setPreviewMode(mode: FontPreviewMode): void;
  
  /**
   * Get the current preview mode
   * @returns The current preview mode
   */
  getPreviewMode(): FontPreviewMode;
  
  /**
   * Clear the current preview
   */
  clear(): void;
  
  /**
   * Add an event listener for font preview events
   * @param event The event name
   * @param callback The callback function
   */
  addEventListener(event: 'font-dropped' | 'options-changed' | 'mode-changed', callback: Function): void;
  
  /**
   * Remove an event listener
   * @param event The event name
   * @param callback The callback function
   */
  removeEventListener(event: 'font-dropped' | 'options-changed' | 'mode-changed', callback: Function): void;
}
```

## HTML Structure

```html
<font-preview>
  <div class="preview-toolbar">
    <div class="text-controls">
      <input type="text" class="text-input" value="The quick brown fox jumps over the lazy dog">
    </div>
    
    <div class="size-controls">
      <label for="size-slider">Size:</label>
      <input type="range" id="size-slider" min="8" max="72" value="24">
      <span class="size-value">24px</span>
    </div>
    
    <div class="style-controls">
      <button class="style-button" data-style="normal">Normal</button>
      <button class="style-button" data-style="italic">Italic</button>
    </div>
    
    <div class="weight-controls">
      <button class="weight-button" data-weight="normal">Normal</button>
      <button class="weight-button" data-weight="bold">Bold</button>
    </div>
    
    <div class="color-controls">
      <input type="color" class="color-picker" value="#000000">
      <input type="color" class="background-color-picker" value="#ffffff">
    </div>
    
    <div class="mode-controls">
      <button class="mode-button" data-mode="single">Single</button>
      <button class="mode-button" data-mode="comparison">Compare</button>
      <button class="mode-button" data-mode="paragraph">Paragraph</button>
    </div>
  </div>
  
  <div class="preview-container">
    <div class="preview-area" tabindex="0">
      <!-- Preview content will be dynamically generated -->
    </div>
  </div>
  
  <div class="preview-info">
    <div class="font-info">
      <h3 class="font-name">No font selected</h3>
      <p class="font-family"></p>
      <p class="font-details"></p>
    </div>
  </div>
</font-preview>
```

## CSS Structure

```css
:host {
  display: block;
  width: 100%;
  height: 100%;
  overflow: hidden;
}

.preview-toolbar {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  padding: 8px;
  border-bottom: 1px solid var(--border-color);
}

.preview-container {
  height: calc(100% - 150px);
  overflow: auto;
  padding: 16px;
}

.preview-area {
  min-height: 200px;
  padding: 16px;
  border: 1px dashed var(--border-color);
  border-radius: 4px;
  transition: all 0.2s ease;
}

.preview-area:focus {
  outline: none;
  border-color: var(--primary-color);
}

.preview-area.drag-over {
  border-color: var(--primary-color);
  background-color: var(--primary-color-light);
}

.preview-info {
  padding: 8px;
  border-top: 1px solid var(--border-color);
}

.font-name {
  margin: 0;
  font-size: 16px;
}

.font-family, .font-details {
  margin: 4px 0;
  font-size: 14px;
  color: var(--text-secondary-color);
}

/* Preview modes */
.preview-area.single-mode {
  display: flex;
  align-items: center;
  justify-content: center;
}

.preview-area.comparison-mode {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 16px;
}

.preview-area.paragraph-mode {
  display: block;
}

.preview-item {
  margin-bottom: 16px;
}

.preview-item-header {
  font-size: 12px;
  color: var(--text-secondary-color);
  margin-bottom: 4px;
}

.preview-item-content {
  padding: 8px;
  border: 1px solid var(--border-color);
  border-radius: 4px;
}
```

## Implementation Details

### Font Preview Generation

The FontPreviewComponent will generate font previews using the following approach:

1. Load the font using CSS @font-face or the FontFace API
2. Create a preview element with the specified text
3. Apply the font and style options to the preview element
4. Update the preview when options change

### Preview Modes

The FontPreviewComponent will support three preview modes:

1. **Single Mode**: Display a single font preview
2. **Comparison Mode**: Display multiple fonts side-by-side for comparison
3. **Paragraph Mode**: Display a longer text sample to see how the font looks in paragraphs

### Customization Options

The FontPreviewComponent will provide the following customization options:

1. **Text**: Custom text for the preview
2. **Size**: Font size adjustment
3. **Weight**: Font weight selection (normal, bold, etc.)
4. **Style**: Font style selection (normal, italic)
5. **Color**: Text color selection
6. **Background Color**: Background color selection
7. **Alignment**: Text alignment (left, center, right)
8. **Line Height**: Line height adjustment
9. **Letter Spacing**: Letter spacing adjustment
10. **Text Transform**: Text transformation (none, uppercase, lowercase, capitalize)

### Drag and Drop

The FontPreviewComponent will support drag and drop functionality:

1. Make the preview area a drop zone
2. Accept font drops from the FontGridComponent
3. Update the preview when a font is dropped
4. Provide visual feedback during drag operations

### Event System

The FontPreviewComponent will emit the following events:

1. **font-dropped**: Emitted when a font is dropped on the preview area
2. **options-changed**: Emitted when preview options change
3. **mode-changed**: Emitted when the preview mode changes

## Usage Examples

### Basic Usage

```html
<font-preview id="fontPreview"></font-preview>

<script>
  // Get the font preview element
  const fontPreview = document.getElementById('fontPreview');
  
  // Set the font to preview
  fontPreview.setFont(font);
  
  // Set preview options
  fontPreview.setOptions({
    text: 'The quick brown fox jumps over the lazy dog',
    size: 24,
    weight: 'normal',
    style: 'normal',
    color: '#000000',
    backgroundColor: '#ffffff'
  });
  
  // Listen for option changes
  fontPreview.addEventListener('options-changed', (event) => {
    const options = event.detail;
    console.log('Preview options changed:', options);
  });
</script>
```

### Font Comparison

```javascript
// Set multiple fonts for comparison
fontPreview.setFontsForComparison([font1, font2, font3]);

// Set comparison mode
fontPreview.setPreviewMode({ mode: 'comparison' });
```

### Drag and Drop

```javascript
// Listen for font drop events
fontPreview.addEventListener('font-dropped', (event) => {
  const font = event.detail;
  console.log('Font dropped:', font.name);
});
```

## Dependencies

The FontPreviewComponent will depend on the following:

1. **FontIndexService**: For retrieving font metadata
2. **DragDropManager**: For handling drag and drop operations
3. **EventEmitter**: For implementing the event system

## Testing Strategy

The FontPreviewComponent will be tested using the following approach:

1. **Unit Tests**: Test individual methods with mock data
2. **Integration Tests**: Test the component with the FontIndexService
3. **Visual Tests**: Test the component's appearance with different fonts and options
4. **Interaction Tests**: Test customization controls and drag and drop

## Accessibility Considerations

The FontPreviewComponent will implement the following accessibility features:

1. **Keyboard Navigation**: Support for navigating and adjusting options using keyboard
2. **ARIA Attributes**: Proper ARIA attributes for screen readers
3. **Focus Management**: Proper focus handling for interactive elements
4. **Color Contrast**: Sufficient color contrast for text and UI elements

## Performance Considerations

The FontPreviewComponent will implement the following performance optimizations:

1. **Font Loading**: Efficient font loading and caching
2. **Debouncing**: Debounce option changes to avoid excessive updates
3. **Lazy Rendering**: Only render visible content in comparison mode
4. **Memory Management**: Clean up resources when fonts are changed or removed

## Future Enhancements

1. **Animation**: Add animation options for text preview
2. **Export**: Allow exporting the preview as an image
3. **Templates**: Predefined templates for different preview scenarios
4. **Advanced Typography**: Support for advanced typography features like ligatures and alternates
