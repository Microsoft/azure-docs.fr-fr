---
title: Configurer le chiffrement avec des clés gérées par le client stockées dans Azure Key Vault
titleSuffix: Azure Storage
description: Découvrez comment configurer le chiffrement du service Stockage Azure avec des clés gérées par le client stockées dans Azure Key Vault à l’aide de portail Azure, de PowerShell ou d’Azure CLI.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 09/16/2020
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.custom: devx-track-azurepowershell, devx-track-azurecli
ms.openlocfilehash: 02661c9c2a581ab21a2ae9dc31e5da95426c0edd
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94843381"
---
# <a name="configure-encryption-with-customer-managed-keys-stored-in-azure-key-vault"></a>Configurer le chiffrement avec des clés gérées par le client stockées dans Azure Key Vault

Le stockage Azure chiffre toutes les données dans un compte de stockage au repos. Par défaut, les données sont chiffrées avec des clés managées par Microsoft Pour un contrôle supplémentaire des clés de chiffrement, vous pouvez gérer vos propres clés. Les clés gérées par le client doivent être stockées dans Azure Key Vault ou dans un module de sécurité matériel (HSM) géré par Azure Key Vault (préversion).

Cet article explique comment configurer le chiffrement avec des clés gérées par le client stockées dans un coffre de clés à l’aide du portail Azure, de PowerShell ou d’Azure CLI. Pour savoir comment configurer le chiffrement avec des clés gérées par le client stockées dans un HSM géré, consultez [Configurer le chiffrement avec des clés gérées par le client stockées dans un HSM géré par Azure Key Vault (préversion)](customer-managed-keys-configure-key-vault-hsm.md).

> [!NOTE]
> Azure Key Vault et le HSM géré par Azure Key Vault prennent en charge les mêmes API et interfaces de gestion pour la configuration.

## <a name="configure-a-key-vault"></a>Configurer un coffre de clés

Vous pouvez utiliser un coffre de clés nouveau ou existant pour stocker les clés gérées par le client. Le compte de stockage et le coffre de clés doivent se trouver dans la même région, mais ils peuvent appartenir à des abonnements différents.

L’utilisation de clés gérées par le client avec le chiffrement Azure Storage requiert que la suppression réversible et la protection contre le vidage soient activées pour le coffre de clés. Lorsque vous créez un coffre de clés, la suppression réversible est activée par défaut et vous ne pouvez pas la désactiver. Vous pouvez activer la protection contre le vidage lorsque vous créez le coffre de clés ou après la création de celui-ci.

# <a name="azure-portal"></a>[Azure portal](#tab/portal)

Pour en savoir plus sur la création d’un coffre de clés via le portail Azure, consultez [Démarrage rapide : Créer un coffre de clés avec le portail Azure](../../key-vault/general/quick-create-portal.md). Lorsque vous créez le coffre de clés, sélectionnez **Activer la protection contre le vidage**, comme illustré dans l’image suivante.

:::image type="content" source="media/customer-managed-keys-configure-key-vault/configure-key-vault-portal.png" alt-text="Capture d’écran montrant comment activer la protection contre le vidage lors de la création d’un coffre de clés":::

Pour activer la protection contre le vidage sur un coffre de clés existant, procédez comme suit :

1. Accédez à votre coffre de clés dans le portail Azure.
1. Sous **Paramètres**, choisissez **Propriétés**.
1. Dans la section **Protection contre le vidage**, choisissez **Activer la protection contre le vidage**.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Pour créer un coffre de clés à l’aide de PowerShell, installez la version 2.0.0 ou une version ultérieure du module PowerShell [Az.KeyVault](https://www.powershellgallery.com/packages/Az.KeyVault/2.0.0). Appelez ensuite [New-AzKeyVault](/powershell/module/az.keyvault/new-azkeyvault) pour créer un coffre de clés. Dans les versions 2.0.0 et ultérieures du module Az.KeyVault, la suppression réversible est activée par défaut lorsque vous créez un coffre de clés.

L’exemple suivant crée un coffre de clés pour lequel la suppression réversible et la protection contre le vidage sont activées. N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs.

```powershell
$keyVault = New-AzKeyVault -Name <key-vault> `
    -ResourceGroupName <resource_group> `
    -Location <location> `
    -EnablePurgeProtection
```

Pour savoir comment activer la protection contre le vidage sur un coffre de clés existant avec PowerShell, consultez [Guide pratique pour utiliser la suppression réversible avec Power​Shell](../../key-vault/general/key-vault-recovery.md).

Ensuite, affectez une identité managée affectée par le système à votre compte de stockage. Cette identité managée vous sera utile pour autoriser le compte de stockage à accéder au coffre de clés. Pour en savoir plus sur les identités managées affectées par le système, consultez la section [Que sont les identités managées pour les ressources Azure ?](../../active-directory/managed-identities-azure-resources/overview.md).

Pour attribuer une identité managée à l’aide de PowerShell, appelez la cmdlet [Set-AzStorageAccount](/powershell/module/az.storage/set-azstorageaccount) :

```powershell
$storageAccount = Set-AzStorageAccount -ResourceGroupName <resource_group> `
    -Name <storage-account> `
    -AssignIdentity
```

Enfin, configurez la stratégie d’accès au coffre de clés de sorte que le compte de stockage dispose des autorisations nécessaires pour y accéder. À cette étape, vous utiliserez l’identité managée que vous avez précédemment affectée au compte de stockage.

Pour définir la stratégie d’accès du coffre de clés, appelez la commande [Set-AzKeyVaultAccessPolicy](/powershell/module/az.keyvault/set-azkeyvaultaccesspolicy) :

```powershell
Set-AzKeyVaultAccessPolicy `
    -VaultName $keyVault.VaultName `
    -ObjectId $storageAccount.Identity.PrincipalId `
    -PermissionsToKeys wrapkey,unwrapkey,get
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour créer un coffre de clés avec Azure CLI, appelez [az keyvault create](/cli/azure/keyvault#az-keyvault-create). N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs :

```azurecli-interactive
az keyvault create \
    --name <key-vault> \
    --resource-group <resource_group> \
    --location <region> \
    --enable-purge-protection
```

Pour savoir comment activer la protection contre le vidage sur un coffre de clés existant avec Azure CLI, consultez [Guide pratique pour utiliser la suppression réversible avec Azure CLI](../../key-vault/general/key-vault-recovery.md).

Ensuite, attribuez une identité managée affectée par le système au compte de stockage. Cette identité managée vous sera utile pour autoriser le compte de stockage à accéder au coffre de clés. Pour en savoir plus sur les identités managées affectées par le système, consultez la section [Que sont les identités managées pour les ressources Azure ?](../../active-directory/managed-identities-azure-resources/overview.md).

Pour attribuer une identité managée à l’aide d’Azure CLI, appelez la commande [az storage account update](/cli/azure/storage/account#az-storage-account-update) :

```azurecli-interactive
az storage account update \
    --name <storage-account> \
    --resource-group <resource_group> \
    --assign-identity
```

Enfin, configurez la stratégie d’accès au coffre de clés de sorte que le compte de stockage dispose des autorisations nécessaires pour y accéder. À cette étape, vous utiliserez l’identité managée que vous avez précédemment affectée au compte de stockage.

Pour définir la stratégie d’accès au coffre de clés, appelez [az keyvault set-policy](/cli/azure/keyvault#az-keyvault-set-policy) :

```azurecli-interactive
storage_account_principal=$(az storage account show \
    --name <storage-account> \
    --resource-group <resource-group> \
    --query identity.principalId \
    --output tsv)
az keyvault set-policy \
    --name <key-vault> \
    --resource-group <resource_group>
    --object-id $storage_account_principal \
    --key-permissions get unwrapKey wrapKey
```

---

## <a name="add-a-key"></a>Ajouter une clé

Ensuite, ajoutez une clé au coffre de clés.

Le chiffrement du service Stockage Azure prend en charge les clés RSA et RSA-HSM dans les tailles 2048, 3072 et 4096. Pour plus d’informations sur les clés, consultez [À propos des clés](../../key-vault/keys/about-keys.md).

# <a name="azure-portal"></a>[Azure portal](#tab/portal)

Pour savoir comment ajouter une clé avec le portail Azure, consultez [Démarrage rapide : Définir et récupérer une clé dans Azure Key Vault avec le portail Azure](../../key-vault/keys/quick-create-portal.md).

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Pour ajouter une clé avec PowerShell, appelez la cmdlet [Add-AzKeyVaultKey](/powershell/module/az.keyvault/add-azkeyvaultkey). N’oubliez pas de remplacer les valeurs de l’espace réservé entre crochets par vos propres valeurs et d’utiliser les variables définies dans les exemples précédents.

```powershell
$key = Add-AzKeyVaultKey -VaultName $keyVault.VaultName `
    -Name <key> `
    -Destination 'Software'
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour ajouter une clé avec Azure CLI, appelez la commande [az keyvault key create](/cli/azure/keyvault/key#az-keyvault-key-create). N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs.

```azurecli-interactive
az keyvault key create \
    --name <key> \
    --vault-name <key-vault>
```

---

## <a name="configure-encryption-with-customer-managed-keys"></a>Configurer le chiffrement avec les clés gérées par le client

Ensuite, configurez votre compte de stockage Azure pour utiliser des clés gérées par le client à l’aide d’Azure Key Vault, puis spécifiez la clé à lui associer.

Quand vous configurez le chiffrement avec des clés gérées par le client, vous pouvez choisir de mettre à jour automatiquement la version de clé utilisée pour le chiffrement du service Stockage Azure chaque fois qu’une nouvelle version est disponible dans le coffre de clés associé. Vous pouvez également spécifier explicitement une version de clé à utiliser pour le chiffrement jusqu’à ce que la version de clé soit mise à jour manuellement.

> [!NOTE]
> Pour effectuer la rotation d’une clé, créez une version de la clé dans Azure Key Vault. Le stockage Azure ne gère pas la rotation de la clé dans Azure Key Vault. Vous devez donc effectuer la rotation de votre clé manuellement ou créer une fonction pour effectuer cette opération selon une planification.

### <a name="configure-encryption-for-automatic-updating-of-key-versions"></a>Configurer le chiffrement pour la mise à jour automatique des versions de clé

Le stockage Azure peut automatiquement mettre à jour la clé gérée par le client qui est utilisée pour le chiffrement afin d’utiliser la dernière version de la clé. Quand la clé gérée par le client subit une rotation dans Azure Key Vault, le stockage Azure commence automatiquement à utiliser la dernière version de la clé pour le chiffrement.

# <a name="azure-portal"></a>[Azure portal](#tab/portal)

Pour configurer les clés gérées par le client avec une mise à jour automatique de la version de la clé dans le Portail Azure, procédez comme suit :

1. Accédez à votre compte de stockage.
1. Sur le panneau **Paramètres** du compte de stockage, cliquez sur **Chiffrement**. Sélectionnez l’option **Clés gérées par le client**, comme illustré dans l’image suivante.

    ![Capture d’écran du portail affichant l’option de chiffrement](./media/customer-managed-keys-configure-key-vault/portal-configure-encryption-keys.png)

1. Choisissez l’option **Sélectionner dans le coffre de clés**.
1. Sélectionnez **Sélectionner un coffre de clés et une clé**.
1. Sélectionnez le coffre de clés contenant la clé que vous souhaitez utiliser.
1. Sélectionnez la clé dans le coffre de clés.

   ![Capture d’écran montrant comment sélectionner un coffre de clés et une clé](./media/customer-managed-keys-configure-key-vault/portal-select-key-from-key-vault.png)

1. Enregistrez vos modifications.

Une fois que vous avez spécifié la clé, le portail Azure indique que la mise à jour automatique de la version de la clé est activée, et affiche la version de clé en cours d’utilisation pour le chiffrement.

:::image type="content" source="media/customer-managed-keys-configure-key-vault/portal-auto-rotation-enabled.png" alt-text="Capture d’écran dans laquelle la mise à jour automatique de la version de la clé est activée":::

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Pour configurer des clés gérées par le client avec la mise à jour automatique de la version de la clé avec PowerShell, installez le module [Az.Storage](https://www.powershellgallery.com/packages/Az.Storage), version 2.0.0 ou ultérieure.

Pour mettre à jour automatiquement la version d’une clé gérée par le client, omettez la version de la clé quand vous configurez le chiffrement avec les clés gérées par le client pour le compte de stockage. Appelez [Set-AzStorageAccount](/powershell/module/az.storage/set-azstorageaccount) pour mettre à jour les paramètres de chiffrement du compte de stockage, comme indiqué dans l’exemple suivant, et incluez l’option **-KeyvaultEncryption** pour activer les clés gérées par le client pour le compte de stockage.

N’oubliez pas de remplacer les valeurs de l’espace réservé entre crochets par vos propres valeurs et d’utiliser les variables définies dans les exemples précédents.

```powershell
Set-AzStorageAccount -ResourceGroupName $storageAccount.ResourceGroupName `
    -AccountName $storageAccount.StorageAccountName `
    -KeyvaultEncryption `
    -KeyName $key.Name `
    -KeyVaultUri $keyVault.VaultUri
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour configurer des clés gérées par le client avec la mise à jour automatique de la version de clé avec Azure CLI, installez [Azure CLI version 2.4.0](/cli/azure/release-notes-azure-cli#april-21-2020) ou version ultérieure. Pour plus d’informations, consultez la rubrique [Installation de l’interface de ligne de commande Azure (CLI)](/cli/azure/install-azure-cli).

Pour mettre à jour automatiquement la version d’une clé gérée par le client, omettez la version de la clé quand vous configurez le chiffrement avec les clés gérées par le client pour le compte de stockage. Pour mettre à jour les paramètres de chiffrement du compte de stockage, appelez [az storage account update](/cli/azure/storage/account#az-storage-account-update), comme illustré dans l’exemple suivant. Incluez le paramètre `--encryption-key-source` et définissez-le sur `Microsoft.Keyvault` pour activer les clés gérées par le client pour le compte.

N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs.

```azurecli-interactive
key_vault_uri=$(az keyvault show \
    --name <key-vault> \
    --resource-group <resource_group> \
    --query properties.vaultUri \
    --output tsv)
az storage account update
    --name <storage-account> \
    --resource-group <resource_group> \
    --encryption-key-name <key> \
    --encryption-key-source Microsoft.Keyvault \
    --encryption-key-vault $key_vault_uri
```

---

### <a name="configure-encryption-for-manual-updating-of-key-versions"></a>Configurer le chiffrement pour la mise à jour manuelle des versions de clé

Si vous préférez mettre à jour manuellement la version de la clé, spécifiez explicitement la version au moment où vous configurez le chiffrement avec les clés gérées par le client. Dans ce cas, le service Stockage Azure ne met pas automatiquement à jour la version de la clé quand une nouvelle version est créée dans le coffre de clés. Pour utiliser une nouvelle version de la clé, vous devez mettre à jour manuellement la version utilisée pour le chiffrement de Stockage Azure.

# <a name="azure-portal"></a>[Azure portal](#tab/portal)

Pour configurer les clés gérées par le client avec la mise à jour manuelle de la version de la clé dans le portail Azure, spécifiez l’URI de la clé, en incluant la version. Pour spécifier une clé en tant qu’URI, procédez comme suit :

1. Pour localiser l’URI de la clé dans le portail Azure, naviguez jusqu'à votre coffre de clés, puis sélectionnez le paramètre **Clés**. Sélectionnez la clé souhaitée, puis cliquez dessus pour afficher ses versions. Sélectionnez une version de clé pour afficher les paramètres de cette version.
1. Copiez la valeur du champ **Identificateur de clé**, qui fournit l’URI.

    ![Capture d’écran montrant l’URI de la clé du coffre de clés](media/customer-managed-keys-configure-key-vault/portal-copy-key-identifier.png)

1. Dans les paramètres de **clé de chiffrement** de votre compte de stockage, choisissez l’option **Entrer l’URI de la clé**.
1. Collez l’URI que vous avez copié dans le champ **URI de clé**. Omettez la version de la clé dans l’URI pour activer la mise à jour automatique de la version de la clé.

   ![Capture d’écran montrant comment entrer l’URI d’une clé](./media/customer-managed-keys-configure-key-vault/portal-specify-key-uri.png)

1. Spécifiez l’abonnement qui contient le coffre de clés.
1. Enregistrez vos modifications.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Pour configurer des clés gérées par le client avec la mise à jour manuelle de la version de la clé, fournissez explicitement la version de la clé lorsque vous configurez le chiffrement pour le compte de stockage. Appelez [Set-AzStorageAccount](/powershell/module/az.storage/set-azstorageaccount) pour mettre à jour les paramètres de chiffrement du compte de stockage, comme indiqué dans l’exemple suivant, et incluez l’option **-KeyvaultEncryption** pour activer les clés gérées par le client pour le compte de stockage.

N’oubliez pas de remplacer les valeurs de l’espace réservé entre crochets par vos propres valeurs et d’utiliser les variables définies dans les exemples précédents.

```powershell
Set-AzStorageAccount -ResourceGroupName $storageAccount.ResourceGroupName `
    -AccountName $storageAccount.StorageAccountName `
    -KeyvaultEncryption `
    -KeyName $key.Name `
    -KeyVersion $key.Version `
    -KeyVaultUri $keyVault.VaultUri
```

Quand vous mettez à jour manuellement la version de clé, vous devez mettre à jour les paramètres de chiffrement du compte de stockage afin d’utiliser la nouvelle version. Tout d’abord, appelez la commande [Get-AzKeyVaultKey](/powershell/module/az.keyvault/get-azkeyvaultkey) pour obtenir la dernière version de la clé. Appelez ensuite la commande [Set-AzStorageAccount](/powershell/module/az.storage/set-azstorageaccount) pour mettre à jour les paramètres de chiffrement du compte de stockage afin d’utiliser la nouvelle version de la clé, comme indiqué dans l’exemple précédent.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour configurer des clés gérées par le client avec la mise à jour manuelle de la version de la clé, fournissez explicitement la version de la clé lorsque vous configurez le chiffrement pour le compte de stockage. Pour mettre à jour les paramètres de chiffrement du compte de stockage, appelez [az storage account update](/cli/azure/storage/account#az-storage-account-update), comme illustré dans l’exemple suivant. Incluez le paramètre `--encryption-key-source` et définissez-le sur `Microsoft.Keyvault` pour activer les clés gérées par le client pour le compte.

N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs.

```azurecli-interactive
key_vault_uri=$(az keyvault show \
    --name <key-vault> \
    --resource-group <resource_group> \
    --query properties.vaultUri \
    --output tsv)
key_version=$(az keyvault key list-versions \
    --name <key> \
    --vault-name <key-vault> \
    --query [-1].kid \
    --output tsv | cut -d '/' -f 6)
az storage account update
    --name <storage-account> \
    --resource-group <resource_group> \
    --encryption-key-name <key> \
    --encryption-key-version $key_version \
    --encryption-key-source Microsoft.Keyvault \
    --encryption-key-vault $key_vault_uri
```

Quand vous mettez à jour manuellement la version de clé, vous devez mettre à jour les paramètres de chiffrement du compte de stockage afin d’utiliser la nouvelle version. Tout d’abord, lancez une requête pour l’URI du coffre de clés en appelant [az keyvault show](/cli/azure/keyvault#az-keyvault-show)et pour la version de la clé en appelant [az keyvault key list-versions](/cli/azure/keyvault/key#az-keyvault-key-list-versions). Appelez ensuite [az storage account update](/cli/azure/storage/account#az-storage-account-update) pour mettre à jour les paramètres de chiffrement du compte de stockage afin d’utiliser la nouvelle version de la clé, comme indiqué dans l’exemple précédent.

---

## <a name="change-the-key"></a>Modifier la clé

Vous pouvez modifier la clé que vous utilisez pour le chiffrement du service Stockage Azure à tout moment.

# <a name="azure-portal"></a>[Azure portal](#tab/portal)

Pour modifier la clé avec le portail Azure, procédez comme suit :

1. Accédez à votre compte de stockage et affichez les paramètres de **chiffrement**.
1. Sélectionnez le coffre de clés, puis choisissez une nouvelle clé.
1. Enregistrez vos modifications.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Pour modifier la clé avec PowerShell, appelez la cmdlet [Set-AzStorageAccount](/powershell/module/az.storage/set-azstorageaccount) comme indiqué dans [Configurer des clés de chiffrement gérées par le client](#configure-encryption-with-customer-managed-keys), puis entrez le nom et la version de la nouvelle clé. Si la nouvelle clé se trouve dans un coffre de clés différent, vous devez également mettre à jour l’URI du coffre de clés.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour modifier la clé avec Azure CLI, appelez la commande [az storage account update](/cli/azure/storage/account#az-storage-account-update) comme indiqué dans [Configurer des clés de chiffrement gérées par le client](#configure-encryption-with-customer-managed-keys) et indiquez le nom et la version de la nouvelle clé. Si la nouvelle clé se trouve dans un coffre de clés différent, vous devez également mettre à jour l’URI du coffre de clés.

---

## <a name="revoke-customer-managed-keys"></a>Révoquer des clés gérées par le client

La révocation d’une clé gérée par le client supprime l’association entre le compte de stockage et le coffre de clés.

# <a name="azure-portal"></a>[Azure portal](#tab/portal)

Pour révoquer les clés gérées par le client avec le portail Azure, désactivez la clé en procédant de la manière décrite dans [Désactiver les clés gérées par le client](#disable-customer-managed-keys).

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Vous pouvez révoquer des clés gérées par le client en supprimant la stratégie d’accès du coffre de clés. Pour révoquer une clé gérée par le client avec PowerShell, appelez la cmdlet [Remove-AzKeyVaultAccessPolicy](/powershell/module/az.keyvault/remove-azkeyvaultaccesspolicy), comme indiqué dans l’exemple suivant. N’oubliez pas de remplacer les valeurs de l’espace réservé entre crochets par vos propres valeurs et d’utiliser les variables définies dans les exemples précédents.

```powershell
Remove-AzKeyVaultAccessPolicy -VaultName $keyVault.VaultName `
    -ObjectId $storageAccount.Identity.PrincipalId `
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Vous pouvez révoquer des clés gérées par le client en supprimant la stratégie d’accès du coffre de clés. Pour révoquer une clé gérée par le client avec Azure CLI, appelez la commande [az keyvault delete-policy](/cli/azure/keyvault#az-keyvault-delete-policy), comme indiqué dans l’exemple suivant. N’oubliez pas de remplacer les valeurs de l’espace réservé entre crochets par vos propres valeurs et d’utiliser les variables définies dans les exemples précédents.

```azurecli-interactive
az keyvault delete-policy \
    --name <key-vault> \
    --object-id $storage_account_principal
```

---

## <a name="disable-customer-managed-keys"></a>Désactiver les clés gérées par le client

Quand vous désactivez les clés gérées par le client, votre compte de stockage est de nouveau chiffré avec des clés gérées par Microsoft.

# <a name="azure-portal"></a>[Azure portal](#tab/portal)

Pour désactiver des clés gérées par le client dans le portail Azure, procédez comme suit :

1. Accédez à votre compte de stockage et affichez les paramètres de **chiffrement**.
1. Désactivez la case à cocher en regard du paramètre **Utiliser votre propre clé**.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Pour désactiver les clés gérées par le client avec PowerShell, appelez la cmdlet [Set-AzStorageAccount](/powershell/module/az.storage/set-azstorageaccount) avec l’option `-StorageEncryption`, comme indiqué dans l’exemple suivant. N’oubliez pas de remplacer les valeurs de l’espace réservé entre crochets par vos propres valeurs et d’utiliser les variables définies dans les exemples précédents.

```powershell
Set-AzStorageAccount -ResourceGroupName $storageAccount.ResourceGroupName `
    -AccountName $storageAccount.StorageAccountName `
    -StorageEncryption  
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour désactiver les clés gérées par le client avec Azure CLI, appelez la commande [az storage account update](/cli/azure/storage/account#az-storage-account-update) et définissez le `--encryption-key-source parameter` sur `Microsoft.Storage`, comme indiqué dans l’exemple suivant. N’oubliez pas de remplacer les valeurs de l’espace réservé entre crochets par vos propres valeurs et d’utiliser les variables définies dans les exemples précédents.

```azurecli-interactive
az storage account update
    --name <storage-account> \
    --resource-group <resource_group> \
    --encryption-key-source Microsoft.Storage
```

---

## <a name="next-steps"></a>Étapes suivantes

- [Chiffrement du stockage Azure pour les données au repos](storage-service-encryption.md)
- [Clés gérées par le client pour le chiffrement du service Stockage Azure](customer-managed-keys-overview.md)
- [Configurer le chiffrement avec les clés gérées par le client stockées dans le HSM géré par Azure Key Vault (préversion)](customer-managed-keys-configure-key-vault-hsm.md)
