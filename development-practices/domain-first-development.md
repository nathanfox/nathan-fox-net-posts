# Domain-First Development: Building Robust Applications from the Core Out

## The Problem with Database-First Development

Over my 30+ years in software development, the most common approach I've seen in enterprise applications follows this database-first pattern:

1. Design the database schema
2. Build an ORM layer around the database
3. Create HTTP CRUD interfaces at the service layer
4. Spread business logic across stored procedures, service methods, validation layers, and the UI

This approach creates numerous problems:

- **Scattered Business Logic**: Your domain rules end up distributed across database constraints, stored procedures, service validators, and UI validation
- **Complex Debugging**: When something goes wrong, you need to trace through multiple layers to find the issue
- **Fragile Architecture**: Database schema changes ripple through every layer of your application
- **Difficult Testing**: Unit testing requires mocking databases, ORMs, and various infrastructure concerns
- **Refactoring Nightmares**: Simple changes require updates across multiple layers and technologies

## The Domain-First Alternative

After years of building enterprise applications, I've adopted a completely different approach: **develop your domain model first, completely independent of any storage or infrastructure concerns**.

I know this isn't a new idea—Clean Code and Domain-Driven Design have advocated for this approach for a long time. Yet despite its well-documented benefits, I've rarely seen it actually implemented in practice. Most teams still default to database-first development.

Here's the fundamental principle I follow: create a pure domain model assembly with zero dependencies on databases, HTTP, or any infrastructure. While I use F# for its powerful type system and functional paradigm, this approach works in any language. The domain model contains only:

- Type definitions representing your domain
- Pure functions implementing business logic
- Domain-specific validations and rules

## Building a Domain Model in F#

Let's walk through a practical example of building a domain model for an e-commerce system:

```fsharp
// Domain.fs - Pure domain model with zero infrastructure dependencies

namespace ECommerce.Domain

// Value types with built-in validation
type EmailAddress = EmailAddress of string
module EmailAddress =
    let create (email: string) =
        if String.IsNullOrWhiteSpace(email) then
            Error "Email cannot be empty"
        elif not (email.Contains("@")) then
            Error "Invalid email format"
        else
            Ok (EmailAddress email)
    
    let value (EmailAddress email) = email

type ProductId = ProductId of Guid
type CustomerId = CustomerId of Guid
type OrderId = OrderId of Guid

type Money = {
    Amount: decimal
    Currency: string
}

// Domain entities
type Product = {
    Id: ProductId
    Name: string
    Description: string
    Price: Money
    StockLevel: int
}

type Customer = {
    Id: CustomerId
    Name: string
    Email: EmailAddress
    ShippingAddress: Address
    BillingAddress: Address
}

and Address = {
    Street: string
    City: string
    PostalCode: string
    Country: string
}

type OrderLine = {
    Product: Product
    Quantity: int
    PriceAtOrder: Money
}

type OrderStatus =
    | Draft
    | Placed
    | Paid
    | Shipped
    | Delivered
    | Cancelled

type Order = {
    Id: OrderId
    Customer: Customer
    Lines: OrderLine list
    Status: OrderStatus
    PlacedAt: DateTime option
    ShippedAt: DateTime option
}

// Pure domain logic
module Order =
    let calculateTotal (order: Order) =
        order.Lines
        |> List.sumBy (fun line -> 
            line.PriceAtOrder.Amount * decimal line.Quantity)
        |> fun total -> { Amount = total; Currency = "USD" }
    
    let canBeCancelled (order: Order) =
        match order.Status with
        | Draft | Placed | Paid -> true
        | _ -> false
    
    let cancel (order: Order) =
        if canBeCancelled order then
            Ok { order with Status = Cancelled }
        else
            Error "Order cannot be cancelled in its current status"
    
    let place (order: Order) =
        match order.Status with
        | Draft when order.Lines.Length > 0 ->
            Ok { order with 
                    Status = Placed
                    PlacedAt = Some DateTime.UtcNow }
        | Draft ->
            Error "Cannot place an empty order"
        | _ ->
            Error "Order has already been placed"
    
    let ship (order: Order) (trackingNumber: string) =
        match order.Status with
        | Paid ->
            Ok { order with 
                    Status = Shipped
                    ShippedAt = Some DateTime.UtcNow }
        | _ ->
            Error "Order must be paid before shipping"

// Domain services
type InventoryService = {
    CheckAvailability: Product -> int -> Async<bool>
    ReserveStock: Product -> int -> Async<Result<unit, string>>
    ReleaseStock: Product -> int -> Async<unit>
}

type PricingService = {
    CalculateDiscount: Customer -> Order -> Money
    CalculateTax: Address -> Money -> Money
    CalculateShipping: Address -> Order -> Money
}

// Complex domain operations
module Checkout =
    type CheckoutError =
        | InsufficientStock of Product * available: int * requested: int
        | InvalidPayment of string
        | ShippingNotAvailable of Address
    
    let processCheckout 
        (inventory: InventoryService) 
        (pricing: PricingService)
        (order: Order) = async {
        
        // Check inventory for all items
        let! availabilityChecks = 
            order.Lines
            |> List.map (fun line -> async {
                let! available = inventory.CheckAvailability line.Product line.Quantity
                return line, available
            })
            |> Async.Parallel
        
        let unavailableItems = 
            availabilityChecks
            |> Array.filter (fun (_, available) -> not available)
            |> Array.map fst
        
        if unavailableItems.Length > 0 then
            let firstUnavailable = unavailableItems.[0]
            return Error (InsufficientStock (firstUnavailable.Product, 0, firstUnavailable.Quantity))
        else
            // Reserve inventory
            let! reservations =
                order.Lines
                |> List.map (fun line -> 
                    inventory.ReserveStock line.Product line.Quantity)
                |> Async.Sequential
            
            let failed = reservations |> Array.tryFind (Result.isError)
            match failed with
            | Some (Error msg) -> 
                return Error (InvalidPayment msg)
            | _ ->
                // Calculate final pricing
                let subtotal = Order.calculateTotal order
                let discount = pricing.CalculateDiscount order.Customer order
                let shipping = pricing.CalculateShipping order.Customer.ShippingAddress order
                let tax = pricing.CalculateTax order.Customer.BillingAddress subtotal
                
                let finalOrder = Order.place order
                
                return match finalOrder with
                       | Ok placed -> Ok placed
                       | Error msg -> Error (InvalidPayment msg)
    }
```

## Test-Driven Development with 100% Coverage

Because the domain model is pure F# with no dependencies, achieving 100% test coverage is straightforward. Using the e-commerce domain defined above:

```fsharp
// Domain.Tests.fs
open Expecto
open ECommerce.Domain
open System

// Test helpers
let createTestOrder lines =
    let email = EmailAddress "test@example.com"
    {
        Id = OrderId (Guid.NewGuid())
        Customer = {
            Id = CustomerId (Guid.NewGuid())
            Name = "Test Customer"
            Email = email
            ShippingAddress = { Street = "Test St"; City = "Test City"; PostalCode = "12345"; Country = "USA" }
            BillingAddress = { Street = "Test St"; City = "Test City"; PostalCode = "12345"; Country = "USA" }
        }
        Lines = lines
        Status = Draft
        PlacedAt = None
        ShippedAt = None
    }

let createTestOrderLine() =
    {
        Product = {
            Id = ProductId (Guid.NewGuid())
            Name = "Test Product"
            Description = "Test Description"
            Price = { Amount = 10.00m; Currency = "USD" }
            StockLevel = 100
        }
        Quantity = 1
        PriceAtOrder = { Amount = 10.00m; Currency = "USD" }
    }

let orderTests =
    testList "Order" [
        test "should calculate order total correctly" {
            // Arrange
            let product1 = {
                Id = ProductId (Guid.NewGuid())
                Name = "Widget"
                Description = "A useful widget"
                Price = { Amount = 10.00m; Currency = "USD" }
                StockLevel = 100
            }
            
            let product2 = {
                Id = ProductId (Guid.NewGuid())
                Name = "Gadget"
                Description = "A fancy gadget"
                Price = { Amount = 25.00m; Currency = "USD" }
                StockLevel = 50
            }
            
            let order = createTestOrder [
                { Product = product1; Quantity = 3; PriceAtOrder = product1.Price }
                { Product = product2; Quantity = 2; PriceAtOrder = product2.Price }
            ]
            
            // Act
            let total = Order.calculateTotal order
            
            // Assert
            Expect.equal total.Amount 80.00m "Total should be 3*10 + 2*25 = 80.00"
            Expect.equal total.Currency "USD" "Currency should be USD"
        }
        
        test "should not place empty order" {
            let order = { createTestOrder [] with Lines = [] }
            let result = Order.place order
            
            Expect.isError result "Empty order should not be placed"
            match result with
            | Error msg -> Expect.stringContains msg "empty" "Error should mention empty"
            | _ -> ()
        }
        
        test "should cancel order in Draft status" {
            let order = { createTestOrder [createTestOrderLine()] with Status = Draft }
            let result = Order.cancel order
            
            Expect.isOk result "Draft order should be cancellable"
            match result with
            | Ok cancelled -> Expect.equal cancelled.Status Cancelled "Status should be Cancelled"
            | _ -> ()
        }
    ]

let checkoutTests =
    testList "Checkout" [
        testAsync "should fail checkout with insufficient stock" {
            // Arrange
            let inventory = {
                CheckAvailability = fun _ _ -> async { return false }
                ReserveStock = fun _ _ -> async { return Error "No stock" }
                ReleaseStock = fun _ _ -> async { return () }
            }
            
            let pricing = {
                CalculateDiscount = fun _ _ -> { Amount = 0m; Currency = "USD" }
                CalculateTax = fun _ total -> { Amount = total.Amount * 0.1m; Currency = total.Currency }
                CalculateShipping = fun _ _ -> { Amount = 5.00m; Currency = "USD" }
            }
            
            let order = createTestOrder [createTestOrderLine()]
            
            // Act
            let! result = Checkout.processCheckout inventory pricing order
            
            // Assert
            Expect.isError result "Checkout should fail"
            match result with
            | Error (Checkout.InsufficientStock _) -> ()
            | _ -> failtest "Expected InsufficientStock error"
        }
    ]

// Run all tests
[<EntryPoint>]
let main argv =
    runTestsWithCLIArgs [] argv (
        testList "All Domain Tests" [
            orderTests
            checkoutTests
        ]
    )
```

## Sharing the Domain Model Across All Layers

Once you have a solid, tested domain model, you can use it everywhere:

### Service Layer with Fable.Remoting and Giraffe

Fable.Remoting.Server provides seamless integration with Giraffe for building type-safe APIs:

```fsharp
// OrderApi.fs
open Fable.Remoting.Server
open Fable.Remoting.Giraffe
open Giraffe
open Microsoft.AspNetCore.Http

// Define your API interface (shared between client and server)
type IOrderApi = {
    PlaceOrder: Order -> Async<Result<Order, CheckoutError>>
    GetOrder: OrderId -> Async<Order option>
    CancelOrder: OrderId -> Async<Result<Order, string>>
    GetOrderHistory: CustomerId -> Async<Order list>
}

// Create the API implementation
let createOrderApi (repos: IRepositories) : IOrderApi = {
    PlaceOrder = fun order ->
        Checkout.processCheckout repos.Inventory repos.Pricing order
    
    GetOrder = repos.Orders.GetById
    
    CancelOrder = fun orderId -> async {
        let! order = repos.Orders.GetById orderId
        match order with
        | Some o ->
            match Order.cancel o with
            | Ok cancelled ->
                do! repos.Orders.Save cancelled
                return Ok cancelled
            | Error msg ->
                return Error msg
        | None ->
            return Error "Order not found"
    }
    
    GetOrderHistory = repos.Orders.GetByCustomer
}

// Error handling
type ApiError = { ErrorMsg: string }

let errorHandler (ex: exn) (routeInfo: RouteInfo<HttpContext>) =
    let errorMsg = sprintf "Error at %s on method %s: %A" 
                          routeInfo.path routeInfo.methodName ex
    printfn "%s" errorMsg  // Log the error
    let customError = { ErrorMsg = errorMsg }
    Propagate customError  // Return error to client

// Create HTTP handler with Fable.Remoting
let orderApiHandler: HttpHandler =
    Remoting.createApi()
    |> Remoting.withErrorHandler errorHandler
    |> Remoting.fromContext (fun (ctx: HttpContext) ->
        // Get dependencies from DI container
        let repos = ctx.GetService<IRepositories>()
        createOrderApi repos)
    |> Remoting.buildHttpHandler

// Multiple APIs with different authentication requirements
let productApiHandler: HttpHandler =
    Remoting.createApi()
    |> Remoting.withErrorHandler errorHandler
    |> Remoting.fromContext (fun (ctx: HttpContext) ->
        let repos = ctx.GetService<IRepositories>()
        let cache = ctx.GetService<IMemoryCache>()
        createProductApi repos cache)
    |> Remoting.buildHttpHandler

let adminApiHandler: HttpHandler =
    Remoting.createApi()
    |> Remoting.withErrorHandler errorHandler
    |> Remoting.fromContext (fun (ctx: HttpContext) ->
        let repos = ctx.GetService<IRepositories>()
        let logger = ctx.GetService<ILogger<IAdminApi>>()
        createAdminApi repos logger)
    |> Remoting.buildHttpHandler

// Program.fs - Wire up with Giraffe
open Microsoft.AspNetCore.Builder
open Microsoft.Extensions.DependencyInjection

let requiresAuth = requiresAuthentication (RequestErrors.UNAUTHORIZED "Auth" "API" "Login required")
let requiresAdmin = requiresRole "Admin" (RequestErrors.FORBIDDEN "Admin access required")

let webApp =
    choose [
        // Public API endpoints
        subRoute "/api/public" (
            choose [
                productApiHandler
            ])
        
        // Authenticated API endpoints
        requiresAuth >=> subRoute "/api" (
            choose [
                orderApiHandler
                
                // Admin-only endpoints
                subRoute "/admin" (
                    requiresAdmin >=> adminApiHandler
                )
            ])
        
        // Health checks
        route "/health" >=> text "Healthy"
    ]

let configureServices (services: IServiceCollection) =
    // Register repositories
    services.AddSingleton<IRepositories>(fun sp ->
        let config = sp.GetRequiredService<IConfiguration>()
        let connString = config.GetConnectionString("Default")
        {
            Orders = createOrderRepository connString
            Products = createProductRepository connString
            Customers = createCustomerRepository connString
        }) |> ignore
    
    // Add authentication
    services.AddAuthentication() |> ignore
    services.AddAuthorization() |> ignore
    
    // Add Giraffe
    services.AddGiraffe() |> ignore

let configureApp (app: IApplicationBuilder) =
    app
        .UseAuthentication()
        .UseAuthorization()
        .UseGiraffe webApp

[<EntryPoint>]
let main _ =
    WebHostBuilder()
        .UseKestrel()
        .Configure(configureApp)
        .ConfigureServices(configureServices)
        .Build()
        .Run()
    0
```

### Generate TypeScript for Browser Clients

Using Fable, you can generate JavaScript/TypeScript from your F# domain model. Here's how I structure these projects in production:

#### Project Structure

```
ecommerce_domain/
├── ecommerce_domain.fsproj           # Main .NET project
├── src/
│   ├── ecommerce_domain.fable.fsproj # Fable-specific project
│   ├── Domain.fs                     # Core domain types
│   ├── Order.fs                      # Order domain logic
│   ├── Customer.fs                   # Customer domain logic
│   ├── Product.fs                    # Product domain logic
│   ├── Inventory/                    # Subdomain modules
│   │   ├── Stock.fs
│   │   ├── Warehouse.fs
│   │   └── Allocation.fs
│   └── index.ts                      # TypeScript entry point
├── package.json                      # NPM package definition
└── tsconfig.json                     # TypeScript configuration
```

#### Package Configuration

```json
// package.json
{
  "name": "@mycompany/ecommerce-domain",
  "version": "1.0.0",
  "private": false,
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "scripts": {
    "build": "tsc",
    "compile-fable": "dotnet fable src/ecommerce_domain.fable.fsproj --outDir src --extension .js",
    "install-fable": "dotnet tool install fable --local"
  },
  "devDependencies": {
    "typescript": "^5.4.3"
  },
  "dependencies": {
    "@mycompany/shared-domain": "^1.0.0"  // Shared domain if needed
  }
}
```

```json
// tsconfig.json
{
    "compilerOptions": {
        "target": "ES2020",
        "module": "ESNext",
        "lib": ["ES2020", "DOM"],
        "moduleResolution": "Bundler",
        "outDir": "dist",
        "declaration": true,
        "allowJs": true
    },
    "include": ["src/*.ts", "src/**/*.js"]
}
```

#### Fable Project File

The key is having a separate `.fable.fsproj` file for Fable compilation:

```xml
<!-- src/ecommerce_domain.fable.fsproj -->
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFrameworks>net6.0;net8.0</TargetFrameworks>
    <GenerateDocumentationFile>true</GenerateDocumentationFile>
    <PackageTags>fable;fable-binding;fable-javascript</PackageTags>
    <DefineConstants>FABLE_COMPILER</DefineConstants>
  </PropertyGroup>
  <ItemGroup>
    <Content Include="*.fsproj; **\*.fs; **\*.fsi" PackagePath="fable" />
  </ItemGroup>
  <ItemGroup>
    <!-- Include your domain files -->
    <Compile Include="Domain.fs" />
    <Compile Include="Customer.fs" />
    <Compile Include="Product.fs" />
    <Compile Include="Order.fs" />
    <Compile Include="Inventory\Stock.fs" />
    <Compile Include="Inventory\Warehouse.fs" />
    <Compile Include="Inventory\Allocation.fs" />
  </ItemGroup>
  <ItemGroup>
    <PackageReference Include="Fable.Core" Version="4.3.0" />
    <PackageReference Include="FSharp.Json" Version="0.4.1" />
    <!-- Reference other Fable-compatible domain packages -->
    <PackageReference Include="shared_domain.fable" Version="1.0.0" />
  </ItemGroup>
</Project>
```

#### TypeScript Entry Point

Create an index.ts that exports all the generated JavaScript modules:

```typescript
// src/index.ts
export * from "./Domain.fs";
export * from "./Customer.fs";
export * from "./Product.fs";
export * from "./Order.fs";
export * from "./Inventory/Stock.fs";
export * from "./Inventory/Warehouse.fs";
export * from "./Inventory/Allocation.fs";
```

#### Build Process

```bash
# Install Fable as a local dotnet tool (first time only)
npm run install-fable

# Create a .config/dotnet-tools.json if it doesn't exist
dotnet new tool-manifest

# Install Fable locally
dotnet tool install fable

# Compile F# to JavaScript using Fable
npm run compile-fable

# Compile TypeScript declarations
npm run build

# Publish to NPM registry (private or public)
npm publish
```

#### Using in TypeScript/React Applications

Now your TypeScript frontend can use the exact same domain types and logic:

```typescript
import { 
    Order, 
    OrderStatus, 
    Customer,
    Product,
    calculateOrderTotal,
    validateOrder,
    canCancelOrder 
} from '@mycompany/ecommerce-domain';

// Use F# domain types directly in TypeScript
const order: Order = {
    id: orderId,
    customer: currentCustomer,
    lines: cartItems,
    status: OrderStatus.Draft,
    placedAt: null,
    shippedAt: null
};

// Use F# validation functions
const validation = validateOrder(order);
if (validation.isValid) {
    const total = calculateOrderTotal(order);
    
    // Use F# business logic
    if (canCancelOrder(order)) {
        setCancelButtonEnabled(true);
    }
    
    await api.placeOrder(order);
}
```

#### Benefits of This Approach

1. **Single Source of Truth**: Domain logic defined once in F#
2. **Type Safety**: Full TypeScript type definitions generated automatically
3. **Business Logic Sharing**: Complex validations and calculations available in both frontend and backend
4. **Version Consistency**: NPM versioning ensures frontend and backend stay in sync
5. **IDE Support**: Full IntelliSense in VS Code, WebStorm, etc.

This approach has proven invaluable in production systems where domain consistency between frontend and backend is critical.

### Elmish Applications

For Elmish applications, reference the domain assembly directly:

```fsharp
// App.fs
open ECommerce.Domain

type Model = {
    CurrentOrder: Order option
    Products: Product list
    Customer: Customer option
}

type Msg =
    | LoadOrder of OrderId
    | OrderLoaded of Order
    | AddToCart of Product
    | RemoveFromCart of Product
    | PlaceOrder
    | OrderPlaced of Result<Order, CheckoutError>
```

## Repository Layer: Persisting the Domain

Only after the domain model is complete and tested do we build the repository layer:

```fsharp
// Repositories.fs
type OrderRepository = {
    GetById: OrderId -> Async<Order option>
    GetByCustomer: CustomerId -> Async<Order list>
    Save: Order -> Async<unit>
    Delete: OrderId -> Async<Result<unit, string>>
}

// SQL implementation
let createSqlOrderRepository (connectionString: string) =
    let mapOrderFromDb (row: OrderRow) : Order = 
        // Map from database representation to domain model
        {
            Id = OrderId row.Id
            Customer = mapCustomer row
            Lines = mapOrderLines row.Lines
            Status = mapStatus row.Status
            PlacedAt = row.PlacedAt
            ShippedAt = row.ShippedAt
        }
    
    let mapOrderToDb (order: Order) : OrderRow =
        // Map from domain model to database representation
        {
            Id = order.Id |> fun (OrderId id) -> id
            CustomerId = order.Customer.Id |> fun (CustomerId id) -> id
            Status = order.Status.ToString()
            PlacedAt = order.PlacedAt
            ShippedAt = order.ShippedAt
        }
    
    {
        GetById = fun orderId -> async {
            use conn = new SqlConnection(connectionString)
            let! row = // Query database
            return row |> Option.map mapOrderFromDb
        }
        
        Save = fun order -> async {
            use conn = new SqlConnection(connectionString)
            let row = mapOrderToDb order
            // Save to database
        }
        
        // Other operations...
    }
```

## Benefits of This Approach

This domain-first approach has proven itself in multiple enterprise-grade production systems:

### 1. **Testability**
- Pure domain logic with no infrastructure dependencies
- Easy to achieve 100% test coverage
- Fast unit tests that don't require mocking

### 2. **Maintainability**
- Business logic concentrated in one place
- Clear separation of concerns
- Easy to understand and reason about

### 3. **Flexibility**
- Can change storage mechanisms without touching domain logic
- Can add new APIs or interfaces without modifying the core
- Database schema can evolve independently

### 4. **Code Reuse**
- Same domain model used across all layers
- TypeScript generation for browser clients
- Shared validation and business rules

### 5. **Type Safety**
- F#'s type system catches errors at compile time
- Domain invariants enforced by types
- Impossible states are unrepresentable

### 6. **Developer Experience**
- Fast feedback loop during development
- Easy onboarding for new team members
- Refactoring is safe and straightforward

## Real-World Production Success

This approach isn't theoretical—I've used it successfully in multiple production systems for vendor-facing business operations, including:

- **Vendor Product Management Systems**
- **Financial Query and Reporting Platforms**
- **Special Product Promotion Booking Applications**

In each case, the domain-first approach resulted in:
- Fewer production bugs
- Faster feature development
- Easier maintenance and updates
- Consistent business logic across all interfaces

## Conclusion

By developing your domain model first and keeping it independent of infrastructure, you create a solid foundation for your entire application. This approach, combined with F#'s powerful type system and functional programming paradigm, results in systems that are robust, maintainable, and a joy to work with.

The key is to resist the temptation to think about databases, APIs, or UI concerns when designing your domain. Focus purely on the business problem you're solving, encode the rules in types and functions, and only then consider how to persist and expose your domain to the outside world.

This investment in a clean domain model pays dividends throughout the lifetime of your application, making it one of the most important architectural decisions you can make.