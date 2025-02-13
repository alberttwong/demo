### Demo of StarRocks using Delta Lake External Catalog on MinIO + HMS

![StarRocks Technical Overview](https://github.com/StarRocks/demo/assets/749093/aec4ee69-8b1c-49f4-814d-b90ad4a06f70)
![Screenshot 2024-03-04 at 4 15 00 PM](https://github.com/StarRocks/demo/assets/749093/38899ca4-d5e8-4b40-befa-878fb71d63d5)


1. Start the environment

`docker compose up --detach --wait --wait-timeout 60`

2. Create the bucket for Delta Lake files

Go to http://localhost:9000/ and login with admin:password and create the bucket `warehouse`

3. Run the Spark SQL code to insert data

Log into the spark container. Please note that there are spark defaults already set via conf files and run the following to set additional spark configs.

```
yum install -y python3
spark-sql --packages io.delta:delta-core_2.12:2.0.0 \
--conf "spark.sql.extensions=io.delta.sql.DeltaSparkSessionExtension" \
--conf "spark.sql.catalog.spark_catalog=org.apache.spark.sql.delta.catalog.DeltaCatalog" \
--conf "spark.sql.catalogImplementation=hive"
```

```
CREATE SCHEMA delta_db LOCATION 's3://warehouse/';

create table delta_db.albert (number Int, word String) using delta location 's3://warehouse/dl_albert';

insert into delta_db.albert values (1, "albert");
```

4. Have StarRocks connect to Delta Lake on S3

```
mysql -P 9030 -h 127.0.0.1 -u root --prompt="StarRocks > "
```
```
CREATE EXTERNAL CATALOG deltalake_catalog_hms
PROPERTIES
(
    "type" = "deltalake",
    "hive.metastore.type" = "hive",
    "hive.metastore.uris" = "thrift://hive-metastore:9083",
    "aws.s3.use_instance_profile" = "false",
    "aws.s3.access_key" = "admin",
    "aws.s3.secret_key" = "password",
    "aws.s3.region" = "us-east-1",
    "aws.s3.enable_ssl" = "false",
    "aws.s3.enable_path_style_access" = "true",
    "aws.s3.endpoint" = "http://minio:9000"
);
set catalog deltalake_catalog_hms;
show databases;
use delta_db;
show tables;
```
