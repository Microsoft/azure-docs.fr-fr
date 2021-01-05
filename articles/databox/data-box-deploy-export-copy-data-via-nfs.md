---
title: Tutoriel sur la copie de données depuis Azure Data Box via NFS | Microsoft Docs
description: Découvrir comment copier des données à partir d’Azure Data Box via NFS
services: databox
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: tutorial
ms.date: 10/01/2020
ms.author: alkohli
ms.openlocfilehash: bd8e6d4175c57bd31c3fd83bf6f9669d2b65ffb2
ms.sourcegitcommit: 487a9f5272300d60df2622c3d13e794d54680f90
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/02/2020
ms.locfileid: "91660839"
---
# <a name="tutorial-copy-data-from-azure-data-box-via-nfs-preview"></a>Tutoriel : Copier des données depuis Azure Data Box via NFS (préversion)

Ce tutoriel explique comment connecter l’interface utilisateur web locale de votre Data Box à un serveur de données local par le biais de NFS et de copier des données à partir de celle-ci. Les données de votre Data Box sont exportées à partir de votre compte de Stockage Azure.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> * Prérequis
> * Se connecter à Data Box
> * Copier des données à partir de la Data Box

[!INCLUDE [Data Box feature is in preview](../../includes/data-box-feature-is-preview-info.md)]

## <a name="prerequisites"></a>Prérequis

Avant de commencer, assurez-vous que :

1. Vous avez passé la commande pour Azure Data Box.
    - Pour une commande d’importation, consultez [Tutoriel : Commander Azure Data Box](data-box-deploy-ordered.md).
    - Pour une commande d’exportation, consultez [Tutoriel : Commander Azure Data Box](data-box-deploy-export-ordered.md).
2. Vous avez reçu votre Data Box et que l’état de la commande dans le portail est **Remis**.
3. Vous disposez d’un ordinateur hôte sur lequel vous voulez copier les données de votre Data Box. Votre ordinateur hôte doit
   * Exécuter un [système d’exploitation pris en charge](data-box-system-requirements.md)
   * Connectez-vous à un réseau haut débit. Nous vous recommandons vivement d’utiliser au minimum une connexion 10 GbE. Si vous ne disposez pas d’une connexion 10 GbE, utilisez une liaison de données 1 GbE. Cependant, les vitesses de copie seront affectées.

## <a name="connect-to-data-box"></a>Se connecter à Data Box

[!INCLUDE [data-box-shares](../../includes/data-box-shares.md)]

Si vous utilisez un ordinateur hôte Linux, procédez comme suit afin de configurer Data Box pour autoriser l’accès aux clients NFS. Data Box peut connecter jusqu’à cinq clients NFS à la fois.

1. Spécifiez les adresses IP des clients autorisés pouvant accéder au partage :

    1.  Dans l’interface utilisateur web locale, accédez à la page **Connexion et copie**. Sous **Paramètres NFS**, cliquez sur **Accès au client NFS**. 

        ![Ouvrir l’accès d’un client NFS](media/data-box-deploy-export-copy-data/nfs-client-access-1.png)

    1. Pour ajouter un client NFS, spécifiez l’adresse IP du client, puis cliquez sur **Ajouter**. Data Box peut connecter jusqu’à cinq clients NFS à la fois. Quand vous avez terminé, cliquez sur **OK**.

         ![Ajouter un client NFS](media/data-box-deploy-export-copy-data/nfs-client-access-2.png)

2. Assurez-vous qu’une [version prise en charge](data-box-system-requirements.md) du client NFS est installée sur l’ordinateur hôte Linux. Utilisez la version spécifiquement adaptée à votre distribution Linux. 

3. Une fois le client NFS installé, utilisez la commande suivante pour monter le partage NFS sur votre appareil Data Box :

    `sudo mount <Data Box device IP>:/<NFS share on Data Box device> <Path to the folder on local Linux computer>`

    L’exemple suivant montre comment se connecter à un partage Data Box via NFS. L’adresse IP de l’appareil Data Box est `10.161.23.130`. Le partage `Mystoracct_Blob` est monté sur la machine virtuelle ubuntuVM avec le point de montage `/home/databoxubuntuhost/databox`.

    `sudo mount -t nfs 10.161.23.130:/Mystoracct_Blob /home/databoxubuntuhost/databox`
    
    Pour les clients Mac, vous devez ajouter une option supplémentaire, comme suit : 
    
    `sudo mount -t nfs -o sec=sys,resvport 10.161.23.130:/Mystoracct_Blob /home/databoxubuntuhost/databox`

    **Toujours créer un dossier pour les fichiers que vous envisagez de copier sous le partage, puis copier les fichiers dans ce dossier**. Le dossier créé sous les partages d’objets blob de pages et d’objets blob de blocs représente un conteneur dans lequel les données sont chargées en tant qu’objets blob. Vous ne pouvez pas copier de fichiers directement dans le dossier *root* du compte de stockage.

## <a name="copy-data-from-data-box"></a>Copier des données à partir de la Data Box

Une fois que vous êtes connecté aux partages Data Box, l’étape suivante consiste à copier les données.

[!INCLUDE [data-box-export-review-logs](../../includes/data-box-export-review-logs.md)]

 Vous pouvez maintenant commencer la copie des données. Si vous utilisez un ordinateur hôte Linux, utilisez un utilitaire de copie similaire à Robocopy. Plusieurs solutions alternatives sont disponibles pour Linux, par exemple, [rsync](https://rsync.samba.org/), [FreeFileSync](https://www.freefilesync.org/), [Unison](https://www.cis.upenn.edu/~bcpierce/unison/) et [Ultracopier](https://ultracopier.first-world.info/).  

La commande `cp` constitue l’une des meilleures options pour copier un répertoire. Pour plus d’informations sur son utilisation, reportez-vous aux [pages man sur cp](http://man7.org/linux/man-pages/man1/cp.1.html).

Si vous utilisez l’option rsync pour une copie multithread, suivez ces instructions :

* Installez le package **CIFS Utils** ou **NFS Utils** selon le système de fichiers utilisé par votre client Linux.

    `sudo apt-get install cifs-utils`

    `sudo apt-get install nfs-utils`

* Installez **Rsync** et **Parallel** (selon la version distribuée de Linux).

    `sudo apt-get install rsync`
   
    `sudo apt-get install parallel` 

* Créez un point de montage.

    `sudo mkdir /mnt/databox`

* Montez le volume.

    `sudo mount -t NFS4  //Databox IP Address/share_name /mnt/databox` 

* Faites une mise en miroir de la structure de répertoires du dossier.  

    `rsync -za --include='*/' --exclude='*' /local_path/ /mnt/databox`

* Copiez les fichiers.

    `cd /local_path/; find -L . -type f | parallel -j X rsync -za {} /mnt/databox/{}`

     où j spécifie le nombre de parallélisations et X = le nombre de copies parallèles

     Nous vous recommandons de commencer avec 16 copies parallèles et d’augmenter le nombre de threads selon les ressources disponibles.

> [!IMPORTANT]
> Les types de fichiers Linux suivants ne sont pas pris en charge : liens symboliques, fichiers de caractères, fichiers de blocs, sockets et pipes. Ces types de fichiers entraînent des échecs au cours de l’étape **Préparer l’expédition**.

Une fois la copie terminée, accédez au **Tableau de bord**, puis vérifiez l’espace utilisé et l’espace libre sur votre appareil.

Vous pouvez maintenant procéder à l’expédition de votre Data Box à Microsoft.

## <a name="next-steps"></a>Étapes suivantes

Ce tutoriel vous a apporté des connaissances concernant Azure Data Box, notamment concernant les points suivants :

> [!div class="checklist"]
>
> * Prérequis
> * Se connecter à Data Box
> * Copier des données à partir de la Data Box

Passez au tutoriel suivant pour découvrir comment renvoyer votre Data Box à Microsoft.

> [!div class="nextstepaction"]
> [Ship your Azure Data Box to Microsoft](./data-box-deploy-export-picked-up.md) (Expédier votre Azure Data Box à Microsoft)
