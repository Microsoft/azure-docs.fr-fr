---
title: 'Network Watcher : créer des journaux de flux NSG avec un modèle Azure Resource Manager'
description: Utilisez un modèle Azure Resource Manager et PowerShell pour configurer facilement des journaux de flux NSG.
services: network-watcher
documentationcenter: na
author: damendo
manager: twooley
editor: ''
tags: azure-resource-manager
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/26/2020
ms.author: damendo
ms.openlocfilehash: c3ac2eab68963d75cd5c916e06285d49edd37aed
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97656085"
---
# <a name="configure-nsg-flow-logs-from-an-azure-resource-manager-template"></a>Configurer des journaux de flux NSG avec un modèle Azure Resource Manager

> [!div class="op_single_selector"]
> - [Azure portal](network-watcher-nsg-flow-logging-portal.md)
> - [PowerShell](network-watcher-nsg-flow-logging-powershell.md)
> - [Azure CLI](network-watcher-nsg-flow-logging-cli.md)
> - [REST API](network-watcher-nsg-flow-logging-rest.md)
> - [Azure Resource Manager](network-watcher-nsg-flow-logging-azure-resource-manager.md)


[Azure Resource Manager](https://azure.microsoft.com/features/resource-manager/) est la méthode native et puissante d’Azure pour gérer votre [infrastructure en tant que code](/azure/devops/learn/what-is-infrastructure-as-code).

Cet article explique comment activer les [journaux de flux NSG](./network-watcher-nsg-flow-logging-overview.md) programmatiquement avec un modèle Azure Resource Manager et Azure PowerShell. Nous commençons par fournir une vue d’ensemble des propriétés de l’objet de journal de flux NSG, suivie de quelques exemples de modèles. Ensuite, nous déployons le modèle à l’aide d’une instance PowerShell locale.


## <a name="nsg-flow-logs-object"></a>Objet de journaux de flux NSG

L'objet de journaux de flux NSG ainsi que tous les paramètres sont indiqués ci-après.
Pour obtenir une vue d’ensemble complète des propriétés, vous pouvez consulter les [informations de référence sur le modèle de journaux de flux NSG](/azure/templates/microsoft.network/2019-11-01/networkwatchers/flowlogs#RetentionPolicyParameters).

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
Pour créer une ressource Microsoft.Network/networkWatchers/flowLogs, ajoutez le code JSON ci-dessus à la section des ressources de votre modèle.


## <a name="creating-your-template"></a>Création de votre modèle

Si vous utilisez des modèles Azure Resource Manager pour la première fois, vous pouvez en savoir plus à leur sujet à l’aide des liens ci-dessous.

* [Déployer des ressources à l’aide de modèles Resource Manager et d’Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md#deploy-local-template)
* [Tutoriel : Créer et déployer votre premier modèle Azure Resource Manager](../azure-resource-manager/templates/template-tutorial-create-first-template.md?tabs=azure-powershell)


Vous trouverez ci-dessous deux exemples de modèles complets permettant de configurer des journaux de flux NSG.

**Exemple 1** :  version la plus simple des informations ci-dessus dans laquelle sont transmis les paramètres minimaux. Le modèle ci-dessous active les journaux de flux NSG sur un groupe de sécurité réseau cible et les stocke dans un compte de stockage donné.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "apiProfile": "2019-09-01",
  "resources": [
 {
    "name": "NetworkWatcher_centraluseuap/Microsoft.NetworkDalanDemoPerimeterNSG",
    "type": "Microsoft.Network/networkWatchers/FlowLogs/",
    "location": "centraluseuap",
    "apiVersion": "2019-09-01",
    "properties": {
      "targetResourceId": "/subscriptions/56abfbd6-ec72-4ce9-831f-bc2b6f2c5505/resourceGroups/DalanDemo/providers/Microsoft.Network/networkSecurityGroups/PerimeterNSG",
      "storageId": "/subscriptions/56abfbd6-ec72-4ce9-831f-bc2b6f2c5505/resourceGroups/MyCanaryFlowLog/providers/Microsoft.Storage/storageAccounts/storagev2ira",
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
> * Le nom de la ressource a le format « Parent Resource_Child resource ». Ici, la ressource parente est l’instance de Network Watcher régionale (format : NetworkWatcher_NomRégion. Exemple : NetworkWatcher_centraluseuap)
> * targetResourceId est l’ID de ressource du groupe de sécurité réseau cible
> * storageId est l’ID de ressource du compte de stockage de destination

**Exemple 2 :** les modèles suivants activent les journaux de flux NSG (version 2) avec une conservation de 5 jours, et Traffic Analytics avec un intervalle de traitement de 10 minutes.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "apiProfile": "2019-09-01",
  "resources": [
    {
      "name": "NetworkWatcher_centraluseuap/Microsoft.NetworkDalanDemoPerimeterNSG",
      "type": "Microsoft.Network/networkWatchers/FlowLogs/",
      "location": "centraluseuap",
      "apiVersion": "2019-09-01",
      "properties": {
        "targetResourceId": "/subscriptions/56abfbd6-ec72-4ce9-831f-bc2b6f2c5505/resourceGroups/DalanDemo/providers/Microsoft.Network/networkSecurityGroups/PerimeterNSG",
        "storageId": "/subscriptions/56abfbd6-ec72-4ce9-831f-bc2b6f2c5505/resourceGroups/MyCanaryFlowLog/providers/Microsoft.Storage/storageAccounts/storagev2ira",
        "enabled": true,
        "flowAnalyticsConfiguration": {
          "networkWatcherFlowAnalyticsConfiguration": {
            "enabled": true,
            "workspaceResourceId": "/subscriptions/56abfbd6-ec72-4ce9-831f-bc2b6f2c5505/resourceGroups/defaultresourcegroup-wcus/providers/Microsoft.OperationalInsights/workspaces/1c4f42e5-3a02-4146-ac9b-3051d8501db0",
            "trafficAnalyticsInterval": 10
          }
        },
        "retentionPolicy": {
          "days": 5,
          "enabled": true
        },
        "format": {
          "type": "JSON",
          "version": 2          
        }
      }
    }
  ]
}
```

## <a name="deploying-your-azure-resource-manager-template"></a>Déploiement de votre modèle Azure Resource Manager

Ce tutoriel part du principe que vous disposez d’un groupe de ressources et d’un NSG sur lequel vous pouvez activer la journalisation de flux.
Vous pouvez enregistrer les exemples de modèles ci-dessus localement en tant que `azuredeploy.json`. Mettez à jour les valeurs de propriété afin qu’elles pointent vers des ressources valides dans votre abonnement.

Pour déployer le modèle, exécutez la commande suivante dans PowerShell.
```azurepowershell
$context = Get-AzSubscription -SubscriptionId 56acfbd6-vc72-43e9-831f-bcdb6f2c5505
Set-AzContext $context
New-AzResourceGroupDeployment -Name EnableFlowLog -ResourceGroupName NetworkWatcherRG `
    -TemplateFile "C:\MyTemplates\azuredeploy.json"
```

> [!NOTE]
> Les commandes ci-dessus déploient une ressource dans le groupe de ressources NetworkWatcherRG et non le groupe de ressources contenant le groupe de sécurité réseau


## <a name="verifying-your-deployment"></a>Vérification de votre déploiement

Il existe deux façons de vérifier si votre déploiement a réussi. Votre console PowerShell doit indiquer « Succeeded » (réussite) pour « ProvisioningState ». De plus, vous pouvez consulter la [page du portail dédiée aux journaux de flux NSG](https://ms.portal.azure.com/#blade/Microsoft_Azure_Network/NetworkWatcherMenuBlade/flowLogs) pour vérifier vos modifications. Si vous rencontrez des problèmes lors du déploiement, consultez [Résolution des erreurs courantes dans un déploiement Azure avec Azure Resource Manager](../azure-resource-manager/templates/common-deployment-errors.md).

## <a name="deleting-your-resource"></a>Suppression de votre ressource
Azure permet la suppression des ressources via le mode de déploiement « Complet ». Pour supprimer une ressource de journaux de flux, spécifiez un déploiement en mode complet sans inclure la ressource que vous souhaitez supprimer. En savoir plus sur le [mode de déploiement complet](../azure-resource-manager/templates/deployment-modes.md#complete-mode)

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment visualiser vos données de flux NSG avec les outils suivants :
* [Microsoft Power BI](network-watcher-visualize-nsg-flow-logs-power-bi.md)
* [Outils open source](network-watcher-visualize-nsg-flow-logs-open-source-tools.md)
* [Azure Traffic Analytics](./traffic-analytics.md)