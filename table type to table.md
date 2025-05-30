# SQL Query: Querying a PL/SQL Table Type Function

This SQL query demonstrates how to select data from a PL/SQL function that returns a table type. It casts the result of the `XXGTN_HR_Approver_group` function (which presumably returns a `XXGTN_HR_POSITION_HIER_TT` table type) into a selectable table, allowing standard SQL operations like ordering.

```sql
select role_name  from TABLE ( cast( XXGTN_HR_Approver_group(:transactionId) as XXGTN_HR_POSITION_HIER_TT ) ) order by SEQ_NO asc