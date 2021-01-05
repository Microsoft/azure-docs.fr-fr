---
title: Explorer les données d’une machine virtuelle SQL Server – Team Data Science Process
description: Explorer des données de traitement supplémentaires et générer des fonctionnalités à l’aide de Python ou de SQL dans une machine virtuelle SQL Server sur Azure.
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
ms.openlocfilehash: 8be878cf40967356d68e9be0765e898c81b5ba0a
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93314631"
---
# <a name="process-data-in-sql-server-virtual-machine-on-azure"></a><a name="heading"></a>Traitement des données d’une machine virtuelle SQL Server sur Azure
Ce document décrit l'exploration des données et la génération de fonctionnalités pour les données stockées dans une machine virtuelle SQL Server sur Azure. Cet objectif peut être atteint par data wrangling à l’aide de SQL ou en utilisant un langage de programmation comme Python.

> [!NOTE]
> Les exemples d’instructions SQL qui figurent dans ce document reposent sur l’hypothèse que les données sont stockées dans SQL Server. Dans le cas contraire, reportez-vous à la cartographie du processus de science des données du cloud pour découvrir comment déplacer vos données vers SQL Server.
> 
> 

## <a name="using-sql"></a><a name="SQL"></a>Utilisation de SQL
Dans cette section, nous décrivons les tâches de retraitement des données via SQL ci-après :

1. [Exploration des données](#sql-dataexploration)
2. [Génération de fonctionnalités](#sql-featuregen)

### <a name="data-exploration"></a><a name="sql-dataexploration"></a>Exploration des données
Voici quelques exemples de scripts SQL utilisables pour l’exploration de magasins de données dans SQL Server.

> [!NOTE]
> Pour découvrir un exemple pratique, vous pouvez utiliser le [jeu de données des taxis new-yorkais NYC Taxi](https://www.andresmh.com/nyctaxitrips/) et vous reporter au notebook IPython intitulé [NYC Data wrangling using IPython Notebook and SQL Server](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/iPythonNotebooks/machine-Learning-data-science-process-sql-walkthrough.ipynb) (Retraitement des données de New-York City à l’aide de Notebook IPython et de SQL Server) pour connaître la procédure pas à pas.
> 
> 

1. Obtenir le nombre d’observations par jour
   
    `SELECT CONVERT(date, <date_columnname>) as date, count(*) as c from <tablename> group by CONVERT(date, <date_columnname>)` 
2. Obtenir les niveaux dans une colonne catégorielle
   
    `select  distinct <column_name> from <databasename>`
3. Obtenir le nombre de niveaux en combinant deux colonnes catégorielles 
   
    `select <column_a>, <column_b>,count(*) from <tablename> group by <column_a>, <column_b>`
4. Obtenir la distribution relative aux colonnes numériques
   
    `select <column_name>, count(*) from <tablename> group by <column_name>`

### <a name="feature-generation"></a><a name="sql-featuregen"></a>Génération de fonctionnalités
Dans cette section, nous décrivons plusieurs manières de générer des fonctionnalités via SQL :  

1. [Génération de fonctionnalités utilisant des décomptes](#sql-countfeature)
2. [Génération de caractéristiques de compartimentage](#sql-binningfeature)
3. [Déploiement des caractéristiques à partir d’une seule colonne](#sql-featurerollout)

> [!NOTE]
> Une fois que vous avez généré des fonctionnalités supplémentaires, vous pouvez soit les ajouter sous forme de colonnes à la table existante, soit créer une autre table avec les fonctionnalités supplémentaires et la clé primaire que vous pouvez joindre à la table d’origine. 
> 
> 

### <a name="count-based-feature-generation"></a><a name="sql-countfeature"></a>Génération de fonctionnalités utilisant des décomptes
Les exemples suivants décrivent deux manières de générer des fonctionnalités utilisant des décomptes. La première méthode a recours à une somme conditionnelle, tandis que la seconde méthode utilise la clause « where ». Ces résultats peuvent ensuite être joints à la table d’origine (à l’aide des colonnes de clé primaire) pour disposer de fonctionnalités de décompte parallèlement aux données d’origine.

```sql
select <column_name1>,<column_name2>,<column_name3>, COUNT(*) as Count_Features from <tablename> group by <column_name1>,<column_name2>,<column_name3> 

select <column_name1>,<column_name2> , sum(1) as Count_Features from <tablename> 
where <column_name3> = '<some_value>' group by <column_name1>,<column_name2> 
```

### <a name="binning-feature-generation"></a><a name="sql-binningfeature"></a>Génération de caractéristiques de compartimentage
L’exemple ci-dessous illustre comment générer des fonctionnalités compartimentées en divisant (à l’aide de cinq emplacements) une colonne numérique qui peut être plutôt utilisée sous la forme d’une fonctionnalité :

```sql
SELECT <column_name>, NTILE(5) OVER (ORDER BY <column_name>) AS BinNumber from <tablename>
```

### <a name="rolling-out-the-features-from-a-single-column"></a><a name="sql-featurerollout"></a>Déploiement des caractéristiques à partir d’une seule colonne
Dans cette section, nous décrivons comment déployer une seule colonne dans une table afin de générer des fonctionnalités supplémentaires. Cet exemple présuppose l’existence d’une colonne de latitude ou de longitude dans la table à partir de laquelle vous essayez de générer des fonctionnalités.

Voici une petite présentation des données de latitude/longitude issue du site stackoverflow : [How to measure the accuracy of latitude and longitude?](https://gis.stackexchange.com/questions/8650/how-to-measure-the-accuracy-of-latitude-and-longitude)(Comment mesurer la précision de la latitude et de la longitude). Il est utile de comprendre ces conseils avant d’inclure l’emplacement comme une ou plusieurs fonctionnalités :

* Le signe indique si nous nous trouvons au nord, au sud, à l’est ou à l’ouest.
* Un chiffre des centaines différent de zéro indique que nous utilisons la longitude, et non la latitude.
* Le chiffre des dizaines équivaut à environ 1 000 kilomètres. Il nous fournit des informations utiles sur le continent ou l’océan dans lequel nous nous trouvons.
* Le chiffre des unités (un degré décimal) équivaut à 111 kilomètres maximum (60 milles marins, soit environ 69 milles terrestres). Il peut vous indiquer à peu près l’état, le pays ou la région où vous vous trouvez.
* La première décimale équivaut à 11,1 km maximum : elle permet de distinguer la position d’une grande ville de celle d’une autre grande localité voisine.
* La deuxième décimale équivaut à 1,1 km maximum : elle permet de différencier un village du suivant.
* La troisième décimale équivaut à 110 m maximum : elle permet d’identifier un domaine agricole ou un campus universitaire de grande taille.
* La quatrième décimale correspond à 11 m maximum : elle permet d’identifier une parcelle de terre. Cette information est comparable à la précision classique d’un appareil GPS non corrigé sans aucune interférence.
* La cinquième décimale équivaut à 1,1 m maximum : elle permet de distinguer un arbre d’un autre. Un tel niveau de précision sur les appareils GPS commerciaux ne peut être atteint qu’au moyen d’une correction différentielle.
* La sixième décimale équivaut à 0,11 m maximum : vous pouvez notamment l’utiliser pour représenter des structures en détail, pour concevoir des plans d’aménagement paysager et pour construire des routes. Elle devrait se révéler amplement suffisante pour assurer le suivi des mouvements des glaciers et des rivières. L’obtention d’une telle précision sur un GPS nécessite l’emploi de mesures rigoureuses, telles qu’une correction différentielle.

Vous pouvez implémenter les informations de localisation comme illustré ci-dessous, en les répartissant par région, par emplacement et par ville. Vous pouvez aussi appeler un point de terminaison REST, tel que l’API Bing Maps disponible dans [Rechercher un emplacement par point](/bingmaps/rest-services/locations/find-a-location-by-point), pour obtenir des informations sur la région/le secteur.

```sql
select 
    <location_columnname>
    ,round(<location_columnname>,0) as l1        
    ,l2=case when LEN (PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1)) >= 1 then substring(PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1),1,1) else '0' end     
    ,l3=case when LEN (PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1)) >= 2 then substring(PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1),2,1) else '0' end     
    ,l4=case when LEN (PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1)) >= 3 then substring(PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1),3,1) else '0' end     
    ,l5=case when LEN (PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1)) >= 4 then substring(PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1),4,1) else '0' end     
    ,l6=case when LEN (PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1)) >= 5 then substring(PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1),5,1) else '0' end     
    ,l7=case when LEN (PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1)) >= 6 then substring(PARSENAME(round(ABS(<location_columnname>) - FLOOR(ABS(<location_columnname>)),6),1),6,1) else '0' end     
from <tablename>
```

Vous pouvez en outre exploiter ces fonctionnalités de localisation pour générer d’autres fonctionnalités utilisant des décomptes comme décrit précédemment. 

> [!TIP]
> Vous pouvez insérer les enregistrements par programmation en utilisant le langage de votre choix. Vous aurez peut-être besoin d’insérer les données dans des blocs pour améliorer l’efficacité des écritures (pour obtenir un exemple illustrant la marche à suivre, consultez cet [exemple HelloWorld qui montre comment accéder à SQL Server avec python](https://code.google.com/p/pypyodbc/wiki/A_HelloWorld_sample_to_access_mssql_with_python)). Une autre solution consiste à insérer les données dans la base de données à l’aide de l’ [utilitaire BCP](/sql/tools/bcp-utility).
> 
> 

### <a name="connecting-to-azure-machine-learning"></a><a name="sql-aml"></a>Connexion à Azure Machine Learning
La fonctionnalité que vous venez de générer peut être ajoutée sous la forme d’une colonne à une table existante ou stockée dans une nouvelle table et associée à la table d’origine pour l’apprentissage automatique. Vous pouvez générer des fonctionnalités ou y accéder si elles sont déjà créées à l’aide du module [Importer des données][import-data] dans Azure Machine Learning comme expliqué ci-dessous :

![lecteurs azureml][1] 

## <a name="using-a-programming-language-like-python"></a><a name="python"></a>Utilisation d’un langage de programmation tel que Python
L’utilisation de Python pour explorer les données et générer des fonctionnalités quand les données sont stockées dans SQL Server est comparable au traitement des données dans l’objet blob Azure à l’aide de Python comme expliqué dans [Traiter les données Azure Blob dans votre environnement de science des données](data-blob.md). Chargez les données issues de la base de données dans une trame de données pandas pour un traitement plus approfondi. Nous décrivons dans cette section le processus de connexion à la base de données et de chargement des données dans la trame de données.

Le format de chaîne de connexion ci-après vous permet de vous connecter à une base de données SQL Server à partir de Python à l’aide de pyodbc (en remplaçant les variables servername, dbname, username et password par les valeurs qui vous correspondent) :

```python
#Set up the SQL Azure connection
import pyodbc    
conn = pyodbc.connect('DRIVER={SQL Server};SERVER=<servername>;DATABASE=<dbname>;UID=<username>;PWD=<password>')
```

La [bibliothèque Pandas](https://pandas.pydata.org/) de Python offre un ensemble élaboré de structures de données et d’outils d’analyse des données pour la manipulation des données dans le cadre d’une programmation en Python. Le code ci-après lit les résultats renvoyés par une base de données SQL Server dans une trame de données pandas :

```python
# Query database and load the returned results in pandas data frame
data_frame = pd.read_sql('''select <columnname1>, <columnname2>... from <tablename>''', conn)
```

Vous pouvez à présent utiliser la trame de données Pandas comme décrit dans l’article [Traiter les données Azure Blob dans votre environnement de science des données](data-blob.md).

## <a name="azure-data-science-in-action-example"></a>Exemple de processus de science des données Azure en action
Pour découvrir un exemple de procédure pas à pas du processus de science des données Azure à l’aide d’un jeu de données public, consultez la rubrique [Processus de science des données Azure en action](sql-walkthrough.md).

[1]: ./media/sql-server-virtual-machine/reader_db_featurizedinput.png


<!-- Module References -->
[import-data]: /azure/machine-learning/studio-module-reference/import-data