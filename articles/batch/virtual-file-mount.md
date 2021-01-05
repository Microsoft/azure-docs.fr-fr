---
title: Monter un système de fichiers virtuel sur un pool
description: Découvrez comment monter un système de fichiers virtuel sur un pool Batch.
ms.topic: how-to
ms.custom: devx-track-csharp
ms.date: 08/13/2019
ms.openlocfilehash: df03275fdeea88df1a2f2b6e2cda55021497cdf7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89145482"
---
# <a name="mount-a-virtual-file-system-on-a-batch-pool"></a>Monter un système de fichiers virtuel sur un pool Batch

Azure Batch prend désormais en charge le montage de stockage cloud ou d’un système de fichiers externe sur les nœuds de calcul Windows ou Linux dans vos pools Batch. Lorsqu’un nœud de calcul rejoint un pool, le système de fichiers virtuel est monté et traité comme un lecteur local sur ce nœud. Vous pouvez monter des systèmes de fichiers tels qu’Azure Files, le stockage blob Azure, NFS (Network File System), avec un [cache Avere vFXT](../avere-vfxt/avere-vfxt-overview.md), ou CIFS (Common Internet File System).

Dans cet article, vous allez apprendre à monter un système de fichiers virtuel sur un pool de nœuds de calcul à l’aide de la [Bibliothèque de gestion Batch pour .NET](/dotnet/api/overview/azure/batch).

> [!NOTE]
> Le montage d’un système de fichiers virtuel est pris en charge sur les pools Batch créés le ou après le 19/8/2019. Les pools Batch créés avant le 19/8/2019 ne prennent pas en charge cette fonctionnalité.
> 
> Les API servant à monter des systèmes de fichiers sur un nœud de calcul font partie de la bibliothèque [Batch .NET](/dotnet/api/microsoft.azure.batch).

## <a name="benefits-of-mounting-on-a-pool"></a>Avantages du montage sur un pool

Le montage du système de fichiers sur le pool, au lieu de laisser les tâches récupérer leurs propres données à partir d’un jeu de données volumineux, facilite et rend plus efficace l’accès aux données nécessaires pour les tâches.

Imaginez un scénario avec plusieurs tâches nécessitant l’accès à un jeu de données commun, comme le rendu d’un film. Chaque tâche restitue un ou plusieurs frames à la fois à partir des fichiers de scène. En montant un lecteur contenant les fichiers de scène, il est plus facile pour les nœuds de calcul d’accéder aux données partagées. En outre, le système de fichiers sous-jacent peut être choisi et mis à l’échelle indépendamment en fonction des performances et de l’échelle (débit et IOPS) requises par le nombre de nœuds de calcul qui accèdent simultanément aux données. Par exemple, il est possible d'utiliser un cache en mémoire distribué [Avere vFXT](../avere-vfxt/avere-vfxt-overview.md), qui peut être utilisé pour prendre en charge des rendus d’images à grande échelle avec des milliers de nœuds de rendu simultanés, en accédant aux données source qui résident localement. En guise d’alternative, pour les données qui se trouvent déjà dans le stockage d'objets blob sur le cloud, [blobfuse](../storage/blobs/storage-how-to-mount-container-linux.md) peut être utilisé pour monter ces données en tant que système de fichiers local. Blobfuse est disponible uniquement sur les nœuds Linux. Toutefois, [Azure Files](https://azure.microsoft.com/blog/a-new-era-for-azure-files-bigger-faster-better/) fournit un workflow similaire et est disponible sur Windows et Linux.

## <a name="mount-a-virtual-file-system-on-a-pool"></a>Monter un système de fichiers virtuel sur un pool  

Le montage d’un système de fichiers virtuel sur un pool rend le système de fichiers disponible pour chaque nœud de calcul du pool. Le système de fichiers est configuré soit lorsqu’un nœud de calcul rejoint un pool, soit lorsque le nœud est redémarré ou réinitialisé.

Pour monter un système de fichiers sur un pool, créez un objet `MountConfiguration`. Choisissez l’objet qui correspond à votre système de fichiers virtuel : `AzureBlobFileSystemConfiguration`, `AzureFileShareConfiguration`, `NfsMountConfiguration` ou `CifsMountConfiguration`.

Tous les objets de configuration de montage ont besoin des paramètres de base suivants. Certaines configurations de montage ont des paramètres spécifiques au système de fichiers utilisé, ce que nous évoquons plus en détail dans les exemples de code.

- **Nom ou source du compte** : Pour monter un partage de fichiers virtuel, vous avez besoin du nom du compte de stockage ou de sa source.
- **Chemin de montage relatif ou source** : L’emplacement du système de fichiers monté sur le nœud de calcul, par rapport au répertoire `fsmounts` standard accessible sur le nœud via `AZ_BATCH_NODE_MOUNTS_DIR`. L’emplacement exact varie selon le système d’exploitation utilisé sur le nœud. Par exemple, l’emplacement physique sur un nœud Ubuntu est mappé sur `mnt\batch\tasks\fsmounts`, et sur un nœud CentOS, il est mappé sur `mnt\resources\batch\tasks\fsmounts`.
- **Options de montage ou options blobfuse** : Ces options décrivent les paramètres spécifiques pour le montage d’un système de fichiers.

Une fois l’objet `MountConfiguration` créé, affectez l’objet à la propriété `MountConfigurationList` lorsque vous créez le pool. Le système de fichiers est monté soit lorsqu’un nœud rejoint un pool, soit lorsque le nœud est redémarré ou réinitialisé.

Lorsque le système de fichiers est monté, une variable d’environnement `AZ_BATCH_NODE_MOUNTS_DIR` est créée ; elle pointe vers l’emplacement des systèmes de fichiers montés, ainsi que des fichiers journaux, qui sont utiles pour la résolution des problèmes et le débogage. Les fichiers journaux sont expliqués plus en détail dans la section [Diagnostiquer les erreurs de montage](#diagnose-mount-errors).  

> [!IMPORTANT]
> Le nombre maximal de systèmes de fichiers montés sur un pool est de 10. Consultez [Quotas et limites du service Batch](batch-quota-limit.md#other-limits) pour plus d’informations et pour connaître les autres limitations.

## <a name="examples"></a>Exemples

Les exemples de code suivants illustrent le montage d’une grande diversité de partages de fichiers sur un pool de nœuds de calcul.

### <a name="azure-files-share"></a>Partage Azure Files

Azure Files est l’offre standard de systèmes de fichiers cloud d’Azure. Pour en savoir plus sur l’obtention de n’importe lequel des paramètres dans l’exemple de code de configuration de montage, consultez [Utiliser un partage Azure Files](../storage/files/storage-how-to-use-files-windows.md).

```csharp
new PoolAddParameter
{
    Id = poolId,
    MountConfiguration = new[]
    {
        new MountConfiguration
        {
            AzureFileShareConfiguration = new AzureFileShareConfiguration
            {
                AccountName = "{storage-account-name}",
                AzureFileUrl = "https://{storage-account-name}.file.core.windows.net/{file-share-name}",
                AccountKey = "{storage-account-key}",
                RelativeMountPath = "S",
                MountOptions = "-o vers=3.0,dir_mode=0777,file_mode=0777,sec=ntlmssp"
            },
        }
    }
}
```

### <a name="azure-blob-file-system"></a>Système de fichiers blob Azure

Une autre option consiste à utiliser le stockage d'objets blob Azure via [blobfuse](../storage/blobs/storage-how-to-mount-container-linux.md). Le montage d’un système de fichiers blob nécessite une `AccountKey` ou `SasKey` pour votre compte de stockage. Pour plus d’informations sur l’obtention de ces clés, consultez [Gérer les clés d’accès des comptes de stockage](../storage/common/storage-account-keys-manage.md) ou [Utiliser des signatures d’accès partagé (SAS)](../storage/common/storage-sas-overview.md). Pour plus d’informations sur l’utilisation de blobfuse, consultez le [Forum aux questions sur la résolution des problèmes](https://github.com/Azure/azure-storage-fuse/wiki/3.-Troubleshoot-FAQ) de blobfuse. Pour accéder par défaut au répertoire monté avec blobfuse, exécutez la tâche en tant **qu'administrateur**. Blobfuse monte le répertoire au niveau de l’espace utilisateur et, lors de la création du pool, il est monté en tant que racine. Dans Linux, toutes les tâches **de l’Administrateur** sont racine. Toutes les options du module FUSE sont décrites dans la [Page de référence sur FUSE](https://manpages.ubuntu.com/manpages/xenial/man8/mount.fuse.8.html).

Outre le guide de résolution des problèmes, les problèmes GitHub dans le référentiel blobfuse sont un moyen utile de vérifier les problèmes et solutions pour blobfuse. Pour plus d’informations, consultez [Problèmes blobfuse](https://github.com/Azure/azure-storage-fuse/issues).

```csharp
new PoolAddParameter
{
    Id = poolId,
    MountConfiguration = new[]
    {
        new MountConfiguration
        {
            AzureBlobFileSystemConfiguration = new AzureBlobFileSystemConfiguration
            {
                AccountName = "StorageAccountName",
                ContainerName = "containerName",
                AccountKey = "StorageAccountKey",
                SasKey = "",
                RelativeMountPath = "RelativeMountPath",
                BlobfuseOptions = "-o attr_timeout=240 -o entry_timeout=240 -o negative_timeout=120 "
            },
        }
    }
}
```

### <a name="network-file-system"></a>Système de gestion de fichiers en réseau

Les systèmes de fichiers réseau (NFS) peuvent également être montés sur des nœuds de pool, ce qui permet aux nœuds Azure Batch d’accéder facilement aux systèmes de fichiers traditionnels. Il peut s’agir d’un serveur NFS unique déployé dans le cloud ou d’un serveur NFS local accessible via un réseau virtuel. Vous pouvez également tirer parti de la solution de mise en cache en mémoire distribuée [Avere vFXT](../avere-vfxt/avere-vfxt-overview.md), qui fournit une connectivité transparente au stockage local, lit les données à la demande dans son cache et offre des performances et une évolutivité élevées pour les nœuds de calcul basés sur le cloud.

```csharp
new PoolAddParameter
{
    Id = poolId,
    MountConfiguration = new[]
    {
        new MountConfiguration
        {
            NfsMountConfiguration = new NFSMountConfiguration
            {
                Source = "source",
                RelativeMountPath = "RelativeMountPath",
                MountOptions = "options ver=1.0"
            },
        }
    }
}
```

### <a name="common-internet-file-system"></a>Common Internet File System

Les systèmes de fichiers Common Internet File Systems (CIFS) peuvent également être montés sur des nœuds de pool, ce qui permet aux nœuds Azure Batch d’accéder facilement aux systèmes de fichiers traditionnels. CIFS est un protocole de partage de fichiers qui fournit un mécanisme ouvert et multiplateforme pour la demande de fichiers et de services de serveur réseau. CIFS est basé sur la version améliorée du protocole SMB (Server Message Block) de Microsoft pour le partage de fichiers intranet et Internet, et est utilisé pour monter des systèmes de fichiers externes sur des nœuds Windows. Pour en savoir plus sur SMB, consultez [Serveur de fichiers et SMB](/windows-server/storage/file-server/file-server-smb-overview).

```csharp
new PoolAddParameter
{
    Id = poolId,
    MountConfiguration = new[]
    {
        new MountConfiguration
        {
            CifsMountConfiguration = new CIFSMountConfiguration
            {
                Username = "StorageAccountName",
                RelativeMountPath = "cifsmountpoint",
                Source = "source",
                Password = "StorageAccountKey",
                MountOptions = "-o vers=3.0,dir_mode=0777,file_mode=0777,serverino"
            },
        }
    }
}
```

## <a name="diagnose-mount-errors"></a>Diagnostiquer les erreurs de montage

Si une configuration de montage échoue, le nœud de calcul dans le pool échoue et l’état du nœud devient inutilisable. Pour diagnostiquer un échec de configuration de montage, examinez la propriété [`ComputeNodeError`](/rest/api/batchservice/computenode/get#computenodeerror) pour plus d’informations sur l’erreur.

Pour obtenir les fichiers journaux de débogage, utilisez [OutputFiles](batch-task-output-files.md) pour télécharger les fichiers `*.log`. Les fichiers `*.log` contiennent des informations sur le montage du système de fichiers à l’emplacement `AZ_BATCH_NODE_MOUNTS_DIR`. Les fichiers journaux de montage ont le format : `<type>-<mountDirOrDrive>.log` pour chaque montage. Par exemple, un montage `cifs` dans un répertoire de montage nommé `test` aura un fichier journal de montage nommé `cifs-test.log`.

## <a name="supported-skus"></a>Références prises en charge

| Serveur de publication | Offre | SKU | Partage Azure Files | Blobfuse | Montage NFS | Montage CIFS |
|---|---|---|---|---|---|---|
| lot | rendering-centos73 | rendu | :heavy_check_mark: <br>Remarque : Compatible avec CentOS 7.7</br>| :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| Canonical | UbuntuServer | 16.04-LTS, 18.04-LTS | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| Credativ | Debian | 8| :heavy_check_mark: | :x: | :heavy_check_mark: | :heavy_check_mark: |
| Credativ | Debian | 9 | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| microsoft-ads | linux-data-science-vm | linuxdsvm | :heavy_check_mark: <br>Remarque : Compatible avec CentOS 7.4. </br> | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| microsoft-azure-batch | centos-container | 7.6 | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| microsoft-azure-batch | centos-container-rdma | 7.4 | :heavy_check_mark: <br>Remarque : Prend en charge le stockage A_8 ou 9</br> | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| microsoft-azure-batch | ubuntu-server-container | 16.04-LTS | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| microsoft-dsvm | linux-data-science-vm-ubuntu | linuxdsvmubuntu | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| OpenLogic | CentOS | 7.6 | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| OpenLogic | CentOS-HPC | 7.4, 7.3, 7.1 | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| Oracle | Oracle-Linux | 7.6 | :x: | :x: | :x: | :x: |
| Windows | WindowsServer | 2012, 2016, 2019 | :heavy_check_mark: | :x: | :x: | :x: |

## <a name="next-steps"></a>Étapes suivantes

- Découvrez-en davantage sur le montage d’un partage Azure Files avec [Windows](../storage/files/storage-how-to-use-files-windows.md) ou [Linux](../storage/files/storage-how-to-use-files-linux.md).
- Découvrez-en davantage sur l’utilisation et le montage de systèmes de fichiers virtuels [blobfuse](https://github.com/Azure/azure-storage-fuse).
- Consultez [Vue d’ensemble du système de fichiers réseau](/windows-server/storage/nfs/nfs-overview) pour en savoir plus sur NFS et ses applications.
- Pour plus d’informations sur CIFS, consultez [Vue d’ensemble du protocole SMB et du protocole CIFS de Microsoft](/windows/desktop/fileio/microsoft-smb-protocol-and-cifs-protocol-overview).
