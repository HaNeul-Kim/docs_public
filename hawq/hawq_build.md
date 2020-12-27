# Apache HAWQ build and install (includes PXF)

https://github.com/apache/incubator-hawq

Apache Hadoop 2.7.2 버전 위에 HAWQ 2.0 dev 버전을 resource manager 를 yarn 으로 설치하고 HAWQ 에서 hdfs 와 web 의 파일을 읽어서 external table 을 만드는 것을 목표로 합니다.  
Apache HAWQ 를 build, install 하는 순서와 명령어를 아주 자세하게 적어놓았습니다.  
hadoop install 방법은 포함하지 않았고 pxf install 을 포함하고 있습니다.  
먼저 챕터별 전체 실행 스크립트를 적어놓았고 그 뒤에 부분별 실행 스크립트를 적어놓았고 그 다음에 실행 한 결과 로그를 적어놓았습니다.  
문서는 asciidoc 으로 작성하였으며 repository 내에 asciidoc 파일 및 pdf 파일도 포함되어 있습니다.  
잘못된 점이 있으면 hskimsky@gmail.com 으로 메일 주시기 바랍니다.  
빌드 성공을 기원합니다.

## Requirements for this build

* CentOS 6.7
* Python 2.6.6
* Oracle JDK 1.7.0_80
* Apache Maven 3.3.9
* Apache Hadoop 2.7.2

## ROLE

|FQDN|ROLE|
|:---:|:---:|
|hawq1.apache.local|Namenode, Secondary Namenode, ResourceManager, HAWQ Standby, PXF|
|hawq2.apache.local|HAWQ Master, PXF|
|hawq3.apache.local|Datanode, HAWQ Segment, PXF|
|hawq4.apache.local|Datanode, HAWQ Segment, PXF|

## Environment Variables
* export JAVA_HOME=/usr/local/java/default
* export M2_HOME/usr/local/maven/default
* export HADOOP_HOME=/usr/local/hadoop/default
* export HADOOP_MAPRED_HOME=${HADOOP_HOME}
* export HADOOP_COMMON_HOME=${HADOOP_HOME}
* export HADOOP_HDFS_HOME=${HADOOP_HOME}
* export YARN_HOME=${HADOOP_HOME}
* export HADOOP_COMMON_LIB_NATIVE_DIR=${HADOOP_HOME}/lib/native
* export HADOOP_OPTS="-Djava.library.path=${HADOOP_HOME}/lib"
* export HADOOP_CLASSPATH=${JAVA_HOME}/lib/tools.jar
* export CLASSPATH=${HADOOP_HOME}/share/hadoop/common/hadoop-common-2.7.2.jar:${HADOOP_HOME}/share/hadoop/mapreduce/:${HADOOP_HOME}/share/hadoop/common/lib/commons-cli-1.2.jar
* export PATH=${JAVA_HOME}/bin:${M2_HOME}/bin:${HADOOP_HOME}/bin:${HADOOP_HOME}/sbin:${PATH}

## Precondition

* Linux machine 은 이미 4대가 구성되어 있다고 가정합니다.
* Java, Maven, Hadoop 은 위 환경변수에 맞게 이미 설치되어 있다고 가정합니다.
