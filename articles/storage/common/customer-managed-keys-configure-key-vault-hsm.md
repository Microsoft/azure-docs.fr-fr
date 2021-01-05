---
title: Configurer le chiffrement avec des clés gérées par le client stockées dans un module HSM géré par Azure Key Vault Azure (préversion)
titleSuffix: Azure Storage
description: Apprenez à configurer le chiffrement du service Stockage Azure avec des clés gérées par le client stockées dans un module HSM géré par Azure Key Vault Azure (préversion) à l'aide d'Azure CLI.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 09/21/2020
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.custom: devx-track-azurepowershell, devx-track-azurecli
ms.openlocfilehash: 2f57e801720c6b546a58b216422629d192e8d2e6
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94843313"
---
# <a name="configure-encryption-with-customer-managed-keys-stored-in-azure-key-vault-managed-hsm-preview"></a>Configurer le chiffrement avec des clés gérées par le client stockées dans un module HSM géré par Azure Key Vault Azure (préversion)

Le stockage Azure chiffre toutes les données dans un compte de stockage au repos. Par défaut, les données sont chiffrées avec des clés managées par Microsoft Pour mieux contrôler les clés de chiffrement, vous pouvez gérer vos propres clés. Les clés gérées par le client doivent être stockées dans Azure Key Vault ou dans un module de sécurité matériel (HSM) géré par Azure Key Vault Azure (préversion). Un module HSM géré par Azure Key Vault est un module HSM certifié FIPS 140-2 de niveau 3.

Cet article explique comment configurer le chiffrement avec des clés gérées par le client stockées dans un module HSM géré à l'aide d'Azure CLI. Pour savoir comment configurer le chiffrement avec des clés gérées par le client stockées dans un coffre de clés, consultez [Configurer le chiffrement avec des clés gérées par le client stockées dans Azure Key Vault](customer-managed-keys-configure-key-vault.md).

> [!NOTE]
> Azure Key Vault et le module HSM géré par Azure Key Vault prennent en charge les mêmes API et interfaces de gestion pour la configuration.

## <a name="assign-an-identity-to-the-storage-account"></a>Affecter une identité au compte de stockage

Tout d'abord, attribuez une identité managée affectée par le système au compte de stockage. Cette identité managée vous sera utile pour autoriser le compte de stockage à accéder au module HSM géré. Pour en savoir plus sur les identités managées affectées par le système, consultez la section [Que sont les identités managées pour les ressources Azure ?](../../active-directory/managed-identities-azure-resources/overview.md).

Pour attribuer une identité managée à l’aide d’Azure CLI, appelez [az storage account update](/cli/azure/storage/account#az-storage-account-update). N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs :

```azurecli
az storage account update \
    --name <storage-account> \
    --resource-group <resource_group> \
    --assign-identity
```

## <a name="assign-a-role-to-the-storage-account-for-access-to-the-managed-hsm"></a>Attribuer un rôle au compte de stockage pour l'accès au module HSM géré

Ensuite, attribuez le rôle **Chiffrement du service cryptographique du module HSM géré** à l'identité managée du compte de stockage afin que celui-ci dispose d'autorisations sur le module HSM géré. Microsoft vous recommande d'étendre l'attribution de rôle au niveau de la clé individuelle afin d'octroyer le moins de privilèges possible à l'identité managée.

Pour créer l'attribution de rôle du compte de stockage, appelez [az key vault role assignment create](/cli/azure/role/assignment#az_role_assignment_create). N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs.
  
```azurecli
storage_account_principal = $(az storage account show \
    --name <storage-account> \
    --resource-group <resource-group> \
    --query identity.principalId \
    --output tsv)

az keyvault role assignment create \
    --hsm-name <hsm-name> \
    --role "Managed HSM Crypto Service Encryption" \
    --assignee $storage_account_principal \
    --scope /keys/<key-name>
```

## <a name="configure-encryption-with-a-key-in-the-managed-hsm"></a>Configurer le chiffrement avec une clé dans le module HSM géré

Enfin, configurez le chiffrement du service Stockage Azure avec des clés gérées par le client pour utiliser une clé stockée dans le module HSM géré. Les types de clés pris en charge incluent les clés RSA-HSM de tailles 2048, 3072 et 4096. Installez Azure CLI 2.12.0 ou version ultérieure pour configurer le chiffrement de manière à utiliser une clé gérée par le client dans un module HSM géré. Pour plus d’informations, consultez la rubrique [Installation de l’interface de ligne de commande Azure (CLI)](/cli/azure/install-azure-cli).

Pour mettre à jour automatiquement la version d’une clé gérée par le client, omettez la version de la clé quand vous configurez le chiffrement avec les clés gérées par le client pour le compte de stockage. Pour mettre à jour les paramètres de chiffrement du compte de stockage, appelez [az storage account update](/cli/azure/storage/account#az_storage_account_update), comme illustré dans l’exemple suivant. Incluez le paramètre `--encryption-key-source parameter` et définissez-le sur `Microsoft.Keyvault` afin d'activer les clés gérées par le client pour le compte. N’oubliez pas de remplacer les valeurs d’espace réservé entre crochets par vos propres valeurs.

```azurecli
hsmurl = $(az keyvault show \
    --hsm-name <hsm-name> \
    --query properties.hsmUri \
    --output tsv)

az storage account update \
    --name <storage-account> \
    --resource-group <resource_group> \
    --encryption-key-name <key> \
    --encryption-key-source Microsoft.Keyvault \
    --encryption-key-vault $hsmurl
```

Pour une mise à jour manuelle de la version d'une clé gérée par le client, incluez la version de la clé lorsque vous configurez le chiffrement pour le compte de stockage :

```azurecli-interactive
az storage account update
    --name <storage-account> \
    --resource-group <resource_group> \
    --encryption-key-name <key> \
    --encryption-key-version $key_version \
    --encryption-key-source Microsoft.Keyvault \
    --encryption-key-vault $hsmurl
```

Quand vous mettez à jour manuellement la version de clé, vous devez mettre à jour les paramètres de chiffrement du compte de stockage afin d’utiliser la nouvelle version. Tout d’abord, lancez une requête pour l’URI du coffre de clés en appelant [az keyvault show](/cli/azure/keyvault#az-keyvault-show)et pour la version de la clé en appelant [az keyvault key list-versions](/cli/azure/keyvault/key#az-keyvault-key-list-versions). Appelez ensuite [az storage account update](/cli/azure/storage/account#az-storage-account-update) pour mettre à jour les paramètres de chiffrement du compte de stockage afin d’utiliser la nouvelle version de la clé, comme indiqué dans l’exemple précédent.

## <a name="next-steps"></a>Étapes suivantes

- [Chiffrement du stockage Azure pour les données au repos](storage-service-encryption.md)
- [Clés gérées par le client pour le chiffrement du service Stockage Azure](customer-managed-keys-overview.md)
