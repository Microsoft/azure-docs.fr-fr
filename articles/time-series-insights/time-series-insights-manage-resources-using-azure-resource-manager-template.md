---
title: Gérer votre environnement à l’aide de modèles Azure Resource Manager - Azure Time Series Insights | Microsoft Docs
description: Découvrez comment gérer un environnement Azure Time Series Insights par programmation à l’aide d’Azure Resource Manager.
ms.service: time-series-insights
services: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.devlang: csharp
ms.workload: big-data
ms.topic: conceptual
ms.date: 09/30/2020
ms.custom: seodec18
ms.openlocfilehash: 493750e69b1fdc935b04d6dc705cfd046b6b086e
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "96011657"
---
# <a name="create-azure-time-series-insights-gen-1-resources-using-azure-resource-manager-templates"></a>Créer des ressources Azure Time Series Insights Gen1 à l’aide de modèles Resource Manager

> [!CAUTION]
> Il s’agit d’un article Gen1.

Cet article explique comment créer et déployer des ressources Azure Time Series Insights à l’aide de [modèles Resource Manager](../azure-resource-manager/index.yml), de PowerShell et du fournisseur de ressources Azure Time Series Insights.

Azure Time Series Insights prend en charge les ressources suivantes :

   | Ressource | Description |
   | --- | --- |
   | Environnement | Un environnement Azure Time Series Insights est un regroupement logique d’événements lus à partir de répartiteurs, stockés et rendus interrogeables. Pour plus d’informations, consultez [Planifier votre environnement Azure Time Series Insights](time-series-insights-environment-planning.md) |
   | Source de l’événement | Une source d’événement est une connexion à un répartiteur d’événements à partir duquel Azure Time Series Insights lit et ingère des événements dans l’environnement. Sont actuellement pris en charge IoT Hub et Event Hub. |
   | Jeu de données de référence | Les jeux de données de référence fournissent des métadonnées sur les événements de l’environnement. Les métadonnées des jeux de données de référence seront jointes à des événements au cours de l’entrée. Les jeux de données de référence sont définis comme des ressources par leurs propriétés de clé d’événement. Les métadonnées qui composent le jeu de données de référence sont chargées ou modifiées par le biais d’API de plan de données. |
   | Stratégie d’accès | Les stratégies d’accès accordent l’autorisation de générer des requêtes de données, de manipuler les données de référence dans l’environnement et de partager des requêtes enregistrées et des perspectives associées à l’environnement. Pour plus d’informations, consultez [Accorder l’accès aux données à un environnement Azure Time Series Insights à l’aide du portail Azure](./concepts-access-policies.md). |

Un modèle Resource Manager est un fichier JSON qui définit l’infrastructure et la configuration de ressources dans un groupe de ressources. Les documents suivants décrivent plus en détail les fichiers modèles :

- [Déploiement du modèle Azure Resource Manager](../azure-resource-manager/templates/overview.md)
- [Déployer des ressources à l’aide de modèles Resource Manager et d’Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md)
- [Types de ressource Microsoft.TimeSeriesInsights](/azure/templates/microsoft.timeseriesinsights/allversions)

Le modèle de démarrage rapide [201-timeseriesinsights-environment-with-eventhub](https://github.com/Azure/azure-quickstart-templates/tree/master/201-timeseriesinsights-environment-with-eventhub) est publié sur GitHub. Il crée un environnement Azure Time Series Insights, une source d’événement enfant configurée pour consommer des événements à partir d’un Event Hub et des stratégies qui accordent l’accès aux données de l’environnement. Si aucun hub d’événements existant n’est spécifié, il en sera créé un avec le déploiement.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="specify-deployment-template-and-parameters"></a>Spécifier le modèle et les paramètres de déploiement

La procédure suivante explique comment utiliser PowerShell pour déployer un modèle Resource Manager qui crée un environnement Azure Time Series Insights, une source d’événement enfant configurée pour consommer des événements à partir d’un Event Hub et des stratégies qui accordent l’accès aux données de l’environnement. Si aucun hub d’événements existant n’est spécifié, il en sera créé un avec le déploiement.

1. Installez Azure PowerShell en suivant les instructions disponibles dans [Prise en main d’Azure PowerShell](/powershell/azure/get-started-azureps).

1. Clonez ou copiez le modèle [201-timeseriesinsights-environment-with-eventhub](https://github.com/Azure/azure-quickstart-templates/blob/master/201-timeseriesinsights-environment-with-eventhub/azuredeploy.json) sur GitHub.

   - Créer un fichier de paramètres

     Pour créer un fichier de paramètres, copiez le fichier [201-timeseriesinsights-environment-with-eventhub](https://github.com/Azure/azure-quickstart-templates/blob/master/201-timeseriesinsights-environment-with-eventhub/azuredeploy.parameters.json).

      [!code-json[deployment-parameters](~/quickstart-templates/201-timeseriesinsights-environment-with-eventhub/azuredeploy.parameters.json)]

    <div id="required-parameters"></div>

   - Paramètres obligatoires

     | Paramètre | Description |
     | --- | --- |
     | eventHubNamespaceName | Espace de noms du hub de la source de l’événement. |
     | eventHubName | Nom du hub de la source de l’événement. |
     | consumerGroupName | Nom du groupe de consommateurs que le service Azure Time Series Insights utilisera pour lire les données à partir de l’Event Hub. **REMARQUE :** Pour éviter les conflits de ressources, ce groupe de consommateurs doit être dédié au service Azure Time Series Insights et ne pas être partagé avec d’autres lecteurs. |
     | environmentName | Nom de l’environnement. Il ne peut pas inclure : `<`, `>`, `%`, `&`, `:`, `\\`, `?`, `/`, ni aucun caractère de contrôle. Tous les autres caractères sont autorisés.|
     | eventSourceName | Nom de la ressource enfant de la source de l’événement. Il ne peut pas inclure : `<`, `>`, `%`, `&`, `:`, `\\`, `?`, `/`, ni aucun caractère de contrôle. Tous les autres caractères sont autorisés. |

    <div id="optional-parameters"></div>

   - Paramètres facultatifs

     | Paramètre | Description |
     | --- | --- |
     | existingEventHubResourceId | ID de ressource facultatif d’un Event Hub existant qui sera connecté à l’environnement Azure Time Series Insights par le biais de la source de l’événement. **REMARQUE :** L'utilisateur qui déploie le modèle doit disposer de privilèges pour effectuer l'opération listkeys sur le hub d'événements. Si aucune valeur n’est transmise, un nouveau hub d’événements sera créé par le modèle. |
     | environmentDisplayName | Nom convivial facultatif à afficher dans les outils et les interfaces utilisateur à la place du nom de l’environnement. |
     | environmentSkuName | Nom du la référence SKU. Pour plus d’informations, consultez la [page Tarification d’Azure Time Series Insights](https://azure.microsoft.com/pricing/details/time-series-insights/).  |
     | environmentSkuCapacity | Capacité unitaire de la référence SKU. Pour plus d’informations, consultez la [page Tarification d’Azure Time Series Insights](https://azure.microsoft.com/pricing/details/time-series-insights/).|
     | environmentDataRetentionTime | Période minimale durant laquelle les événements de l’environnement seront interrogeables. La valeur doit être spécifiée au format ISO 8601, par exemple, `P30D` pour une stratégie de rétention de 30 jours. |
     | eventSourceDisplayName | Nom convivial facultatif à afficher dans les outils et les interfaces utilisateur à la place du nom de la source de l’événement. |
     | eventSourceTimestampPropertyName | Propriété de l’événement qui sera utilisée comme horodateur de la source de l’événement. Si aucune valeur n’est spécifiée pour timestampPropertyName, ou si la valeur Null ou une chaîne vide est spécifiée, l’heure de création de l’événement sera utilisée. |
     | eventSourceKeyName | Nom de la clé d’accès partagé que le service Azure Time Series Insights utilisera pour se connecter à l’Event Hub. |
     | accessPolicyReaderObjectIds | Liste des ID objet des utilisateurs et des applications d’Azure AD qui doivent avoir l’accès Lecteur à l’environnement. L’ID objet du principal du service s’obtient en appelant la cmdlet **Get-AzADUser** ou la cmdlet **Get-AzADServicePrincipal**. La création d’une stratégie d’accès n’est pas encore prise en charge pour les groupes Azure AD. |
     | accessPolicyContributorObjectIds | Liste des ID objet des utilisateurs ou des applications d’Azure AD qui doivent avoir l’accès Collaborateur à l’environnement. L’ID objet du principal du service s’obtient en appelant la cmdlet **Get-AzADUser** ou la cmdlet **Get-AzADServicePrincipal**. La création d’une stratégie d’accès n’est pas encore prise en charge pour les groupes Azure AD. |

   - Par exemple, le fichier de paramètres suivant serait utilisé pour créer un environnement et une source d’événement qui lirait les événements à partir d’un hub d’événements existant. Il crée également deux stratégies qui accordent l’accès Collaborateur à l’environnement.

     ```JSON
     {
         "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
         "contentVersion": "1.0.0.0",
         "parameters": {
             "eventHubNamespaceName": {
                 "value": "tsiTemplateTestNamespace"
             },
             "eventHubName": {
                 "value": "tsiTemplateTestEventHub"
             },
             "consumerGroupName": {
                 "value": "tsiTemplateTestConsumerGroup"
             },
             "environmentName": {
                 "value": "tsiTemplateTestEnvironment"
             },
             "eventSourceName": {
                 "value": "tsiTemplateTestEventSource"
             },
             "existingEventHubResourceId": {
                 "value": "/subscriptions/{yourSubscription}/resourceGroups/MyDemoRG/providers/Microsoft.EventHub/namespaces/tsiTemplateTestNamespace/eventhubs/tsiTemplateTestEventHub"
             },
             "accessPolicyContributorObjectIds": {
                 "value": [
                     "AGUID001-0000-0000-0000-000000000000",
                     "AGUID002-0000-0000-0000-000000000000"
                 ]
             }
         }
     }
     ```

   - Pour plus d’informations, consultez l’article [Paramètres](../azure-resource-manager/templates/parameter-files.md).

## <a name="deploy-the-quickstart-template-locally-using-powershell"></a>Déployer le modèle de démarrage rapide localement avec PowerShell

> [!IMPORTANT]
> Les opérations de ligne de commande affichées ci-dessous décrivent le [Module Az PowerShell](/powershell/azure/).

1. Dans PowerShell, connectez-vous à votre compte Azure.

    - À partir d’une invite de commandes PowerShell, exécutez la commande suivante :

      ```powershell
      Connect-AzAccount
      ```

    - Vous êtes invité à ouvrir une session sur votre compte Azure. Une fois connecté, exécutez la commande suivante pour afficher vos abonnements disponibles :

      ```powershell
      Get-AzSubscription
      ```

    - Cette commande renvoie la liste des abonnements Azure disponibles. Choisissez un abonnement pour la session en cours en exécutant la commande suivante. Remplacez `<YourSubscriptionId>` par le GUID de l’abonnement Azure que vous souhaitez utiliser :

      ```powershell
      Set-AzContext -SubscriptionID <YourSubscriptionId>
      ```

1. Créez un groupe de ressources s'il n'en existe pas.

   - Si vous n’avez pas de groupe de ressources, créez-en un avec la commande **New-AzResourceGroup**. Indiquez le nom du groupe de ressources et l'emplacement que vous souhaitez utiliser. Par exemple :

     ```powershell
     New-AzResourceGroup -Name MyDemoRG -Location "West US"
     ```

   - En cas de réussite, un résumé du nouveau groupe de ressources s’affiche.

     ```powershell
     ResourceGroupName : MyDemoRG
     Location          : westus
     ProvisioningState : Succeeded
     Tags              :
     ResourceId        : /subscriptions/<GUID>/resourceGroups/MyDemoRG
     ```

1. Testez le déploiement.

   - Validez votre déploiement en exécutant l’applet de commande `Test-AzResourceGroupDeployment`. Lorsque vous testez le déploiement, indiquez les paramètres exactement comme vous le feriez lors de l'exécution du déploiement.

     ```powershell
     Test-AzResourceGroupDeployment -ResourceGroupName MyDemoRG -TemplateFile <path to template file>\azuredeploy.json -TemplateParameterFile <path to parameters file>\azuredeploy.parameters.json
     ```

1. Créer le déploiement

    - Pour créer le déploiement, exécutez l’applet de commande `New-AzResourceGroupDeployment` et indiquez les paramètres nécessaires quand vous y êtes invité. Les paramètres incluent un nom pour votre déploiement, le nom de votre groupe de ressources, le chemin d’accès ou l’URL du fichier de modèle. Si le paramètre **Mode** n’est pas spécifié, la valeur par défaut **Incremental** est utilisée. Pour plus d’informations, consultez [Déploiements incrémentiels et complets](../azure-resource-manager/templates/deployment-modes.md).

    - La commande suivante vous invite à entrer les cinq paramètres requis dans la fenêtre PowerShell :

      ```powershell
      New-AzResourceGroupDeployment -Name MyDemoDeployment -ResourceGroupName MyDemoRG -TemplateFile <path to template file>\azuredeploy.json
      ```

    - Pour spécifier un fichier de paramètres à la place, utilisez la commande suivante :

      ```powershell
      New-AzResourceGroupDeployment -Name MyDemoDeployment -ResourceGroupName MyDemoRG -TemplateFile <path to template file>\azuredeploy.json -TemplateParameterFile <path to parameters file>\azuredeploy.parameters.json
      ```

    - Vous pouvez également utiliser des paramètres inclus lorsque vous exécutez l'applet de commande de déploiement. La commande est la suivante :

      ```powershell
      New-AzResourceGroupDeployment -Name MyDemoDeployment -ResourceGroupName MyDemoRG -TemplateFile <path to template file>\azuredeploy.json -parameterName "parameterValue"
      ```

    - Pour exécuter un déploiement [complet](../azure-resource-manager/templates/deployment-modes.md), affectez la valeur **Complet** au paramètre **Mode** :

      ```powershell
      New-AzResourceGroupDeployment -Name MyDemoDeployment -Mode Complete -ResourceGroupName MyDemoRG -TemplateFile <path to template file>\azuredeploy.json
      ```

1. Vérifier le déploiement

    - Si les ressources sont déployées avec succès, un résumé du déploiement s’affiche dans la fenêtre PowerShell :

      ```powershell
       DeploymentName          : MyDemoDeployment
       ResourceGroupName       : MyDemoRG
       ProvisioningState       : Succeeded
       Timestamp               : 10/11/2019 3:20:37 AM
       Mode                    : Incremental
       TemplateLink            :
       Parameters              :
                                 Name                                Type                       Value
                                 ==================================  =========================  ==========
                                 eventHubNewOrExisting               String                     new
                                 eventHubResourceGroup               String                     MyDemoRG
                                 eventHubNamespaceName               String                     tsiquickstartns
                                 eventHubName                        String                     tsiquickstarteh
                                 consumerGroupName                   String                     tsiquickstart
                                 environmentName                     String                     tsiquickstart
                                 environmentDisplayName              String                     tsiquickstart
                                 environmentSkuName                  String                     S1
                                 environmentSkuCapacity              Int                        1
                                 environmentDataRetentionTime        String                     P30D
                                 eventSourceName                     String                     tsiquickstart
                                 eventSourceDisplayName              String                     tsiquickstart
                                 eventSourceTimestampPropertyName    String
                                 eventSourceKeyName                  String                     manage
                                 accessPolicyReaderObjectIds         Array                      []
                                 accessPolicyContributorObjectIds    Array                      []
                                 location                            String                     westus

       Outputs                 :
                                  Name              Type                       Value
                                  ================  =========================  ==========
                                  dataAccessFQDN    String
                                  11aa1aa1-a1aa-1a1a-a11a-aa111a111a11.env.timeseries.azure.com

       DeploymentDebugLogLevel :
      ```

1. Déployer le modèle de démarrage rapide sur le Portail Azure

   - La page d’accueil du modèle de démarrage rapide sur GitHub comporte également un bouton **Déployer sur Azure**. Il ouvre une page Déploiement personnalisé sur le Portail Azure. Sur cette page, vous pouvez entrer ou sélectionner des valeurs pour chacun des paramètres des tables [Paramètres requis](#required-parameters) et [Paramètres facultatifs](#optional-parameters). Après avoir rempli les paramètres, cliquez sur le bouton **Acheter** pour lancer le déploiement du modèle.
    </br>
    </br>
    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-timeseriesinsights-environment-with-eventhub%2Fazuredeploy.json" target="_blank">
       <img src="https://azuredeploy.net/deploybutton.png" alt="The Deploy to Azure button."/>
    </a>

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur la gestion programmatique des ressources Azure Time Series Insights à l’aide des API REST, consultez [Gestion d’Azure Time Series Insights](/rest/api/time-series-insights-management/).