---
title: Utiliser un stockage de sauvegarde moderne avec un Serveur de sauvegarde Azure
description: Découvrez les nouvelles fonctionnalités du Serveur de sauvegarde Azure. Cet article décrit comment mettre à niveau votre installation de serveur de sauvegarde.
ms.topic: conceptual
ms.date: 11/13/2018
ms.openlocfilehash: b077296e58e1193e454a686a392d802e905500a5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91292895"
---
# <a name="add-storage-to-azure-backup-server"></a>Ajouter du stockage à un serveur de sauvegarde Azure

Azure Backup Server V2 et les versions ultérieures prennent en charge le stockage de sauvegarde moderne, qui offre des économies de stockage de 50 %, des sauvegardes trois fois plus rapides et un stockage plus efficace. Il s’agit également d’un stockage prenant en compte la charge de travail.

> [!NOTE]
> Pour utiliser le stockage de sauvegarde moderne, vous devez exécuter le Serveur de sauvegarde V2 ou V3 sur Windows Server 2016 ou V3 sur Windows Server 2019.
> Si vous exécutez le Serveur de sauvegarde V2 sur une version antérieure de Windows Server, le Serveur de sauvegarde Azure ne peut pas tirer parti du stockage de sauvegarde moderne. Au lieu de cela, il protège les charges de travail comme il le fait avec le Serveur de sauvegarde V1. Pour plus d’informations, voir la [matrice protection](backup-mabs-protection-matrix.md) de la version du Serveur de sauvegarde.
>
> Pour améliorer les performances de sauvegarde, nous vous recommandons de déployer MABS v3 avec un stockage hiérarchisé sur Windows Server 2019. Reportez-vous à la section « [Configurer MBS avec un stockage hiérarchisé](/system-center/dpm/add-storage#set-up-mbs-with-tiered-storage) » de l'article DPM pour apprendre à configurer un stockage hiérarchisé.

## <a name="volumes-in-backup-server"></a>Volumes dans le Serveur de sauvegarde

Le Serveur de sauvegarde V2 ou versions ultérieures accepte les volumes de stockage. Lorsque vous ajoutez un volume, le Serveur de sauvegarde formate le volume au format Resilient File System (ReFS) que nécessite le stockage de sauvegarde moderne. Pour ajouter un volume et le développer ultérieurement si nécessaire, nous vous suggérons d’utiliser flux de travail suivant :

1. Configurez le Serveur de sauvegarde sur une machine virtuelle.
2. Créez un volume sur un disque virtuel dans un pool de stockage :
    1. Ajoutez un disque à un pool de stockage, et créez un disque virtuel avec une disposition de stockage simple.
    2. Ajoutez des disques supplémentaires, puis étendez le disque virtuel.
    3. Créez des volumes sur le disque virtuel.
3. Ajoutez les volumes au Serveur de sauvegarde.
4. Configurez un stockage prenant en compte la charge de travail.

## <a name="create-a-volume-for-modern-backup-storage"></a>Créer un volume pour le stockage de sauvegarde moderne

L’utilisation du Serveur de sauvegarde V2 ou versions ultérieures avec des volumes tels qu’un stockage sur disque peut vous aider à contrôler le stockage. Un volume peut être un disque unique. Toutefois, si vous souhaitez étendre le stockage à l’avenir, créez un volume à partir d’un disque créé à l’aide d’espaces de stockage. Cela peut être utile si vous souhaitez étendre le volume du stockage de sauvegarde. Cette section présente les meilleures pratiques en matière de création de volume avec ce programme d’installation.

1. Dans le Gestionnaire de serveur, sélectionnez **Services de fichiers et de stockage** > **Volumes** > **Pools de stockage**. Sous **Disques physiques**, sélectionnez **Nouveau pool de stockage**.

    ![Créer un pool de stockage](./media/backup-mabs-add-storage/mabs-add-storage-1.png)

2. Dans la zone de liste déroulante **Tâches**, sélectionnez **Nouveau disque virtuel**.

    ![Ajouter un disque virtuel](./media/backup-mabs-add-storage/mabs-add-storage-2.png)

3. Sélectionnez le pool de stockage, puis choisissez **Ajouter un disque physique**.

    ![Ajouter un disque physique](./media/backup-mabs-add-storage/mabs-add-storage-3.png)

4. Sélectionnez le disque physique, puis choisissez **Étendre le disque virtuel**.

    ![Étendre le disque virtuel](./media/backup-mabs-add-storage/mabs-add-storage-4.png)

5. Sélectionnez le disque virtuel, puis choisissez **Nouveau volume**.

    ![Créer un volume](./media/backup-mabs-add-storage/mabs-add-storage-5.png)

6. Dans la boîte de dialogue **sélectionner le serveur et le disque**, sélectionnez le serveur et le nouveau disque. Ensuite, sélectionnez **Suivant**.

    ![Sélectionner le serveur et le disque](./media/backup-mabs-add-storage/mabs-add-storage-6.png)

## <a name="add-volumes-to-backup-server-disk-storage"></a>Ajouter des volumes de stockage au stockage sur disque du Serveur de sauvegarde

> [!NOTE]
>
> - Ajoutez un seul disque au pool pour que le nombre de colonnes reste égal à 1. Vous pourrez ajouter des disques ultérieurement en fonction de vos besoins.
> - Si vous ajoutez plusieurs disques au pool de stockage en une fois, le nombre de disques est stocké en tant que nombre de colonnes. Lorsque vous ajoutez des disques, leur nombre doit être un multiple du nombre de colonnes.

Pour ajouter un volume au Serveur de sauvegarde, dans le volet **Gestion**, réanalysez le stockage, puis sélectionnez **Ajouter**. La liste de tous les volumes disponibles pour ajout au stockage du Serveur de sauvegarde s’affiche. Une fois les volumes disponibles ajoutés à la liste des volumes sélectionnés, vous pouvez leur donner un nom convivial pour faciliter leur gestion. Pour formater ces volumes au format ReFS de façon à ce que le Serveur de sauvegarde puisse tirer parti des avantages du stockage de sauvegarde moderne, sélectionnez **OK**.

![Ajouter des volumes disponibles](./media/backup-mabs-add-storage/mabs-add-storage-7.png)

## <a name="set-up-workload-aware-storage"></a>Configurer un stockage prenant en compte la charge de travail

Avec un stockage prenant en compte les charges de travail, vous pouvez sélectionner les volumes qui stockent de préférence certains genres de charges de travail. Par exemple, vous pouvez définir des volumes coûteux prenant en charge un nombre élevé d’opérations d’entrée/sortie par seconde (IOPS) pour stocker uniquement les charges de travail qui nécessitent des sauvegardes de volumes importants fréquentes. Un exemple est SQL Server avec les journaux d’activité des transactions. D’autres charges de travail sauvegardées moins fréquemment, telles que les machines virtuelles, peuvent être sauvegardées sur des volumes bon marché.

### <a name="update-dpmdiskstorage"></a>Update-DPMDiskStorage

Vous pouvez configurer un stockage prenant en compte les charges de travail à l’aide de l’applet de commande PowerShell Update-DPMDiskStorage, ce qui a pour effet de mettre à jour les propriétés d’un volume dans le pool de stockage sur un serveur de sauvegarde Azure.

Syntaxe :

`Parameter Set: Volume`

```powershell
Update-DPMDiskStorage [-Volume] <Volume> [[-FriendlyName] <String> ] [[-DatasourceType] <VolumeTag[]> ] [-Confirm] [-WhatIf] [ <CommonParameters>]
```

La capture d’écran suivante montre l’applet de commande Update-DPMDiskStorage dans la fenêtre PowerShell.

![Commande Update-DPMDiskStorage dans la fenêtre PowerShell](./media/backup-mabs-add-storage/mabs-add-storage-8.png)

Les modifications apportées à l’aide de PowerShell apparaissent sur la Console Administrateur du Serveur de sauvegarde.

![Disques et volumes dans la Console Administrateur](./media/backup-mabs-add-storage/mabs-add-storage-9.png)

## <a name="migrate-legacy-storage-to-modern-backup-storage"></a>Migrer un stockage existant vers Modern Backup Storage

Après avoir installé le serveur de sauvegarde V2 ou effectué la mise à niveau vers ce dernier et mis à niveau le système d’exploitation vers Windows Server 2016, mettez à jour vos groupes de protection pour qu’ils utilisent Modern Backup Storage. Par défaut, les groupes de protection ne sont pas modifiés. Ils continuent de fonctionner comme ils ont été initialement configurés.

La mise à jour des groupes de protection pour qu’ils utilisent Modern Backup Storage est facultative. Pour mettre à jour le groupe de protection, arrêtez la protection de toutes les sources de données à l’aide de l’option de conservation des données. Ensuite, ajoutez les sources de données à un nouveau groupe de protection.

1. Dans la console Administrateur du serveur de sauvegarde, sélectionnez la fonctionnalité **Protection**. Dans la liste **Membre du groupe de Protection**, cliquez sur le membre, puis sélectionnez **Arrêter la protection du membre**.

   ![Arrêter la protection du membre](/system-center/dpm/media/upgrade-to-dpm-2016/dpm-2016-stop-protection1.png)

2. Dans la boîte de dialogue **Supprimer du groupe**, vérifiez l’espace disque utilisé et l’espace libre disponible pour le pool de stockage. La valeur par défaut consiste à conserver les points de récupération sur le disque et à les laisser expirer suivant la stratégie de rétention associée. Sélectionnez **OK**.

   Si vous souhaitez restituer immédiatement l’espace disque utilisé pour le pool de stockage libre, sélectionnez la case à cocher **Supprimer le réplica sur le disque** pour supprimer les données de sauvegarde (et les points de récupération) associées à ce membre.

   ![Boîte de dialogue Supprimer du groupe](/system-center/dpm/media/upgrade-to-dpm-2016/dpm-2016-retain-data.png)

3. Créez un groupe de protection qui utilise Modern Backup Storage. Incluez les sources de données non protégées.

## <a name="add-disks-to-increase-legacy-storage"></a>Ajouter des disques pour augmenter le stockage existant

Si vous souhaitez utiliser un stockage existant avec le serveur de sauvegarde, vous devrez peut-être ajouter des disques pour augmenter la capacité de stockage.

Pour ajouter un stockage sur disque :

1. Dans la console Administrateur, sélectionnez **Gestion** > **Stockage sur disque** > **Ajouter**.

    ![Boîte de dialogue Ajouter un stockage sur disque](/system-center/dpm/media/upgrade-to-dpm-2016/dpm-2016-add-disk-storage.png)

2. Dans la boîte de dialogue **Ajouter un stockage sur disque**, sélectionnez **Ajouter des disques**.

3. Dans la liste des disques disponibles, sélectionnez les disques à ajouter, choisissez **Ajouter**, puis sélectionnez **OK**.

## <a name="next-steps"></a>Étapes suivantes

Après avoir installé le Serveur de sauvegarde, apprenez à préparer votre serveur ou commencez à protéger une charge de travail.

- [Préparer les charges de travail du serveur de sauvegarde](backup-azure-microsoft-azure-backup.md)
- [Utiliser le Serveur de sauvegarde pour sauvegarder un serveur VMware](backup-azure-backup-server-vmware.md)
- [Utiliser le serveur de sauvegarde pour sauvegarder SQL Server](backup-azure-sql-mabs.md)
