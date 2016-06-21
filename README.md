Getting Started On Azure (An Unofficial Guide by a Linux User)
==============================================================

# Basic Terminology

Last updated June 21, 2016.

* [Subscriptions](https://azure.microsoft.com/en-us/documentation/articles/active-directory-how-subscriptions-associated-directory/) let you log in to Azure.

* [Service Principals](https://azure.microsoft.com/en-us/documentation/articles/resource-group-create-service-principal-portal/) give you long-lasting credentials for use in an app or automated/long-running scripts.

* [ASM vs. ARM](https://azure.microsoft.com/en-us/documentation/articles/resource-manager-deployment-model/): Azure Resource Manager (ARM) is the new stack; Azure Service Management (ASM or "classic") is the old stack. Use ARM whenever possible. If you have ASM infrastructure that you want to migrate to ARM, refer to [this documentation](https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-windows-migration-classic-resource-manager-deep-dive/). **This getting started guide is ARM-centric.**

* [Resource Groups](https://azure.microsoft.com/en-us/documentation/articles/resource-group-overview/#resource-groups) hold resources that share the same lifecycle. I use them to delete entire projects worth of infrastructure with one command instead of manually keeping track of all the resources I create over the lifetime of a project.


# Ways to access Azure

* [REST API](https://msdn.microsoft.com/en-us/library/azure/dn790568.aspx)
* [xPlat CLI](https://azure.microsoft.com/en-us/documentation/articles/xplat-cli-install/)
* [Powershell](https://azure.microsoft.com/en-us/documentation/articles/powershell-install-configure/)
* [SDKs in various languages](https://azure.microsoft.com/en-us/downloads/)
* [Old Portal](https://manage.windowsazure.com) (**only supports ASM**)
* [New Portal](https://portal.azure.com) (**supports both ASM and ARM**)
* [ARM Templates](https://azure.microsoft.com/en-us/documentation/articles/resource-group-authoring-templates/)


# Scripting Example

```bash
# add -h to any command to see the help
azure -h

# log in to your account; could use a service principal here if you want
azure login

# make sure we are in ARM mode
azure config mode arm

# TODO how to get list of regions

# TODO make these use command line arguments instead of prompts

# TODO how to find URN; link to page listing images
azure vm quick-create

# TODO create VM from custom image

# TODO show adding extension; link to page listing extensions

# TODO explanation
azure vmss quick-create
```


# ARM Templates

TODO


# Potential Surprises

AAD and vm image creation are currently ASM-only.

[Storage Accounts](https://azure.microsoft.com/en-us/documentation/articles/storage-create-storage-account/): All persistent disks must go in a container within a storage account. Storage accounts can have different replication types and can be either `Standard` or `Premium` (premium is faster but more expensive). To use premium storage to back VM disks, the VMs must have an `S` in the instance size (e.g. `DS1`). You probably shouldn't put more than 40 VM disks in a storage account for performance reasons. When using multiple storage accounts to get better performance, try to spread across the alphabet the prefix of the storage account name (e.g. prepending a pseudo-random hash to your storage account names).

TODO VMs created from custom image end up in same SA as image.

TODO temporary disk

TODO Networking; VMs open if no NSGs specified

TODO Deleting a VM does not delete related resources (NIC, PIP, SA, etc.)

TODO special steps for deploying marketplace images (e.g. Bitnami)