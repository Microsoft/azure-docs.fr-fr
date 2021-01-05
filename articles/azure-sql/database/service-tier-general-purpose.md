---
title: Niveau de service Usage général
titleSuffix: Azure SQL Database & Azure SQL Managed Instance
description: Découvrez le niveau de service Usage général pour Azure SQL Database et Azure SQL Managed Instance.
services: sql-database
ms.service: sql-db-mi
ms.subservice: features
ms.custom: sqldbrb=2
ms.devlang: ''
ms.topic: conceptual
author: jovanpop-msft
ms.author: jovanpop
ms.reviewer: sstein
ms.date: 02/07/2019
ms.openlocfilehash: ee218253309995e721c97f4a7f7b4547b32f7c36
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85986639"
---
# <a name="general-purpose-service-tier---azure-sql-database-and-azure-sql-managed-instance"></a>Niveau de service Usage général - Azure SQL Database et Azure SQLAzure SQL Managed Instance
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

> [!NOTE]
> Le niveau de service Usage général du modèle d’achat vCore correspond au niveau de service Standard du modèle d’achat DTU. Pour obtenir une comparaison du modèle d’achat vCore avec le modèle d’achat DTU, consultez [Ressources et modèles d’achat](purchasing-models.md).

Azure SQL Database et Azure SQL Managed Instance sont basés sur une architecture de moteur de base de données SQL Server qui est adaptée à l’environnement cloud afin de garantir une disponibilité de 99,99 % même en cas de défaillance de l’infrastructure. 

Deux niveaux de service sont utilisés par Azure SQL Database et SQL Managed Instance : 

- Usage général
- Critique pour l’entreprise

Azure SQL Database a également un troisième niveau de service, qui n’est pas actuellement disponible pour Azure SQL Managed Instance :

- Hyperscale

Le modèle architectural du niveau de service Usage général est basé sur la séparation du calcul et du stockage. Ce modèle architectural s’appuie sur la haute disponibilité et la fiabilité du Stockage Blob Azure qui réplique les fichiers de base de données de façon transparente et qui garantit l’absence de perte de données en cas de panne de l’infrastructure sous-jacente.

L’illustration suivante montre quatre nœuds dans un modèle architectural Standard avec les couches de calcul et de stockage séparées.

![Séparation du calcul et du stockage](./media/service-tier-general-purpose/general-purpose-service-tier.png)

Le modèle architectural correspondant au niveau de service Usage général présente deux couches :

- Une couche de calcul sans état qui exécute le processus `sqlservr.exe` et contient uniquement des données transitoires et mises en cache (par exemple : cache du plan, pool de mémoires tampons, pool de stockage de colonnes). Ce nœud sans état est géré par Azure Service Fabric qui initialise le processus, contrôle l’intégrité du nœud et effectue le basculement vers un autre emplacement si nécessaire.
- Une couche de données avec état comprenant les fichiers de base de données (.mdf/.ldf) stockés dans le service Stockage Blob Azure. Le service Stockage Blob Azure garantit qu'aucun enregistrement placé dans un fichier de base de données ne subira de perte de données. Stockage Azure est doté de fonctionnalités intégrées de redondance et de disponibilité des données qui garantissent la préservation des enregistrements d’un fichier journal ou des pages d’un fichier de données, même en cas de plantage du processus.

Dès que le moteur de base de données ou le système d’exploitation est mis à niveau, qu’une partie de l’infrastructure sous-jacente est défaillante ou qu’un problème critique est détecté dans le processus `sqlservr.exe`, Azure Service Fabric déplace le processus sans état vers un autre nœud de calcul sans état. Afin de réduire le temps de basculement, un ensemble de nœuds de réserve se tient prêt à exécuter le nouveau service de calcul en cas de basculement du nœud principal. Les données dans la couche Stockage Azure ne sont pas affectées, et les fichiers de données/journaux sont attachés à des processus nouvellement initialisés. Ce processus garantit une disponibilité de 99,99 %, mais il peut impacter les performances des lourdes charges de travail en cours d’exécution, et ce en raison des délais de transition et du fait que le nouveau nœud démarre avec un cache à froid.

## <a name="when-to-choose-this-service-tier"></a>Quand choisir ce niveau de service ?

Le niveau de service Usage général est un niveau de service par défaut dans Azure SQL Database et Azure SQL Managed Instance conçu pour la plupart des charges de travail génériques. Si vous avez besoin d’un moteur de base de données complètement managé, avec un SLA de 99,99 % et une latence de stockage comprise entre cinq et dix millisecondes, adapté à SQL Server sur une machine virtuelle Azure dans la plupart des cas, choisissez le niveau Usage général.

## <a name="next-steps"></a>Étapes suivantes

- Recherchez les caractéristiques des ressources (nombre de cœurs, E/S, mémoire) du niveau Usage général/standard dans [SQL Managed Instance](../managed-instance/resource-limits.md#service-tier-characteristics), base de données unique dans [Modèle vCore](resource-limits-vcore-single-databases.md#general-purpose---provisioned-compute---gen4) ou [Modèle DTU](resource-limits-dtu-single-databases.md#single-database-storage-sizes-and-compute-sizes), ou pool élastique dans [Modèle vCore](resource-limits-vcore-elastic-pools.md#general-purpose---provisioned-compute---gen4) et [Modèle DTU](resource-limits-dtu-elastic-pools.md#standard-elastic-pool-limits).
- Découvrez les niveaux [Critique pour l’entreprise](service-tier-business-critical.md) et [Hyperscale](service-tier-hyperscale.md).
- Découvrez [Service Fabric](../../service-fabric/service-fabric-overview.md).
- Pour découvrir d’autres options de haute disponibilité et de récupération d’urgence, consultez [Continuité d’activité](business-continuity-high-availability-disaster-recover-hadr-overview.md).
