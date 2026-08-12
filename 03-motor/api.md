# Module 3: Motor

The endpoints, the flow that binds a policy, the lifecycle and the error paths. The entities and
fields are in [`data-model.md`](data-model.md), and the rules that apply on every call are in
[`../conventions.md`](../conventions.md).

## Resource model

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

## Endpoints

Three resources, each with the same shape: create and list on the collection, retrieve and replace
on the item. `motorCoverage` adds three lifecycle actions that CRUD cannot express.

Each endpoint names the scope it requires. `opin-vn.admin` is the write scope and
`opin-vn.developer` is read-only. See [`../conventions.md`](../conventions.md).

| Endpoint | Scope | What it does |
| :--- | :--- | :--- |
| `POST /vehicle` | admin | Create a vehicle |
| `GET /vehicle` | developer | List vehicles |
| `GET /vehicle/{id}` | developer | Retrieve one vehicle |
| `PUT /vehicle/{id}` | admin | Replace a vehicle |
| `POST /driver` | admin | Create a driver |
| `GET /driver` | developer | List drivers |
| `GET /driver/{id}` | developer | Retrieve one driver |
| `PUT /driver/{id}` | admin | Replace a driver |
| `POST /motorCoverage` | admin | Bind a policy against an existing vehicle and driver |
| `GET /motorCoverage` | developer | List coverage, filterable by `policyNumber` |
| `GET /motorCoverage/{id}` | developer | Retrieve one coverage record |
| `PUT /motorCoverage/{id}` | admin | Replace a coverage record |
| `POST /motorCoverage/{id}:endorse` | admin | Amend a policy in force. Body carries an `endorsementType` |
| `POST /motorCoverage/{id}:cancel` | admin | End a policy before expiry |
| `POST /motorCoverage/{id}:renew` | admin | Issue a new coverage record for a new term |

Claims against a motor policy go to `POST /claim`, which serves every coverage type from one
endpoint. See [Claims](../11-claims/).

## Primary flow: bind a motor policy

The vehicle and the driver are created first, and the coverage refers to both. Three writes rather
than one, because a vehicle and a driver outlive the policy that binds them.

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

## Lifecycle

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

**This diagram is normative.** A transition it does not draw is not one an implementation may make.

Four states, and they are the `policyStatus` value set: in force, cancelled, lapsed, extended. Two
of the operations behave in ways worth stating plainly.

**Endorsing does not change the state.** An endorsement amends a policy that stays in force
throughout. It applies an `endorsementType` and mutates fields, and the policy is in force before
and after.

**Renewing does not transition the record.** It writes a second `motorCoverage` with its own policy
number, and the original runs to its own expiry. Two records, not one moved forward.

## Errors

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

Every error body is an RFC 7807 problem document, described in
[`../conventions.md`](../conventions.md). The distinction to note is between 404 and 400 here: a
missing referenced record is a 404 because the caller named something that does not exist, and an
expired licence or an inverted policy term is a 400 because the caller named something real and
described it wrongly.
