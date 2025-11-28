# Notification Endpoint - Code Explanation

## Table of Contents
1. [Architecture Overview](#architecture-overview)
2. [Request Flow](#request-flow)
3. [Component Breakdown](#component-breakdown)
4. [Line-by-Line Code Explanation](#line-by-line-code-explanation)
5. [Example Request/Response](#example-requestresponse)

---

## Architecture Overview

The notification endpoint follows a standard **3-tier Spring Boot architecture**:

```
┌──────────────────────┐
│   Frontend/Client    │
└──────────┬───────────┘
           │ HTTP GET /api/v1/notifications/{userId}
           ▼
┌──────────────────────┐
│ NotificationController│ ◄── REST Layer (handles HTTP)
└──────────┬───────────┘
           │ calls getUserNotifications()
           ▼
┌──────────────────────┐
│ NotificationService  │ ◄── Business Logic Layer
└──────────┬───────────┘
           │ returns
           ▼
┌──────────────────────┐
│   DTOs/Models        │ ◄── Data Transfer Objects
│ - NotificationResponse│
│ - Notification       │
│ - NotificationType   │
│ - Priority           │
└──────────────────────┘
```

---

## Request Flow

**Step-by-step flow when a client requests notifications:**

1. **Client makes HTTP request**: `GET /api/v1/notifications/user123`
2. **Spring routes to NotificationController**: Matches the `/api/v1` prefix and `/notifications/{userId}` pattern
3. **Controller extracts userId**: From path variable (`user123`)
4. **Controller calls service**: `notificationService.getUserNotifications("user123")`
5. **Service generates notifications**: Creates sample notification data with metadata
6. **Service returns DTO**: `NotificationResponse` object
7. **Spring serializes to JSON**: Converts Java objects to JSON automatically
8. **Controller returns HTTP response**: `ResponseEntity.ok(response)` → HTTP 200 with JSON body

---

## Component Breakdown

### 1. Controller Layer
**File**: `NotificationController.java`
- **Purpose**: Handle HTTP requests and responses
- **Responsibility**: Route mapping, request validation, response formatting

### 2. Service Layer
**File**: `NotificationService.java`
- **Purpose**: Business logic for notification generation
- **Responsibility**: Create notifications, populate metadata, apply business rules

### 3. Data Transfer Objects (DTOs)
**Files**: `NotificationResponse.java`, `Notification.java`, `NotificationType.java`, `Priority.java`
- **Purpose**: Define data structure for API responses
- **Responsibility**: Ensure consistent JSON format for frontend

---

## Line-by-Line Code Explanation

### NotificationController.java

```java
package com.trinity.hermes.api;
```
**Line 1**: Package declaration - organizes code under `com.trinity.hermes.api` namespace

```java
import com.trinity.hermes.dto.NotificationResponse;
import com.trinity.hermes.service.NotificationService;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
```
**Lines 3-11**: Import statements
- `NotificationResponse`: Our custom DTO for response structure
- `NotificationService`: Service class containing business logic
- `RequiredArgsConstructor`: Lombok annotation for constructor injection
- `Slf4j`: Lombok annotation for logging
- `ResponseEntity`: Spring class for HTTP responses with status codes
- `GetMapping`: Annotation for HTTP GET endpoints
- `PathVariable`: Extract variables from URL path
- `RequestMapping`: Base URL mapping for the controller
- `RestController`: Marks this as a REST API controller

```java
@Slf4j
```
**Line 13**: Lombok annotation that automatically creates a logger field
- Creates: `private static final Logger log = LoggerFactory.getLogger(NotificationController.class);`
- Allows us to use `log.info()`, `log.error()`, etc.

```java
@RestController
```
**Line 14**: Spring annotation that:
- Marks this class as a REST controller
- Combines `@Controller` + `@ResponseBody`
- Automatically serializes return values to JSON

```java
@RequestMapping("/api/v1")
```
**Line 15**: Base path for all endpoints in this controller
- All methods will be prefixed with `/api/v1`
- Example: `/api/v1/notifications/{userId}`
- **Versioning**: `/v1` allows future API versions (`/v2`, `/v3`) without breaking existing clients

```java
@RequiredArgsConstructor
```
**Line 16**: Lombok annotation that generates a constructor
- Creates constructor for all `final` fields
- Enables **constructor-based dependency injection** (Spring best practice)
- Equivalent to manually writing:
  ```java
  public NotificationController(NotificationService notificationService) {
      this.notificationService = notificationService;
  }
  ```

```java
public class NotificationController {
```
**Line 17**: Class declaration - contains REST endpoint methods

```java
    private final NotificationService notificationService;
```
**Line 19**: Dependency injection field
- `private`: Only accessible within this class
- `final`: Must be initialized in constructor (immutability)
- Spring will inject an instance of `NotificationService` here

```java
    @GetMapping("/notifications/{userId}")
```
**Line 21**: Maps HTTP GET requests
- **Full path**: `/api/v1/notifications/{userId}` (combines with `@RequestMapping`)
- `{userId}`: Path variable (placeholder) - matches any value
- Examples:
  - `/api/v1/notifications/user123` → `userId = "user123"`
  - `/api/v1/notifications/abc-xyz` → `userId = "abc-xyz"`

```java
    public ResponseEntity<NotificationResponse> getNotifications(@PathVariable String userId) {
```
**Line 22**: Method signature
- `public`: Accessible from outside the class
- `ResponseEntity<NotificationResponse>`: Return type
  - `ResponseEntity`: Allows setting HTTP status codes (200, 404, 500, etc.)
  - `<NotificationResponse>`: The response body type
- `getNotifications`: Method name
- `@PathVariable String userId`: Extracts `{userId}` from URL path
  - Spring automatically binds the URL segment to this parameter

```java
        log.info("Fetching notifications for user: {}", userId);
```
**Line 23**: Logging statement
- `log.info()`: Logs at INFO level
- `{}`: Placeholder replaced with `userId` value
- Output example: `Fetching notifications for user: user123`
- **Purpose**: Debugging, monitoring, audit trail

```java
        NotificationResponse response = notificationService.getUserNotifications(userId);
```
**Line 24**: Call service layer
- Delegates business logic to `NotificationService`
- Passes `userId` to service method
- Receives `NotificationResponse` object containing notifications

```java
        return ResponseEntity.ok(response);
```
**Line 25**: Return HTTP response
- `ResponseEntity.ok()`: Creates HTTP 200 (OK) response
- `response`: Body of the HTTP response (auto-converted to JSON)
- Equivalent to:
  ```java
  return ResponseEntity
      .status(HttpStatus.OK)
      .body(response);
  ```

---

### NotificationService.java

```java
package com.trinity.hermes.service;
```
**Line 1**: Package declaration for service layer

```java
import com.trinity.hermes.dto.Notification;
import com.trinity.hermes.dto.NotificationResponse;
import com.trinity.hermes.dto.NotificationType;
import com.trinity.hermes.dto.Priority;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.UUID;
```
**Lines 3-15**: Import statements
- DTOs for data structures
- `Slf4j`: Logging
- `Service`: Spring annotation
- `LocalDateTime`: For timestamps
- Collections: `List`, `Map`, `ArrayList`, `HashMap`
- `UUID`: Generate unique notification IDs

```java
@Slf4j
```
**Line 17**: Lombok annotation for logging (same as controller)

```java
@Service
```
**Line 18**: Spring annotation
- Marks this as a service component
- Spring automatically creates a singleton instance
- Makes it available for dependency injection
- Part of Spring's component scanning

```java
public class NotificationService {
```
**Line 19**: Service class declaration

```java
    public NotificationResponse getUserNotifications(String userId) {
```
**Line 21**: Public method called by controller
- Takes `userId` as parameter
- Returns `NotificationResponse` DTO
- Contains all business logic for fetching notifications

```java
        log.info("Generating notifications for user: {}", userId);
```
**Line 22**: Log the operation
- Helps track which user's notifications are being generated
- Useful for debugging and monitoring

```java
        List<Notification> notifications = generateNotifications(userId);
```
**Line 24**: Generate notification list
- Calls private helper method
- Returns list of `Notification` objects
- Currently generates sample data (in real app, might fetch from database)

```java
        return NotificationResponse.builder()
                .userId(userId)
                .notifications(notifications)
                .totalCount(notifications.size())
                .build();
```
**Lines 26-30**: Build and return response object
- `NotificationResponse.builder()`: Lombok builder pattern (fluent API)
- `.userId(userId)`: Set the user ID field
- `.notifications(notifications)`: Set the list of notifications
- `.totalCount(notifications.size())`: Count of notifications
- `.build()`: Create the final object
- **Builder pattern benefits**: Readable, immutable, prevents invalid states

```java
    private List<Notification> generateNotifications(String userId) {
```
**Line 33**: Private helper method
- Only called within this class
- Generates sample notifications
- In production: would query database or call recommendation engine

```java
        List<Notification> notifications = new ArrayList<>();
```
**Line 34**: Initialize empty list
- `ArrayList`: Resizable array implementation
- Will store `Notification` objects

```java
        // Route recommendation notification
        notifications.add(Notification.builder()
                .id(UUID.randomUUID().toString())
                .type(NotificationType.ROUTE_RECOMMENDATION)
                .message("Recommended route to downtown: Via Highway 101 saves 15 minutes")
                .priority(Priority.MEDIUM)
                .timestamp(LocalDateTime.now().minusMinutes(10))
                .metadata(createRouteMetadata("Highway 101", 15, "downtown"))
                .read(false)
                .build());
```
**Lines 36-45**: Create first notification (Route Recommendation)
- `UUID.randomUUID().toString()`: Generate unique ID like `"550e8400-e29b-41d4-a716-446655440000"`
- `NotificationType.ROUTE_RECOMMENDATION`: Enum value for type
- `message`: Human-readable text for frontend
- `Priority.MEDIUM`: Importance level
- `LocalDateTime.now().minusMinutes(10)`: Timestamp 10 minutes ago
- `metadata`: Additional structured data (route name, time saved, destination)
- `read: false`: Mark as unread
- `.build()`: Create the notification object

```java
        // Alert notification
        notifications.add(Notification.builder()
                .id(UUID.randomUUID().toString())
                .type(NotificationType.ALERT)
                .message("Heavy traffic detected on your usual route")
                .priority(Priority.HIGH)
                .timestamp(LocalDateTime.now().minusMinutes(5))
                .metadata(createAlertMetadata("Main Street", "heavy_traffic"))
                .read(false)
                .build());
```
**Lines 47-55**: Create second notification (Alert)
- Similar structure to first notification
- `NotificationType.ALERT`: Different type
- `Priority.HIGH`: Higher priority than route recommendation
- `minusMinutes(5)`: More recent (5 minutes ago)
- Alert-specific metadata (location, alert type)

```java
        // Update notification
        notifications.add(Notification.builder()
                .id(UUID.randomUUID().toString())
                .type(NotificationType.UPDATE)
                .message("Your route preferences have been updated based on recent travel patterns")
                .priority(Priority.LOW)
                .timestamp(LocalDateTime.now().minusHours(1))
                .metadata(createUpdateMetadata("route_preferences"))
                .read(true)
                .build());
```
**Lines 57-65**: Create third notification (Update)
- `NotificationType.UPDATE`: System update type
- `Priority.LOW`: Less urgent
- `minusHours(1)`: Older notification (1 hour ago)
- `read: true`: Already read by user
- Update-specific metadata

```java
        return notifications;
```
**Line 67**: Return the list of notifications

```java
    private Map<String, Object> createRouteMetadata(String routeName, int timeSaved, String destination) {
```
**Line 70**: Helper method for route metadata
- Returns `Map<String, Object>`: Key-value pairs
- `Object` value type: Allows different data types (String, Integer, Boolean)

```java
        Map<String, Object> metadata = new HashMap<>();
```
**Line 71**: Create new HashMap
- `HashMap`: Key-value storage

```java
        metadata.put("routeName", routeName);
        metadata.put("timeSavedMinutes", timeSaved);
        metadata.put("destination", destination);
        metadata.put("alternativeAvailable", true);
```
**Lines 72-75**: Add route-specific metadata
- `routeName`: Which route is recommended
- `timeSavedMinutes`: How much time saved (integer)
- `destination`: Where the route goes
- `alternativeAvailable`: Boolean flag
- **Frontend can use this data**: Show route on map, display time savings, etc.

```java
        return metadata;
```
**Line 76**: Return the populated map

```java
    private Map<String, Object> createAlertMetadata(String location, String alertType) {
        Map<String, Object> metadata = new HashMap<>();
        metadata.put("location", location);
        metadata.put("alertType", alertType);
        metadata.put("severity", "high");
        return metadata;
    }
```
**Lines 79-85**: Alert metadata helper
- Similar structure to route metadata
- Alert-specific fields: location, alertType, severity

```java
    private Map<String, Object> createUpdateMetadata(String updateType) {
        Map<String, Object> metadata = new HashMap<>();
        metadata.put("updateType", updateType);
        metadata.put("source", "recommendation_engine");
        return metadata;
    }
```
**Lines 87-92**: Update metadata helper
- Update-specific fields
- Tracks what was updated and source system

---

### NotificationResponse.java (DTO)

```java
package com.trinity.hermes.dto;
```
**Line 1**: Package for DTOs

```java
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.util.List;
```
**Lines 3-7**: Imports
- Lombok annotations for reducing boilerplate
- `List` for storing notifications

```java
@Data
```
**Line 9**: Lombok annotation that generates:
- `toString()`: String representation
- `equals()` and `hashCode()`: Object comparison
- Getters for all fields: `getUserId()`, `getNotifications()`, `getTotalCount()`
- Setters for all fields: `setUserId()`, `setNotifications()`, `setTotalCount()`

```java
@Builder
```
**Line 10**: Lombok annotation
- Generates builder pattern code
- Enables fluent API: `NotificationResponse.builder().userId("123").build()`
- Makes object construction readable

```java
@NoArgsConstructor
```
**Line 11**: Lombok annotation
- Generates no-argument constructor: `new NotificationResponse()`
- Required for JSON deserialization (Jackson)

```java
@AllArgsConstructor
```
**Line 12**: Lombok annotation
- Generates constructor with all fields
- Example: `new NotificationResponse("user123", notifications, 3)`

```java
public class NotificationResponse {
```
**Line 13**: Class declaration

```java
    private String userId;
```
**Line 14**: User identifier field
- Will be serialized to JSON as `"userId": "user123"`

```java
    private List<Notification> notifications;
```
**Line 15**: List of notification objects
- Will be serialized to JSON array
- Contains multiple `Notification` objects

```java
    private int totalCount;
```
**Line 16**: Total number of notifications
- Useful for pagination
- Frontend can show "You have 3 new notifications"

---

### Notification.java (DTO)

```java
package com.trinity.hermes.dto;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.time.LocalDateTime;
import java.util.Map;
```
**Lines 1-9**: Package and imports (similar to NotificationResponse)

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
```
**Lines 11-14**: Same Lombok annotations as before

```java
public class Notification {
```
**Line 15**: Individual notification class

```java
    private String id;
```
**Line 16**: Unique identifier
- UUID string like `"550e8400-e29b-41d4-a716-446655440000"`
- Frontend can use this to mark notifications as read

```java
    private NotificationType type;
```
**Line 17**: Notification category
- Enum: `ROUTE_RECOMMENDATION`, `ALERT`, `UPDATE`, `SYSTEM`
- Frontend can use different icons/colors based on type

```java
    private String message;
```
**Line 18**: Human-readable message
- Displayed directly to user
- Example: "Recommended route to downtown: Via Highway 101 saves 15 minutes"

```java
    private Priority priority;
```
**Line 19**: Importance level
- Enum: `LOW`, `MEDIUM`, `HIGH`, `URGENT`
- Frontend can sort by priority or show urgent notifications first

```java
    private LocalDateTime timestamp;
```
**Line 20**: When notification was created
- Serialized to ISO-8601 format: `"2025-11-28T10:30:00"`
- Frontend can display relative time: "10 minutes ago"

```java
    private Map<String, Object> metadata;
```
**Line 21**: Additional structured data
- Flexible key-value storage
- Different notification types have different metadata
- Examples:
  - Route: `{"routeName": "Highway 101", "timeSavedMinutes": 15}`
  - Alert: `{"location": "Main Street", "alertType": "heavy_traffic"}`

```java
    private boolean read;
```
**Line 22**: Read status
- `false`: Unread (new notification)
- `true`: Already seen by user
- Frontend can show unread count badge

---

### NotificationType.java (Enum)

```java
package com.trinity.hermes.dto;

public enum NotificationType {
    ROUTE_RECOMMENDATION,
    ALERT,
    UPDATE,
    SYSTEM
}
```
**Lines 1-8**: Enum definition
- **Enum**: Fixed set of constants
- **ROUTE_RECOMMENDATION**: Route suggestions
- **ALERT**: Warnings (traffic, weather, etc.)
- **UPDATE**: System updates, preference changes
- **SYSTEM**: General system messages
- Serialized to JSON as strings: `"type": "ROUTE_RECOMMENDATION"`

---

### Priority.java (Enum)

```java
package com.trinity.hermes.dto;

public enum Priority {
    LOW,
    MEDIUM,
    HIGH,
    URGENT
}
```
**Lines 1-8**: Priority levels enum
- **LOW**: Non-urgent updates
- **MEDIUM**: Normal recommendations
- **HIGH**: Important alerts
- **URGENT**: Critical notifications requiring immediate attention
- Serialized to JSON as strings: `"priority": "HIGH"`

---

## Example Request/Response

### Request
```http
GET /api/v1/notifications/user123 HTTP/1.1
Host: localhost:8080
Accept: application/json
```

### Flow Through Code

1. **Spring receives request** → Routes to `NotificationController`
2. **Controller extracts userId** → `"user123"` from path
3. **Controller logs** → `"Fetching notifications for user: user123"`
4. **Controller calls service** → `notificationService.getUserNotifications("user123")`
5. **Service logs** → `"Generating notifications for user: user123"`
6. **Service generates notifications** → Creates 3 sample notifications
7. **Service builds response** → `NotificationResponse` with userId, notifications, count
8. **Controller wraps in ResponseEntity** → HTTP 200 status
9. **Spring serializes to JSON** → Converts Java objects to JSON
10. **HTTP response sent to client**

### Response
```json
HTTP/1.1 200 OK
Content-Type: application/json

{
  "userId": "user123",
  "notifications": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "type": "ROUTE_RECOMMENDATION",
      "message": "Recommended route to downtown: Via Highway 101 saves 15 minutes",
      "priority": "MEDIUM",
      "timestamp": "2025-11-28T10:20:00",
      "metadata": {
        "routeName": "Highway 101",
        "timeSavedMinutes": 15,
        "destination": "downtown",
        "alternativeAvailable": true
      },
      "read": false
    },
    {
      "id": "7c9e6679-7425-40de-944b-e07fc1f90ae7",
      "type": "ALERT",
      "message": "Heavy traffic detected on your usual route",
      "priority": "HIGH",
      "timestamp": "2025-11-28T10:25:00",
      "metadata": {
        "location": "Main Street",
        "alertType": "heavy_traffic",
        "severity": "high"
      },
      "read": false
    },
    {
      "id": "3f333df6-90a4-4fda-8dd3-9485d27cee36",
      "type": "UPDATE",
      "message": "Your route preferences have been updated based on recent travel patterns",
      "priority": "LOW",
      "timestamp": "2025-11-28T09:30:00",
      "metadata": {
        "updateType": "route_preferences",
        "source": "recommendation_engine"
      },
      "read": true
    }
  ],
  "totalCount": 3
}
```

---

## Key Design Patterns Used

### 1. **Dependency Injection (DI)**
- **Where**: `NotificationController` → `NotificationService`
- **How**: Constructor injection via `@RequiredArgsConstructor`
- **Why**: Loose coupling, easier testing, Spring manages lifecycle

### 2. **Builder Pattern**
- **Where**: All DTOs (`@Builder` annotation)
- **How**: Fluent API for object construction
- **Why**: Readable code, immutable objects, prevents invalid states

### 3. **Data Transfer Object (DTO)**
- **Where**: `NotificationResponse`, `Notification`
- **How**: Plain Java objects for data transfer
- **Why**: Decouples API contract from domain model, versioning support

### 4. **Layered Architecture**
- **Layers**: Controller → Service → DTOs
- **Why**: Separation of concerns, maintainability, testability

### 5. **RESTful API Design**
- **Resource**: `/notifications`
- **Versioning**: `/api/v1`
- **HTTP Method**: `GET` for retrieving data
- **Status Codes**: `200 OK` for successful requests

---

## How Frontend Will Use This

```javascript
// Frontend JavaScript/TypeScript example
async function fetchNotifications(userId) {
  const response = await fetch(`/api/v1/notifications/${userId}`);
  const data = await response.json();

  // Display total count
  showBadge(data.totalCount);

  // Render each notification
  data.notifications.forEach(notification => {
    const icon = getIconForType(notification.type);
    const color = getColorForPriority(notification.priority);
    const timeAgo = formatTimestamp(notification.timestamp);

    renderNotification({
      id: notification.id,
      icon: icon,
      message: notification.message,
      time: timeAgo,
      color: color,
      isRead: notification.read,
      metadata: notification.metadata // Can show route on map, etc.
    });
  });
}
```

---

## Future Enhancements

1. **Database Integration**: Replace sample data with real database queries
2. **Pagination**: Add `page` and `size` query parameters
3. **Filtering**: Filter by type, priority, read/unread status
4. **Real-time Updates**: WebSocket support for push notifications
5. **Mark as Read**: `PATCH /api/v1/notifications/{id}/read`
6. **Delete Notification**: `DELETE /api/v1/notifications/{id}`
7. **User Preferences**: Customize notification types per user

---

## Summary

This implementation provides a **thin slice** of a notification system that:
- ✅ Exposes a versioned REST API
- ✅ Returns frontend-ready JSON
- ✅ Includes metadata for rich UI experiences
- ✅ Supports multiple notification types with priorities
- ✅ Follows Spring Boot best practices
- ✅ Uses Lombok to reduce boilerplate
- ✅ Implements proper separation of concerns

The code is production-ready for the thin slice requirement and can be extended with database integration, authentication, and more advanced features as needed.
