---
title: Activer SQL TDE avec Azure Key Vault
titleSuffix: Azure SQL Database & SQL Managed Instance & Azure Synapse Analytics
description: Découvrez comment configurer Azure SQL Database et Azure Synapse Analytics pour commencer à utiliser Transparent Data Encryption (TDE) pour le chiffrement au repos à l’aide de PowerShell ou Azure CLI.
services: sql-database
ms.service: sql-db-mi
ms.subservice: security
ms.custom: seo-lt-2019 sqldbrb=1, devx-track-azurecli
ms.devlang: ''
ms.topic: how-to
author: jaszymas
ms.author: jaszymas
ms.reviewer: vanto
ms.date: 03/12/2019
ms.openlocfilehash: b4480f3d28cb89165a6ba3c5b26b10b1aba9765c
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96461843"
---
# <a name="powershell-and-the-azure-cli-enable-transparent-data-encryption-with-customer-managed-key-from-azure-key-vault"></a>PowerShell et Azure CLI : Activer Transparent Data Encryption à l’aide d'une clé gérée par le client à partir d'Azure Key Vault
[!INCLUDE[appliesto-sqldb-sqlmi-asa](../includes/appliesto-sqldb-sqlmi-asa.md)]

Cet article explique comment utiliser une clé Azure Key Vault pour Transparent Data Encryption (TDE) sur Azure SQL Database ou Azure Synapse Analytics. Pour en savoir plus sur TDE avec intégration Azure Key Vault - Prise en charge de BYOK (Bring Your Own Key), consultez [TDE avec clés managées dans Azure Key Vault](transparent-data-encryption-byok-overview.md).

> [!NOTE] 
> Azure SQL prend désormais en charge l’utilisation d’une clé RSA stockée dans un HSM géré en tant que protecteur TDE. Cette fonctionnalité est disponible en **préversion publique**. Azure Key Vault HSM géré est un service cloud entièrement géré, à haut niveau de disponibilité et à un seul locataire, qui vous permet de protéger les clés de chiffrement de vos applications cloud à l’aide de modules de sécurité matériels certifiés FIPS 140-2 de niveau 3. En savoir plus sur les [HSM managés](../../key-vault/managed-hsm/index.yml).

## <a name="prerequisites-for-powershell"></a>Prérequis pour PowerShell

- Vous devez avoir un abonnement Azure et être un administrateur de cet abonnement.
- [Recommandé mais facultatif] Avoir un module de sécurité matériel (HSM) ou un magasin de clés local pour créer une copie locale de l’élément de clé du protecteur TDE.
- Azure PowerShell doit être installé et en cours d’exécution.
- Créez un coffre Azure Key Vault et une clé pour utiliser TDE.
  - [Instructions pour utiliser un module de sécurité matériel (HSM) et Key Vault](../../key-vault/keys/hsm-protected-keys.md)
    - Le coffre de clés doit avoir la propriété suivante à utiliser pour TDE :
  - [soft-delete](../../key-vault/general/soft-delete-overview.md) et protection contre le vidage
- La clé doit avoir les attributs suivants à utiliser pour TDE :
  - Aucune date d’expiration
  - Non activée
  - En mesure d’effectuer des opérations *get*, *wrap key*, *unwrap key*
- **(En préversion)** Pour utiliser une clé de HSM managé, suivez les instructions pour [créer et activer un HSM managé à l’aide d’Azure CLI](../../key-vault/managed-hsm/quick-create-cli.md)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Pour des instructions d’installation du module Az, consultez [Installer Azure PowerShell](/powershell/azure/install-az-ps). Pour des applets de commande spécifiques, consultez [AzureRM.Sql](/powershell/module/AzureRM.Sql/).

Pour obtenir des informations spécifiques sur Key Vault, consultez [Instructions pour utiliser PowerShell à partir de Key Vault](../../key-vault/secrets/quick-create-powershell.md) et [Guide pratique pour utiliser la suppression réversible Key Vault avec l’interface PowerShell](../../key-vault/general/key-vault-recovery.md).

> [!IMPORTANT]
> Le module PowerShell Azure Resource Manager (RM) est toujours pris en charge, mais tous les développements à venir sont destinés au module Az.Sql. Le module AzureRM continue à recevoir des résolutions de bogues jusqu’à au moins décembre 2020.  Les arguments des commandes dans le module Az sont sensiblement identiques à ceux des modules AzureRm. Pour en savoir plus sur leur compatibilité, consultez [Présentation du nouveau module Az Azure PowerShell](/powershell/azure/new-azureps-module-az).

## <a name="assign-an-azure-active-directory-azure-ad-identity-to-your-server"></a>Affecter une identité Azure Active Directory (Azure AD) à votre serveur

Si vous avez un [serveur](logical-servers.md) existant, utilisez ce qui suit pour ajouter une identité Azure Active Directory (Azure AD) à votre serveur :

   ```powershell
   $server = Set-AzSqlServer -ResourceGroupName <SQLDatabaseResourceGroupName> -ServerName <LogicalServerName> -AssignIdentity
   ```

Si vous créez un serveur, utilisez la cmdlet [New-AzSqlServer](/powershell/module/az.sql/new-azsqlserver) avec la balise-Identity pour ajouter une identité Azure AD lors de la création du serveur :

   ```powershell
   $server = New-AzSqlServer -ResourceGroupName <SQLDatabaseResourceGroupName> -Location <RegionName> `
       -ServerName <LogicalServerName> -ServerVersion "12.0" -SqlAdministratorCredentials <PSCredential> -AssignIdentity
   ```

## <a name="grant-key-vault-permissions-to-your-server"></a>Accorder des autorisations Key Vault à votre serveur

Utilisez la cmdlet [Set-AzKeyVaultAccessPolicy](/powershell/module/az.keyvault/set-azkeyvaultaccesspolicy) pour accorder à votre serveur l’accès au coffre de clés avant d’utiliser une clé à partir de celui-ci pour TDE.

   ```powershell
   Set-AzKeyVaultAccessPolicy -VaultName <KeyVaultName> `
       -ObjectId $server.Identity.PrincipalId -PermissionsToKeys get, wrapKey, unwrapKey
   ```
Pour ajouter des autorisations à votre serveur sur un HSM managé, ajoutez le rôle RBAC local « Chiffrement du service du HSM managé » au serveur. Cela permet au serveur d’effectuer des opérations get, wrap key et unwrap key sur les clés du HSM managé.
[Instructions pour la configuration de l’accès serveur sur un HSM managé](../../key-vault/managed-hsm/role-management.md)

## <a name="add-the-key-vault-key-to-the-server-and-set-the-tde-protector"></a>Ajouter la clé Key Vault au serveur et définir le protecteur TDE

- Utilisez le cmdlet [Get-AzKeyVaultKey](/powershell/module/az.keyvault/get-azkeyvaultkey?view=azps-2.4.0) pour récupérer l’ID de la clé à partir du coffre de clé
- Utilisez la cmdlet [Add-AzSqlServerKeyVaultKey](/powershell/module/az.sql/add-azsqlserverkeyvaultkey) pour ajouter la clé de Key Vault au serveur.
- Utilisez la cmdlet [Set-AzSqlServerTransparentDataEncryptionProtector](/powershell/module/az.sql/set-azsqlservertransparentdataencryptionprotector) pour définir la clé en tant que protecteur TDE pour toutes les ressources de serveur.
- Utilisez la cmdlet [Get-AzSqlServerTransparentDataEncryptionProtector](/powershell/module/az.sql/get-azsqlservertransparentdataencryptionprotector) pour confirmer que le protecteur TDE a été configuré comme prévu.

> [!NOTE]
> **(En préversion)** Pour les clés de HSM managé, utilisez la version Az.Sql 2.11.1 de PowerShell.

> [!NOTE]
> La longueur combinée du nom du coffre de clés et du nom de la clé ne peut pas dépasser 94 caractères.

> [!TIP]
> Voici un exemple de KeyId de Key Vault : <br/>https://contosokeyvault.vault.azure.net/keys/Key1/1a1a2b2b3c3c4d4d5e5e6f6f7g7g8h8h
>
> Exemple de KeyId de HSM managé :<br/>https://contosoMHSM.managedhsm.azure.net/keys/myrsakey

```powershell
# add the key from Key Vault to the server
Add-AzSqlServerKeyVaultKey -ResourceGroupName <SQLDatabaseResourceGroupName> -ServerName <LogicalServerName> -KeyId <KeyVaultKeyId>

# set the key as the TDE protector for all resources under the server
Set-AzSqlServerTransparentDataEncryptionProtector -ResourceGroupName <SQLDatabaseResourceGroupName> -ServerName <LogicalServerName> `
   -Type AzureKeyVault -KeyId <KeyVaultKeyId>

# confirm the TDE protector was configured as intended
Get-AzSqlServerTransparentDataEncryptionProtector -ResourceGroupName <SQLDatabaseResourceGroupName> -ServerName <LogicalServerName>
```

## <a name="turn-on-tde"></a>Activer TDE

Utilisez la cmdlet [Set-AzSqlDatabaseTransparentDataEncryption](/powershell/module/az.sql/set-azsqldatabasetransparentdataencryption) pour activer TDE.

```powershell
Set-AzSqlDatabaseTransparentDataEncryption -ResourceGroupName <SQLDatabaseResourceGroupName> `
   -ServerName <LogicalServerName> -DatabaseName <DatabaseName> -State "Enabled"
```

TDE est maintenant activé pour l’entrepôt de données ou la base de données, avec une clé de chiffrement dans Key Vault.

## <a name="check-the-encryption-state-and-encryption-activity"></a>Vérifier l’état de chiffrement et l’activité de chiffrement

Utilisez la cmdlet [Get-AzSqlDatabaseTransparentDataEncryption](/powershell/module/az.sql/get-azsqldatabasetransparentdataencryption) pour obtenir l’état de chiffrement et la cmdlet [Get-AzSqlDatabaseTransparentDataEncryptionActivity](/powershell/module/az.sql/get-azsqldatabasetransparentdataencryptionactivity) pour vérifier la progression du chiffrement d’une base de données ou d’un entrepôt de données.

```powershell
# get the encryption state
Get-AzSqlDatabaseTransparentDataEncryption -ResourceGroupName <SQLDatabaseResourceGroupName> `
   -ServerName <LogicalServerName> -DatabaseName <DatabaseName> `

# check the encryption progress for a database or data warehouse
Get-AzSqlDatabaseTransparentDataEncryptionActivity -ResourceGroupName <SQLDatabaseResourceGroupName> `
   -ServerName <LogicalServerName> -DatabaseName <DatabaseName>  
```

# <a name="the-azure-cli"></a>[L’interface de ligne de commande Microsoft Azure](#tab/azure-cli)

Pour installer la version requise d’Azure CLI (version 2.0 ou ultérieure) et la connecter à votre abonnement Azure, consultez [Installer et configurer l’interface de ligne de commande multiplateforme Azure 2.0](/cli/azure/install-azure-cli).

Pour obtenir des informations spécifiques sur Key Vault, consultez [Gérer Key Vault à l’aide de l’interface de ligne de commande (CLI) 2.0](../../key-vault/general/manage-with-cli2.md) et [Guide pratique pour utiliser la suppression réversible Key Vault avec l’interface CLI](../../key-vault/general/key-vault-recovery.md).

## <a name="assign-an-azure-ad-identity-to-your-server"></a>Assigner une identité Azure AD à votre serveur

```azurecli
# create server (with identity) and database
az sql server create --name <servername> --resource-group <rgname>  --location <location> --admin-user <user> --admin-password <password> --assign-identity
az sql db create --name <dbname> --server <servername> --resource-group <rgname>
```

> [!TIP]
> Conserver l'élément « principalID » lié à la création du serveur ; il correspond à l'ID d'objet utilisé pour attribuer les autorisations d'accès au coffre de clés de l'étape suivante

## <a name="grant-key-vault-permissions-to-your-server"></a>Accorder des autorisations Key Vault à votre serveur

```azurecli
# create key vault, key and grant permission
az keyvault create --name <kvname> --resource-group <rgname> --location <location> --enable-soft-delete true
az keyvault key create --name <keyname> --vault-name <kvname> --protection software
az keyvault set-policy --name <kvname>  --object-id <objectid> --resource-group <rgname> --key-permissions wrapKey unwrapKey get
```

> [!TIP]
> Conserver l’URI de clé ou keyID de la nouvelle clé pour l'étape suivante, par exemple : https://contosokeyvault.vault.azure.net/keys/Key1/1a1a2b2b3c3c4d4d5e5e6f6f7g7g8h8h

## <a name="add-the-key-vault-key-to-the-server-and-set-the-tde-protector"></a>Ajouter la clé Key Vault au serveur et définir le protecteur TDE

```azurecli
# add server key and update encryption protector
az sql server key create --server <servername> --resource-group <rgname> --kid <keyID>
az sql server tde-key set --server <servername> --server-key-type AzureKeyVault  --resource-group <rgname> --kid <keyID>
```

> [!NOTE]
> La longueur combinée du nom du coffre de clés et du nom de la clé ne peut pas dépasser 94 caractères.

## <a name="turn-on-tde"></a>Activer TDE

```azurecli
# enable encryption
az sql db tde set --database <dbname> --server <servername> --resource-group <rgname> --status Enabled
```

TDE est maintenant activé pour l’entrepôt de données ou la base de données, avec une clé de chiffrement gérée par le client dans Azure Key Vault.

## <a name="check-the-encryption-state-and-encryption-activity"></a>Vérifier l’état de chiffrement et l’activité de chiffrement

```azurecli
# get encryption scan progress
az sql db tde list-activity --database <dbname> --server <servername> --resource-group <rgname>  

# get whether encryption is on or off
az sql db tde show --database <dbname> --server <servername> --resource-group <rgname>
```

* * *

## <a name="useful-powershell-cmdlets"></a>Applets de commande PowerShell utiles

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

- Utilisez la cmdlet [Set-AzSqlDatabaseTransparentDataEncryption](/powershell/module/az.sql/set-azsqldatabasetransparentdataencryption) pour désactiver TDE.

   ```powershell
   Set-AzSqlDatabaseTransparentDataEncryption -ServerName <LogicalServerName> -ResourceGroupName <SQLDatabaseResourceGroupName> `
      -DatabaseName <DatabaseName> -State "Disabled"
   ```

- Utilisez la cmdlet [Get-AzSqlServerKeyVaultKey](/powershell/module/az.sql/get-azsqlserverkeyvaultkey) pour renvoyer la liste de clés Key Vault ajoutées au serveur.

   ```powershell
   # KeyId is an optional parameter, to return a specific key version
   Get-AzSqlServerKeyVaultKey -ServerName <LogicalServerName> -ResourceGroupName <SQLDatabaseResourceGroupName>
   ```

- Utilisez la cmdlet [Remove-AzSqlServerKeyVaultKey](/powershell/module/az.sql/remove-azsqlserverkeyvaultkey) pour supprimer une clé Key Vault du serveur.

   ```powershell
   # the key set as the TDE Protector cannot be removed
   Remove-AzSqlServerKeyVaultKey -KeyId <KeyVaultKeyId> -ServerName <LogicalServerName> -ResourceGroupName <SQLDatabaseResourceGroupName>
   ```

# <a name="the-azure-cli"></a>[L’interface de ligne de commande Microsoft Azure](#tab/azure-cli)

- Pour plus d’informations sur les paramètres de base de données généraux, consultez [az sql](/cli/azure/sql).

- Pour plus d’informations sur les paramètres de clé de coffre, consultez [AZ SQL Server Key](/cli/azure/sql/server/key).

- Pour plus d’informations sur les paramètres TDE, consultez [az sql server tde-key](/cli/azure/sql/server/tde-key) et [az sql db tde](/cli/azure/sql/db/tde).

* * *

## <a name="troubleshooting"></a>Dépannage

Vérifiez les points suivants en cas de problème :

- Si le coffre de clés est introuvable, vérifiez que vous êtes dans le bon abonnement.

   # <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

   ```powershell
   Get-AzSubscription -SubscriptionId <SubscriptionId>
   ```

   # <a name="the-azure-cli"></a>[L’interface de ligne de commande Microsoft Azure](#tab/azure-cli)

   ```powershell
   az account show - s <SubscriptionId>
   ```

   * * *

- Si la nouvelle clé ne peut pas être ajoutée au serveur ou si elle ne peut pas être mise à jour en tant que protecteur TDE, vérifiez les points suivants :
   - La clé ne doit pas avoir de date d’expiration
   - La clé doit avoir les opérations *get*, *wrap key* et *unwrap key* activées.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment faire pivoter le protecteur TDE d’un serveur pour se conformer à des exigences de sécurité : [Faites pivoter le protecteur Transparent Data Encryption à l’aide de PowerShell](transparent-data-encryption-byok-key-rotation.md).
- En cas de risque pour la sécurité, découvrez comment supprimer un protecteur TDE potentiellement compromis : [Supprimez une clé potentiellement compromise](transparent-data-encryption-byok-remove-tde-protector.md).
