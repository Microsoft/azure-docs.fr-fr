---
title: Guide pratique pour utiliser le Stockage Azure pour la sauvegarde et la restauration de SQL Server | Microsoft Docs
description: Découvrez comment sauvegarder SQL Server avec Azure Storage. Explique les avantages de la sauvegarde des bases de données SQL sur le stockage Azure.
services: virtual-machines-windows
documentationcenter: ''
author: MikeRayMSFT
tags: azure-service-management
ms.assetid: 0db7667d-ef63-4e2b-bd4d-574802090f8b
ms.service: virtual-machines-sql
ms.subservice: backup
ms.topic: conceptual
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 01/31/2017
ms.author: mathoma
ms.openlocfilehash: b4100800385792557358d3fb6438f52650483f89
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97359788"
---
# <a name="use-azure-storage-for-sql-server-backup-and-restore"></a>Utiliser le Stockage Azure pour la sauvegarde et la restauration de SQL Server
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

À partir de SQL Server 2012 SP1 CU2, vous pouvez écrire des sauvegardes SQL Server directement dans le stockage Blob Azure. Vous pouvez utiliser cette fonctionnalité pour effectuer des sauvegardes et des restaurations à partir du stockage Blob Azure et d’une base de données SQL Server. La sauvegarde dans le cloud offre les avantages de la disponibilité, du stockage hors site géorépliqué sans limite et de la facilité de migration des données vers et depuis le cloud. Vous pouvez émettre des instructions BACKUP ou RESTORE en utilisant Transact-SQL ou SMO.

## <a name="overview"></a>Vue d’ensemble
SQL Server 2016 introduit de nouvelles fonctionnalités ; Vous pouvez utiliser la [sauvegarde instantanée de fichier](/sql/relational-databases/backup-restore/file-snapshot-backups-for-database-files-in-azure) pour effectuer des sauvegardes quasi instantanées et des restaurations extrêmement rapides.

Cette rubrique explique pourquoi vous pourriez préférer utiliser le Stockage Azure pour les sauvegardes SQL Server, puis décrit les composants impliqués. Vous pouvez utiliser les ressources fournies à la fin de l’article pour accéder aux procédures pas à pas et à des informations supplémentaires pour utiliser ce service avec vos sauvegardes SQL Server.

## <a name="benefits-of-using-azure-blob-storage-for-sql-server-backups"></a>Avantages de l’utilisation du stockage Blob Azure pour les sauvegardes SQL Server
La sauvegarde de SQL Server pose diverses difficultés. Ces défis comprennent la gestion du stockage, le risque de défaillance du stockage, l’accès à un stockage hors site et la configuration matérielle. La plupart de ces difficultés peuvent être résolues avec le stockage Blob Azure pour les sauvegardes SQL Server. Considérez les avantages suivants :

* **Simplicité d’utilisation** : Le stockage de vos sauvegardes dans des objets blob Azure peut représenter une option hors site pratique, flexible et facile d’accès. La création d’un stockage hors site pour vos sauvegardes SQL Server peut être aussi facile que la modification de vos scripts/tâches existants pour utiliser la syntaxe **BACKUP TO URL** . En principe, le stockage hors site doit être suffisamment éloigné de l'emplacement de la base de données de production pour empêcher qu'un éventuel sinistre n'affecte tous les emplacements (bases de données hors site et de production). En choisissant de [répliquer géographiquement vos objets blob Azure](../../../storage/common/storage-redundancy.md), vous obtenez une couche de protection supplémentaire en cas de sinistre susceptible d’affecter l’ensemble de la région.
* **Archive de sauvegarde** : Le stockage Blob Azure offre une meilleure alternative à l’option sur bande souvent utilisée pour archiver les sauvegardes. Le stockage sur bande peut nécessiter le transport physique vers un emplacement hors site, ainsi que des mesures de protection des supports. Le stockage de vos sauvegardes dans le stockage Blob Azure offre une option d’archivage instantané, hautement disponible et durable.
* **Matériel géré** : Il n’y a aucun frais supplémentaires de gestion du matériel avec les services Azure. Ces derniers gèrent le matériel et fournissent une géo-réplication à des fins de redondance et de protection contre les défaillances matérielles.
* **Stockage illimité** : En activant une sauvegarde directe dans des objets blob Azure, vous avez accès à un stockage quasi illimité. De façon alternative, la sauvegarde sur un disque de machine virtuelle Azure présente des limites basées sur la taille de la machine. Le nombre de disques que vous pouvez attacher à une machine virtuelle Azure est limité pour les sauvegardes. Cette limite est de 16 disques pour une instance extra-large et inférieure pour les instances plus petites.
* **Disponibilité des sauvegardes** : Les sauvegardes stockées dans des objets blob Azure sont disponibles à partir de n’importe quel endroit, à tout moment et sont facilement accessibles pour les restaurations sur une instance SQL Server, sans avoir à attacher/détacher la base de données ni télécharger et attacher le disque dur virtuel.
* **Coût** : payez uniquement pour le service utilisé. Peut être économique comme option de sauvegarde et d'archivage hors site. Pour plus d’informations, consultez les pages [Calculatrice de prix Azure](https://go.microsoft.com/fwlink/?LinkId=277060 "Calculatrice de prix") et l’[article sur les prix Azure](https://go.microsoft.com/fwlink/?LinkId=277059 "Article sur les prix").
* **Captures instantanées du stockage** : Quand les fichiers de base de données sont stockés dans un objet blob Azure et que vous utilisez SQL Server 2016, vous pouvez utiliser la [sauvegarde d’instantanés de fichier](/sql/relational-databases/backup-restore/file-snapshot-backups-for-database-files-in-azure) pour effectuer des sauvegardes quasi instantanées et des restaurations extrêmement rapides.

Pour plus d’informations, consultez [Sauvegarde et restauration SQL Server avec le stockage Blob Azure](/sql/relational-databases/backup-restore/sql-server-backup-and-restore-with-microsoft-azure-blob-storage-service).

Les deux sections suivantes présentent le stockage Blob Azure, notamment les composants SQL Server requis. Il est important de comprendre les composants et leur interaction pour réaliser une sauvegarde ou une restauration depuis le stockage Blob Azure.

## <a name="azure-blob-storage-components"></a>Composants du stockage Blob Azure
Les composants Azure suivants sont utilisés pour les sauvegardes sur le stockage Blob Azure.

| Composant | Description |
| --- | --- |
| **Compte de stockage** |Le compte de stockage constitue le point de départ de tous les services de stockage. Pour accéder au stockage Blob Azure, commencez par créer un compte de stockage Azure. Pour plus d’informations sur le stockage Blob Azure, consultez [Utilisation du stockage Blob Azure](https://azure.microsoft.com/develop/net/how-to-guides/blob-storage/). |
| **Conteneur** |Un conteneur fournit un regroupement contenant un nombre illimité d’objets blob. Pour écrire une sauvegarde SQL Server sur un stockage Blob Azure, au moins un conteneur racine doit être créé. |
| **Objet blob** |Fichier de tout type et de toute taille. Les objets blob sont adressables à l’aide du format d’URL suivant : **https:\\[compte stockage].blob.core.windows.net/[conteneur]/[blob]** . Pour plus d’informations sur les blobs, consultez [Présentation des objets blob de blocs et des objets blob de pages](/rest/api/storageservices/Understanding-Block-Blobs--Append-Blobs--and-Page-Blobs) |

## <a name="sql-server-components"></a>Composants SQL Server
Les composants SQL Server suivants sont utilisés pour les sauvegardes sur le stockage Blob Azure.

| Composant | Description |
| --- | --- |
| **URL** |Une URL spécifie un URI (Uniform Resource Identifier) pour un fichier de sauvegarde unique. L’URL fournit l’emplacement et le nom du fichier de sauvegarde SQL Server. L’URL doit pointer vers un objet blob réel et pas un simple conteneur. Si l’objet blob n’existe pas, il est créé. Si un objet blob existant est indiqué, BACKUP échoue, sauf si l’option > WITH FORMAT est indiquée. Voici un exemple d’URL que vous pouvez spécifier dans la commande BACKUP : **http[s]://[comptestockage].blob.core.windows.net/[conteneur]/[NOMFICHIER.bak]** . HTTPS n’est pas requis, mais recommandé. |
| **Informations d'identification** |Les informations requises pour se connecter au stockage Blob Azure et pour s’y authentifier sont stockées sous la forme d’informations d’identification. Vous devez créer des informations d'identification SQL Server afin que ce dernier écrive les sauvegardes sur un service BLOB Azure ou les restaure depuis celui-ci. Pour plus d’informations, consultez [Informations d’identification SQL Server](/sql/t-sql/statements/create-credential-transact-sql). |

> [!NOTE]
> SQL Server 2016 a été mis à jour pour prendre en charge les objets blob de blocs. Consultez [Tutoriel : Utilisation du stockage Blob Microsoft Azure avec des bases de données SQL Server 2016](/sql/relational-databases/tutorial-use-azure-blob-storage-service-with-sql-server-2016) pour plus d’informations.
> 
> 

## <a name="next-steps"></a>Étapes suivantes
1. Création d’un compte Azure si vous n’en avez pas encore. Si vous évaluez Azure, envisagez l’ [essai gratuit](https://azure.microsoft.com/free/).
2. Consultez ensuite un des didacticiels suivants pour créer un compte de stockage et effectuez une restauration.
   
   * **SQL Server 2014** : [Tutoriel : Sauvegarde et restauration SQL Server 2014 sur le stockage Blob Microsoft Azure](https://msdn.microsoft.com/library/jj720558\(v=sql.120\).aspx).
   * **SQL Server 2016** : [Tutoriel : Utilisation du stockage Blob Microsoft Azure avec des bases de données SQL Server 2016](/sql/relational-databases/tutorial-use-azure-blob-storage-service-with-sql-server-2016)
3. Continuez à consulter la documentation en commençant avec [Sauvegarde et restauration SQL Server avec le stockage Blob Microsoft Azure](/sql/relational-databases/backup-restore/sql-server-backup-and-restore-with-microsoft-azure-blob-storage-service).

Si vous rencontrez des problèmes, consultez la rubrique [Meilleures pratiques et résolution des problèmes pour la sauvegarde de SQL Server vers une URL](/sql/relational-databases/backup-restore/sql-server-backup-to-url-best-practices-and-troubleshooting).

Pour les autres options de sauvegarde et de restauration de SQL Server, consultez [Sauvegarde et restauration de SQL Server sur les machines virtuelles Azure](backup-restore.md).