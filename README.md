## Auto Scaling
* Launch configuration: a template used by ASG to launch new instance. Stuff like ami, instance type, ip address, storage volume, SG etc... Similar steps as to when creating a new EC2 instance. 
* Auto Scaling Group: defines the desired capacity of group using scaling policies. And where the group should place resources such as which AZ. 
* ELB can be associated with ASG to attach/detach instances based on scaling policy
* According to [some exam questions](http://vceguide.com/what-advise-would-you-give-to-the-user/), looks like the asg can be scheduled up to a month in future. 
* ASG will terminate the instance first and then launches a new instance.
* We can use `as-setinstance-health` command from CLI to manually set an instance back to health, Auto Scaling will throw an error if the instance is already terminating or else it will mark it healthy
* if an ASG failed to launch an single instance for more than 20 hours, it will suspend the scaling process.
* AZ-rebalance will try to launch new instance then terminate old one, so if termination is suspended by user, ASG could grow up to 10% larger than its max size because ASG allow this temporarily during rebalancing activities
* ASG lifecycle hooks
  * You can use Amazon CloudWatch Events, Amazon SNS, or Amazon SQS to receive the notifications when the instance enters a wait state. 
  * default wait time is one hour. The maximum amount of time that you can keep an instance in a wait state is 48 hours or 100 times the heartbeat timeout, whichever is smaller.
  * At the conclusion of a lifecycle hook, the result is either ABANDON or CONTINUE.
  * cooldowns or health check grace period starts after the lifecycle hook complete. 

## ECS(EC2 Container Service)
* clusters can only scale in a single region.
* Instances within ECS cluster have Docker daemon and ECS agent so that the ECS command can be translated to Docker commands. 
* ECS `task definitioin`(json format) is like ASG for ec2 defining variables for the cluster. The cluster uses IAM role to access required resources.
* Max 10 tasks per instance(host). Max 10 containers per task definition. One LB per service.  

## Elastic BeanStalk
* A service to help you deploy web application
* the application version typically points to some artifacts in S3.
* All in all, it takes the uploaded Web application code and automatically provision and deploy the appropriate resources to make the web application operational.
* Java/tomcat, .net/IIS, Go, php, python, ruby, nodejs, docker

## Aws Batch
* Job-> executable/shell-scripts etc
* Job definitions -> specific params for jobs like cpus/data volumes/iam roles etc
* job queue -> scheduled jobs are placed into a job queue until they run. 
* job scheduling -> takes care of when a job should be run and from which Env, typically FIFO. 
* compute env -> ecs or unmanaged Env(managed/maintain ourself). 

## Lightsail
* a Virtual Private Server backed by aws. Resembles EC2 but simpler and easier, good for small scale use cases.
  * Pick a instance, add a launch script(sh), key-pair, instance plan(pay). AZ(optional), 

## S3
* multi-regions. Objects are copied within region across different AZs. Bucket-name cannot be changed. 
* object-size: **0-5TB**. Single upload put limit: **5GB**.
* Maximum number of S3 buckets per aws account: **100**.
* Amazon S3 bucket names are globally unique, regardless of the AWS Region in which you create the bucket.
* Access control
  * IAM policy -> user/role
  * ACL(Access control list) which is legacy -> add permissions on individual objects. 
  * bucket policy -> add/deny permissions across some or all objects within a single bucket. (higher priority to user/role policy). 
  * query string auth -> share objects via URLs that are valid for a specified period of time. 
* [encrypt content](https://aws.amazon.com/blogs/security/how-to-prevent-uploads-of-unencrypted-objects-to-amazon-s3/). The below values are passed via http header when `put`. 
  * SSE with Amazon managed keys (SSE-S3)
  * SSE with KMS-managed keys, customer maintains a master key rather than encryption key. (SSE-KMS)
  * SSE with customer provided keys. You provide the encryption keys to Amazon, and they encrypt all data with your public key so that ONLY you can only read the data with your private key. This means nobody at Amazon can ever read your files, but you are totally screwed if you lose or damage your key; Amazon cannot help you recover.  (SSE-C), the key/encryption algorithm are provided along with the data when uploading. 
* Storage classes
  ![s3-storage-classes](/images/s3-storage-classes.png?raw=true "types of S3 Strage classes")
  * `Infrequent Access` (Standard - IA) is an Amazon S3 storage class for data that is accessed less frequently, but requires rapid access when needed. Standard - IA offers the high durability, throughput, and low latency of Amazon S3 Standard, with a low per GB storage price and per GB retrieval fee. This combination of low cost and high performance make Standard - IA ideal for long-term storage, backups, and as a data store for disaster recovery.
  * `Reduced Redundancy Storage`: [deprecated according to here](https://mysteriouscode.io/blog/aws-s3-storage-classes-pricing-is-not-what-you-think/). An Amazon S3 storage option that enables customers to store noncritical, reproducible data at lower levels of redundancy than Amazon S3’s standard storage. It provides a highly available solution for distributing or sharing content that is durably stored elsewhere, or for storing thumbnails, transcoded media, or other processed data that can be easily reproduced. 
* versioning(file with same name), life-cycle( days to move lower classes or glacier). Access log. Event integrate with sns/sqs/lambda. Cross region replication. 
* to make files available to CDN, we could make the bucket files public by setting bucket policy:
```
  "Id": "Policy1380877762691",
  "Statement": [
    {
      "Sid": "Stmt1380877761162",
      "Action": [
        "s3:GetObject"
      ],
      "Effect": "Allow",
      "Resource": "arn:aws:s3:::<bucket-name>/*",
      "Principal": {
        "AWS": [
          "*"
        ]
      }
    }
  ]
}
```
* URL pattern can be v-host style or path style
  * virtual host style: http://bucket.s3.amazonaws.com OR http://bucket.s3-aws-region.amazonaws.com
  * path style: `US East (N. Virginia) region endpoint`, http://s3.amazonaws.com/*bucketName*.  `other Region-specific endpoint`, http://s3-*aws-region*.amazonaws.com/*bucketName*
  * for website, it will be http(s)://*bucketName*.s3-website-*aws-region*.amazonaws.com/
* s3 has `read-after-write` consistency for `PUT` of new object, and `eventual` consistency for `PUT` on existing object and `Delete` http request.
* Minimum object size for S3-IA is 128KB. 
* add `x-amz-website-redirect-location` for website/page redirect. 
* in a versioned bucket, Only the **bucket owner** can delete a specified object version.
* `S3 Transfer Acceleration` is especially useful in cases where your bucket resides in a Region other than the one in which the file transfer was originated.

## EBS
* snapshot store data on volumes in S3 which is replicated to multiple AZs. EBS volumes are replicated within a specific AZ, snapshots are tied to the region. Snapshots can be shared across regions. Snapshots are incremental and asynchronous. 
* HDD[thruput optimized(ETL etc...) or cold], SSD, Magnetic volumes 
* max volume **16TB**, max volume 5000, max snapshot 10000.
* Data stored on EBS volumes is automatically and redundantly stored in multiple physical volumes in the same availability zone as part of the normal operations of the EBS service at no additional charge.
* Provisioned IOPS must be 4G-16Tb, 4G-16Tb   at most 50:1 (5000IOPS: 100G) or 640GB up with 32000 IOPS . 
* use `dd`(disk duplicator) command to [prewarm the volume](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/disk-performance.html). Prewarm are not needed for new volumes, only for volumes restored from snapshot.
* Recommended queue length is 1 per 200 IOPS. 
* IOPS is measured at 256KB or less as one for SSD and 1024KB for HDD. When small I/O operations are physically contiguous, Amazon EBS attempts to merge them into a single I/O up to the maximum size. For example, for SSD volumes, a single 1,024 KiB I/O operation counts as 4 operations (1,024÷256=4), while 8 contiguous I/O operations at 32 KiB each count as 1operation (8×32=256). However, 8 random I/O operations at 32 KiB each count as 8 operations. Each I/O operation under 32 KiB counts as 1 operation. 
* RAID 5 and RAID 6 are not recommended for Amazon EBS because the parity write operations of these RAID modes consume some of the IOPS available to your volumes

## AWS storage gateway
* a hybrid storage service that enables your on-premises applications to seamlessly use AWS cloud storage.
* It could run on premise as a  virtual appliance that can be used to cache S3 locally at a customers site.

## SQS
* default visibility timeout 30s. Max 12 hours.  
* Msg retention period: 4 days(1 min to 2 weeks). Max msg size 256K. Receive msg wait time 0s. 
* An SQS request can contain up to TEN (10) individual messages, as long as he total size of the request does not exceed 256KB.  
* Message order not guaranteed.  
* When the message visibility timeout expires, the message becomes available for processing by other EC2 instances. The maximum `VisibilityTimeout` of a message is 12 hours and default is 30s.
* Most of time,  short poll is not ideal since it does not query all the servers. Long polling default timeout: `20s`.  
  * `ReceiveMessageWaitTimeSeconds` is an attribute of queue, and `WaitTimeSeconds` is a parameter when doing ReceiveMessage call. WaitTimeSeconds has higher priority. 
* A FIFO SQS queue will end with the `.fifo` suffix.
* The queue can be deleted by aws if inactive for [30 day](https://forums.aws.amazon.com/ann.jspa?annID=2532). 
* You can manage Amazon SQS messages with Amazon S3. This is especially useful for storing and consuming messages with a message size of up to 2 GB. To manage Amazon SQS messages with Amazon S3, use the Amazon SQS Extended Client Library for Java
* use `backlog per instance` metric to auto scale ec2s. 

## SNS
* Fully Managed Push message service. (email/sms/email)  
* A request can be up to 256kb(xml/json/unfomratted). 
* `CreatePlatformEndpoint` call to upload token and register 

## SWF(simple workflow service)
* A full managed State Tracker and Task Coordinator. Components: Task, Marker, Timer, Signal. 

## CloudTrail
* A web service that records API calls made on your account and delivers log files to your amazon s3 bucket 
* An event has info about: Logs, the identity of the caller, the time of the call, the source IP, the request params, the response elements returned by the AWS service
* delivers an event within 15 minutes of the API call. Logs file every 5 minutes. 

## Ops works
* A configuration management service that enables you and operate applications of all shapes and sizes using **Chef**.
* A stack is a container or group of resources such as ELBs, EC2 instances, RDS instances etc. 
* a layer exists within a stack and consists of things like a web application layer, application processing layer or database layer. 
  * recipes are applied based on layer. 
  * an instance must be assigned to at least 1 layer.
* Create up to 40 stacks, each stack can hold up to 40 layers, 40 instances and 40 apps. 

## EC2
* By default, you can only have **5** Elastic IP addresses per region.
* You are billed instance-hours as long as your EC2 instance is in a running state.
* The Elastic IP is a static public IP address that is associated with you Amazon account. When you have an Elastic IP address, you can seamlessly disassociate the IP address from an Instance and re-associate it to another instance. When this occurs, the name of the new instance is automatically mapped in DNS. With a standard static public IP address, there is no seamless transition, this process must be done by the user which creates service downtime. 
* ec2 metadata url: http://169.254.169.254/latest/meta-data/
* we can use IAM roles for temporary credentials in ec2 which has a hidden process to retrieve temp credential automatically. In [java](http://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/java-dg-roles.html), we can use `InstanceProfileCredentialsProvider` to create client without having to get credential from sts manually. With `aws cli`, it should work out of box. Some more explanation [here](http://parthicloud.com/how-to-access-s3-bucket-from-application-on-amazon-ec2-without-access-credentials/). To access the temp credential via command line, run: `curl http://169.254.169.254/latest/meta-data/iam/security-credentials/role_name_goes_here` according to this [post](https://derflounder.wordpress.com/2017/04/27/using-iam-roles-on-amazon-web-services-to-generate-temporary-credentials-for-ec2-instances/), which also provided the piped sed/awk command to extract the accessid/secret.   
* Uptime SLA for EC2/EBS is 99.95% 
* When the user gets an `InsufficientInstanceCapacity` error while launching or starting an EC2 instance, it means that AWS does not currently have enough available capacity to service the user request
* every stop/start will be charged an extra hour, while reboot does not charge. 
* Shared <-> Dedicated are concepts for hardware. Reserved, On-Demand, Spot are for pricing and usage mode. For Dedicated tenancy, reserved instances is also possible.

## ELB
* The ELB  `X-Forwarded-For` request header helps you identify the IP address of a client when you use HTTP/HTTPS load balancer.
* Security policy is a combination of SSL protocols(TSL 1.0/1.1/1.2 SSL 2.0/3.0), SSL ciphers and the Server Order Preference option. 
* Elastic Load Balancer allows using a Predefined Security Policies or creating a Custom Security Policy for specific needs. If none is specified, ELB selects the `latest` Predefined Security Policy.
* When you register an instance with an elastic network interface (ENI) attached, the load balancer routes traffic to the primary IP address of the **iprimary** interface (eth0) of the instance.
* ELB supports Proxy Protocol which user could identify client IP for TCP traffic.

## RDS
* auto backup, stored in S3. Might be slightly delay/latency during backup. 
* snapshot, manually, exist even the RDS instance is gone. 
* encryption cannot be applied to the running instances. 
* multi-AZ, auto failover to the standby without change connection string etc. for **Disaster only**.  Amazon RDS automatically provisions and maintains a synchronous standby replica
* read-replica, for read performance/scaling, up to 5 instances and must have the auto-backup turned on. Using `async` replication. Same AZ.  
* Aurora
  * mysql compatible relational database. 
  * auto scale from 10G to 64TB. 2 copies in each AZ and minimum in 3 AZ, so 6 copies. 
* backup retention period, 1(default) to 35 days. 	 
* user can view error log, slow query log and general log.
* Min IOPS 1000, min storage for IOPS 100G.
* RDS can have cross-region read replica. 

## dynamo
* eventual vs read consistency(auto copy to 3 AZs). 
* 1 write per second is one unit. Good price for read, and not for write. $0.0065 per 10 write/50 read.  
  * Read: 4KB/sec one unit, round to 4KB. For `eventually consistent` the required unit number is half, i.e. divided it by 2. , 
  * Write: 1KB/sec one unit, round to 1KB.
* Easy to scale(push button) comparing to RDS relational. 
* major limitations: 1. 400KB max item size. 2. 10 indexes per table
* The maximum item size in DynamoDB is 400 KB, which includes both attribute name binary length (UTF-8 length) and attribute value lengths (again binary length). The attribute name counts towards the size limit.
* Length of a **partition key** value: 1- 2048 byte, maximum length of a sort key value is 1024 byte.
* A single DynamoDB table partition can support a maximum of 3,000 read capacity units or 1,000 write capacity units.
* DynamoDB uses optimistic concurrency control, and support conditional write
* number of tables and provisioned thruput can be raised by contacting aws support
* You can define a maximum of 5 local secondary indexes and 5 global secondary indexes per table.
  * GSI(Global Secondary Index) is sparse indexes. Unlike the requirement of having a primary key, an item in a DynamoDB table does not have to contain any of the GSI keys. If a GSI key has both hash and range elements, and a table item omits either of them, then that item will not be indexed by the corresponding GSI. In such cases, a GSI can be very useful in efficiently locating items that have an uncommon attribute.
* `scan` operation has a limit of 1MB, use LastEvaluatdKey to apply in a subsequent operation. The thruput is 4KB. 
* dynamoDB stream is useful for replication and trigger. 4 Types of Stream view: (key only, new/old image, new&old image) 
* provisioned read/write units are spread evenly across all partitions. Each partition up to 10 GB and 3000 read and 1000 write unit 


## redshift
* single node, multi-node(1.lead node which manage connections and queries. 2.compute node). 
* column based, advanced compression, only avail in 1 AZ
* default block size for columnar storage is 1M, which is more efficient and further reduces the number of I/O requests needed to perform any database loading or other operations that are part of query execution.
* Auto backup. 

## elasticache
* improve latency and thruput of read heavy.
* Options: Redis and Memcached. Redis is more complex. 
  * Redis has read replica. And auto backup option, persistence, advanced data-type, in-memory sort, pub/sub. 
  * memcached has multi-thread and scale horizontally capability
* You cannot use ElastiCache in a VPC that is configured for dedicated instance tenancy.
* Default cache port: for Memcached 11211 and for Redis 6379.

## Route 53
* difference with cname, alias record can map naked domain name(`example.com`), but cname cannot(it can only map to like www.example.com, server.example.com). And cname get charged but alias does not. 
* In addition to hosting domains, Route 53 serves as a domain registrar
* R53 DNS failover can be integrated with ELB to health check on both elb and the instances behind. So it could direct traffic away from failed elb instance, or failover to other regions when problem.

## VPC
* A VPC spans all the Availability Zones in the region. After creating a VPC, you can add one or more subnets in each Availability Zone. When you create a subnet, you specify the CIDR block for the subnet, which is a subset of the VPC CIDR block. **Each subnet must reside entirely within one Availability Zone and cannot span zones**. Availability Zones are distinct locations that are engineered to be isolated from failures in other Availability Zones. By launching instances in separate Availability Zones, you can protect your applications from the failure of a single location. We assign a unique ID to each subnet. 
* AZ is like a 机房。vpc is like a data-center. Region is like 一个地区。
* good explanation of [difference between public and private subnet](https://stackoverflow.com/a/22212017/1486742). Basically public subnet use `Internet Gateway` as default route and private subnet uses `NAT` as route. So the instance in public subnet with public IP will have internet access since the route will use that IP directly. Inbound wise, with public IP, it could also be accessed if SG/ACL allows. For instance in private subnet, instance-initiated outbound internet access is possible via NAT. However, outside-world-init inbound access cannot be done since there is no public IP associate to the private subnet instance.
* To create a public subnet: 
  * create a internet gateway and attached to the custom vpc(one IGW for 1 vpc)
  * create a new route table so that we do not change the default `main` route table. Add the ip range(0.0.0.0/0) below the `local` and use the IGW(internet gateway) as target. This means all destination other than our local subnet ip addresses will go to internet via IGW. We do not use main otherwise all subnet will have internet access. 
  * associate subnet to the route table we just created in the `Subnet Association` tab in this route table. 
  * for public subnet, we can turn on auto assign public ip  in the `subnet actions` . 
* for NAT instance, we can create a NAT ec2 instance in the public subnet. Then in `actions -> networking -> change source/dest check`, disable it so that it would act as the middle man rather than a source/dest. In the `main` route table, add `0.0.0.0/0` below the local and add the target as the NAT Ec2 instance, which means all subnet other than public will use this NAT to access internet.   
* For NAT Gateway(more preferable, no ec2 maintenance comparing to NAT instance which is a single point of failure). On creation, selection public subnet where this NAT gateway will be deployed to. And then create a Elastic IP. Then in the route table, do the same thing as in the NAT instance.  
* ACL vs SG, ACL is on the subnet level, support allow and deny rule, stateless that return traffic must be explicitly allowed, rules are tested sequentially(smaller first/higher priority). Where as SG is on instance level, support allow rule only, stateful that return traffic is automatically allowed regardless of any rule.  
* One subnet can only associate with one network ACL. We need to add [Ephemeral port](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_ACLs.html#VPC_ACLs_Ephemeral_Ports) to the allow list if client init traffic. 
* vpc log flow allow log all traffic to the vpc into the cloudwatch. 
* vpc peering does not support *edge-to-edge* routing, so settings on vpc-a cannot be shared by vpc-b even they are peered. 
* VPC allows the user to set up a connection between his VPC and corporate or home network data centre `without` IP rage overlapping. 
* If you have multiple VPN connections, you can provide secure communication between sites using the AWS VPN CloudHub. 
* Public IP cannot be assigned to instance with multiple ENIs.  
* [AWS reserves](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Subnets.html#VPC_Sizing) the first 4 IPs and last IP in each subnet's CIDR block. 
* vpn connection between on-prem and vpc requires Hardware VPN Access, on-prem Customer Gateway and a Virtual Private Gateway. 
  * A customer gateway is a physical device or software application on your side of the VPN connection. A CG or route in CG is usually a single point of failure and need redundancy. 
  * A virtual private gateway is the VPN concentrator on the Amazon side of the VPN connection. If you've attached a virtual private gateway to your VPC and enabled route propagation on your route table, routes representing your VPN connection automatically appear as propagated routes in your route table
  * each [vpn connection](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_VPN.html#VPNTunnels) has 2 tunnels, with each tunnel using a virtual private gateway public IP address. By adding another CG, we can provide 2 vpn connections and 4 tunnels for redundancy. The customer gateway IP address for the 2nd vpn connection must be publicly accessible.  
  * VGW is not the initiator; CGW must initiate the tunnels

## IAM
* [IAM Policy Evaluation Logic](http://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html#policy-eval-denyallow). It assumes deny first, and then  evaluates deny then evaluate allow. 
* The distinction between a request being denied by default and an explicit deny in a policy is important. By default, a request is denied, but this can be overridden by an allow. In contrast, if a policy explicitly denies a request, that deny can't be overridden. [good example](http://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html#AccessPolicyLanguage_Interplay)
* limits: 
  * Groups per AWS account: 300
  * Users per AWS account: 5000
  * Roles per AWS account: 1000
  * Groups an IAM user can be a member of: 10

## Cloudfront
* you can write file directly to edge
* object expires in 24hr by default and minimum is 3600s (one hour). You can change the CloudFront settings for Minimum TTL, Maximum TTL, and Default TTL for a cache behavior. 
* invalidate has 3000 object per distribution one time.  
* Create an Origin Access Identity (OAI) for CloudFront and grant access to the objects in your S3 bucket to that OAI and create signed url using java/perl etc... 
* Geo restriction: can be white/black list.
* If CloudFront forwards a request to the origin using the HTTPS protocol, and if the origin server returns an invalid certificate or a self-signed certificate, CloudFront drops the TCP connection.

## Cloudformation
* The **required** `Resources` section declares the AWS resources that you want to include in the stack, such as an Amazon EC2 instance or an Amazon S3 bucket.
* The optional `Conditions` section includes statements that define when a resource is created or when a property is defined. For example, you can compare whether a value is equal to another value. Based on the result of that condition, you can conditionally create resources.
* Use the optional `Parameters` section to customize your templates. Parameters enable you to input custom values to your template each time you create or update a stack. 
* The optional `Outputs` section declares output values that you can import into other stacks (to create cross-stack references), return in response (to describe stack calls), or view on the AWS CloudFormation console. 
* limit: 200 stacks, 60 param and outputs in a single template, 4096 chars for description fields. 
* AWS CloudFormation provides a `WaitCondition` resource which acts as a barrier and blocks the creation of other resources until a completion signal is received from an external source
* Deletion Policy has 3 options: Delete, Retain, Snapshot.
* To get the status of a stack
  * for automation script, poll with the cli `list-stacks`. 
  * for IFTTT systems like cicd system, use the `–notification-arns ` option for `create-stack` to pass message to a sns topic. 
* [custom resource](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-custom-resources.html) can be used to communicate with other system and get info back or create other resources, via SNS or lambda. 
* To run commands on instance at launch, leverage [User Data](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html) with 2 options: shell script or cloud-init directives. In cloudformation template, it can be [passed](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/deploying.applications.html#deployment-walkthrough-lamp-install) with the `Fn::Base64`.

## Cloudwatch
* basic metric every 5 min, and detailed metrics every 1 min. NOTE: no detail support for service other than `RDS/EC2/ASG/ELB/R53`, and ASG is detail by default
* basic ec2 monitoring: cpu, disk, status, network. Memory usage is customise metrics.
* system status check is for Host(phiscal), to fix, start and stop the instance so it runs in a new physical host. And Instance status check is for VM, to fix, reboot/modify OS.
* cloudwatch file can store logs up to 15 month.
* EBS Volume status check, warning(degraded but still functioning), impaired(stalled/Not Available).
* elasticCache, eviction monitoring(redis can only scale out, memcached can out/up). Concurrency monitoring
* when sending data to metrics, if no data in some period, it is still recommended to send `0` instead of no value to monitor the health of the application
* use `statistic-values` parameter for **put-metric-data** when sending aggregate data.  8KB for HTTP GET requests and 40KB for HTTP POST requests
* data can be 2 weeks in the past and 2 hours into the future.
* CloudWatch supports Sum, Min,Max, Sample Data and Average statistics aggregation.
* Log -> CloudWatch Log -> Cloudwatch log filter -> cloudwatch alarm. We can config cw agent in EC2 and send logs to cw log, then we can establish cw log filter to get certain logs and build graph, then we can create criteria so if that is passed, we send alarm.
* cw alarm are sent to SNS or ASG. Alarm will only sent when state changes. 
  * `mon-disable-alarm-actions` to disable all actions for the specific alarms. 
  * `mon-set-alarm-state` command to temporarily changes the alarm state of the specified alarm, `ALARM, OK or INSUFFICIENT_DATA`. 

## DR
* large RTO/RPO to small: 
  * backup&restore, backup data to S3/Glacier, and restore when DR. 
  * Pilot Light(RDS/AD replicated and elastic IP/ENI Or R53+ELB), 
  * Warm Standby, run a mini version live all the time. When DR, scale up/out 
  * Multi-Site. Active-active, both running at the same time. Still write to the main DB. Failover to backup db when DR.

## Snowball
* import/export to S3. Regular snowball(80TB), snowball edge(TB with computation ability), snowball mobile(PB)

## Cloud HSM hardware security modules
* a physical device used within a VPC. And can be used for VPC peering. 
* The service is to protect your encryption keys within HSM. It can be integrated with Apache/S3/RDS/EBS.
* Not fault tolerate. 
* an EC2 instance need to be deployed in the same subnet as the HSM to serve as control instance, and a SG with port 22/3389(SSH/RDP) open to your network.  

## Data pipeline
* AWS Data Pipeline is a web service that helps you reliably process and move data between different AWS compute and storage services, as well as on-premises data sources, at specified intervals.
* By default, an activity retries three times before entering a hard failure state. You can increase the number of automatic retries to 10

## Kinesis
* Data in Kinesis is stored for 24 hours by default and up to 7 days if required. 
* Shard is a measure of capacity, 1000 PUT(PutRecord[s] call) per second. 1MB/s input and 2MB/s output. Specified when steam is created and can be changed dynamically. 
  * partition key tells which shard the data belongs to, specified by applications that putting data into a stream. 
  * the `sequence number` is assigned by Streams after the putRecord[s] call.  
  * data blob is 1MB.
* You may use Kinesis Streams if you want to do some custom processing with streaming data. With Kinesis Firehose you are simply ingesting it into S3, Redshift or ElasticSearch

## OpsWorks
* chef agent + OpsWorks automation engine.  Stack -> Layer -> instance
* An RDS instance can only be associated with one OpsWorks stack. A `stack clone` operation does NOT copy an existing RDS instance.
* life cycle events: setup, configure(execute when instance online/offline on all instances), deploy, undeploy, shutdown.
* `create-deployment` command. 
* `berkshelf` added from chef 11.10 to allow cookbooks from multiple repositories.
* `Databags` are global json objects accessible from within the Chef framework. Can be searched.

## lambda
* lambda currently have a 1000 concurrent limit on account per region. 
* each lambda container can only handle one request at a time. It only starts to serve other requests [after it frees up](https://stackoverflow.com/questions/45780776/how-does-aws-lambda-serve-multiple-requests). 

## Misc
* AWS support 2 `Virtualizations`: para and Hardware 
* AWS Trusted Advisor: Security Groups - Specific Ports Unrestricted, IAM Use, MFA on Root Account, EBS Public Snapshots, RDS Public Snapshots
* large file transfer: snowball or aws-import/export 
* storage gateway.
  * Gateway cached: storing all the data on s3 and cache frequently-used data locally.
  * Gateway stored: use s3 to backup the data but store locally.
* Killing feature for EFS against EBS is its concurrency, it can be mounted/accessed by multiple ec2 instances at the same time. 
* The maximum size of a tag key is 128 unicode characters.
* Arn format: arn:partition:service:region:account-id:resource-type(/or:)resource. Some resource type does not require region/account-id, so those part will be omitted and show as double/triple colon. 
* By default, temporary security credentials for an IAM user are valid for a maximum of 12 hours, but you can request a duration as short as 15 minutes or as long as 36 hours. For security reasons, a token for an AWS account root user is restricted to a duration of one hour.
* You must create a virtual interface to begin using your AWS Direct Connect connection. You can create a private virtual interface to connect to your VPC, or you can create a public virtual interface to connect to AWS services that aren't in a VPC, such as Amazon S3 and Amazon Glacier. 
* [global vs region vs AZ](http://jayendrapatil.com/tag/regional/)
  ![global vs region vs AZ](/images/AWS-Global-vs-Regional-vs-AZs.png?raw=true "AWS-Global-vs-Regional-vs-AZs")

