---
title: FAQ sur la mise à l’échelle élastique
description: Questions fréquentes (FAQ) sur la mise à l’échelle élastique d’Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 01/25/2019
ms.openlocfilehash: 51e15a8dc5e9f918c630397d6d6593f5bf561755
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92786902"
---
# <a name="elastic-database-tools-frequently-asked-questions-faq"></a>Forum aux questions sur les outils de bases de données élastiques
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

## <a name="if-i-have-a-single-tenant-per-shard-and-no-sharding-key-how-do-i-populate-the-sharding-key-for-the-schema-info"></a>Si je dispose d'un seul locataire par partition sans aucune clé de sharding, comment puis-je renseigner la clé de sharding pour les informations de schéma ?

L’objet d’informations de schéma est utilisé uniquement dans les scénarios de fractionnement/fusion. Si une application ne possède par nature qu’un seul locataire, elle n’a pas besoin de l’outil de fractionnement/fusion, et il n’est donc pas nécessaire de remplir l’objet d’informations de schéma.

## <a name="ive-provisioned-a-database-and-i-already-have-a-shard-map-manager-how-do-i-register-this-new-database-as-a-shard"></a>J'ai configuré une base de données et je dispose déjà d'un Gestionnaire de cartes de partitions. Comment puis-je enregistrer cette nouvelle base de données en tant que partition ?

Consultez [Ajout d'une partition à une application à l'aide de la bibliothèque de client de bases de données élastiques](elastic-scale-add-a-shard.md).

## <a name="how-much-do-elastic-database-tools-cost"></a>Combien coûtent les outils de bases de données élastiques ?

L’utilisation de la bibliothèque cliente des bases de données élastiques est gratuite. Les coûts sont uniquement liés à l’utilisation des bases de données dans Azure SQL Database pour les partitions et le gestionnaire des mappages de partition, ainsi qu’aux rôles web/de travail configurés pour le service de fractionnement/fusion.

## <a name="why-are-my-credentials-not-working-when-i-add-a-shard-from-a-different-server"></a>Pourquoi mes informations d'identification ne fonctionnent-elles pas lorsque j'ajoute une partition issue d'un autre serveur ?

N’utilisez pas d’informations d’identification de type « ID d’utilisateur =username@servername » ; utilisez simplement « ID d’utilisateur = nom_utilisateur » à la place.  Par ailleurs, assurez-vous que l’ID de connexion « nom_utilisateur » dispose d’autorisations sur la partition.

## <a name="do-i-need-to-create-a-shard-map-manager-and-populate-shards-every-time-i-start-my-applications"></a>Dois-je créer un Gestionnaire de cartes de partitions puis renseigner les partitions chaque fois que je démarre mes applications ?

Non, le Gestionnaire de cartes de partitions (par exemple, [ShardMapManagerFactory.CreateSqlShardMapManager](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.createsqlshardmapmanager)) est créé une bonne fois pour toutes.  Votre application doit appeler [ShardMapManagerFactory.TryGetSqlShardMapManager()](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.trygetsqlshardmapmanager) lorsqu'elle démarre.  Un seul appel de ce type ne doit être possible par domaine d’application.

## <a name="i-have-questions-about-using-elastic-database-tools-how-do-i-get-them-answered"></a>Je me pose des questions sur l'utilisation des outils de bases de données élastiques. Où puis-je trouver des réponses ?

Veuillez nous contacter sur la [page de questions Microsoft Q&A pour SQL Database](/answers/topics/azure-sql-database.html).

## <a name="when-i-get-a-database-connection-using-a-sharding-key-i-can-still-query-data-for-other-sharding-keys-on-the-same-shard--is-this-by-design"></a>Quand j’obtiens une connexion de base de données à l’aide d’une clé de partitionnement, je peux encore interroger les données pour d’autres clés de partitionnement sur la même partition.  Est-ce normal ?

Les API d’infrastructure élastique vous donnent une connexion à la base de données correspondant à votre clé de partitionnement, mais elles ne permettent pas de filtrer les clés de partitionnement.  Ajoutez des clauses **WHERE** à votre requête pour restreindre l’étendue à la clé de partitionnement fournie, si nécessaire.

## <a name="can-i-use-a-different-sql-database-edition-for-each-shard-in-my-shard-set"></a>Puis-je utiliser une édition différente de SQL Database pour chacune des partitions de mon ensemble de partitions ?

Oui, une partition est une base de données individuelle, par conséquent, une partition peut utiliser une édition Premium alors qu’une autre utilise une édition Standard. Par ailleurs, l’édition d’une partition peut monter ou descendre en puissance plusieurs fois pendant la durée de vie de la partition.

## <a name="does-the-split-merge-tool-provision-or-delete-a-database-during-a-split-or-merge-operation"></a>Le service de fractionnement/fusion configure-t-il (ou supprime-t-il) une base de données pendant une opération de fractionnement ou de fusion ?

Non. Pour les opérations de **fractionnement** , la base de données cible doit exister avec le schéma approprié et être enregistrée dans le gestionnaire de mappages de partition.  Pour les opérations de **fusion** , vous devez supprimer la partition à partir du gestionnaire de mappages de partition, puis supprimer la base de données.

[!INCLUDE [elastic-scale-include](../../../includes/elastic-scale-include.md)]