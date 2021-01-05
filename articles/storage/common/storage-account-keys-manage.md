---
title: Gérer les touches d’accès rapide au compte
titleSuffix: Azure Storage
description: Découvrez comment consulter, gérer et permuter les clés d’accès de vos comptes de stockage.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 04/24/2020
ms.author: tamram
ms.custom: devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: e5ea94fea00771b64634d6c28a7879fabb195f09
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89069657"
---
# <a name="manage-storage-account-access-keys"></a>Gérer les clés d’accès au compte de stockage

Quand vous créez un compte de stockage, Azure génère deux clés d’accès 512 bits pour ce compte. Ces clés peuvent être utilisées pour autoriser l’accès aux données de votre compte de stockage par le biais de l’autorisation de clé partagée.

Microsoft vous recommande d’utiliser Azure Key Vault pour gérer vos clés d’accès, ainsi que de permuter et de regénérer régulièrement vos clés. L’utilisation d’Azure Key Vault facilite la permutation de vos clés sans interruption de vos applications. Vous pouvez également permuter manuellement vos clés.

[!INCLUDE [storage-account-key-note-include](../../../includes/storage-account-key-note-include.md)]

## <a name="view-account-access-keys"></a>Afficher les clés d’accès au compte

Vous pouvez afficher et copier les clés d’accès à votre compte avec le portail Azure, PowerShell ou Azure CLI. Le portail Azure fournit également une chaîne de connexion pour votre compte de stockage, que vous pouvez copier.

# <a name="portal"></a>[Portail](#tab/azure-portal)

Pour afficher et copier les clés d’accès ou la chaîne de connexion de votre compte de stockage à partir du portail Azure :

1. Accédez à votre compte de stockage dans le [Portail Azure](https://portal.azure.com).
1. Sous **Paramètres**, sélectionnez **Clés d’accès**. Vos clés d’accès au compte s’affichent, ainsi que la chaîne de connexion complète de chaque clé.
1. Recherchez la valeur de **Key** sous **key1**, puis cliquez sur le bouton **Copier** pour copier la clé du compte.
1. Vous pouvez aussi copier la chaîne de connexion complète. Recherchez la valeur de **Chaîne de connexion** sous **clé1**, puis cliquez sur le bouton **Copier** pour copier la chaîne de connexion.

    :::image type="content" source="media/storage-account-keys-manage/portal-connection-string.png" alt-text="Capture d’écran montrant comment afficher les clés d’accès dans le portail Azure":::

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Pour récupérer les clés d’accès à votre compte avec PowerShell, appelez la commande [Get-AzStorageAccountKey](/powershell/module/az.Storage/Get-azStorageAccountKey).

L’exemple suivant récupère la première clé. Pour récupérer la deuxième, utilisez `Value[1]` au lieu de `Value[0]`. N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs.

```powershell
$storageAccountKey = `
    (Get-AzStorageAccountKey `
    -ResourceGroupName <resource-group> `
    -Name <storage-account>).Value[0]
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour répertorier les clés d’accès à votre compte avec Azure CLI, appelez la commande [az storage account keys list](/cli/azure/storage/account/keys#az-storage-account-keys-list), comme dans l’exemple suivant. N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs. 

```azurecli-interactive
az storage account keys list \
  --resource-group <resource-group> \
  --account-name <storage-account>
```

---

Vous pouvez utiliser la clé de votre choix pour accéder au Stockage Azure, mais il est en général conseillé d’utiliser la première et de réserver la deuxième pour le rotation des clés.

Pour visualiser ou lire les clés d’accès d’un compte, l’utilisateur doit être administrateur de service ou titulaire d’un rôle Azure qui comprend **Microsoft.Storage/storageAccounts/listkeys/action**. Voici quelques rôles AZURE intégrés qui incluent cette action : **Propriétaire**, **Contributeur** et **Rôle de service d’opérateur de clé de compte de stockage**. Pour plus d’informations sur le rôle d’administrateur de service, consultez [Rôles d’administrateur d’abonnement classique, rôles Azure et rôles Azure AD](../../role-based-access-control/rbac-and-directory-admin-roles.md). Pour plus d’informations sur les rôles intégrés pour Stockage Azure, consultez la section **Stockage** dans [Rôles intégrés Azure pour Azure RBAC](../../role-based-access-control/built-in-roles.md#storage).

## <a name="use-azure-key-vault-to-manage-your-access-keys"></a>Utiliser Azure Key Vault pour gérer vos clés d’accès

Microsoft recommande d’utiliser Azure Key Vault pour gérer et permuter vos clés d’accès. Votre application peut accéder de manière sécurisée à vos clés dans Key Vault, afin que vous puissiez éviter de les stocker avec votre code d’application. Pour plus d’informations sur l’utilisation de Key Vault pour la gestion des clés, consultez les articles suivants :

- [Gérer les clés de compte de stockage avec Azure Key Vault et PowerShell](../../key-vault/secrets/overview-storage-keys-powershell.md)
- [Gérer les clés de compte de stockage avec Azure Key Vault et Azure CLI](../../key-vault/secrets/overview-storage-keys.md)

## <a name="manually-rotate-access-keys"></a>Permuter des clés d’accès manuellement

Microsoft vous recommande de permuter vos clés d’accès régulièrement pour garantir une sécurité optimale de votre compte de stockage. Si possible, utilisez Azure Key Vault pour gérer vos clés d’accès. Si vous n’utilisez pas Key Vault, vous devrez permuter vos clés manuellement.

Deux clés d’accès sont assignées pour vous permettre de les permuter. L’existence de deux clés permet de garantir que votre application maintient l’accès au Stockage Azure tout au long du processus.

> [!WARNING]
> La regénération des clés d’accès peut impacter des applications ou des services Azure qui ont une dépendance avec la clé du compte de stockage. Tous les clients qui utilisent la clé de compte pour accéder au compte de stockage doivent être mis à jour afin d’utiliser la nouvelle clé, y compris les services multimédias, les applications cloud, de bureau et mobiles, et les applications d’interface utilisateur graphique pour le stockage Azure, comme [l’Explorateur Stockage Azure](https://azure.microsoft.com/features/storage-explorer/).

# <a name="portal"></a>[Portail](#tab/azure-portal)

Pour opérer la rotation des clés d’accès de votre compte de stockage dans le portail Azure :

1. Mettez à jour les chaînes de connexion dans votre code d’application pour désigner la clé d’accès secondaire du compte de stockage.
1. Accédez à votre compte de stockage dans le [Portail Azure](https://portal.azure.com).
1. Sous **Paramètres**, sélectionnez **Clés d’accès**.
1. Pour régénérer la clé d’accès primaire de votre compte de stockage, sélectionnez le bouton **Régénérer** en regard de la clé d’accès primaire.
1. Mettez à jour les chaînes de connexion dans votre code pour désigner la nouvelle clé d’accès principale.
1. Régénérez la clé d’accès secondaire de la même manière.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Pour opérer la rotation des clés d’accès de votre compte de stockage avec PowerShell :

1. Mettez à jour les chaînes de connexion dans votre code d’application pour désigner la clé d’accès secondaire du compte de stockage.
1. Appelez la commande [New-AzStorageAccountKey](/powershell/module/az.storage/new-azstorageaccountkey) pour régénérer la clé d’accès primaire, comme illustré dans l’exemple suivant :

    ```powershell
    New-AzStorageAccountKey -ResourceGroupName <resource-group> `
      -Name <storage-account> `
      -KeyName key1
    ```

1. Mettez à jour les chaînes de connexion dans votre code pour désigner la nouvelle clé d’accès principale.
1. Régénérez la clé d’accès secondaire de la même manière. Pour régénérer la clé secondaire, utilisez `key2` comme nom de clé à la place de `key1`.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour opérer la rotation des clés d’accès de votre compte de stockage avec Azure CLI :

1. Mettez à jour les chaînes de connexion dans votre code d’application pour désigner la clé d’accès secondaire du compte de stockage.
1. Appelez la commande [az storage account keys renew](/cli/azure/storage/account/keys#az-storage-account-keys-renew) pour régénérer la clé d’accès primaire, comme illustré dans l’exemple suivant :

    ```azurecli-interactive
    az storage account keys renew \
      --resource-group <resource-group> \
      --account-name <storage-account>
      --key primary
    ```

1. Mettez à jour les chaînes de connexion dans votre code pour désigner la nouvelle clé d’accès principale.
1. Régénérez la clé d’accès secondaire de la même manière. Pour régénérer la clé secondaire, utilisez `key2` comme nom de clé au lieu de `key1`.

---

> [!NOTE]
> Microsoft recommande d’utiliser uniquement l’une des clés dans toutes vos applications en même temps. Si vous utilisez parfois la clé 1 et parfois la clé 2, vous ne pouvez effectuer aucune rotation de vos clés sans qu’une application ne perde l’accès.

Pour effectuer la rotation des clés d’accès d’un compte, l’utilisateur doit être administrateur de service ou titulaire d’un rôleAzure qui comprend **Microsoft.Storage/storageAccounts/regeneratekey/action**. Voici quelques rôles AZURE intégrés qui incluent cette action : **Propriétaire**, **Contributeur** et **Rôle de service d’opérateur de clé de compte de stockage**. Pour plus d’informations sur le rôle d’administrateur de service, consultez [Rôles d’administrateur d’abonnement classique, rôles Azure et rôles Azure AD](../../role-based-access-control/rbac-and-directory-admin-roles.md). Pour plus d’informations sur les rôles Azure intégrés pour Stockage Azure, consultez la section **Stockage** dans [Rôles intégrés Azure pour Azure RBAC](../../role-based-access-control/built-in-roles.md#storage).

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble des comptes de stockage Azure](storage-account-overview.md)
- [Créez un compte de stockage](storage-account-create.md)
