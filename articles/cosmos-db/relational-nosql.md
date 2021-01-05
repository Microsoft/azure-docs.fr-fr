---
title: Comprendre les différences entre les bases de données relationnelles et NoSQL d’Azure Cosmos DB
description: Cet article énumère les différences entre les bases de données relationnelles et NoSQL.
author: TheovanKraay
ms.author: thvankra
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 12/16/2019
ms.reviewer: sngun
ms.openlocfilehash: d986106337eb1ede2f6d61303d8a4c487bbed276
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93088469"
---
# <a name="understanding-the-differences-between-nosql-and-relational-databases"></a>Comprendre les différences entre les bases de données relationnelles et NoSQL d’Azure Cosmos DB
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Cet article énumère certains des principaux avantages des bases de données NoSQL par rapport aux bases de données relationnelles. Nous aborderons également certains des défis liés à l’utilisation de NoSQL. Pour examiner en détail les différents magasins de données existants, consultez notre article relatif au [choix du magasin de données approprié](/azure/architecture/guide/technology-choices/data-store-overview).

## <a name="high-throughput"></a>Débit élevé

L’un des défis les plus évidents de la gestion d’un système de base de données relationnelle est que la plupart des moteurs relationnels appliquent des restrictions et des verrous pour appliquer une [sémantique ACID](https://en.wikipedia.org/wiki/ACID) stricte. Cette approche présente des avantages en matière de garantie d’un état de données cohérent dans la base de données. Toutefois, il existe des compromis importants en ce qui concerne la concurrence, la latence et la disponibilité. En raison de ces restrictions architecturales fondamentales, les volumes transactionnels élevés peuvent entraîner la nécessité de partitionner manuellement les données. L’implémentation manuelle d’un partitionnement peut être un exercice fastidieux et pénible.

Dans ces scénarios, les [bases de données distribuées](https://en.wikipedia.org/wiki/Distributed_database) peuvent offrir une solution plus évolutive. Toutefois, la maintenance peut toujours être un exercice coûteux et chronophage. Les administrateurs devront peut-être effectuer des tâches supplémentaires pour s’assurer que la nature distribuée du système est transparente. Ils devront peut-être également tenir compte de la nature « déconnectée » de la base de données.

[Azure Cosmos DB](./introduction.md) simplifie ces défis en étant déployé dans le monde entier dans toutes les régions Azure. Les plages de partitions peuvent être réparties de manière dynamique pour augmenter la taille de la base de données en fonction de l’application, tout en maintenant une haute disponibilité. La gouvernance des ressources précise, cloud native, à l’architecture mutualisée et rigoureusement contrôlée facilite des [garanties de latence](./consistency-levels.md#consistency-levels-and-latency) et des performances prévisibles. Le partitionnement étant complètement managé, les administrateurs n’ont pas besoin d’écrire du code ou de gérer des partitions.

Si vos volumes transactionnels atteignent des niveaux extrêmes, par exemple plusieurs milliers de transactions par seconde, vous devriez envisager une base de données NoSQL distribuée. Envisagez Azure Cosmos DB pour une efficacité maximale, une facilité de maintenance et une réduction du coût total de possession.

:::image type="content" source="./media/relational-or-nosql/backend-scaled.png" alt-text="BackEnd" border="false":::

## <a name="hierarchical-data"></a>Données hiérarchiques

Il existe un nombre important de cas d’usage où les transactions dans la base de données peuvent contenir de nombreuses relations parent-enfant. Ces relations peuvent croître considérablement au fil du temps et s’avérer difficiles à gérer. Les formes de [bases de données hiérarchiques](https://en.wikipedia.org/wiki/Hierarchical_database_model) émergent au cours des années 1980, mais n’étaient pas populaires en raison de l’inefficacité du stockage. Ils ont également perdu de leur attrait lorsque le [modèle relationnel de Ted Codd](https://en.wikipedia.org/wiki/Relational_model) est devenu la norme de facto utilisée par pratiquement tous les systèmes de gestion de base de données courants.

Toutefois, aujourd’hui, la popularité des bases de données de style document a augmenté de manière significative. Ces bases de données peuvent être considérées comme une réinvention du paradigme de la base de données hiérarchique, désormais libérées des préoccupations concernant le coût du stockage des données sur disque. Par conséquent, le maintien de nombreuses relations d’entité parent-enfant complexes dans une base de données relationnelle peut désormais être considérée comme un anti-modèle par rapport aux approches modernes orientées documents.

L’émergence de la [conception orientée objet](https://en.wikipedia.org/wiki/Object-oriented_design) et l’[incohérence d’impédance](https://en.wikipedia.org/wiki/Object-relational_impedance_mismatch) qui survient lorsqu’elle est combinée avec des modèles relationnels mettent également en évidence un anti-modèle dans les bases de données relationnelles pour certains cas d’usage. Des coûts de maintenance cachés, mais souvent élevés, peuvent en résulter. Bien que les [approches ORM](https://en.wikipedia.org/wiki/Object-relational_mapping) aient évolué pour atténuer partiellement ce risque, les bases de données orientées documents fusionnent néanmoins beaucoup mieux avec les approches orientées objet. Avec cette approche, les développeurs ne sont pas obligés de s’engager à utiliser des pilotes ORM ou des [moteurs de base de données OO](https://en.wikipedia.org/wiki/Object_database) spécifiques à un langage sur mesure. Si vos données contiennent de nombreuses relations parent-enfant et des niveaux de hiérarchie détaillés, vous pouvez envisager d’utiliser une base de données de documents NoSQL telle que l’[API SQL Azure Cosmos DB](./introduction.md).

:::image type="content" source="./media/relational-or-nosql/order-orderdetails.jpg" alt-text="OrderDetails":::

## <a name="complex-networks-and-relationships"></a>Réseaux et relations complexes

Ironiquement, étant donné leur nom, les bases de données relationnelles présentent une solution moins qu’optimale pour la modélisation des relations approfondies et complexes. Cela est dû au fait que les relations entre les entités n’existent pas réellement dans une base de données relationnelle. Elles doivent être calculées au moment de l’exécution, avec des relations complexes nécessitant des jointures cartésiennes afin d’autoriser le mappage à l’aide de requêtes. Par conséquent, les opérations deviennent exponentiellement plus coûteuses en matière de calcul à mesure que le nombre de relations augmente. Dans certains cas, une base de données relationnelle qui tente de gérer ces entités devient inutilisable.

Plusieurs formes de bases de données de « réseau » sont apparues à l’époque où les bases de données relationnelles ont vu le jour, mais comme pour les bases de données hiérarchiques, ces systèmes ont eu du mal à gagner en popularité. La lenteur de l’adoption était due à un manque de cas d’usage à l’époque, et à des inefficacités de stockage. Aujourd’hui, les moteurs de base de données de graphe pourraient être considérés comme une réémergence du paradigme des bases de données de réseau. L’avantage principal de ces systèmes est que les relations sont stockées en tant que « citoyens de première classe » dans la base de données. Ainsi, le parcours des relations peut être effectué en une durée constante, plutôt que d’augmenter la complexité temporelle avec chaque nouvelle jointure ou produit croisé.

Si vous entretenez un réseau complexe de relations dans votre base de données, vous pouvez envisager d’utiliser une base de données de graphe telle que l’[API Gremlin d’Azure Cosmos DB](./graph-introduction.md) pour la gestion de ces données.

:::image type="content" source="./media/relational-or-nosql/graph.png" alt-text="Le schéma de base de données représente plusieurs employés et départements connectés les uns aux autres.":::

Azure Cosmos DB est un service de base de données multi-modèle, qui offre une projection d’API pour tous les principaux types de modèles NoSQL : famille de colonnes, document, graphe et clé-valeur. Les couches de l’API de document [Gremlin (graphe)](./gremlin-support.md) et SQL (cœur) sont entièrement interopérables. Cela présente des avantages pour basculer entre les différents modèles au niveau de la programmabilité. Les banques de graphes peuvent être interrogées en matière de parcours réseau complexes et de transactions modélisées en tant qu’enregistrements de document dans la même banque.

## <a name="fluid-schema"></a>Schéma fluide

Une autre caractéristique particulière des bases de données relationnelles est que les schémas doivent être définis au moment de la conception. Cela présente des avantages en matière d’intégrité référentielle et de conformité des données. Toutefois, cela peut également être restrictif au fur et à mesure que l’application se développe. La réponse aux modifications du schéma sur des modèles séparés logiquement et partageant la même table ou la même définition de base de données peut devenir complexe au fil du temps. Ces cas d’usage bénéficient souvent du fait que le schéma est dévolu à l’application pour la gérer par enregistrement. Cela requiert que la base de données soit « indépendante du schéma » et que les enregistrements soient « auto-descriptifs » en matière de données qu’ils contiennent.

Si vous gérez des données dont les structures changent constamment à un taux élevé, en particulier si les transactions peuvent provenir de sources externes où il est difficile d’appliquer la conformité dans la base de données, vous pouvez envisager d’utiliser une approche plus indépendante du schéma à l’aide d’un service de base de données NoSQL managé comme Azure Cosmos DB.

## <a name="microservices"></a>Microservices

Le modèle des [microservices](https://en.wikipedia.org/wiki/Microservices) a connu une croissance importante au cours des dernières années. Ce modèle a ses racines dans l’[architecture orientée services](https://en.wikipedia.org/wiki/Service-oriented_architecture). La norme de facto pour la transmission de données dans ces architectures modernes de microservices est [JSON](https://en.wikipedia.org/wiki/JSON), qui est également le support de stockage pour la grande majorité des bases de données NoSQL orientées documents. Cela permet aux banques de documents NoSQL d’être bien plus adaptées à la persistance et à la synchronisation (à l’aide de [modèles d’approvisionnement d’événements](https://en.wikipedia.org/wiki/Event-driven_architecture)) sur des implémentations complexes de microservices. Les bases de données relationnelles plus traditionnelles peuvent être bien plus complexes à gérer dans ces architectures. Cela est dû à la plus grande quantité de transformation requise pour l’état et la synchronisation entre les API. Azure Cosmos DB en particulier dispose d’un certain nombre de fonctionnalités qui le rendent encore plus adapté aux architectures de microservices basées sur JSON que de nombreuses bases de données NoSQL :

* un choix de types de données JSON pures ;
* un moteur JavaScript et une [API de requête](./javascript-query-api.md) intégrés à la base de données ;
* un [flux de modification](./change-feed.md) de pointe auquel les clients peuvent s’abonner pour être averti des modifications apportées à un conteneur.

## <a name="some-challenges-with-nosql-databases"></a>Quelques défis avec les bases de données NoSQL

Bien qu’il existe des avantages évidents lors de l’implémentation de bases de données NoSQL, vous devriez prendre en compte certains des défis qu’elles présentent. Ces défis peuvent ne pas être présents au même niveau lorsque vous travaillez avec le modèle relationnel :

* Transactions avec de nombreuses relations pointant vers la même entité.
* Transactions nécessitant une cohérence forte sur l’ensemble du jeu de données.

En ce qui concerne le premier défi, la règle empirique dans les bases de données NoSQL est généralement la dénormalisation, qui, comme nous l’avons expliqué précédemment, produit des lectures plus efficaces dans un système distribué. Toutefois, cette approche pose certains problèmes de conception. Prenons l’exemple d’un produit lié à une catégorie et plusieurs étiquettes :

:::image type="content" source="./media/relational-or-nosql/many-joins.png" alt-text="Joins":::

Une approche de meilleure pratique dans une base de données de documents NoSQL consiste à dénormaliser le nom de catégorie et les noms d’étiquette directement dans un « document produit ». Toutefois, pour que les catégories, les étiquettes et les produits soient synchronisés, les options de conception permettant de faciliter cette tâche ont ajouté une complexité de la maintenance, car les données sont dupliquées sur plusieurs enregistrements du produit, plutôt que d’être une simple mise à jour dans une relation « un-à-plusieurs », et une jointure pour récupérer les données. 

Le compromis est que les lectures sont plus efficaces dans l’enregistrement dénormalisé et deviennent de plus en plus efficaces à mesure que le nombre d’entités conceptuellement jointes augmente. Cependant, tout comme l’efficacité de la lecture augmente avec le nombre croissant d’entités jointes dans un enregistrement dénormalisé, la complexité de la maintenance pour garder les entités synchronisées augmente également. L’une des façons d’atténuer ce compromis consiste à créer un [modèle de données hybride](./modeling-data.md#hybrid-data-models).

Bien qu’il y ait plus de flexibilité dans les bases de données NoSQL pour gérer ces compromis, une flexibilité accrue peut également produire davantage de décisions de conception. Consultez notre article [Comment modéliser et partitionner des données sur Azure Cosmos DB à l’aide d’un exemple réel](./how-to-model-partition-example.md), qui comprend une approche visant à maintenir [la synchronisation des données utilisateur dénormalisées](./how-to-model-partition-example.md#denormalizing-usernames) où les utilisateurs se trouvent non seulement dans des partitions différentes, mais dans des conteneurs différents.

En ce qui concerne la cohérence forte, il est rare qu’elle soit requise dans l’ensemble du jeu de données. Toutefois, dans les cas où cela est nécessaire, cela peut représenter un défi dans les bases de données distribuées. Pour garantir une cohérence forte, les données doivent être synchronisées sur tous les réplicas et régions avant d’autoriser les clients à les lire. Cela peut augmenter la latence des lectures.

Là encore, Azure Cosmos DB offre plus de flexibilité que les bases de données relationnelles pour les divers compromis pertinents ici, mais, pour les implémentations à petite échelle, cette approche peut ajouter d’autres considérations de conception. Pour plus d’informations sur ce sujet, consultez notre article sur [Cohérence, disponibilité et compromis sur les performances](./consistency-levels.md).

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment gérer votre compte Azure Cosmos et d’autres concepts :

* [Comment gérer votre compte Azure Cosmos](how-to-manage-database-account.md)
* [Diffusion mondiale](distribute-data-globally.md)
* [Niveaux de cohérence](consistency-levels.md)
* [Utilisation des bases de données, des conteneurs et des éléments Azure Cosmos](account-databases-containers-items.md)
* [Point de terminaison du service de réseau virtuel pour votre compte Azure Cosmos](how-to-configure-vnet-service-endpoint.md)
* [Pare-feu IP de votre compte Azure Cosmos](how-to-configure-firewall.md)
* [Comment ajouter et supprimer des régions Azure à votre compte Azure Cosmos](how-to-manage-database-account.md)
* [Contrats de niveau de service Azure Cosmos DB](https://azure.microsoft.com/support/legal/sla/cosmos-db/v1_2/)