---
title: Glossaire des termes
titleSuffix: Azure SQL
description: Glossaire des termes relatifs à l’utilisation d’Azure SQL Database, d’Azure SQL Managed Instance et de SQL sur une machine virtuelle Azure.
services: sql-database
ms.service: sql-database
ms.subservice: service
ms.custom: sqldbrb=4
ms.devlang: ''
ms.topic: reference
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 01/22/2020
ms.openlocfilehash: 18ff2c9690de1708cd8382d83a0c01662a8e6fb0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91619778"
---
# <a name="azure-sql-database-glossary-of-terms"></a>Lexique Azure SQL Database
[!INCLUDE[appliesto-asf](includes/appliesto-asf.md)]

## <a name="azure-sql-database"></a>Azure SQL Database

|Context|Terme|Informations complémentaires|
|:---|:---|:---|
|Service Azure|Azure SQL Database ou SQL Database|[Azure SQL Database](database/sql-database-paas-overview.md)|
|Modèle d'achat|Modèle d’achat DTU|[Modèle d’achat DTU](database/service-tiers-dtu.md)|
||Modèle d’achat vCore|[Modèle d’achat vCore](database/service-tiers-vcore.md)|
|Option de déploiement |Base de données unique|[Bases de données uniques](database/single-database-overview.md)|
||Pool élastique|[Pool élastique](database/elastic-pool-overview.md)|
|Niveau de service|De base, Standard, Premium, Usage général, Hyperscale, Critique pour l'entreprise|Pour les niveaux de service du modèle vCore, consultez [Niveaux de service SQL Database](database/service-tiers-vcore.md#service-tiers). Pour les niveaux de service du modèle DTU, consultez [Modèle DTU](database/service-tiers-dtu.md#compare-the-dtu-based-service-tiers).|
|Niveau de calcul|Calcul serverless|[Calcul serverless](database/service-tiers-vcore.md#compute-tiers)
||Calcul provisionné|[Calcul provisionné](database/service-tiers-vcore.md#compute-tiers)
|Génération de calcul|Gen5, Série M, Série Fsv2|[Génération du matériel](database/service-tiers-vcore.md#hardware-generations)
|Entité serveur| Serveur |[Serveurs SQL logiques](database/logical-servers.md)|
|Type de ressource|vCore|Cœur de processeur fourni à la ressource de calcul pour une base de données unique ou un pool élastique. |
||Taille de calcul et quantité de stockage|La taille de calcul correspond à la quantité maximale d’UC, de mémoire et d’autres ressources non liées au stockage disponibles pour une base de données unique ou un pool élastique.  La taille de stockage est la quantité maximale de stockage disponible pour une base de données unique ou un pool élastique. Pour plus d’informations sur les options de dimensionnement du modèle vCore, consultez [Bases de données uniques vCore](database/resource-limits-vcore-single-databases.md) et [Pools élastiques vCore](database/resource-limits-vcore-elastic-pools.md).  (../managed-instance/resource-limits.md).  Pour plus d'informations sur les options de dimensionnement du modèle DTU, consultez [Bases de données uniques DTU](database/resource-limits-dtu-single-databases.md) et [Pools élastiques DTU](database/resource-limits-dtu-elastic-pools.md).

## <a name="azure-sql-managed-instance"></a>Azure SQL Managed Instance

|Context|Terme|Informations complémentaires|
|:---|:---|:---|
|Service Azure|Azure SQL Managed Instance|[SQL Managed Instance](managed-instance/sql-managed-instance-paas-overview.md)|
|Modèle d'achat|Modèle d’achat vCore|[Modèle d’achat vCore](database/service-tiers-vcore.md)|
|Option de déploiement |Instance unique|[Instance unique](managed-instance/sql-managed-instance-paas-overview.md)|
||Pool d’instances (préversion)|[Pools d’instances](managed-instance/instance-pools-overview.md)|
|Niveau de service|Usage général, critique pour l’entreprise|[Niveaux de service de SQL Managed Instance](managed-instance/sql-managed-instance-paas-overview.md#service-tiers)|
|Niveau de calcul|Calcul provisionné|[Calcul provisionné](database/service-tiers-vcore.md#compute-tiers)|
|Génération de calcul|Gen5|[Génération du matériel](database/service-tiers-vcore.md#hardware-generations)
|Entité serveur|Instance ou instance gérée| s.o., car l’instance gérée SQL est en soi le serveur |
|Type de ressource|vCore|Cœur de processeur fourni à la ressource de calcul pour SQL Managed Instance.|
||Taille de calcul et quantité de stockage|La taille de calcul correspond à la quantité maximale d’UC, de mémoire et d’autres ressources non liées au stockage pour SQL Managed Instance.  La taille de stockage est la quantité maximale de stockage disponible pour une instance gérée SQL.  Pour les options de dimensionnement, voir [Instances gérées SQL](managed-instance/resource-limits.md). |

## <a name="sql-on-azure-vm"></a>SQL sur machine virtuelle Azure

need more stuff here
