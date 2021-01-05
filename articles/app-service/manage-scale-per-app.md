---
title: Mise à l’échelle par application pour un hébergement haute densité
description: Mettez à l’échelle les applications indépendamment des plans App Service et optimisez les instances ayant fait l’objet d’un scale-out dans votre plan.
author: btardif
ms.assetid: a903cb78-4927-47b0-8427-56412c4e3e64
ms.topic: article
ms.date: 05/13/2019
ms.author: byvinyal
ms.custom: seodec18
ms.openlocfilehash: f1ca4958fe2608d0c040ef5b93827a7e71a4151c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "74672346"
---
# <a name="high-density-hosting-on-azure-app-service-using-per-app-scaling"></a>Hébergement haute densité sur Azure App Service à l’aide de la mise à l’échelle par application

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Lorsque vous utilisez App Service, vous pouvez mettre à l’échelle vos applications en mettant à l’échelle le [plan App Service](overview-hosting-plans.md) sur lequel elles sont exécutées. Lorsque plusieurs applications sont exécutées dans un même plan App Service, chaque instance mise à l’échelle exécute toutes les applications du plan.

La *mise à l’échelle par application* peut être activée au niveau du plan App Service pour permettre la mise à l’échelle d’une application en ignorant le plan App Service qui l’héberge. De cette manière, un plan App Service peut passer à 10 instances, mais une application peut être définie de manière à n’en utiliser que cinq.

> [!NOTE]
> La mise à l’échelle par application est uniquement disponible pour les niveaux tarifaires **Standard**, **Premium**, **Premium V2** et **Isolé**.
>

Les applications sont allouées au plan App Service disponible à l’aide d’une approche essayant au mieux de répartir de manière uniforme sur toutes les instances. La répartition ne sera pas obligatoirement uniforme, mais la plateforme s’assurera que les deux instances de la même application ne seront pas hébergées sur la même instance de plan App Service.

La plateforme ne répartit pas les rôles de travail en fonction des mesures. Les applications sont rééquilibrées uniquement lorsque des instances sont ajoutées ou supprimées du plan App Service.

## <a name="per-app-scaling-using-powershell"></a>Mise à l’échelle par application à l’aide de PowerShell

Créez un plan avec mise à l’échelle par application en passant le paramètre ```-PerSiteScaling $true``` dans l’applet de commande ```New-AzAppServicePlan```.

```powershell
New-AzAppServicePlan -ResourceGroupName $ResourceGroup -Name $AppServicePlan `
                            -Location $Location `
                            -Tier Premium -WorkerSize Small `
                            -NumberofWorkers 5 -PerSiteScaling $true
```

Activez la mise à l’échelle par application avec un plan App Service existant en passant le paramètre `-PerSiteScaling $true` dans l’applet de commande ```Set-AzAppServicePlan```.

```powershell
# Enable per-app scaling for the App Service Plan using the "PerSiteScaling" parameter.
Set-AzAppServicePlan -ResourceGroupName $ResourceGroup `
   -Name $AppServicePlan -PerSiteScaling $true
```

Pour chaque application, configurez le nombre d’instances que celle-ci peut utiliser dans le plan App Service.

Dans l’exemple ci-dessous, l’application est limitée à deux instances quel que soit le nombre d’instances mis à l’échelle pour le plan App Service sous-jacent.

```powershell
# Get the app we want to configure to use "PerSiteScaling"
$newapp = Get-AzWebApp -ResourceGroupName $ResourceGroup -Name $webapp

# Modify the NumberOfWorkers setting to the desired value.
$newapp.SiteConfig.NumberOfWorkers = 2

# Post updated app back to azure
Set-AzWebApp $newapp
```

> [!IMPORTANT]
> `$newapp.SiteConfig.NumberOfWorkers` est différent de `$newapp.MaxNumberOfWorkers`. La mise à l’échelle par application utilise `$newapp.SiteConfig.NumberOfWorkers` pour déterminer les caractéristiques de mise à l’échelle de l’application.

## <a name="per-app-scaling-using-azure-resource-manager"></a>Mise à l’échelle par application avec Azure Resource Manager

Le modèle Azure Resource Manager ci-dessous crée les éléments suivants :

- Un plan App Service réparti sur 10 instances
- Une application configurée pour une mise à l’échelle vers un maximum de cinq instances.

Le plan App Service définit la propriété **PerSiteScaling** sur true `"perSiteScaling": true`. L’application définit le **nombre de Workers** à utiliser sur 5 `"properties": { "numberOfWorkers": "5" }`.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters":{
        "appServicePlanName": { "type": "string" },
        "appName": { "type": "string" }
        },
    "resources": [
    {
        "comments": "App Service Plan with per site perSiteScaling = true",
        "type": "Microsoft.Web/serverFarms",
        "sku": {
            "name": "P1",
            "tier": "Premium",
            "size": "P1",
            "family": "P",
            "capacity": 10
            },
        "name": "[parameters('appServicePlanName')]",
        "apiVersion": "2015-08-01",
        "location": "West US",
        "properties": {
            "name": "[parameters('appServicePlanName')]",
            "perSiteScaling": true
        }
    },
    {
        "type": "Microsoft.Web/sites",
        "name": "[parameters('appName')]",
        "apiVersion": "2015-08-01-preview",
        "location": "West US",
        "dependsOn": [ "[resourceId('Microsoft.Web/serverFarms', parameters('appServicePlanName'))]" ],
        "properties": { "serverFarmId": "[resourceId('Microsoft.Web/serverFarms', parameters('appServicePlanName'))]" },
        "resources": [ {
                "comments": "",
                "type": "config",
                "name": "web",
                "apiVersion": "2015-08-01",
                "location": "West US",
                "dependsOn": [ "[resourceId('Microsoft.Web/Sites', parameters('appName'))]" ],
                "properties": { "numberOfWorkers": "5" }
            } ]
        }]
}
```

## <a name="recommended-configuration-for-high-density-hosting"></a>Configuration recommandée pour l’hébergement haute densité

La fonctionnalité de mise à l’échelle par application est activée dans les régions Azure et les [environnements App Service](environment/app-service-app-service-environment-intro.md) globaux. Toutefois, la stratégie recommandée consiste à utiliser des environnements App Service Environment pour tirer parti de leurs fonctionnalités avancées et de la capacité supérieure du plan App Service.  

Suivez ces étapes pour configurer un hébergement haute densité pour vos applications :

1. Désignez un plan App Service en tant que plan haute densité et augmentez la taille des instances de manière à obtenir la capacité de votre choix.
1. Définissez l’indicateur `PerSiteScaling` sur true dans le plan App Service.
1. Les nouvelles applications sont créées et attribuées à ce plan App Service avec la propriété **numberOfWorkers** définie sur **1**.
   - Cette configuration vous offre la densité la plus élevée possible.
1. Le nombre de Workers peut être configuré indépendamment par application pour accorder des ressources supplémentaires en fonction des besoins. Par exemple :
   - Une application à haut niveau d’utilisation peut définir **numberOfWorkers** sur **3** pour accroître la capacité de traitement de cette application.
   - Les applications à faible niveau d’utilisation définiraient **numberOfWorkers** sur **1**.

## <a name="next-steps"></a>Étapes suivantes

- [Présentation détaillée des plans Azure App Service](overview-hosting-plans.md)
- [Présentation de l'environnement App Service](environment/app-service-app-service-environment-intro.md)
