---
title: Déployer et mettre à niveau avec Azure Resource Manager
description: Découvrez comment déployer des applications et services sur un cluster Service Fabric à l’aide d’un modèle Azure Resource Manager.
ms.topic: conceptual
ms.date: 12/06/2017
ms.openlocfilehash: bb866eb24fb1b286f496bad9845d1ee557baa221
ms.sourcegitcommit: c157b830430f9937a7fa7a3a6666dcb66caa338b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94681667"
---
# <a name="manage-applications-and-services-as-azure-resource-manager-resources"></a>Gérer des applications et services en tant que ressources Azure Resource Manager

Vous pouvez déployer des applications et services sur votre cluster Service Fabric par le biais d’Azure Resource Manager. Cela signifie qu’au lieu d’attendre que le cluster soit prêt pour déployer et gérer des applications par le biais de PowerShell ou de l’interface CLI, vous pouvez maintenant spécifier les applications et les services dans du code JSON et les déployer dans le même modèle Resource Manager que votre cluster. Le processus d’inscription d’application, d’approvisionnement et de déploiement s’effectue en une seule étape.

Nous vous recommandons cette méthode pour déployer toutes les applications de configuration, de gouvernance ou de gestion de cluster nécessaires dans votre cluster. Cela inclut [l’application d’orchestration des correctifs](service-fabric-patch-orchestration-application.md), les agents de surveillance ou toute application qui doit être en cours d’exécution dans votre cluster avant que d’autres applications ou services ne soient déployés. 

Le cas échéant, gérez vos applications en tant que ressources Resource Manager pour améliorer les scénarios suivants :
* Piste d’audit : Resource Manager audite chaque opération et conserve un *journal d’activité* détaillé qui peut vous aider à suivre les modifications apportées à ces applications et à votre cluster.
* Contrôle d’accès en fonction du rôle Azure (Azure RBAC) : la gestion de l’accès aux clusters, ainsi qu’aux applications déployées sur le cluster, peut être effectuée par le biais du même modèle Resource Manager.
* Azure Resource Manager (par le biais du portail Azure) devient un guichet unique pour la gestion de votre cluster et des déploiements d’applications critiques.

L’extrait de code suivant montre les différents types de ressources qui peuvent être gérés au moyen d’un modèle :

```json
{
    "apiVersion": "2019-03-01",
    "type": "Microsoft.ServiceFabric/clusters/applicationTypes",
    "name": "[concat(parameters('clusterName'), '/', parameters('applicationTypeName'))]",
    "location": "[variables('clusterLocation')]",
},
{
    "apiVersion": "2019-03-01",
    "type": "Microsoft.ServiceFabric/clusters/applicationTypes/versions",
    "name": "[concat(parameters('clusterName'), '/', parameters('applicationTypeName'), '/', parameters('applicationTypeVersion'))]",
    "location": "[variables('clusterLocation')]",
},
{
    "apiVersion": "2019-03-01",
    "type": "Microsoft.ServiceFabric/clusters/applications",
    "name": "[concat(parameters('clusterName'), '/', parameters('applicationName'))]",
    "location": "[variables('clusterLocation')]",
},
{
    "apiVersion": "2019-03-01",
    "type": "Microsoft.ServiceFabric/clusters/applications/services",
    "name": "[concat(parameters('clusterName'), '/', parameters('applicationName'), '/', parameters('serviceName'))]",
    "location": "[variables('clusterLocation')]"
}
```


## <a name="add-a-new-application-to-your-resource-manager-template"></a>Ajouter une nouvelle application à votre modèle Resource Manager

1. Préparez le modèle Resource Manager de votre cluster en vue du déploiement. Pour plus d’informations à ce sujet, consultez [Créer un cluster Service Fabric à l’aide d’Azure Resource Manager](service-fabric-cluster-creation-via-arm.md).
2. Pensez à certaines des applications que vous prévoyez de déployer sur le cluster. En existe-t-il exécutées en permanence sur lesquelles d’autres applications prendront des dépendances ? Envisagez-vous de déployer des applications de configuration ou de gouvernance de cluster ? Ces types d’applications sont mieux gérés par le biais d’un modèle Resource Manager, comme expliqué plus haut. 
3. Une fois que vous avez déterminé quelles applications déployer de cette manière, vous devez les empaqueter, les compresser, puis les placer sur un partage de fichiers. Le partage doit être accessible par le biais d’un point de terminaison REST utilisable par Azure Resource Manager pendant le déploiement.
4. Dans votre modèle Resource Manager, sous votre déclaration de cluster, décrivez les propriétés de chaque application. Ces propriétés incluent le nombre de réplicas ou d’instance et toutes les chaînes de dépendances entre les ressources (autres applications ou services). Pour obtenir la liste complète des propriétés, consultez les [Spécifications Swagger de l’API REST](https://aka.ms/sfrpswaggerspec). Notez que cela ne remplace pas les manifestes Application ou Service, mais décrit plutôt certaines de leurs composantes dans le cadre du modèle Resource Manager du cluster. Voici un exemple de modèle qui inclut le déploiement d’un service sans état *Service1* et d’un service avec état *Service2* dans le cadre *d’Application1* :

   ```json
   {
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json",
    "contentVersion": "1.0.0.0",
    "parameters": {
      "clusterName": {
        "type": "string",
        "defaultValue": "Cluster",
        "metadata": {
          "description": "Name of your cluster - Between 3 and 23 characters. Letters and numbers only."
        }
      },
      "applicationTypeName": {
        "type": "string",
        "defaultValue": "ApplicationType",
        "metadata": {
          "description": "The application type name."
        }
      },
      "applicationTypeVersion": {
        "type": "string",
        "defaultValue": "1",
        "metadata": {
          "description": "The application type version."
        }
      },
      "appPackageUrl": {
        "type": "string",
        "metadata": {
          "description": "The URL to the application package sfpkg file."
        }
      },
      "applicationName": {
        "type": "string",
        "defaultValue": "Application1",
        "metadata": {
          "description": "The name of the application resource."
        }
      },
      "serviceName": {
        "type": "string",
        "defaultValue": "Application1~Service1",
        "metadata": {
          "description": "The name of the service resource in the format of {applicationName}~{serviceName}."
        }
      },
      "serviceTypeName": {
        "type": "string",
        "defaultValue": "Service1Type",
        "metadata": {
          "description": "The name of the service type."
        }
      },
      "serviceName2": {
        "type": "string",
        "defaultValue": "Application1~Service2",
        "metadata": {
          "description": "The name of the service resource in the format of {applicationName}~{serviceName}."
        }
      },
      "serviceTypeName2": {
        "type": "string",
        "defaultValue": "Service2Type",
        "metadata": {
          "description": "The name of the service type."
        }
      }
    },
    "variables": {
      "clusterLocation": "[resourcegroup().location]"
    },
    "resources": [
      {
        "apiVersion": "2019-03-01",
        "type": "Microsoft.ServiceFabric/clusters/applicationTypes",
        "name": "[concat(parameters('clusterName'), '/', parameters('applicationTypeName'))]",
        "location": "[variables('clusterLocation')]",
        "dependsOn": [],
        "properties": {
          "provisioningState": "Default"
        }
      },
      {
        "apiVersion": "2019-03-01",
        "type": "Microsoft.ServiceFabric/clusters/applicationTypes/versions",
        "name": "[concat(parameters('clusterName'), '/', parameters('applicationTypeName'), '/', parameters('applicationTypeVersion'))]",
        "location": "[variables('clusterLocation')]",
        "dependsOn": [
          "[concat('Microsoft.ServiceFabric/clusters/', parameters('clusterName'), '/applicationTypes/', parameters('applicationTypeName'))]"
        ],
        "properties": {
          "provisioningState": "Default",
          "appPackageUrl": "[parameters('appPackageUrl')]"
        }
      },
      {
        "apiVersion": "2019-03-01",
        "type": "Microsoft.ServiceFabric/clusters/applications",
        "name": "[concat(parameters('clusterName'), '/', parameters('applicationName'))]",
        "location": "[variables('clusterLocation')]",
        "dependsOn": [
          "[concat('Microsoft.ServiceFabric/clusters/', parameters('clusterName'), '/applicationTypes/', parameters('applicationTypeName'), '/versions/', parameters('applicationTypeVersion'))]"
        ],
        "properties": {
          "provisioningState": "Default",
          "typeName": "[parameters('applicationTypeName')]",
          "typeVersion": "[parameters('applicationTypeVersion')]",
          "parameters": {},
          "upgradePolicy": {
            "upgradeReplicaSetCheckTimeout": "01:00:00.0",
            "forceRestart": "false",
            "rollingUpgradeMonitoringPolicy": {
              "healthCheckWaitDuration": "00:02:00.0",
              "healthCheckStableDuration": "00:05:00.0",
              "healthCheckRetryTimeout": "00:10:00.0",
              "upgradeTimeout": "01:00:00.0",
              "upgradeDomainTimeout": "00:20:00.0"
            },
            "applicationHealthPolicy": {
              "considerWarningAsError": "false",
              "maxPercentUnhealthyDeployedApplications": "50",
              "defaultServiceTypeHealthPolicy": {
                "maxPercentUnhealthyServices": "50",
                "maxPercentUnhealthyPartitionsPerService": "50",
                "maxPercentUnhealthyReplicasPerPartition": "50"
              }
            }
          }
        }
      },
      {
        "apiVersion": "2019-03-01",
        "type": "Microsoft.ServiceFabric/clusters/applications/services",
        "name": "[concat(parameters('clusterName'), '/', parameters('applicationName'), '/', parameters('serviceName'))]",
        "location": "[variables('clusterLocation')]",
        "dependsOn": [
          "[concat('Microsoft.ServiceFabric/clusters/', parameters('clusterName'), '/applications/', parameters('applicationName'))]"
        ],
        "properties": {
          "provisioningState": "Default",
          "serviceKind": "Stateless",
          "serviceTypeName": "[parameters('serviceTypeName')]",
          "instanceCount": "-1",
          "partitionDescription": {
            "partitionScheme": "Singleton"
          },
          "correlationScheme": [],
          "serviceLoadMetrics": [],
          "servicePlacementPolicies": []
        }
      },
      {
        "apiVersion": "2019-03-01",
        "type": "Microsoft.ServiceFabric/clusters/applications/services",
        "name": "[concat(parameters('clusterName'), '/', parameters('applicationName'), '/', parameters('serviceName2'))]",
        "location": "[variables('clusterLocation')]",
        "dependsOn": [
          "[concat('Microsoft.ServiceFabric/clusters/', parameters('clusterName'), '/applications/', parameters('applicationName'))]"
        ],
        "properties": {
          "provisioningState": "Default",
          "serviceKind": "Stateful",
          "serviceTypeName": "[parameters('serviceTypeName2')]",
          "targetReplicaSetSize": "3",
          "minReplicaSetSize": "2",
          "replicaRestartWaitDuration": "00:01:00.0",
          "quorumLossWaitDuration": "00:02:00.0",
          "standByReplicaKeepDuration": "00:00:30.0",
          "partitionDescription": {
            "partitionScheme": "UniformInt64Range",
            "count": "5",
            "lowKey": "1",
            "highKey": "5"
          },
          "hasPersistedState": "true",
          "correlationScheme": [],
          "serviceLoadMetrics": [],
          "servicePlacementPolicies": [],
          "defaultMoveCost": "Low"
        }
      }
    ]
   }
   ```

   > [!NOTE] 
   > *apiVersion* doit avoir la valeur `"2019-03-01"`. Vous pouvez également déployer ce modèle indépendamment du cluster, à condition que celui-ci ait déjà été déployé.

5. Déployez ! 

## <a name="remove-service-fabric-resource-provider-application-resource"></a>Supprimer la ressource de l’application du fournisseur de ressources Fabric Service
La procédure suivante déclenchera l’annulation de l’approvisionnement du package d’application dans le cluster, ce qui permet de nettoyer l’espace disque utilisé :
```powershell
Get-AzureRmResource -ResourceId /subscriptions/{sid}/resourceGroups/{rg}/providers/Microsoft.ServiceFabric/clusters/{cluster}/applicationTypes/{apptType}/versions/{version} -ApiVersion "2019-03-01" | Remove-AzureRmResource -Force -ApiVersion "2017-07-01-preview"
```
Supprimer Microsoft.ServiceFabric/clusters/application du modèle ARM ne suffit pas pour annuler l’approvisionnement de l’application.

>[!NOTE]
> Une fois la suppression terminée, la version du package ne s’affichera plus dans SFX ou ARM. Vous ne pouvez pas supprimer la ressource de la version de type d’application que l’application utilise pour fonctionner, car ARM/SFRP vous en empêche. Si vous essayez d’annuler l’approvisionnement du package en cours d’exécution, le runtime SF vous en empêchera.


## <a name="manage-an-existing-application-via-resource-manager"></a>Gérer une application existante par le biais de Resource Manager

Si votre cluster est déjà disponible et que des applications que vous souhaitez gérer en tant que ressources Resource Manager y sont déjà déployées, au lieu de supprimer les applications et de les redéployer, vous pouvez utiliser un appel PUT avec les mêmes API pour que les applications soient reconnues en tant que ressources Resource Manager. Pour plus d’informations, consultez [Qu’est-ce qu’un modèle de ressource d’application Service Fabric ?](./service-fabric-concept-resource-model.md).

> [!NOTE]
> Pour permettre une mise à niveau du cluster destinée à ignorer les applications non saines, le client peut spécifier « maxPercentUnhealthyApplications: 100 » dans la section « upgradeDescription/healthPolicy ». Les descriptions détaillées de tous les paramètres se trouvent dans la [documentation sur les stratégies de mise à niveau de cluster pour les API REST de Service Fabric](/rest/api/servicefabric/sfrp-model-clusterupgradepolicy).

## <a name="next-steps"></a>Étapes suivantes

* Utilisez [l’interface CLI Service Fabric](service-fabric-cli.md) ou [PowerShell](service-fabric-deploy-remove-applications.md) pour déployer d’autres applications sur votre cluster. 
* [Mettez à niveau votre cluster Service Fabric](service-fabric-cluster-upgrade.md).
