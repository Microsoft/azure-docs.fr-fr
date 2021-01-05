---
title: Surveiller les applications qui s’exécutent sur Azure Functions à l’aide d’Application Insights – Azure Monitor | Microsoft Docs
description: Azure Monitor s’intègre harmonieusement à votre application s’exécutant sur Azure Functions et vous permet de surveiller les performances et de détecter les problèmes de vos applications en un rien de temps.
ms.topic: conceptual
author: MS-jgol
ms.author: jgol
ms.date: 06/26/2020
ms.openlocfilehash: 6d790a6f02fdc07609e374639c6e452b9088262e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87024569"
---
# <a name="monitoring-azure-functions-with-azure-monitor-application-insights"></a>Surveillance d’Azure Functions avec Application Insights d’Azure Monitor

[Azure Functions](../../azure-functions/functions-overview.md) est intégré à Azure Application Insights pour surveiller des fonctions. 

Application Insights collecte les données des journaux, des performances et des erreurs et détecte automatiquement les anomalies de performances. Application Insights intègre de puissants outils d’analyse conçus pour aider à diagnostiquer les problèmes et à comprendre la manière dont vos fonctions sont utilisées. Lorsque vous avez la possibilité de visualiser les données de vos applications, vous pouvez améliorer de manière continue les performances et la convivialité. Vous pouvez également utiliser Application Insights lors du développement d'un projet d'application de fonction local. 

L’instrumentation Application Insights nécessaire est intégrée à Azure Functions. La seule chose dont vous avez besoin est une clé d’instrumentation valide pour connecter votre application de fonction à une ressource Application Insights. La clé d’instrumentation doit être ajoutée aux paramètres de votre application lorsque la ressource de votre application de fonction est créée dans Azure. Si votre application de fonction n’a pas encore cette clé, vous pouvez la définir manuellement. Pour plus d’informations, lisez l’article relatif à la[surveillance d’Azure Functions](../../azure-functions/functions-monitoring.md?tabs=cmd).

## <a name="distributed-tracing-for-java-applications-on-windows-public-preview"></a>Suivi distribué pour les applications Java sur Windows (préversion publique)

> [!IMPORTANT]
> Cette fonctionnalité est actuellement disponible en préversion publique pour Java Azure Functions sur Windows, le suivi distribué pour Java Azure Functions sur Linux n’est pas pris en charge. Pour le plan Consommation, le démarrage à froid est de 8 à 9 secondes.

Si vos applications sont écrites en Java, vous pouvez afficher des données plus riches à partir de vos applications de fonction, notamment les requêtes, les dépendances, les journaux et les métriques. Les données supplémentaires vous permettent également de voir et de diagnostiquer des transactions de bout en bout et de voir la cartographie d’application, qui regroupe de nombreuses transactions afin d’afficher une vue topologique des interactions entre les systèmes et des taux moyens de performance et d’erreur.

Les diagnostics de bout en bout et la cartographie d’application offrent une visibilité sur une transaction/demande unique. Ensemble, ces deux fonctionnalités sont très utiles pour trouver la cause racine des problèmes de fiabilité et des goulots d’étranglement des performances par demande.

### <a name="how-to-enable-distributed-tracing-for-java-function-apps"></a>Comment activer le traçage distribué pour les applications de fonction Java ?

Accédez au panneau Vue d’ensemble des applications de fonction, puis à Configurations. Sous Paramètres d’application, cliquez sur « + Nouveau paramètre d’application ». Ajoutez les deux paramètres d’application suivants avec les valeurs ci-dessous, puis cliquez sur Enregistrer dans le coin supérieur gauche. C’est terminé !

```
XDT_MicrosoftApplicationInsights_Java -> 1
ApplicationInsightsAgent_EXTENSION_VERSION -> ~2
```

## <a name="next-steps"></a>Étapes suivantes

* Lire plus d’instructions et d’informations sur la [surveillance d’Azure Functions](../../azure-functions/functions-monitoring.md)
* Obtenir une vue d’ensemble du [traçage distribué](./distributed-tracing.md)
* Découvrir ce que la [cartographie d’application](./app-map.md?tabs=net) peut apporter à votre entreprise
* En savoir plus sur les [requêtes et les dépendances des applications Java](./java-in-process-agent.md)
* En savoir plus sur [Azure Monitor](../overview.md) et [Application Insights](./app-insights-overview.md)
