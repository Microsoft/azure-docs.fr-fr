---
title: Créer un compte de stockage avec le chiffrement d’infrastructure activé à des fins de double chiffrement des données
titleSuffix: Azure Storage
description: Les clients qui doivent s’assurer que leurs données sont sécurisées peuvent également activer le chiffrement AES 256 bits au niveau de l’infrastructure Stockage Azure. Lorsque le chiffrement d’infrastructure est activé, les données d’un compte de stockage sont chiffrées deux fois avec deux algorithmes de chiffrement et deux clés différents.
services: storage
author: tamram
ms.service: storage
ms.date: 09/17/2020
ms.topic: conceptual
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.custom: devx-track-azurecli
ms.openlocfilehash: 612ba18ba71a22ad6c346b26008e688195c1d1e4
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92746569"
---
# <a name="create-a-storage-account-with-infrastructure-encryption-enabled-for-double-encryption-of-data"></a>Créer un compte de stockage avec le chiffrement d’infrastructure activé à des fins de double chiffrement des données

Stockage Azure chiffre automatiquement toutes les données d’un compte de stockage au niveau du service à l’aide du chiffrement AES 256 bits, un des chiffrements par blocs les plus puissants actuellement disponibles, et est conforme à la norme FIPS 140-2. Les clients qui doivent s’assurer que leurs données sont sécurisées peuvent également activer le chiffrement AES 256 bits au niveau de l’infrastructure Stockage Azure. Lorsque le chiffrement d’infrastructure est activé, les données d’un compte de stockage sont chiffrées deux fois &mdash; une fois au niveau du service et une fois au niveau de l’infrastructure &mdash; avec deux algorithmes de chiffrement et deux clés différents. Le double chiffrement des données Stockage Azure permet d’éviter un scénario impliquant une possible compromission d’un algorithme ou d’une clé de chiffrement. Dans un tel scénario, la couche de chiffrement supplémentaire continue de protéger vos données.

Le chiffrement au niveau du service prend en charge l’utilisation de clés gérées par Microsoft ou de clés gérées par le client avec Azure Key Vault ou un module de sécurité matériel (HSM) géré par Azure Key Vault (préversion). Le chiffrement au niveau de l’infrastructure s’appuie sur des clés gérées par Microsoft et utilise systématiquement une clé distincte. Pour plus d’informations sur la gestion des clés avec chiffrement Stockage Azure, consultez [À propos de la gestion des clés de chiffrement](storage-service-encryption.md#about-encryption-key-management).

Pour chiffrer doublement vos données, vous devez d’abord créer un compte de stockage configuré pour le chiffrement d’infrastructure. Cet article explique comment créer un compte de stockage qui active le chiffrement d’infrastructure.

## <a name="register-to-use-infrastructure-encryption"></a>S’inscrire pour utiliser le chiffrement d’infrastructure

Pour créer un compte de stockage pour lequel le chiffrement d’infrastructure est activé, vous devez commencer par vous inscrire de manière à utiliser cette fonctionnalité avec Azure à l’aide de PowerShell ou d’Azure CLI.

# <a name="azure-portal"></a>[Azure portal](#tab/portal)

N/A

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Pour vous inscrire à l’aide de PowerShell, appelez la commande [Register-AzProviderFeature](/powershell/module/az.resources/register-azproviderfeature).

```powershell
Register-AzProviderFeature -ProviderNamespace Microsoft.Storage `
    -FeatureName AllowRequireInfraStructureEncryption
```

Pour vérifier l’état de votre inscription à l’aide de PowerShell, appelez la commande [Get-AzProviderFeature](/powershell/module/az.resources/get-azproviderfeature).

```powershell
Get-AzProviderFeature -ProviderNamespace Microsoft.Storage `
    -FeatureName AllowRequireInfraStructureEncryption
```

Une fois votre inscription approuvée, vous devez réinscrire le fournisseur de ressources Stockage Azure. Pour réinscrire le fournisseur de ressources à l’aide de PowerShell, appelez la commande [Register-AzResourceProvider](/powershell/module/az.resources/register-azresourceprovider).

```powershell
Register-AzResourceProvider -ProviderNamespace 'Microsoft.Storage'
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour vous inscrire avec Azure CLI, appelez la commande [az feature register](/cli/azure/feature#az-feature-register).

```azurecli
az feature register --namespace Microsoft.Storage \
    --name AllowRequireInfraStructureEncryption
```

Pour vérifier l’état de votre inscription à l’aide d’Azure CLI, appelez la commande [az feature](/cli/azure/feature#az-feature-show).

```azurecli
az feature show --namespace Microsoft.Storage \
    --name AllowRequireInfraStructureEncryption
```

Une fois votre inscription approuvée, vous devez réinscrire le fournisseur de ressources Stockage Azure. Pour réinscrire le fournisseur de ressources à l’aide d’Azure CLI, appelez la commande [az provider register](/cli/azure/provider#az-provider-register).

```azurecli
az provider register --namespace 'Microsoft.Storage'
```

# <a name="template"></a>[Modèle](#tab/template)

N/A

---

## <a name="create-an-account-with-infrastructure-encryption-enabled"></a>Créer un compte avec le chiffrement d’infrastructure activé

Vous devez configurer un compte de stockage pour utiliser le chiffrement d’infrastructure au moment où vous créez le compte. Le compte de stockage doit être de type universel v2.

Le chiffrement d’infrastructure ne peut pas être activé ou désactivé une fois le compte créé.

# <a name="azure-portal"></a>[Azure portal](#tab/portal)

Pour utiliser PowerShell afin de créer un compte de stockage avec le chiffrement d’infrastructure activé, procédez comme suit :

1. Dans le portail Azure, accédez à la page **Comptes de stockage** .
1. Choisissez le bouton **ajouter un** pour ajouter un compte de stockage v2 à usage général.
1. Sous l’onglet **avancé** , localisez le chiffrement d’ **Infrastructure** , puis sélectionnez **Activé** .
1. Sélectionnez **Vérifier + créer** pour achever la création du compte de stockage.

    :::image type="content" source="media/infrastructure-encryption-enable/create-account-infrastructure-encryption-portal.png" alt-text="Capture d’écran montrant comment activer le chiffrement de l’infrastructure lors de la création d’un compte":::

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Pour utiliser PowerShell afin de créer un compte de stockage avec le chiffrement d’infrastructure activé, vérifiez que vous avez installé le [module PowerShell Az.Storage](https://www.powershellgallery.com/packages/Az.Storage) (version 2.2.0 ou ultérieure). Pour plus d’informations, consultez l’article [Installation et configuration d’Azure PowerShell](/powershell/azure/install-az-ps).

Ensuite, créez un compte de stockage à usage général v2 en appelant la commande [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount). Incluez l’option `-RequireInfrastructureEncryption` pour activer le chiffrement d'infrastructure.

L’exemple suivant montre comment créer un compte de stockage à usage général v2 configuré pour le stockage géoredondant avec accès en lecture (RA-GRS) et chiffrement d’infrastructure activé pour le double chiffrement des données. N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs :

```powershell
New-AzStorageAccount -ResourceGroupName <resource_group> `
    -AccountName <storage-account> `
    -Location <location> `
    -SkuName "Standard_RAGRS" `
    -Kind StorageV2 `
    -RequireInfrastructureEncryption
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour utiliser Azure CLI afin créer un compte de stockage avec le chiffrement d’infrastructure activé, vérifiez que Azure CLI version 2.8.0 ou ultérieure est installé. Pour plus d’informations, consultez la rubrique [Installation de l’interface de ligne de commande Azure (CLI)](/cli/azure/install-azure-cli).

Ensuite, créez un compte de stockage à usage générale v2 en appelant la commande [az storage account create](/cli/azure/storage/account#az-storage-account-create) et incluez `--require-infrastructure-encryption option` pour activer le chiffrement d’infrastructure.

L’exemple suivant montre comment créer un compte de stockage à usage général v2 configuré pour le stockage géoredondant avec accès en lecture (RA-GRS) et chiffrement d’infrastructure activé pour le double chiffrement des données. N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs :

```azurecli-interactive
az storage account create \
    --name <storage-account> \
    --resource-group <resource-group> \
    --location <location> \
    --sku Standard_RAGRS \
    --kind StorageV2 \
    --require-infrastructure-encryption
```

# <a name="template"></a>[Modèle](#tab/template)

L’exemple JSON suivant crée un compte de stockage à usage général v2 configuré pour le stockage géoredondant avec accès en lecture (RA-GRS) et chiffrement d’infrastructure activé pour le double chiffrement des données. N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs :

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
                "keySource": "Microsoft.Storage",
                "requireInfrastructureEncryption": true,
                "services": {
                    "blob": { "enabled": true },
                    "file": { "enabled": true }
              }
            }
        }
    }
],
```

---

## <a name="verify-that-infrastructure-encryption-is-enabled"></a>Vérifier que le chiffrement d’infrastructure est activé

# <a name="azure-portal"></a>[Azure portal](#tab/portal)

Pour vérifier que le chiffrement d’infrastructure est activé pour un compte de stockage avec le portail Azure, procédez comme suit :

1. Accédez à votre compte de stockage dans le portail Azure.
1. Sous **Paramètres** , choisissez **Chiffrement** .

    :::image type="content" source="media/infrastructure-encryption-enable/verify-infrastructure-encryption-portal.png" alt-text="Capture d’écran montrant comment activer le chiffrement de l’infrastructure lors de la création d’un compte":::

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Pour vérifier que le chiffrement d’infrastructure est activé pour un compte de stockage avec PowerShell, appelez la cmdlet [Get-AzStorageAccount](/powershell/module/az.storage/get-azstorageaccount). Cette commande retourne un ensemble de propriétés du compte de stockage et leurs valeurs. Récupérez le champ `RequireInfrastructureEncryption` dans la propriété `Encryption` et vérifiez qu’il est défini sur `True`.

L'exemple de code suivant récupère la valeur de la propriété `RequireInfrastructureEncryption`. N’oubliez pas de remplacer les valeurs des espaces réservés entre crochets par vos propres valeurs :

```powershell
$account = Get-AzStorageAccount -ResourceGroupName <resource-group> `
    -StorageAccountName <storage-account>
$account.Encryption.RequireInfrastructureEncryption
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour vérifier que le chiffrement d’infrastructure est activé pour un compte de stockage avec Azure CLI, appelez la commande [az storage account show](/cli/azure/storage/account#az-storage-account-show). Cette commande retourne un ensemble de propriétés du compte de stockage et leurs valeurs. Recherchez le champ `requireInfrastructureEncryption` dans la propriété `encryption` et vérifiez qu’il est défini sur `true`.

L'exemple de code suivant récupère la valeur de la propriété `requireInfrastructureEncryption`. N’oubliez pas de remplacer les valeurs des espaces réservés entre crochets par vos propres valeurs :

```azurecli-interactive
az storage account show /
    --name <storage-account> /
    --resource-group <resource-group>
```

# <a name="template"></a>[Modèle](#tab/template)

N/A

---

## <a name="next-steps"></a>Étapes suivantes

- [Chiffrement du stockage Azure pour les données au repos](storage-service-encryption.md)
- [Clés gérées par le client pour le chiffrement du service Stockage Azure](customer-managed-keys-overview.md)
