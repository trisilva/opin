# Module 3: Motor

The entities, fields, enumerated values and relationships. The endpoints over them are in
[`api.md`](api.md), and the terms used throughout are defined in
[Insurance concepts](../concepts.md).

Six entities carry motor. **`motorCoverage`** is the policy: term, status, premium, perils and
deductibles. **`Vehicle`** is the thing insured and it is by far the largest entity in the standard.
**`Driver`** is the person being priced, with **`drivingLicence`**, **`conviction`** and
**`medicalCondition`** hanging off it because all three change what the risk costs. **`motorPeril`**
is the enumerated set of causes a motor policy will pay for.

Read `Vehicle` as three groups rather than one list. Static identity and registration (plate, VIN,
chassis, country) barely change. Manufacturer specification (body, fuel, power, doors, autonomy
level) is fixed at build. Telematics and condition (position, speed, tyre pressure, brake wear,
airbag state) change constantly, and are only present at all when `consentGranted` is true.

```mermaid
erDiagram
    MOTOR_COVERAGE {
        string policyNumber
        datetime inceptionDate "ISO 8601"
        datetime expiryDate "ISO 8601"
        enum status "policyStatus ref"
        float discountAmount
        float premiumRate
        float grossWrittenPremium
        float salesTax
        float brokeragePercentage
        float brokerageAmount
        int premiumPaymentFrequency
        int indemnityLimitPolicy
        int indemnityLimitAccident
        bool isAgreedValue
        string endorsementID
        datetime endorsementDate
        enum endorsementType
        enum peril "motorPeril ref"
        float voluntaryDeductiblePercentage
        int voluntaryDeductibleAmount
        float compulsoryDeductiblePercentage
        int compulsoryDeductibleAmount
        float windscreenDeductiblePercentage
        int windscreenDeductibleAmount
        enum distanceUnit
        float pleasureDistance
        float businessDistance
        int numberOfVehicles
    }
    DRIVER {
        string name
        enum gender
        date driverDOB
        bool isPrimaryDriver
        int noClaimsDiscount
        float loading "young driver loading"
        bool isBlueBadge
        string nonMotorConviction
        enum workStatus
        string occupation "ISCO-08"
    }
    DRIVING_LICENCE {
        string licenceNumber
        date issueDate
        date expiryDate
        string country "ISO 3166-1 alpha-2"
        string licenceCategory
        string licenceCodes
    }
    CONVICTION {
        date offenceDate
        enum offenceCode "offenceCode ref - DVLA UK"
        date date
        int points
        float fine
        enum fineCurrency "currency ref"
        bool suspension
        int suspensionLength
    }
    MEDICAL_CONDITION {
        enum notifiableCondition
        string status
        string medicalDvlaRestriction
        string medicalTreatment
        bool bypassOperation
        bool insulinInjected
        int dailyInsulinUnits
    }
    VEHICLE {
        string plateNumber
        date registrationDate
        string countryOfRegistration "ISO 3166-1 alpha-2"
        string chassisNumber
        string vin
        string engineNumber
        string vehicleWeight
        bool agencyRepair
        bool vehicleGarage
        enum bodyType
        enum fuelType
        enum aiClassification "SAE International"
        enum vehicleUse
        int yearlyMileage
        string vehicleBrand "KBA HSN/TSN"
        string vehicleModel
        date modelYear
        string seats
        string colour "ISO 14726"
        bool trailerIncluded
        int sumInsured
        string accessories
        int accessoryValue
        float engineCapacity "cc"
        bool co2Emissions
        bool automaticTransmission
        bool lefthandDrive
        bool blueBadgeAdapted
        int doors
        string securityDevice
        string modification
        bool digitalKeyUsed
        int power
        int torque
        int evPower "kilowatts"
        int evTorque
        string acceleration "0-60mph seconds"
        string vehicleTopSpeed
        bool hasTractionEnabled
        bool hasImmobilizer
        bool hasTheftDetection
        float currentMileage
        float currentMileageDynamic
        float yearlyMileageDynamic
        float highwayYearlyMileageDynamic
        float dailyMileageDynamic
        string serviceHistory
        bool serviceDue
        int timeToService
        string recallHistory
        bool tractionControlEngaged
        float accelerationLongitudinal
        float accelerationLateral
        float accelerationVertical
        float brakingFrequency
        float brakePedalForce
        float brakePedalSpeed
        bool performanceMode
        bool emergencyBraking
        bool ignitionOn
        bool ignitionOff
        float ignitionOnTime
        float ignitionOffTime
        float longitude
        float latitude
        float altitude
        float heading
        bool isMoving
        bool hornIsActive
        float drivingSpeed
        bool wheelSpin
        float decelerationRate
        float steeringSpeedTurn
        bool laneDepartureWarning
        bool adasAbsIsActive
        bool obstacleDetectionIsActive
        bool driverIntervention
        bool obstacleDetectionWarning
        float speedSet
        float yaw
        float pitch
        float roll
        float gForce
        bool row1Pos1Isbelted
        bool row1Pos2Isbelted
        bool row2Pos1Isbelted
        bool row2Pos2Isbelted
        float cabinTemp
        float cabinHumidity
        string tireConditionRow1Left
        string tireConditionRow1Right
        string tireConditionRow2Left
        string tireConditionRow2Right
        float tirePressureRow1Left
        bool tirePressureRow1LeftLow
        float brakePadWearRow1Left
        bool brakesWornRow1Right
        float clutchWear
        bool hasAirbagDeployed
        bool hasAbsError
        bool engineWarning
        string knownVehicleDamage
        string damagedParts
        float damagedPartsCost
        int occupiedSeats
        bool childSeatOccupiesSeat
        bool consentGranted
    }
    MOTOR_PERIL {
        int code "0-16, seventeen values"
        string description
    }

    MOTOR_COVERAGE ||--|{ DRIVER : "covers"
    MOTOR_COVERAGE ||--|{ VEHICLE : "insures"
    DRIVER ||--|| DRIVING_LICENCE : "holds"
    DRIVER ||--o{ CONVICTION : "has"
    DRIVER ||--o{ MEDICAL_CONDITION : "has"
    MOTOR_COVERAGE ||--|{ MOTOR_PERIL : "covers"
```

This diagram carries corrected field names throughout, because the data model governs what a field
means. Several of those names travel misspelled, and what you actually send is in
[What to watch](#what-to-watch) below. Take wire spellings from there or from
[`api.md`](api.md), never from this diagram.

## Selected fields

The fields most likely to need explanation. Everything else is in the diagram above.

| Entity | Field | Type | What it means |
|---|---|---|---|
| MotorCoverage | policyNumber | Text | The linkage key. Globally unique across every coverage type, which is what lets one claim endpoint serve all eight |
| MotorCoverage | inceptionDate | DateTime (ISO 8601) | When cover starts. A loss before this date is not covered |
| MotorCoverage | expiryDate | DateTime (ISO 8601) | When cover ends |
| MotorCoverage | status | enum (policyStatus) | In force, cancelled, lapsed or extended |
| MotorCoverage | premiumRate | Number/Float | The rate the premium is calculated at, as a percentage |
| MotorCoverage | grossWrittenPremium | Number/Float | The full contracted premium for the term, before tax and before acquisition costs are deducted |
| MotorCoverage | indemnityLimitPolicy | Number/integer | The most the policy pays across the whole year |
| MotorCoverage | indemnityLimitAccident | Number/integer | The most it pays for any one accident. Not the same cap, and both apply |
| MotorCoverage | peril | enum (motorPeril) | What the policy pays for. Seventeen values: third-party liability, fire, theft, accidental damage, windshield damage, malicious damage, terrorism and sabotage, flood, earthquake, volcanic eruption, tsunami, hail, unknown or hit-and-run, riots, strikes, civil commotion, war |
| MotorCoverage | compulsoryDeductibleAmount | Number/integer | The excess the insurer imposes. The customer cannot decline it |
| MotorCoverage | voluntaryDeductibleAmount | Number/integer | Additional excess the customer accepts in exchange for a cheaper premium |
| MotorCoverage | windscreenDeductibleAmount | Number/integer | A separate, usually lower, excess for glass-only claims. Glass is claimed often and cheap to fix, so it is priced apart from the rest |
| MotorCoverage | distanceUnit | enum (distanceUnit) | Kilometres or miles. Set it before you read any distance field |
| MotorCoverage | pleasureDistance | Number/Float | Distance driven privately. Supports pay-as-you-drive pricing, where premium follows usage |
| MotorCoverage | businessDistance | Number/Float | Distance driven for work, priced differently because the exposure differs |
| Driver | driverDOB | Date | Date of birth. Age is the single strongest predictor in motor pricing |
| Driver | isPrimaryDriver | Boolean | Whether this is the main driver. A policy has one, and additional drivers are priced against it |
| Driver | licence | ref (drivingLicence) | The licence held, with issue date, expiry and category |
| Driver | noClaimsDiscount | Number/integer | Years of claim-free driving, which earns a discount that grows year on year |
| Driver | conviction | ref (conviction) | Motoring offences, multi-valued. Carries points, fine and any suspension |
| Driver | medicalCondition | ref (medicalCondition) | Conditions that affect fitness to drive |
| Driver | loading | Number/Float | A percentage added to the premium for identified extra risk, typically a young or newly licensed driver |
| Driver | isBlueBadge | Boolean | Whether the driver holds a UK disabled parking badge |
| Driver | workStatus | enum (workStatus) | Self-employed, retired, employed or redundant |
| Vehicle | plateNumber | Text | Registration plate |
| Vehicle | vin | Text | The 17-character vehicle identification number, unique to the vehicle worldwide |
| Vehicle | countryOfRegistration | Text (ISO 3166-1 alpha-2) | Where the vehicle is registered |
| Vehicle | bodyType | enum | Twelve values, from motor car to construction equipment |
| Vehicle | fuelType | enum | Petrol, diesel, electric, hybrid, gas or hydrogen |
| Vehicle | aiClassification | enum | Self-driving capability on the SAE International scale, level 0 (none) to level 5 (fully autonomous) |
| Vehicle | vehicleUse | enum | Business, business and leisure, commercial, sharing or subscription. Changes the exposure and therefore the price |
| Vehicle | sumInsured | Number/integer | What the vehicle is insured for, and the basis the premium is calculated from |
| Vehicle | currentMileageDynamic | Number/Float | Live odometer reading, sent by the vehicle rather than declared by the customer |
| Vehicle | hasAirbagDeployed | Boolean | Crash signal. Present because a deployed airbag is a strong indicator that a claim is coming |
| Vehicle | consentGranted | Boolean | Whether the driver agreed to share telematics. Gates every dynamic field on this entity |

## What to watch

**`Vehicle` is one entity carrying more than a hundred fields.** They span registration, manufacturer
specification, live telematics, driver assistance state, seat occupancy, tyre pressure, brake wear
and consent, and the standard does not factor them into sub-entities. Two consequences when you
build. Most of the entity has to be optional, because almost no caller populates all of it. And
static attributes sit in the same record as high-frequency telematics, so a naive implementation
rewrites the whole row to record a change in speed.

**Several field names on the wire are misspelled.** They are stable in every existing implementation,
so they stay as they are and the corrections wait for a major version. Send the left-hand column.

| Send this | It was meant to be |
| :--- | :--- |
| `countryOfRegisteration` | `countryOfRegistration` |
| `engnitionOn`, `engnitionOff` | `ignitionOn`, `ignitionOff` |
| `logitude` | `longitude` |
| `laneDepartureWarnning` | `laneDepartureWarning` |
| `decelrationRate` | `decelerationRate` |
| `yearlyMilage`, and the `Dynamic` variants of it | `yearlyMileage` |
| `medicalConditon` (on `Driver`) | `medicalCondition`, which is also the correct name of the enum it points at |

**Two `motorPeril` descriptions are misspelled**, `unkown or hit and run` and
`volcanic erruption`. The codes are what travel and the codes are stable, so match on code rather
than on description text.

**Telematics fields are storage, not a stream.** The vehicle record holds the latest values. Ingesting
telematics, streaming events, computing usage-based billing and correlating driver-assistance events
all sit above the standard, in whatever platform you build. See [`../SCOPE.md`](../SCOPE.md).
