---
title: Analyse des tendances dans Visual Studio | Microsoft Docs
description: Analysez, visualisez et examinez les tendances de vos données de télémétrie Application Insights dans Visual Studio.
ms.topic: conceptual
ms.date: 03/17/2017
ms.custom: vs-azure
ms.openlocfilehash: 9b78dceed3c2a25d6afedf5fca348726d7aafd3d
ms.sourcegitcommit: 50802bffd56155f3b01bfb4ed009b70045131750
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91932241"
---
# <a name="analyzing-trends-in-visual-studio"></a>Analyse des tendances dans Visual Studio
L’outil Tendances Application Insights visualise l’évolution des événements de télémétrie importants de votre application web au fil du temps, ce qui vous permet d’identifier rapidement les problèmes et les anomalies. En vous fournissant des informations de diagnostic plus détaillées, l’outil Tendances peut vous aider à améliorer les performances de votre application, à repérer les causes des exceptions et à découvrir les perspectives de vos événements personnalisés.

![Exemple de fenêtre Tendances](./media/visual-studio-trends/app-insights-trends-hero-750.png)

## <a name="configure-your-web-app-for-application-insights"></a>Configurer votre application web pour Application Insights

Si vous ne l’avez pas encore fait, [configurez votre application web pour Application Insights](./app-insights-overview.md). Elle pourra ainsi envoyer les données de télémétrie au portail Application Insights. L’outil Tendances lit les données de télémétrie à partir de ce portail.

L’outil Tendances Application Insights est disponible dans Visual Studio 2015 Update 3 et versions ultérieures.

## <a name="open-application-insights-trends"></a>Ouvrir Tendances Application Insights
Pour ouvrir la fenêtre Tendances Application Insights :

* À partir du bouton de la barre d’outils Application Insights, choisissez **Explorer les tendances de la télémétrie**, ou
* Dans le menu contextuel du projet, choisissez **Application Insights > Explorer les tendances de la télémétrie**, ou
* À partir de la barre de menus Visual Studio, choisissez **Affichage > Autres fenêtres > Tendances Application Insights**.

Une invite s’affiche et vous permet de sélectionner une ressource. Cliquez sur **Sélectionner une ressource**, connectez-vous avec un abonnement Azure, puis choisissez une ressource Application Insights dans la liste pour laquelle vous souhaitez analyser les tendances de la télémétrie.

## <a name="choose-a-trend-analysis"></a>Choisir une analyse de tendances
![Menu des types courants d’analyse des tendances](./media/visual-studio-trends/app-insights-trends-1-750.png)

Commencez par choisir l’une des cinq analyses de tendances courantes, chacune portant sur les données des dernières 24 heures :

* **Enquêtez sur les problèmes de performances de vos requêtes serveur** : requêtes effectuées auprès de votre service, regroupées par temps de réponse.
* **Analysez les erreurs de vos requêtes serveur** : requêtes effectuées auprès de votre service, regroupées par code de réponse HTTP.
* **Examinez les exceptions de votre application** : exceptions de votre service, regroupées par type d’exception.
* **Vérifiez les performances des dépendances de votre application** : services appelés par votre service, regroupés par temps de réponse.
* **Inspectez vos événements personnalisés** : événements personnalisés que vous avez configurés pour votre service, regroupés par type d’événement.

Ces analyses prédéfinies sont disponibles à partir du bouton **Afficher les types courants d’analyse de la télémétrie** situé dans le coin supérieur gauche de la fenêtre Tendances.

## <a name="visualize-trends-in-your-application"></a>Visualiser les tendances de votre application
L’outil Tendances Application Insights crée une visualisation de série chronologique à partir des données de télémétrie de votre application. Chaque visualisation de série chronologique affiche un type de données de télémétrie regroupées par propriété sur un intervalle de temps donné. Exemple : vous souhaitez afficher les requêtes serveur, regroupées en fonction du pays/de la région dont elles proviennent, au cours des dernières 24 heures. Dans cet exemple, chaque bulle sur la visualisation représente un nombre de requêtes serveur d’un pays/d’une région pendant une heure.

Utilisez les contrôles en haut de la fenêtre pour ajuster les types de données de télémétrie que vous affichez. Commencez par choisir les types de données de télémétrie souhaités :

* **Type de télémétrie** : requêtes serveur, exceptions, dépendances ou événements personnalisés.
* **Intervalle de temps** : plage comprise entre les 30 dernières minutes et les 3 derniers jours.
* **Grouper par** : type d’exception, ID du problème, pays/région, etc.

Cliquez ensuite sur **Analyser la télémétrie** pour exécuter la requête.

Pour naviguer entre les bulles de la visualisation :

* Cliquez pour sélectionner une bulle. Les filtres situés en bas de la fenêtre sont alors mis à jour et seuls les événements qui se sont produits pendant une période spécifique sont affichés.
* Double-cliquez sur une bulle pour accéder à l’outil de recherche et afficher tous les événements de télémétrie individuels qui se sont produits pendant cette période.
* Cliquez sur une bulle tout en appuyant sur Ctrl pour la désélectionner dans la visualisation.

> [!TIP]
> Les outils Tendances et Search fonctionnent ensemble pour vous permettre d’identifier les causes des problèmes de votre service parmi des milliers d’événements de télémétrie. Par exemple, si un après-midi vos clients remarquent que votre application est moins réactive, utilisez l’outil Tendances. Analysez les requêtes effectuées auprès de votre service au cours des dernières heures, regroupées par temps de réponse. Vérifiez si un groupe de requêtes lentes inhabituellement volumineux existe, puis double-cliquez sur cette bulle pour accéder à l’outil Search, en filtrant votre recherche sur les événements de requête. Dans Search, vous pouvez examiner le contenu de ces requêtes et accéder au code impliqué pour résoudre le problème.
> 
> 

## <a name="filter"></a>Filtrer
Découvrez des tendances plus spécifiques avec les contrôles de filtre situés au bas de la fenêtre. Pour appliquer un filtre, cliquez sur son nom. Vous pouvez rapidement basculer entre différents filtres pour découvrir les tendances qui peuvent être masquées dans une dimension particulière de vos données de télémétrie. Si vous appliquez un filtre dans une dimension, par exemple Type d’exception, vous pouvez toujours cliquer sur les filtres des autres dimensions même s’ils apparaissent grisés. Pour annuler l’application d’un filtre, cliquez à nouveau dessus. Cliquez tout en appuyant sur Ctrl pour sélectionner plusieurs filtres dans la même dimension.

![Filtres de tendances](./media/visual-studio-trends/TrendsFiltering-750.png)

Comment faire pour appliquer plusieurs filtres ? 

1. Appliquez le premier filtre. 
2. Cliquez sur le bouton **Apply selected filters and query again (Appliquer les filtres sélectionnés et réexécuter la requête)** correspondant à la dimension de votre premier filtre. La requête portant sur vos données de télémétrie est réexécutée uniquement pour les événements qui correspondent au premier filtre. 
3. Appliquez un second filtre. 
4. Répétez la procédure pour trouver les tendances dans des sous-ensembles spécifiques des données de télémétrie. Exemple : les requêtes serveur nommées « GET Home/Index » *et* provenant d’Allemagne *et* ayant reçu le code de réponse 500. 

Pour annuler l’application d’un de ces filtres, cliquez sur le bouton **Remove selected filters and query again (Supprimer les filtres sélectionnés et réexécuter la requête)** de la dimension.

![Filtres multiples](./media/visual-studio-trends/TrendsFiltering2-750.png)

## <a name="find-anomalies"></a>Rechercher des anomalies
L’outil Tendances peut mettre en évidence des bulles d’événements anormales par rapport aux autres bulles d’une même série chronologique. Dans la liste déroulante Type d’affichage, choisissez **Décomptes dans l’intervalle de planification (anomalies en surbrillance)** ou **Pourcentages dans l’intervalle de planification (anomalies en surbrillance)** . Les bulles rouges sont anormales. Les anomalies sont définies comme des bulles dont les décomptes/pourcentages excèdent 2,1 fois l’écart type des décomptes/pourcentages des deux dernières périodes (48 heures si vous affichez les dernières 24 heures, etc.).

![Les points colorés indiquent des anomalies](./media/visual-studio-trends/TrendsAnomalies-750.png)

> [!TIP]
> La mise en surbrillance des anomalies est particulièrement utile pour détecter les valeurs hors norme dans les séries chronologiques des petites bulles dont les tailles pourraient sinon sembler identiques.  
> 
> 

## <a name="next-steps"></a><a name="next"></a>Étapes suivantes
* **[Utilisation d’Application Insights dans Visual Studio](./visual-studio.md)** . Rechercher les données de télémétrie, voir les données dans CodeLens et configurer Application Insights. le tout dans Visual Studio. 
* **[Utilisation du portail Application Insights](./overview-dashboard.md)** . Tableaux de bord, puissants outils de diagnostic et d’analyse, alertes, mappage direct des dépendances de votre application et exportation des données de télémétrie. 

