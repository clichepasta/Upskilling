# SQL vs. NoSQL (When to Use What)

| Feature | Relational (SQL) | Non-Relational (NoSQL) |
|---|---|---|
| Schema | Rigid, predefined schema. | Flexible, dynamic (Document, Key-Value, Graph). |
| Scaling | Primarily Vertical (Horizontal requires complex sharding). | Horizontal by design via partitioning. |
| Transactions | ACID compliant (Strict Consistency). | BASE compliant (Eventual Consistency). |
| Joins | Natively supports complex relational joins. | Handled at application level or denormalized. |

## The Interview Pitch

"Choose SQL (e.g., PostgreSQL, MySQL) when data integrity is paramount, financial transactions require ACID compliance, or the access patterns are unpredictable and require complex relational joins. Choose NoSQL (e.g., DynamoDB, Cassandra, MongoDB) when dealing with massive write/read throughput, unstructured/semi-structured data, and when horizontal scalability is required out of the box with predictable query paths."

## Potential Interview Follow-ups

### Q: "Amazon shifted its core shopping cart and checkout data from Oracle (SQL) to DynamoDB (NoSQL). Why?"

**Answer:** At Amazon’s scale, SQL databases bottlenecked on writes and vertical limits. Shopping carts map perfectly to a Key-Value lookup (`customer_id -> cart_items`), requiring sub-millisecond, predictable horizontal scaling over strict relational constraints.
