# 3-Tier .NET & MongoDB Application

This project is a 3-tier web application built with .NET and MongoDB. The application consists of a presentation layer, a business logic layer, and a data access layer. MongoDB is used as the database to store and manage application data.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Installation](#installation)
  - [1. Installing .NET SDK and Runtime](#1-installing-net-sdk-and-runtime)
  - [2. Installing MongoDB](#2-installing-mongodb)
  - [3. Setting Up MongoDB](#3-setting-up-mongodb)
- [Running the Application](#running-the-application)
- [Using MongoDB Shell](#using-mongodb-shell)
- [License](#license)

## Prerequisites

Before setting up the project, ensure you have the following installed on your machine:

- Ubuntu (or another compatible Linux distribution)
- [.NET SDK 8.0](https://dotnet.microsoft.com/download/dotnet/8.0) 
- [MongoDB 7.0](https://www.mongodb.com/try/download/community) 

## Installation

### 1. Installing .NET SDK and Runtime

To install the .NET SDK and Runtime, execute the following commands in your terminal:

1. **Install .NET SDK 8.0:**

   ```bash
   sudo apt-get update && \
   sudo apt-get install -y dotnet-sdk-8.0
   ```

2. **Install .NET Runtime 8.0:**

   ```bash
   sudo apt-get update && \
   sudo apt-get install -y aspnetcore-runtime-8.0
   ```

### 2. Installing MongoDB

To install MongoDB, please follow the guide [here](https://github.com/ibtisamops/nectar/blob/main/mongodb/MongoDB.md.)


### 3. Setting Up MongoDB

1. **Install MongoDB Shell:**

   Follow the installation guide at [MongoDB Shell Installation](https://www.mongodb.com/docs/mongodb-shell/install/).

2. **Access MongoDB Terminal:**

   To interact with your MongoDB instance, open the MongoDB shell using:

   ```bash
   mongosh
   ```

3. **Manipulate Databases and Collections:**

   - Show databases:

     ```bash
     show dbs;
     ```

   - Use a specific database:

     ```bash
     use db_name;
     ```

   - Show collections in the database:

     ```bash
     show collections;
     ```

   - Query the `Products` collection:

     ```bash
     db.Products.find().pretty();
     ```

## Running the Application

To run the .NET application:

1. **Navigate to the root directory where `Program.cs` is located.**

2. **Build the application:**

   ```bash
   dotnet build
   ```

3. **Run the application:**

   ```bash
   dotnet run
   ```

   The application will start, and you can access it in your web browser.

## Using MongoDB Shell

To manipulate your MongoDB database using MongoDB Shell:

1. **Start the shell:**

   ```bash
   mongosh
   ```

2. **Example commands:**

   - **List all databases:**

     ```bash
     show dbs;
     ```

   - **Switch to a specific database:**

     ```bash
     use db_name;
     ```

   - **Show collections in the current database:**

     ```bash
     show collections;
     ```

   - **Find all documents in a collection:**

     ```bash
     db.Products.find().pretty();
     ```

## Project Structure

Please refer to `consoleOutput.txt` for more details. 😊

---

## MongoDB Connection Troubleshooting

### **1. Use `127.0.0.1` Instead of `localhost`**
- In some Linux setups, **`localhost`** might resolve to `::1` (IPv6), while MongoDB is bound to `127.0.0.1` (IPv4).
- Try updating the connection string:
  
  ```json
  "ConnectionString": "mongodb://127.0.0.1:27017"
  ```

### **2. Database Doesn't Exist Yet**
- MongoDB **does not create databases until you insert data**. Ensure `ProductDB` exists by running:
  
  ```bash
  mongosh
  ```
  
  Then, check databases:
  
  ```js
  show dbs
  ```
  
  If `ProductDB` is missing, manually create it:
  
  ```js
  use ProductDB 
  db.Products.insertOne({ "test": "testValue" })
  ```

#### **3. MongoDB Server Not Running**
- Ensure the MongoDB server is running:

  ```bash
  sudo netstat -tulnp | grep 27017
  telnet 127.0.0.1 27017
  ```

### **4. MongoDB Might Not Be Running Properly**
- If MongoDB is listening, but `mongosh` is still failing, Check MongoDB logs for errors:
  
  ```bash
  sudo cat /var/log/mongodb/mongod.log | tail -50
  ```

### **5. Port Blocked by Firewall**
- Ensure **port 27017** is open:
  
  ```bash
  sudo ufw allow 27017/tcp 
  sudo ufw status
  ```

### **6. Connection String Authentication Issues**
- If MongoDB **requires authentication**, you need to add credentials:
  
  ```json
  "ConnectionString": "mongodb://username:password@127.0.0.1:27017/ProductDB"
  ```

- Check authentication settings in `/etc/mongod.conf` under the `security` section:
  
  ```yaml
  security:
    authorization: enabled
  ```
  
- If authentication is enabled, login manually:
  
  ```bash
  mongosh "mongodb://127.0.0.1:27017" --username yourUser --password yourPassword --authenticationDatabase admin
  
---

## **Difference Between `mongosh` and `mongosh "mongodb://127.0.0.1:27017/?directConnection=true"`**

### **1. `mongosh "mongodb://127.0.0.1:27017/?directConnection=true"`**
*   This explicitly connects to MongoDB running on `127.0.0.1:27017`.
*   The `?directConnection=true` flag forces a direct connection to a **single** MongoDB instance, bypassing automatic discovery of replica sets or sharded clusters.
*   Useful when connecting to **standalone** instances or when you want to avoid any delay in server selection.

### **2. `mongosh`**
*   This command tries to connect to the **default MongoDB instance** on `mongodb://localhost:27017/`.
*   If `localhost` resolves to `::1` (IPv6), but MongoDB is bound to `127.0.0.1` (IPv4), it may fail.
*   It automatically attempts to detect whether it's connecting to a standalone, replica set, or sharded cluster.

### **When to Use Which?**
*   If you're troubleshooting connection issues, **explicitly specifying `127.0.0.1` with `directConnection=true`** ensures you are directly connecting to the correct instance.
*   If MongoDB is running normally and bound to the default `localhost:27017`, the shorter `mongosh` command works fine.
