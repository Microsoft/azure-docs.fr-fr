---
title: Niveau de service Critique pour l’entreprise
titleSuffix: Azure SQL Database & Azure SQL Managed Instance
description: Découvrez le niveau de service Critique pour l'entreprise d'Azure SQL Database et Azure SQL Managed Instance.
services: sql-database
ms.service: sql-db-mi
ms.subservice: features
ms.custom: sqldbrb=2
ms.devlang: ''
ms.topic: conceptual
author: jovanpop-msft
ms.author: jovanpop
ms.reviewer: sstein
ms.date: 12/04/2018
ms.openlocfilehash: 830ecc44d0def13e51cb06704bef429bb8860cd6
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92780221"
---
# <a name="business-critical-tier---azure-sql-database-and-azure-sql-managed-instance"></a>Niveau Critique pour l’entreprise - Azure SQL Database et Azure SQL Managed Instance 
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

> [!NOTE]
> Le niveau Critique pour l'entreprise s'appelle Premium dans le modèle d'achat DTU. Pour obtenir une comparaison du modèle d’achat vCore avec le modèle d’achat DTU, consultez [Ressources et modèles d’achat Azure SQL Database](purchasing-models.md).

Azure SQL Database et Azure SQL Managed Instance sont tous deux basés sur une architecture de moteur de base de données SQL Server qui est adaptée à l'environnement cloud afin de garantir une disponibilité de 99,99 % même en cas de défaillance de l'infrastructure. Trois modèles d'architecture sont utilisés :
- Usage général/Standard 
- Critique pour l’entreprise/Premium
- Hyperscale

Le modèle de niveau de service Premium/Critique pour l’entreprise est basé sur un cluster de processus de moteur de base de données. Ce modèle architectural s’appuie sur le fait qu’un quorum de nœuds de moteur de base de données est toujours disponible. Il n’a qu’un impact minime sur les performances de votre charge de travail, même durant les activités de maintenance. Le niveau de service Hyperscale est actuellement disponible uniquement pour Azure SQL Database (pas pour SQL Managed Instance). Il s’agit d’un niveau de performances de calcul et de stockage hautement scalable qui tire partie de l’architecture Azure pour effectuer un scale-out du stockage et des ressources de calcul d’une base de données dans Azure SQL Database bien au-delà des limites disponibles des niveaux Usage général et Critique pour l’entreprise.

Azure met à niveau et corrige le système d'exploitation, les pilotes et le moteur de base de données SQL Server sous-jacents de manière transparente avec un temps d'arrêt minimal pour les utilisateurs finaux. 

La disponibilité Premium, activée dans les niveaux de service Premium et Critique pour l'entreprise, est conçue pour les charges de travail intensives qui ne tolèrent aucun impact sur les performances résultant des opérations de maintenance en cours.

Dans le modèle Premium, le calcul et le stockage sont intégrés sur le nœud unique. La haute disponibilité offerte par ce modèle d'architecture est obtenue par la réplication du calcul (processus du moteur de base de données SQL Server) et du stockage (disque SSD attaché localement). Elle est déployée dans un cluster à quatre nœuds avec une technologie similaire aux [groupes de disponibilité AlwaysOn](/sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server) de SQL Server.

![Cluster de nœuds de moteur de base de données](./media/service-tier-business-critical/business-critical-service-tier.png)

Le processus du moteur de base de données SQL Server et les fichiers .mdf/.ldf sous-jacents sont placés sur le même nœud, et le stockage SSD attaché localement permet à votre charge de travail de bénéficier d’une faible latence. La haute disponibilité est implémentée à l'aide d'une technologie similaire aux [groupes de disponibilité AlwaysOn](/sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server) de SQL Server. Chaque base de données est un cluster de nœuds de base de données, avec une base de données primaire accessible aux charges de travail clientes, et trois processus secondaires contenant des copies des données. Le nœud principal envoie (push) constamment les changements aux nœuds secondaires afin de garantir la disponibilité des données sur les réplicas secondaires en cas de défaillance du nœud principal. Le basculement est géré par le moteur de base de données SQL Server : un réplica secondaire devient le nœud principal, et un réplica secondaire est créé pour garantir un nombre suffisant de nœuds dans le cluster. La charge de travail est automatiquement redirigée vers le nouveau nœud principal.

En outre, le cluster Critique pour l’entreprise dispose d’une fonctionnalité [Échelle horizontale en lecture ](read-scale-out.md) intégrée qui fournit un nœud intégré en lecture seule gratuit. Ce dernier peut être utilisé pour exécuter des requêtes en lecture seule (des rapports, par exemple) qui ne doivent pas affecter les performances de votre charge de travail principale.

## <a name="when-to-choose-this-service-tier"></a>Quand choisir ce niveau de service ?

Le niveau de service Critique pour l'entreprise a été conçu pour les applications qui exigent des réponses à faible latence du stockage SSD sous-jacent (de 1 à 2 ms en moyenne), une récupération rapide en cas de défaillance de l'infrastructure sous-jacente, ou qui ont besoin de décharger des rapports, des analyses et des requêtes en lecture seule sur le réplica secondaire gratuit de la base de données primaire.

Les principales raisons pour lesquelles vous devez choisir le niveau de service Critique pour l’entreprise plutôt que le niveau Usage général sont les suivantes :
-   **Exigences de faible latence pour les E/S**  : les charges de travail nécessitant une réponse rapide de la couche de stockage (une à deux millisecondes en moyenne) doivent utiliser le niveau Critique pour l’entreprise. 
-   **Communication fréquente entre l'application et la base de données** . Les applications qui ne peuvent pas tirer parti de la mise en cache de la couche application ou du [traitement par lot des demandes](../performance-improve-use-batching.md) et doivent envoyer de nombreuses requêtes SQL exigeant un traitement rapide peuvent bénéficier du niveau Critique pour l'entreprise.
-   **Grand nombre de mises à jour**  : des opérations d'insertion, de mise à jour et de suppression modifient les pages de données en mémoire (page de modifications) qui doivent être enregistrées dans des fichiers de données avec l'opération `CHECKPOINT`. Un incident potentiel du processus du moteur de base de données ou un basculement de la base de données avec un grand nombre de pages de modifications peut augmenter le temps de récupération au niveau Usage général. Utilisez le niveau Critique pour l’entreprise si vous avez une charge de travail qui entraîne de nombreuses modifications en mémoire. 
-   **Transactions de longue durée qui modifient les données** . Les transactions qui sont ouvertes pour une durée plus longue empêchent la troncation du fichier journal, ce qui peut augmenter la taille du journal et le nombre de [fichiers journaux virtuels](/sql/relational-databases/sql-server-transaction-log-architecture-and-management-guide#physical_arch). Un nombre élevé de fichiers journaux virtuels peut ralentir la récupération de la base de données après le basculement.
-   **Charge de travail avec requêtes de rapport et requêtes analytiques** qui peuvent être redirigées vers le réplica secondaire disponible en lecture seule gratuit.
- **Plus grande résilience et récupération plus rapide après les défaillances** . En cas de défaillance du système, la base de données de l'instance principale est désactivée et l'un des réplicas secondaires devient immédiatement la nouvelle base de données principale en lecture-écriture, prête à traiter les requêtes. Le moteur de base de données n'a pas besoin d'analyser et de réexécuter des transactions à partir du fichier journal, ni de charger toutes les données dans la mémoire tampon.
- **Protection avancée contre l’altération des données** . Le niveau Critique pour l’entreprise tire parti des réplicas de base de données exécutés en arrière-plan à des fins de continuité de l’activité, ce qui permet aussi au service de tirer parti de la réparation automatique des pages, technologie également utilisée pour la [mise en miroir et les groupes de disponibilité](/sql/sql-server/failover-clusters/automatic-page-repair-availability-groups-database-mirroring) des bases de données SQL Server. Dans le cas où un réplica ne peut pas lire une page en raison d’un problème d’intégrité des données, une nouvelle copie de la page sera extraite d’un autre réplica, remplaçant la page illisible sans perte de données ou temps d’arrêt du client. Cette fonctionnalité s’applique au niveau Usage général si la base de données a un réplica géo-secondaire.
- **Plus grande disponibilité**  : le niveau Critique pour l'entreprise de la configuration à plusieurs zones de disponibilité garantit une disponibilité de 99,995 %, contre 99,99 % pour le niveau Usage général.
- **Géo-récupération rapide**  : le niveau Critique pour l'entreprise configuré avec la géo-réplication a un objectif de point de récupération garanti de 5 secondes et un objectif de délai de récupération de 30 secondes pour 100 % des heures déployées.

## <a name="next-steps"></a>Étapes suivantes

- Recherchez les caractéristiques des ressources (nombre de cœurs, E/S, mémoire) du niveau Critique pour l’entreprise dans [SQL Managed Instance](../managed-instance/resource-limits.md#service-tier-characteristics), Base de données unique dans [Modèle vCore](resource-limits-vcore-single-databases.md#business-critical---provisioned-compute---gen4) ou [Modèle DTU](resource-limits-dtu-single-databases.md#premium-service-tier), ou Pool élastique dans [Modèle vCore](resource-limits-vcore-elastic-pools.md#business-critical---provisioned-compute---gen4) et [Modèle DTU](resource-limits-dtu-elastic-pools.md#premium-elastic-pool-limits).
- Découvrez les niveaux [Usage général](service-tier-general-purpose.md) et [Hyperscale](service-tier-hyperscale.md).
- Découvrez [Service Fabric](../../service-fabric/service-fabric-overview.md).
- Pour découvrir d’autres options de haute disponibilité et de récupération d’urgence, consultez [Continuité d’activité](business-continuity-high-availability-disaster-recover-hadr-overview.md).