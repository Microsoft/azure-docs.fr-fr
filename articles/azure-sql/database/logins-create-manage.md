---
title: Autoriser l’accès aux serveurs et bases de données à l’aide de connexions et de comptes d’utilisateur
titleSuffix: Azure SQL Database & SQL Managed Instance & Azure Synapse Analytics
description: Découvrez comment Azure SQL Database, SQL Managed Instance et Azure Synapse authentifient les utilisateurs à des fins d'accès à l'aide de connexions et de comptes d'utilisateur. Apprenez également à attribuer des rôles de base de données et des autorisations explicites pour autoriser les connexions, et permettre aux utilisateurs d'effectuer des actions et d'interroger des données.
keywords: sécurité sql database, gestion de la sécurité de base de données, sécurité de connexion,sécurité de base de données, accès aux bases de données
services: sql-database
ms.service: sql-db-mi
ms.subservice: security
ms.custom: sqldbrb=3
ms.devlang: ''
ms.topic: conceptual
author: VanMSFT
ms.author: vanto
ms.reviewer: sstein
ms.date: 03/23/2020
ms.openlocfilehash: d03bce1566d4f56a576c980723571f587296236f
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96452429"
---
# <a name="authorize-database-access-to-sql-database-sql-managed-instance-and-azure-synapse-analytics"></a>Configurer SQL Database, SQL Managed Instance et Azure Synapse Analytics pour autoriser l'accès aux bases de données
[!INCLUDE[appliesto-sqldb-sqlmi-asa](../includes/appliesto-sqldb-sqlmi-asa.md)]

Cet article porte sur les points suivants :

- Options de configuration d'Azure SQL Database, d'Azure SQL Managed Instance et d'Azure Synapse Analytics pour permettre aux utilisateurs d'effectuer des tâches d'administration et d'accéder aux données stockées dans ces bases de données
- Configuration de l'accès et des autorisations après la création initiale d'un serveur
- Ajout de connexions et de comptes d'utilisateur dans la base de données MASTER, puis attribution des autorisations d'administration aux comptes
- Ajout de comptes d'utilisateur dans des bases de données utilisateur, associés à des connexions ou en tant que comptes d'utilisateur autonomes
- Configuration de comptes d'utilisateur avec des autorisations dans des bases de données utilisateur à l'aide de rôles de base de données et d'autorisations explicites

> [!IMPORTANT]
> Dans cet article, les bases de données d'Azure SQL Database, d'Azure SQL Managed Instance et d'Azure Synapse sont collectivement désignées sous le nom de « bases de données », et le terme « serveur » fait référence au [serveur](logical-servers.md) qui gère les bases de données d'Azure SQL Database et d'Azure Synapse.

## <a name="authentication-and-authorization"></a>Authentification et autorisation

L’[**authentification**](security-overview.md#authentication) est le processus consistant à prouver que l’utilisateur est bien celui qu’il prétend être. Un utilisateur se connecte à une base de données à l’aide d’un compte d’utilisateur.
Lorsqu’un utilisateur tente de se connecter à une base de données, il fournit un compte d’utilisateur et des informations d’authentification. L’utilisateur est authentifié à l’aide de l’une des deux méthodes d’authentification suivantes :

- [Authentification SQL](/sql/relational-databases/security/choose-an-authentication-mode#connecting-through-sql-server-authentication).

  Avec cette méthode d’authentification, l’utilisateur envoie un nom de compte d’utilisateur et un mot de passe associé pour établir une connexion. Ce mot de passe est stocké dans la base de données master pour les comptes d’utilisateur liés à une connexion ou stockés dans la base de données contenant les comptes d’utilisateur *non* liés à une connexion.
- [Authentification Azure Active Directory](authentication-aad-overview.md)

  Avec cette méthode d'authentification, l'utilisateur soumet un nom de compte d'utilisateur et demande que le service utilise les informations d'identification stockées dans Azure Active Directory (Azure AD).

**Connexions et utilisateurs** : dans une base de données, un compte d'utilisateur peut être associé à une connexion stockée dans la base de données MASTER, ou il peut s'agir d'un nom d'utilisateur stocké dans une base de données individuelle.

- Une **connexion** est un compte individuel dans la base de données master, auquel un compte d’utilisateur dans une ou plusieurs bases de données peut être lié. Avec une connexion, les informations d’identification du compte d’utilisateur sont stockées avec la connexion.
- Un **compte d'utilisateur** est un compte individuel qui figure dans une base de données et qui peut être lié à une connexion, mais cela n'est pas obligatoire. Avec un compte d’utilisateur qui n’est pas lié à une connexion, les informations d’identification sont stockées avec le compte d’utilisateur.

L’[**autorisation**](security-overview.md#authorization) d’accéder aux données et d’effectuer diverses actions sont gérées à l’aide des rôles de base de données et des autorisations explicites. L’autorisation fait référence aux autorisations accordées à un utilisateur et détermine ce que l’utilisateur est autorisé à faire. Elle est contrôlée par les [appartenances aux rôles](/sql/relational-databases/security/authentication-access/database-level-roles) et les [autorisations au niveau objet](/sql/relational-databases/security/permissions-database-engine) de la base de données de votre compte d’utilisateur. Nous vous recommandons, à titre de meilleure pratique, d’accorder aux utilisateurs des privilèges aussi réduits que possible.

## <a name="existing-logins-and-user-accounts-after-creating-a-new-database"></a>Connexions et comptes d’utilisateur existants après la création d’une base de données

Lorsque vous déployez Azure SQL pour la première fois, vous spécifiez une connexion d'administrateur et un mot de passe associé à cette connexion. Ce compte d’administration est appelé **administration de serveur**. La configuration suivante des connexions et des utilisateurs dans les bases de données master et utilisateur se produit pendant le déploiement :

- Une connexion SQL avec des privilèges d’administrateur est créée à l’aide du nom de connexion que vous avez spécifié. Une [connexion](/sql/relational-databases/security/authentication-access/principals-database-engine#sa-login) est un compte d'utilisateur individuel qui permet de se connecter à SQL Database, SQL Managed Instance et Azure Synapse.
- Cette connexion se voit accorder des autorisations d’administration complètes sur toutes les bases de données en tant que [principal au niveau du serveur](/sql/relational-databases/security/authentication-access/principals-database-engine). La connexion dispose de toutes les autorisations disponibles et ne peut pas être limitée. Dans SQL Managed Instance, cette connexion est ajoutée au [rôle serveur fixe sysadmin](/sql/relational-databases/security/authentication-access/server-level-roles) (ce rôle n'existe pas dans Azure SQL Database).
- Un [compte d’utilisateur](/sql/relational-databases/security/authentication-access/getting-started-with-database-engine-permissions#database-users) appelé `dbo` est créé pour cette connexion dans chaque base de données utilisateur. L’utilisateur [dbo](/sql/relational-databases/security/authentication-access/principals-database-engine) dispose de toutes les autorisations de base de données dans la base de données et est mappé au rôle de base de données fixe `db_owner`. Des rôles de bases de données fixes supplémentaires sont décrits plus loin dans cet article.

Pour identifier les comptes d’administrateur d’une base de données, ouvrez le Portail Azure, puis accédez à l’onglet **Propriétés** de votre serveur ou instance gérée.

![Administrateurs SQL Server](./media/logins-create-manage/sql-admins.png)

![Capture d’écran mettant en évidence l’option de menu Propriétés.](./media/logins-create-manage/sql-admins2.png)

> [!IMPORTANT]
> Une fois créé, le nom de connexion de l'administrateur n'est pas modifiable. Pour réinitialiser le mot de passe pour l’administrateur du serveur, accédez au [portail Azure](https://portal.azure.com), cliquez sur **Serveurs SQL**, sélectionnez le serveur dans la liste, puis cliquez sur **Réinitialiser le mot de passe**. Pour réinitialiser le mot de passe de SQL Managed Instance, accédez au portail Azure, cliquez sur l'instance, puis cliquez sur **Réinitialiser le mot de passe**. Vous pouvez également utiliser PowerShell ou Azure CLI.

## <a name="create-additional-logins-and-users-having-administrative-permissions"></a>Créer des connexions et des utilisateurs supplémentaires disposant d’autorisations administratives

À ce stade, votre serveur ou instance gérée est uniquement configuré pour l'accès à l'aide d'une connexion SQL unique et d'un compte d'utilisateur. Pour créer des connexions supplémentaires avec des autorisations d’administration complètes ou partielles, vous disposez des options suivantes (en fonction de votre mode de déploiement) :

- **Créer un compte d'administrateur Azure Active Directory disposant d'autorisations d'administration complètes**

  Activez l’authentification Azure Active Directory et créez une connexion d’administrateur Azure AD. Un compte Azure Active Directory peut être configuré en tant qu'administrateur du déploiement Azure SQL avec des autorisations d'administration complètes. Il peut s’agir d’un compte de groupe de sécurité ou individuel. Un administrateur Azure AD **doit** être configuré si vous souhaitez utiliser des comptes Azure AD pour vous connecter à SQL Database, SQL Managed Instance ou Azure Synapse. Pour plus d'informations sur l'activation de l'authentification Azure AD pour tous les types de déploiements Azure SQL, consultez les articles suivants :

  - [Utiliser l'authentification Azure Active Directory pour l'authentification avec SQL](authentication-aad-overview.md)
  - [Configurer et gérer l’authentification Azure Active Directory avec SQL](authentication-aad-configure.md)

- **Dans SQL Managed Instance, créer des connexions SQL avec des autorisations d'administration complètes**

  - Créez une connexion SQL supplémentaire dans la base de données MASTER.
  - Ajoutez la connexion au [rôle serveur fixe sysadmin](/sql/relational-databases/security/authentication-access/server-level-roles) à l’aide de l’instruction [ALTER SERVER ROLE](/sql/t-sql/statements/alter-server-role-transact-sql). Cette connexion a des autorisations d’administration complètes.
  - Vous pouvez également créer une [connexion Azure AD](authentication-aad-configure.md#provision-azure-ad-admin-sql-managed-instance) à l’aide de la syntaxe [CREATE LOGIN](/sql/t-sql/statements/create-login-transact-sql?view=azuresqldb-mi-current).

- **Dans SQL Database, créer des connexions SQL avec des autorisations d'administration limitées**

  - Créez une connexion SQL supplémentaire dans la base de données MASTER.
  - Créez un compte d'utilisateur dans la base de données MASTER associée à cette nouvelle connexion.
  - Dans la base de données `master`, ajoutez le compte d’utilisateur au rôle `dbmanager`, au rôle `loginmanager` ou aux deux en utilisant l’instruction [ALTER ROLE](/sql/t-sql/statements/alter-role-transact-sql) (pour Azure Synapse, utilisez l’instruction [sp_addrolemember](/sql/relational-databases/system-stored-procedures/sp-addrolemember-transact-sql)).

  > [!NOTE]
  > Les rôles `dbmanager` et `loginmanager` ne se rapportent **pas** aux déploiements SQL Managed Instance.

  Les membres de ces [rôles spéciaux de la base de données MASTER](/sql/relational-databases/security/authentication-access/database-level-roles#special-roles-for--and-) d'Azure SQL Database peuvent créer et gérer des bases de données ou des connexions. Dans les bases de données créées par un utilisateur membre du rôle `dbmanager`, le membre est mappé au rôle de base de données fixe `db_owner` et peut se connecter à cette base de données et gérer celle-ci à l’aide du compte d’utilisateur `dbo`. Ces rôles n’ont pas d’autorisations explicites en dehors de la base de données master.

  > [!IMPORTANT]
  > Vous ne pouvez pas créer de connexion SQL supplémentaire avec des autorisations d'administration complètes dans SQL Database.

## <a name="create-accounts-for-non-administrator-users"></a>Créer des comptes pour les utilisateurs non-administrateurs

Vous pouvez créer des comptes pour les utilisateurs non-administrateurs à l’aide de l’une des deux méthodes suivantes :

- **Créer une connexion**

  Créez une connexion SQL dans la base de données master. Créez ensuite un compte d’utilisateur dans chaque base de données à laquelle cet utilisateur a besoin d’accéder et associez le compte d’utilisateur à cette connexion. Cette approche est préférable lorsque l’utilisateur doit accéder à plusieurs bases de données et que vous souhaitez synchroniser les mots de passe. Toutefois, cette approche présente des complexités lorsqu’elle est utilisée avec la géoréplication, car la connexion doit être créée sur le serveur principal et sur le ou les serveurs secondaires. Pour plus d’informations, consultez [Configurer et gérer la sécurité Azure SQL Database pour la géorestauration ou le basculement](active-geo-replication-security-configure.md).
- **Créer un compte d’utilisateur**

  Créez un compte d’utilisateur dans la base de données à laquelle un utilisateur a besoin d’accéder (également appelé [utilisateur autonome](/sql/relational-databases/security/contained-database-users-making-your-database-portable)).

  - Avec SQL Database, vous pouvez toujours créer ce type de compte d'utilisateur.
  - Avec l'instance gérée SQL Managed Instance qui prend en charge les [principaux de serveur Azure AD](authentication-aad-configure.md#create-contained-users-mapped-to-azure-ad-identities), vous pouvez créer des comptes d'utilisateur pour l'authentification auprès de SQL Managed Instance sans avoir à créer les utilisateurs de la base de données en tant qu'utilisateurs autonomes.

  Avec cette approche, les informations d’authentification utilisateur sont stockées dans chaque base de données et répliquées automatiquement dans les bases de données géorépliquées. Toutefois, si le même compte existe dans plusieurs bases de données et que vous utilisez l'authentification Azure SQL, vous devez synchroniser manuellement les mots de passe. En outre, si un utilisateur a un compte dans des bases de données différentes avec différents mots de passe, la mémorisation de ces mots de passe peut devenir un problème.

> [!IMPORTANT]
> Pour créer des utilisateurs autonomes mappés à des identités Azure AD, vous devez être connecté à l'aide d'un compte Azure AD qui dispose du rôle d'administrateur dans la base de données d'Azure SQL Database. Dans SQL Managed Instance, une connexion SQL avec des autorisations `sysadmin` peut également créer une connexion ou un utilisateur Azure AD.

Pour obtenir des exemples montrant comment créer des connexions et des utilisateurs, consultez :

- [Créer une connexion pour Azure SQL Database](/sql/t-sql/statements/create-login-transact-sql?view=azuresqldb-current#examples-1)
- [Créer une connexion pour Azure SQL Managed Instance](/sql/t-sql/statements/create-login-transact-sql?view=azuresqldb-mi-current#examples-2)
- [Créer une connexion pour Azure Synapse](/sql/t-sql/statements/create-login-transact-sql?view=azure-sqldw-latest#examples-3)
- [Créer un utilisateur](/sql/t-sql/statements/create-user-transact-sql#examples)
- [Création d’utilisateurs autonomes Azure AD](authentication-aad-configure.md#create-contained-users-mapped-to-azure-ad-identities)

> [!TIP]
> Pour accéder à un tutoriel sur la sécurité couvrant la création d'utilisateurs dans Azure SQL Database, reportez-vous à [Tutoriel : Sécuriser Azure SQL Database](secure-database-tutorial.md).

## <a name="using-fixed-and-custom-database-roles"></a>Utilisation des rôles de base de données fixes et personnalisés

Après avoir créé un compte d’utilisateur dans une base de données, sur la base d’une connexion ou d’un utilisateur autonome, vous pouvez autoriser cet utilisateur à effectuer diverses actions et à accéder aux données d’une base de données particulière. Vous pouvez utiliser les méthodes suivantes pour autoriser l’accès :

- **Rôles de base de données fixes**

  Ajoutez le compte d’utilisateur à un [rôle de base de données fixe](/sql/relational-databases/security/authentication-access/database-level-roles). Il existe 9 rôles de base de données fixes, chacun avec un ensemble défini d’autorisations. Les rôles de base de données fixes les plus courants sont les suivants : **db_owner**, **db_ddladmin**, **db_datawriter**, **db_datareader**, **db_denydatawriter** et **db_denydatareader**. **db_owner** est couramment utilisé pour accorder toutes les autorisations à quelques utilisateurs seulement. Les autres rôles de base de données fixe sont utiles pour obtenir rapidement une base de données simple en développement, mais ne sont pas recommandés pour la plupart des bases de données de production. Par exemple, le rôle de base de données fixe **db_datareader** accorde l’accès en lecture à toutes les tables de la base de données, ce qui est plus que le minimum nécessaire.

  - Pour ajouter un utilisateur de base de données à un rôle de base de données fixe :

    - Dans Azure SQL Database, utilisez l’instruction [ALTER ROLE](/sql/t-sql/statements/alter-role-transact-sql). Pour obtenir des exemples, consultez [Exemples ALTER ROLE](/sql/t-sql/statements/alter-role-transact-sql#examples).
    - Dans Azure Synapse, utilisez l'instruction [sp_addrolemember](/sql/relational-databases/system-stored-procedures/sp-addrolemember-transact-sql). Pour obtenir des exemples, consultez [Exemples sp_addrolemember](/sql/t-sql/statements/alter-role-transact-sql).

- **Rôle de base de données personnalisé**

  Créez un rôle de base de données personnalisé à l’aide de l’instruction [CREATE ROLE](/sql/t-sql/statements/create-role-transact-sql). Un rôle personnalisé vous permet de créer vos propres rôles de base de données définis par l’utilisateur et d’accorder soigneusement à chaque rôle les autorisations minimales nécessaires aux besoins de l’entreprise. Vous pouvez ensuite ajouter des utilisateurs au rôle personnalisé. Lorsqu’un utilisateur est membre de plusieurs rôles, toutes les autorisations sont agrégées.
- **Accorder des autorisations directement**

  Accordez directement des [autorisations](/sql/relational-databases/security/permissions-database-engine) au compte d’utilisateur. Il existe plus de 100 autorisations qui peuvent être accordées ou refusées individuellement dans la base de données SQL. La plupart de ces autorisations sont imbriquées. Par exemple, l’autorisation `UPDATE` sur un schéma inclut l’autorisation `UPDATE` sur chaque table dans ce schéma. Comme dans la plupart des systèmes d’autorisation, le refus d’une autorisation remplace l’octroi. En raison de la nature imbriquée et du nombre d’autorisations, la plus grande attention est requise pour concevoir un système d’autorisation approprié capable de protéger correctement votre base de données. Démarrez avec la liste des autorisations sous [Autorisations (moteur de base de données)](/sql/relational-databases/security/permissions-database-engine) et passez en revue le [graphique de taille affiche](/sql/relational-databases/security/media/database-engine-permissions.png) des autorisations.

## <a name="using-groups"></a>Utilisation de groupes

La gestion efficace de l’accès utilise des autorisations affectées à des groupes de sécurité Active Directory et des rôles fixes ou personnalisés plutôt qu’à des utilisateurs individuels.

- Lorsque vous utilisez l’authentification Azure Active Directory, placez les utilisateurs Azure Active Directory dans un groupe de sécurité Azure Active Directory. Créez un utilisateur de base de données autonome pour le groupe. Ajoutez un ou plusieurs utilisateurs de base de données en tant que membre de rôles de base de données personnalisés ou intégrés disposant des autorisations spécifiques appropriées pour ce groupe d’utilisateurs.

- Lorsque vous utilisez l’authentification SQL, créez des utilisateurs de base de données autonome dans la base de données. Placez un ou plusieurs utilisateurs de base de données dans un rôle de base de données personnalisé avec des autorisations spécifiques appropriées pour ce groupe d’utilisateurs.

  > [!NOTE]
  > Vous pouvez également utiliser des groupes pour les utilisateurs de bases de données non autonomes.

Vous devez vous familiariser avec les fonctionnalités suivantes qui peuvent être utilisées pour limiter ou élever les autorisations :

- Vous pouvez recourir à l’[emprunt d’identité](/dotnet/framework/data/adonet/sql/customizing-permissions-with-impersonation-in-sql-server) et à la [signature de module](/dotnet/framework/data/adonet/sql/signing-stored-procedures-in-sql-server) pour élever le niveau des autorisations temporairement, en toute sécurité.
- [sécurité au niveau des lignes](/sql/relational-databases/security/row-level-security) peut être utilisée pour limiter le nombres de lignes accessibles par l’utilisateur.
- [masquage des données](dynamic-data-masking-overview.md) permet de restreindre l’exposition des données sensibles.
- [procédures stockées](/sql/relational-databases/stored-procedures/stored-procedures-database-engine) , vous pouvez limiter le nombre d’actions susceptibles d’être exécutées sur la base de données.

## <a name="next-steps"></a>Étapes suivantes

Pour accéder à une vue d'ensemble de toutes les fonctionnalités de sécurité d'Azure SQL Database et de SQL Managed Instance, consultez [Vue d'ensemble de la sécurité](security-overview.md).