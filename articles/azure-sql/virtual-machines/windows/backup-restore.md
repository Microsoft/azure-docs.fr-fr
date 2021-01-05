---
title: Sauvegarde et restauration de SQL Server sur des machines virtuelles Azure | Microsoft Docs
description: Décrit les considérations relatives à la sauvegarde et à la restauration des bases de données SQL Server s’exécutant sur des machines virtuelles Azure.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
editor: ''
tags: azure-resource-management
ms.assetid: 95a89072-0edf-49b5-88ed-584891c0e066
ms.service: virtual-machines-sql
ms.subservice: backup
ms.topic: conceptual
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 06/04/2018
ms.author: mikeray
ms.openlocfilehash: 2fcba81bcd20db321d791fcda589f40fb0699702
ms.sourcegitcommit: a89a517622a3886b3a44ed42839d41a301c786e0
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/22/2020
ms.locfileid: "97733071"
---
# <a name="backup-and-restore-for-sql-server-on-azure-vms"></a>Sauvegarde et restauration pour SQL Server sur des machines virtuelles Azure
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Cet article fournit des conseils sur les options de sauvegarde et de restauration disponibles pour SQL Server s’exécutant sur une machine virtuelle Windows dans Azure. Stockage Azure conserve trois copies de chaque disque de machine virtuelle Azure pour garantir la protection contre la perte de données ou l’altération physique des données. Par conséquent, contrairement aux configurations locales de SQL Server, vous n’avez pas besoin de vous occuper des défaillances matérielles. Toutefois, vous devez toujours sauvegarder vos bases de données SQL Server pour les protéger des erreurs d’application ou d’utilisateur, telles que les insertions ou les suppressions de données effectuées par inadvertance. Dans ce cas, il est important de pouvoir procéder à une restauration à un point donné dans le temps.

La première partie de cet article présente les options de restauration et de sauvegarde disponibles. Elle est suivie de sections qui contiennent des informations supplémentaires sur chaque stratégie.

## <a name="backup-and-restore-options"></a>Options de sauvegarde et de restauration

Le tableau suivant contient des informations sur les différentes options de sauvegarde et de restauration pour SQL Server sur des machines virtuelles Azure :

| Stratégie | Versions de SQL | Description |
|---|---|---|
| [Sauvegarde automatisée](#automated) | 2014<br/> 2016<br/> 2017 | La sauvegarde automatisée vous permet de planifier des sauvegardes régulières pour toutes les bases de données d’une machine virtuelle SQL Server. Les sauvegardes sont stockées dans Stockage Azure pendant 30 jours maximum. Depuis SQL Server 2016, Sauvegarde automatisée v2 offre des options supplémentaires telles que la configuration d’une planification manuelle et la fréquence des sauvegardes complètes et des sauvegardes de fichiers journaux. |
| [Sauvegarde Azure pour machines virtuelles SQL](#azbackup) | 2008<br/> 2012<br/> 2014<br/> 2016<br/> 2017<br/> 2019 | Sauvegarde Azure fournit une fonctionnalité de sauvegarde Entreprise pour SQL Server sur des machines virtuelles Azure. Avec ce service, vous pouvez gérer de manière centralisée les sauvegardes de plusieurs serveurs et de milliers de bases de données. Les bases de données peuvent être restaurées à un point spécifique dans le temps dans le portail. Ce service propose une stratégie de conservation personnalisable qui peut conserver des sauvegardes pendant des années. |
| [Sauvegarde manuelle](#manual) | Tous | En fonction de votre version de SQL Server, il existe différentes techniques permettant de sauvegarder et de restaurer manuellement SQL Server sur une machine virtuelle Azure. Dans ce scénario, vous êtes chargé du mode de sauvegarde de vos bases de données ainsi que de l’emplacement et la gestion du stockage de ces sauvegardes. |

Les sections suivantes décrivent plus en détail chacune de ces options. La dernière section de cet article contient un résumé sous la forme d’une matrice des fonctionnalités.

## <a name="automated-backup"></a><a id="automated"></a> Sauvegarde automatisée

La sauvegarde automatisée fournit un service de sauvegarde automatique pour les éditions SQL Server Standard et Entreprise s’exécutant sur une machine virtuelle Windows dans Azure. Ce service est fourni par [l’extension SQL Server IaaS Agent](sql-server-iaas-agent-extension-automate-management.md), qui est automatiquement installée sur les images de machine virtuelle Windows SQL Server dans le portail Azure.

Toutes les bases de données sont sauvegardées sur un compte de stockage Azure que vous configurez. Les sauvegardes peuvent être chiffrées et conservées pendant 30 jours.

Les machines virtuelles SQL Server versions 2016 et supérieures proposent des options de personnalisation supplémentaires avec Sauvegarde automatisée v2. Ces améliorations sont notamment les suivantes :

- Sauvegardes de bases de données système
- Fenêtre de temps et planification de sauvegarde manuelle
- Fréquence des sauvegardes complètes et de fichier journal

Pour restaurer une base de données, vous devez localiser le ou les fichiers de sauvegarde requis dans le compte de stockage, puis effectuer une restauration sur votre machine virtuelle SQL à l’aide de SQL Server Management Studio (SSMS) ou de commandes Transact-SQL.

Pour plus d’informations sur la configuration de la sauvegarde automatisée des machines virtuelles SQL, consultez les articles suivants :

- **SQL Server 2016/2017** : [Sauvegarde automatisée version 2 pour les machines virtuelles Azure](automated-backup.md)
- **SQL Server 2014** : [Sauvegarde automatisée pour les machines virtuelles SQL Server 2014](automated-backup-sql-2014.md)

## <a name="azure-backup-for-sql-vms"></a><a id="azbackup"></a> Sauvegarde Azure pour les machines virtuelles SQL

[Sauvegarde Azure](../../../backup/index.yml) fournit une fonctionnalité de sauvegarde Entreprise pour SQL Server sur des machines virtuelles Azure. Toutes les sauvegardes sont stockées et gérées dans un coffre Recovery Services. Cette solution apporte plusieurs avantages, en particulier pour les entreprises :

- **Sauvegarde sans infrastructure** : vous n’avez pas à gérer les serveurs de sauvegarde ni les emplacements de stockage.
- **Mise à l’échelle** : assurez la protection de nombreuses machines virtuelles SQL et de milliers de bases de données.
- **Paiement à l’utilisation** : cette fonctionnalité est un service distinct fourni par Sauvegarde Azure, mais comme pour tous les services Azure, vous ne payez que ce que vous utilisez.
- **Surveillance et gestion centralisées** : gérez de manière centralisée toutes vos sauvegardes, y compris les autres charges de travail prises en charge par Sauvegarde Azure, depuis un tableau de bord unique d’Azure.
- **Rétention et sauvegarde basées sur des stratégies** : créez des stratégies de sauvegarde standard pour procéder à des sauvegardes régulières. Établissez des stratégies de conservation pour gérer des sauvegardes pendant des années.
- **Prise en charge de SQL Always On** : détectez et protégez une configuration de SQL Server Always On et respectez la préférence de sauvegarde du groupe de disponibilité des sauvegardes.
- **Objectif de point de récupération (RPO) de 15 minutes** : configurez des sauvegardes de fichiers journaux de transactions SQL toutes les 15 minutes.
- **Limite de restauration dans le temps** : utilisez le portail pour récupérer des bases de données à un point précis dans le temps sans avoir à restaurer manuellement plusieurs sauvegardes complètes, différentielles et de fichiers journaux.
- **Alertes par e-mail consolidées pour informer des défaillances** : configurez des notifications par e-mail consolidées pour informer des défaillances.
- **Contrôle d’accès en fonction du rôle Azure** : déterminez qui peut gérer les opérations de sauvegarde et de restauration via le portail.

Pour une vue d’ensemble rapide de son fonctionnement et une démonstration, regardez la vidéo suivante :

> [!VIDEO https://www.youtube.com/embed/wmbANpHos_E]

Cette solution Sauvegarde Azure pour les machines virtuelles SQL est mise à la disposition générale. Pour plus d’informations, consultez [Sauvegarder une base de données SQL Server dans Azure](../../../backup/backup-azure-sql-database.md).

## <a name="manual-backup"></a><a id="manual"></a> Sauvegarde manuelle

Si vous souhaitez gérer manuellement les opérations de sauvegarde et de restauration sur vos machines virtuelles SQL, plusieurs options sont à votre disposition selon la version de SQL Server que vous utilisez. Pour une vue d’ensemble de la sauvegarde et de la restauration, consultez les articles suivants en fonction de votre version de SQL Server :

- [Sauvegarde et restauration pour SQL Server 2016 et versions ultérieures](/sql/relational-databases/backup-restore/back-up-and-restore-of-sql-server-databases)
- [Sauvegarde et restauration pour SQL Server 2014](/sql/relational-databases/backup-restore/back-up-and-restore-of-sql-server-databases?viewFallbackFrom=sql-server-2014)
- [Sauvegarde et restauration pour SQL Server 2012](/previous-versions/sql/sql-server-2012/ms187048(v=sql.110))
- [Sauvegarde et restauration pour SQL Server 2008 R2](/previous-versions/sql/sql-server-2008-r2/ms187048(v=sql.105))
- [Sauvegarde et restauration pour SQL Server 2008](/previous-versions/sql/sql-server-2008/ms187048(v=sql.100))

Les sections suivantes décrivent de façon plus détaillée plusieurs options de sauvegarde et de restauration manuelles.

### <a name="backup-to-attached-disks"></a>Sauvegarde sur des disques attachés

Pour SQL Server sur des machines virtuelles Azure, vous pouvez utiliser les techniques de sauvegarde et restauration en mode natif en utilisant des disques attachés sur la machine virtuelle comme destination des fichiers de sauvegarde. Néanmoins, le nombre de disques que vous pouvez attacher à une machine virtuelle Azure est limité et basé sur la [taille de la machine virtuelle](../../../virtual-machines/sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json). Il convient également de tenir compte du traitement de la gestion des disques.

Pour obtenir un exemple de création manuelle d’une sauvegarde complète de base de données à l’aide de SQL Server Management Studio (SSMS) ou de Transact-SQL, consultez [Créer une sauvegarde complète de base de données](/sql/relational-databases/backup-restore/create-a-full-database-backup-sql-server).

### <a name="backup-to-url"></a>Sauvegarde vers une URL

Depuis SQL Server 2012 SP1 CU2, vous pouvez effectuer des sauvegardes et restaurations directement vers le stockage d’objets blob Microsoft Azure. Cette procédure directe est également appelée sauvegarde vers une URL. SQL Server 2016 contient également les améliorations suivantes pour cette fonctionnalité :

| Améliorations de la version 2016 | Détails |
| --- | --- |
| **Entrelacement** |Lors de la sauvegarde dans le stockage d’objets blob Microsoft Azure, SQL Server 2016 prend en charge la sauvegarde dans plusieurs objets blob, ce qui permet de sauvegarder des bases de données volumineuses allant jusqu’à 12,8 To. |
| **Sauvegarde instantanée** |Avec l’utilisation d’instantanés Azure, la fonctionnalité Sauvegarde instantanée de fichier SQL Server fournit des sauvegardes quasi instantanées et des restaurations rapides des fichiers de base de données stockés à l’aide du service de stockage d’objets blob Azure. Cette fonctionnalité vous permet de simplifier vos stratégies de sauvegarde et de restauration. La sauvegarde instantanée de fichier prend également en charge la limite de restauration dans le temps. Pour plus d'informations, consultez [File-Snapshot Backups for Database Files in Azure](/sql/relational-databases/backup-restore/file-snapshot-backups-for-database-files-in-azure)(en anglais). |

Pour plus d’informations, consultez les articles suivants en fonction de votre version de SQL Server :

- **SQL Server 2016/2017** : [Sauvegarde SQL Server vers une URL](/sql/relational-databases/backup-restore/sql-server-backup-and-restore-with-microsoft-azure-blob-storage-service)
- **SQL Server 2014** : [Sauvegarde SQL Server 2014 vers une URL](/sql/relational-databases/backup-restore/sql-server-backup-and-restore-with-microsoft-azure-blob-storage-service?viewFallbackFrom=sql-server-2014)
- **SQL Server 2012** : [Sauvegarde SQL Server 2012 vers une URL](/previous-versions/sql/sql-server-2012/jj919148(v=sql.110))

### <a name="managed-backup"></a>Sauvegarde managée

Depuis SQL Server 2014, la fonctionnalité Sauvegarde managée automatise la création de sauvegardes dans le stockage Azure. En arrière-plan, elle utilise la fonctionnalité Sauvegarde vers une URL décrite dans la section précédente de cet article. La fonctionnalité Sauvegarde managée est également la fonctionnalité sous-jacente qui prend en charge le service Sauvegarde automatisée sur les machines virtuelles SQL Server.

Depuis SQL Server 2016, la fonctionnalité Sauvegarde managée comporte des options supplémentaires pour la planification, la sauvegarde de bases de données système, ainsi que la fréquence des sauvegardes complètes et de fichiers journaux.

Pour plus d’informations, consultez les articles suivants en fonction de votre version de SQL Server :

- [Managed Backup to Microsoft Azure for SQL Server 2016 and later](/sql/relational-databases/backup-restore/sql-server-managed-backup-to-microsoft-azure) (Sauvegarde managée dans Microsoft Azure pour SQL Server 2016 et ultérieur)
- [Managed Backup to Microsoft Azure for SQL Server 2014](/sql/relational-databases/backup-restore/sql-server-managed-backup-to-microsoft-azure?viewFallbackFrom=sql-server-2014) (Sauvegarde managée dans Microsoft Azure pour SQL Server 2014)

## <a name="decision-matrix"></a>Matrice décisionnelle

Le tableau ci-dessous récapitule les fonctionnalités de chaque option de sauvegarde et de restauration pour les machines virtuelles SQL Server dans Azure.

| Option | Sauvegarde automatisée | Sauvegarde Azure pour SQL | Sauvegarde manuelle |
|---|---|---|---|
| Service Azure supplémentaire nécessaire |   | ![Coche verte.](./media/backup-restore/yes.png) |   |
| Configuration d’une stratégie de sauvegarde dans le portail Azure | ![Coche verte.](./media/backup-restore/yes.png) | ![Coche verte.](./media/backup-restore/yes.png) |   |
| Restauration des bases de données dans le portail Azure |   | ![Coche verte.](./media/backup-restore/yes.png) |   |
| Gestion de plusieurs serveurs dans un tableau de bord |   | ![Coche verte.](./media/backup-restore/yes.png) |   |
| Restauration dans le temps | ![Coche verte.](./media/backup-restore/yes.png) | ![Coche verte.](./media/backup-restore/yes.png) | ![Coche verte.](./media/backup-restore/yes.png) |
| Objectif de point de récupération (RPO) de 15 minutes | ![Coche verte.](./media/backup-restore/yes.png) | ![Coche verte.](./media/backup-restore/yes.png) | ![Coche verte.](./media/backup-restore/yes.png) |
| Stratégie de conservation de sauvegarde à court terme (jours) | ![Coche verte.](./media/backup-restore/yes.png) | ![Coche verte.](./media/backup-restore/yes.png) |   |
| Stratégie de conservation de sauvegarde à long terme (mois, années) |   | ![Coche verte.](./media/backup-restore/yes.png) |   |
| Prise en charge intégrée de SQL Server Always On |   | ![Coche verte.](./media/backup-restore/yes.png) |   |
| Sauvegarde vers des comptes de stockage Azure | ![Coche verte.](./media/backup-restore/yes.png)(automatique) | ![Coche verte.](./media/backup-restore/yes.png)(automatique) | ![Coche verte.](./media/backup-restore/yes.png)(gérée par le client) |
| Gestion des fichiers de sauvegarde et de stockage | | ![Coche verte.](./media/backup-restore/yes.png) |  |
| Sauvegarde sur des disques attachés sur la machine virtuelle |   |   | ![Coche verte.](./media/backup-restore/yes.png) |
| Rapports de sauvegarde personnalisables établis de façon centralisée |   | ![Coche verte.](./media/backup-restore/yes.png) |   |
| Alertes par e-mail consolidées pour informer des défaillances |   | ![Coche verte.](./media/backup-restore/yes.png) |   |
| Personnaliser la supervision basée sur les journaux Azure Monitor |   | ![Coche verte.](./media/backup-restore/yes.png) |   |
| Surveillance des travaux de sauvegarde avec SSMS ou des scripts Transact-SQL | ![Coche verte.](./media/backup-restore/yes.png) | ![Coche verte.](./media/backup-restore/yes.png) | ![Coche verte.](./media/backup-restore/yes.png) |
| Restauration des bases de données avec SSMS ou des scripts Transact-SQL | ![Coche verte.](./media/backup-restore/yes.png) |   | ![Coche verte.](./media/backup-restore/yes.png) |

## <a name="next-steps"></a>Étapes suivantes

Si vous planifiez le déploiement de SQL Server sur une machine virtuelle Azure, vous trouverez des conseils de provisionnement dans le guide suivant : [Guide pratique pour provisionner une machine virtuelle Windows SQL Server dans le portail Azure](create-sql-vm-portal.md).

Même si la sauvegarde et la restauration permettent de migrer vos données, il existe des chemins de migration de données potentiellement plus simples vers SQL Server sur une machine virtuelle. Pour une description complète des options de migration et des recommandations connexes, consultez [Migration d’une base de données vers SQL Server sur une machine virtuelle Azure](migrate-to-vm-from-sql-server.md).
