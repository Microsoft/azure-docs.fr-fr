---
title: Tutoriel pour copier des données sur Azure Data Box Heavy par le biais de NFS | Microsoft Docs
description: Dans ce tutoriel, découvrez comment établir une connexion entre votre ordinateur hôte et Azure Data Box Heavy par le biais de NFS avec l’interface utilisateur web locale.
services: databox
author: alkohli
ms.service: databox
ms.subservice: heavy
ms.topic: tutorial
ms.date: 07/03/2019
ms.author: alkohli
ms.openlocfilehash: a8199039ea803d5919e168edccaaa03c18985725
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87921025"
---
# <a name="tutorial-copy-data-to-azure-data-box-heavy-via-nfs"></a>Tutoriel : Copier des données sur Azure Data Box Heavy par le biais de NFS

Ce tutoriel explique comment établir une connexion à votre ordinateur hôte et copier des données de cet ordinateur sur votre Azure Data Box Heavy à l’aide de l’interface utilisateur web locale.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Prérequis
> * Se connecter à Data Box Heavy
> * Copier des données sur Data Box Heavy

## <a name="prerequisites"></a>Prérequis

Avant de commencer, assurez-vous que :

1. Vous avez terminé le [Tutoriel : Configurer Azure Data Box Heavy](data-box-heavy-deploy-set-up.md).
2. Vous avez reçu votre Data Box Heavy et l’état de la commande dans le portail est **Livré**.
3. Vous disposez d’un ordinateur hôte contenant les données que vous souhaitez copier sur Data Box Heavy. Votre ordinateur hôte doit
    - Exécuter un [système d’exploitation pris en charge](data-box-heavy-system-requirements.md)
    - Connectez-vous à un réseau haut débit. Pour obtenir des vitesses de copie plus rapides, il est possible d’utiliser deux connexions 40 GbE (une par nœud) en parallèle. Si aucune connexion 40-GbE n’est disponible, nous vous recommandons d’avoir au moins deux connexions 10 GbE (une par nœud). 

## <a name="connect-to-data-box-heavy"></a>Se connecter à Data Box Heavy

En fonction du compte de stockage sélectionné, Data Box Heavy crée jusqu’à :
- Trois partages pour chaque compte de stockage associé pour GPv1 et GPv2.
- Un partage pour le stockage Premium.
- Un partage pour le compte de stockage Blob.

Ces partages sont créés sur les deux nœuds de l’appareil.

Sous les partages d’objet blob de blocs et d’objet blob de pages :
- Les entités de premier niveau sont des conteneurs.
- Les entités de second niveau sont des objets blob.

Sous les partages Azure Files :
- Les entités de premier niveau sont des partages.
- Les entités de second niveau sont des fichiers.

Le tableau suivant montre le chemin UNC des partages sur votre Data Box Heavy et l’URL du chemin du Stockage Azure où les données sont chargées. La dernière URL de chemin de Stockage Azure peut être dérivée à partir du chemin de partage UNC.
 
| Stockage           | Chemin UNC                                                                       |
|-------------------|--------------------------------------------------------------------------------|
| Objets blob de blocs Azure | <li>Chemin UNC aux partages : `//<DeviceIPAddress>/<StorageAccountName_BlockBlob>/<ContainerName>/files/a.txt`</li><li>URL de Stockage Azure : `https://<StorageAccountName>.blob.core.windows.net/<ContainerName>/files/a.txt`</li> |  
| Objets blob de pages Azure  | <li>Chemin UNC aux partages : `//<DeviceIPAddres>/<StorageAccountName_PageBlob>/<ContainerName>/files/a.txt`</li><li>URL de Stockage Azure : `https://<StorageAccountName>.blob.core.windows.net/<ContainerName>/files/a.txt`</li>   |  
| Azure Files       |<li>Chemin UNC aux partages : `//<DeviceIPAddres>/<StorageAccountName_AzFile>/<ShareName>/files/a.txt`</li><li>URL de Stockage Azure : `https://<StorageAccountName>.file.core.windows.net/<ShareName>/files/a.txt`</li>        |

Si vous utilisez un ordinateur hôte Linux, effectuez les étapes suivantes afin de configurer votre appareil pour autoriser l’accès aux clients NFS.

1. Indiquez les adresses IP des clients autorisés pouvant accéder au partage. Dans l’interface utilisateur web locale, accédez à la page **Connect and copy** (Connexion et copie). Sous **Paramètres NFS**, cliquez sur **Accès au client NFS**. 

    ![Configurer l’accès au client NFS 1](media/data-box-deploy-copy-data/nfs-client-access.png)

2. Indiquez l’adresse IP du client NFS, puis cliquez sur **Ajouter**. Vous pouvez configurer un accès à plusieurs clients NFS en répétant cette étape. Cliquez sur **OK**.

    ![Configurer l’accès au client NFS 2](media/data-box-deploy-copy-data/nfs-client-access2.png)

2. Assurez-vous qu’une [version prise en charge](data-box-system-requirements.md) du client NFS est installée sur l’ordinateur hôte Linux. Utilisez la version spécifiquement adaptée à votre distribution Linux. 

3. Une fois le client NFS installé, utilisez la commande suivante pour monter le partage NFS sur votre appareil Data Box :

    `sudo mount <Data Box Heavy device IP>:/<NFS share on Data Box Heavy device> <Path to the folder on local Linux computer>`

    L’exemple suivant montre comment se connecter à un partage Data Box Heavy par le biais de NFS. L’adresse IP de Data Box Heavy est `10.161.23.130`. Le partage `Mystoracct_Blob` est monté sur la machine virtuelle ubuntuVM avec le point de montage `/home/databoxheavyubuntuhost/databoxheavy`.

    `sudo mount -t nfs 10.161.23.130:/Mystoracct_Blob /home/databoxheavyubuntuhost/databoxheavy`
    
    Pour les clients Mac, vous devez ajouter une option supplémentaire, comme suit : 
    
    `sudo mount -t nfs -o sec=sys,resvport 10.161.23.130:/Mystoracct_Blob /home/databoxheavyubuntuhost/databoxheavy`

    **Toujours créer un dossier pour les fichiers que vous envisagez de copier sous le partage, puis copier les fichiers dans ce dossier**. Le dossier créé sous les partages d’objets blob de pages et d’objets blob de blocs représente un conteneur dans lequel les données sont chargées en tant qu’objets blob. Vous ne pouvez pas copier de fichiers directement dans le dossier *root* du compte de stockage.

## <a name="copy-data-to-data-box-heavy"></a>Copier des données sur Data Box Heavy

Une fois que vous êtes connecté aux partages Data Box Heavy, l’étape suivante consiste à copier des données. Avant de commencer la copie des données, passez en revue les considérations suivantes :

- Assurez-vous que les données sont copiées vers des partages compatibles avec le format des données. Par exemple, les données d’objet blob de blocs doivent être copiées dans le partage des objets blob de blocs. Copiez les disques durs virtuels dans des objets blob de pages. Si le format des données ne correspond pas au type de partage, les données ne pourront pas être chargées dans Azure.
-  Quand vous copiez des données, vérifiez que leur taille est conforme aux limites de taille spécifiées dans l’article [Limitations relatives au Stockage Azure et à Data Box Heavy](data-box-heavy-limits.md). 
- Si les données qui sont en cours de chargement par Data Box Heavy sont chargées simultanément par d’autres applications en dehors de Data Box Heavy, cela peut entraîner l’échec du chargement ou une altération des données.
- Nous vous recommandons de ne pas utiliser SMB et NFS simultanément et de ne pas copier les mêmes données vers la même destination finale sur Azure. En effet, le résultat final ne pourrait être déterminé.
- **Toujours créer un dossier pour les fichiers que vous envisagez de copier sous le partage, puis copier les fichiers dans ce dossier**. Le dossier créé sous les partages d’objets blob de pages et d’objets blob de blocs représente un conteneur dans lequel les données sont chargées en tant qu’objets blob. Vous ne pouvez pas copier de fichiers directement dans le dossier *root* du compte de stockage.
- Si vous ingérez des noms de répertoires et de fichiers sensibles à la casse à partir d’un partage NFS vers NFS sur Data Box Heavy : 
    - La casse est conservée dans le nom.
    - Les fichiers sont sensibles à la casse.
    
    Par exemple, si vous copiez `SampleFile.txt` et `Samplefile.Txt`, la casse est conservée dans le nom lors de la copie dans l’appareil, mais le second fichier remplace le premier car ils sont considérés comme le même fichier.


Si vous utilisez un ordinateur hôte Linux, utilisez un utilitaire de copie similaire à Robocopy. Plusieurs solutions alternatives sont disponibles pour Linux, par exemple, [rsync](https://rsync.samba.org/), [FreeFileSync](https://www.freefilesync.org/), [Unison](https://www.cis.upenn.edu/~bcpierce/unison/) et [Ultracopier](https://ultracopier.first-world.info/).  

La commande `cp` constitue l’une des meilleures options pour copier un répertoire. Pour plus d’informations sur son utilisation, reportez-vous aux [pages man sur cp](http://man7.org/linux/man-pages/man1/cp.1.html).

Si vous utilisez l’option rsync pour une copie multithread, suivez ces instructions :

 - Installez le package **CIFS Utils** ou **NFS Utils** selon le système de fichiers utilisé par votre client Linux.

    `sudo apt-get install cifs-utils`

    `sudo apt-get install nfs-utils`

 -  Installez **Rsync** et **Parallel** (selon la version distribuée de Linux).

    `sudo apt-get install rsync`
   
    `sudo apt-get install parallel` 

 - Créez un point de montage.

    `sudo mkdir /mnt/databoxheavy`

 - Montez le volume.

    `sudo mount -t NFS4  //Databox-heavy-IP-Address/share_name /mnt/databoxheavy` 

 - Faites une mise en miroir de la structure de répertoires du dossier.  

    `rsync -za --include='*/' --exclude='*' /local_path/ /mnt/databoxheavy`

 - Copiez les fichiers. 

    `cd /local_path/; find -L . -type f | parallel -j X rsync -za {} /mnt/databoxheavy/{}`

     où j spécifie le nombre de parallélisations et X = le nombre de copies parallèles

     Nous vous recommandons de commencer avec 16 copies parallèles et d’augmenter le nombre de threads selon les ressources disponibles.

> [!IMPORTANT]
> Les types de fichiers Linux suivants ne sont pas pris en charge : liens symboliques, fichiers de caractères, fichiers de blocs, sockets et pipes. Ces types de fichiers entraînent des échecs au cours de l’étape **Préparer l’expédition**.

Ouvrez le dossier cible pour afficher et vérifier les fichiers copiés. Si vous rencontrez des erreurs au cours du processus de copie, téléchargez les fichiers d’erreurs pour résoudre les problèmes. Pour plus d’informations, consultez [Afficher les journaux d’erreurs pendant la copie de données vers Data Box Heavy](data-box-logs.md#view-error-log-during-data-copy). Pour obtenir une liste détaillée des erreurs lors de la copie de données, consultez [Résolution les problèmes liés à Data Box Heavy](data-box-troubleshoot.md).

Pour garantir l’intégrité des données, la somme de contrôle est calculée par le biais d’une fonction inline lors de la copie des données. Une fois la copie terminée, vérifiez l’espace utilisé et l’espace libre sur votre appareil.
    
   ![Vérifier l’espace libre et l’espace utilisé sur le tableau de bord](media/data-box-deploy-copy-data/verify-used-space-dashboard.png)


## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez approfondi vos connaissances sur Azure Data Box Heavy, notamment concernant les points suivants :

> [!div class="checklist"]
> * Prérequis
> * Se connecter à Data Box Heavy
> * Copier des données sur Data Box Heavy


Passez au tutoriel suivant pour découvrir comment renvoyer votre Data Box à Microsoft.

> [!div class="nextstepaction"]
> [Expédier votre Azure Data Box Heavy à Microsoft](./data-box-heavy-deploy-picked-up.md)

