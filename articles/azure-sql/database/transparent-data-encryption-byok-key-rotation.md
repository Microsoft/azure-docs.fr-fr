---
title: Faire pivoter le protecteur TDE (PowerShell et Azure CLI)
titleSuffix: Azure SQL Database & Azure Synapse Analytics
description: Découvrez comment faire pivoter le protecteur Transparent Data Encryption (TDE) pour un serveur dans Azure et utilisé par Azure SQL Database et Azure Synapse Analytics à l’aide de PowerShell et d’Azure CLI.
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: seo-lt-2019 sqldbrb=1, devx-track-azurecli
ms.devlang: ''
ms.topic: how-to
author: jaszymas
ms.author: jaszymas
ms.reviewer: vanto
ms.date: 03/12/2019
ms.openlocfilehash: 37931aee24e2dbcab03bca400d58f236601c87e5
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93321420"
---
# <a name="rotate-the-transparent-data-encryption-tde-protector"></a>Faire pivoter le protecteur Transparent Data Encryption (TDE)
[!INCLUDE[appliesto-sqldb-asa](../includes/appliesto-sqldb-asa.md)]


Cet article décrit la rotation des clés pour un [serveur](logical-servers.md) à l’aide d’un protecteur TDE d’Azure Key Vault. La rotation du protecteur TDE logique pour serveur désigne le basculement vers une nouvelle clé asymétrique qui protège les bases de données sur le serveur. La rotation des clés est une opération en ligne et ne doit prendre que quelques secondes, car elle ne déchiffre et rechiffre que la clé de chiffrement des données de la base de données, et non la base de données entière.

Ce guide décrit deux options pour faire pivoter le protecteur TDE sur le serveur.

> [!NOTE]
> Un pool SQL dédié mis en pause dans Azure Synapse Analytics doit reprendre avant les rotations de clés.

> [!IMPORTANT]
> Ne supprimez pas les versions précédentes de la clé après une substitution. Lorsque les clés sont substituées, certaines données sont toujours chiffrées avec les clés précédentes, comme les anciennes sauvegardes de base de données.

## <a name="prerequisites"></a>Prérequis

- Ce guide pratique part du principe que vous utilisez déjà une clé Azure Key Vault comme protecteur TDE pour Azure SQL Database ou Azure Synapse Analytics. Consultez [Transparent Data Encryption with BYOK Support](transparent-data-encryption-byok-overview.md) (Chiffrement transparent des données avec prise en charge BYOK).
- Azure PowerShell doit être installé et en cours d’exécution.
- [Recommandé mais facultatif] Créez tout d’abord le matériel de clé pour le protecteur TDE dans un module de sécurité matériel (HSM) ou le magasin de clés local, puis importez le matériel de clé dans Azure Key Vault. Suivez le [instructions sur l’utilisation d’un module de sécurité matériel (HSM) et Key Vault](../../key-vault/general/overview.md) pour en savoir plus.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Pour des instructions d’installation du module Az, consultez [Installer Azure PowerShell](/powershell/azure/install-az-ps). Pour des applets de commande spécifiques, consultez [AzureRM.Sql](/powershell/module/AzureRM.Sql/).

> [!IMPORTANT]
> Le module PowerShell Azure Resource Manager (RM) est toujours pris en charge, mais tous les développements à venir sont destinés au module Az.Sql. Le module AzureRM continue à recevoir des résolutions de bogues jusqu’à au moins décembre 2020.  Les arguments des commandes dans le module Az sont sensiblement identiques à ceux des modules AzureRm. Pour en savoir plus sur leur compatibilité, consultez [Présentation du nouveau module Az Azure PowerShell](/powershell/azure/new-azureps-module-az).

# <a name="the-azure-cli"></a>[L’interface de ligne de commande Microsoft Azure](#tab/azure-cli)

Pour l’installation, consultez [Installer l’interface de ligne de commande Azure](/cli/azure/install-azure-cli).

* * *

## <a name="manual-key-rotation"></a>Rotation des clés manuelle

La rotation manuelle des clés utilise les commandes suivantes pour ajouter une clé entièrement nouvelle, qui peut être sous un nouveau nom de clé ou même un autre coffre de clés. L’utilisation de cette approche prend en charge l’ajout de la même clé à différents coffres de clés afin de prendre en charge les scénarios de haute disponibilité et de géo-reprise.

> [!NOTE]
> La longueur combinée du nom du coffre de clés et du nom de la clé ne peut pas dépasser 94 caractères.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Utilisez les applets de commande [Add-AzKeyVaultKey](/powershell/module/az.keyvault/Add-AzKeyVaultKey), [Add-AzSqlServerKeyVaultKey](/powershell/module/az.sql/add-azsqlserverkeyvaultkey) et [Set-AzSqlServerTransparentDataEncryptionProtector](/powershell/module/az.sql/set-azsqlservertransparentdataencryptionprotector).

```powershell
# add a new key to Key Vault
Add-AzKeyVaultKey -VaultName <keyVaultName> -Name <keyVaultKeyName> -Destination <hardwareOrSoftware>

# add the new key from Key Vault to the server
Add-AzSqlServerKeyVaultKey -KeyId <keyVaultKeyId> -ServerName <logicalServerName> -ResourceGroup <SQLDatabaseResourceGroupName>
  
# set the key as the TDE protector for all resources under the server
Set-AzSqlServerTransparentDataEncryptionProtector -Type AzureKeyVault -KeyId <keyVaultKeyId> `
   -ServerName <logicalServerName> -ResourceGroup <SQLDatabaseResourceGroupName>
```

# <a name="the-azure-cli"></a>[L’interface de ligne de commande Microsoft Azure](#tab/azure-cli)

Utilisez les commandes [az keyvault key create](/cli/azure/keyvault/key#az-keyvault-key-create), [az sql server key create](/cli/azure/sql/server/key#az-sql-server-key-create) et [az sql server tde-key set](/cli/azure/sql/server/tde-key#az-sql-server-tde-key-set).

```azurecli
# add a new key to Key Vault
az keyvault key create --name <keyVaultKeyName> --vault-name <keyVaultName> --protection <hsmOrSoftware>

# add the new key from Key Vault to the server
az sql server key create --kid <keyVaultKeyId> --resource-group <SQLDatabaseResourceGroupName> --server <logicalServerName>

# set the key as the TDE protector for all resources under the server
az sql server tde-key set --server-key-type AzureKeyVault --kid <keyVaultKeyId> --resource-group <SQLDatabaseResourceGroupName> --server <logicalServerName>
```

* * *

## <a name="switch-tde-protector-mode"></a>Basculer le mode du protecteur TDE

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

- Pour faire passer le protecteur TDE du mode géré par Microsoft au mode BYOK, utilisez la cmdlet [Set-AzSqlServerTransparentDataEncryptionProtector](/powershell/module/az.sql/set-azsqlservertransparentdataencryptionprotector).

   ```powershell
   Set-AzSqlServerTransparentDataEncryptionProtector -Type AzureKeyVault `
       -KeyId <keyVaultKeyId> -ServerName <logicalServerName> -ResourceGroup <SQLDatabaseResourceGroupName>
   ```

- Pour faire passer le protecteur TDE du mode BYOK au mode géré par Microsoft, utilisez la cmdlet [Set-AzSqlServerTransparentDataEncryptionProtector](/powershell/module/az.sql/set-azsqlservertransparentdataencryptionprotector).

   ```powershell
   Set-AzSqlServerTransparentDataEncryptionProtector -Type ServiceManaged `
       -ServerName <logicalServerName> -ResourceGroup <SQLDatabaseResourceGroupName>
   ```

# <a name="the-azure-cli"></a>[L’interface de ligne de commande Microsoft Azure](#tab/azure-cli)

Les exemples suivants utilisent [az sql server tde-key set](/powershell/module/az.sql/set-azsqlservertransparentdataencryptionprotector).

- Pour basculer le protecteur TDE du mode géré par Microsoft au mode BYOK,

   ```azurecli
   az sql server tde-key set --server-key-type AzureKeyVault --kid <keyVaultKeyId> --resource-group <SQLDatabaseResourceGroupName> --server <logicalServerName>
   ```

- Pour basculer le protecteur TDE du mode BYOK au mode géré par Microsoft,

   ```azurecli
   az sql server tde-key set --server-key-type ServiceManaged --resource-group <SQLDatabaseResourceGroupName> --server <logicalServerName>
   ```

* * *

## <a name="next-steps"></a>Étapes suivantes

- En cas de risque de sécurité, découvrez comment supprimer un protecteur TDE potentiellement compromis : [Supprimez une clé potentiellement compromise](transparent-data-encryption-byok-remove-tde-protector.md).

- Prise en main de l’intégration d’Azure Key Vault et de la prise en charge Bring Your Own Key pour TDE : [Activer TDE à l’aide de votre propre clé Key Vault à l’aide de PowerShell](transparent-data-encryption-byok-configure.md).