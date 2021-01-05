---
title: 'Démarrage rapide : Restaurer une sauvegarde (SSMS)'
titleSuffix: Azure SQL Managed Instance
description: Ce guide de démarrage rapide vous explique comment restaurer une sauvegarde de base de données dans SQL Managed Instance à l’aide de SSMS (SQL Server Management Studio).
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: ''
ms.devlang: ''
ms.topic: quickstart
author: srdan-bozovic-msft
ms.author: srbozovi
ms.reviewer: sstein, bonova
ms.date: 12/14/2018
ms.openlocfilehash: 9b2333e38415a2c0ad50ce36c213ead711c70ab4
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96928798"
---
# <a name="quickstart-restore-a-database-to-azure-sql-managed-instance-with-ssms"></a>Démarrage rapide : Restaurer une base de données dans SQL Managed Instance avec SSMS
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Dans ce guide de démarrage rapide, vous utilisez SSMS (SQL Server Management Studio) pour restaurer une base de données (le fichier de sauvegarde standard Wide World Importers) à partir du stockage Blob Azure vers [SQL Managed Instance](sql-managed-instance-paas-overview.md).

> [!VIDEO https://www.youtube.com/embed/RxWYojo_Y3Q]

> [!NOTE]
> Pour plus d’informations sur la migration à l’aide d’Azure Database Migration Service, consultez [Migration de SQL Managed Instance à l’aide de Database Migration Service](../../dms/tutorial-sql-server-to-managed-instance.md).
> Pour plus d’informations sur les diverses méthodes de migration, consultez [Migration d’une instance SQL Server vers Azure SQL Managed Instance](migrate-to-instance-from-sql-server.md).

## <a name="prerequisites"></a>Prérequis

Dans ce guide de démarrage rapide :

- Vous utilisez des ressources du guide de démarrage rapide [Créer une instance managée](instance-create-quickstart.md).
- Vous avez besoin de la dernière version de [SSMS](/sql/ssms/sql-server-management-studio-ssms) installée.
- Vous avez besoin d’utiliser SSMS pour la connexion à SQL Managed Instance. Consultez ces guides de démarrage rapide sur la façon de se connecter :
  - [Activer le point de terminaison public](public-endpoint-configure.md) sur SQL Managed Instance (approche recommandée pour ce tutoriel).
  - [Se connecter à SQL Managed Instance à partir d’une machine virtuelle Azure](connect-vm-instance-configure.md).
  - [Configurer une connexion point à site à SQL Managed Instance en local](point-to-site-p2s-configure.md).

> [!NOTE]
> Pour plus d’informations sur la sauvegarde et la restauration d’une base de données SQL Server à l’aide du stockage Blob Azure et d’une [clé de signature d’accès partagé (SAP)](../../storage/common/storage-sas-overview.md), consultez [Sauvegarde SQL Server vers une URL](/sql/relational-databases/backup-restore/sql-server-backup-to-url?view=sql-server-2017).

## <a name="restore-from-a-backup-file"></a>Restaurer à partir d’un fichier de sauvegarde

Dans SQL Server Management Studio, suivez ces étapes pour restaurer la base de données Wide World Importers vers SQL Managed Instance. Le fichier de sauvegarde de base de données est stocké dans un compte de stockage Blob Azure préconfiguré.

1. Ouvrez SSMS et connectez-vous à votre instance managée.
2. Dans l’**Explorateur d’objets**, cliquez avec le bouton droit sur votre instance managée et sélectionnez **Nouvelle requête** pour ouvrir une nouvelle fenêtre de requête.
3. Exécutez le script SQL suivant, qui utilise un compte de stockage préconfiguré et une clé SAS pour [créer des informations d’identification](/sql/t-sql/statements/create-credential-transact-sql) dans votre instance managée.
 
   > [!IMPORTANT]
   > `CREDENTIAL` doit correspondre au chemin du conteneur, commencer par `https` et ne pas contenir de barre oblique de fin. `IDENTITY` doit être `SHARED ACCESS SIGNATURE`. `SECRET` doit être le jeton de signature d’accès partagé, et ne pas contenir de `?` au début.

   ```sql
   CREATE CREDENTIAL [https://mitutorials.blob.core.windows.net/databases]
   WITH IDENTITY = 'SHARED ACCESS SIGNATURE'
   , SECRET = 'sv=2017-11-09&ss=bfqt&srt=sco&sp=rwdlacup&se=2028-09-06T02:52:55Z&st=2018-09-04T18:52:55Z&spr=https&sig=WOTiM%2FS4GVF%2FEEs9DGQR9Im0W%2BwndxW2CQ7%2B5fHd7Is%3D'
   ```

    ![créer des informations d’identification](./media/restore-sample-database-quickstart/credential.png)

4. Pour vérifier vos informations d’identification, exécutez le script suivant, qui utilise une URL de [conteneur](https://azure.microsoft.com/services/container-instances/) permettant d’obtenir une liste de fichiers de sauvegarde.

   ```sql
   RESTORE FILELISTONLY FROM URL =
      'https://mitutorials.blob.core.windows.net/databases/WideWorldImporters-Standard.bak'
   ```

    ![liste de fichiers](./media/restore-sample-database-quickstart/file-list.png)

5. Exécutez le script suivant pour restaurer la base de données Wide World Importers.

   ```sql
   RESTORE DATABASE [Wide World Importers] FROM URL =
     'https://mitutorials.blob.core.windows.net/databases/WideWorldImporters-Standard.bak'
   ```

    ![Capture d’écran montrant l’exécution du script dans l’Explorateur d’objets, avec un message de réussite.](./media/restore-sample-database-quickstart/restore.png)

6. Exécutez le script suivant pour suivre l’état de votre restauration.

   ```sql
   SELECT session_id as SPID, command, a.text AS Query, start_time, percent_complete
      , dateadd(second,estimated_completion_time/1000, getdate()) as estimated_completion_time
   FROM sys.dm_exec_requests r
   CROSS APPLY sys.dm_exec_sql_text(r.sql_handle) a
   WHERE r.command in ('BACKUP DATABASE','RESTORE DATABASE')
   ```

7. Une fois la restauration terminée, affichez la base de données dans l’Explorateur d’objets. Vous pouvez vérifier que la restauration de la base de données est terminée dans la vue [sys.dm_operation_status](/sql/relational-databases/system-dynamic-management-views/sys-dm-operation-status-azure-sql-database).

> [!NOTE]
> Une opération de restauration de base de données est asynchrone et peut être retentée. SQL Server Management Studio peut générer une erreur en cas d’interruption de la connexion ou d’expiration d’un délai d’attente. Azure SQL Database continue d’essayer de restaurer la base de données en arrière-plan et vous pouvez suivre l’avancement de la restauration dans les vues [sys.dm_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-requests-transact-sql) et [sys.dm_operation_status](/sql/relational-databases/system-dynamic-management-views/sys-dm-operation-status-azure-sql-database).
> Dans certaines phases du processus de restauration, un identificateur unique s’affiche au lieu du nom réel de la base de données dans les vues système. Découvrez les `RESTORE`différences de comportement d’instruction [ici](./transact-sql-tsql-differences-sql-server.md#restore-statement).

## <a name="next-steps"></a>Étapes suivantes

- Si, à l’étape 5, une restauration de base de données se termine avec l’ID de message 22003, créez un fichier de sauvegarde contenant les sommes de contrôle de sauvegarde et relancez la restauration. Consultez [Activer ou désactiver les sommes de contrôle de sauvegarde au cours d’opérations de sauvegarde ou de restauration](/sql/relational-databases/backup-restore/enable-or-disable-backup-checksums-during-backup-or-restore-sql-server).
- Pour résoudre les problèmes liés à la sauvegarde vers une URL, consultez [Bonnes pratiques et résolution des problèmes liés à la sauvegarde SQL Server vers une URL](/sql/relational-databases/backup-restore/sql-server-backup-to-url-best-practices-and-troubleshooting).
- Pour obtenir une vue d’ensemble des options de connexion pour les applications, consultez [Connecter vos applications à SQL Managed Instance](connect-application-instance.md).
- Pour lancer des requêtes à l’aide d’un de vos outils ou langages préférés, consultez [Démarrage rapide : Se connecter à une base de données Azure SQL et l’interroger](../database/connect-query-content-reference-guide.md).
