# Font-Viewer Architecture

## Overview

The Font-Viewer application follows a component-based architecture with clear separation of concerns. It is designed to be modular, maintainable, and extensible, with a focus on reusing simple components from jellicule where possible and composing them to create more complex functionality.

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                        FontViewerApp                            │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                      Component Orchestration                    │
├─────────────┬─────────────┬─────────────┬─────────────┬─────────┤
│FontGrid     │FontPreview  │Collections  │FontDetails  │Settings │
│Component    │Component    │Component    │Component    │Component│
└─────────────┴─────────────┴─────────────┴─────────────┴─────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                        Service Layer                            │
├─────────────────────┬─────────────────────┬─────────────────────┤
│  FontIndexService   │FontCollectionService│   DragDropManager   │
└─────────────────────┴─────────────────────┴─────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                        Storage Layer                            │
├─────────────────────┬─────────────────────┬─────────────────────┤
│     IndexedDB       │    LocalStorage     │     File System     │
└─────────────────────┴─────────────────────┴─────────────────────┘
```

## Component Hierarchy

```
FontViewerApp
├── AppHeader
│   ├── AppTitle
│   └── AppActions
│       ├── RefreshButton
│       └── ThemeButton
├── AppContent
│   ├── AppSidebar
│   │   └── FontGridComponent
│   ├── AppMain
│   │   └── FontPreviewComponent
│   └── AppSidebarRight
│       ├── CollectionsPanel
│       │   └── CollectionsList
│       └── FontDetailsPanel
│           └── FontDetails
└── AppFooter
```

## Component Responsibilities

### UI Components

1. **FontViewerApp**: Main application component that orchestrates all other components
   - Initializes and coordinates all application components
   - Manages the application state and data flow
   - Handles user interactions and events
   - Provides the overall layout and structure

2. **FontGridComponent**: Displays a grid or list of available fonts
   - Displays fonts in a grid or list view
   - Supports searching and filtering
   - Enables font selection and drag-and-drop
   - Provides sorting and pagination

3. **FontPreviewComponent**: Displays a preview of selected fonts
   - Shows how fonts look with different text and styles
   - Provides controls for customizing the preview
   - Supports comparing multiple fonts
   - Accepts font drops for quick preview

4. **CollectionsComponent**: Displays and manages font collections
   - Shows a list of collections
   - Allows creating, editing, and deleting collections
   - Supports adding and removing fonts from collections
   - Provides a favorites collection

5. **FontDetailsComponent**: Displays detailed information about a font
   - Shows font metadata (name, family, style, etc.)
   - Provides information about character sets
   - Displays font metrics
   - Shows font license information

### Service Components

1. **FontIndexService**: Scans and indexes available fonts
   - Scans the font directory for font files
   - Extracts metadata from font files
   - Creates and maintains a font index
   - Provides methods for searching and filtering fonts

2. **FontCollectionService**: Manages collections of fonts
   - Creates and manages font collections
   - Stores collections in IndexedDB
   - Provides a favorites collection
   - Emits events when collections change

3. **DragDropManager**: Handles drag and drop operations
   - Makes elements draggable
   - Creates drop zones
   - Manages the transfer of data between elements
   - Provides visual feedback during drag operations

## Data Flow

1. **Font Discovery**:
   - FontIndexService scans the font directory
   - Extracts metadata from font files
   - Creates a font index
   - FontGridComponent displays the fonts

2. **Font Selection**:
   - User selects a font in the FontGridComponent
   - FontViewerApp receives the selection event
   - FontPreviewComponent displays the selected font
   - FontDetailsComponent shows font details

3. **Font Customization**:
   - User adjusts settings in the FontPreviewComponent
   - FontPreviewComponent updates the preview
   - Settings are stored in LocalStorage

4. **Collection Management**:
   - User creates a collection in the CollectionsComponent
   - FontCollectionService stores the collection
   - User drags a font to a collection
   - FontCollectionService adds the font to the collection

5. **Favorites**:
   - User adds a font to favorites
   - FontCollectionService adds the font to the favorites collection
   - CollectionsComponent updates to show the change

## Storage Strategy

1. **Font Index**:
   - In-memory cache for fast access
   - LocalStorage for persistence across sessions
   - Refreshed on demand

2. **Font Metadata**:
   - Extracted from font files
   - Cached in IndexedDB for efficient querying
   - Includes name, family, style, weight, format, etc.

3. **Collections**:
   - Stored in IndexedDB
   - Includes name, description, and list of font IDs
   - Special "favorites" collection

4. **User Preferences**:
   - Stored in LocalStorage
   - Includes theme preference, view mode, sort order, etc.

## Event System

The application uses a custom event system for communication between components:

1. **Component Events**:
   - Each component emits events when its state changes
   - Components listen for events from other components
   - Events include data relevant to the event

2. **Service Events**:
   - Services emit events when their state changes
   - Components listen for events from services
   - Events include data relevant to the event

3. **Global Events**:
   - The FontViewerApp emits global events
   - Components and services can listen for global events
   - Events include application-wide state changes

## Composition Strategy

The Font-Viewer application follows a composition-based approach:

1. **Reuse from jellicule**:
   - Reuse basic UI components from jellicule where possible
   - Examples: buttons, inputs, dropdowns, etc.

2. **Compose from simple components**:
   - Build complex components by composing simple ones
   - Use functional composition for business logic
   - Keep components focused on a single responsibility

3. **Custom components when needed**:
   - Create custom components only when necessary
   - Focus on specific functionality not available in jellicule
   - Examples: FontGridComponent, FontPreviewComponent

## Integration with Cyberdeck

The Font-Viewer application integrates with the Cyberdeck ecosystem:

1. **Theme Support**:
   - Supports Cyberdeck themes
   - Adapts to light and dark modes
   - Uses Cyberdeck design language

2. **Window Management**:
   - Integrates with Cyberdeck window manager
   - Supports resizing and repositioning
   - Remembers window state

3. **API**:
   - Exposes an API for other Cyberdeck applications
   - Allows querying available fonts
   - Supports font selection events

## Extensibility

The Font-Viewer application is designed to be extensible:

1. **Plugin System**:
   - Support for plugins to add functionality
   - Examples: font converters, font analyzers, etc.

2. **API**:
   - Well-defined API for integration with other applications
   - Documented endpoints and events

3. **Custom Themes**:
   - Support for custom themes
   - CSS variables for easy customization

## Performance Considerations

1. **Lazy Loading**:
   - Load components and data on demand
   - Defer non-critical operations

2. **Virtualization**:
   - Only render visible items in long lists
   - Reduce DOM size and improve performance

3. **Caching**:
   - Cache font metadata and previews
   - Reduce disk access and processing

4. **Efficient Updates**:
   - Only update changed elements
   - Use efficient DOM operations

## Accessibility Considerations

1. **Keyboard Navigation**:
   - Support for navigating the application using keyboard
   - Logical tab order

2. **Screen Readers**:
   - Proper ARIA attributes
   - Meaningful text alternatives

3. **Color Contrast**:
   - Sufficient contrast for text and UI elements
   - Support for high contrast mode

4. **Focus Management**:
   - Visible focus indicators
   - Proper focus handling

## Conclusion

The Font-Viewer application follows a component-based architecture with clear separation of concerns. By focusing on composition and reuse, it achieves a modular, maintainable, and extensible design while leveraging existing components from jellicule where possible.
