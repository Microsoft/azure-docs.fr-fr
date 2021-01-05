---
title: Activer Azure Monitor pour machines virtuelles à l’aide de modèles Resource Manager
description: Cet article décrit l’activation d’Azure Monitor pour machines virtuelles sur une ou plusieurs machines virtuelles (ou sur un ou plusieurs groupes de machines virtuelles identiques) à l’aide des modèles Azure PowerShell ou Azure Resource Manager.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 07/27/2020
ms.openlocfilehash: cee7072f9bc844fb1f89168de3547dc726472b67
ms.sourcegitcommit: b6267bc931ef1a4bd33d67ba76895e14b9d0c661
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/19/2020
ms.locfileid: "97695901"
---
# <a name="enable-azure-monitor-for-vms-using-resource-manager-templates"></a>Activer Azure Monitor pour machines virtuelles à l’aide de modèles Resource Manager
Cet article explique comment activer Azure Monitor pour machines virtuelles pour une machine virtuelle ou un groupe de machines virtuelles identiques à l’aide de modèles Resource Manager. Cette procédure peut être utilisée pour les éléments suivants :

- Machine virtuelle Azure
- Groupe de machines virtuelles identiques Azure
- Machine virtuelle hybride connectée avec Azure Arc

## <a name="prerequisites"></a>Prérequis

- [Créer et configurer un espace de travail Log Analytics](vminsights-configure-workspace.md) 
- Pour vous assurer que le système d’exploitation de la machine virtuelle ou du groupe de machines virtuelles identiques que vous activez est pris en charge, consulter [Systèmes d’exploitation pris en charge](vminsights-enable-overview.md#supported-operating-systems). 

## <a name="resource-manager-templates"></a>Modèles Resource Manager

Nous avons créé des exemples de modèles Azure Resource Manager pour l’intégration de vos machines virtuelles et groupes de machines virtuelles identiques. Ces modèles incluent des scénarios que vous pouvez utiliser pour activer la surveillance sur une ressource existante et pour créer une ressource où la surveillance est activée.

>[!NOTE]
>Le modèle doit être déployé dans le même groupe de ressources que la machine virtuelle ou le groupe de machines virtuelles identiques sont activés.


Les modèles Azure Resource Manager sont fournis dans un fichier d’archive (.zip) que vous pouvez [télécharger](https://aka.ms/VmInsightsARMTemplates) à partir de notre référentiel GitHub. Le fichier contient des dossiers représentant chaque scénario de déploiement, avec un fichier de modèle et un fichier de paramètres. Avant de les exécuter, modifiez le fichier des paramètres et spécifiez les valeurs requises. 

Le fichier de téléchargement contient les modèles suivants pour différents scénarios :

- Le modèle **ExistingVmOnboarding** active Azure Monitor pour machines virtuelles si la machine virtuelle existe déjà.
- Le modèle **NewVmOnboarding** crée une machine virtuelle et active Azure Monitor pour machines virtuelles à des fins de surveillance.
- Le modèle **ExistingVmssOnboarding** active Azure Monitor pour machines virtuelles si le groupe de machines virtuelles identiques existe déjà.
- Le modèle **NewVmssOnboarding** crée un groupe de machines virtuelles identiques et active Azure Monitor pour machines virtuelles afin de les surveiller.
- Le modèle **ConfigureWorkspace** configure votre espace de travail Log Analytics pour prendre en charge Azure Monitor pour machines virtuelles en activant les solutions et la collection de compteurs de performances pour les systèmes d’exploitation Linux et Windows.

>[!NOTE]
>Si les groupes de machines virtuelles identiques étaient déjà présents et que la stratégie de mise à niveau est **manuelle**, Azure Monitor pour machines virtuelles n’est pas activé pour les instances par défaut après l’exécution du modèle Resource Manager **ExistingVmssOnboarding**. Vous devez mettre à niveau les instances manuellement.

## <a name="deploy-templates"></a>Déployer des modèles
Les modèles peuvent être déployés à l’aide de [n’importe quelle méthode de déploiement pour les modèles Resource Manager](../../azure-resource-manager/templates/deploy-powershell.md), y compris les exemples suivants utilisant PowerShell et CLI.

```powershell
New-AzResourceGroupDeployment -Name OnboardCluster -ResourceGroupName <ResourceGroupName> -TemplateFile <Template.json> -TemplateParameterFile <Parameters.json>
```


```azurecli
az deployment group create --resource-group <ResourceGroupName> --template-file <Template.json> --parameters <Parameters.json>
```



## <a name="next-steps"></a>Étapes suivantes

Une fois la supervision activée pour vos machines virtuelles, ces informations peuvent être analysées par Azure Monitor pour machines virtuelles.

- Pour afficher les dépendances des applications détectées, consultez [Utilisation de la fonctionnalité Map d’Azure Monitor pour machines virtuelles dans le but de comprendre les composants d’application](vminsights-maps.md).

- Pour identifier les goulots d’étranglement et l’utilisation globale avec les performances de votre machine virtuelle, consultez [Afficher les performances des machines virtuelles Azure](vminsights-performance.md).
