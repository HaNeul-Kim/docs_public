# [HAWQ](http://hawq.apache.org)

* HAdoop With Query
* Apache top level project
* mpp query engine 을 기반으로 하는 pivotal 의 Greenplum 을 기반으로 storage 를 hadoop 을 사용하는 query engine
* greenplum 이 postgresql 8.2 기반이므로 hawq 또한 postgresql 8.2 에서 사용가능한 모든 query 사용 가능하나, hadoop 을 기반으로 하기 때문에, update, delete, index, primary key, foreign key 등이 불가능하여 제약사항이 있음
* DB 라고 생각하기 보다는 sql 을 이용하여 hadoop 의 데이터를 조회할 수 있게 해주는 tool 로 보는것이 맞음
* hadoop 에 존재하는 파일을 sql 을 이용하여 조회가 가능하게 하거나, table 의 데이터를 hadoop 에 저장함
* hawq 로 etl 보다는 MR 의 검증 용도로 주로 사용함

## PXF

* Pivotal eXtention Framework
* hawq 에서 external table 을 조회할 때는 PXF 라는 서비스를 사용함
* pxf 실행시 tomcat 이 실행됨
* pxf 를 사용하여 hdfs, hive, hbase, parquet 등에서 데이터를 조회 가능
