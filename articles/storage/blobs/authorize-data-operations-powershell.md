---
title: Exécuter des commandes PowerShell avec des informations d’identification Azure AD pour accéder aux données Blob
titleSuffix: Azure Storage
description: PowerShell prend en charge la connexion avec des informations d’identification Azure AD pour exécuter des commandes sur les données Blob dans Stockage Azure. Un jeton d’accès est fourni pour la session et utilisé pour autoriser les opérations d’appel. Les autorisations dépendent du rôle Azure attribué au principal de sécurité Azure AD.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 09/14/2020
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: blobs
ms.openlocfilehash: de42854f512a69b7b70a15a10fe8499ca1d0bf8b
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96017727"
---
# <a name="run-powershell-commands-with-azure-ad-credentials-to-access-blob-data"></a>Exécuter des commandes PowerShell avec des informations d’identification Azure AD pour accéder aux données Blob

Stockage Azure fournit des extensions pour PowerShell qui vous permettent de vous connecter et d’exécuter des commandes de script avec des informations d’identification Azure Active Directory (Azure AD). Lorsque vous vous connectez à PowerShell avec les informations d’identification Azure AD, un jeton d’accès OAuth 2.0 est retourné. Ce jeton est utilisé automatiquement par PowerShell pour autoriser les opérations de données suivantes sur Stockage Blob. Pour les opérations prises en charge, vous n’avez plus besoin de fournir une clé de compte ou le jeton SAP avec la commande.

Vous pouvez attribuer des autorisations sur les données Blob à un principal de sécurité Azure AD via le contrôle d’accès Azure en fonction du rôle (RBAC). Pour plus d’informations sur les rôles Azure dans Stockage Azure, consultez [Gérer les droits d’accès aux données Stockage Azure avec Azure RBAC](../common/storage-auth-aad-rbac-portal.md).

## <a name="supported-operations"></a>Opérations prises en charge

Les extensions Stockage Azure sont prises en charge pour les opérations sur les données Blob. Les opérations que vous pouvez appeler dépendent des autorisations accordées au principal de sécurité Azure AD avec lequel vous vous connectez à PowerShell. Les autorisations sur les conteneurs Stockage Azure sont attribuées via Azure RBAC. Par exemple, si le rôle **Lecteur des données Blob** vous a été attribué, vous pouvez exécuter des commandes de script qui lisent les données d’un conteneur. Si le rôle **Contributeur aux données Blob** vous a été attribué, vous pouvez exécuter des commandes de script qui lisent, écrivent ou suppriment un conteneur ou les données qu’il contient.

Pour plus d’informations sur les autorisations requises pour chaque opération Stockage Azure sur un conteneur, consultez [Appeler des opérations de stockage avec des jetons OAuth](/rest/api/storageservices/authorize-with-azure-active-directory#call-storage-operations-with-oauth-tokens).  

## <a name="call-powershell-commands-using-azure-ad-credentials"></a>Appeler des commandes PowerShell à l’aide des informations d’identification Azure AD

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Pour utiliser Azure PowerShell pour vous connecter et exécuter les opérations suivantes sur le service Stockage Azure à l’aide des informations d’identification Azure AD, créez un contexte de stockage pour faire référence au compte de stockage en incluant le paramètre `-UseConnectedAccount`.

L’exemple suivant montre comment créer un conteneur dans un nouveau compte de stockage à partir d’Azure PowerShell à l’aide de vos informations d’identification Azure AD. N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs :

1. Connectez-vous à votre compte Azure à l’aide de la commande [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount) :

    ```powershell
    Connect-AzAccount
    ```

    Pour plus d’informations sur la connexion à Azure avec PowerShell, consultez [Se connecter avec Azure PowerShell](/powershell/azure/authenticate-azureps).

1. Créez un groupe de ressources Azure en appelant [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). 

    ```powershell
    $resourceGroup = "sample-resource-group-ps"
    $location = "eastus"
    New-AzResourceGroup -Name $resourceGroup -Location $location
    ```

1. Créez un compte de stockage en appelant [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount).

    ```powershell
    $storageAccount = New-AzStorageAccount -ResourceGroupName $resourceGroup `
      -Name "<storage-account>" `
      -SkuName Standard_LRS `
      -Location $location `
    ```

1. Obtenez le contexte du compte de stockage qui spécifie le nouveau compte de stockage en appelant [New-AzStorageContext](/powershell/module/az.storage/new-azstoragecontext). Si un compte de stockage est utilisé, vous pouvez référencer le contexte au lieu d’entrer les informations d’identification à plusieurs reprises. Incluez le paramètre `-UseConnectedAccount` pour appeler toutes les opérations de données suivantes à l’aide de vos informations d’identification Azure AD :

    ```powershell
    $ctx = New-AzStorageContext -StorageAccountName "<storage-account>" -UseConnectedAccount
    ```

1. Avant de créer le conteneur, attribuez-vous le rôle [Contributeur aux données Blob du stockage](../../role-based-access-control/built-in-roles.md#storage-blob-data-contributor). Même si vous êtes le propriétaire du compte, vous avez besoin d’autorisations explicites pour effectuer des opérations de données sur le compte de stockage. Pour plus d’informations sur l’attribution de rôles Azure, consultez [Utiliser le Portail Azure afin d’attribuer un rôle Azure pour l’accès aux données de blob et de file d’attente](../common/storage-auth-aad-rbac-portal.md).

    > [!IMPORTANT]
    > La propagation des attributions de rôles Azure peut prendre plusieurs minutes.

1. Créez un conteneur en appelant [New-AzStorageContainer](/powershell/module/az.storage/new-azstoragecontainer). Étant donné que cet appel utilise le contexte créé lors des étapes précédentes, le conteneur est créé à l’aide de vos informations d’identification Azure AD.

    ```powershell
    $containerName = "sample-container"
    New-AzStorageContainer -Name $containerName -Context $ctx
    ```

## <a name="next-steps"></a>Étapes suivantes

- [Utiliser PowerShell pour attribuer un rôle Azure pour l’accès aux données de blob et de file d’attente](../common/storage-auth-aad-rbac-powershell.md)
- [Autoriser l’accès à des données blob et de files d’attente avec des identités managées pour les ressources Azure](../common/storage-auth-aad-msi.md)