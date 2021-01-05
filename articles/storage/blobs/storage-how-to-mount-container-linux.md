---
title: Comment monter le stockage Blob Azure en tant que système de fichiers sur Linux | Microsoft Docs
description: Découvrez comment monter un conteneur de stockage Azure Blob avec blobfuse, un pilote de système de fichiers virtuel sur Linux.
author: tamram
ms.service: storage
ms.subservice: blobs
ms.topic: how-to
ms.date: 2/1/2019
ms.author: tamram
ms.reviewer: twooley
ms.openlocfilehash: 8de395e34b43a4edad2affa591adb8ab34ff9e66
ms.sourcegitcommit: fec60094b829270387c104cc6c21257826fccc54
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96921702"
---
# <a name="how-to-mount-blob-storage-as-a-file-system-with-blobfuse"></a>Comment monter le stockage Blob en tant que système de fichiers avec blobfuse

## <a name="overview"></a>Vue d’ensemble
[Blobfuse](https://github.com/Azure/azure-storage-fuse) est un pilote de système de fichiers virtuel pour Stockage Blob Azure. Blobfuse vous permet d’accéder à vos données d’objets blob de blocs existantes dans votre compte de stockage via le système de fichiers Linux. Blobfuse utilise le schéma de répertoire virtuel avec la barre oblique (/) comme délimiteur.  

Ce guide vous explique comment utiliser blobfuse, monter un conteneur de stockage Blob sur Linux et accéder aux données. Pour en savoir plus sur blobfuse, consultez le [référentiel blobfuse](https://github.com/Azure/azure-storage-fuse).

> [!WARNING]
> Blobfuse ne garantit pas la conformité totale à POSIX, car il traduit simplement les demandes en [API REST Blob](/rest/api/storageservices/blob-service-rest-api). Par exemple, les opérations de changement de nom sont atomiques dans POSIX, mais pas dans blobfuse.
> Pour obtenir la liste complète des différences entre un système de fichiers natif et blobfuse, consultez le [référentiel de code source blobfuse](https://github.com/azure/azure-storage-fuse).
> 

## <a name="install-blobfuse-on-linux"></a>Installer blobfuse sur Linux
Les fichiers binaires de blobfuse sont disponibles sur les [dépôts de logiciels Microsoft pour Linux](/windows-server/administration/Linux-Package-Repository-for-Microsoft-Software) (distributions Ubuntu et RHEL). Pour installer blobfuse sur ces distributions, configurez l’un des dépôts de la liste. Si les fichiers binaires ne sont pas disponibles pour votre distribution, vous pouvez les générer à partir du code source en suivant les [étapes d’installation du Stockage Azure](https://github.com/Azure/azure-storage-fuse/wiki/1.-Installation#option-2---build-from-source).

Blobfuse prend en charge l’installation sur Ubuntu 14.04, 16.04, 18.04 et 20.04. Exécutez cette commande pour vous assurer qu’une de ces versions est déployée :
```
lsb_release -a
```

### <a name="configure-the-microsoft-package-repository"></a>Configurer le référentiel de packages Microsoft
Configurez le [référentiel de packages Linux pour les produits Microsoft](/windows-server/administration/Linux-Package-Repository-for-Microsoft-Software).

Par exemple, sur une distribution Enterprise Linux 6 :
```bash
sudo rpm -Uvh https://packages.microsoft.com/config/rhel/6/packages-microsoft-prod.rpm
```

De manière similaire, remplacez l’URL par `.../rhel/7/...` pour pointer vers une distribution Enterprise Linux 7.

Autre exemple sur une distribution Ubuntu 14.04 :
```bash
wget https://packages.microsoft.com/config/ubuntu/14.04/packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
sudo apt-get update
```

De même, remplacez l’URL par `.../ubuntu/16.04/...` ou `.../ubuntu/18.04/...` pour référencer une autre version d’Ubuntu.

### <a name="install-blobfuse"></a>Installer blobfuse

Sur une distribution Ubuntu/Debian :
```bash
sudo apt-get install blobfuse
```

Sur une distribution Enterprise Linux :
```bash    
sudo yum install blobfuse
```

## <a name="prepare-for-mounting"></a>Préparer le montage
Blobfuse permet d’obtenir des performances de type natif. Pour ce faire, il requiert un chemin d’accès temporaire dans le système de fichiers pour mettre les fichiers ouverts en mémoire tampon et en cache. Pour ce chemin d’accès temporaire, choisissez le disque le plus performant ou utilisez un disque virtuel afin d’obtenir des performances optimales. 

> [!NOTE]
> Blobfuse stocke tout le contenu des fichiers ouverts dans ce chemin d’accès temporaire. Veillez à disposer d’un espace suffisant pour prendre en compte tous les fichiers ouverts. 
> 

### <a name="optional-use-a-ramdisk-for-the-temporary-path"></a>(Facultatif) Utiliser un disque virtuel pour le chemin d’accès temporaire
L’exemple suivant crée un disque virtuel de 16 Go, ainsi qu’un répertoire pour blobfuse. Choisissez la taille selon vos besoins. Ce disque virtuel permet à blobfuse d’ouvrir des fichiers d’une taille maximale de 16 Go. 
```bash
sudo mkdir /mnt/ramdisk
sudo mount -t tmpfs -o size=16g tmpfs /mnt/ramdisk
sudo mkdir /mnt/ramdisk/blobfusetmp
sudo chown <youruser> /mnt/ramdisk/blobfusetmp
```

### <a name="use-an-ssd-as-a-temporary-path"></a>Utiliser un disque SSD comme chemin d’accès temporaire
Dans Azure, vous pouvez utiliser les disques éphémères (SSD) disponibles sur vos machines virtuelles pour fournir une mémoire tampon à faible latence à blobfuse. Dans les distributions Ubuntu, ce disque éphémère est monté sur '/mnt'. Dans les distributions Red Hat et CentOS, le disque est monté sur '/mnt/resource/'.

Vérifiez que votre utilisateur a accès au chemin d’accès temporaire :
```bash
sudo mkdir /mnt/resource/blobfusetmp -p
sudo chown <youruser> /mnt/resource/blobfusetmp
```

### <a name="configure-your-storage-account-credentials"></a>Configurer les informations d’identification de votre compte de stockage
Blobfuse requiert que vos informations d’identification soient stockées dans un fichier texte au format suivant : 

```
accountName myaccount
accountKey storageaccesskey
containerName mycontainer
```
`accountName` est le préfixe pour votre compte de stockage, pas l’URL complète.

Créez ce fichier à l’aide de :

```
touch ~/fuse_connection.cfg
```

Dès que vous avez créé et modifié ce fichier, veillez à en restreindre l’accès afin qu’aucun autre utilisateur ne puisse le lire.
```bash
chmod 600 fuse_connection.cfg
```

> [!NOTE]
> Si vous avez créé le fichier de configuration sur Windows, veillez à exécuter `dos2unix` pour l’assainir et le convertir au format Unix. 
>

### <a name="create-an-empty-directory-for-mounting"></a>Créer un répertoire vide pour le montage
```bash
mkdir ~/mycontainer
```

## <a name="mount"></a>Monter

> [!NOTE]
> Pour obtenir la liste complète des options de montage, consultez le [référentiel blobfuse](https://github.com/Azure/azure-storage-fuse#mount-options).  
> 

Pour monter blobfuse, exécutez la commande suivante avec votre utilisateur. Cette commande monte le conteneur spécifié dans '/path/to/fuse_connection.cfg' sur l’emplacement '/mycontainer'.

```bash
sudo blobfuse ~/mycontainer --tmp-path=/mnt/resource/blobfusetmp  --config-file=/path/to/fuse_connection.cfg -o attr_timeout=240 -o entry_timeout=240 -o negative_timeout=120
```

Vous devez maintenant avoir accès à vos objets blob de blocs via les API normales du système de fichiers. L’utilisateur qui monte le répertoire est la seule personne qui peut y accéder, par défaut, ce qui en sécurise l’accès. Pour autoriser l’accès à tous les utilisateurs, vous pouvez effectuer le montage avec l’option ```-o allow_other```. 

```bash
cd ~/mycontainer
mkdir test
echo "hello world" > test/blob.txt
```

## <a name="next-steps"></a>Étapes suivantes

* [Page d’accueil de blobfuse](https://github.com/Azure/azure-storage-fuse#blobfuse)
* [Signalement des problèmes de blobfuse](https://github.com/Azure/azure-storage-fuse/issues)
