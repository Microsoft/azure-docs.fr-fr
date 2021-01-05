---
title: Niveaux de service Usage général et Critique pour l’entreprise
titleSuffix: Azure SQL Database & SQL Managed Instance
description: L’article aborde les niveaux de service Usage général et Critique pour l’entreprise dans le modèle d’achat vCore utilisé par Azure SQL Database et Azure SQL Managed Instance.
services: sql-database
ms.service: sql-db-mi
ms.subservice: features
ms.custom: sqldbrb=2
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: sashan, moslake
ms.date: 12/14/2020
ms.openlocfilehash: 9ee7440b10bc348d3ba87a4779208791a7b0e9ac
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97512026"
---
# <a name="azure-sql-database-and-azure-sql-managed-instance-service-tiers"></a>Niveaux de service Azure SQL Database et Azure SQL Managed Instance
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Azure SQL Database et Azure SQL Managed Instance sont basés sur une architecture de moteur de base de données SQL Server qui est ajustée pour l’environnement cloud afin de garantir une disponibilité de 99,99 % même en cas de défaillance d’infrastructure. Deux niveaux de service sont utilisés par Azure SQL Database et Azure SQL Managed Instance, chacun doté d’un modèle architectural distinct. Ces niveaux de service sont les suivants :

- [Usage général](service-tier-general-purpose.md), conçu pour les charges de travail axées sur le budget.
- [Critique pour l’entreprise](service-tier-business-critical.md), conçu pour les charges de travail à faible latence avec une haute résilience aux défaillances et des basculements rapides.

Azure SQL Database a un niveau de service supplémentaire : 

- [Hyperscale](service-tier-hyperscale.md), conçu pour la plupart des charges de travail métier, offrant un stockage hautement scalable, un scale-out en lecture et des fonctionnalités de restauration rapide de base de données.

Cet article présente les différences entre les niveaux de service, les considérations relatives au stockage et à la sauvegarde pour les niveaux de service Usage général et Critique pour l’entreprise dans le modèle d’achat vCore.

## <a name="service-tier-comparison"></a>Comparaison des niveaux de service

Le tableau suivant décrit les principales différences entre les niveaux de service pour la dernière génération (Gen5). Notez que les caractéristiques des niveaux de service peuvent être différentes dans SQL Database et SQL Managed Instance.

|-| Type de ressource | Usage général |  Hyperscale | Critique pour l’entreprise |
|:---:|:---:|:---:|:---:|:---:|
| **Idéal pour** | |  Offre des options de calcul et de stockage équilibrées et économiques. | La plupart des charges de travail d’entreprise. Mise à l’échelle automatique de la taille de stockage jusqu’à 100 To, mise à l’échelle verticale et horizontale du calcul, restauration rapide de la base de données. | Applications OLTP avec des débits de transactions élevés et une faible latence des E/S. Offre une meilleure résilience aux défaillances et des basculements rapides à l’aide de plusieurs réplicas mis à jour de façon synchrone.|
|  **Disponible dans le type de ressource :** ||SQL Database / SQL Managed Instance | Base de données Azure SQL unique | SQL Database / SQL Managed Instance |
| **Taille de calcul**| SQL Database | 1 à 80 cœurs virtuels | 1 à 80 vCores | 1 à 80 cœurs virtuels |
| | Instance managée SQL | 4, 8, 16, 24, 32, 40, 64, 80 vCores | N/A | 4, 8, 16, 24, 32, 40, 64, 80 vCores |
| | Pools SQL Managed Instance | 2, 4, 8, 16, 24, 32, 40, 64, 80 vCores | N/A | N/A |
| **Type de stockage** | Tous | Stockage distant Premium (par instance) | Stockage découplé avec cache disque SSD local (par instance) | Stockage SSD local ultra-rapide (par instance) |
| **Taille de la base de données** | SQL Database | 5 Go - 4 To | Jusqu’à 100 To | 5 Go - 4 To |
| | Instance managée SQL  | 32 Go - 8 To | N/A | 32 Go - 4 To |
| **Taille de stockage** | SQL Database | 5 Go - 4 To | Jusqu’à 100 To | 5 Go - 4 To |
| | Instance managée SQL  | 32 Go - 8 To | N/A | 32 Go - 4 To |
| **Taille de TempDB** | SQL Database | [32 Go par vCore](resource-limits-vcore-single-databases.md#general-purpose---provisioned-compute---gen4) | [32 Go par vCore](resource-limits-vcore-single-databases.md#hyperscale---provisioned-compute---gen5) | [32 Go par vCore](resource-limits-vcore-single-databases.md#business-critical---provisioned-compute---gen4) |
| | Instance managée SQL  | [24 Go par vCore](../managed-instance/resource-limits.md#service-tier-characteristics) | N/A | Jusqu’à 4 To - [limité par taille de stockage](../managed-instance/resource-limits.md#service-tier-characteristics) |
| **Débit d’écriture des journaux** | SQL Database | [1,875 Mo/s par vCore (30 Mo/s max)](resource-limits-vcore-single-databases.md#general-purpose---provisioned-compute---gen4) | 100 Mo/s | [6 Mo/s par vCore (96 Mo/s max)](resource-limits-vcore-single-databases.md#business-critical---provisioned-compute---gen4) |
| | Instance managée SQL | [3 Mo/s par vCore (22 Mo/s max)](../managed-instance/resource-limits.md#service-tier-characteristics) | N/A | [4 Mo/s par vCore (48 Mo/s max)](../managed-instance/resource-limits.md#service-tier-characteristics) |
|**Disponibilité**|Tous| 99,99 % |  [99,95 % avec un réplica secondaire, 99,99 % avec d’autres réplicas](service-tier-hyperscale-frequently-asked-questions-faq.md#what-slas-are-provided-for-a-hyperscale-database) | 99,99 % <br/> [99,995 % avec base de données unique et redondante dans une zone](https://azure.microsoft.com/blog/understanding-and-leveraging-azure-sql-database-sla/) |
|**Sauvegardes**|Tous|RA-GRS, 7 à 35 jours (7 jours par défaut) La durée de rétention pour le niveau De base est de 7 jours maximum. | RA-GRS, 7 jours, récupération jusqu’à une date et heure (PITR) à durée constante | RA-GRS, 7 à 35 jours (7 jours par défaut) |
|**OLTP en mémoire** | | N/A | N/A | Disponible |
|**Réplicas en lecture seule**| | 0 intégré <br> 0 à 4 utilisant la [géoréplication](active-geo-replication-overview.md) | 0 à 4 intégrés | 1 intégré et inclus dans le prix <br> 0 à 4 utilisant la [géoréplication](active-geo-replication-overview.md) |
|**Tarification/facturation** | SQL Database | [vCore, stockage réservé et stockage de sauvegarde](https://azure.microsoft.com/pricing/details/sql-database/single/) sont facturés. <br/>Les IOPS ne sont pas facturées. | [vCore pour chaque réplica et le stockage utilisé](https://azure.microsoft.com/pricing/details/sql-database/single/) sont facturés. <br/>IOPS non encore facturées. | [vCore, stockage réservé et stockage de sauvegarde](https://azure.microsoft.com/pricing/details/sql-database/single/) sont facturés. <br/>Les IOPS ne sont pas facturées. |
|| Instance managée SQL | [vCore, stockage réservé et stockage de sauvegarde](https://azure.microsoft.com/pricing/details/sql-database/managed/) sont facturés. <br/>Les IOPS ne sont pas facturées.| N/A | [vCore, stockage réservé et stockage de sauvegarde](https://azure.microsoft.com/pricing/details/sql-database/managed/) sont facturés. <br/>Les IOPS ne sont pas facturées.| 
|**Modèles de remise**| | [Instances réservées](reserved-capacity-overview.md)<br/>[Azure Hybrid Benefit](../azure-hybrid-benefit.md) (non disponible avec les abonnements dev/test)<br/>Abonnements Dev/Test [Entreprise](https://azure.microsoft.com/offers/ms-azr-0148p/) et [Paiement à l’utilisation](https://azure.microsoft.com/offers/ms-azr-0023p/)| [Azure Hybrid Benefit](../azure-hybrid-benefit.md) (non disponible avec les abonnements dev/test)<br/>Abonnements Dev/Test [Entreprise](https://azure.microsoft.com/offers/ms-azr-0148p/) et [Paiement à l’utilisation](https://azure.microsoft.com/offers/ms-azr-0023p/)| [Instances réservées](reserved-capacity-overview.md)<br/>[Azure Hybrid Benefit](../azure-hybrid-benefit.md) (non disponible avec les abonnements dev/test)<br/>Abonnements Dev/Test [Entreprise](https://azure.microsoft.com/offers/ms-azr-0148p/) et [Paiement à l’utilisation](https://azure.microsoft.com/offers/ms-azr-0023p/)|

Pour plus d’informations, consultez les détails des différences entre les niveaux de service dans les pages [Azure SQL Database (vCore)](resource-limits-vcore-single-databases.md), [Base de données unique Azure SQL (DTU)](resource-limits-dtu-single-databases.md), [Base de données Azure SQL mise en pool (DTU)](resource-limits-dtu-single-databases.md) et [Azure SQL Managed Instance](../managed-instance/resource-limits.md).

> [!NOTE]
> Pour plus d’informations sur le niveau de service Hyperscale dans le modèle d’achat vCore, consultez [Niveau de service hyperscale](service-tier-hyperscale.md). Pour obtenir une comparaison du modèle d’achat vCore avec le modèle d’achat DTU, consultez [Ressources et modèles d’achat](purchasing-models.md).

## <a name="data-and-log-storage"></a>Stockage des données et des journaux

Les facteurs suivants affectent la quantité de stockage utilisée pour les fichiers de données et les fichiers journaux, et s’appliquent aux niveaux Usage général et Critique pour l’entreprise. Pour plus d’informations sur le stockage des données et des journaux dans Hyperscale, consultez [Niveau de service Hyperscale](service-tier-hyperscale.md).

- Le stockage alloué est utilisé par les fichiers de données (MDF) et les fichiers journaux (LDF).
- Chaque taille de calcul d’une base de données unique accepte une taille de base de données maximale, qui correspond, par défaut, à 32 Go.
- Lorsque vous configurez la taille de base de données unique nécessaire (la taille du fichier MDF), 30 pour cent de stockage supplémentaire sont automatiquement ajoutés pour prendre en charge les fichiers LDF.
- Vous pouvez choisir n’importe quelle taille de base de données unique située entre 10 Go et la taille maximale prise en charge.
  - Pour le stockage des niveaux de service Standard et Usage général, augmentez ou diminuez la taille par incrément de 10 Go.
  - Pour le stockage des niveaux de service premium et critique pour l’entreprise, augmentez ou diminuez la taille par incrément de 250 Go.
- Dans le niveau de service usage général, `tempdb` utilise un disque SSD attaché et le coût de ce stockage est inclus dans le prix du modèle vCore.
- Dans le niveau de service Critique pour l’entreprise, `tempdb` partage le disque SSD attaché avec les fichiers MDF et LDF, et le coût du stockage `tempdb` est inclus dans le prix du modèle vCore.
- La taille de stockage d’une instance managée SQL doit être spécifiée en multiples de 32 Go.


> [!IMPORTANT]
> Le stockage total alloué aux fichiers MDF et LDF vous est facturé.

Pour surveiller la taille totale actuelle de vos fichiers MDF et LDF, utilisez [sp_spaceused](/sql/relational-databases/system-stored-procedures/sp-spaceused-transact-sql). Pour surveiller la taille actuelle de chaque fichier MDF et LDF, utilisez [sys.database_files](/sql/relational-databases/system-catalog-views/sys-database-files-transact-sql).

> [!IMPORTANT]
> Dans certaines circonstances, vous devrez peut-être réduire une base de données pour récupérer l’espace inutilisé. Pour plus d’informations, consultez [Gérer l’espace des fichiers dans Azure SQL Database](file-space-manage.md).

## <a name="backups-and-storage"></a>Sauvegardes et stockage

Du stockage pour les sauvegardes de base de données est alloué afin de prendre en charge les fonctionnalités de récupération jusqu’à une date et heure (PITR) et de [conservation à long terme (LTR)](long-term-retention-overview.md) de SQL Database et SQL Managed Instance. Ce stockage est alloué séparément pour chaque base de données. De plus, les fonctionnalités Limite de restauration dans le temps et Conservation à long terme sont, elles aussi, facturées séparément.

- **Limite de restauration dans le temps** : les sauvegardes de bases de données individuelles sont automatiquement copiées vers le [stockage géographiquement redondant avec accès en lecture (RA-GRS)](../../storage/common/geo-redundant-design.md). La taille de stockage augmente dynamiquement avec chaque nouvelle création de sauvegarde. Le stockage est utilisé pour des sauvegardes complètes hebdomadaires, des sauvegardes différentielles quotidiennes et des sauvegardes de fichiers journaux qui sont copiés toutes les 5 minutes. La consommation du stockage dépend du taux de change de la base de données et de la période de conservation pour les sauvegardes. Vous pouvez configurer une période de conservation distincte pour chaque base de données, allant de 7 à 35 jours. Un volume de stockage minimal égal à 100 pour cent (1x) de la taille de la base de données est fourni sans frais supplémentaires. Pour la plupart des bases de données, cette quantité est suffisante pour stocker l’équivalent de 7 jours de sauvegardes.
- **Conservation à long terme** : Vous pouvez également configurer la conservation à long terme des sauvegardes complètes pendant 10 ans maximum (cette fonctionnalité est en [préversion publique limitée pour SQL Managed Instance](long-term-retention-overview.md#sql-managed-instance-support)). Si vous configurez une stratégie de conservation à long terme, ces sauvegardes sont stockées automatiquement dans le stockage RA-GRS. Toutefois, vous pouvez contrôler la fréquence à laquelle les sauvegardes sont copiées. Pour répondre aux différentes exigences de conformité, vous pouvez sélectionner plusieurs périodes de conservation pour les sauvegardes hebdomadaires, mensuelles ou annuelles. La configuration que vous choisissez détermine la quantité de stockage utilisée pour les sauvegardes de conservation à long terme. Pour estimer le coût du stockage de conservation à long terme, vous pouvez utiliser la calculatrice de prix LTR. Pour plus d’informations, consultez [Conservation à long terme de SQL Database](long-term-retention-overview.md).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les tailles de calcul et de stockage spécifiques disponibles dans les niveaux de service Usage général et Critique pour l’entreprise, consultez : 

- [Limites de ressources vCore pour Azure SQL Database](resource-limits-vcore-single-databases.md)
- [Limites de ressources vCore pour les bases de données mises en pool dans Azure SQL Database](resource-limits-vcore-elastic-pools.md).
- [Limites de ressources vCore pour Azure SQL Managed Instance](../managed-instance/resource-limits.md).
