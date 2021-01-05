---
title: Configurer la réplication entre des instances managées
titleSuffix: Azure SQL Managed Instance
description: Découvrez comment configurer la réplication transactionnelle entre une base de données du serveur de publication/distribution Azure SQL Managed Instance et un abonné SQL Managed Instance.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: data-movement
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: tutorial
author: MashaMSFT
ms.author: ferno
ms.reviewer: mathoma
ms.date: 04/28/2020
ms.openlocfilehash: 67902073b1484835d23566c91cbfae6f1d80d249
ms.sourcegitcommit: 693df7d78dfd5393a28bf1508e3e7487e2132293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92900457"
---
# <a name="tutorial-configure-replication-between-two-managed-instances"></a>Tutoriel : Configurer la réplication entre deux instances managées

[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

La réplication transactionnelle vous permet de répliquer des données d’une base de données vers une autre hébergée sur SQL Server ou [Azure SQL Managed Instance](sql-managed-instance-paas-overview.md). SQL Managed Instance peut être une base de données du serveur de publication, une base de données du serveur de distribution ou un abonné dans la topologie de réplication. Consultez [Configurations de réplication transactionnelle](replication-transactional-overview.md#common-configurations) pour connaître les configurations disponibles. 

La réplication transactionnelle est actuellement disponible en préversion publique pour SQL Managed Instance. 

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> - Configurer une instance gérée en tant que base de données du serveur de publication et base de données du serveur de distribution de réplication
> - Configurer une instance managée en tant qu’abonné de réplication

![Répliquer entre deux instances managées](./media/replication-between-two-instances-configure-tutorial/sqlmi-sqlmi-repl.png)

Ce tutoriel s’adresse à un public expérimenté et suppose que l’utilisateur est familiarisé avec le déploiement ainsi qu’avec la connexion aux instances managées et aux machines virtuelles SQL Server dans Azure. 


> [!NOTE]
> - Cet article décrit l’utilisation de la [réplication transactionnelle](/sql/relational-databases/replication/transactional/transactional-replication) dans SQL Database Managed Instance. Celle-ci n’est pas liée aux [groupes de basculement](../database/auto-failover-group-overview.md), une fonctionnalité d’Azure SQL Managed Instance qui vous permet de créer des réplicas lisibles complets d’instances individuelles. Des considérations supplémentaires sont à prendre en compte pour la configuration de la [réplication transactionnelle avec des groupes de basculement](replication-transactional-overview.md#with-failover-groups).



## <a name="requirements"></a>Spécifications

La configuration de SQL Managed Instance pour fonctionner en tant que base de données du serveur de publication et/ou base de données du serveur de distribution implique que les conditions suivantes soient respectées :

- L’instance gérée du serveur de publication se trouve sur le même réseau virtuel que le serveur de distribution et l’abonné, ou des [Passerelles VPN](../../vpn-gateway/vpn-gateway-howto-vnet-vnet-resource-manager-portal.md) ont été configurées entre les réseaux virtuels des trois entités. 
- La connectivité doit utiliser l’authentification SQL entre les participants de la réplication.
- Il existe un partage de compte de stockage Azure pour le répertoire de travail de réplication.
- Le port 445 (TCP sortant) est ouvert dans les règles de sécurité du NSG pour que les instances managées puissent accéder au partage de fichiers Azure.  Si vous rencontrez l’erreur `failed to connect to azure storage \<storage account name> with os error 53`, vous devez ajouter une règle de trafic sortant au groupe de sécurité réseau du sous-réseau SQL Managed Instance approprié.

## <a name="1---create-a-resource-group"></a>1 - Créer un groupe de ressources

Utilisez le [Portail Azure](https://portal.azure.com) pour créer un groupe de ressources avec le nom `SQLMI-Repl`.  

## <a name="2---create-managed-instances"></a>2 - Créer des instances managées

Utilisez le [portail Azure](https://portal.azure.com) pour créer deux [instances managées SQL](instance-create-quickstart.md) sur le même réseau virtuel et le même sous-réseau. Par exemple, nommez les deux instances managées comme suit :

- `sql-mi-pub` (avec certains caractères pour la randomisation)
- `sql-mi-sub` (avec certains caractères pour la randomisation)

Vous devrez également [configurer une machine virtuelle Azure pour qu’elle se connecte](connect-vm-instance-configure.md) à vos instances managées. 

## <a name="3---create-an-azure-storage-account"></a>3 - Créer un compte de stockage Azure

[Créez un compte de stockage Azure](../../storage/common/storage-account-create.md#create-a-storage-account) pour le répertoire de travail, puis créez un [partage de fichiers](../../storage/files/storage-how-to-create-file-share.md) au sein du compte de stockage. 

Copiez le chemin d’accès au partage de fichier au format : `\\storage-account-name.file.core.windows.net\file-share-name`

Exemple : `\\replstorage.file.core.windows.net\replshare`

Copiez les clés d’accès de stockage au format : `DefaultEndpointsProtocol=https;AccountName=<Storage-Account-Name>;AccountKey=****;EndpointSuffix=core.windows.net`

Exemple : `DefaultEndpointsProtocol=https;AccountName=replstorage;AccountKey=dYT5hHZVu9aTgIteGfpYE64cfis0mpKTmmc8+EP53GxuRg6TCwe5eTYWrQM4AmQSG5lb3OBskhg==;EndpointSuffix=core.windows.net`

Pour plus d’informations, consultez [Gérer les clés d’accès au compte de stockage](../../storage/common/storage-account-keys-manage.md). 

## <a name="4---create-a-publisher-database"></a>4 - Créer une base de données du serveur de publication

Connectez-vous à votre instance managée `sql-mi-pub` à l’aide de SQL Server Management Studio et exécutez le code Transact-SQL (T-SQL) suivant pour créer la base de données du serveur de publication :

```sql
USE [master]
GO

CREATE DATABASE [ReplTran_PUB]
GO

USE [ReplTran_PUB]
GO
CREATE TABLE ReplTest (
  ID INT NOT NULL PRIMARY KEY,
  c1 VARCHAR(100) NOT NULL,
  dt1 DATETIME NOT NULL DEFAULT getdate()
)
GO


USE [ReplTran_PUB]
GO

INSERT INTO ReplTest (ID, c1) VALUES (6, 'pub')
INSERT INTO ReplTest (ID, c1) VALUES (2, 'pub')
INSERT INTO ReplTest (ID, c1) VALUES (3, 'pub')
INSERT INTO ReplTest (ID, c1) VALUES (4, 'pub')
INSERT INTO ReplTest (ID, c1) VALUES (5, 'pub')
GO
SELECT * FROM ReplTest
GO
```

## <a name="5---create-a-subscriber-database"></a>5 - Créer une base de données de l’abonné

Connectez-vous à votre instance managée `sql-mi-sub` à l’aide de SQL Server Management Studio et exécutez le code T-SQL suivant pour créer la base de données de l’abonné vide :

```sql
USE [master]
GO

CREATE DATABASE [ReplTran_SUB]
GO

USE [ReplTran_SUB]
GO
CREATE TABLE ReplTest (
  ID INT NOT NULL PRIMARY KEY,
  c1 VARCHAR(100) NOT NULL,
  dt1 DATETIME NOT NULL DEFAULT getdate()
)
GO
```

## <a name="6---configure-distribution"></a>6 - Configurer la distribution

Connectez-vous à votre instance managée `sql-mi-pub` à l’aide de SQL Server Management Studio et exécutez le code T-SQL suivant pour configurer la base de données de distribution.

```sql
USE [master]
GO

EXEC sp_adddistributor @distributor = @@ServerName;
EXEC sp_adddistributiondb @database = N'distribution';
GO
```

## <a name="7---configure-publisher-to-use-distributor"></a>7 - Configurer la base de données du serveur de publication pour utiliser la base de données du serveur de distribution

Sur l’instance managée SQL de la base de données du serveur de publication `sql-mi-pub`, définissez l’exécution des requêtes en mode [SQLCMD](/sql/ssms/scripting/edit-sqlcmd-scripts-with-query-editor) et exécutez le code suivant pour inscrire la nouvelle base de données du serveur de distribution auprès de la base de données du serveur de publication.

```sql
:setvar username loginUsedToAccessSourceManagedInstance
:setvar password passwordUsedToAccessSourceManagedInstance
:setvar file_storage "\\storage-account-name.file.core.windows.net\file-share-name"
-- example: file_storage "\\replstorage.file.core.windows.net\replshare"
:setvar file_storage_key "DefaultEndpointsProtocol=https;AccountName=<Storage-Account-Name>;AccountKey=****;EndpointSuffix=core.windows.net"
-- example: file_storage_key "DefaultEndpointsProtocol=https;AccountName=replstorage;AccountKey=dYT5hHZVu9aTgIteGfpYE64cfis0mpKTmmc8+EP53GxuRg6TCwe5eTYWrQM4AmQSG5lb3OBskhg==;EndpointSuffix=core.windows.net"

USE [master]
EXEC sp_adddistpublisher
  @publisher = @@ServerName,
  @distribution_db = N'distribution',
  @security_mode = 0,
  @login = N'$(username)',
  @password = N'$(password)',
  @working_directory = N'$(file_storage)',
  @storage_connection_string = N'$(file_storage_key)'; -- Remove this parameter for on-premises publishers
```

   > [!NOTE]
   > Utilisez uniquement des barres obliques inverses (`\`) pour le paramètre file_storage. L’utilisation d’une barre oblique (`/`) peut provoquer une erreur lors de la connexion au partage de fichiers.

Ce script configure une base de données du serveur de publication locale sur l’instance managée, ajoute un serveur lié et crée un ensemble de travaux pour SQL Server Agent.

## <a name="8---create-publication-and-subscriber"></a>8 - Créer la publication et la base de données de l’abonné

En utilisant le mode [SQLCMD](/sql/ssms/scripting/edit-sqlcmd-scripts-with-query-editor), exécutez le script T-SQL suivant pour activer la réplication pour votre base de données et configurer la réplication entre la base de données du serveur de publication, la base de données du serveur de distribution et la base de données de l’abonné.

```sql
-- Set variables
:setvar username sourceLogin
:setvar password sourcePassword
:setvar source_db ReplTran_PUB
:setvar publication_name PublishData
:setvar object ReplTest
:setvar schema dbo
:setvar target_server "sql-mi-sub.wdec33262scj9dr27.database.windows.net"
:setvar target_username targetLogin
:setvar target_password targetPassword
:setvar target_db ReplTran_SUB

-- Enable replication for your source database
USE [$(source_db)]
EXEC sp_replicationdboption
  @dbname = N'$(source_db)',
  @optname = N'publish',
  @value = N'true';

-- Create your publication
EXEC sp_addpublication
  @publication = N'$(publication_name)',
  @status = N'active';


-- Configure your log reader agent
EXEC sp_changelogreader_agent
  @publisher_security_mode = 0,
  @publisher_login = N'$(username)',
  @publisher_password = N'$(password)',
  @job_login = N'$(username)',
  @job_password = N'$(password)';

-- Add the publication snapshot
EXEC sp_addpublication_snapshot
  @publication = N'$(publication_name)',
  @frequency_type = 1,
  @publisher_security_mode = 0,
  @publisher_login = N'$(username)',
  @publisher_password = N'$(password)',
  @job_login = N'$(username)',
  @job_password = N'$(password)';

-- Add the ReplTest table to the publication
EXEC sp_addarticle
  @publication = N'$(publication_name)',
  @type = N'logbased',
  @article = N'$(object)',
  @source_object = N'$(object)',
  @source_owner = N'$(schema)';

-- Add the subscriber
EXEC sp_addsubscription
  @publication = N'$(publication_name)',
  @subscriber = N'$(target_server)',
  @destination_db = N'$(target_db)',
  @subscription_type = N'Push';

-- Create the push subscription agent
EXEC sp_addpushsubscription_agent
  @publication = N'$(publication_name)',
  @subscriber = N'$(target_server)',
  @subscriber_db = N'$(target_db)',
  @subscriber_security_mode = 0,
  @subscriber_login = N'$(target_username)',
  @subscriber_password = N'$(target_password)',
  @job_login = N'$(username)',
  @job_password = N'$(password)';

-- Initialize the snapshot
EXEC sp_startpublication_snapshot
  @publication = N'$(publication_name)';
```

## <a name="9---modify-agent-parameters"></a>9 - Modifier les paramètres de l’agent

Azure SQL Managed Instance rencontre actuellement des problèmes de back-end concernant la connectivité avec les agents de réplication. Ce problème est en cours de résolution. La solution de contournement consiste à augmenter la valeur du délai d’expiration de connexion pour les agents de réplication.

Exécutez la commande T-SQL suivante sur le serveur de publication pour allonger le délai d’expiration de connexion :

```sql
-- Increase login timeout to 150s
update msdb..sysjobsteps set command = command + N' -LoginTimeout 150'
where subsystem in ('Distribution','LogReader','Snapshot') and command not like '%-LoginTimeout %'
```

Exécutez de nouveau la commande T-SQL suivante pour redéfinir le délai d’expiration de connexion sur la valeur par défaut si nécessaire :

```sql
-- Increase login timeout to 30
update msdb..sysjobsteps set command = command + N' -LoginTimeout 30'
where subsystem in ('Distribution','LogReader','Snapshot') and command not like '%-LoginTimeout %'
```

Redémarrez les trois agents pour appliquer ces modifications.

## <a name="10---test-replication"></a>10 - Tester la réplication

Une fois que la réplication a été configurée, vous pouvez la tester en insérant de nouveaux éléments sur la base de données du serveur de publication et en observant les modifications se propager vers la base de données de l’abonné.

Exécutez l’extrait de code T-SQL suivant pour afficher les lignes sur la base de données de l’abonné :

```sql
select * from dbo.ReplTest
```

Exécutez l’extrait de code T-SQL suivant pour insérer des lignes supplémentaires sur la base de données du serveur de publication, puis vérifiez de nouveau les lignes sur la base de données de l’abonné.

```sql
INSERT INTO ReplTest (ID, c1) VALUES (15, 'pub')
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Pour supprimer la publication, exécutez la commande T-SQL suivante :

```sql
-- Drops the publication
USE [ReplTran_PUB]
EXEC sp_droppublication @publication = N'PublishData'
GO
```

Pour supprimer l’option de réplication de la base de données, exécutez la commande T-SQL suivante :

```sql
-- Disables publishing of the database
USE [ReplTran_PUB]
EXEC sp_removedbreplication
GO
```

Pour désactiver la publication et la distribution, exécutez la commande T-SQL suivante :

```sql
-- Drops the distributor
USE [master]
EXEC sp_dropdistributor @no_checks = 1
GO
```

Vous pouvez nettoyer vos ressources Azure en [supprimant les ressources SQL Managed Instance du groupe de ressources](../../azure-resource-manager/management/manage-resources-portal.md#delete-resources), puis en supprimant le groupe de ressources `SQLMI-Repl`. 

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez également obtenir plus d’informations sur la [réplication transactionnelle avec Azure SQL Managed Instance](replication-transactional-overview.md) ou découvrir comment configurer la réplication entre une [base de données du serveur de publication/distribution SQL Managed Instance et un abonné SQL sur Machine virtuelle Azure](replication-two-instances-and-sql-server-configure-tutorial.md).