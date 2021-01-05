---
title: Utiliser le serveur d’historique Spark étendu pour déboguer des applications
description: Utilisez le serveur d’historique Spark étendu pour déboguer et diagnostiquer des applications Spark dans Azure Synapse Analytics.
services: synapse-analytics
author: euangMS
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: spark
ms.date: 10/15/2020
ms.author: euang
ms.reviewer: euang
ms.openlocfilehash: 94c30cad1e09a01686a9042a6271a152d0f433fd
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95995384"
---
# <a name="use-extended-apache-spark-history-server-to-debug-and-diagnose-apache-spark-applications"></a>Utiliser le serveur d’historique Apache Spark étendu pour déboguer et diagnostiquer des applications Apache Spark

Cet article vous guide dans l’utilisation du serveur d’historique Apache Spark étendu pour déboguer et diagnostiquer des applications Spark terminées et en cours d’exécution.

L’extension comprend les onglets Data (Données), Graph (Graphique) et Diagnosis (Diagnostic). Sous l’onglet **Data** (Données), vous pouvez vérifier les données d’entrée et de sortie du travail Spark. L’onglet **Graph** (Graphique) affiche le flux de données et la relecture du graphique du travail. L’onglet **Diagnosis** (Diagnostic) affiche les informations **Data Skew** (Asymétrie des données), **Time Skew** (Asymétrie temporelle) et **Executor Usage Analysis** (Analyse de l’utilisation de l’exécuteur).

## <a name="access-the-apache-spark-history-server"></a>Accéder au serveur d’historique Apache Spark

Le serveur d’historique Apache Spark est l’interface utilisateur web pour les applications Spark terminées et en cours d’exécution. Vous pouvez ouvrir l’interface web du serveur d’historique Apache Spark à partir d’Azure Synapse Analytics.

### <a name="open-the-spark-history-server-web-ui-from-apache-spark-applications-node"></a>Ouvrir l’interface utilisateur web du serveur d’historique Spark à partir du nœud Applications Apache Spark

1. Ouvrez [Azure Synapse Analytics](https://web.azuresynapse.net/).

2. Sélectionnez **Superviser**, puis **Applications Apache Spark**.

    ![Sélectionnez Superviser, puis sélectionnez une application Spark.](./media/apache-spark-history-server/click-monitor-spark-application.png)

3. Sélectionnez une application, puis ouvrez la fenêtre **Requête de journal** en la sélectionnant.

    ![Ouvrez la fenêtre Requête de journal.](./media/apache-spark-history-server/open-application-window.png)

4. Sélectionnez **Serveur d’historique Spark**. L’interface utilisateur web du serveur d’historique Spark s’affiche.

    ![Ouvrez le serveur d’historique Spark.](./media/apache-spark-history-server/open-spark-history-server.png)

### <a name="open-the-spark-history-server-web-ui-from-data-node"></a>Ouvrir l’interface utilisateur web du serveur d’historique Spark à partir du nœud Données

1. Dans votre bloc-notes Azure Synapse Studio, sélectionnez **Serveur d’historique Spark** dans la cellule de sortie de l’exécution du travail ou dans le panneau d’état au bas du document du bloc-notes. Sélectionnez **Détails de la session**.

   ![Lancer le serveur d’historique Spark 1](./media/apache-spark-history-server/launch-history-server2.png "Lancer le serveur d’historique Spark")

2. Sélectionnez **Serveur d’historique Spark** dans le panneau coulissant.

   ![Lancer le serveur d’historique Spark 2](./media/apache-spark-history-server/launch-history-server.png "Lancer le serveur d’historique Spark")

## <a name="explore-the-data-tab-in-spark-history-server"></a>Explorer l’onglet Données dans le serveur d’historique Spark

Sélectionnez l’ID du travail à afficher. Sélectionnez ensuite **Données** dans le menu des outils pour ouvrir la vue de données. Cette section montre comment effectuer diverses tâches sous l’onglet Données.

* Vérifiez les entrées (**Inputs**), sorties (**Outputs**) et opérations de table (**Table Operations**) en sélectionnant les onglets correspondants.

    ![Données pour l’application Spark - Onglets](./media/apache-spark-history-server/apache-spark-data-tabs.png)

* Copiez toutes les lignes en sélectionnant **Copier**.

    ![Données pour l’application Spark - Copier](./media/apache-spark-history-server/apache-spark-data-copy.png)

* Enregistrez toutes les données dans un fichier CSV en sélectionnant **csv**.

    ![Données pour l’application Spark - Enregistrer](./media/apache-spark-history-server/apache-spark-data-save.png)

* Effectuez une recherche en entrant des mots clés dans le champ **Recherche**. Les résultats de recherche s’affichent immédiatement.

    ![Données pour l’application Spark - Rechercher](./media/apache-spark-history-server/apache-spark-data-search.png)

* Sélectionnez l’en-tête de colonne pour trier la table, le signe plus pour développer une ligne et afficher plus de détails, ou le signe moins pour réduire une ligne.

    ![Données pour l’application Spark - Table](./media/apache-spark-history-server/apache-spark-data-table.png)

* Téléchargez un fichier unique en sélectionnant **Téléchargement partiel**. Le fichier sélectionné est téléchargé en local. Si le fichier n’existe plus, un nouvel onglet affiche un message d’erreur.

    ![Données pour l’application Spark - Télécharger](./media/apache-spark-history-server/sparkui-data-download-row.png)

* Pour copier un chemin complet ou relatif, sélectionnez l’option **Copier le chemin complet** ou l’option **Copier le chemin relatif** qui se développent à partir du menu déroulant. Pour les fichiers Azure Data Lake Storage, l’option **Ouvrir dans l’Explorateur Stockage Azure** lance l’Explorateur Stockage Azure et localise le dossier lorsque vous êtes connecté.

    ![Données pour l’application Spark - Copier le chemin](./media/apache-spark-history-server/sparkui-data-copy-path.png)

* Sélectionnez les numéros de page sous le tableau pour parcourir les pages quand les lignes sont trop nombreuses pour tenir sur une seule page.

    ![Données pour l’application Spark - Page](./media/apache-spark-history-server/apache-spark-data-page.png)

* Pointez sur le point d’interrogation en regard de **Données** pour afficher l’info-bulle, ou sélectionnez le point d’interrogation pour obtenir des informations supplémentaires.

    ![Données pour l’application Spark - Plus d’informations](./media/apache-spark-history-server/sparkui-data-more-info.png)

* Pour signaler des problèmes, envoyez des commentaires en sélectionnant **Envoyez-nous vos commentaires**.

    ![Graphe Spark - Envoyez-nous vos commentaires](./media/apache-spark-history-server/sparkui-graph-feedback.png)

## <a name="graph-tab-in-apache-spark-history-server"></a>Onglet Graph dans le serveur d’historique Apache Spark

Sélectionnez l’ID du travail à afficher. Ensuite, sélectionnez **Graph** dans le menu des outils pour obtenir l’affichage du graphique du travail.

### <a name="overview"></a>Vue d’ensemble

Vous pouvez voir une vue d’ensemble de votre travail dans le graphique du travail ainsi généré. Par défaut, le graphique affiche tous les travaux. Vous pouvez filtrer cet affichage par **ID de travail**.

![Application Spark - ID du travail - Graphe du travail](./media/apache-spark-history-server/apache-spark-graph-jobid.png)

### <a name="display"></a>Afficher

Par défaut, l’affichage **Progress** (Progression) est sélectionné. Vous pouvez vérifier le flux de données en sélectionnant **Read** (Lues) ou **Written** (Écrites) dans le menu déroulant **Display** (Affichage).

![Application Spark - Graphe du travail - Affichage](./media/apache-spark-history-server/sparkui-graph-display.png)

Le nœud Graph affiche les couleurs indiquées dans la légende de la carte thermique.

![Application Spark - Graphe du travail - Carte thermique](./media/apache-spark-history-server/sparkui-graph-heatmap.png)

### <a name="playback"></a>Lecture

Pour lire le travail, sélectionnez **Playback** (Lecture). Vous pouvez sélectionner **Stop** (Arrêter) à tout moment. Les couleurs des travaux affichent différents états lors de la relecture :

|Couleur|Signification|
|-|-|
|Vert|Succeeded : le travail s’est correctement effectué.|
|Orange|Réessayé : instances de tâches qui ont échoué, mais qui n’affectent pas le résultat final du travail. Ces tâches comportaient des instances dupliquées ou renouvelées qui peuvent réussir ultérieurement.|
|Bleu|Running : la tâche est en cours d’exécution.|
|White|En attente ou ignoré : la tâche est en attente d’exécution, ou la phase a été ignorée.|
|Rouge|Échoué : la tâche a échoué.|

L’image suivante montre les couleurs d’état verte, orange et bleue.

![Application Spark - Exemple de couleur du graphe de travail - Exécution](./media/apache-spark-history-server/sparkui-graph-color-running.png)

L’image suivante montre les couleurs d’état verte et blanche.

![Application Spark et exemple de couleur de graphe de travail, ignorer](./media/apache-spark-history-server/sparkui-graph-color-skip.png)

L’image suivante montre les couleurs d’état rouge et verte.

![Application Spark - Exemple de couleur du graphe de travail - Échec](./media/apache-spark-history-server/sparkui-graph-color-failed.png)

> [!NOTE]  
> La lecture est autorisée pour chaque travail. La lecture n’est pas prise en charge pour des travaux incomplets.

### <a name="zoom"></a>Zoom

Utilisez la molette de la souris pour effectuer un zoom avant/arrière du graphe de travail, ou sélectionnez **Zoom to fit** (Zoom d’ajustement) pour l’adapter à la taille de l’écran.

![Application Spark - Graphe de travail - Zoomer pour ajuster](./media/apache-spark-history-server/sparkui-graph-zoom2fit.png)

### <a name="tooltips"></a>Info-bulles

Pointez sur un nœud de graphique pour afficher l’info-bulle en cas d’échec de tâches, puis sélectionnez une phase pour ouvrir la page correspondante.

![Application Spark - Graphe de travail - Info-bulle](./media/apache-spark-history-server/sparkui-graph-tooltip.png)

Sous l’onglet du graphique du travail, une info-bulle et une petite icône s’affichent en regard des phases qui comprennent des tâches qui remplissent les conditions suivantes :

|Condition|Description|
|-|-|
|Asymétrie des données|taille de lecture des données > taille moyenne de lecture des données de toutes les tâches à l’intérieur de cette phase * 2 et taille de lecture des données > 10 Mo|
|Asymétrie temporelle|durée d’exécution > durée d’exécution moyenne de toutes les tâches à l’intérieur de cette phase * 2 et durée d’exécution > 2 minutes|
   
![Application Spark - Graphe de travail - Icône de l’asymétrie](./media/apache-spark-history-server/sparkui-graph-skew-icon.png)

### <a name="graph-node-description"></a>Description du nœud de graphique

Le nœud de graphique du travail affiche les informations suivantes de chaque phase :

  * ID.
  * Nom ou description.
  * Nombre total de tâches.
  * Lecture de données : somme de la taille d’entrée et de la taille de lecture aléatoire.
  * Écriture de données : somme de la taille de sortie et de la taille des écritures aléatoires.
  * Durée d’exécution : durée entre l’heure de début de la première tentative et l’heure de fin de la dernière tentative.
  * Nombre de lignes : somme des enregistrements d’entrée, des enregistrements de sortie, des enregistrements de lecture aléatoire et des enregistrements d’écriture aléatoire.
  * Avancement.

    > [!NOTE]  
    > Par défaut, le nœud de graphique de travail affiche les informations de la dernière tentative de chaque phase (à l’exception de la durée d’exécution de la phase). Toutefois, en cours de lecture, le nœud de graphique affiche les informations de chaque tentative.
    >  
    > Pour la taille des données de lecture et d’écriture, nous utilisons 1 Mo = 1000 Ko = 1000 *1000 octets.

### <a name="provide-feedback"></a>Fournir des commentaires

Pour signaler des problèmes, envoyez des commentaires en sélectionnant **Envoyez-nous vos commentaires**.

![Application Spark - Graphe de travail - Commentaires](./media/apache-spark-history-server/sparkui-graph-feedback.png)

## <a name="explore-the-diagnosis-tab-in-apache-spark-history-server"></a>Explorer l’onglet Diagnostic dans le serveur d’historique Apache Spark

Pour accéder à l’onglet Diagnostic, sélectionnez un ID de travail. Sélectionnez ensuite **Diagnostic** dans le menu des outils pour ouvrir la vue Diagnostic du travail. L’onglet du diagnostic comprend **Data Skew** (Asymétrie des données), **Time Skew** (Asymétrie temporelle) et **Executor Usage Analysis** (Analyse de l’utilisation de l’exécuteur).

Vérifiez l’**asymétrie des données**, l’**asymétrie temporelle** et l’**Analyse de l’utilisation de l’exécuteur** en sélectionnant l’onglet correspondant.

![Diagnostic SparkUI - Onglet Asymétrie des données](./media/apache-spark-history-server/sparkui-diagnosis-tabs.png)

### <a name="data-skew"></a>Asymétrie des données

Lorsque vous sélectionnez l’onglet **Data Skew** (Asymétrie des données), les tâches asymétriques correspondante sont affichées en fonction des paramètres spécifiés.

* **Spécifier les paramètres** : la première section affiche les paramètres qui sont utilisés pour détecter l’asymétrie des données. La règle par défaut est : Les données des tâches lues sont trois fois supérieures aux données des tâches lues en moyenne, et les données des tâches lues sont supérieures à 10 Mo. Si vous souhaitez définir votre propre règle pour les tâches asymétriques, vous pouvez choisir vos paramètres. Les sections **Skewed Stage**(Phase asymétrique) et **Skew Chart** (Graphe d’asymétrie) sont actualisées en conséquence.

* **Phase asymétrique** : la seconde section affiche les phases comportant des tâches asymétriques qui répondent aux critères spécifiés ci-dessus. S’il y a plusieurs tâches asymétriques dans une phase, la table des phases asymétriques affiche uniquement la tâche la plus asymétrique (par exemple, les données les plus volumineuses pour l’asymétrie des données).

    ![Diagnostic SparkUI - Onglet Asymétrie des données](./media/apache-spark-history-server/sparkui-diagnosis-dataskew-section2.png)

* **Skew Chart** (Graphique des asymétries) : quand une ligne de la table des phases asymétriques est sélectionnée, le graphique des asymétries affiche plus de détails sur la distribution des tâches en fonction de la lecture des données et de la durée d’exécution. Les tâches asymétriques sont marquées en rouge et les tâches normales sont marquées en bleu. Le graphique affiche jusqu’à 100 exemples de tâches et les détails des tâches s’affichent dans le volet inférieur droit.

    ![SparkUI - Graphique d’asymétrie pour l’étape 10](./media/apache-spark-history-server/sparkui-diagnosis-dataskew-section3.png)

### <a name="time-skew"></a>Asymétrie temporelle

L’onglet **Time Skew** affiche les tâches asymétriques en fonction de la durée d’exécution des tâches.

* **Spécifier les paramètres** : la première section affiche les paramètres qui sont utilisés pour détecter l’asymétrie temporelle. Par défaut, les critères de détection de l’asymétrie temporelle sont les suivants : la durée d’exécution de la tâche est plus de trois fois supérieure à la durée d’exécution moyenne et supérieure à 30 secondes. Vous pouvez changer les paramètres en fonction de vos besoins. Les sections **Skewed Stage** et **Skew Chart** affichent les informations sur les phases et les tâches correspondantes, à l’image de l’onglet **Data Skew** ci-dessus.

* Sélectionnez **Asymétrie temporelle** pour afficher les résultats filtrés dans la section **Phase asymétrique**, en fonction des paramètres définis dans la section **Spécifier les paramètres**. Sélectionnez un élément dans la section **Phase asymétrique**. Le graphique correspondant est alors illustré dans la section 3 et les détails de la tâche sont affichés dans le panneau inférieur droit.

    ![SparkUI - Diagnostic - Section Asymétrie temporelle](./media/apache-spark-history-server/sparkui-diagnosis-timeskew-section2.png)

### <a name="executor-usage-analysis"></a>Analyse de l’utilisation des exécuteurs

Le graphique de l’utilisation de l’exécuteur présente l’allocation et l’état de l’exécution de l’exécuteur du travail Spark.  

1. Sélectionnez **Executor Usage Analysis** (Analyse de l’utilisation de l’exécuteur) pour afficher quatre types de courbes portant sur l’utilisation de l’exécuteur, à savoir **Allocated Executors** (Exécuteurs alloués), **Running Executors** (Exécuteurs en cours d’exécution), **Idle Executors** (Exécuteurs inactifs) et **Max Executor Instances** (Nombre maximal d’instances d’exécuteur). En ce qui concerne les exécuteurs alloués, chaque événement d’ajout ou de suppression d’exécuteur augmente ou réduit le nombre d’exécuteurs alloués. Vous pouvez cocher la case « Event Timeline » (Chronologie des événements) sous l’onglet « Jobs » (Travaux) pour effectuer d’autres comparaisons.

   ![SparkUI - Diagnostic - Onglet Exécuteurs](./media/apache-spark-history-server/sparkui-diagnosis-executors.png)

2. Sélectionnez l’icône de couleur pour sélectionner ou désélectionner le contenu correspondant dans tous les brouillons.

    ![diagnostic sparkui sélectionner un graphique](./media/apache-spark-history-server/sparkui-diagnosis-select-chart.png)

## <a name="known-issues"></a>Problèmes connus

Les données d’entrée/sortie utilisant des jeux de données distribués résilients (RDD) ne s’affichent pas sous l’onglet Data (Données).

## <a name="next-steps"></a>Étapes suivantes

- [Azure Synapse Analytics](../overview-what-is.md)
- [Documentation .NET pour Apache Spark](/dotnet/spark?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)

