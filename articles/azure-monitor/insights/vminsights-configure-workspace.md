---
title: Configurer l’espace de travail Log Analytics pour Azure Monitor pour machines virtuelles
description: Décrit comment créer et configurer l’espace de travail Log Analytics utilisé par Azure Monitor pour machines virtuelles.
ms.subservice: ''
ms.topic: conceptual
ms.custom: references_regions
author: bwren
ms.author: bwren
ms.date: 07/27/2020
ms.openlocfilehash: a4380ae7fdb523f8b2f5bc14fefa094b6d96e547
ms.sourcegitcommit: 1bf144dc5d7c496c4abeb95fc2f473cfa0bbed43
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95750526"
---
# <a name="configure-log-analytics-workspace-for-azure-monitor-for-vms"></a>Configurer l’espace de travail Log Analytics pour Azure Monitor pour machines virtuelles
Azure Monitor pour machines virtuelles collecte ses données à partir d’un ou plusieurs espaces de travail Log Analytics dans Azure Monitor. Avant d’intégrer des agents, vous devez créer et configurer un espace de travail. Cet article décrit les exigences de l’espace de travail et sa configuration pour Azure Monitor pour machines virtuelles.

## <a name="overview"></a>Vue d’ensemble
Un seul abonnement peut utiliser n’importe quel nombre d’espaces de travail selon vos besoins, à condition que l’espace de travail se trouve dans un emplacement pris en charge et soit configuré avec la solution *VMInsights*.

Une fois l’espace de travail configuré, vous pouvez utiliser l’une des options disponibles pour installer les agents requis sur les machines virtuelles et VMSS, puis spécifier un espace de travail afin qu’ils envoient leurs données. Azure Monitor pour machines virtuelles collectera les données de n’importe quel espace de travail configuré dans son abonnement.

> [!NOTE]
> Quand vous activez Azure Monitor pour machines virtuelles sur une machine virtuelle ou VMSS à l’aide du portail Azure, vous avez la possibilité de sélectionner un espace de travail existant ou d’en créer un. La solution *VMInsights* sera installée dans cet espace de travail, si ce n’est déjà fait. Vous pouvez ensuite utiliser cet espace de travail pour d’autres agents.


## <a name="create-log-analytics-workspace"></a>Créer un espace de travail Log Analytics

>[!NOTE]
>Les informations décrites dans cette section s’appliquent également à la solution [Service Map](service-map.md). 

Accédez aux espace de travail Log Analytics dans le portail Azure à partir du menu **Espaces de travail Log Analytics**.

[![Espaces de travail Log Analytics](media/vminsights-configure-workspace/log-analytics-workspaces.png)](media/vminsights-configure-workspace/log-analytics-workspaces.png#lightbox)

Vous pouvez créer un espace de travail Log Analytics à l’aide de l’une des méthodes suivantes. Consultez [Conception de votre déploiement de journaux Azure Monitor](../platform/design-logs-deployment.md) pour obtenir des conseils sur la détermination du nombre d'espaces de travail que vous devez utiliser dans votre environnement et sur la façon de concevoir leur stratégie d'accès.


* [Azure portal](../../azure-monitor/learn/quick-create-workspace.md)
* [Azure CLI](../../azure-monitor/learn/quick-create-workspace-cli.md)
* [PowerShell](../platform/powershell-workspace-configuration.md)
* [Azure Resource Manager](../samples/resource-manager-workspace.md)

## <a name="supported-regions"></a>Régions prises en charge

Azure Monitor pour machines virtuelles prend en charge les espaces de travail Log Analytics dans les régions suivantes, même si vous pouvez surveiller des machines virtuelles dans n’importe quelle région. Les machines virtuelles elles-mêmes ne sont pas limitées aux régions prises en charge par l’espace de travail Log Analytics.

- Centre-USA Ouest
- USA Ouest
- USA Ouest 2
- États-Unis - partie centrale méridionale
- USA Est
- USA Est 2
- USA Centre
- Centre-Nord des États-Unis
- US Gov Arizona
- US Gov Va
- Centre du Canada
- Sud du Royaume-Uni
- Europe Nord
- Europe Ouest
- Asie Est
- Asie Sud-Est
- Inde centrale
- Japon Est
- Australie Est
- Sud-Australie Est

## <a name="azure-role-based-access-control"></a>Contrôle d'accès en fonction du rôle Azure
Pour activer les fonctionnalités et y accéder dans Azure Monitor pour les machines virtuelles, vous devez avoir le rôle [Contributeur Log Analytics](../platform/manage-access.md#manage-access-using-azure-permissions) dans l’espace de travail. Pour afficher les données de performances, d’intégrité et de mappage, vous devez avoir le rôle [Lecteur d’analyse](../platform/roles-permissions-security.md#built-in-monitoring-roles) pour la machine virtuelle Azure. Pour plus d’informations sur la façon de contrôler l’accès à un espace de travail Log Analytics, consultez [Gérer les espaces de travail](../platform/manage-access.md).

## <a name="add-vminsights-solution-to-workspace"></a>Ajout de la solution VMInsights à l’espace de travail
Pour pouvoir utiliser un espace de travail Log Analytics avec Azure Monitor pour machines virtuelles, la solution *VMInsights* doit être installée. Les méthodes de configuration de l’espace de travail sont décrites dans les sections suivantes.

> [!NOTE]
> Lorsque vous ajoutez la solution *VMInsights* à l’espace de travail, toutes les machines virtuelles existantes connectées à l’espace de travail commencent à envoyer des données à InsightsMetrics. Les données des autres types de données ne seront pas collectées tant que vous n'aurez pas ajouté Dependency Agent aux machines virtuelles existantes connectées à l'espace de travail.

### <a name="azure-portal"></a>Portail Azure
Il existe trois options pour configurer un espace de travail existant à l’aide du portail Azure. Chacune est décrite ci-dessous.

Pour configurer un espace de travail individuel, accédez à l’option **Machines virtuelles** dans le menu **Azure Monitor**, puis sélectionnez **Autres options d’intégration** et **Configurer un espace de travail**. Sélectionnez un abonnement et un espace de travail, puis cliquez sur **Configurer**.

[![Configurer l’espace de travail](media/vminsights-enable-at-scale-policy/configure-workspace.png)](media/vminsights-enable-at-scale-policy/configure-workspace.png#lightbox)

Pour configurer plusieurs espaces de travail, sélectionnez l’onglet **Configuration de l’espace de travail** dans le sous-menu **Machines virtuelles** du menu **Monitor** du portail Azure. Définissez les valeurs de filtre pour afficher la liste des espaces de travail existants. Cochez la case en regard de chaque espace de travail à activer, puis cliquez sur **Configurer sélectionné**.

[![Configuration de l’espace de travail](media/vminsights-enable-at-scale-policy/workspace-configuration.png)](media/vminsights-enable-at-scale-policy/workspace-configuration.png#lightbox)


Quand vous activez Azure Monitor pour machines virtuelles sur une machine virtuelle ou VMSS à l’aide du portail Azure, vous avez la possibilité de sélectionner un espace de travail existant ou d’en créer un. La solution *VMInsights* sera installée dans cet espace de travail, si ce n’est déjà fait. Vous pouvez ensuite utiliser cet espace de travail pour d’autres agents.

[![Activer une machine virtuelle unique dans le portail](media/vminsights-enable-single-vm/enable-vminsights-vm-portal.png)](media/vminsights-enable-single-vm/enable-vminsights-vm-portal.png#lightbox)


### <a name="resource-manager-template"></a>Modèle Resource Manager
Les modèles Azure Resource Manager pour Azure Monitor pour machines virtuelles sont fournis dans un fichier d’archive (.zip) que vous pouvez [télécharger à partir de notre référentiel GitHub](https://aka.ms/VmInsightsARMTemplates). Cela inclut un modèle appelé **ConfigureWorkspace** qui configure un espace de travail Log Analytics pour Azure Monitor pour machines virtuelles. Vous déployez ce modèle à l’aide de l’une des méthodes standard, y compris les exemples de commandes PowerShell et CLI ci-dessous : 

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/CLI)

```azurecli
az deployment group create --name ConfigureWorkspace --resource-group my-resource-group --template-file CreateWorkspace.json  --parameters workspaceResourceId='/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/my-resource-group/providers/microsoft.operationalinsights/workspaces/my-workspace' workspaceLocation='eastus'

```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```powershell
New-AzResourceGroupDeployment -Name ConfigureWorkspace -ResourceGroupName my-resource-group -TemplateFile ConfigureWorkspace.json -workspaceResourceId /subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/my-resource-group/providers/microsoft.operationalinsights/workspaces/my-workspace -location eastus
```

---



## <a name="next-steps"></a>Étapes suivantes
- Consultez [Intégrer des agents à Azure Monitor pour machines virtuelles](vminsights-enable-overview.md) pour connecter des agents à Azure Monitor pour machines virtuelles.
- Consultez [Ciblage des solutions de supervision dans Azure Monitor (préversion)](solution-targeting.md) pour limiter la quantité de données envoyées d'une solution à l'espace de travail.
