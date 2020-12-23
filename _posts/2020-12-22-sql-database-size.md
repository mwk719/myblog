---
layout: post
title: sql查询数据库表多少和所占空间
date: 2020-12-22
Author: minweikai
tags: [sql]
comments: true
---

```mysql
-- 查询test_database_name表的数量
SELECT COUNT(*) TABLES, table_schema
FROM information_schema.TABLES
WHERE table_schema = 'test_database_name'
GROUP BY table_schema;

-- 查询test_database_name表数据和索引占磁盘空间
select TABLE_NAME, concat(truncate(data_length/1024/1024,2),' MB') as data_size,
       concat(truncate(index_length/1024/1024,2),' MB') as index_size
from information_schema.tables where TABLE_SCHEMA = 'test_database_name'
group by TABLE_NAME
order by data_length desc;
```

