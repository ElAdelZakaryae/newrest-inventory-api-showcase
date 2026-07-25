# Implementation Notes and Hardening Backlog

This document distinguishes features present in the reviewed Flask file from recommended next steps.

## Present in the Code

- 52 Flask route declarations
- Direct MySQL access with parameterized statements
- Werkzeug password hashing
- Random login/registration token generation
- Inventory-to-stock synchronization on validation
- Daily snapshot aggregation with `ROW_NUMBER()`
- Product threshold configuration and expiry-window days
- Order totals calculated server-side
- Order attachments with an extension allowlist
- FCM token storage and site-targeted multicast notification
- Profile image upload and serving
- Report and order-export history
- Stock simulation that generates snapshots

## Not Present in the Reviewed File

- Verified JWT authentication
- Persisted/validated opaque sessions
- Authentication decorators on protected routes
- Server-enforced role permissions
- Server-enforced site scoping for every applicable route
- A dedicated replenishment-calculation endpoint
- A task scheduler that invokes the stock simulation automatically
- Durable cloud object storage for uploads
- Versioned schema migrations

## Priority Improvements

### Critical

- Rotate and purge credentials that appeared in comments/history.
- Add verified authentication and authorization.
- Protect or remove debug/simulation endpoints.
- Disable production debug mode.

### High

- Restrict CORS.
- Validate order and inventory state transitions.
- Replace raw exception responses with safe error messages.
- Add upload size/MIME validation and durable storage.

### Maintainability

- Split the monolithic application into blueprints/services/repositories.
- Add automated route, transaction, and authorization tests.
- Add database migrations.
- Add structured logging and request correlation.

