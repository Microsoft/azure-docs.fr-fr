---
title: Automatiser le déploiement des ressources d’application de fonction dans Azure
description: Découvrez comment créer un modèle Azure Resource Manager qui déploie votre application de fonction.
ms.assetid: d20743e3-aab6-442c-a836-9bcea09bfd32
ms.topic: conceptual
ms.date: 04/03/2019
ms.custom: fasttrack-edit
ms.openlocfilehash: 2403e3a4bfe7824659a9c34c407e55e6c56e6046
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95994653"
---
# <a name="automate-resource-deployment-for-your-function-app-in-azure-functions"></a>Automatiser le déploiement de ressources pour votre application de fonction dans Azure Functions

Vous pouvez utiliser un modèle Azure Resource Manager pour déployer une application de fonction. Cet article présente les ressources et paramètres nécessaires pour effectuer cette opération. Vous devrez peut-être déployer des ressources supplémentaires, selon les [déclencheurs et liaisons](functions-triggers-bindings.md) présents dans votre application de fonction.

Pour en savoir plus sur la création de modèles, voir [Création de modèles Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md).

Pour des exemples de modèles, consultez :
- [Function app on Consumption plan] (Application de fonction dans le plan Consommation)
- [Function app on Azure App Service plan] (Application de fonction dans le plan Azure App Service)

## <a name="required-resources"></a>Ressources nécessaires

Un déploiement d’Azure Functions comprend généralement les ressources suivantes :

| Ressource                                                                           | Condition requise | Référence sur la syntaxe et les propriétés                                                         |
|------------------------------------------------------------------------------------|-------------|-----------------------------------------------------------------------------------------|
| Une application de fonction                                                                     | Obligatoire    | [Microsoft.Web/sites](/azure/templates/microsoft.web/sites)                             |
| Un compte de [stockage Azure](../storage/index.yml)                                   | Obligatoire    | [Microsoft.Storage/storageAccounts](/azure/templates/microsoft.storage/storageaccounts) |
| Un composant [Application Insights](../azure-monitor/app/app-insights-overview.md) | Facultatif    | [Microsoft.Insights/components](/azure/templates/microsoft.insights/components)         |
| Un [plan d’hébergement](./functions-scale.md)                                             | Facultatif<sup>1</sup>    | [Microsoft.Web/serverfarms](/azure/templates/microsoft.web/serverfarms)                 |

<sup>1</sup>Un plan d’hébergement n’est requis que si vous choisissez d’exécuter votre application de fonction sur un [plan Premium](./functions-premium-plan.md) ou sur un [plan App Service](../app-service/overview-hosting-plans.md).

> [!TIP]
> Bien que ce ne soit pas obligatoire, il est fortement recommandé de configurer Application Insights pour votre application.

<a name="storage"></a>
### <a name="storage-account"></a>Compte de stockage

Un compte de stockage Azure est nécessaire pour une application de fonction. Vous avez besoin d’un compte à usage général qui prend en charge les objets blob, les tables, les files d’attente et les fichiers. Pour plus d’informations, consultez [Configuration requise du compte de stockage Azure Functions](storage-considerations.md#storage-account-requirements).

```json
{
    "type": "Microsoft.Storage/storageAccounts",
    "name": "[variables('storageAccountName')]",
    "apiVersion": "2019-06-01",
    "location": "[resourceGroup().location]",
    "kind": "StorageV2",
    "sku": {
        "name": "[parameters('storageAccountType')]"
    }
}
```

En outre, la propriété `AzureWebJobsStorage` doit être spécifiée comme paramètre d’application dans la configuration du site. Si l’application de fonction n’utilise pas Application Insights pour l’analyse, elle doit également spécifier `AzureWebJobsDashboard` comme paramètre d’application.

Le runtime d’Azure Functions utilise la chaîne de connexion `AzureWebJobsStorage` pour créer des files d’attente internes.  Lorsque Application Insights n’est pas activé, le runtime utilise la chaîne de connexion `AzureWebJobsDashboard` pour se connecter au stockage de table Azure et alimenter l’onglet **Surveiller** du portail.

Ces propriétés sont spécifiées dans la collection `appSettings` de l’objet `siteConfig` :

```json
"appSettings": [
    {
        "name": "AzureWebJobsStorage",
        "value": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountid'),'2019-06-01').keys[0].value)]"
    },
    {
        "name": "AzureWebJobsDashboard",
        "value": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountid'),'2019-06-01').keys[0].value)]"
    }
]
```

### <a name="application-insights"></a>Application Insights

Application Insights est recommandé pour la surveillance de vos applications de fonction. La ressource Application Insights est définie avec le type **Microsoft.Insights/Components** et le genre **web** :

```json
        {
            "apiVersion": "2015-05-01",
            "name": "[variables('appInsightsName')]",
            "type": "Microsoft.Insights/components",
            "kind": "web",
            "location": "[resourceGroup().location]",
            "tags": {
                "[concat('hidden-link:', resourceGroup().id, '/providers/Microsoft.Web/sites/', variables('functionAppName'))]": "Resource"
            },
            "properties": {
                "Application_Type": "web",
                "ApplicationId": "[variables('appInsightsName')]"
            }
        },
```

De plus, la clé d’instrumentation doit être fourni à l’application de fonction à l’aide du paramètre d’application `APPINSIGHTS_INSTRUMENTATIONKEY`. Cette propriété est spécifiée dans la collection `appSettings` de l’objet `siteConfig` :

```json
"appSettings": [
    {
        "name": "APPINSIGHTS_INSTRUMENTATIONKEY",
        "value": "[reference(resourceId('microsoft.insights/components/', variables('appInsightsName')), '2015-05-01').InstrumentationKey]"
    }
]
```

### <a name="hosting-plan"></a>Plan d’hébergement

Il existe plusieurs définitions du plan d’hébergement :
* [Plan Consommation](#consumption) (par défaut)
* [Plan Premium](#premium)
* [Plan App Service](#app-service-plan)

### <a name="function-app"></a>Conteneur de fonctions

La ressource d’application de fonction est définie à l’aide d’une ressource de type **Microsoft.Web/sites** et de genre **functionapp** :

```json
{
    "apiVersion": "2015-08-01",
    "type": "Microsoft.Web/sites",
    "name": "[variables('functionAppName')]",
    "location": "[resourceGroup().location]",
    "kind": "functionapp",
    "dependsOn": [
        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]",
        "[resourceId('Microsoft.Insights/components', variables('appInsightsName'))]"
    ]
}
```

> [!IMPORTANT]
> Si vous définissez explicitement un plan d’hébergement, un élément supplémentaire est nécessaire dans le tableau dependsOn : `"[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]"`

Une application de fonction doit inclure les paramètres d’application suivants :

| Nom du paramètre                 | Description                                                                               | Exemples de valeurs                        |
|------------------------------|-------------------------------------------------------------------------------------------|---------------------------------------|
| AzureWebJobsStorage          | Chaîne de connexion à un compte de stockage que le runtime Functions utilise pour la file d’attente interne | Voir [Compte de stockage](#storage)       |
| FUNCTIONS_EXTENSION_VERSION  | Version du runtime Azure Functions                                                | `~2`                                  |
| FUNCTIONS_WORKER_RUNTIME     | Pile de langage à utiliser pour les fonctions dans cette application                                   | `dotnet`, `node`, `java`, `python` ou`powershell` |
| WEBSITE_NODE_DEFAULT_VERSION | Nécessaire uniquement si vous utilisez la pile de langage `node`. Spécifie la version à utiliser              | `10.14.1`                             |

Ces propriétés sont spécifiées dans la collection `appSettings` de la propriété `siteConfig` :

```json
"properties": {
    "siteConfig": {
        "appSettings": [
            {
                "name": "AzureWebJobsStorage",
                "value": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountid'),'2019-06-01').keys[0].value)]"
            },
            {
                "name": "FUNCTIONS_WORKER_RUNTIME",
                "value": "node"
            },
            {
                "name": "WEBSITE_NODE_DEFAULT_VERSION",
                "value": "10.14.1"
            },
            {
                "name": "FUNCTIONS_EXTENSION_VERSION",
                "value": "~2"
            }
        ]
    }
}
```

<a name="consumption"></a>

## <a name="deploy-on-consumption-plan"></a>Déployer sur un plan Consommation

Le plan consommation alloue automatiquement de la puissance de calcul lorsque votre code est en cours d’exécution, s’adapte en fonction des besoins pour gérer la charge, puis se met à l’échelle lorsque le code n’est pas en cours d’exécution. Vous n’avez pas à payer pour des machines virtuelles inactives ni à disposer d’une capacité de réserve à l’avance. Pour en savoir plus, voir [Mise à l’échelle et hébergement d’Azure Functions](functions-scale.md#consumption-plan).

Pour un exemple de modèle Azure Resource Manager, consultez [Function app on Consumption plan] (Application de fonction dans le plan Consommation).

### <a name="create-a-consumption-plan"></a>Créer un plan Consommation

Un plan Consommation ne doit pas nécessairement être défini. Un tel plan est automatiquement créé ou sélectionné sur une base régionale lorsque vous créez la ressource d’application de fonction.

Le plan Consommation est un type spécial de ressource « serverfarm ». Pour Windows, vous pouvez le spécifier en utilisant la valeur `Dynamic` pour les propriétés `computeMode` et `sku` :

```json
{  
   "type":"Microsoft.Web/serverfarms",
   "apiVersion":"2016-09-01",
   "name":"[variables('hostingPlanName')]",
   "location":"[resourceGroup().location]",
   "properties":{  
      "name":"[variables('hostingPlanName')]",
      "computeMode":"Dynamic"
   },
   "sku":{  
      "name":"Y1",
      "tier":"Dynamic",
      "size":"Y1",
      "family":"Y",
      "capacity":0
   }
}
```

> [!NOTE]
> Le plan Consommation ne peut pas être défini explicitement pour Linux. Il est créé automatiquement.

Si vous définissez explicitement votre plan Consommation, vous devez définir la propriété `serverFarmId` sur l’application afin qu’elle pointe vers l’ID de ressource du plan. Vous devez vous assurer que l’application de fonction a également un paramètre `dependsOn` pour le plan.

### <a name="create-a-function-app"></a>Créer une application de fonction

#### <a name="windows"></a>Windows

Sous Windows, un plan Consommation nécessite deux paramètres supplémentaires dans la configuration du site : `WEBSITE_CONTENTAZUREFILECONNECTIONSTRING` et `WEBSITE_CONTENTSHARE`. Ces propriétés configurent le compte de stockage et le chemin de fichier où le code de l’application de fonction et la configuration sont stockés.

```json
{
    "apiVersion": "2016-03-01",
    "type": "Microsoft.Web/sites",
    "name": "[variables('functionAppName')]",
    "location": "[resourceGroup().location]",
    "kind": "functionapp",
    "dependsOn": [
        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
    ],
    "properties": {
        "siteConfig": {
            "appSettings": [
                {
                    "name": "AzureWebJobsStorage",
                    "value": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountid'),'2019-06-01').keys[0].value)]"
                },
                {
                    "name": "WEBSITE_CONTENTAZUREFILECONNECTIONSTRING",
                    "value": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountid'),'2019-06-01').keys[0].value)]"
                },
                {
                    "name": "WEBSITE_CONTENTSHARE",
                    "value": "[toLower(variables('functionAppName'))]"
                },
                {
                    "name": "FUNCTIONS_WORKER_RUNTIME",
                    "value": "node"
                },
                {
                    "name": "WEBSITE_NODE_DEFAULT_VERSION",
                    "value": "10.14.1"
                },
                {
                    "name": "FUNCTIONS_EXTENSION_VERSION",
                    "value": "~2"
                }
            ]
        }
    }
}
```

#### <a name="linux"></a>Linux

Sous Linux, l’application de fonction doit avoir son paramètre `kind` défini sur `functionapp,linux`, et la propriété `reserved` définie sur `true` :

```json
{
    "apiVersion": "2016-03-01",
    "type": "Microsoft.Web/sites",
    "name": "[variables('functionAppName')]",
    "location": "[resourceGroup().location]",
    "kind": "functionapp,linux",
    "dependsOn": [
        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
    ],
    "properties": {
        "siteConfig": {
            "appSettings": [
                {
                    "name": "AzureWebJobsStorage",
                    "value": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountName'),'2019-06-01').keys[0].value)]"
                },
                {
                    "name": "FUNCTIONS_WORKER_RUNTIME",
                    "value": "node"
                },
                {
                    "name": "WEBSITE_NODE_DEFAULT_VERSION",
                    "value": "10.14.1"
                },
                {
                    "name": "FUNCTIONS_EXTENSION_VERSION",
                    "value": "~2"
                }
            ]
        },
        "reserved": true
    }
}
```



<a name="premium"></a>

## <a name="deploy-on-premium-plan"></a>Déployer sur un plan Premium

Le plan Premium offre la même mise à l’échelle que le plan Consommation, mais inclut des ressources dédiées et des fonctionnalités supplémentaires. Pour plus d’informations, consultez [Plan Premium Azure Functions](./functions-premium-plan.md).

### <a name="create-a-premium-plan"></a>Créer un plan Premium

Un plan Premium est un type spécial de ressource « serverfarm ». Vous pouvez le spécifier à l’aide de `EP1`, `EP2` ou `EP3` pour la valeur de propriété `Name` dans l’[objet de description](/azure/templates/microsoft.web/2018-02-01/serverfarms#skudescription-object) `sku`.

```json
{
    "type": "Microsoft.Web/serverfarms",
    "apiVersion": "2018-02-01",
    "name": "[parameters('hostingPlanName')]",
    "location": "[resourceGroup().location]",
    "properties": {
        "name": "[parameters('hostingPlanName')]",
        "workerSize": "[parameters('workerSize')]",
        "workerSizeId": "[parameters('workerSizeId')]",
        "numberOfWorkers": "[parameters('numberOfWorkers')]",
        "hostingEnvironment": "[parameters('hostingEnvironment')]",
        "maximumElasticWorkerCount": "20"
    },
    "sku": {
        "Tier": "ElasticPremium",
        "Name": "EP1"
    }
}
```

### <a name="create-a-function-app"></a>Créer une application de fonction

Une application de fonction sur un plan Premium doit avoir la propriété `serverFarmId` définie sur l’ID de ressource du plan créé précédemment. De plus, un plan Premium nécessite deux paramètres supplémentaires dans la configuration du site : `WEBSITE_CONTENTAZUREFILECONNECTIONSTRING` et `WEBSITE_CONTENTSHARE`. Ces propriétés configurent le compte de stockage et le chemin de fichier où le code de l’application de fonction et la configuration sont stockés.

```json
{
    "apiVersion": "2016-03-01",
    "type": "Microsoft.Web/sites",
    "name": "[variables('functionAppName')]",
    "location": "[resourceGroup().location]",
    "kind": "functionapp",            
    "dependsOn": [
        "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]",
        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
    ],
    "properties": {
        "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]",
        "siteConfig": {
            "appSettings": [
                {
                    "name": "AzureWebJobsStorage",
                    "value": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountid'),'2019-06-01').keys[0].value)]"
                },
                {
                    "name": "WEBSITE_CONTENTAZUREFILECONNECTIONSTRING",
                    "value": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountid'),'2019-06-01').keys[0].value)]"
                },
                {
                    "name": "WEBSITE_CONTENTSHARE",
                    "value": "[toLower(variables('functionAppName'))]"
                },
                {
                    "name": "FUNCTIONS_WORKER_RUNTIME",
                    "value": "node"
                },
                {
                    "name": "WEBSITE_NODE_DEFAULT_VERSION",
                    "value": "10.14.1"
                },
                {
                    "name": "FUNCTIONS_EXTENSION_VERSION",
                    "value": "~2"
                }
            ]
        }
    }
}
```


<a name="app-service-plan"></a>

## <a name="deploy-on-app-service-plan"></a>Déployer sur un plan App Service

Dans le plan App Service, vos applications de fonction sont exécutées sur des machines virtuelles dédiées sur des références de base, Standard et Premium, à l’instar des applications web. Pour plus d’informations sur le fonctionnement du plan App Service, consultez l’article [Présentation détaillée des plans d’Azure App Service](../app-service/overview-hosting-plans.md).

Pour un exemple de modèle Azure Resource Manager, consultez [Function app on Azure App Service plan] (Application de fonction dans le plan Azure App Service).

### <a name="create-an-app-service-plan"></a>Créer un plan App Service

Un plan App Service est défini par une ressource « serverfarm ».

```json
{
    "type": "Microsoft.Web/serverfarms",
    "apiVersion": "2018-02-01",
    "name": "[variables('hostingPlanName')]",
    "location": "[resourceGroup().location]",
    "sku": {
        "name": "S1",
        "tier": "Standard",
        "size": "S1",
        "family": "S",
        "capacity": 1
    }
}
```

Pour exécuter votre application sous Linux, vous devez également définir la propriété `kind` sur `Linux` :

```json
{
    "type": "Microsoft.Web/serverfarms",
    "apiVersion": "2018-02-01",
    "name": "[variables('hostingPlanName')]",
    "location": "[resourceGroup().location]",
    "kind": "Linux",
    "sku": {
        "name": "S1",
        "tier": "Standard",
        "size": "S1",
        "family": "S",
        "capacity": 1
    }
}
```

### <a name="create-a-function-app"></a>Créer une application de fonction

Une application de fonction sur un plan App Service doit avoir la propriété `serverFarmId` définie sur l’ID de ressource du plan créé précédemment.

```json
{
    "apiVersion": "2016-03-01",
    "type": "Microsoft.Web/sites",
    "name": "[variables('functionAppName')]",
    "location": "[resourceGroup().location]",
    "kind": "functionapp",
    "dependsOn": [
        "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]",
        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
    ],
    "properties": {
        "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]",
        "siteConfig": {
            "appSettings": [
                {
                    "name": "AzureWebJobsStorage",
                    "value": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountid'),'2019-06-01').keys[0].value)]"
                },
                {
                    "name": "FUNCTIONS_WORKER_RUNTIME",
                    "value": "node"
                },
                {
                    "name": "WEBSITE_NODE_DEFAULT_VERSION",
                    "value": "10.14.1"
                },
                {
                    "name": "FUNCTIONS_EXTENSION_VERSION",
                    "value": "~2"
                }
            ]
        }
    }
}
```

Les applications Linux doivent également inclure une propriété `linuxFxVersion` sous `siteConfig`. Si vous déployez simplement du code, cette valeur est déterminée par votre pile de runtime souhaitée :

| Pile            | Valeur d'exemple                                         |
|------------------|-------------------------------------------------------|
| Python           | `DOCKER|microsoft/azure-functions-python3.6:2.0`      |
| JavaScript       | `DOCKER|microsoft/azure-functions-node8:2.0`          |
| .NET             | `DOCKER|microsoft/azure-functions-dotnet-core2.0:2.0` |

```json
{
    "apiVersion": "2016-03-01",
    "type": "Microsoft.Web/sites",
    "name": "[variables('functionAppName')]",
    "location": "[resourceGroup().location]",
    "kind": "functionapp",
    "dependsOn": [
        "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]",
        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
    ],
    "properties": {
        "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]",
        "siteConfig": {
            "appSettings": [
                {
                    "name": "AzureWebJobsStorage",
                    "value": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountid'),'2019-06-01').keys[0].value)]"
                },
                {
                    "name": "FUNCTIONS_WORKER_RUNTIME",
                    "value": "node"
                },
                {
                    "name": "WEBSITE_NODE_DEFAULT_VERSION",
                    "value": "10.14.1"
                },
                {
                    "name": "FUNCTIONS_EXTENSION_VERSION",
                    "value": "~2"
                }
            ],
            "linuxFxVersion": "DOCKER|microsoft/azure-functions-node8:2.0"
        }
    }
}
```

Si vous [déployez une image de conteneur personnalisé](./functions-create-function-linux-custom-image.md), vous devez la spécifier avec `linuxFxVersion` et inclure une configuration permettant l’extraction de votre image, comme dans [Web App pour conteneurs](../app-service/index.yml). Par ailleurs, définissez `WEBSITES_ENABLE_APP_SERVICE_STORAGE` sur `false`, car le contenu de votre application est fourni dans le conteneur lui-même :

```json
{
    "apiVersion": "2016-03-01",
    "type": "Microsoft.Web/sites",
    "name": "[variables('functionAppName')]",
    "location": "[resourceGroup().location]",
    "kind": "functionapp",
    "dependsOn": [
        "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]",
        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
    ],
    "properties": {
        "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]",
        "siteConfig": {
            "appSettings": [
                {
                    "name": "AzureWebJobsStorage",
                    "value": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountid'),'2019-06-01').keys[0].value)]"
                },
                {
                    "name": "FUNCTIONS_WORKER_RUNTIME",
                    "value": "node"
                },
                {
                    "name": "WEBSITE_NODE_DEFAULT_VERSION",
                    "value": "10.14.1"
                },
                {
                    "name": "FUNCTIONS_EXTENSION_VERSION",
                    "value": "~2"
                },
                {
                    "name": "DOCKER_REGISTRY_SERVER_URL",
                    "value": "[parameters('dockerRegistryUrl')]"
                },
                {
                    "name": "DOCKER_REGISTRY_SERVER_USERNAME",
                    "value": "[parameters('dockerRegistryUsername')]"
                },
                {
                    "name": "DOCKER_REGISTRY_SERVER_PASSWORD",
                    "value": "[parameters('dockerRegistryPassword')]"
                },
                {
                    "name": "WEBSITES_ENABLE_APP_SERVICE_STORAGE",
                    "value": "false"
                }
            ],
            "linuxFxVersion": "DOCKER|myacr.azurecr.io/myimage:mytag"
        }
    }
}
```

## <a name="customizing-a-deployment"></a>Personnalisation d’un déploiement

Une application de fonction dispose de nombreuses ressources enfant que vous pouvez utiliser dans votre développement, notamment les paramètres de l’application et les options de contrôle de code source. Vous pouvez également choisir de supprimer la ressource enfant **sourcecontrols** et utiliser une autre [option de déploiement](functions-continuous-deployment.md) à la place.

> [!IMPORTANT]
> Il est important de comprendre comment les ressources sont déployées dans Azure pour déployer correctement votre application en utilisant Azure Resource Manager. Dans l’exemple suivant, les configurations de niveau supérieur sont appliquées à l’aide de **siteConfig**. Il est important de définir ces configurations à un niveau supérieur, car elles fournissent des informations au moteur de déploiement et au runtime Functions. Des informations de niveau supérieur sont requises avant que la ressource enfant **sourcecontrols/web** soit appliquée. Bien qu’il soit possible de configurer ces paramètres dans la ressource **configuration/appSettings** au niveau enfant, dans certains cas, votre application de fonction doit être déployée *avant* que la ressource **configuration/appSettings** soit appliquée. Par exemple, quand vous utilisez des fonctions avec [Logic Apps](../logic-apps/index.yml), vos fonctions sont une dépendance d’une autre ressource.

```json
{
  "apiVersion": "2015-08-01",
  "name": "[parameters('appName')]",
  "type": "Microsoft.Web/sites",
  "kind": "functionapp",
  "location": "[parameters('location')]",
  "dependsOn": [
    "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]",
    "[resourceId('Microsoft.Web/serverfarms', parameters('appName'))]"
  ],
  "properties": {
     "serverFarmId": "[variables('appServicePlanName')]",
     "siteConfig": {
        "alwaysOn": true,
        "appSettings": [
            {
                "name": "FUNCTIONS_EXTENSION_VERSION",
                "value": "~2"
            },
            {
                "name": "Project",
                "value": "src"
            }
        ]
     }
  },
  "resources": [
     {
        "apiVersion": "2015-08-01",
        "name": "appsettings",
        "type": "config",
        "dependsOn": [
          "[resourceId('Microsoft.Web/Sites', parameters('appName'))]",
          "[resourceId('Microsoft.Web/Sites/sourcecontrols', parameters('appName'), 'web')]",
          "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
        ],
        "properties": {
          "AzureWebJobsStorage": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountid'),'2019-06-01').keys[0].value)]",
          "AzureWebJobsDashboard": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountid'),'2019-06-01').keys[0].value)]",
          "FUNCTIONS_EXTENSION_VERSION": "~2",
          "FUNCTIONS_WORKER_RUNTIME": "dotnet",
          "Project": "src"
        }
     },
     {
          "apiVersion": "2015-08-01",
          "name": "web",
          "type": "sourcecontrols",
          "dependsOn": [
            "[resourceId('Microsoft.Web/sites/', parameters('appName'))]"
          ],
          "properties": {
            "RepoUrl": "[parameters('sourceCodeRepositoryURL')]",
            "branch": "[parameters('sourceCodeBranch')]",
            "IsManualIntegration": "[parameters('sourceCodeManualIntegration')]"
          }
     }
  ]
}
```
> [!TIP]
> Dans ce modèle, la valeur du paramètre d’application [Project](https://github.com/projectkudu/kudu/wiki/Customizing-deployments#using-app-settings-instead-of-a-deployment-file), qui définit le répertoire de base dans lequel le moteur de déploiement de fonctions (Kudu) recherche le code pouvant être déployé, est utilisée. Dans notre référentiel, nos fonctions se trouvent dans un sous-dossier du dossier **src**. Par conséquent, dans l’exemple précédent, nous définissons la valeur de paramètres d’application sur `src`. Si vos fonctions sont à la racine de votre référentiel, ou si vous ne déployez pas de contrôle de code source, vous pouvez supprimer cette valeur de paramètres d’application.

## <a name="deploy-your-template"></a>Déployer votre modèle

Vous pouvez utiliser une des méthodes suivantes pour déployer votre modèle :

* [PowerShell](../azure-resource-manager/templates/deploy-powershell.md)
* [Azure CLI](../azure-resource-manager/templates/deploy-cli.md)
* [Azure portal](../azure-resource-manager/templates/deploy-portal.md)
* [REST API](../azure-resource-manager/templates/deploy-rest.md)

### <a name="deploy-to-azure-button"></a>Bouton Déployer dans Azure

Remplacez ```<url-encoded-path-to-azuredeploy-json>``` par une version [d’URL encodée](https://www.bing.com/search?q=url+encode) du chemin brut de votre fichier `azuredeploy.json` dans GitHub.

Voici un exemple qui utilise markdown :

```markdown
[![Deploy to Azure](https://azuredeploy.net/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/<url-encoded-path-to-azuredeploy-json>)
```

Voici un exemple qui utilise HTML :

```html
<a href="https://portal.azure.com/#create/Microsoft.Template/uri/<url-encoded-path-to-azuredeploy-json>" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"></a>
```

### <a name="deploy-using-powershell"></a>Déployer à l’aide de PowerShell

Les commandes PowerShell suivantes créent un groupe de ressources et déploient un modèle qui crée une application de fonction avec ses ressources requises. Pour une exécution locale, [Azure PowerShell](/powershell/azure/install-az-ps) doit être installé. Exécutez [`Connect-AzAccount`](/powershell/module/az.accounts/connect-azaccount) pour vous connecter.

```powershell
# Register Resource Providers if they're not already registered
Register-AzResourceProvider -ProviderNamespace "microsoft.web"
Register-AzResourceProvider -ProviderNamespace "microsoft.storage"

# Create a resource group for the function app
New-AzResourceGroup -Name "MyResourceGroup" -Location 'West Europe'

# Create the parameters for the file, which for this template is the function app name.
$TemplateParams = @{"appName" = "<function-app-name>"}

# Deploy the template
New-AzResourceGroupDeployment -ResourceGroupName "MyResourceGroup" -TemplateFile template.json -TemplateParameterObject $TemplateParams -Verbose
```

Pour tester ce déploiement, vous pouvez utiliser un [modèle comme celui-ci](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-function-app-create-dynamic/azuredeploy.json), qui crée une application de fonction sur Windows dans un plan Consommation. Remplacez `<function-app-name>` par un nom unique pour votre application de fonction.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur le développement et la configuration d’Azure Functions.

* [Informations de référence pour les développeurs sur Azure Functions](functions-reference.md)
* [Guide pratique pour configurer les paramètres d’application de fonction Azure](functions-how-to-use-azure-function-app-settings.md)
* [Créer votre première fonction Azure](functions-create-first-azure-function.md)

<!-- LINKS -->

[Function app on Consumption plan]: https://github.com/Azure/azure-quickstart-templates/blob/master/101-function-app-create-dynamic/azuredeploy.json (Application de fonction dans le plan Consommation)
[Function app on Azure App Service plan]: https://github.com/Azure/azure-quickstart-templates/blob/master/101-function-app-create-dedicated/azuredeploy.json (Application de fonction dans le plan Azure App Service)