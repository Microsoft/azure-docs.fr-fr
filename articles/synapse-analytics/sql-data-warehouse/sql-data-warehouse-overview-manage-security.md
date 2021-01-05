---
title: Sécuriser un pool SQL dédié (anciennement SQL DW)
description: Conseils pour sécuriser un pool SQL dédié (anciennement SQL DW) et développer des solutions dans Azure Synapse Analytics.
author: julieMSFT
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 04/17/2018
ms.author: jrasnick
ms.reviewer: igorstan
ms.custom: seo-lt-2019
tags: azure-synapse
ms.openlocfilehash: ce09488e2323aada5f99494ef3920681b685ec0b
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96453643"
---
# <a name="secure-a-dedicated-sql-pool-formerly-sql-dw-in-azure-synapse-analytics"></a>Sécuriser un pool SQL dédié (anciennement SQL DW) dans Azure Synapse Analytics

> [!div class="op_single_selector"]
>
> * [Présentation de la sécurité](sql-data-warehouse-overview-manage-security.md)
> * [Authentification](sql-data-warehouse-authentication.md)
> * [Chiffrement (portail)](sql-data-warehouse-encryption-tde.md)
> * [Chiffrement (T-SQL)](sql-data-warehouse-encryption-tde-tsql.md)

Cet article présente les bases de la sécurisation de votre pool SQL dédié (anciennement SQL DW). Plus précisément, il offre un aperçu des ressources permettant de limiter l’accès, de protéger les données et de superviser les activités à l’aide d’un pool SQL dédié (anciennement SQL DW).

## <a name="connection-security"></a>Sécurité de la connexion

L’expression « sécurité de la connexion » fait référence au mode de restriction et de sécurisation appliqué aux connexions à votre base de données, au moyen de règles de pare-feu et d’une fonction de chiffrement des connexions.

Le [serveur SQL logique](../../azure-sql/database/logical-servers.md) et ses bases de données utilisent des règles de pare-feu pour rejeter les tentatives de connexion provenant d'adresses IP qui n'ont pas été explicitement approuvées. Pour autoriser les connexions à partir de l’adresse IP publique de l’ordinateur client ou de votre application, vous devez d’abord créer une règle de pare-feu au niveau du serveur à l’aide du portail Azure, de l’API REST ou de PowerShell.

Nous vous recommandons, à titre de meilleure pratique, de limiter autant que possible le nombre de plages d'adresses IP autorisées à traverser le pare-feu au niveau de votre serveur.  Pour accéder à votre pool SQL dédié (anciennement SQL DW) à partir de votre ordinateur local, vérifiez que le pare-feu présent sur votre réseau et sur l’ordinateur local autorise les communications sortantes sur le port TCP 1433.  

Le pool SQL dédié (anciennement SQL DW) utilise des règles de pare-feu IP au niveau du serveur. Il ne prend pas en charge les règles de pare-feu IP au niveau de la base de données. Pour plus d’information, voir [Règles de pare-feu Azure SQL Database](../../azure-sql/database/firewall-configure.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json).

Par défaut, les connexions à votre pool SQL dédié (anciennement SQL DW) sont chiffrées.  La modification des paramètres de connexion pour désactiver le chiffrement est ignorée.

## <a name="authentication"></a>Authentification

Le terme « authentification » fait référence au processus de validation de votre identité lorsque vous vous connectez à la base de données. Le pool SQL dédié (anciennement SQL DW) prend actuellement en charge l’authentification SQL Server avec un nom d’utilisateur et un mot de passe, et avec Azure Active Directory.

Lorsque vous avez créé le serveur de votre base de données, vous avez spécifié un compte de connexion « Admin serveur » associé à un nom d'utilisateur et à un mot de passe. À l’aide de ces informations d’identification, vous pouvez vous authentifier auprès de n’importe quelle base de données sur ce serveur, en tant que propriétaire de la base de données, ou « dbo » via l’authentification SQL Server.

Toutefois, en guise de meilleure pratique, il est recommandé que les utilisateurs de votre organisation utilisent un compte différent pour s’authentifier. Cela vous permet de limiter les autorisations accordées à cette application et de réduire les risques d’activité malveillante, au cas où le code de votre application serait vulnérable à une attaque par injection de code SQL.

Pour créer un utilisateur authentifié par SQL Server, connectez-vous à la base de données **master** sur votre serveur avec votre identifiant de connexion d’administrateur du serveur, et créez un nouvel identifiant de connexion au serveur.  Il est judicieux de créer également un utilisateur dans la base de données MASTER. La création d’un utilisateur dans la base de données master permet à un utilisateur de se connecter à l’aide d’outils tels que SSMS sans spécifier de nom de base de données.  Elle permet également d'utiliser l'Explorateur d'objets pour afficher toutes les bases de données d'un serveur.

```sql
-- Connect to master database and create a login
CREATE LOGIN ApplicationLogin WITH PASSWORD = 'Str0ng_password';
CREATE USER ApplicationUser FOR LOGIN ApplicationLogin;
```

Ensuite, connectez-vous à votre **pool SQL dédié (anciennement SQL DW)** avec vos informations de connexion d’administrateur de serveur, puis créez un utilisateur de base de données basé sur le compte de connexion au serveur que vous avez créé.

```sql
-- Connect to the database and create a database user
CREATE USER ApplicationUser FOR LOGIN ApplicationLogin;
```

Pour autoriser un utilisateur à effectuer d’autres opérations telles que la création de connexions ou de bases de données, affectez-le aux rôles `Loginmanager` et `dbmanager` dans la base de données master.

Pour en savoir plus sur ces rôles supplémentaires et sur l’authentification auprès d’une base de données SQL, consultez [Gestion des bases de données et connexions dans Azure SQL Database](../../azure-sql/database/logins-create-manage.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json).  Pour plus d’informations sur la connexion à l’aide d’Azure Active Directory, voir [Connexion à l’aide de l’authentification Azure Active Directory](sql-data-warehouse-authentication.md).

## <a name="authorization"></a>Autorisation

L’autorisation fait référence à ce que vous pouvez faire à l’intérieur d’une base de données une fois que vous êtes authentifié et connecté. Les privilèges d’autorisation sont déterminés par les autorisations et les appartenances aux rôles. Nous vous recommandons, à titre de meilleure pratique, d’accorder aux utilisateurs des privilèges aussi réduits que possible. Pour gérer les rôles, vous pouvez utiliser les procédures stockées suivantes :

```sql
EXEC sp_addrolemember 'db_datareader', 'ApplicationUser'; -- allows ApplicationUser to read data
EXEC sp_addrolemember 'db_datawriter', 'ApplicationUser'; -- allows ApplicationUser to write data
```

Le compte d’administrateur du serveur avec lequel vous vous connectez appartient au groupe « db_owner », qui est autorisé à effectuer tout type d’opérations dans la base de données. Enregistrez ce compte pour l’utiliser lors du déploiement des mises à niveau de schéma et d’autres opérations de gestion. Utilisez le compte « ApplicationUser », doté d’autorisations plus limitées, pour vous connecter à la base de données à partir de votre application, en bénéficiant du niveau de privilèges le moins élevé requis par cette dernière.

Il existe d’autres méthodes pour limiter le nombre d’actions que peut réaliser un utilisateur dans la base de données :

* Des [autorisations](/sql/relational-databases/security/permissions-database-engine?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) granulaires vous permettent de contrôler les opérations que vous pouvez exécuter sur des colonnes, des tables, des vues, des schémas, des procédures et d’autres objets individuels dans la base de données. Utilisez les autorisations granulaires pour avoir un contrôle optimal et accordez les autorisations minimales nécessaires.
* Les [rôles de base de données](/sql/relational-databases/security/authentication-access/database-level-roles?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) autres que « db_datareader » et « db_datawriter » peuvent être utilisés pour créer des comptes d’utilisateur plus puissants ou des comptes de gestion moins puissants pour votre application. Les rôles de base de données fixes intégrés offrent un moyen facile d'accorder des autorisations, mais peuvent entraîner l'octroi d'autorisations excessives.
* [procédures stockées](/sql/relational-databases/stored-procedures/stored-procedures-database-engine?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) , vous pouvez limiter le nombre d’actions susceptibles d’être exécutées sur la base de données.

L’exemple suivant accorde un accès en lecture à un schéma défini par l’utilisateur.

```sql
--CREATE SCHEMA Test
GRANT SELECT ON SCHEMA::Test to ApplicationUser
```

La gestion des bases de données et serveurs à partir du portail Azure et l'utilisation de l'API Azure Resource Manager sont contrôlées par les attributions de rôle de votre compte d'utilisateur sur le portail. Pour plus d’informations, consultez [Ajouter ou supprimer des attributions de rôles Azure à l’aide du portail Azure](../../role-based-access-control/role-assignments-portal.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json).

## <a name="encryption"></a>Chiffrement

La technologie Transparent Data Encryption (TDE) vous aide à vous protéger contre des activités malveillantes en chiffrant et déchiffrant vos données au repos. Lorsque vous chiffrez votre base de données, les fichiers de sauvegardes et les journaux de transactions associés sont chiffrés, sans que cela ne nécessite de modifications de vos applications. Le chiffrement transparent des données chiffre le stockage d’une base de données entière à l’aide d’une clé symétrique appelée clé de chiffrement de base de données.

Dans SQL Database, la clé de chiffrement de base de données est protégée par un certificat de serveur intégré. Le certificat de serveur intégré est propre à chaque serveur. Microsoft fait automatiquement alterner ces certificats au moins tous les 90 jours. L’algorithme de chiffrement utilisé est AES-256. Pour obtenir une description générale du chiffrement transparent des données, consultez la page [Chiffrement transparent des données (TDE)](/sql/relational-databases/security/encryption/transparent-data-encryption?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).

Vous pouvez chiffrer votre base de données à l’aide du [Portail Azure](sql-data-warehouse-encryption-tde.md) ou de [T-SQL](sql-data-warehouse-encryption-tde-tsql.md).

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir des informations et des exemples de connexion à votre entrepôt avec différents protocoles, consultez [Se connecter à un pool SQL dédié (anciennement SQL DW)](sql-data-warehouse-connect-overview.md).
