---
title: Tableau de bord de vue d’ensemble d’Azure Application Insights | Microsoft Docs
description: Analysez les applications avec des fonctionnalités Azure Application Insights et de tableau de bord de vue d’ensemble.
ms.topic: conceptual
ms.date: 06/03/2019
ms.openlocfilehash: 1b0708fa70d3a3ecb406f1d974bb1f2b47e55b40
ms.sourcegitcommit: 2ba6303e1ac24287762caea9cd1603848331dd7a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97504098"
---
# <a name="application-insights-overview-dashboard"></a>Tableau de bord de vue d’ensemble d’Application Insights

Application Insights a toujours fourni un volet d’aperçu du résumé pour permettre l’évaluation en un coup d’œil rapide de l’intégrité et des performances de votre application. Le nouveau tableau de bord de vue d’ensemble offre une expérience plus souple et plus rapide.

## <a name="how-do-i-test-out-the-new-experience"></a>Comment tester la nouvelle expérience ?

Le nouveau tableau de bord de vue d’ensemble se lance maintenant par défaut :

![Volet d’aperçu de la vue d’ensemble](./media/overview-dashboard/overview.png)

## <a name="better-performance"></a>Meilleures performances

La sélection de plage horaire a été simplifiée en une interface en un clic simple.

![Plage temporelle](./media/overview-dashboard/app-insights-overview-dashboard-03.png)

Les performances globales ont considérablement augmenté. Des fonctionnalités courantes comme la **Recherche** et **l’Analyse** sont accessibles d’un seul clic. Chaque vignette de KPI, qui se met à jour dynamiquement par défaut, donne des insights sur les fonctionnalités d’Application Insights correspondantes. Pour en savoir plus sur les demandes ayant échoué, sélectionnez **Échecs** sous l’en-tête **Examiner** :

![Échecs](./media/overview-dashboard/app-insights-overview-dashboard-04.png)

## <a name="application-dashboard"></a>Tableau de bord de l’application

Le tableau de bord de l’application tire parti de la technologie de tableau de bord existante dans Azure pour fournir une vue unique entièrement personnalisable de l’intégrité des applications et de leurs performances.

Pour accéder au tableau de bord par défaut, sélectionnez _Tableau de bord de l’application_ dans le coin supérieur gauche.

![Capture d'écran sur laquelle le bouton Tableau de bord d'application est en surbrillance.](./media/overview-dashboard/app-insights-overview-dashboard-05.png)

Lors du premier accès au tableau de bord, cette action lance une vue par défaut :

![Vue du tableau de bord](./media/overview-dashboard/0001-dashboard.png)

Vous pouvez conserver la vue par défaut si elle vous convient, ou bien ajouter et supprimer des éléments du tableau de bord pour mieux répondre aux besoins de votre équipe.

> [!NOTE]
> Tous les utilisateurs ayant accès à la ressource Application Insights partagent la même expérience de tableau de bord d’Application Insights. Si un utilisateur apporte des changements, l’affichage est modifié pour tous les utilisateurs.

Pour revenir à l’expérience de vue d’ensemble, sélectionnez simplement :

![Bouton de la vue d’ensemble](./media/overview-dashboard/app-insights-overview-dashboard-07.png)

## <a name="troubleshooting"></a>Dépannage

Il existe actuellement une limite de 30 jours de données pour les données affichées dans un tableau de bord. Si vous sélectionnez un filtre de temps au-delà de 30 jours ou si vous sélectionnez **Configurer les paramètres de vignette** et définissez un intervalle de temps personnalisé dépassant 30 jours, votre tableau de bord n’affichera pas plus de 30 jours de données, même avec la conservation des données par défaut de 90 jours. Il n’existe actuellement aucun moyen de contourner ce comportement.

## <a name="next-steps"></a>Étapes suivantes

- [Entonnoirs](./usage-funnels.md)
- [Rétention](./usage-retention.md)
- [Flux d’utilisateurs](./usage-flows.md)

