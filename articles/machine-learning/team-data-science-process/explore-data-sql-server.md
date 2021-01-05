---
title: Explorer des données dans une machine virtuelle SQL Server – Team Data Science Process
description: Comment explorer des données stockées dans une machine virtuelle SQL Server sur Azure avec SQL ou un langage de programmation comme Python.
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
ms.openlocfilehash: 33b55afb7796b197f7130ec9288abb01cc115651
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86085648"
---
# <a name="explore-data-in-sql-server-virtual-machine-on-azure"></a>Explorer les données d’une machine virtuelle SQL Server sur Azure

Cet article explique comment explorer les données stockées dans une machine virtuelle SQL Server sur Azure. Utilisez SQL ou Python pour examiner les données.

Cette tâche est une étape du [processus Team Data Science Process](overview.md).

> [!NOTE]
> Les exemples d’instructions SQL qui figurent dans ce document reposent sur l’hypothèse que les données sont stockées dans SQL Server. Dans le cas contraire, reportez-vous à la cartographie du processus de science des données du cloud pour découvrir comment déplacer vos données vers SQL Server.
> 
> 

## <a name="explore-sql-data-with-sql-scripts"></a><a name="sql-dataexploration"></a>Explorer les données SQL avec des scripts SQL
Voici quelques exemples de scripts SQL utilisables pour l’exploration de magasins de données dans SQL Server.

1. Obtenir le nombre d’observations par jour
   
    `SELECT CONVERT(date, <date_columnname>) as date, count(*) as c from <tablename> group by CONVERT(date, <date_columnname>)` 
2. Obtenir les niveaux dans une colonne catégorielle
   
    `select  distinct <column_name> from <databasename>`
3. Obtenir le nombre de niveaux en combinant deux colonnes catégorielles 
   
    `select <column_a>, <column_b>,count(*) from <tablename> group by <column_a>, <column_b>`
4. Obtenir la distribution relative aux colonnes numériques
   
    `select <column_name>, count(*) from <tablename> group by <column_name>`

> [!NOTE]
> Pour découvrir un exemple pratique, vous pouvez utiliser le [jeu de données des taxis new-yorkais NYC Taxi](https://www.andresmh.com/nyctaxitrips/) et vous reporter au notebook IPython intitulé [NYC Data wrangling using IPython Notebook and SQL Server](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Misc/DataScienceProcess/iPythonNotebooks/machine-Learning-data-science-process-sql-walkthrough.ipynb) (Retraitement des données de New-York City à l’aide de Notebook IPython et de SQL Server) pour connaître la procédure pas à pas.
> 
> 

## <a name="explore-sql-data-with-python"></a><a name="python"></a>Explorer les données SQL avec Python
L’utilisation de Python pour explorer les données et générer des fonctionnalités quand les données sont stockées dans SQL Server est comparable au traitement des données dans l’objet blob Azure à l’aide de Python comme expliqué dans [Traiter les données Azure Blob dans votre environnement de science des données](data-blob.md). Chargez les données à partir de la base de données dans une trame de données pandas, puis laissez-les faire l’objet d’un traitement complémentaire. Nous décrivons dans cette section le processus de connexion à la base de données et de chargement des données dans la trame de données.

Le format de chaîne de connexion ci-après vous permet de vous connecter à une base de données SQL Server à partir de Python à l’aide de pyodbc (en remplaçant les variables servername, dbname, username et password par les valeurs qui vous correspondent) :

```python
#Set up the SQL Azure connection
import pyodbc    
conn = pyodbc.connect('DRIVER={SQL Server};SERVER=<servername>;DATABASE=<dbname>;UID=<username>;PWD=<password>')
```

La [bibliothèque Pandas](https://pandas.pydata.org/) de Python offre un ensemble élaboré de structures de données et d’outils d’analyse des données pour la manipulation des données dans le cadre d’une programmation en Python. Le code suivant lit les résultats renvoyés par une base de données SQL Server dans une trame de données Pandas :

```python
# Query database and load the returned results in pandas data frame
data_frame = pd.read_sql('''select <columnname1>, <columnname2>... from <tablename>''', conn)
```

Vous pouvez à présent utiliser la trame de données Pandas comme décrit dans la rubrique [Traiter les données Azure Blob dans votre environnement de science des données](data-blob.md).

## <a name="the-team-data-science-process-in-action-example"></a>Exemple de processus TDSP (Team Data Science Process) en action
Pour obtenir un exemple de procédure pas à pas complet du processus Cortana Analytics à l’aide d’un jeu de données public, consultez [Processus TDSP (Team Data Science Process) en action : utilisation de SQL Server](sql-walkthrough.md).

