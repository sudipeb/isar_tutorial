# Isar Tutorial

A Flutter application demonstrating CRUD operations using the [Isar](https://pub.dev/packages/isar_community) local database.

## Overview

This app provides a simple user management interface where you can:

- **Create** users with a name and age
- **Read** and list all users in real-time using streams
- **Update** existing user records
- **Delete** users from the database
- **Filter** users by name and age using Isar's query API

## Tech Stack

- **Flutter** (SDK ^3.11.0)
- **Isar Community** (^3.3.0) — fast, lightweight NoSQL database
- **Path Provider** (^2.1.5) — for accessing device storage directories

## Project Structure

```
lib/
├── main.dart          # App entry point and UI (home page, add/update modal)
├── user.dart          # User model with Isar collection annotation
├── user.g.dart        # Generated Isar schema for User
└── isar_service.dart  # Database service (open, save, read, update, delete, filter)
```

## Isar Architecture

Isar is a fast, lightweight NoSQL database for Flutter. It stores data locally on the device using a binary format for high performance.

### How Isar Works

```
┌─────────────────────────────────────────────────┐
│                   Flutter UI                    │
│              (main.dart - MyHomePage)            │
│    StreamBuilder listens for real-time updates   │
└──────────────────────┬──────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────┐
│              IsarService (Service Layer)         │
│            (isar_service.dart)                   │
│                                                 │
│  saveUser()  ──▶ writeTxnSync / putSync         │
│  getAllUser() ──▶ where().findAll()              │
│  listenUser()──▶ where().watch() (Stream)       │
│  UpdateUser()──▶ writeTxn / put                 │
│  deleteUser()──▶ writeTxn / delete              │
│  filterName()──▶ filter().nameContains().age... │
└──────────────────────┬──────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────┐
│                Isar Database                    │
│         (Binary storage on device)              │
│                                                 │
│  Collections:  users (User schema)              │
│  Location:     Application Documents Directory  │
│  Schema:       Auto-generated from user.g.dart  │
└─────────────────────────────────────────────────┘
```

### Key Concepts

| Concept | Description |
|---|---|
| **Collection** | A table equivalent — each Dart class annotated with `@collection` becomes a collection |
| **Schema** | Auto-generated code (`user.g.dart`) that tells Isar how to serialize/deserialize objects |
| **Transaction** | All write operations (`put`, `delete`) must run inside a transaction (`writeTxn`) |
| **Query** | Isar provides `where()` clauses, `filter()` chains, and sorting for reading data |
| **Watch (Stream)** | `watch(fireImmediately: true)` returns a reactive stream that emits on every collection change |

### Step-by-Step Setup Guide

#### Step 1 — Add Dependencies

Add the Isar packages to `pubspec.yaml`:

```yaml
dependencies:
  isar_community: ^3.3.0
  isar_community_flutter_libs: ^3.3.0  # Native binaries
  path_provider: ^2.1.5                # For storage directory
```

#### Step 2 — Define a Collection (Model)

Create a Dart class with the `@collection` annotation. Each collection needs an `Id` field.

```dart
// user.dart
import 'package:isar_community/isar.dart';
part 'user.g.dart';

@collection
class User {
  User(this.name, this.age);
  Id id = Isar.autoIncrement; // Auto-incrementing primary key
  String? name;
  int? age;
}
```

#### Step 3 — Generate the Schema

Run the build runner to generate `user.g.dart`:

```bash
dart run build_runner build
```

This generates the `UserSchema` used when opening the database.

#### Step 4 — Open the Database

Use `path_provider` to get the device directory, then open Isar with the required schemas:

```dart
final dir = await getApplicationDocumentsDirectory();
final isar = await Isar.open(
  [UserSchema],       // Register all collection schemas
  directory: dir.path,
);
```

#### Step 5 — Create a Service Layer

Encapsulate all database operations in a service class (`IsarService`) for clean separation:

- **`saveUser()`** — Writes a new user via `writeTxnSync` + `putSync`
- **`getAllUser()`** — Reads all users via `where().findAll()`
- **`listenUser()`** — Returns a `Stream<List<User>>` via `where().watch()`
- **`UpdateUser()`** — Updates a user via `writeTxn` + `put`
- **`deleteUser()`** — Deletes by ID via `writeTxn` + `delete`
- **`filterName()`** — Queries with `filter().nameContains().ageEqualTo()`

#### Step 6 — Connect UI to the Service

Use `StreamBuilder` in the widget tree to reactively display data:

```dart
StreamBuilder<List<User>>(
  stream: isarService.listenUser(),
  builder: (context, snapshot) {
    if (snapshot.hasData) {
      return ListView.builder(
        itemCount: snapshot.data!.length,
        itemBuilder: (context, index) => Text(snapshot.data![index].name!),
      );
    }
    return CircularProgressIndicator();
  },
)
```

The UI automatically rebuilds whenever data in the Isar `users` collection changes.

## Getting Started

### Prerequisites

- Flutter SDK ^3.11.0
- Dart SDK ^3.11.0

### Installation

1. Clone the repository:
   ```bash
   git clone <repository-url>
   cd isartutorial
   ```

2. Install dependencies:
   ```bash
   flutter pub get
   ```

3. Generate Isar schemas (if needed):
   ```bash
   dart run build_runner build
   ```

4. Run the app:
   ```bash
   flutter run
   ```

## Usage

- Tap the **+** icon in the app bar to add a new user (name and age).
- Tap the **refresh** icon on a user row to update their details.
- Tap the **delete** icon to remove a user.
- The user list updates in real-time via Isar's `watch` stream.
