# User Manual: Getting the Next Specific Day of the Week in Oracle SQL

## 1. Purpose

This SQL query is used to find the date of the *next occurrence* of a specific day of the week (e.g., Saturday) based on a given starting date.

## 2. SQL Query

```sql
SELECT NEXT_DAY(TRUNC(sysdate,'IW'),'SATURDAY') FROM dual;
```