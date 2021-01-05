---
title: Recommandations de performances pour SQL Server dans Azure | Microsoft Docs
description: Présente les recommandations pour optimiser les performances de SQL Server dans les machines virtuelles Microsoft Azure.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
editor: ''
tags: azure-service-management
ms.assetid: a0c85092-2113-4982-b73a-4e80160bac36
ms.service: virtual-machines-sql
ms.subservice: performance
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 11/09/2020
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: 2b4e8d980ee2b5c69687fc7ad8975e26fe38071a
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97360116"
---
# <a name="performance-guidelines-for-sql-server-on-azure-virtual-machines"></a>Recommandations de performances pour SQL Server sur les machines virtuelles Azure
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Cet article donne des conseils pour optimiser les performances de SQL Server dans les machines virtuelles Microsoft Azure.

## <a name="overview"></a>Vue d’ensemble

Quand vous exécutez SQL Server sur des machines virtuelles Azure, nous vous recommandons de continuer à utiliser les mêmes options de réglage des performances de base de données qui s’appliquent à SQL Server dans des environnements serveur locaux. Toutefois, les performances d’une base de données relationnelle dans un cloud public dépendent de nombreux facteurs, comme la taille de la machine virtuelle et la configuration des disques de données.

[Les images SQL Server approvisionnées dans le portail Azure](sql-vm-create-portal-quickstart.md) suivent les [meilleures pratiques en matière de configuration](storage-configuration.md) du stockage général. Après l’approvisionnement, vous pouvez appliquer les autres optimisations présentées dans cet article. Faites votre choix en fonction de votre charge de travail et vérifiez en effectuant des tests.

> [!TIP]
> Il existe généralement un compromis entre l’optimisation des coûts et l’optimisation des performances. Cet article est axé sur l’obtention des *meilleures* performances pour SQL Server sur les machines virtuelles Azure. Si votre charge de travail est moindre, vous n’aurez peut-être pas besoin de toutes les optimisations suivantes. Tenez compte de vos besoins de performances, des coûts et des modèles de charges de travail lors de l’évaluation de ces recommandations.

## <a name="quick-checklist"></a>Liste de vérification rapide

Voici une liste de vérification rapide pour optimiser les performances de SQL Server sur machines virtuelles Azure :

| Domaine | Optimisations |
| --- | --- |
| [Taille de la machine virtuelle](#vm-size-guidance) | - Utilisez des tailles de machine virtuelle offrant au moins 4 processeurs virtuels, telles que les séries [Standard_M8-4ms](/azure/virtual-machines/m-series), [E4ds_v4](../../../virtual-machines/edv4-edsv4-series.md#edv4-series) ou [DS12_v2](../../../virtual-machines/dv2-dsv2-series-memory.md#dsv2-series-11-15), ou versions ultérieures. <br/><br/> - Utilisez des tailles de machine virtuelle à [mémoire optimisée](../../../virtual-machines/sizes-memory.md) pour optimiser les performances des charges de travail SQL Server. <br/><br/> - Les séries [DSv2 11-15](../../../virtual-machines/dv2-dsv2-series-memory.md), [Edsv4](../../../virtual-machines/edv4-edsv4-series.md), [M-](/azure/virtual-machines/m-series) et [MV2-](../../../virtual-machines/mv2-series.md) offrent le ratio mémoire/vCore optimal requis pour les charges de travail OLTP. Les deux machines virtuelles de série M offrent le ratio mémoire/vCore le plus élevé requis pour les charges de travail stratégiques, et sont idéales pour des charges de travail d’entrepôt de données. <br/><br/> - Un ratio mémoire/vCore plus élevé peut être requis pour des charges de travail stratégiques et d’entrepôt de données. <br/><br/> - Tirez parti des images de la place de marché des machines virtuelles Azure, car les paramètres et les options de stockage de SQL Server sont configurés pour optimiser les performances de SQL Server. <br/><br/> - Collectez les caractéristiques de performances de la charge de travail cible, et utilisez-les pour déterminer la taille de machine virtuelle appropriée pour votre entreprise.|
| [Stockage](#storage-guidance) | - Le détail des tests de performances de SQL Server sur les machines virtuelles Azure avec les benchmarks TPC-E et TPC_C est consultable sur le billet de blog relatif à l’[optimisation des performances OLTP](https://techcommunity.microsoft.com/t5/SQL-Server/Optimize-OLTP-Performance-with-SQL-Server-on-Azure-VM/ba-p/916794). <br/><br/> - Pour bénéficier du meilleur rapport prix/performances, utilisez des [SSD Premium](https://techcommunity.microsoft.com/t5/SQL-Server/Optimize-OLTP-Performance-with-SQL-Server-on-Azure-VM/ba-p/916794). Configurez un [cache en lecture seule](../../../virtual-machines/premium-storage-performance.md#disk-caching) pour les fichiers de données et aucun cache pour le fichier journal. <br/><br/> - Utilisez des [disques de stockage Ultra](../../../virtual-machines/disks-types.md#ultra-disk) si la charge de travail nécessite des latences de stockage inférieures à 1 ms. Pour plus d’informations, consultez [Migration vers un disque Ultra](storage-migrate-to-ultradisk.md). <br/><br/> - Identifiez les besoins en latence de stockage pour les fichiers de données, le fichier journal et les fichiers TempDB SQL Server en [analysant l’application](../../../virtual-machines/premium-storage-performance.md#application-performance-requirements-checklist) avant de choisir le type de disque. Si des latences de stockage inférieures à 1 ms sont nécessaires, utilisez des disques Ultra. Sinon, utilisez des SSD Premium. Si de faibles latences sont exigées uniquement pour le fichier journal et non pour les fichiers de données, [provisionnez le disque Ultra](../../../virtual-machines/disks-enable-ultra-ssd.md) aux niveaux d’IOPS et de débit exigés uniquement pour le fichier journal. <br/><br/>  - Le stockage standard est préconisé uniquement pour le développement et les tests ou pour les fichiers de sauvegarde. Il ne doit pas être utilisé pour les charges de travail de production. <br/><br/> - Conservez le [compte de stockage](../../../storage/common/storage-account-create.md) et la machine virtuelle SQL Server dans la même région.<br/><br/> - Désactivez le [stockage géoredondant](../../../storage/common/storage-redundancy.md) (géoréplication) Azure sur le compte de stockage.  |
| [Disques](#disks-guidance) | - Utilisez au moins 2 [disques SSD Premium](../../../virtual-machines/disks-types.md#premium-ssd) (1 pour le fichier journal et 1 pour les fichiers de données). <br/><br/> - Pour les charges de travail nécessitant des latences d’E/S inférieures à 1 ms, activez l’accélérateur d’écriture pour la série M et envisagez d’utiliser des disques SSD Ultra pour les séries ES et DS. <br/><br/> - Activez la [mise en cache en lecture seule](../../../virtual-machines/premium-storage-performance.md#disk-caching) sur le ou les disques hébergeant les fichiers de données.<br/><br/> - Ajoutez 20 % de capacité d’IOPS/débit Premium en plus de celle exigée par votre charge de travail pendant la [configuration du stockage pour les fichiers de données, le fichier journal et les fichiers TempDB SQL Server](storage-configuration.md). <br/><br/> - Évitez d’utiliser des disques de système d’exploitation ou temporaires pour le stockage ou la journalisation des bases de données.<br/><br/> - N’activez pas la mise en cache sur le ou les disques hébergeant le fichier journal.  **Important !** Arrêtez le service SQL Server quand vous changez le paramètre de cache d’un disque Machines virtuelles Azure.<br/><br/> - Entrelacez plusieurs disques de données Azure pour bénéficier d’un débit de stockage accru.<br/><br/> - Formatez avec des tailles d’allocation documentées. <br/><br/> - Placez TempDB sur le lecteur `D:\` SSD local pour les charges de travail SQL Server stratégiques (après avoir choisi la taille de machine virtuelle appropriée). Si vous créez la machine virtuelle à partir du portail Azure ou de modèles de démarrage rapide Azure et que vous [placez TempDB sur le disque local](https://techcommunity.microsoft.com/t5/SQL-Server/Announcing-Performance-Optimized-Storage-Configuration-for-SQL/ba-p/891583), vous n’avez rien d’autre à faire. Pour tous les autres cas, suivez les étapes décrites dans le billet de blog relatif à l’[utilisation de disques SSD pour le stockage de TempDB](https://cloudblogs.microsoft.com/sqlserver/2014/09/25/using-ssds-in-azure-vms-to-store-sql-server-TempDB-and-buffer-pool-extensions/) pour éviter des défaillances après les redémarrages. Si la capacité du lecteur local n’est pas suffisante par rapport à la taille de vos fichiers TempDB, transférez ces derniers d’un pool de stockage [supprimé](../../../virtual-machines/premium-storage-performance.md) vers des disques SSD Premium avec la [mise en cache en lecture seule](../../../virtual-machines/premium-storage-performance.md#disk-caching). |
| [E/S](#io-guidance) |- Activez la compression des pages de base de données.<br/><br/> - Activez l’initialisation de fichiers instantanée pour les fichiers de données.<br/><br/> - Limitez la croissance automatique sur la base de données.<br/><br/> - Désactivez la réduction automatique sur la base de données.<br/><br/> - Déplacez toutes les bases de données vers des disques de données, y compris les bases de données système.<br/><br/> - Déplacez les répertoires des journaux d’erreurs et des fichiers de trace SQL Server vers des disques de données.<br/><br/> - Configurez les emplacements par défaut des fichiers de sauvegarde et de base de données.<br/><br/> - [Activez les pages verrouillées en mémoire](/sql/database-engine/configure-windows/enable-the-lock-pages-in-memory-option-windows).<br/><br/> - Évaluez et appliquez les [dernières mises à jour cumulatives](/sql/database-engine/install-windows/latest-updates-for-microsoft-sql-server) pour la version installée de SQL Server. |
| [Spécifique aux fonctionnalités](#feature-specific-guidance) | - Sauvegardez directement dans le stockage Blob Azure.<br/><br/>- Utilisez des [sauvegardes d’instantanés de fichiers](/sql/relational-databases/backup-restore/file-snapshot-backups-for-database-files-in-azure) pour les bases de données de plus de 12 To. <br/><br/>- Utilisez plusieurs fichiers TempDB, 1 fichier par cœur, 8 fichiers au maximum.<br/><br/>- Définissez la mémoire maximum du serveur à 90 % ou jusqu’à 50 Go réservés au système d’exploitation. <br/><br/>- Activez le NUMA logiciel. |


<br/>
Pour plus d’informations sur *comment* et *pourquoi* effectuer ces optimisations, passez en revue les détails et les instructions fournies dans les sections suivantes.
<br/><br/>

## <a name="getting-started"></a>Prise en main

Si vous créez un SQL Server sur une machine virtuelle Azure et ne migrez pas un système source existant, créez votre nouvelle machine virtuelle SQL Server conformément aux exigences de votre fournisseur.  Les exigences du fournisseur pour une machine virtuelle SQL Server sont les mêmes que celles applicables à un déploiement local. 

Si vous créez une machine virtuelle SQL Server avec une nouvelle application conçue pour le cloud, vous pouvez facilement dimensionner votre machine virtuelle SQL Server à mesure que vos besoins en matière de données et d’utilisation évoluent.
Démarrez les environnements de développement avec des machines la série D de niveau inférieur, de la série B ou la série Av-2, et développez votre environnement au fil du temps. 

La configuration minimale recommandée pour un environnement OLTP de production est 4 vCore, 32 Go de mémoire et un ratio mémoire/vCore de 8. Pour de nouveaux environnements, commencez avec 4 machines vCore et augmentez l’échelle jusqu’à 8, 16, 32 vCore ou davantage lorsque vos besoins en matière de données et de calcul évoluent. Pour le débit OLTP, ciblez des machines virtuelles SQL Server offrant une capacité de 5000 IOPS pour chaque vCore. 

Utilisez les images de la place de marché de machines virtuelles SQL Server avec la configuration de stockage dans le portail. Cela facilitera la création des pools de stockage nécessaires pour obtenir la taille, les IOPS et le débit dont vos charges de travail ont besoin. Il est important de choisir des machines virtuelles SQL Server qui prennent en charge le Stockage Premium et la mise en cache du Stockage Premium. Pour en savoir plus, consultez la section [Stockage](#storage-guidance). 

L’entrepôt de données et les environnements stratégiques de SQL Server doivent souvent évoluer au-delà du ratio mémoire/vCore de 8. Pour des environnements de taille moyenne, vous pouvez choisir un ratio cœur/mémoire de 16 et, pour les environnements d’entrepôt de données plus volumineux, un ratio cœur/mémoire de 32. 

Les environnements d’entrepôt de données SQL Server tirent souvent bénéfice du traitement parallèle de machines de plus grande taille. C’est la raison pour laquelle les séries M et Mv2 sont des options solides pour des environnements d’entrepôt de données volumineux.

## <a name="vm-size-guidance"></a>Conseils liés à la taille des machines virtuelles

Utilisez la configuration de processeur virtuel et de mémoire de votre machine source comme ligne de base pour la migration d’une base de données SQL Server locale existante vers SQL Server sur machines virtuelles Azure. Intégrez votre licence principale à Azure pour tirer parti d’[Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-benefit/) et réaliser des économies sur les coûts de licences SQL Server.

**Microsoft recommande un ratio mémoire/vCore de 8 comme point de départ pour les charges de travail SQL Server de production.** Des ratios inférieurs sont acceptables pour des charges de travail hors production. 

Choisissez une taille de machine virtuelle [à mémoire optimisée](../../../virtual-machines/sizes-memory.md), [à usage général](../../../virtual-machines/sizes-general.md), [à stockage optimisé](../../../virtual-machines/sizes-storage.md) ou [à vCore contraint](../../../virtual-machines/constrained-vcpu.md) optimale pour les performances de SQL Server en fonction de votre charge de travail (OLTP ou entrepôt de données). 

### <a name="memory-optimized"></a>Mémoire optimisée

Les [tailles de machine virtuelle à mémoire optimisée](../../../virtual-machines/sizes-memory.md) constituent une cible primaire pour les machines virtuelles SQL Server et le choix recommandé par Microsoft. Les machines virtuelles à mémoire optimisée offrent des options plus solides en matière de ratio mémoire/processeur et de cache de taille moyenne à grande. 

#### <a name="m-and-mv2-series"></a>Séries M et Mv2

La [série M](/azure/virtual-machines/m-series) offre des nombres de vCore et une quantité de mémoire appropriés pour certaines des charges de travail SQL Server les plus volumineuses.  

La [série Mv2](../../../virtual-machines/mv2-series.md) offre les nombres de vCore et la quantité de mémoire les plus élevés. Elle est recommandée pour les charges de travail stratégiques et d’entrepôt de données. Les instances de la série Mv2 sont des tailles de machine virtuelle à mémoire optimisée fournissant des performances de calcul inégalées pour prendre en charge des bases de données et des charges de travail en mémoire volumineuses, avec un ratio mémoire/processeur élevé parfait pour les serveurs de base de données relationnelle, les caches volumineux et l’analytique en mémoire.

Par exemple, les machines [Standard_M64ms](/azure/virtual-machines/m-series) offrent un ratio mémoire/vCore de 28.

Parmi les caractéristiques des séries M et Mv2 attrayantes pour les performances de SQL Server, citons la prise en charge du [Stockage Premium](../../../virtual-machines/premium-storage-performance.md) et de la [mise en cache du Stockage Premium](../../../virtual-machines/premium-storage-performance.md#disk-caching), la prise en charge des [disques Ultra](../../../virtual-machines/disks-enable-ultra-ssd.md) et l’[accélération d’écriture](../../../virtual-machines/how-to-enable-write-accelerator.md).

#### <a name="edsv4-series"></a>Série Edsv4

La [série Edsv4](../../../virtual-machines/edv4-edsv4-series.md) est conçue pour les applications gourmandes en mémoire. Ces machines virtuelles ont une capacité de disque SSD de stockage local importante, des IOPS de disque local solides, jusqu’à 504 Gio de RAM et une capacité de calcul améliorée par rapport aux tailles Ev3/Esv3 précédentes avec des machines virtuelles Gen2. Ces machines virtuelles offrent un ratio mémoire/vCore presque cohérent de 8, idéal pour des charges de travail SQL Server standard. 

Les machines virtuelles de cette série sont idéales pour des applications d’entreprise gourmandes en mémoire et des applications qui tirent un bénéfice d’un stockage local à faible latence et à haut débit.

Les machines virtuelles de la série Edsv4 prennent en charge le [Stockage Premium](../../../virtual-machines/premium-storage-performance.md) et la [mise en cache du Stockage Premium](../../../virtual-machines/premium-storage-performance.md#disk-caching).

#### <a name="dsv2-series-11-15"></a>Série DSv2 11-15

La [série DSv2 11-15](../../../virtual-machines/dv2-dsv2-series-memory.md#dsv2-series-11-15) offre les mêmes configurations de mémoire et disque que la série D précédente. Cette série offre un ratio mémoire/processeur cohérent de 7 sur toutes les machines virtuelles. 

La [série DSv2 11-15](../../../virtual-machines/dv2-dsv2-series-memory.md#dsv2-series-11-15) prend en charge le [Stockage Premium](../../../virtual-machines/premium-storage-performance.md) et la [mise en cache du Stockage Premium](../../../virtual-machines/premium-storage-performance.md#disk-caching), ce qui est fortement recommandé pour des performances optimales.

### <a name="general-purpose"></a>Usage général

Les [tailles de machines virtuelles à usage général](../../../virtual-machines/sizes-general.md) sont conçues pour offrir des ratios mémoire/vCore équilibrés pour des charges de travail d’entrée de gamme plus petites, telles que le développement et les tests, les serveurs web et les serveurs de base de données plus petits. 

En raison des moindres ratios mémoire/vCore des machines virtuelles à usage général, il est important de surveiller attentivement les compteurs de performances basés sur la mémoire pour s’assurer que SQL Server est en mesure d’obtenir la mémoire cache tampon dont il a besoin. Pour plus d’informations, consultez la [ligne de base des performances de la mémoire](#memory). 

Étant donné que la recommandation de départ pour les charges de travail de production est un ratio mémoire/vCore de 8, la configuration minimale recommandée pour une machine virtuelle à usage général exécutant SQL Server est de 32 Go de mémoire et 4 processeurs virtuels. 

#### <a name="ddsv4-series"></a>Série Ddsv4

La [série Ddsv4](../../../virtual-machines/ddv4-ddsv4-series.md) offre une juste combinaison de processeurs virtuels, de mémoire et de disque temporaire, mais avec un moindre ratio mémoire/vCore. 

Les machines virtuelles de la série Ddsv4 incluent un stockage local à latence plus faible et plus rapide.

Ces machines sont idéales pour des déploiements côte à côte de SQL et d’applications, qui requièrent un accès rapide au stockage temporaire et aux bases de données relationnelles de services. L’ensemble des machines virtuelles de cette série offrent un ratio mémoire/vCore standard de 4. 

C’est pourquoi, il est recommandé d’utiliser comme machine virtuelle de démarrage dans cette série le modèle D8ds_v4 qui offre 8 vCore et 32 Go de mémoire. La plus grande machine est le modèle D64ds_v4, avec 64 vCore et 256 Go de mémoire.

Les machines virtuelles de la série [Ddsv4](../../../virtual-machines/ddv4-ddsv4-series.md) prennent en charge le [Stockage Premium](../../../virtual-machines/premium-storage-performance.md) et la [mise en cache du Stockage Premium](../../../virtual-machines/premium-storage-performance.md#disk-caching).

> [!NOTE]
> La [série Ddsv4](../../../virtual-machines/ddv4-ddsv4-series.md) n’offre pas le ratio mémoire/vCore de 8 recommandé pour les charges de travail SQL Server. Par conséquent, l’utilisation de ces machines virtuelles ne doit être envisagée que pour des charges de travail d’application et de développement relativement modestes.

#### <a name="b-series"></a>Série B

Les tailles de machine virtuelle de la [série B burstable](../../../virtual-machines/sizes-b-series-burstable.md) sont idéales pour des charges de travail nécessitant pas de performances soutenues, par exemple, pour une preuve de concept ou de très petits serveurs d’applications et de développement. 

La plupart des machines virtuelles de la [série B burstable](../../../virtual-machines/sizes-b-series-burstable.md) ont un ratio mémoire/vCore de 4. La plus grande de ces machines est le modèle [Standard_B20ms](../../../virtual-machines/sizes-b-series-burstable.md) doté de 20 vCore et de 80 Go de mémoire.

Cette série est unique, car elle permet aux applications d’opérer par **rafales** pendant les heures de bureau avec des crédits burstables variables en fonction de la taille de la machine. 

Une fois ces crédits épuisés, la machine virtuelle revient aux performances de base.

L’avantage de la série B a trait aux économies de calcul que vous pouvez réaliser par rapport aux tailles de machines virtuelles d’autres séries, en particulier si vous avez besoin d’une puissance de traitement modérée tout au long de la journée.

Cette série prend en charge le [Stockage Premium](../../../virtual-machines/premium-storage-performance.md), mais **ne prend pas en charge** la [mise en cache du Stockage Premium](../../../virtual-machines/premium-storage-performance.md#disk-caching).

> [!NOTE] 
> La [série B burstable](../../../virtual-machines/sizes-b-series-burstable.md) n’offre pas le ratio mémoire/vCore de 8 recommandé pour les charges de travail SQL Server. Par conséquent, n’envisagez d’utiliser ces machines virtuelles que pour des applications, serveurs web et charges de travail de développement de taille modeste.

#### <a name="av2-series"></a>Série Av2

Les machines virtuelles de la [série AV2](../../../virtual-machines/av2-series.md) sont idéales pour des charges de travail d’entrée de gamme, par exemple, le développement et les tests, les serveurs web à faible trafic, les bases de données d’applications petites à moyennes et les preuves de concept.

Seuls les modèles [Standard_A2m_v2](../../../virtual-machines/av2-series.md) (2 vCore et 16 Go de mémoire), [Standard_A4m_v2](../../../virtual-machines/av2-series.md) (4 vCore et 32 Go de mémoire) et [Standard_A8m_v2](../../../virtual-machines/av2-series.md) (8 vCore et 64 Go de mémoire) offrent un bon ratio mémoire/vCore de 8. 

Ces machines virtuelles constituent de bonnes options pour les machines SQL Server de développement et de test de taille modeste. 

Le modèle [Standard_A8m_v2](../../../virtual-machines/av2-series.md) à 8 vCore peut également être une bonne option pour des serveurs web et d’applications de petite taille.

> [!NOTE] 
> La série AV2 ne prenant pas en charge le Stockage Premium, elle n’est pas recommandée pour des charges de travail SQL Server de production, même avec les machines virtuelles offrant un ratio mémoire/vCore de 8.

### <a name="storage-optimized"></a>Optimisé pour le stockage

Les [tailles de machine virtuelle à stockage optimisé](../../../virtual-machines/sizes-storage.md) sont destinées à des cas d’usage spécifiques. Ces machines virtuelles sont spécialement conçues avec un débit de disque et des E/S optimisés. Cette série de machines virtuelles est conçue pour les scénarios de Big Data, l’entreposage de données et les bases de données transactionnelles volumineuses. 

#### <a name="lsv2-series"></a>Série Lsv2

La [série Lsv2](../../../virtual-machines/lsv2-series.md) offre un débit élevé, une faible latence et un stockage NVMe local. Les machines virtuelles de la série Lsv2 sont optimisées pour utiliser le disque local du nœud attaché directement à la machine virtuelle au lieu d’utiliser des disques de données durables. 

Ces machines virtuelles constituent des options solides pour des charges de travail de Big Data, d’entrepôt de données, de création de rapports et d’extraction, transformation et chargement (ETL). Le débit élevé et les IOPS du stockage NVMe local sont appropriés pour traiter des fichiers qui seront chargés dans votre base de données et d’autres scénarios dans lesquels les données sources peuvent être recréées à partir du système source ou d’autres référentiels tels que Stockage Blob Azure ou Azure Data Lake. Les machines virtuelles de la [série Lsv2](../../../virtual-machines/lsv2-series.md) peuvent également augmenter en rafale leurs performances de disque pendant jusqu’à 30 minutes.

Ces machines virtuelles offrent une taille de 8 à 80 processeurs virtuels, avec 8 Gio de mémoire par processeur et 1,92 To de disque SSD NVMe par groupe de 8 processeurs virtuels. Cela signifie que la plus grande machine virtuelle de cette série, le modèle [L80s_v2](../../../virtual-machines/lsv2-series.md), est dotée de 80 processeurs virtuels, de 640 Gio de mémoire et de 10 x 1,92 To de stockage NVMe.  Toutes ces machines virtuelles offrent un ratio mémoire/vCore cohérent de 8.

Le stockage NVMe est éphémère, ce qui signifie que les données stockées sur ces disques sont perdues si vous redémarrez votre machine virtuelle.

Les séries Lsv2 et Ls prennent en charge le [Stockage Premium](../../../virtual-machines/premium-storage-performance.md), mais pas la mise en cache du Stockage Premium. La création d’un cache local pour augmenter les IOPS n’est pas prise en charge. 

> [!WARNING]
> Le stockage de vos fichiers de données sur le stockage NVMe éphémère peut entraîner une perte de données lors de la désallocation de la machine virtuelle. 

### <a name="constrained-vcores"></a>VCore contraints

Les charges de travail SQL Server hautes performances nécessitent davantage de mémoire, d’E/S et de débit, sans compter le nombre plus élevé de vCore. 

La plupart des charges de travail OLTP sont des bases de données d’application pilotées par de grands nombres de transactions plus petites. Avec des charges de travail OLTP, seule une petite quantité des données sont lues ou modifiées, mais les volumes des transactions pilotées par l’utilisateur sont beaucoup plus importants. Il est important d’avoir la mémoire SQL Server disponible pour mettre en cache des plans, stocker des données récemment consultées à des fins de performances et de veiller à ce que les lectures physiques puissent être lues rapidement en mémoire. 

Ces environnements OLTP ont besoin de davantage de mémoire, d’un stockage rapide et de la bande passante d’E/S nécessaire pour une exécution optimale. 

Pour maintenir ce niveau de performances sans exposer des coûts de licences SQL Server plus élevés, Azure propose des tailles de machine virtuelle avec des [processeurs virtuels en nombre limité](../../../virtual-machines/constrained-vcpu.md). 

Cela permet de contrôler les coûts de licences en réduisant les vCore disponibles tout en conservant les mémoire, stockage et bande passante d’E/S de la machine virtuelle parente.

Le nombre de processeurs virtuels peut être contraint à la moitié ou au quart de la taille de la machine virtuelle d’origine. En réduisant le nombre de vCore disponibles pour la machine virtuelle, vous obtiendrez des ratios mémoire/vCore plus élevés.

Pour faciliter leur identification, ces nouvelles tailles de machine virtuelle sont assorties d’un suffixe indiquant le nombre de processeurs virtuels actifs. 

Par exemple, le modèle [M64-32ms](../../../virtual-machines/constrained-vcpu.md) requiert des licences uniquement pour 32 vCore SQL Server avec la mémoire, les E/S et le débit du modèle [M64ms](/azure/virtual-machines/m-series), et le modèle [M64-16ms](../../../virtual-machines/constrained-vcpu.md) requiert des licences uniquement pour 16 vCore.  Bien que le modèle [M64-16ms](../../../virtual-machines/constrained-vcpu.md) soit proposé au quart du coût de licence SQL Server du modèle M64ms, le coût de calcul de la machine virtuelle est le même.

> [!NOTE] 
> - Les charges de travail d’entrepôt de données de taille moyenne à grande peuvent toujours tirer bénéfice des [machines virtuelles vCore contraintes](../../../virtual-machines/constrained-vcpu.md), mais les charges de travail d’entrepôt de données sont généralement caractérisées par un moindre nombre d’utilisateurs et de processus traitant de grandes quantités de données via des plans de requête s’exécutant en parallèle. 
> - Le coût de calcul, qui inclut la licence du système d’exploitation, reste identique à celui de la machine virtuelle parente. 

## <a name="storage-guidance"></a>Conseils liés au stockage

Pour connaître le détail des tests de performances de SQL Server sur des machines virtuelles Azure avec les benchmarks TPC-E et TPC_C, consultez le blog relatif à l’[optimisation des performances OLTP](https://techcommunity.microsoft.com/t5/SQL-Server/Optimize-OLTP-Performance-with-SQL-Server-on-Azure-VM/ba-p/916794). 

Le cache Blob Azure avec disques SSD Premium est recommandé pour toutes les charges de travail de production. 

> [!WARNING]
> Les disques durs standard et les disques SSD possèdent différents temps de latence et une bande passante variable. Ils sont recommandés uniquement pour les charges de travail de développement/de test. Les charges de production doivent utiliser des disques SSD premium.

Par ailleurs, nous vous recommandons de créer votre compte de stockage Azure dans le même centre de données que vos machines virtuelles SQL Server afin de réduire les délais de transfert. Lors de la création d’un compte de stockage, désactivez la géo-réplication, étant donné que la cohérence de l’ordre d’écriture sur différents disques n’est pas garantie. Envisagez plutôt de configurer une technologie de récupération d’urgence de SQL Server entre deux centres de données Azure. Pour plus d’informations, voir [Haute disponibilité et récupération d’urgence pour SQL Server dans Azure Virtual Machines](business-continuity-high-availability-disaster-recovery-hadr-overview.md).

## <a name="disks-guidance"></a>Conseils liés aux disques

Il existe trois types de disques principaux sur les machines virtuelles Azure :

* **Disque de système d’exploitation** : Quand vous créez une machine virtuelle Azure, la plateforme attache au moins un disque (désigné par la lettre **C**) à la machine virtuelle en tant que disque de système d’exploitation. Ce disque est un disque dur virtuel (VHD) stocké en tant qu’objet blob de pages dans le stockage.
* **Disque temporaire** : Les machines virtuelles Azure contiennent un autre disque, appelé disque temporaire (portant le nom de lecteur **D**:). C’est un disque du nœud qui peut être utilisé comme un espace de travail temporaire.
* **Disques de données** : Vous pouvez également attacher des disques supplémentaires à votre machine virtuelle en tant que disques de données ; ces disques sont également stockés en tant qu’objets blob de pages.

Les sections suivantes décrivent des recommandations pour l’utilisation de ces disques différents.

### <a name="operating-system-disk"></a>Disque de système d’exploitation

Un disque de système d’exploitation est un disque dur virtuel (VHD) que vous pouvez amorcer et monter comme version d’exécution d’un système d’exploitation. Il est désigné par la lettre de lecteur **C**.

La stratégie de mise en cache par défaut sur le disque du système d’exploitation est **Lecture/Écriture**. Pour les applications sensibles aux performances, nous vous recommandons d’utiliser le disque de données au lieu du disque du système d’exploitation. Reportez-vous à la section sur les disques de données ci-dessous.

### <a name="temporary-disk"></a>Disque temporaire

Le lecteur de stockage temporaire, désigné par la lettre **D**, n’est pas persistant dans le Stockage Blob Azure. Ne stockez pas vos fichiers de base de données utilisateur ou vos fichiers journaux des transactions utilisateur sur le lecteur **D**:.

Placez TempDB sur le lecteur `D:\` SSD local pour les charges de travail SQL Server stratégiques (après avoir choisi la bonne taille de machine virtuelle). Si vous créez la machine virtuelle à partir du portail Azure ou de modèles de démarrage rapide Azure et que vous [placez TempDB sur le disque local](https://techcommunity.microsoft.com/t5/SQL-Server/Announcing-Performance-Optimized-Storage-Configuration-for-SQL/ba-p/891583), vous n’avez rien d’autre à faire. Pour tous les autres cas, suivez les étapes décrites dans le billet de blog relatif à l’[utilisation de disques SSD pour le stockage de TempDB](https://cloudblogs.microsoft.com/sqlserver/2014/09/25/using-ssds-in-azure-vms-to-store-sql-server-TempDB-and-buffer-pool-extensions/) pour éviter des défaillances après les redémarrages. Si la capacité du lecteur local n’est pas suffisante par rapport à la taille de vos fichiers TempDB, transférez ces derniers d’un pool de stockage [supprimé](../../../virtual-machines/premium-storage-performance.md) vers des disques SSD Premium avec la [mise en cache en lecture seule](../../../virtual-machines/premium-storage-performance.md#disk-caching).

Pour des machines virtuelles qui prennent en charge les SSD Premium, vous pouvez aussi stocker TempDB sur un disque qui prend en charge les SSD Premium avec la mise en cache en lecture activée.


### <a name="data-disks"></a>Disques de données

* **Utilisez des disques de données pour les fichiers journaux et de données** : si vous n’utilisez pas l’entrelacement de disques, utilisez deux disques SSD premium (un pour le fichier journal et l’autre pour les données). Chaque SSD Premium offre un nombre d’IOPS et une bande passante (Mo/s) qui varient en fonction de sa taille, comme décrit dans l’article [Sélectionner un type de disque](../../../virtual-machines/disks-types.md). Si vous utilisez une technique d’entrelacement de disques, comme des espaces de stockage, vous obtenez des performances optimales en ayant deux pools, un pour les fichiers journaux et l’autre pour les fichiers de données. Cependant, si vous envisagez d’utiliser des instances de cluster de basculement SQL Server, vous devez configurer un pool ou utiliser des [partages de fichiers Premium](failover-cluster-instance-premium-file-share-manually-configure.md).

   > [!TIP]
   > - Pour obtenir les résultats des tests sur différentes configurations de disque et de charge de travail, consultez le billet de blog suivant : [Storage Configuration Guidelines for SQL Server on Azure Virtual Machines](/archive/blogs/sqlserverstorageengine/storage-configuration-guidelines-for-sql-server-on-azure-vm).
   > - Pour les performances stratégiques des serveurs SQL Server nécessitant ~50 000 E/S par seconde, envisagez de remplacer 10 disques -P30 par un disque SSD Ultra. Pour plus d’informations, consultez le billet de blog suivant : [Performances stratégiques avec un disque SSD Ultra](https://azure.microsoft.com/blog/mission-critical-performance-with-ultra-ssd-for-sql-server-on-azure-vm/).

   > [!NOTE]
   > Lorsque vous configurez une machine virtuelle SQL Server dans le portail, vous avez la possibilité de modifier votre configuration de stockage. Selon votre configuration, Azure configure un ou plusieurs disques. Plusieurs disques sont combinés en un pool de stockage unique par entrelacement. Les fichiers journaux et de données se trouvent dans cette configuration. Pour plus d’informations, voir [Configuration du stockage pour les machines virtuelles SQL Server](storage-configuration.md).

* **Entrelacement de disques** : Pour augmenter le débit, vous pouvez ajouter des disques de données et utiliser l’entrelacement de disques. Pour déterminer le nombre de disques de données, vous devez analyser le nombre d’opérations d’E/S par seconde et de bande passante disponibles pour vos fichiers journaux, vos données et vos fichiers TempDB. Notez que les différentes tailles de machine virtuelle sont associées à des limites spécifiques concernant la bande passante et le nombre d’opérations d’E/S par seconde pris en charge. Pour en savoir plus, consultez les tableaux relatifs aux nombre d’opérations d’E/S par seconde en fonction des [tailles de machine virtuelle](../../../virtual-machines/sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json). Respectez les recommandations suivantes :

  * Pour Windows 8/Windows Server 2012 ou version ultérieure, utilisez des [espaces de stockage](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/hh831739(v=ws.11)) en respectant les consignes suivantes :

      1. Définissez l’intervalle (taille de bande) sur 64 Ko (65 536 octets) pour les charges de travail OLTP et sur 256 Ko (262 144 octets) pour les charges de travail d’entrepôt de données, afin d’éviter qu’un alignement de partition incorrect n’affecte les performances. Ces paramètres doivent être définis avec PowerShell.
      2. Nombre de colonnes définies = nombre de disques physiques. Utilisez PowerShell lorsque vous configurez plus de 8 disques (et non l’interface utilisateur du gestionnaire de serveur). 

    Par exemple, le code PowerShell suivant crée un pool de stockage avec un entrelacement de 64 Ko et un nombre de colonnes équivalent à la quantité de disques physiques dans le pool de stockage :

    ```powershell
    $PhysicalDisks = Get-PhysicalDisk | Where-Object {$_.FriendlyName -like "*2" -or $_.FriendlyName -like "*3"}
    
    New-StoragePool -FriendlyName "DataFiles" -StorageSubsystemFriendlyName "Storage Spaces*" `
        -PhysicalDisks $PhysicalDisks | New- VirtualDisk -FriendlyName "DataFiles" `
        -Interleave 65536 -NumberOfColumns $PhysicalDisks .Count -ResiliencySettingName simple `
        –UseMaximumSize |Initialize-Disk -PartitionStyle GPT -PassThru |New-Partition -AssignDriveLetter `
        -UseMaximumSize |Format-Volume -FileSystem NTFS -NewFileSystemLabel "DataDisks" `
        -AllocationUnitSize 65536 -Confirm:$false 
    ```

  * Pour Windows 2008 R2 ou version antérieure, vous pouvez utiliser des disques dynamiques (volumes entrelacés de système d’exploitation). La taille de l’entrelacement restera 64 Ko. Cette option est déconseillée à partir de Windows 8/Windows Server 2012. Pour plus d’informations, consultez les instructions du support dans [Transition du service de disque virtuel vers l’API de gestion de stockage Windows](/windows/win32/w8cookbook/vds-is-transitioning-to-wmiv2-based-windows-storage-management-api).

  * Si vous utilisez les [espaces de stockage direct (S2D)](/windows-server/storage/storage-spaces/storage-spaces-direct-in-vm) avec des [instances de cluster de basculement SQL Server](failover-cluster-instance-storage-spaces-direct-manually-configure.md), vous devez configurer un pool unique. Même si vous créez différents volumes sur ce pool unique, tous partageront les mêmes caractéristiques, par exemple la même stratégie de mise en cache.

  * Déterminez le nombre de disques associés à votre pool de stockage en fonction de vos attentes en matière de charge. N’oubliez pas que les différentes tailles de machines virtuelles autorisent différents nombres de disques de données attachés. Pour plus d’informations, consultez [Tailles des machines virtuelles](../../../virtual-machines/sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

  * Si vous n’utilisez pas les SSD premium (scénarios de développement/de test), nous vous recommandons d’ajouter le nombre maximal de disques de données pris en charge par la [taille de votre machine virtuelle](../../../virtual-machines/sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) et d’utiliser l’entrelacement de disques.

* **Stratégie de mise en cache** : Notez les recommandations suivantes pour la stratégie de mise en cache en fonction de la configuration de votre stockage.

  * Si vous utilisez des disques différents pour les fichiers journaux et de données, activez la mise en cache de lecture sur les disques de données hébergeant vos fichiers de données et les fichiers de données TempDB. Cela devrait vous permettre d’obtenir un gain de performances significatif. N’activez pas la mise en cache sur le disque contenant le fichier journal, car cela entraîne une légère diminution des performances.

  * Si vous utilisez l’entrelacement de disques dans un pool de stockage unique, la plupart des charges de travail peuvent tirer parti de la mise en cache en lecture. Si vous avez des pools de stockage distincts pour les fichiers journaux et de données, activez la mise en cache en lecture uniquement sur le pool de stockage pour les fichiers de données. Avec certaines charges de travail nécessitant beaucoup d’écritures, il est possible d’améliorer les performances sans mise en cache. Cela ne peut être déterminé que par le biais de test.

  * Les suggestions précédentes s’appliquent aux disques SSD premium. Si vous n’utilisez pas des disques SSD premium, n’activez aucune mise en cache sur les disques de données.

  * Pour obtenir des instructions sur la configuration de la mise en cache des disques, consultez les articles suivants. Pour plus d’informations sur le modèle de déploiement classique (ASM), consultez : [Set-AzureOSDisk](/previous-versions/azure/jj152847(v=azure.100)) et [Set-AzureDataDisk](/previous-versions/azure/jj152851(v=azure.100)). Pour plus d’informations sur le modèle de déploiement Azure Resource Manager, consultez : [Set-AzOSDisk](/powershell/module/az.compute/set-azvmosdisk) et [Set-AzVMDataDisk](/powershell/module/az.compute/set-azvmdatadisk).

     > [!WARNING]
     > Arrêtez le service SQL Server quand vous changez le paramètre de cache de disques de machines virtuelles Azure pour éviter toute altération de la base de données.

* **Taille d’unité d’allocation NTFS** : Lors du formatage du disque de données, il est recommandé d’utiliser une taille d’unité d’allocation de 64 Ko pour les fichiers de données et les fichiers journaux, ainsi que pour TempDB. Si la base de données TempDB est placée sur le disque temporaire (lecteur D:\), le gain de performances obtenu en tirant parti de ce lecteur l’emporte sur la nécessité d’une taille d’unité d’allocation de 64 Ko. 

* **Bonnes pratiques de gestion des disques** : Lors de la suppression d’un disque de données ou de la modification de son type de cache, arrêtez le service SQL Server. Lorsque les paramètres de mise en cache sont modifiés sur le disque du système d’exploitation, Azure arrête la machine virtuelle, modifie le type de cache et redémarre la machine virtuelle. Lorsque les paramètres du cache d’un disque de données sont modifiés, la machine virtuelle n’est pas arrêtée, mais le disque de données est détaché de la machine virtuelle lors de la modification, puis rattaché.

  > [!WARNING]
  > Si le service SQL Server n’est pas arrêté au cours de ces opérations, cela peut provoquer une altération de la base de données.


## <a name="io-guidance"></a>Recommandations liées aux E/S

* Pour obtenir les meilleurs résultats avec les disques SSD premium, vous devez paralléliser vos applications et vos demandes. Les disques SSD Premium sont conçus pour les scénarios où la profondeur de file d’attente est supérieure à 1. Les améliorations seront donc minimes ou inexistantes pour les demandes de série à thread unique (même si elles sont gourmandes en stockage). Par exemple, cela peut affecter les résultats des tests à thread unique des outils d’analyses de performances, par exemple SQLIO.

* Envisagez d’utiliser un [compression des pages de base de données](/sql/relational-databases/data-compression/data-compression) , car elle peut améliorer les performances de charges de travail gourmandes en E/S. Toutefois, la compression de données peut augmenter la consommation d’UC sur le serveur de base de données.

* Envisagez d’activer l’initialisation instantanée des fichiers pour réduire le temps requis pour l’allocation initiale des fichiers. Pour tirer parti de l’initialisation instantanée des fichiers, vous devez accorder l’autorisation SE_MANAGE_VOLUME_NAME au compte de service SQL Server (MSSQLSERVER) et l’ajouter à la stratégie de sécurité **Effectuer des tâches de maintenance sur les volumes**. Si vous utilisez une image de plateforme SQL Server pour Azure, le compte de service par défaut (NT Service\MSSQLSERVER) n’est pas ajouté à la stratégie de sécurité **Effectuer des tâches de maintenance sur les volumes**. En d’autres termes, l’initialisation instantanée des fichiers n’est pas activée dans une image de plateforme SQL Server pour Azure. Après avoir ajouté le compte de service SQL Server à la stratégie de sécurité **Effectuer les tâches de maintenance de volume** , redémarrez le service SQL Server. Des considérations de sécurité liées à l’utilisation de cette fonctionnalité peuvent exister. Pour plus d’informations, consultez [Initialisation des fichiers de base de données](/sql/relational-databases/databases/database-instant-file-initialization).

* Gardez à l’esprit qu’**autogrow** est considéré comme un simple plan d’urgence en cas de croissance plus rapide que prévu. Ne gérez pas la croissance de vos données et journaux quotidiennement avec la croissance automatique. En cas d’utilisation de la croissance automatique, augmentez préalablement le fichier à l’aide du commutateur Taille.

* Vérifiez que la fonctionnalité de réduction automatique ( **autoshrink** ) est désactivée afin d’éviter une surcharge inutile susceptible d’affecter négativement les performances.

* Déplacer toutes les bases de données vers des disques de données, y compris les bases de données système. Pour plus d’informations, consultez [Déplacer des bases de données système](/sql/relational-databases/databases/move-system-databases).

* Déplacer les répertoires des journaux d’erreurs et des fichiers de trace SQL Server vers des disques de données. Cette opération peut être effectuée dans le Gestionnaire de configuration SQL Server en double-cliquant sur l’instance de SQL Server et en sélectionnant des propriétés. Il est possible de modifier les paramètres du fichier journal et du fichier de trace sous l’onglet **Paramètres de démarrage** . Le répertoire de vidage est spécifié sous l’onglet **Avancé** . La capture d’écran suivante indique où rechercher le paramètre de démarrage du journal des erreurs.

    ![Capture d’écran du journal des erreurs SQL](./media/performance-guidelines-best-practices/sql_server_error_log_location.png)

* Configurez les emplacements par défaut du fichier de sauvegarde et du fichier de base de données. Utilisez les recommandations de cet article, puis apportez les modifications souhaitées dans la fenêtre Propriétés du serveur. Pour obtenir des instructions, consultez [Afficher ou modifier les emplacements par défaut des fichiers de données et des fichiers journaux (SQL Server Management Studio)](/sql/database-engine/configure-windows/view-or-change-the-default-locations-for-data-and-log-files). La capture d’écran suivante montre comment effectuer ces modifications.

    ![Journal des données et fichiers de sauvegarde SQL](./media/performance-guidelines-best-practices/sql_server_default_data_log_backup_locations.png)
* Activez les pages verrouillées pour réduire les activités d’E/S et de pagination. Pour plus d’informations, consultez [Activer l’option Lock Pages in Memory (Windows)](/sql/database-engine/configure-windows/enable-the-lock-pages-in-memory-option-windows).

* Si vous exécutez SQL Server 2012, installez la mise à jour cumulative 10 Service Pack 1. Elle contient le correctif qui permet de résoudre les problèmes de performances d’E/S lorsque vous exécutez une sélection dans l’instruction de table temporaire dans SQL Server 2012. Pour plus d’informations, consultez cet [article de la Base de connaissances](https://support.microsoft.com/kb/2958012).

* Envisagez de compresser tous les fichiers de données lors des transferts vers et depuis Azure.

## <a name="feature-specific-guidance"></a>Conseils spécifiques aux fonctionnalités

Certains déploiements peuvent bénéficier de plus grands avantages en termes de performances à l’aide de techniques de configuration avancées. La liste suivante présente certaines fonctionnalités SQL Server qui peuvent vous aider à améliorer les performances :

### <a name="back-up-to-azure-storage"></a>Sauvegarder sur Stockage Azure
Durant la création de sauvegardes pour un serveur SQL Server s’exécutant sur des machines virtuelles Azure, vous pouvez vous référer à [Sauvegarde de SQL Server vers une URL](/sql/relational-databases/backup-restore/sql-server-backup-to-url). Cette fonctionnalité est disponible à partir de SQL Server 2012 SP1 CU2, et recommandée pour la sauvegarde vers les disques de données attachés. Lors de la sauvegarde ou la restauration vers/depuis Stockage Azure, suivez les recommandations indiquées dans [Meilleures pratiques et résolution des problèmes pour la sauvegarde de SQL Server vers une URL et Restauration à partir de sauvegardes stockées dans Azure Storage](/sql/relational-databases/backup-restore/sql-server-backup-to-url-best-practices-and-troubleshooting). Vous pouvez également automatiser ces sauvegardes en utilisant la [Sauvegarde automatisée pour SQL Server sur les machines virtuelles Azure](../../../azure-sql/virtual-machines/windows/automated-backup-sql-2014.md).

Pour les versions antérieures à SQL Server 2012, vous pouvez utiliser l’ [outil de sauvegarde SQL Server vers Azure](https://www.microsoft.com/download/details.aspx?id=40740). Cet outil peut augmenter le débit de sauvegarde à l’aide de plusieurs cibles d’entrelacement de sauvegarde.

### <a name="sql-server-data-files-in-azure"></a>Fichiers de données SQL Server dans Azure

Cette nouvelle fonctionnalité, [Fichiers de données SQL Server dans Azure](/sql/relational-databases/databases/sql-server-data-files-in-microsoft-azure), est disponible à partir de SQL Server 2014. L’exécution de SQL Server avec des fichiers de données dans Azure offre des caractéristiques de performances comparables à l’utilisation de disques de données Azure.

### <a name="failover-cluster-instance-and-storage-spaces"></a>Instance de cluster de basculement et espaces de stockage

Si vous utilisez des espaces de stockage, lors de l’ajout de nœuds dans le cluster sur la page **Confirmation**, décochez la case **Ajouter la totalité du stockage disponible au cluster**. 

![Décochez la case du stockage éligible](./media/performance-guidelines-best-practices/uncheck-eligible-cluster-storage.png)

Si vous utilisez des espaces de stockage et que vous ne désactivez pas la case à cocher **Ajouter la totalité du stockage disponible au cluster**, Windows détache les disques virtuels pendant le processus de mise en cluster. Par conséquent, ils n’apparaissent pas dans le Gestionnaire de disque ou dans l’Explorateur, jusqu’à ce que les espaces de stockage soient supprimés du cluster et rattachés à l’aide de PowerShell. Les espaces de stockage regroupent plusieurs disques dans des pools de stockage. Pour plus d’informations, consultez la page [Espaces de stockage](/windows-server/storage/storage-spaces/overview).

## <a name="multiple-instances"></a>Plusieurs instances 

Tenez compte des meilleures pratiques suivantes lorsque vous déployez plusieurs instances SQL sur une seule machine virtuelle : 

- Définissez la mémoire maximale du serveur pour chaque instance SQL, en vous assurant qu’il reste de la mémoire pour le système d’exploitation. Veillez à mettre à jour les restrictions de mémoire pour les instances SQL si vous modifiez la quantité de mémoire allouée à la machine virtuelle. 
- Ayez des numéros d’unité logique distincts pour les données, les journaux et TempDB puisqu’ils ont tous des modèles de charge de travail différents et que vous ne souhaitez pas qu’ils aient un impact les uns sur les autres. 
- Testez minutieusement votre environnement avec des charges de travail importantes semblables à celle de production pour vous assurer qu’il peut gérer la capacité de pointe conformément au contrat de niveau de service de l’application. 

Les signes d’un système surchargé incluent notamment l’épuisement des threads de travail, des temps de réponse longs et/ou le blocage de la mémoire système du répartiteur. 



## <a name="collect-performance-baseline"></a>Collecter un référentiel de performance

Pour une approche plus prescriptive, recueillez les compteurs de performances à l’aide de PerfMon/LogMan, et capturez les statistiques d’attente de SQL Server pour mieux comprendre les pressions générales et les goulots d’étranglement potentiels de l’environnement source. 

Commencez par collecter le processeur, la mémoire, les [IOPS](../../../virtual-machines/premium-storage-performance.md#iops), le [débit](../../../virtual-machines/premium-storage-performance.md#throughput) et la [latence](../../../virtual-machines/premium-storage-performance.md#latency) de la charge de travail source aux heures de pointe en suivant la [liste de contrôle des performances de l’application](../../../virtual-machines/premium-storage-performance.md#application-performance-requirements-checklist). 

Recueillez des données pendant les heures de pointe, telles que les charges de travail d’une journée de travail ordinaire, mais aussi d’autres processus de charge élevée, tels que le traitement de fin de journée et les charges de travail ETL effectuées pendant le weekend. Envisagez d’augmenter l’échelle de vos ressources pour des charges de travail anormalement lourdes, telles qu’un traitement de fin de trimestre, puis de réduire l’échelle une fois la charge de travail accomplie. 

Utilisez l’analyse des performances pour sélectionner la [taille de machine virtuelle](../../../virtual-machines/sizes-memory.md) pouvant adapter son échelle aux exigences de performances de votre charge de travail.


### <a name="iops-and-throughput"></a>IOPS et débit

Les performances de SQL Server dépendent fortement du sous-système d’E/S. Si votre base de données ne tient pas dans la mémoire physique, SQL Server place en permanence des pages de base de données dans et hors du pool de mémoires tampons. Les fichiers de données pour SQL Server doivent être traités différemment. L’accès aux fichiers journaux est séquentiel, sauf quand une transaction doit être restaurée à l’emplacement où des fichiers de données, dont la base de données TempDB, font l’objet d’un accès aléatoire. Si vous disposez d’un sous-système d’E/S lent, vos utilisateurs peuvent rencontrer des problèmes de performances, tels que des temps de réponse lents et des tâches qui ne se terminent pas en raison de délais d’expiration. 

Les machines virtuelles de la Place de marché Azure ont des fichiers journaux sur un disque physique séparé des fichiers de données par défaut. Le nombre et la taille des fichiers de données de la base de données TempDB sont conformes aux meilleures pratiques et ciblent le lecteur D:/ éphémère. 

Les compteurs PerfMon suivants peuvent vous aider à valider le débit d’E/S que requiert votre SQL Server : 
* **\LogicalDisk\Disk Reads/Sec** (IOPS de lecture et d’écriture)
* **\LogicalDisk\Disk Writes/Sec** (IOPS de lecture et d’écriture) 
* **\LogicalDisk\Disk Bytes/Sec** (débit requis pour les fichiers de données, de journal et de base de données TempDB)

À l’aide des exigences d’IOPS et de débit aux niveaux de pointe, évaluez les tailles de machine virtuelle correspondant à la capacité de vos mesures. 

Si votre charge de travail nécessite 20 000 IOPS de lecture et 10 000 IOPS d’écriture, vous pouvez choisir le modèle E16s_v3 (jusqu’à 32 000 IOPS mises en cache et 25 600 non mises en cache) ou le modèle M16_s (jusqu’à 20 000 IOPS mises en cache et 10 000 non mises en cache) avec 2 disques P30 agrégés par bandes à l’aide d’espaces de stockage. 

Veillez à bien identifier les besoins de la charge de travail en termes de débit et d’IOPS, car les machines virtuelles ont des limites d’échelle différentes pour les IOPS et le débit.

### <a name="memory"></a>Mémoire

Suivez la mémoire externe que le système d’exploitation utilise, ainsi que la mémoire que SQL Server utilise en interne. L’identification de la pression sur chaque composant vous aidera à dimensionner les machines virtuelles et à identifier des opportunités de réglage. 

Les compteurs PerfMon suivants peuvent vous aider à valider l’intégrité de la mémoire d’une machine virtuelle SQL Server : 
* [\Memory\Available MBytes](/azure/monitoring/infrastructure-health/vmhealth-windows/winserver-memory-availmbytes)
* [\SQLServer:Memory Manager\Target Server Memory (Ko)](/sql/relational-databases/performance-monitor/sql-server-buffer-manager-object)
* [\SQLServer:Memory Manager\Total Server Memory (Ko)](/sql/relational-databases/performance-monitor/sql-server-buffer-manager-object)
* [\SQLServer:Buffer Manager\Lazy writes/sec](/sql/relational-databases/performance-monitor/sql-server-buffer-manager-object)
* [\SQLServer:Buffer Manager\Page life expectancy](/sql/relational-databases/performance-monitor/sql-server-buffer-manager-object)

### <a name="compute--processing"></a>Calcul/traitement

Le calcul dans Azure est géré autrement que dans l’environnement local. Les serveurs locaux sont conçus pour durer plusieurs années sans mise à niveau en raison de la surcharge de gestion et du coût d’acquisition de nouveau matériel. La virtualisation atténue certains de ces problèmes mais les applications sont optimisées pour tirer le meilleur parti du matériel sous-jacent, ce qui signifie que toute modification significative de l’utilisation des ressources requiert un rééquilibrage de l’ensemble de l’environnement physique. 

Ce n’est pas un problème dans Azure où une nouvelle machine virtuelle sur une série différente de matériel, voire dans une région différente, est facile à obtenir. 

Dans Azure, vous souhaitez exploiter le plus possible les ressources des machines virtuelles. Par conséquent, les machines virtuelles Azure doivent être configurées pour maintenir l’utilisation moyenne du processeur la plus élevée possible sans que cela ait une incidence sur la charge de travail. 

Les compteurs PerfMon suivants peuvent vous aider à valider l’intégrité de calcul d’une machine virtuelle SQL Server :
* **\Processor Information(_Total)\% Temps processeur**
* **\Process(sqlservr)\% Temps processeur**

> [!NOTE] 
> Dans l’idéal, essayez d’atteindre une utilisation à 80 % de votre capacité de calcul, avec des pics supérieurs à 90 %, mais sans atteindre 100 % pendant une période prolongée. Fondamentalement, vous souhaitez seulement approvisionner la capacité de calcul dont l’application a besoin, puis planifier l’augmentation ou la diminution d’échelle en fonction des besoins de l’entreprise. 

## <a name="next-steps"></a>Étapes suivantes

Pour connaître les meilleures pratiques en matière de sécurité, consultez [Considérations relatives à la sécurité de SQL Server sur les machines virtuelles Azure](security-considerations-best-practices.md).

Consultez d’autres articles relatifs aux machines virtuelles avec SQL Server à la page [Vue d’ensemble de SQL Server sur les machines virtuelles Azure](sql-server-on-azure-vm-iaas-what-is-overview.md). Si vous avez des questions sur les machines virtuelles SQL Server, consultez le [Forum aux Questions](frequently-asked-questions-faq.md).
