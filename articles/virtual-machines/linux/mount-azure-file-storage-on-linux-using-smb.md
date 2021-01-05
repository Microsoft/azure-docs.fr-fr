---
title: Monter le stockage de fichiers Azure sur les machines virtuelles Linux à l’aide de SMB
description: Procédure de montage du stockage de fichiers Azure sur les machines virtuelles Linux à l’aide de SMB avec Azure CLI
author: cynthn
ms.service: virtual-machines-linux
ms.topic: how-to
ms.workload: infrastructure
ms.date: 06/28/2018
ms.author: cynthn
ms.openlocfilehash: 6a5d4f388d9e5f47a13812687489525590c520c3
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96016095"
---
# <a name="mount-azure-file-storage-on-linux-vms-using-smb"></a>Monter le stockage de fichiers Azure sur les machines virtuelles Linux à l’aide de SMB

Cet article vous montre comment utiliser le service de stockage de fichiers Azure sur une machine virtuelle Linux à l’aide d’un montage SMB avec Azure CLI. Le stockage de fichiers Azure propose des partages de fichiers dans le cloud s’appuyant sur le protocole SMB standard. 

Le stockage de fichiers propose des partages de fichiers dans le cloud qui utilisent le protocole SMB standard. Vous pouvez monter un partage de fichiers à partir de n’importe quel système d’exploitation qui prend en charge SMB 3.0. Un montage SMB sur Linux permet d’effectuer facilement une sauvegarde vers un emplacement de stockage d’archivage robuste et permanent pris en charge par un contrat de niveau de service.

Un bon moyen de déboguer les journaux d’activité consiste à déplacer les fichiers vers un montage SMB hébergé sur le stockage de fichiers à partir d’une machine virtuelle. Un même partage SMB peut être monté localement sur votre station de travail Windows, Linux ou Mac. SMB ne constitue pas la meilleure solution pour transmettre en continu des journaux d’activité d’applications ou Linux en temps réel, car le protocole SMB n’est pas conçu pour gérer des tâches de journalisation aussi lourdes. Un outil de couche de journalisation unifié et dédié comme Fluentd représente un meilleur choix que SMB pour collecter la sortie de journalisation d’applications et Linux.

Ce guide nécessite que vous exécutiez Azure CLI version 2.0.4 ou ultérieure. Pour déterminer la version, exécutez la commande **az --version**. Si vous devez effectuer une installation ou une mise à niveau, consultez [Installer Azure CLI](/cli/azure/install-azure-cli). 


## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Créez un groupe de ressources nommé *MyResourceGroup* dans l’emplacement *USA Est*.

```azurecli
az group create --name myResourceGroup --location eastus
```

## <a name="create-a-storage-account"></a>Créez un compte de stockage.

Créez un compte de stockage, au sein du groupe de ressources que vous avez créé, à l’aide de la commande [az storage account create](/cli/azure/storage/account). Cet exemple crée un compte de stockage nommé *mySTORAGEACCT\<random number>* et place le nom de ce compte de stockage dans la variable **STORAGEACCT**. Les noms de compte de stockage doivent être uniques ; l’utilisation de `$RANDOM` ajoute un numéro à la fin pour les rendre uniques.

```azurecli
STORAGEACCT=$(az storage account create \
    --resource-group "myResourceGroup" \
    --name "mystorageacct$RANDOM" \
    --location eastus \
    --sku Standard_LRS \
    --query "name" | tr -d '"')
```

## <a name="get-the-storage-key"></a>Obtenir la clé de stockage

Quand vous créez un compte de stockage, les clés du compte sont créées par paires afin qu’elles puissent faire l’objet d’une rotation sans interruption du service. Lorsque vous basculez vers la deuxième clé de la paire, vous créez une nouvelle paire de clés. Les clés de compte de stockage étant toujours créées par paires, vous disposez toujours d’au moins une clé de stockage inutilisée prête au basculement.

Affichez les clés du compte de stockage à l’aide de la commande [az storage account keys list](/cli/azure/storage/account/keys). Cet exemple stocke la valeur de clé 1 dans la variable **STORAGEKEY**.

```azurecli
STORAGEKEY=$(az storage account keys list \
    --resource-group "myResourceGroup" \
    --account-name $STORAGEACCT \
    --query "[0].value" | tr -d '"')
```

## <a name="create-a-file-share"></a>Créer un partage de fichiers

Créez le partage de stockage de fichiers à l’aide de la commande [az storage share create](/cli/azure/storage/share). 

Le nom des partages ne doit contenir que des minuscules, des nombres et des traits d’union uniques, mais ne peut commencer par un trait d’union. Pour plus d’informations sur la façon de nommer des partages de fichiers et des fichiers, consultez la rubrique [Affectation de noms et références aux partages, répertoires, fichiers et métadonnées](/rest/api/storageservices/naming-and-referencing-shares--directories--files--and-metadata).

Cet exemple crée un partage nommé *myshare* avec un quota de 10 Gio. 

```azurecli
az storage share create --name myshare \
    --quota 10 \
    --account-name $STORAGEACCT \
    --account-key $STORAGEKEY
```

## <a name="create-a-mount-point"></a>Créer un point de montage

Pour monter le partage de fichiers Azure sur votre ordinateur Linux, vous devez vérifier que le package **cifs-utils** est installé. Pour obtenir des instructions d’installation, consultez [Installez le package cifs-utils pour votre distribution Linux](../../storage/files/storage-how-to-use-files-linux.md#install-cifs-utils).

Azure Files utilise le protocole SMB, qui communique via le port TCP 445.  Si vous ne parvenez pas à monter votre partage de fichiers Azure, vérifiez que votre pare-feu ne bloque pas le port TCP 445.


```bash
mkdir -p /mnt/MyAzureFileShare
```

## <a name="mount-the-share"></a>Monter le partage

Montez le partage de fichiers Azure dans le répertoire local. 

```bash
sudo mount -t cifs //$STORAGEACCT.file.core.windows.net/myshare /mnt/MyAzureFileShare -o vers=3.0,username=$STORAGEACCT,password=$STORAGEKEY,dir_mode=0777,file_mode=0777,serverino
```

La commande ci-dessus utilise la commande [mount](https://linux.die.net/man/8/mount) pour monter le partage de fichiers Azure et les options spécifiques à [cifs](https://linux.die.net/man/8/mount.cifs). Plus précisément, les options file_mode et dir_mode définissent les fichiers et répertoires pour l’autorisation `0777`. L’autorisation `0777` octroie à tous les utilisateurs des droits de lecture, écriture et exécution. Vous pouvez modifier ces autorisations en remplaçant les valeurs par d’autres [autorisations chmod](https://en.wikipedia.org/wiki/Chmod). Vous pouvez également utiliser d’autres options [cifs](https://linux.die.net/man/8/mount.cifs) telles que gid ou uid. 


## <a name="persist-the-mount"></a>Rendre le montage permanent

Lorsque vous redémarrez la machine virtuelle Linux, le partage SMB monté est démonté lors de l’arrêt. Pour remonter le partage SMB au démarrage, ajoutez une ligne à /etc/fstab dans Linux. Linux utilise le fichier fstab pour lister les systèmes de fichiers à monter pendant le processus de démarrage. L’ajout du partage SMB garantit que le partage de stockage de fichiers est un système de fichiers monté définitivement pour la machine virtuelle Linux. Il est possible d’ajouter le partage SMB du Stockage Fichier sur une nouvelle machine virtuelle si vous utilisez cloud-init.

```bash
//myaccountname.file.core.windows.net/mystorageshare /mnt/mymountpoint cifs vers=3.0,username=mystorageaccount,password=myStorageAccountKeyEndingIn==,dir_mode=0777,file_mode=0777
```

Pour renforcer la sécurité dans les environnements de production, vous devez stocker vos informations d’identification en dehors de fstab.

## <a name="next-steps"></a>Étapes suivantes

- [Utilisation de cloud-init pour personnaliser une machine virtuelle Linux lors de la création](using-cloud-init.md)
- [Ajouter un disque à une machine virtuelle Linux](add-disk.md)
- [Azure Disk Encryption pour les machines virtuelles Linux](disk-encryption-overview.md)
