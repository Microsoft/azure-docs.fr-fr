---
title: Créer un hub d’événement avec capture activée - Azure Event Hubs | Microsoft Docs
description: Créer un espace de noms Azure Event Hubs avec un concentrateur d’événements et activer Capture à l’aide d’un modèle Azure Resource Manager
ms.topic: conceptual
ms.date: 06/23/2020
ms.custom: devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: 658defcef5c9d145d499eb72ae372d1a7edfb740
ms.sourcegitcommit: e7152996ee917505c7aba707d214b2b520348302
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/20/2020
ms.locfileid: "97705419"
---
# <a name="create-a-namespace-with-event-hub-and-enable-capture-using-a-template"></a>Créer un espace de noms avec un Event Hub et activer Capture à l’aide d’un modèle

Cet article explique comment utiliser un modèle Azure Resource Manager qui crée un espace de noms [Event Hubs](./event-hubs-about.md) avec une instance de concentrateur d’événements, et active la [fonctionnalité Capture](event-hubs-capture-overview.md) sur le concentrateur d’événements. L’article décrit comment définir les ressources à déployer et configurer les paramètres qui sont spécifiés lors de l’exécution du déploiement. Vous pouvez utiliser ce modèle pour vos propres déploiements, ou le personnaliser afin qu’il réponde à vos besoins.

Cet article explique également comment spécifier la capture des événements soit dans Azure Storage Blobs, soit dans Azure Data Lake Store, en fonction de la destination choisie.

Pour en savoir plus sur la création de modèles, voir [Création de modèles Azure Resource Manager][Authoring Azure Resource Manager templates]. Pour connaître la syntaxe JSON et les propriétés à utiliser dans un modèle, consultez [Types de ressources Microsoft.EventHub](/azure/templates/microsoft.eventhub/allversions).

Pour plus d’informations sur les modèles et les pratiques des conventions d’affectation de noms des ressources Azure, voir [Conventions d’affectation de noms des ressources Azure][Azure Resources naming conventions].

Pour les modèles terminés, cliquez sur les liens GitHub suivants :

- [Event Hub et activer le modèle de Capture vers le stockage][Event Hub and enable Capture to Storage template] 
- [Event Hub et activer le modèle Azure Data Lake Store][Event Hub and enable Capture to Azure Data Lake Store template]

> [!NOTE]
> Pour connaître les derniers modèles, recherchez Event Hubs dans la galerie de [modèles de démarrage rapide Azure][Azure Quickstart Templates] .
> 
> 

## <a name="what-will-you-deploy"></a>Qu'allez-vous déployer ?

Avec ce modèle, vous déployez un espace de noms Event Hubs avec un concentrateur d’événements, et activez [Event Hubs Capture](event-hubs-capture-overview.md). Event Hubs Capture vous permet de transmettre automatiquement les données de streaming des instances Event Hubs au stockage Blob Azure ou à Azure Data Lake Store et ce, dans l’intervalle de temps ou de taille que vous spécifiez. Cliquez sur le bouton suivant pour activer Event Hubs Capture dans le stockage Azure :

[![Déployer sur Azure](./media/event-hubs-resource-manager-namespace-event-hub/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-eventhubs-create-namespace-and-enable-capture%2Fazuredeploy.json)

Cliquez sur le bouton suivant pour activer Event Hubs Capture dans Azure Data Lake Store :

[![Déployer sur Azure](./media/event-hubs-resource-manager-namespace-event-hub/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-eventhubs-create-namespace-and-enable-capture-for-adls%2Fazuredeploy.json)

## <a name="parameters"></a>Paramètres

Azure Resource Manager vous permet de définir des paramètres pour les valeurs que vous voulez spécifier lorsque le modèle est déployé. Ce modèle inclut une section appelée `Parameters` , qui contient toutes les valeurs des paramètres. Vous devez définir un paramètre pour les valeurs qui varient en fonction du projet que vous déployez ou de l’environnement dans lequel vous effectuez le déploiement. Ne définissez pas de paramètres pour les valeurs qui restent inchangées. Chaque valeur de paramètre est utilisée dans le modèle pour définir les ressources déployées.

Le modèle définit les paramètres suivants.

### <a name="eventhubnamespacename"></a>eventHubNamespaceName

Nom de l’espace de noms Event Hubs à créer.

```json
"eventHubNamespaceName":{  
     "type":"string",
     "metadata":{  
         "description":"Name of the EventHub namespace"
      }
}
```

### <a name="eventhubname"></a>eventHubName

Nom du concentrateur d’événements créé dans l’espace de noms Event Hubs.

```json
"eventHubName":{  
    "type":"string",
    "metadata":{  
        "description":"Name of the event hub"
    }
}
```

### <a name="messageretentionindays"></a>messageRetentionInDays

Nombre de jours pendant lesquels conserver les messages dans le concentrateur d’événements. 

```json
"messageRetentionInDays":{
    "type":"int",
    "defaultValue": 1,
    "minValue":"1",
    "maxValue":"7",
    "metadata":{
       "description":"How long to retain the data in event hub"
     }
 }
```

### <a name="partitioncount"></a>partitionCount

Nombre de partitions à créer dans le concentrateur d’événements.

```json
"partitionCount":{
    "type":"int",
    "defaultValue":2,
    "minValue":2,
    "maxValue":32,
    "metadata":{
        "description":"Number of partitions chosen"
    }
 }
```

### <a name="captureenabled"></a>captureEnabled

Active la fonction Capture sur le concentrateur d’événements.

```json
"captureEnabled":{
    "type":"string",
    "defaultValue":"true",
    "allowedValues": [
    "false",
    "true"],
    "metadata":{
        "description":"Enable or disable the Capture for your event hub"
    }
 }
```
### <a name="captureencodingformat"></a>captureEncodingFormat

Format de codage que vous spécifiez pour sérialiser les données d’événement.

```json
"captureEncodingFormat":{
    "type":"string",
    "defaultValue":"Avro",
    "allowedValues":[
    "Avro"],
    "metadata":{
        "description":"The encoding format in which Capture serializes the EventData"
    }
}
```

### <a name="capturetime"></a>captureTime

L’intervalle de temps pendant lequel la fonctionnalité Event Hubs Capture démarre la capture des données.

```json
"captureTime":{
    "type":"int",
    "defaultValue":300,
    "minValue":60,
    "maxValue":900,
    "metadata":{
         "description":"The time window in seconds for the capture"
    }
}
```

### <a name="capturesize"></a>captureSize
L’intervalle de taille pour lequel la fonctionnalité Capture démarre la capture des données.

```json
"captureSize":{
    "type":"int",
    "defaultValue":314572800,
    "minValue":10485760,
    "maxValue":524288000,
    "metadata":{
        "description":"The size window in bytes for capture"
    }
}
```

### <a name="capturenameformat"></a>captureNameFormat

Le format de nom utilisé par Event Hubs Capture pour écrire les fichiers Avro. Notez qu’un format de nom de Capture doit contenir les champs `{Namespace}`, `{EventHub}`, `{PartitionId}`, `{Year}`, `{Month}`, `{Day}`, `{Hour}`, `{Minute}` et `{Second}`. Ceux-ci peuvent être organisés dans n’importe quel ordre, avec ou sans délimiteurs.
 
```json
"captureNameFormat": {
      "type": "string",
      "defaultValue": "{Namespace}/{EventHub}/{PartitionId}/{Year}/{Month}/{Day}/{Hour}/{Minute}/{Second}",
      "metadata": {
        "description": "A Capture Name Format must contain {Namespace}, {EventHub}, {PartitionId}, {Year}, {Month}, {Day}, {Hour}, {Minute} and {Second} fields. These can be arranged in any order with or without delimeters. E.g.  Prod_{EventHub}/{Namespace}\\{PartitionId}_{Year}_{Month}/{Day}/{Hour}/{Minute}/{Second}"
      }
    }
  
```

### <a name="apiversion"></a>apiVersion

Version d’API du modèle.

```json
 "apiVersion":{  
    "type":"string",
    "defaultValue":"2017-04-01",
    "metadata":{  
        "description":"ApiVersion used by the template"
    }
 }
```

Utilisez les paramètres suivants si vous choisissez le stockage Azure comme destination.

### <a name="destinationstorageaccountresourceid"></a>destinationStorageAccountResourceId

La fonctionnalité Capture nécessite un ID de ressource de compte de stockage Azure pour activer la capture dans le compte de stockage souhaité.

```json
 "destinationStorageAccountResourceId":{
    "type":"string",
    "metadata":{
        "description":"Your existing Storage account resource ID where you want the blobs be captured"
    }
 }
```

### <a name="blobcontainername"></a>blobContainerName

Conteneur d’objets blob dans lequel vos données d’événement sont capturées.

```json
 "blobContainerName":{
    "type":"string",
    "metadata":{
        "description":"Your existing storage container in which you want the blobs captured"
    }
}
```

Utilisez les paramètres suivants si vous choisissez Azure Data Lake Store Gen 1 comme destination. Vous devez définir des autorisations sur le chemin d’accès Data Lake Store dans lequel vous souhaitez capturer l’événement. Pour définir des autorisations, voir [Capturer des données dans Azure Data Lake Storage Gen1](event-hubs-capture-enable-through-portal.md#capture-data-to-azure-data-lake-storage-gen-1).

### <a name="subscriptionid"></a>subscriptionId

ID d’abonnement pour l’espace de noms Event Hubs et Azure Data Lake Store. Ces deux ressources doivent être liées au même ID d’abonnement.

```json
"subscriptionId": {
    "type": "string",
    "metadata": {
        "description": "Subscription ID of both Azure Data Lake Store and Event Hubs namespace"
     }
 }
```

### <a name="datalakeaccountname"></a>dataLakeAccountName

Le nom Azure Data Lake Store pour les événements capturés.

```json
"dataLakeAccountName": {
    "type": "string",
    "metadata": {
        "description": "Azure Data Lake Store name"
    }
}
```

### <a name="datalakefolderpath"></a>dataLakeFolderPath

Le chemin d’accès du dossier de destination pour les événements capturés. C’est le dossier dans votre Data Lake Store vers lequel les événements sont envoyés au cours de l’opération de capture. Pour définir des autorisations sur ce dossier, consultez [Utiliser Azure Data Lake Store pour capturer des données Event Hubs](../data-lake-store/data-lake-store-archive-eventhub-capture.md).

```json
"dataLakeFolderPath": {
    "type": "string",
    "metadata": {
        "description": "Destination capture folder path"
    }
}
```

## <a name="resources-to-deploy-for-azure-storage-as-destination-to-captured-events"></a>Ressources à déployer pour le stockage Azure en tant que destination pour les événements capturés

Crée un espace de noms de type **EventHub** avec un Event Hub et active Capture pour le Stockage Blob Azure.

```json
"resources":[  
      {  
         "apiVersion":"[variables('ehVersion')]",
         "name":"[parameters('eventHubNamespaceName')]",
         "type":"Microsoft.EventHub/Namespaces",
         "location":"[variables('location')]",
         "sku":{  
            "name":"Standard",
            "tier":"Standard"
         },
         "resources": [
    {
      "apiVersion": "2017-04-01",
      "name": "[parameters('eventHubNamespaceName')]",
      "type": "Microsoft.EventHub/Namespaces",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Standard"
      },
      "properties": {
        "isAutoInflateEnabled": "true",
        "maximumThroughputUnits": "7"
      },
      "resources": [
        {
          "apiVersion": "2017-04-01",
          "name": "[parameters('eventHubName')]",
          "type": "EventHubs",
          "dependsOn": [
            "[concat('Microsoft.EventHub/namespaces/', parameters('eventHubNamespaceName'))]"
          ],
          "properties": {
            "messageRetentionInDays": "[parameters('messageRetentionInDays')]",
            "partitionCount": "[parameters('partitionCount')]",
            "captureDescription": {
              "enabled": "true",
              "skipEmptyArchives": false,
              "encoding": "[parameters('captureEncodingFormat')]",
              "intervalInSeconds": "[parameters('captureTime')]",
              "sizeLimitInBytes": "[parameters('captureSize')]",
              "destination": {
                "name": "EventHubArchive.AzureBlockBlob",
                "properties": {
                  "storageAccountResourceId": "[parameters('destinationStorageAccountResourceId')]",
                  "blobContainer": "[parameters('blobContainerName')]",
                  "archiveNameFormat": "[parameters('captureNameFormat')]"
                }
              }
            }
          }

        }
      ]
    }
  ]
```

## <a name="resources-to-deploy-for-azure-data-lake-store-as-destination"></a>Ressources à déployer pour Azure Data Lake Store en tant que destination

Crée un espace de noms de type **EventHub** avec un Event Hub et active Capture pour Azure Data Lake Store.

```json
 "resources": [
        {
            "apiVersion": "2017-04-01",
            "name": "[parameters('namespaceName')]",
            "type": "Microsoft.EventHub/Namespaces",
            "location": "[variables('location')]",
            "sku": {
                "name": "Standard",
                "tier": "Standard"
            },
            "resources": [
                {
                    "apiVersion": "2017-04-01",
                    "name": "[parameters('eventHubName')]",
                    "type": "EventHubs",
                    "dependsOn": [
                        "[concat('Microsoft.EventHub/namespaces/', parameters('namespaceName'))]"
                    ],
                    "properties": {
                        "path": "[parameters('eventHubName')]",
                        "captureDescription": {
                            "enabled": "true",
                            "skipEmptyArchives": false,
                            "encoding": "[parameters('archiveEncodingFormat')]",
                            "intervalInSeconds": "[parameters('captureTime')]",
                            "sizeLimitInBytes": "[parameters('captureSize')]",
                            "destination": {
                                "name": "EventHubArchive.AzureDataLake",
                                "properties": {
                                    "DataLakeSubscriptionId": "[parameters('subscriptionId')]",
                                    "DataLakeAccountName": "[parameters('dataLakeAccountName')]",
                                    "DataLakeFolderPath": "[parameters('dataLakeFolderPath')]",
                                    "ArchiveNameFormat": "[parameters('captureNameFormat')]"
                                }
                            }
                        }
                    }
                }
            ]
        }
    ]
```

> [!NOTE]
> La propriété **skipEmptyArchives** vous permet d’activer ou de désactiver l’envoi de fichiers vides lorsqu’aucun événement ne se produit pendant la fenêtre de capture. 

## <a name="commands-to-run-deployment"></a>Commandes pour l’exécution du déploiement

[!INCLUDE [app-service-deploy-commands](../../includes/app-service-deploy-commands.md)]

## <a name="powershell"></a>PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Déployer votre modèle pour activer Event Hubs Capture dans le stockage Azure :
 
```powershell
New-AzResourceGroupDeployment -ResourceGroupName \<resource-group-name\> -TemplateFile https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-eventhubs-create-namespace-and-enable-capture/azuredeploy.json
```

Déployer votre modèle pour activer Event Hubs Capture dans Azure Data Lake Store :

```powershell
New-AzResourceGroupDeployment -ResourceGroupName \<resource-group-name\> -TemplateFile https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-eventhubs-create-namespace-and-enable-capture-for-adls/azuredeploy.json
```

## <a name="azure-cli"></a>Azure CLI

Stockage Blob Azure comme destination :

```azurecli
az deployment group create \<my-resource-group\> \<my-deployment-name\> --template-uri [https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-eventhubs-create-namespace-and-enable-capture/azuredeploy.json][]
```

Azure Data Lake Store comme destination :

```azurecli
az deployment group create \<my-resource-group\> \<my-deployment-name\> --template-uri [https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-eventhubs-create-namespace-and-enable-capture-for-adls/azuredeploy.json][]
```

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez également configurer la fonctionnalité Event Hubs Capture via le [portail Azure](https://portal.azure.com). Pour en savoir plus, voir [Activer Event Hubs Capture à l’aide du portail Azure](event-hubs-capture-enable-through-portal.md).

Vous pouvez en apprendre plus sur Event Hubs en consultant les liens suivants :

* [Vue d’ensemble d’Event Hubs](./event-hubs-about.md)
* [Créer un concentrateur d’événements](event-hubs-create.md)
* [FAQ sur les hubs d’événements](event-hubs-faq.md)

[Authoring Azure Resource Manager templates]: ../azure-resource-manager/templates/template-syntax.md
[Azure Quickstart Templates]:  https://azure.microsoft.com/documentation/templates/?term=event+hubs
[Azure Resources naming conventions]: /azure/cloud-adoption-framework/ready/azure-best-practices/naming-and-tagging
[Event hub and enable Capture to Storage template]: https://github.com/Azure/azure-quickstart-templates/tree/master/201-eventhubs-create-namespace-and-enable-capture
[Event hub and enable Capture to Azure Data Lake Store template]: https://github.com/Azure/azure-quickstart-templates/tree/master/201-eventhubs-create-namespace-and-enable-capture-for-adls
