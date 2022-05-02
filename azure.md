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
* Archive -> Glacier. A `Blob` tier so same tool will work. 
  * To read or download a blob in archive, you must first rehydrate it to an online tier.
* Azure Storage redundancy
  * Locally-redundant storage, LRS (replicated in same data center)
  * Zone-redundant storage, ZRS (replicated in different Data center within same AZ)
  * Geo-redundant storage (GRS): primary region with LRS
  * Geo-zone-redundant storage (GZRS): primary region with ZRS
  * For *read* access to the secondary region, configure your storage account to use read-access geo-redundant storage (RA-GRS) or read-access geo-zone-redundant storage (RA-GZRS). 
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
* Azure SQL: managed SQL-Server, up to 100TB.
* Managed MySQL/PostgreSQL and DMS

## Messaging
* Azure Event Hubs is a big data streaming platform and event ingestion service. It can receive and process millions of events per second. Data sent to an event hub can be transformed and stored by using any real-time analytics provider or batching/storage adapters.

## AuthN & AuthR
* Azure Active Directory(AAD) 
  * manage users and permissions.
  * AAD is completely different from Active Directory. It is mandatory(cannot use Azure account without AAD service).
  * Tenant and User(member of up to 500 tenants) are many to many.
  * done with Group/Member/Role etc.

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

##  misc
* ResourceGroup can be shutdown any time as long as the resources are not subject to protection
* Serverless: Azure functions, App Logic(step functions?), EventGrid(-> EventBridge?)
* BigData: DataLake analytics, Azure DataBricks(managed spark), HDInsight(hadoop/spark/kafka), Synapse Analytics(data warehouse) & Synapse SQL,
* Traditionally, IT expenses have been considered a [Capital Expenditure](https://tutorialsdojo.com/azure-capex-vs-opex/) (CapEx) like buying server etc. Today, with the move to the cloud and the pay-as-you-go model, organizations have the ability to stretch their budgets and are shifting their IT CapEx costs to Operating Expenditures (OpEx) instead. 
* `Azure Site Recovery` helps ensure business continuity by keeping business apps and workloads running during outages. Site Recovery replicates workloads running on physical and virtual machines (VMs) from a primary site to a secondary location.
* The `Azure Total Cost of Ownership`(TCO) Calculator is used to estimate the cost savings you can achieve by migrating your application workloads to Microsoft ..
* `Azure Cost Management` is the process of effectively planning and controlling costs involved in your business. Customers with an Azure Enterprise Agreement(EA), Microsoft Customer Agreement(MCA), or Microsoft Partner Agreement(MPA) can use Azure Cost Management.
* Traffic Manager -> Route53
