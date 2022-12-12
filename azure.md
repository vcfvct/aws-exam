## General
* 54 regions, 140 countries
* HA(High Availability), FT(Fault Tolerate), DA(Disaster Recovery)
  * The difference between FT and HA, is this: A FT environment has no service interruption but a significantly higher cost, while a HA environment has a minimal service interruption.
* Scalability
  * up -> increase size, but need to bring down existing one(interruptive)
  * out -> increase count, non-interruptive
* Elasticity: scale down when traffic/demand decreases.

## CloudShell
* available in powershell and  bash, no package installation allowed as `sudo` is not available.
* `az` and other things like node/python are already there.

## DevOps
* ARM
  * When a user sends a request from any of the Azure tools, APIs, or SDKs, Resource Manager receives the request. It authenticates and authorizes the request. Resource Manager sends the request to the Azure service, which takes the requested action. Because all requests are handled through the same API, you see consistent results and capabilities in all the different tools.
  * 2 deployment mode: *complete and incremental*.
    * In complete mode, Resource Manager deletes resources that exist in the resource group that aren't specified in the template.
    * In incremental mode, Resource Manager leaves unchanged resources that exist in the resource group but aren't specified in the template.
  * ARM is the layer managing infrastructure no matter from cli/port etc.
  * something similar to CloudFormation but without yaml support, json only. Can be nested
  * Source control, Declarative, Idempotent.
  * no similar things like CDK for now for IaS, alternative is Terraform(or CDK to Terraform).
  * Every resource need to be inside a resource group. No nested RG. RG can contain resources from multiple regions.
  * Before deploying an Azure Resource Manager template (ARM template), you can preview the changes that will happen. Azure Resource Manager provides the *what-if operation* to let you see how resources will change if you deploy the template. The what-if operation doesn't make any changes to existing resources.
  * The *ARM template test toolkit* checks whether your template uses recommended practices. When your template isn't compliant with recommended practices, it returns a list of warnings with the suggested changes. By using the test toolkit, you can learn how to avoid common problems in template development.
* Blueprint
  * [Blueprint](https://docs.microsoft.com/en-us/azure/governance/blueprints/overview) often consists of a set of resource groups, policies, role assignments, and ARM template deployments. A blueprint is a package to bring each of these artifact types together and allow you to compose and version that package, including through a continuous integration and continuous delivery (CI/CD) pipeline. Ultimately, each is assigned to a subscription in a single operation that can be audited and tracked.

## Networking
* Virtual Network(VNet) -> VPC, in one region, with one subscription.
  * VNet peering with backbone network. Low Latency, Link Separate Network, Transfer data between subscriptions and deployment models in separate regions.
  * NSG can be associated with Subnet in either VNet or NSG UI.
  * NSG can be associated to a *subnet or network interface*, recommendations is not to both as they could [conflict with each other](https://docs.microsoft.com/en-us/azure/virtual-network/network-security-group-how-it-works#intra-subnet-traffic).
  * CLI: 
    * `az deployment group create -g "YourResourcesGroupName" --template-file azuredeploy.json --parameters "alias=xxx"`
    * `az network vnet show -g "<alias>-rg" -n "vnetName" --query addressSpace`. The `--query` is handy for using a [JMESPath query](https://jmespath.org/) to return specific properties rather than the full response object. This option is available on any command while using az CLI.
    * `az network vnet subnet list -g "<alias>-rg" --vnet-name "vnetName" -o table`. The `-o` option enables you to interact with results in different formats (ex: JSON, table, tsv). It can be combined with the `--query` option even when the output is table instead of JSON.
* Application Gateway -> ALB, Load Balancer -> ELB, 
* CDN: [profile + endpoint](https://docs.microsoft.com/en-us/azure/cdn/cdn-create-new-endpoint). A cdn profile is a container for CDN endpoints and specifies a pricing tier. An endpoint is where we can specify origins/pat/path and other behaviors.
  * Different Tiers including Standard Microsoft, Verizon, Akamai.
  * You can purge content in [several ways](https://learn.microsoft.com/en-us/training/modules/develop-for-storage-cdns/3-control-cache-behavior):
    * On an endpoint by endpoint basis, or all endpoints simultaneously should you want to update everything on your CDN at once.
    * Specify a file, by including the path to that file or all assets on the selected endpoint by checking the Purge All checkbox in the Azure portal.
    * Based on wildcards (*) or using the root (/).
  * Default TTL values are as follows:
    * Generalized web delivery optimizations: 7 days 
    * Large file optimizations: 1 day 
    * Media streaming optimizations: 1 year

* ExpressRoute -> DirectConnect. Connect to on-premises stuff with private connection instead of public Internet.
* A Point-to-Site (P2S) VPN gateway connection lets you create a secure connection to your virtual network from an individual client computer.
* A Site-to-Site (S2S) VPN gateway is good for Dev/test/ lab scenarios and small to medium scale production workloads for cloud services and virtual machines
* A `Local Network Gateway` is an object in Azure that represents your on-premise VPN device. A `Virtual Network Gateway` is the VPN object at the Azure end of the VPN. A `connection` is what connects the Local Network Gateway and the Virtual Network Gateway to bring up the VPN.
* Not all Azure regions support availability zones.

## Compute
* Azure VM
  * availability zone vs availability set
    * availability zones are used to protect applications from entire Azure data center failures, while availability sets are used to protect applications from hardware failures within an azure datacenter.
  * azure will NOT restart more than one *update domain* at one time.
  * Azure Instance Metadata Service (IMDS) endpoint: `http://169.254.169.254/metadata/identity/oauth2/token` to retrieve access token(managed identity) for services like Azure Storage.
* Scale Sets -> ASG
* App Services: PaaS with `Web Apps` for standard app or Containers or APIs, with different run time like node/php/.net etc.
  * use `az webapp list-runtimes --os-type linux` to retrieve the current supported runtime list.
  * multiple apps can be deployed to the same AppService Plan, they will scan together(all apps run on all instances each). Isolate the app into a new AppService Plan when app is resource intensive, or want to scale the app independently, or needs resource in a different geographical region.
  * development slot: deploy app to a staging environment and swap staging and production slots when ready(Standard AppService Plan tier+). Traffic can be routed automatically or manually(by using `x-ms-routing-name` query parameter). After a client is routed to a specific slot, it is "pinned" to that slot for the life of that client session(due to the `x-ms-routing-name` cookie).
    * deployment slot does NOT incur extra instance cost.
  * `Application settings` under `Configuration` defines environment variables for the app.
  * `general settings` under `Configuration` has some common settings like stack/runtime-version/tls etc.
  * auto-scale by 1. metric(cpu/memory/disk-queue/http-queue/data-numberOfBytes in or out) min-duration 5 minutes or 2. schedule.
  * Standard Service Plan+ can do automatic scaling of, not Free-Shared or Basic plan.
  * sequences: 1. create ResourceGroup, 2. create appServicePlan, 3. create webApp,
  * The *Custom Script Extension* V2 downloads and runs scripts on Azure virtual machines (VMs). This extension is useful for post-deployment configuration, software installation, or any other configuration or management task. You can download scripts from Azure Storage or another accessible internet location, or you can provide them to the extension runtime.
    * The Custom Script Extension integrates with Azure Resource Manager templates. You can also run it by using the Azure CLI, PowerShell, or the Azure Virtual Machines REST API.
    * Once the `Custom Script Extension` has been added to a VM, it needs to then be removed before another instance can be run. This is not a difficult operation but when running multiple scripts against a VM is less than desirable. `Run Command` is an alternative and more lightweight method for running scripts against Azure VMs. Multiple instances can be run without the need of any type of clean-up action.
  * use [Azure Hybrid runbook worker](https://docs.microsoft.com/en-us/azure/automation/automation-hybrid-runbook-worker) in Azure Automation for installing services via Azure Powershell scripts stored at Azure Storage.
    * Runbooks in Azure Automation can run on either an Azure sandbox or a Hybrid Runbook Worker.
  * Multiple containers support is only available on Linux, not Windows. Use `--deployment-container-image-name` to specify image tag when `az webapp create `.
* [WebJobs](https://docs.microsoft.com/en-us/azure/app-service/webjobs-create) is a feature of Azure `App Service` that enables you to run a program or script in the same instance as a web app, API app, or mobile app. There is no additional cost to use WebJobs
  * WebJobs is not yet supported for App Service on Linux.(As of Aug 2022)
  * Continuous WebJob: keep running, on *all* instances
  * Triggered WebJob: only run when manually triggered or scheduled, on a *single* instance.
* `Azure Container Instances` -> FarGate: provide portable environments for virtualized applications. It can start containers in Azure in seconds, without the need to *provision and manage* VMs.
  * The top-level resource in Azure Container Instances is the [container group](https://docs.microsoft.com/en-us/learn/modules/create-run-container-images-azure-container-instances/2-azure-container-instances-overview). A container group is a collection of containers that get scheduled on the same host machine. The containers in a container group share a lifecycle, resources, local network, and storage volumes. It's similar in concept to a pod in Kubernetes.
  * There are two common ways to deploy a multi-container group: use a Resource Manager template or a YAML file.
    1. A Resource Manager template is recommended when you need to deploy additional Azure service resources (for example, an Azure Files share) when you deploy the container instances. 
    2. Due to the YAML format's more concise nature, a YAML file is recommended when your deployment includes only container instances.
* AKS(k8s) + ACR(container registry)
  * build image using `az zcr build xxx` could offload the image building to ACR.
* Azure DevTest Labs -> support windows/linux, quickly provision development and test environments, Set automated shutdowns to minimize costs
* `Azure Databricks` is an Apache *Spark*-based analytics platform.
* [API-Management](https://learn.microsoft.com/en-us/training/modules/explore-api-management/2-api-management-overview) -> api-gateway.
  * allow group apis into [products](https://github.com/Azure-Samples/Serverless-APIs/blob/main/readme/3%20-%20Products.md) for easier management like publish/unpublish, applying policies etc.
  * Components:
    1. apim come with a *developer portal* showing the APIs details like documentation, API try-out interactively, create account and subscribe to get API key, Access Analytics on own usage.
    2. *API Gateway* which accepts call and route to backend, verify API keys/jwt/certificates, caching, usage quotas and rate limits.
    3. *Azure portal* which Define/import API Schema, package APIS into products, manage users, set policies like quotas or transformation on APIs.
  * *Policies* are a collection of statements that are executed sequentially on the request or response of an API. Popular statements include format conversion from XML to JSON and call rate limiting to restrict the number of incoming calls from a developer, and many other policies are available.
  * [Gateway pattern](https://learn.microsoft.com/en-us/training/modules/explore-api-management/3-api-gateways):
    1. Gateway Routing: Use the gateway as a reverse proxy to route requests to one or more backend services, using layer 7 routing.
    2. Gateway aggregation: when a single operation requires calls to multiple backend services, the client sends one request to the gateway. The gateway dispatches requests to the various backend services, and then aggregates the results and sends them back to the client. This helps to reduce chattiness between the client and the backend.
    3. Gateway Offloading: Use the gateway to offload functionality from individual services to the gateway, particularly cross-cutting concerns. It can be useful to consolidate these functions into one place, rather than making every service responsible for implementing them like SSL termination, auth, rate limit, logging, caching, gzip etc.

## Storage
* Blobs/Files/Queues(messages)/Tables(no-SQL)/Disks(Block-level).
* Storage Account: Unique Azure namespace(Every Object in Azure has its own web address). The base of all Azure storage types.
* `Storage Account` can have multiple `Containers` which in term contains multiple `Objects`. 
  * The public url can be `https://{StorageAccountName}.blob.core.windows.net/{ContainerName}/{fileName}`.
  * When you create a storage account, Azure generates two 512-bit *storage account access keys* for that account. These keys can be used to authorize access to data in your storage account via Shared Key authorization. Your storage account access keys are similar to a root password for your storage account. Always be careful to protect your access keys. 
* Blob Types: Block(up to 4.7TB, for uploading large file), Append(Logging), Page(up to 8TB, virtual Hard Drive)
  * pricing: Hot/Cool/Archive
  * The `Cool Access` is for Short-term backup and disaster recovery. Archive Access is for long term backup.
* Disk Types: HDD, standard/premium SSD, Ultra Disk(up to 64TB, for data intensive)
* Blob storage *access tiers*. it can be set on a blob during or after upload.
  * Hot - Optimized for storing data that is accessed frequently.
  * Cool - Optimized for storing data that is infrequently accessed and stored for at least 30 days. Data in the *cool* access tier has slightly *lower availability*, but still has high durability, retrieval latency, and throughput characteristics similar to hot data.
  * Archive - Optimized for storing data that is rarely accessed and stored for at least 180 days with flexible latency requirements, on the order of hours. The archive access tier can only be set at the blob level, not the account level
* Azure AD integration is supported for *blob and queue* data operations. You can assign RBAC roles scoped to a subscription, resource group, storage account, or an individual container or queue to a security principal or a managed identity for Azure resources.
* Customer-managed vs Customer-provided keys: CMK can be accessed by Microsoft and used by azure portal/CLI/Powershell, where CPK cannot and accessed only by Customer. Key rotation responsibility is on *Customer* for both.
* list files: `az storage blob list --connection-string 'xxx' --container-name YourContainer -otable`
* A *shared access signature* [SAS](https://docs.microsoft.com/en-us/learn/modules/implement-shared-access-signatures/2-shared-access-signatures-overview) is a signed URI that points to one or more storage resources and includes a token that contains a special set of query parameters. The token indicates how the resources may be accessed by the client. One of the query parameters, the signature, is constructed from the SAS parameters and signed with the key that was used to create the SAS. This signature is used by Azure Storage to authorize access to the storage resource.
  * like S3 pre-signed URL. useful when uploading and other temporary access.

* Archive -> Glacier. A `Blob` tier so same tool will work. 
  * To read or download a blob in archive, you must first rehydrate it to an online tier.
  * Data in the archive access tier is stored offline. The archive tier offers the lowest storage costs but also the highest access costs and latency.
  * The archive tier supports only LRS, GRS, and RA-GRS redundancy options.
  * When you rehydrate a blob, you can set the priority for the rehydration operation via the optional x-ms-rehydrate-priority header on a Set Blob Tier. Standard priority or High Priority.
  * it takes *1-15 hours* to recover data form the archive tier.
* Azure Storage redundancy
  * Locally-redundant storage, LRS (replicated in same data center)
  * Zone-redundant storage, ZRS (replicated in different Data center within same AZ)
  * Geo-redundant storage (GRS): primary region with LRS
  * Geo-zone-redundant storage (GZRS): primary region with ZRS
  * For *read* access to the secondary region, configure your storage account to use read-access geo-redundant storage (RA-GRS) or read-access geo-zone-redundant storage (RA-GZRS). 
* Blob Storage [life-cycle police](https://learn.microsoft.com/en-us/azure/storage/blobs/lifecycle-management-overview#archive-data-after-ingest): transition your data to the appropriate access tiers or expire at the end of the data's lifecycle.
  * a collection of rules in JSON format, we can use type/prefix in *filter* sections and define *actions* to do things like `tierToCool/enableAutoTierToHotFromCool/tierToArchive/delete` etc with *conditions* like `daysAfterModificationGreaterThan/daysAfterCreationGreaterThan`. 
  * can be updated via cli/ui/REST. In portal, it is under Storage Account -> Data Management -> Lifecycle Management.
  * the `version` element defines actions on previous versions.
* Azure Files -> AWS EFS, storage/sharing with NFS(Network File System) protocol.
  * File Storage use case: Hybrid, Lift and Shift.
* You can use Power BI to analyze and visualize data stored in `Azure Data Lake` and `Azure SQL Data Warehouse`.
* `Azure containers` are the backbone of the *virtual disks* platform for Azure IaaS. Both Azure OS and data disks are implemented as virtual disks where data is durably persisted in the Azure Storage platform and then delivered to the *virtual machines* for maximum performance. Azure Disks are persisted in Hyper-V VHD format and stored as a page blob in Azure Storage.
* You can copy blobs, directories, and containers between storage accounts by using the *AzCopy* command-line utility. `AZCopy` should theoretically offer better performance over `az storage blob xxx` or `Start-AzureStorageBlobCopy` as it is designed for optimal performance and operating a *bulk* mode.
* As datasets get larger, finding a specific object in a sea of data can be difficult. [Blob index tags](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-manage-find-blobs?tabs=azure-portal) provide data management and discovery capabilities by using key- value index tag attributes. You can categorize and find objects within a single container or across all containers in your storage account. As data requirements change, objects can be dynamically categorized by updating their index tags

## Databases
* CosmosDB: Global from start, single digit latency, pay for use. 
  * max document size: `2MB`
  * cosmos `db account` (with free tier), specify global distribution, networking, backup, encryption etc
  * cosmos db has an `id` and a `partition key`(a path to the json separated by `/`) and their combination has to be unique. We can use the same field for both. A [documentation on partition key choice](https://docs.microsoft.com/en-us/azure/cosmos-db/partitioning-overview#choose-partitionkey)
  * CosmosDB automatically indexes every property for all items in the container/collection [without having to defining schema or secondary indexes](https://docs.microsoft.com/en-us/azure/cosmos-db/index-overview).
    * Range index
    * Spatial Index(geoJSOn)
    * Composite index(multiple fields)
    * to optimize performance, we can [use include or exclude strategy](https://dev.to/willvelida/understanding-indexing-in-azure-cosmos-db-21kc) to index only certain fields or explicitly exclude some from the indexing.
  * cosmosDB does not have GSI so an equivalent way is to add items into another container with corresponding id by using a different partition key(as a discriminator property). like ProductID with `productType` as partition key in a new container.
  * The cost to do a point read, which is fetching a single item by its ID and partition key value, for a 1KB item is *1RU*.
  * 3 Billing mode: *Provisioned throughput, Serverless, Autoscale*.
  * A container is scaled by distributing data and throughput across *physical* partitions. each individual *physical* partition can store up to 50GB data and 10000 RU/s
  * partition key should 
    1. be a property that has a value which does not change.
    2. have a high cardinality, meaning the property should have a wide range of possible values.
    3. Spread Request Unit(RU) consumption and data storage evenly across all logical partitions.
  * JS based [store procedure](https://docs.microsoft.com/en-us/learn/modules/work-with-cosmos-db/4-cosmos-db-stored-procedures) and [trigger/user-defined-functions](https://docs.microsoft.com/en-us/learn/modules/work-with-cosmos-db/5-cosmos-db-triggers-user-defined-functions)
  * Azure Cosmos DB allows developers to choose among the five consistency models: *strong, bounded staleness, session, consistent prefix and eventual*.
    * If your Azure Cosmos account is configured with a consistency level other than the strong consistency, you can find out the probability that your clients may get strong and consistent reads for your workloads by looking at the Probabilistically Bounded Staleness (PBS) metric, which shows how eventual your eventual consistency is.
    * *Strong Consistency* : the order of operation is preserved and the reads are guaranteed to return the most recent version of the item in the database. The client always gets the latest changes in the data when requested.
    * *Bounded Staleness*: In the Bounded Staleness level, data is replicated asynchronously with a predetermined staleness window defined either by numbers of writes or a period of time. The reads query may lag behind by either a certain number of writes or by a pre-defined time period.
    * *Session consistency* is the default consistency that you get while configuring the cosmos DB account. This level of consistency honors the client session. It ensures a strong consistency for an application session with the same session token. What that means is that whatever is written by a session will return the latest version for reads as well, from that same session.
    * *Consistent Prefix* model is similar to bounded staleness except, the operational or time lag guarantee. The replicas guarantee the consistency and order of the writes however the data is not always current. This model ensures that the user never sees an out-of-order write. For example, if data is written in the order A, B, and C, the user may either see A, A,B or A,B,C, but never out-of-order entry like A,C or B,A,C. This model provides high availability and very low latency which is best for certain applications that can afford the lag and still function as expected.
    * *Eventual consistency* is the weakest consistency level of all. The first thing to consider in this model is that there is no guarantee on the order of the data and also no guarantee of how long the data can take to replicate. As the name suggests, the reads are consistent, but eventually.
  * .NET client order: `CosmosClient -> Database -> Container -> Item`.
  * Built-in [RBAC roles for common management scenarios](https://docs.microsoft.com/en-us/azure/cosmos-db/role-based-access-control):
     1. DocumentDB Account Contributor:	Can manage Azure Cosmos DB accounts.
     2. Cosmos DB Account Reader:	Can read Azure Cosmos DB account data.
     3. CosmosBackupOperator:	Can submit a restore request in the Azure portal for a periodic backup enabled database or a container. Can modify the backup interval and retention in the Azure portal. Cannot access any data or use Data Explorer.
     4. CosmosRestoreOperator:	Can perform a restore action for an Azure Cosmos DB account with continuous backup mode.
     5. Cosmos DB Operator:	Can provision Azure Cosmos DB accounts, databases, and containers. Cannot access any data or use Data Explorer.
  * The [change feed processor](https://learn.microsoft.com/en-us/azure/cosmos-db/nosql/change-feed-processor?tabs=dotnet) is part of the Azure Cosmos DB .NET V3 and Java V4 SDKs. There are four main components of implementing the change feed processor:
    1. The monitored container: The monitored container has the data from which the change feed is generated. Any inserts and updates to the monitored container are reflected in the change feed of the container.
    2. The lease container: The lease container acts as a state storage and coordinates processing the change feed across multiple workers. The lease container can be stored in the same account as the monitored container or in a separate account.
    3. The compute instance: A compute instance hosts the change feed processor to listen for changes. Depending on the platform, it could be represented by a VM, a kubernetes pod, an Azure App Service instance, an actual physical machine. It has a unique identifier referenced as the instance name throughout this article.
    4. The delegate: The delegate is the code that defines what you, the developer, want to do with each batch of changes that the change feed processor reads.

* Azure SQL: managed SQL-Server, up to 100TB.
* Managed MySQL/PostgreSQL and DMS
* Azure Table storage: Every entity stored in a table must have a unique combination of *PartitionKey* and *RowKey*. The Table service does not create any secondary indexes, so PartitionKey and RowKey are the only indexed properties.
* Azure Cache for Redis, Microsoft recommends always use Standard or Premium Tier for production systems. The Basic Tier is a single node system with no data replication and no SLA.
  * The Premium tier allows you to persist data in two ways to provide disaster recovery:
    1. RDB persistence takes a periodic snapshot and can rebuild the cache using the snapshot in case of failure.
    2. AOF persistence saves every write operation to a log that is saved at least once per second. This creates bigger files than RDB but has less data loss.[]
  * The Premium tier also deployment to a virtual network.

## Messaging
* Azure Event Hubs(~=AWS Managed Streaming for Kafka / MSK) is a big data streaming platform and event ingestion service. It can receive and process millions of events per second. Data sent to an event hub can be transformed and stored by using any real-time analytics provider or batching/storage adapters.
  * Event Hubs traffic is controlled by throughput units. A single throughput unit allows 1 MB per second or 1000 events per second of ingress and twice that amount of egress. Standard Event Hubs can be configured with 1-20 throughput units, and you can purchase more with a quota increase support request.
* Azure Service Bus: Queues(like sqs, one Consumer, meaning message consume once unless failed) and Topic(multiple Consumer, like kafka)
  * Message sessions enable joint and ordered handling of unbounded sequences of related messages.(FIFO)
  * [Azure Storage Queues](https://medium.com/awesome-azure/azure-difference-between-azure-storage-queue-and-service-bus-queue-azure-queue-storage-vs-servicebus-3f7921b0159e) are simpler to use but are less sophisticated and flexible than Service Bus queues.
  * For Queues, You can specify two different modes in which Service Bus receives messages: *Receive and delete* or *Peek lock*(contains a visibility timeout).
  * Service Bus supports [three filter conditions](https://learn.microsoft.com/en-us/azure/service-bus-messaging/topic-filters#filters):
    * SQL Filters: A SqlFilter holds a SQL-like conditional expression that is evaluated in the broker against the arriving messages' user-defined properties and system properties.
    * Boolean filters - The TrueFilter and FalseFilter either cause all arriving messages (true) or none of the arriving messages (false) to be selected for the subscription.
    * Correlation Filters: A simplified SqlFilter with only `AND-ed`/`Equal` conditions. It has better performance.
* EventGrid(-> EventBridge?)
  * Event Grid doesn't guarantee order for event delivery, so subscribers may receive them out of order.
  * Subscribers use the subject to filter and route events. Consider providing the path for where the event happened, so subscribers can filter by segments of that path. For example, the Storage Accounts publisher provides the subject `/blobServices/default/containers/<container-name>/blobs/<file>` when a file is added to a container. A subscriber could filter by the path `/blobServices/default/containers/testcontainer` to get all events for that container but not other containers in the storage account. A subscriber could also filter or route by the suffix .txt to only work with text files.
  * [retry](https://docs.microsoft.com/en-us/training/modules/azure-event-grid/4-event-grid-delivery-retry) does *NOT* happen if it is client error like 400(bad request)/413(request too large)/403(forbidden). Otherwise Event Grid waits 30 seconds for a response after delivering a message. After 30 seconds, if the endpoint hasn’t responded, the message is queued for retry. Event Grid uses an exponential backoff retry policy for event delivery.
  * If the endpoint responds within 3 minutes, Event Grid will attempt to remove the event from the retry queue on a best effort basis but duplicates may still be received. Event Grid adds a small randomization to all retry steps and may opportunistically skip certain retries if an endpoint is consistently unhealthy, down for a long period, or appears to be overwhelmed.
  * When Event Grid can't deliver an event within a certain time period or after trying to deliver the event a certain number of times, it can send the undelivered event to a storage account. By default, Event Grid doesn't turn on dead-lettering. To enable it, you must specify a storage account to hold undelivered events when creating the event subscription.
  * When creating an event subscription, you have three options for [filtering](https://docs.microsoft.com/en-us/training/modules/azure-event-grid/7-event-grid-filtering):
    1. Event types
    2. Subject begins with or ends with
    3. Advanced fields and operators. To filter by values in the data fields and specify the comparison operator, use the advanced filtering option.

## AuthN & AuthR
* Azure Active Directory(AAD) 
  * manage users and permissions.
  * AAD is completely different from Active Directory. It is mandatory(cannot use Azure account without AAD service).
  * Tenant and User(member of up to 500 tenants) are many to many.
  * done with Group/Member/Role etc.
* application objects and service principals, reference at [ms docs](https://docs.microsoft.com/en-us/azure/active-directory/develop/app-objects-and-service-principals).
  * The application object is the *global* representation of your application for use across all tenants, and the service principal is the *local* representation for use in a specific tenant. The application object serves as the template from which common and default properties are derived for use in creating corresponding service principal objects.
  * An application object has:
    * 1:1 relationship with the software application, and
    * 1:many relationship with its corresponding service principal object(s).
  * [service principal](https://identitydocs.azurewebsites.net/static/v2/Service-principal-provisioning/Overview-of-service-principal-provisioning.html) are AAD objects which represent your app in each customer's tenant. They define what the app can actually do in the specific tenant, who can access the app, and what resources the app can access.
  * An `Azure AD application` is defined by its one and only `application object`, which resides in the Azure AD tenant where the application was registered (known as the application's "home" tenant). 
  * An `application object` is used as a template or blueprint to create one or more `service principal` objects. The `application object` describes three aspects of an application: 1. how the service can issue tokens in order to access the application, 2. resources that the application might need to access, and 3. the actions that the application can take. A `service principal` is created in *every* tenant where the application is used. Similar to a class in object-oriented programming, the application object has some static properties that are applied to all the created service principals (or application instances). 
  * The `security principal` defines the access policy and permissions for the user/application in the Azure AD tenant. This enables core features such as authentication of the user/application during sign-in, and authorization during resource access. A *service principal* must be created in *each* tenant where the application is used, enabling it to establish an identity for sign-in and/or access to resources being secured by the tenant. A single-tenant application has only one service principal (in its home tenant), created and consented for use during application registration. A multi-tenant application also has a service principal created in each tenant where a user from that tenant has consented to its use.
  * An application object has:
    * A 1:1 relationship with the software application, and
    * A 1:many relationship with its corresponding service principal object(s).
  * The Microsoft [identity platform](https://docs.microsoft.com/en-us/training/modules/explore-microsoft-identity-platform/4-permission-consent) implements the OAuth 2.0 authorization protocol
    *  Any web-hosted resource that integrates with the Microsoft identity platform has a *resource identifier*, or application ID URI. Here are some examples of Microsoft web-hosted resources:
        Microsoft Graph: https://graph.microsoft.com
        Microsoft 365 Mail API: https://outlook.office.com
        Azure Key Vault: https://vault.azure.net
      The same is true for any third-party resources that have integrated with the Microsoft identity platform. Any of these resources also can define a set of permissions that can be used to divide the functionality of that resource into smaller chunks. When a resource's functionality is chunked into small permission sets, third-party apps can be built to request only the permissions that they need to perform their function. Users and administrators can know what data the app can access. In OAuth 2.0, these types of permission sets are called `scopes`. They're also often referred to as `permissions`. In the Microsoft identity platform, a permission is represented as a string value. An app requests the permissions it needs by specifying the permission in the *scope* query parameter. Identity platform supports several well-defined OpenID Connect scopes as well as resource-based permissions (each permission is indicated by appending the permission value to the resource's identifier or application ID URI). For example, the permission string https://graph.microsoft.com/Calendars.Read is used to request permission to read users calendars in Microsoft Graph.
  * *Managed identities* provide an identity for applications to use when connecting to resources that support Azure Active Directory (Azure AD) authentication. Applications may use the managed identity to obtain Azure AD tokens.
    * `system-assgined managed identity` is created by Azure and used on Azuer VM or App service like aws instance profile that the lifecycle is the same as the instance. `user-assigned managed identity` is created separated and can be reused by many services.
    * a service principal is created for the managed identity, and this service principal client Id and certificate is use later for getting the JWT tokens etc.
* [Azure roles vs AAD(Azure AD) roles](https://learn.microsoft.com/en-us/training/modules/manage-subscription-access-azure-rbac/2-identify-appropriate-role-assign)
  * The main difference between `Azure roles` and `Azure AD roles` is the areas they cover. Azure roles apply to Azure resources, and Azure AD roles apply to Azure AD resources (particularly users, groups, and domains). Also, Azure AD has only one scope: the directory. The Azure RBAC scope covers management groups, subscriptions, resource groups, and resources.
  * The roles share a key area of overlap. An Azure AD Global Administrator can elevate their access to manage all Azure subscriptions and management groups. This greater access grants them the Azure RBAC User Access Administrator role for all subscriptions of their directory. Through the User Access Administrator role, the Global Administrator can give other users access to Azure resources.
  * In our scenario, you need to grant full Azure RBAC management and billing permissions to a new manager. To achieve this, you'll temporarily elevate your access to include the User Access Administrator role(revoke later). You can then grant the new manager the Owner role so that they can create and manage resources. You'll also set the scope to the subscription level, so that the manager can do this for all resources in the subscription.

## Monitoring
* `Azure Service Health` notifies you about Azure service incidents and planned maintenance so you can take action to mitigate downtime. Configure customizable cloud alerts and use your personalized dashboard to analyze health issues, monitor the impact to your cloud resources, get guidance and support, and share details and updates.
* `Azure Monitor` -> CloudWatch: Collect, analyze, and act on telemetry data from your Azure and on-premises environments. Azure Monitor helps you maximize performance and availability of your applications and proactively identify problems in seconds.
  * Availability test: After you've deployed your web app or website, you can set up recurring tests to monitor availability and responsiveness. Application Insights sends web requests to your application at regular intervals from points around the world. It can alert you if your application isn't responding or responds too slowly.
  * *Application Map* helps you spot performance bottlenecks or failure hotspots across all components of your distributed application. Each node on the map represents an application component or its dependencies; and has health KPI and alerts status.
* [KQL, Kusto Query Language](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/). A Kusto query is a read-only request to process data and return results.
* `Azure Advisor` provides you with a consistent, consolidated view of recommendations for all your Azure resources. It integrates with Azure Security Center to bring you *security* recommendations. It helps you optimize and reduce your overall Azure spend/cost by identifying idle and underutilized resources.

## Security
* Azure Security Center -> overall monitoring/scoring on security
  * The advanced monitoring capabilities in Security Center lets you track and manage *compliance and governance* over time. The overall compliance provides you with a measure of how much your subscriptions are compliant with policies associated with your workload.
* Azure Key vault -> storing credentials and grant/revoke access. Similar to AWS Secret Manager.
  * ps command: `Get-AzKeyVaultSecret -VaultName "keyvaultName"`
  * Azure Key Vault has two service tiers: Standard, which encrypts with a software key, and a Premium tier, which includes hardware security module(HSM)-protected keys.
  * A Key Vault *access policy* determines whether a given security principal, namely a user, application or user group, can perform different operations on Key Vault secrets, keys, and certificates.
  * [Bring your own key](https://learn.microsoft.com/en-us/azure/key-vault/keys/byok-specification#user-steps):  use Key Exchange Key(KEK) which is a HSM backed RSK key pair generated in AKV, retrieve the public key of KEK, import KEK public key into target HSM and export the Target Key protected by the KEK, import the protected Target Key to the AKV.
* Azure Information protection -> protect files like word/excel etc while sharing as email attachments etc.
* Azure sentinel, collect/aggregate/analyze security issues automatically.
* Defender for Identity(formerly Advanced Threat Protection) -> manage/monitor user behaviors in organization, and reporting.
* You can view list of compliance certifications(Azure has 90+ including over 50 specific to global regions/countries) in the `Trust Center` to determine whether Azure meets your *regional requirements*.
* MFA login can be configured with [Conditional Access policy](https://learn.microsoft.com/en-us/azure/active-directory/authentication/tutorial-enable-azure-mfa).
  1. Select Users/Groups to assign for the policy to apply to.
  2. Define the cloud-app/actions that trigger the policy
  3. *Access Control* let you define the requirements for a user to be granted access. They might be required to use an approved client app or a device that's hybrid-joined to Azure AD.
* For client certificate auth in TLS mutual authentication, it would be *base64* encoded inside the *Request Header*.

## Pricing
* An account can have multiple subscriptions. Billing Admin and Management Group
* data transferred within same `billing zone` is free.
* Pricing calculator to help better estimation.

## Subscription
* You cannot merge two subscriptions into a single subscription. However, you can move some Azure resources from one subscription to another. You can also transfer ownership of a subscription and change the billing type for a subscription.
* You can assign service administrators and co-administrators in the Azure Portal but there can only be one account administrator.
* Support plans
  * *developer* 3rd part software support with interoperability and configuration guidance and troubleshooting.
  * *Standard* provides 7/24 tech support by email/phone after a support request is submitted. 
  * *Premier/professional-direct* provides architectural support such as design reviews, performance tuning, configuration and implementation assistance.

## IoT/AI
* `Azure IoT Hub` is paas(provides SDK), a managed service hosted in the cloud that acts as a central message hub for communication between an IoT application and its attached devices. You can connect millions of devices and their backend solutions reliably and securely. Almost any device can be connected to an IoT hub.
  * There are two storage services IoT Hub can route messages to -- Azure *Blob Storage* and Azure *Data Lake Storage* Gen2 (ADLS Gen2) accounts. Azure Data Lake Storage accounts are hierarchical namespace-enabled storage accounts built on top of blob storage. Both of these use blobs for their storage.
* `Azure IoT Central` is saas, which lets you quickly connect devices, monitor device conditions, create rules, and manage millions of devices and their data throughout their life cycle.
* `Azure Sphere` is a secured, high-level application platform with built-in communication and security features for internet-connected devices. It is a software and *hardware* solution.
* `Azure Cognitive Services` are APIs, SDKs, and services available to help developers build intelligent applications without having direct AI or data science skills or knowledge. Azure Cognitive Services enable developers to easily add cognitive features into their applications. The goal of Azure Cognitive Services is to help developers create applications that can see, hear, speak, understand, and even begin to reason. The catalog of services within Azure Cognitive Services can be categorized into five main pillars - Vision, Speech, Language, Web Search, and Decision.
* `Azure Machine Learning designer` lets you visually connect datasets and modules on an interactive canvas to create machine learning models.
* `Azure Bot Services` provides a digital online assistant that provides speech support. Bots provide an experience that feels less like using a computer and more like dealing with a person - or at least an intelligent robot. They can be used to shift simple, repetitive tasks, such as taking a dinner reservation or gathering profile information, on to automated systems that may no longer require direct human intervention. 

## Serverless
* Azure functions, App Logic(step functions?), 
  * For Azure Functions, you develop orchestrations by writing code and using the Durable Functions extension. For Logic Apps, you create orchestrations by using a GUI or editing configuration files.
* Azure function hosting plans
  * *Consumption plan* 	This is the default hosting plan. It scales automatically and you only pay for compute resources when your functions are running. Instances of the Functions host are dynamically added and removed based on the number of incoming events.
  * *Functions Premium plan* 	Automatically scales based on demand using pre-warmed workers which run applications with no delay after being idle, runs on more powerful instances, and connects to virtual networks. 
  * *App service plan* 	Run your functions within an App Service plan at regular App Service plan rates. Best for long-running scenarios where Durable Functions can't be used.
* Azure function scales [at the unit scale of 'functin app'](https://docs.microsoft.com/en-us/learn/modules/explore-azure-functions/4-scale-azure-functions).
  * Maximum instances: A single function app only scales out to a maximum of 200 instances for Consumption plan and 100 for Premium plan. A single instance may process more than one message or request at a time though, so there isn't a set limit on number of concurrent executions. `functionAppScaleLimit` is the variable to tweak the number of instances, which can be set to 0 or *null* for unrestricted.
  * New instance rate: For HTTP triggers, new instances are allocated, at most, once per second. For non-HTTP triggers, new instances are allocated, at most, once every 30 seconds. Scaling is faster when running in a Premium plan.
  * Using an *App Service plan*, you can manually scale out by adding more VM instances. You can also enable autoscale, though autoscale will be slower than the elastic scale of the Premium plan.
* Triggers and Bindings
  * Triggers cause a function to run. A trigger defines how a function is invoked and a function must have exactly one trigger. Triggers have associated data, which is often provided as the payload of the function.
  * Binding to a function is a way of declaratively connecting another resource to the function; bindings may be connected as input bindings, output bindings, or both. Data from bindings is provided to the function as parameters.
  * [Triggers and bindings](https://learn.microsoft.com/en-us/azure/azure-functions/functions-triggers-bindings) let you avoid hard-coding access to other services. Your function receives data (for example, the content of a queue message) in function parameters. You send data (for example, to create a queue message) by using the return value of the function.
  * [Timer trigger](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-timer) configuration.
* [Durable Functions](https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-overview) is an extension of Azure Functions that lets you write stateful functions in a serverless compute environment. The extension lets you define stateful workflows by writing [orchestrator functions](https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-orchestrations) and [stateful entities](https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-entities) by writing entity functions using the Azure Functions programming model. Behind the scenes, the extension manages state, checkpoints, and restarts for you, allowing you to focus on your business logic.
  * Durable Functions is good for function-chain/fan-in-out/async-api/monitoring/human-interaction/aggregation etc.
  * [billing](https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-billing) is the same way as normal Functions.
  * [Durable Entities](https://markheath.net/post/durable-entities-what-are-they-good-for) can be good for things like [IoT on-offline](https://case.schollaart.net/2019/10/31/device-offline-detection-with-durable-entities.html)/[CircuitBreaker](https://dev.to/azure/serverless-circuit-breakers-with-durable-entities-3l2f) etc.
  * If you have multiple function apps sharing a shared storage account, you must explicitly [configure different names for each task hub](https://docs.microsoft.com/en-us/learn/modules/implement-durable-functions/4-durable-functions-task-hubs) in the host.json files. Otherwise the multiple function apps will compete with each other for messages, which could result in undefined behavior, including orchestrations getting unexpectedly "stuck" in the Pending or Running state.
  * The Durable Functions extension *automatically* adds a set of *HTTP APIs* to the Azure Functions host. With these APIs, you can interact with and manage orchestrations and entities without writing any code.
* When integrating with Azure Queue Storage
  * use [maxDequeueCount](https://learn.microsoft.com/en-us/azure/azure-functions/functions-bindings-storage-queue?tabs=in-process%2Cextensionv5%2Cextensionv3&pivots=programming-language-csharp#host-json) to define the number of times to try processing a message before moving it to the poison queue. Default is 5.
  * default batch size is 16, max is 32.
* Regardless of the function app timeout setting, *230 seconds* is the maximum amount of time that an *HTTP triggered* function can take to respond to a request. This is because of the *default idle timeout of Azure Load Balancer*. For longer processing times, consider using the Durable Functions async pattern or defer the actual work and return an immediate response.
* [Azure Functions custom handlers](https://learn.microsoft.com/en-us/azure/azure-functions/functions-custom-handlers) are lightweight web servers that receive events from Functions host. Any language that supports HTTP primitives can implement a custom handler.

## Data
* [Microsoft Graph](https://docs.microsoft.com/en-us/training/modules/microsoft-graph/2-microsoft-graph-overview) is the gateway to data and intelligence in Microsoft 365. It provides a unified programmability model that you can use to access the tremendous amount of data in Microsoft 365, Windows 10, and Enterprise Mobility + Security.
  * The Microsoft Graph API offers a single endpoint, https://graph.microsoft.com.
  * Microsoft Graph is a RESTful web API that enables you to access Microsoft Cloud service resources. After you register your app and get authentication tokens for a user or service, you can make requests to the Microsoft Graph API.


##  misc
* ResourceGroup can be shutdown any time as long as the resources are not subject to protection
* BigData: DataLake analytics, Azure DataBricks(managed spark), HDInsight(hadoop/spark/kafka), Synapse Analytics(data warehouse) & Synapse SQL,
* Traditionally, IT expenses have been considered a [Capital Expenditure](https://tutorialsdojo.com/azure-capex-vs-opex/) (CapEx) like buying server etc. Today, with the move to the cloud and the pay-as-you-go model, organizations have the ability to stretch their budgets and are shifting their IT CapEx costs to Operating Expenditures (OpEx) instead. 
* `Azure Site Recovery` helps ensure business continuity by keeping business apps and workloads running during outages. Site Recovery replicates workloads running on physical and virtual machines (VMs) from a primary site to a secondary location.
* The `Azure Total Cost of Ownership`(TCO) Calculator is used to estimate the cost savings you can achieve by migrating your application workloads to Microsoft ..
* `Azure Cost Management` is the process of effectively planning and controlling costs involved in your business. Customers with an Azure Enterprise Agreement(EA), Microsoft Customer Agreement(MCA), or Microsoft Partner Agreement(MPA) can use Azure Cost Management.
* Traffic Manager -> Route53
* Azure *App Configuration* is designed to be a centralized repository for feature flags. 
* Azure Search
  * *Microsoft.Azure.Search.Models.SearchParameters*
    * Filter: Gets or sets the OData $filter expression to apply to the search query
    * QueryType: lucene query, like Regex/fulltext-search etc.
    * SearchMode: match criteria -> "any" or "all"
  * create `SearchIndexClient` to connect to the search index, create IndexBatch that contains the documents, call `Documents.Index` and pass the IndexBatch.
