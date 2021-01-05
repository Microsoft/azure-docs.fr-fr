---
title: Azure Hybrid Benefit pour Windows Server
description: Découvrez comment optimiser les avantages de votre contrat Software Assurance pour Windows pour mettre des licences locales sur Azure.
author: xujing-ms
ms.service: virtual-machines-windows
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 4/22/2018
ms.author: xujing
ms.openlocfilehash: 43784fe2bb94d5abd012f22ed530e4631a304614
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91335984"
---
# <a name="azure-hybrid-benefit-for-windows-server"></a>Azure Hybrid Benefit pour Windows Server
Pour les clients avec Software Assurance, Azure Hybrid Benefit pour Windows Server permet d’utiliser les licences Windows Server locales et d’exécuter des machines virtuelles Windows sur Azure à moindre coût. Vous pouvez utiliser Azure Hybrid Benefit pour Windows Server pour déployer de nouvelles machines virtuelles avec le système d’exploitation Windows. Cet article récapitule les étapes du déploiement de nouvelles machines virtuelles avec Azure Hybrid Benefit pour Windows Server. Il explique également comment mettre à jour des machines virtuelles existantes en cours d’exécution. Pour plus d’informations sur les licences et les réductions de coût relatives à Azure Hybrid Benefit pour Windows Server, consultez la [page de gestion des licences Azure Hybrid Benefit pour Windows Server](https://azure.microsoft.com/pricing/hybrid-use-benefit/).

Chaque licence à 2 processeurs ou chaque ensemble de licences à 16 cœurs a droit à 2 instances pouvant contenir jusqu’à 8 cœurs ou à 1 instance pouvant contenir jusqu’à 16 cœurs. L’offre Azure Hybrid Benefit pour les licences de l’édition Standard ne peut être utilisée qu’une seule fois : localement ou dans Azure. Les avantages de l’édition Datacenter permettent une utilisation simultanée en local et dans Azure.

Vous pouvez désormais utiliser Azure Hybrid Benefit pour Windows Server dans toutes les régions avec n’importe quelle machine virtuelle exécutant le système d’exploitation Windows Server, même si elle exécute des logiciels supplémentaires comme SQL Server ou des logiciels tiers de la Place de marché. 


## <a name="classic-vms"></a>les machines virtuelles Classic,

Pour les machines virtuelles classiques, seul le déploiement d’une nouvelle machine virtuelle à partir des images personnalisées locales est pris en charge. Pour tirer parti des fonctionnalités prises en charge dans cet article, vous devez d’abord migrer des machines virtuelles classiques vers le modèle Resource Manager.

[!INCLUDE [classic-vm-deprecation](../../../includes/classic-vm-deprecation.md)]
 

## <a name="ways-to-use-azure-hybrid-benefit-for-windows-server"></a>Façons d’utiliser Azure Hybrid Benefit pour Windows Server
Il existe plusieurs façons d’utiliser des machines virtuelles Windows avec Azure Hybrid Benefit :

1. Vous pouvez déployer des machines virtuelles à partir de l’une des images Windows Server fournies sur la Place de marché Microsoft Azure
2. Vous pouvez charger une machine virtuelle personnalisée et la déployer à l’aide d’un modèle Resource Manager ou d’Azure PowerShell.
3. Vous pouvez convertir une machine virtuelle existante de sorte qu’elle s’exécute avec Azure Hybrid Benefit ou selon un paiement à l’utilisation pour Windows Server ou basculer entre les deux méthodes.
4. Vous pouvez également appliquer Azure Hybrid Benefit pour Windows Server sur un groupe de machines virtuelles identiques.


## <a name="create-a-vm-with-azure-hybrid-benefit-for-windows-server"></a>Créer une machine virtuelle avec Azure Hybrid Benefit pour Windows Server
Toutes les images basées sur le système d’exploitation Windows Server sont prises en charge pour Azure Hybrid Benefit pour Windows Server. Vous pouvez utiliser des images de prise en charge de plateforme Azure ou charger des images Windows Server personnalisées. 

### <a name="portal"></a>Portail
Pour créer une machine virtuelle avec Azure Hybrid pour Windows Server, utilisez le bouton bascule sous la section « Faites des économies ».

### <a name="powershell"></a>PowerShell


```powershell
New-AzVm `
    -ResourceGroupName "myResourceGroup" `
    -Name "myVM" `
    -Location "East US" `
    -ImageName "Win2016Datacenter" `
    -LicenseType "Windows_Server"
```

### <a name="cli"></a>Interface de ligne de commande
```azurecli
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --location eastus \
    --license-type Windows_Server
```

### <a name="template"></a>Modèle
Dans vos modèles Resource Manager, vous pouvez spécifier un paramètre supplémentaire `licenseType`. Pour en savoir plus sur la création de modèles Azure Resource Manager, [cliquez ici](../../azure-resource-manager/templates/template-syntax.md).
```json
"properties": {
    "licenseType": "Windows_Server",
    "hardwareProfile": {
        "vmSize": "[variables('vmSize')]"
    }
```

## <a name="convert-an-existing-vm-using-azure-hybrid-benefit-for-windows-server"></a>Convertir une machine virtuelle existante pour utiliser Azure Hybrid Benefit pour Windows Server
Si vous souhaitez convertir une machine virtuelle existante de sorte qu’elle puisse tirer parti d’Azure Hybrid Benefit pour Windows Server, vous pouvez mettre à jour le type de licence de votre machine virtuelle en suivant les instructions ci-dessous.

> [!NOTE]
> La modification du type de licence sur la machine virtuelle n’entraîne pas le redémarrage du système ni l’interruption de service.  Il s’agit simplement d’une mise à jour vers un indicateur de métadonnées.
> 

### <a name="portal"></a>Portail
Dans le panneau de la machine virtuelle du Portail, vous pouvez mettre à jour la machine virtuelle pour utiliser Azure Hybrid Benefit en sélectionnant l’option « Configuration », puis en activant l’option « Azure Hybrid Benefit ».

### <a name="powershell"></a>PowerShell
- Convertir des machines virtuelles Windows Server existantes pour qu’elles utilisent Azure Hybrid Benefit pour Windows Server

    ```powershell
    $vm = Get-AzVM -ResourceGroup "rg-name" -Name "vm-name"
    $vm.LicenseType = "Windows_Server"
    Update-AzVM -ResourceGroupName rg-name -VM $vm
    ```
    
- Convertir des machines virtuelles Windows Server avec Azure Hybrid Benefit en mode paiement à l’utilisation

    ```powershell
    $vm = Get-AzVM -ResourceGroup "rg-name" -Name "vm-name"
    $vm.LicenseType = "None"
    Update-AzVM -ResourceGroupName rg-name -VM $vm
    ```
    
### <a name="cli"></a>Interface de ligne de commande
- Convertir des machines virtuelles Windows Server existantes pour qu’elles utilisent Azure Hybrid Benefit pour Windows Server

    ```azurecli
    az vm update --resource-group myResourceGroup --name myVM --set licenseType=Windows_Server
    ```

### <a name="how-to-verify-your-vm-is-utilizing-the-licensing-benefit"></a>Comment vérifier si votre machine virtuelle utilise Azure Hybrid Benefit
Une fois votre machine virtuelle déployée par le biais de PowerShell, d’un modèle Resource Manager ou du portail, utilisez les méthodes suivantes pour vérifier le paramètre.

### <a name="portal"></a>Portail
Dans le panneau de machines virtuelles du portail, vous pouvez voir le bouton bascule pour Azure Hybrid Benefit pour Windows Server en sélectionnant l’onglet Configuration.

### <a name="powershell"></a>PowerShell
L’exemple suivant montre le type de licence pour une machine virtuelle unique.
```powershell
Get-AzVM -ResourceGroup "myResourceGroup" -Name "myVM"
```

Sortie :
```powershell
Type                     : Microsoft.Compute/virtualMachines
Location                 : westus
LicenseType              : Windows_Server
```

Ce résultat contraste avec la machine virtuelle suivante déployée sans licence Azure Hybrid Benefit pour Windows Server :
```powershell
Type                     : Microsoft.Compute/virtualMachines
Location                 : westus
LicenseType              :
```

### <a name="cli"></a>Interface de ligne de commande
```azurecli
az vm get-instance-view -g MyResourceGroup -n MyVM --query "[?licenseType=='Windows_Server']" -o table
```

> [!NOTE]
> La modification du type de licence sur la machine virtuelle n’entraîne pas le redémarrage du système ni l’interruption de service. Il s’agit uniquement d’un indicateur de gestionnaire de licences de métadonnées.
>

## <a name="list-all-vms-with-azure-hybrid-benefit-for-windows-server-in-a-subscription"></a>Lister toutes les machines virtuelles avec Azure Hybrid Benefit pour Windows Server dans un abonnement
Pour afficher et compter toutes les machines virtuelles déployées avec Azure Hybrid Benefit pour Windows Server, vous pouvez exécuter la commande suivante à partir de votre abonnement :

### <a name="portal"></a>Portail
À partir de la machine virtuelle ou du panneau de ressources du groupe de machines virtuelles identiques, vous pouvez voir la liste de toutes vos machines virtuelles et le type de licence en ajoutant « Azure Hybrid Benefit » dans la colonne du tableau. Le paramètre de machine virtuelle peut indiquer l’état « Activé », « Non activé » ou « Non pris en charge».

### <a name="powershell"></a>PowerShell
```powershell
$vms = Get-AzVM
$vms | ?{$_.LicenseType -like "Windows_Server"} | select ResourceGroupName, Name, LicenseType
```

### <a name="cli"></a>Interface de ligne de commande
```azurecli
az vm list --query "[?licenseType=='Windows_Server']" -o table
```

## <a name="deploy-a-virtual-machine-scale-set-with-azure-hybrid-benefit-for-windows-server"></a>Déployer un groupe de machines virtuelles identiques avec Azure Hybrid Benefit pour Windows Server
Au sein des modèles Resource Manager de votre groupe de machines virtuelles identiques, vous devez spécifier un paramètre supplémentaire `licenseType` dans votre propriété VirtualMachineProfile. Vous pouvez le faire durant la création ou la mise à jour de votre groupe identique avec un modèle ARM, PowerShell, Azure CLI ou REST.

L’exemple qui suit utilise un modèle ARM avec une image Windows Server 2016 Datacenter :
```json
"virtualMachineProfile": {
    "storageProfile": {
        "osDisk": {
            "createOption": "FromImage"
        },
        "imageReference": {
            "publisher": "MicrosoftWindowsServer",
            "offer": "WindowsServer",
            "sku": "2016-Datacenter",
            "version": "latest"
        }
    },
    "licenseType": "Windows_Server",
    "osProfile": {
            "computerNamePrefix": "[parameters('vmssName')]",
            "adminUsername": "[parameters('adminUsername')]",
            "adminPassword": "[parameters('adminPassword')]"
    }
```
Vous pouvez également consulter [Modifier un groupe de machines virtuelles identiques](../../virtual-machine-scale-sets/virtual-machine-scale-sets-upgrade-scale-set.md) pour accéder à d’autres méthodes de mise à jour d’un groupe.

## <a name="next-steps"></a>Étapes suivantes
- Découvrez comment [réaliser des économies avec Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-use-benefit/).
- Obtenez plus d’informations dans [Questions fréquentes sur Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-use-benefit/faq/).
- Consultez les [instructions détaillées sur la gestion des licences avec Azure Hybrid Benefit pour Windows Server](/windows-server/get-started/azure-hybrid-benefit).
- Découvrez comment [Azure Hybrid Benefit pour Windows Server et Azure Site Recovery rendent la migration d’applications vers Azure encore plus rentable](https://azure.microsoft.com/blog/hybrid-use-benefit-migration-with-asr/).
- En savoir plus sur [Windows 10 sur Azure avec des droits d’hébergement multilocataire](./windows-desktop-multitenant-hosting-deployment.md).
- En savoir plus sur [l’utilisation de modèles Resource Manager](../../azure-resource-manager/management/overview.md).
