---
title: Niveaux de cohérence Apache Cassandra et Azure Cosmos DB
description: Niveaux de cohérence Apache Cassandra et Azure Cosmos DB.
author: TheovanKraay
ms.author: thvankra
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: conceptual
ms.date: 10/12/2020
ms.reviewer: sngun
ms.openlocfilehash: 3107610215d5b37c43124ce4129b2eb5437e3b62
ms.sourcegitcommit: 66479d7e55449b78ee587df14babb6321f7d1757
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97516821"
---
# <a name="apache-cassandra-and-azure-cosmos-db-cassandra-api-consistency-levels"></a>Niveaux de cohérence API Cassandra d’Apache Cassandra et Azure Cosmos DB
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

Contrairement à Azure Cosmos DB, Apache Cassandra ne fournit pas en mode natif des garanties de cohérence précises. Au lieu de cela, Apache Cassandra fournit un niveau de cohérence d’écriture et un niveau de cohérence de lecture, pour permettre les compromis de haute disponibilité, de cohérence et de latence. Lors de l’utilisation de l’API Cassandra Azure Cosmos DB :

* Le niveau de cohérence d’écriture d’Apache Cassandra est mappé au niveau de cohérence par défaut configuré sur votre compte Azure Cosmos. La cohérence d’une opération d’écriture ne peut pas être changée en fonction de la demande.

* Azure Cosmos DB mappe de manière dynamique le niveau de cohérence de lecture spécifié par le pilote client Cassandra à l’un des niveaux de cohérence Azure Cosmos DB configurés dynamiquement sur une requête de lecture.

## <a name="multi-region-writes-vs-single-region-writes"></a>Écritures multirégions et écritures dans une seule région

La base de données Apache Cassandra est un système multimaître par défaut qui n’offre pas d’option prête à l’emploi pour les écritures dans une seule région avec la réplication multirégion pour les lectures. Toutefois, Azure Cosmos DB fournit une capacité clé en main d’avoir des configurations dans une seule région ou [multirégions](how-to-multi-master.md). L’un des avantages de la possibilité de choisir une configuration d’écriture dans une seule région pour plusieurs régions est d’éviter des conflits entre régions, et la possibilité de maintenir une cohérence forte dans plusieurs régions. 

Avec les écritures dans une seule région, vous pouvez maintenir une cohérence forte, tout en conservant un niveau de haute disponibilité entre les régions avec le [basculement automatique](high-availability.md#multi-region-accounts-with-a-single-write-region-write-region-outage). Dans cette configuration, vous pouvez toujours exploiter la localité des données pour réduire la latence de lecture en passant à la cohérence finale en fonction de la demande. Outre ces fonctionnalités, la plateforme Azure Cosmos DB offre également la possibilité d’activer la [redondance de zone](high-availability.md#availability-zone-support) lors de la sélection d’une région. Par conséquent, contrairement à Apache Cassandra natif, Azure Cosmos DB vous permet de naviguer dans le [spectre de compromis](consistency-levels.md#rto) du théorème CAP avec plus de granularité.

## <a name="mapping-consistency-levels"></a>Correspondance des niveaux de cohérence

La plateforme Azure Cosmos DB fournit un ensemble de cinq paramètres de cohérence orientés cas d’utilisation en entreprise, bien définis et adaptés à la réplication, ainsi que les compromis définis par le [théorème CAP](https://en.wikipedia.org/wiki/CAP_theorem) et le [théorème PACLC](https://en.wikipedia.org/wiki/PACELC_theorem). Comme cette approche diffère considérablement d’Apache Cassandra, nous vous recommandons de prendre le temps d’examiner et de comprendre les paramètres de cohérence d’Azure Cosmos DB dans notre [documentation](consistency-levels.md) ou de regarder ce bref guide [vidéo](https://www.youtube.com/watch?v=t1--kZjrG-o) pour comprendre les paramètres de cohérence dans la plateforme Azure Cosmos DB.

Le tableau suivant illustre les mappages possibles entre Apache Cassandra et les niveaux de cohérence Azure Cosmos DB lors de l’utilisation d’API Cassandra. Cela montre les configurations pour les lectures dans une seule région ou multirégions avec des écritures dans une seule région et multirégions.

> [!NOTE]
> Il ne s’agit pas de mappages exacts. Nous avons plutôt fourni les analogues les plus proches à Apache Cassandra et levé l’ambiguïté sur les différences qualitatives de la colonne la plus à droite. Comme mentionné ci-dessus, nous vous recommandons de consulter les [paramètres de cohérence](consistency-levels.md) d’Azure Cosmos DB. 

:::image type="content" source="./media/cassandra-consistency/account.png" alt-text="Mappage du niveau de compte de cohérence Cassandra" lightbox="./media/cassandra-consistency/account.png" :::

:::image type="content" source="./media/cassandra-consistency/dynamic.png" alt-text="Mappage dynamique de cohérence Cassandra" lightbox="./media/cassandra-consistency/dynamic.png" :::

Si votre compte Azure Cosmos est configuré avec un niveau de cohérence autre que la cohérence forte, vous pouvez déterminer la probabilité que vos clients bénéficieront de lectures fortes et cohérentes pour vos charges de travail en examinant la métrique *Probabilistically Bounded Staleness* (PBS). Cette métrique est exposée dans le portail Azure. Pour en savoir plus, consultez [Surveiller la métrique de probabilités en fonction de l’obsolescence limitée (PBS)](how-to-manage-consistency.md#monitor-probabilistically-bounded-staleness-pbs-metric).

La métrique de probabilités en fonction de l’obsolescence limitée montre comment la valeur éventuelle représente la cohérence éventuelle. Cette métrique fournit une idée de la fréquence à laquelle vous pouvez obtenir une cohérence plus forte que le niveau de cohérence que vous avez configuré sur votre compte Azure Cosmos. En d’autres termes, vous pouvez voir la probabilité (mesurée en millisecondes) d’obtenir des lectures fortement cohérentes pour une combinaison de régions d’écriture et de lecture.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les niveaux de distribution et de cohérence globaux pour Azure Cosmos DB :

* [Vue d’ensemble de la distribution mondiale](distribute-data-globally.md)
* [Vue d’ensemble d’un niveau de cohérence](consistency-levels.md)
* [Haute disponibilité](high-availability.md)
