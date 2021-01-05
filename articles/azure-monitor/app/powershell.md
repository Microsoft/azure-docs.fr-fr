---
title: Automatiser Azure Application Insights avec PowerShell | Microsoft Docs
description: Automatisez la création et la gestion des ressources, des alertes et des tests de disponibilité dans PowerShell à l’aide d’un modèle Azure Resource Manager.
ms.topic: conceptual
ms.date: 05/02/2020
ms.openlocfilehash: c380880845c162a1e8fb38e699a439ac04fb3b5a
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96186267"
---
#  <a name="manage-application-insights-resources-using-powershell"></a>Gérer les ressources Application Insights à l’aide de PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Cet article explique comment automatiser la création et la mise à jour de ressources [Application Insights](./app-insights-overview.md) à l’aide du service de gestion des ressources Azure. Cette opération peut par exemple avoir lieu dans le cadre du processus de génération. Avec la ressource Application Insights de base, vous pouvez créer des [tests web de disponibilité](./monitor-web-app-availability.md), configurer [des alertes](../platform/alerts-log.md) et un [mécanisme de tarification](pricing.md), mais aussi créer d’autres ressources Azure.

Les éléments importants pour la création de ces ressources sont les modèles JSON pour [Azure Resource Manager](../../azure-resource-manager/management/manage-resources-powershell.md). La procédure de base est la suivante : téléchargez les définitions JSON des ressources existantes, paramétrez certaines valeurs telles que les noms, puis exécutez le modèle chaque fois que vous souhaitez créer une ressource. Vous pouvez regrouper plusieurs ressources pour les créer en une fois, par exemple une analyse d’application avec des tests de disponibilité, des alertes et le stockage pour l’exportation continue. Certains paramètres ont des particularités que nous aborderons ici.

## <a name="one-time-setup"></a>Installation unique
Si vous n’avez pas utilisé précédemment PowerShell avec votre abonnement Azure :

Installez le module Azure PowerShell sur la machine dans laquelle vous souhaitez exécuter les scripts :

1. Installez le programme [Microsoft Web Platform Installer (v5 ou version ultérieure)](https://www.microsoft.com/web/downloads/platform.aspx).
2. Utilisez-le pour installer Microsoft Azure PowerShell.

En plus des modèles Resource Manager, il existe un ensemble complet d’[applets de commande PowerShell Application Insights](/powershell/module/az.applicationinsights), qui facilite la configuration des ressources Application Insights par programmation. Les fonctionnalités activées par les applets de commande sont les suivantes :

* Créer et supprimer des ressources Application Insights
* Obtenir les listes de ressources Application Insights et leurs propriétés
* Créer et gérer une exportation continue
* Créer et gérer des clés d’application
* Définir la limite quotidienne
* Définir le plan tarifaire

## <a name="create-application-insights-resources-using-a-powershell-cmdlet"></a>Créer des ressources Application Insights à l’aide d’une applet de commande PowerShell

Voici comment créer une ressource Application Insights dans le centre de données Azure USA Est à l’aide de l’applet de commande [New-AzApplicationInsights](/powershell/module/az.applicationinsights/new-azapplicationinsights) :

```PS
New-AzApplicationInsights -ResourceGroupName <resource group> -Name <resource name> -location eastus
```


## <a name="create-application-insights-resources-using-a-resource-manager-template"></a>Créer des ressources Application Insights à l’aide d’un modèle Resource Manager

Voici comment créer une ressource Application Insights à l’aide d’un modèle Resource Manager.

### <a name="create-the-azure-resource-manager-template"></a>Créer le modèle Azure Resource Manager

Créez un fichier .json appelé `template1.json` dans cet exemple. Copiez-y ce contenu :

```JSON
    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "appName": {
                "type": "string",
                "metadata": {
                    "description": "Enter the name of your Application Insights resource."
                }
            },
            "appType": {
                "type": "string",
                "defaultValue": "web",
                "allowedValues": [
                    "web",
                    "java",
                    "other"
                ],
                "metadata": {
                    "description": "Enter the type of the monitored application."
                }
            },
            "appLocation": {
                "type": "string",
                "defaultValue": "eastus",
                "metadata": {
                    "description": "Enter the location of your Application Insights resource."
                }
            },
            "retentionInDays": {
                "type": "int",
                "defaultValue": 90,
                "allowedValues": [
                    30,
                    60,
                    90,
                    120,
                    180,
                    270,
                    365,
                    550,
                    730
                ],
                "metadata": {
                    "description": "Data retention in days"
                }
            },
            "ImmediatePurgeDataOn30Days": {
                "type": "bool",
                "defaultValue": false,
                "metadata": {
                    "description": "If set to true when changing retention to 30 days, older data will be immediately deleted. Use this with extreme caution. This only applies when retention is being set to 30 days."
                }
            },
            "priceCode": {
                "type": "int",
                "defaultValue": 1,
                "allowedValues": [
                    1,
                    2
                ],
                "metadata": {
                    "description": "Pricing plan: 1 = Per GB (or legacy Basic plan), 2 = Per Node (legacy Enterprise plan)"
                }
            },
            "dailyQuota": {
                "type": "int",
                "defaultValue": 100,
                "minValue": 1,
                "metadata": {
                    "description": "Enter daily quota in GB."
                }
            },
            "dailyQuotaResetTime": {
                "type": "int",
                "defaultValue": 0,
                "metadata": {
                    "description": "Enter daily quota reset hour in UTC (0 to 23). Values outside the range will get a random reset hour."
                }
            },
            "warningThreshold": {
                "type": "int",
                "defaultValue": 90,
                "minValue": 1,
                "maxValue": 100,
                "metadata": {
                    "description": "Enter the % value of daily quota after which warning mail to be sent. "
                }
            }
        },
        "variables": {
            "priceArray": [
                "Basic",
                "Application Insights Enterprise"
            ],
            "pricePlan": "[take(variables('priceArray'),parameters('priceCode'))]",
            "billingplan": "[concat(parameters('appName'),'/', variables('pricePlan')[0])]"
        },
        "resources": [
            {
                "type": "microsoft.insights/components",
                "kind": "[parameters('appType')]",
                "name": "[parameters('appName')]",
                "apiVersion": "2014-04-01",
                "location": "[parameters('appLocation')]",
                "tags": {},
                "properties": {
                    "ApplicationId": "[parameters('appName')]",
                    "retentionInDays": "[parameters('retentionInDays')]"
                },
                "dependsOn": []
            },
            {
                "name": "[variables('billingplan')]",
                "type": "microsoft.insights/components/CurrentBillingFeatures",
                "location": "[parameters('appLocation')]",
                "apiVersion": "2015-05-01",
                "dependsOn": [
                    "[resourceId('microsoft.insights/components', parameters('appName'))]"
                ],
                "properties": {
                    "CurrentBillingFeatures": "[variables('pricePlan')]",
                    "DataVolumeCap": {
                        "Cap": "[parameters('dailyQuota')]",
                        "WarningThreshold": "[parameters('warningThreshold')]",
                        "ResetTime": "[parameters('dailyQuotaResetTime')]"
                    }
                }
            }
        ]
    }
```

### <a name="use-the-resource-manager-template-to-create-a-new-application-insights-resource"></a>Utiliser le modèle Resource Manager pour créer une ressource Application Insights

1. Dans PowerShell, connectez-vous à Azure via `$Connect-AzAccount`
2. Affectez un abonnement à votre contexte avec `Set-AzContext "<subscription ID>"`
2. Exécutez un nouveau déploiement pour créer une ressource Application Insights :
   
    ```PS
        New-AzResourceGroupDeployment -ResourceGroupName Fabrikam `
               -TemplateFile .\template1.json `
               -appName myNewApp

    ``` 
   
   * `-ResourceGroupName` est le groupe dans lequel vous souhaitez créer les nouvelles ressources.
   * `-TemplateFile` doit se produire avant les paramètres personnalisés.
   * `-appName` est le nom de la ressource à créer.

Vous pouvez ajouter d’autres paramètres. Vous trouverez leurs descriptions dans la section Paramètres du modèle.

## <a name="get-the-instrumentation-key"></a>Obtenir la clé d’instrumentation

Après avoir créé une ressource d’application, vous voulez obtenir la clé d’instrumentation : 

1. `$Connect-AzAccount`
2. `Set-AzContext "<subscription ID>"`
3. `$resource = Get-AzResource -Name "<resource name>" -ResourceType "Microsoft.Insights/components"`
4. `$details = Get-AzResource -ResourceId $resource.ResourceId`
5. `$details.Properties.InstrumentationKey`

Pour voir la liste des nombreuses autres propriétés de votre ressource Application Insights, utilisez :

```PS
Get-AzApplicationInsights -ResourceGroupName Fabrikam -Name FabrikamProd | Format-List
```

Des propriétés supplémentaires sont disponibles via les applets de commande :
* `Set-AzApplicationInsightsDailyCap`
* `Set-AzApplicationInsightsPricingPlan`
* `Get-AzApplicationInsightsApiKey`
* `Get-AzApplicationInsightsContinuousExport`

Consultez la [documentation détaillée](/powershell/module/az.applicationinsights) pour connaître les paramètres de ces applets de commande.  

## <a name="set-the-data-retention"></a>Définir la conservation des données

Voici trois méthodes pour définir programmatiquement la conservation des données sur une ressource Application Insights.

### <a name="setting-data-retention-using-a-powershell-commands"></a>Définition de la conservation des données au moyen de commandes PowerShell

Voici un ensemble simple de commandes PowerShell permettant de définir la conservation des données pour votre ressource Application Insights :

```PS
$Resource = Get-AzResource -ResourceType Microsoft.Insights/components -ResourceGroupName MyResourceGroupName -ResourceName MyResourceName
$Resource.Properties.RetentionInDays = 365
$Resource | Set-AzResource -Force
```

### <a name="setting-data-retention-using-rest"></a>Définition de la conservation des données à l’aide de REST

Pour obtenir la conservation des données actuelle de votre ressource Application Insights, vous pouvez utiliser l’outil OSS [ARMClient](https://github.com/projectkudu/ARMClient).  (Découvrez plus en détail ARMClient dans les articles de [David Ebbo](http://blog.davidebbo.com/2015/01/azure-resource-manager-client.html) et [Daniel Bowbyes](https://blog.bowbyes.co.nz/2016/11/02/using-armclient-to-directly-access-azure-arm-rest-apis-and-list-arm-policy-details/).)  Voici un exemple d’utilisation de `ARMClient` pour obtenir la conservation actuelle :

```PS
armclient GET /subscriptions/00000000-0000-0000-0000-00000000000/resourceGroups/MyResourceGroupName/providers/microsoft.insights/components/MyResourceName?api-version=2018-05-01-preview
```

Pour définir la conservation, la commande est un PUT similaire :

```PS
armclient PUT /subscriptions/00000000-0000-0000-0000-00000000000/resourceGroups/MyResourceGroupName/providers/microsoft.insights/components/MyResourceName?api-version=2018-05-01-preview "{location: 'eastus', properties: {'retentionInDays': 365}}"
```

Pour définir la conservation des données sur 365 jours à l’aide du modèle ci-dessus, exécutez :

```PS
New-AzResourceGroupDeployment -ResourceGroupName "<resource group>" `
       -TemplateFile .\template1.json `
       -retentionInDays 365 `
       -appName myApp
```

### <a name="setting-data-retention-using-a-powershell-script"></a>Définition de la conservation des données au moyen d’un script PowerShell

Le script suivant peut également être utilisé pour modifier la rétention. Copiez ce script pour l’enregistrer en tant que `Set-ApplicationInsightsRetention.ps1`.

```PS
Param(
    [Parameter(Mandatory = $True)]
    [string]$SubscriptionId,

    [Parameter(Mandatory = $True)]
    [string]$ResourceGroupName,

    [Parameter(Mandatory = $True)]
    [string]$Name,

    [Parameter(Mandatory = $True)]
    [string]$RetentionInDays
)
$ErrorActionPreference = 'Stop'
if (-not (Get-Module Az.Accounts)) {
    Import-Module Az.Accounts
}
$azProfile = [Microsoft.Azure.Commands.Common.Authentication.Abstractions.AzureRmProfileProvider]::Instance.Profile
if (-not $azProfile.Accounts.Count) {
    Write-Error "Ensure you have logged in before calling this function."    
}
$currentAzureContext = Get-AzContext
$profileClient = New-Object Microsoft.Azure.Commands.ResourceManager.Common.RMProfileClient($azProfile)
$token = $profileClient.AcquireAccessToken($currentAzureContext.Tenant.TenantId)
$UserToken = $token.AccessToken
$RequestUri = "https://management.azure.com/subscriptions/$($SubscriptionId)/resourceGroups/$($ResourceGroupName)/providers/Microsoft.Insights/components/$($Name)?api-version=2015-05-01"
$Headers = @{
    "Authorization"         = "Bearer $UserToken"
    "x-ms-client-tenant-id" = $currentAzureContext.Tenant.TenantId
}
## Get Component object via ARM
$GetResponse = Invoke-RestMethod -Method "GET" -Uri $RequestUri -Headers $Headers 

## Update RetentionInDays property
if($($GetResponse.properties | Get-Member "RetentionInDays"))
{
    $GetResponse.properties.RetentionInDays = $RetentionInDays
}
else
{
    $GetResponse.properties | Add-Member -Type NoteProperty -Name "RetentionInDays" -Value $RetentionInDays
}
## Upsert Component object via ARM
$PutResponse = Invoke-RestMethod -Method "PUT" -Uri "$($RequestUri)" -Headers $Headers -Body $($GetResponse | ConvertTo-Json) -ContentType "application/json"
$PutResponse
```

Ce script peut être alors utilisé comme suit  :

```PS
Set-ApplicationInsightsRetention `
        [-SubscriptionId] <String> `
        [-ResourceGroupName] <String> `
        [-Name] <String> `
        [-RetentionInDays <Int>]
```

## <a name="set-the-daily-cap"></a>Définir le plafond quotidien

Pour accéder aux propriétés du plafond quotidien, utilisez l’applet de commande [Set-AzApplicationInsightsPricingPlan](/powershell/module/az.applicationinsights/set-azapplicationinsightspricingplan) : 

```PS
Set-AzApplicationInsightsDailyCap -ResourceGroupName <resource group> -Name <resource name> | Format-List
```

Pour définir les propriétés du plafond quotidien, utilisez la même applet de commande. Par exemple, pour définir le plafond à 300 Go/jour,

```PS
Set-AzApplicationInsightsDailyCap -ResourceGroupName <resource group> -Name <resource name> -DailyCapGB 300
```

Vous pouvez également utiliser [ARMClient](https://github.com/projectkudu/ARMClient) pour récupérer et définir des paramètres de limite quotidienne.  Pour obtenir les valeurs actuelles, utilisez la commande suivante :

```PS
armclient GET /subscriptions/00000000-0000-0000-0000-00000000000/resourceGroups/MyResourceGroupName/providers/microsoft.insights/components/MyResourceName/CurrentBillingFeatures?api-version=2018-05-01-preview
```

## <a name="set-the-daily-cap-reset-time"></a>Définir l’heure de réinitialisation de la limite quotidienne

Pour définir l’heure de réinitialisation de la limite quotidienne, vous pouvez utiliser [ARMClient](https://github.com/projectkudu/ARMClient). Voici un exemple d’utilisation de `ARMClient` pour définir l’heure de réinitialisation sur une nouvelle heure (dans cet exemple, 12:00 UTC) :

```PS
armclient PUT /subscriptions/00000000-0000-0000-0000-00000000000/resourceGroups/MyResourceGroupName/providers/microsoft.insights/components/MyResourceName/CurrentBillingFeatures?api-version=2018-05-01-preview "{'CurrentBillingFeatures':['Basic'],'DataVolumeCap':{'ResetTime':12}}"
```

<a id="price"></a>
## <a name="set-the-pricing-plan"></a>Définir le plan tarifaire 

Pour obtenir le plan tarifaire actuel, utilisez l’applet de commande [Set-AzApplicationInsightsPricingPlan](/powershell/module/az.applicationinsights/set-azapplicationinsightspricingplan) :

```PS
Set-AzApplicationInsightsPricingPlan -ResourceGroupName <resource group> -Name <resource name> | Format-List
```

Pour définir le plan tarifaire, utilisez la même applet de commande avec le `-PricingPlan` spécifié :  

```PS
Set-AzApplicationInsightsPricingPlan -ResourceGroupName <resource group> -Name <resource name> -PricingPlan Basic
```

Vous pouvez également définir le plan tarifaire en fonction d’une ressource Application Insights existante à l’aide du modèle Resource Manager ci-dessus, en omettant la ressource « microsoft.insights/components » et le nœud `dependsOn` dans la ressource de facturation. Par exemple, pour définir le plan tarifaire en fonction du plan Par Go (anciennement appelé Plan de base), exécutez :

```PS
        New-AzResourceGroupDeployment -ResourceGroupName "<resource group>" `
               -TemplateFile .\template1.json `
               -priceCode 1 `
               -appName myApp
```

Le `priceCode` est défini comme :

|priceCode|Plan|
|---|---|
|1|Par Go (anciennement appelé Plan de base)|
|2|Par nœud (anciennement appelé Plan Entreprise)|

Enfin, vous pouvez utiliser [ARMClient](https://github.com/projectkudu/ARMClient) pour récupérer et définir des plans de tarification et des paramètres de limite quotidienne.  Pour obtenir les valeurs actuelles, utilisez la commande suivante :

```PS
armclient GET /subscriptions/00000000-0000-0000-0000-00000000000/resourceGroups/MyResourceGroupName/providers/microsoft.insights/components/MyResourceName/CurrentBillingFeatures?api-version=2018-05-01-preview
```

Vous pouvez définir tous ces paramètres à l’aide de la commande suivante :

```PS
armclient PUT /subscriptions/00000000-0000-0000-0000-00000000000/resourceGroups/MyResourceGroupName/providers/microsoft.insights/components/MyResourceName/CurrentBillingFeatures?api-version=2018-05-01-preview
"{'CurrentBillingFeatures':['Basic'],'DataVolumeCap':{'Cap':200,'ResetTime':12,'StopSendNotificationWhenHitCap':true,'WarningThreshold':90,'StopSendNotificationWhenHitThreshold':true}}"
```

Cela définit la limite quotidienne sur 200 Go/jour, configure l’heure de réinitialisation de la limite quotidienne sur 12:00 UTC, envoie des e-mails lorsque la limite et le niveau d’avertissement sont atteints, et définit le seuil d’avertissement sur 90 % de la limite.  

## <a name="add-a-metric-alert"></a>Ajouter une alerte métrique

Pour automatiser la création d’alertes de métrique, consultez l’[article sur le modèle d’alertes de métrique](../platform/alerts-metric-create-templates.md#template-for-a-simple-static-threshold-metric-alert)


## <a name="add-an-availability-test"></a>Ajouter un test de disponibilité

Pour automatiser les tests de disponibilité, consultez l’[article sur le modèle d’alertes de métrique](../platform/alerts-metric-create-templates.md#template-for-an-availability-test-along-with-a-metric-alert).

## <a name="add-more-resources"></a>Ajouter des ressources

Pour automatiser la création de toute autre ressource de tout type, créez un exemple manuellement, puis copiez-le et paramétrez son code à partir [d’Azure Resource Manager](https://resources.azure.com/). 

1. Ouvrez [Azure Resource Manager](https://resources.azure.com/). Accédez à votre ressource d’application qui se trouve après `subscriptions/resourceGroups/<your resource group>/providers/Microsoft.Insights/components`. 
   
    ![Navigation dans Azure Resource Explorer](./media/powershell/01.png)
   
    *composants* sont les ressources Application Insights de base pour l’affichage des applications. Il existe des ressources distinctes pour les règles d’alertes et les tests web de disponibilité associés.
2. Copiez le JSON du composant dans l’emplacement approprié dans `template1.json`.
3. Supprimez les propriétés suivantes :
   
   * `id`
   * `InstrumentationKey`
   * `CreationDate`
   * `TenantId`
4. Ouvrez les sections `webtests` et `alertrules`, puis copiez le contenu JSON des éléments individuels dans votre modèle. (N’effectuez pas de copie à partir des nœuds `webtests` ou `alertrules`, accédez aux éléments situés juste en dessous.)
   
    Chaque test web étant associé à une règle d’alerte, vous devez copier les deux.
   
5. Insérez cette ligne dans chaque ressource :
   
    `"apiVersion": "2015-05-01",`

### <a name="parameterize-the-template"></a>Définir les paramètres du modèle
Vous devez à présent remplacer les noms spécifiques par des paramètres. Pour [définir les paramètres d’un modèle](../../azure-resource-manager/templates/template-syntax.md), écrivez des expressions utilisant un [ensemble de fonctions d’assistance](../../azure-resource-manager/templates/template-functions.md). 

Comme vous ne pouvez pas définir uniquement les paramètres d’une portion de chaîne, utilisez `concat()` pour générer les chaînes.

Voici des exemples de substitutions que vous allez effectuer. Il existe plusieurs occurrences de chaque substitution. Il vous en faudra peut-être d’autres dans votre modèle. Ces exemples utilisent les paramètres et les variables que nous avons définis en haut du modèle.

| trouver | replace with |
| --- | --- |
| `"hidden-link:/subscriptions/.../../components/MyAppName"` |`"[concat('hidden-link:',`<br/>`resourceId('microsoft.insights/components',` <br/> `parameters('appName')))]"` |
| `"/subscriptions/.../../alertrules/myAlertName-myAppName-subsId",` |`"[resourceId('Microsoft.Insights/alertrules', variables('alertRuleName'))]",` |
| `"/subscriptions/.../../webtests/myTestName-myAppName",` |`"[resourceId('Microsoft.Insights/webtests', parameters('webTestName'))]",` |
| `"myWebTest-myAppName"` |`"[variables(testName)]"'` |
| `"myTestName-myAppName-subsId"` |`"[variables('alertRuleName')]"` |
| `"myAppName"` |`"[parameters('appName')]"` |
| `"myappname"` (minuscules) |`"[toLower(parameters('appName'))]"` |
| `"<WebTest Name=\"myWebTest\" ...`<br/>`Url=\"http://fabrikam.com/home\" ...>"` |`[concat('<WebTest Name=\"',` <br/> `parameters('webTestName'),` <br/> `'\" ... Url=\"', parameters('Url'),` <br/> `'\"...>')]"`|

### <a name="set-dependencies-between-the-resources"></a>Définir les dépendances entre les ressources
Azure doit configurer les ressources dans un ordre strict. Pour vous assurer de l’achèvement d’une installation avant que la suivante ne commence, ajoutez les lignes de dépendance :

* Dans la ressource du test de disponibilité :
  
    `"dependsOn": ["[resourceId('Microsoft.Insights/components', parameters('appName'))]"],`
* Dans la ressource d’alerte pour un test de disponibilité :
  
    `"dependsOn": ["[resourceId('Microsoft.Insights/webtests', variables('testName'))]"],`



## <a name="next-steps"></a>Étapes suivantes
Autres articles sur l’automation :

* [Création d'une ressource Application Insights](./create-new-resource.md#creating-a-resource-automatically) - méthode rapide sans utiliser de modèle.
* [Créer des tests web](../samples/resource-manager-alerts-metric.md#availability-test-with-metric-alert)
* [Envoyer des diagnostics Azure vers Application Insights.](powershell-azure-diagnostics.md)
* [Créer des annotations de version](https://github.com/MohanGsk/ApplicationInsights-Home/blob/master/API/CreateReleaseAnnotation.ps1)