---
title: Identifier des scénarios pour Azure Machine Learning - Team Data Science Process
description: Sélectionnez les scénarios appropriés pour l’analyse prédictive avancée à l’aide du processus TDSP (Team Data Science Process).
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: 2d589d6c3394556499daf033c4c1d528a214b0e3
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93319297"
---
# <a name="scenarios-for-advanced-analytics-in-azure-machine-learning"></a>Scénarios d’analyses avancées dans Azure Machine Learning
Cet article présente les divers exemples de sources de données et les scénarios cibles qui peuvent être gérés par le processus [TDSP (Team Data Science Process)](overview.md). Le processus TDSP fournit une approche systématique permettant aux équipes de collaborer à la création d’applications intelligentes. Les scénarios présentés ici illustrent les options disponibles dans le flux de travail de traitement basées sur les caractéristiques des données, les emplacements sources et les référentiels cibles dans Azure.

L’ **arbre de décision** permettant de sélectionner les exemples de scénarios qui conviennent dans le cas de vos données et objectifs est présenté à la dernière section.

Les sections suivantes présentent quelques exemples de scénarios. Pour chaque scénario, un flux possible de science des données ou d’analyse avancée et les ressources Azure connexes sont répertoriés.

> [!NOTE]
> **Pour tous les scénarios suivants, vous devez effectuer les opérations suivantes :**
> <br/>
> 
> * [Créez un compte de stockage](../../storage/common/storage-account-create.md)
>   <br/>
> * [Création d’un espace de travail Microsoft Azure Machine Learning](../classic/create-workspace.md)
> 
> 

## <a name="scenario-1-small-to-medium-tabular-dataset-in-local-files"></a><a name="smalllocal"></a>Scénario \#1 : jeu de données tabulaires petit à moyen dans des fichiers locaux
![Fichiers locaux petits à moyens][1]

#### <a name="additional-azure-resources-none"></a>Autres ressources Azure : None
1. Connectez-vous à [Azure Machine Learning Studio](https://studio.azureml.net/).
1. Téléchargez un jeu de données.
1. Créez un flux d’expérience Azure Machine Learning commençant par le ou les jeux téléchargés.

## <a name="scenario-2-small-to-medium-dataset-of-local-files-that-require-processing"></a><a name="smalllocalprocess"></a>Scénario \#2 : jeu de données petit à moyen de fichiers locaux nécessitant un traitement
![Fichiers locaux petits à moyens avec traitement][2]

#### <a name="additional-azure-resources-azure-virtual-machine-ipython-notebook-server"></a>Autres ressources Azure : Machine virtuelle Azure (serveur IPython Notebook)
1. Créez une machine virtuelle Azure exécutant IPython Notebook.
1. Chargez des données dans un conteneur Stockage Azure.
1. Pré-traitez et nettoyez les données dans IPython Notebook, en accédant aux données à partir du conteneur Stockage Azure.
1. Transformez les données sous forme de tableau nettoyé.
1. Enregistrez les données transformées dans des objets blob Azure.
1. Connectez-vous à [Azure Machine Learning Studio](https://studio.azureml.net/).
1. Lisez les données des objets blob Azure à l’aide du module [Importer des données][import-data].
1. Créez un flux d’expérience Azure Machine Learning commençant par le ou les jeux de données ingérés.

## <a name="scenario-3-large-dataset-of-local-files-targeting-azure-blobs"></a><a name="largelocal"></a>Scénario \#3 : jeu de données volumineux de fichiers locaux, ciblant des objets blob Azure
![Fichiers locaux volumineux][3]

#### <a name="additional-azure-resources-azure-virtual-machine-ipython-notebook-server"></a>Autres ressources Azure : Machine virtuelle Azure (serveur IPython Notebook)
1. Créez une machine virtuelle Azure exécutant IPython Notebook.
1. Chargez des données dans un conteneur Stockage Azure.
1. Pré-traitez et nettoyez les données dans IPython Notebook, en accédant aux données des objets blob Azure.
1. Si besoin, transformez les données sous forme de tableau nettoyé.
1. Le cas échéant, explorez des données et créez des fonctionnalités.
1. Extrayez un échantillon de données petit à moyen.
1. Enregistrez les données échantillonnées dans des objets blob Azure.
1. Connectez-vous à [Azure Machine Learning Studio](https://studio.azureml.net/).
1. Lisez les données des objets blob Azure à l’aide du module [Importer des données][import-data].
1. Créez un flux d’expérience Azure Machine Learning commençant par le ou les jeux de données ingérés.

## <a name="scenario-4-small-to-medium-dataset-of-local-files-targeting-sql-server-in-an-azure-virtual-machine"></a><a name="smalllocaltodb"></a>Scénario \#4 : jeu de données petit à moyen de fichiers locaux, ciblant SQL Server dans une machine virtuelle Azure
![Fichiers locaux petits à moyens vers une base de données SQL dans Azure][4]

#### <a name="additional-azure-resources-azure-virtual-machine-sql-server--ipython-notebook-server"></a>Autres ressources Azure : Machine virtuelle Azure (SQL Server / serveur IPython Notebook)
1. Créez une Machine virtuelle Azure exécutant SQL Server + IPython Notebook.
1. Chargez des données dans un conteneur Stockage Azure.
1. Pré-traitez et nettoyez les données dans un conteneur Stockage Azure à l’aide d’IPython Notebook.
1. Si besoin, transformez les données sous forme de tableau nettoyé.
1. Enregistrez les données dans des fichiers locaux de la machine virtuelle (IPython Notebook est en cours d’exécution sur l’ordinateur virtuel, les lecteurs locaux font référence aux lecteurs de machine virtuelle).
1. Chargez des données dans la base de données SQL Server s’exécutant sur une machine virtuelle Azure.
   
   Option \#1 : Avec SQL Server Management Studio.
   
   * Connectez-vous à la machine virtuelle SQL Server.
   * Exécutez SQL Server Management Studio.
   * Créez la base de données et les tables cibles.
   * Utilisez une des méthodes d’importation en bloc pour charger les données à partir des fichiers locaux de la machine virtuelle.
   
   Option \#2 : utilisation d’IPython Notebook – déconseillé pour les jeux de données de tailles moyenne et supérieure
   
   <!-- -->    
   * Utilisez la chaîne de connexion ODBC pour accéder à SQL Server sur la machine virtuelle.
   * Créez la base de données et les tables cibles.
   * Utilisez une des méthodes d’importation en bloc pour charger les données à partir des fichiers locaux de la machine virtuelle.
1. Le cas échéant, explorez des données et créez des fonctionnalités. Les fonctionnalités ne doivent pas être matérialisées dans les tables de base de données. Notez seulement la requête nécessaire pour les créer.
1. Choisissez une taille d’échantillon de données, si nécessaire et/ou souhaité.
1. Connectez-vous à [Azure Machine Learning Studio](https://studio.azureml.net/).
1. Lisez les données directement à partir de SQL Server à l’aide du module [Importer des données][import-data]. Si besoin, collez la requête nécessaire qui extrait les champs, crée les fonctionnalités et échantillonne les données directement dans la requête [Importer des données][import-data].
1. Créez un flux d’expérience Azure Machine Learning commençant par le ou les jeux de données ingérés.

## <a name="scenario-5-large-dataset-in-local-files-target-sql-server-in-azure-vm"></a><a name="largelocaltodb"></a>Scénario \#5 : Jeu de données volumineux dans des fichiers locaux, ciblant SQL Server dans une machine virtuelle Azure
![Fichiers locaux volumineux vers une base de données SQL dans Azure][5]

#### <a name="additional-azure-resources-azure-virtual-machine-sql-server--ipython-notebook-server"></a>Autres ressources Azure : Machine virtuelle Azure (SQL Server / serveur IPython Notebook)
1. Créez une machine virtuelle Azure exécutant SQL Server et le serveur IPython Notebook.
1. Chargez des données dans un conteneur Stockage Azure.
1. (Facultatif) Pré-traitez et nettoyez les données.
   
    a.  Pré-traitez et nettoyez les données dans IPython Notebook, en accédant aux données des objets blob Azure.
   
    b.  Si besoin, transformez les données sous forme de tableau nettoyé.
   
    c.  Enregistrez les données dans des fichiers locaux de la machine virtuelle (IPython Notebook est en cours d’exécution sur l’ordinateur virtuel, les lecteurs locaux font référence aux lecteurs de machine virtuelle).
1. Chargez des données dans la base de données SQL Server s’exécutant sur une machine virtuelle Azure.
   
    a.  Connectez-vous à la machine virtuelle SQL Server.
   
    b.  Si les données n'ont pas encore été enregistrées, téléchargez les fichiers de données à partir d’un conteneur Stockage Azure vers le dossier de machine virtuelle local.
   
    c.  Exécutez SQL Server Management Studio.
   
    d.  Créez la base de données et les tables cibles.
   
    e.  Utilisez l’une des méthodes d’importation en bloc pour charger les données.
   
    f.  Si les jointures de table sont nécessaires, créez des index pour accélérer les jointures.
   
   > [!NOTE]
   > Pour accélérer le chargement des formats de données volumineux, il est recommandé de créer des tables partitionnées et d’importer en bloc les données en parallèle. Pour plus d’informations, consultez la rubrique [Importation de données en parallèle dans des tables partitionnées SQL](parallel-load-sql-partitioned-tables.md).
   > 
   > 
1. Le cas échéant, explorez des données et créez des fonctionnalités. Les fonctionnalités ne doivent pas être matérialisées dans les tables de base de données. Notez seulement la requête nécessaire pour les créer.
1. Choisissez une taille d’échantillon de données, si nécessaire et/ou souhaité.
1. Connectez-vous à [Azure Machine Learning Studio](https://studio.azureml.net/).
1. Lisez les données directement à partir de SQL Server à l’aide du module [Importer des données][import-data]. Si besoin, collez la requête nécessaire qui extrait les champs, crée les fonctionnalités et échantillonne les données directement dans la requête [Importer des données][import-data].
1. Flux d’expérience Azure Machine Learning simple commençant par le jeu de données téléchargé

## <a name="scenario-6-large-dataset-in-a-sql-server-database-on-premises-targeting-sql-server-in-an-azure-virtual-machine"></a><a name="largedbtodb"></a>Scénario \#6 : jeu de données volumineux dans une base de données SQL Server locale, ciblant SQL Server sur une machine virtuelle Azure
![Base de données SQL volumineuse sur site vers une base de données SQL dans Azure][6]

#### <a name="additional-azure-resources-azure-virtual-machine-sql-server--ipython-notebook-server"></a>Autres ressources Azure : Machine virtuelle Azure (SQL Server / serveur IPython Notebook)
1. Créez une machine virtuelle Azure exécutant SQL Server et le serveur IPython Notebook.
1. Utilisez l’une des méthodes d’exportation des données pour exporter les données à partir de SQL Server vers des fichiers de vidage.
   
   > [!NOTE]
   > Si vous décidez de déplacer toutes les données de la base de données locale, il existe une autre méthode (plus rapide) pour déplacer la base de données entière vers l’instance SQL Server dans Azure. Ignorez les étapes d’exportation des données, de création de la base de données et de chargement/importation des données dans la base de données cible, et suivez l’autre méthode.
   > 
   > 
1. Chargez les fichiers de vidage vers le conteneur de Stockage Azure.
1. Chargez les données dans une base de données SQL Server sur une machine virtuelle Azure.
   
   a.  Connectez-vous à la machine virtuelle SQL Server.
   
   b.  Téléchargez les fichiers de données à partir d’un conteneur Stockage Azure vers le dossier de machine virtuelle local.
   
   c.  Exécutez SQL Server Management Studio.
   
   d.  Créez la base de données et les tables cibles.
   
   e.  Utilisez l’une des méthodes d’importation en bloc pour charger les données.
   
   f.  Si les jointures de table sont nécessaires, créez des index pour accélérer les jointures.
   
   > [!NOTE]
   > Pour accélérer le chargement des formats de données volumineux, créez des tables partitionnées et importez en bloc les données en parallèle. Pour plus d’informations, consultez la rubrique [Importation de données en parallèle dans des tables partitionnées SQL](parallel-load-sql-partitioned-tables.md).
   > 
   > 
1. Le cas échéant, explorez des données et créez des fonctionnalités. Les fonctionnalités ne doivent pas être matérialisées dans les tables de base de données. Notez seulement la requête nécessaire pour les créer.
1. Choisissez une taille d’échantillon de données, si nécessaire et/ou souhaité.
1. Connectez-vous à [Azure Machine Learning Studio](https://studio.azureml.net/).
1. Lisez les données directement à partir de SQL Server à l’aide du module [Importer des données][import-data]. Si besoin, collez la requête nécessaire qui extrait les champs, crée les fonctionnalités et échantillonne les données directement dans la requête [Importer des données][import-data].
1. Flux d’expérience Azure Machine Learning simple commençant par le jeu de données téléchargé.

### <a name="alternate-method-to-copy-a-full-database-from-an-on-premises--sql-server-to-azure-sql-database"></a>Autre méthode pour copier une base de données complète à partir d’un serveur SQL local vers une base de données Azure SQL
![Détacher la base de données locale et l’attacher à la base de données SQL dans Azure][7]

#### <a name="additional-azure-resources-azure-virtual-machine-sql-server--ipython-notebook-server"></a>Autres ressources Azure : Machine virtuelle Azure (SQL Server / serveur IPython Notebook)
Pour répliquer l’ensemble de la base de données SQL Server dans votre machine virtuelle SQL Server, vous devez copier une base de données à partir d’un emplacement/serveur vers un autre, en supposant que la base de données puisse être mise temporairement hors connexion. Vous pouvez utiliser l’Explorateur d’objets SQL Server Management Studio ou les commandes Transact-SQL équivalentes.

1. Détachez la base de données à l’emplacement source. Pour plus d’informations, consultez la rubrique [Détacher une base de données](https://technet.microsoft.com/library/ms191491\(v=sql.110\).aspx).
1. Dans l’Explorateur Windows ou l’invite de commandes Windows, copiez les fichiers de la base de données détachée et les fichiers journaux à l’emplacement cible sur la machine virtuelle SQL Server dans Azure.
1. Attachez les fichiers copiés à l’instance SQL Server cible. Pour plus d’informations, consultez [Attach a Database](https://technet.microsoft.com/library/ms190209\(v=sql.110\).aspx).

[Déplacer une base de données à l’aide de la méthode de détachement et d’attachement (Transact-SQL)](https://technet.microsoft.com/library/ms187858\(v=sql.110\).aspx)

## <a name="scenario-7-big-data-in-local-files-target-hive-database-in-azure-hdinsight-hadoop-clusters"></a><a name="largedbtohive"></a>Scénario \#7 : données volumineuses (« Big Data ») dans des fichiers locaux, ciblant une base de données Hive dans des clusters Hadoop Azure HDInsight
![Données volumineuses (« Big Data ») dans la base de données Hive cible locale][9]

#### <a name="additional-azure-resources-azure-hdinsight-hadoop-cluster-and-azure-virtual-machine-ipython-notebook-server"></a>Autres ressources Azure : cluster Hadoop Azure HDInsight et machine virtuelle Azure (serveur IPython Notebook)
1. Créez une machine virtuelle Azure exécutant le serveur IPython Notebook.
1. Créez un cluster Hadoop Azure HDInsight.
1. (Facultatif) Pré-traitez et nettoyez les données.
   
   a.  Pré-traitez et nettoyez les données dans IPython Notebook, en accédant aux données des objets blob Azure.
   
   b.  Si besoin, transformez les données sous forme de tableau nettoyé.
   
   c.  Enregistrez les données dans des fichiers locaux de la machine virtuelle (IPython Notebook est en cours d’exécution sur l’ordinateur virtuel, les lecteurs locaux font référence aux lecteurs de machine virtuelle).
1. Téléchargez des données vers le conteneur par défaut du cluster Hadoop sélectionné à l’étape 2.
1. Chargez des données dans la base de données Hive du cluster Hadoop Azure HDInsight.
   
   a.  Connectez-vous au nœud principal du cluster Hadoop.
   
   b.  Ouvrez la ligne de commande Hadoop.
   
   c.  Entrez le répertoire racine Hive en utilisant la commande `cd %hive_home%\bin` sur la ligne de commande Hadoop.
   
   d.  Exécutez les requêtes Hive pour créer la base de données et les tables, puis chargez des données depuis le stockage d’objets blob vers des tables Hive.
   
   > [!NOTE]
   > Si les données sont volumineuses, les utilisateurs peuvent créer la table Hive avec des partitions. Ils peuvent ensuite utiliser une boucle `for` dans la ligne de commande Hadoop sur le nœud principal pour charger les données dans la table Hive une partition à la fois.
   > 
   > 
1. Le cas échéant, explorez des données et créez des fonctionnalités dans la ligne de commande Hadoop. Les fonctionnalités ne doivent pas être matérialisées dans les tables de base de données. Notez seulement la requête nécessaire pour les créer.
   
   a.  Connectez-vous au nœud principal du cluster Hadoop.
   
   b.  Ouvrez la ligne de commande Hadoop.
   
   c.  Entrez le répertoire racine Hive en utilisant la commande `cd %hive_home%\bin` sur la ligne de commande Hadoop.
   
   d.  Exécutez les requêtes Hive sur la ligne de commande Hadoop sur le nœud principal du cluster Hadoop pour explorer les données et créer des fonctionnalités en fonction des besoins.
1. Si nécessaire et/ou souhaité, échantillonnez les données pour les adapter à Azure Machine Learning Studio.
1. Connectez-vous à [Azure Machine Learning Studio](https://studio.azureml.net/).
1. Lisez les données directement à partir de `Hive Queries` à l’aide du module [Importer des données][import-data]. Si besoin, collez la requête nécessaire qui extrait les champs, crée les fonctionnalités et échantillonne les données directement dans la requête [Importer des données][import-data].
1. Flux d’expérience Azure Machine Learning simple commençant par le jeu de données téléchargé.

## <a name="decision-tree-for-scenario-selection"></a><a name="decisiontree"></a>Arbre de décision pour le choix du scénario
---
Le schéma suivant résume les scénarios décrits ci-dessus et les processus de science des données ainsi que les choix technologiques effectués qui vous guident vers chacun des scénarios détaillés. Le traitement des données, l’exploration, la conception de fonctionnalités et l’échantillonnage peuvent survenir dans un(e) ou plusieurs méthodes/environnements (dans l’environnement source, l’environnement intermédiaire et/ou l’environnement cible) et peuvent s’effectuer de manière itérative en fonction des besoins. Le schéma illustre uniquement les flux possibles et ne fournit pas d’énumération exhaustive.

![Exemples de scénarios de procédure pas à pas pour le processus DS][8]

### <a name="advanced-analytics-in-action-examples"></a>Analyse avancée en action, exemples
Pour connaître les procédures pas à pas de bout en bout pour Azure Machine Learning qui utilisent le processus et la technologie d’analyse avancée à l’aide de jeux de données publics, consultez :

* [Processus TDSP (Team Data Science Process) en action : utilisation de SQL Server](sql-walkthrough.md).
* [Processus TDSP (Team Data Science Process) en action : utilisation de clusters Hadoop HDInsight](hive-walkthrough.md).

[1]: ./media/plan-sample-scenarios/dsp-plan-small-in-aml.png
[2]: ./media/plan-sample-scenarios/dsp-plan-local-with-processing.png
[3]: ./media/plan-sample-scenarios/dsp-plan-local-large.png
[4]: ./media/plan-sample-scenarios/dsp-plan-local-to-db.png
[5]: ./media/plan-sample-scenarios/dsp-plan-large-to-db.png
[6]: ./media/plan-sample-scenarios/dsp-plan-db-to-db.png
[7]: ./media/plan-sample-scenarios/dsp-plan-attach-db.png
[8]: ./media/plan-sample-scenarios/dsp-plan-sample-scenarios.png
[9]: ./media/plan-sample-scenarios/dsp-plan-local-to-hive.png


<!-- Module References -->
[import-data]: /azure/machine-learning/studio-module-reference/import-data