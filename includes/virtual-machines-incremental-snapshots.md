---
title: Fichier Include
description: Fichier Include
services: storage
author: roygara
ms.service: storage
ms.topic: include
ms.date: 09/15/2018
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: a662598efdca05769c7da9fbeecdf692dccdacb5
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96028129"
---
## <a name="overview"></a>Vue d’ensemble
Azure Storage offre la possibilité de prendre des instantanés d’objets blob. Les instantanés capturent l’état de l’objet blob à un instant précis. Dans cet article, nous décrivons un scénario dans lequel vous pouvez gérer des sauvegardes de disques de machine virtuelle à l’aide de captures instantanées. Vous pouvez utiliser cette méthode comme une alternative à Azure Backup et Recovery Service et si vous voulez créer une stratégie de sauvegarde personnalisée pour vos disques de machine virtuelle. Pour les machines virtuelles exécutant des charges de travail métier ou stratégiques, nous vous recommandons d’utiliser [Sauvegarde Azure](../articles/backup/backup-azure-vms-introduction.md) dans le cadre de la stratégie de sauvegarde.  

Les disques de machine virtuelle Azure sont stockés en tant qu’objets blob de pages dans Azure Storage. Dans cet article, nous décrivons une stratégie de sauvegarde pour les disques de machine virtuelle. Nous nous référons donc aux captures instantanées dans le contexte d’objets blob de pages. Pour plus d’informations sur les instantanés, reportez-vous à [Création d’un instantané d’objet blob](/rest/api/storageservices/Creating-a-Snapshot-of-a-Blob).

## <a name="what-is-a-snapshot"></a>Qu’est-ce qu’un instantané ?
Un instantané d’objet blob est une version en lecture seule d’un objet blob capturé à un instant donné. Un instantané peut être lu, copié ou supprimé, mais pas modifié. Les instantanés sont une façon de sauvegarder un objet blob à un instant T. La copie des captures instantanées complètes était possible jusqu’à la version REST 2015-04-05. À partir de la version REST 2015-07-08, vous pouvez également copier des instantanés incrémentiels.

## <a name="full-snapshot-copy"></a>Copie d’un instantané complet
Les instantanés peuvent être copiés vers un autre compte de stockage en tant qu’objet blob pour conserver des sauvegardes de l’objet blob de base. Vous pouvez également copier un instantané sur l’objet blob de base, ce qui revient à restaurer l’objet blob à une version antérieure. Lorsqu’une capture instantanée est copiée d’un compte de stockage à un autre, elle occupe le même espace que l’objet blob de pages de base. Par conséquent, la copie de captures instantanées complètes d’un compte de stockage à un autre est lente et consomme beaucoup d’espace dans le compte de stockage cible.

> [!NOTE]
> Si vous copiez l’objet blob de base vers une autre destination, les instantanés de l’objet blob ne sont pas copiés en même temps. De même, si vous remplacez un blob de base par une copie, les captures instantanées associées au blob de base ne sont pas affectées et restent intactes sous le nom du blob de base.
> 
> 

### <a name="back-up-disks-using-snapshots"></a>Sauvegarder des disques à l’aide d’instantanés
Comme stratégie de sauvegarde pour vos disques de machine virtuelle, vous pouvez prendre des instantanés périodiques du disque ou de l’objet blob de pages, et les copier dans un autre compte de stockage à l’aide d’outils tels que l’opération [Copy Blob](/rest/api/storageservices/Copy-Blob) ou [AzCopy](../articles/storage/common/storage-use-azcopy-v10.md). Vous pouvez copier un instantané sur un objet blob de pages de destination avec un nom différent. L’objet blob de pages de destination qui en découle est modifiable et n’est pas un instantané. Plus loin dans cet article, nous décrirons les étapes permettant d’effectuer des sauvegardes de disques de machine virtuelle à l’aide de captures instantanées.

### <a name="restore-disks-using-snapshots"></a>Restaurer des disques à l’aide d’instantanés
Lorsqu’il est temps de restaurer votre disque vers une version stable ayant été précédemment capturée dans une des captures instantanées de sauvegarde, vous pouvez copier une capture instantanée sur l’objet blob de pages de base. Une fois que l’instantané est copié sur l’objet blob de pages de base, il reste, mais sa source est remplacée par une copie qui peut être lue et modifiée. Plus loin dans cet article, nous allons décrire les étapes permettant de restaurer une version précédente de votre disque à partir de sa capture instantanée.

### <a name="implementing-full-snapshot-copy"></a>Implémentation de la copie d’un instantané complet
Vous pouvez implémenter la copie d’un instantané complet de la manière suivante :

* Tout d’abord, prenez un instantané de l’objet blob de base à l’aide de l’opération [Snapshot Blob](/rest/api/storageservices/Snapshot-Blob) .
* Ensuite, copiez l’instantané vers un compte de stockage cible à l’aide de [Copy Blob](/rest/api/storageservices/Copy-Blob).
* Répétez cette procédure pour mettre à jour les copies de sauvegarde de votre objet blob de base.

## <a name="incremental-snapshot-copy"></a>Copie d’un instantané incrémentiel
La nouvelle fonctionnalité incluse dans l’API [GetPageRanges](/rest/api/storageservices/Get-Page-Ranges) offre une bien meilleure méthode de sauvegarde des captures instantanées de vos objets blob de pages ou de vos disques. L’API retourne la liste des modifications entre le blob de base et les captures instantanées, ce qui réduit la quantité d’espace de stockage utilisée sur le compte de sauvegarde. L’API prend en charge les objets blob de pages sur le Stockage Premium et le Stockage Standard. À l’aide de cette API, vous pouvez créer des solutions de sauvegarde plus rapides et efficaces pour les machines virtuelles Azure. Cette API sera disponible avec la version REST 2015-07-08 et les versions supérieures.

La copie d’instantané incrémentiel vous permet de copier d’un compte de stockage à un autre la différence entre

* l’objet blob de base et son instantané OU
* les deux instantanés de l’objet blob de base

Si les conditions suivantes sont respectées,

* L’objet blob a été créé le 1er janvier 2016 ou ultérieurement.
* L’objet blob n’a pas été remplacé avec [PutPage](/rest/api/storageservices/Put-Page) ou [Copy Blob](/rest/api/storageservices/Copy-Blob) entre deux instantanés.

>[!NOTE]
>cette fonctionnalité est disponible pour les objets blob de pages Azure Standard et Premium.

Lorsque vous disposez d’une stratégie de sauvegarde personnalisée à l’aide de captures instantanées, la copie des captures instantanées d’un compte de stockage à un autre peut être lente et consommer beaucoup d’espace de stockage. Au lieu de copier l’instantané en entier sur un compte de stockage de sauvegarde, vous pouvez écrire la différence entre des instantanés consécutifs dans un objet blob de pages de sauvegarde. De cette façon, le temps de copie et l’espace de stockage des sauvegardes sont sensiblement réduits.

### <a name="implementing-incremental-snapshot-copy"></a>Implémentation de la copie d’un instantané incrémentiel
Vous pouvez implémenter la copie d’un instantané incrémentiel de la manière suivante :

* Prenez un instantané de l’objet blob de base à l’aide de [Snapshot Blob](/rest/api/storageservices/Snapshot-Blob).
* Copiez l’instantané vers le compte de stockage de sauvegarde cible dans la même région Azure ou dans une autre région à l’aide de l’opération [Copy Blob](/rest/api/storageservices/Copy-Blob). Il s’agit de l’objet blob de pages de sauvegarde. Prenez une capture instantanée de l’objet blob de pages de sauvegarde et stockez-la dans le compte de stockage de sauvegarde.
* Prenez un autre instantané de l’objet blob de base à l’aide de Snapshot Blob.
* Comparez la première et la deuxième capture instantanée du blob de base à l’aide de [GetPageRanges](/rest/api/storageservices/Get-Page-Ranges). Utilisez le nouveau paramètre **prevsnapshot** pour spécifier la valeur DateTime de la capture instantanée à comparer. Quand ce paramètre est présent, la réponse REST n’inclut que les pages ayant été modifié entre la capture instantanée cible et la capture instantanée précédente, y compris les pages effacées.
* Utilisez [PutPage](/rest/api/storageservices/Put-Page) pour appliquer ces modifications à l’objet blob de pages de sauvegarde.
* Enfin, prenez un instantané de l’objet blob de pages de sauvegarde et stockez-le dans le compte de stockage de sauvegarde.

Dans la section suivante, nous allons décrire plus en détail comment conserver des sauvegardes de disques à l’aide de la copie d’instantané incrémentiel

## <a name="scenario"></a>Scénario
Dans cette section, nous décrivons un scénario qui implique une stratégie de sauvegarde personnalisée pour les disques de machine virtuelle à l’aide de captures instantanées.

Prenez une machine virtuelle Azure de série DS avec un disque P30 de stockage Premium attaché. Le disque P30 appelé *mypremiumdisk* est stocké dans un compte de stockage Premium appelé *mypremiumaccount*. Un compte de stockage standard appelé *mybackupstdaccount* est utilisé pour stocker la sauvegarde de *mypremiumdisk*. Nous voulons conserver un instantané de *mypremiumdisk* toutes les 12 heures.

Pour plus d’informations sur la création d’un compte de stockage, consultez la section [Créer un compte de stockage](../articles/storage/common/storage-account-create.md).

Pour en savoir plus sur la sauvegarde de machines virtuelles Azure, reportez-vous à [Planifier les sauvegardes de machine virtuelle Azure](../articles/backup/backup-azure-vms-introduction.md).

## <a name="steps-to-maintain-backups-of-a-disk-using-incremental-snapshots"></a>Étapes pour gérer les sauvegardes d’un disque à l’aide d’instantanés incrémentiels
Les étapes suivantes décrivent comment effectuer des captures instantanées de *mypremiumdisk* et conserver les sauvegardes dans *mybackupstdaccount*. La sauvegarde est un objet blob de pages standard appelé *mybackupstdpageblob*. L’objet blob de pages de sauvegarde reflète toujours le même état en tant que dernière capture instantanée de *mypremiumdisk*.

1. Créer l’objet blob de pages de sauvegarde pour votre disque de stockage premium, en prenant une capture instantanée de *mypremiumdisk* appelée *mypremiumdisk_ss1*.
2. Copiez cet instantané vers mybackupstdaccount en tant qu’objet blob de pages nommé *mybackupstdpageblob*.
3. Prenez un instantané de *mybackupstdpageblob* nommé *mybackupstdpageblob_ss1*, à l’aide de [Snapshot Blob](/rest/api/storageservices/Snapshot-Blob), et stockez-le dans *mybackupstdaccount*.
4. Pendant la fenêtre de sauvegarde, créez un autre instantané de *mypremiumdisk*, appelez-le *mypremiumdisk_ss2* et stockez-le dans *mypremiumaccount*.
5. Identifiez les modifications incrémentielles entre les deux captures instantanées, *mypremiumdisk_ss2* et *mypremiumdisk_ss1*, à l’aide de [GetPageRanges](/rest/api/storageservices/Get-Page-Ranges) sur *mypremiumdisk_ss2* avec le paramètre **prevsnapshot** défini sur l’horodatage de *mypremiumdisk_ss1*. Écrivez ces modifications incrémentielles dans l’objet blob de pages de sauvegarde *mybackupstdpageblob* dans *mybackupstdaccount*. Si des modifications incrémentielles consistent en la suppression de plages, elles doivent être effacées de l’objet blob de pages de sauvegarde. Utilisez [Put Page](/rest/api/storageservices/Put-Page) pour écrire des modifications incrémentielles dans l’objet blob de pages de sauvegarde.
6. Prenez un instantané de l’objet blob de pages de sauvegarde *mybackupstdpageblob*, appelé *mybackupstdpageblob_ss2*. Supprimez l’instantané précédent *mypremiumdisk_ss1* du compte de stockage Premium.
7. Répétez les étapes 4 à 6 pour chaque fenêtre de sauvegarde. De cette façon, vous pouvez gérer les sauvegardes de *mypremiumdisk* dans un compte de stockage Standard.

![Sauvegarder un disque à l’aide d’instantanés incrémentiels](../articles/virtual-machines/windows/media/incremental-snapshots/storage-incremental-snapshots-1.png)

## <a name="steps-to-restore-a-disk-from-snapshots"></a>Procédure de restauration d’un disque à partir d’instantanés
Les étapes suivantes décrivent comment restaurer une ancienne capture instantanée du disque premium, *mypremiumdisk* à partir du compte de stockage de sauvegarde *mybackupstdaccount*.

1. Identifiez le point dans le temps que vous souhaitez utiliser afin de restaurer le disque Premium. Supposons qu’il s’agit de la capture instantanée *mybackupstdpageblob_ss2*, qui est stockée dans le compte de stockage de sauvegarde *mybackupstdaccount*.
2. Dans mybackupstdaccount, déclarez l’instantané *mybackupstdpageblob_ss2* comme nouvel objet blob de pages de base de sauvegarde *mybackupstdpageblobrestored*.
3. Prenez un instantané de cet objet blob de pages de sauvegarde restauré appelé *mybackupstdpageblobrestored_ss1*.
4. Copiez l’objet blob restauré *mybackupstdpageblobrestored* de *mybackupstdaccount* vers *mypremiumaccount* en tant que nouveau disque Premium *mypremiumdiskrestored*.
5. Prenez un instantané de *mypremiumdiskrestored*, appelé *mypremiumdiskrestored_ss1* pour les futures sauvegardes incrémentielles.
6. Pointez la machine virtuelle de série DS sur le disque restauré *mypremiumdiskrestored* et détachez l’ancien *mypremiumdisk* de la machine virtuelle.
7. Lancez le processus de sauvegarde décrit dans la section précédente pour le disque restauré *mypremiumdiskrestored*, à l’aide de *mybackupstdpageblobrestored* en tant qu’objet blob de pages de sauvegarde.

![Restaurer un disque à partir d’instantanés](../articles/virtual-machines/windows/media/incremental-snapshots/storage-incremental-snapshots-2.png)

## <a name="next-steps"></a>Étapes suivantes
Utilisez les liens suivants pour en savoir plus sur la création de captures instantanées d’un blob et sur la planification de votre infrastructure de sauvegarde de machine virtuelle.

* [Création d’un instantané d’objet blob](/rest/api/storageservices/Creating-a-Snapshot-of-a-Blob)
* [Planification de votre infrastructure de sauvegarde de machines virtuelles](../articles/backup/backup-azure-vms-introduction.md)