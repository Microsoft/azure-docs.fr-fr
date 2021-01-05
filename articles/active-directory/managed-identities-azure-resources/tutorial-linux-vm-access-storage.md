---
title: Tutoriel `:` Utiliser une identité managée pour accéder au Stockage Azure - Linux - Azure AD
description: Ce didacticiel vous guide tout au long du processus consistant à utiliser une identité managée affectée par le système de machine virtuelle Linux pour accéder au Stockage Azure.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 10/23/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: c093dcff46676dc5f8a25974c3c38c74ae7666b7
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92546685"
---
# <a name="tutorial-use-a-linux-vm-system-assigned-managed-identity-to-access-azure-storage"></a>Didacticiel : Utiliser une identité managée affectée par le système de machine virtuelle Linux pour accéder au Stockage Azure 

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

Ce didacticiel vous indique comment utiliser une identité managée affectée par le système pour une machine virtuelle Linux afin d’accéder au service Stockage Azure. Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Créez un compte de stockage.
> * Créer un conteneur d’objets blob dans un compte de stockage
> * Accorder à l’identité gérée de la machine virtuelle Linux l’accès à un conteneur de stockage Azure
> * Obtenir un jeton d’accès et l’utiliser pour appeler le stockage Azure

> [!NOTE]
> L’authentification Azure Active Directory pour le stockage Azure est en préversion publique.

## <a name="prerequisites"></a>Conditions préalables requises

[!INCLUDE [msi-tut-prereqs](../../../includes/active-directory-msi-tut-prereqs.md)]

Pour exécuter les exemples de script CLI dans ce didacticiel, vous avez deux possibilités :

- Utiliser [Azure Cloud Shell](~/articles/cloud-shell/overview.md) dans le portail Azure ou via le bouton **Essayer** situé en haut à droite de chaque bloc de code.
- [Installer la dernière version de CLI 2.0](/cli/azure/install-azure-cli) (2.0.23 ou ultérieure) si vous préférez utiliser une console CLI locale.

## <a name="create-a-storage-account"></a>Créez un compte de stockage. 

Dans cette section, vous créez un compte de stockage. 

1. Cliquez sur le bouton **+ Créer une ressource** dans le coin supérieur gauche du portail Azure.
2. Cliquez sur **Stockage** , puis sur **Compte de stockage - blob, fichier, table, file d’attente**.
3. Sous **Nom** , entrez un nom pour le compte de stockage.  
4. **Modèle de déploiement** et **Type de compte** doivent être définis sur **Gestionnaire des ressources** et **Storage (general purpose v1)** (Stockage (usage général v1)). 
5. Assurez-vous que les champs **Abonnement** et **Groupe de ressources** correspondent à ceux que vous avez spécifiés lorsque vous avez créé votre machine virtuelle à l’étape précédente.
6. Cliquez sur **Créer**.

    ![Créer un nouveau compte de stockage](./media/msi-tutorial-linux-vm-access-storage/msi-storage-create.png)

## <a name="create-a-blob-container-and-upload-a-file-to-the-storage-account"></a>Créer un conteneur d’objets blob et charger un fichier vers le compte de stockage

Les fichiers nécessitent un stockage d’objets blob, vous devez donc créer un conteneur d’objets blob dans lequel stocker le fichier. Vous chargez ensuite un fichier vers le conteneur d’objets blob dans le nouveau compte de stockage.

1. Revenez à votre compte de stockage nouvellement créé.
2. Sous **Service Blob** , cliquez sur **Conteneurs**.
3. Cliquez sur **+ Conteneur** en haut de la page.
4. Sous **Nouveau conteneur** , entrez un nom pour le conteneur puis, sous **Public access level** (Niveau d’accès public), conservez la valeur par défaut.

    ![Créer un conteneur de stockage](./media/msi-tutorial-linux-vm-access-storage/create-blob-container.png)

5. À l’aide de l’éditeur de votre choix, créez un fichier intitulé *hello world.txt* sur votre ordinateur local.  Ouvrez le fichier et ajoutez le texte (sans les guillemets) « Hello world! :) », puis enregistrez-le. 

6. Chargez le fichier vers le conteneur nouvellement créé en cliquant sur le nom du conteneur, puis sur **Charger**
7. Dans le volet **Charger l’objet blob** , sous **Fichiers** , cliquez sur l’icône de dossier et recherchez le fichier **hello_world.txt** sur votre ordinateur local, sélectionnez le fichier, puis cliquez sur **Charger**.

    ![Charger un fichier texte](./media/msi-tutorial-linux-vm-access-storage/upload-text-file.png)

## <a name="grant-your-vm-access-to-an-azure-storage-container"></a>Accorder à votre machine virtuelle l’accès au conteneur de stockage Azure 

Vous pouvez utiliser les identités gérées de la machine virtuelle pour récupérer les données dans l’objet blob de stockage Azure.

>[!NOTE]
> Pour plus d’informations sur les différents rôles que vous pouvez utiliser pour accorder des autorisations sur le stockage, consultez [Autoriser l’accès aux objets blob et aux files d’attente avec Azure Active Directory](../../storage/common/storage-auth-aad.md#assign-azure-roles-for-access-rights).

1. Revenez à votre compte de stockage nouvellement créé.  
2. Cliquez sur le lien **(IAM) de contrôle d’accès** dans le panneau de gauche.  
3. Cliquez sur **+ Ajouter une attribution de rôle** en haut de la page pour ajouter une nouvelle attribution de rôle à votre machine virtuelle.
4. Sous **Rôle** , dans la liste déroulante, sélectionnez **Lecteur des données Blob du stockage**. 
5. Dans la liste déroulante suivante, sous **Attribuer l’accès à** , choisissez **Machine virtuelle**.  
6. Ensuite, assurez-vous que l’abonnement approprié est répertorié dans la liste déroulante **Abonnement** , puis définissez **Groupe de ressources** sur **Tous les groupes de ressources**.  
7. Sous **Sélectionner** , choisissez votre machine virtuelle, puis cliquez sur **Enregistrer**.

    ![Affecter des autorisations](./media/tutorial-linux-vm-access-storage/access-storage-perms.png)

## <a name="get-an-access-token-and-use-it-to-call-azure-storage"></a>Obtenir un jeton d’accès et l’utiliser pour appeler le stockage Azure

Le stockage Azure prend en charge l’authentification Azure AD en mode natif, il peut donc accepter directement des jetons d’accès obtenus à l’aide d’une identité gérée. Cela fait partie de l’intégration du stockage Azure avec Azure AD, et diffère de la fourniture d’informations d’identification sur la chaîne de connexion.

Pour effectuer les étapes suivantes, vous devez travailler depuis la machine virtuelle créée précédemment. Il vous faut également un client SSH pour vous y connecter. Si vous utilisez Windows, vous pouvez utiliser le client SSH dans le [Sous-système Windows pour Linux](/windows/wsl/about). Si vous avez besoin d’aide pour configurer les clés de votre client SSH, consultez [Comment utiliser les clés SSH avec Windows sur Azure](~/articles/virtual-machines/linux/ssh-from-windows.md), ou [Comment créer et utiliser une paire de clés publique et privée SSH pour les machines virtuelles Linux dans Azure](~/articles/virtual-machines/linux/mac-create-ssh-keys.md).

1. Dans le portail Azure, accédez à **Machines virtuelles** , accédez à votre machine virtuelle Linux, puis sur la page **Vue d’ensemble** , cliquez sur **Se connecter**. Copiez la chaîne permettant de se connecter à votre machine virtuelle.
2. **Connectez-vous** à la machine virtuelle à l’aide du client SSH de votre choix. 
3. Dans la fenêtre de terminal, envoyez une requête contenant CURL au point de terminaison de l’identité gérée local en vue d’obtenir un jeton d’accès pour le stockage Azure.
    
    ```bash
    curl 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fstorage.azure.com%2F' -H Metadata:true
    ```
4. Utilisez à présent le jeton d’accès pour vous rendre sur le stockage Azure, par exemple pour y lire le contenu de l’exemple de fichier précédemment chargé sur le conteneur. Remplacez les valeurs de `<STORAGE ACCOUNT>`, `<CONTAINER NAME>` et `<FILE NAME>` par les valeurs que vous avez spécifiées auparavant, et `<ACCESS TOKEN>` par le jeton retourné à l’étape précédente.

   ```bash
   curl https://<STORAGE ACCOUNT>.blob.core.windows.net/<CONTAINER NAME>/<FILE NAME> -H "x-ms-version: 2017-11-09" -H "Authorization: Bearer <ACCESS TOKEN>"
   ```

   La réponse contient le contenu du fichier :

   ```bash
   Hello world! :)
   ```

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à activer une identité managée affectée par le système de machine virtuelle Linux pour accéder au Stockage Azure.  Pour en savoir plus sur le stockage Azure, consultez :

> [!div class="nextstepaction"]
> [Stockage Azure](../../storage/common/storage-introduction.md)