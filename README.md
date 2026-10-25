# Basic File Manager API

A REST API for user-owned directories and file operations, built with **Java 17** and **Spring Boot 3.4.4**. The project uses PostgreSQL for relational data, MongoDB for activity events, and the local filesystem for uploaded file contents.

## Features

- Create, retrieve, and delete users
- Create, list, and delete user-owned directories
- Upload, list, download, and delete files
- Store per-user, per-directory permission values: `READ`, `WRITE`, or `NONE`
- Record upload, download, and delete events in MongoDB
- Explore the API through generated OpenAPI documentation and Swagger UI

This repository provides a backend API; it does not include a separate frontend.

## Technology

| Area | Tools |
|---|---|
| Language | Java 17 |
| Application framework | Spring Boot 3.4.4, Spring MVC |
| Relational persistence | Spring Data JPA, Hibernate, PostgreSQL |
| Activity events | Spring Data MongoDB, MongoDB |
| API documentation | Springdoc OpenAPI 2.3.0 |
| Validation | Spring Boot Validation |
| Build | Maven Wrapper |
| Other | Lombok |

## Architecture

The code is organized into controllers, services, repositories, and models under `src/main/java/ir/hamtasharif/filemanager/`.

- **Controllers** expose the REST endpoints.
- **Services** implement user, directory, file, permission-record, and activity-log operations.
- **Repositories** use Spring Data JPA or MongoDB.
- **Storage** keeps uploaded file bytes on disk at `<working-directory>/uploads/<directory-id>/`; file metadata is stored in PostgreSQL, and activity events are stored in MongoDB.

## Getting started

### Prerequisites

- JDK 17
- PostgreSQL
- MongoDB
- Git

### 1. Clone the repository

```bash
git clone https://github.com/niusha-adib20/basic_filemanager.git
cd basic_filemanager
```

### 2. Create the PostgreSQL database

Start PostgreSQL and create a database named `filemanager`:

```sql
CREATE DATABASE filemanager;
```

Start MongoDB as well. The default MongoDB URI targets `mongodb://localhost:27017/filemanager`.

### 3. Configure database access

The application reads Spring Boot configuration properties. Set environment variables for your own local database settings before starting it. These values override the corresponding values in `src/main/resources/application.properties`.

**Windows PowerShell:**

```powershell
$env:SPRING_DATASOURCE_URL = "jdbc:postgresql://localhost:5432/filemanager"
$env:SPRING_DATASOURCE_USERNAME = "your-postgres-username"
$env:SPRING_DATASOURCE_PASSWORD = "your-local-password"
$env:SPRING_DATA_MONGODB_URI = "mongodb://localhost:27017/filemanager"
```

**macOS / Linux:**

```bash
export SPRING_DATASOURCE_URL="jdbc:postgresql://localhost:5432/filemanager"
export SPRING_DATASOURCE_USERNAME="your-postgres-username"
export SPRING_DATASOURCE_PASSWORD="your-local-password"
export SPRING_DATA_MONGODB_URI="mongodb://localhost:27017/filemanager"
```

Use credentials appropriate for your local environment. Do not commit passwords, tokens, or production credentials.

### 4. Run the application

The Maven Wrapper is included, so a separate Maven installation is not required.

**Windows (PowerShell or Command Prompt):**

```powershell
.\mvnw.cmd spring-boot:run
```

**macOS / Linux:**

```bash
chmod +x mvnw
./mvnw spring-boot:run
```

When startup completes, the API is available at [http://localhost:8080](http://localhost:8080). Spring Boot's default port is used unless overridden.

## API documentation

With the application running, open:

- [Swagger UI](http://localhost:8080/swagger-ui/index.html)
- [OpenAPI JSON](http://localhost:8080/v3/api-docs)

## REST endpoints

### Users

| Method | Endpoint | Description |
|---|---|---|
| POST | `/users` | Create a user |
| GET | `/users/{username}` | Retrieve a user by username |
| DELETE | `/users/{id}` | Delete a user by ID |

Create a user:

```bash
curl -X POST http://localhost:8080/users \
  -H "Content-Type: application/json" \
  -d '{"username":"alice","password":"use-a-local-test-password"}'
```

### Directories

| Method | Endpoint | Description |
|---|---|---|
| POST | `/directories/{username}` | Create a directory for a user |
| GET | `/directories/{username}` | List a user's directories |
| DELETE | `/directories/{id}` | Delete a directory by ID |

Create a directory:

```bash
curl -X POST http://localhost:8080/directories/alice \
  -H "Content-Type: application/json" \
  -d '{"name":"Documents","path":"Documents"}'
```

### Files

| Method | Endpoint | Description |
|---|---|---|
| POST | `/files/upload/{directoryId}` | Upload multipart form data using the `file` field |
| GET | `/files/list/{directoryId}` | List file metadata for a directory |
| GET | `/files/download/{username}/{fileId}` | Download a file by ID |
| DELETE | `/files/{fileId}` | Delete a file by ID |

Upload a file:

```bash
curl -X POST http://localhost:8080/files/upload/1 \
  -F "file=@./example.txt"
```

List files:

```bash
curl http://localhost:8080/files/list/1
```

### Permissions

| Method | Endpoint | Description |
|---|---|---|
| POST | `/permissions/{username}/{directoryId}?accessType={type}` | Create or update a permission record |
| GET | `/permissions/{username}/{directoryId}` | Read a user's permission value |

Supported values are `READ`, `WRITE`, and `NONE`.

For example:

```bash
curl -X POST "http://localhost:8080/permissions/alice/1?accessType=READ"
```

Permission records are currently stored and retrievable through these endpoints. They are not yet enforced as authorization checks on file or directory operations.

## Tests

Run the project tests with the Maven Wrapper:

**Windows:**

```powershell
.\mvnw.cmd test
```

**macOS / Linux:**

```bash
./mvnw test
```

## Current scope and security considerations

This project is a backend learning/prototype project and should not be exposed to untrusted users without additional security work:

- The current API does not implement an authentication flow or consistently enforce authorization.
- Permission endpoints manage permission records, but file and directory endpoints do not apply those records as access-control checks.
- Passwords are currently handled as plain values by the user model/API. A real service must hash passwords and must never return them in API responses.
- Uploaded filenames and file sizes need stronger validation and limits before accepting untrusted uploads.
- Uploaded files are stored on the local filesystem, so deployment needs appropriate storage permissions, backups, and cleanup behavior.
- The current JPA schema setting uses `ddl-auto=update`, which is convenient for development but should be replaced with controlled migrations for production.

## License

No license file is currently included. Add a license file before redistributing this project under open-source terms.
