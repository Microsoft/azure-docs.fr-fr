---
title: Limites des ressources vCore des bases de données uniques
description: Cette page décrit certaines limites de ressources vCore courantes pour une base de données unique dans Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: single-database
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: reference
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 10/15/2020
ms.openlocfilehash: d6936d6c72855bc22e3fc9323028fd0b8cd0fb29
ms.sourcegitcommit: 93329b2fcdb9b4091dbd632ee031801f74beb05b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92097573"
---
# <a name="resource-limits-for-single-databases-using-the-vcore-purchasing-model"></a>Limites de ressources pour des bases de données uniques suivant le modèle d’achat vCore
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Cet article détaille les limites de ressources des bases de données uniques dans Azure SQL Database suivant le modèle d’achat vCore.

Pour connaître les limites du modèle d’achat DTU pour les bases de données uniques sur un serveur, consultez [Vue d’ensemble des limites de ressources sur un serveur](resource-limits-logical-server.md).

Vous pouvez définir le niveau de service, la taille de calcul (objectif de service) et la quantité de stockage pour une base de données unique à l’aide du [portail Azure](single-database-manage.md#the-azure-portal), de [Transact-SQL](single-database-manage.md#transact-sql-t-sql), de [PowerShell](single-database-manage.md#powershell), d’[Azure CLI](single-database-manage.md#the-azure-cli) ou de l’[API REST](single-database-manage.md#rest-api).

> [!IMPORTANT]
> Pour obtenir des instructions et informations sur la mise à l’échelle, consultez [Rendre scalable une seule base de données](single-database-scale.md).

## <a name="general-purpose---serverless-compute---gen5"></a>Usage universel - calcul serverless - Gen5

Le [niveau de calcul serverless](serverless-tier-overview.md) est actuellement disponible sur le matériel Gen5 uniquement.

### <a name="gen5-compute-generation-part-1"></a>Génération de calcul Gen5 (partie 1)

|Taille de calcul (objectif de service)|GP_S_Gen5_1|GP_S_Gen5_2|GP_S_Gen5_4|GP_S_Gen5_6|GP_S_Gen5_8|
|:--- | --: |--: |--: |--: |--: |
|Génération de calcul|Gen5|Gen5|Gen5|Gen5|Gen5|
|Nombre minimal-maximal de vCores|0,5-1|0,5-2|0,5-4|0,75-6|1,0-8|
|Mémoire minimale-maximale (Go)|2,02-3|2,05-6|2,10-12|2,25-18|3,00-24|
|Délai minimal-maximal de pause automatique (minutes)|60-10080|60-10080|60-10080|60-10080|60-10080|
|Prise en charge de ColumnStore|Oui|Oui|Oui|Oui|Oui|
|Stockage In-Memory OLTP (Go)|N/A|N/A|N/A|N/A|N/A|
|Taille maximale des données (Go)|512|1 024|1 024|1 024|1536|
|Taille maximale du journal (Go)|154|307|307|307|461|
|Taille maximale des données TempDB (Go)|32|64|128|192|256|
|Type de stockage|SSD distant|SSD distant|SSD distant|SSD distant|SSD distant|
|Latence d’E/S (approximative)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|
|Nombre maximal d’IOPS de données *|320|640|1 280|1920|2560|
|Taux de journalisation maximal (Mbits/s)|4.5|9|18|27|36|
|Nombre maximal d’ouvriers simultanés (demandes)|75|150|300|450|600|
|Nombre maximal de sessions simultanées|30,000|30,000|30,000|30,000|30,000|
|Nombre de réplicas|1|1|1|1|1|
|Plusieurs zones de disponibilités|N/A|N/A|N/A|N/A|N/A|
|Lecture du Scale-out|N/A|N/A|N/A|N/A|N/A|
|Stockage de sauvegarde inclus|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|

\* La valeur maximale pour les tailles d’E/S est comprise entre 8 Ko et 64 Ko. Les IOPS réelles dépendent de la charge de travail. Pour plus d’informations, consultez [Gouvernance des E/S de données](resource-limits-logical-server.md#resource-governance).

### <a name="gen5-compute-generation-part-2"></a>Génération de calcul Gen5 (partie 2)

|Taille de calcul (objectif de service)|GP_S_Gen5_10|GP_S_Gen5_12|GP_S_Gen5_14|GP_S_Gen5_16|
|:--- | --: |--: |--: |--: |
|Génération de calcul|Gen5|Gen5|Gen5|Gen5|
|Nombre minimal-maximal de vCores|1,25-10|1,50-12|1,75-14|2,00-16|
|Mémoire minimale-maximale (Go)|3,75-30|4,50-36|5,25-42|6,00-48|
|Délai minimal-maximal de pause automatique (minutes)|60-10080|60-10080|60-10080|60-10080|
|Prise en charge de ColumnStore|Oui|Oui|Oui|Oui|
|Stockage In-Memory OLTP (Go)|N/A|N/A|N/A|N/A|
|Taille maximale des données (Go)|1536|3 072|3 072|3 072|
|Taille maximale du journal (Go)|461|461|461|922|
|Taille maximale des données TempDB (Go)|320|384|448|512|
|Type de stockage|SSD distant|SSD distant|SSD distant|SSD distant|
|Latence d’E/S (approximative)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|
|Nombre maximal d’IOPS de données *|3200|3840|4480|5120|
|Taux de journalisation maximal (Mbits/s)|36|36|36|36|
|Nombre maximal d’ouvriers simultanés (demandes)|750|900|1050|1200|
|Nombre maximal de sessions simultanées|30,000|30,000|30,000|30,000|
|Nombre de réplicas|1|1|1|1|
|Plusieurs zones de disponibilités|N/A|N/A|N/A|N/A|
|Lecture du Scale-out|N/A|N/A|N/A|N/A|
|Stockage de sauvegarde inclus|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|

\* La valeur maximale pour les tailles d’E/S est comprise entre 8 Ko et 64 Ko. Les IOPS réelles dépendent de la charge de travail. Pour plus d’informations, consultez [Gouvernance des E/S de données](resource-limits-logical-server.md#resource-governance).

### <a name="gen5-compute-generation-part-3"></a>Génération de calcul Gen5 (partie 3)

|Taille de calcul (objectif de service)|GP_S_Gen5_18|GP_S_Gen5_20|GP_S_Gen5_24|GP_S_Gen5_32|GP_S_Gen5_40|
|:--- | --: |--: |--: |--: |--:|
|Génération de calcul|Gen5|Gen5|Gen5|Gen5|Gen5|
|Nombre minimal-maximal de vCores|2,25-18|2,5-20|3-24|4-32|5-40|
|Mémoire minimale-maximale (Go)|6,75-54|7,5-60|9-72|12-96|15-120|
|Délai minimal-maximal de pause automatique (minutes)|60-10080|60-10080|60-10080|60-10080|60-10080|
|Prise en charge de ColumnStore|Oui|Oui|Oui|Oui|Oui|
|Stockage In-Memory OLTP (Go)|N/A|N/A|N/A|N/A|N/A|
|Taille maximale des données (Go)|3 072|3 072|4096|4096|4096|
|Taille maximale du journal (Go)|922|922|1 024|1 024|1 024|
|Taille maximale des données TempDB (Go)|576|640|768|1 024|1 280|
|Type de stockage|SSD distant|SSD distant|SSD distant|SSD distant|SSD distant|
|Latence d’E/S (approximative)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|
|Nombre maximal d’IOPS de données *|5760|6 400|7680|10240|12800|
|Taux de journalisation maximal (Mbits/s)|36|36|36|36|36|
|Nombre maximal d’ouvriers simultanés (demandes)|1 350|1500|1800|2 400|3000|
|Nombre maximal de sessions simultanées|30,000|30,000|30,000|30,000|30,000|
|Nombre de réplicas|1|1|1|1|1|
|Plusieurs zones de disponibilités|N/A|N/A|N/A|N/A|N/A|
|Lecture du Scale-out|N/A|N/A|N/A|N/A|N/A|
|Stockage de sauvegarde inclus|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|

\* La valeur maximale pour les tailles d’E/S est comprise entre 8 Ko et 64 Ko. Les IOPS réelles dépendent de la charge de travail. Pour plus d’informations, consultez [Gouvernance des E/S de données](resource-limits-logical-server.md#resource-governance).


## <a name="hyperscale---provisioned-compute---gen4"></a>Hyperscale - calcul provisionné - Gen4

### <a name="gen4-compute-generation-part-1"></a>Génération de calcul Gen4 (partie 1)

|Taille de calcul (objectif de service)|HS_Gen4_1|HS_Gen4_2|HS_Gen4_3|HS_Gen4_4|HS_Gen4_5|HS_Gen4_6|
|:--- | --: |--: |--: |---: | --: |--: |
|Génération de calcul|Gen4|Gen4|Gen4|Gen4|Gen4|Gen4|
|vCores|1|2|3|4|5|6|
|Mémoire (Go)|7|14|21|28|35|42|
|Taille [RBPEX](service-tier-hyperscale.md#compute)|3X mémoire|3X mémoire|3X mémoire|3X mémoire|3X mémoire|3X mémoire|
|Prise en charge de ColumnStore|Oui|Oui|Oui|Oui|Oui|Oui|
|Stockage In-Memory OLTP (Go)|N/A|N/A|N/A|N/A|N/A|N/A|
|Taille maximale des données (To)|100 |100 |100 |100 |100 |100|
|Taille maximale du journal (To)|Illimité |Illimité |Illimité |Illimité |Illimité |Illimité |
|Taille maximale des données TempDB (Go)|32|64|96|128|160|192|
|Type de stockage| [Remarque 1](#notes) |[Remarque 1](#notes)|[Remarque 1](#notes) |[Remarque 1](#notes) |[Remarque 1](#notes) |[Remarque 1](#notes) |
|Nbre max. d’IOPS de disque SSD local*|4000 |8000 |12 000 |16000 |20000 |24 000 |
|Taux de journalisation maximal (Mbits/s)|100 |100 |100 |100 |100 |100 |
|Latence d’E/S (approximative)|[Remarque 2](#notes)|[Remarque 2](#notes)|[Remarque 2](#notes)|[Remarque 2](#notes)|[Remarque 2](#notes)|[Remarque 2](#notes)|
|Nombre maximal d’ouvriers simultanés (demandes)|200|400|600|800|1 000|1200|
|Nombre maximal de sessions simultanées|30,000|30,000|30,000|30,000|30,000|30,000|
|Réplicas secondaires|0-4|0-4|0-4|0-4|0-4|0-4|
|Plusieurs zones de disponibilités|N/A|N/A|N/A|N/A|N/A|N/A|
|Lecture du Scale-out|Oui|Oui|Oui|Oui|Oui|Oui|
|Conservation du stockage de sauvegarde|7 jours|7 jours|7 jours|7 jours|7 jours|7 jours|
|||

\* Outre les E/S de disque SSD local, les charges de travail utilisent les E/S de [serveur de pages](service-tier-hyperscale.md#page-server) distant. L’efficacité des IOPS dépend de la charge de travail. Pour plus d’informations, consultez [Gouvernance des E/S de données](resource-limits-logical-server.md#resource-governance) et [E/S de données dans les statistiques d’utilisation des ressources](hyperscale-performance-diagnostics.md#data-io-in-resource-utilization-statistics).

### <a name="gen4-compute-generation-part-2"></a>Génération de calcul Gen4 (partie 2)

|Taille de calcul (objectif de service)|HS_Gen4_7|HS_Gen4_8|HS_Gen4_9|HS_Gen4_10|HS_Gen4_16|HS_Gen4_24|
|:--- | ---: |--: |--: | --: |--: |--: |
|Génération de calcul|Gen4|Gen4|Gen4|Gen4|Gen4|Gen4|
|vCores|7|8|9|10|16|24|
|Mémoire (Go)|49|56|63|70|112|159,5|
|Taille [RBPEX](service-tier-hyperscale.md#compute)|3X mémoire|3X mémoire|3X mémoire|3X mémoire|3X mémoire|3X mémoire|
|Prise en charge de ColumnStore|Oui|Oui|Oui|Oui|Oui|Oui|
|Stockage In-Memory OLTP (Go)|N/A|N/A|N/A|N/A|N/A|N/A|
|Taille maximale des données (To)|100 |100 |100 |100 |100 |100 |
|Taille maximale du journal (To)|Illimité |Illimité |Illimité |Illimité |Illimité |Illimité |
|Taille maximale des données TempDB (Go)|224|256|288|320|512|768|
|Type de stockage| [Remarque 1](#notes) |[Remarque 1](#notes) |[Remarque 1](#notes) |[Remarque 1](#notes) |[Remarque 1](#notes) |[Remarque 1](#notes) |
|Nbre max. d’IOPS de disque SSD local*|28000 |32000 |36000 |40000 |64 000 |76 800 |
|Taux de journalisation maximal (Mbits/s)|100 |100 |100 |100 |100 |100 |
|Latence d’E/S (approximative)|[Remarque 2](#notes)|[Remarque 2](#notes)|[Remarque 2](#notes)|[Remarque 2](#notes)|[Remarque 2](#notes)|[Remarque 2](#notes)|
|Nombre maximal d’ouvriers simultanés (demandes)|1400|1 600|1800|2000|3200|4 800|
|Nombre maximal de sessions simultanées|30,000|30,000|30,000|30,000|30,000|30,000|
|Réplicas secondaires|0-4|0-4|0-4|0-4|0-4|0-4|
|Plusieurs zones de disponibilités|N/A|N/A|N/A|N/A|N/A|N/A|
|Lecture du Scale-out|Oui|Oui|Oui|Oui|Oui|Oui|
|Conservation du stockage de sauvegarde|7 jours|7 jours|7 jours|7 jours|7 jours|7 jours|
|||

\* Outre les E/S de disque SSD local, les charges de travail utilisent les E/S de [serveur de pages](service-tier-hyperscale.md#page-server) distant. L’efficacité des IOPS dépend de la charge de travail. Pour plus d’informations, consultez [Gouvernance des E/S de données](resource-limits-logical-server.md#resource-governance) et [E/S de données dans les statistiques d’utilisation des ressources](hyperscale-performance-diagnostics.md#data-io-in-resource-utilization-statistics).

## <a name="hyperscale---provisioned-compute---gen5"></a>Hyperscale - calcul provisionné - Gen5

### <a name="gen5-compute-generation-part-1"></a>Génération de calcul Gen5 (partie 1)

|Taille de calcul (objectif de service)|HS_Gen5_2|HS_Gen5_4|HS_Gen5_6|HS_Gen_8|HS_Gen5_10|HS_Gen5_12|HS_Gen5_14|
|:--- | --: |--: |--: |--: |---: | --: |--: |--: |
|Génération de calcul|Gen5|Gen5|Gen5|Gen5|Gen5|Gen5|Gen5|
|vCores|2|4|6|8|10|12|14|
|Mémoire (Go)|10,4|20,8|31,1|41,5|51,9|62,3|72,7|
|Taille [RBPEX](service-tier-hyperscale.md#compute)|3X mémoire|3X mémoire|3X mémoire|3X mémoire|3X mémoire|3X mémoire|3X mémoire|
|Prise en charge de ColumnStore|Oui|Oui|Oui|Oui|Oui|Oui|Oui|
|Stockage In-Memory OLTP (Go)|N/A|N/A|N/A|N/A|N/A|N/A|N/A|
|Taille maximale des données (To)|100 |100 |100 |100 |100 |100 |100|
|Taille maximale du journal (To)|Illimité |Illimité |Illimité |Illimité |Illimité |Illimité |Illimité |
|Taille maximale des données TempDB (Go)|64|128|192|256|320|384|448|
|Type de stockage| [Remarque 1](#notes) |[Remarque 1](#notes)|[Remarque 1](#notes) |[Remarque 1](#notes) |[Remarque 1](#notes) |[Remarque 1](#notes) |[Remarque 1](#notes) |
|Nbre max. d’IOPS de disque SSD local*|8000 |16000 |24 000 |32000 |40000 |48 000 |56 000 |
|Taux de journalisation maximal (Mbits/s)|100 |100 |100 |100 |100 |100 |100 |
|Latence d’E/S (approximative)|[Remarque 2](#notes)|[Remarque 2](#notes)|[Remarque 2](#notes)|[Remarque 2](#notes)|[Remarque 2](#notes)|[Remarque 2](#notes)|[Remarque 2](#notes)|
|Nombre maximal d’ouvriers simultanés (demandes)|200|400|600|800|1 000|1200|1400|
|Nombre maximal de sessions simultanées|30,000|30,000|30,000|30,000|30,000|30,000|30,000|
|Réplicas secondaires|0-4|0-4|0-4|0-4|0-4|0-4|0-4|
|Plusieurs zones de disponibilités|N/A|N/A|N/A|N/A|N/A|N/A|N/A|
|Lecture du Scale-out|Oui|Oui|Oui|Oui|Oui|Oui|Oui|
|Conservation du stockage de sauvegarde|7 jours|7 jours|7 jours|7 jours|7 jours|7 jours|7 jours|
|||

\* Outre les E/S de disque SSD local, les charges de travail utilisent les E/S de [serveur de pages](service-tier-hyperscale.md#page-server) distant. L’efficacité des IOPS dépend de la charge de travail. Pour plus d’informations, consultez [Gouvernance des E/S de données](resource-limits-logical-server.md#resource-governance) et [E/S de données dans les statistiques d’utilisation des ressources](hyperscale-performance-diagnostics.md#data-io-in-resource-utilization-statistics).

### <a name="gen5-compute-generation-part-2"></a>Génération de calcul Gen5 (partie 2)

|Taille de calcul (objectif de service)|HS_Gen5_16|HS_Gen5_18|HS_Gen5_20|HS_Gen_24|HS_Gen5_32|HS_Gen5_40|HS_Gen5_80|
|:--- | --: |--: |--: |--: |---: |--: |--: |
|Génération de calcul|Gen5|Gen5|Gen5|Gen5|Gen5|Gen5|Gen5|
|vCores|16|18|20|24|32|40|80|
|Mémoire (Go)|83|93,4|103,8|124,6|166,1|207,6|415,2|
|Taille [RBPEX](service-tier-hyperscale.md#compute)|3X mémoire|3X mémoire|3X mémoire|3X mémoire|3X mémoire|3X mémoire|3X mémoire|
|Prise en charge de ColumnStore|Oui|Oui|Oui|Oui|Oui|Oui|Oui|
|Stockage In-Memory OLTP (Go)|N/A|N/A|N/A|N/A|N/A|N/A|N/A|
|Taille maximale des données (To)|100 |100 |100 |100 |100 |100 |100 |
|Taille maximale du journal (To)|Illimité |Illimité |Illimité |Illimité |Illimité |Illimité |Illimité |
|Taille maximale des données TempDB (Go)|512|576|640|768|1 024|1 280|2560|
|Type de stockage| [Remarque 1](#notes) |[Remarque 1](#notes)|[Remarque 1](#notes)|[Remarque 1](#notes) |[Remarque 1](#notes) |[Remarque 1](#notes) |[Remarque 1](#notes) |
|Nbre max. d’IOPS de disque SSD local*|64 000 |72 000 |80000 |96 000 |160 000 |192 000 |204800 |
|Taux de journalisation maximal (Mbits/s)|100 |100 |100 |100 |100 |100 |100 |
|Latence d’E/S (approximative)|[Remarque 2](#notes)|[Remarque 2](#notes)|[Remarque 2](#notes)|[Remarque 2](#notes)|[Remarque 2](#notes)|[Remarque 2](#notes)|[Remarque 2](#notes)|
|Nombre maximal d’ouvriers simultanés (demandes)|1 600|1800|2000|2 400|3200|4000|8000|
|Nombre maximal de sessions simultanées|30,000|30,000|30,000|30,000|30,000|30,000|30,000|
|Réplicas secondaires|0-4|0-4|0-4|0-4|0-4|0-4|0-4|
|Plusieurs zones de disponibilités|N/A|N/A|N/A|N/A|N/A|N/A|N/A|
|Lecture du Scale-out|Oui|Oui|Oui|Oui|Oui|Oui|Oui|
|Conservation du stockage de sauvegarde|7 jours|7 jours|7 jours|7 jours|7 jours|7 jours|7 jours|
|||

\* Outre les E/S de disque SSD local, les charges de travail utilisent les E/S de [serveur de pages](service-tier-hyperscale.md#page-server) distant. L’efficacité des IOPS dépend de la charge de travail. Pour plus d’informations, consultez [Gouvernance des E/S de données](resource-limits-logical-server.md#resource-governance) et [E/S de données dans les statistiques d’utilisation des ressources](hyperscale-performance-diagnostics.md#data-io-in-resource-utilization-statistics).

#### <a name="notes"></a>Notes

**Remarque 1**  : Hyperscale est une architecture à plusieurs niveaux avec des composants de calcul et de stockage distincts : [Architecture de niveau de service Hyperscale](service-tier-hyperscale.md#distributed-functions-architecture)

**Remarque 2**  : La latence est de 1 ms ou 2 ms pour les données du disque SSD de réplica
de calcul local, qui met en cache la plupart des pages de données utilisées. Elle est plus élevée pour les données récupérées à partir des serveurs de pages.

## <a name="general-purpose---provisioned-compute---gen4"></a>Usage général - calcul provisionné - Gen4

> [!IMPORTANT]
> Les nouvelles bases de données Gen4 ne sont plus prises en charge dans les régions Australie Est et Brésil Sud.

### <a name="gen4-compute-generation-part-1"></a>Génération de calcul Gen4 (partie 1)

|Taille de calcul (objectif de service)|GP_Gen4_1|GP_Gen4_2|GP_Gen4_3|GP_Gen4_4|GP_Gen4_5|GP_Gen4_6
|:--- | --: |--: |--: |--: |--: |--: |
|Génération de calcul|Gen4|Gen4|Gen4|Gen4|Gen4|Gen4|
|vCores|1|2|3|4|5|6|
|Mémoire (Go)|7|14|21|28|35|42|
|Prise en charge de ColumnStore|Oui|Oui|Oui|Oui|Oui|Oui|
|Stockage In-Memory OLTP (Go)|N/A|N/A|N/A|N/A|N/A|N/A|
|Taille maximale des données (Go)|1 024|1 024|1536|1536|1536|3 072|
|Taille maximale du journal (Go)|307|307|461|461|461|922|
|Taille maximale des données TempDB (Go)|32|64|96|128|160|192|
|Type de stockage|SSD distant|SSD distant|SSD distant|SSD distant|SSD distant|SSD distant|
|Latence d’E/S (approximative)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|
|Nombre maximal d’IOPS de données *|320|640|960|1 280|1 600|1920|
|Taux de journalisation maximal (Mbits/s)|4.5|9|13,5|18|22,5|27|
|Nombre maximal d’ouvriers simultanés (demandes)|200|400|600|800|1 000|1200|
|Nombre maximal de sessions simultanées|30,000|30,000|30,000|30,000|30,000|30,000|
|Nombre de réplicas|1|1|1|1|1|1|
|Plusieurs zones de disponibilités|N/A|N/A|N/A|N/A|N/A|N/A|
|Lecture du Scale-out|N/A|N/A|N/A|N/A|N/A|N/A|
|Stockage de sauvegarde inclus|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|

\* La valeur maximale pour les tailles d’E/S est comprise entre 8 Ko et 64 Ko. Les IOPS réelles dépendent de la charge de travail. Pour plus d’informations, consultez [Gouvernance des E/S de données](resource-limits-logical-server.md#resource-governance).

### <a name="gen4-compute-generation-part-2"></a>Génération de calcul Gen4 (partie 2)

|Taille de calcul (objectif de service)|GP_Gen4_7|GP_Gen4_8|GP_Gen4_9|GP_Gen4_10|GP_Gen4_16|GP_Gen4_24
|:--- | --: |--: |--: |--: |--: |--: |
|Génération de calcul|Gen4|Gen4|Gen4|Gen4|Gen4|Gen4|
|vCores|7|8|9|10|16|24|
|Mémoire (Go)|49|56|63|70|112|159,5|
|Prise en charge de ColumnStore|Oui|Oui|Oui|Oui|Oui|Oui|
|Stockage In-Memory OLTP (Go)|N/A|N/A|N/A|N/A|N/A|N/A|
|Taille maximale des données (Go)|3 072|3 072|3 072|3 072|4096|4096|
|Taille maximale du journal (Go)|922|922|922|922|1229|1229|
|Taille maximale des données TempDB (Go)|224|256|288|320|512|768|
|Type de stockage|SSD distant|SSD distant|SSD distant|SSD distant|SSD distant|SSD distant|
|Latence d’E/S (approximative)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)
|Nombre maximal d’IOPS de données *|2240|2560|2880|3200|5120|7680|
|Taux de journalisation maximal (Mbits/s)|31,5|36|36|36|36|36|
|Nombre maximal d’ouvriers simultanés (demandes)|1400|1 600|1800|2000|3200|4 800|
|Nombre maximal de sessions simultanées|30,000|30,000|30,000|30,000|30,000|30,000|
|Nombre de réplicas|1|1|1|1|1|1|
|Plusieurs zones de disponibilités|N/A|N/A|N/A|N/A|N/A|N/A|
|Lecture du Scale-out|N/A|N/A|N/A|N/A|N/A|N/A|
|Stockage de sauvegarde inclus|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|

\* La valeur maximale pour les tailles d’E/S est comprise entre 8 Ko et 64 Ko. Les IOPS réelles dépendent de la charge de travail. Pour plus d’informations, consultez [Gouvernance des E/S de données](resource-limits-logical-server.md#resource-governance).

## <a name="general-purpose---provisioned-compute---gen5"></a>Usage général - calcul provisionné - Gen5

### <a name="gen5-compute-generation-part-1"></a>Génération de calcul Gen5 (partie 1)

|Taille de calcul (objectif de service)|GP_Gen5_2|GP_Gen5_4|GP_Gen5_6|GP_Gen5_8|GP_Gen5_10|GP_Gen5_12|GP_Gen5_14|
|:--- | --: |--: |--: |--: |---: | --: |--: |
|Génération de calcul|Gen5|Gen5|Gen5|Gen5|Gen5|Gen5|Gen5|
|vCores|2|4|6|8|10|12|14|
|Mémoire (Go)|10,4|20,8|31,1|41,5|51,9|62,3|72,7|
|Prise en charge de ColumnStore|Oui|Oui|Oui|Oui|Oui|Oui|Oui|
|Stockage In-Memory OLTP (Go)|N/A|N/A|N/A|N/A|N/A|N/A|N/A|
|Taille maximale des données (Go)|1 024|1 024|1536|1536|1536|3 072|3 072|
|Taille maximale du journal (Go)|307|307|461|461|461|922|922|
|Taille maximale des données TempDB (Go)|64|128|192|256|320|384|384|
|Type de stockage|SSD distant|SSD distant|SSD distant|SSD distant|SSD distant|SSD distant|SSD distant|
|Latence d’E/S (approximative)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|
|Nombre maximal d’IOPS de données *|640|1 280|1920|2560|3200|3840|4480|
|Taux de journalisation maximal (Mbits/s)|9|18|27|36|36|36|36|
|Nombre maximal d’ouvriers simultanés (demandes)|200|400|600|800|1 000|1200|1400|
|Nombre maximal de sessions simultanées|30,000|30,000|30,000|30,000|30,000|30,000|30,000|
|Nombre de réplicas|1|1|1|1|1|1|1|
|Plusieurs zones de disponibilités|N/A|N/A|N/A|N/A|N/A|N/A|N/A|
|Lecture du Scale-out|N/A|N/A|N/A|N/A|N/A|N/A|N/A|
|Stockage de sauvegarde inclus|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|

\* La valeur maximale pour les tailles d’E/S est comprise entre 8 Ko et 64 Ko. Les IOPS réelles dépendent de la charge de travail. Pour plus d’informations, consultez [Gouvernance des E/S de données](resource-limits-logical-server.md#resource-governance).

### <a name="gen5-compute-generation-part-2"></a>Génération de calcul Gen5 (partie 2)

|Taille de calcul (objectif de service)|GP_Gen5_16|GP_Gen5_18|GP_Gen5_20|GP_Gen5_24|GP_Gen5_32|GP_Gen5_40|GP_Gen5_80|
|:--- | --: |--: |--: |--: |---: | --: |--: |
|Génération de calcul|Gen5|Gen5|Gen5|Gen5|Gen5|Gen5|Gen5|
|vCores|16|18|20|24|32|40|80|
|Mémoire (Go)|83|93,4|103,8|124,6|166,1|207,6|415,2|
|Prise en charge de ColumnStore|Oui|Oui|Oui|Oui|Oui|Oui|Oui|
|Stockage In-Memory OLTP (Go)|N/A|N/A|N/A|N/A|N/A|N/A|N/A|
|Taille maximale des données (Go)|3 072|3 072|3 072|4096|4096|4096|4096|
|Taille maximale du journal (Go)|922|922|922|1 024|1 024|1 024|1 024|
|Taille maximale des données TempDB (Go)|512|576|640|768|1 024|1 280|2560|
|Type de stockage|SSD distant|SSD distant|SSD distant|SSD distant|SSD distant|SSD distant|SSD distant|
|Latence d’E/S (approximative)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|
|Nombre maximal d’IOPS de données *|5120|5760|6 400|7680|10240|12800|12800|
|Taux de journalisation maximal (Mbits/s)|36|36|36|36|36|36|36|
|Nombre maximal d’ouvriers simultanés (demandes)|1 600|1800|2000|2 400|3200|4000|8000|
|Nombre maximal de sessions simultanées|30,000|30,000|30,000|30,000|30,000|30,000|30,000|
|Nombre de réplicas|1|1|1|1|1|1|1|
|Plusieurs zones de disponibilités|N/A|N/A|N/A|N/A|N/A|N/A|N/A|
|Lecture du Scale-out|N/A|N/A|N/A|N/A|N/A|N/A|N/A|
|Stockage de sauvegarde inclus|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|

\* La valeur maximale pour les tailles d’E/S est comprise entre 8 Ko et 64 Ko. Les IOPS réelles dépendent de la charge de travail. Pour plus d’informations, consultez [Gouvernance des E/S de données](resource-limits-logical-server.md#resource-governance).

## <a name="general-purpose---provisioned-compute---fsv2-series"></a>Usage général - calcul provisionné - série Fsv2

### <a name="fsv2-series-compute-generation-part-1"></a>Génération de calcul de série Fsv2 (partie 1)

|Taille de calcul (objectif de service)|GP_Fsv2_8|GP_Fsv2_10|GP_Fsv2_12|GP_Fsv2_14| GP_Fsv2_16|
|:---| ---:|---:|---:|---:|---:|
|Génération de calcul|Série Fsv2|Série Fsv2|Série Fsv2|Série Fsv2|Série Fsv2|
|vCores|8|10|12|14|16|
|Mémoire (Go)|15,1|18,9|22,7|26,5|30,2|
|Prise en charge de ColumnStore|Oui|Oui|Oui|Oui|Oui|
|Stockage In-Memory OLTP (Go)|N/A|N/A|N/A|N/A|N/A|
|Taille maximale des données (Go)|1 024|1 024|1 024|1 024|1536|
|Taille maximale du journal (Go)|336|336|336|336|512|
|Taille maximale des données TempDB (Go)|333|333|333|333|333|
|Type de stockage|SSD distant|SSD distant|SSD distant|SSD distant|SSD distant|
|Latence d’E/S (approximative)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|
|Nombre maximal d’IOPS de données *|2560|3200|3840|4480|5120|
|Taux de journalisation maximal (Mbits/s)|36|36|36|36|36|
|Nombre maximal d’ouvriers simultanés (demandes)|400|500|600|700|800|
|Nombre maximal de connexions simultanées|800|1 000|1200|1400|1 600|
|Nombre maximal de sessions simultanées|30,000|30,000|30,000|30,000|30,000|
|Nombre de réplicas|1|1|1|1|1|
|Plusieurs zones de disponibilités|N/A|N/A|N/A|N/A|N/A|
|Lecture du Scale-out|N/A|N/A|N/A|N/A|N/A|
|Stockage de sauvegarde inclus|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|

\* La valeur maximale pour les tailles d’E/S est comprise entre 8 Ko et 64 Ko. Les IOPS réelles dépendent de la charge de travail. Pour plus d’informations, consultez [Gouvernance des E/S de données](resource-limits-logical-server.md#resource-governance).

### <a name="fsv2-series-compute-generation-part-2"></a>Génération de calcul de série Fsv2 (partie 2)

|Taille de calcul (objectif de service)|GP_Fsv2_18|GP_Fsv2_20|GP_Fsv2_24|GP_Fsv2_32| GP_Fsv2_36|GP_Fsv2_72|
|:---| ---:|---:|---:|---:|---:|---:|
|Génération de calcul|Série Fsv2|Série Fsv2|Série Fsv2|Série Fsv2|Série Fsv2|Série Fsv2|
|vCores|18|20|24|32|36|72|
|Mémoire (Go)|34,0|37,8|45.4|60,5|68,0|136,0|
|Prise en charge de ColumnStore|Oui|Oui|Oui|Oui|Oui|Oui|
|Stockage In-Memory OLTP (Go)|N/A|N/A|N/A|N/A|N/A|N/A|
|Taille maximale des données (Go)|1536|1536|1536|3 072|3 072|4096|
|Taille maximale du journal (Go)|512|512|512|1 024|1 024|1 024|
|Taille maximale des données TempDB (Go)|83,25|92,5|111|148|166,5|333|
|Type de stockage|SSD distant|SSD distant|SSD distant|SSD distant|SSD distant|SSD distant|
|Latence d’E/S (approximative)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|5-7 ms (écriture)<br>5-10 ms (lecture)|
|Nombre maximal d’IOPS de données *|5760|6 400|7680|10240|11520|23040|
|Taux de journalisation maximal (Mbits/s)|36|36|36|36|36|36|
|Nombre maximal d’ouvriers simultanés (demandes)|900|1 000|1200|1 600|1800|3600|
|Nombre maximal de connexions simultanées|1800|2000|2 400|3200|3600|7200|
|Nombre maximal de sessions simultanées|30,000|30,000|30,000|30,000|30,000|30,000|
|Nombre de réplicas|1|1|1|1|1|1|
|Plusieurs zones de disponibilités|N/A|N/A|N/A|N/A|N/A|N/A|
|Lecture du Scale-out|N/A|N/A|N/A|N/A|N/A|N/A|
|Stockage de sauvegarde inclus|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|

\* La valeur maximale pour les tailles d’E/S est comprise entre 8 Ko et 64 Ko. Les IOPS réelles dépendent de la charge de travail. Pour plus d’informations, consultez [Gouvernance des E/S de données](resource-limits-logical-server.md#resource-governance).

## <a name="business-critical---provisioned-compute---gen4"></a>Vital pour l’entreprise - calcul provisionné - Gen4

> [!IMPORTANT]
> Les nouvelles bases de données Gen4 ne sont plus prises en charge dans les régions Australie Est et Brésil Sud.

### <a name="gen4-compute-generation-part-1"></a>Génération de calcul Gen4 (partie 1)

|Taille de calcul (objectif de service)|BC_Gen4_1|BC_Gen4_2|BC_Gen4_3|BC_Gen4_4|BC_Gen4_5|BC_Gen4_6|
|:--- | --: |--: |--: |--: |--: |--: |
|Génération de calcul|Gen4|Gen4|Gen4|Gen4|Gen4|Gen4|
|vCores|1|2|3|4|5|6|
|Mémoire (Go)|7|14|21|28|35|42|
|Prise en charge de ColumnStore|Oui|Oui|Oui|Oui|Oui|Oui|
|Stockage In-Memory OLTP (Go)|1|2|3|4|5|6|
|Type de stockage|SSD local|SSD local|SSD local|SSD local|SSD local|SSD local|
|Taille maximale des données (Go)|1 024|1 024|1 024|1 024|1 024|1 024|
|Taille maximale du journal (Go)|307|307|307|307|307|307|
|Taille maximale des données TempDB (Go)|32|64|96|128|160|192|
|Latence d’E/S (approximative)|1-2 ms (écriture)<br>1-2 ms (lecture)|1-2 ms (écriture)<br>1-2 ms (lecture)|1-2 ms (écriture)<br>1-2 ms (lecture)|1-2 ms (écriture)<br>1-2 ms (lecture)|1-2 ms (écriture)<br>1-2 ms (lecture)|1-2 ms (écriture)<br>1-2 ms (lecture)|
|Nombre maximal d’IOPS de données *|4 000|8,000|12,000|16 000|20 000|24 000|
|Taux de journalisation maximal (Mbits/s)|8|16|24|32|40|48|
|Nombre maximal d’ouvriers simultanés (demandes)|200|400|600|800|1 000|1200|
|Nombre maximal de connexions simultanées|200|400|600|800|1 000|1200|
|Nombre maximal de sessions simultanées|30,000|30,000|30,000|30,000|30,000|30,000|
|Nombre de réplicas|4|4|4|4|4|4|
|Plusieurs zones de disponibilités|Oui|Oui|Oui|Oui|Oui|Oui|
|Lecture du Scale-out|Oui|Oui|Oui|Oui|Oui|Oui|
|Stockage de sauvegarde inclus|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|

\* La valeur maximale pour les tailles d’E/S est comprise entre 8 Ko et 64 Ko. Les IOPS réelles dépendent de la charge de travail. Pour plus d’informations, consultez [Gouvernance des E/S de données](resource-limits-logical-server.md#resource-governance).

### <a name="gen4-compute-generation-part-2"></a>Génération de calcul Gen4 (partie 2)

|Taille de calcul (objectif de service)|BC_Gen4_7|BC_Gen4_8|BC_Gen4_9|BC_Gen4_10|BC_Gen4_16|BC_Gen4_24|
|:--- | --: |--: |--: |--: |--: |--: |
|Génération de calcul|Gen4|Gen4|Gen4|Gen4|Gen4|Gen4|
|vCores|7|8|9|10|16|24|
|Mémoire (Go)|49|56|63|70|112|159,5|
|Prise en charge de ColumnStore|Oui|Oui|Oui|Oui|Oui|Oui|
|Stockage In-Memory OLTP (Go)|7|8|9.5|11|20|36|
|Type de stockage|SSD local|SSD local|SSD local|SSD local|SSD local|SSD local|
|Taille maximale des données (Go)|1 024|1 024|1 024|1 024|1 024|1 024|
|Taille maximale du journal (Go)|307|307|307|307|307|307|
|Taille maximale des données TempDB (Go)|224|256|288|320|512|768|
|Latence d’E/S (approximative)|1-2 ms (écriture)<br>1-2 ms (lecture)|1-2 ms (écriture)<br>1-2 ms (lecture)|1-2 ms (écriture)<br>1-2 ms (lecture)|1-2 ms (écriture)<br>1-2 ms (lecture)|1-2 ms (écriture)<br>1-2 ms (lecture)|1-2 ms (écriture)<br>1-2 ms (lecture)|
|Nombre maximal d’IOPS de données |28 000|32 000|36 000|40 000|64 000|76 800|
|Taux de journalisation maximal (Mbits/s)|56|64|64|64|64|64|
|Nombre maximal d’ouvriers simultanés (demandes)|1400|1 600|1800|2000|3200|4 800|
|Nombre maximal de connexions simultanées (requêtes)|1400|1 600|1800|2000|3200|4 800|
|Nombre maximal de sessions simultanées|30,000|30,000|30,000|30,000|30,000|30,000|
|Nombre de réplicas|4|4|4|4|4|4|
|Plusieurs zones de disponibilités|Oui|Oui|Oui|Oui|Oui|Oui|
|Lecture du Scale-out|Oui|Oui|Oui|Oui|Oui|Oui|
|Stockage de sauvegarde inclus|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|

\* La valeur maximale pour les tailles d’E/S est comprise entre 8 Ko et 64 Ko. Les IOPS réelles dépendent de la charge de travail. Pour plus d’informations, consultez [Gouvernance des E/S de données](resource-limits-logical-server.md#resource-governance).

## <a name="business-critical---provisioned-compute---gen5"></a>Vital pour l’entreprise - calcul provisionné - Gen5

### <a name="gen5-compute-generation-part-1"></a>Génération de calcul Gen5 (partie 1)

|Taille de calcul (objectif de service)|BC_Gen5_2|BC_Gen5_4|BC_Gen5_6|BC_Gen5_8|BC_Gen5_10|BC_Gen5_12|BC_Gen5_14|
|:--- | --: |--: |--: |--: |---: | --: |--: |
|Génération de calcul|Gen5|Gen5|Gen5|Gen5|Gen5|Gen5|Gen5|
|vCores|2|4|6|8|10|12|14|
|Mémoire (Go)|10,4|20,8|31,1|41,5|51,9|62,3|72,7|
|Prise en charge de ColumnStore|Oui|Oui|Oui|Oui|Oui|Oui|Oui|
|Stockage In-Memory OLTP (Go)|1,57|3,14|4.71|6,28|8,65|11,02|13,39|
|Taille maximale des données (Go)|1 024|1 024|1536|1536|1536|3 072|3 072|
|Taille maximale du journal (Go)|307|307|461|461|461|922|922|
|Taille maximale des données TempDB (Go)|64|128|192|256|320|384|448|
|Type de stockage|SSD local|SSD local|SSD local|SSD local|SSD local|SSD local|SSD local|
|Latence d’E/S (approximative)|1-2 ms (écriture)<br>1-2 ms (lecture)|1-2 ms (écriture)<br>1-2 ms (lecture)|1-2 ms (écriture)<br>1-2 ms (lecture)|1-2 ms (écriture)<br>1-2 ms (lecture)|1-2 ms (écriture)<br>1-2 ms (lecture)|1-2 ms (écriture)<br>1-2 ms (lecture)|1-2 ms (écriture)<br>1-2 ms (lecture)|
|Nombre maximal d’IOPS de données *|8000|16 000|24 000|32 000|40 000|48 000|56 000|
|Taux de journalisation maximal (Mbits/s)|24|48|72|96|96|96|96|
|Nombre maximal d’ouvriers simultanés (demandes)|200|400|600|800|1 000|1200|1400|
|Nombre maximal de connexions simultanées|200|400|600|800|1 000|1200|1400|
|Nombre maximal de sessions simultanées|30,000|30,000|30,000|30,000|30,000|30,000|30,000|
|Nombre de réplicas|4|4|4|4|4|4|4|
|Plusieurs zones de disponibilités|Oui|Oui|Oui|Oui|Oui|Oui|Oui|
|Lecture du Scale-out|Oui|Oui|Oui|Oui|Oui|Oui|Oui|
|Stockage de sauvegarde inclus|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|

\* La valeur maximale pour les tailles d’E/S est comprise entre 8 Ko et 64 Ko. Les IOPS réelles dépendent de la charge de travail. Pour plus d’informations, consultez [Gouvernance des E/S de données](resource-limits-logical-server.md#resource-governance).

### <a name="gen5-compute-generation-part-2"></a>Génération de calcul Gen5 (partie 2)

|Taille de calcul (objectif de service)|BC_Gen5_16|BC_Gen5_18|BC_Gen5_20|BC_Gen5_24|BC_Gen5_32|BC_Gen5_40|BC_Gen5_80|
|:--- | --: |--: |--: |--: |---: | --: |--: |
|Génération de calcul|Gen5|Gen5|Gen5|Gen5|Gen5|Gen5|Gen5|
|vCores|16|18|20|24|32|40|80|
|Mémoire (Go)|83|93,4|103,8|124,6|166,1|207,6|415,2|
|Prise en charge de ColumnStore|Oui|Oui|Oui|Oui|Oui|Oui|Oui|
|Stockage In-Memory OLTP (Go)|15,77|18,14|20,51|25,25|37,94|52,23|131,64|
|Taille maximale des données (Go)|3 072|3 072|3 072|4096|4096|4096|4096|
|Taille maximale du journal (Go)|922|922|922|1 024|1 024|1 024|1 024|
|Taille maximale des données TempDB (Go)|512|576|640|768|1 024|1 280|2560|
|Type de stockage|SSD local|SSD local|SSD local|SSD local|SSD local|SSD local|SSD local|
|Latence d’E/S (approximative)|1-2 ms (écriture)<br>1-2 ms (lecture)|1-2 ms (écriture)<br>1-2 ms (lecture)|1-2 ms (écriture)<br>1-2 ms (lecture)|1-2 ms (écriture)<br>1-2 ms (lecture)|1-2 ms (écriture)<br>1-2 ms (lecture)|1-2 ms (écriture)<br>1-2 ms (lecture)|1-2 ms (écriture)<br>1-2 ms (lecture)|
|Nombre maximal d’IOPS de données *|64 000|72 000|80 000|96 000|128 000|160 000|204 800|
|Taux de journalisation maximal (Mbits/s)|96|96|96|96|96|96|96|
|Nombre maximal d’ouvriers simultanés (demandes)|1 600|1800|2000|2 400|3200|4000|8000|
|Nombre maximal de connexions simultanées|1 600|1800|2000|2 400|3200|4000|8000|
|Nombre maximal de sessions simultanées|30,000|30,000|30,000|30,000|30,000|30,000|30,000|
|Nombre de réplicas|4|4|4|4|4|4|4|
|Plusieurs zones de disponibilités|Oui|Oui|Oui|Oui|Oui|Oui|Oui|
|Lecture du Scale-out|Oui|Oui|Oui|Oui|Oui|Oui|Oui|
|Stockage de sauvegarde inclus|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|

\* La valeur maximale pour les tailles d’E/S est comprise entre 8 Ko et 64 Ko. Les IOPS réelles dépendent de la charge de travail. Pour plus d’informations, consultez [Gouvernance des E/S de données](resource-limits-logical-server.md#resource-governance).

## <a name="business-critical---provisioned-compute---m-series"></a>Vital pour l’entreprise - calcul provisionné - série M

### <a name="m-series-compute-generation-part-1"></a>Génération de calcul de série M (partie 1)

|Taille de calcul (objectif de service)|BC_M_8|BC_M_10|BC_M_12|BC_M_14|BC_M_16|BC_M_18|
|:---| ---:|---:|---:|---:|---:|---:|
|Génération de calcul|Série M|Série M|Série M|Série M|Série M|Série M|
|vCores|8|10|12|14|16|18|
|Mémoire (Go)|235,4|294,3|353,2|412,0|470,9|529,7|
|Prise en charge de ColumnStore|Oui|Oui|Oui|Oui|Oui|Oui|
|Stockage In-Memory OLTP (Go)|64|80|96|112|128|150|
|Taille maximale des données (Go)|512|640|768|896|1 024|1152|
|Taille maximale du journal (Go)|171|213|256|299|341|384|
|Taille maximale des données TempDB (Go)|256|320|384|448|512|576|
|Type de stockage|SSD local|SSD local|SSD local|SSD local|SSD local|SSD local|
|Latence d’E/S (approximative)|1-2 ms (écriture)<br>1-2 ms (lecture)|1-2 ms (écriture)<br>1-2 ms (lecture)|1-2 ms (écriture)<br>1-2 ms (lecture)|1-2 ms (écriture)<br>1-2 ms (lecture)|1-2 ms (écriture)<br>1-2 ms (lecture)|1-2 ms (écriture)<br>1-2 ms (lecture)|
|Nombre maximal d’IOPS de données *|12 499|15 624|18 748|21 873|24 998|28 123|
|Taux de journalisation maximal (Mbits/s)|48|60|72|84|96|108|
|Nombre maximal d’ouvriers simultanés (demandes)|800|1 000|1,200|1 400|1 600|1 800|
|Nombre maximal de connexions simultanées|800|1 000|1,200|1 400|1 600|1 800|
|Nombre maximal de sessions simultanées|30000|30000|30000|30000|30000|30000|
|Nombre de réplicas|4|4|4|4|4|4|
|Plusieurs zones de disponibilités|Non|Non|Non|Non|Non|Non|
|Lecture du Scale-out|Oui|Oui|Oui|Oui|Oui|Oui|
|Stockage de sauvegarde inclus|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|

\* La valeur maximale pour les tailles d’E/S est comprise entre 8 Ko et 64 Ko. Les IOPS réelles dépendent de la charge de travail. Pour plus d’informations, consultez [Gouvernance des E/S de données](resource-limits-logical-server.md#resource-governance).

> [!IMPORTANT]
> Dans certaines circonstances, vous devrez peut-être réduire une base de données pour récupérer l’espace inutilisé. Pour plus d’informations, consultez [Gérer l’espace des fichiers dans Azure SQL Database](file-space-manage.md).

### <a name="m-series-compute-generation-part-2"></a>Génération de calcul de série M (partie 2)

|Taille de calcul (objectif de service)|BC_M_20|BC_M_24|BC_M_32|BC_M_64|BC_M_128|
|:---| ---:|---:|---:|---:|---:|
|Génération de calcul|Série M|Série M|Série M|Série M|Série M|
|vCores|20|24|32|64|128|
|Mémoire (Go)|588,6|706,3|941,8|1883,5|3767,0|
|Prise en charge de ColumnStore|Oui|Oui|Oui|Oui|Oui|
|Stockage In-Memory OLTP (Go)|172|216|304|704|1768|
|Taille maximale des données (Go)|1 280|1536|2 048|4096|4096|
|Taille maximale du journal (Go)|427|512|683|1 024|1 024|
|Taille maximale des données TempDB (Go)|4096|2 048|1 024|768|640|
|Type de stockage|SSD local|SSD local|SSD local|SSD local|SSD local|
|Latence d’E/S (approximative)|1-2 ms (écriture)<br>1-2 ms (lecture)|1-2 ms (écriture)<br>1-2 ms (lecture)|1-2 ms (écriture)<br>1-2 ms (lecture)|1-2 ms (écriture)<br>1-2 ms (lecture)|1-2 ms (écriture)<br>1-2 ms (lecture)|
|Nombre maximal d’IOPS de données *|31 248|37 497|49 996|99 993|160 000|
|Taux de journalisation maximal (Mbits/s)|120|144|192|264|264|
|Nombre maximal d’ouvriers simultanés (demandes)|2 000|2 400|3 200|6 400|12 800|
|Nombre maximal de connexions simultanées|2 000|2 400|3 200|6 400|12 800|
|Nombre maximal de sessions simultanées|30000|30000|30000|30000|30000|
|Nombre de réplicas|4|4|4|4|4|
|Plusieurs zones de disponibilités|Non|Non|Non|Non|Non|
|Lecture du Scale-out|Oui|Oui|Oui|Oui|Oui|
|Stockage de sauvegarde inclus|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|1X taille de la base de données|

\* La valeur maximale pour les tailles d’E/S est comprise entre 8 Ko et 64 Ko. Les IOPS réelles dépendent de la charge de travail. Pour plus d’informations, consultez [Gouvernance des E/S de données](resource-limits-logical-server.md#resource-governance).

> [!IMPORTANT]
> Dans certaines circonstances, vous devrez peut-être réduire une base de données pour récupérer l’espace inutilisé. Pour plus d’informations, consultez [Gérer l’espace des fichiers dans Azure SQL Database](file-space-manage.md).



## <a name="next-steps"></a>Étapes suivantes

- Pour connaître les limites de ressources DTU d’une base de données unique, consultez l’article consacré aux [limites de ressources pour les bases de données uniques suivant le modèle d’achat DTU](resource-limits-dtu-single-databases.md)
- Pour connaître les limites de ressources vCore des pools élastiques, consultez l’article consacré aux [limites de ressources pour les pools élastiques suivant le modèle d’achat vCore](resource-limits-vcore-elastic-pools.md)
- Pour connaître les limites de ressources DTU des pools élastiques, consultez l’article consacré aux [limites de ressources pour les pools élastiques suivant le modèle d’achat DTU](resource-limits-dtu-elastic-pools.md)
- Pour connaître les limites de ressources pour SQL Managed Instance, consultez l’article consacré aux [limites de ressources SQL Managed Instance](../managed-instance/resource-limits.md).
- Pour plus d’informations sur les limites générales d’Azure, consultez [Abonnement Azure et limites, quotas et contraintes du service](../../azure-resource-manager/management/azure-subscription-service-limits.md).
- Pour plus d’informations sur les limites de ressources au niveau du serveur et de l’abonnement, consultez l’article [Vue d’ensemble des limites de ressources sur un serveur](resource-limits-logical-server.md).
