---
title: Migration de bases de données existantes pour effectuer un scale-out
description: Conversion de bases de données partitionnées pour utiliser les outils de base de données élastique en créant un gestionnaire de cartes de partitions
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 01/25/2019
ms.openlocfilehash: c6ad8b4c80f4b9c2fdb3c1a14209dcf0febc89e9
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92787140"
---
# <a name="migrate-existing-databases-to-scale-out"></a>Migration de bases de données existantes pour effectuer un scale-out
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Gérez facilement vos bases de données partitionnées et montées en charge existantes à l’aide d’outils (comme la [bibliothèque de client de base de données élastique](elastic-database-client-library.md)). Convertissez d’abord un ensemble existant de bases de données pour utiliser le [Gestionnaire de cartes de partitions](elastic-scale-shard-map-management.md).

## <a name="overview"></a>Vue d’ensemble

Pour migrer une base de données partitionnée existante :

1. Préparez la [base de données pour le Gestionnaire de cartes de partitions](elastic-scale-shard-map-management.md).
2. Créer la carte de partitions.
3. Préparer les partitions individuelles.  
4. Ajoutez les mappages à la carte de partitions.

Ces techniques peuvent être implémentées à l’aide de la [bibliothèque de client .NET Framework](https://www.nuget.org/packages/Microsoft.Azure.SqlDatabase.ElasticScale.Client/) ou des scripts PowerShell (voir [Azure SQL Database – Scripts d’outils de base de données élastique](https://gallery.technet.microsoft.com/scriptcenter/Azure-SQL-DB-Elastic-731883db)). Les exemples fournis ici utilisent les scripts PowerShell.

Pour plus d’informations sur la classe ShardMapManager, consultez la page [Gestion des cartes de partitions](elastic-scale-shard-map-management.md). Pour obtenir une présentation des outils de base de données élastique, consultez [Vue d’ensemble des fonctionnalités de base de données élastique](elastic-scale-introduction.md).

## <a name="prepare-the-shard-map-manager-database"></a>Préparer la base de données pour le Gestionnaire de cartes de partitions

Le Gestionnaire de cartes de partitions est une base de données spéciale qui contient les données permettant de gérer les bases de données avec montée en charge. Vous pouvez utiliser une base de données existante ou en créer une. Une base de données agissant en tant que Gestionnaire de cartes de partitions ne peut pas être identique à une partition. Le script PowerShell ne crée pas la base de données à votre place.

## <a name="step-1-create-a-shard-map-manager"></a>Étape 1 : Créer un gestionnaire de cartes de partitions

```powershell
# Create a shard map manager
New-ShardMapManager -UserName '<user_name>' -Password '<password>' -SqlServerName '<server_name>' -SqlDatabaseName '<smm_db_name>'
#<server_name> and <smm_db_name> are the server name and database name
# for the new or existing database that should be used for storing
# tenant-database mapping information.
```

### <a name="to-retrieve-the-shard-map-manager"></a>Pour récupérer le gestionnaire de cartes de partitions

Après la création, vous pouvez récupérer le gestionnaire de cartes de partitions avec cette applet de commande. Cette étape est nécessaire chaque fois que vous devez utiliser l’objet ShardMapManager.

```powershell
# Try to get a reference to the Shard Map Manager  
$ShardMapManager = Get-ShardMapManager -UserName '<user_name>' -Password '<password>' -SqlServerName '<server_name>' -SqlDatabaseName '<smm_db_name>'
```

## <a name="step-2-create-the-shard-map"></a>Étape 2 : Créer la carte de partitions

Sélectionnez le type de carte de partitions à créer. Votre choix dépend de l’architecture de la base de données :

1. Client unique par base de données (Pour rechercher des termes spécifiques, consultez le [glossaire](elastic-scale-glossary.md).)
2. Plusieurs clients par base de données (deux types) :
   1. Mappage de liste
   2. Mappage de plage

Pour un modèle de client unique, créez une carte de partitions de **mappage de liste** . Le modèle à un seul client attribue une base de données par client. Il s’agit d’un modèle efficace pour les développeurs SaaS, car il simplifie la gestion.

![Mappage de liste][1]

Le modèle multi-locataire affecte plusieurs locataires à une seule base de données (et vous pouvez distribuer des groupes de locataires sur plusieurs bases de données). Utilisez ce modèle lorsque vous pensez que chaque client va avoir de faibles besoins en termes de données. Dans ce modèle, attribuez une plage de clients à une base de données à l’aide du **mappage de plage** .

![Mappage de plage][2]

Vous pouvez également implémenter un modèle de base de données multi-locataire à l’aide d’un *mappage de liste* pour affecter plusieurs locataires à une base de données individuelle. Par exemple, DB1 est utilisée pour stocker les informations d’ID client 1 et 5 et DB2 stocke les données pour les clients 7 et 10.

![Plusieurs clients sur une base de données unique][3]

**Selon votre choix, procédez de l’une des manières suivantes :**

### <a name="option-1-create-a-shard-map-for-a-list-mapping"></a>Option 1 : Créer une carte de partitions pour un mappage de liste

Créez une carte de partitions à l’aide de l’objet ShardMapManager.

```powershell
# $ShardMapManager is the shard map manager object
$ShardMap = New-ListShardMap -KeyType $([int]) -ListShardMapName 'ListShardMap' -ShardMapManager $ShardMapManager
```

### <a name="option-2-create-a-shard-map-for-a-range-mapping"></a>Option n°2 : Créer une carte de partitions pour un mappage de plage

Pour utiliser ce modèle de mappage, les valeurs d’ID client doivent être des plages continues. De plus, il est raisonnable d’avoir un écart dans les plages en ignorant la plage pendant la création de bases de données.

```powershell
# $ShardMapManager is the shard map manager object
# 'RangeShardMap' is the unique identifier for the range shard map.  
$ShardMap = New-RangeShardMap -KeyType $([int]) -RangeShardMapName 'RangeShardMap' -ShardMapManager $ShardMapManager
```

### <a name="option-3-list-mappings-on-an-individual-database"></a>Option 3 : Mappages de liste sur une base de données individuelle

La configuration de ce modèle nécessite également la création d’un mappage de liste comme indiqué à l’étape 2, option 1.

## <a name="step-3-prepare-individual-shards"></a>Étape 3 : Préparer les partitions individuelles

Ajoutez chaque partition (base de données) dans le gestionnaire de cartes de partitions. Cela prépare les bases de données individuelles à stocker les informations de mappage. Exécutez cette méthode sur chaque partition.

```powershell
Add-Shard -ShardMap $ShardMap -SqlServerName '<shard_server_name>' -SqlDatabaseName '<shard_database_name>'
# The $ShardMap is the shard map created in step 2.
```

## <a name="step-4-add-mappings"></a>Étape 4 : Ajouter des mappages

L’ajout de mappages varie selon le type de carte de partitions que vous avez créé. Si vous avez créé un mappage de liste, vous ajoutez des mappages de liste. Si vous avez créé un mappage de plage, vous ajoutez des mappages de plage.

### <a name="option-1-map-the-data-for-a-list-mapping"></a>Option 1 : Mapper les données pour un mappage de liste

Mappez les données en ajoutant un mappage de liste pour chaque client.  

```powershell
# Create the mappings and associate it with the new shards
Add-ListMapping -KeyType $([int]) -ListPoint '<tenant_id>' -ListShardMap $ShardMap -SqlServerName '<shard_server_name>' -SqlDatabaseName '<shard_database_name>'
```

### <a name="option-2-map-the-data-for-a-range-mapping"></a>Option n°2 : Mapper les données pour un mappage de plage

Ajoutez les mappages de plage pour la plage d’ID client – associations de bases de données :

```powershell
# Create the mappings and associate it with the new shards
Add-RangeMapping -KeyType $([int]) -RangeHigh '5' -RangeLow '1' -RangeShardMap $ShardMap -SqlServerName '<shard_server_name>' -SqlDatabaseName '<shard_database_name>'
```

### <a name="step-4-option-3-map-the-data-for-multiple-tenants-on-an-individual-database"></a>Étape 4, option 3 : Mapper les données de plusieurs locataires sur une base de données individuelle

Pour chaque client, exécutez la commande Add-ListMapping (option 1).

## <a name="checking-the-mappings"></a>Vérification des mappages

Vous pouvez interroger les informations sur les partitions existantes et les mappages associés à l’aide des commandes suivantes :  

```powershell
# List the shards and mappings
Get-Shards -ShardMap $ShardMap
Get-Mappings -ShardMap $ShardMap
```

## <a name="summary"></a>Résumé

Une fois que vous avez terminé l’installation, vous pouvez commencer à utiliser la bibliothèque cliente de base de données élastique. Vous pouvez également utiliser le [routage dépendant des données](elastic-scale-data-dependent-routing.md) et la [requête sur plusieurs partitions](elastic-scale-multishard-querying.md).

## <a name="next-steps"></a>Étapes suivantes

Obtenez les scripts PowerShell à partir des [scripts d’outils de base de données élastique Azure SQL Database](https://gallery.technet.microsoft.com/scriptcenter/Azure-SQL-DB-Elastic-731883db).

Les outils se trouvent également sur GitHub : [Azure/elastic-db-tools](https://github.com/Azure/elastic-db-tools).

Utilisez l’outil de fractionnement et de fusion pour déplacer des données, à partir d’un modèle mutualisé ou vers celui-ci, vers un modèle de client unique. Consultez [Outil de fractionnement et de fusion](elastic-scale-get-started.md).

## <a name="additional-resources"></a>Ressources supplémentaires

Pour plus d’informations sur les modèles d’architecture de données courants pour les applications de base de données SaaS (software as a service) multilocataires, consultez [Modèles de conception pour les applications SaaS multilocataires avec Azure SQL Database](saas-tenancy-app-design-patterns.md).

## <a name="questions-and-feature-requests"></a>Questions et demandes de fonctionnalités

Pour toute question, referez-vous à la [page de questions Microsoft Q&A pour SQL Database](/answers/topics/azure-sql-database.html) et formulez vos demandes de fonctionnalités éventuelles sur le [forum de commentaires SQL Database](https://feedback.azure.com/forums/217321-sql-database/).

<!--Image references-->
[1]: ./media/elastic-convert-to-use-elastic-tools/listmapping.png
[2]: ./media/elastic-convert-to-use-elastic-tools/rangemapping.png
[3]: ./media/elastic-convert-to-use-elastic-tools/multipleonsingledb.png