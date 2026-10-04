# Revenue Reporting Experiment

## Reference Result

The base query is treated as the reference because it calculates the result directly from the base tables.

| Operator | Date       | Captured Amount | Captured Payments |
| -------- | ---------- | --------------: | ----------------: |
| OP-BUS   | 2026-04-29 |              36 |                 1 |

---

## Test Results

### 1. Captured payment insert

**Action:** Insert a new payment with `status = 'Captured'`.

| Approach          | Captured Amount | Captured Payments | Matches Base Query? |
| ----------------- | --------------: | ----------------: | :-----------------: |
| Base query        |              72 |                 2 |      Reference      |
| SQL function      |              72 |                 2 |         Yes         |
| Materialized view |              36 |                 1 |          No         |
| Trigger summary   |              72 |                 2 |         Yes         |

---

### 2. Failed payment insert

**Action:** Insert a new payment with `status = 'Failed'`.

| Approach          | Captured Amount | Captured Payments | Matches Base Query? |
| ----------------- | --------------: | ----------------: | :-----------------: |
| Base query        |              72 |                 2 |      Reference      |
| SQL function      |              72 |                 2 |         Yes         |
| Materialized view |              36 |                 1 |          No         |
| Trigger summary   |              72 |                 2 |         Yes         |

---

### 3. Status correction: Failed → Captured

**Action:** Change an existing payment from `Failed` to `Captured`.

| Approach          | Captured Amount | Captured Payments | Matches Base Query? |
| ----------------- | --------------: | ----------------: | :-----------------: |
| Base query        |             122 |                 3 |      Reference      |
| SQL function      |             122 |                 3 |         Yes         |
| Materialized view |              36 |                 1 |          No         |
| Trigger summary   |              72 |                 2 |          No         |

---

### 4. Status correction: Captured → Refunded

**Action:** Change the inserted captured payment of 36 from `Captured` to `Refunded`.

| Approach          | Captured Amount | Captured Payments | Matches Base Query? |
| ----------------- | --------------: | ----------------: | :-----------------: |
| Base query        |              86 |                 2 |      Reference      |
| SQL function      |              86 |                 2 |         Yes         |
| Materialized view |              36 |                 1 |          No         |
| Trigger summary   |              72 |                 2 |          No         |

---

### 5. Deletion or replacement of test data

**Action:** Delete the corrected payment of 50.

| Approach          | Captured Amount | Captured Payments | Matches Base Query? |
| ----------------- | --------------: | ----------------: | :-----------------: |
| Base query        |              36 |                 1 |      Reference      |
| SQL function      |              36 |                 1 |         Yes         |
| Materialized view |              36 |                 1 |         Yes         |
| Trigger summary   |              72 |                 2 |          No         |

---

### 6. Duplicate delivery of the same external payment reference

**Action:** Insert a duplicate reference for the payment of 36.

| Approach          | Captured Amount | Captured Payments | Duplicate Prevented? | Matches Base Query? |
| ----------------- | --------------: | ----------------: | :------------------: | :-----------------: |
| Base query        |              72 |                 2 |          No          |      Reference      |
| SQL function      |              72 |                 2 |          No          |         Yes         |
| Materialized view |              36 |                 1 |          No          |          No         |
| Trigger summary   |             108 |                 3 |          No          |          No         |

---

# Materialized View Staleness Test

## Before Source Data Changes

At the initial setup:

| Approach          | Captured Amount | Captured Payments |
| ----------------- | --------------: | ----------------: |
| Base query        |              36 |                 1 |
| SQL function      |              36 |                 1 |
| Materialized view |              36 |                 1 |
| Trigger summary   |              36 |                 1 |

## After Inserting/Changing Source Data Without Refreshing

Using the state after the captured payment was inserted:

| Approach          | Captured Amount | Captured Payments | Fresh? |
| ----------------- | --------------: | ----------------: | :----: |
| Base query        |              72 |                 2 |   Yes  |
| SQL function      |              72 |                 2 |   Yes  |
| Materialized view |              36 |                 1 |   No   |
| Trigger summary   |              72 |                 2 |   Yes  |


## After Refreshing the Materialized View

```sql
REFRESH MATERIALIZED VIEW daily_captured_revenue;
```

After the refresh:

| Approach          | Captured Amount | Captured Payments | Fresh? |
| ----------------- | --------------: | ----------------: | :----: |
| Base query        |              72 |                 2 |   Yes  |
| SQL function      |              72 |                 2 |   Yes  |
| Materialized view |              72 |                 2 |   Yes  |
| Trigger summary   |             108 |                 3 |   No   |

---

# Rebuild Trigger Summary

After rebuilding the trigger summary:

| Approach          | Captured Amount | Captured Payments |
| ----------------- | --------------: | ----------------: |
| Base query        |              72 |                 2 |
| SQL function      |              72 |                 2 |
| Materialized view |              72 |                 2 |
| Trigger summary   |              72 |                 2 |


---
