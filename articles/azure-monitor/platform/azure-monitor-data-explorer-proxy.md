---
title: Requête inter-ressources d’Azure Data Explorer avec Azure Monitor
description: Utilisez Azure Monitor pour effectuer des requêtes inter-produits entre Azure Data Explorer, des espaces de travail Log Analytics et des applications Application Insights classiques dans Azure Monitor.
author: orens
ms.author: bwren
ms.reviewer: bwren
ms.subservice: logs
ms.topic: conceptual
ms.date: 12/02/2020
ms.openlocfilehash: 5cb2f7b3b07c20e09d61e97412bc35f03b15cb3b
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96572148"
---
# <a name="cross-resource-query-azure-data-explorer-using-azure-monitor"></a>Requête inter-ressources d’Azure Data Explorer avec Azure Monitor
Azure Monitor prend en charge les requêtes inter-services entre Azure Data Explorer, [Application Insights (AI)](/azure/azure-monitor/app/app-insights-overview) et [Log Analytics (LA)](/azure/azure-monitor/platform/data-platform-logs). Vous pouvez ensuite interroger votre cluster Azure Data Explorer à l'aide des outils Log Analytics/Application Insights et y faire référence dans une requête inter-services. L’article montre comment effectuer une requête inter-services.

Flux inter-services d’Azure Monitor : :::image type="content" source="media\azure-data-explorer-monitor-proxy\azure-monitor-data-explorer-flow.png" alt-text="Flux inter-services d’Azure Monitor et Azure Data Explorer.":::

>[!NOTE]
>* La requête inter-services d’Azure Monitor est en préversion privée - AllowListing est obligatoire.
>* Pour toute question, contactez [l’équipe du service](mailto:ADXProxy@microsoft.com).
## <a name="cross-query-your-log-analytics-or-application-insights-resources-and-azure-data-explorer"></a>Interrogation croisée de vos ressources Log Analytics ou Application Insights et d’Azure Data Explorer

Vous pouvez exécuter les requêtes inter-ressources à l’aide des outils clients qui prennent en charge les requêtes Kusto, par exemple : IU web Log Analytics, Workbooks, PowerShell, API REST et plus encore.

* Entrez l’identificateur d’un cluster Azure Data Explorer dans une requête au sein du modèle ‘adx’ suivi du nom et de la table de la base de données.

```kusto
adx('https://help.kusto.windows.net/Samples').StormEvents
```
:::image type="content" source="media/azure-data-explorer-monitor-proxy/azure-monitor-cross-service-query-example.png" alt-text="Exemple de requête inter-services.":::

> [!NOTE]
>* Les noms de base de données respectent la casse.
>* Les requêtes inter-ressources en tant qu’alerte ne sont pas prises en charge.
## <a name="combining-azure-data-explorer-cluster-tables-using-union-and-join-with-la-workspace"></a>Combinaison de tables de cluster Azure Data Explorer (avec union et join) et d’un espace de travail LA.

```kusto
union customEvents, adx('https://help.kusto.windows.net/Samples').StormEvents
| take 10
```
```kusto
let CL1 = adx('https://help.kusto.windows.net/Samples').StormEvents;
union customEvents, CL1 | take 10
```
:::image type="content" source="media/azure-data-explorer-monitor-proxy/azure-monitor-union-cross-query.png" alt-text="Exemple de requête inter-services avec union.":::

>[!Tip]
>* Le format abrégé est autorisé - ClusterName/InitialCatalog. Par exemple, adx('help/Samples') est traduit en adx('help.kusto.windows.net/Samples')
## <a name="join-data-from-an-azure-data-explorer-cluster-in-one-tenant-with-an-azure-monitor-resource-in-another"></a>Joindre les données d’un cluster Azure Data Explorer dans un locataire avec une ressource Azure Monitor située dans un autre

Les requêtes interlocataires entre les services ne sont pas prises en charge. Vous êtes connecté à un seul locataire pour exécuter la requête qui couvre les deux ressources.

Si la ressource Azure Data Explorer figure dans le locataire « A » et que l’espace de travail Log Analytics figure dans le locataire « B », utilisez l’une des deux méthodes suivantes :

*  Azure Data Explorer vous permet d’ajouter des rôles pour les principaux dans différents locataires. Ajoutez votre ID d’utilisateur dans le locataire « B » en tant qu’utilisateur autorisé sur le cluster Azure Data Explorer. Validez la propriété *['TrustedExternalTenant'](https://docs.microsoft.com/powershell/module/az.kusto/update-azkustocluster)* sur le cluster Azure Data Explorer qui contient le locataire « B ». Exécutez la requête croisée intégralement dans le locataire « B ».
*  Utilisez [Lighthouse](https://docs.microsoft.com/azure/lighthouse/) pour projeter la ressource Azure Monitor dans le locataire « A ».
## <a name="connect-to-azure-data-explorer-clusters-from-different-tenants"></a>Se connecter à des clusters Azure Data Explorer à partir de différents locataires

Kusto Explorer vous connecte automatiquement au locataire auquel le compte d’utilisateur appartient à l’origine. Pour accéder aux ressources d’autres locataires avec le même compte d’utilisateur, le `tenantId` doit être spécifié explicitement dans la chaîne de connexion : `Data Source=https://ade.applicationinsights.io/subscriptions/SubscriptionId/resourcegroups/ResourceGroupName;Initial Catalog=NetDefaultDB;AAD Federated Security=True;Authority ID=`**TenantId**

## <a name="next-steps"></a>Étapes suivantes
* [Écrire des requêtes](https://docs.microsoft.com/azure/data-explorer/write-queries)
* [Interroger des données dans Azure Monitor avec Azure Data Explorer](https://docs.microsoft.com/azure/data-explorer/query-monitor-data)
* [Effectuer des requêtes de journal inter-ressources dans Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/log-query/cross-workspace-query)
