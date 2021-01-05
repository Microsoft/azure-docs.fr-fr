---
title: Guide de maintenance prédictive pour l’industrie aérospatiale - Team Data Science Process
description: Guide technique d'utilisation du modèle de solution pour la maintenance prédictive dans les secteurs de l'aérospatial, des services publics et du transport.
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: previous-author=fboylu, previous-ms.author=fboylu
ms.openlocfilehash: 3edeee8f41c806c90f32208c0c4f174c76ba38d0
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93321987"
---
# <a name="technical-guide-to-the-solution-template-for-predictive-maintenance-in-aerospace"></a>Guide technique du modèle de solution pour la maintenance prédictive dans l'industrie aérospatiale

> [!Important]
> Cet article est obsolète. La présentation sur la maintenance prédictive dans l’industrie aérospatiale est toujours appropriée mais, pour plus d’informations, reportez-vous à [Vue d’ensemble de la solution destinée aux entreprises](https://github.com/Azure/cortana-intelligence-predictive-maintenance-aerospace).


Les modèles de solution visent à accélérer le processus de création d'une démonstration E2E. Un modèle déployé approvisionne votre abonnement avec les composants nécessaires, puis établit les relations entre eux. Il alimente également le pipeline de données avec des exemples de données à partir d’une application de générateur de données, que vous téléchargez et installez sur votre ordinateur local après avoir déployé le modèle de solution. Les données issues du générateur alimentent le pipeline de données et commencent à générer des prédictions d’apprentissage automatique, que vous pouvez ensuite visualiser sur le tableau de bord Power BI.

Le processus de déploiement vous guide à travers plusieurs étapes permettant de configurer les informations d’identification de votre solution. Veillez à consigner les informations d’identification, telles que le nom de la solution, le nom d’utilisateur et le mot de passe, que vous renseignez pendant le déploiement. 


Les objectifs de cet article sont les suivants :
- Décrire l’architecture de référence et les composants provisionnés dans votre abonnement.
- Montrer comment remplacer les exemples de données par vos propres données. 
- Montrer comment modifier le modèle de solution.  

## <a name="overview"></a>Vue d’ensemble
![Architecture de maintenance prédictive](./media/predictive-maintenance-technical-guide/predictive-maintenance-architecture.png)

Le déploiement de la solution active les services Azure, notamment Event Hub, Stream Analytics, HDInsight, Data Factory et Machine Learning. Le schéma d’architecture illustre la manière dont est construit le modèle de solution de maintenance prédictive pour le secteur aérospatial. Vous pouvez passer en revue ces services dans le portail Azure en cliquant dessus dans le schéma du modèle de solution créé avec le déploiement de la solution (à l’exception de HDInsight, qui est provisionné à la demande quand les activités de pipeline connexes doivent être exécutées avant d’être supprimées dans la foulée).
Téléchargez ce [schéma en taille réelle](https://download.microsoft.com/download/1/9/B/19B815F0-D1B0-4F67-AED3-A40544225FD1/ca-topologies-maintenance-prediction.png).

Les sections suivantes décrivent les parties de la solution.

## <a name="data-source-and-ingestion"></a>Source et ingestion de données
### <a name="synthetic-data-source"></a>Source de données de synthèse
Pour ce modèle, la source de données utilisée est générée depuis une application de bureau téléchargée que vous exécutez localement, après la réussite du déploiement.

Pour rechercher les instructions de téléchargement et d’installation de cette application, sélectionnez le premier nœud, Predictive Maintenance Data Generator, sur le schéma du modèle de solution. Les instructions se trouvent dans la barre Propriétés. Cette application alimente le service [Azure Event Hub](#azure-event-hub) avec des points de données, ou événements, utilisés dans le reste du flux de la solution. Cette source de données est dérivée des données accessibles au public à partir du [référentiel de données de la NASA](https://c3.nasa.gov/dashlink/resources/139/) à l’aide du [jeu de données de simulation de la dégradation du turbomoteur](https://ti.arc.nasa.gov/tech/dash/groups/pcoe/prognostic-data-repository/#turbofan).

L’application de génération d’événements alimente le service Azure Event Hub uniquement lorsqu’elle est exécutée sur votre ordinateur.  

### <a name="azure-event-hub"></a>Azure Event Hub  
Le service [Azure Event Hub](https://azure.microsoft.com/services/event-hubs/) est le destinataire des données fournies par la source de données de synthèse.

## <a name="data-preparation-and-analysis"></a>Préparation et analyse des données  
### <a name="azure-stream-analytics"></a>Azure Stream Analytics
Utilisez [Azure Stream Analytics](https://azure.microsoft.com/services/stream-analytics/) pour fournir des analyses quasiment en temps réel du flux d’entrée en provenance du service [Azure Event Hub](#azure-event-hub). Ensuite, vous publiez les résultats sur un tableau de bord [Power BI](https://powerbi.microsoft.com) et archivez tous les événements entrants bruts sur le service [Stockage Azure](https://azure.microsoft.com/services/storage/) en vue de leur traitement par le service [Azure Data Factory](https://azure.microsoft.com/documentation/services/data-factory/).

### <a name="hdinsight-custom-aggregation"></a>Agrégation personnalisée HDInsight
Exécutez des scripts [Hive](/archive/blogs/uk_faculty_connection/getting-started-with-microsoft-big-data-hive-hdinsight-jump-start) (orchestrés par Azure Data Factory) à l’aide de HDInsight pour agréger les événements bruts archivés à l’aide de la ressource Azure Stream Analytics.

### <a name="azure-machine-learning"></a>Azure Machine Learning
Prédisez la durée de vie restante (RUL) d’un moteur d’avion donné en utilisant les entrées reçues par le [service Azure Machine Learning](https://azure.microsoft.com/services/machine-learning/) (orchestré par Azure Data Factory). 

## <a name="data-publishing"></a>Publication des données
### <a name="azure-sql-database"></a>Azure SQL Database
Utilisez le service [Azure SQL Database](https://azure.microsoft.com/services/sql-database/) pour stocker les prédictions reçues par Azure Machine Learning, qui sont ensuite utilisées dans le tableau de bord [Power BI](https://powerbi.microsoft.com).

## <a name="data-consumption"></a>Consommation des données
### <a name="power-bi"></a>Power BI
Utilisez [Power BI](https://powerbi.microsoft.com) pour afficher un tableau de bord contenant les agrégations et alertes fournies par [Azure Stream Analytics](https://azure.microsoft.com/services/stream-analytics/), ainsi que les prédictions de durée de vie restante stockées dans [Azure SQL Database](https://azure.microsoft.com/services/sql-database/) qui ont été générées avec [Azure Machine Learning](https://azure.microsoft.com/services/machine-learning/).

## <a name="how-to-bring-in-your-own-data"></a>Comment importer vos propres données
Cette section explique comment importer vos propres données dans Azure et décrit les éléments à modifier compte tenu des données que vous importez dans cette architecture.

Il est peu probable que votre jeu de données corresponde parfaitement au [jeu de données de simulation de la dégradation du turbomoteur](https://ti.arc.nasa.gov/tech/dash/groups/pcoe/prognostic-data-repository/#turbofan) utilisé pour ce modèle de solution. Il est important de bien comprendre vos données et besoins pour déterminer la façon de modifier ce modèle pour l’utiliser avec vos propres données. 

Dans les sections suivantes, nous allons décrire les parties du modèle que vous devez modifier à l’occasion de l’introduction d’un nouveau jeu de données.

### <a name="azure-event-hub"></a>Azure Event Hub
Azure Event Hub est générique ; les données peuvent être publiées sur le hub au format CSV ou JSON. Bien que le service Azure Event Hub n’implique aucun traitement particulier, il est important de comprendre les données qui y sont importées.

Ce document ne décrit pas le mode de réception de vos données, mais vous pouvez facilement envoyer des événements ou des données vers un service Azure Event Hub à l’aide des API Event Hub.

### <a name="azure-stream-analytics"></a><a name="azure-stream-analytics-1"></a>Azure Stream Analytics
Utilisez la ressource Azure Stream Analytics pour fournir des analyses en temps quasi-réel grâce à une lecture des flux de données et à l’envoi de données vers diverses sources.

Dans le cas du modèle de solution de maintenance prédictive pour le secteur aérospatial, la requête Azure Stream Analytics se compose de quatre sous-requêtes, chaque requête utilisant les événements dérivés du service Azure Event Hub, avec des sorties à quatre emplacements distincts. Ces sorties comprennent trois jeux de données Power BI et un emplacement Azure Storage.

Pour accéder à la requête Azure Stream Analytics :

* Connectez-vous au portail Azure
* Recherchez les travaux Stream Analytics ![icône Stream Analytics](./media/predictive-maintenance-technical-guide/icon-stream-analytics.png) qui ont été générés au moment du déploiement de la solution ( *par exemple* , **maintenancesa02asapbi** et **maintenancesa02asablob** pour la solution de maintenance prédictive)
* Sélectionnez
  
  * ***INPUTS** _ pour afficher l’entrée de requête _ * **QUERY** _ pour afficher la requête elle-même _ * **OUTPUTS** _ pour afficher les différentes sorties

Pour plus d’informations sur la construction des requêtes Azure Stream Analytics, consultez la [référence de requête Stream Analytics](/stream-analytics-query/stream-analytics-query-language-reference) sur MSDN.

Dans cette solution, les requêtes génèrent trois jeux de données avec des informations d’analyse en temps quasi-réel concernant le flux de données entrant dans un tableau de bord Power BI fourni dans le cadre de ce modèle de solution. Étant donné que le format des données entrantes est implicitement connu, ces requêtes doivent être modifiées en fonction de votre format de données.

La requête dans le deuxième travail Stream Analytics _ *maintenancesa02asablob* * génère simplement tous les événements [Event Hub](https://azure.microsoft.com/services/event-hubs/) dans [Stockage Azure](https://azure.microsoft.com/services/storage/) et ne nécessite donc aucune modification, quel que soit votre format de données, puisque les informations d’événement complètes sont transmises en continu vers le stockage.

### <a name="azure-data-factory"></a>Azure Data Factory
Le service [Azure Data Factory](https://azure.microsoft.com/documentation/services/data-factory/) orchestre le déplacement et le traitement des données. Dans le modèle de solution de maintenance prédictive pour le secteur aérospatial, la fabrique de données est constituée de trois [pipelines](../../data-factory/concepts-pipelines-activities.md) qui déplacent et traitent les données à l’aide de différentes technologies.  Accédez à votre fabrique de données en ouvrant le nœud Data Factory en bas du diagramme du modèle de solution créé avec le déploiement de la solution. Les erreurs dans vos jeux de données sont liées à la fabrique de données déployée avant le démarrage du générateur de données. Vous pouvez ignorer ces erreurs, qui ne perturbent pas le fonctionnement de votre fabrique de données.

![Erreurs de jeu de données Data Factory](./media/predictive-maintenance-technical-guide/data-factory-dataset-error.png)

Cette section décrit les [pipelines et activités](../../data-factory/concepts-pipelines-activities.md) nécessaires qui sont contenus dans le service [Azure Data Factory](https://azure.microsoft.com/documentation/services/data-factory/). Voici une vue schématique de la solution.

![Azure Data Factory](./media/predictive-maintenance-technical-guide/azure-data-factory.png)

Deux des pipelines de cette fabrique contiennent des scripts [Hive](/archive/blogs/uk_faculty_connection/getting-started-with-microsoft-big-data-hive-hdinsight-jump-start) utilisés pour partitionner et agréger les données. Quand ils sont référencés, les scripts apparaissent dans le compte [Stockage Azure](https://azure.microsoft.com/services/storage/) créé pendant l’installation, à l’emplacement suivant : maintenancesascript\\\\script\\\\hive\\\\ (ou https://[nom de votre solution].blob.core.windows.net/maintenancesascript).

De la même manière que des requêtes [Azure Stream Analytics](#azure-stream-analytics-1), les scripts [Hive](/archive/blogs/uk_faculty_connection/getting-started-with-microsoft-big-data-hive-hdinsight-jump-start) ont une connaissance implicite du format des données entrantes et doivent être modifiés en fonction de votre format de données.

#### <a name="aggregateflightinfopipeline"></a>*AggregateFlightInfoPipeline*
Ce [pipeline](../../data-factory/concepts-pipelines-activities.md) contient une seule activité, une activité [HDInsightHive](../../data-factory/transform-data-using-hadoop-hive.md) qui utilise un [HDInsightLinkedService](/previous-versions/azure/dn893526(v=azure.100)) exécutant un script [Hive](/archive/blogs/uk_faculty_connection/getting-started-with-microsoft-big-data-hive-hdinsight-jump-start) pour partitionner les données placées dans [Stockage Azure](https://azure.microsoft.com/services/storage/) pendant l’exécution du travail [Azure Stream Analytics](https://azure.microsoft.com/services/stream-analytics/).

Le script [Hive](/archive/blogs/uk_faculty_connection/getting-started-with-microsoft-big-data-hive-hdinsight-jump-start) pour cette tâche de partitionnement est * **AggregateFlightInfo.hql** _.

#### <a name="_mlscoringpipeline"></a>_MLScoringPipeline*
Ce [pipeline](../../data-factory/concepts-pipelines-activities.md) contient plusieurs activités dont le résultat final contient les prédictions notées à partir de l’expérimentation [Azure Machine Learning](https://azure.microsoft.com/services/machine-learning/) associée à ce modèle de solution.

Les activités incluses sont les suivantes :

* Activité [HDInsightHive](../../data-factory/transform-data-using-hadoop-hive.md) utilisant un [HDInsightLinkedService](/previous-versions/azure/dn893526(v=azure.100)) qui exécute un script [Hive](/archive/blogs/uk_faculty_connection/getting-started-with-microsoft-big-data-hive-hdinsight-jump-start) pour effectuer les agrégations et les tâches d’ingénierie de caractéristiques nécessaires pour l’expérience [Azure Machine Learning](https://azure.microsoft.com/services/machine-learning/).
  Le script [Hive](/archive/blogs/uk_faculty_connection/getting-started-with-microsoft-big-data-hive-hdinsight-jump-start) pour cette tâche de partitionnement est * **PrepareMLInput.hql** _.
  _ Activité [Copy](/previous-versions/azure/dn835035(v=azure.100)) qui déplace les résultats de l’activité [HDInsightHive](../../data-factory/transform-data-using-hadoop-hive.md) vers un blob [Stockage Azure](https://azure.microsoft.com/services/storage/) unique, accessible à l’activité [AzureMLBatchScoring](/previous-versions/azure/dn894009(v=azure.100)).
* L’activité [AzureMLBatchScoring](/previous-versions/azure/dn894009(v=azure.100)) appelle l’expérimentation [Azure Machine Learning](https://azure.microsoft.com/services/machine-learning/) et les résultats sont placés dans un objet blob [Stockage Azure](https://azure.microsoft.com/services/storage/) unique.

#### <a name="copyscoredresultpipeline"></a>*CopyScoredResultPipeline*
Ce [pipeline](../../data-factory/concepts-pipelines-activities.md) contient une seule activité, une activité [Copy](/previous-versions/azure/dn835035(v=azure.100)) qui déplace les résultats de l’expérience [Azure Machine Learning](#azure-machine-learning) entre le service * **MLScoringPipeline** _ et la [base de données Azure SQL Database](https://azure.microsoft.com/services/sql-database/) configurée dans le cadre de l’installation du modèle de solution.

### <a name="azure-machine-learning"></a>Azure Machine Learning
L’expérience [Azure Machine Learning](https://azure.microsoft.com/services/machine-learning/) utilisée pour ce modèle de solution permet de prédire la durée de vie restante (RUL) d’un moteur d’avion. L’expérimentation est spécifique au jeu de données consommé et doit être modifiée ou remplacée compte tenu des données importées.

## <a name="monitor-progress"></a>Surveiller la progression
Une fois le Générateur de données lancé, le pipeline commence à manquer de ressources et les différents composants de votre solution entrent en action selon les commandes émises par la fabrique de données. Il existe deux méthodes de surveillance du pipeline.

_ Un des travaux Stream Analytics enregistre les données brutes entrantes dans Stockage Blob. Si vous cliquez sur le composant Stockage Blob de votre solution dans l’écran à partir duquel vous avez déployé la solution, puis cliquez sur Ouvrir dans le volet droit, vous accédez au [portail Azure](https://portal.azure.com/). Une fois dans le portail, cliquez sur Objets BLOB. Dans le panneau suivant figure la liste des conteneurs. Cliquez sur **maintenancesadata**. Dans le volet suivant se trouve le dossier **rawdata**. Le dossier rawdata comprend des dossiers nommés par exemple hour=17 et hour=18. La présence de ces dossiers signifie que des données brutes sont générées sur votre ordinateur et stockées dans des objets blob. Vous devriez voir des fichiers csv avec des tailles finies en Mo dans ces dossiers.
* La dernière étape du pipeline consiste à écrire des données (par exemple, les prédictions générées à partir de l’apprentissage automatique) dans SQL Database. Vous devrez peut-être attendre un maximum de trois heures pour les données s'affichent dans la base de données SQL. Le [portail Azure](https://portal.azure.com/) offre un moyen de surveiller la quantité de données disponible dans votre base de données SQL Database. Dans le volet gauche, identifiez BASES DE DONNÉES SQL :::image type="icon" source="./media/predictive-maintenance-technical-guide/icon-SQL-databases.png" border="false"::: et cliquez dessus. Recherchez ensuite votre base de données **pmaintenancedb** et cliquez dessus. En bas de la page suivante, cliquez sur GÉRER.
   
    ![Icône Gérer](./media/predictive-maintenance-technical-guide/icon-manage.png)
   
    Ici, vous pouvez cliquer sur Nouvelle requête et rechercher le nombre de lignes (par exemple, select count(*) from PMResult). À mesure que votre base de données augmente, le nombre de lignes de la table devrait augmenter.

## <a name="power-bi-dashboard"></a>tableau de bord Power BI

Configurez un tableau de bord Power BI afin de visualiser vos données Azure Stream Analytics (séquence à chaud ou « hot path ») et les résultats des prédictions par lots à partir d’Azure Machine Learning (séquence à froid ou « cold path »).

### <a name="set-up-the-cold-path-dashboard"></a>Configurer le tableau de bord de séquence à froid
Dans le pipeline des données de séquence à froid, l’objectif consiste à obtenir la durée de vie restante prévisionnelle de chaque moteur d’avion au terme d’un vol (cycle). Le résultat de la prédiction est mis à jour toutes les 3 heures pour prédire la durée de vie restante des moteurs d’avion qui ont terminé un vol au cours des 3 dernières heures.

Power BI se connecte à une base de données Azure SQL en tant que source de données où sont stockés les résultats de la prédiction. 

Remarque : 
1.    Après le déploiement de votre solution, une prédiction apparaît dans la base de données dans un délai de 3 heures. Le fichier pbix fourni avec le téléchargement de Generator contient certaines données d’amorçage afin que vous puissiez créer immédiatement le tableau de bord Power BI. 
2.    Dans cette étape, le prérequis consiste à télécharger et installer le logiciel gratuit [Power BI Desktop](/power-bi/fundamentals/desktop-get-the-desktop).

Au cours des étapes suivantes, nous allons vous expliquer comment connecter le fichier pbix à la base de données SQL qui a été lancée au moment du déploiement de la solution et qui contient des données (par exemple, des résultats de prédiction) pour la visualisation.

1. Obtenez les informations d’identification de la base de données.
   
   Avant de passer aux étapes suivantes, vous avez besoin du **nom du serveur de base de données, du nom de la base de données, du nom d’utilisateur et du mot de passe** . Voici les étapes à suivre pour obtenir ces informations.
   
   * Une fois qu’ **Azure SQL Database** apparaît en vert sur votre schéma de modèle de solution, cliquez dessus, puis cliquez sur **Ouvrir**.
   * La page du portail Azure s’affiche dans un nouvel onglet/une nouvelle fenêtre de navigateur. Cliquez sur **« Groupes de ressources »** dans le volet gauche.
   * Sélectionnez l’abonnement que vous utilisez pour le déploiement de la solution, puis sélectionnez **« NomVotreSolution\_GroupeRessources »** .
   * Dans le nouveau volet contextuel, cliquez sur l’icône :::image type="icon" source="./media/predictive-maintenance-technical-guide/icon-sql.png" border="false"::: pour accéder à votre base de données. Le nom de votre base de données apparaît en regard de cette icône (par exemple, **pmaintenancedb** ) et le **nom du serveur de base de données** est listé sous la propriété du nom de serveur et doit être du type **NomVotreSolution.database.windows.net**.
   * Les **nom d’utilisateur** et **mot de passe** de votre base de données sont identiques aux nom d’utilisateur et mot de passe précédemment enregistrés au moment du déploiement de la solution.
2. Mettre à jour la source de données du fichier de rapport de chemin à froid avec Power BI Desktop.
   
   * Dans le dossier où vous avez téléchargé et décompressé le fichier Generator, double-cliquez sur le fichier **PowerBI\\PredictiveMaintenanceAerospace.pbix**. Si des messages d’avertissement s’affichent lorsque vous ouvrez le fichier, ignorez-les simplement. En haut du fichier, cliquez sur **Modifier les requêtes**.
     
     ![Modifier les requêtes](./media/predictive-maintenance-technical-guide/edit-queries.png)
   * Deux tables apparaissent : **RemainingUsefulLife** et **PMResult**. Sélectionnez la première table, puis cliquez sur ![Icône de paramètres de requête](./media/predictive-maintenance-technical-guide/icon-query-settings.png) à côté de **'Source'** , sous **'ÉTAPES APPLIQUÉES'** sur le panneau **’Paramètres de requête’** à droite. Ignorer les messages d’avertissement qui s’affichent.
   * Dans la fenêtre contextuelle, remplacez **Serveur** et **Base de données** par vos propres noms de serveur et de base de données, puis cliquez sur **OK**. Pour le nom du serveur, veillez à spécifier le port 1433 ( **NomVotreSolution.database.windows.net, 1433** ). Laissez le champ de base de données en tant que **pmaintenancedb**. Ignorez les messages d’avertissement qui s’affichent à l’écran.
   * Dans la fenêtre contextuelle suivante, deux options s’affichent dans le volet gauche ( **Windows** et **Base de données** ). Cliquez sur **Base de données** et renseignez vos **Nom d’utilisateur** et **Mot de passe** (il s’agit du nom d’utilisateur et du mot de passe que vous avez entrés au moment du déploiement de la solution et de la création d’une base de données Azure SQL Database). Dans **_Sélectionnez le niveau auquel appliquer ces paramètres_ *_, cochez l’option du niveau de base de données. Ensuite, cliquez sur _* « Connexion »** .
   * Cliquez sur la deuxième table **PMResult** , puis cliquez sur ![l’icône de navigation](./media/predictive-maintenance-technical-guide/icon-navigation.png) à côté de **'Source'** sous **'ÉTAPES APPLIQUÉES'** sur le panneau **Paramètres de requête** de droite, et mettez à jour les noms de serveur et de base de données comme lors des étapes ci-dessus, puis cliquez sur OK.
   * Une fois redirigé sur la page précédente, fermez la fenêtre. Un message s’affiche ; cliquez sur **Appliquer**. Pour finir, cliquez sur le bouton **Enregistrer** pour enregistrer les modifications. Votre fichier Power BI a maintenant établi la connexion au serveur. Si vos visualisations sont vides, veillez à désactiver les sélections de visualisation pour visualiser toutes les données en cliquant sur l’icône de suppression en haut à droite des légendes. Utilisez le bouton Actualiser pour afficher les nouvelles données sur les visualisations. Au départ, vous voyez uniquement les données d’amorçage sur vos visualisations, étant donné que la fabrique de données est planifiée pour s’actualiser toutes les 3 heures. Au bout de 3 heures, les nouvelles prédictions sont reflétées dans vos visualisations lorsque vous actualisez les données.
3. (Facultatif) Publier le tableau de bord de chemin à froid dans [Power BI en ligne](https://www.powerbi.com/). Pour cette étape, vous avez besoin d’un compte Power BI (ou d’un compte professionnel ou scolaire).
   
   * Cliquez sur **Publier** et attendez quelques secondes : une fenêtre s’affiche pour confirmer que la publication dans Power BI a bien été effectuée, avec une coche verte. Cliquez sur le lien sous « Ouvrir PredictiveMaintenanceAerospace.pbix dans Power BI ». Pour obtenir des instructions détaillées, consultez la page [Publier à partir de Power BI Desktop](https://support.powerbi.com/knowledgebase/articles/461278-publish-from-power-bi-desktop).
   * Pour créer un tableau de bord : cliquez sur le signe **+** en regard de la section **Tableaux de bord** dans le volet gauche. Entrez le nom de ce tableau de bord, dans notre exemple « Démo de maintenance prédictive ».
   * Une fois que vous ouvrez le rapport, cliquez sur ![l’icône PIN](./media/predictive-maintenance-technical-guide/icon-pin.png) pour épingler toutes les visualisations à votre tableau de bord. Pour obtenir des instructions détaillées, consultez la page [Épingler une vignette à un tableau de bord Power BI à partir d’un rapport](https://support.powerbi.com/knowledgebase/articles/430323-pin-a-tile-to-a-power-bi-dashboard-from-a-report).
     Accédez à la page Tableau de bord et ajustez la taille et l’emplacement de vos visualisations, puis modifier leurs titres. Pour obtenir des instructions détaillées sur la modification de vos vignettes, consultez [Modifier une vignette -- redimensionner, déplacer, renommer, épingler, supprimer, ajouter un lien hypertexte](https://powerbi.microsoft.com/documentation/powerbi-service-edit-a-tile-in-a-dashboard/#rename). Voici un exemple de tableau de bord sur lequel sont épinglées des visualisations de chemin à froid.  Selon la durée pendant laquelle vous exécutez votre générateur de données, les numéros des visualisations peuvent être différents.
     <br/>
     ![Affichage final](./media/predictive-maintenance-technical-guide/final-view.png)
     <br/>
   * Pour planifier l'actualisation des données, pointez votre souris sur le jeu de données **PredictiveMaintenanceAerospace** , puis cliquez sur les ![points de suspension](./media/predictive-maintenance-technical-guide/icon-elipsis.png) et sur **Planifier l'actualisation**.
     <br/>
     > [!NOTE]
     > Si un message d’avertissement s’affiche, cliquez sur **Modifier les informations d’identification** et vérifiez que vos informations d’identification de base de données sont les mêmes que celles décrites à l’étape 1.
     <br/>
     ![Planifier l’actualisation](./media/predictive-maintenance-technical-guide/schedule-refresh.png)
     <br/>
   * Développez la section **Planifier l’actualisation** . Activez l’option Maintenir vos données à jour.
     <br/>
   * Planifiez l’actualisation selon vos besoins. Pour plus d’informations, consultez la page [Actualisation des données dans Power BI](https://support.powerbi.com/knowledgebase/articles/474669-data-refresh-in-power-bi).

### <a name="setup-hot-path-dashboard"></a>Configuration du tableau de bord de chemin à chaud
Dans les étapes suivantes, nous allons vous expliquer comment visualiser la sortie de données des tâches Stream Analytics qui ont été générées au moment du déploiement de la solution. Les étapes suivantes nécessitent un compte [Power BI en ligne](https://www.powerbi.com/) . Si vous ne possédez pas de compte, vous pouvez en [créer un](https://powerbi.microsoft.com/pricing).

1. Ajouter une sortie Power BI dans Azure Stream Analytics (ASA).
   
   * Vous devez suivre les instructions qui sont dans [Azure Stream Analytics et Power BI : tableau de bord d’analytique permettant de visualiser en temps réel les données de streaming](../../stream-analytics/stream-analytics-power-bi-dashboard.md) pour configurer la sortie de votre travail Azure Stream Analytics en tant que tableau de bord Power BI.
   * La requête ASA présente trois sorties, à savoir **aircraftmonitor** , **aircraftalert** et **flightsbyhour**. Vous pouvez afficher la requête en cliquant sur l’onglet Requête. Pour chacune de ces tables, vous devez ajouter une sortie vers ASA. Quand vous ajoutez la première sortie ( **aircraftmonitor** ) vérifiez que les paramètres **Alias de sortie** , **Nom du jeu de données** et **Nom de la table** sont identiques ( **aircraftmonitor** ). Répétez les étapes pour ajouter des sorties pour **aircraftalert** et **flightsbyhour**. Une fois que vous avez ajouté les trois tables de sortie et démarré le travail ASA, vous devriez obtenir un message de confirmation (« La tâche Stream Analytics maintenancesa02asapbi a bien démarré »).
2. Se connecter à [Power BI en ligne](https://www.powerbi.com)
   
   * Dans la section Jeux de données du volet gauche, dans Mon espace de travail, les noms de * **JEU DE DONNÉES** _ _*aircraftmonitor**, **aircraftalert** et **flightsbyhour** doivent apparaître. Il s'agit des données de diffusion en continu qui ont fait l’objet d’un Push à partir d'Azure Stream Analytics au cours de l'étape précédente. Le jeu de données **flightsbyhour** peut ne pas apparaître en même temps que les deux autres jeux de données en raison de la nature de la requête SQL sous-jacente à ce jeu. Toutefois, il doit s’afficher après une heure.
   * Assurez-vous que le volet * **Visualisations** _ est ouvert et qu’il s’affiche à droite de l’écran.
3. Une fois que les données affluent dans Power BI, vous pouvez commencer à visualiser les données de diffusion. Le tableau ci-dessous est un exemple de tableau de bord sur lequel sont épinglées des visualisations de chemin à chaud. Vous pouvez créer d’autres vignettes de tableau de bord en fonction des jeux de données appropriés. Selon la durée pendant laquelle vous exécutez votre générateur de données, les numéros des visualisations peuvent être différents.

    ![Vue du tableau de bord](media/predictive-maintenance-technical-guide/dashboard-view.png)

1. Voici comment vous pouvez créer l’une des vignettes ci-dessus : - « Fleet View of Sensor 11 vs. Threshold 48.26 » :
   
   _ Cliquez sur le jeu de données **aircraftmonitor** dans la section Jeux de données du volet gauche.
   * Cliquez sur l’icône **Courbes** .
   * Cliquez sur **Traités** dans le volet **Champs** pour l’afficher sous « Axe » dans le volet **Visualisations**.
   * Cliquez sur « s11 » et « s11alert\_alert » pour faire apparaître les deux sous « Valeurs ». Cliquez sur la flèche située en regard de **s11** et **s11\_alert** et remplacez « Somme » par « Moyenne ».
   * Cliquez sur **ENREGISTRER** en haut de l’écran et nommez le rapport « aircraftmonitor ». Le rapport « aircraftmonitor » s’affiche dans la section **Rapports** du volet gauche **Navigateur**.
   * Cliquez sur l’icône **Épingler le visuel** en haut à droite de ce graphique en courbes. Une fenêtre « Épingler au tableau de bord » peut apparaître pour vous permettre de sélectionner un tableau de bord. Sélectionnez « Démo de Maintenance prédictive », puis cliquez sur « Épingler ».
   * Dans le tableau de bord, pointez la souris sur cette vignette, cliquez sur l’icône Modifier située en haut à droite pour donner le titre « Fleet View of Sensor 11 vs. Threshold 48.26 » et le sous-titre « Average across fleet over time ».

## <a name="delete-your-solution"></a>Supprimer votre solution
Veillez à arrêter le générateur de données quand vous n’utilisez pas la solution activement, car l’exécution du générateur de données entraîne des coûts plus élevés. Supprimez la solution si vous ne l’utilisez pas. La suppression de votre solution supprime tous les composants qui étaient configurés dans votre abonnement quand vous avez déployé la solution. Pour supprimer la solution, cliquez sur son nom dans le volet gauche du modèle de solution, puis cliquez sur **Supprimer**.

## <a name="cost-estimation-tools"></a>Outils d’estimation des coûts
Les deux outils suivants peuvent vous aider à mieux comprendre les coûts impliqués dans l’exécution du modèle de solution de maintenance prédictive pour le secteur aérospatial dans votre abonnement :

* [Microsoft Azure Cost Estimator Tool (en ligne)](https://azure.microsoft.com/pricing/calculator/)
* [Microsoft Azure Cost Estimator Tool (de bureau)](https://www.microsoft.com/download/details.aspx?id=43376)