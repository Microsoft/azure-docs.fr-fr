---
title: Transactions de base de données et contrôle d’accès concurrentiel optimiste dans Azure Cosmos DB
description: Cet article décrit les transactions de base de données et le contrôle d’accès concurrentiel optimiste dans Azure Cosmos DB
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 12/04/2019
ms.reviewer: sngun
ms.openlocfilehash: bdfbe5106f220a9fe4a3568709187b9071bc7917
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93334274"
---
# <a name="transactions-and-optimistic-concurrency-control"></a>Transactions et contrôle d’accès concurrentiel optimiste
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Les transactions de base de données offrent un modèle de programmation prédictif et sécurisé pour gérer les modifications simultanées des données. Les bases de données relationnelles classiques, telles que SQL Server, vous permettent d’écrire la logique métier à l’aide de procédures stockées et/ou de déclencheurs, et de l’envoyer au serveur pour une exécution directe dans le moteur de base de données. Avec les bases de données relationnelles classiques, vous êtes amené à traiter deux langages de programmation différents : le langage de programmation d’application non transactionnel (JavaScript, Python, C#, Java, etc.) et le langage de programmation transactionnel (par exemple, T-SQL) exécuté en mode natif par la base de données.

Le moteur de base de données dans Azure Cosmos DB prend en charge les transactions entièrement conformes à ACID (atomicité, cohérence, isolation, durabilité) avec isolement de capture instantanée. Toutes les opérations de base de données dans l’étendue de la [partition logique](partitioning-overview.md) d’un conteneur sont exécutées par le biais de transactions dans le moteur de base de données qui est hébergé par le réplica de la partition. Ces opérations comprennent à la fois les opérations d’écriture (mise à jour d’un ou de plusieurs éléments dans la partition logique) et de lecture. Le tableau suivant illustre différents types de transactions et d’opérations :

| **opération**  | **Type d’opération** | **Transaction à un seul ou plusieurs éléments** |
|---------|---------|---------|
| Insérer (sans pré/postdéclencheur) | Write | Transaction à un seul élément |
| Insérer (avec pré/postdéclencheur) | Écrire et lire | Transaction à plusieurs éléments |
| Remplacer (sans pré/postdéclencheur) | Write | Transaction à un seul élément |
| Remplacer (avec pré/postdéclencheur) | Écrire et lire | Transaction à plusieurs éléments |
| Upsert (sans pré/postdéclencheur) | Write | Transaction à un seul élément |
| Upsert (avec pré/postdéclencheur) | Écrire et lire | Transaction à plusieurs éléments |
| Supprimer (sans pré/postdéclencheur) | Write | Transaction à un seul élément |
| Supprimer (avec pré/postdéclencheur) | Écrire et lire | Transaction à plusieurs éléments |
| Exécuter une procédure stockée | Écrire et lire | Transaction à plusieurs éléments |
| Exécution lancée par le système d’une procédure de fusion | Write | Transaction à plusieurs éléments |
| Exécution lancée par le système de la suppression des éléments en fonction de l’expiration (durée de vie ou TTL) d’un élément | Write | Transaction à plusieurs éléments |
| Lire | Lire | Transaction à un seul élément |
| Modifier le flux | Lire | Transaction à plusieurs éléments |
| Lecture paginée | Lire | Transaction à plusieurs éléments |
| Requête paginée | Lire | Transaction à plusieurs éléments |
| Exécuter une fonction définie par l’utilisateur dans le cadre de la requête paginée | Lire | Transaction à plusieurs éléments |

## <a name="multi-item-transactions"></a>Transactions à plusieurs éléments

Azure Cosmos DB vous permet d’écrire des [procédures stockées, des pré/postdéclencheurs, des fonctions définies par l’utilisateur](stored-procedures-triggers-udfs.md) et des procédures de fusion en JavaScript. Azure Cosmos DB prend en charge en mode natif l’exécution de JavaScript à l’intérieur de son moteur de base de données. Vous pouvez inscrire des procédures stockées, des pré/postdéclencheurs, des fonctions définies par l’utilisateur et des procédures de fusion sur un conteneur et les exécuter par la suite via des transactions dans le moteur de base de données Azure Cosmos. L’écriture de la logique d’application en JavaScript permet une expression naturelle du flux de contrôle, de l’étendue variable, de l’attribution et de l’intégration des primitives de gestion des exceptions au sein des transactions de base de données directement dans le langage JavaScript.

Les procédures stockées, déclencheurs, fonctions définies par l’utilisateur et procédures de fusion JavaScript sont inclus dans une transaction ACID ambiante avec un isolement de capture instantanée dans tous les éléments de la partition logique. Lors de son exécution, si le programme JavaScript lève une exception, toute la transaction est abandonnée et restaurée. Le modèle de programmation obtenu est simple, mais puissant. Les développeurs JavaScript obtiennent un modèle de programmation durable tout en continuant d’utiliser les constructions de langage et les primitives de bibliothèques qui leurs sont familières.

La possibilité d’exécuter JavaScript directement dans le moteur de base de données offre une exécution performante et transactionnelle des opérations de base de données sur les éléments d’un conteneur. De plus, comme le moteur de base de données Azure Cosmos prend en charge en mode natif JSON et JavaScript, il n’existe aucun risque d’incohérence en matière d’impédance entre les systèmes de type d’une application et la base de données.

## <a name="optimistic-concurrency-control"></a>Contrôle de concurrence optimiste

Le contrôle d’accès concurrentiel optimiste vous permet d’empêcher les suppressions et mises à jour perdues. Les opérations simultanées en conflit sont soumises au verrouillage pessimiste standard du moteur de base de données hébergé par la partition logique qui détient l’élément. Quand deux opérations simultanées tentent de mettre à jour la dernière version d’un élément dans une partition logique, l’une d’elles réussit et l’autre échoue. Toutefois, si l’une ou les deux opérations qui tentent de mettre à jour simultanément le même élément ont déjà lu une ancienne valeur de l’élément, la base de données ne sait pas si la valeur précédemment lue par l’une des opérations en conflit (ou les deux) était effectivement la valeur la plus récente de l’élément. Heureusement, cette situation peut être détectée avec le **contrôle d’accès concurrentiel optimiste** avant de laisser les deux opérations entrer dans la limite de transaction à l’intérieur du moteur de base de données. Le contrôle d’accès concurrentiel optimiste protège vos données contre tout remplacement involontaire de modifications apportées par d’autres utilisateurs. Il empêche également d’autres personnes d’écraser accidentellement vos propres modifications.

Les mises à jour simultanées d’un élément sont soumises au contrôle d’accès concurrentiel optimiste par la couche de protocole de communication d’Azure Cosmos DB. La base de données Azure Cosmos garantit que la version côté client de l’élément que vous mettez à jour (ou supprimez) est identique à la version de l’élément dans le conteneur Azure Cosmos. Cela garantit que vos écritures sont protégées contre tout remplacement involontaire par les écritures d’autres utilisateurs, et inversement. Dans un environnement multi-utilisateur, le contrôle d’accès concurrentiel optimiste vous empêche de supprimer ou de mettre à jour accidentellement la version incorrecte d’un élément. Par conséquent, les éléments sont protégés contre les problèmes notoires de perte de mise à jour ou de suppression.

Chaque élément stocké dans un conteneur Azure Cosmos dispose d’une propriété `_etag` définie par le système. La valeur de `_etag` est automatiquement générée et mise à jour par le serveur chaque fois que l’élément est mis à jour. La propriété `_etag` peut être utilisée avec l’en-tête de requête `if-match` fourni par le client pour permettre au serveur de déterminer si un élément peut être mis à jour de manière conditionnelle. Si la valeur de l’en-tête `if-match` correspond à la valeur de `_etag` au niveau du serveur, l’élément est alors mis à jour. Si la valeur de l’en-tête de requête `if-match` n’est plus actuelle, le serveur rejette l’opération avec un message de réponse de type « HTTP 412 Échec de la condition préalable ». Le client peut alors réextraire l’élément pour acquérir sa version actuelle sur le serveur ou remplacer la version de l’élément sur le serveur par sa propre valeur `_etag` pour l’élément. De plus, la propriété `_etag` peut être utilisée avec l’en-tête `if-none-match` pour déterminer si la nouvelle extraction d’une ressource est nécessaire.

La valeur `_etag` de l’élément change chaque fois que l’élément est mis à jour. Pour les opérations de remplacement d’élément, `if-match` doit être exprimé explicitement dans le cadre des options de requête. Pour obtenir un exemple, consultez l’exemple de code dans [GitHub](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/ItemManagement/Program.cs#L676-L772). Les valeurs `_etag` sont implicitement vérifiées pour tous les éléments écrits affectés par la procédure stockée. Si un conflit est détecté, la procédure stockée restaure la transaction et lève une exception. Avec cette méthode, l’ensemble ou aucune des écritures dans la procédure stockée sont appliquées de façon atomique. Il s’agit d’un signal à l’application pour réappliquer les mises à jour et réessayer la demande du client d’origine.

## <a name="next-steps"></a>Étapes suivantes

Découvrez plus en détail les transactions de base de données et le contrôle d’accès concurrentiel optimiste dans les articles suivants :

- [Utilisation des bases de données, des conteneurs et des éléments Azure Cosmos](account-databases-containers-items.md)
- [Niveaux de cohérence](consistency-levels.md)
- [Types de conflits et stratégies de résolution](conflict-resolution-policies.md)
- [Utilisation de TransactionalBatch](transactional-batch.md)
- [Procédures stockées, déclencheurs et fonctions définies par l’utilisateur](stored-procedures-triggers-udfs.md)
