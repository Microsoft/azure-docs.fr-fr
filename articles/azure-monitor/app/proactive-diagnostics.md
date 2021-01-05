---
title: Détection intelligente dans Azure Application Insights | Microsoft Docs
description: Application Insights réalise une analyse télémétrique approfondie automatique de votre application et vous avertit des éventuels problèmes de performances.
ms.topic: conceptual
ms.date: 02/07/2019
ms.openlocfilehash: 2b6e9370387bf3bb87555261b3949db2917e9911
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96186250"
---
# <a name="smart-detection-in-application-insights"></a>Détection intelligente dans Application Insights
 La détection intelligente vous informe automatiquement des éventuels problèmes de performances et des défaillances dans votre application web. Elle effectue une analyse proactive des données de télémétrie que votre application envoie à [Application Insights](./app-insights-overview.md). S’ils détectent une augmentation soudaine du taux d’échec, ou des modèles anormaux de performances client ou serveur, vous recevez une alerte. Cette fonctionnalité ne nécessite aucune configuration. Elle fonctionne si votre application envoie suffisamment de données de télémétrie.

Vous pouvez accéder aux détections émises par la détection intelligente via les e-mails que vous recevez et à partir du panneau Détection intelligente.

## <a name="review-your-smart-detections"></a>Passer en revue vos détections intelligentes
Vous pouvez découvrir des détections de deux manières :

* **Vous recevez un message électronique** de la part d’Application Insights. Voici un exemple typique :
  
    ![Alerte par courrier électronique](./media/proactive-diagnostics/03.png)
  
    Cliquez sur le grand bouton pour afficher plus de détails dans le portail.
* **Panneau Détection intelligente** dans Application Insights. Sélectionnez **Détection intelligente** sous le menu **Examiner** pour afficher une liste des détections récentes.

![Afficher les détections récentes](./media/proactive-diagnostics/04.png)

Sélectionnez une détection pour en afficher les détails.

## <a name="what-problems-are-detected"></a>Quels sont les problèmes détectés ?
La fonctionnalité de détection intelligente vous envoie une notification quand elle détecte divers problèmes, notamment :

* [Détection intelligente des anomalies de type échec](./proactive-failure-diagnostics.md). Nous utilisons l’apprentissage automatique pour définir la fréquence attendue des demandes entraînant un échec pour votre application, en la mettant en corrélation avec la charge et d’autres facteurs. Si le taux d’échec est situé en dehors de la plage attendue, nous envoyons une alerte.
* [Détection intelligente des anomalies de performances](./proactive-performance-diagnostics.md). Vous recevez des notifications si le temps de réponse d’une opération ou la durée d’une dépendance ralentit par rapport à l’historique de la ligne de base ou si nous identifions un modèle anormal dans le temps de réponse ou le temps de chargement de la page.   
* Les dégradations et problèmes généraux, notamment la [dégradation de Trace](./proactive-trace-severity.md), la [fuite de mémoire](./proactive-potential-memory-leak.md), l’[élévation anormale du volume des exceptions](./proactive-exception-volume.md) et les [anti-modèles de sécurité](./proactive-application-security-detection-pack.md).

(Les liens d’aide dans chaque notification vous renvoient vers les articles pertinents.)

## <a name="smart-detection-email-notifications"></a>Notifications par e-mail de détection intelligente

Toutes les règles de détection intelligente, à l’exception des règles marquées en tant que _préversion_, sont configurées par défaut pour envoyer des notifications par e-mail en cas de détection.

La configuration de notifications par e-mail pour une règle de détection intelligente spécifique peut être effectuée en ouvrant le panneau **Paramètres** de détection intelligente et en sélectionnant la règle, ce qui ouvre le panneau **Modifier une règle**.

Vous pouvez également utiliser les modèles Azure Resource Manager pour modifier la configuration. Pour plus d'informations, consultez [Gérer les règles de détection intelligente d’Application Insights à l’aide de modèles Azure Resource Manager](./proactive-arm-config.md).

## <a name="video"></a>Vidéo

> [!VIDEO https://channel9.msdn.com/events/Connect/2016/112/player]

## <a name="next-steps"></a>Étapes suivantes
Ces outils de diagnostic vous aident à inspecter les données de télémétrie à partir de votre application :

* [Metrics Explorer](../platform/metrics-charts.md)
* [Navigateur de recherche](./diagnostic-search.md)
* [Analytics : un puissant langage de requête](../log-query/log-analytics-tutorial.md)

La détection intelligente est entièrement automatique. Mais vous souhaitez peut-être configurer des alertes supplémentaires ?

* [Alertes de mesures configurées manuellement](../platform/alerts-log.md)
* [Tests web de disponibilité](./monitor-web-app-availability.md)