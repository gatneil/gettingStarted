Getting Started On Azure (An Unofficial Guide by a Linux User)
==============================================================

# Basic Terminology

Last updated July 7, 2016.

* [Subscriptions](https://azure.microsoft.com/en-us/documentation/articles/active-directory-how-subscriptions-associated-directory/) let you log in to Azure. You can find your subscription id in the "Subscriptions" tab of the New Portal.

* [Service Principals](https://azure.microsoft.com/en-us/documentation/articles/resource-group-create-service-principal-portal/) give you long-lasting credentials for use in an app or automated/long-running scripts. If you need a clientId/applicationId/tenantId, you get them by creating a service principal.

* [ASM vs. ARM](https://azure.microsoft.com/en-us/documentation/articles/resource-manager-deployment-model/): Azure Resource Manager (ARM) is the new stack; Azure Service Management (ASM or "classic") is the old stack. Use ARM whenever possible. If you have ASM infrastructure that you want to migrate to ARM, refer to [this documentation](https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-windows-migration-classic-resource-manager-deep-dive/). **This getting started guide deals almost exclusively with ARM.**

* [Resource Groups](https://azure.microsoft.com/en-us/documentation/articles/resource-group-overview/#resource-groups) hold resources that share the same lifecycle. I use them to delete entire projects with one command instead of manually keeping track of all the resources I create over the lifetime of a project.

* [Resource Providers](https://azure.microsoft.com/en-us/documentation/articles/resource-group-overview/#resource-providers) are services that provide resources. They look like `Microsoft.Compute`, `Microsoft.Storage`, `Microsoft.Network`, etc. A virtual machine resource looks like `Microsoft.Compute/virtualMachines`.


# Ways to access Azure

* [REST API](https://msdn.microsoft.com/en-us/library/azure/dn790568.aspx)
* [xPlat CLI](https://azure.microsoft.com/en-us/documentation/articles/xplat-cli-install/)
* [Powershell](https://azure.microsoft.com/en-us/documentation/articles/powershell-install-configure/)
* [SDKs in various languages](https://azure.microsoft.com/en-us/downloads/)
* [Old Portal](https://manage.windowsazure.com) (**only supports ASM**)
* [New Portal](https://portal.azure.com) (**supports both ASM and ARM**)
* [ARM Templates](https://azure.microsoft.com/en-us/documentation/articles/resource-group-authoring-templates/) (examples [here](https://github.com/Azure/azure-quickstart-templates))


# Scripting Example (Azure xPlat CLI)

```bash
# add -h to any command to see the help
azure -h

# log in to your account; could use a service principal here if you want
azure login

# make sure we are in ARM mode
azure config mode arm

# get list of regions we can deploy to
azure location list

# quick-create a VM (uses reasonable defaults; for more control, use `azure vm create`)
azure vm quick-create --resource-group nsgquickvmrg --name nsgquickvm --location westus --os-type Linux --image-urn UbuntuLTS --vm-size Standard_D2_v2 --admin-username negat --admin-password P4%%w0rd

#
# you can find more info about VM sizes here:
# https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-windows-sizes/
#
# commonly used VM images have aliases, which can be seen in the help text for `azure vm quick-create`.
# If you want to use an image that doesn't have an alias, you will need to use the commands
# `azure vm image publisher list` and `azure vm image list <publisher>` to find the image you want.
# The image-urn is of the form Publisher:Offer:Sku:Version (version can be `latest` if you want the latest set of patches).
# I have a script that runs daily to get the full list; it dumps the output here: http://armtg.azurewebsites.net/vm_image_list.html
#

# TODO create VM from custom image
# assumes you have defined variables AZURE_STORAGE_ACCOUNT and AZURE_STORAGE_ACCESS_KEY
azure vm quick-create --resource-group nsgquickvmrg --name nsgquickvm --location westus --os-type Linux --image-urn https://negat.blob.core.windows.net/public/ubuntu.vhd --vm-size Standard_D2_v2 --admin-username negat --admin-password P4%%w0rd

# TODO TEST
# use a custom script extension for linux to run a script on the vm
azure vm extension set --resource-group nsgquickvmrg --vm-name nsgquickvm --publisher-name Microsoft.OSTCExtensions --name CustomScriptForLinux --version 1.5 --public-config '{"fileUris": ["https://raw.githubusercontent.com/gatneil/scripts/master/hello.sh"], "commandToExecute": "sh hello.sh"}'

# TODO TEST
# if you want to pass in secrets to the script, please use the private config as such:
azure vm extension set --resource-group nsgquickvmrg --vm-name nsgquickvm --publisher-name Microsoft.OSTCExtensions --name CustomScriptForLinux --version 1.5 --public-config '{"fileUris": ["https://raw.githubusercontent.com/gatneil/scripts/master/hello.sh"]}' --private-config '{"commandToExecute": "sh hello.sh MY_AWESOME_SECRET"}'

#
# you can find more info about VM extensions here:
# https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-windows-extensions-features/
#
# each extension has a publisher, type, and a typeHandlerVersion (which is basically just the extension version). You can get the list with `azure vm extension-image list`
# I have a script that runs daily to get the full list; it dumps the output here: http://armtg.azurewebsites.net/extension_list.html
#

# VM Scale Sets (VMSS) are sets of identical VMs in a highly available configuration that can autoscale/manual scale (more info here: https://azure.microsoft.com/en-us/documentation/articles/virtual-machine-scale-sets-overview/)
azure vmss quick-create --resource-group-name nsgquickvmssrg --name nsgquickvmss --location westus --image-urn UbuntuLTS --vm-size Standard_D2_v2 --admin-username negat --admin-password P4%%w0rd --capacity 3
```

# Debugging

* [resources.azure.com](https://resources.azure.com) gives you a hierarchical view of your resources and the "instance view", which can be very helpful for debugging.
* [Audit Logs](https://azure.microsoft.com/en-us/documentation/articles/resource-group-audit/) show you all write operations performed on your resources.


# Getting help

* [MSDN Forums](https://social.msdn.microsoft.com/Forums/en-US/home?category=windowsazureplatform&filter=alltypes&sort=lastpostdesc)
* [Stack Overflow](http://stackoverflow.com/questions/tagged/azure)
* [Azure Support](https://azure.microsoft.com/en-us/support/plans/) (you can submit a support ticket from the question mark button in the new portal even without a support agreement).


# ARM Templates

ARM templates allow you to describe infrastructure and services in JSON files. You pass such a file to Azure, and Azure will deploy it for you, handling retry logic, internal throttling behavior, etc. If you redeploy the template with some changes, it will only redeploy the differences, not the whole thing. Below is a minimal template to deploy a VM. For more details, see [this documentation](https://azure.microsoft.com/en-us/documentation/articles/resource-group-authoring-templates/), and for examples see [this github repo](https://github.com/Azure/azure-quickstart-templates).

You could put the json below into a file called `azuredeploy.json`, then use the CLI command `azure group create -n RESOURCE_GROUP_NAME -d DEPLOYMENT_NAME -l REGION -f azuredeploy.json` to deploy it.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {},
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Storage/storageAccounts",
            "name": "ninjasa",
            "location": "West US",
            "apiVersion": "2015-06-15",
            "properties": { "accountType": "Standard_LRS" }
        },
        {
            "type": "Microsoft.Network/virtualNetworks",
            "name": "ninjavnet",
            "location": "West US",
            "apiVersion": "2015-06-15",
            "properties": {
                "addressSpace": { "addressPrefixes": ["10.0.0.0/16"] },
                "subnets": [
                    {
                        "name": "ninjasubnet",
                        "properties": { "addressPrefix": "10.0.0.0/24" }
                    }
                ]
            }
        },
        {
            "type": "Microsoft.Network/publicIPAddresses",
            "name": "ninjapip",
            "location": "West US",
            "apiVersion": "2015-06-15",
            "properties": {
                "publicIPAllocationMethod": "Dynamic",
                "dnsSettings": { "domainNameLabel": "ninja" }
            }
        },
        {
            "type": "Microsoft.Network/networkInterfaces",
            "name": "ninjanic",
            "location": "West US",
            "apiVersion": "2015-06-15",
            "dependsOn": [ "Microsoft.Network/publicIPAddresses/ninjapip", "Microsoft.Network/virtualNetworks/ninjavnet" ],
            "properties": {
                "ipConfigurations":
                [
                    {
                        "name": "ninjaipconfig",
                        "properties": {
                            "privateIPAllocationMethod": "Dynamic",
                            "publicIPAddress": { "id": "[resourceId('Microsoft.Network/publicIPAddresses', 'ninjapip')]" },
                            "subnet": { "id": "[concat(resourceId('Microsoft.Network/virtualNetworks', 'ninjavnet'), '/subnets/ninjasubnet')]" }
                            }
                        }
                    ]
                }
        },
        {
            "type": "Microsoft.Compute/virtualMachines",
            "name": "ninjavm",
            "location": "West US",
            "apiVersion": "2015-06-15",
            "dependsOn": ["Microsoft.Storage/storageAccounts/ninjasa", "Microsoft.Network/networkInterfaces/ninjanic"],
            "properties": {
                "hardwareProfile": { "vmSize": "Standard_D1_v2" },
                "osProfile": { "computerName": "ninjavm", "adminUsername": "me", "adminPassword": "P4$$w0rd" },
                "storageProfile": {
                    "imageReference": { "publisher": "Canonical", "offer": "UbuntuServer", "sku": "15.10", "version": "latest" },
                    "osDisk": {
                        "name": "ninjaosdisk",
                        "vhd": { "uri": "https://ninjasa.blob.core.windows.net/vhds/ninjaosdisk.vhd" },
                        "createOption": "FromImage"
                        }
                    },
                "networkProfile": {
                    "networkInterfaces": [ { "id": "[resourceId('Microsoft.Network/networkInterfaces', 'ninjanic')]" } ]
                }
            }
        }
    ],
    "outputs": {}
}
```

# Useful Miscellaneous Information

* [VM custom image creation](https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-linux-classic-create-upload-vhd/) is currently ASM-only.

* All persistent disks must go in a container within a [**storage account**](https://azure.microsoft.com/en-us/documentation/articles/storage-create-storage-account/). When you create a storage account, you must give it a name that is globally unique across all of Azure. Storage accounts can have different [replication types](https://azure.microsoft.com/en-us/documentation/articles/storage-redundancy/) (e.g. `LRS`, `ZRS`, `GRS`, `RA-GRS`) and can be either `Standard` or `Premium` (premium is faster but more expensive). To use premium storage to back VM disks, the VMs must have an `S` in the instance size (e.g. `DS1`). You probably shouldn't put more than 40 VM disks in a storage account for performance reasons. When using multiple storage accounts to get better performance, try to spread across the alphabet the prefix of the storage account name (e.g. prepending a pseudo-random hash to your storage account names). [Premium storage only supports page blobs](https://msdn.microsoft.com/en-us/library/azure/dn889922.aspx), which allows VMs to use premium storage to back VM disks, but if you want to use [other storage types](https://azure.microsoft.com/en-us/documentation/articles/storage-introduction/#introducing-the-azure-storage-services) (e.g. append blobs, block blobs, files, tables), you will have to use standard storage. **NOTE:** all of this info is relevant for `General` storage accounts. There are also [`Blob` storage accounts](https://azure.microsoft.com/en-us/documentation/articles/storage-create-storage-account/), which allow you to specify which blobs are hot and which are cold, but they support only block and append blobs.

* VMs created from a custom image end up in same storage account as the image. Since we recommend using no more than 40 VM disks in one storage account, this can become a scale limit. To get around this, replicate your vm image across multiple storage accounts. [This ARM template](https://github.com/Azure/azure-quickstart-templates/tree/master/301-custom-images-at-scale) is a fairly complex example of how to do this, but looking at the `upload.sh` script should give you an idea of what it looks like to replicate the VM image across storage accounts.

* [There are 3 kinds of disks in Azure](https://azure.microsoft.com/en-us/documentation/articles/virtual-machines-linux-about-disks-vhds/): OS disks, temporary disks, and data disks. The OS disk and data disks are provided by the `Microsoft.Storage` resource provider and are therefore persistent. **The temporary disk is physically attached to the VM host, so it is fast but NOT PERSISTENT**. In the screenshot below (from an Ubuntu 14.04-LTS machine on Azure), the temporary disk is mounted on /mnt. Looking in that directory, we see a file named `DATALOSS_WARNING_README.txt`. This isn't a joke. If you put data there, Azure can and will lose that data if it needs to migrate the VM to a new host.

```
negat@nsgubuntu:~/repos/gettingStarted$ lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0  29.3G  0 disk
└─sda1   8:1    0  29.3G  0 part /
sdb      8:16   0    50G  0 disk
└─sdb1   8:17   0    50G  0 part /mnt
sr0     11:0    1   1.1M  0 rom
negat@nsgubuntu:~/repos/gettingStarted$ ls /mnt
cdrom  DATALOSS_WARNING_README.txt  lost+found
```

* When you create a VM, all ports are open (unless you have a firewall on the VM itself). To block/allow ports, please use [network security groups](https://azure.microsoft.com/en-us/documentation/articles/virtual-networks-nsg/).

* If you issue a command to delete a VM (e.g. `azure vm delete`), it will NOT delete related resources, such as the network interface, public IP address, storage account, etc. This is by design.

* The [Azure Marketplace](https://azure.microsoft.com/en-us/marketplace/) is a place to find both first and third party solutions on Azure. Within the marketplace, there are certain solutions referred as `Marketplace Images`. **Not all solutions in the Azure Marketplace are Marketplace Images**. By default, Marketplace Images require you to click a button essentially saying "yes, I accept the legal and payment terms associated with this solution". If you want to deploy such an image programmatically, see [this documentation](https://azure.microsoft.com/en-us/blog/working-with-marketplace-images-on-azure-resource-manager/).

* When you create VMs in a virtual network, you get DNS resolution on the names of the VMs for free within the virtual network. For instance, if I make VMs with names `vm1` and `vm2` in the same virtual network and put a web server on `vm2`, then from `vm1` I can do `curl vm2` and see the web page. For more information, see [this documentation](https://azure.microsoft.com/en-us/documentation/articles/virtual-networks-name-resolution-for-vms-and-role-instances/).