---
title: Échantillonner des données dans SQL Server sur Azure - Team Data Science Process
description: Échantillonnez des données stockées dans SQL Server sur Azure à l’aide de SQL ou du langage de programmation Python, puis déplacez-les vers Azure Machine Learning.
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
ms.openlocfilehash: 7ac1fc5688dad3406041f36ff858e6fd27c7272f
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93321864"
---
# <a name="sample-data-in-sql-server-on-azure"></a><a name="heading"></a>Échantillonner des données dans SQL Server sur Azure

Cet article montre comment échantillonner des données stockées dans SQL Server sur Azure à l’aide de SQL ou du langage de programmation Python. Il montre également comment déplacer les données échantillonnées vers Azure Machine Learning en les enregistrant dans un fichier, en les chargeant vers un objet blob Azure, puis en les lisant dans Azure Machine Learning Studio.

L’échantillonnage Python utilise la bibliothèque ODBC [pyodbc](https://code.google.com/p/pyodbc/) pour se connecter à SQL Server sur Azure et la bibliothèque [Pandas](https://pandas.pydata.org/) pour effectuer l’échantillonnage proprement dit.

> [!NOTE]
> L’exemple de code SQL figurant dans ce document repose sur l’hypothèse que les données sont stockées dans SQL Server sur Azure. Si ce n’est pas le cas, consultez l’article [Déplacer des données vers SQL Server sur Azure](move-sql-server-virtual-machine.md) pour savoir comment déplacer vos données vers SQL Server sur Azure.
> 
> 

**Pourquoi échantillonner vos données ?**
Si vous prévoyez d’analyser un jeu de données volumineux, il est généralement recommandé de sous-échantillonner les données afin de réduire leur taille sous une forme plus facilement exploitable, mais toujours représentative. L’échantillonnage facilite la compréhension des données, l’exploration ainsi que l’ingénierie de fonctionnalités. Son rôle dans le [processus TDSP (Team Data Science Process)](./index.yml) consiste à permettre le prototypage rapide des fonctions de traitement des données et des modèles d’apprentissage automatique.

Cette tâche d’échantillonnage est une étape du [processus TDSP (Team Data Science Process)](./index.yml).

## <a name="using-sql"></a><a name="SQL"></a>Utilisation de SQL
Cette section décrit différentes méthodes permettant d’effectuer un échantillonnage aléatoire simple des données de la base de données via SQL. Choisissez une méthode en fonction de la taille de vos données et de leur distribution.

Les deux options suivantes indiquent comment utiliser `newid` dans SQL Server pour procéder à l’échantillonnage. La méthode que vous choisissez dépend du degré aléatoire qui doit caractériser l’échantillon (dans l’exemple de code suivant, l’élément pk_id est supposé correspondre à une clé primaire générée automatiquement).

1. Échantillon aléatoire moins strict

    ```sql
    select  * from <table_name> where <primary_key> in 
    (select top 10 percent <primary_key> from <table_name> order by newid())
    ```

2. Échantillon plus aléatoire 

    ```sql
    SELECT * FROM <table_name>
    WHERE 0.1 >= CAST(CHECKSUM(NEWID(), <primary_key>) & 0x7fffffff AS float)/ CAST (0x7fffffff AS int)
    ```

Vous pouvez aussi utiliser Tablesample pour l’échantillonnage des données. Cette option peut constituer une meilleure approche si vos données sont volumineuses (en supposant que les données figurant sur des pages différentes ne sont pas corrélées) et que vous souhaitez que la requête s’exécute dans un délai acceptable.

```sql
SELECT *
FROM <table_name> 
TABLESAMPLE (10 PERCENT)
```

> [!NOTE]
> Vous pouvez explorer et générer des fonctionnalités à partir de ces données échantillonnées en les stockant dans une nouvelle table.
> 
> 

### <a name="connecting-to-azure-machine-learning"></a><a name="sql-aml"></a>Connexion à Azure Machine Learning
Vous pouvez utiliser directement les exemples de requêtes ci-dessus dans le module [Importer les données][import-data] d’Azure Machine Learning afin de sous-échantillonner les données à la volée et de les importer dans une expérience Azure Machine Learning. La capture d’écran ci-dessous illustre l’utilisation du module lecteur pour lire les données échantillonnées :

![lecteur sql][1]

## <a name="using-the-python-programming-language"></a><a name="python"></a>Utilisation du langage de programmation Python
Cette section décrit l’utilisation de la [bibliothèque pyodbc](https://code.google.com/p/pyodbc/) pour établir une connexion ODBC à une base de données SQL Server dans Python. La chaîne de connexion de base de données se présente comme suit (remplacez les variables servername, dbname, username et password par les valeurs de votre configuration) :

```python
#Set up the SQL Azure connection
import pyodbc    
conn = pyodbc.connect('DRIVER={SQL Server};SERVER=<servername>;DATABASE=<dbname>;UID=<username>;PWD=<password>')
```

La bibliothèque [Pandas](https://pandas.pydata.org/) de Python offre un ensemble élaboré de structures de données et d’outils d’analyse des données pour la manipulation des données dans le cadre d’une programmation en Python. Le code suivant lit un échantillon de 0,1 % des données d’une table d’Azure SQL Database dans une trame de données Pandas :

```python
import pandas as pd

# Query database and load the returned results in pandas data frame
data_frame = pd.read_sql('''select column1, column2... from <table_name> tablesample (0.1 percent)''', conn)
```

Vous pouvez à présent travailler sur les données échantillonnées dans la trame de données pandas. 

### <a name="connecting-to-azure-machine-learning"></a><a name="python-aml"></a>Connexion à Azure Machine Learning
Vous pouvez utiliser l’exemple de code ci-après pour enregistrer les données sous-échantillonnées dans un fichier et les charger dans un objet blob Azure. Les données figurant dans le blob peuvent être lues directement dans une expérimentation Azure Machine Learning à l’aide du module [Importer les données][import-data]. La procédure comporte trois étapes : 

1. Écrire la trame de données pandas dans un fichier local

    ```python
    dataframe.to_csv(os.path.join(os.getcwd(),LOCALFILENAME), sep='\t', encoding='utf-8', index=False)
    ```

2. Charger le fichier local dans un objet blob Azure

    ```python
    from azure.storage import BlobService
    import tables

    STORAGEACCOUNTNAME= <storage_account_name>
    LOCALFILENAME= <local_file_name>
    STORAGEACCOUNTKEY= <storage_account_key>
    CONTAINERNAME= <container_name>
    BLOBNAME= <blob_name>

    output_blob_service=BlobService(account_name=STORAGEACCOUNTNAME,account_key=STORAGEACCOUNTKEY)    
    localfileprocessed = os.path.join(os.getcwd(),LOCALFILENAME) #assuming file is in current working directory

    try:

    #perform upload
    output_blob_service.put_block_blob_from_path(CONTAINERNAME,BLOBNAME,localfileprocessed)

    except:            
        print ("Something went wrong with uploading blob:"+BLOBNAME)
    ```

3. Lisez les données du blob Azure à l’aide du module [Importer les données][import-data] d’Azure Machine Learning, comme l’illustre la capture d’écran ci-dessous :

![objet blob de lecteur][2]

## <a name="the-team-data-science-process-in-action-example"></a>Exemple de processus TDSP (Team Data Science Process) en action
Pour obtenir un exemple de procédure pas à pas du processus TDSP (Team Data Science Process) à l’aide d’un jeu de données public, consultez [Processus TDSP (Team Data Science Process) en action : utilisation de SQL Server](sql-walkthrough.md).

[1]: ./media/sample-sql-server-virtual-machine/reader_database.png
[2]: ./media/sample-sql-server-virtual-machine/reader_blob.png

[import-data]: /azure/machine-learning/studio-module-reference/import-data