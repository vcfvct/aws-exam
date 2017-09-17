## Auto Scaling
* Launch configuration: a template used by ASG to launch new instance. stuff like ami, instance type, ip address, storage volumne, SG etc... similar steps as to when creating a new EC2 instance. 
* Auto Scaling Group: defines the desired capacity of group using scaling policie. and where the group should place resources such as which AZ. 
* ELB can be associated with ASG to attach/detach instances based on scaling policy

## ECS(EC2 Container Service)
* clusters can only scale in a single region.
* Instances within ECS cluster have Docker daemon and ECS agent so that the ECS command can be translated to Docker commands. 

## Elastic BeanStalk
* A service to help you deploy web application
* the application version typically points to some artifacts in S3.
* All in all, it takes the uploaded Web application code and automatically provision and deploy the appropriate resources to make the web application operational.

## Aws Batch
* Job-> exeutable/shellscripts etc
* Job definitions -> specific params for jobs like cpus/data volumns/iam roles etc
* job queue -> scheduled jobs are placed into a job queue until they run. 
* job scheduling -> takes care of when a job shoudl be run and from which Env. typically FIFO. 
* compute env -> ecs or unmanged Env(maanged/maintain ourself). 

## Lightsail
* a Virtual Private Server backed by aws. resembles EC2 but simpler and easier, good for small scale use cases.
  * pick a instance, add a launch script(sh), keypair, instance plan(pay). AZ(optional), 

## S3
* multi-regions.  objects are copied within region across different AZs. Bucketname cannot be changed. 
* object-size: **0-5TB**. single upload put limit: **5GB**.
* maximun number of S3 buckets per aws account: **100**.
* Amazon S3 bucket names are globally unique, regardless of the AWS Region in which you create the bucket.
* Access control
  * IAM policy -> user/role
  * ACL(Access control list) which is legacy -> add permissions on indicidual objects. 
  * bucket policy -> add/deny permissions across some or all objects within a single bucket. (higher priority to user/role policy). 
  * query string auth -> share objects via URLs that are valid for a specified period of time. 
* encrypt content
  * SSE with Amazon managed keys (SSE-S3)
  * SSE with KMS-managed keys (SSE-KMS)
  * SSE with customer privoded keys (SSE-C)
* Storage classes
  ![s3-storage-classes](/images/s3-storage-classes.png?raw=true "types of S3 Strage classes")
  * `Infrequent Access` (Standard - IA) is an Amazon S3 storage class for data that is accessed less frequently, but requires rapid access when needed. Standard - IA offers the high durability, throughput, and low latency of Amazon S3 Standard, with a low per GB storage price and per GB retrieval fee. This combination of low cost and high performance make Standard - IA ideal for long-term storage, backups, and as a data store for disaster recovery.
  * `Reduced Redundancy Storage`: an Amazon S3 storage option that enables customers to store noncritical, reproducible data at lower levels of redundancy than Amazon S3’s standard storage. It provides a highly available solution for distributing or sharing content that is durably stored elsewhere, or for storing thumbnails, transcoded media, or other processed data that can be easily reproduced. 
* versioning(file with same name), lifecycle( days to move lower classes or glacier). access log. event integrate with sns/sqs/lambda. cross region replication. 
* to make files available to CDN, we could make the bucket files public by setting bucket policy:
```
{
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

## EBS
* snapshot store data on volumns in S3 which is replicated to multiple AZs. EBS volumns are replicated within a specific AZ, snapshots are tied to the region. snapshots can be shared across regions. 
* HDD[thrput optimized(ETL etc...) or cold], SSD, Magnetic volumns 

## AWS storage gateway
* a hybrid storage service that enables your on-premises applications to seamlessly use AWS cloud storage.
* It could run on premise as a  virtual appliance that can be used to cache S3 locally at a customers site.

## SQS
* default visibility timeout 30s. Max 12 hours.  
* msg retention period: 4 days(1 min to 2 weeks). Max msg size 256K. Receive msg wait time 0s.  
* message order not guarateed.  
* 

## SNS
* Fully Managed Push message service. (email/sms/email)  
* A request can be up to 256kb(xml/json/unfomratted). 
* 

## SWF(simple workflow service)
* A full managed State Tracker and Task Coordinator. Components: Taks, Marker, Timer, Singal. 

## CloudTrail
* A web service that records API calls made on your accound and delivers log files to your amazon s3 bucket 
* An event has info about: Logs, the identity of the caller, the time of te call, the source IP, the request params, the response elements returned by the AWS service
* delivers an event within 15 mins of the API call. logs file every 5 mins. 

## Ops works
* A configuration management service that enables you and operate applications of all shapes and sizes using **Chef**.
* create up to 40 stacks, each stack can hold up to 40 layers, 40 instances and 40 apps. 

## EC2
* By default, you can only have **5** Elastic IP addresses per region.
* You are billed instance-hours as long as your EC2 instance is in a running state.
* The Elastic IP is a static public IP address that is associated with you Amazon account. When you have an Elastic IP address, you can seamlessly disassociate the IP address from an Instance and re-associate it to another instance. When this occurs, the name of the new instance is automatically mapped in DNS. With a standard static public IP address, there is no seamless transition, this process must be done by the user which creates service downtime. 
* ec3 metadata url: http://169.254.169.254/latest/meta-data/
* we can use IAM roles for temporary credentials in ec2 which has a hidden process to retrieve temp credential automatically. In [java](http://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/java-dg-roles.html), we can use `InstanceProfileCredentialsProvider` to create client without having to get credential from sts manually. With `aws cli`, it should work out of box. some more explanation [here](http://parthicloud.com/how-to-access-s3-bucket-from-application-on-amazon-ec2-without-access-credentials/). To access the temp credentail via command line, run: `curl http://169.254.169.254/latest/meta-data/iam/security-credentials/role_name_goes_here` according to this [post](https://derflounder.wordpress.com/2017/04/27/using-iam-roles-on-amazon-web-services-to-generate-temporary-credentials-for-ec2-instances/), which also provided the piped sed/awk command to extract the accessid/secrte.   

## RDS
* auto backup, stored in S3. might be slightly deley/latency during backup. 
* snapshot, maually, exist even the RDS instance is gone. 
* encryption cannot be applied to the running instances. 
* multi-AZ, auto failover to the standby without change connection string etc. for **Disaster only**. 
* read-replica, for read performance/scaling, up to 5 instances and must have the auto-backup turned on. using `async` replication. Same AZ.  
	* Aurora
	* mysql compatible relational database. 
	* auto scale from 10G to 64TB. 2 copies in each AZ and minium in 3 az, so 6 copies. 
	 
## dynamo
* eventual vs read consistency. 
* 1 write per second is one uint. good price for read, and not for write. $0.0065 per 10 write/50 read.  
* easy to scale(push button) comparing to RDS relational. 

## redshift
* single node, multi-node(1. lead node which manage connections and queries. 2.compute node). 
* column based. advanced compresion. only avail in 1 AZ

## elasticache
* improve latency and thruput of read heavy.
* Options: Redis and Memcached. 

## Route 53
* difference with cname, alias record can map naked domain name(`example.com`), but cname cannot(it can only map to like www.example.com, server1.example.com). and cname get charged but alias does not. 

## VPC
* A VPC spans all the Availability Zones in the region. After creating a VPC, you can add one or more subnets in each Availability Zone. When you create a subnet, you specify the CIDR block for the subnet, which is a subset of the VPC CIDR block. **Each subnet must reside entirely within one Availability Zone and cannot span zones**. Availability Zones are distinct locations that are engineered to be isolated from failures in other Availability Zones. By launching instances in separate Availability Zones, you can protect your applications from the failure of a single location. We assign a unique ID to each subnet. 
* AZ is like a 机房。vpc is like a datacenter. region is like 一个地区。
* good explanation of [difference between public and private subnet](https://stackoverflow.com/a/22212017/1486742). Basically public subnet use `Internet Gateway` as default route and private subnet uses `NAT` as route. So the instance in public subnet with public IP will have internet acess since the route will use that IP directly. Inbound wise, with public IP, it could also be accessed if SG/ACL allows. For instance in private subnet, instance-initiated outbound internet access is possbile via NAT. However, ourside-world-init inbound access cannot be done since there is no public IP associate to the private subnet instance.
* To create a public subnet: 
  * create a internet gateway and attached to the custom vpc(one IGW for 1 vpc)
  * create a new route table so that we do not change the default `main` route table. add the ip range(0.0.0.0/0) below the `local` and use the IGW(internet gateway) as target. This means all desitination other than our local subnet ip addresses will go to internet via IGW. We do not use main otherwise all subnet will have internet access. 
  * associate subnet to the route tabel we just created in the `Subnet Association` tab in this route table. 
  * for public subnet, we can turn on auto assign public ip  in the `subnet actions` . 
* for NAT instance, we can create a NAT ec2 instance in the public subnet. then in `actions -> networking -> change source/dest check`, disable it so that it would act as the middle man rather than a source/dest. In the `main` route table, add `0.0.0.0/0` below the local and add the target as the the NAT Ec2 instance, which means all subnet other than public will use this NAT to access internet.   
* for NAT Gateway(more preferable, no ec2 maintanence comparing to NAT instance which is a single point of failure). On creation, selection public subnet where this NAT gateway will be deployed to. And then create a Elastic IP. Then in the route table, do the same thing as in the NAT instance.  
* ACL vs SG, ACL is on the subnet level, support allow and deny rule, stateless that return traffic must be explicitly allowd, rules are tested sequetially(smaller first/higher priority).   where as SG is on instance level, support allow rule only, stateful that return traffic is automatically allowed regardless of any rule.  
* one subnet can only associate with one network ACL. We need to add [Ephemeral port](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_ACLs.html#VPC_ACLs_Ephemeral_Ports) to the allow list if client init traffic. 
* vpc log flow allow log all traffic to the vpc into the cloudwatch. 
