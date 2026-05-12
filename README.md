# 🛒 Buffalo Basket — E-Commerce Delivery System Database Design

> End-to-end relational database design for a neighborhood delivery platform, built from scratch with full normalization analysis, storage estimation, and indexing strategy.

---

## Overview

Buffalo Basket is a local delivery platform that connects customers, stores, products, payments, and couriers — similar in spirit to Amazon or Instacart. This project covers the complete database design lifecycle: from business rules and ER modeling to schema definition, normalization, storage analysis, and query optimization.

---

## Skills Demonstrated

| Area | Details |
|------|---------|
| **Data Modeling** | Entity-Relationship Design, relational schema translation |
| **SQL & Schema Design** | PostgreSQL-style DDL, constraints, composite keys, FK cascades |
| **Normalization** | Functional dependency analysis, 3NF / BCNF decomposition, anomaly identification |
| **Storage Analysis** | Tuple width estimation, page calculations, I/O impact of wide attributes |
| **Query Optimization** | Workload-driven index design, composite index ordering, write amplification tradeoffs |
| **Constraint Engineering** | Enforcing business rules at the DB layer (triggers, composite FKs, bridge tables) |

---

## Database Schema

The schema covers **11 tables** modeling a full e-commerce and delivery workflow:

```
CUSTOMER → ADDRESS
CATEGORY → PRODUCT ← STORE_PRODUCT ← STORE
CUSTOMER → ORDER → ORDER_ITEM → DELIVERY
ORDER → PAYMENT
PURCHASE_ITEM → REVIEW
COURIER → DELIVERY
```

### Key Design Decisions

**Price snapshot in `ORDER_ITEM`**
Stores `unit_price` as a snapshot at checkout rather than joining to `STORE_PRODUCT`. This prevents historical order totals from silently changing when a store updates its prices — critical for accurate receipts and refunds.

**Review eligibility via bridge table**
A `PURCHASE_ITEM` bridge table is populated when an order is delivered, and `REVIEW` references it via a composite foreign key `(customer_id, product_id, order_item_id)`. The database enforces the "must have purchased to review" rule at the constraint level — not just application code.

**Split-delivery support**
A `DELIVERY` entity (separate from `ORDER`) allows one order to be split across multiple deliveries, each handled by a different courier. `ORDER_ITEM.delivery_id` links each line item to its assigned delivery segment.

---

## Normalization Analysis

- **Extended flat relation** (`OrderExport`) analyzed for functional dependencies
- **Candidate key** identified as `(order_id, product_id)` with full justification
- **Update and deletion anomalies** demonstrated with concrete examples
- **BCNF decomposition** into 10+ normalized relations — every non-key attribute depends on the full key

---

## Storage & Indexing

**Storage estimate for `ORDER_ITEM` at 2M rows:**
- Tuple width ≈ 53 bytes → ~152 tuples/page → ~13,158 pages → **~103 MB**
- Adding a 200-char `TEXT` column drops to ~31 tuples/page → **5× more pages**, 5× slower full scans

**Proposed indexes with justification:**

| Index | Table | Purpose |
|-------|-------|---------|
| `(customer_id, status, order_ts DESC)` | `ORDER` | Most frequent customer-facing query — open orders sorted by recency |
| `(order_id)` | `ORDER_ITEM` | Fast item lookup per order; likely auto-created from FK |
| `(store_id, quantity_on_hand)` | `STORE_PRODUCT` | Low-stock checks across large store catalogs |

Write-heavy tables like `DELIVERY` are intentionally kept with minimal indexes to avoid write amplification on frequent status updates.

---

## Project Structure

```
buffalo_basket_db_design.docx   # Full design report
README.md                       # This file
```

---

## Tools & Concepts

`PostgreSQL` · `Relational Modeling` · `ERD` · `Normalization (BCNF)` · `Composite Foreign Keys` · `Index Design` · `Storage Estimation` · `Trigger Logic`

---

## Author

**Tsomorlig Khishigbold**
