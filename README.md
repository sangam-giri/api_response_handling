# 📄 API Response Handling in Flutter (Without Freezed)

This document describes a **reusable and scalable approach** for handling common API response structures in Flutter using regular Dart classes. It avoids code repetition and improves maintainability by wrapping responses in a generic model.

---

## ✅ Use Case

Most API responses include **common metadata** like:

```json
{
  "data": { ... },
  "pagination": { "total": 100, "page": 1 },
  "createdAt": "2023-01-01T00:00:00Z",
  "updatedAt": "2023-01-02T00:00:00Z"
}
```

Instead of manually repeating this structure in each model, we create a **generic API response wrapper**.

---

## 📦 Project Structure

```
lib/
├── models/
│   ├── user.dart
│   ├── pagination_model.dart
│   └── api_response.dart
```

---

## 🔧 1. `PaginationModel`

```dart
// models/pagination_model.dart
class PaginationModel {
  final int total;
  final int page;

  PaginationModel({required this.total, required this.page});

  factory PaginationModel.fromJson(Map<String, dynamic> json) {
    return PaginationModel(
      total: json['total'],
      page: json['page'],
    );
  }
}
```

---

## 🔧 2. `ApiResponse<T>` (Generic Wrapper)

```dart
// models/api_response.dart
import 'pagination_model.dart';

class ApiResponse<T> {
  final T data;
  final PaginationModel? pagination;
  final DateTime? createdAt;
  final DateTime? updatedAt;

  ApiResponse({
    required this.data,
    this.pagination,
    this.createdAt,
    this.updatedAt,
  });

  static ApiResponse<T> fromJson<T>(
    Map<String, dynamic> json,
    T Function(Map<String, dynamic>) fromJsonT,
  ) {
    return ApiResponse<T>(
      data: fromJsonT(json['data']),
      pagination: json['pagination'] != null
          ? PaginationModel.fromJson(json['pagination'])
          : null,
      createdAt: json['createdAt'] != null
          ? DateTime.parse(json['createdAt'])
          : null,
      updatedAt: json['updatedAt'] != null
          ? DateTime.parse(json['updatedAt'])
          : null,
    );
  }

  static ApiResponse<List<T>> fromJsonList<T>(
    Map<String, dynamic> json,
    T Function(Map<String, dynamic>) fromJsonT,
  ) {
    return ApiResponse<List<T>>(
      data: List<T>.from(
        (json['data'] as List).map((item) => fromJsonT(item)),
      ),
      pagination: json['pagination'] != null
          ? PaginationModel.fromJson(json['pagination'])
          : null,
      createdAt: json['createdAt'] != null
          ? DateTime.parse(json['createdAt'])
          : null,
      updatedAt: json['updatedAt'] != null
          ? DateTime.parse(json['updatedAt'])
          : null,
    );
  }
}
```

---

## 🔧 3. Example Data Model – `User`

```dart
// models/user.dart
class User {
  final String name;
  final String email;

  User({required this.name, required this.email});

  factory User.fromJson(Map<String, dynamic> json) {
    return User(
      name: json['name'],
      email: json['email'],
    );
  }
}
```

---

## 🚀 4. Parsing the API Response

### Single Object:

```dart
final jsonResponse = jsonDecode(response.body);
final userResponse = ApiResponse<User>.fromJson(
  jsonResponse,
  (json) => User.fromJson(json),
);

print(userResponse.data.name); // "John"
```

### List of Objects:

```dart
final jsonResponse = jsonDecode(response.body);
final usersResponse = ApiResponse<List<User>>.fromJsonList(
  jsonResponse,
  (json) => User.fromJson(json),
);

print(usersResponse.data.length); // e.g., 10 users
```

---

## ✅ Benefits

* Centralized response handling
* Reusable for **all models**
* Works without code generation
* Keeps individual models clean and focused

---
