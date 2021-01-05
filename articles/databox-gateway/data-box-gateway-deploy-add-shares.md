---
title: Transférer des données avec Azure Data Box Gateway | Microsoft Docs
description: Découvrez comment ajouter des partages et vous y connecter sur votre Azure Data Box Gateway. Votre appareil Data Box Gateway pourra ensuite transférer des données vers Azure.
services: databox
author: alkohli
ms.service: databox
ms.subservice: gateway
ms.topic: tutorial
ms.date: 10/14/2020
ms.author: alkohli
ms.openlocfilehash: aaa75988e1177590a0ff6f4ca2e00f92de68789f
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96580362"
---
# <a name="tutorial-transfer-data-with-azure-data-box-gateway"></a>Tutoriel : Transférer des données avec Azure Data Box Gateway


## <a name="introduction"></a>Introduction

Cet article explique comment ajouter et se connecter à des partages sur votre appareil Data Box Gateway. Une fois que vous avez ajouté les partages, l’appareil est en mesure de transférer des données vers Azure.

Cette procédure peut prendre environ dix minutes.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> * Ajouter un partage
> * Vous connecter à un partage

## <a name="prerequisites"></a>Prérequis

Avant d’ajouter des partages à votre appareil Data Box Gateway, assurez-vous que :

- Vous avez provisionné un appareil virtuel et vous y êtes connecté comme indiqué dans [Provisionner Azure Data Box Gateway dans Hyper-V](data-box-gateway-deploy-provision-hyperv.md) ou [Provisionner Azure Data Box Gateway dans VMware](data-box-gateway-deploy-provision-vmware.md).

- Vous avez activé l’appareil virtuel comme décrit dans [Connecter et activer votre appareil Azure Data Box Gateway](data-box-gateway-deploy-connect-setup-activate.md).

- L’appareil est prêt pour que vous puissiez créer des partages et transférer des données.

## <a name="add-a-share"></a>Ajouter un partage

Pour créer un partage, procédez comme suit :

1. Dans le [portail Azure](https://portal.azure.com/), sélectionnez votre ressource Data Box Gateway, puis accédez à **Vue d’ensemble**. Votre appareil doit être en ligne. Sélectionnez **+ Ajouter un partage** dans la barre de commandes de l’appareil.
   
   ![Ajouter un partage](./media/data-box-gateway-deploy-add-shares/click-add-share.png)

4. Dans **Ajouter un partage**, procédez comme suit :

    1. Indiquez un nom unique pour votre partage. Les noms de partage peuvent uniquement contenir des lettres minuscules, des traits d’union et des chiffres. Le nom de partage doit contenir entre 3 et 63 caractères et commencer par une lettre ou un chiffre. Chaque trait d’union doit être précédé et suivi d’un caractère autre qu’un tiret.
    
    2. Sélectionnez un **type** de partage. Le type peut être SMB ou NFS, SMB étant la valeur par défaut. SMB est la norme pour les clients Windows, tandis que NFS est utilisé pour les clients Linux. Selon que vous choisissez un partage SMB ou NFS, les options proposées diffèrent légèrement.

    3. Fournissez un compte de stockage dans lequel résidera le partage. S’il n’existe pas de conteneur, un conteneur est créé dans le compte de stockage avec le nom du nouveau partage. Si le conteneur existe déjà, ce conteneur est utilisé.
       > [!IMPORTANT]
       > Vérifiez que le compte Stockage Azure que vous utilisez n’a pas de stratégies d’immuabilité définies dans le cadre d’une utilisation avec un appareil Data Box Gateway. Pour plus d’informations, consultez [Définir et gérer des stratégies d’immuabilité pour le stockage Blob](https://docs.microsoft.com/azure/storage/blobs/storage-blob-immutability-policies-manage).
    
    4. Choisissez le **service de stockage** d’objet blob de blocs, d’objet blob de pages ou de fichier. Le type de service choisi varie selon le format dans lequel vous souhaitez que les données résident dans Azure. Par exemple, dans cette instance, nous voulons que les données résident en tant qu’objets blob de blocs dans Azure. C’est pourquoi nous sélectionnons Objet blob de blocs. Si vous choisissez un objet blob de pages, vous devez vous assurer que vos données sont de 512 octets alignés. Par exemple, un VHDX est toujours de 512 octets alignés.
   
    5. Cette étape varie selon que vous créez un partage SMB ou NFS.
     
    - **Partage SMB** : sous **Utilisateur local avec tous les privilèges**, sélectionnez **Créer** ou **Utiliser l’existant**. Si vous créez un utilisateur local, entrez un **nom d’utilisateur** et un **mot de passe**, puis **confirmez le mot de passe**. Cette action affecte les autorisations à l’utilisateur local. La modification des autorisations au niveau du partage n’est pas prise en charge.
    
        ![Ajouter un partage SMB](./media/data-box-gateway-deploy-add-shares/add-share-smb-1.png)
        
        Si vous cochez la case **Autoriser uniquement les opérations de lecture** pour ces données de partage, vous pouvez indiquer des utilisateurs en lecture seule.
        
    - **Partage NFS** : indiquez les adresses IP des clients autorisés pouvant accéder au partage.

        ![Ajouter un partage NFS](./media/data-box-gateway-deploy-add-shares/add-share-nfs-1.png)
   
9. Sélectionnez **Créer** pour créer le partage.
    
    Vous êtes averti que le partage est en cours de création. Une fois le partage créé avec les paramètres spécifiés, la vignette **Partages** s’actualise pour refléter le nouveau partage.
    
    ![Vignette Partages mise à jour](./media/data-box-gateway-deploy-add-shares/updated-list-of-shares.png) 

## <a name="connect-to-the-share"></a>Se connecter au partage

Vous pouvez maintenant vous connecter à un ou plusieurs partages que vous avez créés à l’étape précédente. Selon que vous disposez d’un partage SMB ou NFS, les étapes peuvent varier.

### <a name="connect-to-an-smb-share"></a>Se connecter à un partage SMB

Sur votre client Windows Server connecté à votre appareil Data Box Gateway, connectez-vous à un partage SMB en entrant les commandes :


1. Dans la fenêtre de commande, tapez :

    `net use \\<IP address of the device>\<share name>  /u:<user name for the share>`

    Lorsque vous y êtes invité, entrez le mot de passe du partage. L’exemple de sortie de cette commande est présenté ici.

    ```powershell
    Microsoft Windows [Version 18.8.16299.192) 
    (c) 2017 microsoft Corporation. All rights reserved . 
    
    C: \Users\GatewayUser>net use \\10.10.10.60\newtestuser /u:Tota11yNewUser 
    Enter the password for 'TotallyNewUser' to connect to '10.10.10.60'  
    The command completed successfully. 
    
    C: \Users\GatewayUser>
    ```   


2. Sur votre clavier, sélectionnez Windows + R. 
3. Dans la fenêtre **Exécuter**, spécifiez l’adresse `\\<device IP address>`, puis sélectionnez **OK**. L’Explorateur de fichiers s’ouvre. Vous devriez maintenant voir les partages que vous avez créés sous forme de dossiers. Dans l’Explorateur de fichiers, double-cliquez sur un partage (dossier) pour afficher son contenu.
 
    ![Se connecter à un partage SMB](./media/data-box-gateway-deploy-add-shares/connect-to-share2.png)-->

    Les données sont écrites dans ces partages lors de leur génération et l’appareil envoie les données vers le cloud.

### <a name="connect-to-an-nfs-share"></a>Se connecter à un partage NFS

Sur votre client Linux connecté à votre appareil, procédez comme suit :

1. Vérifiez que le client NFSv4 est installé sur le client. Pour installer le client NFS, utilisez la commande suivante :

   `sudo apt-get install nfs-common`

    Pour plus d’informations, accédez à [Installer le client NFSv4](https://help.ubuntu.com/community/SettingUpNFSHowTo#NFSv4_client).

2. Une fois le client NFS installé, utilisez la commande suivante pour monter le partage NFS que vous avez créé sur votre appareil Data Box Gateway :

   `sudo mount -t nfs -o sec=sys,resvport <device IP>:/<NFS shares on device> /home/username/<Folder on local Linux computer>`

    Avant de configurer les montages, assurez-vous que les répertoires qui agiront comme points de montage sur votre ordinateur local sont déjà créés et ne contiennent pas de fichiers ou de sous-dossiers.

    L’exemple suivant montre comment se connecter à un partage sur l’appareil Gateway via NFS. L’IP de l’appareil virtuel est `10.10.10.60`, le partage `mylinuxshare2` est monté sur la machine virtuelle ubuntu, le point de montage étant `/home/databoxubuntuhost/gateway`.

    `sudo mount -t nfs -o sec=sys,resvport 10.10.10.60:/mylinuxshare2 /home/databoxubuntuhost/gateway`

> [!NOTE] 
> Les mises en garde suivantes s’appliquent à cette version :
> - Une fois qu’un fichier est créé dans les partages, vous ne pouvez plus le renommer.
> - La suppression d’un fichier à partir d’un partage ne supprime pas l’entrée du compte de stockage.
> - Si vous utilisez `rsync` pour copier des données, l’option `rsync -a` n’est pas prise en charge.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez approfondi vos connaissances sur Data Box Gateway et avez notamment appris à :

> [!div class="checklist"]
> * Ajouter un partage
> * Vous connecter à un partage


Passez au didacticiel suivant pour découvrir comment administrer votre appareil Data Box Gateway.

> [!div class="nextstepaction"]
> [Utiliser l’interface utilisateur web locale pour administrer un appareil Data Box Gateway](https://aka.ms/dbg-docs)