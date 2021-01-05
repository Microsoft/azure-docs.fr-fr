---
title: Utiliser des outils en ligne de commande pour démarrer et arrêter des machines virtuelles Azure DevTest Labs
description: Découvrez comment utiliser les outils de ligne de commande pour démarrer et arrêter des machines virtuelles dans Azure DevTest Labs.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: e5a42658f2b83f101271f158c9af70833601b56d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85476414"
---
# <a name="use-command-line-tools-to-start-and-stop-azure-devtest-labs-virtual-machines"></a>Utiliser des outils de ligne de commande pour démarrer et arrêter des machines virtuelles Azure DevTest Labs
Cet article vous montre comment utiliser Azure PowerShell ou Azure CLI pour démarrer ou arrêter des machines virtuelles dans un labo Azure DevTest Labs. Vous pouvez créer des scripts PowerShell/CLI pour automatiser ces opérations. 

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="overview"></a>Vue d’ensemble
Azure DevTest Labs permet de créer des environnements de développement/test rapides, simples et légers. Cela vous permet de gérer rapidement les coûts, d’approvisionner approvisionner les machines virtuelles et de réduire le gaspillage.  Il existe des fonctionnalités intégrées au Portail Azure qui vous permettent de configurer les machines virtuelles dans un labo pour qu’elles démarrent et s’arrêtent automatiquement à un moment donné. 

Toutefois, dans certains scénarios, il peut être souhaitable d’automatiser le démarrage et l’arrêt des machines virtuelles à partir de scripts PowerShell/CLI. Cela vous offre une certaine flexibilité pour démarrer et arrêter à tout moment certaines machines virtuelles plutôt que de le faire à un moment déterminé. Voici quelques situations dans lesquelles il pourrait être utile d’exécuter ces tâches à l’aide de scripts.

- Lorsque vous utilisez une application à 3 niveaux dans un environnement de test, les niveaux doivent être démarrés dans l’ordre. 
- Désactiver une machine virtuelle selon un critère personnalisé pour faire des économies. 
- L’utiliser comme tâche dans un flux de travail CI/CD pour commencer au début du flux, utiliser des machines virtuelles comme ordinateurs de build, comme ordinateurs de test ou comme infrastructure, puis arrêter les machines virtuelles une fois le processus terminé. Un bon exemple est la fabrique d’images personnalisées avec Azure DevTest Labs.  

## <a name="azure-powershell"></a>Azure PowerShell

> [!NOTE]
> Le script suivant utilise le module Azure PowerShell Az. 

Les scripts PowerShell suivants démarrent une machine virtuelle dans un labo. [Invoke-AzResourceAction](/powershell/module/az.resources/invoke-azresourceaction?view=azps-1.7.0) est au cœur de ce script. Le paramètre **ResourceId** est l’ID de ressource qualifiée pour la machine virtuelle dans le labo. Le paramètre **Action** permet de définir les options **Démarrer** ou **Arrêter** en fonction des besoins.

```powershell
# The id of the subscription
$subscriptionId = "111111-11111-11111-1111111"

# The name of the lab
$devTestLabName = "yourlabname"

# The name of the virtual machine to be started
$vMToStart = "vmname"

# The action on the virtual machine (Start or Stop)
$vmAction = "Start"

# Select the Azure subscription
Select-AzSubscription -SubscriptionId $subscriptionId

# Get the lab information
$devTestLab = Get-AzResource -ResourceType 'Microsoft.DevTestLab/labs' -ResourceName $devTestLabName

# Start the VM and return a succeeded or failed status
$returnStatus = Invoke-AzResourceAction `
                    -ResourceId "$($devTestLab.ResourceId)/virtualmachines/$vMToStart" `
                    -Action $vmAction `
                    -Force

if ($returnStatus.Status -eq 'Succeeded') {
    Write-Output "##[section] Successfully updated DTL machine: $vMToStart, Action: $vmAction"
}
else {
    Write-Error "##[error]Failed to update DTL machine: $vMToStart, Action: $vmAction"
}
```


## <a name="azure-cli"></a>Azure CLI
[Azure CLI](/cli/azure/get-started-with-azure-cli?view=azure-cli-latest) est un autre moyen d’automatiser le démarrage et l’arrêt des machines virtuelles DevTest Labs. Azure CLI peut être [installé](/cli/azure/install-azure-cli?view=azure-cli-latest) sur différents systèmes d’exploitation. Le script suivant vous fournit les commandes pour démarrer et arrêter une machine virtuelle dans un labo. 

```azurecli
# Sign in to Azure
az login 

## Get the name of the resource group that contains the lab
az resource list --resource-type "Microsoft.DevTestLab/labs" --name "yourlabname" --query "[0].resourceGroup" 

## Start the VM
az lab vm start --lab-name yourlabname --name vmname --resource-group labResourceGroupName

## Stop the VM
az lab vm stop --lab-name yourlabname --name vmname --resource-group labResourceGroupName
```


## <a name="next-steps"></a>Étapes suivantes
Consultez l’article suivant qui explique comment utiliser le portail Azure pour effectuer ces opérations : [Redémarrer une machine virtuelle](devtest-lab-restart-vm.md).
