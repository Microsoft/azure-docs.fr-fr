---
title: Présentation de l’API Cassandra Azure Cosmos DB
description: Découvrez comment utiliser Azure Cosmos DB pour répliquer (« lift-and-shift ») des applications existantes et en créer de nouvelles à l’aide des pilotes Cassandra et du langage CQL
author: kanshiG
ms.author: govindk
ms.reviewer: sngun
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: overview
ms.date: 11/25/2020
ms.openlocfilehash: 2a778c2e9a44bda148fd40112776858943cffe4e
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96188613"
---
# <a name="introduction-to-the-azure-cosmos-db-cassandra-api"></a>Présentation de l’API Cassandra Azure Cosmos DB
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

Vous pouvez utiliser l’API Cassandra Azure Cosmos DB comme magasin de données pour les applications écrites pour [Apache Cassandra](https://cassandra.apache.org). Cela signifie que votre application Cassandra existante peut désormais communiquer avec l’API Cassandra Azure Cosmos DB à l’aide des [pilotes Apache](https://cassandra.apache.org/doc/latest/getting_started/drivers.html?highlight=driver) existants conformes à CQLv4. Dans de nombreux cas, vous pouvez passer d’Apache Cassandra à l’API Cassandra d’Azure Cosmos DB en changeant simplement une chaîne de connexion. 

L’API Cassandra vous permet d’interagir avec les données stockées dans Azure Cosmos DB à l’aide du langage CQL (Cassandra Query Language), des outils basés sur Cassandra (comme cqlsh) et des pilotes clients Cassandra que vous connaissez déjà.

> [!NOTE]
> Le [mode de capacité serverless](serverless.md) est maintenant disponible sur l’API Cassandra d’Azure Cosmos DB.

## <a name="what-is-the-benefit-of-using-apache-cassandra-api-for-azure-cosmos-db"></a>Quel est l’avantage de l’utilisation de l’API Apache Cassandra pour Azure Cosmos DB ?

**Aucune gestion des opérations** : comme il s’agit d’un service cloud complètement managé, l’API Cassandra pour Azure Cosmos DB élimine la surcharge liée à la gestion et à la supervision d’une multitude de paramètres entre les fichiers du système d’exploitation, JVM et yaml, et leurs interactions. Azure Cosmos DB permet de superviser le débit, la latence, le stockage, la disponibilité et les alertes configurables.

**Norme open source** : bien qu’il s’agisse d’un service complètement managé, l’API Cassandra prend quand même en charge une grande surface d’exposition du [protocole filaire Apache Cassandra](cassandra-support.md), ce qui vous permet de créer des applications sur une norme open source largement utilisée et agnostique du point de vue du cloud.

**Gestion des performances** : Azure Cosmos DB garantit des opérations de lecture et d’écriture à faible latence au 99e centile, conformément aux SLA. Les utilisateurs peuvent donc garantir des performances élevées et des opérations de lecture et d’écriture à faible latence sans se préoccuper de la surcharge opérationnelle. Cela signifie aussi qu’ils peuvent s’affranchir des contraintes liées à la planification du compactage, à la gestion des objets tombstone et à la configuration manuelle de réplicas et de filtres de Bloom. En éliminant la surcharge causée par la gestion de ces problèmes, Azure Cosmos DB vous permet de vous concentrer sur la logique d’application.

**Possibilité d’utiliser le code et les outils existants** : Azure Cosmos DB fournit une compatibilité au niveau du protocole filaire avec les SDK et outils Cassandra existants. Cette compatibilité garantit la possibilité d’utiliser votre code base de code avec l’API Cassandra Azure Cosmos DB sans changements majeurs.

**Élasticité du débit et du stockage** : Azure Cosmos DB offre un débit entre toutes les régions et peut mettre à l’échelle le débit provisionné par le biais du portail Azure, de PowerShell ou de l’interface CLI. Vous pouvez effectuer une [mise à l’échelle élastique](manage-scale-cassandra.md) du stockage et du débit associés à vos tables en fonction des besoins avec des performances prévisibles.

**Disponibilité et distribution à l’échelle mondiale** : Azure Cosmos DB vous permet de distribuer des données dans toutes les régions Azure du monde et de fournir les données localement tout en garantissant la haute disponibilité des données et des accès à faible latence. Azure Cosmos DB offre une haute disponibilité de 99,99 % dans une région et 99,999 % de disponibilité en lecture et écriture entre plusieurs régions sans surcharge des opérations. Pour en savoir plus, consultez l’article [Distribuer des données dans le monde](distribute-data-globally.md). 

**Choix de la cohérence** : Azure Cosmos DB vous permet de choisir parmi cinq niveaux de cohérence bien définis pour atteindre des compromis optimaux entre cohérence et performance. Ces niveaux de cohérence sont : Forte, Obsolescence limitée, Session, Préfixe cohérent et Éventuelle. Ces niveaux de cohérence bien définis, pratiques et intuitifs permettent au développeur de trouver des compromis précis entre cohérence, disponibilité et latence. Pour en savoir plus, consultez l’article [Niveaux de cohérence](consistency-levels.md). 

**De qualité professionnelle** : Azure Cosmos DB fournit des [certifications de conformité](https://www.microsoft.com/trustcenter) pour que les utilisateurs puissent utiliser la plateforme en toute sécurité. Azure Cosmos DB fournit également le chiffrement au repos et en mouvement, un pare-feu IP et des journaux d’audit pour les activités de plan de contrôle.

**Provisionnement en événements** : l’API Cassandra permet d’accéder à un journal des modifications persistant, le [flux de modification](cassandra-change-feed.md), ce qui peut faciliter le provisionnement des événements directement à partir de la base de données. Dans Apache Cassandra, le seul équivalent est la capture des changements de données (CDC), qui est simplement un mécanisme de marquage de tables spécifiques pour l’archivage et de rejet des écritures dans ces tables une fois que la taille sur disque configurable du journal CDC est atteinte (ces fonctionnalités sont redondantes dans Cosmos DB, car les aspects pertinents sont régis automatiquement).

## <a name="next-steps"></a>Étapes suivantes

* Commencez par générer les applications spécifiques à un langage suivantes pour créer et gérer les données de l’API Cassandra :
  - [Application Node.js](create-cassandra-nodejs.md)
  - [Application .NET](create-cassandra-dotnet.md)
  - [Application Python](create-cassandra-python.md)

* Familiarisez-vous avec la [création d’un compte d’API Cassandra, d’une base de données et d’une table](create-cassandra-api-account-java.md) à l’aide d’une application Java.

* [Chargez des exemples de données dans la table de l’API Cassandra](cassandra-api-load-data.md) à l’aide d’une application Java.

* [Interrogez les données du compte d’API Cassandra](cassandra-api-query-data.md) à l’aide d’une application Java.

* Pour en savoir plus sur les fonctionnalités d’Apache Cassandra prises en charge par l’API Cassandra Azure Cosmos DB, consultez l’article [Prise en charge de Cassandra](cassandra-support.md).

* Consultez les [questions fréquentes](cassandra-faq.md).
