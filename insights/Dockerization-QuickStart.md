## .NET Project Analysis and Dockerization

## Project Structure

```plaintext
MyDotNetApp
│
├── Controllers
│   ├── HomeController.cs
│   └── ProductController.cs
├── Models
│   ├── ErrorViewModel.cs
│   ├── IMongoDBSettings.cs
│   ├── MongoDBSettings.cs
│   └── Product.cs
├── Services
│   └── ProductService.cs
├── Views
│   ├── Home
│   │   ├── Index.cshtml
│   │   └── Privacy.cshtml
│   ├── Product
│   │   ├── Create.cshtml
│   │   ├── Delete.cshtml
│   │   ├── Details.cshtml
│   │   ├── Edit.cshtml
│   │   └── Index.cshtml
│   ├── Shared
│   │   ├── Error.cshtml
│   │   ├── _HomeLayout.cshtml
│   │   ├── _Layout.cshtml
│   │   ├── _Layout.cshtml.css
│   │   └── _ValidationScriptsPartial.cshtml
│   ├── _ViewImports.cshtml
│   └── _ViewStart.cshtml
├── wwwroot
│   ├── css
│   │   └── site.css
│   ├── favicon.ico
│   └── js
│       └── site.js
├── appsettings.Development.json
├── appsettings.json
├── Dockerfile
├── docker-compose.yml
├── DotNetMongoCRUDApp.csproj
├── Program.cs
├── Startup.cs
└── README.md
```


---
## Multi-Stage Dockerfile

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

## Docker Compose File

```yml
version: '3.8'

services:
  mongodb:
    image: mongo:7.0
    container_name: mongodb
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: ibtisam
    ports:
      - "27017:27017"
    volumes:
      - mongo-data:/data/db
    healthcheck:
      test: "echo 'db.stats()' | mongosh --host localhost:27017 --authenticationDatabase admin -u root -p example || exit 1"
      interval: 10s
      retries: 5
      start_period: 30s
      timeout: 10s

  webapp:
    build: .
    container_name: dotnet-mongo-crud
    ports:
      - "8080:5035"  # Maps port 5035 in the container to port 8080 on the host
    depends_on:
      mongodb:
        condition: service_healthy
    environment:
      ASPNETCORE_ENVIRONMENT: Development
      MongoDB__ConnectionString: "mongodb://root:ibtisam@mongodb:27017"
      MongoDB__DatabaseName: "ProductDB"
    volumes:
      - ./appsettings.json:/app/appsettings.json

volumes:
  mongo-data:
```

## Commands to Run

1. **Build the Docker images:**

   ```sh
   docker-compose build
   ```

2. **Start the containers:**

   ```sh
   docker-compose up
   ```

3. **Access the application:**

   Open your browser and navigate to `http://localhost:8080`.

## Application Architecture

### 2-Tier Architecture

1. **Presentation Layer (Controllers):**
   - Handles HTTP requests and returns responses.
   - Example: [`Controllers/ProductController.cs`](Controllers/ProductController.cs )

2. **Business Logic and Data Layer (Services):**
   - Handles core application logic and interacts with the database.
   - Example: [`Services/ProductService.cs`](Services/ProductService.cs )


## Reference

You can find in-depth information [here](Dockerization.md).