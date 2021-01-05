---
title: Tutoriel expliquant comment transférer des données vers des partages avec Azure Stack Edge Pro | Microsoft Docs
description: Ce tutoriel vous explique comment ajouter des partages sur votre appareil Azure Stack Edge Pro et vous y connecter pour permettre à Azure Stack Edge Pro de transférer des données vers Azure.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: tutorial
ms.date: 03/21/2019
ms.author: alkohli
Customer intent: As an IT admin, I need to understand how to add and connect to shares on Azure Stack Edge Pro so I can use it to transfer data to Azure.
ms.openlocfilehash: 64fe65019a66013323b1c6d27055051badb47cee
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96350311"
---
# <a name="tutorial-transfer-data-with-azure-stack-edge-pro"></a>Tutoriel : Transférer des données avec Azure Stack Edge Pro

Ce tutoriel vous explique comment ajouter des partages et vous y connecter sur votre appareil Azure Stack Edge Pro. Une fois les partages ajoutés, Azure Stack Edge Pro peut transférer des données vers Azure.

Cette procédure peut prendre environ dix minutes.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Ajouter un partage
> * Se connecter au partage

 
## <a name="prerequisites"></a>Prérequis

Avant d'ajouter des partages à Azure Stack Edge Pro, vérifiez que :

- Vous avez installé votre appareil physique, comme décrit dans [Installer Azure Stack Edge Pro](azure-stack-edge-deploy-install.md).

- Vous avez activé l'appareil physique, comme décrit dans [Connecter, configurer et activer Azure Stack Edge Pro](azure-stack-edge-deploy-connect-setup-activate.md).


## <a name="add-a-share"></a>Ajouter un partage

Pour créer un partage, procédez comme suit :

1. Sur le [portail Azure](https://portal.azure.com/), sélectionnez votre ressource Azure Stack Edge, puis accédez à la **Vue d’ensemble**. Votre appareil doit être en ligne.

   ![Appareil en ligne](./media/azure-stack-edge-deploy-add-shares/device-online-1.png)

2. Sélectionnez **+ Ajouter un partage** dans la barre de commandes de l’appareil.

   ![Ajouter un partage](./media/azure-stack-edge-deploy-add-shares/select-add-share-1.png)

3. Dans le volet **Ajouter un partage**, procédez comme suit :

    a. Dans la zone **Nom**, indiquez un nom unique pour votre partage.  
    Le nom du partage peut comporter uniquement des lettres minuscules, des chiffres et des traits d’union. Il doit comporter entre 3 et 63 caractères et commencer par une lettre ou un chiffre. Les traits d’union doivent être précédés et suivis d’une lettre ou d’un chiffre.
    
    b. Sélectionnez un **type** de partage.  
    Le type peut être **SMB** ou **NFS**, SMB étant la valeur par défaut. SMB est la norme pour les clients Windows, tandis que NFS est utilisé pour les clients Linux.  
    Selon que vous choisissez un partage SMB ou NFS, les autres options diffèrent légèrement. 

    c. Fournissez un compte de stockage dans lequel résidera le partage. 

    > [!IMPORTANT]
    > Vérifiez que le compte de stockage Azure que vous utilisez ne comporte pas de stratégies d'immuabilité si vous l'utilisez avec un appareil Azure Stack Edge Pro ou Data Box Gateway. Pour plus d’informations, consultez [Définir et gérer des stratégies d’immuabilité pour le stockage Blob](../storage/blobs/storage-blob-immutability-policies-manage.md).
    
    d. Dans la liste déroulante **Service de stockage**, sélectionnez **Objet blob de blocs**, **Objet blob de pages** ou **Fichiers**.  
    Le type de service que vous sélectionnez varie selon le format dans lequel vous souhaitez que les données soient utilisées dans Azure. Dans cet exemple, comme nous voulons que les données soient des objets blob de blocs dans Azure, nous sélectionnons **Objet blob de blocs**. Si vous sélectionnez **Objet blob de pages**, assurez-vous que vos données sont de 512 octets alignés. Par exemple, un VHDX est toujours de 512 octets alignés.

    e. Créez un conteneur d’objets blob ou utilisez-en un existant dans la liste déroulante. Si vous créez un conteneur d’objets blob, fournissez un nom de conteneur. S’il n’existe pas de conteneur, un conteneur est créé dans le compte de stockage avec le nom du nouveau partage.
   
    f. Selon que vous avez créé un partage SMB ou un partage NFS, effectuez l’une des étapes suivantes : 
     
    - **Partage SMB** : Sous **Utilisateur local avec tous les privilèges**, sélectionnez **Créer** ou **Utiliser l’existant**. Si vous créez un nouvel utilisateur local, entrez un nom d’utilisateur et un mot de passe, puis confirmez le mot de passe. Cette action affecte des autorisations à l’utilisateur local. Après avoir attribué les autorisations à cet emplacement, vous pouvez utiliser l’Explorateur de fichiers pour les modifier.

        Si vous cochez la case **Autoriser uniquement les opérations de lecture** pour ces données de partage, vous pouvez indiquer des utilisateurs en lecture seule.

        ![Ajouter un partage SMB](./media/azure-stack-edge-deploy-add-shares/add-share-smb-1.png)
   
    - **Partage NFS** : Entrez les adresses IP des clients autorisés pouvant accéder au partage.

        ![Ajouter un partage NFS](./media/azure-stack-edge-deploy-add-shares/add-share-nfs-1.png)
   
4. Sélectionnez **Créer** pour créer le partage.
    
    Vous êtes averti que le partage est en cours de création. Une fois le partage créé avec les paramètres spécifiés, la vignette **Partages** s’actualise pour refléter le nouveau partage.
    

## <a name="connect-to-the-share"></a>Se connecter au partage

Vous pouvez maintenant vous connecter à un ou plusieurs partages que vous avez créés à l’étape précédente. Selon que vous disposez d’un partage SMB ou NFS, les étapes peuvent varier.

### <a name="connect-to-an-smb-share"></a>Se connecter à un partage SMB

Sur le client Windows Server connecté à votre appareil Azure Stack Edge Pro, connectez-vous à un partage SMB en entrant les commandes suivantes :


1. Dans la fenêtre de commande, tapez :

    `net use \\<IP address of the device>\<share name>  /u:<user name for the share>`

2. Lorsque vous y êtes invité à le faire, entrez le mot de passe du partage.  
   L’exemple de sortie de cette commande est présenté ici.

    ```powershell
    Microsoft Windows [Version 10.0.16299.192)
    (c) 2017 Microsoft Corporation. All rights reserved.
    
    C: \Users\DataBoxEdgeUser>net use \\10.10.10.60\newtestuser /u:Tota11yNewUser
    Enter the password for 'TotallyNewUser' to connect to '10.10.10.60':
    The command completed successfully.
    
    C: \Users\DataBoxEdgeUser>
    ```   


3. Sur votre clavier, sélectionnez Windows + R.

4. Dans la fenêtre **Exécuter**, spécifiez le(la) `\\<device IP address>`, puis sélectionnez **OK**.  
   L’Explorateur de fichiers s’ouvre. Vous devriez maintenant voir les partages que vous avez créés sous forme de dossiers. Dans l’Explorateur de fichiers, double-cliquez sur un partage (dossier) pour afficher son contenu.
 
    ![Se connecter à un partage SMB](./media/azure-stack-edge-deploy-add-shares/connect-to-share2.png)

    Les données sont écrites dans ces partages lors de leur génération et l’appareil envoie les données vers le cloud.

### <a name="connect-to-an-nfs-share"></a>Se connecter à un partage NFS

Sur le client Linux connecté à votre appareil Azure Stack Edge Pro, procédez comme suit :

1. Vérifiez que le client NFSv4 est installé sur le client. Pour installer le client NFS, utilisez la commande suivante :

   `sudo apt-get install nfs-common`

    Pour plus d’informations, accédez à [Installer le client NFSv4](https://help.ubuntu.com/community/SettingUpNFSHowTo#NFSv4_client).

2. Une fois le client NFS installé, montez le partage NFS que vous avez créé sur votre appareil Azure Stack Edge Pro à l'aide de la commande suivante :

   `sudo mount -t nfs -o sec=sys,resvport <device IP>:/<NFS shares on device> /home/username/<Folder on local Linux computer>`

    > [!IMPORTANT]
    > L’utilisation de l’option `sync` lors du montage de partages améliore les taux de transfert des fichiers volumineux.
    > Avant de monter le partage, assurez-vous que les répertoires qui serviront de points de montage sur votre ordinateur local sont déjà créés. Ces répertoires ne doivent contenir aucun fichier ou sous-dossier.

    L'exemple suivant montre comment se connecter à un partage sur l'appareil Azure Stack Edge Pro via NFS. L’adresse IP de l’appareil est `10.10.10.60`. Le partage `mylinuxshare2` est monté sur la machine virtuelle ubuntuVM. Le point de montage du partage est `/home/databoxubuntuhost/edge`.

    `sudo mount -t nfs -o sec=sys,resvport 10.10.10.60:/mylinuxshare2 /home/databoxubuntuhost/Edge`

> [!NOTE] 
> Les mises en garde suivantes s’appliquent à cette version :
> - Une fois qu’un fichier est créé dans le partage, il ne peut pas être renommé. 
> - La suppression d’un fichier d’un partage ne supprime pas l’entrée dans le compte de stockage.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez approfondi vos connaissances sur Azure Stack Edge Pro et avez appris à :

> [!div class="checklist"]
> * Ajouter un partage
> * Vous connecter à un partage

Pour savoir comment transformer vos données à l'aide d'Azure Stack Edge Pro, passez au tutoriel suivant :

> [!div class="nextstepaction"]
> [Transformer des données avec Azure Stack Edge Pro](./azure-stack-edge-deploy-configure-compute.md)