---
title: Choisir les colonnes de distribution – Hyperscale (Citus) – Azure Database pour PostgreSQL
description: Découvrez comment choisir des colonnes de distribution dans des scénarios courants dans Azure Database pour PostgreSQL – Hyperscale (Citus).
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: conceptual
ms.date: 05/06/2019
ms.openlocfilehash: 129eff8c954c0c5469d3607e6ae16ce3202630ed
ms.sourcegitcommit: 50802bffd56155f3b01bfb4ed009b70045131750
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91929334"
---
# <a name="choose-distribution-columns-in-azure-database-for-postgresql--hyperscale-citus"></a>Choisir les colonnes de distribution dans Azure Database pour PostgreSQL – Hyperscale (Citus)

Le choix de la colonne de distribution de chaque table est l’une des décisions les plus importantes que vous allez prendre en matière de modélisation. Azure Database pour PostgreSQL – Hyperscale (Citus) stocke des lignes dans des partitions en fonction de la valeur de la colonne de distribution des lignes.

Le choix correct permet de regrouper les données sur les mêmes nœuds physiques, ce qui a pour effet d’accélérer les requêtes et d’ajouter la prise en charge toutes les fonctionnalités SQL. Un choix incorrect ralentit le système et ne permet pas de prendre en charge toutes les fonctionnalités SQL sur les nœuds.

Cet article donne des conseils sur la colonne de distribution pour les deux scénarios Hyperscale (Citus) les plus courants.

### <a name="multi-tenant-apps"></a>Applications multi-locataires

L’architecture mutualisée utilise une forme de modélisation de base de données hiérarchique pour distribuer les requêtes sur les nœuds dans le groupe de serveurs. Le haut de la hiérarchie de données est appelé l’*ID de locataire* et doit être stocké dans une colonne sur chaque table.

Hyperscale (Citus) inspecte les requêtes pour repérer l’ID de locataire impliqué et recherche la partition de table correspondante. Il achemine la requête à un nœud Worker unique qui contient la partition. L’exécution d’une requête avec toutes les données pertinentes placées sur le même nœud est appelée « colocation ».

Le diagramme suivant illustre la colocation dans le modèle de données mutualisé. Il contient deux tables (comptes et campagnes), chacune répartie par `account_id`. Les zones grisées représentent des partitions. Les partitions vertes sont stockées ensemble sur un nœud Worker, et les partitions bleues sur un autre. Notez comment une requête de jointure entre des comptes et des campagnes dispose de toutes les données nécessaires regroupées sur un seul nœud quand les deux tables sont limitées à la même valeur account\_id.

![Colocation mutualisée](media/concepts-hyperscale-choosing-distribution-column/multi-tenant-colocation.png)

Pour appliquer cette conception dans votre propre schéma, identifiez ce qui constitue un locataire dans votre application. Les instances courantes incluent une société, un compte, une organisation ou un client. Le nom de colonne ressemble à `company_id` ou `customer_id`. Examinez chacune de vos requêtes et demandez-vous si elle fonctionnerait si elle contenait des clauses WHERE supplémentaires pour restreindre toutes les tables impliquées aux lignes avec le même ID de locataire ?
Les requêtes dans le modèle mutualisé sont étendues à un locataire. Par exemple, des requêtes sur les ventes ou l’inventaire sont étendues dans un magasin donné.

#### <a name="best-practices"></a>Meilleures pratiques

-   **Partitionnez les tables distribuées selon une colonne tenant\_id commune.** Par exemple, dans une application SaaS où les locataires sont des entreprises, la valeur tenant\_id est susceptible d’être la valeur company\_id.
-   **Convertissez les tables de petite taille partagées entre locataires en tables de référence.** Lorsque plusieurs locataires partagent une petite table d’informations, distribuez-la comme une table de référence.
-   **Filtrez toutes les requêtes d’application selon tenant\_id.** Chaque requête doit demander des informations pour un seul locataire à la fois.

Lisez le [didacticiel mutualisé](./tutorial-design-database-hyperscale-multi-tenant.md) pour obtenir un exemple de création de ce type d’application.

### <a name="real-time-apps"></a>Applications en temps réel

L’architecture mutualisée présente une structure hiérarchique et utilise la colocation de données pour acheminer les requêtes par locataire. En revanche, les architectures en temps réel dépendent des propriétés de distribution spécifiques de leurs données à obtenir un traitement hautement parallèle.

Nous utilisons « ID d’entité » pour désigner les colonnes de distribution dans le modèle en temps réel. Les entités standard sont des utilisateurs, des hôtes ou des appareils.

Les requêtes en temps réel nécessitent en général des agrégats numériques regroupés par date ou par catégorie. Hyperscale (Citus) envoie ces requêtes à chaque partition pour obtenir des résultats partiels et assemble la réponse finale sur le nœud coordinateur. Les requêtes s’exécutent plus rapidement lorsqu’un maximum de nœuds contribuent, et lorsque aucun nœud unique ne doit faire une quantité disproportionnée du travail.

#### <a name="best-practices"></a>Meilleures pratiques

-   **Choisissez une colonne présentant une cardinalité élevée comme colonne de distribution.** À titre de comparaison, un champ d’état sur une table de commandes avec les valeurs « nouvelle », « payée » et « livrée » est un choix médiocre de colonne de distribution. Il part du principe que seules ces quelques valeurs existent, ce qui limite le nombre de partitions pouvant stocker les données et le nombre de nœuds pouvant les traiter. Parmi les colonnes présentant une cardinalité élevée, il est également judicieux de choisir les celles qui sont fréquemment utilisées dans les clauses group-by ou en tant que clés de jointure.
-   **Choisissez une colonne avec une distribution uniforme.** Si vous distribuez une table sur une colonne déviée vers certaines valeurs courantes, les données de la table ont tendance à s’accumuler dans certaines partitions. Les nœuds contenant ces partitions finissent par effectuer plus de travail que les autres nœuds.
-   **Distribuez les tables de faits et de dimension sur leurs colonnes communes.**
    Votre table de faits ne peut avoir qu’une seule clé de distribution. Les tables qui créent une jointure sur une autre clé ne se trouvent pas au même emplacement que la table de faits. Choisissez une dimension pour effectuer la colocation selon la taille des lignes de jointure et la fréquence de jointure.
-   **Modifiez des tables de dimension en tables de référence.** Si une table de dimension ne peut pas être au même emplacement que la table de faits, vous pouvez améliorer les performances des requêtes en distribuant des copies de la table de dimension à tous les nœuds sous la forme d’une table de référence.

Lisez le [didacticiel de tableau de bord mutualisé](./tutorial-design-database-hyperscale-realtime.md) pour obtenir un exemple de création de ce type d’application.

### <a name="time-series-data"></a>Données de série chronologique

Dans une charge de travail de série chronologique, les applications interrogent les informations récentes lors de l’archivage d’anciennes informations.

L’erreur la plus fréquente lors de la modélisation des informations de série chronologique dans Hyperscale (Citus) consiste à utiliser l’horodatage comme colonne de distribution. Une distribution de hachage basée sur le temps distribue le temps de façon apparemment aléatoire dans différentes partitions plutôt que de conserver des plages de temps groupées dans des partitions. Les requêtes qui impliquent du temps font généralement référence à des plages de temps, par exemple, pour obtenir les données les plus récentes. Ce type de distribution de hachage entraîne une surcharge du réseau.

#### <a name="best-practices"></a>Meilleures pratiques

-   **Ne choisissez pas un horodatage comme colonne de distribution.** Choisissez une colonne de distribution différente. Dans une application mutualisée, utilisez l’ID de locataire ou, dans une application en temps réel, l’ID d’entité.
-   **Utilisez le partitionnement de table PostgreSQL pour les données temporelles à la place.** Utilisez le partitionnement de table pour diviser une table volumineuse de données chronologiques en plusieurs tables héritées contenant chacune différentes plages de temps. La distribution d’une table partitionnée Postgres dans Hyperscale (Citus) crée des partitions pour les tables héritées.

## <a name="next-steps"></a>Étapes suivantes
- Découvrez comment la [colocation](concepts-hyperscale-colocation.md) entre les requêtes distribuées permettent d’exécuter rapidement les requêtes.
