# Requirement Specification: Device IoT Registry API

## 1. Resource Schema
Each device resource must contain the following data fields:
- id (String / UUID): Unique identifier. Generated automatically by the backend on creation.
- deviceName (String): Required. Alphanumeric name of the asset.
- deviceType (String): Required. Restricted to values: ["sensor", "actuator", "gateway"].
- macAddress (String): Required. Must follow standard MAC string format.
- status (String): Optional. Defaults to "offline". Restricted to values: ["online", "offline", "maintenance"].
- telemetryInterval (Integer): Required. Must be a positive integer representing seconds.
- createdAt (Timestamp/ISO8601): Generated automatically by the backend on creation.

---

## 2. API Endpoint Matrix

### POST /api/v1/devices
- Description: Register a new IoT device.
- Request Body:
  ```json
  {
    "deviceName": "Thermostat-01",
    "deviceType": "sensor",
    "macAddress": "00:1A:2B:3C:4D:5E",
    "telemetryInterval": 60
  }
  ```
- Success Response (201 Created): Returns the complete object including generated id, status: "offline", and createdAt.
- Validation Rules: Return 400 Bad Request if deviceName is empty, deviceType is invalid, or telemetryInterval is negative.

### GET /api/v1/devices
- Description: Fetch all registered devices.
- Success Response (200 OK): Returns a JSON array of all items.

### GET /api/v1/devices/:id
- Description: Fetch a single device by its ID.
- Success Response (200 OK): Returns the matching device object.
- Error Response (404 Not Found): Returns when the ID does not exist.

### PUT /api/v1/devices/:id
- Description: Update mutable device settings (status or telemetryInterval).
- Request Body:
  ```json
  {
    "status": "online",
    "telemetryInterval": 30
  }
  ```
- Success Response (200 OK): Returns the updated resource object.
- Error Response (404 Not Found): Returns if the target ID is missing.

### DELETE /api/v1/devices/:id
- Description: Remove a device from the database system.
- Success Response (204 No Content): Returns empty body on successful deletion.
- Error Response (404 Not Found): Returns if the target ID does not exist.

---

## 3. Automation Task Instructions
1. Runtimes: Build this resource across Node.js (Port 3000), Spring Boot (Port 8080), and Go (Port 5000). Ensure the payload structures match down to the exact field casing (camelCase).
2. Database Verification: Confirm that SQLite (Node/Go) stores data permanently across application restarts, while H2 (Spring Boot) initializes a fresh schema in-memory.
3. Newman Parity Check: Pass these endpoint rules to api-tester. Have it run a global Newman regression collection containing tests for creation validation (201), structural arrays (200), missing lookups (404), and deletion teardowns (204) across all three application ports.


### 4. Test cases in table format
| Method | Endpoint | Request Body | Expected Response | Notes |
|--------|----------|--------------|------------------|-------| 
| POST   | /api/v1/devices | {"deviceName": "Thermostat-01", "deviceType": "sensor", "macAddress": "00:1A:2B:3C:4D:5E", "telemetryInterval": 60} | 201 Created | Creates a new device |
| GET    | /api/v1/devices | N/A | 200 OK | Fetches all registered devices |
| GET    | /api/v1/devices/:id | N/A | 200 OK | Fetches a single device by its ID |
| PUT    | /api/v1/devices/:id | {"status": "online", "telemetryInterval": 30} | 200 OK | Updates mutable device settings |
| DELETE | /api/v1/devices/:id | N/A | 204 No Content | Deletes a device by its ID |
| POST   | /api/v1/devices | {"deviceName": "", "deviceType": "sensor", "macAddress": "00:1A:2B:3C:4D:5E", "telemetryInterval": 60} | 400 Bad Request | Fails validation due to empty deviceName |
| POST   | /api/v1/devices | {"deviceName": "Thermostat-01", "deviceType": "invalidType", "macAddress": "00:1A:2B:3C:4D:5E", "telemetryInterval": 60} | 400 Bad Request | Fails validation due to invalid deviceType |
| POST   | /api/v1/devices | {"deviceName": "Thermostat-01", "deviceType": "sensor", "macAddress": "00:1A:2B:3C:4D:5E", "telemetryInterval": -10} | 400 Bad Request | Fails validation due to negative telemetryInterval |
| POST   | /api/v1/devices | {"deviceName": "Thermostat-01", "deviceType": "sensor", "macAddress": "", "telemetryInterval": 60} | 400 Bad Request | Fails validation due to empty macAddress |
| POST   | /api/v1/devices | {"deviceName": "Thermostat-01", "deviceType": "sensor", "macAddress": "00:1A:2B:3C:4D:5E", "telemetryInterval": 0} | 400 Bad Request | Fails validation due to zero telemetryInterval |  
| POST   | /api/v1/devices | {"deviceName": "Thermostat-01", "deviceType": "sensor", "macAddress": "00:1A:2B:3C:4D:5E"} | 400 Bad Request | Fails validation due to missing telemetryInterval |
| POST   | /api/v1/devices | {"deviceName": "Thermostat-01", "deviceType": "sensor", "telemetryInterval": 60} | 400 Bad Request | Fails validation due to missing macAddress |
| POST   | /api/v1/devices | {"deviceName": "Thermostat-01", "deviceType": "sensor"} | 400 Bad Request | Fails validation due to missing macAddress and telemetryInterval |