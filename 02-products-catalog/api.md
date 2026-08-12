# Module 2: Products and Catalog

The endpoints, the flow that registers a product, the lifecycle and the error paths. The entities
and fields are in [`data-model.md`](data-model.md), and the rules that apply on every call are in
[`../conventions.md`](../conventions.md).

## Resource model

```mermaid
classDiagram
    class Product {
        +UUID id
        +ProductCatalog lineOfBusiness
        +ProductModel productModel
        +ContractType contractType
        +Currency currency
        +float policyFee
        +PremiumPaymentFrequency paymentFrequency
        +create() Product
        +retrieve(id) Product
        +update(id) Product
        +list(filters) Product[]
    }
    class ProductCatalog {
        +int code
        +string description
    }
    class PolicyWording {
        +UUID id
        +string name
        +string version
        +Date effectiveDate
        +retrieve(id) PolicyWording
    }
    Product --> ProductCatalog : lineOfBusiness
    Product --> PolicyWording : wording
```

`policyWording` carries `version` and `effectiveDate` as well as a name. Together they are what let
you evidence which wording governed a policy sold on a given date, which is a routine compliance
ask and the reason the entity is more than a label.

## Endpoints

`productCatalog` is read-only. It is the standard's fixed list of 65 lines of business, not
something an implementation adds to.

| Endpoint | Scope | What it does |
| :--- | :--- | :--- |
| `POST /product` | admin | Register a product |
| `GET /product` | developer | List, filterable by `lineOfBusiness`, `productModel` and `currency` |
| `GET /product/{id}` | developer | Retrieve one product |
| `PUT /product/{id}` | admin | Replace a product |
| `GET /productCatalog` | developer | List the 65 lines of business |
| `POST /policyWording` | admin | Register a wording |
| `GET /policyWording` | developer | List wordings |
| `GET /policyWording/{id}` | developer | Retrieve one wording |
| `PUT /policyWording/{id}` | admin | Replace a wording |

## Primary flow: register a product

```mermaid
sequenceDiagram
    participant Client as Admin Client
    participant Gateway as API Gateway
    participant Service as Product Service
    participant Catalog as Product Catalog
    participant Wording as Policy Wording Store
    Client->>Gateway: POST /product {lineOfBusiness, productModel, currency, wordingId}
    Gateway->>Service: createProduct(payload)
    Service->>Catalog: validate lineOfBusiness in 1..65
    Service->>Wording: retrieve(wordingId)
    Wording-->>Service: wording
    Service->>Service: Persist record
    Service-->>Gateway: 201 Created {id}
    Gateway-->>Client: 201 Created
```

## Lifecycle

```mermaid
stateDiagram-v2
    [*] --> Registered : POST /product
    Registered --> Registered : PUT updates fields
    Registered --> [*] : record retained
```

A product exists or it does not. There is no draft state and no deprecated state, so nothing in the
standard says whether a product is open for new business today. That is an operational question
about one insurer's catalogue rather than a fact about the product, so carry it as an extension
field.

## Errors

```mermaid
flowchart TD
    A[POST /product] --> B{lineOfBusiness in productCatalog 1..65?}
    B -->|No| E1[400 - unknown productCatalog code]
    B -->|Yes| C{wordingId resolvable?}
    C -->|No| E2[404 - policy wording not found]
    C -->|Yes| D{Currency in OPIN currency enum?}
    D -->|No| E3[400 - unsupported currency]
    D -->|Yes| F{Idempotency-Key replay?}
    F -->|Yes| R1[200 OK - idempotent replay]
    F -->|No| G[Persist record]
    G --> H[201 Created]
```
