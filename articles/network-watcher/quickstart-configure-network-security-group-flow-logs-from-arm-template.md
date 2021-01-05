---
title: 'Démarrage rapide : Configurer les journaux de flux de groupe de sécurité réseau à l’aide d’un modèle Azure Resource Manager (modèle ARM)'
description: Découvrez comment activer par programme les journaux de flux de groupe de sécurité réseau (NSG) avec un modèle Azure Resource Manager (modèle ARM) et Azure PowerShell.
services: network-watcher
author: damendo
Customer intent: I need to enable the network security group flow logs by using an Azure Resource Manager template.
ms.service: network-watcher
ms.topic: quickstart
ms.date: 07/22/2020
ms.author: damendo
ms.custom: subject-armqs
ms.openlocfilehash: 563f111a656376899fcd0201b42f87bfea445865
ms.sourcegitcommit: b4880683d23f5c91e9901eac22ea31f50a0f116f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94488041"
---
# <a name="quickstart-configure-network-security-group-flow-logs-by-using-an-arm-template"></a>Démarrage rapide : Configurer les journaux de flux de groupe de sécurité réseau au moyen d’un modèle ARM

Dans ce guide de démarrage rapide, vous apprenez à activer les [journaux de flux de groupe de sécurité réseau (NSG)](network-watcher-nsg-flow-logging-overview.md) en utilisant un modèle [Azure Resource Manager](../azure-resource-manager/management/overview.md) (modèle ARM) et Azure PowerShell.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Nous commençons par une vue d’ensemble des propriétés de l’objet journal de flux NSG. Nous fournissons des exemples de modèle. Ensuite, nous utilisons une instance Azure PowerShell locale pour déployer le modèle.

Si votre environnement remplit les prérequis et que vous êtes déjà familiarisé avec l’utilisation des modèles ARM, sélectionnez le bouton **Déployer sur Azure**. Le modèle s’ouvre dans le portail Azure.

[![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-networkwatcher-flowLogs-create%2Fazuredeploy.json)

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="review-the-template"></a>Vérifier le modèle

Le modèle que nous utilisons dans ce guide de démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/101-networkwatcher-flowlogs-create).

:::code language="json" source="~/quickstart-templates/101-networkwatcher-flowlogs-create/azuredeploy.json":::

Ces ressources Azure sont définies dans le modèle :

- [Microsoft.Storage/storageAccounts](/azure/templates/microsoft.storage/storageaccounts)
- [Microsoft.Resources/deployments](/azure/templates/microsoft.resources/deployments)

## <a name="nsg-flow-logs-object"></a>Objet journaux de flux NSG

Le code suivant illustre un objet journaux de flux NSG et ses paramètres. Pour créer une ressource `Microsoft.Network/networkWatchers/flowLogs`, ajoutez ce code à la section des ressources de votre modèle :

```json
{
  "name": "string",
  "type": "Microsoft.Network/networkWatchers/flowLogs",
  "location": "string",
  "apiVersion": "2019-09-01",
  "properties": {
    "targetResourceId": "string",
    "storageId": "string",
    "enabled": "boolean",
    "flowAnalyticsConfiguration": {
      "networkWatcherFlowAnalyticsConfiguration": {
        "enabled": "boolean",
        "workspaceResourceId": "string",
        "trafficAnalyticsInterval": "integer"
      },
      "retentionPolicy": {
        "days": "integer",
        "enabled": "boolean"
      },
      "format": {
        "type": "string",
        "version": "integer"
      }
    }
  }
}
```

Pour une présentation complète des propriétés de l’objet journaux de flux NSG, consultez [Microsoft.Network networkWatchers/flowLogs](/azure/templates/microsoft.network/networkwatchers/flowlogs).

## <a name="create-your-template"></a>Créer votre modèle

Si vous utilisez des modèles ARM pour la première fois, apprenez-en davantage sur ces modèles ARM en consultant les articles suivants :

- [Déployer des ressources à l’aide de modèles Resource Manager et d’Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md#deploy-local-template)
- [Tutoriel : Créer et déployer votre premier modèle Resource Manager](../azure-resource-manager/templates/template-tutorial-create-first-template.md)

L’exemple suivant est un modèle complet. Il s’agit également de la version la plus simple du modèle. L’exemple contient les paramètres minimaux qui sont passés pour configurer les journaux de flux NSG. Pour plus d’exemples, consultez l’article de présentation [Configurer des journaux de flux NSG avec un modèle Azure Resource Manager](network-watcher-nsg-flow-logging-azure-resource-manager.md).

### <a name="example"></a>Exemple

Le modèle suivant active les journaux de flux pour un groupe de sécurité réseau, puis stocke les journaux dans un compte de stockage spécifique :

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "apiProfile": "2019-09-01",
  "resources": [
    {
      "name": "NetworkWatcher_centraluseuap/Microsoft.NetworkDalanDemoPerimeterNSG",
      "type": "Microsoft.Network/networkWatchers/FlowLogs/",
      "location": "centraluseuap",
      "apiVersion": "2019-09-01",
      "properties": {
        "targetResourceId": "/subscriptions/<subscription Id>/resourceGroups/DalanDemo/providers/Microsoft.Network/networkSecurityGroups/PerimeterNSG",
        "storageId": "/subscriptions/<subscription Id>/resourceGroups/MyCanaryFlowLog/providers/Microsoft.Storage/storageAccounts/storagev2ira",
        "enabled": true,
        "flowAnalyticsConfiguration": {},
        "retentionPolicy": {},
        "format": {}
      }
    }
  ]
}
```

> [!NOTE]
> - Le nom de la ressource utilise le format _ParentResource_ChildResource_. Dans notre exemple, la ressource parente est l’instance régionale d’Azure Network Watcher :
>    - **Format** : NetworkWatcher_RegionName
>    - **Exemple** : NetworkWatcher_centraluseuap
> - `targetResourceId` est l’ID de ressource du groupe de sécurité réseau cible.
> - `storageId` est l’ID de ressource du compte de stockage de destination.

## <a name="deploy-the-template"></a>Déployer le modèle

Ce tutoriel part du principe que vous disposez d’un groupe de ressources et d’un groupe de sécurité réseau existants sur lesquels vous pouvez activer la journalisation des flux.

Vous pouvez enregistrer localement les exemples de modèles présentés dans cet article sous la forme d’un fichier *azuredeploy.json*. Mettez à jour les valeurs de propriété afin qu’elles pointent vers des ressources valides dans votre abonnement.

Pour déployer le modèle, exécutez la commande suivante dans Azure PowerShell :

```azurepowershell-interactive
$context = Get-AzSubscription -SubscriptionId <subscription Id>
Set-AzContext $context
New-AzResourceGroupDeployment -Name EnableFlowLog -ResourceGroupName NetworkWatcherRG `
    -TemplateFile "C:\MyTemplates\azuredeploy.json"
```

> [!NOTE]
> Ces commandes déploient une ressource dans l’exemple de groupe de ressources NetworkWatcherRG, et non dans le groupe de ressources qui contient le groupe de sécurité réseau.

## <a name="validate-the-deployment"></a>Valider le déploiement

Deux options s’offrent à vous pour déterminer si votre déploiement a réussi :

- Votre console PowerShell indique `ProvisioningState` comme étant `Succeeded`.
- Accédez à la [page du portail dédiée aux journaux de flux NSG](https://ms.portal.azure.com/#blade/Microsoft_Azure_Network/NetworkWatcherMenuBlade/flowLogs). 

Si vous rencontrez des problèmes lors du déploiement, consultez [Résolution des erreurs courantes dans un déploiement Azure avec Azure Resource Manager](../azure-resource-manager/templates/common-deployment-errors.md).

## <a name="clean-up-resources"></a>Nettoyer les ressources

Vous pouvez supprimer des ressources Azure en utilisant le mode de déploiement complet. Pour supprimer une ressource de journaux de flux, spécifiez un déploiement en mode complet sans inclure la ressource que vous voulez supprimer. Découvrez-en davantage sur le [mode de déploiement complet](../azure-resource-manager/templates/deployment-modes.md#complete-mode).

Vous pouvez également désactiver un journal de flux NSG dans le portail Azure :

1. Connectez-vous au portail Azure.
1. Sélectionnez **Tous les services**. Dans la zone **Filtre**, entrez **network watcher**. Dans les résultats de recherche, sélectionnez **Network Watcher**.
1. Sous **Journaux**, sélectionnez **Journaux de flux NSG**.
1. Dans la liste des groupes de sécurité réseau, sélectionnez celui pour lequel vous souhaitez désactiver les journaux de flux.
1. Sous **Paramètres des journaux de flux**, sélectionnez **Désactivé**.
1. Sélectionnez **Enregistrer**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez appris à activer les journaux de flux NSG à l’aide d’un modèle ARM. À présent, découvrez comment visualiser vos données de flux NSG en utilisant l’une de ces options :

- [Microsoft Power BI](network-watcher-visualize-nsg-flow-logs-power-bi.md)
- [Outils open source](network-watcher-visualize-nsg-flow-logs-open-source-tools.md)
- [Azure Traffic Analytics](traffic-analytics.md)
