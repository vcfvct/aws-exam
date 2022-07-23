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
  * ARM is the layer managing infrastructure no matter from cli/port etc.
  * something similar to CloudFormation but without yaml support, json only. Can be nested
  * Source control, Declarative, Idempotent.
  * no similar things like CDK for now for IaS, alternative is Terraform(or CDK to Terraform).
  * Every resource need to be inside a resource group. No nested RG. RG can contain resources from multiple regions.
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
* ExpressRoute -> DirectConnect. Connect to on-premises stuff with private connection instead of public Internet.
* A Point-to-Site (P2S) VPN gateway connection lets you create a secure connection to your virtual network from an individual client computer.
* A Site-to-Site (S2S) VPN gateway is good for Dev/test/ lab scenarios and small to medium scale production workloads for cloud services and virtual machines
* A `Local Network Gateway` is an object in Azure that represents your on-premise VPN device. A `Virtual Network Gateway` is the VPN object at the Azure end of the VPN. A `connection` is what connects the Local Network Gateway and the Virtual Network Gateway to bring up the VPN.
* Not all Azure regions support availability zones.

## Compute
* Scale Sets -> ASG
* App Services: PaaS with `Web Apps` for standard app or Containers or APIs, with different run time like node/php/.net etc.
  * use `az webapp list-runtimes --os-type linux` to retrieve the current supported runtime list.
  * multiple apps can be deployed to the same AppService Plan, they will scan together(all apps run on all instances each). Isolate the app into a new AppService Plan when app is resource intensive, or want to scale the app independently, or needs resource in a different geographical region.
  * development slot: deploy app to a staging environment and swap staging and production slots when ready(Standard AppService Plan tier+). Traffic can be routed automatically or manually(by using `x-ms-routing-name` query parameter). After a client is routed to a specific slot, it is "pinned" to that slot for the life of that client session(due to the `x-ms-routing-name` cookie).
  * `Application settings` under `Configuration` defines environment variables for the app.
  * `general settings` under `Configuration` has some common settings like stack/runtime-version/tls etc.
  * auto-scale by 1. metric(cpu/memory/disk-queue/http-queue/data-numberOfBytes in or out) min-duration 5 minutes or 2. schedule.
* `Azure Container Instances` -> FarGate: provide portable environments for virtualized applications. It can start containers in Azure in seconds, without the need to *provision and manage* VMs.
* AKS(k8s) + ACR(container registry)
* Azure DevTest Labs -> support windows/linux, quickly provision development and test environments, Set automated shutdowns to minimize costs
* `Azure Databricks` is an Apache *Spark*-based analytics platform.
* API-Management -> api-gateway.
  * allow group apis into [products](https://github.com/Azure-Samples/Serverless-APIs/blob/main/readme/3%20-%20Products.md) for easier management like publish/unpublish, applying policies etc.

## Storage
* Blobs/Files/Queues(messages)/Tables(no-SQL)/Disks(Block-level).
* Storage Account: Unique Azure namespace(Every Object in Azure has its own web address). The base of all Azure storage types.
* `Storage Account` can have multiple `Containers` which in term contains multiple `Objects`. 
  * The public url can be `https://{StorageAccountName}.blob.core.windows.net/{ContainerName}/{fileName}`.
* Blob Types: Block(up to 4.7TB), Append(Logging), Page(up to 8TB, virtual Hard Drive)
  * pricing: Hot/Cool/Archive
  * The `Cool Access` is for Short-term backup and disaster recovery. Archive Access is for long term backup.
* Disk Types: HDD, standard/premium SSD, Ultra Disk(up to 64TB, for data intensive)
* Blob storage *access tiers*. it can be set on a blob during or after upload.
  * Hot - Optimized for storing data that is accessed frequently.
  * Cool - Optimized for storing data that is infrequently accessed and stored for at least 30 days. Data in the *cool* access tier has slightly *lower availability*, but still has high durability, retrieval latency, and throughput characteristics similar to hot data.
  * Archive - Optimized for storing data that is rarely accessed and stored for at least 180 days with flexible latency requirements, on the order of hours. The archive access tier can only be set at the blob level, not the account level

* Archive -> Glacier. A `Blob` tier so same tool will work. 
  * To read or download a blob in archive, you must first rehydrate it to an online tier.
  * Data in the archive access tier is stored offline. The archive tier offers the lowest storage costs but also the highest access costs and latency.
  * The archive tier supports only LRS, GRS, and RA-GRS redundancy options.
  * When you rehydrate a blob, you can set the priority for the rehydration operation via the optional x-ms-rehydrate-priority header on a Set Blob Tier. Standard priority or High Priority.
* Azure Storage redundancy
  * Locally-redundant storage, LRS (replicated in same data center)
  * Zone-redundant storage, ZRS (replicated in different Data center within same AZ)
  * Geo-redundant storage (GRS): primary region with LRS
  * Geo-zone-redundant storage (GZRS): primary region with ZRS
  * For *read* access to the secondary region, configure your storage account to use read-access geo-redundant storage (RA-GRS) or read-access geo-zone-redundant storage (RA-GZRS). 
* Blob Storage life-cycle police: transition your data to the appropriate access tiers or expire at the end of the data's lifecycle.
  * a collection of rules in JSON format, we can use type/prefix in *filter* sections and define *actions* to do things like `tierToCool/enableAutoTierToHotFromCool/tierToArchive/delete` etc with *conditions* like `daysAfterModificationGreaterThan/daysAfterCreationGreaterThan`. 
  * can be updated via cli/ui/REST. In portal, it is under Storage Account -> Data Management -> Lifecycle Management.
* Azure Files -> AWS EFS, storage/sharing with NFS(Network File System) protocol.
  * File Storage use case: Hybrid, Lift and Shift.
* You can use Power BI to analyze and visualize data stored in `Azure Data Lake` and `Azure SQL Data Warehouse`.
* `Azure containers` are the backbone of the *virtual disks* platform for Azure IaaS. Both Azure OS and data disks are implemented as virtual disks where data is durably persisted in the Azure Storage platform and then delivered to the *virtual machines* for maximum performance. Azure Disks are persisted in Hyper-V VHD format and stored as a page blob in Azure Storage.

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
* Azure SQL: managed SQL-Server, up to 100TB.
* Managed MySQL/PostgreSQL and DMS
* Azure Table storage: Every entity stored in a table must have a unique combination of *PartitionKey* and *RowKey*. The Table service does not create any secondary indexes, so PartitionKey and RowKey are the only indexed properties.

## Messaging
* Azure Event Hubs is a big data streaming platform and event ingestion service. It can receive and process millions of events per second. Data sent to an event hub can be transformed and stored by using any real-time analytics provider or batching/storage adapters.
* Azure Service Bus: Queues(like sqs, one Consumer, meaning message consume once unless failed) and Topic(multiple Consumer, like kafka)

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
  * An `application object` is used as a template or blueprint to create one or more `service principal` objects. The application object describes three aspects of an application: how the service can issue tokens in order to access the application, resources that the application might need to access, and the actions that the application can take. A `service principal` is created in *every* tenant where the application is used. Similar to a class in object-oriented programming, the application object has some static properties that are applied to all the created service principals (or application instances). The `application object` describes three aspects of an application: 1. how the service can issue tokens in order to access the application, 2. resources that the application might need to access, and 3. the actions that the application can take.
  * The `security principal` defines the access policy and permissions for the user/application in the Azure AD tenant. This enables core features such as authentication of the user/application during sign-in, and authorization during resource access.

## Monitoring
* `Azure Service Health` notifies you about Azure service incidents and planned maintenance so you can take action to mitigate downtime. Configure customizable cloud alerts and use your personalized dashboard to analyze health issues, monitor the impact to your cloud resources, get guidance and support, and share details and updates.
* `Azure Monitor` -> CloudWatch: Collect, analyze, and act on telemetry data from your Azure and on-premises environments. Azure Monitor helps you maximize performance and availability of your applications and proactively identify problems in seconds.
* `Azure Advisor` provides you with a consistent, consolidated view of recommendations for all your Azure resources. It integrates with Azure Security Center to bring you *security* recommendations. It helps you optimize and reduce your overall Azure spend/cost by identifying idle and underutilized resources.

## Security
* Azure Security Center -> overall monitoring/scoring on security
  * The advanced monitoring capabilities in Security Center lets you track and manage *compliance and governance* over time. The overall compliance provides you with a measure of how much your subscriptions are compliant with policies associated with your workload.
* Azure Key vault -> storing credentials and grant/revoke access. Similar to AWS Secret Manager.
  * ps command: `Get-AzKeyVaultSecret -VaultName "keyvaultName"`
* Azure Information protection -> protect files like word/excel etc while sharing as email attachments etc.
* Azure sentinel, collect/aggregate/analyze security issues automatically.
* Defender for Identity(formerly Advanced Threat Protection) -> manage/monitor user behaviors in organization, and reporting.
* You can view list of compliance certifications(Azure has 90+ including over 50 specific to global regions/countries) in the `Trust Center` to determine whether Azure meets your *regional requirements*.

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
* Azure functions, App Logic(step functions?), EventGrid(-> EventBridge?)
  * For Azure Functions, you develop orchestrations by writing code and using the Durable Functions extension. For Logic Apps, you create orchestrations by using a GUI or editing configuration files.
* Azure function hosting plans
  * *Consumption plan* 	This is the default hosting plan. It scales automatically and you only pay for compute resources when your functions are running. Instances of the Functions host are dynamically added and removed based on the number of incoming events.
  * *Functions Premium plan* 	Automatically scales based on demand using pre-warmed workers which run applications with no delay after being idle, runs on more powerful instances, and connects to virtual networks. 
  * *App service plan* 	Run your functions within an App Service plan at regular App Service plan rates. Best for long-running scenarios where Durable Functions can't be used.
* Azure function scales [at the unit scale of 'functin app'](https://docs.microsoft.com/en-us/learn/modules/explore-azure-functions/4-scale-azure-functions).
  * Maximum instances: A single function app only scales out to a maximum of 200 instances for Consumption plan and 100 for Premium plan. A single instance may process more than one message or request at a time though, so there isn't a set limit on number of concurrent executions. `functionAppScaleLimit` is the variable to tweak the number of instances, which can be set to 0 or *null* for unrestricted.
  * New instance rate: For HTTP triggers, new instances are allocated, at most, once per second. For non-HTTP triggers, new instances are allocated, at most, once every 30 seconds. Scaling is faster when running in a Premium plan.
  * Using an *App Service plan*, you can manually scale out by adding more VM instances. You can also enable autoscale, though autoscale will be slower than the elastic scale of the Premium plan.
* [Timer trigger](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-timer) configuration.
* [Durable Functions](https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-overview) is an extension of Azure Functions that lets you write stateful functions in a serverless compute environment. The extension lets you define stateful workflows by writing [orchestrator functions](https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-orchestrations) and [stateful entities](https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-entities) by writing entity functions using the Azure Functions programming model. Behind the scenes, the extension manages state, checkpoints, and restarts for you, allowing you to focus on your business logic.
  * Durable Functions is good for function-chain/fan-in-out/async-api/monitoring/human-interaction/aggregation etc.
  * [billing](https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-billing) is the same way as normal Functions.
  * [Durable Entities](https://markheath.net/post/durable-entities-what-are-they-good-for) can be good for things like [IoT on-offline](https://case.schollaart.net/2019/10/31/device-offline-detection-with-durable-entities.html)/[CircuitBreaker](https://dev.to/azure/serverless-circuit-breakers-with-durable-entities-3l2f) etc.
  * If you have multiple function apps sharing a shared storage account, you must explicitly [configure different names for each task hub](https://docs.microsoft.com/en-us/learn/modules/implement-durable-functions/4-durable-functions-task-hubs) in the host.json files. Otherwise the multiple function apps will compete with each other for messages, which could result in undefined behavior, including orchestrations getting unexpectedly "stuck" in the Pending or Running state.


##  misc
* ResourceGroup can be shutdown any time as long as the resources are not subject to protection
* BigData: DataLake analytics, Azure DataBricks(managed spark), HDInsight(hadoop/spark/kafka), Synapse Analytics(data warehouse) & Synapse SQL,
* Traditionally, IT expenses have been considered a [Capital Expenditure](https://tutorialsdojo.com/azure-capex-vs-opex/) (CapEx) like buying server etc. Today, with the move to the cloud and the pay-as-you-go model, organizations have the ability to stretch their budgets and are shifting their IT CapEx costs to Operating Expenditures (OpEx) instead. 
* `Azure Site Recovery` helps ensure business continuity by keeping business apps and workloads running during outages. Site Recovery replicates workloads running on physical and virtual machines (VMs) from a primary site to a secondary location.
* The `Azure Total Cost of Ownership`(TCO) Calculator is used to estimate the cost savings you can achieve by migrating your application workloads to Microsoft ..
* `Azure Cost Management` is the process of effectively planning and controlling costs involved in your business. Customers with an Azure Enterprise Agreement(EA), Microsoft Customer Agreement(MCA), or Microsoft Partner Agreement(MPA) can use Azure Cost Management.
* Traffic Manager -> Route53
