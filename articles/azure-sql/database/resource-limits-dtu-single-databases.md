---
title: Limites de ressources DTU pour les bases de données uniques
description: Cette page décrit certaines limites de ressources DTU courantes pour des bases de données uniques dans Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: single-database
ms.custom: references_regions, seo-lt-2019, sqldbrb=1
ms.devlang: ''
ms.topic: reference
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 03/20/2019
ms.openlocfilehash: fd9a811fd1c19d115f3ff15194b7e632114140df
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92790251"
---
# <a name="resource-limits-for-single-databases-using-the-dtu-purchasing-model---azure-sql-database"></a>Limites de ressources pour des bases de données uniques suivant le modèle d’achat DTU - Azure SQL Database
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Cet article détaille les limites de ressources des bases de données uniques Azure SQL Database suivant le modèle d’achat DTU.

Pour connaître les limites de ressources des pools élastiques du modèle d’achat DTU, consultez [Limites des ressources DTU : pools élastiques](resource-limits-dtu-elastic-pools.md). Pour connaître les limites de ressources vCore, consultez [Limites de ressources vCore : bases de données uniques](resource-limits-vcore-single-databases.md) et [Limites de ressources vCore : pools élastiques](resource-limits-vcore-elastic-pools.md). Pour plus d’informations concernant les différents modèles d’achat, consultez l’article décrivant les [modèles d’achat et niveaux de service](purchasing-models.md).

## <a name="single-database-storage-sizes-and-compute-sizes"></a>Base de données unique : tailles de stockage et tailles de calcul

Les tableaux suivants indiquent les ressources disponibles pour une base de données unique à chaque niveau de service et taille de calcul. Vous pouvez définir le niveau de service, la taille de calcul et la quantité de stockage pour une base de données unique à l’aide du [Portail Azure](single-database-manage.md#the-azure-portal), de [Transact-SQL](single-database-manage.md#transact-sql-t-sql), de [PowerShell](single-database-manage.md#powershell), [d’Azure CLI](single-database-manage.md#the-azure-cli) ou de [l’API REST](single-database-manage.md#rest-api).

> [!IMPORTANT]
> Pour obtenir des instructions et informations sur la mise à l’échelle, consultez [Mettre à l’échelle des ressources de base de données unique dans Azure SQL Database](single-database-scale.md)

### <a name="basic-service-tier"></a>Niveau de service De base

| **Taille de calcul** | **De base** |
| :--- | --: |
| DTU max | 5 |
| Espace de stockage inclus (Go) | 2 |
| Stockage max. (Go) | 2 |
| Stockage In-Memory OLTP maximal (Go) |N/A |
| Nombre maximal d’ouvriers simultanés (demandes) | 30 |
| Nombre maximal de sessions simultanées | 300 |
|||

> [!IMPORTANT]
> Le niveau de service De base propose moins d’un vCore (processeur).  Pour les charges de travail gourmandes en ressources processeur, un niveau de service S3 ou supérieur est recommandé.
>
>En ce qui concerne le stockage des données, le niveau de service De base est placé dans des objets blob de pages Standard. Les objets blob de pages Standard utilisent un support de stockage basé sur un lecteur de disque dur (HDD) et sont mieux adaptés aux charges de travail de développement, de test et d’accès peu fréquent, qui sont moins sensibles à la variabilité du niveau de performance.
>

### <a name="standard-service-tier"></a>Niveau de service Standard

| **Taille de calcul** | **S0** | **S1** | **S2** | **S3** |
| :--- |---:| ---:|---:|---:|
| DTU max | 10 | 20 | 50 | 100 |
| Espace de stockage inclus (Go) <sup>1</sup> | 250 | 250 | 250 | 250 |
| Stockage max. (Go) | 250 | 250 | 250 | 1 024 |
| Stockage In-Memory OLTP maximal (Go) | N/A | N/A | N/A | N/A |
| Nombre maximal d’ouvriers simultanés (demandes)| 60 | 90 | 120 | 200 |
| Nombre maximal de sessions simultanées |600 | 900 | 1200 | 2 400 |
||||||

<sup>1</sup> Consultez [Options tarifaires SQL Database](https://azure.microsoft.com/pricing/details/sql-database/single/) pour obtenir des détails sur le coût additionnel engendré par le provisionnement d’espace de stockage supplémentaire.

> [!IMPORTANT]
> Les niveaux Standard S0, S1 et S2 proposent moins d’un vCore (processeur).  Pour les charges de travail gourmandes en ressources processeur, un niveau de service S3 ou supérieur est recommandé.
>
>En ce qui concerne le stockage des données, les niveaux de service Standard S0 et S1 sont placés dans des objets blob de pages Standard. Les objets blob de pages Standard utilisent un support de stockage basé sur un lecteur de disque dur (HDD) et sont mieux adaptés aux charges de travail de développement, de test et d’accès peu fréquent, qui sont moins sensibles à la variabilité du niveau de performance.
>

### <a name="standard-service-tier-continued"></a>Niveau de service Standard (suite)

| **Taille de calcul** | **S4** | **S6** | **S7** | **S9** | **S12** |
| :--- |---:| ---:|---:|---:|---:|
| DTU max | 200 | 400 | 800 | 1 600 | 3000 |
| Espace de stockage inclus (Go) <sup>1</sup> | 250 | 250 | 250 | 250 | 250 |
| Stockage max. (Go) | 1 024 | 1 024 | 1 024 | 1 024 | 1 024 |
| Stockage In-Memory OLTP maximal (Go) | N/A | N/A | N/A | N/A |N/A |
| Nombre maximal d’ouvriers simultanés (demandes)| 400 | 800 | 1 600 | 3200 |6000 |
| Nombre maximal de sessions simultanées |4 800 | 9 600 | 19 200 | 30000 |30000 |
|||||||

<sup>1</sup> Consultez [Options tarifaires SQL Database](https://azure.microsoft.com/pricing/details/sql-database/single/) pour obtenir des détails sur le coût additionnel engendré par le provisionnement d’espace de stockage supplémentaire.

### <a name="premium-service-tier"></a>Niveau de service Premium

| **Taille de calcul** | **P1** | **P2** | **P4** | **P6** | **P11** | **P15** |
| :--- |---:|---:|---:|---:|---:|---:|
| DTU max | 125 | 250 | 500 | 1 000 | 1750 | 4000 |
| Espace de stockage inclus (Go) <sup>1</sup> | 500 | 500 | 500 | 500 | 4 096 <sup>2</sup> | 4 096 <sup>2</sup> |
| Stockage max. (Go) | 1 024 | 1 024 | 1 024 | 1 024 | 4 096 <sup>2</sup> | 4 096 <sup>2</sup> |
| Stockage In-Memory OLTP maximal (Go) | 1 | 2 | 4 | 8 | 14 | 32 |
| Nombre maximal d’ouvriers simultanés (demandes)| 200 | 400 | 800 | 1 600 | 2 800 | 6 400 |
| Nombre maximal de sessions simultanées | 30000 | 30000 | 30000 | 30000 | 30000 | 30000 |
|||||||

<sup>1</sup> Consultez [Options tarifaires SQL Database](https://azure.microsoft.com/pricing/details/sql-database/single/) pour obtenir des détails sur le coût additionnel engendré par le provisionnement d’espace de stockage supplémentaire.

<sup>2</sup> De 1 024 Go à 4 096 Go par incréments de 256 Go.

> [!IMPORTANT]
> Un espace de stockage supérieur à 1 To au niveau Premium est actuellement disponible dans les toutes régions sauf les suivantes : Chine Est, Chine Nord, Allemagne Centre et Allemagne Nord-Est. Dans ces régions, l’espace de stockage maximal au niveau Premium est limité à 1 To.  Pour plus d’informations, voir les [limitations actuelles P11-P15](single-database-scale.md#p11-and-p15-constraints-when-max-size-greater-than-1-tb).
> [!NOTE]
> Pour les limites `tempdb`, consultez [Limites tempdb](/sql/relational-databases/databases/tempdb-database?view=sql-server-2017#tempdb-database-in-sql-database).

## <a name="next-steps"></a>Étapes suivantes

- Pour connaître les limites de ressources vCore d’une base de données unique, consultez l’article consacré aux [limites de ressources pour les bases de données uniques suivant le modèle d’achat vCore](resource-limits-vcore-single-databases.md)
- Pour connaître les limites de ressources vCore des pools élastiques, consultez l’article consacré aux [limites de ressources pour les pools élastiques suivant le modèle d’achat vCore](resource-limits-vcore-elastic-pools.md)
- Pour connaître les limites de ressources DTU des pools élastiques, consultez l’article consacré aux [limites de ressources pour les pools élastiques suivant le modèle d’achat DTU](resource-limits-dtu-elastic-pools.md)
- Pour connaître les limites de ressources des instances gérées dans Azure SQL Managed Instance, consultez [Limites de ressources de SQL Managed Instance](../managed-instance/resource-limits.md).
- Pour plus d’informations sur les limites générales d’Azure, consultez [Abonnement Azure et limites, quotas et contraintes du service](../../azure-resource-manager/management/azure-subscription-service-limits.md).
- Pour plus d'informations sur les limites de ressources sur un serveur SQL logique et au niveau de l’abonnement, consultez la [vue d'ensemble des limites de ressources sur un serveur SQL logique](resource-limits-logical-server.md).