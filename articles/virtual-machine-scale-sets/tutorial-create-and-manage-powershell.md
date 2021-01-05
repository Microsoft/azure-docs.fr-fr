---
title: 'Tutoriel : Créer et gérer un groupe de machines virtuelles identiques Azure – Azure PowerShell'
description: Découvrez comment utiliser Azure PowerShell pour créer un groupe de machines virtuelles identiques, ainsi que certaines tâches de gestion courantes comme le démarrage et l’arrêt d’une instance, ou la modification de la capacité du groupe identique.
author: ju-shim
ms.author: jushiman
ms.topic: tutorial
ms.service: virtual-machine-scale-sets
ms.subservice: management
ms.date: 05/18/2018
ms.reviewer: mimckitt
ms.custom: mimckitt, devx-track-azurepowershell
ms.openlocfilehash: 1ab52c197ee57351a53919291f1c2e41108396ec
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89078518"
---
# <a name="tutorial-create-and-manage-a-virtual-machine-scale-set-with-azure-powershell"></a>Tutoriel : Créer et gérer un groupe de machines virtuelles identiques avec Azure PowerShell

Un groupe de machines virtuelles identiques vous permet de déployer et de gérer un ensemble de machines virtuelles identiques prenant en charge la mise à l’échelle automatique. Tout au long du cycle de vie du groupe de machines virtuelles identiques, vous devrez peut-être exécuter une ou plusieurs tâches de gestion. Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * Créer un groupe de machines virtuelles identiques et se connecter
> * Sélectionner et utiliser des images de machine virtuelle
> * Afficher et utiliser des tailles d’instance de VM spécifiques
> * Mettre un groupe identique à l’échelle manuellement
> * Effectuer des tâches courantes de gestion de groupe identique

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

[!INCLUDE [updated-for-az.md](../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]



## <a name="create-a-resource-group"></a>Créer un groupe de ressources
Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure sont déployées et gérées. Un groupe de ressources doit être créé avant le groupe de machines virtuelles identiques. Créez un groupe de ressources avec la commande [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Dans cet exemple, un groupe de ressources nommé *myResourceGroup* est créé dans la région *EastUS*. 

```azurepowershell-interactive
New-AzResourceGroup -ResourceGroupName "myResourceGroup" -Location "EastUS"
```
Le nom du groupe de ressources est spécifié lorsque vous créez ou modifiez un groupe identique dans ce didacticiel.


## <a name="create-a-scale-set"></a>Créer un groupe identique
Tout d’abord, définissez un nom d’utilisateur administrateur et un mot de passe pour les instances de machine virtuelle avec [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential?view=powershell-5.1) :

```azurepowershell-interactive
$cred = Get-Credential
```

À présent, créez un groupe de machines virtuelles identiques avec [New-AzVmss](/powershell/module/az.compute/new-azvmss). Pour distribuer le trafic aux différentes instances de machine virtuelle, un équilibreur de charge est également créé. L’équilibreur de charge inclut des règles pour distribuer le trafic sur le port TCP 80, ainsi que pour autoriser le trafic Bureau à distance sur le port TCP 3389 et le trafic Accès distant PowerShell sur le port TCP 5985 :

```azurepowershell-interactive
New-AzVmss `
  -ResourceGroupName "myResourceGroup" `
  -VMScaleSetName "myScaleSet" `
  -Location "EastUS" `
  -VirtualNetworkName "myVnet" `
  -SubnetName "mySubnet" `
  -PublicIpAddressName "myPublicIPAddress" `
  -LoadBalancerName "myLoadBalancer" `
  -Credential $cred
```

La création et la configuration de l’ensemble des ressources et des instances de machine virtuelle du groupe identique prennent quelques minutes.

> [!IMPORTANT]
> Si vous ne parvenez pas à vous connecter à votre groupe identique, vous devrez peut-être créer un groupe de sécurité réseau en ajoutant le paramètre *[-SecurityGroupName "mySecurityGroup"](/powershell/module/az.compute/new-azvmss)* .


## <a name="view-the-vm-instances-in-a-scale-set"></a>Afficher les instances de machine virtuelle dans un groupe identique
Pour afficher la liste des instances de machine virtuelle dans un groupe identique, utilisez [Get-AzVmssVM](/powershell/module/az.compute/get-azvmssvm) comme suit :

```azurepowershell-interactive
Get-AzVmssVM -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet"
```

L’exemple suivant montre deux instances de machine virtuelle dans le groupe identique :

```powershell
ResourceGroupName         Name Location             Sku InstanceID ProvisioningState
-----------------         ---- --------             --- ---------- -----------------
MYRESOURCEGROUP   myScaleSet_0   eastus Standard_DS1_v2          0         Succeeded
MYRESOURCEGROUP   myScaleSet_1   eastus Standard_DS1_v2          1         Succeeded
```

Pour afficher des informations supplémentaires sur une instance spécifique de la machine virtuelle, ajoutez le paramètre `-InstanceId` à [Get-AzVmssVM](/powershell/module/az.compute/get-azvmssvm). L’exemple suivant présente des informations sur l’instance de machine virtuelle *1* :

```azurepowershell-interactive
Get-AzVmssVM -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId "1"
```


## <a name="list-connection-information"></a>Obtenir les informations de connexion
Une adresse IP publique est assignée à l’équilibreur de charge qui achemine le trafic vers les instances de machine virtuelle individuelles. Par défaut, des règles de traduction d’adresses réseau (NAT) sont ajoutées à l’équilibreur de charge Azure qui transfère le trafic de connexion distant vers chaque machine virtuelle sur un port donné. Pour vous connecter aux instances de machine virtuelle d’un groupe identique, vous devez créer une connexion distante vers une adresse IP publique et un numéro de port assignés.

Pour lister les ports NAT permettant une connexion à des instances de machine virtuelle dans un groupe identique, obtenez tout d’abord l’objet d’équilibrage de charge avec [Get-AzLoadBalancer](/powershell/module/az.network/Get-AzLoadBalancer). Affichez ensuite les règles NAT de trafic entrant avec [Get-AzLoadBalancerInboundNatRuleConfig](/powershell/module/az.network/Get-AzLoadBalancerInboundNatRuleConfig) :


```azurepowershell-interactive
# Get the load balancer object
$lb = Get-AzLoadBalancer -ResourceGroupName "myResourceGroup" -Name "myLoadBalancer"

# View the list of inbound NAT rules
Get-AzLoadBalancerInboundNatRuleConfig -LoadBalancer $lb | Select-Object Name,Protocol,FrontEndPort,BackEndPort
```

L’exemple de sortie suivant affiche le nom de l’instance, l’adresse IP publique de l’équilibreur de charge et le numéro de port où les règles NAT transfèrent le trafic :

```powershell
Name             Protocol FrontendPort BackendPort
----             -------- ------------ -----------
myScaleSet3389.0 Tcp             50001        3389
myScaleSet5985.0 Tcp             51001        5985
myScaleSet3389.1 Tcp             50002        3389
myScaleSet5985.1 Tcp             51002        5985
```

Le *nom* de la règle s’aligne avec celui de l’instance de machine virtuelle indiqué dans une commande [Get-AzVmssVM](/powershell/module/az.compute/get-azvmssvm) précédente. Par exemple, pour vous connecter à une instance de machine virtuelle *0*, vous utilisez *myScaleSet3389.0* et vous vous connectez au port *50001*. Pour vous connecter à une instance de machine virtuelle *1*, utilisez la valeur tirée de *myScaleSet3389.1* et connectez-vous au port *50002*. Pour utiliser la communication à distance PowerShell, vous vous connectez à la règle d’instance de machine virtuelle appropriée pour le port *TCP* *5985*.

Affichez l’adresse IP publique de l’équilibreur de charge avec [Get-AzPublicIpAddress](/powershell/module/az.network/Get-AzPublicIpAddress) :


```azurepowershell-interactive
Get-AzPublicIpAddress -ResourceGroupName "myResourceGroup" -Name "myPublicIPAddress" | Select IpAddress
```

Exemple de sortie :

```powershell
IpAddress
---------
52.168.121.216
```

Créez une connexion distante vers votre première instance de machine virtuelle. Spécifiez votre adresse IP publique et le numéro de port de l’instance de machine virtuelle, comme indiqué dans les commandes précédentes. Lorsque vous y êtes invité, entrez les informations d’identification utilisées lors de la création du groupe identique (par défaut dans les exemples de commandes, *azureuser* et *P\@ssw0rd!* ). Si vous utilisez Azure Cloud Shell, effectuez cette étape à partir d’une invite de commandes PowerShell locale ou du client Bureau à distance. L’exemple suivant établit une connexion à l’instance de machine virtuelle *1* :

```powershell
mstsc /v 52.168.121.216:50001
```

Une fois connecté à l’instance de machine virtuelle, vous pouvez modifier la configuration manuelle, le cas échéant. Pour l’instant, fermez la connexion à distance.


## <a name="understand-vm-instance-images"></a>Comprendre les images d’instance de machine virtuelle
La Place de marché Azure comprend de nombreuses images qui permettent de créer des instances de machine virtuelle. Pour voir la liste des serveurs de publication disponibles, utilisez la commande [Get-AzVMImagePublisher](/powershell/module/az.compute/get-azvmimagepublisher).

```azurepowershell-interactive
Get-AzVMImagePublisher -Location "EastUS"
```

Pour afficher la liste des images pour un serveur de publication donné, utilisez [Get-AzVMImageSku](/powershell/module/az.compute/get-azvmimagesku). Vous pouvez également utiliser `-PublisherName` ou `-Offer` pour filtrer la liste d’images. Dans l’exemple suivant, la liste est filtrée pour toutes les images avec le nom du serveur de publication de *MicrosoftWindowsServer* et une offre qui correspond à *WindowsServer* :

```azurepowershell-interactive
Get-AzVMImageSku -Location "EastUS" -PublisherName "MicrosoftWindowsServer" -Offer "WindowsServer"
```

L’exemple de sortie suivant illustre toutes les images Windows Server disponibles :

```powershell
Skus                                  Offer         PublisherName          Location
----                                  -----         -------------          --------
2008-R2-SP1                           WindowsServer MicrosoftWindowsServer eastus
2008-R2-SP1-smalldisk                 WindowsServer MicrosoftWindowsServer eastus
2012-Datacenter                       WindowsServer MicrosoftWindowsServer eastus
2012-Datacenter-smalldisk             WindowsServer MicrosoftWindowsServer eastus
2012-R2-Datacenter                    WindowsServer MicrosoftWindowsServer eastus
2012-R2-Datacenter-smalldisk          WindowsServer MicrosoftWindowsServer eastus
2016-Datacenter                       WindowsServer MicrosoftWindowsServer eastus
2016-Datacenter-Server-Core           WindowsServer MicrosoftWindowsServer eastus
2016-Datacenter-Server-Core-smalldisk WindowsServer MicrosoftWindowsServer eastus
2016-Datacenter-smalldisk             WindowsServer MicrosoftWindowsServer eastus
2016-Datacenter-with-Containers       WindowsServer MicrosoftWindowsServer eastus
2016-Datacenter-with-RDSH             WindowsServer MicrosoftWindowsServer eastus
2016-Nano-Server                      WindowsServer MicrosoftWindowsServer eastus
```

Lorsque vous avez créé un groupe identique au début de ce didacticiel, une image de machine virtuelle par défaut de *Windows Server 2016 DataCenter* a été fournie pour les instances de machine virtuelle. Vous pouvez spécifier une image de machine virtuelle différente en fonction de la sortie de [Get-AzVMImageSku](/powershell/module/az.compute/get-azvmimagesku). L’exemple suivant créerait un groupe identique avec le paramètre `-ImageName` pour spécifier une image de machine virtuelle de *MicrosoftWindowsServer:WindowsServer:2016-Datacenter-with-Containers:latest*. Comme la création et la configuration de toutes les ressources et les instances de machine virtuelle du groupe identique prennent quelques minutes, il est inutile de déployer le groupe identique suivant :

```azurepowershell-interactive
New-AzVmss `
  -ResourceGroupName "myResourceGroup2" `
  -Location "EastUS" `
  -VMScaleSetName "myScaleSet2" `
  -VirtualNetworkName "myVnet2" `
  -SubnetName "mySubnet2" `
  -PublicIpAddressName "myPublicIPAddress2" `
  -LoadBalancerName "myLoadBalancer2" `
  -UpgradePolicyMode "Automatic" `
  -ImageName "MicrosoftWindowsServer:WindowsServer:2016-Datacenter-with-Containers:latest" `
  -Credential $cred
```

> [!IMPORTANT]
> Nous vous recommandons d’utiliser la *toute dernière* version de l’image. Spécifiez « latest » pour utiliser la version la plus récente d’une image disponible au moment du déploiement. Notez que, même si vous utilisez « latest », l’image de machine virtuelle ne sera pas automatiquement mise à jour après le déploiement, quand bien même une nouvelle version serait disponible.

## <a name="understand-vm-instance-sizes"></a>Comprendre les tailles d’instance de machine virtuelle
Une taille d’instance de machine virtuelle, ou *référence SKU*, détermine la quantité de ressources de calcul, comme l’UC, le GPU et la mémoire, qui sont mises à la disposition de l’instance de machine virtuelle. Les instances de machine virtuelle doivent être correctement dimensionnées en fonction de la charge de travail attendue.

### <a name="vm-instance-sizes"></a>Taille des instances de machine virtuelle
Le tableau suivant classe les tailles courantes de machine virtuelle en fonction des cas d’usage.

| Type                     | Tailles courantes           |    Description       |
|--------------------------|-------------------|------------------------------------------------------------------------------------------------------------------------------------|
| [Usage général](../virtual-machines/sizes-general.md)         |Dsv3, Dv3, DSv2, Dv2, DS, D, Av2, A0-7| Ratio processeur/mémoire équilibré. Idéale pour le développement/test et pour les petites et moyennes applications et solutions de données.  |
| [Optimisé pour le calcul](../virtual-machines/sizes-compute.md)   | Fs, F             | Ratio processeur/mémoire élevé. Convient pour les applications au trafic moyen, les appliances réseau et les processus de traitement par lots.        |
| [Mémoire optimisée](../virtual-machines/sizes-memory.md)    | Esv3, Ev3, M, GS, G, DSv2, DS, Dv2, D   | Ratio mémoire/cœur élevé. Idéale pour les bases de données relationnelles, les caches moyens à grands et l’analytique en mémoire.                 |
| [Optimisé pour le stockage](../virtual-machines/sizes-storage.md)      | Ls                | Débit de disque et E/S élevés. Idéale pour les bases de données NoSQL, SQL et Big Data.                                                         |
| [GPU](../virtual-machines/sizes-gpu.md)          | NV, NC            | Machines virtuelles spécialisées conçues pour les opérations graphiques lourdes et la retouche vidéo.       |
| [Hautes performances](../virtual-machines/sizes-hpc.md) | H, A8-11          | Nos machines virtuelles dotées des processeurs les plus puissants avec interfaces réseau haut débit en option (RDMA). 

### <a name="find-available-vm-instance-sizes"></a>Rechercher des tailles d’instance de machine virtuelle disponibles
Pour afficher la liste des tailles d’instances de machine virtuelle disponibles dans une région particulière, utilisez la commande [Get-AzVMSize](/powershell/module/az.compute/get-azvmsize). 

```azurepowershell-interactive
Get-AzVMSize -Location "EastUS"
```

La sortie est semblable à l’exemple condensé suivant qui présente les ressources assignées à chaque taille de machine virtuelle :

```powershell
Name                   NumberOfCores MemoryInMB MaxDataDiskCount OSDiskSizeInMB ResourceDiskSizeInMB
----                   ------------- ---------- ---------------- -------------- --------------------
Standard_DS1_v2                    1       3584                4        1047552                 7168
Standard_DS2_v2                    2       7168                8        1047552                14336
[...]
Standard_A0                        1        768                1        1047552                20480
Standard_A1                        1       1792                2        1047552                71680
[...]
Standard_F1                        1       2048                4        1047552                16384
Standard_F2                        2       4096                8        1047552                32768
[...]
Standard_NV6                       6      57344               24        1047552               389120
Standard_NV12                     12     114688               48        1047552               696320
```

Lorsque vous avez créé un groupe identique au début de ce didacticiel, une référence SKU de machine virtuelle par défaut *Standard_DS1_v2* a été fournie pour les instances de machine virtuelle. Vous pouvez spécifier une taille d’instance de machine virtuelle différente en fonction de la sortie de [Get-AzVMSize](/powershell/module/az.compute/get-azvmsize). L’exemple suivant crée un groupe identique avec le paramètre `-VmSize` afin de spécifier une taille d’instance de machine virtuelle *Standard_F1*. Comme la création et la configuration de toutes les ressources et les instances de machine virtuelle du groupe identique prennent quelques minutes, il est inutile de déployer le groupe identique suivant :

```azurepowershell-interactive
New-AzVmss `
  -ResourceGroupName "myResourceGroup3" `
  -Location "EastUS" `
  -VMScaleSetName "myScaleSet3" `
  -VirtualNetworkName "myVnet3" `
  -SubnetName "mySubnet3" `
  -PublicIpAddressName "myPublicIPAddress3" `
  -LoadBalancerName "myLoadBalancer3" `
  -UpgradePolicyMode "Automatic" `
  -VmSize "Standard_F1" `
  -Credential $cred
```


## <a name="change-the-capacity-of-a-scale-set"></a>Modifier la capacité d’un groupe identique
Lorsque vous avez créé un groupe identique, vous avez demandé de deux instances de machine virtuelle. Pour augmenter ou diminuer le nombre d’instances dans le groupe identique, vous pouvez modifier manuellement la capacité. Le groupe identique crée ou supprime le nombre requis d’instances de machine virtuelle, puis configure l’équilibreur de charge pour distribuer le trafic.

Commencez par créer un objet de groupe identique avec [Get-AzVmss](/powershell/module/az.compute/get-azvmss), puis spécifiez une nouvelle valeur pour `sku.capacity`. Pour appliquer la modification de la capacité, utilisez [Update-AzVmss](/powershell/module/az.compute/update-azvmss). L’exemple suivant fixe le nombre d’instances de machine virtuelle présentes dans votre groupe identique à *3* :

```azurepowershell-interactive
# Get current scale set
$vmss = Get-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet"

# Set and update the capacity of your scale set
$vmss.sku.capacity = 3
Update-AzVmss -ResourceGroupName "myResourceGroup" -Name "myScaleSet" -VirtualMachineScaleSet $vmss 
```

Quelques minutes sont nécessaires pour mettre à jour la capacité de votre groupe identique. Pour afficher le nombre d’instances désormais présentes dans le groupe identique, utilisez [Get-AzVmss](/powershell/module/az.compute/get-azvmss) :

```azurepowershell-interactive
Get-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet"
```

L’exemple de sortie suivant montre que la capacité du groupe identique est désormais *3* :

```powershell
Sku        :
  Name     : Standard_DS2
  Tier     : Standard
  Capacity : 3
```


## <a name="common-management-tasks"></a>Tâches de gestion courantes
Vous pouvez à présent créer un groupe identique, obtenir les informations de connexion et vous connecter aux instances de machine virtuelle. Vous avez appris à utiliser une autre image de système d’exploitation pour vos instances de machine virtuelle, sélectionner une autre taille de machine virtuelle et modifier manuellement le nombre d’instances. Dans le cadre de la gestion quotidienne, vous devrez peut-être arrêter, démarrer ou redémarrer les instances de machine virtuelle dans votre groupe identique.

### <a name="stop-and-deallocate-vm-instances-in-a-scale-set"></a>Arrêter et libérer des instances de machine virtuelle dans un groupe identique
Pour arrêter une ou plusieurs machines virtuelles dans un groupe identique, utilisez [Stop-AzVmss](/powershell/module/az.compute/stop-azvmss). Le paramètre `-InstanceId` vous permet de spécifier une ou plusieurs machines virtuelles à arrêter. Si vous ne spécifiez pas d’ID d’instance, toutes les machines virtuelles dans le groupe identique sont arrêtées. L’exemple suivant présente l’arrêt de l’instance *1* :

```azurepowershell-interactive
Stop-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId "1"
```

Par défaut, les machines virtuelles arrêtées sont libérées et n’entraînent pas de frais de calcul. Si vous souhaitez que la machine virtuelle reste dans un état configuré lors de l’arrêt, ajoutez le paramètre `-StayProvisioned` à la commande précédente. Les machines virtuelles arrêtées qui conservent leur configuration peuvent occasionner des frais de calcul standard.

### <a name="start-vm-instances-in-a-scale-set"></a>Démarrer les instances de machine virtuelle dans un groupe identique
Pour démarrer une ou plusieurs machines virtuelles dans un groupe identique, utilisez [Start-AzVmss](/powershell/module/az.compute/start-azvmss). Le paramètre `-InstanceId` vous permet de spécifier une ou plusieurs machines virtuelles à démarrer. Si vous ne spécifiez pas d’ID d’instance, toutes les machines virtuelles dans le groupe identique sont démarrées. L’exemple suivant montre le démarrage de l’instance *1* :

```azurepowershell-interactive
Start-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId "1"
```

### <a name="restart-vm-instances-in-a-scale-set"></a>Redémarrer les instances de machine virtuelle dans un groupe identique
Pour redémarrer une ou plusieurs machines virtuelles dans un groupe identique, utilisez [Restart-AzVmss](/powershell/module/az.compute/restart-azvmss). Le paramètre `-InstanceId` vous permet de spécifier une ou plusieurs machines virtuelles à redémarrer. Si vous ne spécifiez pas d’ID d’instance, toutes les machines virtuelles dans le groupe identique sont redémarrées. L’exemple suivant présente le redémarrage de l’instance *1* :

```azurepowershell-interactive
Restart-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId "1"
```


## <a name="clean-up-resources"></a>Nettoyer les ressources
Lorsque vous supprimez un groupe de ressources, toutes les ressources qu’il contient, comme les instances de machine virtuelle, le réseau virtuel et les disques, sont également supprimées. Le paramètre `-Force` confirme que vous souhaitez supprimer les ressources sans passer par une invite supplémentaire à cette fin. Le paramètre `-AsJob` retourne le contrôle à l’invite de commandes sans attendre que l’opération se termine.

```azurepowershell-interactive
Remove-AzResourceGroup -Name "myResourceGroup" -Force -AsJob
```


## <a name="next-steps"></a>Étapes suivantes
Dans ce didacticiel, vous avez appris à effectuer quelques tâches de création et de gestion de base concernant le groupe identique avec Azure PowerShell :

> [!div class="checklist"]
> * Créer un groupe de machines virtuelles identiques et se connecter
> * Sélectionner et utiliser des images de machine virtuelle
> * Afficher et utiliser des tailles de machine virtuelle spécifiques
> * Mettre un groupe identique à l’échelle manuellement
> * Effectuer des tâches courantes de gestion de groupe identique

Passez au didacticiel suivant pour en savoir plus sur les disques de groupe identique.

> [!div class="nextstepaction"]
> [Utiliser des disques de données avec des groupes identiques](tutorial-use-disks-powershell.md)
