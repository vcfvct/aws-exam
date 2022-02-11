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
