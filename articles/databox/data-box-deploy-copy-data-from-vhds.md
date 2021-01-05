---
title: 'Tutoriel : Copier à partir de disques durs virtuels vers des disques managés'
titleSuffix: Azure Data Box
description: Découvrez comment copier des données à partir de disques durs virtuels de charges de travail de machine virtuelle locales vers votre Azure Data Box
services: databox
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: tutorial
ms.date: 10/20/2019
ms.author: alkohli
ms.openlocfilehash: 1394cf6511a65a0e406e51229953e8666d4d4d8d
ms.sourcegitcommit: 2a8a53e5438596f99537f7279619258e9ecb357a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "94337676"
---
# <a name="tutorial-use-data-box-to-import-data-as-managed-disks-in-azure"></a>Tutoriel : Utiliser Data Box pour importer des données en tant que disque managé dans Azure

Ce didacticiel explique comment utiliser l’Azure Data Box vous migrer des disques durs virtuels locaux vers des disques managés dans Azure. Les disques durs virtuels de machines virtuelles locales sont copiés vers la Data Box en tant qu’objets blob de pages, puis chargés dans Azure en tant que disques managés. Vous pouvez ensuite attacher ces disques managés à des machines virtuelles Azure.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> * Vérifier les conditions préalables
> * Se connecter à Data Box
> * Copier des données sur Data Box

## <a name="prerequisites"></a>Prérequis

Avant de commencer, assurez-vous que :

1. Vous avez terminé le [Tutoriel : Configurer Azure Data Box](data-box-deploy-set-up.md).
2. Vous avez reçu votre Data Box et que l’état de la commande dans le portail est **Remis**.
3. Vous êtes connecté à un réseau haut débit. Nous vous recommandons vivement d’utiliser au minimum une connexion 10 GbE. Si vous ne disposez pas d’une connexion 10 GbE, utilisez une liaison de données 1 GbE. Cela affecte cependant les vitesses de copie.
4. Vous avez pris connaissance des informations suivantes :

    - [tailles de disques managés prises en charge dans la section sur les limites de taille des objets Azure](data-box-limits.md#azure-object-size-limits).
    - [Introduction aux disques managés Azure](../virtual-machines/managed-disks-overview.md). 

5. Vous avez conservé une copie des données sources jusqu’à ce que vous ayez eu la confirmation que le Data Box avait transféré vos données dans le stockage Azure.

## <a name="connect-to-data-box"></a>Se connecter à Data Box

Selon les groupes de ressources spécifiés, Data Box crée un partage pour chaque groupe de ressources associé. Par exemple, si `mydbmdrg1` et `mydbmdrg2` ont été créés au moment de la commande, les partages suivants sont créés :

- `mydbmdrg1_MDisk`
- `mydbmdrg2_MDisk`

Au sein de chaque partage, les trois dossiers suivants, qui correspondent aux conteneurs dans votre compte de stockage, sont créés.

- SSD Premium
- HDD Standard
- SSD Standard

Le tableau suivant présente les chemins d’accès UNC aux partages sur votre Data Box.
 
|        Protocole de connexion           |             Chemin d’accès UNC au partage                                               |
|-------------------|--------------------------------------------------------------------------------|
| SMB |`\\<DeviceIPAddress>\<ResourceGroupName_MDisk>\<PremiumSSD>\file1.vhd`<br> `\\<DeviceIPAddress>\<ResourceGroupName_MDisk>\<StandardHDD>\file2.vhd`<br> `\\<DeviceIPAddress>\<ResourceGroupName_MDisk>\<StandardSSD>\file3.vhd` |  
| NFS |`//<DeviceIPAddress>/<ResourceGroup1_MDisk>/<PremiumSSD>/file1.vhd`<br> `//<DeviceIPAddress>/<ResourceGroupName_MDisk>/<StandardHDD>/file2.vhd`<br> `//<DeviceIPAddress>/<ResourceGroupName_MDisk>/<StandardSSD>/file3.vhd` |

La procédure de connexion varie selon que vous utilisez SMB ou NFS pour la connexion aux partages Azure Data Box.

> [!NOTE]
> Cette fonctionnalité ne prend pas en charge une connexion via REST.

### <a name="connect-to-data-box-via-smb"></a>Se connecter à Data Box via SMB

Si vous utilisez un ordinateur hôte Windows Server, effectuez les étapes suivantes pour vous connecter à la Data Box.

1. La première étape consiste à vous authentifier et à démarrer une session. Accédez à la page **Connect and copy** (Connexion et copie). Cliquez sur **Get credentials** (Obtenir les informations d’identification) afin d’obtenir les informations d’identification requises pour accéder aux partages associés à votre groupe de ressources. Vous pouvez également obtenir les informations d’identification d’accès à partir des **détails de l’appareil** dans le portail Azure.

    > [!NOTE]
    > Les informations d’identification sont identiques pour tous les partages des disques managés.

    ![Se connecter et copier, Obtenir des informations d’identification de partage](media/data-box-deploy-copy-data-from-vhds/get-share-credentials1.png)

2. À partir de la boîte de dialogue **Accéder au partage et copier les données**, copiez les valeurs de **Nom d’utilisateur** et **Mot de passe** correspondant au partage. Cliquez sur **OK**.
    
    ![Se connecter et copier, Copier des informations d’identification de partage](media/data-box-deploy-copy-data-from-vhds/get-share-credentials2.png)

3. Pour accéder aux partages associés à votre ressource (*mydbmdrg1* dans l’exemple suivant) à partir de votre ordinateur hôte, ouvrez une fenêtre de commande. À l’invite de commandes, tapez :

    `net use \\<IP address of the device>\<share name>  /u:<user name for the share>`

    Les chemins d’accès de votre partage UNC dans cet exemple sont les suivants :

    - `\\169.254.250.200\mydbmdrg1_MDisk`
    - `\\169.254.250.200\mydbmdrg2_MDisk`
    
4. Lorsque vous y êtes invité, entrez le mot de passe du partage. L’exemple suivant illustre la connexion à un partage à l’aide de la commande précédente.

    ```
    C:\>net use \\169.254.250.200\mydbmdrgl_MDisk /u:mdisk
    Enter the password for 'mdisk' to connect to '169.254.250.200':
    The command completed successfully.
    C: \>
    ```

5. Appuyez sur Windows + R. Dans la fenêtre **Exécuter**, spécifiez `\\<device IP address>\<ShareName>`. Cliquez sur **OK** pour ouvrir l’Explorateur de fichiers.
    
    ![Se connecter au partage via l’Explorateur de fichiers](media/data-box-deploy-copy-data-from-vhds/connect-shares-file-explorer1.png)

    Vous devriez maintenant voir les dossiers pré-créés suivants dans chaque partage.
    
    ![Se connecter au partage par le biais de l’Explorateur de fichiers, dossiers d’un partage](media/data-box-deploy-copy-data-from-vhds/connect-shares-file-explorer2.png)


### <a name="connect-to-data-box-via-nfs"></a>Se connecter à Data Box via NFS

Si vous utilisez un ordinateur hôte Linux, procédez comme suit afin de configurer Data Box pour autoriser l’accès aux clients NFS.

1. Indiquez les adresses IP des clients autorisés pouvant accéder au partage. Dans l’interface utilisateur web locale, accédez à la page **Connexion et copie**. Sous **Paramètres NFS**, cliquez sur **Accès au client NFS**.

    ![Configurer l’accès du client NFS](media/data-box-deploy-copy-data-from-vhds/nfs-client-access1.png)

2. Indiquez l’adresse IP du client NFS, puis cliquez sur **Ajouter**. Vous pouvez configurer un accès à plusieurs clients NFS en répétant cette étape. Cliquez sur **OK**.

    ![Configurer l’adresse IP du client NFS](media/data-box-deploy-copy-data-from-vhds/nfs-client-access2.png)

2. Assurez-vous qu’une [version prise en charge](data-box-system-requirements.md) du client NFS est installée sur l’ordinateur hôte Linux. Utilisez la version spécifiquement adaptée à votre distribution Linux.

3. Une fois le client NFS installé, utilisez la commande suivante pour monter le partage NFS sur votre appareil Data Box :

    `sudo mount <Data Box device IP>:/<NFS share on Data Box device> <Path to the folder on local Linux computer>`

    L’exemple suivant montre comment se connecter à un partage Data Box via NFS. L’adresse IP de l’appareil Data Box est `169.254.250.200`. Le partage `mydbmdrg1_MDisk` est monté sur la machine virtuelle ubuntuVM avec le point de montage `/home/databoxubuntuhost/databox`.

    `sudo mount -t nfs 169.254.250.200:/mydbmdrg1_MDisk /home/databoxubuntuhost/databox`


## <a name="copy-data-to-data-box"></a>Copier des données sur Data Box

Une fois que vous êtes connecté au serveur de données, l’étape suivante consiste à copier celles-ci. Le fichier de disque dur virtuel est copié vers le compte de stockage intermédiaire en tant qu’objet blob de pages. L’objet blob de page est ensuite converti en un disque managé et déplacé vers un groupe de ressources.

Avant de commencer la copie des données, examinez les considérations suivantes :

- Vous devez toujours copier les disques durs virtuels dans l’un des dossiers pré-créés. Si vous copiez les disques durs virtuels ailleurs que dans ces dossiers ou dans un dossier que vous avez créé vous-même, les disques durs virtuels sont chargés sur le compte de stockage Azure en tant qu’objets blob de pages plutôt qu’en tant que disques managés.
- Seuls les disques durs virtuels fixes peuvent être chargés pour créer des disques managés. Les disques durs virtuels dynamiques et de différenciation, ou les fichiers de disque dur virtuel (VHDX) ne sont pas pris en charge.
- Vous ne pouvez avoir qu’un seul disque managé d’un même nom dans l’ensemble des dossiers pré-créés d’un groupe de ressources. Les disques durs virtuels chargés dans les dossiers pré-créés doivent donc avoir des noms uniques. Assurez-vous que le nom donné n’est pas déjà attribué à un autre disque managé existant dans un groupe de ressources.
- Vérifiez les [limites applicables aux disques managés dans la section sur les limites de taille des objets Azure](data-box-limits.md#azure-object-size-limits).

Selon que vous vous connectez via SMB ou NFS, vous pouvez utiliser :

- [Copier des données via SMB](data-box-deploy-copy-data.md#copy-data-to-data-box)
- [Copier des données via NFS](data-box-deploy-copy-data-via-nfs.md#copy-data-to-data-box)

Attendez la fin des tâches de copie. Avant de passer à l’étape suivante, assurez-vous que les travaux de copie ont été accomplis sans erreurs.

![Aucune erreur dans la page **Connexion et copie**](media/data-box-deploy-copy-data-from-vhds/verify-no-errors-connect-and-copy.png)

Si des erreurs se sont produites durant le processus de copie, téléchargez les journaux à partir de la page **Connexion et copie**.

- Si vous avez copié un fichier qui n’est pas de 512 octets alignés, celui-ci n’est pas chargé en tant qu’objet blob de pages sur votre compte de stockage intermédiaire. Vous verrez une erreur dans les journaux. Supprimez le fichier et copiez un fichier de 512 octets alignés.

- Si vous avez copié un fichier de disque dur virtuel (VHDX) (ces fichiers ne sont pas pris en charge) avec un nom long, vous verrez une erreur dans les journaux.

    ![Erreur dans les journaux de la page **Connexion et copie**](media/data-box-deploy-copy-data-from-vhds/errors-connect-and-copy.png)

    Résolvez les erreurs avant de passer à l’étape suivante.

Pour garantir l’intégrité des données, la somme de contrôle est calculée par le biais d’une fonction inline lors de la copie des données. Une fois la copie terminée, vérifiez l’espace utilisé et l’espace libre sur votre appareil.
    
![Vérifier l’espace libre et l’espace utilisé sur le tableau de bord](media/data-box-deploy-copy-data-from-vhds/verify-used-space-dashboard.png)

Une fois la tâche de copie terminée, vous pouvez accéder à **Préparer l’expédition**.


## <a name="next-steps"></a>Étapes suivantes

Ce tutoriel vous a apporté des connaissances concernant Azure Data Box, notamment concernant les points suivants :

> [!div class="checklist"]
> * Vérifier les conditions préalables
> * Se connecter à Data Box
> * Copier des données sur Data Box


Passez au tutoriel suivant pour découvrir comment renvoyer votre Data Box à Microsoft.

> [!div class="nextstepaction"]
> [Ship your Azure Data Box to Microsoft](./data-box-deploy-picked-up.md) (Expédier votre Azure Data Box à Microsoft)