---
title: Créer des rapports sur des bases de données cloud montées en charge (scale out)
description: Utilisez des requêtes de base de données croisées pour créer des rapports sur plusieurs bases de données.
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: MladjoA
ms.author: mlandzic
ms.reviewer: sstein
ms.date: 10/10/2019
ms.openlocfilehash: 586dad7439cc57ed2c863ee5f6692e12f7a78c50
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92781224"
---
# <a name="report-across-scaled-out-cloud-databases-preview"></a>Créer des rapports sur des bases de données cloud avec montée en charge (version préliminaire)
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Vous pouvez créer des rapports tirés de plusieurs bases de données à partir d’un point de connexion unique par le biais d’une [requête élastique](elastic-query-overview.md). Les bases de données doivent être partitionnées horizontalement.

Si vous avez déjà une base de données, consultez [Migrer des bases de données existantes vers des bases de données mises à l’échelle](elastic-convert-to-use-elastic-tools.md).

Pour connaître les objets SQL requis dans le cadre d’une requête, consultez [Interroger plusieurs bases de données partitionnées horizontalement](elastic-query-horizontal-partitioning.md).

## <a name="prerequisites"></a>Prérequis

Téléchargez et exécutez l’exemple de la rubrique [Prise en main des outils de base de données élastique](elastic-scale-get-started.md).

## <a name="create-a-shard-map-manager-using-the-sample-app"></a>Créez un gestionnaire des cartes de partitions à l’aide de l’exemple d’application
Ici vous allez créer un gestionnaire des cartes de partitions avec plusieurs partitions, puis insérer des données dans les partitions. Si vos partitions comportent déjà des données partitionnées, vous pouvez ignorer ces étapes et passer à la section suivante.

1. Générez et exécutez l’exemple d’application **Prise en main des outils de base de données élastique** en suivant les étapes décrites dans la section de l’article [Télécharger et exécuter l’exemple d’application](elastic-scale-get-started.md#download-and-run-the-sample-app-1). Une fois toutes les étapes terminées, l’invite de commandes suivante s’affiche :

    ![invite de commandes][1]
2. Dans la fenêtre de commande, entrez « 1 » et appuyez sur **Entrée** . Cela crée le gestionnaire des cartes de partitions et ajoute deux partitions sur le serveur. Tapez « 3 », puis appuyez sur **Entrée** . Répétez l’action quatre fois. Cela permet d’insérer des lignes d’exemples de données dans vos partitions.
3. Le [portail Azure](https://portal.azure.com) doit alors montrer trois nouvelles bases de données dans votre serveur :

   ![Confirmation Visual Studio][2]

   À ce stade, les requêtes de bases de données croisées sont prises en charge via les bibliothèques clientes de bases de données élastiques. Par exemple, utilisez l’option 4 dans la fenêtre de commande. Les résultats d’une requête de plusieurs partitions sont toujours un élément **UNION ALL** des résultats de toutes les partitions.

   Dans la section suivante, nous créons un exemple de point de terminaison de base de données qui prend en charge une interrogation plus riche des données entre les partitions.

## <a name="create-an-elastic-query-database"></a>Créez une base de données de requête élastique

1. Ouvrez le [portail Azure](https://portal.azure.com) et connectez-vous.
2. Créez une nouvelle base de données dans Azure SQL Database sur le même serveur que votre configuration de partition. Nommez la base de données « ElasticDBQuery ».

    ![Portail Azure et tarification][3]

    > [!NOTE]
    > Vous pouvez utiliser une base de données existante. Si c’est le cas, il ne doit pas s’agir de l’une des partitions sur laquelle vous souhaitez exécuter les requêtes. Cette base de données sera utilisée pour la création d’objets de métadonnées pour une requête de base de données élastique.
    >

## <a name="create-database-objects"></a>Créez des objets de base de données
### <a name="database-scoped-master-key-and-credentials"></a>Clé principale et informations d’identification de la base de données
Celles-ci sont utilisées pour se connecter au gestionnaire des cartes de partition et aux partitions :

1. Ouvrez SQL Server Management Studio ou SQL Server Data Tools dans Visual Studio.
2. Connectez-vous à la base de données ElasticDBQuery et exécutez les commandes T-SQL suivantes :

    ```tsql
    CREATE MASTER KEY ENCRYPTION BY PASSWORD = '<master_key_password>';

    CREATE DATABASE SCOPED CREDENTIAL ElasticDBQueryCred
    WITH IDENTITY = '<username>',
    SECRET = '<password>';
    ```

    Le nom d’utilisateur et le mot de passe doivent être les mêmes que les informations de connexion utilisées à l’étape 3 de la section [Télécharger et exécuter l’exemple d’application](elastic-scale-get-started.md#download-and-run-the-sample-app) dans l’article **Prise en main des outils de base de données élastique** .

### <a name="external-data-sources"></a>Sources de données externes
Pour créer une source de données externe, exécutez la commande suivante sur la base de données ElasticDBQuery :

```tsql
CREATE EXTERNAL DATA SOURCE MyElasticDBQueryDataSrc WITH
    (TYPE = SHARD_MAP_MANAGER,
    LOCATION = '<server_name>.database.windows.net',
    DATABASE_NAME = 'ElasticScaleStarterKit_ShardMapManagerDb',
    CREDENTIAL = ElasticDBQueryCred,
    SHARD_MAP_NAME = 'CustomerIDShardMap'
) ;
```    

 « CustomerIDShardMap » est le nom de la carte de partition, si vous avez créé la carte de partition et le gestionnaire des cartes de partition à l’aide de l’exemple d’outils souples de la base de données. Toutefois, si vous avez utilisé l’installation personnalisée pour cet exemple, le nom de carte de partition doit correspondre au nom choisi dans l’application.

### <a name="external-tables"></a>Tables externes
Créez une table externe qui correspond à la table des clients sur les partitions en exécutant la commande suivante sur la base de données ElasticDBQuery :

```tsql
CREATE EXTERNAL TABLE [dbo].[Customers]
( [CustomerId] [int] NOT NULL,
    [Name] [nvarchar](256) NOT NULL,
    [RegionId] [int] NOT NULL)
WITH
( DATA_SOURCE = MyElasticDBQueryDataSrc,
    DISTRIBUTION = SHARDED([CustomerId])
) ;
```

## <a name="execute-a-sample-elastic-database-t-sql-query"></a>Exécutez un exemple de requête T-SQL de base de données élastique
Après avoir défini vos tables externes et votre source de données externe, vous pouvez utiliser l’ensemble T-SQL sur vos tables externes.

Exécutez cette requête sur la base de données ElasticDBQuery :

```tsql
select count(CustomerId) from [dbo].[Customers]
```

Vous remarquerez que la requête regroupe les résultats de toutes les partitions et donne le résultat suivant :

![Détails des résultats][4]

## <a name="import-elastic-database-query-results-to-excel"></a>Importez les résultats de la requête de base de données élastique dans Excel
 Vous pouvez importer les résultats d’une requête vers un fichier Excel.

1. Lancez Excel 2013.
2. Accédez au ruban **Données** .
3. Cliquez sur **À partir d’autres sources** et sur **À partir de SQL Server** .

   ![Importation au format Excel à partir d’autres sources][5]
4. Dans l’ **Assistant de connexion de données** saisissez le nom du serveur et les informations de connexion. Cliquez ensuite sur **Suivant** .
5. Dans la boîte de dialogue **Sélectionner la base de données qui contient les données que vous souhaitez** , sélectionnez la base de données **ElasticDBQuery** .
6. Sélectionnez la table **Clients** dans la liste et cliquez sur **Suivant** . Puis, cliquez sur **Terminer** .
7. Dans le formulaire **Importer des données** , sous **Sélectionner le mode d’affichage des données dans votre classeur** , sélectionnez **Table** et cliquez sur **OK** .

Toutes les lignes de la table **Clients** , stockées dans des partitions différentes, remplissent la feuille Excel.

Vous pouvez maintenant utiliser les fonctions de visualisation de données puissantes d’Excel. Vous pouvez utiliser la chaîne de connexion avec votre nom de serveur, votre nom de base de données et les informations d’identification pour vous connecter vos outils d’intégration BI et de données dans la base de données de requête élastique. Assurez-vous que SQL Server est pris en charge comme source de données pour votre outil. Vous pouvez vous traiter la base de données de requête élastique et les tables externes comme n’importe quelles bases de données SQL Server et tables SQL Server auxquelles vous vous connectez avec votre outil.

### <a name="cost"></a>Coût
La fonction de requête de base de données élastique n’entraîne aucuns frais supplémentaires.

Pour plus d’informations sur la tarification, consultez la page [Tarification - SQL Database](https://azure.microsoft.com/pricing/details/sql-database/).

## <a name="next-steps"></a>Étapes suivantes

* Pour une présentation des requêtes élastiques, consultez [Présentation des requêtes élastiques](elastic-query-overview.md).
* Pour le didacticiel sur le partitionnement vertical, consultez [Prise en main des requêtes de bases de données croisées (partitionnement vertical)](elastic-query-getting-started-vertical.md).
* Pour la syntaxe et des exemples de requêtes pour des données partitionnées verticalement, consultez [Requêtes sur des données partitionnées verticalement](elastic-query-vertical-partitioning.md)
* Pour la syntaxe et des exemples de requêtes pour des données partitionnées horizontalement, consultez [Requêtes sur des données partitionnées horizontalement](elastic-query-horizontal-partitioning.md)
* Consultez [sp\_execute \_remote](/sql/relational-databases/system-stored-procedures/sp-execute-remote-azure-sql-database) pour une procédure stockée qui exécute une instruction Transact-SQL sur une seule base de données Azure SQL distante ou un ensemble de bases de données servant de partitions dans un schéma de partitionnement horizontal.


<!--Image references-->
[1]: ./media/elastic-query-getting-started/cmd-prompt.png
[2]: ./media/elastic-query-getting-started/portal.png
[3]: ./media/elastic-query-getting-started/tiers.png
[4]: ./media/elastic-query-getting-started/details.png
[5]: ./media/elastic-query-getting-started/exel-sources.png
<!--anchors-->