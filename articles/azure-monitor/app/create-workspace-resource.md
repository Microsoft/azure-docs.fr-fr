---
title: Créer une ressource Azure Monitor Application Insights basée sur un espace de travail | Microsoft Docs
description: En savoir plus sur les étapes requises pour activer les nouvelles ressources Azure Monitor Application Insights basées sur un espace de travail.
ms.topic: conceptual
ms.date: 10/06/2020
ms.openlocfilehash: ecbac02bcb4d9b4f0db36eab854a91366c774d6c
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95536912"
---
# <a name="workspace-based-application-insights-resources"></a>Ressources Application Insights basées sur l’espace de travail

Les ressources basées sur un espace de travail prennent en charge l’intégration complète entre Application Insights et Log Analytics. Vous pouvez maintenant choisir d’envoyer votre télémétrie Application Insights à un espace de travail Log Analytics commun, ce qui vous permet d’accéder entièrement à toutes les fonctionnalités de Log Analytics tout en conservant les journaux des applications, des infrastructures et des plateformes dans un emplacement consolidé unique.

Cela permet également de disposer d’un contrôle d’accès en fonction du rôle Azure (Azure RBAC) commun sur vos ressources et élimine le besoin de requêtes entre des applications ou des espaces de travail.

> [!NOTE]
> L’ingestion et la conservation des données pour les ressources Application Insights basées sur un espace de travail sont facturées dans l’espace de travail Log Analytics où se trouvent les données. [En savoir plus]( ./pricing.md#workspace-based-application-insights) sur la facturation des ressources Application Insights basées sur un espace de travail.

## <a name="new-capabilities"></a>Nouvelles fonctionnalités

Les Application Insights basées sur l’espace de travail vous permettent de tirer profit des dernières fonctionnalités d’Azure Monitor et Log Analytics, y compris :

* [Les clés gérées par le client (CMK)](../platform/customer-managed-keys.md) fournissent un chiffrement au repos pour vos données avec des clés de chiffrement auxquelles vous seul avez accès.
* [Azure Private Link](../platform/private-link-security.md) vous permet de lier en toute sécurité les services PaaS Azure à votre réseau virtuel à l’aide de points de terminaison privés.
* [BYOS (apportez votre propre stockage) pour Profiler et Débogueur de capture instantanée](./profiler-bring-your-own-storage.md) vous offre un contrôle total sur la stratégie de chiffrement au repos, la stratégie de gestion de la durée de vie et l’accès réseau pour toutes les données associées à Application Insights Profiler et Débogueur de capture instantanée. 
* Les [niveaux de réservation de capacité](../platform/manage-cost-storage.md#pricing-model) vous permettent d’économiser jusqu’à 25 % par rapport au tarif du paiement à l’utilisation. 
* Ingestion plus rapide des données via l’ingestion de diffusion en continu Log Analytics.

## <a name="create-workspace-based-resource"></a>Créer une ressource basée sur un espace de travail

Connectez-vous au [portail Azure](https://portal.azure.com) et créez une ressource Application Insights :

![Ressource Application Insights basée sur un espace de travail](./media/create-workspace-resource/create-workspace-based.png)

Si vous ne disposez pas déjà d’un espace de travail Log Analytics, [consultez la documentation relative à la création d’un espace de travail Log Analytics](../learn/quick-create-workspace.md).

**Les ressources basées sur l’espace de travail sont actuellement disponibles dans toutes les régions commerciales et Azure Government**

Une fois votre ressource créée, les informations correspondantes de l’espace de travail s’affichent dans le volet **Vue d’ensemble** :

![Nom de l’espace de travail](./media/create-workspace-resource/workspace-name.png)

Le fait de cliquer sur le texte du lien bleu vous permet d’accéder à l’espace de travail Log Analytics associé dans lequel vous pouvez tirer parti du nouvel environnement de requête de l’espace de travail unifié.

> [!NOTE]
> Nous proposons toujours une compatibilité descendante complète pour vos classeurs, alertes basées sur les journaux et requêtes de ressources Application Insights classiques au sein de l’expérience Application Insights. Pour interroger/afficher [le nouveau schéma ou la nouvelle structure de table basé sur un espace de travail](apm-tables.md), vous devez d’abord accéder à votre espace de travail Log Analytics. La sélection de **Journaux (Analytics)** dans les volets d’Application Insights vous donne accès à l’expérience de requête Application Insights classique.

## <a name="copy-the-connection-string"></a>Copier la chaîne de connexion

La [chaîne de connexion](./sdk-connection-string.md?tabs=net) identifie la ressource à laquelle vous souhaitez associer vos données de télémétrie. Elle vous permet également de modifier les points de terminaison que votre ressource utilisera comme destination pour votre télémétrie. Vous devrez copier la chaîne de connexion et l’ajouter au code de votre application ou à une variable d’environnement.

## <a name="monitoring-configuration"></a>Configuration de la supervision

Une fois qu’une ressource Application Insights basée sur un espace de travail a été créée, la configuration de la supervision est relativement simple.

### <a name="code-based-application-monitoring"></a>Supervision des applications basée sur du code

Pour la supervision des applications basée sur du code, vous devez simplement installer le Kit de développement logiciel (SDK) Application Insights approprié et le pointer vers la clé d’instrumentation ou la chaîne de connexion associée à la ressource que vous venez de créer.  

Pour obtenir une documentation détaillée sur la configuration d’un Kit de développement logiciel (SDK) Application Insights pour la supervision basée sur du code, consultez la documentation spécifique au langage ou à l’infrastructure :

- [ASP.NET](./asp-net.md)
- [ASP.NET Core](./asp-net-core.md)
- [Tâches en arrière-plan et applications console modernes (.NET/.NET Core)](./worker-service.md)
- [Applications console classiques (.NET)](./console.md) 
- [Java](./java-get-started.md?tabs=maven)
- [JavaScript](./javascript.md)
- [Node.JS](./nodejs.md)
- [Python](./opencensus-python.md)

### <a name="codeless-monitoring-and-visual-studio-resource-creation"></a>Supervision sans code et création de ressources Visual Studio

Pour la supervision sans code des services tels qu’Azure Functions et Azure App Service, vous devez d’abord créer votre ressource Application Insights basée sur l’espace de travail, puis pointer sur cette ressource pendant la phase de configuration de la supervision.

Bien que ces services offrent la possibilité de créer une ressource Application Insights dans le cadre de leur propre processus de création de ressources, les ressources créées via ces options d’interface utilisateur sont actuellement limitées à l’expérience Application Insights classique.

Il en va de même pour l’expérience de création de ressources Application Insights dans Visual Studio pour ASP.NET et ASP.NET Core. Vous devez sélectionner une ressource existante basée sur l’espace de travail à l’aide de l’interface utilisateur de supervision de Visual Studio. En choisissant de créer une nouvelle ressource à partir de Visual Studio, vous serez limité à la création d’une ressource Application Insights classique.

## <a name="creating-a-resource-automatically"></a>Création automatique d’une ressource

### <a name="azure-cli"></a>Azure CLI

Pour accéder à la préversion des commandes Azure CLI pour Application Insights, vous devez d’abord exécuter :

```azurecli
 az extension add -n application-insights
```

Si vous n’exécutez pas la commande `az extension add`, vous verrez un message d’erreur qui indique : `az : ERROR: az monitor: 'app-insights' is not in the 'az monitor' command group. See 'az monitor --help'.`.

Vous pouvez maintenant exécuter la commande suivante pour créer votre ressource Application Insights :

```azurecli
az monitor app-insights component create --app
                                         --location
                                         --resource-group
                                         [--application-type]
                                         [--ingestion-access {Disabled, Enabled}]
                                         [--kind]
                                         [--only-show-errors]
                                         [--query-access {Disabled, Enabled}]
                                         [--tags]
                                         [--workspace]
```

#### <a name="example"></a>Exemple

```azurecli
az monitor app-insights component create --app demoApp --location eastus --kind web -g my_resource_group --workspace "/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/test1234/providers/microsoft.operationalinsights/workspaces/test1234555"
```

Pour obtenir la documentation Azure CLI complète pour cette commande, consultez la [documentation Azure CLI](/cli/azure/ext/application-insights/monitor/app-insights/component?view=azure-cli-latest#ext-application-insights-az-monitor-app-insights-component-create).

### <a name="azure-powershell"></a>Azure PowerShell

La commande PowerShell `New-AzApplicationInsights` ne prend pas actuellement en charge la création d’une ressource Application Insights basée sur l’espace de travail. Pour créer une ressource basée sur l’espace de travail avec PowerShell, vous pouvez utiliser les modèles Azure Resource Manager ci-dessous et les déployer à l’aide de PowerShell.

### <a name="azure-resource-manager-templates"></a>Modèles Microsoft Azure Resource Manager

#### <a name="template-file"></a>Fichier de modèle

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "name": {
            "type": "string"
        },
        "type": {
            "type": "string"
        },
        "regionId": {
            "type": "string"
        },
        "tagsArray": {
            "type": "object"
        },
        "requestSource": {
            "type": "string"
        },
        "workspaceResourceId": {
            "type": "string"
        }
    },
    "resources": [
        {
            "name": "[parameters('name')]",
            "type": "microsoft.insights/components",
            "location": "[parameters('regionId')]",
            "tags": "[parameters('tagsArray')]",
            "apiVersion": "2020-02-02-preview",
            "properties": {
                "ApplicationId": "[parameters('name')]",
                "Application_Type": "[parameters('type')]",
                "Flow_Type": "Redfield",
                "Request_Source": "[parameters('requestSource')]",
                "WorkspaceResourceId": "[parameters('workspaceResourceId')]"
            }
        }
    ]
}
```

#### <a name="parameters-file"></a>Fichier de paramètres

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "type": {
            "value": "web"
        },
        "name": {
            "value": "customresourcename"
        },
        "regionId": {
            "value": "eastus"
        },
        "tagsArray": {
            "value": {}
        },
        "requestSource": {
            "value": "Custom"
        },
        "workspaceResourceId": {
            "value": "/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/my_resource_group/providers/microsoft.operationalinsights/workspaces/myworkspacename"
        }
    }
}

```

## <a name="modifying-the-associated-workspace"></a>Modification de l’espace de travail associé

Une fois qu’une ressource Application Insights basée sur l’espace de travail a été créée, vous pouvez modifier l’espace de travail Log Analytics associé.

Dans le volet des ressources Application Insights, sélectionnez **Propriétés** > **Modifier l’espace de travail** > **Espaces de travail Log Analytics**

## <a name="export-telemetry"></a>Exporter la télémétrie

La fonctionnalité héritée d’exportation continue n’est pas prise en charge pour les ressources basées sur l’espace de travail. Au lieu de cela, sélectionnez **Paramètres de diagnostic** > **Ajouter un paramètre de diagnostic** à partir de votre ressource Application Insights. Vous pouvez sélectionner toutes les tables ou un sous-ensemble de tables à archiver dans un compte de stockage ou à transmettre en continu à un Event Hub Azure.

> [!NOTE]
> Il n’y a actuellement pas de frais supplémentaires pour l’exportation de la télémétrie. Les informations de tarification pour cette fonctionnalité sont disponibles sur la [page de tarification Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/).  Avant le début de la facturation, des notifications seront envoyées. Si vous choisissez de continuer à utiliser <feature name> après la période de préavis, vous serez facturé au tarif en vigueur. 
 

## <a name="next-steps"></a>Étapes suivantes

* [Exploration des mesures](../platform/metrics-charts.md)
* [Écriture de requêtes Analytics](../log-query/log-query-overview.md)