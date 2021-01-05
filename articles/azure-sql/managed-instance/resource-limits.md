---
title: Limites des ressources
titleSuffix: Azure SQL Managed Instance
description: Cet article fournit une vue d’ensemble des limites de ressources pour Azure SQL Managed Instance.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: ''
ms.devlang: ''
ms.topic: reference
author: bonova
ms.author: bonova
ms.reviewer: sstein, jovanpop, sachinp
ms.date: 09/14/2020
ms.openlocfilehash: 47c837e7a2ee859c7805d6b2e11058bcc02e6c22
ms.sourcegitcommit: cc13f3fc9b8d309986409276b48ffb77953f4458
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97400571"
---
# <a name="overview-of-azure-sql-managed-instance-resource-limits"></a>Vue d’ensemble des limites de ressources Azure SQL Managed Instance
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Cet article fournit une vue d’ensemble des caractéristiques techniques et des limites des ressources pour Azure SQL Managed Instance, ainsi que des informations sur la façon de demander une augmentation de ces limites.

> [!NOTE]
> Pour connaître les différences entre les fonctionnalités prises en charge et les instructions T-SQL, consultez [Différences de fonctionnalités](../database/features-comparison.md) et [Prise en charge des instructions T-SQL](transact-sql-tsql-differences-sql-server.md). Pour connaître les différences générales entre les niveaux de service pour Azure SQL Database et SQL Managed Instance, consultez [Comparaison des niveaux de service](../database/service-tiers-general-purpose-business-critical.md#service-tier-comparison).

## <a name="hardware-generation-characteristics"></a>Caractéristiques de la génération du matériel

SQL Managed Instance a des caractéristiques et des limites de ressources qui dépendent de l’infrastructure et de l’architecture sous-jacentes. SQL Managed Instance peut être déployé sur deux générations de matériel : Gen4 et Gen5. Les générations du matériel ont différentes caractéristiques, qui sont décrites dans le tableau suivant :

|   | **Gen4** | **Gen5** |
| --- | --- | --- |
| **Matériel** | Processeurs Intel® E5-2673 v3 (Haswell) 2,4 GHz, disque SSD attaché, vCore = 1 PP (cœur physique) | Processeurs Intel® E5-2673 v4 (Broadwell) 2,3 GHz, Intel® SP-8160 (Skylake) et Intel® 8272CL (Cascade Lake) 2,5 GHz, disque SSD NVMe rapide, vCore = 1 LP (hyper-thread) |
| **Nombre de vCore** | 8, 16, 24 vCores | 4, 8, 16, 24, 32, 40, 64, 80 vCores |
| **Mémoire maximale (ratio mémoire/cœur)** | 7 Go par vCore<br/>Ajoutez plus de vCores pour obtenir davantage de mémoire. | 5,1 Go par vCore<br/>Ajoutez plus de vCores pour obtenir davantage de mémoire. |
| **Mémoire OLTP maximale en mémoire** | Limite de l’instance : 1 à 1,5 Go par vCore| Limite de l’instance : 0,8 à 1,65 Go par vCore |
| **Stockage réservé d’instances maximal** |  Usage général : 8 To<br/>Critique pour l’entreprise : 1 To | Usage général : 8 To<br/> Critique pour l’entreprise 1 To, 2 To ou 4 To, en fonction du nombre de cœurs |

> [!IMPORTANT]
> - Le matériel Gen4 est en cours de retrait et n’est plus disponible pour les nouveaux déploiements. Toutes les nouvelles instances de SQL Managed Instance doivent être déployées sur le matériel Gen5.
> - [Déplacez votre instance managée SQL vers le matériel Gen5](../database/service-tiers-vcore.md) pour profiter d’une plus grande scalabilité en termes de vCore et de stockage, d’une accélération réseau, de performances d’E/S optimales et d’une latence minimale.

### <a name="in-memory-oltp-available-space"></a>Espace disponible OLTP en mémoire 

La quantité d’espace OLTP en mémoire au niveau de service [Critique pour l’entreprise](../database/service-tier-business-critical.md) dépend du nombre de vCores et de la génération du matériel. Le tableau suivant liste les limites de mémoire utilisable pour des objets OLTP en mémoire.

| Espace OLTP en mémoire  | **Gen5** | **Gen4** |
| --- | --- | --- |
| 4 vCores  | 3,14 Go | |   
| 8 vCores  | 6,28 Go | 8 Go |
| 16 vCores | 15,77 Go | 20 Go |
| 24 vCores | 25,25 Go | 36 Go |
| 32 vCores | 37,94 Go | |
| 40 vCores | 52,23 Go | |
| 64 vCores | 99,9 Go    | |
| 80 vCores | 131,68 Go| |

## <a name="service-tier-characteristics"></a>Caractéristiques du niveau de service

SQL Managed Instance a deux niveaux de service : [Usage général](../database/service-tier-general-purpose.md) et [Critique pour l’entreprise](../database/service-tier-business-critical.md). Ces niveaux offrent des [fonctionnalités différentes](../database/service-tiers-general-purpose-business-critical.md), comme décrit dans le tableau ci-dessous.

> [!Important]
> Le niveau de service Critique pour l’entreprise fournit une copie intégrée supplémentaire de l’instance managée SQL (réplica secondaire) qui peut être utilisée pour les charges de travail en lecture seule. Si vous pouvez séparer les requêtes de lecture-écriture des requêtes en lecture seule/analytique/de création de rapports, vous obtenez deux fois plus de vCores et de mémoire pour le même prix. Le réplica secondaire peut présenter un décalage de quelques secondes par rapport à l’instance principale. Il est donc conçu pour déplacer les charges de travail de création de rapports/analytiques qui n’ont pas besoin de l’état actuel exact des données. Dans le tableau ci-dessous, les **requêtes en lecture seule** sont les requêtes exécutées sur le réplica secondaire.

| **Fonctionnalité** | **Usage général** | **Critique pour l’entreprise** |
| --- | --- | --- |
| Nombre de vCores\* | Gen4 : 8, 16, 24<br/>Gen5 : 4, 8, 16, 24, 32, 40, 64, 80 | Gen4 : 8, 16, 24 <br/> Gen5 : 4, 8, 16, 24, 32, 40, 64, 80 <br/>\*Le même nombre de vCores est dédié pour les requêtes en lecture seule. |
| Mémoire maximale | Gen4 : 56-168 Go (7 Go/vCore)<br/>Gen5 : 20,4-408 Go (5,1 Go/vCore)<br/>Ajoutez plus de vCores pour obtenir davantage de mémoire. | Gen4 : 56-168 Go (7 Go/vCore)<br/>Gen5 : 20,4 Go-408 Go (5,1 Go/vCore) pour les requêtes en lecture-écriture<br/>+ 20,4 Go-408 Go (5,1 Go/vCore) supplémentaires pour les requêtes en lecture seule.<br/>Ajoutez plus de vCores pour obtenir davantage de mémoire. |
| Taille de stockage maximale d’instance (réservée) | - 2 To pour les 4 vCores (Gen5 uniquement)<br/>- 8 To pour les autres tailles | Gen4 : 1 To <br/> Gen5 : <br/>- 1 To pour 4, 8, 16 vCores<br/>- 2 To pour 24 vCores<br/>- 4 To pour 32, 40, 64, 80 vCores |
| Taille de base de données maximale | Jusqu’à la taille d’instance actuellement disponible (maximum 2-8 To en fonction du nombre de vCores). | Jusqu’à la taille d’instance actuellement disponible (maximum 1-4 To en fonction du nombre de vCores). |
| Taille maximale de tempDB | Limitée à 24 Go/vCore (96-1920 Go) et à la taille de stockage d’instance actuellement disponible.<br/>Ajoutez plus de vCores pour obtenir davantage d’espace TempDB.<br/> La taille du fichier journal est limitée à 120 Go.| Jusqu’à la taille de stockage d’instance actuellement disponible. |
| Nombre maximal de bases de données par instance | 100 bases de données utilisateur, sauf si la limite de taille de stockage d’instance a été atteinte. | 100 bases de données utilisateur, sauf si la limite de taille de stockage d’instance a été atteinte. |
| Nombre maximal de fichiers de base de données par instance | Jusqu’à 280, sauf si la limite de taille de stockage d’instance ou d’[espace d’allocation de stockage sur disque Premium Azure](../database/doc-changes-updates-release-notes.md#exceeding-storage-space-with-small-database-files) a été atteinte. | 32 767 fichiers par base de données, sauf si la limite de taille de stockage d’instance a été atteinte. |
| Taille maximale du fichier de données | Limitée à la taille de stockage d’instance actuellement disponible (maximum 2-8 To) et à l’[espace d’allocation de stockage sur disque Premium Azure](../database/doc-changes-updates-release-notes.md#exceeding-storage-space-with-small-database-files). | Limitée à la taille de stockage d’instances actuellement disponible (jusqu’à 1-4 To). |
| Taille maximale du fichier journal | Limitée à 2 To et à la taille de stockage d’instance actuellement disponible. | Limitée à 2 To et à la taille de stockage d’instance actuellement disponible. |
| IOPS de données/journal (approximatives) | Jusqu’à 30-40 K IOPS par instance*, 500-7500 par fichier<br/>\*[Augmentez la taille de fichier pour obtenir davantage d’IOPS](#file-io-characteristics-in-general-purpose-tier)| 10 000 à 200 000 (4 000 IOPS/vCore)<br/>Ajoutez plus de vCores pour obtenir de meilleures performances d’E/S. |
| Limite du débit d’écriture du journal (par instance) | 3 Mo/s par vCore<br/>120 Mo/s max. par instance<br/>22 à 65 Mo/s par base de données<br/>\*[Augmentez la taille de fichier pour obtenir de meilleures performances d’E/S](#file-io-characteristics-in-general-purpose-tier) | 4 Mo/s par vCore<br/>96 Mo/s max. |
| Débit de données (approximatif) | 100 - 250 Mo/s par fichier<br/>\*[Augmentez la taille de fichier pour obtenir de meilleures performances d’E/S](#file-io-characteristics-in-general-purpose-tier) | Non limité. |
| Latence d’E/S de stockage (approximative) | 5 - 10 ms | 1 - 2 ms |
| OLTP en mémoire | Non pris en charge | Disponible, [la taille dépend du nombre de vCores](#in-memory-oltp-available-space) |
| Nombre maximal de sessions | 30000 | 30000 |
| Nombre maximal d’ouvriers simultanés (demandes) | Gen4 : 210 * nombre de vCores + 800<br>Gen5 : 105 * nombre de vCores + 800 | Gen4 : 210 * nombre de vCores + 800<br>Gen5 : 105 * nombre de vCores + 800 |
| [Réplicas en lecture seule](../database/read-scale-out.md) | 0 | 1 (inclus dans le prix) |
| Isolation du calcul | Gen5 :<br/>\- Pris en charge pour 80 vCores<br/>\- Non pris en charge pour les autres tailles<br/><br/>Gen4 n’est pas pris en charge en raison d’une dépréciation|Gen5 :<br/>\- Pris en charge pour 60, 64, 80 vCores<br/>\- Non pris en charge pour les autres tailles<br/><br/>Gen4 n’est pas pris en charge en raison d’une dépréciation|


Quelques considérations supplémentaires : 

- **La taille de stockage d’instance actuellement disponible** est la différence entre la taille d’instance réservée et l’espace de stockage utilisé.
- Les tailles des données et des fichiers journaux dans les bases de données utilisateur et système sont comprises dans la taille de stockage d’instance qui est comparée à la limite de taille de stockage maximale. Utilisez la vue système [sys.master_files](/sql/relational-databases/system-catalog-views/sys-master-files-transact-sql) pour déterminer l’espace total utilisé par les bases de données. Les journaux d’activité d’erreurs ne sont ni conservés ni compris dans la taille. Les sauvegardes ne sont pas comprises dans la taille de stockage.
- Le débit et les IOPS dans le niveau Usage général dépendent également de la [taille de page](#file-io-characteristics-in-general-purpose-tier) qui n’est pas explicitement limitée par l’instance managée SQL.
  Vous pouvez créer un autre réplica lisible dans une région Azure différente à l’aide de [groupes de basculement automatique](../database/auto-failover-group-configure.md).
- Le nombre maximal d’IOPS d’instance dépend de la disposition des fichiers et de la distribution de la charge de travail. Par exemple, si vous créez sept fichiers de 1 To avec un maximum de 5 000 IOPS et sept fichiers de petite taille (moins de 128 Go) avec 500 IOPS chacun, vous pouvez vous obtenir 38500 IOPS par instance (7x5000+7x500) si votre charge de travail peut utiliser tous les fichiers. Notez que certaines IOPS sont également utilisées pour les sauvegardes automatiques.

Vous trouverez plus d’informations sur les [limites des ressources dans les pools SQL Managed Instance dans cet article](instance-pools-overview.md#resource-limitations).

### <a name="file-io-characteristics-in-general-purpose-tier"></a>Caractéristiques d’E/S de fichier avec le niveau Usage général

Avec le niveau de service Usage général, chaque fichier de base de données reçoit des IOPS et un débit dédiés qui dépendent de la taille du fichier. Les fichiers plus volumineux reçoivent plus d’IOPS et de débit. Les caractéristiques d’E/S des fichiers de base de données sont indiquées dans le tableau suivant :

| Taille du fichier | >= 0 et <= 128 Gio | > 128 et <= 512 Gio | > 0,5 et <= 1 Tio    | > 1 et <= 2 Tio    | > 2 et <= 4 Tio | > 4 et <= 8 Tio |
|---------------------|-------|-------|-------|-------|-------|-------|-------|
| IOPS par fichier       | 500   | 2300              | 5 000              | 7500              | 7500              | 12 500   |
| Débit par fichier | 100 Mio/s | 150 Mio/s | 200 Mio/s | 250 Mio/s | 250 Mio/s | 480 Mio/s | 

Si vous constatez une latence élevée des E/S sur un fichier de base de données, ou si vous constatez que les IOPS ou le débit atteignent la limite, vous pouvez améliorer les performances en [augmentant la taille de fichier](https://techcommunity.microsoft.com/t5/Azure-SQL-Database/Increase-data-file-size-to-improve-HammerDB-workload-performance/ba-p/823337).

Il existe également une limite au niveau de l’instance pour le débit maximal d’écriture dans le journal (voir ci-dessus pour les valeurs, p. ex., 22 Mo/s) ; ainsi, vous ne pourrez peut-être pas atteindre le débit de fichier maximal dans le fichier journal, car vous atteignez la limite de débit de l’instance.

## <a name="supported-regions"></a>Régions prises en charge

SQL Managed Instance peut être créé uniquement dans des [régions prises en charge](https://azure.microsoft.com/global-infrastructure/services/?products=sql-database&regions=all). Si vous voulez créer une instance managée SQL dans une région qui n’est actuellement pas prise en charge, vous pouvez [envoyer une demande de support par le biais du portail Microsoft Azure](../database/quota-increase-request.md).

## <a name="supported-subscription-types"></a>Types d’abonnements pris en charge

SQL Managed Instance prend actuellement en charge le déploiement uniquement sur les types d’abonnements suivants :

- [Contrat Entreprise (EA)](https://azure.microsoft.com/pricing/enterprise-agreement/)
- [Paiement à l’utilisation](https://azure.microsoft.com/offers/ms-azr-0003p/)
- [Fournisseur de services cloud (CSP)](/partner-center/csp-documents-and-learning-resources)
- [Enterprise Dev/Test](https://azure.microsoft.com/offers/ms-azr-0148p/)
- [Dev/Test – Paiement à l’utilisation](https://azure.microsoft.com/offers/ms-azr-0023p/)
- [Abonnements avec crédit Azure mensuel pour les abonnés Visual Studio](https://azure.microsoft.com/pricing/member-offers/credit-for-visual-studio-subscribers/)

## <a name="regional-resource-limitations"></a>Limitations des ressources régionales

> [!Note]
> Pour obtenir les dernières informations sur la disponibilité des régions pour les abonnements, veuillez d’abord [sélectionner une région](https://aka.ms/sqlcapacity).

Les types d’abonnements pris en charge peuvent contenir un nombre limité de ressources par région. SQL Managed Instance a deux limites par défaut par région Azure (qui peuvent être augmentées à la demande en créant une [demande spéciale de support dans le portail Azure](../database/quota-increase-request.md)) en fonction du type d’abonnement :

- **Limite de sous-réseaux** : nombre maximal de sous-réseaux sur lesquels des instances managées SQL sont déployées dans une seule et même région.
- **Limite d’unités de vCore** : nombre maximal d’unités de vCores qui peuvent être déployées parmi toutes les instances dans une seule région. Un vCore GP utilise une unité de vCore et un vCore BC prend quatre unités de vCore. Le nombre total d’instances n’est pas limité du moment qu’il se trouve dans la limite du nombre d’unités de vCores.

> [!Note]
> Ces limites sont des paramètres par défaut : il ne s’agit pas de limitations techniques. Ces limites peuvent être augmentées en créant une [demande de support spéciale dans le portail Microsoft Azure](../database/quota-increase-request.md) si vous avez besoin de plus d’instances dans la région actuelle. Vous pouvez aussi créer des instances managées SQL dans une autre région Azure sans envoyer de demandes de support.

Le tableau suivant montre les **limites régionales par défaut** pour les types d’abonnement pris en charge (les limites par défaut peuvent être étendues à l’aide de la demande de support décrite ci-dessous) :

|Type d’abonnement| Nombre maximal de sous-réseaux SQL Managed Instance | Nombre maximal d’unités de vCore* |
| :---| :--- | :--- |
|Paiement à l’utilisation|3|320|
|CSP |8 (15 dans certaines régions**)|960 (1 440 dans certaines régions**)|
|Pay-As-You-Go Dev/Test|3|320|
|Enterprise Dev/Test|3|320|
|EA|8 (15 dans certaines régions**)|960 (1 440 dans certaines régions**)|
|Visual Studio Enterprise|2 |64|
|Visual Studio Professional et plateformes MSDN|2|32|

\* Lors de la planification de déploiements, prenez en considération le fait que le niveau de service critique pour l’entreprise (BC) requiert quatre (4) fois plus de capacité vCore que le niveau de service usage général (GP). Par exemple : 1 GP vCore = 1 unité vCore et 1 BC vCore = 4 unités vCore. Pour simplifier votre analyse de la consommation par rapport aux limites par défaut, récapitulez les unités vCore de tous les sous-réseaux de la région où SQL Managed Instance est déployé et comparez les résultats avec les limites d’unités d’instance pour votre type d’abonnement. La limite **Nombre maximal d’unités de vCore** s’applique à chaque abonnement dans une région. Il n’y a pas de limite par sous-réseau individuel sauf que la somme de tous les vCores déployés sur plusieurs sous-réseaux doit être inférieure ou égale à **nombre maximum d’unités vCore**.

\*\* Des limites de sous-réseau et de vCore plus importantes s’appliquent dans les régions suivantes : Australie Est, USA Est, USA Est 2, Europe Nord, USA Centre Sud, Asie Sud-Est, Royaume-Uni Sud, Europe Ouest, USA Ouest 2.

> [!IMPORTANT]
> Si la limite de vCore et de sous-réseau est 0, cela signifie que la limite régionale par défaut pour votre type d’abonnement n’est pas définie. Vous pouvez également utiliser la demande d’augmentation de quota pour obtenir l’accès à l’abonnement dans une région spécifique en suivant la même procédure, en fournissant les valeurs de vCore et de sous-réseau requises.

## <a name="request-a-quota-increase"></a>Demander une augmentation de quota

Si vous avez besoin de davantage d’instances dans vos régions actuelles, vous pouvez envoyer une demande de support pour étendre le quota par le biais du portail Azure. Pour plus d’informations, consultez [Demander des augmentations de quota pour Azure SQL Database](../database/quota-increase-request.md).

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur SQL Managed Instance, consultez [Présentation de SQL Managed Instance](sql-managed-instance-paas-overview.md).
- Pour obtenir des informations sur les prix, consultez [Tarification SQL Managed Instance](https://azure.microsoft.com/pricing/details/sql-database/managed/).
- Pour découvrir comment créer votre première instance managée SQL, consultez le [Guide de démarrage rapide](instance-create-quickstart.md).
