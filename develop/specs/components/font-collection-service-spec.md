# FontCollectionService Specification

## Overview

The FontCollectionService is responsible for managing collections of fonts in the Font-Viewer application. It provides functionality for creating, retrieving, updating, and deleting collections, as well as adding and removing fonts from collections.

## Responsibilities

1. Create and manage font collections
2. Store and retrieve collections from persistent storage
3. Add and remove fonts from collections
4. Provide a favorites collection for quick access
5. Emit events when collections change

## Interface

```typescript
interface FontCollection {
  id: string;
  name: string;
  description?: string;
  fonts: string[]; // Array of font IDs
  createdAt: number;
  updatedAt: number;
}

class FontCollectionService {
  /**
   * Get all collections
   * @returns Promise resolving to an array of collections
   */
  getCollections(): Promise<FontCollection[]>;
  
  /**
   * Get a collection by ID
   * @param id The collection ID
   * @returns Promise resolving to the collection or undefined if not found
   */
  getCollectionById(id: string): Promise<FontCollection | undefined>;
  
  /**
   * Create a new collection
   * @param name The collection name
   * @param description Optional collection description
   * @returns Promise resolving to the created collection
   */
  createCollection(name: string, description?: string): Promise<FontCollection>;
  
  /**
   * Update a collection
   * @param id The collection ID
   * @param updates The updates to apply
   * @returns Promise resolving to the updated collection
   */
  updateCollection(id: string, updates: Partial<FontCollection>): Promise<FontCollection>;
  
  /**
   * Delete a collection
   * @param id The collection ID
   * @returns Promise resolving to true if successful
   */
  deleteCollection(id: string): Promise<boolean>;
  
  /**
   * Add a font to a collection
   * @param collectionId The collection ID
   * @param fontId The font ID
   * @returns Promise resolving to the updated collection
   */
  addFontToCollection(collectionId: string, fontId: string): Promise<FontCollection>;
  
  /**
   * Remove a font from a collection
   * @param collectionId The collection ID
   * @param fontId The font ID
   * @returns Promise resolving to the updated collection
   */
  removeFontFromCollection(collectionId: string, fontId: string): Promise<FontCollection>;
  
  /**
   * Check if a font is in a collection
   * @param collectionId The collection ID
   * @param fontId The font ID
   * @returns Promise resolving to true if the font is in the collection
   */
  isFontInCollection(collectionId: string, fontId: string): Promise<boolean>;
  
  /**
   * Get the favorites collection
   * @returns Promise resolving to the favorites collection
   */
  getFavorites(): Promise<FontCollection>;
  
  /**
   * Add a font to favorites
   * @param fontId The font ID
   * @returns Promise resolving to the updated favorites collection
   */
  addToFavorites(fontId: string): Promise<FontCollection>;
  
  /**
   * Remove a font from favorites
   * @param fontId The font ID
   * @returns Promise resolving to the updated favorites collection
   */
  removeFromFavorites(fontId: string): Promise<FontCollection>;
  
  /**
   * Check if a font is in favorites
   * @param fontId The font ID
   * @returns Promise resolving to true if the font is in favorites
   */
  isInFavorites(fontId: string): Promise<boolean>;
  
  /**
   * Add an event listener for collection events
   * @param event The event name
   * @param callback The callback function
   */
  addEventListener(event: 'collection-created' | 'collection-updated' | 'collection-deleted' | 'font-added' | 'font-removed', callback: Function): void;
  
  /**
   * Remove an event listener
   * @param event The event name
   * @param callback The callback function
   */
  removeEventListener(event: 'collection-created' | 'collection-updated' | 'collection-deleted' | 'font-added' | 'font-removed', callback: Function): void;
}
```

## Implementation Details

### Storage Strategy

The FontCollectionService will use IndexedDB for persistent storage of collections:

1. Create an IndexedDB database for storing collections
2. Define a collections object store with an index on the collection name
3. Use promises for asynchronous operations
4. Cache collections in memory for faster access

```javascript
async _initDatabase() {
  return new Promise((resolve, reject) => {
    const request = indexedDB.open('font-viewer-collections', 1);
    
    request.onupgradeneeded = (event) => {
      const db = event.target.result;
      
      // Create collections object store
      const collectionsStore = db.createObjectStore('collections', { keyPath: 'id' });
      collectionsStore.createIndex('name', 'name', { unique: false });
      
      // Create initial favorites collection
      const favoritesCollection = {
        id: 'favorites',
        name: 'Favorites',
        description: 'Your favorite fonts',
        fonts: [],
        createdAt: Date.now(),
        updatedAt: Date.now()
      };
      
      collectionsStore.add(favoritesCollection);
    };
    
    request.onsuccess = (event) => {
      this._db = event.target.result;
      resolve();
    };
    
    request.onerror = (event) => {
      reject(new Error('Failed to open collections database'));
    };
  });
}
```

### Collection Management

The FontCollectionService will provide methods for managing collections:

1. **Create**: Create a new collection with a unique ID
2. **Retrieve**: Get collections by ID or get all collections
3. **Update**: Update collection properties
4. **Delete**: Delete a collection by ID

```javascript
async createCollection(name, description) {
  // Generate a unique ID
  const id = `collection-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  
  // Create the collection
  const collection = {
    id,
    name,
    description,
    fonts: [],
    createdAt: Date.now(),
    updatedAt: Date.now()
  };
  
  // Store the collection
  await this._storeCollection(collection);
  
  // Emit the collection-created event
  this._emitEvent('collection-created', { collection });
  
  return collection;
}

async getCollectionById(id) {
  return new Promise((resolve, reject) => {
    const transaction = this._db.transaction(['collections'], 'readonly');
    const store = transaction.objectStore('collections');
    const request = store.get(id);
    
    request.onsuccess = (event) => {
      resolve(event.target.result);
    };
    
    request.onerror = (event) => {
      reject(new Error(`Failed to get collection with ID ${id}`));
    };
  });
}

async getCollections() {
  return new Promise((resolve, reject) => {
    const transaction = this._db.transaction(['collections'], 'readonly');
    const store = transaction.objectStore('collections');
    const request = store.getAll();
    
    request.onsuccess = (event) => {
      resolve(event.target.result);
    };
    
    request.onerror = (event) => {
      reject(new Error('Failed to get collections'));
    };
  });
}

async updateCollection(id, updates) {
  // Get the existing collection
  const collection = await this.getCollectionById(id);
  
  if (!collection) {
    throw new Error(`Collection with ID ${id} not found`);
  }
  
  // Apply updates
  const updatedCollection = {
    ...collection,
    ...updates,
    updatedAt: Date.now()
  };
  
  // Store the updated collection
  await this._storeCollection(updatedCollection);
  
  // Emit the collection-updated event
  this._emitEvent('collection-updated', { collection: updatedCollection });
  
  return updatedCollection;
}

async deleteCollection(id) {
  // Don't allow deleting the favorites collection
  if (id === 'favorites') {
    throw new Error('Cannot delete the favorites collection');
  }
  
  return new Promise((resolve, reject) => {
    const transaction = this._db.transaction(['collections'], 'readwrite');
    const store = transaction.objectStore('collections');
    const request = store.delete(id);
    
    request.onsuccess = (event) => {
      // Emit the collection-deleted event
      this._emitEvent('collection-deleted', { id });
      
      resolve(true);
    };
    
    request.onerror = (event) => {
      reject(new Error(`Failed to delete collection with ID ${id}`));
    };
  });
}
```

### Font Management

The FontCollectionService will provide methods for managing fonts within collections:

1. **Add**: Add a font to a collection
2. **Remove**: Remove a font from a collection
3. **Check**: Check if a font is in a collection

```javascript
async addFontToCollection(collectionId, fontId) {
  // Get the collection
  const collection = await this.getCollectionById(collectionId);
  
  if (!collection) {
    throw new Error(`Collection with ID ${collectionId} not found`);
  }
  
  // Check if the font is already in the collection
  if (collection.fonts.includes(fontId)) {
    return collection;
  }
  
  // Add the font to the collection
  const updatedCollection = {
    ...collection,
    fonts: [...collection.fonts, fontId],
    updatedAt: Date.now()
  };
  
  // Store the updated collection
  await this._storeCollection(updatedCollection);
  
  // Emit the font-added event
  this._emitEvent('font-added', { collectionId, fontId });
  
  return updatedCollection;
}

async removeFontFromCollection(collectionId, fontId) {
  // Get the collection
  const collection = await this.getCollectionById(collectionId);
  
  if (!collection) {
    throw new Error(`Collection with ID ${collectionId} not found`);
  }
  
  // Check if the font is in the collection
  if (!collection.fonts.includes(fontId)) {
    return collection;
  }
  
  // Remove the font from the collection
  const updatedCollection = {
    ...collection,
    fonts: collection.fonts.filter(id => id !== fontId),
    updatedAt: Date.now()
  };
  
  // Store the updated collection
  await this._storeCollection(updatedCollection);
  
  // Emit the font-removed event
  this._emitEvent('font-removed', { collectionId, fontId });
  
  return updatedCollection;
}

async isFontInCollection(collectionId, fontId) {
  // Get the collection
  const collection = await this.getCollectionById(collectionId);
  
  if (!collection) {
    return false;
  }
  
  // Check if the font is in the collection
  return collection.fonts.includes(fontId);
}
```

### Favorites Management

The FontCollectionService will provide special methods for managing the favorites collection:

1. **Get**: Get the favorites collection
2. **Add**: Add a font to favorites
3. **Remove**: Remove a font from favorites
4. **Check**: Check if a font is in favorites

```javascript
async getFavorites() {
  return this.getCollectionById('favorites');
}

async addToFavorites(fontId) {
  return this.addFontToCollection('favorites', fontId);
}

async removeFromFavorites(fontId) {
  return this.removeFontFromCollection('favorites', fontId);
}

async isInFavorites(fontId) {
  return this.isFontInCollection('favorites', fontId);
}
```

### Event System

The FontCollectionService will implement an event system for collection events:

1. **collection-created**: Emitted when a collection is created
2. **collection-updated**: Emitted when a collection is updated
3. **collection-deleted**: Emitted when a collection is deleted
4. **font-added**: Emitted when a font is added to a collection
5. **font-removed**: Emitted when a font is removed from a collection

```javascript
_eventListeners = {
  'collection-created': [],
  'collection-updated': [],
  'collection-deleted': [],
  'font-added': [],
  'font-removed': []
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

### Creating and Managing Collections

```javascript
const fontCollectionService = new FontCollectionService();

// Create a new collection
fontCollectionService.createCollection('Sans-Serif Fonts', 'A collection of sans-serif fonts')
  .then(collection => {
    console.log('Created collection:', collection);
    
    // Add a font to the collection
    return fontCollectionService.addFontToCollection(collection.id, 'helvetica');
  })
  .then(collection => {
    console.log('Added font to collection:', collection);
    
    // Get all collections
    return fontCollectionService.getCollections();
  })
  .then(collections => {
    console.log('All collections:', collections);
  })
  .catch(error => {
    console.error('Error:', error);
  });
```

### Working with Favorites

```javascript
// Add a font to favorites
fontCollectionService.addToFavorites('arial')
  .then(favorites => {
    console.log('Added font to favorites:', favorites);
    
    // Check if a font is in favorites
    return fontCollectionService.isInFavorites('arial');
  })
  .then(isInFavorites => {
    console.log('Is font in favorites:', isInFavorites);
    
    // Get all favorites
    return fontCollectionService.getFavorites();
  })
  .then(favorites => {
    console.log('Favorites:', favorites);
  })
  .catch(error => {
    console.error('Error:', error);
  });
```

### Listening for Events

```javascript
// Listen for collection created events
fontCollectionService.addEventListener('collection-created', (data) => {
  console.log('Collection created:', data.collection);
});

// Listen for font added events
fontCollectionService.addEventListener('font-added', (data) => {
  console.log(`Font ${data.fontId} added to collection ${data.collectionId}`);
});
```

## Dependencies

The FontCollectionService will depend on the following:

1. **IndexedDB**: For persistent storage of collections
2. **EventEmitter**: For implementing the event system

## Testing Strategy

The FontCollectionService will be tested using the following approach:

1. **Unit Tests**: Test individual methods with mock data
2. **Integration Tests**: Test with actual IndexedDB
3. **Event Tests**: Test the event system
4. **Error Handling Tests**: Test error conditions and edge cases

## Error Handling

The FontCollectionService will handle the following error conditions:

1. **Collection Not Found**: Throw an error when trying to access a non-existent collection
2. **Database Errors**: Handle errors when interacting with IndexedDB
3. **Invalid Input**: Validate input parameters before performing operations
4. **Favorites Protection**: Prevent deletion of the favorites collection

## Performance Considerations

The FontCollectionService will implement the following performance optimizations:

1. **Caching**: Cache collections in memory for faster access
2. **Batch Operations**: Support batch operations for adding/removing multiple fonts
3. **Efficient Queries**: Use indexes for efficient querying
4. **Transaction Management**: Use transactions for atomic operations

## Future Enhancements

1. **Collection Sharing**: Allow sharing collections between users
2. **Collection Export/Import**: Support exporting and importing collections
3. **Collection Tags**: Add support for tagging collections
4. **Collection Sorting**: Add support for custom collection sorting
5. **Collection Thumbnails**: Generate thumbnails for collections
