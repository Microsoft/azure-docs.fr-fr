---
title: Utiliser PowerShell pour créer une SAS de délégation d’utilisateur pour un conteneur ou un objet blob
titleSuffix: Azure Storage
description: Découvrez comment créer une SAP de délégation d’utilisateur avec des informations d’identification Azure Active Directory à l’aide de PowerShell.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 12/18/2019
ms.author: tamram
ms.reviewer: dineshm
ms.subservice: blobs
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 875b2a9f35562dd8f0d5df3c631e5ade1e3fbf75
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91714529"
---
# <a name="create-a-user-delegation-sas-for-a-container-or-blob-with-powershell"></a>Créer une SAP de délégation d’utilisateur pour un conteneur ou un objet blob avec PowerShell

[!INCLUDE [storage-auth-sas-intro-include](../../../includes/storage-auth-sas-intro-include.md)]

Cet article explique comment utiliser les informations d’identification Azure Active Directory (Azure AD) pour créer une SAP de délégation d’utilisateur pour un conteneur ou un objet blob avec Azure PowerShell.

[!INCLUDE [storage-auth-user-delegation-include](../../../includes/storage-auth-user-delegation-include.md)]

## <a name="install-the-powershell-module"></a>Installer le module PowerShell

Pour créer une SAP de délégation d’utilisateur avec PowerShell, installez la version 1.10.0 ou ultérieure du module Az.Storage. Pour installer la dernière version du module, procédez comme suit :

1. Désinstallez les installations précédentes d’Azure PowerShell :

    - Supprimez toutes les anciennes installations d’Azure PowerShell de Windows à l’aide du paramètre **Applications et fonctionnalités** situé sous **Paramètres**.
    - Supprimez tous les modules **Azure** de `%Program Files%\WindowsPowerShell\Modules`.

1. Vérifiez que la dernière version de PowerShellGet est installée. Ouvrez une fenêtre Windows PowerShell et exécutez la commande suivante pour installer la dernière version :

    ```powershell
    Install-Module PowerShellGet –Repository PSGallery –Force
    ```

1. Fermez, puis rouvrez la fenêtre PowerShell après l’installation de PowerShellGet.

1. Installez la dernière version d’Azure PowerShell :

    ```powershell
    Install-Module Az –Repository PSGallery –AllowClobber
    ```

1. Assurez-vous qu’Azure PowerShell version 3.2.0 ou ultérieure est installé. Exécutez la commande suivante pour installer la dernière version du module PowerShell de Stockage Azure :

    ```powershell
    Install-Module -Name Az.Storage -Repository PSGallery -Force
    ```

1. Fermez, puis rouvrez la fenêtre PowerShell.

Pour identifier quelle version du module Az.Storage est installée, exécutez la commande suivante :

```powershell
Get-Module -ListAvailable -Name Az.Storage -Refresh
```

Pour en savoir plus sur l’installation d’Azure PowerShell, voir [Installer Azure PowerShell avec PowerShellGet](/powershell/azure/install-az-ps).

## <a name="sign-in-to-azure-powershell-with-azure-ad"></a>Connectez-vous à Azure PowerShell avec Azure AD

Appelez la commande [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount) pour vous connecter avec votre compte Azure AD :

```powershell
Connect-AzAccount
```

Pour plus d’informations sur la connexion avec PowerShell, consultez [Se connecter avec Azure PowerShell](/powershell/azure/authenticate-azureps).

## <a name="assign-permissions-with-azure-rbac"></a>Attribuer des autorisations avec Azure RBAC

Pour créer une SAP de délégation d’utilisateur à partir d’Azure PowerShell, le compte Azure AD utilisé pour se connecter à PowerShell doit se voir attribuer un rôle incluant l’action **Microsoft.Storage/storageAccounts/blobServices/generateUserDelegationKey**. Cette autorisation permet au compte Azure AD de demander la *clé de délégation d’utilisateur*. La clé de délégation d’utilisateur est utilisée pour signer les SAP de délégation d’utilisateur. Le rôle qui fournit l’action **Microsoft. Storage/storageAccounts/blobServices/generateUserDelegationKey** doit être attribué au niveau du compte de stockage, du groupe de ressources ou de l’abonnement. Pour plus d’informations sur les autorisations Azure RBAC pour la création d’une SAP de délégation d’utilisateur, consultez la section **Attribuer des autorisations avec Azure RBAC** dans [Créer une SAP de délégation d’utilisateur](/rest/api/storageservices/create-user-delegation-sas).

Si vous ne disposez pas des autorisations suffisantes pour attribuer des rôles Azure au principal de sécurité Azure AD, vous devrez peut-être demander au propriétaire du compte ou à l’administrateur d’octroyer les autorisations nécessaires.

L’exemple suivant attribue le rôle de **Contributeur de données d’objet blob de stockage**, qui inclut l’action **Microsoft. Storage/storageAccounts/blobServices/generateUserDelegationKey**. Le rôle est étendu au niveau du compte de stockage.

N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs :

```powershell
New-AzRoleAssignment -SignInName <email> `
    -RoleDefinitionName "Storage Blob Data Contributor" `
    -Scope  "/subscriptions/<subscription>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>"
```

Pour plus d’informations sur les rôles intégrés qui incluent l’action **Microsoft. Storage/storageAccounts/blobServices/generateUserDelegationKey**, consultez [Rôles intégrés Azure](../../role-based-access-control/built-in-roles.md).

## <a name="use-azure-ad-credentials-to-secure-a-sas"></a>Utiliser des informations d’identification Azure AD pour sécuriser une SAP

Lorsque vous créez une SAP de délégation d’utilisateur avec Azure PowerShell, la clé de délégation d’utilisateur utilisée pour signer la SAP est créée de manière implicite. L’heure de début et l’heure d’expiration que vous spécifiez pour la SAP sont également utilisées comme heure de début et heure d’expiration pour la clé de délégation d’utilisateur. 

Étant donné que la clé de délégation d’utilisateur est valide au maximum pendant 7 jours à partir de la date de début, vous devez spécifier pour la SAP un délai d’expiration se situant dans les 7 jours après l’heure de début. La SAP n’est pas valide après l’expiration de la clé de délégation d’utilisateur. Par conséquent, une SAP dont le délai d’expiration est supérieur à 7 jours ne sera toujours valide que pendant 7 jours.

Pour créer une SAP de délégation d’utilisateur pour un conteneur ou un objet blob avec Azure PowerShell, commencez par créer un objet de contexte de stockage Azure, en spécifiant le paramètre `-UseConnectedAccount`. Le paramètre `-UseConnectedAccount` spécifie que la commande crée l’objet de contexte sous le compte Azure AD avec lequel vous vous êtes connecté.

N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs :

```powershell
$ctx = New-AzStorageContext -StorageAccountName <storage-account> -UseConnectedAccount
```

### <a name="create-a-user-delegation-sas-for-a-container"></a>Créer une SAP de délégation d’utilisateur pour un conteneur

Pour retourner un jeton SAS de délégation d’utilisateur pour un conteneur, appelez la commande [New-AzStorageContainerSASToken](/powershell/module/az.storage/new-azstoragecontainersastoken) en passant dans l’objet de contexte de stockage Azure que vous avez créé précédemment.

L’exemple suivant retourne un jeton SAS de délégation d’utilisateur pour un conteneur. N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs :

```powershell
New-AzStorageContainerSASToken -Context $ctx `
    -Name <container> `
    -Permission racwdl `
    -ExpiryTime <date-time>
```

Le jeton SAS de délégation d’utilisateur retourné sera semblable à ce qui suit :

```output
?sv=2018-11-09&sr=c&sig=<sig>&skoid=<skoid>&sktid=<sktid>&skt=2019-08-05T22%3A24%3A36Z&ske=2019-08-07T07%3A
00%3A00Z&sks=b&skv=2018-11-09&se=2019-08-07T07%3A00%3A00Z&sp=rwdl
```

### <a name="create-a-user-delegation-sas-for-a-blob"></a>Créer une SAP de délégation d’utilisateur pour un objet blob

Pour retourner un jeton SAS de délégation d’utilisateur pour un objet blob, appelez la commande [New-AzStorageBlobSASToken](/powershell/module/az.storage/new-azstorageblobsastoken) en passant dans l’objet de contexte de stockage Azure que vous avez créé précédemment.

La syntaxe suivante retourne une SAP de délégation d’utilisateur pour un objet blob. L’exemple spécifie le paramètre `-FullUri`, qui retourne l’URI de l’objet blob auquel le jeton SAS est ajouté. N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs :

```powershell
New-AzStorageBlobSASToken -Context $ctx `
    -Container <container> `
    -Blob <blob> `
    -Permission racwd `
    -ExpiryTime <date-time>
    -FullUri
```

L’URI de SAP de délégation d’utilisateur retournée sera semblable à ce qui suit :

```output
https://storagesamples.blob.core.windows.net/sample-container/blob1.txt?sv=2018-11-09&sr=b&sig=<sig>&skoid=<skoid>&sktid=<sktid>&skt=2019-08-06T21%3A16%3A54Z&ske=2019-08-07T07%3A00%3A00Z&sks=b&skv=2018-11-09&se=2019-08-07T07%3A00%3A00Z&sp=racwd
```

> [!NOTE]
> Une SAP de délégation d’utilisateur ne prend pas en charge la définition d’autorisations avec une stratégie d’accès stockée.

## <a name="revoke-a-user-delegation-sas"></a>Révoquer une SAP de délégation d’utilisateur

Pour révoquer une SAP de délégation d’utilisateur d’Azure PowerShell, appelez la commande **Revoke-AzStorageAccountUserDelegationKeys**. Cette commande révoque toutes les clés de délégation d’utilisateur associées au compte de stockage spécifié. Toutes les signatures d’accès partagé associées à ces clés sont invalidées.

N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs :

```powershell
Revoke-AzStorageAccountUserDelegationKeys -ResourceGroupName <resource-group> `
    -StorageAccountName <storage-account>
```

> [!IMPORTANT]
> Les attributions de rôles Azure et de clés de délégation utilisateur sont mises en cache par le stockage Azure. Il peut donc y avoir un certain délai entre le moment où vous lancez le processus de révocation et celui où la SAP de délégation utilisateur devient non valide.

## <a name="next-steps"></a>Étapes suivantes

- [Créer une SAP de délégation d’utilisateur (API REST)](/rest/api/storageservices/create-user-delegation-sas)
- [Obtenir une opération de clé de délégation d’utilisateur](/rest/api/storageservices/get-user-delegation-key)
