---
published: true
---

We recently had a large influx of traffic on one of our websites that tested the durability of our RDS instance. One afternoon at around **17:00** I received an alert email from Amazon Cloudwatch noting that  one of the websites we maintain had an average latency of about 1-2 seconds.

![slow response time]({{site.baseurl}}/_posts/slow_responsse.png)
_Response Time as measured at the load balancer_


I jumped onto the website and confirmed it was at an absolute crawl. 

I immediately checked my Postgres database for any abnormalities. The write latency was 200-300ms. 

![slow write latency rds]({{site.baseurl}}/_posts/write_latency_rds.PNG)
_Write latency of the Postgres RDS database_

Checking the CPU I could see that the CPU baseline had definately changed.

![RDSCPU_2.png]({{site.baseurl}}/_posts/RDSCPU_2.png)

I've experienced this in the past (high write latency and a higher CPU baseline) with a non-production system. A database restart fixed it in that case. 

I contacted the Amazon Support Team with some screenshots of the graphs and a summary of the problem. They advised that the database had exceeded its IOPS quota and was effectively throttled. Admittedly I didn't know a ton about how IOPS was handled in RDS (first mistake).

IOPS is a unit of measurement of input/output operations per second (think of them as 'activity' on your device). You only have a limited number of these IOPS when utilising EBS backed services such as EC2 and RDS.The amount of IOPs that you have is influenced by how much storage you have. For example, my RDS instance had 100gb of storage which meant that Amazon allows my instance 300 IOPs. If the instance exceeds that 300 IOP allowance, Amazon will 'burst' and allow up to 3000 IOPS for a short period of time. If you then run out of those 3000 IOPS, well, you are throttled back down to 300 IOPS. 

You can see below that I was using a huge amount of IOPS, especially leading up to the 17:00

![High usage IOPS on RDS]({{site.baseurl}}/_posts/iops.png)

I was curious on how we could better manage this and my first thought was to create an alarm. I could have certainly added one around the IOPS but I found another useful metric called **Burst Balance** which let me know how much I have eaten into my burst quota, and how close I was to being throttled.

Here is what the burst balance looked like at the time in question:

![burstbalance.png]({{site.baseurl}}/_posts/burstbalance.png)

It's pretty clear why we were throttled - we ran out of burst quota.

**How to resolve**
* I restarted my RDS which reset my burst and IOPs quota
* Luckily I knew exactly what was causing these heavy spikes (refreshing a Postgres Matview too frequently) so I was able to address the high IOPs and further tune our usage
* I could have also increased our storage to increase our IOPS quota - for example if I upgraded the RDS to 400gb of storage I would have 1200 IOPS as a baseline

**For the future**
* I needed to start monitoring IOPs a whole lot better
* I added a few extra Cloudwatch Alarms so we could monitor the issue better and be warned when the IOPs start to creep up quickly
  * Added alarms for 
    * RDS Read/Write Latency 
    * Burst Quota - 80%, 50%, 20% alarms
  





