---
title: Modèle de ressource Azure Cosmos DB
description: Cet article décrit le modèle de ressource Azure Cosmos DB qui comprend le compte, la base de données, le conteneur et les éléments Azure Cosmos. Il traite également de la hiérarchie de ces éléments dans un compte Azure Cosmos.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 10/12/2020
ms.reviewer: sngun
ms.openlocfilehash: 37f1c9f59b6ffb45e1b874d2a6969bf263d2d5eb
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93341363"
---
# <a name="azure-cosmos-db-resource-model"></a>Modèle de ressource Azure Cosmos DB
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Azure Cosmos DB est une plateforme en tant que service (Paas) entièrement gérée. Pour commencer à utiliser Azure Cosmos DB, vous devez initialement créer un compte Azure Cosmos dans votre abonnement Azure et les bases de données, les conteneurs et les éléments qu’il contient. Cet article décrit le modèle de ressource Azure Cosmos DB et les différentes entités dans la hiérarchie du modèle de ressource.

Le compte Azure Cosmos est une unité fondamentale de distribution mondiale et de haute disponibilité. Votre compte Azure Cosmos contient un nom DNS unique et vous pouvez gérer un compte à l’aide du portail Azure, Azure CLI ou à l’aide de différents Kits de développement logiciel (SDK) spécifiques à une langue. Pour en savoir plus, consultez [Comment gérer votre compte Azure Cosmos](how-to-manage-database-account.md). Pour distribuer vos données et votre débit dans plusieurs régions Azure à l’échelle mondiale, vous pouvez ajouter des régions Azure à votre compte et en supprimer à tout moment. Vous pouvez configurer votre compte pour une seule région ou pour plusieurs régions d’écriture. Pour en savoir plus, consultez [comment ajouter et supprimer des régions Azure sur votre compte](how-to-manage-database-account.md). Vous pouvez configurer le [niveau de cohérence par défaut](consistency-levels.md) sur un compte.

## <a name="elements-in-an-azure-cosmos-account"></a>Éléments d’un compte Azure Cosmos

Le conteneur Azure Cosmos est l’unité d’extensibilité de base. Vous pouvez pratiquement avoir un débit approvisionné illimité (RU/s) et un stockage sur un conteneur. Azure Cosmos DB partitionne votre conteneur en toute transparence à l’aide de la clé de partition logique que vous spécifiez pour mettre à l’échelle votre débit approvisionné et votre stockage.

Actuellement, vous pouvez créer jusqu’à 50 comptes Azure Cosmos dans un abonnement Azure (il s’agit d’une limite logicielle qui peut être augmentée en envoyant une demande au support). Un seul compte Azure Cosmos peut gérer une quantité de données et un débit provisionné pratiquement illimités. Pour gérer vos données et votre débit provisionné, vous pouvez créer une ou plusieurs bases de données Azure Cosmos pour votre compte, puis, au sein de ces bases de données, créer un ou plusieurs conteneurs. L’image suivante illustre la hiérarchie de ces différents éléments dans un compte Azure Cosmos :

:::image type="content" source="./media/account-databases-containers-items/hierarchy.png" alt-text="Hiérarchie d’un compte Azure Cosmos" border="false":::

La création d’un compte dans votre abonnement Azure vous permet d’y gérer les données à l’aide de bases de données, de conteneurs et d’éléments. 

L’illustration suivante montre la hiérarchie de ces différentes entités dans un compte Azure Cosmos DB :

:::image type="content" source="./media/account-databases-containers-items/cosmos-entities.png" alt-text="Entités du compte Azure Cosmos" border="false":::

## <a name="azure-cosmos-databases"></a>Bases de données Azure Cosmos

Vous pouvez créer une ou plusieurs bases de données Azure Cosmos dans votre compte. Une base de données est analogue à un espace de noms. Une base de données est une unité de gestion pour un ensemble de conteneurs Azure Cosmos. Le tableau suivant montre la façon dont une base de données est mappée à différentes entités d’API :

| Entité Azure Cosmos | API SQL | API Cassandra | API Azure Cosmos DB pour MongoDB | API Gremlin | API de table |
| --- | --- | --- | --- | --- | --- |
|Base de données Azure Cosmos | Base de données | Espace de clés | Base de données | Base de données | N/D |

> [!NOTE]
> Avec les comptes d’API Table, lorsque vous créez votre première table, une base de données par défaut est automatiquement créée dans votre compte Azure Cosmos.

### <a name="operations-on-an-azure-cosmos-database"></a>Actions effectuées sur une base de données Azure Cosmos

Vous pouvez interagir avec une base de données Azure Cosmos à l’aide des API Azure Cosmos comme décrit dans le tableau suivant :

| Opération | Azure CLI | API SQL | API Cassandra | API Azure Cosmos DB pour MongoDB | API Gremlin | API de table |
| --- | --- | --- | --- | --- | --- | --- |
|Énumérer toutes les bases de données| Oui | Oui | Oui (la base de données est mappée sur un espace de clés) | Oui | N/D | N/D |
|Lire le contenu de la base de données| Oui | Oui | Oui (la base de données est mappée sur un espace de clés) | Oui | N/D | N/D |
|Create new database| Oui | Oui | Oui (la base de données est mappée sur un espace de clés) | Oui | N/D | N/D |
|Mettre à jour la base de données| Oui | Oui | Oui (la base de données est mappée sur un espace de clés) | Oui | N/D | N/D |

## <a name="azure-cosmos-containers"></a>Conteneurs Cosmos Azure

Un conteneur Azure Cosmos correspond à l’unité de scalabilité du débit et du stockage provisionnés. Un conteneur est partitionné horizontalement, puis répliqué dans plusieurs régions. Les éléments que vous ajoutez au conteneur sont automatiquement regroupés en partitions logiques, lesquelles sont distribuées sur des partitions physiques, en fonction de la clé de partition. Le débit d’un conteneur est distribué uniformément entre les partitions physiques. Pour en savoir plus sur le partitionnement et les clés de partition, consultez [Partitionnement des données](partitioning-overview.md). 

Lorsque vous créez un conteneur, vous configurez le débit avec l’un des modes suivants :

* Mode **Débit provisionné dédié**  : Le débit provisionné pour un conteneur lui est exclusivement réservé ; ce débit est défini par les contrats de niveau de service (SLA). Pour plus d’informations, consultez [Approvisionnement du débit sur un conteneur](how-to-provision-container-throughput.md).

* Mode **Débit provisionné partagé**  : ces conteneurs partagent le débit provisionné avec d’autres conteneurs de la même base de données (à l’exclusion des conteneurs qui ont été configurés pour le débit provisionné dédié). En d’autres termes, le débit provisionné pour la base de données est partagé entre tous les conteneurs « débit partagé ». Pour plus d’informations, consultez [Approvisionnement du débit sur une base de données](how-to-provision-database-throughput.md).

> [!NOTE]
> Vous pouvez configurer le débit partagé et dédié uniquement lors de la création de la base de données et du conteneur. Pour passer du mode de débit dédié au mode de débit partagé (et inversement) une fois le conteneur créé, vous devez créer un conteneur et migrer les données vers ce nouveau conteneur. Vous pouvez migrer les données à l’aide de la fonctionnalité Flux de modification d’Azure Cosmos DB.

Un conteneur Azure Cosmos peut être mis à l’échelle élastiquement, que vous créiez des conteneurs en utilisant des modes de débit dédiés ou partagés.

Un conteneur est un conteneur d’éléments indépendant de tout schéma. Les éléments dans conteneur peuvent avoir des schémas différents. Par exemple, un élément qui représente une personne et un élément qui représente une automobile peuvent être placés dans le *même conteneur*. Par défaut, tous les éléments que vous ajoutez à un conteneur sont automatiquement indexés, sans que vous ayez à effectuer une gestion explicite d’index ou de schéma. Vous pouvez personnaliser le comportement d’indexation en configurant la [stratégie d’indexation](index-overview.md) du conteneur. 

Vous pouvez définir la [durée de vie (TTL)](time-to-live.md) de certains éléments d’un conteneur ou de l’intégralité du conteneur afin de supprimer définitivement ces éléments du système. Azure Cosmos DB supprime automatiquement les éléments lorsque ceux-ci expirent. Cela permet également d’éviter qu’une requête envoyée au conteneur ne retourne des éléments ayant expiré. Pour plus d’informations, consultez [Configurer la durée de vie de votre conteneur](how-to-time-to-live.md).

Vous pouvez utiliser la fonctionnalité [Flux de modification](change-feed.md) pour vous abonner au journal des opérations de chacune des partitions logiques de votre conteneur. Flux de modification fournit le journal de toutes les mises à jour appliquées au conteneur ainsi que les images des éléments avant et après mise à jour. Pour plus d’informations, consultez [Générer des applications réactives en utilisant la fonctionnalité Flux de modification](serverless-computing-database.md). Vous pouvez également configurer la durée de conservation du flux de modification à l’aide de la stratégie de flux de modification du conteneur.

Vous pouvez inscrire des [procédures stockées, des déclencheurs, des fonctions définies par l’utilisateur](stored-procedures-triggers-udfs.md) et des [procédures de fusion](how-to-manage-conflicts.md) dans votre conteneur.

Vous pouvez spécifier une [contrainte de clé unique](unique-keys.md) dans votre conteneur Azure Cosmos. En créant une stratégie de clé unique, vous garantissez l’unicité d’une ou de plusieurs valeurs par clé de partition logique. Si vous créez un conteneur à l’aide d’une stratégie clé unique, aucun élément nouveau ou mis à jour dont les valeurs dupliquent les valeurs spécifiées par la contrainte de clé unique ne peut être créé. Pour plus d’informations, consultez [Contraintes de clés uniques](unique-keys.md).

Un conteneur est spécialisé dans les entités d’API, comme l’illustre le tableau suivant :

| Entité Azure Cosmos | API SQL | API Cassandra | API Azure Cosmos DB pour MongoDB | API Gremlin | API de table |
| --- | --- | --- | --- | --- | --- |
|Conteneur Azure Cosmos | Conteneur | Table de charge de travail | Collection | Graph | Table de charge de travail |

> [!NOTE]
> Lorsque vous créez des conteneurs, veillez à ne pas créer deux conteneurs dont les noms ne se distinguent que par la casse. En effet, certaines parties de la plateforme Azure ne respectant pas la casse, cela peut entraîner une confusion ou collision de la télémétrie et des actions sur les conteneurs ainsi nommés.

### <a name="properties-of-an-azure-cosmos-container"></a>Propriétés d’un conteneur Azure Cosmos

Un conteneur Azure Cosmos comprend un ensemble de propriétés définies par le système. Selon l’API que vous utilisez, il est possible que certaines propriétés ne soient pas directement exposées. Le tableau suivant répertorie les propriétés définies par le système :

| Propriété définie par le système | Générée par le système ou configurable par l’utilisateur | Objectif | API SQL | API Cassandra | API Azure Cosmos DB pour MongoDB | API Gremlin | API de table |
| --- | --- | --- | --- | --- | --- | --- | --- |
|\_rid | Générée par le système | Identificateur unique du conteneur | Oui | Non | Non | Non | Non |
|\_etag | Générée par le système | Étiquette d’entité utilisée pour le contrôle de l’accès concurrentiel optimiste | Oui | Non | Non | Non | Non |
|\_ts | Générée par le système | Dernière mise à jour de l’horodatage du conteneur | Oui | Non | Non | Non | Non |
|\_self | Générée par le système | URI adressable du conteneur | Oui | Non | Non | Non | Non |
|id | Configurable par l’utilisateur | Nom unique du conteneur défini par l’utilisateur | Oui | Oui | Oui | Oui | Oui |
|indexingPolicy | Configurable par l’utilisateur | Permet de modifier le chemin d’accès de l’index, le type d’index et le mode d’indexation | Oui | Non | Non | Non | Oui |
|timeToLive | Configurable par l’utilisateur | Permet de supprimer automatiquement les éléments d’un conteneur après une période donnée. Pour plus d’informations, consultez [Durée de vie](time-to-live.md). | Oui | Non | Non | Non | Oui |
|changeFeedPolicy | Configurable par l’utilisateur | Utilisé pour lire les modifications apportées aux éléments d’un conteneur. Pour plus d’informations, consultez [Flux de modification](change-feed.md). | Oui | Non | Non | Non | Oui |
|uniqueKeyPolicy | Configurable par l’utilisateur | Utilisée pour garantir l’unicité d’une ou plusieurs valeurs au sein d’une partition logique. Pour plus d’informations, consultez [Contraintes de clé unique](unique-keys.md). | Oui | Non | Non | Non | Oui |

### <a name="operations-on-an-azure-cosmos-container"></a>Actions effectuées sur un conteneur Azure Cosmos

Un conteneur Azure Cosmos prend en charge les opérations suivantes, qui peuvent être effectuées lorsque vous utilisez l’une des API Azure Cosmos :

| Opération | Azure CLI | API SQL | API Cassandra | API Azure Cosmos DB pour MongoDB | API Gremlin | API de table |
| --- | --- | --- | --- | --- | --- | --- |
| Énumérer les conteneurs d’une base de données | Oui | Oui | Oui | Oui | N/D | N/D |
| Lire le contenu d’un conteneur | Oui | Oui | Oui | Oui | N/D | N/D |
| Créer un conteneur | Oui | Oui | Oui | Oui | N/D | N/D |
| Mettre à jour un conteneur | Oui | Oui | Oui | Oui | N/D | N/D |
| Supprimer un conteneur | Oui | Oui | Oui | Oui | N/D | N/D |

## <a name="azure-cosmos-items"></a>Éléments Azure Cosmos

En fonction de l’API que vous utilisez, un élément Azure Cosmos peut représenter un document d’une collection, une ligne de table, un nœud de graphe ou une arête de graphe. Le tableau suivant présente le mappage d’entités d’API vers un élément Azure Cosmos :

| Entité Cosmos | API SQL | API Cassandra | API Azure Cosmos DB pour MongoDB | API Gremlin | API de table |
| --- | --- | --- | --- | --- | --- |
|Élément Azure Cosmos | Élément | Ligne | Document | Nœud ou arête | Élément |

### <a name="properties-of-an-item"></a>Propriétés d’un élément

Chaque élément Azure Cosmos comprend des propriétés définies par le système. Selon l’API que vous utilisez, il est possible que certaines d’entre elles ne soient pas directement exposées.

| Propriété définie par le système | Générée par le système ou configurable par l’utilisateur| Objectif | API SQL | API Cassandra | API Azure Cosmos DB pour MongoDB | API Gremlin | API de table |
| --- | --- | --- | --- | --- | --- | --- | --- |
|\_rid | Générée par le système | Identificateur unique de l’élément | Oui | Non | Non | Non | Non |
|\_etag | Générée par le système | Étiquette d’entité utilisée pour le contrôle de l’accès concurrentiel optimiste | Oui | Non | Non | Non | Non |
|\_ts | Générée par le système | Horodatage de la dernière mise à jour de l’élément | Oui | Non | Non | Non | Non |
|\_self | Générée par le système | URI adressable de l’élément | Oui | Non | Non | Non | Non |
|id | Vous pouvez soit utiliser | Nom unique défini par l’utilisateur dans une partition logique. | Oui | Oui | Oui | Oui | Oui |
|Propriétés arbitraires définies par l’utilisateur | Défini par l’utilisateur | Propriétés définies par l’utilisateur représentées sous forme d’API native (notamment JSON, BSON et CQL) | Oui | Oui | Oui | Oui | Oui |

> [!NOTE]
> L’unicité de la `id`propriété est appliqué dans chaque partition logique. Plusieurs documents peuvent avoir la même `id` propriété avec des valeurs de clé de partition différentes.

### <a name="operations-on-items"></a>Actions possibles sur les éléments

Les éléments Azure Cosmos prennent en charge les opérations suivantes. Vous pouvez utiliser n’importe laquelle des API Azure Cosmos pour effectuer ces opérations.

| Opération | Azure CLI | API SQL | API Cassandra | API Azure Cosmos DB pour MongoDB | API Gremlin | API de table |
| --- | --- | --- | --- | --- | --- | --- |
| Insérer, remplacer, supprimer, upsert, lire | Non | Oui | Oui | Oui | Oui | Oui |

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment gérer votre compte Azure Cosmos et d’autres concepts :

* [Comment gérer votre compte Azure Cosmos](how-to-manage-database-account.md)
* [Diffusion mondiale](distribute-data-globally.md)
* [Niveaux de cohérence](consistency-levels.md)
* [Point de terminaison du service de réseau virtuel pour votre compte Azure Cosmos](how-to-configure-vnet-service-endpoint.md)
* [Pare-feu IP de votre compte Azure Cosmos](how-to-configure-firewall.md)
* [Comment ajouter et supprimer des régions Azure à votre compte Azure Cosmos](how-to-manage-database-account.md)
* [Contrats de niveau de service Azure Cosmos DB](https://azure.microsoft.com/support/legal/sla/cosmos-db/v1_2/)
