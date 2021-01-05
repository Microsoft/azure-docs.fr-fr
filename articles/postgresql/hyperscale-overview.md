---
title: Vue d’ensemble d’Azure Database pour PostgreSQL - Hyperscale (Citus)
description: Fournit une vue d’ensemble de l’option de déploiement Hyperscale (Citus)
author: jonels-msft
ms.author: jonels
ms.custom: mvc
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: overview
ms.date: 09/01/2020
ms.openlocfilehash: 1734128384d63749d3c777cf6315278fced9d140
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95025138"
---
# <a name="what-is-azure-database-for-postgresql---hyperscale-citus"></a>Qu’est qu’Azure Database pour PostgreSQL - Hyperscale (Citus) ?

Azure Database pour PostgreSQL est un service de base de données relationnelle dans le cloud de Microsoft conçu pour les développeurs. Il est basé sur la version communauté du moteur de base de données [PostgreSQL](https://www.postgresql.org/) open source.

Hyperscale (Citus) est une option de déploiement qui met horizontalement à l’échelle les requêtes sur plusieurs machines à l’aide du partitionnement. Son moteur de requête parallélise les requêtes SQL entrantes sur ces serveurs afin de générer des réponses plus rapides sur les jeux de données volumineux. Elle s’adresse aux applications qui nécessitent des performances et une mise à l’échelle plus importantes que les autres options de déploiement, généralement des charges de travail qui approchent, ou dépassent déjà, 100 Go de données.

Cluster Hyperscale (Citus) permet ce qui suit :

- Mise à l’échelle horizontale sur plusieurs machines à l’aide du partitionnement
- Parallélisation des requêtes sur ces serveurs pour générer des réponses plus rapides sur les jeux de données volumineux
- Excellente prise en charge des applications multilocataires, analytique opérationnelle en temps réel et charges de travail transactionnelles à débit élevé

Les applications générées pour PostgreSQL peuvent exécuter des requêtes distribuées sur Hyperscale (Citus) avec des [bibliothèques de connexions](./concepts-connection-libraries.md) standard et des modifications minimes.

## <a name="next-steps"></a>Étapes suivantes

- Commencez par [créer votre premier](./quickstart-create-hyperscale-portal.md) groupe de serveurs Azure Database pour PostgreSQL - Hyperscale (Citus).
- Consultez la [page de tarification](https://azure.microsoft.com/pricing/details/postgresql/) pour des comparaisons de coûts et des calculatrices. Hyperscale (Citus) propose également des remises d’instances réservées prépayées. Pour plus d’informations, consultez [Tarif des instances réservées Hyperscale (Citus)](concepts-hyperscale-reserved-pricing.md).
- Déterminer la [taille initiale](howto-hyperscale-scale-initial.md) optimale pour votre groupe de serveurs
