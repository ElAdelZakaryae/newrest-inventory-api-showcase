# Deployment

## Observed Configuration

The reviewed application:

- Reads `MYSQL_PUBLIC_URL` from the environment.
- Parses the URL with `urllib.parse`.
- Connects through `mysql.connector`.
- Reads the Firebase credential-file path from `FIREBASE_CREDENTIALS_JSON`, falling back to a local filename.
- Starts Flask on `0.0.0.0:5000` when executed directly.
- Creates local upload directories for profile images and order attachments.

## Portfolio Deployment Model

```text
Flutter Mobile / Flutter Web
              |
            HTTPS
              |
         Flask API
          |      |
        MySQL   Firebase Admin
```

The project deployment uses Render for the API and Railway for MySQL, with Docker used to package the backend environment.

## Filesystem Consideration

The API stores profile images and order attachments on the application filesystem. On platforms with an ephemeral filesystem, these files may disappear after restarts or redeployments. A production improvement is to use durable object storage and keep only metadata/URLs in MySQL.

## Required Private Configuration

- MySQL connection URL
- Firebase Admin service-account file
- Approved CORS origins
- Authentication/session secrets after hardening
- Upload and storage settings

## Safe Deployment Practices

- Never commit secrets, even inside comments.
- Rotate credentials that were previously committed.
- Disable Flask debug mode in hosted production.
- Protect debug and simulation routes.
- Restrict CORS.
- Use durable file storage.
- Return generic server errors and keep details in private logs.
- Apply database schema changes through migrations.

