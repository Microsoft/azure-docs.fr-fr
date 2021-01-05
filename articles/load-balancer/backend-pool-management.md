---
title: Gestion du pool back-end
titleSuffix: Azure Load Balancer
description: Découvrez comment configurer et gérer le pool back-end d’un équilibreur de charge Azure.
services: load-balancer
author: asudbring
ms.service: load-balancer
ms.topic: how-to
ms.date: 07/07/2020
ms.author: allensu
ms.openlocfilehash: 8887474f07928462afe7863ffe2b3667ece536dc
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96575297"
---
# <a name="backend-pool-management"></a>Gestion du pool back-end
Le pool back-end est un composant essentiel de l’équilibreur de charge. Il définit le groupe de ressources qui va servir le trafic pour une règle d’équilibrage de charge donnée.

Il existe deux façons de configurer un pool back-end :
* Carte d’interface réseau (NIC)
* Combinaison de l’adresse IP et de l’ID de la ressource du réseau virtuel (VNET)

Configurez votre pool de back-ends par carte réseau quand vous utilisez des machines virtuelles et des groupes de machines virtuelles identiques existants. Cette méthode génère le lien le plus direct entre votre ressource et le pool back-end. 

Lors de la préallocation de votre pool de back-ends avec une plage d’adresses IP avec laquelle vous envisagez de créer ultérieurement des machines virtuelles et des groupes de machines virtuelles identiques, configurez votre pool de back-ends par une combinaison d’adresses IP et d’ID de réseau virtuel.

Les sections de configuration de cet article se concentrent sur :

* Azure PowerShell
* Azure CLI
* API REST
* Modèles Microsoft Azure Resource Manager 

Ces sections permettent de comprendre comment les pools back-end sont structurés pour chaque option de configuration.

## <a name="configuring-backend-pool-by-nic"></a>Configuration du pool back-end par carte réseau
Le pool back-end est créé dans le cadre de l’opération de l’équilibreur de charge. La propriété de configuration IP de la carte réseau est utilisée pour ajouter des membres de pool back-end.

Les exemples suivants se concentrent sur les opérations de création et de remplissage du pool back-end afin de mettre en évidence ce workflow et cette relation.

  >[!NOTE] 
  >Il est important de noter que les pools back-end configurés par le biais de l’interface réseau ne peuvent pas être mis à jour dans le cadre d’une opération sur un pool back-end. Tout ajout ou suppression de ressources back-end doivent se produire sur l’interface réseau de la ressource.

### <a name="powershell"></a>PowerShell
Créez un pool back-end :
 
```azurepowershell-interactive
$resourceGroup = "myResourceGroup"
$loadBalancerName = "myLoadBalancer"
$backendPoolName = "myBackendPool"

$backendPool = 
New-AzLoadBalancerBackendAddressPool -ResourceGroupName $resourceGroup -LoadBalancerName $loadBalancerName -BackendAddressPoolName $backendPoolName  
```

Créez une interface réseau et ajoutez-la au pool back-end :

```azurepowershell-interactive
$resourceGroup = "myResourceGroup"
$loadBalancerName = "myLoadBalancer"
$backendPoolName = "myBackendPool"
$nicname = "myNic"
$location = "eastus"
$vnetname = <your-vnet-name>

$vnet = 
Get-AzVirtualNetwork -Name $vnetname -ResourceGroupName $resourceGroup

$nic = 
New-AzNetworkInterface -ResourceGroupName $resourceGroup -Location $location -Name $nicname -LoadBalancerBackendAddressPool $backendPoolName -Subnet $vnet.Subnets[0]
```

Récupérez les informations du pool back-end pour que l’équilibreur de charge confirme que cette interface réseau est ajoutée au pool back-end :

```azurepowershell-interactive
$resourceGroup = "myResourceGroup"
$loadBalancerName = "myLoadBalancer"
$backendPoolName = "myBackendPool"

$lb =
Get-AzLoadBalancer -ResourceGroupName $res
Get-AzLoadBalancerBackendAddressPool -ResourceGroupName $resourceGroup -LoadBalancerName $loadBalancerName -BackendAddressPoolName $backendPoolName 
```

Créez une machine virtuelle et attachez l’interface réseau pour la placer dans le pool back-end :

```azurepowershell-interactive
# Create a username and password for the virtual machine
$cred = Get-Credential

# Create a virtual machine configuration
$vmname = "myVM1"
$vmsize = "Standard_DS1_v2"
$pubname = "MicrosoftWindowsServer"
$nicname = "myNic"
$off = "WindowsServer"
$sku = "2019-Datacenter"
$resourceGroup = "myResourceGroup"
$location = "eastus"

$nic =
Get-AzNetworkInterface -Name $nicname -ResourceGroupName $resourceGroup

$vmConfig = 
New-AzVMConfig -VMName $vmname -VMSize $vmsize | Set-AzVMOperatingSystem -Windows -ComputerName $vmname -Credential $cred | Set-AzVMSourceImage -PublisherName $pubname -Offer $off -Skus $sku -Version latest | Add-AzVMNetworkInterface -Id $nic.Id
 
# Create a virtual machine using the configuration
$vm1 = New-AzVM -ResourceGroupName $resourceGroup -Zone 1 -Location $location -VM $vmConfig
```

### <a name="cli"></a>Interface de ligne de commande
Créez le pool back-end :

```azurecli-interactive
az network lb address-pool create \
--resource-group myResourceGroup \
--lb-name myLB \
--name myBackendPool 
```

Créez une interface réseau et ajoutez-la au pool back-end :

```azurecli-interactive
az network nic create \
--resource-group myResourceGroup \
--name myNic \
--vnet-name myVnet \
--subnet mySubnet \
--network-security-group myNetworkSecurityGroup \
--lb-name myLB \
--lb-address-pools myBackEndPool
```

Récupérez le pool back-end pour confirmer que l’adresse IP a été correctement ajoutée :

```azurecli-interactive
az network lb address-pool show \
--resource-group myResourceGroup \
--lb-name myLb \
--name myBackendPool
```

Créez une machine virtuelle et attachez l’interface réseau pour la placer dans le pool back-end :

```azurecli-interactive
az vm create \
--resource-group myResourceGroup \
--name myVM \
--nics myNic \
--image UbuntuLTS \
--admin-username azureuser \
--generate-ssh-keys
```

### <a name="rest-api"></a>API REST
Créez le pool back-end :

```
PUT https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Network/loadBalancers/{load-balancer-name}/backendAddressPools/{backend-pool-name}?api-version=2020-05-01
```

Créez une interface réseau et ajoutez-la au pool back-end que vous avez créé par le biais de la propriété des configurations IP de l’interface réseau :

```
PUT https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Network/networkInterfaces/{nic-name}?api-version=2020-05-01
```

Corps de la demande JSON :
```json
{
  "properties": {
    "enableAcceleratedNetworking": true,
    "ipConfigurations": [
      {
        "name": "ipconfig1",
        "properties": {
          "subnet": {
            "id": "/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Network/virtualNetworks/{vnet-name}/subnets/{subnet-name}"
          },
          "loadBalancerBackendAddressPools": {
                                    "id": "/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Network/loadBalancers/{load-balancer-name}/backendAddressPools/{backend-pool-name}"
          }
        }
      }
    ]
  },
  "location": "eastus"
}
```

Récupérez les informations du pool back-end pour que l’équilibreur de charge confirme que cette interface réseau est ajoutée au pool back-end :

```
GET https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name/providers/Microsoft.Network/loadBalancers/{load-balancer-name/backendAddressPools/{backend-pool-name}?api-version=2020-05-01
```

Créez une machine virtuelle et attachez la carte réseau qui référence le pool back-end :

```
PUT https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Compute/virtualMachines/{vm-name}?api-version=2019-12-01
```

Corps de la demande JSON :
```JSON
{
  "location": "easttus",
  "properties": {
    "hardwareProfile": {
      "vmSize": "Standard_D1_v2"
    },
    "storageProfile": {
      "imageReference": {
        "sku": "2016-Datacenter",
        "publisher": "MicrosoftWindowsServer",
        "version": "latest",
        "offer": "WindowsServer"
      },
      "osDisk": {
        "caching": "ReadWrite",
        "managedDisk": {
          "storageAccountType": "Standard_LRS"
        },
        "name": "myVMosdisk",
        "createOption": "FromImage"
      }
    },
    "networkProfile": {
      "networkInterfaces": [
        {
          "id": "/subscriptions/{subscription-id}/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/{nic-name}",
          "properties": {
            "primary": true
          }
        }
      ]
    },
    "osProfile": {
      "adminUsername": "{your-username}",
      "computerName": "myVM",
      "adminPassword": "{your-password}"
    }
  }
}
```

### <a name="resource-manager-template"></a>Modèle Resource Manager
Suivez ce [modèle Resource Manager de démarrage rapide](https://github.com/Azure/azure-quickstart-templates/tree/master/101-load-balancer-standard-create/) pour déployer un équilibreur de charge et des machines virtuelles, puis ajouter les machines virtuelles au pool back-end par le biais d’une interface réseau.

## <a name="configure-backend-pool-by-ip-address-and-virtual-network"></a>Configurer le pool back-end par adresse IP et réseau virtuel
Dans les scénarios avec des pools de back-ends préremplis, utilisez l’adresse IP et le réseau virtuel.

La gestion de tous les pools back-end s’effectue directement sur l’objet de pool back-end, comme indiqué dans les exemples ci-dessous.

  >[!IMPORTANT] 
  >Actuellement, cette fonctionnalité est uniquement disponible en tant que version préliminaire. Consultez la [section Limites](#limitations) pour connaître les limites actuelles de cette fonctionnalité.

### <a name="powershell"></a>PowerShell
Créez un pool back-end :

```azurepowershell-interactive
$resourceGroup = "myResourceGroup"
$loadBalancerName = "myLoadBalancer"
$backendPoolName = "myBackendPool"
$vnetName = "myVnet"
$location = "eastus"
$nicName = "myNic"

$backendPool = New-AzLoadBalancerBackendAddressPool -ResourceGroupName $resourceGroup -LoadBalancerName $loadBalancerName -Name $backendPoolName  
```

Mettez à jour le pool back-end avec une nouvelle adresse IP du réseau virtuel existant :
 
```azurepowershell-interactive
$virtualNetwork = 
Get-AzVirtualNetwork -Name $vnetName -ResourceGroupName $resourceGroup 
 
$ip1 = New-AzLoadBalancerBackendAddressConfig -IpAddress "10.0.0.5" -Name "TestVNetRef" -VirtualNetwork $virtualNetwork  
 
$backendPool.LoadBalancerBackendAddresses.Add($ip1) 

Set-AzLoadBalancerBackendAddressPool -InputObject $backendPool
```

Récupérez les informations du pool back-end pour que l’équilibreur de charge confirme que les adresses back-end sont ajoutées au pool back-end :

```azurepowershell-interactive
Get-AzLoadBalancerBackendAddressPool -ResourceGroupName $resourceGroup -LoadBalancerName $loadBalancerName -Name $backendPoolName 
```
Créez une interface réseau et ajoutez-la au pool back-end. Définissez l’adresse IP sur l’une des adresses back-end :

```azurepowershell-interactive
$nic = 
New-AzNetworkInterface -ResourceGroupName $resourceGroup -Location $location -Name $nicName -PrivateIpAddress 10.0.0.4 -Subnet $virtualNetwork.Subnets[0]
```

Créez une machine virtuelle et attachez la carte réseau avec une adresse IP dans le pool back-end :
```azurepowershell-interactive
# Create a username and password for the virtual machine
$cred = Get-Credential

# Create a virtual machine configuration
$vmname = "myVM1"
$vmsize = "Standard_DS1_v2"
$pubname = "MicrosoftWindowsServer"
$nicname = "myNic"
$off = "WindowsServer"
$sku = "2019-Datacenter"
$resourceGroup = "myResourceGroup"
$location = "eastus"

$nic =
Get-AzNetworkInterface -Name $nicname -ResourceGroupName $resourceGroup

$vmConfig = 
New-AzVMConfig -VMName $vmname -VMSize $vmsize | Set-AzVMOperatingSystem -Windows -ComputerName $vmname -Credential $cred | Set-AzVMSourceImage -PublisherName $pubname -Offer $off -Skus $sku -Version latest | Add-AzVMNetworkInterface -Id $nic.Id

# Create a virtual machine using the configuration
$vm1 = New-AzVM -ResourceGroupName $resourceGroup -Zone 1 -Location $location -VM $vmConfig
```

### <a name="cli"></a>Interface de ligne de commande
À l’aide de l’interface CLI, vous pouvez remplir le pool back-end par le biais de paramètres de ligne de commande ou d’un fichier de configuration JSON. 

Créez et remplissez le pool back-end par le biais de paramètres de ligne de commande :

```azurecli-interactive
az network lb address-pool create \
--resource-group myResourceGroup \
--lb-name myLB \
--name myBackendPool \
--vnet {VNET resource ID} \
--backend-address name=addr1 ip-address=10.0.0.4 \
--backend-address name=addr2 ip-address=10.0.0.5
```

Créez et remplissez le pool back-end par le biais d’un fichier de configuration JSON :

```azurecli-interactive
az network lb address-pool create \
--resource-group myResourceGroup \
--lb-name myLB \
--name myBackendPool \
--vnet {VNET resource ID} \
--backend-address-config-file @config_file.json
```

Fichier de configuration JSON :
```JSON
        [
          {
            "name": "address1",
            "virtualNetwork": "/subscriptions/{subscriptionId}/resourceGroups/{resource-group-name}/providers/Microsoft.Network/virtualNetworks/{vnet-name}",
            "ipAddress": "10.0.0.4"
          },
          {
            "name": "address2",
            "virtualNetwork": "/subscriptions/{subscriptionId}/resourceGroups/{resource-group-name}/providers/Microsoft.Network/virtualNetworks/{vnet-name}",
            "ipAddress": "10.0.0.5"
          }
        ]
```

Récupérez les informations du pool back-end pour que l’équilibreur de charge confirme que les adresses back-end sont ajoutées au pool back-end :

```azurecli-interactive
az network lb address-pool show \
--resource-group myResourceGroup \
--lb-name MyLb \
--name MyBackendPool
```

Créez une interface réseau et ajoutez-la au pool back-end. Définissez l’adresse IP sur l’une des adresses back-end :

```azurecli-interactive
az network nic create \
  --resource-group myResourceGroup \
  --name myNic \
  --vnet-name myVnet \
  --subnet mySubnet \
  --network-security-group myNetworkSecurityGroup \
  --lb-name myLB \
  --private-ip-address 10.0.0.4
```

Créez une machine virtuelle et attachez la carte réseau avec une adresse IP dans le pool back-end :

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --nics myNic \
  --image UbuntuLTS \
  --admin-username azureuser \
  --generate-ssh-keys
```

### <a name="rest-api"></a>API REST

Créez le pool back-end et définissez les adresses back-end par le biais d’une demande de pool back-end PUT. Configurez les adresses back-end dans le corps JSON de la demande PUT par :

* Nom d’adresse
* Adresse IP
* ID du réseau virtuel 

```
PUT https://management.azure.com/subscriptions/subid/resourceGroups/testrg/providers/Microsoft.Network/loadBalancers/lb/backendAddressPools/backend?api-version=2020-05-01
```

Corps de la demande JSON :
```JSON
{
  "properties": {
    "loadBalancerBackendAddresses": [
      {
        "name": "address1",
        "properties": {
          "virtualNetwork": {
            "id": "/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Network/virtualNetworks/{vnet-name}"
          },
          "ipAddress": "10.0.0.4"
        }
      },
      {
        "name": "address2",
        "properties": {
          "virtualNetwork": {
            "id": "/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Network/virtualNetworks/{vnet-name}"
          },
          "ipAddress": "10.0.0.5"
        }
      }
    ]
  }
}
```

Récupérez les informations du pool back-end pour que l’équilibreur de charge confirme que les adresses back-end sont ajoutées au pool back-end :
```
GET https://management.azure.com/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Network/loadBalancers/{load-balancer-name}/backendAddressPools/{backend-pool-name}?api-version=2020-05-01
```

Créez une interface réseau et ajoutez-la au pool back-end. Définissez l’adresse IP sur l’une des adresses back-end :
```
PUT https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Network/networkInterfaces/{nic-name}?api-version=2020-05-01
```

Corps de la demande JSON :
```JSON
{
  "properties": {
    "enableAcceleratedNetworking": true,
    "ipConfigurations": [
      {
        "name": "ipconfig1",
        "properties": {
          "privateIPAddress": "10.0.0.4",
          "subnet": {
            "id": "/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Network/virtualNetworks/{vnet-name}/subnets/{subnet-name}"
          }
        }
      }
    ]
  },
  "location": "eastus"
}
```

Créez une machine virtuelle et attachez la carte réseau avec une adresse IP dans le pool back-end :

```
PUT https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Compute/virtualMachines/{vm-name}?api-version=2019-12-01
```

Corps de la demande JSON :
```JSON
{
  "location": "eastus",
  "properties": {
    "hardwareProfile": {
      "vmSize": "Standard_D1_v2"
    },
    "storageProfile": {
      "imageReference": {
        "sku": "2016-Datacenter",
        "publisher": "MicrosoftWindowsServer",
        "version": "latest",
        "offer": "WindowsServer"
      },
      "osDisk": {
        "caching": "ReadWrite",
        "managedDisk": {
          "storageAccountType": "Standard_LRS"
        },
        "name": "myVMosdisk",
        "createOption": "FromImage"
      }
    },
    "networkProfile": {
      "networkInterfaces": [
        {
          "id": "/subscriptions/{subscription-id}/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/{nic-name}",
          "properties": {
            "primary": true
          }
        }
      ]
    },
    "osProfile": {
      "adminUsername": "{your-username}",
      "computerName": "myVM",
      "adminPassword": "{your-password}"
    }
  }
}
```

## <a name="limitations"></a>Limites
Un pool de back-ends configuré par adresse IP présente les limites suivantes :
  * Équilibreur de charge standard uniquement.
  * Limite de 100 adresses IP dans le pool de back-ends
  * Les ressources back-end doivent être dans le même réseau virtuel que celui de l’équilibreur de charge.
  * Un équilibreur de charge avec un pool de back-ends basé sur IP ne peut pas fonctionner en tant que service Private Link
  * Cette fonctionnalité n’est actuellement pas prise en charge dans le portail Azure.
  * Les conteneurs ACI ne sont pas pris en charge par cette fonctionnalité
  * Vous ne pouvez pas placer des équilibreurs de charge ou des services mis en front-end par les équilibreurs de charge dans le pool de back-ends de l’équilibreur de charge
  * Les règles NAT de trafic entrant ne peuvent pas être spécifiées par adresse IP
  
## <a name="next-steps"></a>Étapes suivantes
Dans cet article, vous avez découvert la gestion du pool back-end Azure Load Balancer et vous avez appris à configurer un pool back-end par adresse IP et réseau virtuel.

En savoir plus sur [Azure Load Balancer](load-balancer-overview.md).
