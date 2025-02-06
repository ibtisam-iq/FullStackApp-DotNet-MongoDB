# .NET Project Architecture Analysis

> This is quite lenghty file, you can read its short version [here](dockerization.md).

## Table of Contents
- [Understanding the Architecture](#understanding-the-architecture)
  - [Presentation Layer vs. Business Logic Layer](#presentation-layer-vs-business-logic-layer)
  - [Data Layer](#data-layer)
  - [Why It’s 2-Tier](#why-its-2-tier)
  - [To Make It a True 3-Tier Architecture](#to-make-it-a-true-3-tier-architecture)
  - [Conclusion](#conclusion)
- [3-Tier Architecture Structure in .NET](#3-tier-architecture-structure-in-net)
  - [Suggested 3-Tier Architecture Structure](#suggested-3-tier-architecture-structure)
  - [Breakdown of Layers](#breakdown-of-layers)
    - [Presentation Layer (API or UI) - `/src/MyDotNetApp.Api`](#presentation-layer-api-or-ui---srcmydotnetappapi)
    - [Business Logic Layer (BLL) - `/src/MyDotNetApp.BLL`](#business-logic-layer-bll---srcmydotnetappbll)
    - [Data Access Layer (DAL) - `/src/MyDotNetApp.DAL`](#data-access-layer-dal---srcmydotnetappdal)
  - [Flow Example](#flow-example)
  - [Additional Points for Clarity](#additional-points-for-clarity)
- [Understanding the Dockerfile for .NET + MongoDB Project](#understanding-the-dockerfile-for-net--mongodb-project)
  - [Dockerfile Breakdown](#dockerfile-breakdown)
  - [Step-by-Step Explanation](#step-by-step-explanation)
- [Difference Between `dotnet build`, `dotnet run`, and `dotnet publish`](#difference-between-dotnet-build-dotnet-run-and-dotnet-publish)
  - [What Happens When You Run Locally?](#what-happens-when-you-run-locally)
  - [What Happens in the Dockerfile?](#what-happens-in-the-dockerfile)
  - [Key Differences & When to Use What?](#key-differences--when-to-use-what)
  - [Why Dockerfile Uses `dotnet publish` Instead of `dotnet run`?](#why-dockerfile-uses-dotnet-publish-instead-of-dotnet-run)
  - [When Would You Use `dotnet run` Inside a Container?](#when-would-you-use-dotnet-run-inside-a-container)
  - [Conclusion](#conclusion)
  - [Why dotnet build + dotnet run ≠ dotnet publish?](#why-dotnet-build--dotnet-run--dotnet-publish)


## Understanding the Architecture

In this .NET project, there is some distinction between layers, but it still leans more toward a **2-Tier architecture**, and here's why:

### Presentation Layer vs. Business Logic Layer
- Controllers (Presentation Layer) directly call the Services (Business Logic Layer).
- The `ProductService.cs` interacts with the data layer, but this still falls under a service layer and is not decoupled into a separate Data Access Layer (DAL) or Repository Layer.

### Data Layer
- The MongoDB interaction is directly handled in the service (`ProductService.cs`). Ideally, in a true **3-Tier architecture**, there should be a separate **Data Access Layer** or **Repository Layer** that interacts with the database, providing an abstraction for the services. This allows for better scalability and flexibility in case the data source needs to change.

## Why It’s 2-Tier
- The **Service Layer** is directly responsible for accessing the database (MongoDB in this case), making it a **2-Tier architecture**.
- There's no clear separation of the data interaction and business logic, as the service is doing both — handling business logic and interacting with the data source.

## To Make It a True 3-Tier Architecture
You could refactor this application by introducing a **Data Access Layer (DAL)** or **Repository Layer** that abstracts the database interaction away from the service layer. Here's what that might look like:

- **Presentation Layer (Controllers):** Handles HTTP requests and returns responses.
- **Business Logic Layer (Services):** Handles core application logic, and relies on the DAL for data operations.
- **Data Access Layer (DAL/Repository):** Responsible for data operations, such as interacting with MongoDB (insert, update, delete, query), and abstracting these operations away from the service layer.

By introducing a **Repository or DAL pattern**, the service layer will only handle business logic, and the data access will be abstracted, providing better separation and making the architecture **true 3-Tier**.

## Conclusion
You're correct that this project is more of a **2-Tier architecture**, and with slight modifications, it can be refactored into a **3-Tier architecture**.

---

## 3-Tier Architecture Structure in .NET

In a typical **3-Tier Architecture** for a .NET project, you would organize the project into three distinct layers to achieve separation of concerns: **Presentation Layer (UI), Business Logic Layer (BLL), and Data Access Layer (DAL)**. Here's how you might structure it, with a frontend folder for the UI and backend code separated into distinct layers.

### Suggested 3-Tier Architecture Structure
```
MyDotNetApp
│
├── src                        # Contains backend code (Business Logic, Data Access)
│   ├── MyDotNetApp.Api         # API layer (Presentation Layer - Web)
│   │   ├── Controllers         # Contains API controllers (UI Layer)
│   │   ├── Models              # Request and Response Models (DTOs)
│   │   ├── Views               # Razor views or response views (if applicable)
│   │   └── appsettings.json    # Configuration file for API
│   │
│   ├── MyDotNetApp.BLL         # Business Logic Layer
│   │   ├── Interfaces          # Interfaces for services/repositories
│   │   ├── Services            # Core business logic (manipulating data)
│   │   └── Helpers             # Helper classes for business logic
│   │
│   └── MyDotNetApp.DAL         # Data Access Layer
│       ├── Interfaces          # Interfaces for data access
│       ├── Repositories        # MongoDB or SQL interactions
│       └── Models              # Database models (Entity Framework models or MongoDB classes)
│
├── public                      # Frontend code (client-side)
│   ├── wwwroot                 # Static files like CSS, JS, images
│   ├── index.html              # Main entry HTML (for static front-end apps)
│   └── assets                  # Any public assets for frontend
│
└── README.md                   # Project documentation
```

### Breakdown of Layers

#### **Presentation Layer (API or UI) - `/src/MyDotNetApp.Api`**
- **Controllers:** API controllers that handle HTTP requests and return responses.
- **Models:** DTOs (Data Transfer Objects) or ViewModels, used for transferring data between the frontend and backend.

#### **Business Logic Layer (BLL) - `/src/MyDotNetApp.BLL`**
- **Services:** Contains the business logic that manipulates data.
- **Interfaces:** Defines service interfaces, allowing for loose coupling and easier testing.

#### **Data Access Layer (DAL) - `/src/MyDotNetApp.DAL`**
- **Repositories:** Contains the code responsible for interacting with the database.
- **Models:** Contains database models, which define the structure of data stored in the database.

### Flow Example

Let’s consider a scenario where the user wants to **add a product** to the system. Here's how the process might flow in a **3-Tier Architecture**:

1. **Presentation Layer:**
   - The `ProductController` (UI Layer) receives a POST request from the client.
   - It uses `ProductService` from the BLL to process the business logic for adding the product.

2. **Business Logic Layer:**
   - `ProductService` performs any necessary validation, calculations, or transformations on the data.
   - It then calls the DAL (via a `ProductRepository`) to persist the product in the database.

3. **Data Access Layer:**
   - The `ProductRepository` interacts with MongoDB (or a relational database) to add the new product.
   - Once the product is saved, the data is returned to the BLL, which then passes it back to the Presentation Layer to be returned as an API response.

### Additional Points for Clarity

- **Database Independence:** The DAL can be swapped out independently of the business logic. For example, you can switch from **MongoDB to SQL Server**, and only the DAL layer needs to be modified.
- **Decoupling:** The BLL interacts only with **interfaces, not concrete implementations**. This decouples the business logic from the underlying database implementation and allows you to write cleaner, more maintainable code.
- **Testing:** Each layer is independent, making it easier to write **unit tests** for the BLL (using mock repositories) or the DAL.

By following this structure, you can ensure a **clean separation of concerns**, making the application **scalable, maintainable, and flexible**.

---
## Understanding the Dockerfile for .NET + MongoDB Project

### Dockerfile Breakdown

```dockerfile
# Use the official .NET image as a build environment
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build-env
WORKDIR /app

# Copy the csproj and restore as distinct layers
COPY *.csproj ./
RUN dotnet restore

# Copy the rest of the application and build it
COPY . ./
RUN dotnet publish -c Release -o out

# Build runtime image
FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY --from=build-env /app/out .

# Expose port 5035 for your application
EXPOSE 5035

# Set the entry point
ENTRYPOINT ["dotnet", "DotNetMongoCRUDApp.dll"]
```

### **Step-by-Step Explanation**

1. **Building the Application**
   - `FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build-env` → Uses .NET SDK for building the app.
   - `WORKDIR /app` → Sets working directory inside the container.
   - `COPY *.csproj ./` → Copies only the project file first (improves caching efficiency).
   - `RUN dotnet restore` → Restores NuGet dependencies.
   - `COPY . ./` → Copies the rest of the source code.
   - `RUN dotnet publish -c Release -o out` → Builds the app in **Release mode** and outputs to `out/`.

2. **Creating the Runtime Container**
   - `FROM mcr.microsoft.com/dotnet/aspnet:8.0` → Uses a **lighter .NET Runtime image** (not SDK).
   - `WORKDIR /app` → Again sets working directory.
   - `COPY --from=build-env /app/out .` → Copies the published app from the previous stage.
   - `EXPOSE 5035` → Informs Docker that the app listens on port **5035**.
   - `ENTRYPOINT ["dotnet", "DotNetMongoCRUDApp.dll"]` → Runs the application inside the container.

---

## **Difference Between `dotnet build`, `dotnet run`, and `dotnet publish`**

| Command | Purpose | When to Use? |
| --- | --- | --- |
| **`dotnet build`** | Compiles the code and generates intermediate binaries (`.dll` files). | During development for testing changes. |
| **`dotnet run`** | Builds (if not already built) and **runs the app directly from source**. | During local development (fast iteration). |
| **`dotnet publish`** | Compiles, **optimizes for deployment**, and produces a **self-contained output**. | When deploying (Docker, production). |

---

### **What Happens When You Run Locally?**

```sh
dotnet build; dotnet run
```

1. `dotnet build`
   - Compiles the project into **intermediate binaries** (`.dll` files) inside the `bin/Debug/net8.0/` folder.
   - **Not optimized** for production (includes debugging symbols, etc.).
   - The output is **not self-contained**, meaning it requires the .NET SDK to run.

2. `dotnet run`
   - **Builds (if necessary)** and then runs the app **directly from the source code**.
   - This command is useful for **fast testing** but **shouldn’t be used in production** because:
     - It’s slower (does additional checks).
     - Runs in Debug mode by default.
     - Doesn’t produce an optimized, standalone package.

---

### **What Happens in the Dockerfile?**

Instead of `dotnet build` and `dotnet run`, the Dockerfile uses:

```dockerfile
RUN dotnet publish -c Release -o out
ENTRYPOINT ["dotnet", "DotNetMongoCRUDApp.dll"]
```

Here's how it's different:

- **`dotnet publish -c Release -o out`**
  - Produces a **fully optimized build** for production.
  - Removes unnecessary debugging files.
  - Outputs everything into a single directory (`out`).
  - The `.dll` files here **don’t need the full .NET SDK** to run—only the runtime.

- **`ENTRYPOINT ["dotnet", "DotNetMongoCRUDApp.dll"]`**
  - Instead of using `dotnet run`, the container runs:
    
    ```sh
    dotnet DotNetMongoCRUDApp.dll
    ```
  
  - This executes the **pre-built** application using the .NET runtime.
  - Faster and more reliable compared to `dotnet run` inside a container.

---

### **Key Differences & When to Use What?**

| Feature | `dotnet build; dotnet run` (Local) | `dotnet publish` (Docker) |
| --- | --- | --- |
| **Speed** | Slower (builds every time) | Faster (pre-built binary) |
| **Optimization** | Debug mode (default) | Production-ready (Release mode) |
| **Dependencies** | Requires .NET SDK | Needs only .NET Runtime |
| **Output Location** | `bin/Debug/net8.0/` | `out/` folder (single package) |
| **Use Case** | Development & testing | Deployment (Docker, servers) |

---

### **Why Dockerfile Uses `dotnet publish` Instead of `dotnet run`?**

1. **Performance** – Pre-built binaries run faster than source-based execution.
2. **Smaller Image** – The final container **doesn’t include the full SDK**, only what’s needed to run the app.
3. **Production Readiness** – Docker containers are meant to be **immutable**. Using `dotnet run` inside a container would recompile every time, which is inefficient.

---

### **When Would You Use `dotnet run` Inside a Container?**

- Rarely! But if you were **developing inside a container** and wanted to test your app without publishing, you could run:
  
  ```sh
  docker run -it --rm -v $(pwd):/app -w /app mcr.microsoft.com/dotnet/sdk:8.0 dotnet run
  ```
  
- However, for deployment, **always use `dotnet publish`**.

---

### **💡 Conclusion**

- **`dotnet build; dotnet run`** is great for local development.
- **Dockerfile uses `dotnet publish`** to optimize performance for containerized deployment.
- **In production, you should avoid `dotnet run`** and always run a published `.dll` file.

---

## Why dotnet build + dotnet run ≠ dotnet publish?
- dotnet build compiles your code into intermediate binaries, but it doesn’t produce an optimized, ready-to-deploy package.
- dotnet run simply executes the compiled code, but it still depends on the full .NET SDK.
- dotnet publish optimizes everything, strips unnecessary files, and prepares a production-ready package.

🔹 Example
**1️⃣ Running in Development (dotnet build + dotnet run)**

```bash
dotnet build
dotnet run
```
- **What happens?**
 - Compiles the project into bin/Debug/net8.0/
 - Runs the app directly from source
 - Slower startup, not optimized

**2️⃣ Preparing for Production (dotnet publish)**
```bash
dotnet publish -c Release -o out
dotnet out/DotNetMongoCRUDApp.dll
```
- **What happens?**
 - Optimized build created in the out/ folder
 - Removes extra debugging symbols
 - Faster startup, smaller binaries
 - No need for full .NET SDK (only runtime required)

🔹 **Conclusion**

- ❌ dotnet build + dotnet run ≠ dotnet publish
- ✅ If you want a production-ready build, use dotnet publish instead of dotnet run.

In your Dockerfile, dotnet publish is used because it ensures:

- Faster application startup
- Smaller container image
- No unnecessary files from development