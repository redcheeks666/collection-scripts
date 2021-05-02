# aliyun-Flink全托管模式下作业开发文档

> 共享集群全托管集群与独享集群部分语法不同。对于共享集群的作业开发，此文档只作为参考。共享集群兼容原生Flink绝大部分语法。
>
> 全托管共享集群Flink镜像版本:1.12
>
> Ververica Platform 版本:3.0.0
>
> 阿里云官方文档：https://help.aliyun.com/document_detail/172759.html?spm=a2c4g.11186623.6.558.383e6f50ZerLo0
>
> 开源社区文档：https://ci.apache.org/projects/flink/flink-docs-release-1.12/

## SQL开发

### DDL语句

#### 创建数据源表

##### Kafka

**kafka-source-example**

```sql
create table kafka_source(  
  attributeName varchar,
  attributeName varchar,
  attributeName varchar,
  attributeName varchar,
  attributeName varchar        
) with (
  'connector' = 'kafka',
  'topic' = '<yourTopicName>',
  'properties.bootstrap.servers' = '<yourKafkaBrokers>',
  'properties.group.id' = '<yourKafkaConsumerGroupId>',
  'format' = 'json',
  'scan.startup.mode' = 'earliest-offset'
);
```

##### 获取kafka元数据语法

> 社区文档：https://ci.apache.org/projects/flink/flink-docs-release-1.12/dev/table/connectors/kafka.html#available-metadata

| Key              |                  Data Type                   |                         Description                          |  R/W  |
| :--------------- | :------------------------------------------: | :----------------------------------------------------------: | :---: |
| `topic`          |              `STRING NOT NULL`               |               Topic name of the Kafka record.                |  `R`  |
| `partition`      |                `INT NOT NULL`                |              Partition ID of the Kafka record.               |  `R`  |
| `headers`        |        `MAP<STRING, BYTES> NOT NULL`         |      Headers of the Kafka record as a map of raw bytes.      | `R/W` |
| `leader-epoch`   |                  `INT NULL`                  |        Leader epoch of the Kafka record if available.        |  `R`  |
| `offset`         |              `BIGINT NOT NULL`               |         Offset of the Kafka record in the partition.         |  `R`  |
| `timestamp`      | `TIMESTAMP(3) WITH LOCAL TIME ZONE NOT NULL` |                Timestamp of the Kafka record.                | `R/W` |
| `timestamp-type` |              `STRING NOT NULL`               | Timestamp type of the Kafka record. Either "NoTimestampType", "CreateTime" (also set when writing metadata), or "LogAppendTime". |  `R`  |

**示例**

```sql
CREATE TABLE KafkaTable (
  `event_time` TIMESTAMP(3) METADATA FROM 'value.source.timestamp' VIRTUAL,  -- from Debezium format
  `origin_table` STRING METADATA FROM 'value.source.table' VIRTUAL, -- from Debezium format
  `partition_id` BIGINT METADATA FROM 'partition' VIRTUAL,  -- from Kafka connector
  `offset` BIGINT METADATA VIRTUAL,  -- from Kafka connector
  `user_id` BIGINT,
  `item_id` BIGINT,
  `behavior` STRING
) WITH (
  'connector' = 'kafka',
  'topic' = 'user_behavior',
  'properties.bootstrap.servers' = 'localhost:9092',
  'properties.group.id' = 'testGroup',
  'scan.startup.mode' = 'earliest-offset',
  'value.format' = 'debezium-json'
);
```

#### 创建数据结果表

##### PostgreSQL

```sql
create table rds_sink(
  columnName INT,
  columnName INT,
  columnName VARCHAR,
  columnName VARCHAR
) with (
   'connector' = 'jdbc',
   'table-name' = 'tableName',
   'url' = 'jdbc:postgresql://<host>:<port>/<database>?stringtype=unspecified',
   'username' = '<username>',
   'password' = '<password>',
   'driver'='org.shaded.postgresql.Driver' 
    --由于打包的连接器进行了shaded打包,其中org.postgresql.Driver已经重命名为org.shaded.postgresql.Driver。
    --所以必须指定驱动的className
);
```

#### 创建数据维度表

##### PostgreSQL

```sql
CREATE TABLE rds_dim (
  id1 INT,
  id2 VARCHAR
) WITH (
  'connector' = 'jdbc',
  'password' = '<yourPassword>',
  'tableName' = '<yourTablename>',
  'url' = '<yourUrl>',
  'userName' = '<yourUsername>',
  'cache' = 'ALL'
);
```

### DML语句

#### Insert语句

```sql
BEGIN STATEMENT SET;      --写入多个Sink时，必填。
INSERT INTO blackhole_sinkA 
  SELECT UPPER(name), sum(score) 
  FROM datagen_source 
  GROUP BY UPPER(name);
INSERT INTO blackhole_sinkB 
  SELECT LOWER(name), max(score) 
  FROM datagen_source 
  GROUP BY LOWER(name);
END;      --写入多个Sink时，必填。
```

#### Query语句

> 社区文档：https://ci.apache.org/projects/flink/flink-docs-release-1.12/dev/table/sql/queries.html

### Example

```sql
CREATE TEMPORARY TABLE kafka_XXXXXXXXXXX_source
(
    columnName       varchar,
    columnName       varchar
    eventTime VARCHAR(3) METADATA FROM 'timestamp' VIRTUAL 
)WITH(
    'connector' = 'kafka',
    'topic' = 'XXXXXXXXXXX',
    'properties.bootstrap.servers' = '192.168.0.213:9092',
    'properties.group.id' = 'flink_channel_v01_XXXXXXXXX',
    'format' = 'json',
    'scan.startup.mode' = 'earliest-offset',
    'value.format' = 'debezium-json'
 );

CREATE TEMPORARY TABLE ods_XXXXXXXX_sink
(
    columnName varchar,
    columnName varchar,
    columnName varchar
 ) WITH(
   'connector' = 'jdbc',
   'table-name' = 'public.XXXXXXXXXXXX',
   'url' = 'jdbc:postgresql://pgm-bp15se465c3cmau8168190.pg.rds.aliyuncs.com:1921/ods?stringtype=unspecified',
   'username' = 'ods',
   'password' = 'cg9vULcv9HqoPVmwjTKL',
   'driver'='org.shaded.postgresql.Driver'
);

INSERT INTO ods_XXXXXXXX_sink
SELECT t1.columnName as pg_columnName,
       t1.columnName as pg_columnName,
       t1.columnName as pg_columnName,
       t1.columnName as pg_columnName,
       t1.columnName as pg_columnName
from kafka_XXXXXXXXXXX_source t1; 

```

### 自定义Connector

由于此阿里云Flink版本支持连接器有限，如需作业连接至PostgreSQL,Redis,或ES,需自定义连接器。

部分连接器社区版支持情况下，可从源码中抽取对应Module。

#### 管理自定义连接器

> 阿里云文档：https://help.aliyun.com/document_detail/193520.html?spm=a2c4g.11186623.6.668.69cfb9881ZSttu

为了避免JAR包依赖冲突，您需要注意以下几点：

- Flink镜像和Pom依赖的Flink版本请保持一致。
- 请不要上传Runtime层的JAR包，即在依赖中添加`<scope>provided</scope>`。
- 其他第三方依赖(除了Flink的依赖都属于第三方)请采用Shade方式打包，Shade打包详情参见[Apache Maven Shade Plugin](https://maven.apache.org/plugins/maven-shade-plugin/index.html)。

Shade打包方式参考示例：

> 将第三方postgresql驱动包重命名为org.shaded.postgresql

```xml
		<build>
			<plugins>
				<plugin>
					<groupId>org.apache.maven.plugins</groupId>
					<artifactId>maven-shade-plugin</artifactId>
					<version>3.1.1</version>
					<configuration>
						<!-- put your configurations here -->
						<relocations>
							<relocation>
								<pattern>org.postgresql</pattern>
								<shadedPattern>org.shaded.postgresql</shadedPattern>
							</relocation>
						</relocations>
					</configuration>
					<executions>
						<execution>
							<phase>package</phase>
							<goals>
								<goal>shade</goal>
							</goals>
						</execution>
					</executions>
				</plugin>

			</plugins>
		</build>
```

### 自定义UDF

> 阿里云文档：https://help.aliyun.com/document_detail/188052.html?spm=a2c4g.11186623.6.617.1ca1906fPguXSq

### 特殊字符转义问题记录

关于flink sql中字段包含特殊字符的相关转义   

| 原字段           | 转义语法                             |
| ---------------- | ------------------------------------ |
| &#96;column&#96; | &#96;&#96;&#96;column&#96;&#96;&#96; |
| \__column__      | &#96;\__column__&#96;                |
|                  |                                      |

### Offsets读取位置问题记录

#### kafka相关配置

**auto.offset.reset**
	**earliest**：如果groupid没有提交过offset，则从最早偏移量读取。否则从提交过的offset处读取。

​	**latest**：如果groupid没有提交过offset，则从最新偏移量读取。否则从提交过的offset处读取。

#### Flink相关配置

**scan.startup.mode**

​	**earliest-offset：**从Kafka最早分区开始读取。

​	**latest-offset：**从Kafka最新位点开始读取。

​	**group-offsets（默认值）：**根据Group提交的offset读取。

​	**timestamp：**从Kafka指定时间点读取。需要在WITH参数中指定scan.startup.timestamp-millis参数

​	**specific-offsets：**从Kafka指定分区指定偏移量读取。需要在WITH参数中指定scan.startup.specific-offsets参数

> Flink 会先根据`scan.startup.mode`来从相应位置读取，如若找不到对应位移。则根据`auto.offset.reset`配置读取对应位移。





