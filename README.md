# KeywordAnalysis
Word analysis, by domain, on the Common Crawl data set for the purpose of finding industry trends


//From Nelson - March 24th, 2017
As discussed, I have done remote copy for IBM domain and subdomain to our S3 folder. The data are compressed html raw data and old data (2012). You could download it and view the source pages.

C:\Users\jiaon>aws s3 ls s3://jiaon01/common_crawl/
                           PRE ibm-developerworks
                           PRE ibm_cloud_computing
                           PRE ibm_storage/
                           PRE ibm_systems/
                           PRE netapp/
                           PRE wordcount-ibm-storage
                           PRE www-01.ibm.com
                           PRE www-03.ibm.com
                           PRE www.ibm.com/   (3GB)

I also ran a spark word count job for ibm_storage, the result is in s3://jiaon01/common_crawl/wordcount-ibm-storage. You could download it.
Attached please find the process I have done for these data capturing and word count. Sorry for the unorganized format.

Top 10 words
scala> sorted_counts.take(10).foreach(println)
(,28055)
(<!--,4947)
(ibm,4676)
(-->,4403)
(<div,2985)
(</div>,2472)
(storage,2125)
(and,1926)
(<li><a,1850)
(/>,1788)



//Nelson - March 27th, 2017
Successfully ran word count for corpus of IBM.com & NetApp.com domain. Attached please find IBM top 60,000 & NetApp top 20,000 word count results. Whole word count results output could be found: 
s3://CommonCrawl/wordcount-ibm.com  & s3://CommonCrawl/wordcount-netapp.com

To process IBM large file, I created customized large instances (r3.4xlarge) cluster with some configuration setting. I have attached the processes, if you could start EMR cluster properly, you may could run the spark word count process.


