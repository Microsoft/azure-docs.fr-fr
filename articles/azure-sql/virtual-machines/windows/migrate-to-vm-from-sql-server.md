---
title: Migrer une base de données SQL Server vers SQL Server sur une machine virtuelle | Microsoft Docs
description: Découvrez comment migrer une base de données utilisateur locale vers SQL Server sur une machine virtuelle Azure.
services: virtual-machines-windows
documentationcenter: ''
author: MashaMSFT
editor: ''
tags: azure-service-management
ms.assetid: 00fd08c6-98fa-4d62-a3b8-ca20aa5246b1
ms.service: virtual-machines-sql
ms.workload: iaas-sql-server
ms.tgt_pltfrm: vm-windows-sql-server
ms.subservice: migration
ms.topic: how-to
ms.date: 08/18/2018
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: 4cd37128893309be5a1e362671b9e28dcc436b1b
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97356206"
---
# <a name="migrate-a-sql-server-database-to-sql-server-on-an-azure-virtual-machine"></a>Migrez une base de données SQL Server dans SQL Server sur une machine virtuelle Azure

[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Il existe plusieurs moyens permettant de migrer une base de données utilisateur SQL Server locale vers SQL Server sur une machine virtuelle Azure. Cet article abordera brièvement différentes méthodes et recommandera la mieux adaptée à différents scénarios.


[!INCLUDE [learn-about-deployment-models](../../../../includes/learn-about-deployment-models-both-include.md)]

  > [!NOTE]
  > SQL Server 2008 et SQL Server 2008 R2 se rapprochent de la [fin de leur cycle de vie de prise en charge](https://www.microsoft.com/sql-server/sql-server-2008) pour les instances locales. Pour étendre la prise en charge, vous pouvez migrer votre instance SQL Server vers une machine virtuelle Azure ou acheter des mises à jour de sécurité étendues pour la conserver en local. Pour plus d’informations, consultez [Étendre la prise en charge de SQL Server 2008 et SQL Server 2008 R2 avec Azure](sql-server-2008-extend-end-of-support.md)

## <a name="what-are-the-primary-migration-methods"></a>Quelles sont les principales méthodes de migration ?

Les méthodes de migration principales consistent à :

* Effectuer une sauvegarde locale en utilisant la compression, puis copier manuellement le fichier de sauvegarde dans la machine virtuelle Azure.
* Effectuer une sauvegarde vers une URL, puis restaurer dans la machine virtuelle Azure depuis l’URL.
* Détacher les fichiers journaux et de données, les copier vers le Stockage Blob Azure, puis les rattacher à SQL Server dans la machine virtuelle Azure depuis l’URL
* Convertir la machine physique locale en disque dur virtuel Hyper-V, effectuer un téléchargement vers le stockage d’objets Blob Azure, puis exécuter un déploiement en tant que nouvelle machine virtuelle en utilisant le disque dur virtuel téléchargé
* Expédier le disque dur à l’aide du Service d’importation/exportation Windows
* Si vous disposez d’un déploiement de groupe de disponibilité AlwaysOn local, utilisez l’[Assistant Ajouter un réplica Azure](/previous-versions/azure/virtual-machines/windows/sqlclassic/virtual-machines-windows-classic-sql-onprem-availability) pour créer un réplica dans Azure, basculez et pointez les utilisateurs vers l’instance de base de données Azure.
* Utiliser la [réplication transactionnelle](/sql/relational-databases/replication/transactional/transactional-replication) de SQL Server pour configurer l’instance Azure SQL Server en tant qu’abonné, désactiver la réplication et pointer les utilisateurs vers l’instance de base de données Azure.

> [!TIP]
> Vous pouvez également utiliser ces mêmes techniques pour déplacer des bases de données entre machines virtuelles SQL Server dans Azure. Par exemple, il n’existe aucun moyen pris en charge de mettre à niveau une machine virtuelle d’image de la galerie SQL Server d’une version/édition à l’autre. Dans ce cas, il vous faut créer une nouvelle machine virtuelle SQL Server avec la nouvelle version/édition, puis utiliser l’une des techniques de migration de cet article pour déplacer vos bases de données. 

## <a name="choose-a-migration-method"></a>Choisir une méthode de migration

Pour obtenir les meilleures performances de transfert de données, migrez les fichiers de base de données de la machine virtuelle Azure à l’aide d’un fichier de sauvegarde compressé.

Pour réduire le temps d’arrêt pendant la migration de base de données, utilisez l’option AlwaysOn ou l’option de réplication transactionnelle.

S’il n’est pas possible d’utiliser les méthodes décrites ci-dessus, migrez manuellement votre base de données. En règle générale, vous effectuez une sauvegarde de la base de données, puis une copie de la sauvegarde de la base de données dans Azure et enfin une restauration de la base de données. Vous pouvez également copier les fichiers de base de données eux-mêmes dans Azure, puis les joindre. Il existe plusieurs méthodes permettant d’effectuer manuellement le processus de migration d’une base de données vers une machine virtuelle Azure.

> [!NOTE]
> Lorsque vous procédez à une mise à niveau vers SQL Server 2014 ou SQL Server 2016 depuis des versions antérieures de SQL Server, vous devez prendre en compte les éventuelles modifications nécessaires. Nous vous recommandons de gérer toutes les dépendances de fonctionnalités non prises en charge par la nouvelle version de SQL Server dans le cadre de votre projet de migration. Pour plus d’informations sur les éditions et les scénarios pris en charge, consultez [Mise à niveau vers SQL Server](/sql/database-engine/install-windows/upgrade-sql-server).

Le tableau suivant répertorie les principales méthodes de migration et explique dans quelles circonstances utiliser chacune d’elles.

| Méthode | Version de base de données source | Version de base de données de destination | Contrainte de taille pour la sauvegarde de base de données source | Notes |
| --- | --- | --- | --- | --- |
| [Effectuer une sauvegarde locale en utilisant la compression et copier manuellement le fichier de sauvegarde dans la machine virtuelle Azure](#back-up-and-restore) |SQL Server 2005 ou ultérieur |SQL Server 2005 ou ultérieur |[Limite de stockage de machine virtuelle Azure](../../../index.yml) | Cette technique est simple et éprouvée pour déplacer des bases de données d’une machine à l’autre. |
| [Effectuer une sauvegarde vers l’URL et une restauration dans la machine virtuelle Azure depuis l’URL](#backup-to-url-and-restore-from-url) |SQL Server 2012 SP1 CU2 ou ultérieur | SQL Server 2012 SP1 CU2 ou ultérieur | < 12,8 To pour SQL Server 2016, sinon < 1 To | Cette méthode représente simplement une autre façon de déplacer le fichier de sauvegarde sur la machine virtuelle à l’aide du stockage Azure. |
| [Détacher, puis copier les données et les fichiers journaux dans Stockage Blob Azure et les rattacher à SQL Server dans la machine virtuelle Azure à partir d’URL](#detach-and-attach-from-a-url) | SQL Server 2005 ou ultérieur |SQL Server 2014 ou ultérieur | [Limite de stockage de machine virtuelle Azure](../../../index.yml) | Utilisez cette méthode quand vous envisagez de [stocker ces fichiers à l’aide du service de stockage d’objets Blob Azure](/sql/relational-databases/databases/sql-server-data-files-in-microsoft-azure) et de les rattacher à SQL Server s’exécutant dans une machine virtuelle Azure, en particulier avec les bases de données très volumineuses. |
| [Convertir l’ordinateur local pour les disques durs virtuels Hyper-V, charger sur le stockage d’objets Blob Azure, puis déployer une nouvelle machine virtuelle en utilisant le disque dur virtuel chargé](#convert-to-a-vm-upload-to-a-url-and-deploy-as-a-new-vm) |SQL Server 2005 ou ultérieur |SQL Server 2005 ou ultérieur |[Limite de stockage de machine virtuelle Azure](../../../index.yml) |À utiliser lors de [la mise en service de votre propre licence SQL Server](../../../azure-sql/azure-sql-iaas-vs-paas-what-is-overview.md), lors de la migration d’une base de données que vous allez exécuter sur une version plus ancienne de SQL Server ou lors de la migration d’un système et de bases de données d’un système utilisateur dans le cadre de la migration de base de données dépendant des autres bases de données utilisateur et/ou bases de données système. |
| [Expédition du disque dur à l’aide du Service d’importation/exportation Windows](#ship-a-hard-drive) |SQL Server 2005 ou ultérieur |SQL Server 2005 ou ultérieur |[Limite de stockage de machine virtuelle Azure](../../../index.yml) |Utilisez le [Service Import/Export de Windows](../../../storage/common/storage-import-export-service.md) lorsque la méthode de copie manuelle est trop lente, par exemple avec de très grandes bases de données. |
| [Utiliser l’Assistant Ajout de réplica Azure](/previous-versions/azure/virtual-machines/windows/sqlclassic/virtual-machines-windows-classic-sql-onprem-availability) |SQL Server 2012 ou ultérieur |SQL Server 2012 ou ultérieur |[Limite de stockage de machine virtuelle Azure](../../../index.yml) |Réduit le temps d’arrêt ; à utiliser lorsque vous disposez d’un déploiement local Always On. |
| [Utiliser la réplication transactionnelle de SQL Server](/sql/relational-databases/replication/transactional/transactional-replication) |SQL Server 2005 ou ultérieur |SQL Server 2005 ou ultérieur |[Limite de stockage de machine virtuelle Azure](../../../index.yml) |À utiliser si vous devez réduire le temps d’arrêt et que vous ne disposez pas d’un déploiement local Always On. |

## <a name="back-up-and-restore"></a>Sauvegarde et restauration

Sauvegardez votre base de données avec compression, copiez la sauvegarde sur la machine virtuelle, puis restaurez la base de données. Si le volume de votre fichier de sauvegarde est supérieur à 1 To, vous devez créer un jeu agrégé car la taille maximale d’un disque de machine virtuelle est de 1 To. Utilisez les étapes générales suivantes pour faire migrer une base de données utilisateur à l’aide de cette méthode manuelle :

1. Effectuez une sauvegarde complète de la base de données vers un emplacement sur site.
2. Créez ou chargez une machine virtuelle avec la version de SQL Server souhaitée.
3. Configurez la connectivité en fonction de vos besoins. Consultez [Se connecter à une machine virtuelle SQL Server sur Azure (Resource Manager)](ways-to-connect-to-sql.md).
4. Copiez vos fichiers de sauvegarde sur votre machine virtuelle à l’aide du Bureau à distance, de l’Explorateur Windows ou de la commande de copie à partir d’une invite de commandes.

## <a name="backup-to-url-and-restore-from-url"></a>Sauvegarde sur URL et restauration à partir de l’URL

Au lieu de sauvegarder dans un fichier local, vous pouvez utiliser la [sauvegarde sur URL](/sql/relational-databases/backup-restore/sql-server-backup-to-url), puis effectuer la restauration sur la machine virtuelle à partir de l’URL. SQL Server 2016 prend en charge les jeux de sauvegarde distribuée. Ils sont recommandés pour les performances et doivent dépasser les limites de taille par objet blob. Pour les bases de données très volumineuses, il est recommandé d’utiliser le [Service d’importation/exportation Windows](../../../storage/common/storage-import-export-service.md) .

## <a name="detach-and-attach-from-a-url"></a>Détacher et attacher à partir d’une URL

Détachez votre base de données et vos fichiers journaux et transférez-les vers le [stockage Blob Azure](/sql/relational-databases/databases/sql-server-data-files-in-microsoft-azure). Puis attachez la base de données sur votre machine virtuelle Azure à partir de l’URL. Utilisez cette méthode si vous souhaitez que les fichiers de la base de données physique résident dans le stockage Blob, ce qui peut être utile pour les bases de données très volumineuses. Utilisez les étapes générales suivantes pour faire migrer une base de données utilisateur à l’aide de cette méthode manuelle :

1. Détachez les fichiers de base de données de l’instance de base de données sur site.
2. Copiez les fichiers de la base de données détachée dans Stockage Blob Azure à l’aide de l’[utilitaire de ligne de commande AZCopy](../../../storage/common/storage-use-azcopy-v10.md).
3. Attachez les fichiers de base de données à partir de l’URL Azure à l’instance de SQL Server dans la machine virtuelle Azure.

## <a name="convert-to-a-vm-upload-to-a-url-and-deploy-as-a-new-vm"></a>Effectuer une conversion vers une machine virtuelle, télécharger vers une URL et déployer en tant que nouvelle machine virtuelle

Cette méthode permet de migrer toutes les bases de données système et utilisateur dans une instance SQL Server locale vers une machine virtuelle Azure. Utilisez les étapes générales suivantes pour exécuter la migration d’une instance de SQL Server entière à l’aide de cette méthode manuelle :

1. Convertir des machines physiques ou virtuelles en disques durs virtuels Hyper-V.
2. Télécharger les fichiers VHD sur Azure Storage à l’aide de l’ [applet de commande Add-AzureVHD](/previous-versions/azure/dn495173(v=azure.100)).
3. Déployer une nouvelle machine virtuelle en utilisant le VHD téléchargé.

> [!NOTE]
> Pour migrer une application entière, envisagez d’utiliser [Azure Site Recovery](../../../site-recovery/site-recovery-overview.md)].

## <a name="ship-a-hard-drive"></a>Expédier un disque dur

Vous pouvez utiliser la [méthode de service Import/Export Azure](../../../storage/common/storage-import-export-service.md) pour transférer de grandes quantités de données de fichiers vers le stockage d’objets blob Azure lorsque le coût de l’opération de téléchargement via le réseau est prohibitif, voire irréalisable. Avec ce service, vous envoyez un ou plusieurs disques durs contenant ces données à un centre de données Azure, où vos données seront téléchargées sur votre compte de stockage.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez [Présentation de SQL Server sur les machines virtuelles Azure](sql-server-on-azure-vm-iaas-what-is-overview.md).

> [!TIP]
> Si vous avez des questions sur les machines virtuelles SQL Server, consultez le [Forum aux Questions](frequently-asked-questions-faq.md).

Pour obtenir des instructions sur la création d’une instance SQL Server sur une machine virtuelle Azure à partir d’une image capturée, consultez la page [Tips & Tricks on ‘cloning’ Azure SQL virtual machines from captured images](/archive/blogs/psssql/tips-tricks-on-cloning-azure-sql-virtual-machines-from-captured-images) (Trucs et astuces sur le « clonage » des machines virtuelles Azure SQL à partir d’images capturées) du blog des ingénieurs CSS SQL Server.