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

## RDS

