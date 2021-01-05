---
title: Créer un compte qui prend en charge les clés gérées par le client pour les tables et les files d’attente
titleSuffix: Azure Storage
description: Découvrez comment créer un compte de stockage prenant en charge la configuration des clés gérées par le client pour les tables et les files d’attente. Utilisez Azure CLI ou un modèle Azure Resource Manager pour créer un compte de stockage qui s’appuie sur la clé de chiffrement du compte pour le chiffrement de Stockage Azure. Vous pouvez ensuite configurer des clés gérées par le client pour le compte.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 02/05/2020
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.custom: devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: 8150375eff98374e21d200d98c04158b07f1c243
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92789690"
---
# <a name="create-an-account-that-supports-customer-managed-keys-for-tables-and-queues"></a>Créer un compte qui prend en charge les clés gérées par le client pour les tables et les files d’attente

Le stockage Azure chiffre toutes les données dans un compte de stockage au repos. Par défaut, Stockage File d’attente et Stockage Table utilisent une clé qui est limitée au service et gérée par Microsoft. Vous pouvez également choisir d’utiliser des clés gérées par le client pour chiffrer les données des files d’attente ou des tables. Pour utiliser des clés gérées par le client avec des files d’attente et des tables, vous devez d’abord créer un compte de stockage qui utilise une clé de chiffrement limitée au compte et non pas au service. Après avoir créé un compte qui utilise la clé de chiffrement de compte pour les données des files d’attente et des tables, vous pouvez configurer des clés gérées par le client pour ce compte de stockage.

Cet article explique comment créer un compte de stockage qui s’appuie sur une clé limitée au compte. Une fois le compte créé, Microsoft utilise la clé de compte pour chiffrer les données dans le compte et il gère la clé. Vous pouvez par la suite configurer des clés gérées par le client pour le compte pour tirer parti de ces avantages, notamment la capacité à fournir vos propres clés, de mettre à jour la version de la clé, d’effectuer la rotation des clés et de révoquer les contrôles d’accès.

## <a name="about-the-feature"></a>À propos de la fonctionnalité

Pour créer un compte de stockage qui s’appuie sur la clé de chiffrement du compte pour Stockage File d’attente et Stockage Table, vous devez d’abord vous inscrire pour utiliser cette fonctionnalité avec Azure. En raison d’une capacité limitée, notez que plusieurs mois peuvent être nécessaires avant que les demandes d’accès soient approuvées.

Vous pouvez créer un compte de stockage qui s’appuie sur la clé de chiffrement du compte pour Stockage File d’attente et Stockage Table dans les régions suivantes :

- USA Est
- États-Unis - partie centrale méridionale
- USA Ouest 2  

### <a name="register-to-use-the-account-encryption-key"></a>S’inscrire pour utiliser la clé de chiffrement de compte

Afin de vous inscrire pour utiliser la clé de chiffrement de compte avec le Stockage Table ou File d’attente, utilisez PowerShell ou Azure CLI.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Pour vous inscrire à l’aide de PowerShell, appelez la commande [Register-AzProviderFeature](/powershell/module/az.resources/register-azproviderfeature).

```powershell
Register-AzProviderFeature -ProviderNamespace Microsoft.Storage `
    -FeatureName AllowAccountEncryptionKeyForQueues
Register-AzProviderFeature -ProviderNamespace Microsoft.Storage `
    -FeatureName AllowAccountEncryptionKeyForTables
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour vous inscrire avec Azure CLI, appelez la commande [az feature register](/cli/azure/feature#az-feature-register).

```azurecli
az feature register --namespace Microsoft.Storage \
    --name AllowAccountEncryptionKeyForQueues
az feature register --namespace Microsoft.Storage \
    --name AllowAccountEncryptionKeyForTables
```

# <a name="template"></a>[Modèle](#tab/template)

N/A

---

### <a name="check-the-status-of-your-registration"></a>Vérifier l’état de votre inscription

Pour vérifier l’état de votre inscription au Stockage Table ou File d’attente, utilisez PowerShell ou Azure CLI.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Pour vérifier l’état de votre inscription à l’aide de PowerShell, appelez la commande [Get-AzProviderFeature](/powershell/module/az.resources/get-azproviderfeature).

```powershell
Get-AzProviderFeature -ProviderNamespace Microsoft.Storage `
    -FeatureName AllowAccountEncryptionKeyForQueues
Get-AzProviderFeature -ProviderNamespace Microsoft.Storage `
    -FeatureName AllowAccountEncryptionKeyForTables
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour vérifier l’état de votre inscription à l’aide d’Azure CLI, appelez la commande [az feature](/cli/azure/feature#az-feature-show).

```azurecli
az feature show --namespace Microsoft.Storage \
    --name AllowAccountEncryptionKeyForQueues
az feature show --namespace Microsoft.Storage \
    --name AllowAccountEncryptionKeyForTables
```

# <a name="template"></a>[Modèle](#tab/template)

N/A

---

### <a name="re-register-the-azure-storage-resource-provider"></a>Réinscrire le fournisseur de ressources Stockage Azure

Une fois votre inscription approuvée, vous devez réinscrire le fournisseur de ressources Stockage Azure. Utilisez PowerShell ou Azure CLI pour réinscrire le fournisseur de ressources.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Pour réinscrire le fournisseur de ressources à l’aide de PowerShell, appelez la commande [Register-AzResourceProvider](/powershell/module/az.resources/register-azresourceprovider).

```powershell
Register-AzResourceProvider -ProviderNamespace 'Microsoft.Storage'
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour réinscrire le fournisseur de ressources à l’aide d’Azure CLI, appelez la commande [az provider register](/cli/azure/provider#az-provider-register).

```azurecli
az provider register --namespace 'Microsoft.Storage'
```

# <a name="template"></a>[Modèle](#tab/template)

N/A

---

## <a name="create-an-account-that-uses-the-account-encryption-key"></a>Créer un compte qui utilise la clé de chiffrement de compte

Vous devez configurer un nouveau compte de stockage pour utiliser la clé de chiffrement de compte pour les files d’attente et les tables au moment où vous créez le compte de stockage. L’étendue de la clé de chiffrement ne peut pas être changée après la création du compte.

Le compte de stockage doit être de type universel v2. Vous pouvez créer le compte de stockage et le configurer pour qu’il s’appuie sur la clé de chiffrement de compte en utilisant Azure CLI ou un modèle Azure Resource Manager.

> [!NOTE]
> Seul Stockage File d’attente et Stockage Table peuvent être configurés en option pour chiffrer les données avec la clé de chiffrement de compte lors de la création du compte de stockage. Stockage Blob et Azure Files utilisent toujours la clé de chiffrement de compte pour chiffrer les données.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Pour utiliser PowerShell afin de créer un compte de stockage qui s’appuie sur la clé de chiffrement de compte, vérifiez que vous avez installé le module Azure PowerShell (version 3.4.0 ou ultérieure). Pour plus d’informations, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps).

Ensuite, créez un compte de stockage universel v2 en appelant la commande [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount) avec les paramètres appropriés :

- Incluez l’option `-EncryptionKeyTypeForQueue` et définissez sa valeur sur `Account` pour utiliser la clé de chiffrement de compte afin de chiffrer les données dans Stockage File d’attente.
- Incluez l’option `-EncryptionKeyTypeForTable` et définissez sa valeur sur `Account` pour utiliser la clé de chiffrement de compte afin de chiffrer les données dans Stockage Table.

L’exemple suivant montre comment créer un compte de stockage universel v2 configuré pour le stockage géoredondant avec accès en lecture (RA-GRS) et utilisant la clé de chiffrement de compte pour chiffrer les données à la fois pour Stockage File d’attente et pour Stockage Table. N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs :

```powershell
New-AzStorageAccount -ResourceGroupName <resource_group> `
    -AccountName <storage-account> `
    -Location <location> `
    -SkuName "Standard_RAGRS" `
    -Kind StorageV2 `
    -EncryptionKeyTypeForTable Account `
    -EncryptionKeyTypeForQueue Account
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour utiliser Azure CLI afin créer un compte de stockage qui s’appuie sur la clé de chiffrement de compte, vérifié que Azure CLI version 2.0.80 ou ultérieure est installé. Pour plus d’informations, consultez la rubrique [Installation de l’interface de ligne de commande Azure (CLI)](/cli/azure/install-azure-cli).

Ensuite, créez un compte de stockage universel v2 en appelant la commande [az storage account create](/cli/azure/storage/account#az-storage-account-create) avec les paramètres appropriés :

- Incluez l’option `--encryption-key-type-for-queue` et définissez sa valeur sur `Account` pour utiliser la clé de chiffrement de compte afin de chiffrer les données dans Stockage File d’attente.
- Incluez l’option `--encryption-key-type-for-table` et définissez sa valeur sur `Account` pour utiliser la clé de chiffrement de compte afin de chiffrer les données dans Stockage Table.

L’exemple suivant montre comment créer un compte de stockage universel v2 configuré pour le stockage géoredondant avec accès en lecture (RA-GRS) et utilisant la clé de chiffrement de compte pour chiffrer les données à la fois pour Stockage File d’attente et pour Stockage Table. N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs :

```azurecli
az storage account create \
    --name <storage-account> \
    --resource-group <resource-group> \
    --location <location> \
    --sku Standard_RAGRS \
    --kind StorageV2 \
    --encryption-key-type-for-table Account \
    --encryption-key-type-for-queue Account
```

# <a name="template"></a>[Modèle](#tab/template)

L’exemple JSON suivant crée un compte de stockage universel v2 configuré pour le stockage géoredondant avec accès en lecture (RA-GRS) et utilisant la clé de chiffrement de compte pour chiffrer les données à la fois pour Stockage File d’attente et pour Stockage Table. N’oubliez pas de remplacer les valeurs des espaces réservés entre crochets par vos propres valeurs :

```json
"resources": [
    {
        "type": "Microsoft.Storage/storageAccounts",
        "apiVersion": "2019-06-01",
        "name": "[parameters('<storage-account>')]",
        "location": "[parameters('<location>')]",
        "dependsOn": [],
        "tags": {},
        "sku": {
            "name": "[parameters('Standard_RAGRS')]"
        },
        "kind": "[parameters('StorageV2')]",
        "properties": {
            "accessTier": "[parameters('<accessTier>')]",
            "supportsHttpsTrafficOnly": "[parameters('supportsHttpsTrafficOnly')]",
            "largeFileSharesState": "[parameters('<largeFileSharesState>')]",
            "encryption": {
                "services": {
                    "queue": {
                        "keyType": "Account"
                    },
                    "table": {
                        "keyType": "Account"
                    }
                },
                "keySource": "Microsoft.Storage"
            }
        }
    }
],
```

---

Une fois que vous avez créé un compte qui s’appuie sur la clé de chiffrement de compte, vous pouvez configurer les clés gérées par le client qui sont stockées dans Azure Key Vault ou dans un module de sécurité matériel (HSM) géré par Azure Key Vault (préversion). Pour découvrir comment stocker des clés gérées par le client dans un coffre de clés, consultez [Configurer le chiffrement avec des clés gérées par le client stockées dans Azure Key Vault](customer-managed-keys-configure-key-vault.md). Pour découvrir comment stocker des clés gérées par le client dans un HSM managé, consultez [Configurer le chiffrement avec des clés gérées par le client stockées dans un HSM géré par Azure Key Vault (préversion)](customer-managed-keys-configure-key-vault-hsm.md).

## <a name="verify-the-account-encryption-key"></a>Vérifier la clé de chiffrement de compte

Pour vérifier qu’un service dans un compte de stockage utilise la clé de chiffrement de compte, appelez la commande Azure CLI [az storage account](/cli/azure/storage/account#az-storage-account-show). Cette commande retourne un ensemble de propriétés du compte de stockage et leurs valeurs. Recherchez le champ `keyType` pour chaque service dans la propriété de chiffrement et vérifiez qu’il est défini sur `Account`.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Pour vérifier qu’un service dans un compte de stockage utilise la clé de chiffrement de compte, appelez la commande [Get-AzStorageAccount](/powershell/module/az.storage/get-azstorageaccount). Cette commande retourne un ensemble de propriétés du compte de stockage et leurs valeurs. Recherchez le champ `KeyType` pour chaque service dans la propriété `Encryption` et vérifiez qu’il est défini sur `Account`.

```powershell
$account = Get-AzStorageAccount -ResourceGroupName <resource-group> `
    -StorageAccountName <storage-account>
$account.Encryption.Services.Queue
$account.Encryption.Services.Table
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour vérifier qu’un service dans un compte de stockage utilise la clé de chiffrement de compte, appelez la commande [az storage account show](/cli/azure/storage/account#az-storage-account-show). Cette commande retourne un ensemble de propriétés du compte de stockage et leurs valeurs. Recherchez le champ `keyType` pour chaque service dans la propriété de chiffrement et vérifiez qu’il est défini sur `Account`.

```azurecli
az storage account show /
    --name <storage-account> /
    --resource-group <resource-group>
```

# <a name="template"></a>[Modèle](#tab/template)

N/A

---

## <a name="next-steps"></a>Étapes suivantes

- [Chiffrement du stockage Azure pour les données au repos](storage-service-encryption.md)
- [Clés gérées par le client pour le chiffrement du stockage Azure](customer-managed-keys-overview.md)
- [Qu’est-ce qu’Azure Key Vault ?](../../key-vault/general/overview.md)