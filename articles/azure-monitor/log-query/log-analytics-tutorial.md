---
title: Tutoriel Log Analytics
description: Découvrez dans ce tutoriel comment utiliser les fonctionnalités de Log Analytics dans Azure Monitor pour générer et exécuter une requête de journal et analyser ses résultats dans le portail Azure.
ms.subservice: logs
ms.topic: tutorial
author: bwren
ms.author: bwren
ms.date: 10/07/2020
ms.openlocfilehash: 16292999bf8c01615a9125ffaa9f93fc4b2a8ec2
ms.sourcegitcommit: 1bf144dc5d7c496c4abeb95fc2f473cfa0bbed43
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95756211"
---
# <a name="log-analytics-tutorial"></a>Tutoriel Log Analytics
Log Analytics est un outil du portail Azure permettant de modifier et d’exécuter des requêtes de journal à partir de données collectées par les journaux Azure Monitor et d’analyser leurs résultats de manière interactive. Vous pouvez utiliser des requêtes Log Analytics pour récupérer des enregistrements correspondant à des critères particuliers, identifier des tendances, analyser des modèles et fournir divers insights sur vos données. 

Ce tutoriel vous guide dans l’interface Log Analytics, présente quelques requêtes de base et vous montre comment tirer parti des résultats. Vous allez apprendre les opérations suivantes :

> [!div class="checklist"]
> * Comprendre le schéma des données de journal
> * Écrire et exécuter des requêtes simples, et modifier l’intervalle de temps pour les requêtes
> * Filtrer, trier et regrouper les résultats de requête
> * Afficher, modifier et partager des visuels des résultats de requête
> * Charger, exporter et copier des requêtes et des résultats

> [!IMPORTANT]
> Ce tutoriel utilise des fonctionnalités de Log Analytics pour générer et exécuter une requête au lieu d’utiliser la requête elle-même. Vous tirerez parti des fonctionnalités de Log Analytics pour générer une requête et utiliser un autre exemple de requête. Lorsque vous êtes prêt à apprendre la syntaxe des requêtes et à commencer à modifier directement la requête, parcourez le [tutoriel sur le langage de requête Kusto (KQL)](/azure/data-explorer/kusto/query/tutorial?pivots=azuremonitor). Ce tutoriel présente plusieurs exemples de requêtes que vous pouvez modifier et exécuter dans Log Analytics, en tirant parti de plusieurs des fonctionnalités que vous allez découvrir dans ce tutoriel.


## <a name="prerequisites"></a>Prérequis
Ce tutoriel utilise l’[environnement de démonstration Log Analytics](https://ms.portal.azure.com/#blade/Microsoft_Azure_Monitoring_Logs/DemoLogsBlade), qui comprend de nombreux exemples de données prenant en charge les exemples de requêtes. Vous pouvez également utiliser votre propre abonnement Azure, mais vous ne disposerez peut-être pas de données dans les mêmes tables.

## <a name="open-log-analytics"></a>Ouvrir Log Analytics
Ouvrez l’[environnement de démonstration Log Analytics](https://ms.portal.azure.com/#blade/Microsoft_Azure_Monitoring_Logs/DemoLogsBlade) ou sélectionnez **Journaux** dans le menu Azure Monitor de votre abonnement. Cela permet de définir un espace de travail Log Analytics comme étendue initiale, ce qui signifie que votre requête sélectionnera parmi toutes les données de cet espace de travail. Si vous sélectionnez **Journaux** dans le menu d’une ressource Azure, l’étendue est définie sur les seuls enregistrements de cette ressource. Pour plus d’informations sur l’étendue, consultez [Étendue de requête de journal](scope.md).

Vous pouvez voir l’étendue dans l’angle supérieur gauche de l’écran. Si vous utilisez votre propre environnement, vous verrez une option permettant de sélectionner une autre étendue, mais cette option n’est pas disponible dans l’environnement de démonstration.

[![Étendue de requête](media/log-analytics-tutorial/scope.png)](media/log-analytics-tutorial/scope.png#lightbox)

## <a name="table-schema"></a>Schéma de table
Le côté gauche de l’écran comprend l’onglet **Tables**, qui vous permet d’inspecter les tables disponibles dans l’étendue actuelle. Celles-ci sont regroupées par **Solution** par défaut, mais vous pouvez modifier leur regroupement ou les filtrer. 

Développez la solution **Gestion des journaux** et recherchez la table **AzureActivity**. Vous pouvez développer la table pour afficher son schéma, ou pointer sur son nom pour afficher des informations supplémentaires à son sujet. 

[![Affichage de tables](media/log-analytics-tutorial/table-details.png)](media/log-analytics-tutorial/table-details.png#lightbox)

Cliquez sur **En savoir plus** pour accéder à la référence des tables qui documente chaque table et ses colonnes. Cliquez sur **Aperçu des données** pour jeter un œil à quelques enregistrements récents dans la table. Cela peut être utile pour s’assurer qu’il s’agit bien des données que vous attendez, avant d’exécuter une requête avec elles.

[![Exemple de données](media/log-analytics-tutorial/sample-data.png)](media/log-analytics-tutorial/sample-data.png#lightbox)

## <a name="write-a-query"></a>Écrivez votre requête.
Commençons par écrire une requête à l’aide de la table **AzureActivity**. Double-cliquez sur son nom pour l’ajouter à la fenêtre de requête. Vous pouvez également taper directement dans la fenêtre et même faire en sorte qu’IntelliSense vous aide à compléter les noms des tables dans l’étendue actuelle et les commandes KQL.

Il s’agit de la requête la plus simple que nous pouvons écrire. Elle retourne simplement tous les enregistrements d’une table. Exécutez-la en cliquant sur le bouton **Exécuter** ou en appuyant sur Maj+Entrée avec le curseur positionné n’importe où dans le texte de la requête.

[![Résultats de la requête](media/log-analytics-tutorial/query-results.png)](media/log-analytics-tutorial/query-results.png#lightbox)

Vous pouvez constater que nous avons des résultats. Le nombre d’enregistrements retournés par la requête est affiché dans le coin inférieur droit. 

## <a name="filter"></a>Filtrer

Ajoutons un filtre à la requête pour réduire le nombre d’enregistrements retournés. Sélectionnez l’onglet **Filtrer** dans le volet gauche. Cela présente différentes colonnes dans les résultats de la requête, que vous pouvez utiliser pour filtrer les résultats. Les valeurs les plus élevées de ces colonnes sont affichées avec le nombre d’enregistrements ayant cette valeur. Cliquez sur **Administrative** sous **CategoryValue**, puis sur **Appliquer et Exécuter**. 

[![Volet Requête](media/log-analytics-tutorial/query-pane.png)](media/log-analytics-tutorial/query-pane.png#lightbox)

Une instruction **where** est ajoutée à la requête avec la valeur que vous avez sélectionnée. Les résultats incluent désormais uniquement les enregistrements ayant cette valeur, ce qui vous permet de voir que le nombre d’enregistrements est réduit.

[![Résultats de la requête filtrés](media/log-analytics-tutorial/query-results-filter-01.png)](media/log-analytics-tutorial/query-results-filter-01.png#lightbox)


## <a name="time-range"></a>Plage temporelle
Toutes les tables d’un espace de travail Log Analytics ont une colonne nommée **TimeGenerated** qui correspond à l’heure de création de l’enregistrement. Toutes les requêtes ont une plage de temps qui limite les résultats aux enregistrements ayant une valeur **TimeGenerated** comprise dans cette plage. La plage de temps peut être définie dans la requête ou avec le sélecteur en haut de l’écran.

Par défaut, la requête retourne les enregistrements des dernières 24 heures. Sélectionnez la liste déroulante **Plage de temps** et choisissez **7 jours**. Cliquez à nouveau sur **Exécuter** pour retourner les résultats. Vous pouvez constater que des résultats sont retournés, mais nous avons un message indiquant que nous ne voyons pas tous les résultats. Cela est dû au fait que Log Analytics peut retourner un maximum de 10 000 enregistrements, et que notre requête en a retourné davantage. 

[![Plage temporelle](media/log-analytics-tutorial/query-results-max.png)](media/log-analytics-tutorial/query-results-max.png#lightbox)


## <a name="multiple-query-conditions"></a>Plusieurs conditions de requête
Nous allons réduire les résultats en ajoutant une autre condition de filtre. Une requête peut inclure un nombre quelconque de filtres pour cibler exactement le jeu d’enregistrements souhaité. Sélectionnez **Réussite** sous **ActivityStatusValue**, puis cliquez sur **Appliquer et exécuter**. 

[![Résultats de la requête avec plusieurs filtres](media/log-analytics-tutorial/query-results-filter-02.png)](media/log-analytics-tutorial/query-results-filter-02.png#lightbox)


## <a name="analyze-results"></a>Analyser les résultats
En plus de vous aider à écrire et à exécuter des requêtes, Log Analytics offre des fonctionnalités permettant d’exploiter les résultats. Commencez par développer un enregistrement pour afficher les valeurs de toutes ses colonnes.

[![Développer un enregistrement](media/log-analytics-tutorial/expand-record.png)](media/log-analytics-tutorial/expand-record.png#lightbox)

Cliquez sur le nom d’une colonne pour trier les résultats en fonction de cette colonne. Cliquez sur l’icône de filtre en regard de celle-ci pour spécifier une condition de filtre. Cela est similaire à l’ajout d’une condition de filtre à la requête elle-même, sauf que ce filtre est effacé si la requête est réexécutée. Appliquez cette méthode si vous souhaitez analyser rapidement un jeu d’enregistrements dans le cadre de l’analyse interactive.

Par exemple, définissez un filtre sur la colonne **CallerIpAddress** pour limiter les enregistrements à un seul appelant. 

[![Filtre des résultats de la requête](media/log-analytics-tutorial/query-results-filter.png)](media/log-analytics-tutorial/query-results-filter.png#lightbox)

Au lieu de filtrer les résultats, vous pouvez regrouper les enregistrements par une colonne particulière. Effacez le filtre que vous venez de créer, puis activez le curseur **Regrouper les colonnes**. 

[![Regrouper les colonnes](media/log-analytics-tutorial/query-results-group-columns.png)](media/log-analytics-tutorial/query-results-group-columns.png#lightbox)

À présent, faites glisser la colonne **CallerIpAddress** sur la ligne de regroupement. Les résultats sont désormais organisés d’après cette colonne, et vous pouvez réduire chaque groupe pour vous aider dans votre analyse.

[![Résultats de la requête regroupés](media/log-analytics-tutorial/query-results-grouped.png)](media/log-analytics-tutorial/query-results-grouped.png#lightbox)

## <a name="work-with-charts"></a>Utiliser des graphiques
Examinons une requête qui utilise des données numériques que nous pouvons afficher dans un graphique. Au lieu de créer une requête, nous allons sélectionner un exemple de requête.

Dans le volet gauche, cliquez sur **Requêtes**. Ce volet comprend des exemples de requêtes que vous pouvez ajouter à la fenêtre de requête. Si vous utilisez votre propre espace de travail, vous devez disposer de diverses requêtes dans plusieurs catégories, mais si vous utilisez l’environnement de démonstration vous ne verrez qu’une seule catégorie **Espaces de travail Log Analytics**. Développez-la pour afficher les requêtes de la catégorie.

Cliquez sur la requête nommée **Nombre de requêtes par ResponseCode**. Cette opération ajoute la requête à la fenêtre de requête. Notez que la nouvelle requête est séparée de l’autre par une ligne vide. Une requête en KQL se termine lorsqu’elle rencontre une ligne vide. Elles sont donc considérées comme des requêtes distinctes. 

[![Nouvelle requête](media/log-analytics-tutorial/example-query.png)](media/log-analytics-tutorial/example-query.png#lightbox)

La requête active est celle sur laquelle le curseur est positionné. Vous pouvez voir que la première requête est mise en surbrillance, indiquant qu’il s’agit de la requête active. Cliquez n’importe où dans la nouvelle requête pour la sélectionner, puis cliquez sur le bouton **Exécuter** pour l’exécuter.

[![Graphique des résultats de la requête](media/log-analytics-tutorial/example-query-output-chart.png)](media/log-analytics-tutorial/example-query-output-chart.png#lightbox)

Notez que cette sortie est un graphique, et non un tableau comme avec la dernière requête. Cela est dû au fait que l’exemple de requête utilise une commande [render](/azure/data-explorer/kusto/query/renderoperator?pivots=azuremonitor) à la fin. Notez qu’il existe différentes options pour manipuler le graphique, par exemple pour le remplacer par un autre type.

Essayez de sélectionner **Résultats** pour afficher la sortie de la requête sous forme de tableau. 

[![Tableau des résultats de la requête](media/log-analytics-tutorial/example-query-output-table.png)](media/log-analytics-tutorial/example-query-output-table.png#lightbox)



## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous savez comment utiliser Log Analytics, suivez le tutoriel sur l’utilisation des requêtes de journal.
> [!div class="nextstepaction"]
> [Écrire des requêtes de journal dans Azure Monitor](get-started-queries.md)
