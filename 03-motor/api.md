# Module 3: Motor

Resources, endpoints, primary flow, lifecycle and routing. The entities and fields are in [`data-model.md`](data-model.md). Conventions that apply to every module are in [`../conventions.md`](../conventions.md).

### Resource model

```mermaid
classDiagram
    class MotorCoverage {
        +UUID id
        +string policyNumber
        +DateTime inceptionDate
        +DateTime expiryDate
        +PolicyStatus status
        +float grossWrittenPremium
        +int indemnityLimitPolicy
        +MotorPeril[] perils
        +create() MotorCoverage
        +retrieve(id) MotorCoverage
        +endorse(id, endorsementType) MotorCoverage
        +cancel(id) MotorCoverage
        +renew(id) MotorCoverage
    }
    class Vehicle {
        +UUID id
        +string plateNumber
        +string vin
        +string chassisNumber
        +VehicleRegistration registration
        +VehiclePhysical physical
        +VehiclePerformance performance
        +VehicleCondition condition
        +create() Vehicle
        +retrieve(id) Vehicle
        +update(id) Vehicle
    }
    class Driver {
        +UUID id
        +string name
        +Date driverDOB
        +DrivingLicence licence
        +Conviction[] convictions
        +bool isPrimaryDriver
        +create() Driver
        +retrieve(id) Driver
        +update(id) Driver
    }
    MotorCoverage --> Vehicle : insures
    MotorCoverage --> Driver : covers
    Driver --> DrivingLicence : holds
```

### Endpoints

OPIN spec endpoints (kept verbatim, motor only, attributed to OPIN):

- `POST /vehicle` (admin) `[OPIN]`
- `GET /vehicle` (developer) `[OPIN]`
- `POST /claim` (admin) `[OPIN]`: polymorphic across coverages, see Module 11
- `GET /claim` (developer) `[OPIN]`
- `POST /driver` (admin) `[OPIN]`
- `GET /driver` (developer) `[OPIN]`
- `POST /motorCoverage` (admin) `[OPIN]`
- `GET /motorCoverage` (developer) `[OPIN]`

OPIN-VN extensions on the same OPIN schemas:

- `GET /vehicle/{id}` (developer) `[OPIN-VN extension to API; OPIN schema reused]`
- `PUT /vehicle/{id}` (admin) `[OPIN-VN extension to API; OPIN schema reused]`
- `GET /driver/{id}` (developer) `[OPIN-VN extension to API; OPIN schema reused]`
- `PUT /driver/{id}` (admin) `[OPIN-VN extension to API; OPIN schema reused]`
- `GET /motorCoverage/{id}` (developer) `[OPIN-VN extension to API; OPIN schema reused]`
- `PUT /motorCoverage/{id}` (admin) `[OPIN-VN extension to API; OPIN schema reused]`
- `POST /motorCoverage/{id}:endorse` (admin) `[OPIN-VN extension to API; OPIN schema reused]`: body carries OPIN `endorsementType`
- `POST /motorCoverage/{id}:cancel` (admin) `[OPIN-VN extension to API; OPIN schema reused]`
- `POST /motorCoverage/{id}:renew` (admin) `[OPIN-VN extension to API; OPIN schema reused]`

### Primary flow: Bind a motor policy

```mermaid
sequenceDiagram
    participant Client as Client App
    participant Gateway as API Gateway
    participant Motor as Motor Service
    participant Vehicle as Vehicle Service
    participant Driver as Driver Service
    participant Party as Party Service
    Client->>Gateway: POST /vehicle {plateNumber, vin, ...}
    Gateway->>Vehicle: createVehicle(payload)
    Vehicle-->>Gateway: 201 {vehicleId}
    Client->>Gateway: POST /driver {licence, dob, ...}
    Gateway->>Driver: createDriver(payload)
    Driver-->>Gateway: 201 {driverId}
    Client->>Gateway: POST /motorCoverage {policyholderId, vehicleId, driverId, perils, premium}
    Gateway->>Motor: createMotorCoverage(payload)
    Motor->>Party: validate policyholderId
    Motor->>Vehicle: validate vehicleId
    Motor->>Driver: validate driverId
    Motor->>Motor: Persist with policyStatus=in force
    Motor-->>Gateway: 201 {policyNumber, status: in force}
    Gateway-->>Client: 201 Created
```

### Lifecycle

```mermaid
stateDiagram-v2
    [*] --> InForce : POST /motorCoverage
    InForce --> InForce : :endorse (endorsementType applied, status unchanged)
    InForce --> Cancelled : :cancel
    InForce --> Lapsed : non-payment
    InForce --> Extended : :endorse with endorsementType=policy extension
    Extended --> InForce : extension term begins
    InForce --> InForce : :renew (issues a new motorCoverage record)
    Cancelled --> [*]
    Lapsed --> [*]
```

States are exactly OPIN `policyStatus` (in force, cancelled, lapsed, extended). Endorsement is an operation on an in-force policy that mutates fields under an OPIN `endorsementType` value but does not introduce a new lifecycle state. Renewal issues a new record rather than transitioning the existing one.

### Routing and error handling

```mermaid
flowchart TD
    A[POST /motorCoverage] --> B{Policyholder exists?}
    B -->|No| E1[404 - policyholder not found]
    B -->|Yes| C{Vehicle exists?}
    C -->|No| E2[404 - vehicle not found]
    C -->|Yes| D{Driver exists and licence valid?}
    D -->|No| E3[400 - driver licence expired or missing]
    D -->|Yes| F{Inception &lt; expiry?}
    F -->|No| E4[400 - invalid policy term]
    F -->|Yes| G{Perils all in motorPeril enum?}
    G -->|No| E5[400 - unknown peril code]
    G -->|Yes| H[Persist with policyStatus=in force]
    H --> I[201 Created]
```
