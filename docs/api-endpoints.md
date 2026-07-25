# API Endpoint Reference

This reference is derived from the real private `app.py`. It documents route contracts without exposing the production host, credentials, or company data.

## General and Diagnostic

| Method | Route | Observed purpose |
|---|---|---|
| GET | `/` | Return API-running message |
| GET | `/debug-user` | Return the current MySQL account; should be protected/removed in production |
| POST | `/simulate/theoretical-stock` | Simulate category-based consumption and insert stock snapshots; test-only |

## Products

| Method | Route | Observed purpose |
|---|---|---|
| GET | `/products` | List SKU, name, category, activation, and timestamps |
| GET | `/products/configs` | Join catalog, configuration, theoretical stock, and actual stock; optional `siteId` |
| GET | `/products/config-summary/{sku}` | Return unit, supplier, price, 28-day usage estimate, and last order |
| PUT | `/products/configs/{sku}` | Upsert `minQty`, `maxQty`, and `expiryWindowDays` |
| GET | `/products/search?q={text}` | Search active products by name or SKU, limited to eight |

## Reports

| Method | Route | Observed purpose |
|---|---|---|
| GET | `/reports/product/{sku}` | Return product stock/report details; optional `siteId` or `site` |
| GET | `/reports/stock-movement` | Return category/product movement, summary, thresholds, and date-series data |
| GET | `/reports/recent` | Return up to 1,000 report-history rows |
| POST | `/reports/recent` | Insert a report-history record |

### Stock-movement query

`/reports/stock-movement` accepts:

- `type=category|product`
- `startDate=YYYY-MM-DD`
- `endDate=YYYY-MM-DD`
- `site` (optional)
- `category` when `type=category`
- `sku` when `type=product`
- `productName` as returned filter metadata

The query uses the latest snapshot per site, SKU, and day through `ROW_NUMBER()`.

## Orders

| Method | Route | Observed purpose |
|---|---|---|
| GET | `/orders` | Return orders with items and attachments |
| GET | `/orders/{id-or-reference}` | Find an order by numeric ID or reference |
| POST | `/orders` | Create an order and calculate totals from its items |
| PUT | `/orders/{id}` | Update the header and replace its item collection |
| DELETE | `/orders/{id}` | Soft-cancel by changing status to `cancelled` |
| PATCH | `/orders/{id}/status` | Set `pending`, `completed`, `cancelled`, or `delivered` |
| POST | `/orders/{id}/attachments` | Upload one or more allowed attachment files |
| GET | `/uploads/order_attachments/{filename}` | Serve a stored attachment |
| GET | `/orders/summary` | Return counts grouped by order status |
| GET | `/orders/recent-files` | Return the ten latest order-export records |
| POST | `/orders/recent-files` | Insert an order-export record |

Allowed attachment extensions are `pdf`, `xlsx`, `xls`, `jpg`, `jpeg`, and `png`.

## Inventories

| Method | Route | Observed purpose |
|---|---|---|
| GET | `/inventories` | Return inventories with nested site, location, and items |
| GET | `/inventories/{id}` | Return one inventory and its items |
| POST | `/inventories` | Insert an inventory and item collection |
| PUT | `/inventories/{id}` | Update the header and replace its item collection |
| PATCH | `/inventories/{id}/submit` | Validate inventory and synchronize three stock tables |
| DELETE | `/inventories/{id}` | Delete an inventory |

Submission updates:

- `inventories.status`
- `stock_actual`
- `stock_theoretical`
- `stock_snapshots`

## Accounts and Profiles

| Method | Route | Observed purpose |
|---|---|---|
| POST | `/register` | Create an account with a Werkzeug password hash |
| POST | `/login` | Verify credentials, update last login, and return a random token |
| PATCH | `/users/{id}/fcm-token` | Store or clear an FCM token |
| POST | `/users/{id}/profile-image` | Save a profile image and update its URL |
| DELETE | `/users/{id}/profile-image` | Clear the stored profile-image URL |
| GET | `/users` | List user profile and account-state fields |
| PATCH | `/users/{id}/status` | Set status to `Active` or `Inactive` |
| PUT | `/users/{id}` | Update name, email, role, site, and status |
| DELETE | `/users/{id}` | Hard-delete a user row |
| GET | `/uploads/profile_images/{filename}` | Serve a stored profile image |

The random token returned by login/registration is not validated by later routes in the reviewed file.

## Alerts and Notifications

| Method | Route | Observed purpose |
|---|---|---|
| GET | `/alerts` | Filter alerts by archive flag, type, product, or inventory |
| POST | `/alerts` | Insert an alert and attempt site-targeted FCM delivery |
| PATCH | `/alerts/{id}/read` | Mark one alert as read |
| PATCH | `/alerts/read-all` | Mark all non-archived alerts as read |
| PATCH | `/alerts/{id}/archive` | Archive an alert |
| DELETE | `/alerts/{id}` | Delete an alert |
| GET | `/alerts/count` | Count unread, non-archived alerts |

## Dashboard

| Method | Route | Observed purpose |
|---|---|---|
| GET | `/dashboard/stock-alerts` | Return the latest ten non-archived alerts |
| GET | `/dashboard/summary` | Return inventory, product, user, and low-stock counters |
| GET | `/dashboard/inventory-evolution` | Count inventories by creation day |
| GET | `/dashboard/recent-inventories` | Return the five latest inventories with items |
| GET | `/dashboard/inventories` | Return all inventories with items |
| GET | `/dashboard/low-stock-products` | Return up to 20 products below configured minimum |

## Response Behavior

- JSON is used for normal API responses.
- Validation failures generally return `400`.
- Authentication failures return `401`; inactive accounts return `403`.
- Missing records generally return `404`.
- Successful creates generally return `201`.
- Failed multi-step writes generally roll back.
- Several current handlers return raw exception strings with `500`; production hardening should replace them with generic messages.

