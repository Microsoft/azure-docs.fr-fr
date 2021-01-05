---
title: Rééquilibrage des partitions – Hyperscale (Citus) – Azure Database pour PostgreSQL
description: Distribuez les partitions uniformément entre les serveurs pour obtenir un meilleur niveau de performance.
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: how-to
ms.date: 11/17/2020
ms.openlocfilehash: b69c4fc3ff16cf30181a3529f61af7126b92950b
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95026384"
---
# <a name="rebalance-shards-in-hyperscale-citus-server-group"></a>Rééquilibrage des partitions dans un groupe de serveurs Hyperscale (Citus)

Pour tirer parti des nœuds nouvellement ajoutés, vous devez rééquilibrer les [partitions](concepts-hyperscale-distributed-data.md#shards) de la table distribuée, ce qui signifie déplacer des partitions de nœuds existants vers les nouveaux nœuds. Vérifiez d’abord que les nouveaux Workers ont bien effectué le provisionnement. Ensuite, démarrez le rééquilibreur de partition, connectez-vous au nœud coordinateur du cluster avec psql, puis exécutez :

```sql
SELECT rebalance_table_shards('distributed_table_name');
```

La fonction [rebalance_table_shards](reference-hyperscale-functions.md#rebalance_table_shards) permet de rééquilibrer toutes les tables dans le groupe [colocation](concepts-hyperscale-colocation.md) de la table nommée dans son argument. Vous n’avez donc pas besoin d’appeler la fonction pour chaque table distribuée. Il vous suffit de l’appeler sur une table représentative à partir de chaque groupe de colocation.

**Étapes suivantes**


- Apprenez-en davantage sur les [options de performances](concepts-hyperscale-configuration-options.md) du groupe de serveurs.
- [Scale-up ou scale-out d’un groupe de serveurs](howto-hyperscale-scale-grow.md)
- Consultez la documentation de référence [rebalance_table_shards](reference-hyperscale-functions.md#rebalance_table_shards)
