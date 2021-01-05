---
title: 'Tutoriel : Créer et gérer des machines virtuelles Windows avec Azure PowerShell'
description: Dans ce tutoriel, vous allez apprendre à utiliser Azure PowerShell pour créer et gérer des machines virtuelles Windows dans Azure.
author: cynthn
ms.service: virtual-machines-windows
ms.topic: tutorial
ms.workload: infrastructure
ms.date: 06/06/2019
ms.author: cynthn
ms.custom: mvc, devx-track-azurepowershell
ms.openlocfilehash: 3e52a808b187e3823acfee2c260986518f2f6f49
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91978001"
---
# <a name="tutorial-create-and-manage-windows-vms-with-azure-powershell"></a>Tutoriel : Créer et gérer des machines virtuelles Windows avec Azure PowerShell

Les machines virtuelles fournissent un environnement informatique entièrement configurable et flexible. Ce tutoriel traite de tâches de déploiement de machines virtuelles Azure, telles que la sélection d’une taille de machine virtuelle, la sélection d’une image de machine virtuelle et le déploiement d’une machine virtuelle. Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Créer une machine virtuelle et vous y connecter
> * Sélectionner et utiliser des images de machine virtuelle
> * Afficher et utiliser des tailles de machine virtuelle spécifiques
> * Redimensionner une machine virtuelle
> * Consulter et comprendre l’état d’une machine virtuelle

## <a name="launch-azure-cloud-shell"></a>Lancement d’Azure Cloud Shell

Azure Cloud Shell est un interpréteur de commandes interactif et gratuit que vous pouvez utiliser pour exécuter les étapes de cet article. Il contient des outils Azure courants préinstallés et configurés pour être utilisés avec votre compte. 

Pour ouvrir Cloud Shell, sélectionnez simplement **Essayer** en haut à droite d’un bloc de code. Vous pouvez également lancer Cloud Shell dans un onglet distinct du navigateur en accédant à [https://shell.azure.com/powershell](https://shell.azure.com/powershell). Sélectionnez **Copier** pour copier les blocs de code, collez-les dans Cloud Shell, puis appuyez sur Entrée pour les exécuter.

## <a name="create-resource-group"></a>Créer un groupe de ressources

Créez un groupe de ressources avec la commande [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup).

Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure sont déployées et gérées. Un groupe de ressources doit être créé avant les machines virtuelles. Dans l’exemple suivant, un groupe de ressources nommé *myResourceGroupVM* est créé dans la région *EastUS* :

```azurepowershell-interactive
New-AzResourceGroup `
   -ResourceGroupName "myResourceGroupVM" `
   -Location "EastUS"
```

Le groupe de ressources est spécifié lors de la création ou de la modification d’une machine virtuelle, qui peut être vue dans ce didacticiel.

## <a name="create-a-vm"></a>Créer une machine virtuelle

Lorsque vous créez une machine virtuelle, plusieurs options sont disponibles, comme l’image du système d’exploitation, la configuration réseau et les informations d’identification d’administration. Cet exemple crée une machine virtuelle nommée *myVM*, qui exécute la version par défaut de Windows Server 2016 Datacenter.

Définissez le nom d’utilisateur et le mot de passe pour le compte Administrateur sur la machine virtuelle avec [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential?view=powershell-6) :

```azurepowershell-interactive
$cred = Get-Credential
```

Créez la machine virtuelle avec [New-AzVM](/powershell/module/az.compute/new-azvm).

```azurepowershell-interactive
New-AzVm `
    -ResourceGroupName "myResourceGroupVM" `
    -Name "myVM" `
    -Location "EastUS" `
    -VirtualNetworkName "myVnet" `
    -SubnetName "mySubnet" `
    -SecurityGroupName "myNetworkSecurityGroup" `
    -PublicIpAddressName "myPublicIpAddress" `
    -Credential $cred
```

## <a name="connect-to-vm"></a>Se connecter à une machine virtuelle

Une fois le déploiement terminé, créez une connexion Bureau à distance avec la machine virtuelle.

Exécutez les commandes suivantes pour renvoyer l’adresse IP publique de la machine virtuelle. Prenez note de cette adresse IP, afin de pouvoir vous y connecter ultérieurement avec votre navigateur de manière à tester la connectivité web.

```azurepowershell-interactive
Get-AzPublicIpAddress `
   -ResourceGroupName "myResourceGroupVM"  | Select IpAddress
```

Exécutez la commande suivante sur votre machine locale pour créer une session Bureau à distance avec la machine virtuelle. Remplacez l’adresse IP par l’adresse *publicIPAddress* de votre machine virtuelle. À l'invite, saisissez les informations d’identification que vous avez utilisées lors de la création de la machine virtuelle.

```powershell
mstsc /v:<publicIpAddress>
```

Dans la fenêtre **Sécurité Windows**, sélectionnez **Plus de choix**, puis **Utiliser un autre compte**. Entrez le nom d’utilisateur et le mot de passe créés pour la machine virtuelle, puis cliquez sur **OK**.

## <a name="understand-marketplace-images"></a>Comprendre les images de la Place de marché

La Place de marché Azure comprend de nombreuses images qui permettent de créer une nouvelle machine virtuelle. Dans les étapes précédentes, une machine virtuelle a été créée à l’aide de l’image Windows Server 2016 Datacenter. Dans cette étape, le module PowerShell est utilisé pour rechercher d’autres images Windows dans la place de marché, qui peuvent également servir de base pour les nouvelles machines virtuelles. Ce processus consiste à trouver le serveur de publication, l’offre, la référence SKU et éventuellement un numéro de version pour [identifier](cli-ps-findimage.md#terminology) l’image.

Utilisez la commande [Get-AzVMImagePublisher](/powershell/module/az.compute/get-azvmimagepublisher) pour retourner la liste des éditeurs d’images :

```azurepowershell-interactive
Get-AzVMImagePublisher -Location "EastUS"
```

Utilisez la commande [Get-AzVMImageOffer](/powershell/module/az.compute/get-azvmimageoffer) pour retourner la liste des offres d’images. Cette commande permet de filtrer la liste retournée en fonction de l’éditeur spécifié nommé `MicrosoftWindowsServer` :

```azurepowershell-interactive
Get-AzVMImageOffer `
   -Location "EastUS" `
   -PublisherName "MicrosoftWindowsServer"
```

Le résultat doit ressembler à l’exemple suivant : 

```powershell
Offer             PublisherName          Location
-----             -------------          --------
Windows-HUB       MicrosoftWindowsServer EastUS
WindowsServer     MicrosoftWindowsServer EastUS
WindowsServer-HUB MicrosoftWindowsServer EastUS
```

La commande [Get-AzVMImageSku](/powershell/module/az.compute/get-azvmimagesku) filtre ensuite les résultats en fonction du nom de l’éditeur et de l’offre pour retourner la liste des noms d’images.

```azurepowershell-interactive
Get-AzVMImageSku `
   -Location "EastUS" `
   -PublisherName "MicrosoftWindowsServer" `
   -Offer "WindowsServer"
```

Le résultat doit ressembler à l’exemple suivant : 

```powershell
Skus                                      Offer         PublisherName          Location
----                                      -----         -------------          --------
2008-R2-SP1                               WindowsServer MicrosoftWindowsServer EastUS  
2008-R2-SP1-smalldisk                     WindowsServer MicrosoftWindowsServer EastUS  
2012-Datacenter                           WindowsServer MicrosoftWindowsServer EastUS  
2012-Datacenter-smalldisk                 WindowsServer MicrosoftWindowsServer EastUS  
2012-R2-Datacenter                        WindowsServer MicrosoftWindowsServer EastUS  
2012-R2-Datacenter-smalldisk              WindowsServer MicrosoftWindowsServer EastUS  
2016-Datacenter                           WindowsServer MicrosoftWindowsServer EastUS  
2016-Datacenter-Server-Core               WindowsServer MicrosoftWindowsServer EastUS  
2016-Datacenter-Server-Core-smalldisk     WindowsServer MicrosoftWindowsServer EastUS
2016-Datacenter-smalldisk                 WindowsServer MicrosoftWindowsServer EastUS
2016-Datacenter-with-Containers           WindowsServer MicrosoftWindowsServer EastUS
2016-Datacenter-with-Containers-smalldisk WindowsServer MicrosoftWindowsServer EastUS
2016-Datacenter-with-RDSH                 WindowsServer MicrosoftWindowsServer EastUS
2016-Nano-Server                          WindowsServer MicrosoftWindowsServer EastUS
```

Ces informations peuvent être utilisées pour déployer une machine virtuelle avec une image spécifique. Dans cet exemple, une machine virtuelle est déployée à l’aide de la dernière version d’une image Windows Server 2016 avec conteneurs.

```azurepowershell-interactive
New-AzVm `
    -ResourceGroupName "myResourceGroupVM" `
    -Name "myVM2" `
    -Location "EastUS" `
    -VirtualNetworkName "myVnet" `
    -SubnetName "mySubnet" `
    -SecurityGroupName "myNetworkSecurityGroup" `
    -PublicIpAddressName "myPublicIpAddress2" `
    -ImageName "MicrosoftWindowsServer:WindowsServer:2016-Datacenter-with-Containers:latest" `
    -Credential $cred `
    -AsJob
```

Le paramètre `-AsJob` crée la machine virtuelle en tant que tâche en arrière-plan. Vous recevez donc les invites PowerShell. Vous pouvez afficher les détails des travaux en arrière-plan à l’aide du cmdlet `Get-Job`.

## <a name="understand-vm-sizes"></a>Comprendre les tailles de machine virtuelle

La taille de machine virtuelle détermine la quantité de ressources de calcul, comme le processeur, le processeur graphique (GPU) et la mémoire, qui sont mises à la disposition de la machine virtuelle. Les machines virtuelles doivent être créées avec une taille adaptée à la charge de travail. Si une charge de travail augmente, une machine virtuelle existante peut également être redimensionnée.

### <a name="vm-sizes"></a>Tailles de machine virtuelle

Le tableau suivant classe les tailles en fonction des cas d’utilisation.  

| Type                     | Tailles courantes           |    Description       |
|--------------------------|-------------------|------------------------------------------------------------------------------------------------------------------------------------|
| [Usage général](../sizes-general.md)         |B, Dsv3, Dv3, DSv2, Dv2, Av2, DC| Ratio processeur/mémoire équilibré. Idéale pour le développement/test et pour les petites et moyennes applications et solutions de données.  |
| [Optimisé pour le calcul](../sizes-compute.md)   | Fsv2          | Ratio processeur/mémoire élevé. Convient pour les applications au trafic moyen, les appliances réseau et les processus de traitement par lots.        |
| [Mémoire optimisée](../sizes-memory.md)    | Esv3, Ev3, M, DSv2, Dv2  | Ratio mémoire/cœur élevé. Idéale pour les bases de données relationnelles, les caches moyens à grands et l’analytique en mémoire.                 |
| [Optimisé pour le stockage](../sizes-storage.md)      | Lsv2, Ls              | Débit de disque et E/S élevés. Idéale pour les bases de données NoSQL, SQL et Big Data.                                                         |
| [GPU](../sizes-gpu.md)          | NV, NVv2, NC, NCv2, NCv3, ND            | Machines virtuelles spécialisées conçues pour les opérations graphiques lourdes et la retouche vidéo.       |
| [Hautes performances](../sizes-hpc.md) | H        | Nos machines virtuelles dotées des processeurs les plus puissants avec interfaces réseau haut débit en option (RDMA). |

### <a name="find-available-vm-sizes"></a>Rechercher les tailles de machines virtuelles disponibles

Pour afficher la liste des tailles de machine virtuelle disponibles dans une région particulière, utilisez la commande [Get-AzVMSize](/powershell/module/az.compute/get-azvmsize).

```azurepowershell-interactive
Get-AzVMSize -Location "EastUS"
```

## <a name="resize-a-vm"></a>Redimensionner une machine virtuelle

Après avoir déployé une machine virtuelle, vous pouvez la redimensionner pour augmenter ou diminuer l’allocation des ressources.

Avant de redimensionner une machine virtuelle, vérifiez si la taille souhaitée est disponible dans le cluster de machines virtuelles actuel. La commande [Get-AzVMSize](/powershell/module/az.compute/get-azvmsize) retourne la liste des tailles.

```azurepowershell-interactive
Get-AzVMSize -ResourceGroupName "myResourceGroupVM" -VMName "myVM"
```

Si la taille est disponible, la machine virtuelle peut être redimensionnée à partir d’un état sous tension, mais elle est redémarrée au cours de l’opération.

```azurepowershell-interactive
$vm = Get-AzVM `
   -ResourceGroupName "myResourceGroupVM"  `
   -VMName "myVM"
$vm.HardwareProfile.VmSize = "Standard_DS3_v2"
Update-AzVM `
   -VM $vm `
   -ResourceGroupName "myResourceGroupVM"
```

Si la taille souhaitée n’est pas disponible dans le cluster actuel, la machine virtuelle doit être libérée avant de procéder au redimensionnement. La libération d’une machine virtuelle supprime toutes les données du disque temporaire, et l’adresse IP publique est modifiée, sauf si une adresse IP statique est utilisée.

```azurepowershell-interactive
Stop-AzVM `
   -ResourceGroupName "myResourceGroupVM" `
   -Name "myVM" -Force
$vm = Get-AzVM `
   -ResourceGroupName "myResourceGroupVM"  `
   -VMName "myVM"
$vm.HardwareProfile.VmSize = "Standard_E2s_v3"
Update-AzVM -VM $vm `
   -ResourceGroupName "myResourceGroupVM"
Start-AzVM `
   -ResourceGroupName "myResourceGroupVM"  `
   -Name $vm.name
```

## <a name="vm-power-states"></a>États d’alimentation de la machine virtuelle

Une machine virtuelle Azure peut présenter différents états d’alimentation. 


| État d’alimentation | Description
|----|----|
| Démarrage en cours | La machine virtuelle est en cours de démarrage. |
| Exécution en cours | La machine virtuelle est en cours d’exécution. |
| En cours d’arrêt | La machine virtuelle est en cours d’arrêt. |
| Arrêté | La machine virtuelle est arrêtée. Les machines virtuelles à l’état arrêté entraînent toujours des frais de calcul.  |
| Libération | La machine virtuelle est libérée. |
| Libéré | Indique que la machine virtuelle est supprimée de l’hyperviseur, mais reste disponible dans le plan de contrôle. Les machines virtuelles à l’état `Deallocated` n’entraînent pas de frais de calcul. |
| - | L’état d’alimentation de la machine virtuelle est inconnu. |


Pour obtenir l’état d’une machine virtuelle spécifique, utilisez la commande [Get-AzVM](/powershell/module/az.compute/get-azvm). Veillez à spécifier un nom valide de machine virtuelle et de groupe de ressources.

```azurepowershell-interactive
Get-AzVM `
    -ResourceGroupName "myResourceGroupVM" `
    -Name "myVM" `
    -Status | Select @{n="Status"; e={$_.Statuses[1].Code}}
```

Vous obtenez un résultat semblable à l’exemple suivant :

```powershell
Status
------
PowerState/running
```

Pour récupérer l’état d’alimentation de toutes les machines virtuelles de votre abonnement, utilisez l’[API Machines virtuelles - Répertorier tout](/rest/api/compute/virtualmachines/listall) avec le paramètre **statusOnly** défini sur *true*.

## <a name="management-tasks"></a>Tâches de gestion

Pendant le cycle de vie d’une machine virtuelle, vous aurez peut-être besoin d’exécuter des tâches de gestion comme le démarrage, l’arrêt ou la suppression d’une machine virtuelle. En outre, vous souhaiterez peut-être créer des scripts pour automatiser les tâches répétitives ou complexes. À l’aide d’Azure PowerShell, de nombreuses tâches courantes de gestion peuvent être exécutées à partir de la ligne de commande ou dans des scripts.

### <a name="stop-a-vm"></a>Arrêter une machine virtuelle

Arrêtez et libérez une machine virtuelle avec [Stop-AzVM](/powershell/module/az.compute/stop-azvm) :

```azurepowershell-interactive
Stop-AzVM `
   -ResourceGroupName "myResourceGroupVM" `
   -Name "myVM" -Force
```

Si vous souhaitez conserver la machine virtuelle dans un état approvisionné, utilisez le paramètre -StayProvisioned.

### <a name="start-a-vm"></a>Démarrer une machine virtuelle

```azurepowershell-interactive
Start-AzVM `
   -ResourceGroupName "myResourceGroupVM" `
   -Name "myVM"
```

### <a name="delete-resource-group"></a>Supprimer un groupe de ressources

Tout ce que contient un groupe de ressources est supprimé lorsque vous supprimez le groupe de ressources.

```azurepowershell-interactive
Remove-AzResourceGroup `
   -Name "myResourceGroupVM" `
   -Force
```

## <a name="next-steps"></a>Étapes suivantes

Ce didacticiel vous a montré les tâches de base de création et de gestion de machines virtuelles, notamment :

> [!div class="checklist"]
> * Créer une machine virtuelle et vous y connecter
> * Sélectionner et utiliser des images de machine virtuelle
> * Afficher et utiliser des tailles de machine virtuelle spécifiques
> * Redimensionner une machine virtuelle
> * Consulter et comprendre l’état d’une machine virtuelle

Passez au didacticiel suivant pour en savoir plus sur les disques de machine virtuelle.  

> [!div class="nextstepaction"]
> [Créer et gérer des disques de machine virtuelle](./tutorial-manage-data-disk.md)