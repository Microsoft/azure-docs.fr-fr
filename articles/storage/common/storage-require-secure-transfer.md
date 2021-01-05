---
title: Exiger un transfert sécurisé pour garantir des connexions sécurisées
titleSuffix: Azure Storage
description: Découvrez comment exiger un transfert sécurisé pour les demandes vers le stockage Azure. Lorsque vous avez besoin d’un transfert sécurisé pour un compte de stockage, toutes les demandes provenant d’une connexion non sécurisée sont rejetées.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 04/21/2020
ms.author: tamram
ms.reviewer: fryu
ms.subservice: common
ms.custom: devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: 22e012c36f5c2c6f195a7e3b21afe9001a4cad0d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89077957"
---
# <a name="require-secure-transfer-to-ensure-secure-connections"></a>Exiger un transfert sécurisé pour garantir des connexions sécurisées

Vous pouvez configurer votre compte de stockage pour accepter les demandes émanant des connexions sécurisées uniquement en définissant la propriété **Transfert sécurisé requis** pour le compte de stockage. Lorsque vous avez besoin d’un transfert sécurisé, toutes les demandes provenant d’une connexion non sécurisée sont rejetées. Microsoft vous recommande de toujours exiger un transfert sécurisé pour tous vos comptes de stockage.

Lorsque le transfert sécurisé est exigé, un appel vers une opération de l’API REST de stockage Azure doit être effectué via HTTPS. Toute requête effectuée via HTTP est rejetée.

La connexion à un partage de fichiers Azure via SMB sans chiffrement échoue lorsque le transfert sécurisé est requis pour le compte de stockage. Parmi les exemples de connexions non sécurisées figurent celles effectuées sur SMB 2.1, SMB 3.0 sans chiffrement ou certaines versions du client SMB Linux.

Par défaut, la propriété **Transfert sécurisé requis** est activée quand vous créez un compte de stockage.

> [!NOTE]
> Étant donné que Stockage Azure ne prend pas en charge le protocole HTTPS pour les noms de domaine personnalisés, cette option n’est pas appliquée lorsque vous utilisez un nom de domaine personnalisé. En outre, les comptes de stockage classiques ne sont pas pris en charge.

## <a name="require-secure-transfer-in-the-azure-portal"></a>Exiger un transfert sécurisé dans le portail Azure

Vous pouvez activer la propriété **Transfert sécurisé requis** lorsque vous créez un compte de stockage dans le [portail Azure](https://portal.azure.com). Vous pouvez également l’activer pour les comptes de stockage existants.

### <a name="require-secure-transfer-for-a-new-storage-account"></a>Exiger un transfert sécurisé pour un nouveau compte de stockage

1. Ouvrez le volet **Créer un compte de stockage** dans le portail Azure.
1. Sous **Transfert sécurisé requis**, sélectionnez **Activé**.

   ![Panneau Créer un compte de stockage](./media/storage-require-secure-transfer/secure_transfer_field_in_portal_en_1.png)

### <a name="require-secure-transfer-for-an-existing-storage-account"></a>Exiger un transfert sécurisé pour un compte de stockage existant

1. Sélectionnez un compte de stockage existant dans le portail Azure.
1. Dans le volet de menu du compte de stockage, sous **PARAMÈTRES**, sélectionnez **Configuration**.
1. Sous **Transfert sécurisé requis**, sélectionnez **Activé**.

   ![Volet de menu du compte de stockage](./media/storage-require-secure-transfer/secure_transfer_field_in_portal_en_2.png)

## <a name="require-secure-transfer-from-code"></a>Exiger un transfert sécurisé à partir du code

Pour exiger un transfert sécurisé par programme, définissez la propriété _enableHttpsTrafficOnly_ sur _True_ sur le compte de stockage. Vous pouvez définir cette propriété à l’aide de l’API REST du fournisseur de ressources de stockage, de bibliothèques clientes ou d’outils :

* [REST API](/rest/api/storagerp/storageaccounts)
* [PowerShell](/powershell/module/az.storage/set-azstorageaccount)
* [INTERFACE DE LIGNE DE COMMANDE](/cli/azure/storage/account)
* [NodeJS](https://www.npmjs.com/package/azure-arm-storage/)
* [Kit de développement logiciel (SDK) .NET](https://www.nuget.org/packages/Microsoft.Azure.Management.Storage)
* [Kit de développement logiciel (SDK) Python](https://pypi.org/project/azure-mgmt-storage)
* [Kit de développement logiciel (SDK) Ruby](https://rubygems.org/gems/azure_mgmt_storage)

## <a name="require-secure-transfer-with-powershell"></a>Exiger un transfert sécurisé avec PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Cet exemple nécessite le module Az Azure PowerShell version 0.7 ou ultérieure. Exécutez `Get-Module -ListAvailable Az` pour trouver la version. Si vous devez installer ou mettre à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-Az-ps).

Exécutez `Connect-AzAccount` pour créer une connexion avec Azure.

 Utilisez la ligne de commande ci-dessous pour vérifier le paramètre :

```powershell
Get-AzStorageAccount -Name "{StorageAccountName}" -ResourceGroupName "{ResourceGroupName}"
StorageAccountName     : {StorageAccountName}
Kind                   : Storage
EnableHttpsTrafficOnly : False
...

```

Utilisez la ligne de commande ci-dessous pour activer le paramètre :

```powershell
Set-AzStorageAccount -Name "{StorageAccountName}" -ResourceGroupName "{ResourceGroupName}" -EnableHttpsTrafficOnly $True
StorageAccountName     : {StorageAccountName}
Kind                   : Storage
EnableHttpsTrafficOnly : True
...

```

## <a name="require-secure-transfer-with-azure-cli"></a>Exiger un transfert sécurisé avec Azure CLI

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

 Utilisez la commande ci-dessous pour vérifier le paramètre :

```azurecli-interactive
az storage account show -g {ResourceGroupName} -n {StorageAccountName}
{
  "name": "{StorageAccountName}",
  "enableHttpsTrafficOnly": false,
  "type": "Microsoft.Storage/storageAccounts"
  ...
}

```

Utilisez la commande ci-dessous pour activer le paramètre :

```azurecli-interactive
az storage account update -g {ResourceGroupName} -n {StorageAccountName} --https-only true
{
  "name": "{StorageAccountName}",
  "enableHttpsTrafficOnly": true,
  "type": "Microsoft.Storage/storageAccounts"
  ...
}

```

## <a name="next-steps"></a>Étapes suivantes

[Recommandations de sécurité pour Stockage Blob](../blobs/security-recommendations.md)
