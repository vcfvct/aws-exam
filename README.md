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
* Access control
  * IAM policy -> user
  * ACL(Access control list) -> add permissions on indicidual objects. 
  * bucket policy -> add/deny permissions across some or all objects within a single bucket
  * query string auth -> share objects via URLs that are valid for a specified period of time. 
* encrypt content
  * SSE with Amazon managed keys (SSE-S3)
  * SSE with KMS-managed keys (SSE-KMS)
  * SSE with customer privoded keys (SSE-C)
* Storage classes
  ![s3-storage-classes](/images/s3-storage-classes.png?raw=true "types of S3 Strage classes")
* versioning(file with same name), lifecycle( days to move lower classes or glacier). access log. event integrate with sns/sqs/lambda. cross region replication. 

## EBS
* snapshot store data on volumns in S3 which is replicated to multiple AZs. EBS volumns are replicated within a specific AZ, snapshots are tied to the region. snapshots can be shared across regions. 
* HDD[thrput optimized(ETL etc...) or cold], SSD, Magnetic volumns 

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

##
