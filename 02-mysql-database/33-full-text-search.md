# Full-Text Search

## Use Case

Use full-text indexes for efficient keyword search on large text columns.

## Example

```sql
ALTER TABLE articles
ADD FULLTEXT INDEX idx_ft_title_body (title, body);

SELECT id, title
FROM articles
WHERE MATCH(title, body) AGAINST('mysql tutorial');
```

## Next Step

Continue to [33-table-relationship.md](33-table-relationship.md).

