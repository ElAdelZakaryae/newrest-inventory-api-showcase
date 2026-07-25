# Database Design Observed in `app.py`

The API uses direct parameterized SQL through `mysql.connector`. The following tables are referenced by the reviewed code.

| Table | Purpose in the API |
|---|---|
| `users` | Accounts, password hashes, role, site, status, profile image, last login, and FCM token |
| `products` | SKU catalog, category, unit, supplier, price, batch, DLC, and activation state |
| `product_configs` | Minimum, maximum, and expiry-window settings per SKU |
| `categories` | Category storage location |
| `sites` | Site identifiers and names |
| `locations` | Inventory location identifiers and names |
| `stock_actual` | Counted stock by SKU and site |
| `stock_theoretical` | Theoretical stock by SKU and site |
| `stock_snapshots` | Timestamped actual/theoretical stock history and movement source |
| `inventories` | Inventory header, status, site, location, creators, submitters, and note |
| `inventory_items` | Product counts, theoretical snapshot, DLC, and item notes |
| `alerts` | Alert content, type, read/archive state, relationships, site, and location |
| `report_history` | Report name, filters, format, saving path, creator, and creation time |
| `orders` | Reference, supplier, site, dates, status, total, creator, and note |
| `order_items` | SKU, product, unit price, quantity, and calculated line total |
| `order_attachments` | Attachment metadata and generated file URL |
| `recent_order_files` | Order-export history |

## Important Write Flows

### Inventory creation/update

- Writes the inventory header.
- Inserts or replaces the complete item collection.
- Captures the latest theoretical quantity in each item.

### Inventory submission

- Sets status to `validated`.
- Upserts `stock_actual`.
- Upserts `stock_theoretical`.
- Adds `stock_snapshots` rows with movement source `inventory`.

### Order creation/update

- Recalculates `totalAmount` from item quantity × unit price.
- Stores the order header.
- Inserts order items.
- Update replaces the previous item collection.

### Alerts

- Stores the alert.
- Optionally sends an FCM notification to active users with tokens at the alert site.

## Runtime Schema Change

At direct script startup, `ensure_fcm_token_column()` checks for `users.fcmToken` and adds it when missing. A versioned migration system would be safer for future schema changes.

## Public-Safety Boundary

This document intentionally omits credentials, production records, and complete DDL.

