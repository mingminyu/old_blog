# 创建表(CREATE TABLE)


## 1. 引用HDFS数据创建表

很多时候我们手头上有一个数据文件，希望将其快速地在 Impala 中生成数据表，通常会先把该文件上传到 HDFS 目录中，然后执行 SQL 语句加载 HDFS 数据文件到表中。

```SQL
DROP TABLE IF EXISTS test.student_info
;

CREATE EXTERNAL TABLE test.student_info
(
    sid BIGINT,
    name STRING,
    age  INT,
    school STRING
) PARTITIONED BY (`cre_dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
STORED AS TEXTFILE LOCATION '/user/yumingmin/student'
;
```

> 1. 上面创建的是外部分区表，外部表和内部表(即不添加 `EXTERNAL` 关键字)主要区别在于，内部表被 `DROP` 掉时对应的 HDFS 数据文件也会一起被删除掉，而外部表被 `DROP` 掉时是会保留对应的数据文件。
> 2. 如果不是分区表，则可以删除掉 `PARTITIONED BY (`cre_dt` STRING)`。
> 3. `LOCATION` 需要指定的是数据的 HDFS 文件夹路径，而不是具体的文件路径。如果使用 `INPATH` 方式的话，指定的则是具体的 HDFS 文件路径。
>
{style="note"}


当然另外一种方式是使用 `LOAD DATA INPATH` 来加载 HDFS 数据文件，SQL 示例如下:

```SQL
CREATE EXTERNAL TABLE test.student_info
(
    sid BIGINT,
    name STRING,
    age  INT,
    school STRING
) PARTITIONED BY (`cre_dt` STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
STORED AS TEXTFILE
;

LOAD DATA INPATH '/user/yumingmin/student/20240618.csv'
INTO TABLE test.student_info
;
```


创建完表后，还需要执行 `INVALIDATE METADATA` 以及 `COMPUTE STATS` 语句来刷新元数据以及收集统计信息。

```SQL
INVALIDATE METADATA test.student_info;
COMPUTE STATS test.student_info;
```




<seealso>
<category ref="ref_docs">
    <a href="https://impala.apache.org/docs/build/plain-html/topics/impala_create_table.html">CREATE TABLE Statement</a>
</category>
<category ref="ref_github">
</category>
<category ref="ref_issues">
</category>
<category ref="ref_hf">
</category>
<category ref="ref_ms">
</category>
</seealso>





