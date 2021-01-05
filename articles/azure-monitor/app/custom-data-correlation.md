---
title: Azure Application Insights | Microsoft Docs
description: Mettez en corrélation les données Application Insights avec d’autres jeux de données, tels que des tables d’enrichissement ou de recherche de données, des sources de données autres qu'Application Insights et des données personnalisées.
ms.topic: conceptual
author: eternovsky
ms.author: evternov
ms.date: 08/08/2018
ms.reviewer: mbullwin
ms.openlocfilehash: 17de697686485d695586ffa798196efb4a34c251
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87092913"
---
# <a name="correlating-application-insights-data-with-custom-data-sources"></a>Mise en corrélation de données Application Insights avec des sources de données personnalisées

Application Insights collecte plusieurs types de données différents : exceptions, traces, vues de page, etc. Bien que cela suffise bien souvent pour examiner les performances, la fiabilité et l’utilisation de votre application, dans certains cas, il est utile mettre en corrélation les données stockées dans Application Insights avec d’autres jeux de données entièrement personnalisés.

Vous pouvez avoir besoin de données personnalisées dans les situations suivantes :

- Tables d’enrichissement ou de recherche de données : par exemple, compléter un nom de serveur avec le propriétaire du serveur et l’emplacement de laboratoire dans lequel il peut être trouvé 
- Corrélation avec des sources de données autres qu’Application Insights : par exemple, corréler des données concernant un achat effectué dans une boutique en ligne avec des informations obtenues auprès de votre service de traitement des achats pour déterminer la précision des estimations de vos délais de livraison 
- Données complètement personnalisées : parmi nos clients, beaucoup adorent le langage de requête et les performances de la plateforme de journaux Azure Monitor qui sous-tend Application Insights, et cherchent à en tirer parti pour interroger les données qui ne sont pas du tout liées à Application Insights. Par exemple, pour suivre les performances de panneaux solaires dans le cadre d’une installation résidentielle intelligente, comme décrit [ici](https://www.catapultsystems.com/blogs/using-log-analytics-and-a-special-guest-to-forecast-electricity-generation/).

## <a name="how-to-correlate-custom-data-with-application-insights-data"></a>Comment mettre en corrélation des données personnalisées avec des données Application Insights 

Étant donné qu'Application Insights est associé à la puissante plateforme de journaux Azure Monitor, nous sommes en mesure d'exploiter toute la puissance d'Azure Monitor pour ingérer les données. Nous allons ensuite rédiger des requêtes à l’aide de l’opérateur « join » afin de mettre en corrélation ces données personnalisées avec les données dont nous disposons dans les journaux d’activité Azure Monitor. 

## <a name="ingesting-data"></a>Ingestion des données

Dans cette section, vous allez importer vos données dans les journaux d’activité Azure Monitor.

Si vous ne l’avez pas déjà fait, configurez un nouvel espace de travail Log Analytics en suivant [ces instructions](../learn/quick-collect-azurevm.md) jusqu’à l’étape relative à la « création d’un espace de travail ».

Pour commencer à envoyer des données de journal dans Azure Monitor. Vous disposez de plusieurs options :

- Pour un mécanisme synchrone, vous pouvez soit appeler directement [l’API Collecte de données](../platform/data-collector-api.md) soit utiliser notre connecteur Logic App : il vous suffit de rechercher « Azure Log Analytics » et de sélectionner l’option « Envoyer des données » :

  ![Capture d’écran choix et action](./media/custom-data-correlation/01-logic-app-connector.png)  

- Pour une méthode asynchrone, utilisez l’API Collecte de données pour créer un pipeline de traitement. Pour plus d’informations, consultez [cet article](../platform/create-pipeline-datacollector-api.md).

## <a name="correlating-data"></a>Corrélation des données

Application Insights s'appuie sur la plateforme de journaux Azure Monitor. Par conséquent, nous pouvons utiliser des [jointures inter-ressources](../log-query/cross-workspace-query.md) pour mettre en corrélation des données que nous avons ingérées dans Azure Monitor avec nos données Application Insights.

Par exemple, nous pouvons ingérer notre inventaire et nos emplacements de laboratoire dans une table appelée « LabLocations_CL » au sein d'un espace de travail Log Analytics appelé « myLA ». Si nous voulons ensuite passer en revue nos requêtes surveillées dans l'application Application Insights appelée « myAI » et mettre en corrélation les noms d'ordinateurs qui ont traité les requêtes avec les emplacements de ces ordinateurs stockés dans la table personnalisée mentionnée précédemment, nous pouvons exécuter la requête suivante à partir du contexte Application Insights ou Azure Monitor :

```
app('myAI').requests
| join kind= leftouter (
    workspace('myLA').LabLocations_CL
    | project Computer_S, Owner_S, Lab_S
) on $left.cloud_RoleInstance == $right.Computer
```

## <a name="next-steps"></a>Étapes suivantes

- Consultez la référence de [l’API Collecte de données](../platform/data-collector-api.md).
- En savoir plus sur les [jointures inter-ressources](../log-query/cross-workspace-query.md).
