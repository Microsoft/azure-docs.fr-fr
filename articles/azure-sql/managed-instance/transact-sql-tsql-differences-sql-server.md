---
title: Différences T-SQL entre SQL Server et Azure SQL Managed Instance
description: Cet article présente les différences Transact-SQL (T-SQL) entre Azure SQL Managed Instance et SQL Server.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.devlang: ''
ms.topic: reference
author: jovanpop-msft
ms.author: jovanpop
ms.reviewer: sstein, bonova, danil
ms.date: 11/10/2020
ms.custom: seoapril2019, sqldbrb=1
ms.openlocfilehash: c18ee43eefe9c6cf9cba7f4e8f6c3fd3f55bba5a
ms.sourcegitcommit: 1bdcaca5978c3a4929cccbc8dc42fc0c93ca7b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/13/2020
ms.locfileid: "97368696"
---
# <a name="t-sql-differences-between-sql-server--azure-sql-managed-instance"></a>Différences T-SQL entre SQL Server et Azure SQL Managed Instance
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Cet article résume et explique les différences de syntaxe et de comportement entre Azure SQL Managed Instance et SQL Server. 


SQL Managed Instance offre une haute compatibilité avec le moteur de base de données SQL Server, et la plupart des fonctionnalités sont prises en charge dans une instance gérée SQL.

![Migration facile à partir de SQL Server](./media/transact-sql-tsql-differences-sql-server/migration.png)

Certaines limitations PaaS ont été introduites dans SQL Managed Instance et certains comportements ont changé par rapport à SQL Server. Les différences se répartissent en différentes catégories, à savoir : <a name="Differences"></a>

- la [disponibilité](#availability) incluant les différences dans les [groupes de disponibilité Always On](#always-on-availability-groups) et les [sauvegardes](#backup) ;
- la [sécurité](#security) incluant les différences dans l’[audit](#auditing), les [certificats](#certificates), les [informations d’identification](#credential), les [fournisseurs de chiffrement](#cryptographic-providers), les [connexions et les utilisateurs](#logins-and-users) ainsi que la [clé de service et la clé principale du service](#service-key-and-service-master-key) ;
- la [configuration](#configuration) incluant les différences dans l’[extension du pool de mémoires tampons](#buffer-pool-extension), le [classement](#collation), les [niveaux de compatibilité](#compatibility-levels), la [mise en miroir de bases de données](#database-mirroring), les [options de base de données](#database-options), le service [SQL Server Agent](#sql-server-agent) et les [options de Table](#tables) ;
- les [fonctionnalités](#functionalities) incluant [BULK INSERT/OPENROWSET](#bulk-insert--openrowset), [CLR](#clr), [DBCC](#dbcc), les [transactions distribuées](#distributed-transactions), les [événements étendus](#extended-events), les [bibliothèques externes](#external-libraries), le [flux de fichier et FileTable](#filestream-and-filetable), la [recherche sémantique de texte intégral](#full-text-semantic-search), les [serveurs liés](#linked-servers), [Polybase](#polybase), la [Réplication](#replication), la [RESTAURATION](#restore-statement), [Service Broker](#service-broker), les [procédures stockées, fonctions et déclencheurs](#stored-procedures-functions-and-triggers).
- les [paramètres d’environnement](#Environment), tels que les configurations de réseau virtuel et de sous-réseau ;

La plupart de ces fonctionnalités sont des contraintes architecturales et représentent des fonctionnalités de service.

Les problèmes connus temporaires qui ont été détectés dans SQL Managed Instance et qui seront résolus ultérieurement sont décrits dans la [page des notes de version](../database/doc-changes-updates-release-notes.md).

## <a name="availability"></a>Disponibilité

### <a name="always-on-availability-groups"></a><a name="always-on-availability-groups"></a>Groupes de disponibilité Always On

La [haute disponibilité](../database/high-availability-sla.md) étant intégrée à SQL Managed Instance, les utilisateurs ne peuvent pas la contrôler. Les instructions suivantes ne sont pas prises en charge :

- [CREATE ENDPOINT … FOR DATABASE_MIRRORING](/sql/t-sql/statements/create-endpoint-transact-sql)
- [CREATE AVAILABILITY GROUP](/sql/t-sql/statements/create-availability-group-transact-sql)
- [ALTER AVAILABILITY GROUP](/sql/t-sql/statements/alter-availability-group-transact-sql)
- [DROP AVAILABILITY GROUP](/sql/t-sql/statements/drop-availability-group-transact-sql)
- La clause [SET HADR](/sql/t-sql/statements/alter-database-transact-sql-set-hadr) de l’instruction [ALTER DATABASE](/sql/t-sql/statements/alter-database-transact-sql)

### <a name="backup"></a>Sauvegarde

SQL Managed Instance étant doté de sauvegardes automatiques, les utilisateurs peuvent créer des sauvegardes complètes `COPY_ONLY` de bases de données. Les sauvegardes différentielles de fichiers journaux et de captures instantanées de fichiers ne sont pas prises en charge.

- Avec une instance gérée SQL, vous pouvez sauvegarder une base de données d’instance seulement vers un compte de stockage Blob Azure :
  - Seul `BACKUP TO URL` est pris en charge.
  - `FILE`, `TAPE` et les unités de sauvegarde ne sont pas pris en charge.
- La plupart des options générales `WITH` sont prises en charge.
  - `COPY_ONLY` est obligatoire.
  - `FILE_SNAPSHOT` n’est pas pris en charge.
  - Options de bande : `REWIND`, `NOREWIND`, `UNLOAD` et `NOUNLOAD` ne sont pas pris en charge.
  - Options propres au journal : `NORECOVERY`, `STANDBY` et `NO_TRUNCATE` ne sont pas pris en charge.

Limites : 

- Une instance gérée SQL permet de sauvegarder une base de données d’instance vers une sauvegarde comprenant jusqu’à 32 bandes, ce qui est suffisant pour des bases de données allant jusqu’à 4 To si la compression de sauvegarde est utilisée.
- Vous ne pouvez pas exécuter `BACKUP DATABASE ... WITH COPY_ONLY` sur une base de données qui est chiffrée avec Transparent Data Encryption (TDE) managé par le service. TDE managé par le service oblige le chiffrement des sauvegardes à l’aide d’une clé de chiffrement TDE interne. La clé ne pouvant pas être exportée, vous ne pouvez pas restaurer la sauvegarde. utilisez des sauvegardes automatiques et la restauration dans le temps, ou utilisez [TDE managé par le client (BYOK)](../database/transparent-data-encryption-tde-overview.md#customer-managed-transparent-data-encryption---bring-your-own-key) à la place. Vous pouvez également désactiver le chiffrement sur la base de données.
- La taille maximale de la bande de sauvegarde en utilisant la commande `BACKUP` dans une instance gérée SQL est de 195 Go, ce qui représente la taille maximale des objets blob. Augmentez le nombre de bandes défini dans la commande de sauvegarde pour réduire la taille de chaque bande et ainsi ne pas dépasser cette limite.

    > [!TIP]
    > Pour contourner cette limitation, lorsque vous sauvegardez une base de données, soit à partir de SQL Server dans un environnement local, soit sur une machine virtuelle, vous pouvez :
    >
    > - sauvegarder sur `DISK` au lieu de sauvegarder jusqu’à `URL` ;
    > - charger les fichiers de sauvegarde sur le stockage d’objets blob ;
    > - Restaurer dans l’instance gérée SQL.
    >
    > La commande `Restore` dans une instance gérée SQL prend en charge des tailles d’objet blob plus volumineuses dans les fichiers de sauvegarde, car un autre type d’objet blob est utilisé pour le stockage des fichiers de sauvegarde chargés.

Pour plus d’informations sur les sauvegardes à l’aide de T-SQL, consultez [BACKUP](/sql/t-sql/statements/backup-transact-sql).

## <a name="security"></a>Sécurité

### <a name="auditing"></a>Audit

Les principales différences entre l’audit dans Microsoft Azure SQL et dans SQL Server sont les suivantes :

- Avec SQL Managed Instance, l’audit fonctionne au niveau du serveur. Les fichiers journaux `.xel` sont stockés dans le stockage Blob Azure.
- Dans Azure SQL Database, l’audit fonctionne au niveau de la base de données. Les fichiers journaux `.xel` sont stockés dans le stockage Blob Azure.
- Avec SQL Server, en local ou sur machines virtuelles, l’audit s’effectue au niveau serveur. Les événements sont stockés dans le système de fichiers ou dans les journaux des événements Windows.
 
L’audit XEvent dans SQL Managed Instance prend en charge les cibles de Stockage Blob Azure. Les journaux de fichiers et de Windows ne sont pas pris en charge.

Les principales différences de syntaxe `CREATE AUDIT` pour l’audit du Stockage Blob Azure sont :

- Une nouvelle syntaxe `TO URL` est fournie, vous pouvez l’utiliser pour spécifier l’URL du conteneur de stockage Blob Azure où les fichiers `.xel` sont placés.
- La syntaxe `TO FILE` n’est pas prise en charge, car SQL Managed Instance ne peut pas accéder à des partages de fichiers Windows.

Pour plus d'informations, consultez les pages suivantes : 

- [CREATE SERVER AUDIT](/sql/t-sql/statements/create-server-audit-transact-sql) 
- [ALTER SERVER AUDIT](/sql/t-sql/statements/alter-server-audit-transact-sql)
- [Audit](/sql/relational-databases/security/auditing/sql-server-audit-database-engine)

### <a name="certificates"></a>Certificats

SQL Managed Instance ne pouvant pas accéder à des partages de fichiers et à des dossiers Windows, les contraintes suivantes s’appliquent :

- le fichier `CREATE FROM`/`BACKUP TO` n’est pas pris en charge pour les certificats ;
- le certificat `CREATE`/`BACKUP` de `FILE`/`ASSEMBLY` n’est pas pris en charge ; Les fichiers de clés privés ne peuvent pas être utilisés 

Consultez [CREATE CERTIFICATE](/sql/t-sql/statements/create-certificate-transact-sql) et [BACKUP CERTIFICATE](/sql/t-sql/statements/backup-certificate-transact-sql). 
 
**Solution de contournement** : Au lieu de créer une sauvegarde du certificat et de restaurer la sauvegarde, [obtenez le contenu binaire du certificat et la clé privée, stockez-les en tant que fichier .sql et créez à partir du binaire](/sql/t-sql/functions/certencoded-transact-sql#b-copying-a-certificate-to-another-database) :

```sql
CREATE CERTIFICATE  
   FROM BINARY = asn_encoded_certificate
WITH PRIVATE KEY (<private_key_options>)
```

### <a name="credential"></a>Informations d'identification

Seuls Azure Key Vault et les identités `SHARED ACCESS SIGNATURE` sont pris en charge. Les utilisateurs de Windows ne sont pas pris en charge.

Consultez [CREATE CREDENTIAL](/sql/t-sql/statements/create-credential-transact-sql) et [ALTER CREDENTIAL](/sql/t-sql/statements/alter-credential-transact-sql).

### <a name="cryptographic-providers"></a>Fournisseurs de chiffrement

SQL Managed Instance ne pouvant pas accéder aux fichiers, vous ne pouvez pas créer de fournisseur de chiffrement :

- `CREATE CRYPTOGRAPHIC PROVIDER` n’est pas pris en charge. Consultez [CREATE CRYPTOGRAPHIC PROVIDER](/sql/t-sql/statements/create-cryptographic-provider-transact-sql).
- `ALTER CRYPTOGRAPHIC PROVIDER` n’est pas pris en charge. Consultez [ALTER CRYPTOGRAPHIC PROVIDER](/sql/t-sql/statements/alter-cryptographic-provider-transact-sql).

### <a name="logins-and-users"></a>Connexions et utilisateurs

- Les connexions SQL créées à l’aide de `FROM CERTIFICATE`, `FROM ASYMMETRIC KEY` et `FROM SID` sont prises en charge. Consultez [CREATE LOGIN](/sql/t-sql/statements/create-login-transact-sql).
- Les principaux de serveur (connexions) Azure Active Directory (Azure AD) créés avec la syntaxe [CREATE LOGIN](/sql/t-sql/statements/create-login-transact-sql?view=azuresqldb-mi-current) ou la syntaxe [CREATE USER FROM LOGIN [Azure AD Login]](/sql/t-sql/statements/create-user-transact-sql?view=azuresqldb-mi-current) sont pris en charge. Ces connexions sont créées au niveau du serveur.

    SQL Managed Instance prend en charge les principaux de base de données Azure AD avec la syntaxe `CREATE USER [AADUser/AAD group] FROM EXTERNAL PROVIDER`. Cette fonctionnalité est également appelée « utilisateurs de base de données autonome Azure AD ».

- Les connexions Windows créées avec la syntaxe `CREATE LOGIN ... FROM WINDOWS` ne sont pas prises en charge. Utilisez des utilisateurs et des connexions Azure Active Directory.
- L’utilisateur Azure AD qui a créé l’instance dispose de [privilèges d’administrateur illimités](../database/logins-create-manage.md).
- Les utilisateurs au niveau de la base de données Azure AD non-administrateurs peuvent être créés à l’aide de la syntaxe `CREATE USER ... FROM EXTERNAL PROVIDER`. Consultez [CREATE USER ... FROM EXTERNAL PROVIDER](../database/authentication-aad-configure.md#create-contained-users-mapped-to-azure-ad-identities).
- Les principaux de serveur (connexions) Azure AD prennent en charge les fonctionnalités SQL dans une seule instance gérée SQL. Les fonctionnalités nécessitant une interaction entre les instances, peu importe qu’elles se trouvent au sein du même locataire Azure AD ou de locataires différents, ne sont pas prises en charge pour les utilisateurs Azure AD. Il s’agit de fonctionnalités telles que :

  - la réplication transactionnelle SQL ;
  - le serveur de liaisons.

- La définition d’une connexion Azure AD mappée à un groupe Azure AD en tant que propriétaire de base de données n’est pas prise en charge.
- L’emprunt d’identité des principaux au niveau du serveur Azure AD à l’aide d’autres principaux Azure AD est pris en charge, par exemple avec la clause [EXECUTE AS](/sql/t-sql/statements/execute-as-transact-sql). Les limitations EXECUTE AS sont les suivantes :

  - EXECUTE AS USER n’est pas pris en charge pour les utilisateurs Azure AD quand le nom diffère du nom de connexion. Par exemple, quand l’utilisateur est créé par le biais de la syntaxe CREATE USER [myAadUser] FROM LOGIN [john@contoso.com] et que l’emprunt d’identité est tenté par le biais de EXEC AS USER = _myAadUser_. Quand vous créez un **USER** à partir d’un principal de serveur (connexion) Azure AD, spécifiez le même user_name que le login_name à partir de **LOGIN**.
  - Seuls les principaux au niveau de l’instance SQL Server (connexions) faisant partie du rôle `sysadmin` peuvent exécuter les opérations suivantes qui ciblent les principaux Azure AD :

    - EXECUTE AS USER
    - EXECUTE AS LOGIN

  - Pour emprunter l’identité d’un utilisateur avec l’instruction EXECUTE AS, l’utilisateur doit être mappé directement au principal de serveur Azure AD (connexion). Les utilisateurs membres de groupes Azure AD mappés aux principaux de serveur Azure AD ne peuvent pas faire l’objet d’un emprunt d’identité à l’aide de l’instruction EXECUTE AS, même si l’appelant dispose d’autorisations d’emprunt d’identité sur le nom d’utilisateur spécifié.

- L’exportation/importation de bases de données à l’aide de fichiers bacpac est prise en charge pour les utilisateurs Azure AD dans SQL Managed Instance avec [SSMS v18.4 ou version ultérieure](/sql/ssms/download-sql-server-management-studio-ssms) et [SQLPackage.exe](/sql/tools/sqlpackage-download).
  - Les configurations suivantes sont prises en charge avec le fichier de base de données bacpac : 
    - Exporter/importer une base de données entre différentes Managed Instance dans le même domaine Azure AD.
    - Exporter une base de données à partir de SQL Managed Instance et l’importer dans SQL Database au sein du même domaine Azure AD. 
    - Exporter une base de données à partir de SQL Database et l’importer dans SQL Managed Instance au sein du même domaine Azure AD.
    - Exporter une base de données à partir de SQL Managed Instance et l’importer dans SQL Server (version 2012 ou ultérieure).
      - Dans cette configuration, tous les utilisateurs Azure AD sont créés en tant que principaux (utilisateurs) de base de données SQL Server sans connexions. Les utilisateurs sont répertoriés sous le type `SQL` et sont visibles sous le type `SQL_USER` dans sys.database_principals. Leurs autorisations et leurs rôles restent dans les métadonnées de la base de données SQL Server et peuvent être utilisés pour l’emprunt d’identité. Toutefois, ils ne peuvent pas être utilisés pour accéder et se connecter à SQL Server à l’aide de leurs informations d’identification.

- Seule la connexion du principal au niveau du serveur, qui est créée par le processus de provisionnement de SQL Managed Instance, les membres des rôles de serveur, tels que `securityadmin` ou `sysadmin`, ou d’autres connexions disposant de l’autorisation ALTER ANY LOGIN au niveau du serveur peuvent créer les principaux de serveur (connexions) Azure AD dans la base de données master pour SQL Managed Instance.
- Si la connexion est un principal SQL, seules les connexions faisant partie du rôle `sysadmin` peuvent utiliser la commande create pour créer les connexions d’un compte Azure AD.
- La connexion Azure AD doit être membre d’un compte Azure AD dans le même annuaire que celui utilisé pour Azure SQL Managed Instance.
- Les principaux de serveur (connexions) Azure AD sont visibles dans l’Explorateur d’objets à compter de SQL Server Management Studio 18.0 préversion 5.
- Le chevauchement des principaux de serveur (connexions) Azure AD avec un compte d’administrateur Azure AD est autorisé. Les principaux de serveur (connexions) Azure AD sont prioritaires par rapport à l’Administrateur Azure AD lorsque vous résolvez le principal et que vous appliquez des autorisations à SQL Managed Instance.
- Lors de l’authentification, la séquence suivante est appliquée pour résoudre le principal d’authentification :

    1. Si le compte Azure AD existe en tant que mappage direct au principal de serveur (connexion) Azure AD, qui est présent dans sys.server_principals en tant que type « E », l’accès est accordé et les autorisations du principal de serveur (connexion) Azure AD sont appliquées.
    2. Si le compte Azure AD est membre d’un groupe Azure AD mappé au principal de serveur (connexion) Azure AD, qui est présent dans sys.server_principals en tant que type « X », l’accès est accordé et les autorisations de la connexion du groupe Azure AD sont appliquées.
    3. Si le compte Azure AD est un administrateur Azure AD spécial configuré dans le portail pour SQL Managed Instance, et qui n’existe pas dans les vues système de SQL Managed Instance, les autorisations fixes spéciales de l’administrateur Azure AD pour SQL Managed Instance (mode hérité) sont appliquées.
    4. Si le compte Azure AD existe en tant que mappage direct à un utilisateur Azure AD dans une base de données, qui est présent dans sys.database_principals en tant que type « E », l’accès est accordé et les autorisations de l’utilisateur de base de données Azure AD sont appliquées.
    5. Si le compte Azure AD est membre d’un groupe Azure AD mappé à un utilisateur Azure AD dans une base de données, qui est présent dans sys.database_principals en tant que type « X », l’accès est accordé et les autorisations de la connexion du groupe Azure AD sont appliquées.
    6. S’il existe une connexion Azure AD mappée à un compte d’utilisateur Azure AD ou à un compte de groupe Azure AD, qui se résout sur l’utilisateur qui s’authentifie, toutes les autorisations de cette connexion Azure AD sont appliquées.

### <a name="service-key-and-service-master-key"></a>Clé de service et clé principale de service

- [Backup master key](/sql/t-sql/statements/backup-master-key-transact-sql) n’est pas pris en charge (géré par le service SQL Database).
- [Restore master key](/sql/t-sql/statements/restore-master-key-transact-sql) n’est pas pris en charge (géré par le service SQL Database).
- [Backup service master key](/sql/t-sql/statements/backup-service-master-key-transact-sql) n’est pas pris en charge (géré par le service SQL Database).
- [Restore service master key](/sql/t-sql/statements/restore-service-master-key-transact-sql) n’est pas pris en charge (géré par le service SQL Database).

## <a name="configuration"></a>Configuration

### <a name="buffer-pool-extension"></a>Extension du pool de mémoires tampons

- L’[extension du pool de mémoires tampons](/sql/database-engine/configure-windows/buffer-pool-extension) n’est pas prise en charge.
- `ALTER SERVER CONFIGURATION SET BUFFER POOL EXTENSION` n’est pas pris en charge. Consultez [ALTER SERVER CONFIGURATION](/sql/t-sql/statements/alter-server-configuration-transact-sql).

### <a name="collation"></a>Classement

Le classement d’instance par défaut est `SQL_Latin1_General_CP1_CI_AS` et peut être spécifié comme paramètre de création. Consultez [Classements](/sql/t-sql/statements/collations).

### <a name="compatibility-levels"></a>Niveaux de compatibilité

- Les niveaux de compatibilité pris en charge sont 100, 110, 120, 130, 140 et 150.
- Les niveaux de compatibilité inférieurs à 100 ne sont pas pris en charge.
- Le niveau de compatibilité par défaut est de 140 pour les nouvelles bases de données. Pour les bases de données restaurées, le niveau de compatibilité reste inchangé s’il était de 100 et plus.

Consultez [Niveau de compatibilité ALTER TABLE (Transact-SQL)](/sql/t-sql/statements/alter-database-transact-sql-compatibility-level).

### <a name="database-mirroring"></a>Mise en miroir de bases de données

La mise en miroir de bases de données n’est pas prise en charge.

- Les options `ALTER DATABASE SET PARTNER` et `SET WITNESS` ne sont pas prises en charge.
- `CREATE ENDPOINT … FOR DATABASE_MIRRORING` n’est pas pris en charge.

Pour plus d’informations, consultez [ALTER DATABASE SET PARTNER AND SET WITNESS](/sql/t-sql/statements/alter-database-transact-sql-database-mirroring) et [CREATE ENDPOINT … FOR DATABASE_MIRRORING](/sql/t-sql/statements/create-endpoint-transact-sql).

### <a name="database-options"></a>Options de la base de données

- Les fichiers journaux multiples ne sont pas pris en charge.
- Les objets en mémoire ne sont pas pris en charge dans le niveau de service Usage général. 
- Il existe une limite de 280 fichiers par instance Usage général, ce qui implique un maximum de 280 fichiers par base de données. Les fichiers de données et de journaux du niveau Usage général sont tous les deux comptabilisés dans cette limite. [Le niveau Critique pour l’entreprise prend en charge 32 767 fichiers par base de données](./resource-limits.md#service-tier-characteristics).
- La base de données ne peut pas contenir de groupes de fichiers qui contiennent des données flux de fichier. La restauration échoue si le fichier.bak contient des données `FILESTREAM`. 
- Chaque fichier est placé dans Stockage Blob Azure. L’E/S et le débit par fichier dépendent de la taille de chaque fichier.

#### <a name="create-database-statement"></a>CREATE DATABASE, instruction

Les limites suivantes s’appliquent à `CREATE DATABASE` :

- Les fichiers et les groupes de fichiers ne peuvent pas être définis. 
- L’option `CONTAINMENT` n’est pas prise en charge. 
- Les options `WITH` ne sont pas prises en charge. 
   > [!TIP]
   > Comme solution de contournement, utilisez `ALTER DATABASE` après `CREATE DATABASE` pour définir les options de la base de données de manière à ajouter des fichiers ou pour définir l’autonomie. 

- L’option `FOR ATTACH` n’est pas prise en charge.
- L’option `AS SNAPSHOT OF` n’est pas prise en charge.

Pour plus d’informations, consultez [CREATE DATABASE](/sql/t-sql/statements/create-database-sql-server-transact-sql).

#### <a name="alter-database-statement"></a>ALTER DATABASE, instruction

Certaines propriétés de fichiers ne peuvent pas être définies ou modifiées :

- Un chemin de fichier ne peut pas être spécifié dans l’instruction T-SQL `ALTER DATABASE ADD FILE (FILENAME='path')`. Enlevez `FILENAME` du script car SQL Managed Instance place les fichiers automatiquement. 
- Un nom de fichier ne peut pas être modifié à l’aide de l’instruction `ALTER DATABASE`.

Les options suivantes sont définies par défaut et ne peuvent pas être modifiées :

- `MULTI_USER`
- `ENABLE_BROKER ON`
- `AUTO_CLOSE OFF`

Les options suivantes ne peuvent pas être modifiées :

- `AUTO_CLOSE`
- `AUTOMATIC_TUNING(CREATE_INDEX=ON|OFF)`
- `AUTOMATIC_TUNING(DROP_INDEX=ON|OFF)`
- `DISABLE_BROKER`
- `EMERGENCY`
- `ENABLE_BROKER`
- `FILESTREAM`
- `HADR`
- `NEW_BROKER`
- `OFFLINE`
- `PAGE_VERIFY`
- `PARTNER`
- `READ_ONLY`
- `RECOVERY BULK_LOGGED`
- `RECOVERY_SIMPLE`
- `REMOTE_DATA_ARCHIVE` 
- `RESTRICTED_USER`
- `SINGLE_USER`
- `WITNESS`

Pour en savoir plus, consultez [ALTER DATABASE](/sql/t-sql/statements/alter-database-transact-sql-file-and-filegroup-options).

### <a name="sql-server-agent"></a>SQL Server Agent

- L’activation et la désactivation de SQL Server Agent ne sont actuellement pas prises en charge dans SQL Managed Instance. L’Agent SQL est toujours en cours d’exécution.
- Les paramètres de SQL Server Agent sont en lecture seule. La procédure `sp_set_agent_properties` n’est pas prise en charge dans SQL Managed Instance. 
- travaux
  - Les étapes de travail T-SQL sont prises en charge.
  - Les travaux de réplication suivants sont pris en charge :
    - Lecteur de journaux de transactions
    - Instantané
    - Serveur de distribution
  - Les étapes de travail SSIS sont prises en charge.
  - Les autres types d’étapes de travail ne sont pas pris en charge actuellement :
    - L’étape de travail de réplication de fusion n’est pas prise en charge. 
    - L’agent de lecture de la file d’attente n’est pas pris en charge. 
    - L’interpréteur de commandes n’est pas encore pris en charge.
  - SQL Managed Instance ne peut pas accéder à des ressources externes, par exemple des partages réseau via robocopy. 
  - SQL Server Analysis Services n’est pas pris en charge.
- Les notifications sont partiellement prises en charge.
- Les notifications par e-mail sont prises en charge bien qu’elles nécessitent de votre part la configuration d’un profil Database Mail. SQL Server Agent ne peut utiliser qu’un seul profil Database Mail, et il doit être appelé `AzureManagedInstance_dbmail_profile`. 
  - Le récepteur d’appel n’est pas pris en charge.
  - NetSend n’est pas pris en charge.
  - Les alertes ne sont pas encore prises en charge.
  - Les proxies ne sont pas pris en charge.
- EventLog n’est pas pris en charge.
- L’utilisateur doit être directement mappé au principal de serveur Azure AD (connexion) pour pouvoir créer, modifier ou exécuter des travaux SQL Agent. Les utilisateurs qui ne sont pas directement mappés, par exemple les utilisateurs membres d’un groupe Azure AD qui dispose des droits de création, de modification ou d’exécution des travaux SQL Agent, ne peuvent pas effectuer ces actions. Cela est dû à l’emprunt d’identité Managed Instance et aux [limitations d’EXECUTE AS](#logins-and-users).

Les fonctionnalités suivantes de l’agent SQL ne sont pas prises en charge actuellement :

- Proxies
- Planification de travaux sur une UC inactive
- Activation ou désactivation d’un Agent
- Alertes

Pour plus d’informations sur SQL Server Agent, consultez [SQL Server Agent](/sql/ssms/agent/sql-server-agent).

### <a name="tables"></a>Tables

Les types de tables suivantes ne sont pas prises en charge :

- [FILESTREAM](/sql/relational-databases/blob/filestream-sql-server)
- [FILETABLE](/sql/relational-databases/blob/filetables-sql-server)
- [EXTERNAL TABLE](/sql/t-sql/statements/create-external-table-transact-sql) (Polybase)
- [MEMORY_OPTIMIZED](/sql/relational-databases/in-memory-oltp/introduction-to-memory-optimized-tables) (non pris en charge uniquement dans le niveau usage général)

Pour plus d’informations sur la façon de créer et de modifier des tables, consultez [CREATE TABLE](/sql/t-sql/statements/create-table-transact-sql) et [ALTER TABLE](/sql/t-sql/statements/alter-table-transact-sql).

## <a name="functionalities"></a>Fonctionnalités

### <a name="bulk-insert--openrowset"></a>Bulk insert/OPENROWSET

SQL Managed Instance ne pouvant pas accéder à des partages de fichiers et à des dossiers Windows, les fichiers doivent être importés à partir du stockage Blob Azure :

- `DATASOURCE` est requis dans la commande `BULK INSERT` lors de l’importation des fichiers depuis le Stockage Blob Azure. Consultez [BULK INSERT](/sql/t-sql/statements/bulk-insert-transact-sql).
- `DATASOURCE` est nécessaire dans la fonction `OPENROWSET` lorsque vous lisez le contenu d’un fichier à partir du stockage Blob Azure. Consultez [OPENROWSET](/sql/t-sql/functions/openrowset-transact-sql).
- `OPENROWSET` peut être utilisé pour lire des données à partir d’Azure SQL Database, Azure SQL Managed Instance, ou d’instances SQL Server. Les autres sources, telles que les bases de données Oracle ou les fichiers Excel, ne sont pas prises en charge.

### <a name="clr"></a>CLR

SQL Managed Instance ne pouvant pas accéder à des partages de fichiers et à des dossiers Windows, les contraintes suivantes s’appliquent :

- Seul `CREATE ASSEMBLY FROM BINARY` est pris en charge. Consultez [CREATE ASSEMBLY FROM BINARY](/sql/t-sql/statements/create-assembly-transact-sql). 
- `CREATE ASSEMBLY FROM FILE` n’est pas pris en charge. Consultez [CREATE ASSEMBLY FROM FILE](/sql/t-sql/statements/create-assembly-transact-sql).
- `ALTER ASSEMBLY` ne peut pas référencer des fichiers. Consultez [ALTER ASSEMBLY](/sql/t-sql/statements/alter-assembly-transact-sql).

### <a name="database-mail-db_mail"></a>Database Mail (db_mail)
 - `sp_send_dbmail` ne peut pas envoyer de pièces jointes à l’aide du paramètre @file_attachments. Le système de fichiers local ainsi que les partages externes ou le Stockage Blob Azure ne sont pas accessibles avec cette procédure.
 - Consultez les problèmes connus liés au paramètre `@query` et à l’authentification.
 
### <a name="dbcc"></a>DBCC

Les instructions DBCC non documentées qui sont activées dans SQL Server ne sont pas prises en charge dans SQL Managed Instance.

- Seuls un nombre limité d’indicateurs de trace globaux sont pris en charge. Les `Trace flags` de session ne sont pas pris en charge. Consultez les [indicateurs de trace](/sql/t-sql/database-console-commands/dbcc-traceon-trace-flags-transact-sql).
- [DBCC TRACEOFF](/sql/t-sql/database-console-commands/dbcc-traceoff-transact-sql) et [DBCC TRACEON](/sql/t-sql/database-console-commands/dbcc-traceon-transact-sql) fonctionnent avec le nombre limité d’indicateurs de trace globaux.
- [DBCC CHECKDB](/sql/t-sql/database-console-commands/dbcc-checkdb-transact-sql) ne peut pas être utilisé avec les options REPAIR_ALLOW_DATA_LOSS, REPAIR_FAST et REPAIR_REBUILD, car la base de données ne peut pas être définie sur le mode `SINGLE_USER`. Consultez l’article sur les [différences avec ALTER DATABASE](#alter-database-statement). L’endommagement potentiel de la base de données est géré par l’équipe de support Azure. Contactez le support Azure si vous constatez des signes de corruption de la base de données.

### <a name="distributed-transactions"></a>Transactions distribuées

La prise en charge partielle des [transactions distribuées](../database/elastic-transactions-overview.md) est actuellement en préversion publique. Les scénarios pris en charge sont les suivants :
* Transactions où les participants sont uniquement des instances Azure SQL Managed Instance qui font partie du [groupe d’approbations de serveurs](./server-trust-group-overview.md).
* Transactions lancées à partir de .NET (classe TransactionScope) et de Transact-SQL.

Actuellement, Azure SQL Managed Instance ne prend pas en charge d’autres scénarios qui sont régulièrement pris en charge par MSDTC, localement ou dans des machines virtuelles Azure.

### <a name="extended-events"></a>Événements étendus

Certaines cibles propres à Windows pour les événements étendus (XEvent) ne sont pas prises en charge :

- Le `etw_classic_sync` cible n’est pas pris en charge. Stockez les fichiers `.xel` dans le stockage Blob Azure. Consultez [etw_classic_sync target](/sql/relational-databases/extended-events/targets-for-extended-events-in-sql-server#etw_classic_sync_target-target).
- Le `event_file` cible n’est pas pris en charge. Stockez les fichiers `.xel` dans le stockage Blob Azure. Consultez [event_file target](/sql/relational-databases/extended-events/targets-for-extended-events-in-sql-server#event_file-target).

### <a name="external-libraries"></a>Bibliothèques externes

Les bibliothèques externes R et Python dans la base de données sont prises en charge dans la préversion publique limitée. Consultez [Machine Learning Services dans Azure SQL Managed Instance (préversion)](machine-learning-services-overview.md).

### <a name="filestream-and-filetable"></a>Flux de fichier et Filetable

- Les données flux de fichier ne sont pas prises en charge.
- La base de données ne peut pas contenir de groupes de fichiers avec des données `FILESTREAM`.
- `FILETABLE` n’est pas pris en charge.
- Les tables ne peuvent pas avoir de types `FILESTREAM`.
- Les fonctions suivantes ne sont pas prises en charge :
  - `GetPathLocator()`
  - `GET_FILESTREAM_TRANSACTION_CONTEXT()`
  - `PathName()`
  - `GetFileNamespacePat)`
  - `FileTableRootPath()`

Pour plus d’informations, consultez [FILESTREAM](/sql/relational-databases/blob/filestream-sql-server) et [FileTables](/sql/relational-databases/blob/filetables-sql-server).

### <a name="full-text-semantic-search"></a>Recherche sémantique de texte intégral

La [recherche sémantique](/sql/relational-databases/search/semantic-search-sql-server) n’est pas prise en charge.

### <a name="linked-servers"></a>Serveurs liés

Les serveurs liés dans SQL Managed Instance prennent en charge un nombre limité de cibles :

- Les cibles prises en charge sont SQL Managed Instance, SQL Database, Azure Synapse SQL [serverless](https://devblogs.microsoft.com/azure-sql/linked-server-to-synapse-sql-to-implement-polybase-like-scenarios-in-managed-instance/) et les pools dédiés, ainsi que les instances SQL Server. 
- Les serveurs liés ne prennent pas en charge les transactions accessibles en écriture distribuées (MS DTC).
- Les cibles qui ne sont pas prises en charge sont des fichiers, Analysis Services et d’autres SGBDR. Essayez d’utiliser l’importation CSV native à partir du Stockage Blob Azure à l’aide de `BULK INSERT` ou `OPENROWSET` comme alternative pour l’importation de fichiers, ou chargez des fichiers à l’aide d’un [pool SQL serverless dans Azure Synapse Analytics](https://devblogs.microsoft.com/azure-sql/linked-server-to-synapse-sql-to-implement-polybase-like-scenarios-in-managed-instance/).

Opérations : 

- Les transactions d’écriture entre instances ne sont pas prises en charge.
- `sp_dropserver` est pris en charge pour supprimer un serveur lié. Consultez [sp_dropserver](/sql/relational-databases/system-stored-procedures/sp-dropserver-transact-sql).
- La fonction `OPENROWSET` peut être utilisée pour exécuter des requêtes uniquement sur des instances SQL Server. Elles peuvent être managées, locales ou dans des machines virtuelles. Consultez [OPENROWSET](/sql/t-sql/functions/openrowset-transact-sql).
- La fonction `OPENDATASOURCE` peut être utilisée pour exécuter des requêtes uniquement sur des instances SQL Server. Elles peuvent être managées, locales ou dans des machines virtuelles. Seules les valeurs `SQLNCLI`, `SQLNCLI11` et `SQLOLEDB` sont prises en charge en tant que fournisseur. par exemple `SELECT * FROM OPENDATASOURCE('SQLNCLI', '...').AdventureWorks2012.HumanResources.Employee`. Consultez [OPENDATASOURCE](/sql/t-sql/functions/opendatasource-transact-sql).
- Les serveurs liés ne peuvent pas être utilisés pour lire des fichiers (Excel, CSV) à partir des partages réseau. Essayez d’utiliser [BULK INSERT](/sql/t-sql/statements/bulk-insert-transact-sql#e-importing-data-from-a-csv-file), [OPENROWSET](/sql/t-sql/functions/openrowset-transact-sql#g-accessing-data-from-a-csv-file-with-a-format-file) qui lit les fichiers CSV à partir du Stockage Blob Azure, ou un [serveur lié qui fait référence à un pool SQL serverless dans Synapse Analytics](https://devblogs.microsoft.com/azure-sql/linked-server-to-synapse-sql-to-implement-polybase-like-scenarios-in-managed-instance/). Suivez ces requêtes sur l’[élément de commentaires de SQL Managed Instance](https://feedback.azure.com/forums/915676-sql-managed-instance/suggestions/35657887-linked-server-to-non-sql-sources)|

### <a name="polybase"></a>PolyBase

Les seuls types de sources externes disponibles sont SGBDR (en préversion publique) vers Azure SQL Database, Azure SQL Managed Instance et le pool Azure Synapse. Vous pouvez utiliser [une table externe qui fait référence à un pool SQL serverless dans Synapse Analytics](https://devblogs.microsoft.com/azure-sql/read-azure-storage-files-using-synapse-sql-external-tables/) comme solution de contournement pour les tables externes Polybase qui lisent directement à partir du stockage Azure. Dans Azure SQL Managed Instance, vous pouvez utiliser des serveurs liés à [un pool SQL serverless dans Synapse Analytics](https://devblogs.microsoft.com/azure-sql/linked-server-to-synapse-sql-to-implement-polybase-like-scenarios-in-managed-instance/) ou SQL Server pour lire les données de stockage Azure.
Pour plus d’informations sur PolyBase, consultez [PolyBase](/sql/relational-databases/polybase/polybase-guide).

### <a name="replication"></a>Réplication

- Les types de réplication d’instantané et bidirectionnelle sont pris en charge. La réplication de fusion, la réplication d’égal à égal et les abonnements modifiables ne sont pas pris en charge.
- La [réplication transactionnelle](replication-transactional-overview.md) est disponible en préversion publique dans SQL Managed Instance sous certaines contraintes :
    - Tous les types de participants à la réplication (serveur de publication, serveur de distribution, abonné d’extraction et abonné de type push) peuvent être placés sur SQL Managed Instance, mais le serveur de publication et le serveur de distribution doivent être tous deux dans le cloud ou locaux.
    - SQL Managed Instance peut communiquer avec les versions récentes de SQL Server. Consultez la [matrice des versions prises en charge](replication-transactional-overview.md#supportability-matrix) pour plus d’informations.
    - La réplication transactionnelle présente des [exigences de mise en réseau supplémentaires](replication-transactional-overview.md#requirements).

Pour plus d’informations sur la configuration de la réplication transactionnelle, consultez les tutoriels suivants :
- [Réplication entre un éditeur d’instance gérée SQL et un abonné d’instance gérée SQL](replication-between-two-instances-configure-tutorial.md)
- [Réplication entre un éditeur d’instance gérée SQL, un distributeur d’instance gérée SQL et un abonné SQL Server](replication-two-instances-and-sql-server-configure-tutorial.md)

### <a name="restore-statement"></a>L’instruction RESTORE 

- Syntaxe prise en charge :
  - `RESTORE DATABASE`
  - `RESTORE FILELISTONLY ONLY`
  - `RESTORE HEADER ONLY`
  - `RESTORE LABELONLY ONLY`
  - `RESTORE VERIFYONLY ONLY`
- Syntaxe non prise en charge :
  - `RESTORE LOG ONLY`
  - `RESTORE REWINDONLY ONLY`
- Source : 
  - `FROM URL` (Stockage Blob Azure) est l’unique option prise en charge.
  - L’unité de sauvegarde/`FROM DISK`/`TAPE` n’est pas prise en charge.
  - Les jeux de sauvegarde ne sont pas pris en charge.
- Les options `WITH` ne sont pas prises en charge. Les tentatives de restauration incluant `WITH` telles que `DIFFERENTIAL`, `STATS`, `REPLACE`, etc. échouent.
- `ASYNC RESTORE`: la restauration continue même si la connexion cliente s’arrête. Si votre connexion est interrompue, vous pouvez vérifier l’affichage `sys.dm_operation_status` pour l’état d’une opération de restauration, et pour la création et la suppression d’une base de données. Consultez [sys.dm_operation_status](/sql/relational-databases/system-dynamic-management-views/sys-dm-operation-status-azure-sql-database). 

Les options de base de données suivantes sont fixées ou remplacées et ne peuvent pas être modifiées ultérieurement : 

- `NEW_BROKER` si le broker n’est pas activé dans le fichier .bak. 
- `ENABLE_BROKER` si le broker n’est pas activé dans le fichier .bak. 
- `AUTO_CLOSE=OFF` si une base de données dans le fichier .bak comprend `AUTO_CLOSE=ON`. 
- `RECOVERY FULL` si une base de données dans le fichier .bak a le mode de récupération `SIMPLE` ou `BULK_LOGGED`.
- Un groupe de fichiers à mémoire optimisée est ajouté et appelé XTP s’il n’était pas dans le fichier .bak source. 
- Tout groupe de fichiers à mémoire optimisée existant est renommé XTP. 
- Les options `SINGLE_USER` et `RESTRICTED_USER` sont changées en `MULTI_USER`.

Limites : 

- Les sauvegardes des bases de données endommagées peuvent être restaurées en fonction du type d’altération, mais les sauvegardes automatisées ne sont pas effectuées tant que la corruption n’est pas corrigée. Assurez-vous que vous exécutez `DBCC CHECKDB` sur l’instance gérée SQL source et utilisez la sauvegarde `WITH CHECKSUM` afin d’éviter ce problème.
- La restauration d’un fichier `.BAK` d’une base de données qui contient une limitation décrite dans ce document (par exemple, `FILESTREAM` ou des objets `FILETABLE`) ne peut pas être restaurée sur SQL Managed instance.
- Les fichiers `.BAK` qui contiennent plusieurs jeux de sauvegarde ne peuvent pas être restaurés. 
- Les fichiers `.BAK` qui contiennent plusieurs fichiers journaux ne peuvent pas être restaurés.
- Les sauvegardes contenant des bases de données dont la taille excède 8 To, contenant des objets OLTP en mémoire actifs ou comprenant plus de 280 fichiers par instance ne peuvent pas être restaurées sur une instance d’usage général. 
- Les sauvegardes contenant des bases de données dont la taille excède 4 To ou contenant des objets OLTP en mémoire dont la taille totale est supérieure à la taille spécifiée dans les [limites de ressources](resource-limits.md) ne peuvent pas être restaurées sur une instance critique pour l’entreprise.
Pour plus d’informations sur les instructions de restauration, consultez [Instructions RESTORE](/sql/t-sql/statements/restore-statements-transact-sql).

 > [!IMPORTANT]
 > Les mêmes limitations s’appliquent à une opération de restauration dans le temps intégrée. À titre d’exemple, une base de données à usage général dont la taille est supérieure à 4 To ne peut pas être restaurée sur une instance critique pour l’entreprise. Une base de données critique pour l’entreprise comprenant des fichiers OLTP en mémoire ou plus 280 fichiers ne peut pas être restaurée sur une instance à usage général.

### <a name="service-broker"></a>Service Broker

Le Service Broker entre instances n’est pas pris en charge :

- `sys.routes`: Comme prérequis, vous devez sélectionner l’adresse à partir de sys.routes. L’adresse doit être LOCAL sur tous les itinéraires. Voir [sys.routes](/sql/relational-databases/system-catalog-views/sys-routes-transact-sql).
- `CREATE ROUTE`: vous ne pouvez pas utiliser `CREATE ROUTE` avec `ADDRESS` si la valeur de celle-ci est différente de `LOCAL`. Voir [CREATE ROUTE](/sql/t-sql/statements/create-route-transact-sql).
- `ALTER ROUTE`: vous ne pouvez pas utiliser `ALTER ROUTE` avec `ADDRESS` si la valeur de celle-ci est différente de `LOCAL`. Voir [ALTER ROUTE](/sql/t-sql/statements/alter-route-transact-sql). 

### <a name="stored-procedures-functions-and-triggers"></a>Procédures stockées, fonctions et déclencheurs

- `NATIVE_COMPILATION` n’est pas pris en charge dans le niveau Usage général.
- Les options [sp_configure](/sql/relational-databases/system-stored-procedures/sp-configure-transact-sql) suivantes ne sont pas prises en charge : 
  - `allow polybase export`
  - `allow updates`
  - `filestream_access_level`
  - `remote access`
  - `remote data archive`
  - `remote proc trans`
- `sp_execute_external_scripts` n’est pas pris en charge. Consultez [sp_execute_external_scripts](/sql/relational-databases/system-stored-procedures/sp-execute-external-script-transact-sql#examples).
- `xp_cmdshell` n’est pas pris en charge. Consultez [xp_cmdshell](/sql/relational-databases/system-stored-procedures/xp-cmdshell-transact-sql).
- `Extended stored procedures` ne sont pas pris en charge, ce qui inclut `sp_addextendedproc` et `sp_dropextendedproc`. Consultez [Procédures stockées étendues](/sql/relational-databases/system-stored-procedures/general-extended-stored-procedures-transact-sql).
- `sp_attach_db`, `sp_attach_single_file_db` et `sp_detach_db` ne sont pas pris en charge. Consultez [sp_attach_db](/sql/relational-databases/system-stored-procedures/sp-attach-db-transact-sql), [sp_attach_single_file_db](/sql/relational-databases/system-stored-procedures/sp-attach-single-file-db-transact-sql) et [sp_detach_db](/sql/relational-databases/system-stored-procedures/sp-detach-db-transact-sql).

### <a name="system-functions-and-variables"></a>Fonctions et variables système

Les variables, fonctions et vues suivantes retournent des résultats différents :

- `SERVERPROPERTY('EngineEdition')` retourne la valeur 8. Cette propriété identifie une instance gérée SQL de façon unique. Consultez [SERVERPROPERTY](/sql/t-sql/functions/serverproperty-transact-sql).
- `SERVERPROPERTY('InstanceName')` retourne la valeur NULL, car le concept d’instance tel qu’il existe pour SQL Server ne s’applique pas à une instance gérée SQL. Consultez [SERVERPROPERTY(« InstanceName »)](/sql/t-sql/functions/serverproperty-transact-sql).
- `@@SERVERNAME` retourne un nom DNS « connectable » complet, par exemple, my-managed-instance.wcus17662feb9ce98.database.windows.net. Consultez [@@SERVERNAME](/sql/t-sql/functions/servername-transact-sql). 
- `SYS.SERVERS` retourne le nom DNS « connectable » complet, tel que `myinstance.domain.database.windows.net` pour les propriétés « name » et « data_source ». Consultez [SYS. SERVERS](/sql/relational-databases/system-catalog-views/sys-servers-transact-sql).
- `@@SERVICENAME` retourne la valeur NULL, car le concept de service tel qu’il existe pour SQL Server ne s’applique pas à une instance gérée SQL. Consultez [@@SERVICENAME](/sql/t-sql/functions/servicename-transact-sql).
- `SUSER_ID` est pris en charge. Il retourne NULL si la connexion Azure AD n’est pas dans sys.syslogins. Consultez [SUSER_ID](/sql/t-sql/functions/suser-id-transact-sql). 
- `SUSER_SID` n’est pas pris en charge. Les données incorrectes sont retournées, ce qui est un problème temporaire connu. Consultez [SUSER_SID](/sql/t-sql/functions/suser-sid-transact-sql). 

## <a name="environment-constraints"></a><a name="Environment"></a>Contraintes d’environnement

### <a name="subnet"></a>Subnet
-  Vous ne pouvez pas placer d’autres ressources (par exemple des machines virtuelles) dans le sous-réseau sur lequel vous avez déployé votre instance gérée SQL. Déployez ces ressources à l’aide d’un sous-réseau différent.
- Le sous-réseau doit avoir un nombre suffisant d’[adresses IP](connectivity-architecture-overview.md#network-requirements) disponibles. Le nombre minimal est 16, mais la recommandation est de disposer d’au moins 32 adresses IP dans le sous-réseau.
- [Les points de terminaison de service ne peuvent pas être associés au sous-réseau de l’instance gérée SQL](connectivity-architecture-overview.md#network-requirements). Vérifiez que l’option Points de terminaison de service est désactivée quand vous créez le réseau virtuel.
- Le nombre de vCores et de types d’instances que vous pouvez déployer dans une région ont certaines [contraintes et limites](resource-limits.md#regional-resource-limitations).
- Il existe certaines [règles de sécurité qui doivent être appliquées sur le sous-réseau](connectivity-architecture-overview.md#network-requirements).

### <a name="vnet"></a>Réseau virtuel
- Le réseau virtuel peut être déployé à l’aide du modèle de ressource ; le modèle classique pour réseau virtuel n’est pas pris en charge.
- Après la création d’une instance gérée SQL, le déplacement de l’instance gérée SQL ou du réseau virtuel vers un autre groupe de ressources ou vers un autre abonnement n’est pas pris en charge.
- Certains services, tels que App Service Environment, Logic Apps et SQL Managed Instance (utilisés pour la géoréplication, la réplication transactionnelle ou via des serveurs liés), ne peuvent pas accéder aux instances SQL Managed Instance dans des régions différentes si leurs réseaux virtuels sont connectés au moyen du [Peering mondial](../../virtual-network/virtual-networks-faq.md#what-are-the-constraints-related-to-global-vnet-peering-and-load-balancers). Vous pouvez vous connecter à ces ressources via ExpressRoute ou une connexion entre deux réseaux virtuels, par l’intermédiaire de passerelles de réseau virtuel.

### <a name="failover-groups"></a>Groupes de basculement
Les bases de données système ne sont pas répliquées vers l’instance secondaire dans un groupe de basculement. Par conséquent, les scénarios qui dépendent des objets des bases de données système sont impossibles sur l’instance secondaire, à moins que les objets ne soient créés manuellement.

### <a name="tempdb"></a>TEMPDB
- La taille de fichier maximale de `tempdb` ne peut pas être supérieure à 24 Go par cœur sur un niveau Usage général. La taille maximale de `tempdb` sur un niveau Critique pour l’entreprise est limitée à la taille de stockage de SQL Managed Instance. La taille du fichier journal `Tempdb` est limitée à 120 Go sur le niveau Usage général. Certaines requêtes peuvent retourner une erreur si elles ont besoin de plus de 24 Go par cœur dans `tempdb` ou si elles produisent plus de 120 Go de données de journal.
- `Tempdb` est toujours divisée en 12 fichiers de données : 1 fichier de données principal, également appelé Master, et 11 fichiers de données non principaux. La structure de fichier ne peut pas être modifiée et de nouveaux fichiers ne peuvent pas être ajoutés à `tempdb`. 
- Les [métadonnées `tempdb` à mémoire optimisée](/sql/relational-databases/databases/tempdb-database?view=sql-server-ver15#memory-optimized-tempdb-metadata), une nouvelle fonctionnalité de base de données en mémoire SQL Server 2019, ne sont pas prises en charge.
- Les objets créés dans la base de données model ne peuvent pas être créés automatiquement dans `tempdb` après un redémarrage ou un basculement, car `tempdb` n’obtient pas sa liste initiale d’objets de la base de données model. Vous devez créer des objets dans `tempdb` manuellement après chaque redémarrage ou après un basculement.

### <a name="msdb"></a>MSDB

Les schémas MSDB suivants dans SQL Managed Instance doivent être détenus par leurs rôles prédéfinis respectifs :

- Rôles généraux
  - TargetServersRole
- [Rôles de base de données fixes](/sql/ssms/agent/sql-server-agent-fixed-database-roles?view=sql-server-ver15)
  - SQLAgentUserRole
  - SQLAgentReaderRole
  - SQLAgentOperatorRole
- [Rôles DatabaseMail](/sql/relational-databases/database-mail/database-mail-configuration-objects?view=sql-server-ver15#DBProfile) :
  - DatabaseMailUserRole
- [Rôles de service d'intégration](/sql/integration-services/security/integration-services-roles-ssis-service?view=sql-server-ver15) :
  - db_ssisadmin
  - db_ssisltduser
  - db_ssisoperator
  
> [!IMPORTANT]
> La modification des noms de rôles prédéfinis, des noms de schéma et des propriétaires de schéma prédéfinis par les clients aura un impact sur le fonctionnement normal du service. Toutes les modifications apportées à ces valeurs seront restaurées aux valeurs prédéfinies dès qu’elles seront détectées, ou au plus tard lors de la prochaine mise à jour du service et ce, pour garantir un fonctionnement normal de ce dernier.

### <a name="error-logs"></a>Journaux des erreurs

SQL Managed Instance ajoute des informations détaillées dans les journaux des erreurs. Beaucoup d’événements système internes sont journalisés dans le journal des erreurs. utilisez une procédure personnalisée pour lire les journaux des erreurs en excluant les entrées non pertinentes. Pour plus d’informations, consultez [SQL Managed Instance – sp_readmierrorlog](/archive/blogs/sqlcat/azure-sql-db-managed-instance-sp_readmierrorlog) ou [Extension SQL Managed Instance (préversion)](/sql/azure-data-studio/azure-sql-managed-instance-extension#logs) pour Azure Data Studio.

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur SQL Managed Instance, consultez [Présentation de SQL Managed Instance](sql-managed-instance-paas-overview.md).
- Pour consulter la liste des fonctionnalités et les comparer, voir [Comparaison des fonctionnalités Azure SQL Managed Instance](../database/features-comparison.md).
- Pour obtenir les mises à jour de version et l’état des problèmes connus, consultez [Notes de publication SQL Managed Instance](../database/doc-changes-updates-release-notes.md)
- Pour obtenir un guide de démarrage rapide qui montre comment créer une instance gérée SQL, voir [Créer une instance gérée SQL](instance-create-quickstart.md).
