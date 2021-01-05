---
title: Supprimer le protecteur TDE (PowerShell et Azure CLI)
titleSuffix: Azure SQL Database & Azure Synapse Analytics
description: Découvrez comment répondre à un protecteur TDE potentiellement compromis pour Azure SQL Database ou Azure Synapse Analytics à l’aide de TDE avec la prise en charge BYOK (Bring Your Own Key).
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: seo-lt-2019 sqldbrb=1, devx-track-azurecli
ms.devlang: ''
ms.topic: how-to
author: jaszymas
ms.author: jaszymas
ms.reviewer: vanto
ms.date: 02/24/2020
ms.openlocfilehash: 31298be4d50c7f562e2e2b9adbf889b165b197e5
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96461855"
---
# <a name="remove-a-transparent-data-encryption-tde-protector-using-powershell"></a>Supprimer un protecteur Transparent Data Encryption (TDE) à l’aide de PowerShell
[!INCLUDE[appliesto-sqldb-asa](../includes/appliesto-sqldb-asa.md)]


Cette rubrique décrit comment répondre à un protecteur TDE potentiellement compromis pour Azure SQL Database ou Azure Synapse Analytics à l’aide de TDE avec des clés gérées par le client dans Azure Key Vault – Prise en charge du modèle BYOK (Bring Your Own Key). Pour en savoir plus sur la prise en charge BYOK pour TDE, consultez la [page de vue d’ensemble](transparent-data-encryption-byok-overview.md).

> [!CAUTION]
> Les procédures décrites dans cet article doivent uniquement être effectuées dans les cas extrêmes ou dans des environnements de test. Examinez attentivement ces étapes, car la suppression des protecteurs TDE activement utilisés d’Azure Key Vault entraîne l’**indisponibilité de la base de données**.

Si vous suspectez qu’une clé est compromise, en cas d’accès non autorisé d’un utilisateur ou d’un service à cet clé, par exemple, nous vous recommandons de supprimer cette clé.

Gardez à l’esprit qu’une fois le protecteur TDE supprimé au sein de Key Vault, après un délai maximal de 10 minutes, toutes les bases de données chiffrées se mettent à refuser l’ensemble des connexions avec le message d’erreur correspondant, puis passent à l’état [Inaccessible](./transparent-data-encryption-byok-overview.md#inaccessible-tde-protector).

Ce guide pratique expose deux approches possibles en fonction du résultat souhaité après la réponse à un incident :

- Pour rendre **inaccessibles** les bases de données dans Azure SQL Database/Azure Synapse Analytics.
- Pour rendre **inaccessibles** les bases de données dans Azure SQL Database/Azure Synapse Analytics.

## <a name="prerequisites"></a>Prérequis

- Vous devez avoir un abonnement Azure et être un administrateur de cet abonnement
- Azure PowerShell doit être installé et en cours d’exécution.
- Ce guide pratique part du principe que vous utilisez déjà une clé Azure Key Vault comme protecteur TDE pour Azure SQL Database ou Azure Synapse. Consultez [Transparent Data Encryption with Bring Your Own Key support for Azure SQL Database and Data Warehouse](transparent-data-encryption-byok-overview.md) (Transparent Data Encryption avec la prise en charge Bring Your Own Key pour Azure SQL Database et Data Warehouse) pour en savoir plus.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

 Pour des instructions d’installation du module Az, consultez [Installer Azure PowerShell](/powershell/azure/install-az-ps). Pour des applets de commande spécifiques, consultez [AzureRM.Sql](/powershell/module/AzureRM.Sql/).

> [!IMPORTANT]
> Le module PowerShell Azure Resource Manager (RM) est toujours pris en charge, mais tous les développements à venir sont destinés au module Az.Sql. Le module AzureRM continue à recevoir des résolutions de bogues jusqu’à au moins décembre 2020.  Les arguments des commandes dans le module Az sont sensiblement identiques à ceux des modules AzureRm. Pour en savoir plus sur leur compatibilité, consultez [Présentation du nouveau module Az Azure PowerShell](/powershell/azure/new-azureps-module-az).

# <a name="the-azure-cli"></a>[L’interface de ligne de commande Microsoft Azure](#tab/azure-cli)

Pour l’installation, consultez [Installer l’interface de ligne de commande Azure](/cli/azure/install-azure-cli).

* * *

## <a name="check-tde-protector-thumbprints"></a>Vérifier les empreintes du protecteur TDE

Les étapes suivantes indiquent comment vérifier que les empreintes du protecteur TDE sont toujours en cours d’utilisation par des fichiers journaux virtuels d’une base de données spécifique.
Pour trouver l’empreinte du protecteur TDE actuel de la base de données et l’ID de base de données, exécutez la commande suivante :

```sql
SELECT [database_id],
       [encryption_state],
       [encryptor_type], /*asymmetric key means AKV, certificate means service-managed keys*/
       [encryptor_thumbprint]
 FROM [sys].[dm_database_encryption_keys]
```

La requête suivante retourne les fichiers journaux virtuels et les empreintes respectives du protecteur TDE en cours d’utilisation. Chaque empreinte différente fait référence à une autre clé dans Azure Key Vault (AKV) :

```sql
SELECT * FROM sys.dm_db_log_info (database_id)
```

Vous pouvez également utiliser PowerShell ou Azure CLI :

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

La commande PowerShell **Get-AzureRmSqlServerKeyVaultKey** fournit l’empreinte du protecteur TDE utilisé dans la requête, afin que vous puissiez voir quelles clés conserver et lesquelles supprimer dans AKV. Seules les clés qui ne sont plus utilisées par la base de données peuvent être supprimées de façon sécurisée d’Azure Key Vault.

# <a name="the-azure-cli"></a>[L’interface de ligne de commande Microsoft Azure](#tab/azure-cli)

La commande PowerShell **az sql server key show** fournit l’empreinte du protecteur TDE utilisé dans la requête. Vous pouvez ainsi voir quelles clés conserver et supprimer dans AKV. Seules les clés qui ne sont plus utilisées par la base de données peuvent être supprimées de façon sécurisée d’Azure Key Vault.

* * *

## <a name="keep-encrypted-resources-accessible"></a>Garder les ressources chiffrées accessibles

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Créez une [clé dans Key Vault](/powershell/module/az.keyvault/add-azkeyvaultkey). Assurez-vous que cette clé est créée dans un coffre de clés distinct à partir du protecteur TDE potentiellement compromis, étant donné que le contrôle d’accès est configuré au niveau du coffre.

2. Ajoutez la nouvelle clé au serveur en utilisant les cmdlets [Add-AzSqlServerKeyVaultKey](/powershell/module/az.sql/add-azsqlserverkeyvaultkey) et [Set-AzSqlServerTransparentDataEncryptionProtector](/powershell/module/az.sql/set-azsqlservertransparentdataencryptionprotector) et mettez-la à jour en tant que nouveau protecteur TDE du serveur.

   ```powershell
   # add the key from Key Vault to the server  
   Add-AzSqlServerKeyVaultKey -ResourceGroupName <SQLDatabaseResourceGroupName> -ServerName <LogicalServerName> -KeyId <KeyVaultKeyId>

   # set the key as the TDE protector for all resources under the server
   Set-AzSqlServerTransparentDataEncryptionProtector -ResourceGroupName <SQLDatabaseResourceGroupName> `
       -ServerName <LogicalServerName> -Type AzureKeyVault -KeyId <KeyVaultKeyId>
   ```

3. Assurez-vous que le serveur et tous les réplicas ont été mis à jour pour le nouveau protecteur TDE à l’aide de la cmdlet [Get-AzSqlServerTransparentDataEncryptionProtector](/powershell/module/az.sql/get-azsqlservertransparentdataencryptionprotector).

   > [!NOTE]
   > Quelques minutes peuvent être nécessaires pour que le nouveau protecteur TDE soit distribué à l’ensemble des bases de données et bases de données secondaires du serveur.

   ```powershell
   Get-AzSqlServerTransparentDataEncryptionProtector -ServerName <LogicalServerName> -ResourceGroupName <SQLDatabaseResourceGroupName>
   ```

4. Faites une [sauvegarde de la nouvelle clé](/powershell/module/az.keyvault/backup-azkeyvaultkey) dans Key Vault.

   ```powershell
   # -OutputFile parameter is optional; if removed, a file name is automatically generated.
   Backup-AzKeyVaultKey -VaultName <KeyVaultName> -Name <KeyVaultKeyName> -OutputFile <DesiredBackupFilePath>
   ```

5. Supprimez la clé compromise de Key Vault à l’aide de la cmdlet [Remove-AzKeyVaultKey](/powershell/module/az.keyvault/remove-azkeyvaultkey).

   ```powershell
   Remove-AzKeyVaultKey -VaultName <KeyVaultName> -Name <KeyVaultKeyName>
   ```

6. Pour restaurer ultérieurement une clé dans Key Vault à l’aide de la cmdlet [Restore-AzKeyVaultKey](/powershell/module/az.keyvault/restore-azkeyvaultkey) :

   ```powershell
   Restore-AzKeyVaultKey -VaultName <KeyVaultName> -InputFile <BackupFilePath>
   ```

# <a name="the-azure-cli"></a>[L’interface de ligne de commande Microsoft Azure](#tab/azure-cli)

Pour obtenir des informations de référence sur les commandes, consultez le [coffre de clés Azure CLI](/cli/azure/keyvault/key).

1. Créez une [clé dans Key Vault](/cli/azure/keyvault/key#az-keyvault-key-create). Assurez-vous que cette clé est créée dans un coffre de clés distinct à partir du protecteur TDE potentiellement compromis, étant donné que le contrôle d’accès est configuré au niveau du coffre.

2. Ajoutez la nouvelle clé au serveur et mettez-la à jour en tant que nouveau protecteur TDE du serveur.

   ```azurecli
   # add the key from Key Vault to the server  
   az sql server key create --kid <KeyVaultKeyId> --resource-group <SQLDatabaseResourceGroupName> --server <LogicalServerName>

   # set the key as the TDE protector for all resources under the server
   az sql server tde-key set --server-key-type AzureKeyVault --kid <KeyVaultKeyId> --resource-group <SQLDatabaseResourceGroupName> --server <LogicalServerName>
   ```

3. Assurez-vous que le serveur et tous les réplicas ont été mis à jour vers le nouveau protecteur TDE.

   > [!NOTE]
   > Quelques minutes peuvent être nécessaires pour que le nouveau protecteur TDE soit distribué à l’ensemble des bases de données et bases de données secondaires du serveur.

   ```azurecli
   az sql server tde-key show --resource-group <SQLDatabaseResourceGroupName> --server <LogicalServerName>
   ```

4. Faites une sauvegarde de la nouvelle clé dans Key Vault.

   ```azurecli
   # --file parameter is optional; if removed, a file name is automatically generated.
   az keyvault key backup --file <DesiredBackupFilePath> --name <KeyVaultKeyName> --vault-name <KeyVaultName>
   ```

5. Supprimez la clé compromise de Key Vault.

   ```azurecli
   az keyvault key delete --name <KeyVaultKeyName> --vault-name <KeyVaultName>
   ```

6. Pour restaurer une clé dans Key Vault à l’avenir.

   ```azurecli
   az keyvault key restore --file <BackupFilePath> --vault-name <KeyVaultName>
   ```

* * *

## <a name="make-encrypted-resources-inaccessible"></a>Rendre les ressources chiffrées inaccessibles

1. Supprimez les bases de données chiffrées avec la clé potentiellement compromise.

   Les bases de données et les fichiers journaux étant automatiquement sauvegardés, une restauration dans le temps peut donc être effectuée à tout moment (à condition que vous fournissiez la clé). Les bases de données doivent être supprimées avant de supprimer un protecteur TDE actif pour empêcher une perte de données éventuelle de 10 minutes des transactions les plus récentes.

2. Sauvegardez l’élément de clé du protecteur TDE dans Key Vault.
3. Supprimez la clé potentiellement compromise de Key Vault.

[!INCLUDE [sql-database-akv-permission-delay](../includes/sql-database-akv-permission-delay.md)]

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment faire pivoter le protecteur TDE d’un serveur pour se conformer à des exigences de sécurité : [Faites pivoter le protecteur Transparent Data Encryption à l’aide de PowerShell](transparent-data-encryption-byok-key-rotation.md)
- Prise en main de la prise en charge de Bring Your Own Key pour TDE : [ Activer TDE à l’aide de votre propre clé depuis Key Vault à l’aide de PowerShell](transparent-data-encryption-byok-configure.md)