# Task 1 — Build the App Locally + Attach MySQL Database Manually

## Goal

Run the Spring PetClinic application locally (outside Docker), and connect it to a real MySQL database running inside a Docker container — replacing the default in-memory database (H2), which loses all data every time the application stops.

## Prerequisites

- Docker installed and running (`docker --version` should return a version, not an error)
- Java 17+ and the project's own Maven wrapper (`./mvnw`) — no need to install Maven separately
- The repo cloned locally:

```bash
git clone https://github.com/spring-projects/spring-petclinic.git
cd spring-petclinic
```

## Step 1 - Start Application locally

```bash
git clone https://github.com/spring-projects/spring-petclinic.git
cd spring-petclinic
./mvnw spring-boot:run
```

## Step 2 — Start MySQL as a standalone Docker container

```bash
docker run --name petclinic-mysql \
  -e MYSQL_USER=petclinic \
  -e MYSQL_PASSWORD=petclinic \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=petclinic \
  -p 3306:3306 \
  -d mysql:8.0
```

| Flag | Purpose |
| --- | --- |
| `--name petclinic-mysql` | Human-readable name for the container |
| `-e MYSQL_USER / -e MYSQL_PASSWORD` | Creates the app's database user automatically on first boot |
| `-e MYSQL_ROOT_PASSWORD` | Sets the admin password (required by the image even if unused) |
| `-e MYSQL_DATABASE=petclinic` | Creates an empty petclinic schema automatically |
| `-p 3306:3306` | Maps host port 3306 to the container's port 3306 |
| `-d` | Runs the container in the background (detached) |

## Step 3 — Connect to the database directly and confirm it's empty

No local MySQL client was installed — instead, the mysql client already bundled inside the container itself was used:

```bash
docker exec -it petclinic-mysql mysql -u petclinic -ppetclinic petclinic
```

## Step 4 — Run the application with the mysql Spring profile

```bash
./mvnw spring-boot:run -Dspring-boot.run.profiles=mysql
```

The profile name mysql comes directly from the properties file name (application-mysql.properties).
No mention of jdbc:h2:mem anywhere — this confirms the app is using the real MySQL container, not the default in-memory database.

## Step 5 — Confirm the application created the schema automatically

Back in the MySQL terminal session:

```sql
SHOW TABLES;
```

Result:

```text
+---------------------+
| Tables_in_petclinic |
+---------------------+
| owners              |
| pets                |
| specialties         |
| types               |
| vet_specialties     |
| vets                |
| visits              |
+---------------------+
7 rows in set
```

## Step 6 — Add Owner from ui

Opened http://localhost:8080 in the browser, then: Find Owners → Add Owner, and submitted a new owner with real form data.

## Step 7 — Verify the write directly in the database

Back in the MySQL session:

```sql
SELECT * FROM owners ORDER BY id DESC LIMIT 1;
```

Actual result:

```text
+----+------------+-----------+---------+---------------+------------+
| id | first_name | last_name | address | city          | telephone  |
+----+------------+-----------+---------+---------------+------------+
| 11 | Reham      | Samir     | sharqia | El-Huseiniaya | 0109508138 |
+----+------------+-----------+---------+---------------+------------+
```

The data entered in the browser matches exactly what's stored in the database — proving the full chain works end to end:

Browser (UI) → Spring Boot Application → Docker MySQL Container

## Step 8 — Update

Edited the owner's phone number from the UI, then confirmed the change via:

```sql
SELECT telephone FROM owners WHERE id = 11;
```

Both the update and the delete were reflected correctly in MySQL, confirming reads, writes, updates, and deletes all flow correctly between the UI and the real database.

## Summary

Started MySQL manually via docker run, no orchestration tool involved, Ran the application locally via ./mvnw with the mysql Spring profile active, Confirmed the application creates its own schema automatically on first connection, Proved the full data flow — UI → Application → Database — for create, read, update, and delete operations, Diagnosed and resolved four real issues independently, using logs as evidence rather than guessing.
