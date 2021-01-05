---
title: Journalisation des ressources de diagnostic pour un groupe de sécurité réseau
titlesuffix: Azure Virtual Network
description: Découvrez comment activer les journaux de ressources de diagnostic des événements et du compteur de règles pour un groupe de sécurité réseau Azure.
services: virtual-network
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 06/04/2018
ms.author: kumud
ms.openlocfilehash: 221f7577b3181b1535ab9f544073dac4d031fe66
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89319438"
---
# <a name="resource-logging-for-a-network-security-group"></a>Journalisation des ressources pour un groupe de sécurité réseau

Un groupe de sécurité réseau (NSG) comprend les règles qui autorisent ou refusent le trafic vers un sous-réseau de réseau virtuel, une interface réseau, ou les deux à la fois. 

Quand vous activez la journalisation pour un groupe de sécurité réseau, vous pouvez collecter les types suivants d’informations sur le journal de ressources :

* **Événement :** les entrées sont enregistrées pour lesquelles des règles NSG sont appliquées aux machines virtuelles, en fonction de l’adresse MAC.
* **Compteur de règles :** contient les entrées correspondant au nombre de fois où chaque règle NSG a été appliquée pour refuser ou autoriser le trafic. L’état de ces règles est collecté toutes les 300 secondes.

Les journaux de ressources ne sont disponibles que pour les groupes de sécurité réseau déployés via le modèle de déploiement Azure Resource Manager. Vous ne pouvez pas activer la journalisation des ressources pour les groupes de sécurité réseau déployés via le modèle de déploiement classique. Pour mieux comprendre ces deux modèles, consultez [Présentation des modèles de déploiement Azure](../resource-manager-deployment-model.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

La journalisation des ressources est activée séparément pour *chaque* groupe de sécurité réseau pour lequel vous souhaitez collecter des données de diagnostic. Si vous êtes plus intéressé par les journaux d’activité (activité des opérations), consultez [Journalisation des activités Azure](../azure-monitor/platform/platform-logs-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

## <a name="enable-logging"></a>Activation de la journalisation

Vous pouvez utiliser le [portail Azure](#azure-portal), [PowerShell](#powershell) ou [Azure CLI](#azure-cli) pour activer la journalisation des ressources.

### <a name="azure-portal"></a>Portail Azure

1. Connectez-vous au [portail](https://portal.azure.com).
2. Sélectionnez **Tous les services**, puis tapez *groupes de sécurité réseau*. Quand la mention **Groupes de sécurité réseau** apparaît dans les résultats de recherche, sélectionnez-la.
3. Sélectionnez le groupe de sécurité réseau pour lequel vous souhaitez activer la journalisation.
4. Sous **SURVEILLANCE**, sélectionnez **Journaux de diagnostic**, puis **Activer les diagnostics**, comme illustré dans l’image suivante :

   ![Activer les diagnostics](./media/virtual-network-nsg-manage-log/turn-on-diagnostics.png)

5. Sous **Paramètres de diagnostic**, entrez, ou sélectionnez, les informations suivantes, puis sélectionnez **Enregistrer** :

    | Paramètre                                                                                     | Valeur                                                          |
    | ---------                                                                                   |---------                                                       |
    | Nom                                                                                        | Un nom de votre choix.  Par exemple : *myNsgDiagnostics*      |
    | **Archiver dans un compte de stockage**, **Diffuser vers Event Hub** et **Envoyer à Log Analytics** | Vous pouvez sélectionner autant de destinations que vous le souhaitez. Pour en savoir plus sur chacune d’elles, consultez [Destinations des journaux](#log-destinations).                                                                                                                                           |
    | LOG                                                                                         | Sélectionnez une catégorie de journal, ou les deux. Pour en savoir plus sur les données consignées pour chaque catégorie, consultez [Catégories de journal](#log-categories).                                                                                                                                             |
6. Consultez et analysez les journaux d’activité. Pour plus d’informations, voir [Afficher et analyser les journaux d’activité](#view-and-analyze-logs).

### <a name="powershell"></a>PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Vous pouvez exécuter les commandes qui suivent dans [Azure Cloud Shell](https://shell.azure.com/powershell), ou en exécutant PowerShell à partir de votre ordinateur. Azure Cloud Shell est un interpréteur de commandes interactif gratuit. Il contient des outils Azure courants préinstallés et configurés pour être utilisés avec votre compte. Si vous exécutez PowerShell sur votre ordinateur, vous devez utiliser le module Azure PowerShell version 1.0.0 ou ultérieure. Exécutez `Get-Module -ListAvailable Az` sur votre ordinateur pour trouver la version installée. Si vous devez effectuer une mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps). Si vous exécutez PowerShell localement, vous devez aussi exécuter `Connect-AzAccount` pour vous connecter à Azure avec un compte disposant des [autorisations nécessaires](virtual-network-network-interface.md#permissions).

Pour activer la journalisation des ressources, vous avez besoin de l’ID d’un groupe de sécurité réseau existant. Si vous ne disposez pas de groupe de sécurité réseau, vous pouvez en créer un avec [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup).

Récupérez le groupe de sécurité réseau pour lequel vous souhaitez activer la journalisation des ressources avec [Get-AzNetworkSecurityGroup](/powershell/module/az.network/get-aznetworksecuritygroup). Par exemple, pour récupérer un groupe de sécurité réseau nommé *myNsg*, qui existe dans un groupe de ressources appelé *myResourceGroup*, entrez la commande suivante :

```azurepowershell-interactive
$Nsg=Get-AzNetworkSecurityGroup `
  -Name myNsg `
  -ResourceGroupName myResourceGroup
```

Vous pouvez écrire des journaux de ressources sur trois types de destinations. Pour plus d’informations, consultez [Destinations de journaux](#log-destinations). Dans cet article, les journaux d’activité sont envoyés à la destination *Log Analytics*, à titre d’exemple. Récupérez un espace de travail Log Analytics avec [Get-AzOperationalInsightsWorkspace](/powershell/module/az.operationalinsights/get-azoperationalinsightsworkspace). Par exemple, pour récupérer un espace de travail existant nommé *myWorkspace*, dans un groupe de ressources appelé *myWorkspaces*, entrez la commande suivante :

```azurepowershell-interactive
$Oms=Get-AzOperationalInsightsWorkspace `
  -ResourceGroupName myWorkspaces `
  -Name myWorkspace
```

Si vous n’avez pas d’espace de travail, vous pouvez en créer un avec [New-AzOperationalInsightsWorkspace](/powershell/module/az.operationalinsights/new-azoperationalinsightsworkspace).

Il existe deux catégories de journalisation pour lesquelles vous pouvez activer les journaux d’activité. Pour plus d’informations, consultez [Catégories de journal](#log-categories). Activez la journalisation des ressources pour le groupe de sécurité réseau avec [Set-AzDiagnosticSetting](/powershell/module/az.monitor/set-azdiagnosticsetting). L’exemple suivant enregistre à la fois les données des catégories d’événements et de compteurs dans l’espace de travail d’un groupe de sécurité réseau, à l’aide des ID du groupe de sécurité réseau et de l’espace de travail récupérés précédemment :

```azurepowershell-interactive
Set-AzDiagnosticSetting `
  -ResourceId $Nsg.Id `
  -WorkspaceId $Oms.ResourceId `
  -Enabled $true
```

Si vous souhaitez uniquement enregistrer les données de l’une de ces catégories, plutôt que des deux, ajoutez l’option `-Categories` à la commande précédente, suivie de *NetworkSecurityGroupEvent* ou *NetworkSecurityGroupRuleCounter*. Si vous voulez consigner les données dans une autre [destination](#log-destinations) que celle d’un espace de travail Log Analytics, utilisez les paramètres appropriés pour un [compte de stockage](../azure-monitor/platform/archive-diagnostic-logs.md?toc=%2fazure%2fvirtual-network%2ftoc.json) Azure ou [Event Hub](../azure-monitor/platform/resource-logs-stream-event-hubs.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

Consultez et analysez les journaux d’activité. Pour plus d’informations, voir [Afficher et analyser les journaux d’activité](#view-and-analyze-logs).

### <a name="azure-cli"></a>Azure CLI

Vous pouvez exécuter les commandes qui suivent dans [Azure Cloud Shell](https://shell.azure.com/bash), ou en exécutant Azure CLI à partir de votre ordinateur. Azure Cloud Shell est un interpréteur de commandes interactif gratuit. Il contient des outils Azure courants préinstallés et configurés pour être utilisés avec votre compte. Si vous exécutez l’interface CLI à partir de votre ordinateur, vous avez besoin de la version 2.0.38 ou ultérieure. Exécutez `az --version` sur votre ordinateur pour trouver la version installée. Si vous devez mettre à niveau, consultez [Installation d’Azure CLI](/cli/azure/install-azure-cli?view=azure-cli-latest). Si vous exécutez l’interface CLI localement, vous devez aussi exécuter `az login` pour vous connecter à Azure avec un compte disposant des [autorisations nécessaires](virtual-network-network-interface.md#permissions).

Pour activer la journalisation des ressources, vous avez besoin de l’ID d’un groupe de sécurité réseau existant. Si vous ne disposez pas de groupe de sécurité réseau, vous pouvez en créer un avec la commande [az network nsg create](/cli/azure/network/nsg#az-network-nsg-create).

Récupérez le groupe de sécurité réseau pour lequel vous souhaitez activer la journalisation des ressources avec [az network nsg show](/cli/azure/network/nsg#az-network-nsg-show). Par exemple, pour récupérer un groupe de sécurité réseau nommé *myNsg*, qui existe dans un groupe de ressources appelé *myResourceGroup*, entrez la commande suivante :

```azurecli-interactive
nsgId=$(az network nsg show \
  --name myNsg \
  --resource-group myResourceGroup \
  --query id \
  --output tsv)
```

Vous pouvez écrire des journaux de ressources sur trois types de destinations. Pour plus d’informations, consultez [Destinations de journaux](#log-destinations). Dans cet article, les journaux d’activité sont envoyés à la destination *Log Analytics*, à titre d’exemple. Pour plus d’informations, consultez [Catégories de journal](#log-categories).

Activez la journalisation des ressources pour le groupe de sécurité réseau avec [az monitor diagnostic-settings create](/cli/azure/monitor/diagnostic-settings#az-monitor-diagnostic-settings-create). L’exemple suivant journalise les données de catégorie événement et compteur dans un espace de travail existant nommé *myWorkspace*, qui se trouve dans un groupe de ressources nommé *myWorkspaces*, et l’ID de groupe de sécurité réseau que vous avez récupéré :

```azurecli-interactive
az monitor diagnostic-settings create \
  --name myNsgDiagnostics \
  --resource $nsgId \
  --logs '[ { "category": "NetworkSecurityGroupEvent", "enabled": true, "retentionPolicy": { "days": 30, "enabled": true } }, { "category": "NetworkSecurityGroupRuleCounter", "enabled": true, "retentionPolicy": { "days": 30, "enabled": true } } ]' \
  --workspace myWorkspace \
  --resource-group myWorkspaces
```

Si vous n’avez pas d’espace de travail, vous pouvez en créer un à l’aide du [portail Azure](../azure-monitor/learn/quick-create-workspace.md?toc=%2fazure%2fvirtual-network%2ftoc.json) ou de [PowerShell](/powershell/module/az.operationalinsights/new-azoperationalinsightsworkspace). Il existe deux catégories de journalisation pour lesquelles vous pouvez activer les journaux d’activité.

Si vous souhaitez journaliser les données d’une seule des deux catégories, supprimez l’autre catégorie de la commande précédente. Si vous voulez consigner les données dans une autre [destination](#log-destinations) que celle d’un espace de travail Log Analytics, utilisez les paramètres appropriés pour un [compte de stockage](../azure-monitor/platform/archive-diagnostic-logs.md?toc=%2fazure%2fvirtual-network%2ftoc.json) Azure ou [Event Hub](../azure-monitor/platform/resource-logs-stream-event-hubs.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

Consultez et analysez les journaux d’activité. Pour plus d’informations, voir [Afficher et analyser les journaux d’activité](#view-and-analyze-logs).

## <a name="log-destinations"></a>Destinations des journaux

Les données de diagnostic peuvent être :
- [Écrites dans un compte de stockage Azure](../azure-monitor/platform/archive-diagnostic-logs.md?toc=%2fazure%2fvirtual-network%2ftoc.json), pour l’audit ou l’inspection manuelle. Vous pouvez spécifier la durée de rétention (en jours) à l’aide des paramètres de diagnostic des ressources.
- [Diffusées vers un hub d’événements](../azure-monitor/platform/resource-logs-stream-event-hubs.md?toc=%2fazure%2fvirtual-network%2ftoc.json) pour qu’un service tiers ou une solution d’analyse personnalisée (p. ex. PowerBI) les ingère.
- [Écrit dans les journaux d’activité Azure Monitor](../azure-monitor/platform/resource-logs-collect-storage.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

## <a name="log-categories"></a>Catégories de journal

Les données au format JSON sont écrites pour les catégories de journal suivantes :

### <a name="event"></a>Événement

Le journal des événements contient des informations sur les règles de groupe de sécurité réseau qui sont appliquées aux machines virtuelles, en fonction de l’adresse MAC. Les données suivantes sont enregistrées pour chaque événement. Dans l’exemple suivant, les données sont enregistrées pour une machine virtuelle avec l’adresse IP 192.168.1.4 et l’adresse MAC 00-0D-3A-92-6A-7C :

```json
{
    "time": "[DATE-TIME]",
    "systemId": "[ID]",
    "category": "NetworkSecurityGroupEvent",
    "resourceId": "/SUBSCRIPTIONS/[SUBSCRIPTION-ID]/RESOURCEGROUPS/[RESOURCE-GROUP-NAME]/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/[NSG-NAME]",
    "operationName": "NetworkSecurityGroupEvents",
    "properties": {
        "vnetResourceGuid":"[ID]",
        "subnetPrefix":"192.168.1.0/24",
        "macAddress":"00-0D-3A-92-6A-7C",
        "primaryIPv4Address":"192.168.1.4",
        "ruleName":"[SECURITY-RULE-NAME]",
        "direction":"[DIRECTION-SPECIFIED-IN-RULE]",
        "priority":"[PRIORITY-SPECIFIED-IN-RULE]",
        "type":"[ALLOW-OR-DENY-AS-SPECIFIED-IN-RULE]",
        "conditions":{
            "protocols":"[PROTOCOLS-SPECIFIED-IN-RULE]",
            "destinationPortRange":"[PORT-RANGE-SPECIFIED-IN-RULE]",
            "sourcePortRange":"[PORT-RANGE-SPECIFIED-IN-RULE]",
            "sourceIP":"[SOURCE-IP-OR-RANGE-SPECIFIED-IN-RULE]",
            "destinationIP":"[DESTINATION-IP-OR-RANGE-SPECIFIED-IN-RULE]"
            }
        }
}
```

### <a name="rule-counter"></a>Compteur de règles

Le journal du compteur de règles contient des informations sur chacune des règles appliquées aux ressources. Les données de l’exemple suivant sont enregistrées chaque fois qu’une règle est appliquée. Dans l’exemple suivant, les données sont enregistrées pour une machine virtuelle avec l’adresse IP 192.168.1.4 et l’adresse MAC 00-0D-3A-92-6A-7C :

```json
{
    "time": "[DATE-TIME]",
    "systemId": "[ID]",
    "category": "NetworkSecurityGroupRuleCounter",
    "resourceId": "/SUBSCRIPTIONS/[SUBSCRIPTION ID]/RESOURCEGROUPS/[RESOURCE-GROUP-NAME]/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/[NSG-NAME]",
    "operationName": "NetworkSecurityGroupCounters",
    "properties": {
        "vnetResourceGuid":"[ID]",
        "subnetPrefix":"192.168.1.0/24",
        "macAddress":"00-0D-3A-92-6A-7C",
        "primaryIPv4Address":"192.168.1.4",
        "ruleName":"[SECURITY-RULE-NAME]",
        "direction":"[DIRECTION-SPECIFIED-IN-RULE]",
        "type":"[ALLOW-OR-DENY-AS-SPECIFIED-IN-RULE]",
        "matchedConnections":125
        }
}
```

> [!NOTE]
> L’adresse IP source pour la communication n’est pas enregistrée. En revanche, vous pouvez activer l’[enregistrement des flux NSG](../network-watcher/network-watcher-nsg-flow-logging-portal.md) pour un groupe de sécurité réseau ; il enregistre toutes les informations de compteur de règles, ainsi que l’adresse IP source qui a initié la communication. Les données du journal de flux NSG sont écrites dans un compte Stockage Azure. Vous pouvez analyser les données avec la fonctionnalité d’[analytique du trafic](../network-watcher/traffic-analytics.md) d’Azure Network Watcher.

## <a name="view-and-analyze-logs"></a>Afficher et analyser les journaux d’activité

Pour savoir comment afficher les données des journaux de ressources, consultez [Vue d’ensemble des journaux de la plateforme Azure](../azure-monitor/platform/platform-logs-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json). Si vous envoyez des données de diagnostic à :
- **Journaux d’activité Azure Monitor** : vous pouvez utiliser la solution d’[analytique de groupe de sécurité réseau](../azure-monitor/insights/azure-networking-analytics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-network-security-group-analytics-solution-in-azure-monitor
) pour obtenir des insights plus poussés. La solution offre des visualisations pour les règles NSG qui autorisent ou refusent le trafic, par adresse MAC, de l’interface réseau sur une machine virtuelle.
- **Compte Stockage Azure** : les données sont écrites dans un fichier PT1H.json. Vous pouvez trouver le :
  - Journal des événements dans le chemin suivant : `insights-logs-networksecuritygroupevent/resourceId=/SUBSCRIPTIONS/[ID]/RESOURCEGROUPS/[RESOURCE-GROUP-NAME-FOR-NSG]/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/[NSG NAME]/y=[YEAR]/m=[MONTH/d=[DAY]/h=[HOUR]/m=[MINUTE]`
  - Journal du compteur de règles dans le chemin suivant : `insights-logs-networksecuritygrouprulecounter/resourceId=/SUBSCRIPTIONS/[ID]/RESOURCEGROUPS/[RESOURCE-GROUP-NAME-FOR-NSG]/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/[NSG NAME]/y=[YEAR]/m=[MONTH/d=[DAY]/h=[HOUR]/m=[MINUTE]`

## <a name="next-steps"></a>Étapes suivantes

- Apprenez-en davantage sur la [journalisation des activités](../azure-monitor/platform/platform-logs-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json). La journalisation des activités est activée par défaut pour les groupes de sécurité réseau créés au moyen d’un modèle de déploiement Azure. Pour déterminer les opérations qui ont été effectuées sur les groupes de sécurité réseau dans le journal d’activité, recherchez les entrées qui contiennent les types de ressources suivants :
  - Microsoft.ClassicNetwork/networkSecurityGroups
  - Microsoft.ClassicNetwork/networkSecurityGroups/securityRules
  - Microsoft.Network/networkSecurityGroups
  - Microsoft.Network/networkSecurityGroups/securityRules
- Pour savoir comment enregistrer les informations de diagnostic et y inclure l’adresse IP source de chaque flux, consultez [Journalisation des flux NSG](../network-watcher/network-watcher-nsg-flow-logging-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json).
