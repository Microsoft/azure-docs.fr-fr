---
title: Plusieurs adresses IP pour les machines virtuelles Azure - PowerShell | Microsoft Docs
description: Découvrez comment attribuer plusieurs adresses IP à une machine virtuelle à l’aide de PowerShell. | Resource Manager
services: virtual-network
documentationcenter: na
author: asudbring
manager: KumudD
ms.service: virtual-network
ms.subservice: ip-services
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/24/2017
ms.author: allensu
ms.openlocfilehash: 5cd050c88fbc954a211c3a75cdabcb557ae998c4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87073920"
---
# <a name="assign-multiple-ip-addresses-to-virtual-machines-using-powershell"></a>Affecter plusieurs adresses IP à des machines virtuelles avec PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[!INCLUDE [virtual-network-multiple-ip-addresses-intro.md](../../includes/virtual-network-multiple-ip-addresses-intro.md)]

Cet article explique comment créer une machine virtuelle dans le modèle de déploiement Azure Resource Manager à l’aide de PowerShell. Il n’est pas possible d’affecter plusieurs adresses IP à des ressources créées à l’aide du modèle de déploiement classique. Pour en savoir plus sur les modèles de déploiement Azure, voir [Comprendre les modèles de déploiement](../resource-manager-deployment-model.md).

[!INCLUDE [virtual-network-multiple-ip-addresses-scenario.md](../../includes/virtual-network-multiple-ip-addresses-scenario.md)]

## <a name="create-a-vm-with-multiple-ip-addresses"></a><a name = "create"></a>Créer une machine virtuelle avec plusieurs adresses IP

Les étapes qui suivent expliquent comment créer un exemple de machine virtuelle avec plusieurs adresses IP, comme décrit dans le scénario. Modifiez les valeurs des variables en fonction des besoins de votre implémentation.

1. Ouvrez une invite de commandes PowerShell et effectuez les étapes restantes de cette section dans une même session PowerShell. Si vous n’avez pas installé, ni configuré PowerShell, effectuez les étapes de l’article [Installation et configuration d’Azure PowerShell](/powershell/azure/) .
2. Connectez-vous à votre compte avec la commande `Connect-AzAccount`.
3. Remplacez *myResourceGroup* et *westus* par le nom et l’emplacement de votre choix. Créez un groupe de ressources. Un groupe de ressources est un conteneur logique dans lequel les ressources Azure sont déployées et gérées.

   ```powershell
   $RgName   = "MyResourceGroup"
   $Location = "westus"

   New-AzResourceGroup `
   -Name $RgName `
   -Location $Location
   ```

4. Créez un réseau virtuel (VNet) et un sous-réseau dans le même emplacement que le groupe de ressources :

   ```powershell

   # Create a subnet configuration
   $SubnetConfig = New-AzVirtualNetworkSubnetConfig `
   -Name MySubnet `
   -AddressPrefix 10.0.0.0/24

   # Create a virtual network
   $VNet = New-AzVirtualNetwork `
   -ResourceGroupName $RgName `
   -Location $Location `
   -Name MyVNet `
   -AddressPrefix 10.0.0.0/16 `
   -Subnet $subnetConfig

   # Get the subnet object
   $Subnet = Get-AzVirtualNetworkSubnetConfig -Name $SubnetConfig.Name -VirtualNetwork $VNet
   ```

5. Créez un groupe de sécurité réseau (NSG) et une règle. Le groupe de sécurité réseau permet de sécuriser la machine virtuelle à l’aide de règles entrantes et sortantes. Dans ce cas, une règle entrante est créée pour le port 3389, qui autorise les connexions Bureau à distance entrantes.

    ```powershell
    
    # Create an inbound network security group rule for port 3389

    $NSGRule = New-AzNetworkSecurityRuleConfig `
    -Name MyNsgRuleRDP `
    -Protocol Tcp `
    -Direction Inbound `
    -Priority 1000 `
    -SourceAddressPrefix * `
    -SourcePortRange * `
    -DestinationAddressPrefix * `
    -DestinationPortRange 3389 -Access Allow
    
    # Create a network security group
    $NSG = New-AzNetworkSecurityGroup `
    -ResourceGroupName $RgName `
    -Location $Location `
    -Name MyNetworkSecurityGroup `
    -SecurityRules $NSGRule
    ```

6. Définissez la configuration IP principale de la carte réseau. Remplacez 10.0.0.4 par une adresse valide dans le sous-réseau que vous avez créé, si vous n’utilisez pas la valeur définie précédemment. Avant d’attribuer une adresse IP statique, il est recommandé de vérifier tout d’abord qu’elle n’est pas déjà utilisée. Entrez la commande `Test-AzPrivateIPAddressAvailability -IPAddress 10.0.0.4 -VirtualNetwork $VNet`. Si l’adresse est disponible, la sortie retourne *True*. S’il n’est pas disponible, la sortie retourne *False* et la liste des adresses disponibles. 

    Dans les commandes suivantes, **remplacez \<replace-with-your-unique-name> par le nom DNS unique à utiliser.** Le nom doit être unique sur toutes les adresses IP publiques au sein d’une région Azure. Il s'agit d'un paramètre facultatif. Il peut être supprimé si vous voulez uniquement vous connecter à la machine virtuelle à l’aide de l’adresse IP publique.

    ```powershell
    
    # Create a public IP address
    $PublicIP1 = New-AzPublicIpAddress `
    -Name "MyPublicIP1" `
    -ResourceGroupName $RgName `
    -Location $Location `
    -DomainNameLabel <replace-with-your-unique-name> `
    -AllocationMethod Static
        
    #Create an IP configuration with a static private IP address and assign the public IP address to it
    $IpConfigName1 = "IPConfig-1"
    $IpConfig1     = New-AzNetworkInterfaceIpConfig `
    -Name $IpConfigName1 `
    -Subnet $Subnet `
    -PrivateIpAddress 10.0.0.4 `
    -PublicIpAddress $PublicIP1 `
    -Primary
    ```

    Lorsque vous affectez plusieurs configurations IP à une interface réseau, une configuration doit être affectée en tant que configuration *principale*.

    > [!NOTE]
    > Les adresses IP publiques ont un coût nominal. Pour en savoir plus, lisez la page [Tarification des adresses IP](https://azure.microsoft.com/pricing/details/ip-addresses) . Il existe une limite au nombre d’adresses IP publiques qui peuvent être utilisées dans un abonnement. Pour plus d’informations sur les limites, voir [Limites d’Azure](../azure-resource-manager/management/azure-subscription-service-limits.md#networking-limits).

7. Définissez les configurations IP secondaires pour la carte réseau. Vous pouvez ajouter ou supprimer des configurations si nécessaire. Chaque configuration IP doit avoir une adresse IP privée attribuée. Chaque configuration peut avoir une adresse IP publique attribuée.

    ```powershell
    
    # Create a public IP address
    $PublicIP2 = New-AzPublicIpAddress `
    -Name "MyPublicIP2" `
    -ResourceGroupName $RgName `
    -Location $Location `
    -AllocationMethod Static
        
    #Create an IP configuration with a static private IP address and assign the public IP address to it
    $IpConfigName2 = "IPConfig-2"
    $IpConfig2     = New-AzNetworkInterfaceIpConfig `
    -Name $IpConfigName2 `
    -Subnet $Subnet `
    -PrivateIpAddress 10.0.0.5 `
    -PublicIpAddress $PublicIP2
        
    $IpConfigName3 = "IpConfig-3"
    $IpConfig3 = New-AzNetworkInterfaceIpConfig `
    -Name $IPConfigName3 `
    -Subnet $Subnet `
    -PrivateIpAddress 10.0.0.6
    ```

8. Créez la carte réseau et associez-lui les trois configurations IP :

   ```powershell
   $NIC = New-AzNetworkInterface `
   -Name MyNIC `
   -ResourceGroupName $RgName `
   -Location $Location `
   -NetworkSecurityGroupId $NSG.Id `
   -IpConfiguration $IpConfig1,$IpConfig2,$IpConfig3
   ```

   >[!NOTE]
   >Bien que toutes les configurations soient attribuées à une carte réseau dans cet article, vous pouvez attribuer plusieurs configurations IP à chaque carte réseau connectée à la machine virtuelle. Pour apprendre à créer une machine virtuelle avec plusieurs cartes réseau, consultez l’article [Créer une machine virtuelle avec plusieurs cartes d’interface réseau](../virtual-machines/windows/multiple-nics.md).

9. Créez la machine virtuelle en entrant les commandes suivantes :

    ```powershell
    
    # Define a credential object. When you run these commands, you're prompted to enter a username and password for the VM you're creating.
    $cred = Get-Credential
    
    # Create a virtual machine configuration
    $VmConfig = New-AzVMConfig `
    -VMName MyVM `
    -VMSize Standard_DS1_v2 | `
    Set-AzVMOperatingSystem -Windows `
    -ComputerName MyVM `
    -Credential $cred | `
    Set-AzVMSourceImage `
    -PublisherName MicrosoftWindowsServer `
    -Offer WindowsServer `
    -Skus 2016-Datacenter `
    -Version latest | `
    Add-AzVMNetworkInterface `
    -Id $NIC.Id
    
    # Create the VM
    New-AzVM `
    -ResourceGroupName $RgName `
    -Location $Location `
    -VM $VmConfig
    ```

10. Ajoutez les adresses IP privées pour le système d’exploitation de la machine virtuelle en suivant les étapes pour votre système d’exploitation dans la section [Ajouter des adresses IP à un système d’exploitation de machine virtuelle](#os-config) de cet article. N’ajoutez pas les adresses IP publiques au système d’exploitation.

## <a name="add-ip-addresses-to-a-vm"></a><a name="add"></a>Ajouter des adresses IP à une machine virtuelle

Vous pouvez ajouter des adresses IP privées et publiques à l’interface réseau Azure en suivant les étapes décrites ci-après. Les exemples fournis dans les sections suivantes supposent que vous disposez déjà d’une machine virtuelle avec les trois configurations IP décrites dans le [scénario](#scenario) de cet article, mais ce n’est pas une condition obligatoire.

1. Ouvrez une invite de commandes PowerShell et effectuez les étapes restantes de cette section dans une même session PowerShell. Si vous n’avez pas installé, ni configuré PowerShell, effectuez les étapes de l’article [Installation et configuration d’Azure PowerShell](/powershell/azure/) .
2. Remplacez les « valeurs » des $Variables suivantes par le nom de l’interface réseau à laquelle vous voulez ajouter une adresse IP, et par le groupe de ressources et l’emplacement dans lesquels réside l’interface réseau :

   ```powershell
   $NicName  = "MyNIC"
   $RgName   = "MyResourceGroup"
   $Location = "westus"
   ```

   Si vous ne connaissez pas le nom de l’interface réseau que vous souhaitez modifier, entrez les commandes suivantes, puis modifiez les valeurs des variables précédentes :

   ```powershell
   Get-AzNetworkInterface | Format-Table Name, ResourceGroupName, Location
   ```

3. Créez une variable et affectez-la à l’interface réseau à l’aide de la commande suivante :

   ```powershell
   $MyNIC = Get-AzNetworkInterface -Name $NicName -ResourceGroupName $RgName
   ```

4. Dans les commandes suivantes, remplacez *MyVNet* et *MySubnet* par les noms du réseau virtuel et du sous-réseau auxquels la carte réseau est connectée. Entrez les commandes pour récupérer les objets réseau virtuel et sous-réseau auxquels la carte réseau est connectée :

   ```powershell
   $MyVNet = Get-AzVirtualnetwork -Name MyVNet -ResourceGroupName $RgName
   $Subnet = $MyVnet.Subnets | Where-Object { $_.Name -eq "MySubnet" }
   ```

   Si vous ne connaissez pas le nom du réseau virtuel ou du sous-réseau auxquels la carte réseau est connectée, entrez la commande suivante :

   ```powershell
   $MyNIC.IpConfigurations
   ```

   Dans la sortie, recherchez le texte similaire à l’exemple de sortie suivant :

   ```
   "Id": "/subscriptions/[Id]/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/MyVNet/subnets/MySubnet"
   ```

    Dans cette sortie, *MyVnet* est le réseau virtuel, et *MySubnet* le sous-réseau, auxquels la carte réseau est connectée.

5. Suivez les étapes de l’une des sections suivantes, selon vos besoins :

   **Ajouter une adresse IP privée**

   Pour ajouter une adresse IP privée à une carte d’interface réseau, vous devez créer une configuration IP. La commande suivante crée une configuration avec l’adresse IP statique 10.0.0.7. Lorsque vous spécifiez une adresse IP statique, il doit s’agir d’une adresse non utilisée pour le sous-réseau. Nous vous recommandons de commencer par entrer la commande `Test-AzPrivateIPAddressAvailability -IPAddress 10.0.0.7 -VirtualNetwork $myVnet` afin de tester l’adresse pour vous assurer de sa disponibilité. Si l’adresse IP est disponible, la sortie retourne *True*. S’il n’est pas disponible, la sortie retourne *False* et la liste des adresses disponibles.

   ```powershell
   Add-AzNetworkInterfaceIpConfig -Name IPConfig-4 -NetworkInterface `
   $MyNIC -Subnet $Subnet -PrivateIpAddress 10.0.0.7
   ```

   Créez autant de configurations que nécessaire, à l’aide de noms de configuration unique et d’adresses IP privées (pour les configurations avec des adresses IP statiques).

   Ajoutez l’adresse IP privée au système d’exploitation de la machine virtuelle en suivant les étapes pour votre système d’exploitation dans la section [Ajouter des adresses IP à un système d’exploitation de machine virtuelle](#os-config) de cet article.

   **Ajouter une adresse IP publique**

   Une adresse IP publique est ajoutée en associant une ressource d’adresse IP publique à une nouvelle configuration IP ou une configuration IP existante. Suivez les étapes de l’une des sections ci-après, selon votre besoin.

   > [!NOTE]
   > Les adresses IP publiques ont un coût nominal. Pour en savoir plus, lisez la page [Tarification des adresses IP](https://azure.microsoft.com/pricing/details/ip-addresses) . Il existe une limite au nombre d’adresses IP publiques qui peuvent être utilisées dans un abonnement. Pour plus d’informations sur les limites, voir [Limites d’Azure](../azure-resource-manager/management/azure-subscription-service-limits.md#networking-limits).
   >

   **Associer la ressource d’adresse IP publique à une nouvelle configuration IP**

   Lorsque vous ajoutez une adresse IP publique dans une nouvelle configuration IP, vous devez également ajouter une adresse IP privée, car toutes les configurations IP doivent avoir une adresse IP privée. Vous pouvez ajouter une ressource d’adresse IP publique existante ou en créer une. Pour en créer une nouvelle, saisissez la commande suivante :

   ```powershell
   $myPublicIp3 = New-AzPublicIpAddress `
   -Name "myPublicIp3" `
   -ResourceGroupName $RgName `
   -Location $Location `
   -AllocationMethod Static
   ```

   Pour créer une nouvelle configuration IP avec une adresse IP privée statique et la ressource d’adresse IP publique *myPublicIp3*, saisissez la commande suivante :

   ```powershell
   Add-AzNetworkInterfaceIpConfig `
   -Name IPConfig-4 `
   -NetworkInterface $myNIC `
   -Subnet $Subnet `
   -PrivateIpAddress 10.0.0.7 `
   -PublicIpAddress $myPublicIp3
   ```

   **Associer la ressource d’adresse IP publique à une configuration IP existante**

   Une ressource d’adresse IP publique ne peut être associée qu’à une configuration IP n’ayant pas encore de ressource associée. Pour déterminer si une configuration IP a une adresse IP publique associée, entrez la commande suivante :

   ```powershell
   $MyNIC.IpConfigurations | Format-Table Name, PrivateIPAddress, PublicIPAddress, Primary
   ```

   Vous voyez une sortie similaire à ce qui suit :

   ```
   Name       PrivateIpAddress PublicIpAddress                                           Primary

   IPConfig-1 10.0.0.4         Microsoft.Azure.Commands.Network.Models.PSPublicIpAddress    True
   IPConfig-2 10.0.0.5         Microsoft.Azure.Commands.Network.Models.PSPublicIpAddress   False
   IpConfig-3 10.0.0.6                                                                     False
   ```

   Étant donné que la colonne **PublicIpAddress** pour *IpConfig-3* est vide, aucune ressource d’adresse IP publique n’y est actuellement associée. Vous pouvez ajouter une ressource d’adresse IP publique existante à IpConfig-3, ou entrer la commande suivante pour en créer une :

   ```powershell
   $MyPublicIp3 = New-AzPublicIpAddress `
   -Name "MyPublicIp3" `
   -ResourceGroupName $RgName `
   -Location $Location -AllocationMethod Static
   ```

   Pour associer la ressource d’adresse IP publique à la configuration IP existante nommée *IPConfig-3*, entrez la commande suivante :

   ```powershell
   Set-AzNetworkInterfaceIpConfig `
   -Name IpConfig-3 `
   -NetworkInterface $mynic `
   -Subnet $Subnet `
   -PublicIpAddress $myPublicIp3
   ```

6. Définissez la carte réseau avec la nouvelle configuration IP en entrant la commande suivante :

   ```powershell
   Set-AzNetworkInterface -NetworkInterface $MyNIC
   ```

7. Affichez les ressources d’adresses IP privées et l’adresse IP publique affectées à la carte d’interface réseau en saisissant la commande suivante :

   ```powershell
   $MyNIC.IpConfigurations | Format-Table Name, PrivateIPAddress, PublicIPAddress, Primary
   ```

8. Ajoutez l’adresse IP privée au système d’exploitation de la machine virtuelle en suivant les étapes pour votre système d’exploitation dans la section [Ajouter des adresses IP à un système d’exploitation de machine virtuelle](#os-config) de cet article. N’ajoutez pas l’adresse IP publique au système d’exploitation.

[!INCLUDE [virtual-network-multiple-ip-addresses-os-config.md](../../includes/virtual-network-multiple-ip-addresses-os-config.md)]