---
title: 'Tutoriel : Accéder au Stockage Azure en utilisant des informations d’identification SAS - Linux - Azure AD'
description: Didacticiel montrant comment utiliser une identité managée affectée par le système de machine virtuelle Linux pour accéder au Stockage Azure en utilisant des informations d’identification SAP au lieu d’une clé d’accès au compte de stockage.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: daveba
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 11/03/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 3edc63a1532bb6889fc490e400dbb57e7bce10d0
ms.sourcegitcommit: 6a902230296a78da21fbc68c365698709c579093
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93360409"
---
# <a name="tutorial-use-a-linux-vm-system-assigned-identity-to-access-azure-storage-via-a-sas-credential"></a>Tutoriel : Utiliser une identité managée affectée par le système de machine virtuelle Linux pour accéder au Stockage Azure à l’aide d’informations d’identification SAP

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

Ce didacticiel vous indique comment utiliser une identité managée assignée par le système pour une machine virtuelle Linux, afin d’obtenir des informations d’identification SAP (signature d’accès partagé) de stockage. Plus précisément, des [informations d’identification SAP de service](../../storage/common/storage-sas-overview.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#types-of-shared-access-signatures). 

> [!NOTE]
> La clé SAP générée dans ce tutoriel ne sera pas être limitée/liée à la machine virtuelle.  

Une SAP de service offre la possibilité d’accorder un accès limité à des objets dans un compte de stockage, pendant une durée limitée et pour un service spécifique (en l’occurrence, le service BLOB), sans exposer de clé d’accès de compte. Vous pouvez utiliser des informations d’identification SAP comme d’habitude lors de l’exécution d’opérations de stockage, par exemple, lors de l’utilisation du SDK Stockage Azure. Ce didacticiel montre le chargement et le téléchargement d’un objet blob à l’aide de l’interface de ligne de commande de Stockage Azure. Vous apprendrez à :


> [!div class="checklist"]
> * Créez un compte de stockage.
> * Création d’un conteneur d’objets blob dans le compte de stockage
> * Autoriser votre machine virtuelle à accéder à la SAP d’un compte stockage dans le Gestionnaire des ressources 
> * Obtenir un jeton d’accès à l’aide de l’identité de votre machine virtuelle et l’utiliser pour récupérer la SAP à partir du Gestionnaire des ressources 

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [msi-tut-prereqs](../../../includes/active-directory-msi-tut-prereqs.md)]

## <a name="create-a-storage-account"></a>Créez un compte de stockage. 

Si vous n’en avez pas déjà un, vous allez maintenant créer un compte de stockage.  Vous pouvez également ignorer cette étape, et autoriser votre identité managée affectée par le système de machine virtuelle à accéder aux clés d’un compte de stockage existant. 

1. Cliquez sur le bouton **+/Créer un service** dans l’angle supérieur gauche du portail Azure.
2. Cliquez sur **Stockage** , puis **Compte de stockage** , et un nouveau panneau « Créer un compte de stockage » s’affiche.
3. Saisissez un **nom** pour le compte de stockage, vous l’utiliserez ultérieurement.  
4. **Modèle de déploiement** et **Type de compte** doivent être respectivement définis sur « Gestionnaire de ressources » et « Usage général ». 
5. Assurez-vous que les champs **Abonnement** et **Groupe de ressources** correspondent à ceux que vous avez spécifiés lorsque vous avez créé votre machine virtuelle à l’étape précédente.
6. Cliquez sur **Créer**.

    ![Créer un nouveau compte de stockage](./media/msi-tutorial-linux-vm-access-storage/msi-storage-create.png)

## <a name="create-a-blob-container-in-the-storage-account"></a>Création d’un conteneur d’objets blob dans le compte de stockage

Plus tard, nous chargerons et téléchargerons un fichier vers le nouveau compte de stockage. Étant donné que les fichiers nécessitent un stockage d’objets blob, nous devons créer un conteneur d’objets blob dans lequel stocker le fichier.

1. Revenez à votre compte de stockage nouvellement créé.
2. Cliquez sur le lien **Conteneurs** dans le panneau de gauche, sous « Service BLOB ».
3. Cliquez sur **+ Conteneur** en haut de la page et un panneau « Nouveau conteneur » apparait.
4. Nommez le conteneur, sélectionnez un niveau d’accès, puis cliquez sur **OK**. Le nom spécifié sera utilisé plus loin dans le didacticiel. 

    ![Créer un conteneur de stockage](./media/msi-tutorial-linux-vm-access-storage/create-blob-container.png)

## <a name="grant-your-vms-system-assigned-managed-identity-access-to-use-a-storage-sas"></a>Autoriser votre identité managée affectée par le système de machine virtuelle à utiliser une SAP de stockage

Le stockage Azure ne prend pas en charge l’authentification Azure AD en mode natif.  Toutefois, vous pouvez utiliser une identité managée affectée par le système de machine virtuelle pour récupérer une SAP de stockage à partir de Resource Manager, puis utiliser cette SAP pour accéder au stockage.  Dans cette étape, vous autorisez votre identité managée affectée par le système de machine virtuelle à accéder à la SAP de votre compte de stockage.   

1. Revenez à votre compte de stockage nouvellement créé.
2. Cliquez sur le lien **(IAM) de contrôle d’accès** dans le panneau de gauche.  
3. Cliquez sur **+ Ajouter une attribution de rôle** en haut de la page pour ajouter une nouvelle attribution de rôle à votre machine virtuelle.
4. Définissez **Rôle** sur « Contributeur de comptes de stockage » sur le côté droit de la page. 
5. Dans la liste déroulante suivante, définissez **Attribuer l’accès à** sur la ressource « Machine virtuelle ».  
6. Ensuite, assurez-vous que l’abonnement approprié est répertorié dans la liste déroulante **Abonnement** , puis définissez **Groupe de ressources** sur « Tous les groupes de ressources ».  
7. Enfin, sous **Sélectionner** , choisissez votre machine virtuelle Linux dans la liste déroulante, puis cliquez sur **Enregistrer**.  

    ![Texte de remplacement d’image](./media/msi-tutorial-linux-vm-access-storage/msi-storage-role-sas.png)

## <a name="get-an-access-token-using-the-vms-identity-and-use-it-to-call-azure-resource-manager"></a>Obtenir un jeton d’accès à l’aide de l’identité de la machine virtuelle et l’utiliser pour appeler Azure Resource Manager

Pour la suite de ce didacticiel, nous allons utiliser la machine virtuelle que nous avons créée précédemment.

Pour effectuer cette procédure, vous avez besoin d'un client SSH. Si vous utilisez Windows, vous pouvez utiliser le client SSH dans le [Sous-système Windows pour Linux](/windows/wsl/install-win10). Si vous avez besoin d’aide pour configurer les clés de votre client SSH, consultez [Comment utiliser les clés SSH avec Windows sur Azure](../../virtual-machines/linux/ssh-from-windows.md), ou [Comment créer et utiliser une paire de clés publique et privée SSH pour les machines virtuelles Linux dans Azure](../../virtual-machines/linux/mac-create-ssh-keys.md).

1. Dans le portail Azure, accédez à **Machines virtuelles** , accédez à votre machine virtuelle Linux, puis, en haut de la page **Vue d’ensemble** , cliquez sur **Se connecter**. Copiez la chaîne permettant de se connecter à votre machine virtuelle. 
2. Connectez-vous à votre machine virtuelle en utilisant votre client SSH.  
3. Ensuite, vous serez invité à entrer le **Mot de passe** que vous avez ajouté à la création de la **machine virtuelle Linux**. Vous devez être connecté.  
4. Utilisez CURL pour obtenir un jeton d’accès à partir de Azure Resource Manager.  

    La requête et la réponse CURL pour le jeton d’accès se trouvent ci-dessous :
    
    ```bash
    curl 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fmanagement.azure.com%2F' -H Metadata:true    
    ```
    
    > [!NOTE]
    > Dans la requête précédente, la valeur du paramètre « resource » doit correspondre exactement à ce qu’Azure AD attend. Lorsque vous utilisez l’ID de ressource Azure Resource Manager, vous devez inclure la barre oblique de fin à l’URI.
    > Dans la réponse suivante, l’élément access_token a été raccourci par souci de concision.
    
    ```bash
    {"access_token":"eyJ0eXAiOiJ...",
    "refresh_token":"",
    "expires_in":"3599",
    "expires_on":"1504130527",
    "not_before":"1504126627",
    "resource":"https://management.azure.com",
    "token_type":"Bearer"} 
     ```

## <a name="get-a-sas-credential-from-azure-resource-manager-to-make-storage-calls"></a>Obtenir des informations d’identification SAP d’Azure Resource Manager pour effectuer des appels de stockage

À présent, utilisez CURL pour appeler le Gestionnaire des ressources à l’aide du jeton d’accès récupéré dans la section précédente, afin de créer des informations d’identification SAP. Une fois les informations d’identification SAP récupérées, nous pouvons appeler des opérations de chargement/téléchargement de stockage.

Pour cette demande, nous allons utiliser les paramètres de requête HTTP suivants pour créer les informations d’identification SAP :

```JSON
{
    "canonicalizedResource":"/blob/<STORAGE ACCOUNT NAME>/<CONTAINER NAME>",
    "signedResource":"c",              // The kind of resource accessible with the SAS, in this case a container (c).
    "signedPermission":"rcw",          // Permissions for this SAS, in this case (r)ead, (c)reate, and (w)rite.  Order is important.
    "signedProtocol":"https",          // Require the SAS be used on https protocol.
    "signedExpiry":"<EXPIRATION TIME>" // UTC expiration time for SAS in ISO 8601 format, for example 2017-09-22T00:06:00Z.
}
```

Ces paramètres sont inclus dans le corps POST de la requête pour les informations d’identification SAP. Pour plus d’informations concernant les paramètres pour la création d’informations d’identification SAP, voir [Afficher la référence REST de SAP de service](/rest/api/storagerp/storageaccounts/listservicesas).

Utilisez la demande CURL suivante pour obtenir les informations d’identification SAP. N’oubliez pas de remplacer les valeurs des paramètres `<SUBSCRIPTION ID>`, `<RESOURCE GROUP>`, `<STORAGE ACCOUNT NAME>`, `<CONTAINER NAME>` et `<EXPIRATION TIME>` par vos propres valeurs. Remplacez la valeur de `<ACCESS TOKEN>` par le jeton d’accès que vous avez récupéré précédemment :

```bash 
curl https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Storage/storageAccounts/<STORAGE ACCOUNT NAME>/listServiceSas/?api-version=2017-06-01 -X POST -d "{\"canonicalizedResource\":\"/blob/<STORAGE ACCOUNT NAME>/<CONTAINER NAME>\",\"signedResource\":\"c\",\"signedPermission\":\"rcw\",\"signedProtocol\":\"https\",\"signedExpiry\":\"<EXPIRATION TIME>\"}" -H "Authorization: Bearer <ACCESS TOKEN>"
```

> [!NOTE]
> Le texte de l’URL précédente respectant la casse, veillez à respecter les majuscules et les minuscules pour vos groupes de ressources. En outre, il est important de savoir qu’il s’agit d’une demande POST et non d’une demande GET.

La réponse CURL retourne les informations d’identification SAP :  

```bash 
{"serviceSasToken":"sv=2015-04-05&sr=c&spr=https&st=2017-09-22T00%3A10%3A00Z&se=2017-09-22T02%3A00%3A00Z&sp=rcw&sig=QcVwljccgWcNMbe9roAJbD8J5oEkYoq%2F0cUPlgriBn0%3D"} 
```

Créez un exemple de fichier blob à charger dans votre conteneur de stockage d’objets blob. Ceci peut se faire sur une machine virtuelle Linux avec la commande suivante : 

```bash
echo "This is a test file." > test.txt
```

Ensuite, authentifiez-vous à l’aide de la commande `az storage` de l’interface de ligne de commande en utilisant les informations d’identification SAP, puis chargez le fichier dans le conteneur d’objets blob. Pour cette étape, vous devez [installer la dernière version d’Azure CLI](/cli/azure/install-azure-cli) sur votre machine virtuelle, si ce n’est déjà fait.

```azurecli
 az storage blob upload --container-name 
                        --file 
                        --name
                        --account-name 
                        --sas-token
```

Réponse : 

```JSON
Finished[#############################################################]  100.0000%
{
  "etag": "\"0x8D4F9929765C139\"",
  "lastModified": "2017-09-21T03:58:56+00:00"
}
```

Vous pouvez également télécharger le fichier à l’aide d’Azure CLI et vous authentifier avec les informations d’identification SAP. 

Demande : 

```azurecli
az storage blob download --container-name
                         --file 
                         --name 
                         --account-name
                         --sas-token
```

Réponse : 

```JSON
{
  "content": null,
  "metadata": {},
  "name": "testblob",
  "properties": {
    "appendBlobCommittedBlockCount": null,
    "blobType": "BlockBlob",
    "contentLength": 16,
    "contentRange": "bytes 0-15/16",
    "contentSettings": {
      "cacheControl": null,
      "contentDisposition": null,
      "contentEncoding": null,
      "contentLanguage": null,
      "contentMd5": "Aryr///Rb+D8JQ8IytleDA==",
      "contentType": "text/plain"
    },
    "copy": {
      "completionTime": null,
      "id": null,
      "progress": null,
      "source": null,
      "status": null,
      "statusDescription": null
    },
    "etag": "\"0x8D4F9929765C139\"",
    "lastModified": "2017-09-21T03:58:56+00:00",
    "lease": {
      "duration": null,
      "state": "available",
      "status": "unlocked"
    },
    "pageBlobSequenceNumber": null,
    "serverEncrypted": false
  },
  "snapshot": null
}
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à utiliser une identité managée affectée par le système de machine virtuelle Linux pour accéder au Stockage Azure à l’aide des informations d’identification SAP.  Pour en savoir plus sur le Stockage Azure SAP, consultez :

> [!div class="nextstepaction"]
>[Utilisation des signatures d’accès partagé (SAP)](../../storage/common/storage-sas-overview.md)
