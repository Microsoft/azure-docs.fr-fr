---
title: Créer une ressource Azure Application Insights | Microsoft Docs
description: Configurez manuellement la surveillance d’Application Insights pour une nouvelle application en direct.
ms.topic: conceptual
ms.date: 12/02/2019
ms.openlocfilehash: 3fd05e6bd68be89b964fe1ad32029bf44f3352ea
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96906367"
---
# <a name="create-an-application-insights-resource"></a>Création d’une ressource Application Insights dans Azure

Azure Application Insights affiche les données relatives à votre application dans une *ressource* Microsoft Azure. La création d’une nouvelle ressource fait, par conséquent, partie de la [configuration d’Application Insights pour surveiller une nouvelle application][start]. Après avoir créé votre nouvelle ressource, vous pouvez obtenir sa clé d’instrumentation et l’utiliser pour configurer le Kit de développement logiciel (SDK) Application Insights. La clé d’instrumentation lie vos données de télémétrie à la ressource.

## <a name="sign-in-to-microsoft-azure"></a>Se connecter à Microsoft Azure

Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="create-an-application-insights-resource"></a>Création d’une ressource Application Insights dans Azure

Connectez-vous au [portail Azure](https://portal.azure.com) et créez une ressource Application Insights :

![Cliquez sur le signe « + » dans le coin supérieur gauche. Sélectionner Outils de développement, puis Application Insights](./media/create-new-resource/new-app-insights.png)

   | Paramètres        |  Valeur           | Description  |
   | ------------- |:-------------|:-----|
   | **Nom**      | `Unique value` | Nom identifiant l’application que vous analysez. |
   | **Groupe de ressources**     | `myResourceGroup`      | Nom du nouveau groupe de ressources ou existant pour héberger les données Application Insights. |
   | **Région** | `East US` | Choisissez un emplacement près de chez vous ou proche de l’endroit où votre application est hébergée. |
   | **Mode de ressources** | `Classic` ou `Workspace-based` | Actuellement en préversion publique, les ressources basées sur l’espace de travail vous permettent d’envoyer vos données de télémétrie Application Insights à un espace de travail Log Analytics commun. Pour plus d’informations, consultez l'[article relatif aux ressources basées sur l'espace de travail](create-workspace-resource.md).

> [!NOTE]
> Bien que vous puissiez utiliser le même nom de ressource dans différents groupes de ressources, il est préférable d’utiliser un nom global unique. Ce dernier peut s’avérer utile si vous envisagez d’[exécuter des requêtes de ressources croisées](../log-query/cross-workspace-query.md#identifying-an-application), car cela simplifie la syntaxe nécessaire.

Entrez les valeurs appropriées dans les champs obligatoires, puis sélectionnez **Vérifier + créer**.

![Entrez des valeurs dans les champs obligatoires, puis sélectionnez « Vérifier + créer ».](./media/create-new-resource/review-create.png)

Une fois votre application créée, un nouveau volet s’ouvre. Dans ce volet, vous trouverez des données relatives à l’utilisation et aux performances de votre application analysée. 

## <a name="copy-the-instrumentation-key"></a>Copie de la clé d’instrumentation

La clé d’instrumentation identifie la ressource à laquelle vous souhaitez associer vos données de télémétrie. Vous devez copier la clé d’instrumentation et l’ajouter au code de votre application.

> [!IMPORTANT]
> Les nouvelles régions Azure **exigent** l’utilisation de chaînes de connexion au lieu de clés d’instrumentation. Une [chaîne de connexion](./sdk-connection-string.md?tabs=net) identifie la ressource à laquelle vous souhaitez associer vos données de télémétrie. Elle vous permet également de modifier les points de terminaison que votre ressource utilisera comme destination pour votre télémétrie. Vous devrez copier la chaîne de connexion et l’ajouter au code de votre application ou à une variable d’environnement.

## <a name="install-the-sdk-in-your-app"></a>Installation du Kit SDK dans votre application

Installez le Kit SDK Application Insights dans votre application. Cette étape repose en grande partie sur votre type d’application.

La clé d’instrumentation permet de configurer le [kit de développement logiciel (SDK) que vous avez installé dans votre application][start].

Le Kit de développement logiciel (SDK) inclut des modules standard qui envoient des données de télémétrie sans avoir à écrire du code supplémentaire. Pour suivre les actions des utilisateurs ou diagnostiquer des problèmes plus en détail, [utilisez l'API][api] pour envoyer votre propre télémétrie.

## <a name="creating-a-resource-automatically"></a>Création automatique d’une ressource

### <a name="powershell"></a>PowerShell

Créer une ressource Application Insights dans Azure

```powershell
New-AzApplicationInsights [-ResourceGroupName] <String> [-Name] <String> [-Location] <String> [-Kind <String>]
 [-Tag <Hashtable>] [-DefaultProfile <IAzureContextContainer>] [-WhatIf] [-Confirm] [<CommonParameters>]
```

#### <a name="example"></a>Exemple

```powershell
New-AzApplicationInsights -Kind java -ResourceGroupName testgroup -Name test1027 -location eastus
```
#### <a name="results"></a>Résultats

```powershell
Id                 : /subscriptions/{subid}/resourceGroups/testgroup/providers/microsoft.insights/components/test1027
ResourceGroupName  : testgroup
Name               : test1027
Kind               : web
Location           : eastus
Type               : microsoft.insights/components
AppId              : 8323fb13-32aa-46af-b467-8355cf4f8f98
ApplicationType    : web
Tags               : {}
CreationDate       : 10/27/2017 4:56:40 PM
FlowType           :
HockeyAppId        :
HockeyAppToken     :
InstrumentationKey : 00000000-aaaa-bbbb-cccc-dddddddddddd
ProvisioningState  : Succeeded
RequestSource      : AzurePowerShell
SamplingPercentage :
TenantId           : {subid}
```

Pour obtenir la documentation PowerShell complète de cette applet de commande et apprendre à récupérer la clé d’instrumentation, consultez la [documentation Azure PowerShell](/powershell/module/az.applicationinsights/new-azapplicationinsights?view=azps-2.5.0).

### <a name="azure-cli-preview"></a>Azure CLI (préversion)

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
                                         [--kind]
                                         [--tags]
```

#### <a name="example"></a>Exemple

```azurecli
az monitor app-insights component create --app demoApp --location westus2 --kind web -g demoRg --application-type web
```

#### <a name="results"></a>Résultats

```azurecli
az monitor app-insights component create --app demoApp --location eastus --kind web -g demoApp  --application-type web
{
  "appId": "87ba512c-e8c9-48d7-b6eb-118d4aee2697",
  "applicationId": "demoApp",
  "applicationType": "web",
  "creationDate": "2019-08-16T18:15:59.740014+00:00",
  "etag": "\"0300edb9-0000-0100-0000-5d56f2e00000\"",
  "flowType": "Bluefield",
  "hockeyAppId": null,
  "hockeyAppToken": null,
  "id": "/subscriptions/{subid}/resourceGroups/demoApp/providers/microsoft.insights/components/demoApp",
  "instrumentationKey": "00000000-aaaa-bbbb-cccc-dddddddddddd",
  "kind": "web",
  "location": "eastus",
  "name": "demoApp",
  "provisioningState": "Succeeded",
  "requestSource": "rest",
  "resourceGroup": "demoApp",
  "samplingPercentage": null,
  "tags": {},
  "tenantId": {tenantID},
  "type": "microsoft.insights/components"
}
```

Pour obtenir la documentation complète Azure CLI de cette commande et savoir comment récupérer la clé d’instrumentation, consultez la [documentation Azure CLI](/cli/azure/ext/application-insights/monitor/app-insights/component?view=azure-cli-latest#ext-application-insights-az-monitor-app-insights-component-create).

## <a name="next-steps"></a>Étapes suivantes
* [Recherche de diagnostic](./diagnostic-search.md)
* [Exploration des mesures](../platform/metrics-charts.md)
* [Écriture de requêtes Analytics](../log-query/log-query-overview.md)

<!--Link references-->

[api]: ./api-custom-events-metrics.md
[diagnostic]: ./diagnostic-search.md
[metrics]: ../platform/metrics-charts.md
[start]: ./app-insights-overview.md

