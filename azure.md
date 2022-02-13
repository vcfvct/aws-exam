## General
* 54 regions, 140 countries
* HA(High Availability), FT(Fault Tolerate), DA(Disaster Recovery)
  * The difference between FT and HA, is this: A FT environment has no service interruption but a significantly higher cost, while a HA environment has a minimal service interruption.
* Scalability
  * up -> increase size, but need to bring down existing one(interruptive)
  * out -> increase count, non-interruptive
* Elasticity: scale down when traffic/demand decreases.

## cloudshell
* available in powershell and  bash, no package installation allowed as `sudo` is not available.
* `az` and other things like node/python are already there.

## Azure Resource Manager(ARM) template
* ARM is the layer managing infrastructure no matter from cli/port etc.
* something similar to CloudFormation but without yaml support, json only. Can be nested
* Source control, Declarative, Idempotent.
* no similar things like CDK for now for IaS, alternative is Terraform(or CDK to Terraform).
* Every resource need to be inside a resource group.

## Networking
* Virtual Network(VNet) -> VPC, in one region, with one subscription.
  * VNet peering with backbone network. Low Latency, Link Separate Network, Transfer data between subscriptions and deployment models in separate regions.
  * NSG can be associated with Subnet in either VNet or NSG UI.
* Application Gateway -> ALB, Load Balancer -> ELB, 
* CDN: [profile + endpoint](https://docs.microsoft.com/en-us/azure/cdn/cdn-create-new-endpoint). A cdn profile is a container for CDN endpoints and specifies a pricing tier. An endpoint is where we can specify origins/pat/path and other behaviors.
* ExpressRoute -> DirectConnect. Connect to on-premises stuff with private connection instead of public Internet.

## Compute
* Scale Sets -> ASG
* App Services: PaaS with `Web Apps` for standard app or Containers or APIs, with different run time like node/php/.net etc.
* Azure Container Instances -> ECS Fargate
* AKS(k8s) + ACR(container registry)

## Storage
* Storage Account: Unique Azure namespace(Every Object in Azure has its own web address). The base of all Azure storage types.
* `Storage Account` can have multiple `Containers` which in term contains multiple `Objects`. 
  * The public url can be `https://{StorageAccountName}.blob.core.windows.net/{ContainerName}/{fileName}`.
* Blob Types: Block(up to 4.7TB), Append(Logging), Page(up to 8TB, virtual Hard Drive)
  * pricing: Hot/Cool/Archive
* Disk Types: HDD, standard/premium SSD, Ultra Disk(up to 64TB, for data intensive)
* File Storage use case: Hybrid, Lift and Shift.
* Archive -> Glacier. A `Blob` tier so same tool will work.

##  misc
* ResourceGroup can be shutdown any time as long as the resources are not subject to protection
