Getting Started On Azure (An Unofficial Guide by a Linux User)
==============================================================

# Basics

TODO

## Subscriptions

TODO

## ASM vs. ARM

TODO

## Resource Groups

TODO

## Service Principals

TODO

## Ways to access Azure

* [REST API](https://msdn.microsoft.com/en-us/library/azure/dn790568.aspx)
* [xPlat CLI](https://azure.microsoft.com/en-us/documentation/articles/xplat-cli-install/)
* [Powershell](https://azure.microsoft.com/en-us/documentation/articles/powershell-install-configure/)
* [SDKs in various languages](https://azure.microsoft.com/en-us/downloads/)
* [Old Portal](https://manage.windowsazure.com) (**only supports ASM**)
* [New Portal](https://portal.azure.com) (**supports both ASM and ARM**)
* [ARM Templates](https://azure.microsoft.com/en-us/documentation/articles/resource-group-authoring-templates/)

# Scripting Examples

```bash
# add -h to any command to see the help
azure -h

# make sure we are in ARM mode
azure config mode arm

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

TODO Storage Accounts

TODO Networking; VMs open if no NSGs specified

TODO Deleting a VM does not delete related resources (NIC, PIP, SA, etc.)

TODO special steps for deploying marketplace images (e.g. Bitnami)