# KeywordAnalysis
Word analysis, by domain, on the Common Crawl data set for the purpose of finding industry trends

***
## Process
### Specific Domain Data Capturing
#### Common Crawl IBM data capturing
1. Start one node AWS EMR spark cluster
2. SSH to the instance: ec2-54-196-129-41.compute-1.amazonaws.com (change)   user: hadoop 
3. sudo yum -y install git
4. git clone https://github.com/trivio/common_crawl_index
5. export AWS_ACCESS_KEY= (your access key)
6. export AWS_SECRET_KEY= (your secret key)
7. sudo vi /usr/local/lib/python2.7/site-packages/boto/__init__.py
```
def connect_s3(aws_access_key_id=None, aws_secret_access_key=None, host='s3.amazonaws.com', **kwargs)
return S3Connection(aws_access_key_id, aws_secret_access_key, host='s3.amazonaws.com', **kwargs)
```
8. [hadoop@ip-10-43-215-181 bin]$ ./remote_copy check "com.ibm.www"
files: 26045
webpages: 77768
Source compressed file size (MB): 2604500
Destination compressed file size (MB): 3197
[hadoop@ip-10-43-215-181 bin]$ ./remote_copy check "com.netapp.www"
files: 3363
webpages: 5381
Source compressed file size (MB): 336300
Destination compressed file size (MB): 68
9. ./remote_copy copy "com.ibm.www" --bucket jiaon01 –key common_crawl/ibm_storage --parallel 4

### Remove html tags
I have run “dkpro-c4corpus” boilerplate removal code for three days with about 10 r4.4xlarge EMC instances (16vCPU, 122Gb Memory), because it works for me only on small data (20MB-100MB). And sometimes I got error “Exception in thread "main" java.lang.OutOfMemoryError: GC overhead limit exceeded”, even I have already set instance memory as huge, maybe some setting or code issue to look at later. So I have to split IBM.com data to 84 files, each file data processing (boilerplate removal) consumed 0.5~2 hours. Heavy manual work (split, recurring processing, join etc.) were done. Finally, I got plain text (s3://CommonCrawl/ibm_boiler) of IBM.com (data size decreased from 7GB with html tags to 1GB plain text). And ran spark word count for the IBM.com plain text and got the top60000 (attached) and word count results (s3://CommonCrawl/wordcount-output/wordcount-ibm_bolier).

1. Start 1 nodes AWS EMR spark cluster
Advanced option: r4.4xlarge, 1node, spark, Hadoop
Configuration:
```
[{"Classification": "spark", "Properties": {"maximizeResourceAllocation": "true", "spark.executor.memory": "10G", "yarn.nodemanager.pmem-check-enabled": "false", "yarn.nodemanager.vmem-check-enabled": "false"}}] 
```
VPC: your VPC and subnet

2. SSH to the instance: ec2-54-90-80-85.compute-1.amazonaws.com (change)   user: hadoop 

3. sudo vi /etc/spark/conf.dist/spark-defaults.conf
```
spark.driver.maxResultSize       12g
spark.driver.memory              12g
```
4. sudo yum install -y git
5. wget http://mirrors.hust.edu.cn/apache/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz
6. tar zxvf apache-maven-3.3.9-bin.tar.gz
7. sudo vi .bashrc
```
export MAVEN_HOME=/home/hadoop/apache-maven-3.3.9
export M2_HOME=/home/hadoop/apache-maven-3.3.9
export M2=/home/hadoop/apache-maven-3.3.9
export PATH=/home/hadoop/apache-maven-3.3.9/bin:$PATH
```
8. source .bashrc
9. sudo yum install -y git
10. git clone https://github.com/dkpro/dkpro-c4corpus
11. aws s3 cp s3://CommonCrawl/ibm/26279.gz /var/tmp/
12. gunzip 26279.gz
13. split -n84 ibm (26279) (split file to 20-100MB)
14. Sync file between S3 to EC2
```
aws s3 sync . s3://CommonCrawl/data/ibm84
aws s3 sync s3://CommonCrawl/data/ibm84 /var/tmp/.
aws s3 sync s3://CommonCrawl/boilerplate/pending /var/tmp/
aws s3 cp /var/tmp/xba_boiler s3://CommonCrawl/boilerplate/ibm/
```
15. run dkpro-c4corpus-boilerplate
```
cd dkpro-c4corpus/dkpro-c4corpus-boilerplate/
mvn package
java -jar target/dkpro-c4corpus-boilerplate-1.0.1-SNAPSHOT.jar /var/tmp/26279 /var/tmp/ibm_boiler false
```

### Wordcount process
1. spark-shell

IBM Wordcount process:
```Scala
val file = sc.textFile("s3://CommonCrawl/ibm_boiler")
val counts = file.flatMap(line => line.toLowerCase().replace(".", " ").replace(",", " ").split(" ")).map(word => (word, 1L)).reduceByKey(_ + _)
val sorted_counts = counts.collect().sortBy(wc => -wc._2)    (1mins)
sc.parallelize(sorted_counts.take(60000)).saveAsTextFile("s3://CommonCrawl/boilerplate/ibm_boiler _top60000")
sc.parallelize(sorted_counts).saveAsTextFile("s3://CommonCrawl/boilerplate/wordcount-ibm_bolier")
```
Netapp Wordcount process:
```Scala
val file = sc.textFile("s3://CommonCrawl/boilerplate/netapp_boiler")
val counts = file.flatMap(line => line.toLowerCase().replace(".", " ").replace(",", " ").split(" ")).map(word => (word, 1L)).reduceByKey(_ + _)
val sorted_counts = counts.collect().sortBy(wc => -wc._2)    (3mins)
sc.parallelize(sorted_counts.take(20000)).saveAsTextFile("s3://CommonCrawl/top20000_netapp_boiler")
sc.parallelize(sorted_counts).saveAsTextFile("s3://CommonCrawl/wordcount-netapp_boiler")
```
Top 10 words

|Word |Count|
|-----|:---:|
|	|4327791|
|the	|2103578|
|0	|1159355|
|to	|1097568|
|and	|1057336|
|a	|856529|
|of	|811647|
|for	|737729|
|in	|646580|
|ibm	|623663|
