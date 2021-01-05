---
title: Créer des ressources Azure Service Bus à l’aide de modèles
description: Utilisez les modèles Azure Resource Manager pour automatiser la création de ressources Service Bus
documentationcenter: .net
author: spelluru
ms.topic: article
ms.tgt_pltfrm: dotnet
ms.date: 06/23/2020
ms.author: spelluru
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: df8a7fde9114f03521f0e57e072f81a867efcf39
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89075254"
---
# <a name="create-service-bus-resources-using-azure-resource-manager-templates"></a>Création de ressources Service Bus à l’aide de modèles Azure Resource Manager

Cet article décrit comment créer et déployer des ressources Service Bus à l'aide de modèles Azure Resource Manager, de PowerShell et du fournisseur de ressources Service Bus.

Les modèles Azure Resource Manager vous permettent de définir les ressources à déployer pour une solution et de spécifier les paramètres et variables qui permettent d'entrer des valeurs pour les différents environnements. Le modèle est écrit en JSON et se compose d’expressions que vous pouvez utiliser pour construire des valeurs pour votre déploiement. Pour plus d’informations sur l’écriture de modèles Azure Resource Manager et sur le format du modèle, consultez [Structure et syntaxe de modèles Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md).

> [!NOTE]
> Les exemples de cet article montrent comment utiliser Azure Resource Manager pour créer un espace de noms et une entité de messagerie (file d'attente) Service Bus. Pour accéder à d’autres exemples de modèles, recherchez **Service Bus** dans la [galerie de modèles de démarrage rapide Azure][Azure Quickstart Templates gallery].

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="service-bus-resource-manager-templates"></a>Modèles Resource Manager Service Bus

Ces modèles Azure Resource Manager Service Bus sont disponibles au téléchargement et au déploiement. Cliquez sur les liens suivants pour plus d'informations sur chacun d’eux, ainsi que des liens vers les modèles sur GitHub :

* [Création d'un espace de noms Service Bus](service-bus-resource-manager-namespace.md)
* [Créer un espace de noms Service Bus avec file d’attente](service-bus-resource-manager-namespace-queue.md)
* [Créer un espace de noms Service Bus par rubrique et abonnement](service-bus-resource-manager-namespace-topic.md)
* [Créer un espace de noms Service Bus avec file d'attente et règle d’autorisation](service-bus-resource-manager-namespace-auth-rule.md)
* [Créer un modèle d’espace de noms Service Bus avec rubrique, abonnement et règle](service-bus-resource-manager-namespace-topic-with-rule.md)

## <a name="deploy-with-powershell"></a>Déployer avec PowerShell

La procédure suivante décrit comment utiliser PowerShell pour déployer un modèle Azure Resource Manager qui crée un espace de noms Service Bus de niveau Standard et une file d’attente au sein de cet espace de noms. Cet exemple est basé sur le modèle [Créer un espace de noms Service Bus avec file d’attente](https://github.com/Azure/azure-quickstart-templates/tree/master/201-servicebus-create-queue). Le flux de travail est approximativement le suivant :

1. Installez PowerShell.
2. Créez le modèle et (éventuellement) un fichier de paramètres.
3. Dans PowerShell, connectez-vous à votre compte Azure.
4. Créez un groupe de ressources s'il n'en existe pas.
5. Testez le déploiement.
6. Si vous le souhaitez, définissez le mode de déploiement.
7. Déployez le modèle.

Pour des informations complètes sur le déploiement de modèles Azure Resource Manager, consultez [Déployer des ressources à l’aide de modèles Resource Manager et d’Azure PowerShell][Deploy resources with Azure Resource Manager templates].

### <a name="install-powershell"></a>Installer PowerShell

Installez Azure PowerShell en suivant les instructions disponibles dans [Prise en main d’Azure PowerShell](/powershell/azure/get-started-azureps).

### <a name="create-a-template"></a>Créer un modèle

Clonez le dépôt ou copiez le modèle [201-servicebus-create-queue](https://github.com/Azure/azure-quickstart-templates/blob/master/201-servicebus-create-queue/azuredeploy.json) à partir de GitHub :

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "serviceBusNamespaceName": {
      "type": "string",
      "metadata": {
        "description": "Name of the Service Bus namespace"
      }
    },
    "serviceBusQueueName": {
      "type": "string",
      "metadata": {
        "description": "Name of the Queue"
      }
    }
  },
  "variables": {
    "defaultSASKeyName": "RootManageSharedAccessKey",
    "authRuleResourceId": "[resourceId('Microsoft.ServiceBus/namespaces/authorizationRules', parameters('serviceBusNamespaceName'), variables('defaultSASKeyName'))]",
    "sbVersion": "2017-04-01"
  },
  "resources": [
    {
      "apiVersion": "2017-04-01",
      "name": "[parameters('serviceBusNamespaceName')]",
      "type": "Microsoft.ServiceBus/Namespaces",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Standard"
      },
      "properties": {},
      "resources": [
        {
          "apiVersion": "2017-04-01",
          "name": "[parameters('serviceBusQueueName')]",
          "type": "Queues",
          "dependsOn": [
            "[concat('Microsoft.ServiceBus/namespaces/', parameters('serviceBusNamespaceName'))]"
          ],
          "properties": {
            "lockDuration": "PT5M",
            "maxSizeInMegabytes": "1024",
            "requiresDuplicateDetection": "false",
            "requiresSession": "false",
            "defaultMessageTimeToLive": "P10675199DT2H48M5.4775807S",
            "deadLetteringOnMessageExpiration": "false",
            "duplicateDetectionHistoryTimeWindow": "PT10M",
            "maxDeliveryCount": "10",
            "autoDeleteOnIdle": "P10675199DT2H48M5.4775807S",
            "enablePartitioning": "false",
            "enableExpress": "false"
          }
        }
      ]
    }
  ],
  "outputs": {
    "NamespaceConnectionString": {
      "type": "string",
      "value": "[listkeys(variables('authRuleResourceId'), variables('sbVersion')).primaryConnectionString]"
    },
    "SharedAccessPolicyPrimaryKey": {
      "type": "string",
      "value": "[listkeys(variables('authRuleResourceId'), variables('sbVersion')).primaryKey]"
    }
  }
}
```

### <a name="create-a-parameters-file-optional"></a>Créer un fichier de paramètres (facultatif)

Pour utiliser un fichier de paramètres facultatif, copiez le fichier [201-servicebus-create-queue](https://github.com/Azure/azure-quickstart-templates/blob/master/201-servicebus-create-queue/azuredeploy.parameters.json). Remplacez la valeur de `serviceBusNamespaceName` par le nom de l'espace de noms Service Bus que vous souhaitez créer dans ce déploiement, puis remplacez la valeur de `serviceBusQueueName` par le nom de la file d'attente que vous souhaitez créer.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "serviceBusNamespaceName": {
            "value": "<myNamespaceName>"
        },
        "serviceBusQueueName": {
            "value": "<myQueueName>"
        },
        "serviceBusApiVersion": {
            "value": "2017-04-01"
        }
    }
}
```

Pour plus d’informations, consultez l’article [Paramètres](../azure-resource-manager/templates/parameter-files.md).

### <a name="log-in-to-azure-and-set-the-azure-subscription"></a>Se connecter à Azure et définir l’abonnement Azure

À partir d’une invite de commandes PowerShell, exécutez la commande suivante :

```powershell
Connect-AzAccount
```

Vous êtes invité à ouvrir une session sur votre compte Azure. Une fois connecté, exécutez la commande suivante pour afficher vos abonnements disponibles :

```powershell
Get-AzSubscription
```

Cette commande renvoie la liste des abonnements Azure disponibles. Choisissez un abonnement pour la session en cours en exécutant la commande suivante. Remplacez `<YourSubscriptionId>` par le GUID de l’abonnement Azure que vous souhaitez utiliser :

```powershell
Set-AzContext -SubscriptionID <YourSubscriptionId>
```

### <a name="set-the-resource-group"></a>Définir le groupe de ressources

Si vous n’avez pas de groupe de ressources, créez-en un avec la commande **New-AzResourceGroup**. Indiquez le nom du groupe de ressources et l'emplacement que vous souhaitez utiliser. Par exemple :

```powershell
New-AzResourceGroup -Name MyDemoRG -Location "West US"
```

En cas de réussite, un résumé du nouveau groupe de ressources s’affiche.

```powershell
ResourceGroupName : MyDemoRG
Location          : westus
ProvisioningState : Succeeded
Tags              :
ResourceId        : /subscriptions/<GUID>/resourceGroups/MyDemoRG
```

### <a name="test-the-deployment"></a>test du déploiement

Validez votre déploiement en exécutant l’applet de commande `Test-AzResourceGroupDeployment`. Lorsque vous testez le déploiement, indiquez les paramètres exactement comme vous le feriez lors de l'exécution du déploiement.

```powershell
Test-AzResourceGroupDeployment -ResourceGroupName MyDemoRG -TemplateFile <path to template file>\azuredeploy.json
```

### <a name="create-the-deployment"></a>Créer le déploiement

Pour créer le déploiement, exécutez l’applet de commande `New-AzResourceGroupDeployment` et indiquez les paramètres nécessaires quand vous y êtes invité. Les paramètres incluent un nom pour votre déploiement, le nom de votre groupe de ressources, le chemin d’accès ou l’URL du fichier de modèle. Si le paramètre **Mode** n’est pas spécifié, la valeur par défaut **Incremental** est utilisée. Pour plus d’informations, consultez [Déploiements incrémentiels et complets](../azure-resource-manager/templates/deployment-modes.md).

La commande suivante vous invite à entrer les trois paramètres dans la fenêtre PowerShell :

```powershell
New-AzResourceGroupDeployment -Name MyDemoDeployment -ResourceGroupName MyDemoRG -TemplateFile <path to template file>\azuredeploy.json
```

Pour spécifier un fichier de paramètres à la place, utilisez la commande suivante :

```powershell
New-AzResourceGroupDeployment -Name MyDemoDeployment -ResourceGroupName MyDemoRG -TemplateFile <path to template file>\azuredeploy.json -TemplateParameterFile <path to parameters file>\azuredeploy.parameters.json
```

Vous pouvez également utiliser des paramètres inclus lorsque vous exécutez l'applet de commande de déploiement. La commande est la suivante :

```powershell
New-AzResourceGroupDeployment -Name MyDemoDeployment -ResourceGroupName MyDemoRG -TemplateFile <path to template file>\azuredeploy.json -parameterName "parameterValue"
```

Pour exécuter un déploiement [complet](../azure-resource-manager/templates/deployment-modes.md), affectez la valeur **Complet** au paramètre **Mode** :

```powershell
New-AzResourceGroupDeployment -Name MyDemoDeployment -Mode Complete -ResourceGroupName MyDemoRG -TemplateFile <path to template file>\azuredeploy.json
```

### <a name="verify-the-deployment"></a>Vérifier le déploiement
Si les ressources sont déployées avec succès, un résumé du déploiement s’affiche dans la fenêtre PowerShell :

```powershell
DeploymentName    : MyDemoDeployment
ResourceGroupName : MyDemoRG
ProvisioningState : Succeeded
Timestamp         : 4/19/2017 10:38:30 PM
Mode              : Incremental
TemplateLink      :
Parameters        :
                    Name             Type                       Value
                    ===============  =========================  ==========
                    serviceBusNamespaceName  String             <namespaceName>
                    serviceBusQueueName  String                 <queueName>
                    serviceBusApiVersion  String                2017-04-01

```

## <a name="next-steps"></a>Étapes suivantes
Vous avez maintenant vu le flux de travail et les commandes de base pour le déploiement d'un modèle Azure Resource Manager. Pour plus d'informations, consultez les liens suivants :

* [Présentation d’Azure Resource Manager][Azure Resource Manager overview]
* [Déployer des ressources à l’aide de modèles Resource Manager et d’Azure PowerShell][Deploy resources with Azure Resource Manager templates]
* [Création de modèles Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md)
* [Types de ressources Microsoft.ServiceBus](/azure/templates/microsoft.servicebus/allversions)

[Azure Resource Manager overview]: ../azure-resource-manager/management/overview.md
[Deploy resources with Azure Resource Manager templates]: ../azure-resource-manager/templates/deploy-powershell.md
[Azure Quickstart Templates gallery]: https://azure.microsoft.com/documentation/templates/?term=service+bus
