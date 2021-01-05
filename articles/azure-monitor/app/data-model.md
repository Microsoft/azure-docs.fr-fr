---
title: Modèle de données de télémétrie d’Azure Application Insights | Microsoft Docs
description: Présentation du modèle de données Application Insights
services: application-insights
documentationcenter: .net
manager: carmonm
ms.workload: TBD
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 10/14/2019
ms.reviewer: sergkanz
ms.openlocfilehash: fd5848eb44ecd32612943662b5a4fd0a5091cd9d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91461289"
---
# <a name="application-insights-telemetry-data-model"></a>Modèle de données de télémétrie d’Application Insights

[Azure Application Insights](./app-insights-overview.md) envoie la télémétrie à partir de votre application web sur le portail Azure afin que vous puissiez analyser les performances et l’utilisation de votre application. Le modèle de télémétrie est normalisé afin qu’il soit possible de créer une plateforme et une surveillance indépendante du langage. 

Les données collectées par Application Insights modélisent ce schéma standard d’exécution d’application :

![Modèle d’application Application Insights](./media/data-model/application-insights-data-model.png)

Les types de télémétrie suivants sont utilisés pour surveiller l’exécution de votre application. Les trois types suivants sont généralement collectés automatiquement par le Kit de développement logiciel (SDK) Application Insights à partir de l’infrastructure d’application web :

* [**Demande**](data-model-request-telemetry.md) : généré pour enregistrer une demande reçue par votre application. Par exemple, le Kit de développement logiciel (SDK) web Application Insights génère automatiquement un élément de télémétrie de demande pour chaque demande HTTP reçue par votre application web. 

    Une **opération** désigne le thread d’exécution qui traite une demande. Vous pouvez également [écrire du code](./api-custom-events-metrics.md#trackrequest) pour surveiller d’autres types d’opération, comme une « sortie de veille » dans une tâche ou une fonction web qui traite périodiquement des données.  Chaque opération a un ID. Cet ID peut être utilisé pour [regrouper](./correlation.md) toutes les données de télémétrie générées lors du traitement de la demande par votre application. Chaque opération réussit ou échoue et a une certaine durée.
* [**Exception**](data-model-exception-telemetry.md) : représente normalement une exception qui provoque l’échec d’une opération.
* [**Dépendance**](data-model-dependency-telemetry.md) : représente un appel depuis votre application vers un service ou un stockage externe, comme une API REST ou SQL. Dans ASP.NET, les appels de dépendance vers SQL sont définis par `System.Data`. Les appels vers les points de terminaison HTTP sont définis par `System.Net`. 

Application Insights fournit trois types de données supplémentaires pour la télémétrie personnalisée :

* [Trace](data-model-trace-telemetry.md) : utilisé directement ou via un adaptateur pour implémenter la journalisation des diagnostics à l’aide d’une infrastructure d’instrumentation qui vous est familière, comme `Log4Net` ou `System.Diagnostics`.
* [Événement](data-model-event-telemetry.md) : normalement utilisé pour capturer l’interaction de l’utilisateur avec votre service afin d’analyser les modèles d’utilisation.
* [Métrique](data-model-metric-telemetry.md) : utilisé pour générer des rapports sur les mesures scalaires périodiques.

Chaque élément de télémétrie peut définir les [informations de contexte](data-model-context.md) comme la version de l’application ou l’ID de session utilisateur. Le contexte est un ensemble de champs fortement typés qui débloque certains scénarios. Quand la version de l’application est correctement initialisée, Application Insights peut détecter les nouveaux modèles de comportement de l’application en corrélation avec un redéploiement. L’ID de session peut être utilisé pour calculer l’interruption ou l’impact d’un problème sur les utilisateurs. Calculer des nombres distincts de valeurs d’ID de session pour certaines dépendances en échec, suivis d’erreur ou exceptions critique vous donne une bonne compréhension d’un impact.

Le modèle de télémétrie d’Application Insights définit un moyen de [mettre en corrélation](./correlation.md) la télémétrie et l’opération dont elle fait partie. Par exemple, une requête peut appeler une base de données SQL et enregistrer les informations de diagnostic. Vous pouvez définir le contexte de corrélation pour relier ces éléments de télémétrie aux données de télémétrie de la demande.

## <a name="schema-improvements"></a>Améliorations du schéma

Le modèle de données d’Application Insights est certes simple et basique, mais suffisamment puissant pour modéliser les données de télémétrie de votre application. Nous faisons en sorte que ce modèle reste le plus simple et le plus léger possible pour prendre en charge des scénarios essentiels et permettre l’extension du schéma pour une utilisation avancée.

[Pour signaler des problèmes de modèle de données ou de schéma et des suggestions, utilisez notre dépôt GitHub](https://github.com/microsoft/ApplicationInsights-dotnet/issues/new/choose).

## <a name="next-steps"></a>Étapes suivantes

- [Écrire des données de télémétrie personnalisées](./api-custom-events-metrics.md)
- Découvrez comment [étendre et filtrer la télémétrie](./api-filtering-sampling.md).
- Utilisez [l’échantillonnage](./sampling.md) pour réduire la quantité de données de télémétrie basées sur le modèle de données.
- Découvrez quelles [plateformes](./platforms.md) sont prises en charge par Application Insights.

