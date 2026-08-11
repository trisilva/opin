# Module 2: Products and Catalog

Resources, endpoints, primary flow, lifecycle and routing. The entities and fields are in [`data-model.md`](data-model.md). Conventions that apply to every module are in [`../conventions.md`](../conventions.md).

### Resource model

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

### Endpoints

`[added]`: OPIN publishes the Product and policyWording schemas but no endpoints. CRUD on Product and a read endpoint are added here for the productCatalog enum lookup.

- `POST /product` (admin)
- `GET /product` (developer), filter by lineOfBusiness, productModel, currency
- `GET /product/{id}` (developer)
- `PUT /product/{id}` (admin)
- `GET /productCatalog` (developer), list 65 OPIN productCatalog entries
- `GET /policyWording` (developer)
- `GET /policyWording/{id}` (developer)
- `POST /policyWording` (admin)
- `PUT /policyWording/{id}` (admin)

### Primary flow: Register a new product

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

### Lifecycle

```mermaid
stateDiagram-v2
    [*] --> Registered : POST /product
    Registered --> Registered : PUT updates fields
    Registered --> [*] : record retained
```

`[OPIN concern]`: OPIN does not declare a Product lifecycle (no draft, active, deprecated states). This standard keeps the lifecycle conservative. Product activation and deprecation are operational concerns to be encoded in implementer-specific extensions.

### Routing and error handling

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
