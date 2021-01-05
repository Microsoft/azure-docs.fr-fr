---
title: Authentification Azure Active Directory
description: Découvrez comment utiliser Azure Active Directory pour l’authentification auprès d’Azure SQL Database, Azure SQL Managed Instance et Synapse SQL dans Azure Synapse Analytics
services: sql-database
ms.service: sql-db-mi
ms.subservice: security
ms.custom: azure-synapse, sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: GithubMirek
ms.author: mireks
ms.reviewer: vanto, sstein
ms.date: 04/23/2020
ms.openlocfilehash: a636c0e2a41b636f30ada14d4f16a022f2890b71
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96454296"
---
# <a name="use-azure-active-directory-authentication"></a>Utiliser l’authentification Azure Active Directory

[!INCLUDE[appliesto-sqldb-sqlmi-asa](../includes/appliesto-sqldb-sqlmi-asa.md)]

L’authentification Azure Active Directory (Azure AD) est un mécanisme qui sert à se connecter à [Azure SQL Database](sql-database-paas-overview.md), [Azure SQL Managed Instance](../managed-instance/sql-managed-instance-paas-overview.md) et [Synapse SQL dans Azure Synapse Analytics](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md) à l’aide d’identités se trouvant dans Azure AD.

> [!NOTE]
> Cet article s’applique à Azure SQL Database, SQL Managed Instance et Azure Synapse Analytics.

Avec l’authentification Azure AD, vous pouvez gérer de manière centralisée les identités des utilisateurs de base de données et d’autres services Microsoft dans un emplacement centralisé. La gestion centralisée des ID fournit un emplacement unique pour gérer les utilisateurs de la base de données et simplifie la gestion des autorisations. Les avantages suivants sont inclus :

- Elle fournit une alternative à l’authentification SQL Server.
- Elle aide à bloquer la prolifération des identités utilisateur sur plusieurs serveurs.
- Elle permet une rotation du mot de passe dans un emplacement unique.
- Les clients peuvent gérer les autorisations de base de données à l’aide des groupes (Azure AD) externes.
- Il peut éliminer le stockage des mots de passe en activant l’authentification intégrée Windows et les autres formes d’authentification prises en charge par Azure Active Directory.
- L’authentification Azure AD utilise les utilisateurs de base de données autonome pour authentifier les identités au niveau de la base de données.
- Azure AD prend en charge l’authentification basée sur des jetons pour les applications se connectant à SQL Database et SQL Managed Instance.
- L’authentification Azure AD prend en charge :
  - Identités uniquement cloud Azure AD.
  - Identités hybrides Azure AD prenant en charge :
    - L’authentification dans le cloud avec deux options associées à l’authentification unique (SSO) fluide : authentification **directe** et authentification avec **hachage de mot de passe**.
    - Authentification fédérée.
  - Pour plus d’informations sur les méthodes d’authentification Azure AD et laquelle choisir, consultez l’article suivant :
    - [Choisir la méthode d’authentification adaptée à votre solution d’identité hybride Azure Active Directory](../../active-directory/hybrid/choose-ad-authn.md)

- Azure AD prend en charge les connexions à partir de SQL Server Management Studio qui utilisent l’authentification universelle Active Directory, laquelle inclut et notamment Multi-Factor Authentication. Multi-Factor Authentication inclut une authentification forte avec diverses options pratiques de vérification (appel téléphonique, SMS, cartes à puce avec code PIN ou notification d’application mobile). Pour plus d’informations, consultez [Prise en charge de SSMS pour Azure AD Multi-Factor Authentication avec Azure SQL Database, SQL Managed Instance et Azure Synapse](authentication-mfa-ssms-overview.md)

- Azure AD prend en charge des connexions similaires à partir de SQL Server Data Tools (SSDT) qui utilisent l’authentification interactive Active Directory. Pour plus d’informations, consultez [Prise en charge d’Azure Active Directory dans SQL Server Data Tools (SSDT)](/sql/ssdt/azure-active-directory).

> [!NOTE]  
> La connexion à une instance SQL Server s’exécutant sur une machine virtuelle Azure n’est pas prise en charge à l’aide d’un compte Azure Active Directory. Utilisez plutôt un compte Active Directory du domaine.  

Les étapes de configuration incluent les procédures suivantes pour configurer et utiliser l’authentification Azure Active Directory.

1. Créer et renseigner Azure AD.
2. Facultatif : associez ou modifiez l’annuaire Active Directory actuellement associé à votre abonnement Azure.
3. Créer un administrateur Azure Active Directory.
4. Configurer vos ordinateurs clients.
5. Créer des utilisateurs de base de données autonome dans votre base de données mappés sur les identités Azure AD.
6. Se connecter à la base de données à l’aide des identités Azure AD.

> [!NOTE]
> Pour savoir comment créer et à renseigner Azure AD, puis configurer Azure AD avec Azure SQL Database, SQL Managed Instance et SQL Database dans Azure Synapse Analytics, consultez [Configurer Azure AD avec Azure SQL Database](authentication-aad-configure.md).

## <a name="trust-architecture"></a>Architecture d’approbation

- Seuls la partie cloud et Azure AD, SQL Database, SQL Managed Instance et Azure Synapse sont considérés pour prendre en charge les mots de passe des utilisateurs natifs d’Azure AD.
- Pour prendre en charge les informations d’identification de l’authentification unique Windows (ou utilisateur/mot de passe pour les informations d’identification Windows), utilisez les informations d’identification Azure Active Directory à partir d’un domaine fédéré ou managé configuré pour une authentification unique transparente à des fins d’authentification directe et d’authentification par hachage du mot de passe. Pour plus d’informations, consultez la page [Authentification unique transparente Azure Active Directory](../../active-directory/hybrid/how-to-connect-sso.md).
- Pour prendre en charge l’authentification fédérée (ou utilisateur/mot de passe pour les informations d’identification Windows), la communication avec le bloc ADFS est requise.

Pour plus d’informations sur les identités hybrides Azure AD, la configuration et la synchronisation, consultez les articles suivants :

- Authentification par hachage du mot de passe : [Implémenter la synchronisation de hachage du mot de passe avec la synchronisation Azure AD Connect](../../active-directory/hybrid/how-to-connect-password-hash-synchronization.md)
- Authentification directe : [Authentification directe Azure Active Directory](../../active-directory/hybrid/how-to-connect-pta-quick-start.md)
- Authentification fédérée : [Déploiement des services de fédération Active Directory (AD FS) dans Azure](/windows-server/identity/ad-fs/deployment/how-to-connect-fed-azure-adfs) et [Fédération avec Azure AD Connect](../../active-directory/hybrid/how-to-connect-fed-whatis.md)

Pour obtenir un exemple d’authentification fédérée avec l’infrastructure ADFS (ou utilisateur/mot de passe pour les informations d’identification Windows), consultez le diagramme ci-dessous. Les flèches indiquent les voies de communication.

![diagramme autorisation aad][1]

Le diagramme suivant indique la fédération, l’approbation et les relations d’hébergement qui autorisent un client à se connecter à une base de données en soumettant un jeton. Le jeton est authentifié par une instance Azure AD, et approuvé par la base de données. Le client 1 peut représenter un répertoire Azure Active Directory avec des utilisateurs natifs ou un répertoire Azure AD avec des utilisateurs fédérés. Le client 2 représente une solution possible incluant des utilisateurs importés, qui dans cet exemple proviennent d’un répertoire Azure Active Directory fédéré avec la synchronisation d’ADFS avec Azure Active Directory. Il est important de comprendre que l’accès à une base de données à l’aide de l’authentification Azure AD exige que l’abonnement d’hébergement soit associé à Azure AD. Le même abonnement doit être utilisé pour créer les ressources Azure SQL Database, SQL Managed Instance ou Azure Synapse.

![relation abonnement][2]

## <a name="administrator-structure"></a>Structure de l’administrateur

En cas d’utilisation de l’authentification Azure AD, il existe deux comptes administrateur : administrateur Azure SQL Database d’origine et administrateur Azure AD. Les mêmes concepts s’appliquent à Azure Synapse. Seul l’administrateur basé sur un compte Azure AD peut créer le premier utilisateur de la base de données autonome Azure AD dans une base de données utilisateur. La connexion d’administrateur Azure AD peut être un utilisateur Azure AD ou un groupe Azure AD. Lorsque l’administrateur est un compte de groupe, il peut être utilisé par n’importe quel membre du groupe, activant plusieurs administrateurs Azure AD pour le serveur. L’utilisation d’un compte de groupe en tant qu’administrateur facilite la gestion en vous permettant d’ajouter et de supprimer des membres du groupe dans Azure AD sans modifier les utilisateurs ou les autorisations dans SQL Database ou Azure Synapse. Seul un administrateur Azure AD (utilisateur ou groupe) peut être configuré à tout moment.

![structure admin][3]

## <a name="permissions"></a>Autorisations

Pour créer de nouveaux utilisateurs, vous devez disposer de l’autorisation `ALTER ANY USER` dans la base de données. L’autorisation `ALTER ANY USER` peut être octroyée à un utilisateur de base de données. L’autorisation `ALTER ANY USER` est également détenue par les comptes d’administrateur de serveur et les utilisateurs de base de données avec les autorisations `CONTROL ON DATABASE` ou `ALTER ON DATABASE` pour cette base de données et par les membres du rôle de base de données `db_owner`.

Pour créer un utilisateur de base de données autonome dans Azure SQL Database, SQL Managed Instance ou Azure Synapse, vous devez vous connecter à la base de données ou à l’instance avec une identité Azure AD. Pour créer le premier utilisateur de la base de données autonome, vous devez vous connecter à la base de données à l’aide d’un administrateur Azure AD (le propriétaire de la base de données). Cette procédure est expliquée dans [Configurer et gérer l’authentification Azure Active Directory avec SQL Database ou Azure Synapse](authentication-aad-configure.md). L’authentification Azure AD est possible seulement si l’administrateur Azure AD a été créé pour Azure SQL Database, SQL Managed Instance ou Azure Synapse. Si l’administrateur Azure Active Directory a été supprimé du serveur, les utilisateurs Azure Active Directory existants créés précédemment dans le serveur SQL Server ne peuvent plus se connecter à la base de données à l’aide de leurs informations d’identification Azure Active Directory.

## <a name="azure-ad-features-and-limitations"></a>Limitations et fonctionnalités azure AD

- Les membres suivants d’Azure AD peuvent être provisionnés pour Azure SQL Database :

  - Membre natif : membre créé dans Azure AD dans le domaine géré ou le domaine client. Pour plus d’informations, consultez [Ajout de votre nom de domaine personnalisé à Azure AD](../../active-directory/fundamentals/add-custom-domain.md).
  - Membres d’un domaine Active Directory fédéré avec Azure Active Directory sur un domaine managé configuré pour l’authentification unique transparente avec authentification directe ou par hachage du mot de passe. Pour plus d’informations, consultez [Microsoft Azure prend désormais en charge la fédération avec Windows Server Active Directory](https://azure.microsoft.com/blog/windows-azure-now-supports-federation-with-windows-server-active-directory//) et [Authentification unique transparente Azure Active Directory](../../active-directory/hybrid/how-to-connect-sso.md).
  - Membres importés à partir d’autres répertoires Azure AD qui sont des membres natifs ou de domaine fédéré.
  - Groupes Active Directory créés en tant que groupes de sécurité.

- Les utilisateurs Azure AD qui font partie d’un groupe doté du rôle serveur `db_owner` ne peuvent pas utiliser la syntaxe **[CREATE DATABASE SCOPED CREDENTIAL](/sql/t-sql/statements/create-database-scoped-credential-transact-sql)** dans Azure SQL Database et Azure Synapse. Vous voyez l’erreur suivante :

    `SQL Error [2760] [S0001]: The specified schema name 'user@mydomain.com' either does not exist or you do not have permission to use it.`

    Accordez le rôle `db_owner` directement à l’utilisateur Azure AD individuel pour atténuer le problème lié à **CREATE DATABASE SCOPED CREDENTIAL**.

- Ces fonctions système retournent des valeurs NULL lors de leur exécution sous des entités Azure AD :

  - `SUSER_ID()`
  - `SUSER_NAME(<admin ID>)`
  - `SUSER_SNAME(<admin SID>)`
  - `SUSER_ID(<admin name>)`
  - `SUSER_SID(<admin name>)`

### <a name="sql-managed-instance"></a>Instance managée SQL

- Les principaux de serveur (connexions) et les utilisateurs Azure AD sont pris en charge pour [SQL Managed Instance](../managed-instance/sql-managed-instance-paas-overview.md).
- La définition de principaux de serveur (connexions) Azure AD mappés sur un groupe Azure AD en tant que propriétaire de la base de données n’est pas prise en charge dans [SQL Managed Instance](../managed-instance/sql-managed-instance-paas-overview.md).
  - Quand vous ajoutez un groupe dans le cadre du rôle serveur `dbcreator`, les utilisateurs de ce groupe peuvent donc se connecter à SQL Managed Instance et créer des bases de données, mais ils ne peuvent pas accéder à la base de données. En effet, le propriétaire de la nouvelle base de données est SA et non l’utilisateur Azure AD. Ce problème ne se manifeste pas si l’utilisateur individuel est ajouté au rôle serveur `dbcreator`.
- La gestion et l’exécution de travaux SQL Agent sont prises en charge pour les principaux de serveur (connexions) Azure AD.
- Les opérations de sauvegarde et de restauration de base de données peuvent être exécutées par les principaux de serveur (connexions) Azure AD.
- L’audit de toutes les instructions liées aux principaux de serveur (connexions) Azure AD et aux événements d’authentification Azure AD est pris en charge.
- La connexion administrateur dédiée pour les principaux de serveur (connexions) Azure AD membres du rôle serveur sysadmin est prise en charge.
  - Prise en charge par le biais de l’utilitaire SQLCMD et de SQL Server Management Studio.
- Les déclencheurs d’ouverture de session sont pris en charge pour les événements d’ouverture de session provenant de principaux de serveur (connexions) Azure AD.
- Vous pouvez configurer Service Broker et Database Mail à l’aide d’un principal de serveur (connexion) Azure AD.

## <a name="connect-by-using-azure-ad-identities"></a>Se connecter à l’aide des identités Azure AD

L’authentification Azure Active Directory prend en charge les méthodes suivantes de connexion à une base de données à l’aide d’identités Azure AD :

- Mot de passe Azure Active Directory
- Intégration d’Azure Active Directory
- Authentification universelle Azure Active Directory avec Multi-Factor Authentication (MFA)
- À l’aide de l’authentification par jeton d’application

Les méthodes d’authentification suivantes sont prises en charge pour les principaux de serveur (connexions) Azure AD :

- Mot de passe Azure Active Directory
- Intégration d’Azure Active Directory
- Authentification universelle Azure Active Directory avec Multi-Factor Authentication (MFA)

### <a name="additional-considerations"></a>Considérations supplémentaires

- Pour améliorer la facilité de gestion, nous vous conseillons de mettre en service un groupe Azure AD dédié en tant qu’administrateur.
- Un seul administrateur Azure AD (utilisateur ou groupe) peut être configuré pour un serveur dans SQL Database ou Azure Synapse à tout moment.
  - L’ajout de principaux de serveur (connexions) Azure AD pour SQL Managed Instance permet de créer plusieurs principaux de serveur (connexions) Azure AD pouvant être ajoutés au rôle `sysadmin`.
- Seul un administrateur Azure AD pour le serveur peut se connecter initialement au serveur ou à l’instance managée avec un compte Azure Active Directory. L’administrateur Active Directory peut configurer les utilisateurs de base de données Azure AD suivants.
- Nous vous conseillons de définir l’expiration du délai de connexion à 30 secondes.
- SQL Server 2016 Management Studio et SQL Server Data Tools pour Visual Studio 2015 (version 14.0.60311.1 d’avril 2016 ou ultérieure) prennent en charge l’authentification Azure Active Directory. (L’authentification Azure AD est prise en charge par le **Fournisseur de données .NET Framework pour SQL Server** ; .NET Framework version 4.6 minimum). Par conséquent, les dernières versions de ces outils et applications de la couche Données (DAC et BACPAC) peuvent utiliser l’authentification Azure AD.
- À partir de la version 15.0.1, l’[utilitaire sqlcmd](/sql/tools/sqlcmd-utility) et l’[utilitaire bcp](/sql/tools/bcp-utility) prennent en charge l’authentification interactive Active Directory avec Multi-Factor Authentication (MFA).
- SQL Server Data Tools pour Visual Studio 2015 requiert la version d’avril 2016 (version 14.0.60311.1) ou une version ultérieure. Actuellement, les utilisateurs Azure AD ne sont pas affichés dans l’Explorateur d’objets SSDT. Comme solution de contournement, vous pouvez afficher les utilisateurs dans [sys.database_principals](/sql/relational-databases/system-catalog-views/sys-database-principals-transact-sql).
- Le [pilote Microsoft JDBC 6.0 pour SQL Server](https://www.microsoft.com/download/details.aspx?id=11774) prend en charge l’authentification Azure AD. Consultez également [Définition des propriétés de connexion](/sql/connect/jdbc/setting-the-connection-properties).
- PolyBase ne peut pas s’authentifier avec l’authentification Azure AD.
- L’authentification Azure AD est prise en charge pour Azure SQL Database et Azure Synapse en utilisant les panneaux **Importer la base de données** et **Exporter la base de données** du portail Azure. L’importation et l’exportation à l’aide de l’authentification Azure AD sont également prises en charge depuis une invite de commandes PowerShell.
- L’authentification Azure AD est prise en charge pour SQL Database, SQL Managed Instance et Azure Synapse par le biais de l’interface de ligne de commande (CLI). Pour plus d’informations, consultez [Configurer et gérer l’authentification Azure AD avec SQL Database ou Azure Synapse](authentication-aad-configure.md) et [SQL Server - az sql server](/cli/azure/sql/server).

## <a name="next-steps"></a>Étapes suivantes

- Pour apprendre à créer et à remplir une instance Azure AD, puis à la configurer avec Azure SQL Database, SQL Managed Instance ou Azure Synapse, consultez [Configurer et gérer l’authentification Azure Active Directory avec SQL Database, SQL Managed Instance ou Azure Synapse](authentication-aad-configure.md).
- Pour suivre un tutoriel sur l’utilisation de principaux de serveur (connexions) Azure AD avec SQL Managed Instance, consultez [Principaux de serveur (connexions) Azure AD avec SQL Managed Instance](../managed-instance/aad-security-configure-tutorial.md)
- Pour une vue d’ensemble des connexions, des utilisateurs, des rôles de base de données et des autorisations dans SQL Database, consultez [Connexions, utilisateurs, rôles de base de données et autorisations](logins-create-manage.md).
- Pour en savoir plus sur les principaux de base de données, voir [Principaux](/sql/relational-databases/security/authentication-access/principals-database-engine).
- Pour en savoir plus sur les rôles de base de données, voir [Rôles de base de données](/sql/relational-databases/security/authentication-access/database-level-roles).
- Pour obtenir la syntaxe permettant de créer des principaux de serveur (connexions) Azure AD pour SQL Managed Instance, consultez [CREATE LOGIN](/sql/t-sql/statements/create-login-transact-sql?view=azuresqldb-mi-current&preserve-view=true).
- Pour en savoir plus sur les règles de pare-feu dans la base de données SQL, voir [Règles de pare-feu de la base de données SQL](firewall-configure.md).

<!--Image references-->
[1]: ./media/authentication-aad-overview/1aad-auth-diagram.png
[2]: ./media/authentication-aad-overview/2subscription-relationship.png
[3]: ./media/authentication-aad-overview/3admin-structure.png
