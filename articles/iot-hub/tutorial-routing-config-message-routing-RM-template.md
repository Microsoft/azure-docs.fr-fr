---
title: Configurer le routage des messages pour Azure IoT Hub à l’aide d’un modèle Azure Resource Manager
description: Configurer le routage des messages pour Azure IoT Hub à l’aide d’un modèle Azure Resource Manager
author: robinsh
manager: philmeagit st
ms.service: iot-hub
services: iot-hub
ms.topic: tutorial
ms.date: 03/25/2019
ms.author: robinsh
ms.custom: mvc
ms.openlocfilehash: e5ae5948c8baf1573393c73026c84d0f62e8693e
ms.sourcegitcommit: 3bcce2e26935f523226ea269f034e0d75aa6693a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92480103"
---
# <a name="tutorial-use-an-azure-resource-manager-template-to-configure-iot-hub-message-routing"></a>Tutoriel : Utiliser un modèle Azure Resource Manager pour configurer le routage des messages IoT Hub

[!INCLUDE [iot-hub-include-routing-intro](../../includes/iot-hub-include-routing-intro.md)]

[!INCLUDE [iot-hub-include-routing-create-resources](../../includes/iot-hub-include-routing-create-resources.md)]

## <a name="message-routing"></a>Routage de messages

[!INCLUDE [iot-hub-include-create-routing-description](../../includes/iot-hub-include-create-routing-description.md)]

## <a name="download-the-template-and-parameters-file"></a>Télécharger le modèle et le fichier de paramètres

Dans la deuxième partie de ce tutoriel, vous téléchargez et vous exécutez une application Visual Studio qui envoie des messages au hub IoT. Ce téléchargement contient un dossier où se trouvent le modèle et le fichier de paramètres Azure Resource Manager ainsi que les scripts Azure CLI et PowerShell.

Téléchargez maintenant les [exemples Azure IoT pour C#](https://github.com/Azure-Samples/azure-iot-samples-csharp/archive/master.zip). Décompressez le fichier master.zip. Le modèle Resource Manager et le fichier de paramètres sont dans les fichiers **template_iothub.json** et **template_iothub_parameters.json** , sous /iot-hub/Tutorials/Routing/SimulatedDevice/resources/.

## <a name="create-your-resources"></a>Créer vos ressources

Vous allez utiliser un modèle Azure Resource Manager (RM) pour créer toutes vos ressources. Les scripts Azure CLI et PowerShell peuvent exécuter quelques lignes de code à la fois. Un modèle Resource Manager peut être déployé en une seule étape. Cet article décrit chacune des sections séparément pour faciliter la compréhension. Il vous montre ensuite comment déployer le modèle et créer l’appareil virtuel de test. Une fois que le modèle est déployé, vous pouvez voir la configuration du routage des messages dans le portail.

Plusieurs noms de ressources doivent être globalement uniques, comme le nom du hub IoT et le nom du compte de stockage. Pour faciliter le nommage des ressources, une valeur alphanumérique aléatoire, générée à partir de la date/heure actuelle, est ajoutée à ces noms de ressources. 

Si vous examinez le modèle, vous verrez où les variables sont définies pour ces ressources qui prennent le paramètre passé et concatènent la valeur *randomValue* au paramètre. 

La section suivante présente les différents paramètres utilisés.

### <a name="parameters"></a>Paramètres

La plupart de ces paramètres ont des valeurs par défaut. Les paramètres qui finissent par **_in** sont concaténés avec la valeur *randomValue* pour les rendre globalement uniques. 

**randomValue**  : cette valeur est générée à partir de la date/heure actuelle à laquelle vous déployez le modèle. Ce champ ne figure pas dans le fichier de paramètres, car il est généré directement dans le modèle.

**subscriptionId**  : ce champ est automatiquement défini sur l’abonnement dans lequel vous déployez le modèle. Ce champ ne figure pas dans le fichier de paramètres, car il est défini automatiquement.

**IoTHubName_in**  : ce champ indique le nom du hub IoT de base, qui est concaténé avec la valeur randomValue pour être globalement unique.

**location**  : ce champ indique la région Azure où vous déployez le modèle (par exemple, « westus »).

**consumer_group**  : ce champ indique le groupe de consommateurs défini pour les messages qui entrent par le point de terminaison de routage. Il permet de filtrer les résultats dans Azure Stream Analytics. Par exemple, vous pouvez afficher tout le flux de données ou, si vous avez des données transitant par le groupe de consommateurs **Contoso** , vous pouvez configurer un flux Azure Stream Analytics (et un rapport Power BI) qui affiche uniquement les entrées correspondantes. Ce champ est utilisé dans la deuxième partie de ce tutoriel.

**sku_name**  : ce champ indique le niveau de mise à l’échelle du hub IoT. Cette valeur doit être définie sur S1 ou un niveau supérieur. Le niveau gratuit n’est pas possible dans ce tutoriel, car il n’autorise pas les points de terminaison multiples.

**sku_units**  : ce champ est associé au champ **sku_name**  ; il indique le nombre d’unités IoT Hub pouvant être utilisées.

**d2c_partitions**  : ce champ indique le nombre de partitions utilisées pour le flux d’événements.

**storageAccountName_in**  : ce champ indique le nom du compte de stockage à créer. Les messages sont routés vers un conteneur dans le compte de stockage. Ce champ est concaténé avec la valeur randomValue pour rendre le nom globalement unique.

**storageContainerName**  : ce champ indique le nom du conteneur dans lequel les messages routés vers le compte de stockage sont stockés.

**storage_endpoint**  : ce champ indique le nom du point de terminaison du compte de stockage utilisé par le routage des messages.

**service_bus_namespace_in**  : ce champ indique le nom de l’espace de noms Service Bus à créer. Cette valeur est concaténée avec la valeur randomValue pour rendre le nom globalement unique.

**service_bus_queue_in**  : ce champ indique le nom de la file d’attente Service Bus utilisée pour le routage des messages. Cette valeur est concaténée avec la valeur randomValue pour rendre le nom globalement unique.

**AuthRules_sb_queue**  : ce champ spécifie les règles d’autorisation de la file d’attente Service Bus qui sont utilisées pour récupérer la chaîne de connexion de la file d’attente.

### <a name="variables"></a>Variables

Utilisées dans le modèle, ces valeurs sont essentiellement dérivées des paramètres.

**queueAuthorizationRuleResourceId**  : ce champ indique l’ID de ressource pour la règle d’autorisation de la file d’attente Service Bus. L’ID de ressource est ensuite utilisé pour récupérer la chaîne de connexion de la file d’attente.

**iotHubName**  : ce champ indique le nom du hub IoT après sa concaténation avec la valeur randomValue. 

**storageAccountName**  : ce champ indique le nom du compte de stockage après sa concaténation avec la valeur randomValue. 

**service_bus_namespace**  : ce champ indique l’espace de noms après sa concaténation avec la valeur randomValue.

**service_bus_queue**  : ce champ indique le nom de la file d’attente Service Bus après sa concaténation avec la valeur randomValue.

**sbVersion**  : version de l’API Service Bus à utiliser. Dans ce cas, la version est « 2017-04-01 ».

### <a name="resources-storage-account-and-container"></a>Ressources : compte de stockage et conteneur

La première ressource créée est le compte de stockage, avec le conteneur vers lequel les messages sont routés. Le conteneur est une ressource sous-jacente au compte de stockage. Il comporte une clause de dépendance (`dependsOn`) avec le compte de stockage, qui impose de créer le compte de stockage avant le conteneur.

Cette section ressemble à ceci :

```json
{
    "type": "Microsoft.Storage/storageAccounts",
    "name": "[variables('storageAccountName')]",
    "apiVersion": "2018-07-01",
    "location": "[parameters('location')]",
    "sku": {
        "name": "Standard_LRS",
        "tier": "Standard"
    },
    "kind": "Storage",
    "properties": {},
    "resources": [
        {
        "type": "blobServices/containers",
        "apiVersion": "2018-07-01",
        "name": "[concat('default/', parameters('storageContainerName'))]",
        "properties": {
            "publicAccess": "None"
            } ,
        "dependsOn": [
            "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
            ]
        }
    ]
}
```

### <a name="resources-service-bus-namespace-and-queue"></a>Ressources : espace de noms et file d’attente Service Bus

La deuxième ressource créée est l’espace de noms Service Bus, avec la file d’attente Service Bus vers laquelle les messages sont routés. La référence SKU est définie sur Standard. La version de l’API est récupérée des variables. L’espace de noms Service Bus est également configuré pour être activé au déploiement de cette section (status:Active). 

```json
{
    "type": "Microsoft.ServiceBus/namespaces",
    "comments": "The Sku should be 'Standard' for this tutorial.",
    "sku": {
        "name": "Standard",
        "tier": "Standard"
    },
    "name": "[variables('service_bus_namespace')]",
    "apiVersion": "[variables('sbVersion')]",
    "location": "[parameters('location')]",
    "properties": {
        "provisioningState": "Succeeded",
        "metricId": "[concat('a4295411-5eff-4f81-b77e-276ab1ccda12:', variables('service_bus_namespace'))]",
        "serviceBusEndpoint": "[concat('https://', variables('service_bus_namespace'),'.servicebus.windows.net:443/')]",
        "status": "Active"
    },
    "dependsOn": []
}
```

Cette section crée la file d’attente Service Bus. Cette partie du script contient une clause `dependsOn` qui garantit que l’espace de noms est créé avant la file d’attente.

```json
{
    "type": "Microsoft.ServiceBus/namespaces/queues",
    "name": "[concat(variables('service_bus_namespace'), '/', variables('service_bus_queue'))]",
    "apiVersion": "[variables('sbVersion')]",
    "location": "[parameters('location')]",
    "scale": null,
    "properties": {},
    "dependsOn": [
        "[resourceId('Microsoft.ServiceBus/namespaces', variables('service_bus_namespace'))]"
    ]
}
```

### <a name="resources-iot-hub-and-message-routing"></a>Ressources : hub IoT et routage des messages

Maintenant que le compte de stockage et la file d’attente Service Bus ont été créés, vous créez le hub IoT qui leur envoie les messages. Le modèle Resource Manager utilise des clauses `dependsOn` afin d’empêcher la création du hub avant la création des ressources Service Bus et du compte de stockage. 

Voici la première partie de la section du hub IoT. Cette partie du modèle configure les dépendances et commence par les propriétés.

```json
{
    "apiVersion": "2018-04-01",
    "type": "Microsoft.Devices/IotHubs",
    "name": "[variables('IoTHubName')]",
    "location": "[parameters('location')]",
    "dependsOn": [
        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]",
        "[resourceId('Microsoft.ServiceBus/namespaces', variables('service_bus_namespace'))]",
        "[resourceId('Microsoft.ServiceBus/namespaces/queues', variables('service_bus_namespace'), variables('service_bus_queue'))]"
    ],
    "properties": {
        "eventHubEndpoints": {}
            "events": {
                "retentionTimeInDays": 1,
                "partitionCount": "[parameters('d2c_partitions')]"
                }
            },
```

La section suivante configure le routage des messages pour le hub IoT. Il y a d’abord la section relative aux points de terminaison. Cette partie du modèle configure les points de terminaison de routage pour la file d’attente Service Bus et le compte de stockage, y compris les chaînes de connexion.

Pour créer la chaîne de connexion de la file d’attente, vous devez utiliser la valeur queueAuthorizationRulesResourcedId, qui est récupérée inline. Pour créer la chaîne de connexion du compte de stockage, vous devez récupérer la clé de stockage primaire et l’utiliser au format de la chaîne de connexion.

Dans la section de configuration des points de terminaison, vous définissez également le format blob sur `AVRO` ou `JSON`.

[!INCLUDE [iot-hub-include-blob-storage-format](../../includes/iot-hub-include-blob-storage-format.md)]

 ```json
"routing": {
    "endpoints": {
        "serviceBusQueues": [
        {
            "connectionString": "[Concat('Endpoint=sb://',variables('service_bus_namespace'),'.servicebus.windows.net/;SharedAccessKeyName=',parameters('AuthRules_sb_queue'),';SharedAccessKey=',listkeys(variables('queueAuthorizationRuleResourceId'),variables('sbVersion')).primaryKey,';EntityPath=',variables('service_bus_queue'))]",
            "name": "[parameters('service_bus_queue_endpoint')]",
            "subscriptionId": "[parameters('subscriptionId')]", 
            "resourceGroup": "[resourceGroup().Name]"
        }
        ],
        "serviceBusTopics": [],
        "eventHubs": [],
        "storageContainers": [
            {
                "connectionString": 
                "[Concat('DefaultEndpointsProtocol=https;AccountName=',variables('storageAccountName'),';AccountKey=',listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName')), providers('Microsoft.Storage', 'storageAccounts').apiVersions[0]).keys[0].value)]",
                "containerName": "[parameters('storageContainerName')]",
                "fileNameFormat": "{iothub}/{partition}/{YYYY}/{MM}/{DD}/{HH}/{mm}",
                "batchFrequencyInSeconds": 100,
                "maxChunkSizeInBytes": 104857600,
                "encoding": "avro",
                "name": "[parameters('storage_endpoint')]",
                "subscriptionId": "[parameters('subscriptionId')]",
                "resourceGroup": "[resourceGroup().Name]"
            }
        ]
    },
```

La section suivante configure les routes des messages jusqu’aux points de terminaison. Une route est configurée pour chaque point de terminaison. Il y a donc une route pour la file d’attente Service Bus et une route pour le conteneur du compte de stockage.

N’oubliez pas que la condition de requête pour les messages routés vers le stockage est `level="storage"`, et que celle pour les messages routés vers la file d’attente Service Bus est `level="critical"`.

```json
"routes": [
    {
        "name": "contosoStorageRoute",
        "source": "DeviceMessages",
        "condition": "level=\"storage\"",
        "endpointNames": [
            "[parameters('storage_endpoint')]"
            ],
        "isEnabled": true
    },
    {
        "name": "contosoSBQueueRoute",
        "source": "DeviceMessages",
        "condition": "level=\"critical\"",
        "endpointNames": [
            "[parameters('service_bus_queue_endpoint')]"
            ],
        "isEnabled": true
    }
],
```

Ce code JSON montre le reste de la section du hub IoT, où figurent des informations par défaut et la référence SKU du hub.

```json
            "fallbackRoute": {
                "name": "$fallback",
                "source": "DeviceMessages",
                "condition": "true",
                "endpointNames": [
                    "events"
                ],
                "isEnabled": true
            }
        },
        "storageEndpoints": {
            "$default": {
                "sasTtlAsIso8601": "PT1H",
                "connectionString": "",
                "containerName": ""
            }
        },
        "messagingEndpoints": {
            "fileNotifications": {
                "lockDurationAsIso8601": "PT1M",
                "ttlAsIso8601": "PT1H",
                "maxDeliveryCount": 10
            }
        },
        "enableFileUploadNotifications": false,
        "cloudToDevice": {
            "maxDeliveryCount": 10,
            "defaultTtlAsIso8601": "PT1H",
            "feedback": {
                "lockDurationAsIso8601": "PT1M",
                "ttlAsIso8601": "PT1H",
                "maxDeliveryCount": 10
            }
        }
    },
    "sku": {
        "name": "[parameters('sku_name')]",
        "capacity": "[parameters('sku_units')]"
    }
}
```

### <a name="resources-service-bus-queue-authorization-rules"></a>Ressources : règles d’autorisation de file d’attente Service Bus

La règle d’autorisation de la file d’attente Service Bus est utilisée pour récupérer la chaîne de connexion de la file d’attente Service Bus. Elle utilise une clause `dependsOn` pour empêcher sa création avant la création de l’espace de noms Service Bus et de la file d’attente Service Bus.

```json
{
    "type": "Microsoft.ServiceBus/namespaces/queues/authorizationRules",
    "name": "[concat(variables('service_bus_namespace'), '/', variables('service_bus_queue'), '/', parameters('AuthRules_sb_queue'))]",
    "apiVersion": "[variables('sbVersion')]",
    "location": "[parameters('location')]",
    "scale": null,
    "properties": {
        "rights": [
            "Send"
        ]
    },
    "dependsOn": [
        "[resourceId('Microsoft.ServiceBus/namespaces', variables('service_bus_namespace'))]",
        "[resourceId('Microsoft.ServiceBus/namespaces/queues', variables('service_bus_namespace'), variables('service_bus_queue'))]"
    ]
},
```

### <a name="resources-consumer-group"></a>Ressources : Groupe de consommateurs

Dans cette section, vous créez un groupe de consommateurs pour les données IoT Hub, qui sera utilisé par Azure Stream Analytics dans la deuxième partie de ce tutoriel.

```json
{
    "type": "Microsoft.Devices/IotHubs/eventHubEndpoints/ConsumerGroups",
    "name": "[concat(variables('iotHubName'), '/events/',parameters('consumer_group'))]",
    "apiVersion": "2018-04-01",
    "dependsOn": [
        "[concat('Microsoft.Devices/IotHubs/', variables('iotHubName'))]"
    ]
}
```

### <a name="resources-outputs"></a>Ressources : Outputs

Si vous souhaitez retourner une valeur à afficher au script de déploiement, utilisez une section Outputs. Cette partie du modèle retourne la chaîne de connexion de la file d’attente Service Bus. Il n’est pas obligatoire de retourner une valeur, mais nous incluons cet exemple pour vous montrer comment retourner des résultats au script d’appel.

```json
"outputs": {
    "sbq_connectionString": {
      "type": "string",
      "value": "[Concat('Endpoint=sb://',variables('service_bus_namespace'),'.servicebus.windows.net/;SharedAccessKeyName=',parameters('AuthRules_sb_queue'),';SharedAccessKey=',listkeys(variables('queueAuthorizationRuleResourceId'),variables('sbVersion')).primaryKey,';EntityPath=',variables('service_bus_queue'))]"
    }
  }
```

## <a name="deploy-the-rm-template"></a>Déployer le modèle Resource Manager

Pour déployer le modèle sur Azure, chargez le modèle et le fichier de paramètres dans Azure Cloud Shell, puis exécutez un script de déploiement du modèle. Ouvrez Azure Cloud Shell et connectez-vous. Cet exemple utilise PowerShell.

Pour charger les fichiers, sélectionnez l’icône **Charger/télécharger des fichiers** dans la barre de menus, puis choisissez Charger.

![Capture d’écran qui met en évidence l’icône Charger/Télécharger des fichiers.](media/tutorial-routing-config-message-routing-RM-template/CloudShell_upload_files.png)

Dans l’Explorateur de fichiers qui s’affiche, recherchez les fichiers sur votre disque local et sélectionnez-les, puis choisissez **Ouvrir**.

Une fois que les fichiers ont été chargés, une boîte de dialogue de résultats s’affiche, comme celle-ci.

![Barre de menus Cloud Shell avec l’icône Charger/télécharger des fichiers en surbrillance](media/tutorial-routing-config-message-routing-RM-template/CloudShell_upload_results.png)

Les fichiers sont chargés dans le partage utilisé par votre instance Cloud Shell. 

Exécutez le script pour lancer le déploiement. La dernière ligne de ce script récupère la variable configurée pour être retournée, c’est-à-dire la chaîne de connexion de la file d’attente Service Bus.

Le script définit et utilise les variables suivantes :

**$RGName** indique le nom du groupe de ressources où déployer le modèle. Ce champ est créé avant de déployer le modèle.

**$location** indique l’emplacement Azure à utiliser pour le modèle, par exemple, « westus ».

**deploymentname** est un nom que vous attribuez au déploiement pour récupérer la valeur de la variable retournée.

Voici le script PowerShell. Copiez ce script PowerShell et collez-le dans la fenêtre Cloud Shell, puis appuyez sur Entrée pour l’exécuter.

```powershell
$RGName="ContosoResources"
$location = "westus"
$deploymentname="contoso-routing"

# Remove the resource group if it already exists. 
#Remove-AzResourceGroup -name $RGName 
# Create the resource group.
New-AzResourceGroup -name $RGName -Location $location 

# Set a path to the parameter file. 
$parameterFile = "$HOME/template_iothub_parameters.json"
$templateFile = "$HOME/template_iothub.json"

# Deploy the template.
New-AzResourceGroupDeployment `
    -Name $deploymentname `
    -ResourceGroupName $RGName `
    -TemplateParameterFile $parameterFile `
    -TemplateFile $templateFile `
    -verbose

# Get the returning value of the connection string.
(Get-AzResourceGroupDeployment -ResourceGroupName $RGName -Name $deploymentname).Outputs.sbq_connectionString.value
```

Si vous voyez des erreurs de script, modifiez le script localement, rechargez-le dans Cloud Shell et réexécutez le script. Quand le script s’exécute sans erreur, vous pouvez passer à l’étape suivante.

## <a name="create-simulated-device"></a>Créer un appareil simulé

[!INCLUDE [iot-hub-include-create-simulated-device-portal](../../includes/iot-hub-include-create-simulated-device-portal.md)]

## <a name="view-message-routing-in-the-portal"></a>Visualiser le routage des messages dans le portail

[!INCLUDE [iot-hub-include-view-routing-in-portal](../../includes/iot-hub-include-view-routing-in-portal.md)]

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez configuré toutes les ressources et les routes des messages, passez au tutoriel suivant pour découvrir comment traiter et afficher les informations relatives aux messages routés.

> [!div class="nextstepaction"]
> [Partie 2 - Voir les résultats du routage des messages](tutorial-routing-view-message-routing-results.md)
