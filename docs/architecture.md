# Architecture

## Observed Application Structure

The reviewed backend is a single Flask application containing route handlers, SQL queries, serialization helpers, upload handling, Firebase integration, and application startup.

| Layer/Component | Observed implementation |
|---|---|
| HTTP application | Flask |
| Cross-origin support | Global `CORS(app)` |
| Database access | Direct `mysql.connector` connections and parameterized SQL |
| Passwords | Werkzeug `generate_password_hash` and `check_password_hash` |
| IDs | UUIDs for users, alerts, and uploaded filenames |
| Login token | Random hexadecimal value from `secrets.token_hex(32)` |
| File uploads | Local `uploads/` directories and `send_from_directory` |
| Notifications | Optional Firebase Admin initialization and multicast FCM |
| Configuration | MySQL URL and Firebase credential path from environment variables |

## Runtime Flow

1. A Flutter client sends an HTTP request.
2. The Flask route validates required input fields.
3. A new MySQL connection is opened from `MYSQL_PUBLIC_URL`.
4. Parameterized SQL reads or modifies data.
5. Multi-step writes use commit/rollback transactions.
6. Helper functions convert dates and numeric database values to JSON-safe values.
7. The route returns a JSON response and HTTP status.
8. Alert creation can trigger site-targeted Firebase multicast delivery.

## Shared Clients

- Flutter Mobile handles operational inventory, product, report, and alert workflows.
- Flutter Web handles dashboard, configuration, account, reporting, and order workflows.
- Both clients use the same Flask/MySQL backend.

## Current Packaging

The reviewed file is monolithic. A future refactor can separate:

```text
app/
├── routes/
├── services/
├── repositories/
├── models/
├── auth/
├── notifications/
└── config/
```

This would preserve the existing API contract while improving testing and maintainability.

![System architecture](../diagrams/system-architecture.png)

