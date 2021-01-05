---
title: Créer des tables Hive et charger des données à partir de Stockage Blob | Team Data Science Process
description: Utilisez des requêtes Hive pour créer des tables Hive et charger des données à partir de Stockage Blob Azure. Partitionnez des tables Hive et utilisez le format ORC (Optimized Row Columnar) pour améliorer les performances des requêtes.
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
ms.openlocfilehash: 5d61c0f5f26bc46b9c4a5bc4a793df1e10710004
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96006727"
---
# <a name="create-hive-tables-and-load-data-from-azure-blob-storage"></a>Créer des tables Hive et charger des données à partir de Stockage Blob Azure

Cet article présente des requêtes Hive génériques qui créent des tables Hive et chargent des données à partir d’un stockage d’objets blob Azure. Il donne également quelques conseils sur le partitionnement des tables Hive et sur l’utilisation du format ORC (Optimized Row Columnar) pour améliorer les performances des requêtes.

## <a name="prerequisites"></a>Prérequis
Cet article suppose que vous avez :

* Créé un compte de stockage Azure. Pour obtenir des instructions, consultez [À propos des comptes de stockage Azure](../../storage/common/storage-introduction.md).
* Approvisionné un cluster Hadoop personnalisé avec le service HDInsight.  Si vous avez besoin d’aide, consultez [Configurer des clusters dans HDInsight](../../hdinsight/hdinsight-hadoop-provision-linux-clusters.md).
* Activé l’accès à distance au cluster, saisi les identifiants appropriés et ouvert la console de ligne de commande Hadoop. Si vous avez besoin d’aide, consultez [Gérer des clusters Apache Hadoop](../../hdinsight/hdinsight-administer-use-portal-linux.md).

## <a name="upload-data-to-azure-blob-storage"></a>Téléchargement de données vers le stockage d’objets blob Azure
Si vous avez créé une machine virtuelle Azure en suivant les instructions de l’article [Configurer une machine virtuelle Azure pour l’analyse avancée](../../machine-learning/data-science-virtual-machine/overview.md), ce fichier de script doit avoir été téléchargé dans le répertoire *C:\\Utilisateurs\\\<user name\>\\Documents\\Data Science Scripts* sur la machine virtuelle. Pour pouvoir être envoyées, ces requêtes Hive nécessitent simplement que vous fournissiez un schéma de données et une configuration de stockage de blobs Azure dans les champs appropriés.

Nous partons du principe que les données des tables Hive ont un format tabulaire **non compressé** et qu’elles ont été chargées dans le conteneur par défaut (ou un conteneur supplémentaire) du compte de stockage utilisé par le cluster Hadoop.

Si vous souhaitez vous exercer avec l’exemple **NYC Taxi Trip Data**, vous devez :

* **télécharger** les 24 fichiers [NYC Taxi Trip Data](https://www.andresmh.com/nyctaxitrips) (12 fichiers Trip et 12 fichiers Fare),
* **décompresser** tous les fichiers en fichiers .csv, puis
* les **télécharger** à l’emplacement par défaut (ou un conteneur approprié) du compte de stockage Azure. Les options de ce type de compte sont présentées dans la rubrique [Utiliser Stockage Azure avec des clusters Azure HDInsight](../../hdinsight/hdinsight-hadoop-use-blob-storage.md). Pour découvrir le processus qui vous permet de télécharger les fichiers .csv du conteneur par défaut sur le compte de stockage, consultez cette [page](hive-walkthrough.md#upload).

## <a name="how-to-submit-hive-queries"></a><a name="submit"></a>Envoi de requêtes Hive
Pour envoyer des requêtes Hive, utilisez au choix :

* [Envoyer des requêtes Hive avec la ligne de commande Hadoop dans le nœud principal du cluster Hadoop](#headnode)
* [Envoyer des requêtes Hive avec l'éditeur Hive](#hive-editor)
* [Envoyer des requêtes Hive avec les commandes Azure PowerShell](#ps)

Des requêtes Hive sont similaires à SQL. Si vous maîtrisez SQL, l’ [aide-mémoire Hive pour utilisateurs SQL](https://hortonworks.com/wp-content/uploads/2013/05/hql_cheat_sheet.pdf) peut vous être utile.

Lors de l’envoi d’une requête Hive, vous pouvez également contrôler la destination de sa sortie : écran, fichier local sur le nœud principal ou blob Azure.

### <a name="submit-hive-queries-through-hadoop-command-line-in-headnode-of-hadoop-cluster"></a><a name="headnode"></a>Envoyer des requêtes Hive avec la ligne de commande Hadoop dans le nœud principal du cluster Hadoop
Une requête Hive complexe envoyée directement au nœud principal du cluster Hadoop est traitée plus rapidement qu'avec un éditeur Hive ou des scripts Azure PowerShell.

Connectez-vous au nœud principal du cluster Hadoop, ouvrez la ligne de commande Hadoop sur le bureau du nœud principal et saisissez la commande `cd %hive_home%\bin`.

Vous disposez de trois possibilités pour envoyer des requêtes Hive dans la ligne de commande Hadoop :

* directement ;
* à l’aide de fichiers « .hql » ;
* à l’aide de la console de commande Hive.

#### <a name="submit-hive-queries-directly-in-hadoop-command-line"></a>Envoyer directement des requêtes Hive dans la ligne de commande Hadoop
Vous pouvez exécuter une commande du type `hive -e "<your hive query>;` pour envoyer une requête Hive simple directement dans la ligne de commande Hadoop. Voici un exemple, où l’encadré rouge indique la commande qui envoie la requête Hive et l’encadré vert, la sortie de la requête Hive.

![Commande d’envoi de requête Hive avec la sortie correspondante](./media/move-hive-tables/run-hive-queries-1.png)

#### <a name="submit-hive-queries-in-hql-files"></a>Envoyer des requêtes Hive dans des fichiers « .hql »
Lorsque la requête Hive est plus complexe et comporte plusieurs lignes, la modifier dans la ligne de commande ou la console de commande Hive n’est pas simple. L’alternative consiste à utiliser un éditeur de texte dans le nœud principal du cluster Hadoop pour enregistrer la requête Hive dans un fichier « .hql » situé dans un répertoire local du nœud principal. Ensuite, celle-ci peut être envoyée dans le fichier « .hql » à l’aide de l’argument `-f` , comme indiqué ci-dessous :

```console
hive -f "<path to the '.hql' file>"
```

![Requête Hive dans un fichier « .hql »](./media/move-hive-tables/run-hive-queries-3.png)

**Supprimer l’affichage de l’état d’avancement des requêtes Hive**

Par défaut, après l’envoi d’une requête Hive dans la ligne de commande Hadoop, l’état d’avancement de la tâche Map/Reduce s’affiche à l’écran. Pour supprimer cet affichage, déclarez l'argument `-S` (« S » en majuscule) dans la ligne de commande, comme indiqué ci-dessous :

```console
hive -S -f "<path to the '.hql' file>"
hive -S -e "<Hive queries>"
```

#### <a name="submit-hive-queries-in-hive-command-console"></a>Envoyer des requêtes Hive dans la console de commande Hive
Vous pouvez également ouvrir la console de commande Hive en exécutant la commande `hive` dans la ligne de commande Hadoop, puis envoyer les requêtes Hive dans la console de commande Hive. Voici un exemple. Ici, les deux encadrés rouges indiquent les commandes utilisées pour ouvrir la console de commande Hive et envoyer la requête Hive dans cette console. L’encadré vert montre la sortie de la requête Hive.

![Ouvrez la console de commande Hive et entrez la commande, puis affichez la sortie de la requête Hive](./media/move-hive-tables/run-hive-queries-2.png)

Les exemples précédents affichent directement les résultats de la requête à l’écran. Vous pouvez également consigner la sortie dans un fichier local sur le nœud principal ou dans un blob Azure. Puis, vous pouvez utiliser d’autres outils pour analyser plus finement la sortie de la requête Hive.

**Enregistrer les résultats d’une requête Hive dans un fichier local**
Pour enregistrer les résultats d’une requête Hive dans un répertoire local du nœud principal, vous devez envoyer celle-ci dans la ligne de commande Hadoop, comme indiqué ci-dessous :

```console
hive -e "<hive query>" > <local path in the head node>
```

Dans l'exemple suivant, la sortie de la requête Hive est consignée dans un fichier `hivequeryoutput.txt` situé dans le répertoire `C:\apps\temp`.

![Capture d’écran affichant la sortie de la requête Hive dans une fenêtre de ligne de commande Hadoop.](./media/move-hive-tables/output-hive-results-1.png)

**Enregistrer les résultats d’une requête Hive dans un blob Azure**

Vous pouvez également enregistrer les résultats d’une requête Hive dans un blob Azure situé dans le conteneur par défaut du cluster Hadoop. La requête Hive se présente comme suit :

```console
insert overwrite directory wasb:///<directory within the default container> <select clause from ...>
```

Dans l'exemple suivant, la sortie de la requête Hive est consignée dans le répertoire de blob `queryoutputdir` situé dans le conteneur par défaut du cluster Hadoop. Ici, il suffit d’indiquer le nom du répertoire, sans celui du blob. Une erreur est générée si vous déclarez le nom du répertoire et celui du blob, comme dans `wasb:///queryoutputdir/queryoutput.txt`.

![Capture d’écran affichant la commande précédente dans la fenêtre de ligne de commande Hadoop.](./media/move-hive-tables/output-hive-results-2.png)

Si vous ouvrez le conteneur par défaut du cluster Hadoop à l’aide d’Azure Storage Explorer, la requête Hive renvoie le résultat affiché dans la figure suivante. Vous pouvez utiliser le filtre (encadré rouge) pour retrouver un blob dont le nom comporte les lettres spécifiées.

![Explorateur Stockage Azure montrant la sortie de la requête Hive](./media/move-hive-tables/output-hive-results-3.png)

### <a name="submit-hive-queries-with-the-hive-editor"></a><a name="hive-editor"></a>Envoyer des requêtes Hive avec l’éditeur Hive
Vous pouvez aussi utiliser la console de requête (éditeur Hive) en entrant une URL sous la forme *https:\//\<Hadoop cluster name>.azurehdinsight.net/Home/HiveEditor* dans un navigateur web. Vous devez être connecté pour afficher cette console, et vous devez donc saisir ici vos informations d’identification de cluster Hadoop.

### <a name="submit-hive-queries-with-azure-powershell-commands"></a><a name="ps"></a>Envoyer des requêtes Hive avec les commandes Azure PowerShell
Vous pouvez également utiliser PowerShell pour envoyer des requêtes Hive. Pour obtenir de l'aide, consultez [Envoi de tâches Hive avec PowerShell](../../hdinsight/hadoop/apache-hadoop-use-hive-powershell.md).

## <a name="create-hive-database-and-tables"></a><a name="create-tables"></a>Créer la base de données et les tables Hive
Les requêtes Hive sont disponibles en téléchargement dans le [dépôt GitHub](https://github.com/Azure/Azure-MachineLearning-DataScience/tree/master/Misc/DataScienceProcess/DataScienceScripts/sample_hive_create_db_tbls_load_data_generic.hql).

Voici la requête Hive qui crée une table Hive.

```hiveql
create database if not exists <database name>;
CREATE EXTERNAL TABLE if not exists <database name>.<table name>
(
    field1 string,
    field2 int,
    field3 float,
    field4 double,
    ...,
    fieldN string
)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '<field separator>' lines terminated by '<line separator>'
STORED AS TEXTFILE LOCATION '<storage location>' TBLPROPERTIES("skip.header.line.count"="1");
```

Voici les descriptions des champs que vous devez renseigner et d’autres opérations de configuration :

* **\<database name\>**  : nom de la base de données que vous souhaitez créer. Si vous voulez utiliser la base de données par défaut, la requête « *create database...*  » peut être omise.
* **\<table name\>**  : nom de la table que vous souhaitez créer dans la base de données spécifiée. Si vous voulez utiliser la base de données par défaut, la table peut être désignée directement par *\<table name\>* sans \<database name\>.
* **\<field separator\>**  : séparateur qui délimite les champs dans le fichier de données à charger dans la table Hive.
* **\<line separator\>**  : séparateur qui délimite les lignes dans le fichier de données.
* **\<storage location\>**  : emplacement de Stockage Azure où enregistrer les données des tables Hive. Si vous ne spécifiez pas *LOCATION\<storage location\>* , la base de données et les tables sont stockées par défaut dans le répertoire *hive/warehouse/* du conteneur par défaut du cluster Hive. Si vous souhaitez spécifier l’emplacement de stockage, ce dernier doit se trouver dans le conteneur par défaut de la base de données et des tables. Cet emplacement doit être désigné comme emplacement par rapport au conteneur par défaut du cluster sous la forme *’wasb:///\<directory 1>/’* ou *’wasb:///\<directory 1>/\<directory 2>/’* , etc. Une fois la requête exécutée, les répertoires relatifs sont créés dans le conteneur par défaut.
* **TBLPROPERTIES("skip.header.line.count"="1")** : si le fichier de données contient une ligne d’en-tête, vous devez ajouter cette propriété **à la fin** de la requête *create table*. Sinon, cette ligne d’en-tête est chargée comme un enregistrement dans la table. Si le fichier de données ne contient aucune ligne d’en-tête, cette configuration peut être omise dans la requête.

## <a name="load-data-to-hive-tables"></a><a name="load-data"></a>Chargement des données dans des tables Hive
Voici la requête Hive qui charge les données dans une table Hive.

```hiveql
LOAD DATA INPATH '<path to blob data>' INTO TABLE <database name>.<table name>;
```

* **\<path to blob data\>**  : si le fichier blob à charger dans la table Hive se trouve dans le conteneur par défaut du cluster Hadoop HDInsight, le *\<path to blob data\>* doit se présenter sous la forme *’wasb:///\<directory in this container>/\<blob file name>’* . Le fichier blob peut également se trouver dans un autre conteneur du cluster Hadoop HDInsight. Dans ce cas, *\<path to blob data\>* doit se présenter sous la forme *’wasb://\<container name>@\<storage account name>.blob.core.windows.net/\<blob file name>’* .

  > [!NOTE]
  > Les données blob à charger dans la table Hive doivent se trouver dans le conteneur par défaut ou un autre conteneur du compte de stockage du cluster Hadoop. Sinon, la requête *LOAD DATA* ne peut pas s'exécuter car elle n'aura pas accès aux données.
  >
  >

## <a name="advanced-topics-partitioned-table-and-store-hive-data-in-orc-format"></a><a name="partition-orc"></a>Rubriques avancées : Table partitionnée et Stocker des données Hive au format ORC
Si les données sont volumineuses, le partitionnement de la table est avantageux pour les requêtes qui doivent n’en balayer que quelques partitions. Par exemple, il est raisonnable de partitionner les données journalisées d’un site Web par dates.

Outre le partitionnement des tables Hive, il est également judicieux de stocker les données Hive au format ORC (Optimized Row Columnar). Pour plus d'informations sur le format ORC, consultez l'article <a href="https://cwiki.apache.org/confluence/display/Hive/LanguageManual+ORC#LanguageManualORC-ORCFiles" target="_blank">L'utilisation de fichiers ORC améliore les performances lorsque Hive lit, écrit et traite des données</a>.

### <a name="partitioned-table"></a>Table partitionnée
Voici la requête Hive qui crée une table partitionnée et charge les données dans celle-ci.

```hiveql
CREATE EXTERNAL TABLE IF NOT EXISTS <database name>.<table name>
(field1 string,
...
fieldN string
)
PARTITIONED BY (<partitionfieldname> vartype) ROW FORMAT DELIMITED FIELDS TERMINATED BY '<field separator>'
    lines terminated by '<line separator>' TBLPROPERTIES("skip.header.line.count"="1");
LOAD DATA INPATH '<path to the source file>' INTO TABLE <database name>.<partitioned table name>
    PARTITION (<partitionfieldname>=<partitionfieldvalue>);
```

Lors de l’interrogation de tables partitionnées, il est recommandé d’ajouter la condition de partition au **début** de la clause `where`, ce qui améliore l’efficacité de la recherche.

```hiveql
select
    field1, field2, ..., fieldN
from <database name>.<partitioned table name>
where <partitionfieldname>=<partitionfieldvalue> and ...;
```

### <a name="store-hive-data-in-orc-format"></a><a name="orc"></a>Stocker des données Hive au format ORC
Vous ne pouvez pas charger directement des données au format ORC depuis le stockage blob dans des tables Hive. Voici les étapes que vous devez suivre pour charger des données au format ORC depuis des blobs Azure dans des tables Hive.

Créez une table externe **STORED AS TEXTFILE** et chargez les données du stockage blob dedans.

```hiveql
CREATE EXTERNAL TABLE IF NOT EXISTS <database name>.<external textfile table name>
(
    field1 string,
    field2 int,
    ...
    fieldN date
)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '<field separator>'
    lines terminated by '<line separator>' STORED AS TEXTFILE
    LOCATION 'wasb:///<directory in Azure blob>' TBLPROPERTIES("skip.header.line.count"="1");

LOAD DATA INPATH '<path to the source file>' INTO TABLE <database name>.<table name>;
```

Créez une table interne avec le même schéma que la table externe à l’étape 1 et le même délimiteur de champ. Puis, stockez-y les données Hive au format ORC.

```hiveql
CREATE TABLE IF NOT EXISTS <database name>.<ORC table name>
(
    field1 string,
    field2 int,
    ...
    fieldN date
)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '<field separator>' STORED AS ORC;
```

Sélectionnez les données de la table externe à l’étape 1 et insérez-les dans la table ORC.

```hiveql
INSERT OVERWRITE TABLE <database name>.<ORC table name>
    SELECT * FROM <database name>.<external textfile table name>;
```

> [!NOTE]
> Si la table TEXTFILE *\<database name\>.\<external textfile table name\>* a des partitions, à l’étape 3, la commande `SELECT * FROM <database name>.<external textfile table name>` sélectionne la variable de partition en tant que champ dans le jeu de données retourné. Son insertion dans *\<database name\>.\<ORC table name\>* échoue, car *\<database name\>.\<ORC table name\>* n’a pas de variable de partition en tant que champ dans le schéma de table. Dans ce cas, vous devez sélectionner explicitement les champs à insérer dans *\<database name\>.\<ORC table name\>* comme suit :
>
>

```hiveql
INSERT OVERWRITE TABLE <database name>.<ORC table name> PARTITION (<partition variable>=<partition value>)
    SELECT field1, field2, ..., fieldN
    FROM <database name>.<external textfile table name>
    WHERE <partition variable>=<partition value>;
```

Il n’y a aucun risque à supprimer *\<external text file table name\>* quand la requête suivante est utilisée dès lors que toutes les données ont été insérées dans *\<database name\>.\<ORC table name\>*  :

```hiveql
    DROP TABLE IF EXISTS <database name>.<external textfile table name>;
```

À l’issue de cette procédure, vous devez obtenir une table immédiatement exploitable et contenant des données au format ORC.  
