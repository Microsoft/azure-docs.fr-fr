---
title: Sécurité SQL Managed Instance à l’aide de principaux de serveur (connexions) Azure AD
description: Découvrir les techniques et fonctionnalités permettant de sécuriser Azure SQL Managed Instance à l’aide de principaux de serveur (connexions) Azure AD
services: sql-database
ms.service: sql-managed-instance
ms.subservice: security
ms.custom: sqldbrb=1
ms.topic: tutorial
author: GitHubMirek
ms.author: mireks
ms.reviewer: vanto
ms.date: 11/06/2019
ms.openlocfilehash: 9161bf4f99ddfed479451d2091458ab309aa2c17
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92788619"
---
# <a name="tutorial-security-in-azure-sql-managed-instance-using-azure-ad-server-principals-logins"></a>Tutoriel : Sécurité dans Azure SQL Managed Instance à l’aide de principaux de serveur (connexions) Azure AD
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Azure SQL Managed Instance offre quasiment toutes les fonctionnalités de sécurité du dernier moteur de base de données SQL Server (Édition Entreprise) :

- Limiter l’accès dans un environnement isolé
- Utiliser des mécanismes d’authentification qui nécessitent une identité : Azure Active Directory (Azure AD) et Authentification SQL
- Utiliser des autorisations et appartenances en fonction du rôle
- Activer les fonctionnalités de sécurité

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> - Créer un principal de serveur (connexion) Azure AD pour une instance managée
> - Accorder des autorisations aux principaux de serveur (connexions) Azure AD dans une instance managée
> - Créer des utilisateurs Azure AD à partir de principaux de serveur (connexions) Azure AD
> - Affecter des autorisations aux utilisateurs Azure AD pour gérer la sécurité des bases de données
> - Utiliser l’emprunt d’identité avec des utilisateurs Azure AD
> - Utiliser des requêtes de bases de données croisées avec des utilisateurs Azure AD
> - Découvrir les fonctionnalités de sécurité, notamment la protection contre les menaces, l’audit, le masquage des données et le chiffrement

Pour plus d’informations, consultez la [vue d’ensemble d’Azure SQL Managed Instance](sql-managed-instance-paas-overview.md). 

## <a name="prerequisites"></a>Prérequis

Pour suivre le tutoriel, vérifiez que les prérequis ci-dessous sont remplis :

- [SSMS](/sql/ssms/download-sql-server-management-studio-ssms) (SQL Server Management Studio)
- Une instance managée
  - Suivez cet article : [Démarrage rapide : Créer une instance managée](instance-create-quickstart.md)
- Accès à votre instance managée et [provisionnement d’un administrateur Azure AD pour l’instance managée](../database/authentication-aad-configure.md#provision-azure-ad-admin-sql-managed-instance) Pour plus d'informations, consultez les rubriques suivantes :
  - [Connecter votre application à une instance managée](connect-application-instance.md)
  - [Architecture de connectivité de SQL Managed Instance](connectivity-architecture-overview.md)
  - [Configurer et gérer l’authentification Azure Active Directory avec SQL](../database/authentication-aad-configure.md)

## <a name="limit-access"></a>Limiter l’accès 

Les instances managées sont accessibles par le biais d’une adresse IP privée. Tout comme dans un environnement SQL Server isolé, les applications ou les utilisateurs doivent avoir accès au réseau (virtuel) SQL Managed Instance pour pouvoir établir une connexion. Pour plus d’informations, consultez [Connecter votre application à SQL Managed Instance](connect-application-instance.md).

Il est également possible de configurer un point de terminaison de service sur une instance managée, ce qui permet les connexions publiques, de la même façon que pour Azure SQL Database.
Pour plus d’informations, consultez [Configurer un point de terminaison public dans Azure SQL Managed Instance](public-endpoint-configure.md).

> [!NOTE]
> Même si les points de terminaison de service sont activés, les [règles de pare-feu Azure SQL Database](../database/firewall-configure.md) ne s’appliquent pas. Azure SQL Managed Instance a son propre [pare-feu intégré](management-endpoint-verify-built-in-firewall.md) pour gérer la connectivité.

## <a name="create-an-azure-ad-server-principal-login-using-ssms"></a>Créer un principal de serveur (connexion) Azure AD à l’aide de SSMS

Le premier principal de serveur Azure AD (connexion) peut être créé par le compte administrateur SQL standard (autre qu’Azure AD) qui est un `sysadmin`, ou par l’administrateur Azure AD pour l’instance managée créée pendant le processus de provisionnement. Pour plus d’informations, consultez [Provisionner un administrateur d’Azure Active Directory pour SQL Managed Instance](../database/authentication-aad-configure.md#provision-azure-ad-admin-sql-managed-instance).

Consultez les articles suivants pour obtenir des exemples de connexion à SQL Managed Instance :

- [Démarrage rapide : Configurer une machine virtuelle Azure pour se connecter à SQL Managed Instance](connect-vm-instance-configure.md)
- [Démarrage rapide : Configurer une connexion point à site à SQL Managed Instance en local](point-to-site-p2s-configure.md)

1. Connectez-vous à votre instance managée à l’aide d’un compte de connexion SQL standard (non Azure AD) de type `sysadmin` ou d’un administrateur Azure AD pour SQL Managed Instance, à l’aide de [SQL Server Management Studio ](point-to-site-p2s-configure.md#connect-with-ssms).

2. Dans l’ **Explorateur d’objets** , cliquez avec le bouton droit sur le serveur, puis choisissez **Nouvelle requête** .

3. Dans la fenêtre de requête, utilisez la syntaxe suivante afin de créer une connexion pour un compte Azure AD local :

    ```sql
    USE master
    GO
    CREATE LOGIN login_name FROM EXTERNAL PROVIDER
    GO
    ```

    Cet exemple permet de créer une connexion pour le compte nativeuser@aadsqlmi.onmicrosoft.com.

    ```sql
    USE master
    GO
    CREATE LOGIN [nativeuser@aadsqlmi.onmicrosoft.com] FROM EXTERNAL PROVIDER
    GO
    ```

4. Dans la barre d’outils, sélectionnez **Exécuter** pour créer la connexion.

5. Vérifiez la nouvelle connexion ajoutée en exécutant la commande T-SQL suivante :

    ```sql
    SELECT *  
    FROM sys.server_principals;  
    GO
    ```

    ![Capture d’écran de l’onglet Résultats dans l’Explorateur d’objets SSMS montrant les détails de la connexion nouvellement ajoutée : nom, principal_id, SID, type et type_desc.](./media/aad-security-configure-tutorial/native-login.png)

Pour plus d’informations, consultez [CREATE LOGIN](/sql/t-sql/statements/create-login-transact-sql?view=azuresqldb-mi-current).

## <a name="grant-permissions-to-create-logins"></a>Accorder des autorisations pour créer des connexions

Pour créer d’autres principaux de serveur (connexions) Azure AD, des rôles ou des autorisations SQL Server doivent être accordés au principal (SQL ou Azure AD).

### <a name="sql-authentication"></a>Authentification SQL

- Si la connexion est un principal SQL, seules les connexions faisant partie du rôle `sysadmin` peuvent utiliser la commande create pour créer les connexions d’un compte Azure AD.

### <a name="azure-ad-authentication"></a>Authentification Azure AD

- Pour permettre au principal de serveur (connexion) Azure AD créé de créer d’autres connexions pour d’autres utilisateurs, groupes ou applications Azure AD, attribuez le rôle serveur `sysadmin` ou `securityadmin` à la connexion.
- Au minimum, l’autorisation **ALTER ANY LOGIN** doit être octroyée au principal de serveur (connexion) Azure AD pour permettre la création d’autres principaux de serveur (connexions) Azure AD.
- Par défaut, l’autorisation standard accordée aux principaux de serveur (connexions) Azure AD créés dans la base de données MASTER est la suivante : **CONNECT SQL** et **VIEW ANY DATABASE** .
- Le rôle serveur `sysadmin` peut être attribué à de nombreux principaux de serveur (connexions) Azure AD au sein d’une instance managée.

Pour ajouter la connexion au rôle serveur `sysadmin` :

1. Reconnectez-vous à l’instance managée ou utilisez la connexion `sysadmin` existante à l’administrateur Azure AD ou au principal SQL.

1. Dans l’ **Explorateur d’objets** , cliquez avec le bouton droit sur le serveur, puis choisissez **Nouvelle requête** .

1. Attribuez le rôle serveur `sysadmin` au principal de serveur (connexion) Azure AD à l’aide de la syntaxe T-SQL suivante :

    ```sql
    ALTER SERVER ROLE sysadmin ADD MEMBER login_name
    GO
    ```

    L’exemple suivant attribue le rôle serveur `sysadmin` à la connexion nativeuser@aadsqlmi.onmicrosoft.com.

    ```sql
    ALTER SERVER ROLE sysadmin ADD MEMBER [nativeuser@aadsqlmi.onmicrosoft.com]
    GO
    ```

## <a name="create-additional-azure-ad-server-principals-logins-using-ssms"></a>Créer des principaux de serveur (connexions) Azure AD supplémentaires à l’aide de SSMS

Une fois le principal de serveur (connexion) Azure AD créé et doté des privilèges `sysadmin`, il peut créer des connexions supplémentaires à l’aide de la clause **FROM EXTERNAL PROVIDER** et de **CREATE LOGIN** .

1. Connectez-vous à l’instance managée avec le principal de serveur (connexion) Azure AD, à l’aide de SQL Server Management Studio. Entrez le nom d’hôte de votre instance managée SQL. Pour l’authentification dans SSMS, vous avez le choix entre trois options quand vous vous connectez avec un compte Azure AD :

   - Active Directory - Authentification universelle avec MFA
   - Active Directory - Authentification par mot de passe
   - Active Directory - Authentification intégrée </br>

     ![Capture d’écran de la boîte de dialogue Connexion au serveur dans SSMS avec l’option « Active Directory - Authentification universelle avec MFA » sélectionnée dans la liste déroulante Authentification.](./media/aad-security-configure-tutorial/ssms-login-prompt.png)

     Pour plus d’informations, consultez [Authentification universelle (prise en charge de SSMS pour Multi-Factor Authentication)](../database/authentication-mfa-ssms-overview.md).

1. Sélectionnez **Active Directory - Authentification universelle avec MFA** . Cela entraîne l’ouverture de la fenêtre de connexion Multi-Factor Authentication. Connectez-vous avec votre mot de passe Azure AD.

    ![Capture d’écran de la fenêtre de connexion Multi-Factor Authentication avec le curseur dans le champ Entrer le mot de passe.](./media/aad-security-configure-tutorial/mfa-login-prompt.png)

1. Dans l’ **Explorateur d’objets** SSMS, cliquez avec le bouton droit sur le serveur, puis choisissez **Nouvelle requête** .
1. Dans la fenêtre de requête, utilisez la syntaxe suivante afin de créer une connexion pour un autre compte Azure AD local :

    ```sql
    USE master
    GO
    CREATE LOGIN login_name FROM EXTERNAL PROVIDER
    GO
    ```

    Cet exemple permet de créer une connexion pour l’utilisateur Azure AD bob@aadsqlmi.net, dont le domaine aadsqlmi.net est fédéré avec le domaine Azure AD aadsqlmi.onmicrosoft.com.

    Exécutez la commande T-SQL suivante. Les comptes Azure AD fédérés remplacent SQL Managed Instance pour les connexions et utilisateurs Windows locaux.

    ```sql
    USE master
    GO
    CREATE LOGIN [bob@aadsqlmi.net] FROM EXTERNAL PROVIDER
    GO
    ```

1. Créez une base de données dans l’instance managée à l’aide de la syntaxe [CREATE DATABASE](/sql/t-sql/statements/create-database-transact-sql?view=azuresqldb-mi-current). Cette base de données va être utilisée pour tester les connexions des utilisateurs dans la prochaine section.
    1. Dans l’ **Explorateur d’objets** , cliquez avec le bouton droit sur le serveur, puis choisissez **Nouvelle requête** .
    1. Dans la fenêtre de requête, utilisez la syntaxe suivante pour créer une base de données nommée **MyMITestDB** .

        ```sql
        CREATE DATABASE MyMITestDB;
        GO
        ```

1. Créez une connexion SQL Managed Instance pour un groupe dans Azure AD. Le groupe doit exister dans Azure AD pour permettre l’ajout de la connexion à SQL Managed Instance. Consultez [Créer un groupe de base et ajouter des membres avec Azure Active Directory](../../active-directory/fundamentals/active-directory-groups-create-azure-portal.md). Créez un groupe _mygroup_ et ajoutez des membres à ce groupe.

1. Ouvrez une nouvelle fenêtre de requête dans SQL Server Management Studio.

    Cet exemple suppose qu’il existe un groupe appelé _mygroup_ dans Azure AD. Exécutez la commande suivante :

    ```sql
    USE master
    GO
    CREATE LOGIN [mygroup] FROM EXTERNAL PROVIDER
    GO
    ```

1. À des fins de test, connectez-vous à l’instance managée à l’aide de la connexion ou du groupe que vous venez de créer. Ouvrez une nouvelle connexion à l’instance managée, puis utilisez le nouveau nom de connexion au moment de l’authentification.
1. Dans l’ **Explorateur d’objets** , cliquez avec le bouton droit sur le serveur, puis choisissez **Nouvelle requête** pour la nouvelle connexion.
1. Consultez les autorisations du serveur pour le principal de serveur (connexion) Azure AD créé en exécutant la commande suivante :

      ```sql
      SELECT * FROM sys.fn_my_permissions (NULL, 'DATABASE')
      GO
      ```

> [!NOTE]
> Les utilisateurs invités Azure AD sont pris en charge pour les connexions SQL Managed Instance uniquement quand ils sont ajoutés en tant que membres d’un groupe Azure AD. Un utilisateur invité Azure AD est un compte invité dans l’instance Azure AD à laquelle appartient l’instance managée, à partir d’une autre instance Azure AD. Par exemple, joe@contoso.com (compte Azure AD) ou steve@outlook.com (compte Microsoft) peuvent être ajoutés à un groupe dans l’instance Azure AD aadsqlmi. Une fois les utilisateurs ajoutés à un groupe, vous pouvez créer une connexion dans la base de données **MASTER** SQL Managed Instance pour le groupe à l’aide de la syntaxe **CREATE LOGIN** . Les utilisateurs invités membres de ce groupe peuvent se connecter à l’instance managée à l’aide de leurs connexions actuelles (par exemple joe@contoso.com ou steve@outlook.com).

## <a name="create-an-azure-ad-user-from-the-azure-ad-server-principal-login"></a>Créer un utilisateur Azure AD à partir du principal de serveur (connexion) Azure AD

L’octroi d’autorisations pour des bases de données individuelles fonctionne sensiblement de la même façon dans SQL Managed Instance que pour les bases de données dans SQL Server. Vous pouvez créer un utilisateur à partir d’une connexion existante dans une base de données, et lui octroyer des autorisations pour cette base de données, ou l’ajouter à un rôle de base de données.

Nous avons créé une base de données appelée **MyMITestDB** ainsi qu’une connexion qui a uniquement des autorisations par défaut. La prochaine étape consiste à créer un utilisateur à partir de cette connexion. Pour le moment, le compte de connexion peut se connecter à l’instance managée et voir toutes les bases de données, mais il ne peut pas interagir avec les bases de données. Si vous vous connectez avec le compte Azure AD disposant des autorisations par défaut, et si vous tentez de développer la base de données créée, le message d’erreur suivant s’affiche :

![Capture d’écran d’un message d’erreur de l’Explorateur d’objets SSMS indiquant : « La base de données MyMITestDB n’est pas accessible. (ObjectExplorer) ».](./media/aad-security-configure-tutorial/ssms-db-not-accessible.png)

Pour plus d’informations sur l’octroi d’autorisations de base de données, consultez [Bien démarrer avec les autorisations du Moteur de base de données](/sql/relational-databases/security/authentication-access/getting-started-with-database-engine-permissions).

### <a name="create-an-azure-ad-user-and-create-a-sample-table"></a>Créer un utilisateur Azure AD et un exemple de table

1. Connectez-vous à votre instance managée à l’aide d’un compte `sysadmin` à l’aide de SQL Server Management Studio.
1. Dans l’ **Explorateur d’objets** , cliquez avec le bouton droit sur le serveur, puis choisissez **Nouvelle requête** .
1. Dans la fenêtre de requête, utilisez la syntaxe suivante pour créer un utilisateur Azure AD à partir d’un principal de serveur (connexion) Azure AD :

    ```sql
    USE <Database Name> -- provide your database name
    GO
    CREATE USER user_name FROM LOGIN login_name
    GO
    ```

    L’exemple suivant permet de créer un utilisateur bob@aadsqlmi.net à partir de la connexion bob@aadsqlmi.net:

    ```sql
    USE MyMITestDB
    GO
    CREATE USER [bob@aadsqlmi.net] FROM LOGIN [bob@aadsqlmi.net]
    GO
    ```

1. Il permet également de créer un utilisateur Azure AD à partir d’un principal de serveur (connexion) Azure AD représentant un groupe.

    L’exemple suivant permet de créer une connexion pour le groupe Azure AD _mygroup_ présent dans votre instance Azure AD.

    ```sql
    USE MyMITestDB
    GO
    CREATE USER [mygroup] FROM LOGIN [mygroup]
    GO
    ```

    Tous les utilisateurs appartenant à *mygroup* peuvent accéder à la base de données **MyMITestDB** .

    > [!IMPORTANT]
    > Quand vous créez un **USER** à partir d’un principal de serveur (connexion) Azure AD, spécifiez le même user_name que le login_name à partir de **LOGIN** .

    Pour plus d’informations, consultez [CREATE USER](/sql/t-sql/statements/create-user-transact-sql?view=azuresqldb-mi-current).

1. Dans une nouvelle fenêtre de requête, créez une table de test à l’aide de la commande T-SQL suivante :

    ```sql
    USE MyMITestDB
    GO
    CREATE TABLE TestTable
    (
    AccountNum varchar(10),
    City varchar(255),
    Name varchar(255),
    State varchar(2)
    );
    ```

1. Créez une connexion dans SSMS avec l’utilisateur créé. Notez que vous ne pouvez pas voir la table **TestTable** créée par le `sysadmin`. Nous devons donner à l’utilisateur les autorisations nécessaires pour lire les données de la base de données.

1. Vous pouvez vérifier l’autorisation actuelle de l’utilisateur en exécutant la commande suivante :

    ```sql
    SELECT * FROM sys.fn_my_permissions('MyMITestDB','DATABASE')
    GO
    ```

### <a name="add-users-to-database-level-roles"></a>Ajouter des utilisateurs aux rôles de base de données

Pour que les utilisateurs puissent voir les données de la base de données, nous pouvons leur attribuer des [rôles de base de données](/sql/relational-databases/security/authentication-access/database-level-roles).

1. Connectez-vous à votre instance managée à l’aide d’un compte `sysadmin` à l’aide de SQL Server Management Studio.

1. Dans l’ **Explorateur d’objets** , cliquez avec le bouton droit sur le serveur, puis choisissez **Nouvelle requête** .

1. Attribuez le rôle de base de données `db_datareader` à l’utilisateur Azure AD à l’aide de la syntaxe T-SQL suivante :

    ```sql
    Use <Database Name> -- provide your database name
    ALTER ROLE db_datareader ADD MEMBER user_name
    GO
    ```

    L’exemple suivant fournit à l’utilisateur bob@aadsqlmi.net et au groupe _mygroup_ les autorisations `db_datareader` pour la base de données **MyMITestDB**  :

    ```sql
    USE MyMITestDB
    GO
    ALTER ROLE db_datareader ADD MEMBER [bob@aadsqlmi.net]
    GO
    ALTER ROLE db_datareader ADD MEMBER [mygroup]
    GO
    ```

1. Vérifiez que l’utilisateur Azure AD créé dans la base de données existe, en exécutant la commande suivante :

    ```sql
    SELECT * FROM sys.database_principals
    GO
    ```

1. Créez une connexion à l’instance managée à l’aide de l’utilisateur ajouté au rôle `db_datareader`.
1. Développez la base de données dans l’ **Explorateur d’objets** pour voir la table.

    ![Capture d’écran de l’Explorateur d’objets dans SSMS montrant la structure des dossiers pour les tables dans MyMITestDB. Le dossier dbo.TestTable est mis en évidence.](./media/aad-security-configure-tutorial/ssms-test-table.png)

1. Ouvrez une nouvelle fenêtre de requête et exécutez l’instruction SELECT suivante :

    ```sql
    SELECT *
    FROM TestTable
    ```

    Pouvez-vous voir les données de la table ? Vous devez voir les colonnes retournées.

    ![Capture d’écran de l’onglet Résultats dans l’Explorateur d’objets SSMS montrant les en-têtes de colonnes de table AccountNum, City, Name et State.](./media/aad-security-configure-tutorial/ssms-test-table-query.png)

## <a name="impersonate-azure-ad-server-level-principals-logins"></a>Emprunter l’identité des principaux au niveau du serveur Azure AD (connexions)

SQL Managed Instance prend en charge l’emprunt d’identité des principaux au niveau du serveur Azure AD (connexions).

### <a name="test-impersonation"></a>Tester l’emprunt d’identité

1. Connectez-vous à votre instance managée à l’aide d’un compte `sysadmin` à l’aide de SQL Server Management Studio.

1. Dans l’ **Explorateur d’objets** , cliquez avec le bouton droit sur le serveur, puis choisissez **Nouvelle requête** .

1. Dans la fenêtre de requête, utilisez la commande suivante pour créer une procédure stockée :

    ```sql
    USE MyMITestDB
    GO  
    CREATE PROCEDURE dbo.usp_Demo  
    WITH EXECUTE AS 'bob@aadsqlmi.net'  
    AS  
    SELECT user_name();  
    GO
    ```

1. Utilisez la commande suivante pour voir que l’utilisateur dont vous empruntez l’identité durant l’exécution de la procédure stockée est **bob\@aadsqlmi.net** .

    ```sql
    Exec dbo.usp_Demo
    ```

1. Testez l’emprunt d’identité à l’aide de l’instruction EXECUTE AS LOGIN :

    ```sql
    EXECUTE AS LOGIN = 'bob@aadsqlmi.net'
    GO
    SELECT SUSER_SNAME()
    REVERT
    GO
    ```

> [!NOTE]
> Seuls les principaux au niveau du serveur SQL (connexions) faisant partie du rôle `sysadmin` peuvent exécuter les opérations suivantes ciblant les principaux Azure AD :
>
> - EXECUTE AS USER
> - EXECUTE AS LOGIN

## <a name="use-cross-database-queries"></a>Utiliser des requêtes de bases de données croisées

Les requêtes de bases de données croisées sont prises en charge pour les comptes Azure AD avec des principaux de serveur (connexions) Azure AD. Pour tester une requête de base de données croisée avec un groupe Azure AD, nous devons créer une autre base de données et une autre table. Vous pouvez ignorer la création d’une autre base de données et d’une autre table, si elles existent déjà.

1. Connectez-vous à votre instance managée à l’aide d’un compte `sysadmin` à l’aide de SQL Server Management Studio.
1. Dans l’ **Explorateur d’objets** , cliquez avec le bouton droit sur le serveur, puis choisissez **Nouvelle requête** .
1. Dans la fenêtre de requête, utilisez la commande suivante pour créer une base de données nommée **MyMITestDB2** et une table nommée **TestTable2**  :

    ```sql
    CREATE DATABASE MyMITestDB2;
    GO
    USE MyMITestDB2
    GO
    CREATE TABLE TestTable2
    (
    EmpId varchar(10),
    FirstName varchar(255),
    LastName varchar(255),
    Status varchar(10)
    );
    ```

1. Dans une nouvelle fenêtre de requête, exécutez la commande suivante pour créer l’utilisateur _mygroup_ dans la nouvelle base de données **MyMITestDB2** , puis accordez à _mygroup_ des autorisations SELECT sur cette base de données :

    ```sql
    USE MyMITestDB2
    GO
    CREATE USER [mygroup] FROM LOGIN [mygroup]
    GO
    GRANT SELECT TO [mygroup]
    GO
    ```

1. Connectez-vous à l’instance managée à l’aide de SQL Server Management Studio en tant que membre du groupe Azure AD _mygroup_ . Ouvrez une nouvelle fenêtre de requête et exécutez l’instruction SELECT pour bases de données croisées :

    ```sql
    USE MyMITestDB
    SELECT * FROM MyMITestDB2..TestTable2
    GO
    ```

    Vous devez voir les résultats de la table dans **TestTable2** .

## <a name="additional-supported-scenarios"></a>Autres scénarios pris en charge

- La gestion et les exécutions de travaux SQL Agent sont prises en charge pour les principaux de serveur (connexions) Azure AD.
- Les opérations de sauvegarde et de restauration de base de données peuvent être exécutées par les principaux de serveur (connexions) Azure AD.
- [Audit](auditing-configure.md) de toutes les instructions liées aux principaux de serveur (connexions) Azure AD et aux événements d’authentification Azure AD.
- Connexion administrateur dédiée pour les principaux de serveur (connexions) Azure AD membres du rôle serveur `sysadmin`.
- Les principaux de serveur (connexions) Azure AD sont pris en charge par l’[utilitaire sqlcmd](/sql/tools/sqlcmd-utility) et l’outil [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms).
- Les déclencheurs d’ouverture de session sont pris en charge pour les événements d’ouverture de session provenant de principaux de serveur (connexions) Azure AD.
- Vous pouvez configurer Service Broker et Database Mail à l’aide de principaux de serveur (connexions) Azure AD.

## <a name="next-steps"></a>Étapes suivantes

### <a name="enable-security-features"></a>Activer les fonctionnalités de sécurité

Consultez l’article sur les [fonctionnalités de sécurité de SQL Managed Instance](sql-managed-instance-paas-overview.md#security-features) pour obtenir la liste complète des moyens de sécuriser votre base de données. Les fonctionnalités de sécurité suivantes sont présentées :

- [Audit d’une instance managée SQL](auditing-configure.md)
- [Always Encrypted](/sql/relational-databases/security/encryption/always-encrypted-database-engine)
- [Détection de menaces](threat-detection-configure.md)
- [Masquage des données dynamiques](/sql/relational-databases/security/dynamic-data-masking)
- [Sécurité au niveau des lignes](/sql/relational-databases/security/row-level-security)
- [Chiffrement transparent des données (TDE)](/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql)

### <a name="sql-managed-instance-capabilities"></a>Fonctionnalités de SQL Managed Instance

Pour obtenir une vue d’ensemble des fonctionnalités de SQL Managed Instance, consultez :

> [!div class="nextstepaction"]
> [Fonctionnalités de SQL Managed Instance](sql-managed-instance-paas-overview.md)