---
title: Utiliser les fonctionnalités étendues du serveur d’historique Apache Spark pour déboguer des applications – Azure HDInsight
description: Utilisez les fonctionnalités étendues du serveur d’historique Apache Spark pour déboguer et diagnostiquer des applications Spark – Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,hdiseo17may2017
ms.date: 11/25/2019
ms.openlocfilehash: d8dd9aaeaadf13fa48577cf2853e7bcf58badb41
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86079290"
---
# <a name="use-the-extended-features-of-the-apache-spark-history-server-to-debug-and-diagnose-spark-applications"></a>Utiliser les fonctionnalités étendues du serveur d’historique Apache Spark pour déboguer et diagnostiquer des applications Spark

Cet article vous montre comment utiliser les fonctionnalités étendues du serveur d’historique Apache Spark étendu pour déboguer et diagnostiquer des applications Spark terminées ou en cours d’exécution. L’extension comprend les onglets **Data**, **Graph** et **Diagnosis**. Sous l’onglet **Data**, vous pouvez vérifier les données d’entrée et de sortie du travail Spark. Sous l’onglet **Graph**, vous pouvez vérifier le flux de données et lire à nouveau le graphe des travaux. Sous l’onglet **Diagnosis**, vous pouvez voir les fonctionnalités d’**asymétrie des données**, d’**asymétrie temporelle** et d’**analyse de l’utilisation des exécuteurs**.

## <a name="get-access-to-the-spark-history-server"></a>Accéder au serveur d’historique Spark

Le serveur d’historique Spark est l’interface utilisateur web pour les applications Spark terminées et en cours d’exécution. Vous pouvez l’ouvrir à partir du portail Azure ou d’une URL.

### <a name="open-the-spark-history-server-web-ui-from-the-azure-portal"></a>Ouvrir l’interface utilisateur web du serveur d’historique Spark à partir du portail Azure

1. À partir du [portail Azure](https://portal.azure.com/), ouvrez le cluster Spark. Pour plus d’informations, voir [Énumération et affichage des clusters](../hdinsight-administer-use-portal-linux.md#showClusters).
2. Dans **Tableaux de bord du cluster**, sélectionnez **Serveur d’historique Spark**. Lorsque vous y êtes invité, entrez les informations d’identification d’administrateur pour le cluster Spark.

    ![Lancer le serveur d’historique Spark à partir du portail Azure](./media/apache-azure-spark-history-server/azure-portal-dashboard-spark-history.png "Serveur d’historique Spark")

### <a name="open-the-spark-history-server-web-ui-by-url"></a>Ouvrir l’interface utilisateur web du serveur d’historique Spark au moyen d’une URL

Ouvrez le serveur d’historique Spark en accédant à `https://CLUSTERNAME.azurehdinsight.net/sparkhistory`, où **CLUSTERNAME** est le nom de votre cluster Spark.

L’interface utilisateur web du serveur d’historique Spark peut être similaire à l’image suivante :

![Page du serveur d’historique Spark](./media/apache-azure-spark-history-server/hdinsight-spark-history-server.png)

## <a name="use-the-data-tab-in-the-spark-history-server"></a>Utiliser l’onglet Data du serveur d’historique Spark

Sélectionnez l’ID du travail, puis sélectionnez **Data** dans le menu des outils pour afficher la vue des données.

+ Examinez les **entrées**, les **sorties** et les **opérations de table** en sélectionnant les onglets individuels.

    ![Onglets de données dans la page Data for Spark Application](./media/apache-azure-spark-history-server/apache-spark-data-tabs.png)

+ Copiez toutes les lignes en sélectionnant le bouton **Copy**.

    ![Copier des données dans la page de l’application Spark](./media/apache-azure-spark-history-server/apache-spark-data-copy.png)

+ Enregistrez toutes les données sous la forme d’un fichier CSV en sélectionnant le bouton **csv**.

    ![Enregistrer dans un fichier CSV les données de la page Data for Spark Application](./media/apache-azure-spark-history-server/apache-spark-data-save.png)

+ Effectuez des recherches dans les données en entrant des mots clés dans le champ **Search**. Les résultats de recherche sont affichés immédiatement.

    ![Rechercher des données dans la page Data for Spark Application](./media/apache-azure-spark-history-server/apache-spark-data-search.png)

+ Sélectionnez l’en-tête de colonne pour trier la table. Sélectionnez le signe plus (+) pour développer une ligne et afficher plus de détails. Sélectionnez le signe moins (-) pour réduire une ligne.

    ![Table de données dans la page Data for Spark Application](./media/apache-azure-spark-history-server/apache-spark-data-table.png)

+ Téléchargez un fichier unique en sélectionnant le bouton **Partial Download** (Téléchargement partiel) à droite. Le fichier sélectionné est téléchargé localement. Si le fichier n’existe plus, un nouvel onglet s’ouvre pour afficher les messages d’erreur.

    ![Ligne de téléchargement des données dans la page Data for Spark Application](./media/apache-azure-spark-history-server/sparkui-data-download-row.png)

+ Copiez un chemin complet ou un chemin relatif en sélectionnant l’option **Copy Full Path** (Copier le chemin complet) ou l’option **Copy Relative Path** (Copier le chemin relatif), qui étendent le menu de téléchargement. Pour les fichiers Azure Data Lake Storage, sélectionnez **Open in Azure Storage Explorer** (Ouvrir dans l’Explorateur Stockage Azure) pour lancer l’Explorateur Stockage Azure et localiser le dossier après la connexion.

    ![Options Copy Full Path (Copier le chemin complet) et Copy Relative Path (Copier le chemin relatif) dans la page Data for Spark Application](./media/apache-azure-spark-history-server/sparkui-data-copy-path.png)

+ Si le nombre de lignes à afficher sur une seule page est trop important, sélectionnez les numéros de page en bas de la table pour naviguer.

    ![Numéros de page dans la page Data for Spark Application](./media/apache-azure-spark-history-server/apache-spark-data-page.png)

+ Pour plus d’informations, pointez sur le point d’interrogation en regard de **Data for Spark Application**, ou sélectionnez-le, pour afficher l’info-bulle.

    ![Obtenir plus d’informations dans la page Data for Spark Application (Données pour l’application Spark)](./media/apache-azure-spark-history-server/sparkui-data-more-info.png)

+  Pour envoyer des commentaires sur les problèmes, sélectionnez **Provide us feedback** (Envoyez-nous vos commentaires).

    ![Envoyer des commentaires à partir de la page Data for Spark Application](./media/apache-azure-spark-history-server/sparkui-graph-feedback.png)

## <a name="use-the-graph-tab-in-the-spark-history-server"></a>Utiliser l’onglet Graph du serveur d’historique Spark

+ Sélectionnez l’ID de travail, puis sélectionnez **Graph** dans le menu des outils pour afficher le graphe de travail. Par défaut, le graphe affiche tous les travaux. Filtrez les résultats à l’aide du menu déroulant **Job ID** (ID de travail).

    ![Menu déroulant Job ID (ID de travail) dans la page Spark Application & Job Graph (Application Spark et graphe de travail)](./media/apache-azure-spark-history-server/apache-spark-graph-jobid.png)

+ L’option **Progress** est sélectionnée par défaut. Vérifiez le flux de données en sélectionnant **Read** (Données lues) ou **Written** (Données écrites) dans le menu déroulant **Display** (Affichage).

    ![Vérifier le flux de données dans la page Spark Application & Job Graph (Application Spark et graphe de travail)](./media/apache-azure-spark-history-server/sparkui-graph-display.png)

+ La couleur d’arrière-plan de chaque tâche correspond à une carte thermique.

   ![Carte thermique dans la page Spark Application & Job Graph (Application Spark et graphe de travail)](./media/apache-azure-spark-history-server/sparkui-graph-heatmap.png)


    |Couleur |Description |
    |---|---|
    |Vert|le travail s’est correctement effectué.|
    |Orange|La tâche a échoué, mais cela n’affecte pas le résultat final du travail. Ces tâches comportent des instances dupliquées ou renouvelées qui peuvent réussir ultérieurement.|
    |Bleu|la tâche est en cours d’exécution.|
    |White|la tâche est en attente d’exécution, ou la phase a été ignorée.|
    |Rouge|la tâche a échoué.|

     ![Exécution d’une tâche dans la page Spark Application & Job Graph (Application Spark et graphe de travail)](./media/apache-azure-spark-history-server/sparkui-graph-color-running.png)

     Les phases ignorées s’affichent en blanc.
    ![Tâche ignorée dans la page Spark Application & Job Graph](./media/apache-azure-spark-history-server/sparkui-graph-color-skip.png)

    ![Tâche ayant échoué dans la page Spark Application & Job Graph](./media/apache-azure-spark-history-server/sparkui-graph-color-failed.png)

     > [!NOTE]  
     > La lecture est disponible pour les travaux terminés. Sélectionnez le bouton **Playback** (Lecture) pour relire le travail. Arrêtez le travail à tout moment en sélectionnant le bouton d’arrêt. Lorsqu’un travail est relu, chaque tâche affiche son état à l’aide d’une couleur. La lecture n’est pas prise en charge pour des travaux incomplets.

+ Faites défiler pour effectuer un zoom avant/arrière du graphe de travail, ou sélectionnez **Zoom to fit** (Zoom d’ajustement) pour l’ajuster à l’écran.

    ![Sélectionner Zoom to fit (Zoom d’ajustement) dans la page Spark Application & Job Graph](./media/apache-azure-spark-history-server/sparkui-graph-zoom2fit.png)

+ Lorsque des tâches échouent, pointez sur le nœud du graphe pour afficher l’info-bulle, puis sélectionnez la phase pour l’ouvrir dans une nouvelle page.

    ![Afficher l’info-bulle dans la page Spark Application & Job Graph](./media/apache-azure-spark-history-server/sparkui-graph-tooltip.png)

+ Dans la page Spark Application & Job Graph, les phases affichent des info-bulles et de petites icônes si les tâches remplissent les conditions suivantes :
  + Asymétrie des données : Taille de lecture des données > taille moyenne de lecture des données de toutes les tâches à l’intérieur de cette phase * 2 *et* taille de lecture des données > 10 Mo.
  + Asymétrie temporelle : Durée d’exécution > durée d’exécution moyenne de toutes les tâches à l’intérieur de cette phase * 2 *et* durée d’exécution > 2 minutes.

    ![Icône de tâche asymétrique dans la page Spark Application & Job Graph](./media/apache-azure-spark-history-server/sparkui-graph-skew-icon.png)

+ Le nœud du graphe de travail affiche les informations suivantes sur chaque phase :
  + id
  + Nom ou description
  + Nombre total de tâches
  + Lecture de données : somme de la taille d’entrée et de la taille de lecture aléatoire
  + Écriture de données : somme de la taille de sortie et de la taille d’écriture aléatoire
  + Durée d’exécution : durée entre l’heure de début de la première tentative et l’heure de fin de la dernière tentative
  + Nombre de lignes : somme des enregistrements d’entrée, des enregistrements de sortie, des enregistrements de lecture aléatoire et des enregistrements d’écriture aléatoire
  + Progress

    > [!NOTE]  
    > Par défaut, le nœud de graphe de travail affiche les informations de la dernière tentative de chaque phase (à l’exception de la durée d’exécution de la phase). Toutefois, pendant la lecture, le nœud de graphe de travail affiche des informations sur chaque tentative.

    > [!NOTE]  
    > Pour les tailles de lecture et d’écriture de données, nous utilisons 1 Mo = 1 000 Ko = 1 000 * 1 000 octets.

+ Envoyez des commentaires sur les problèmes en sélectionnant **Provide us feedback** (Envoyez-nous vos commentaires).

    ![Option de commentaires dans la page Spark Application & Job Graph](./media/apache-azure-spark-history-server/sparkui-graph-feedback.png)

## <a name="use-the-diagnosis-tab-in-the-spark-history-server"></a>Utiliser l’onglet Diagnosis du serveur d’historique Spark

Sélectionnez l’ID de travail, puis sélectionnez **Diagnosis** dans le menu des outils pour afficher la vue de diagnostic de travail. L’onglet **Diagnosis** comprend **Data Skew** (Asymétrie des données), **Time Skew** (Asymétrie temporelle) et **Executor Usage Analysis** (Analyse de l’utilisation des exécuteurs).

+ Examinez l’**Asymétrie des données**, l’**asymétrie temporelle** et l’**Analyse de l’utilisation de l’exécuteur** en sélectionnant respectivement chaque onglet.

    ![Onglet Data Skew (Asymétrie des données) sous l’onglet Diagnosis.](./media/apache-azure-spark-history-server/sparkui-diagnosis-tabs.png)

### <a name="data-skew"></a>Asymétrie des données

Sélectionnez l’onglet **Data Skew** (Asymétrie des données). Les tâches asymétriques correspondantes s’affichent en fonction des paramètres spécifiés.

#### <a name="specify-parameters"></a>Spécifier les paramètres

La section **Specify Parameters** (Spécifier les paramètres) affiche les paramètres qui sont utilisés pour détecter l’asymétrie des données. La règle par défaut est : Les données lues de tâche sont supérieures au triple de la moyenne des données lues de tâche, et les données lues de tâche sont supérieures à 10 Mo. Si vous souhaitez définir votre propre règle pour les tâches asymétriques, vous pouvez choisir vos paramètres. Les sections **Phase asymétrique** et **Graphique d’asymétrie** sont mises à jour en conséquence.

#### <a name="skewed-stage"></a>Phase asymétrique

La section **Skewed Stage** (Phase asymétrique) affiche les phases comportant des tâches asymétriques qui répondent aux critères spécifiés. Si plusieurs tâches asymétriques figurent dans une phase, la section **Skewed Stage** (Phase asymétrique) affiche uniquement la tâche la plus décalée (à savoir, le plus important volume de données pour l’asymétrie des données).

![Vue plus large de l’onglet Data Skew (Asymétrie des données) sous l’onglet Diagnosis](./media/apache-azure-spark-history-server/sparkui-diagnosis-dataskew-section2.png)

##### <a name="skew-chart"></a>Graphique d’asymétrie

Lorsque vous sélectionnez une ligne de la table **Skew Stage** (Phase asymétrique), le **graphique d’asymétrie** affiche plus de détails sur les distributions des tâches en fonction des données lues et de la durée d’exécution. Les tâches asymétriques sont marquées en rouge et les tâches normales sont marquées en bleu. Pour des considérations liées aux performances, le graphique affiche jusqu’à 100 exemples de tâches. Les détails des tâches s’affichent dans le volet inférieur droit.

![Graphique d’asymétrie pour la phase 10 dans l’interface utilisateur Spark.](./media/apache-azure-spark-history-server/sparkui-diagnosis-dataskew-section3.png)

### <a name="time-skew"></a>Asymétrie temporelle

L’onglet **Time Skew** affiche les tâches asymétriques en fonction de la durée d’exécution des tâches.

#### <a name="specify-parameters"></a>Spécifier les paramètres

La section **Specify Parameters** (Spécifier les paramètres) affiche les paramètres qui sont utilisés pour détecter l’asymétrie temporelle. La règle par défaut est : La durée d’exécution des tâches est plus de trois fois supérieure à la durée d’exécution moyenne et supérieure à 30 secondes. Vous pouvez changer les paramètres en fonction de vos besoins. Les sections **Skewed Stage** (Phase asymétrique) et **Skew Chart** (Graphique d’asymétrie) affichent les phases correspondantes et les informations de tâche, tout comme dans l’onglet **Data Skew** (Asymétrie des données).

Lorsque vous sélectionnez **Time Skew** (Asymétrie temporelle), le résultat filtré s’affiche dans la section **Skewed Stage** (Phase asymétrique), en fonction des paramètres définis dans la section **Specify Parameters** (Spécifier les paramètres). Lorsque vous sélectionnez un élément dans la section **Skewed Stage** (Phase asymétrique), le graphique correspondant est illustré dans la troisième section et les détails des tâches sont affichés dans le volet inférieur droit.

![Onglet Time skew (Asymétrie temporelle) sous l’onglet Diagnosis](./media/apache-azure-spark-history-server/sparkui-diagnosis-timeskew-section2.png)

### <a name="executor-usage-analysis-graphs"></a>Graphes d’analyse de l’utilisation des exécuteurs

Le **graphe de l’utilisation des exécuteurs** affiche l’allocation réelle des exécuteurs du travail et leur état d’exécution.  

Lorsque vous sélectionnez **Executor Usage Analysis** (Analyse de l’utilisation des exécuteurs), quatre courbes différentes relatives à l’utilisation des exécuteurs sont tracées : **Allocated Executors** (Exécuteurs alloués), **Running Executors** (Exécuteurs en cours d’exécution), **Idle Executors** (Exécuteurs inactifs) et **Max Executor Instances** (Nombre maximal d’instances d’exécuteur). Chaque événement d’**ajout d’exécuteur** ou de **suppression d’exécuteur** augmente ou diminue le nombre d’exécuteurs alloués. Vous pouvez consulter la **chronologie des événements** dans l’onglet **Jobs** (Travaux) pour effectuer d’autres comparaisons.

![Onglet Executor Usage Analysis (Analyse de l’utilisation des exécuteurs) sous l’onglet Diagnosis](./media/apache-azure-spark-history-server/sparkui-diagnosis-executors.png)

Sélectionnez l’icône de couleur pour sélectionner ou désélectionner le contenu correspondant dans tous les brouillons.

 ![Sélectionner le graphique sous l’onglet Executor Usage Analysis (Analyse de l’utilisation des exécuteurs)](./media/apache-azure-spark-history-server/sparkui-diagnosis-select-chart.png)

## <a name="faq"></a>Questions fréquentes (FAQ)

### <a name="how-do-i-revert-to-the-community-version"></a>Comment revenir à la version de la communauté ?

Pour revenir à la version de la communauté, suivez les étapes ci-dessous :

1. Ouvrez le cluster dans Ambari.
1. Accédez à **Spark2** > **Configs**.
1. Sélectionnez **Custom spark2-defaults** (Valeurs spark2-defaults personnalisées).
1. Sélectionnez **Add Property ...** (Ajouter une propriété).
1. Ajoutez **spark.ui.enhancement.enabled=false** et enregistrez-le.
1. La propriété est maintenant définie sur **false**.
1. Sélectionnez **Enregistrer** pour enregistrer la configuration.

    ![Désactiver une fonctionnalité dans Apache Ambari](./media/apache-azure-spark-history-server/apache-spark-turn-off.png)

1. Sélectionnez **Spark2** dans le volet gauche. Ensuite, sous l’onglet **Summary** (Résumé), sélectionnez **Spark2 History Server** (Serveur d’historique Spark2).

    ![Vue de résumé dans Apache Ambari](./media/apache-azure-spark-history-server/apache-spark-restart1.png)

1. Pour redémarrer le serveur d’historique Spark, sélectionnez le bouton **Started** (Démarré) à droite de **Spark2 History Server** (Serveur d’historique Spark2), puis sélectionnez **Restart** (Redémarrer) dans le menu déroulant.

    ![Redémarrer le serveur d’historique Spark dans Apache Ambari](./media/apache-azure-spark-history-server/apache-spark-restart2.png)  

1. Actualisez l’interface utilisateur web du serveur d’historique Spark. Il revient à la version de la communauté.

### <a name="how-do-i-upload-a-spark-history-server-event-to-report-it-as-an-issue"></a>Comment charger un événement de serveur d’historique Spark pour le signaler en tant que problème ?

Si vous rencontrez une erreur au niveau du serveur d’historique Spark, procédez comme suit pour signaler l’événement.

1. Téléchargez l’événement en sélectionnant **Download** dans l’interface utilisateur web du serveur d’historique Spark.

    ![Télécharger l’événement dans l’interface utilisateur du serveur d’historique Spark](./media/apache-azure-spark-history-server/sparkui-download-event.png)

2. Sélectionnez **Provide us feedback** (Envoyez-nous vos commentaires) à partir de la page **Spark Application & Job Graph** (Application Spark et graphe de travail).

    ![Envoyer des commentaires dans la page Spark Application & Job Graph](./media/apache-azure-spark-history-server/sparkui-graph-feedback.png)

3. Fournissez le titre et une description de l’erreur. Ensuite, faites glisser le fichier .zip vers le champ d’édition et sélectionnez **Submit new issue** (Envoyer nouveau problème).

    ![Charger et soumettre un nouveau problème](./media/apache-azure-spark-history-server/apache-spark-file-issue.png)

### <a name="how-do-i-upgrade-a-jar-file-in-a-hotfix-scenario"></a>Comment mettre à niveau un fichier .jar dans un scénario de correctif logiciel ?

Si vous souhaitez effectuer une mise à niveau avec un correctif logiciel, utilisez le script suivant pour mettre à niveau `spark-enhancement.jar*`.

**upgrade_spark_enhancement.sh** :

   ```bash
    #!/usr/bin/env bash

    # Copyright (C) Microsoft Corporation. All rights reserved.

    # Arguments:
    # $1 Enhancement jar path

    if [ "$#" -ne 1 ]; then
        >&2 echo "Please provide the upgrade jar path."
        exit 1
    fi

    install_jar() {
        tmp_jar_path="/tmp/spark-enhancement-hotfix-$( date +%s )"

        if wget -O "$tmp_jar_path" "$2"; then
            for FILE in "$1"/spark-enhancement*.jar
            do
                back_up_path="$FILE.original.$( date +%s )"
                echo "Back up $FILE to $back_up_path"
                mv "$FILE" "$back_up_path"
                echo "Copy the hotfix jar file from $tmp_jar_path   to $FILE"
                cp "$tmp_jar_path" "$FILE"

                "Hotfix done."
                break
            done
        else    
            >&2 echo "Download jar file failed."
            exit 1
        fi
    }

    jars_folder="/usr/hdp/current/spark2-client/jars"
    jar_path=$1

    if ls ${jars_folder}/spark-enhancement*.jar 1>/dev/null 2>&1;   then
        install_jar "$jars_folder" "$jar_path"
    else
        >&2 echo "There is no target jar on this node. Exit with no action."
        exit 0
    fi
   ```

#### <a name="usage"></a>Usage

`upgrade_spark_enhancement.sh https://${jar_path}`

#### <a name="example"></a>Exemple

`upgrade_spark_enhancement.sh https://${account_name}.blob.core.windows.net/packages/jars/spark-enhancement-${version}.jar`

#### <a name="use-the-bash-file-from-the-azure-portal"></a>Utiliser le fichier Bash à partir du Portail Azure

1. Lancez le [Portail Azure](https://ms.portal.azure.com), puis sélectionnez votre cluster.
2. Exécutez une [action de script](../hdinsight-hadoop-customize-cluster-linux.md) avec les paramètres ci-dessous.

    |Propriété |Valeur |
    |---|---|
    |Type de script|- Personnalisé|
    |Nom|UpgradeJar|
    |URI de script bash|`https://hdinsighttoolingstorage.blob.core.windows.net/shsscriptactions/upgrade_spark_enhancement.sh`|
    |Type(s) de nœud|Head, Worker|
    |Paramètres|`https://${account_name}.blob.core.windows.net/packages/jars/spark-enhancement-${version}.jar`|

     ![Portail Azure - Envoyer une action de script](./media/apache-azure-spark-history-server/apache-spark-upload1.png)

## <a name="known-issues"></a>Problèmes connus

+ Actuellement, le serveur d’historique Spark fonctionne uniquement pour Spark 2.3 et 2.4.

+ Les données d’entrée et de sortie qui utilisent RDD ne s’affichent pas sous l’onglet **Data**.

## <a name="next-steps"></a>Étapes suivantes

+ [Gérer les ressources d’un cluster Apache Spark sur HDInsight](apache-spark-resource-manager.md)
+ [Configurer les paramètres d’Apache Spark](apache-spark-settings.md)

## <a name="suggestions"></a>Suggestions

Si vous avez des commentaires ou rencontrez des problèmes lors de l’utilisation de cet outil, envoyez un e-mail à ([hdivstool@microsoft.com](mailto:hdivstool@microsoft.com)).
