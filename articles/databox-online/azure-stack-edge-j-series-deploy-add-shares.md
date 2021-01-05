---
title: Tutoriel pour transférer des données sur des partages avec Azure Stack Edge Pro avec GPU | Microsoft Docs
description: Découvrez comment ajouter des partages et s’y connecter sur un appareil Azure Stack Edge Pro avec GPU.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: tutorial
ms.date: 12/22/2020
ms.author: alkohli
Customer intent: As an IT admin, I need to understand how to add and connect to shares on Azure Stack Edge Pro so I can use it to transfer data to Azure.
ms.openlocfilehash: 845eef6e119823af789c9a263bfb750845d55bc0
ms.sourcegitcommit: 44844a49afe8ed824a6812346f5bad8bc5455030
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/23/2020
ms.locfileid: "97740910"
---
# <a name="tutorial-transfer-data-via-shares-with-azure-stack-edge-pro-gpu"></a>Tutoriel : Transférer des données via des partages avec Azure Stack Edge Pro avec GPU

<!--[!INCLUDE [applies-to-skus](../../includes/azure-stack-edge-applies-to-all-sku.md)]-->

Ce tutoriel explique comment ajouter des partages et s’y connecter sur votre appareil Azure Stack Edge Pro. Une fois que vous avez ajouté les partages, Azure Stack Edge Pro peut transférer des données vers Azure.

Cette procédure peut prendre environ dix minutes.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> * Ajouter un partage
> * Se connecter au partage

## <a name="prerequisites"></a>Prérequis

Avant d’ajouter des partages à Azure Stack Edge Pro, vérifiez que :

* Vous avez installé votre appareil physique, comme décrit dans [Installer Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-install.md).

* Vous avez activé l’appareil physique, comme décrit dans [Activer Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-activate.md).

## <a name="add-a-share"></a>Ajouter un partage

Pour créer un partage, procédez comme suit :

1. Sur le [portail Azure](https://portal.azure.com/), sélectionnez votre ressource Azure Stack Edge, puis accédez à la **Vue d’ensemble**. Votre appareil doit être en ligne.

   ![Appareil en ligne](./media/azure-stack-edge-j-series-deploy-add-shares/device-online-1.png)

2. Sélectionnez **+ Ajouter un partage** dans la barre de commandes de l’appareil.

   ![Ajouter un partage](./media/azure-stack-edge-j-series-deploy-add-shares/select-add-share-1.png)

3. Dans le volet **Ajouter un partage**, procédez comme suit :

    a. Dans la zone **Nom**, indiquez un nom unique pour votre partage.  
    Le nom du partage peut comporter uniquement des lettres, des chiffres et des traits d’union. Il doit comporter entre 3 et 63 caractères et commencer par une lettre ou un chiffre. Les traits d’union doivent être précédés et suivis d’une lettre ou d’un chiffre.
    
    b. Sélectionnez un **type** de partage.  
    Le type peut être **SMB** ou **NFS**, SMB étant la valeur par défaut. SMB est la norme pour les clients Windows, tandis que NFS est utilisé pour les clients Linux.  
    Selon que vous choisissez un partage SMB ou NFS, les autres options diffèrent légèrement. 

    c. Fournissez un compte de stockage dans lequel résidera le partage.

    d. Dans la liste déroulante **Service de stockage**, sélectionnez **Objet blob de blocs**, **Objet blob de pages** ou **Fichiers**.  
    Le type de service que vous sélectionnez varie selon le format dans lequel vous souhaitez que les données soient utilisées dans Azure. Dans cet exemple, comme nous voulons que les données soient des objets blob de blocs dans Azure, nous sélectionnons **Objet blob de blocs**. Si vous sélectionnez **Objet blob de pages**, assurez-vous que vos données sont de 512 octets alignés. Par exemple, un VHDX est toujours de 512 octets alignés.

   > [!IMPORTANT]
   > Vérifiez que le compte Stockage Azure que vous utilisez n’a pas de stratégies d’immuabilité définies si vous l’utilisez avec un appareil Azure Stack Edge Pro ou Data Box Gateway. Pour plus d’informations, consultez [Définir et gérer des stratégies d’immuabilité pour le stockage Blob](../storage/blobs/storage-blob-immutability-policies-manage.md).

    e. Créez un conteneur d’objets blob ou utilisez-en un existant dans la liste déroulante. Si vous créez un conteneur d’objets blob, fournissez un nom de conteneur. S’il n’existe pas de conteneur, un conteneur est créé dans le compte de stockage avec le nom du nouveau partage.
   
    f. Selon que vous avez créé un partage SMB ou un partage NFS, effectuez l’une des étapes suivantes : 
     
    - **Partage SMB** : Sous **Utilisateur local avec tous les privilèges**, sélectionnez **Créer** ou **Utiliser l’existant**. Si vous créez un nouvel utilisateur local, entrez un nom d’utilisateur et un mot de passe, puis confirmez le mot de passe. Cette action affecte des autorisations à l’utilisateur local. La modification des autorisations au niveau du partage n’est pas prise en charge. Si vous cochez la case **Autoriser uniquement les opérations de lecture** pour ces données de partage, vous pouvez indiquer des utilisateurs en lecture seule.
    
        ![Ajouter un partage SMB](./media/azure-stack-edge-j-series-deploy-add-shares/add-share-smb-1.png)
   
    - **Partage NFS** : Entrez les adresses IP des clients autorisés pouvant accéder au partage.

        ![Ajouter un partage NFS](./media/azure-stack-edge-j-series-deploy-add-shares/add-share-nfs-1.png)
   
4. Sélectionnez **Créer** pour créer le partage.
    
    Vous êtes averti que le partage est en cours de création. Une fois le partage créé avec les paramètres spécifiés, la vignette **Partages** s’actualise pour refléter le nouveau partage.
    

## <a name="connect-to-the-share"></a>Se connecter au partage

Vous pouvez maintenant vous connecter à un ou plusieurs partages que vous avez créés à l’étape précédente. Selon que vous disposez d’un partage SMB ou NFS, les étapes peuvent varier.

La première étape consiste à s’assurer que le nom de l’appareil peut être résolu lorsque vous utilisez le partage SMB ou NFS.

### <a name="modify-host-file-for-name-resolution"></a>Modifier le fichier hôte pour la résolution de noms

Vous allez maintenant ajouter l’adresse IP de l’appareil et le nom convivial de l’appareil que vous avez définis dans l’interface utilisateur web locale de l’appareil pour :

- Le fichier d’hôte sur le client, OU,
- Le fichier d’hôte sur le serveur DNS

> [!IMPORTANT]
> Nous vous recommandons de modifier le fichier d’hôte sur le serveur DNS pour la résolution du nom de l’appareil.

Sur le client Windows que vous utilisez pour vous connecter à l’appareil, procédez comme suit :

1. Démarrez le **Bloc-notes** en tant qu’administrateur, puis ouvrez le fichier **hosts** qui se trouve dans `C:\Windows\System32\Drivers\etc`.

    ![Fichier hosts de l’Explorateur Windows](media/azure-stack-edge-j-series-deploy-add-shares/client-hosts-file-1.png)


2. Ajoutez l’entrée suivante à votre fichier **hosts** en remplaçant les valeurs appropriées pour votre appareil : 

    ```
    <Device IP>   <device friendly name>
    ``` 
    Vous pouvez récupérer l’adresse IP de l’appareil à partir de la page **Réseau** et le nom convivial de l’appareil à partir de la page **Appareil** de l’interface utilisateur web locale. La capture d’écran suivante du fichier hosts affiche l’entrée :

    ![Fichier hosts 2 dans l’Explorateur Windows](media/azure-stack-edge-j-series-deploy-add-shares/client-hosts-file-2.png)

### <a name="connect-to-an-smb-share"></a>Se connecter à un partage SMB

Sur votre client Windows Server connecté à votre appareil Azure Stack Edge Pro, connectez-vous à un partage SMB en entrant les commandes :


1. Dans la fenêtre de commande, tapez :

    `net use \\<Device name>\<share name>  /u:<user name for the share>`

    > [!NOTE]
    > Vous pouvez vous connecter à un partage SMB uniquement avec le nom de l’appareil et non par le biais de l’adresse IP de l’appareil.

2. Lorsque vous y êtes invité à le faire, entrez le mot de passe du partage.  
   L’exemple de sortie de cette commande est présenté ici.

    ```powershell
    Microsoft Windows [Version 10.0.18363.476)
    (c) 2017 Microsoft Corporation. All rights reserved.
    
    C: \Users\AzureStackEdgeUser>net use \\myasetest1\myasesmbshare1 /u:aseuser
    Enter the password for 'aseuser' to connect to 'myasetest1':
    The command completed successfully.
    
    C: \Users\AzureStackEdgeUser>
    ```   

3. Sur votre clavier, sélectionnez Windows + R.

4. Dans la fenêtre **Exécuter**, spécifiez le(la) `\\<device name>`, puis sélectionnez **OK**.  

    ![Boîte de dialogue Exécuter de Windows](media/azure-stack-edge-j-series-deploy-add-shares/run-window-1.png)

   L’Explorateur de fichiers s’ouvre. Vous devriez maintenant voir les partages que vous avez créés sous forme de dossiers. Dans l’Explorateur de fichiers, double-cliquez sur un partage (dossier) pour afficher son contenu.
 
    ![Se connecter à un partage SMB](./media/azure-stack-edge-j-series-deploy-add-shares/file-explorer-smbshare-1.png)

    Les données sont écrites dans ces partages lors de leur génération et l’appareil envoie les données vers le cloud.

### <a name="connect-to-an-nfs-share"></a>Se connecter à un partage NFS

Sur votre client Linux connecté à votre appareil Azure Stack Edge Pro, appliquez la procédure suivante :

1. Vérifiez que le client NFSv4 est installé sur le client. Pour installer le client NFS, utilisez la commande suivante :

   `sudo apt-get install nfs-common`

    Pour plus d’informations, accédez à [Installer le client NFSv4](https://help.ubuntu.com/community/NFSv4Howto).

2. Une fois le client NFS installé, montez le partage NFS que vous avez créé sur votre appareil Azure Stack Edge Pro à l’aide de la commande suivante :

   `sudo mount -t nfs -o sec=sys,resvport <device IP>:/<NFS share on device> /home/username/<Folder on local Linux computer>`

    Vous pouvez récupérer l’adresse IP de l’appareil à partir de la page **Réseau** de votre interface utilisateur web locale.

    > [!IMPORTANT]
    > L’utilisation de l’option `sync` lors du montage de partages améliore les taux de transfert des fichiers volumineux.
    > Avant de monter le partage, assurez-vous que les répertoires qui serviront de points de montage sur votre ordinateur local sont déjà créés. Ces répertoires ne doivent contenir aucun fichier ou sous-dossier.

    L’exemple suivant montre comment se connecter à un partage sur l’appareil Azure Stack Edge Pro via NFS. L’adresse IP de l’appareil est `10.10.10.60`. Le partage `mylinuxshare2` est monté sur la machine virtuelle ubuntuVM. Le point de montage du partage est `/home/azurestackedgeubuntuhost/edge`.

    `sudo mount -t nfs -o sec=sys,resvport 10.10.10.60:/mylinuxshare2 /home/azurestackedgeubuntuhost/Edge`

> [!NOTE] 
> Les mises en garde suivantes s’appliquent à cette version :
> - Une fois qu’un fichier est créé dans le partage, il ne peut pas être renommé. 
> - La suppression d’un fichier d’un partage ne supprime pas l’entrée dans le compte Stockage Azure.
> - Lorsque vous utilisez `rsync` pour copier sur NFS, utilisez l’indicateur `--inplace`. 

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez approfondi les connaissances suivantes sur Azure Stack Edge Pro :

> [!div class="checklist"]
> * Ajouter un partage
> * Vous connecter à un partage

Pour savoir comment transformer vos données en utilisant Azure Stack Edge Pro, passez au tutoriel suivant :

> [!div class="nextstepaction"]
> [Transformer des données avec Azure Stack Edge Pro](./azure-stack-edge-j-series-deploy-configure-compute.md)