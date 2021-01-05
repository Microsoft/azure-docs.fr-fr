---
title: Présentation de Log Analytics dans Azure Monitor
description: Décrit Log Analytics, un outil du Portail Azure utilisé pour modifier et exécuter des requêtes de journal pour analyser des données dans des journaux Azure Monitor.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 10/04/2020
ms.openlocfilehash: ba27739ff9e9d992ffe6202629a1db19604b1409
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96186114"
---
# <a name="overview-of-log-analytics-in-azure-monitor"></a>Présentation de Log Analytics dans Azure Monitor
Log Analytics est un outil du Portail Azure utilisé pour modifier et exécuter des requêtes de journal avec des données dans des journaux Azure Monitor. Vous pouvez écrire une requête simple qui retourne un jeu d’enregistrements, puis utiliser les fonctionnalités de Log Analytics pour les trier, les filtrer et les analyser. Vous pouvez également écrire une requête plus avancée pour effectuer une analyse statistique et visualiser les résultats dans un graphique afin d’identifier une tendance particulière. Que vous utilisiez les résultats de vos requêtes de manière interactive ou avec d’autres fonctionnalités d’Azure Monitor, comme les alertes de requête de journal ou les classeurs, Log Analytics est l’outil que vous allez utiliser pour les écrire et les tester. 


> [!TIP]
> Cet article fournit une description de Log Analytics et de chacune de ses fonctionnalités. Si vous souhaitez accéder directement à un tutoriel, consultez le [Tutoriel Log Analytics](./log-analytics-tutorial.md).



## <a name="starting-log-analytics"></a>Démarrage de Log Analytics
Démarrez Log Analytics à partir de **Journaux** dans le menu **Azure Monitor** du Portail Azure. Vous verrez également cette option dans le menu de la plupart des ressources Azure. Quel que soit l’emplacement à partir duquel vous le démarrez, il s’agit du même outil Log Analytics. Le menu que vous utilisez pour démarrer Log Analytics détermine les données qui seront disponibles. Si vous le démarrez à partir du menu **Azure Monitor** ou d’**espaces de travail Log Analytics**, vous avez accès à tous les enregistrements d’un espace de travail. Si vous sélectionnez **Journaux** à partir d’un autre type de ressource, vos données sont limitées aux données de journal pour cette ressource. Pour plus d’informations, consultez [Étendue de requête de journal et intervalle de temps dans la fonctionnalité Log Analytics d’Azure Monitor](scope.md).

[![Démarrer Log Analytics](media/log-analytics-overview/start-log-analytics.png)](media/log-analytics-overview/start-log-analytics.png#lightbox)

Lorsque vous démarrez Log Analytics, la première chose que vous verrez est une boîte de dialogue avec des [requêtes d’exemple](example-queries.md). Celles-ci sont classées par solution et vous pouvez parcourir ou rechercher les requêtes qui correspondent à vos besoins spécifiques. Vous pouvez en trouver une qui effectue exactement ce dont vous avez besoin, ou en charger une dans l’éditeur et la modifier en fonction de vos besoins. L’exploration des requêtes d’exemple est en fait un excellent moyen d’apprendre à écrire vos propres requêtes. Bien entendu, si vous souhaitez commencer avec un script vide et l’écrire vous-même, vous pouvez fermer les exemples. Cliquez simplement sur **Requêtes** en haut de l’écran si vous voulez les retrouver.

## <a name="log-analytics-interface"></a>Interface de Log Analytics
L’image suivante identifie les différents composants de Log Analytics.

[![Log Analytics](media/log-analytics-overview/log-analytics.png)](media/log-analytics-overview/log-analytics.png#lightbox)

### <a name="1-top-action-bar"></a>1. Barre d’action supérieure
Contrôles pour le travail sur la requête dans la fenêtre de requête.

| Option | Description |
|:---|:---|
| Étendue | Spécifie l’étendue des données utilisées pour la requête. Il peut s’agir de toutes les données d’un espace de travail Log Analytics ou de données pour une ressource particulière dans plusieurs espaces de travail. Voir [Étendue de requête](scope.md). |
| Bouton Exécuter | Cliquez sur cette option pour exécuter la requête sélectionnée dans la fenêtre de requête. Pour exécuter une cellule, vous pouvez également appuyer sur Maj + Entrée. |
| Sélecteur d’heure | Sélectionnez l’intervalle de temps pour les données disponibles pour la requête. Cette condition est remplacée si vous incluez un filtre de temps dans la requête. Voir [Étendue de requête de journal et intervalle de temps dans la fonctionnalité Log Analytics d’Azure Monitor](scope.md). |
| Bouton Enregistrer | Enregistrez la requête dans l’Explorateur de requêtes de l’espace de travail. |
 bouton Copier | Copiez un lien vers la requête, le texte de la requête ou les résultats de la requête dans le presse-papiers. |
| Bouton Nouvelle règle d’alerte | Créez un nouvel onglet avec une requête vide. |
| Bouton Exporter | Exportez les résultats de la requête dans un fichier CSV ou la requête au format de langage de formule Power Query pour l’utiliser avec Power BI. |
| Bouton Épingler au tableau de bord | Ajoutez les résultats de la requête à un tableau de bord Azure. |
| Bouton Formater la requête | Arrange le texte sélectionné pour une meilleure lisibilité. |
| Bouton d’exemples de requêtes | Ouvrez la boîte de dialogue d’exemples de requêtes qui s’affiche lorsque vous ouvrez pour la première fois Log Analytics. |
| Bouton de l’Explorateur de requêtes | Ouvrez l’**Explorateur de requêtes**, qui fournit l’accès aux requêtes enregistrées dans l’espace de travail. |


### <a name="2-sidebar"></a>2. Barre latérale
Répertorie les tables dans l’espace de travail, les exemples de requêtes et les options de filtre pour la requête actuelle.

| Onglet | Description |
|:---|:---|
| Tables | Répertorie les tables qui font partie de l’étendue sélectionnée. Sélectionnez **Group by** pour modifier le regroupement des tables. Pointez sur un nom de table pour afficher une boîte de dialogue avec une description de la table et des options permettant d’afficher sa documentation et d’afficher un aperçu de ses données. Développez une table pour afficher ses colonnes. Double-cliquez sur un nom de table ou de colonne pour l’ajouter à la requête. |
| Requêtes | Liste d’exemples de requêtes que vous pouvez ouvrir dans la fenêtre de requête. Il s’agit de la même liste que celle qui s’affiche lorsque vous ouvrez Log Analytics. Sélectionnez **Group by** pour modifier le regroupement des requêtes. Double-cliquez sur une requête pour l’ajouter à la fenêtre de requête ou pointez dessus pour d’autres options. |
| Filtrer | Crée des options de filtre en fonction des résultats d’une requête. Une fois que vous avez exécuté une requête, les colonnes s’affichent avec des valeurs différentes de celles des résultats. Sélectionnez une ou plusieurs valeurs, puis cliquez sur **Appliquer et exécuter** pour ajouter une commande **where** à la requête et l’exécuter à nouveau. |

### <a name="3-query-window"></a>3. Fenêtre de requête
La fenêtre de requête est l’endroit où vous modifiez votre requête. Elle utilise IntelliSense pour les commandes KQL et un code couleur pour améliorer la lisibilité. Cliquez sur le **+** en haut de la fenêtre pour ouvrir un autre onglet.

Une seule fenêtre peut inclure plusieurs requêtes. Une requête ne peut pas inclure de lignes vides, vous pouvez donc séparer plusieurs requêtes dans une fenêtre par une ou plusieurs lignes vides. La requête actuelle est celle sur laquelle le curseur est positionné.

Pour exécuter la requête en cours, cliquez sur le bouton **Exécuter** ou appuyez sur Maj + Entrée.

### <a name="4-results-window"></a>4. Fenêtre de résultats
Les résultats de la requête sont affichés dans la fenêtre de résultats. Par défaut, les résultats sont affichés dans une table. Pour afficher sous forme de graphique, sélectionnez **Graphique** dans la fenêtre de résultats, ou ajoutez une commande **render** à votre requête.

#### <a name="results-view"></a>vue Résultats
Affiche les résultats d’une requête dans une table organisée en colonnes et en lignes. Cliquez sur la gauche d’une ligne pour développer ses valeurs. Cliquez sur la liste déroulante **Colonnes** pour modifier la liste des colonnes. Triez les résultats en cliquant sur un nom de colonne. Filtrez les résultats en cliquant sur l’entonnoir en regard d’un nom de colonne. Désactivez les filtres et réinitialisez le tri en exécutant à nouveau la requête.

Sélectionnez **Regrouper les colonnes** pour afficher la barre de regroupement au-dessus des résultats de la requête. Regroupez les résultats en fonction d’une colonne en la faisant glisser vers la barre. Créez des groupes imbriqués dans les résultats en ajoutant des colonnes supplémentaires. 

#### <a name="chart-view"></a>Vue de graphique
Affiche les résultats sous la forme d’un des différents types de graphiques disponibles. Vous pouvez spécifier le type de graphique dans une commande **render** de votre requête ou le sélectionner dans la liste déroulante **Type de visualisation**.

| Option | Description |
|:---|:---|
| **Type de visualisation** | Type de graphique à afficher. |
| **Axe des abscisses (x)** | Colonne dans les résultats à utiliser pour l’axe X 
| **Axe des ordonnées (y)** | Colonne dans les résultats à utiliser pour l’axe Y. Il s’agit généralement d’une colonne numérique. |
| **Fractionner par** | Colonne dans les résultats qui définit la série dans le graphique. Une série est créée pour chaque valeur de la colonne. |
| **Agrégation** | Type d’agrégation à effectuer sur les valeurs numériques sur l’axe des Y. |

## <a name="relationship-to-azure-data-explorer"></a>Relation avec Azure Data Explorer
Si vous êtes déjà familiarisé avec l’interface utilisateur web d’Azure Data Explorer, Log Analytics vous paraîtra sans doute familier. C’est parce qu’il s’appuie sur Azure Data Explorer et utilise le même langage de requête Kusto (KQL). Log Analytics ajoute des fonctionnalités spécifiques à Azure Monitor, comme le filtrage par intervalle de temps et la possibilité de créer une règle d’alerte à partir d’une requête. Les deux outils comprennent un explorateur qui vous permet d’analyser la structure des tables disponibles, mais l’interface utilisateur web d’Azure Data Explorer fonctionne principalement avec des tables dans les bases de données Azure Data Explorer, tandis que Log Analytics fonctionne avec des tables d’un espace de travail Log Analytics. 

## <a name="next-steps"></a>Étapes suivantes
- Suivez un [didacticiel sur l’utilisation de Log Analytics dans le portail Azure](./log-analytics-tutorial.md).
- Suive un [didacticiel sur l’écriture de requêtes](get-started-queries.md).