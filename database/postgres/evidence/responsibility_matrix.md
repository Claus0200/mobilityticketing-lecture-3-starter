# Responsibility Matrix

| Responsibility             | Direct query                                    | SQL function                                      | Materialized view                                                      | Trigger summary                                                                                           |
| -------------------------- | ----------------------------------------------- | ------------------------------------------------- | ---------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| **Correctness**            | High | High        | Correct as of the last refresh, but can become stale                   | Limited, because i didn't handle `UPDATE`, `DELETE` |
| **Freshness**              | Immediate                                       | Immediate                                         | Depends on when the view is refreshed                                  | Immediate for `INSERT` operations                                                               |
| **Write cost**             | Low             | Low               | Low, refresh adds additional work | Higher, since payment inserts also update the summary table                                                    |
| **Read cost**              | High, performs joins and aggregation         | High, performs the same calculation | Low, reads precomputed results                                        | Low,  reads precomputed results                                                                           |
| **Hidden side effects**    | Low                                             | Low                                               | Low                                                                    | High, inserting a payment modifies another table through the trigger                       |
| **Rebuildability**         | Good     | Good       | Good, can be rebuilt by refreshing from the base tables               | Good                                                |
| **Operational complexity** | Low                                             | Low                                               | Medium — requires a refresh strategy                                   | High, trigger logic must handle every relevant manipulation correctly                                        |

## Recommendation

However, I would still choose the SQL function because it is easier to maintain and provides immediate freshness. This is useful when the revenue calculation needs to reflect the current data accurately. A materialized view is more suitable when precomputed data is acceptable and the information does not need to be updated immediately.
