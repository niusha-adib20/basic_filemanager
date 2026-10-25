# 📁 Basic File Manager API

A RESTful file-management backend built with **Java 17** and **Spring Boot**. It provides endpoints for managing users and directories, uploading and downloading files, assigning directory-level access permissions, and recording file activity.

## ✨ Features

- **User management:** Create, retrieve, and delete users.
- **Directory management:** Create, list, and delete user-owned directories.
- **File operations:** Upload files to a directory, list directory contents, download files, and delete files.
- **Access permissions:** Store a user's access level for a directory using `READ`, `WRITE`, or `NONE`.
- **Activity logging:** Record file actions such as uploads, downloads, and deletions.
- **REST API:** Expose functionality through Spring MVC controllers.
- **API documentation:** Includes Springdoc OpenAPI / Swagger UI dependency.
- **Persistence:** Uses Spring Data JPA with PostgreSQL for relational entities and Spring Data MongoDB for activity logs.

> **Note:** This repository is a backend API project. It does not currently include a dedicated frontend interface.

## 🛠️ Tech Stack

- **Language:** Java 17
- **Framework:** Spring Boot 3.4.4
- **Web layer:** Spring Web (REST controllers)
- **Relational persistence:** Spring Data JPA, Hibernate, PostgreSQL
- **Activity logs:** Spring Data MongoDB
- **Validation:** Spring Boot Validation
- **API documentation:** Springdoc OpenAPI / Swagger UI
- **Build tool:** Maven
- **Boilerplate reduction:** Lombok

## 🧱 Architecture

The application follows a layered structure:

- **Controller:** Defines REST endpoints and HTTP responses.
- **Service:** Implements user, directory, permission, file, and activity-log operations.
- **Repository:** Uses Spring Data repositories to interact with the databases.
- **Model:** Defines the main entities and activity-log document.

Uploaded files are stored on the local filesystem under the repository's `uploads/` directory, organized by directory ID. File metadata is stored in the relational database, while activity logs are stored in MongoDB.

## 📂 Project Structure

```text
basic_filemanager/
├── pom.xml
├── mvnw
├── mvnw.cmd
├── .mvn/
└── src/
    ├── main/
    │   ├── java/ir/hamtasharif/filemanager/
    │   │   ├── FilemanagerApplication.java
    │   │   ├── controller/
    │   │   │   ├── UserController.java
    │   │   │   ├── DirectoryController.java
    │   │   │   ├── StoredFileController.java
    │   │   │   └── PermissionController.java
    │   │   ├── model/
    │   │   ├── repository/
    │   │   └── service/
    │   └── resources/
    │       └── application.properties
    └── test/
```

## 🚀 Getting Started

### Prerequisites

Before running the application, install or prepare:

- **JDK 17**
- **PostgreSQL**
- **MongoDB**
- Git (optional, for cloning the repository)

### 1. Clone the repository

```bash
git clone https://github.com/niusha-adib20/basic_filemanager.git
cd basic_filemanager
```

### 2. Configure the databases

The current configuration expects a PostgreSQL database named `filemanager` and a MongoDB database also named `filemanager`, both running locally with their default ports.

Create the PostgreSQL database if it does not already exist:

```sql
CREATE DATABASE filemanager;
```

Configure database URLs and credentials for your local environment. **Do not commit real passwords or production credentials to source control.** Prefer environment variables or an external configuration file for secrets.

The application uses PostgreSQL for JPA-managed entities and MongoDB for activity logs. Both services should be available before starting the app.

### 3. Run the application

You can use the Maven Wrapper included in the repository.

**Windows (Command Prompt / PowerShell):**

```powershell
.\mvnw.cmd spring-boot:run
```

**macOS / Linux:**

```bash
chmod +x mvnw
./mvnw spring-boot:run
```

Alternatively, if Maven is installed:

```bash
mvn spring-boot:run
```

When the application starts successfully, the API is available at:

```text
http://localhost:8080
```

The project does not explicitly set a server port in its application properties, so Spring Boot's default port is used unless overridden.

## 🔌 REST API Reference

The following endpoints are defined by the current controllers.

### Users

| Method | Endpoint | Description |
|---|---|---|
| POST | `/users` | Create a user |
| GET | `/users/{username}` | Retrieve a user by username |
| DELETE | `/users/{id}` | Delete a user by ID |

Example — create a user:

```bash
curl -X POST http://localhost:8080/users \
  -H "Content-Type: application/json" \
  -d '{
    "username": "alice",
    "password": "change-me"
  }'
```

### Directories

| Method | Endpoint | Description |
|---|---|---|
| POST | `/directories/{username}` | Create a directory owned by the specified user |
| GET | `/directories/{username}` | List directories owned by the specified user |
| DELETE | `/directories/{id}` | Delete a directory by ID |

Example — create a directory:

```bash
curl -X POST http://localhost:8080/directories/alice \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Documents",
    "path": "Documents"
  }'
```

### Files

| Method | Endpoint | Description |
|---|---|---|
| POST | `/files/upload/{directoryId}` | Upload a file as multipart form data using the `file` field |
| GET | `/files/list/{directoryId}` | List files belonging to a directory |
| GET | `/files/download/{username}/{fileId}` | Download a file by ID |
| DELETE | `/files/{fileId}` | Delete a file by ID |

Example — upload a file:

```bash
curl -X POST http://localhost:8080/files/upload/1 \
  -F "file=@./example.txt"
```

Example — list files in a directory:

```bash
curl http://localhost:8080/files/list/1
```

### Permissions

| Method | Endpoint | Description |
|---|---|---|
| POST | `/permissions/{username}/{directoryId}?accessType={type}` | Create or update a user's directory permission |
| GET | `/permissions/{username}/{directoryId}` | Retrieve the user's permission for a directory |

Supported `accessType` values:

- `READ`
- `WRITE`
- `NONE`

Example — set a permission:

```bash
curl -X POST "http://localhost:8080/permissions/alice/1?accessType=READ"
```

### API documentation

Springdoc OpenAPI is included in the project. When the application is running, try:

- Swagger UI: [http://localhost:8080/swagger-ui/index.html](http://localhost:8080/swagger-ui/index.html)
- OpenAPI JSON: [http://localhost:8080/v3/api-docs](http://localhost:8080/v3/api-docs)

## 🗃️ Data Models

The main application models are:

- **User:** Stores a username and password, and owns directories.
- **Directory:** Represents a user-owned directory and its associated files.
- **StoredFile:** Stores file metadata such as name, content type, filesystem path, size, and directory.
- **Permission:** Associates a user and directory with an access type.
- **ActivityLog:** Stores a username, action, details, and timestamp in MongoDB.

## ⚠️ Important Notes

- **Authentication and authorization:** The current controllers do not show a complete authentication flow. Permission records exist, but access checks must be enforced consistently before using this as a multi-user or production service.
- **Password handling:** The current user model includes a password field. For a real deployment, passwords must be securely hashed and never returned in API responses.
- **File safety:** Validate uploaded filenames and content, enforce file-size limits, and prevent path traversal before exposing the service to untrusted users.
- **Database credentials:** Keep passwords and secrets outside committed configuration files.
- **Data storage:** Files are stored locally; a production deployment should plan for backups, storage permissions, and cleanup consistency between files and database metadata.
- **Activity logs:** Upload, download, and delete events are recorded in MongoDB.
- **Testing:** A Spring Boot test source is included. Run the test suite with the Maven Wrapper:

```bash
./mvnw test
```

On Windows:

```powershell
.\mvnw.cmd test
```

## 📚 References

- [Spring Boot Documentation](https://docs.spring.io/spring-boot/index.html)
- [Spring Web MVC](https://docs.spring.io/spring-framework/reference/web/webmvc.html)
- [Spring Data JPA](https://docs.spring.io/spring-data/jpa/reference/)
- [Spring Data MongoDB](https://docs.spring.io/spring-data/mongodb/reference/)
- [Springdoc OpenAPI](https://springdoc.org/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)

---

Built with Java and Spring Boot. ☕
