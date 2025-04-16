# Interlacid Framework Specification

## Overview

Interlacid (Interlinked-ACID) is a compliance framework for API passthroughs that ensures robust handling of failures through automatic fallback policies. It extends the traditional ACID (Atomicity, Consistency, Isolation, Durability) principles to distributed API interactions, providing predictable recovery patterns when operations fail.

## Core Principles

### 1. Automatic Rollback

When an API call fails, the system automatically reverts to the previous state without requiring explicit handling by the client. This ensures that failed operations don't leave the system in an inconsistent state.

### 2. Agreed Fallback Policies

Both client and server agree on fallback behaviors in advance, making the recovery process predictable and consistent. These policies are defined as part of the API specification.

### 3. Transaction Integrity

Operations are treated as atomic transactions when possible, ensuring that they either complete fully or have no effect at all. For operations that can't be made atomic, clear compensation mechanisms are defined.

### 4. Predictable Recovery

Users can rely on consistent recovery patterns across different operations and components, making the system behavior more predictable even in failure scenarios.

## Interlacid Transaction Flow

Each API operation follows this transaction flow:

1. **Snapshot**: Capture the current state before modification
2. **Attempt**: Perform the requested operation
3. **Verify**: Confirm the operation completed successfully
4. **Commit/Rollback**: Either commit the change or roll back to the snapshot
5. **Notify**: Inform the client of the outcome and any fallback actions taken

## Fallback Policy Types

### Cache Fallback

Returns cached data when the primary data source is unavailable.

```json
{
  "policyType": "cacheFallback",
  "cacheExpiration": 3600,
  "refreshBehavior": "background"
}
```

### Default Value Fallback

Returns a predefined default value when the requested resource is not found.

```json
{
  "policyType": "defaultValueFallback",
  "defaultValue": { "type": "empty" }
}
```

### Snapshot Rollback

Reverts to a previous state when an update operation fails.

```json
{
  "policyType": "snapshotRollback",
  "snapshotRetention": 5,
  "notifyOnRollback": true
}
```

### Soft Delete

Marks resources as deleted but preserves the data for potential recovery.

```json
{
  "policyType": "softDelete",
  "recoveryPeriod": 604800,
  "purgeAfterRecovery": true
}
```

### Retry with Backoff

Automatically retries failed operations with an exponential backoff.

```json
{
  "policyType": "retryWithBackoff",
  "maxRetries": 3,
  "initialDelayMs": 100,
  "backoffFactor": 2
}
```

### Partial Success

Accepts partial completion of batch operations, clearly indicating which items succeeded and which failed.

```json
{
  "policyType": "partialSuccess",
  "requireMinimumSuccess": 0.5,
  "detailedResults": true
}
```

## Implementation in API Specifications

Interlacid policies are defined in the API specification using a dedicated section:

```yaml
paths:
  /api/collections:
    post:
      summary: Create a new collection
      interlacid:
        policy: "fullRollback"
        description: "No partial collection creation"
        implementation:
          snapshotBefore: true
          verifyAfter: true
          notifyOnRollback: true
      responses:
        '200':
          description: Collection created successfully
        '400':
          description: Invalid collection data
          interlacid:
            action: "rollback"
            fallbackResponse:
              status: 400
              body:
                error: "Collection creation failed"
                rollbackStatus: "success"
```

## Response Format

API responses include information about any fallback actions taken:

```json
{
  "data": { ... },
  "meta": {
    "interlacid": {
      "fallbackApplied": true,
      "fallbackType": "cacheFallback",
      "originalError": "Database connection failed",
      "dataFreshness": "stale",
      "timestamp": 1623456789
    }
  }
}
```

## Client Implementation

Clients implementing Interlacid should:

1. Check for the `interlacid` field in response metadata
2. Handle fallback notifications appropriately
3. Provide UI feedback when fallbacks are applied
4. Support retry mechanisms when appropriate

Example client code:

```javascript
async function createCollection(name) {
  try {
    const response = await api.post('/api/collections', { name });
    
    // Check if fallback was applied
    if (response.meta?.interlacid?.fallbackApplied) {
      // Notify user of fallback
      notifyUser(`Operation completed with fallback: ${response.meta.interlacid.fallbackType}`);
      
      // Take appropriate action based on fallback type
      if (response.meta.interlacid.fallbackType === 'snapshotRollback') {
        // Refresh UI to show current state
        refreshCollections();
      }
    }
    
    return response.data;
  } catch (error) {
    // Handle complete failures
    console.error('Operation failed:', error);
    throw error;
  }
}
```

## Server Implementation

Servers implementing Interlacid should:

1. Define fallback policies for each API endpoint
2. Implement the transaction flow (snapshot, attempt, verify, commit/rollback)
3. Include fallback information in responses
4. Log fallback actions for monitoring and debugging

Example server code:

```javascript
async function createCollection(req, res) {
  // 1. Snapshot - No state to snapshot for creation
  
  try {
    // 2. Attempt
    const collection = await collectionService.create(req.body);
    
    // 3. Verify
    const verified = await collectionService.verify(collection.id);
    
    if (!verified) {
      // 4. Rollback
      await collectionService.delete(collection.id);
      
      // 5. Notify
      return res.status(500).json({
        error: "Collection creation verification failed",
        meta: {
          interlacid: {
            fallbackApplied: true,
            fallbackType: "fullRollback",
            originalError: "Verification failed"
          }
        }
      });
    }
    
    // Success case
    return res.status(200).json({
      data: collection,
      meta: {
        interlacid: {
          fallbackApplied: false
        }
      }
    });
  } catch (error) {
    // Handle errors
    return res.status(500).json({
      error: "Collection creation failed",
      meta: {
        interlacid: {
          fallbackApplied: true,
          fallbackType: "fullRollback",
          originalError: error.message
        }
      }
    });
  }
}
```

## Integration with Font-Viewer

In the Font-Viewer application, Interlacid is implemented for the following API endpoints:

1. **Font Index Operations**
   - GET /api/fonts - Cache fallback
   - GET /api/fonts/:id - Default font fallback

2. **Collection Operations**
   - POST /api/collections - Full rollback
   - PUT /api/collections/:id - Snapshot rollback
   - DELETE /api/collections/:id - Soft delete

The FontIndexService and FontCollectionService implement the Interlacid transaction flow, ensuring that operations are atomic and have clear fallback behaviors.

## Benefits of Interlacid

1. **Improved Reliability**: Systems continue to function even when components fail
2. **Predictable Behavior**: Users experience consistent recovery patterns
3. **Reduced Error Handling**: Less custom error handling code needed in clients
4. **Better User Experience**: Graceful degradation instead of hard failures
5. **Simplified Integration**: Clear expectations for how failures are handled

## Limitations and Considerations

1. **Performance Overhead**: Snapshots and verifications add some overhead
2. **Complexity**: More complex implementation compared to simple API calls
3. **Storage Requirements**: Snapshots and soft deletes require additional storage
4. **Eventual Consistency**: Some fallbacks may result in temporarily stale data

## Conclusion

The Interlacid framework provides a robust approach to handling API failures through automatic fallback policies. By implementing Interlacid in the Font-Viewer application, we ensure that users experience predictable behavior even when operations fail, improving the overall reliability and user experience of the system.
