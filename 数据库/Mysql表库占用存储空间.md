## Mysql表库占用存储空间

在 information_schema.tables 中，可以看到每张表和该表索引占用的存储空间大小。

```mysql
SELECT
  table_schema,table_name,data_length,index_length
FROM information_schema.tables
WHERE
  table_schema = 'db_name';
```

查看指定数据库  `db_name` 指定表 `table_name` 占用的存储空间大小（MB）

````mysql
SELECT
  round(sum(data_length / 1024 / 1024), 3)  AS data_length_MB,
  round(sum(index_length / 1024 / 1024), 3) AS index_length_MB
FROM information_schema.tables
WHERE
  table_schema = 'db_name'
  AND
  table_name = 'table_name';
````

