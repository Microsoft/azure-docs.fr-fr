---
title: Chiffrement transparent des données
titleSuffix: Azure SQL Database & SQL Managed Instance & Azure Synapse Analytics
description: Vue d’ensemble du chiffrement transparent des données pour Azure SQL Database, Azure SQL Managed Instance et Azure Synapse. Ce document en décrit les avantages et les options de configuration, notamment le chiffrement transparent des données géré par le service et la prise en charge du service Bring Your Own Key.
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: seo-lt-2019 sqldbrb=3
ms.devlang: ''
ms.topic: conceptual
author: jaszymas
ms.author: jaszymas
ms.reviewer: vanto
ms.date: 10/12/2020
ms.openlocfilehash: 8fbbd7a2aabc9de417f1eefd2513edba3119bfc0
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92791390"
---
# <a name="transparent-data-encryption-for-sql-database-sql-managed-instance-and-azure-synapse-analytics"></a>Chiffrement transparent des données pour SQL Database, SQL Managed Instance et Azure Synapse Analytics
[!INCLUDE[appliesto-sqldb-sqlmi-asa](../includes/appliesto-sqldb-sqlmi-asa.md)]

[Transparent Data Encryption (TDE)](/sql/relational-databases/security/encryption/transparent-data-encryption) est une technologie de chiffrement transparent des données qui contribue à protéger Azure SQL Database, Azure SQL Managed Instance et Azure Synapse Analytics contre les menaces d’activités hors connexion malveillantes en chiffrant les données au repos. Il assure le chiffrement et le déchiffrement en temps réel de la base de données, des sauvegardes associées et des fichiers journaux des transactions au repos, sans que cela nécessite de modifier l’application. Par défaut, TDE est activé pour toutes les bases de données SQL déployées récemment et doit être activé manuellement pour les anciennes bases de données d’Azure SQL Database et d’Azure SQL Managed Instance. TDE doit être activé manuellement pour Azure Synapse Analytics.

TDE effectue le chiffrement et le déchiffrement des données d’E/S en temps réel au niveau de la page. Chaque page est déchiffrée lorsqu’elle est lue en mémoire, puis chiffrée avant d’être écrite sur le disque. TDE chiffre le stockage d’une base de données entière à l’aide d’une clé symétrique appelée clé de chiffrement de la base de données (« clé DEK »). Au démarrage de la base de données, la clé DEK chiffrée est déchiffrée, puis elle est utilisée pour déchiffrer et rechiffrer les fichiers de la base de données dans le processus du moteur de base de données SQL Server. La clé de chiffrement DEK est protégée par le protecteur TDE. Le protecteur TDE est soit un certificat géré par le service (chiffrement transparent des données géré par le service), soit une clé asymétrique stockée dans [Azure Key Vault](../../key-vault/general/secure-your-key-vault.md) (chiffrement transparent des données géré par le client).

Pour Azure SQL Database et Azure Synapse, le protecteur TDE est défini au niveau du [serveur](logical-servers.md) logique SQL et est hérité par toutes les bases de données associées à ce serveur. Pour Azure SQL Managed Instance, le protecteur TDE est défini au niveau de l’instance et il est hérité par toutes les bases de données chiffrées sur cette instance. Le terme *serveur* fait référence à la fois au serveur et à l’instance tout au long de ce document, sauf indication contraire.

> [!IMPORTANT]
> Toutes les bases de données SQL Database nouvellement créées sont chiffrées par défaut à l’aide du chiffrement transparent des données géré par le service. Les bases de données SQL existantes créées avant mai 2017 et les bases de données SQL créées via restauration, géoréplication et copie de base de données, ne sont pas chiffrées par défaut. Les bases de données SQL Managed Instance existantes créées avant février 2019 ne sont pas chiffrées par défaut. Les bases de données SQL Managed Instance créées par le biais de la restauration héritent de l’état de chiffrement de la source.

> [!NOTE]
> Vous ne pouvez pas utiliser la technologie TDE pour chiffrer des bases de données système, telle la base de données **master** , dans Azure SQL Database et Azure SQL Managed Instance. La base de données **MASTER** contient les objets nécessaires à l’exécution des opérations TDE sur les bases de données utilisateur. Il est recommandé de ne pas stocker de données sensibles dans les bases de données système. Un [chiffrement d’infrastructure](transparent-data-encryption-byok-overview.md#doubleencryption) est en cours de déploiement, qui chiffre les bases de données système, y compris la base de données master. 

## <a name="service-managed-transparent-data-encryption"></a>Chiffrement transparent des données géré par le service

Dans Azure, la configuration par défaut de TDE spécifie que la clé de chiffrement de la base de données (DEK) est protégée par un certificat de serveur intégré. Le certificat de serveur intégré est unique pour chaque serveur et l’algorithme de chiffrement utilisé est AES 256. Si une base de données figure dans une relation de géoréplication, la base de données primaire et les base de données géo-secondaires sont protégées par la clé du serveur parent de la base de données primaire. Si deux bases de données sont connectées au même serveur, elles partagent également le même certificat intégré. Microsoft fait alterner automatiquement ces certificats conformément à la stratégie de sécurité interne et la clé racine est protégée par un magasin des secrets interne Microsoft. Les clients peuvent vérifier la conformité de SQL Database et SQL Managed Instance avec des stratégies de sécurité internes, dans des rapports d’audit tiers indépendants disponibles dans le [Centre de gestion de la confidentialité Microsoft](https://servicetrust.microsoft.com/).

En outre, Microsoft déplace et gère en toute transparence les clés en fonction des besoins en matière de géoréplication et de restaurations.

## <a name="customer-managed-transparent-data-encryption---bring-your-own-key"></a>Chiffrement transparent des données géré par le client - Bring Your Own Key

Le chiffrement TDE géré par le client est également appelé prise en charge de Bring Your Own Key (BYOK) pour TDE. Dans ce scénario, le protecteur TDE qui chiffre la clé DEK est une clé asymétrique managée par le client, stockée dans une solution Azure Key Vault du client, qui la gère (système d'administration de clés externe d’Azure) et il ne quitte jamais le coffre de clés. Le protecteur TDE peut être [généré par le coffre de clés ou transféré vers le coffre de clés](../../key-vault/keys/hsm-protected-keys.md) à partir d’un appareil HSM local d’un module de sécurité matériel. SQL Database, SQL Managed Instance et Azure Synapse doivent être autorisés à déchiffrer et chiffrer la DEK dans le coffre de clés appartenant au client. Si des autorisations d’accès du serveur au coffre de clés sont supprimées, une base de données devient inaccessible alors que toutes les données sont chiffrées

Avec l’intégration de TDE à Azure Key Vault, les utilisateurs peuvent contrôler les tâches de gestion de clés, y compris les rotations de clés, les autorisations de coffre de clés, les sauvegardes de clés, ainsi que l’audit et le rapport sur tous les protecteurs TDE à l’aide de la fonctionnalité Azure Key Vault. Key Vault centralise la gestion des clés, utilise des modules de sécurité matériels étroitement surveillés (HSM) et permet la séparation des responsabilités entre la gestion de clés et des données pour aider à répondre aux exigences des stratégies de sécurité.
Pour en savoir plus sur BYOK pour Azure SQL Database et Azure Synapse, consultez [Chiffrement transparent des données avec l’intégration d’Azure Key Vault](transparent-data-encryption-byok-overview.md).

Pour commencer à utiliser TDE avec l’intégration d’Azure Key Vault, consultez le guide pratique [Activer le chiffrement transparent des données avec votre propre clé Key Vault](transparent-data-encryption-byok-configure.md).

## <a name="move-a-transparent-data-encryption-protected-database"></a>Déplacer une base de données protégée par le chiffrement transparent des données

Vous n’avez pas besoin de déchiffrer les bases de données pour leur appliquer des opérations dans Azure. La base de données cible hérite de façon transparente des paramètres de TDE sur la base de données source ou sur la base de données principale. Les opérations incluses sont les suivantes :

- La géorestauration
- Restauration à un moment donné en libre-service
- Restauration d’une base de données supprimée
- La géoréplication active
- Création d’une copie de base de données
- Restauration du fichier de sauvegarde vers Azure SQL Managed Instance

> [!IMPORTANT]
> La sauvegarde manuelle COPY-ONLY d’une base de données chiffrée par un TDE géré par le service n’est pas prise en charge dans Azure SQL Managed Instance, car le certificat utilisé pour le chiffrement n’est pas accessible. Utilisez la fonctionnalité de restauration dans le temps pour déplacer ce type de base de données vers un autre service SQL Managed Instance ou basculer vers une clé gérée par le client.

Quand vous exportez une base de données protégée par TDE, le contenu exporté de la base de données n’est pas chiffré. Ce contenu exporté est stocké dans des fichiers BACPAC non chiffrés. Vous devez donc protéger les fichiers BACPAC de façon appropriée et activer TDE après avoir terminé l’importation de la nouvelle base de données.

Par exemple, si le fichier BACPAC est exporté à partir d’une instance SQL Server, le contenu importé de la nouvelle base de données n’est pas chiffré automatiquement. De même, si le fichier BACPAC est exporté vers une instance SQL Server, la nouvelle base de données n’est pas chiffrée automatiquement.

La seule exception survient lorsque vous procédez à une exportation de base de données à destination ou en provenance de SQL Database. TDE est activé dans la nouvelle base de données, mais le fichier BACPAC proprement dit n’est toujours pas chiffré.

## <a name="manage-transparent-data-encryption"></a>Gérer Transparent Data Encryption

# <a name="the-azure-portal"></a>[Le portail Azure](#tab/azure-portal)

Gérer TDE dans le portail Azure.

Pour configurer TDE par le biais du portail Azure, vous devez être connecté en tant que Propriétaire, Collaborateur ou Gestionnaire de sécurité SQL Azure.

Activez et désactivez TDE au niveau de la base de données. Pour Azure SQL Managed Instance, utilisez Transact-SQL (T-SQL) pour activer et désactiver TDE sur une base de données. Pour Azure SQL Database et Azure Synapse, vous pouvez gérer TDE pour la base de données dans le [portail Azure](https://portal.azure.com) après vous être connecté avec le compte administrateur ou contributeur Azure. Accédez aux paramètres TDE pour votre base de données utilisateur. Le chiffrement transparent des données géré par le service est utilisé par défaut. Un certificat TDE est automatiquement généré pour le serveur qui contient la base de données.

![Chiffrement transparent des données géré par le service](./media/transparent-data-encryption-tde-overview/service-managed-transparent-data-encryption.png)  

Vous définissez la clé principale TDE, également appelée protecteur TDE, au niveau du serveur ou de l’instance. Pour utiliser TDE avec BYOK et protéger vos bases de données à l’aide d’une clé fournie par Key Vault, ouvrez les paramètres TDE définis pour votre serveur.

![Chiffrement transparent des données avec prise en charge de Bring Your Own Key](./media/transparent-data-encryption-tde-overview/tde-byok-support.png)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Gérer TDE à l’aide de PowerShell.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]
> [!IMPORTANT]
> Le module PowerShell Azure Resource Manager est toujours pris en charge, mais tous les développements à venir sont destinés au module Az.Sql. Pour ces cmdlets, voir [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Les arguments des commandes dans le module Az sont sensiblement identiques à ceux des modules AzureRm.

Pour configurer TDE par le biais de PowerShell, vous devez être connecté en tant que Propriétaire, Collaborateur ou Gestionnaire de sécurité SQL Azure.

### <a name="cmdlets-for-azure-sql-database-and-azure-synapse"></a>Cmdlets relatives à Azure SQL Database et Azure Synapse

Utilisez les cmdlets suivantes pour Azure SQL Database et Azure Synapse :

| Applet de commande | Description |
| --- | --- |
| [Set-AzSqlDatabaseTransparentDataEncryption](/powershell/module/az.sql/set-azsqldatabasetransparentdataencryption) |Active ou désactive le chiffrement transparent des données pour une base de données.|
| [Get-AzSqlDatabaseTransparentDataEncryption](/powershell/module/az.sql/get-azsqldatabasetransparentdataencryption) |Obtient l’état du chiffrement transparent des données pour une base de données. |
| [Get-AzSqlDatabaseTransparentDataEncryptionActivity](/powershell/module/az.sql/get-azsqldatabasetransparentdataencryptionactivity) |Vérifie la progression du chiffrement pour une base de données. |
| [Add-AzSqlServerKeyVaultKey](/powershell/module/az.sql/add-azsqlserverkeyvaultkey) |Ajoute une clé Key Vault à un serveur. |
| [Get-AzSqlServerKeyVaultKey](/powershell/module/az.sql/get-azsqlserverkeyvaultkey) |Obtient les clés Key Vault pour un serveur  |
| [Set-AzSqlServerTransparentDataEncryptionProtector](/powershell/module/az.sql/set-azsqlservertransparentdataencryptionprotector) |Définit le protecteur du chiffrement transparent des données pour un serveur. |
| [Get-AzSqlServerTransparentDataEncryptionProtector](/powershell/module/az.sql/get-azsqlservertransparentdataencryptionprotector) |Obtient le protecteur du chiffrement transparent des données |
| [Remove-AzSqlServerKeyVaultKey](/powershell/module/az.sql/remove-azsqlserverkeyvaultkey) |Supprime une clé Key Vault sur un serveur. |
|  | |

> [!IMPORTANT]
> Pour Azure SQL Managed Instance, utilisez la commande T-SQL [ALTER DATABASE](/sql/t-sql/statements/alter-database-azure-sql-database) pour activer et désactiver TDE au niveau de la base de données, et [l’exemple de script PowerShell](transparent-data-encryption-byok-configure.md) pour gérer TDE au niveau de l’instance.

# <a name="transact-sql"></a>[Transact-SQL](#tab/azure-TransactSQL)

Gérer TDE à l’aide de Transact-SQL.

Connectez-vous à la base de données à l’aide d’un ID de connexion d’administrateur ou de membre du rôle **dbmanager** dans la base de données MASTER.

| Commande | Description |
| --- | --- |
| [ALTER DATABASE (Azure SQL Database)](/sql/t-sql/statements/alter-database-azure-sql-database) | SET ENCRYPTION ON/OFF chiffre ou déchiffre une base de données |
| [sys.dm_database_encryption_keys](/sql/relational-databases/system-dynamic-management-views/sys-dm-database-encryption-keys-transact-sql) |Renvoie des informations sur l’état de chiffrement d’une base de données et sur ses clés de chiffrement de base de données associées |
| [sys.dm_pdw_nodes_database_encryption_keys](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-nodes-database-encryption-keys-transact-sql) |Renvoie des informations sur l’état de chiffrement de chaque nœud Azure Synapse et de ses clés de chiffrement de base de données associées |
|  | |

Vous ne pouvez pas basculer le protecteur TDE vers une clé provenant de Key Vault à l’aide de Transact-SQL. Utilisez PowerShell ou le Portail Azure.

# <a name="rest-api"></a>[REST API](#tab/azure-RESTAPI)

Gérer TDE à l’aide de l’API REST.

Pour configurer TDE par le biais de l’API REST, vous devez être connecté en tant que Propriétaire, Collaborateur ou Gestionnaire de sécurité SQL Azure.
Utilisez l'ensemble de commandes suivant pour Azure SQL Database et Azure Synapse :

| Commande | Description |
| --- | --- |
|[Créer ou mettre à jour un serveur](/rest/api/sql/servers/createorupdate)|Ajoute une identité Azure Active Directory à un serveur. (pour l'octroi de l'accès au coffre de clés)|
|[Créer ou mettre à jour une clé de serveur](/rest/api/sql/serverkeys/createorupdate)|Ajoute une clé Key Vault à un serveur.|
|[Supprimer une clé de serveur](/rest/api/sql/serverkeys/delete)|Supprime une clé Key Vault sur un serveur. |
|[Obtenir des clés de serveur](/rest/api/sql/serverkeys/get)|Obtient une clé Key Vault spécifique d’un serveur.|
|[Répertorier les clés de serveur par serveur](/rest/api/sql/serverkeys/listbyserver)|Obtient les clés Key Vault pour un serveur. |
|[Créer ou mettre à jour le protecteur du chiffrement](/rest/api/sql/encryptionprotectors/createorupdate)|Définit le protecteur TDE pour un serveur.|
|[Obtenir le protecteur du chiffrement](/rest/api/sql/encryptionprotectors/get)|Obtient le protecteur TDE pour un serveur.|
|[Répertorier les protecteurs du chiffrement par serveur](/rest/api/sql/encryptionprotectors/listbyserver)|Obtient les protecteurs TDE pour un serveur. |
|[Créer ou mettre à jour la configuration du chiffrement transparent des données](/rest/api/sql/transparentdataencryptions/createorupdate)|Active ou désactive TDE pour une base de données.|
|[Obtenir la configuration du chiffrement transparent des données](/rest/api/sql/transparentdataencryptions/get)|Obtient la configuration de TDE pour une base de données.|
|[Répertorier les résultats de configuration du chiffrement transparent des données](/rest/api/sql/transparentdataencryptionactivities/listbyconfiguration)|Obtient le résultat du chiffrement d’une base de données.|

## <a name="see-also"></a>Voir aussi

- SQL Server exécuté sur une machine virtuelle Azure peut également utiliser une clé asymétrique provenant de Key Vault. La procédure de configuration diffère de celle qui s’applique en cas d’utilisation d’une clé asymétrique dans SQL Database et SQL Managed Instance. Pour plus d’informations, consultez l’article [Gestion de clés extensible à l’aide d’Azure Key Vault (SQL Server)](/sql/relational-databases/security/encryption/extensible-key-management-using-azure-key-vault-sql-server).
- Pour obtenir une description générale du chiffrement transparent des données, consultez la page [Transparent Data Encryption](/sql/relational-databases/security/encryption/transparent-data-encryption).
- Pour plus d'informations sur TDE avec prise en charge de BYOK pour Azure SQL Database, Azure SQL Managed Instance et Azure Synapse, consultez [Chiffrement transparent des données avec prise en charge du service Bring Your Own Key](transparent-data-encryption-byok-overview.md).
- Pour commencer à utiliser TDE avec prise en charge de Bring Your Own Key, consultez le guide pratique [Activer le chiffrement transparent des données avec votre propre clé Key Vault](transparent-data-encryption-byok-configure.md).
- Pour plus d’informations sur Key Vault, consultez [Accès sécurisé à un coffre de clés](../../key-vault/general/secure-your-key-vault.md).