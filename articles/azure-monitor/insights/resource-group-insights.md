---
title: Informations du groupe de ressources Azure Monitor | Microsoft Docs
description: Comprendre l’intégrité et les performances de vos applications et services distribués au niveau du groupe de ressources avec Azure Monitor
ms.subservice: ''
ms.topic: conceptual
author: NumberByColors
ms.author: daviste
ms.date: 09/19/2018
ms.reviewer: mbullwin
ms.openlocfilehash: bf8ea45f37e421d70a68c4a6e138f5203e92d78f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87045781"
---
# <a name="monitor-resource-groups-with-azure-monitor-preview"></a>Surveiller les groupes de ressources avec Azure Monitor (préversion)

Les applications modernes sont souvent complexes et hautement distribuées, avec de nombreux éléments distincts qui collaborent pour fournir un service. Compte tenu de cette complexité, Azure Monitor fournit des informations de surveillance pour les groupes de ressources. Cela facilite le triage et le diagnostic des problèmes que rencontrent vos ressources individuelles, tout en offrant un contexte en lien avec l’intégrité et les performances du groupe de ressources (et de votre application) dans sa globalité.

## <a name="access-insights-for-resource-groups"></a>Accéder aux informations sur les groupes de ressources

1. Sélectionnez **Groupes de ressources** dans la barre de navigation à gauche.
2. Choisissez l’un de vos groupes de ressources que vous souhaitez explorer (si vous avez un grand nombre de groupes de ressources, filtrer par abonnement peut parfois être utile).
3. Pour accéder aux informations relatives à un groupe de ressources, cliquez sur **Insights** dans le menu à gauche de ce groupe de ressources.

![Capture d’écran de la page de vue d’ensemble des informations relatives à un groupe de ressources](./media/resource-group-insights/0001-overview.png)

## <a name="resources-with-active-alerts-and-health-issues"></a>Ressources présentant des alertes actives et des problèmes d’intégrité

La page de vue d’ensemble indique le nombre d’alertes déclenchées et toujours actives, ainsi que les informations d’Azure Resource Health pour chaque ressource. Ensemble, ces informations peuvent vous aider à identifier rapidement toutes les ressources qui rencontrent des problèmes. Les alertes vous aident à détecter des problèmes dans votre code, ainsi que la manière dont vous avez configuré votre infrastructure. Azure Resource Health fait ressortir des problèmes liés à la plateforme Azure proprement dite, qui ne sont pas spécifiques de vos applications individuelles.

![Capture d’écran du volet Azure Resource Health](./media/resource-group-insights/0002-overview.png)

### <a name="azure-resource-health"></a>Azure Resource Health

Pour afficher Azure Resource Health, activez la case à cocher **Afficher Azure Resource Health** au-dessus du tableau. Cette colonne est masquée par défaut pour accélérer le chargement de la page.

![Capture d’écran avec le graphique d’intégrité des ressources ajouté](./media/resource-group-insights/0003-overview.png)

Par défaut, les ressources sont regroupées par couche application et type de ressource. La **couche application** est une simple catégorisation des types de ressources, qui n’existe que dans le contexte de la page de vue d’ensemble des informations relatives au groupe de ressources. Il existe des types de ressources liés au code d’application, à l’infrastructure de calcul, à la mise en réseau, au stockage et au bases de données. Les outils de gestion obtiennent leurs propres couches application, et toutes les autres ressources sont catégorisées comme appartenant à l’**autre** couche application. Ce regroupement peut vous aider à voir en un clin d’œil les sous-systèmes de votre application sains et défectueux.

## <a name="diagnose-issues-in-your-resource-group"></a>Diagnostiquer des problèmes dans votre groupe de ressources

La page d’informations sur le groupe de ressources fournit d’autres outils destinés à vous aider à diagnostiquer des problèmes

   |         |          |
   | ---------------- |:-----|
   | [**Alertes**](../platform/alerts-overview.md)      |  Affichez, créez et gérez vos alertes. |
   | [**Métriques**](../platform/data-platform.md) | Visualisez et explorez vos données basées sur des métriques.    |
   | [**Journaux d’activité**](../platform/platform-logs-overview.md) | Événements au niveau abonnement qui se sont produits dans Azure.  |
   | [**Mise en correspondance d’applications**](../app/app-map.md) | Accédez à la topologie de votre application distribuée pour identifier des goulots d’étranglement ou des points de défaillance en lien avec les performances. |

## <a name="failures-and-performance"></a>Échecs et performances

Que se passe-t-il si vous avez remarqué que votre application s’exécute lentement ou que des utilisateurs ont signalé des erreurs ? Effectuer des recherches dans toutes vos ressources pour isoler des problèmes nécessite beaucoup de temps.

Les onglets **Performances** et **Échecs** simplifient ce processus en regroupant les affichages de diagnostic des performances et des échecs pour de nombreux types de ressources courants.

La plupart des types de ressources ouvrent une galerie de modèles de classeur Azure Monitor. Chaque classeur que vous créez peut être personnalisé, enregistré, partagé avec votre équipe, et réutilisé par la suite pour diagnostiquer des problèmes similaires.

### <a name="investigate-failures"></a>Examiner les échecs

Pour tester l’onglet Échecs, sélectionnez **Échecs** sous **Examiner** dans le menu à gauche.

Une fois votre sélection effectuée, la barre de menus à gauche change pour présenter de nouvelles options.

![Capture d’écran du volet d’affichage des échecs](./media/resource-group-insights/00004-failures.png)

Lors du choix d’App Service, une galerie de modèles de classeurs Azure Monitor s’affiche.

![Capture d’écran de la galerie de classeurs d’applications](./media/resource-group-insights/0005-failure-insights-workbook.png)

La sélection du modèle pour l’affichage d’informations sur les échecs a pour effet d’ouvrir le classeur.

![Capture d’écran de rapport d’échec](./media/resource-group-insights/0006-failure-visual.png)

Vous pouvez sélectionner n’importe quelle ligne. La sélection est ensuite affichée dans une vue graphique des détails.

![Capture d’écran de détails d’échec](./media/resource-group-insights/0007-failure-details.png)

Les classeurs éliminent le travail difficile de création de rapports personnalisés et de visualisations dans un format facilement consommable. Si certains utilisateurs ne veulent ajuster que les paramètres prédéfinis, les classeurs sont entièrement personnalisables.

Pour vous faire une idée de la façon dont ce classeur fonctionne en interne, sélectionnez **Modifier** dans la barre supérieure.

![Capture d’écran de l’option de modification supplémentaire](./media/resource-group-insights/0008-failure-edit.png)

Des zones d’édition **Modifier** s’affichent à proximité des différents éléments du classeur. Sélectionnez la zone d’édition **Modifier** sous le tableau des opérations.

![Capture d’écran des zones d’édition](./media/resource-group-insights/0009-failure-edit-graph.png)

Cela fait apparaître la requête de journal sous-jacente qui dirige la visualisation du tableau.

 ![Capture d’écran de la fenêtre de requête de journal](./media/resource-group-insights/0010-failure-edit-query.png)

Vous pouvez modifier la requête directement. Vous pouvez également l’utiliser comme référence et vous en servir lorsque vous concevez votre propre classeur paramétrable personnalisé.

### <a name="investigate-performance"></a>Examiner les performances

Les performances offrent leur propre galerie de classeurs. Pour App Service, le classeur prédéfini des performances de l’application offre l’affichage suivant :

 ![Capture d’écran de l’affichage des performances](./media/resource-group-insights/0011-performance.png)

Dans ce cas, si vous sélectionnez Modifier, vous verrez que cet ensemble de visualisations est alimenté par des métriques Azure Monitor.

 ![Capture d’écran de l’affichage des performances avec des métriques Azure](./media/resource-group-insights/0012-performance-metrics.png)

## <a name="troubleshooting"></a>Dépannage

### <a name="enabling-access-to-alerts"></a>Activation de l’accès aux alertes

Pour visualiser les alertes dans Azure Monitor pour les groupes de ressources, un utilisateur doté du rôle Propriétaire ou Contributeur pour cet abonnement doit ouvrir Azure Monitor pour les groupes de ressources pour n’importe quel groupe de ressources de l’abonnement. Cette opération autorise toute personne disposant d’un accès en lecture à visualiser les alertes dans Azure Monitor pour les groupes de ressources pour tous les groupes de ressources de l’abonnement. Si vous disposez du rôle Propriétaire ou Contributeur, actualisez cette page au bout de quelques minutes.

Azure Monitor pour les groupes de ressources s’appuie sur le système Azure Monitor Alerts Management pour récupérer l’état des alertes. La solution Alerts Management n’est pas configurée pour chaque groupe de ressources et abonnement par défaut, et ne peut être activée que par une personne disposant du rôle Propriétaire ou Contributeur. Elle est activable de deux manières :
* ouverture d’Azure Monitor pour les groupes de ressources pour n’importe quel groupe de ressources de l’abonnement ;
* accès à l’abonnement, clic sur **Fournisseurs de ressources**, puis clic sur **Register for Alerts Management** (S’inscrire à Alerts Management).

## <a name="next-steps"></a>Étapes suivantes

- [Classeurs Azure Monitor](../platform/workbooks-overview.md)
- [Azure Resource Health](../../service-health/resource-health-overview.md)
- [Alertes Azure Monitor](../platform/alerts-overview.md)
