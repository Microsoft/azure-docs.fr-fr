---
title: Migrer une base de données de SQL Server vers Azure Arc enabled SQL Managed Instance
description: Migrer une base de données de SQL Server vers Azure Arc enabled SQL Managed Instance
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: vin-yu
ms.author: vinsonyu
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 86563b0a44bade2cedaf76af3c247821756111fe
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90930282"
---
# <a name="migrate-sql-server-to-azure-arc-enabled-sql-managed-instance"></a>Migrer : SQL Server vers Azure Arc enabled SQL Managed Instance

Ce scénario vous guide tout au long des étapes de migration d’une base de données d’une instance SQL Server vers Azure SQL Managed Instance dans Azure Arc via deux méthodes différentes de sauvegarde et de restauration.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="use-azure-blob-storage"></a>Utiliser le stockage Blob Azure 

Utilisez le stockage Blob Azure pour la migration vers Azure Arc enabled SQL Managed Instance.

Cette méthode utilise le stockage Blob Azure comme emplacement de stockage temporaire où effectuer la sauvegarde, puis à partir duquel effectuer la restauration.

### <a name="prerequisites"></a>Prérequis

- [Installer Azure Data Studio](install-client-tools.md)
- [Installer l’Explorateur Stockage Azure](https://azure.microsoft.com/features/storage-explorer/)
- Abonnement Azure

### <a name="step-1-provision-azure-blob-storage"></a>Étape 1 : Provisionner le stockage Blob Azure

1. Suivez les étapes décrites dans [Créer un compte de stockage Blob Azure](../../storage/blobs/storage-blob-create-account-block-blob.md?tabs=azure-portal)
1. Lancer l’Explorateur Stockage Azure
1. [Connectez-vous à Azure](../../vs-azure-tools-storage-manage-with-storage-explorer.md?tabs=windows#sign-in-to-azure) pour accéder au stockage Blob créé à l’étape précédente.
1. Cliquez avec le bouton droit sur le compte de stockage Blob, puis sélectionnez **Créer un conteneur d’objets blob** pour créer un conteneur où le fichier de sauvegarde sera stocké.

### <a name="step-2-get-storage-blob-credentials"></a>Étape 2 : Obtenir les informations d’identification du stockage Blob

1. Dans l’Explorateur Stockage Azure, cliquez avec le bouton droit sur le conteneur d’objets blob qui vient d’être créé et sélectionnez **Obtenir une signature d’accès partagé**

1. Sélectionnez **Lire**, **Écrire** et **Lister**

1. Sélectionnez **Créer**

   Prenez note de l’URI et de la chaîne de requête figurant dans cet écran. Ils seront nécessaires dans des étapes ultérieures. Cliquez sur le bouton **Copier** pour enregistrer dans le Bloc-notes/OneNote, etc.

1. Fermez la fenêtre **Signature d’accès partagé**.

### <a name="step-3-backup-database-file-to-azure-blob-storage"></a>Étape 3 : Sauvegarder le fichier de base de données dans le stockage Blob Azure

Dans cette étape, nous allons nous connecter au serveur SQL Server source et créer le fichier de sauvegarde de la base de données que vous voulez migrer vers SQL Managed Instance - Azure Arc.

1. Lancer Azure Data Studio
1. Connectez-vous à l’instance SQL Server qui contient la base de données que vous voulez migrer vers SQL Managed Instance - Azure Arc.
1. Cliquez avec le bouton droit sur la base de données et sélectionnez **Nouvelle requête**.
1. Préparez votre requête au format suivant en remplaçant les espaces réservés indiqués par `<...>` en utilisant les informations de la signature d’accès partagé des étapes précédentes.  Une fois que vous avez remplacé les valeurs, exécutez la requête.

   ```sql
   IF NOT EXISTS  
   (SELECT * FROM sys.credentials
   WHERE name = 'https://<mystorageaccountname>.blob.core.windows.net/<mystorageaccountcontainername>')  
   CREATE CREDENTIAL [https://<mystorageaccountname>.blob.core.windows.net/<mystorageaccountcontainername>]
     WITH IDENTITY = 'SHARED ACCESS SIGNATURE',  
      SECRET = '<SAS_TOKEN>';  
   ```

1. De même, préparez la commande **BACKUP DATABASE** comme suit pour créer un fichier de sauvegarde dans le conteneur d’objets blob.  Une fois que vous avez remplacé les valeurs, exécutez la requête.

   ```sql
   BACKUP DATABASE <database name> TO URL = 'https://<mystorageaccountname>.blob.core.windows.net/<mystorageaccountcontainername>'
   ```

1. Ouvrez l’Explorateur Stockage Azure et vérifiez que le fichier de sauvegarde créé à l’étape précédente est visible dans le conteneur d’objets blob.

### <a name="step-4-restore-the-database-from-azure-blob-storage-to-sql-managed-instance---azure-arc"></a>Étape 4 : Restaurer la base de données depuis le stockage Blob Azure vers SQL Managed Instance - Azure Arc

1. Dans Azure Data Studio, connectez-vous à l’instance SQL Managed Instance - Azure Arc.
1. Développez **Bases de données système**, cliquez avec le bouton droit sur la base de données **master** et sélectionnez **Nouvelle requête**.
1. Dans la fenêtre de l’éditeur de requête, préparez et exécutez la même requête que celle de l’étape précédente pour créer les informations d’identification.

   ```sql
   IF NOT EXISTS  
   (SELECT * FROM sys.credentials
   WHERE name = 'https://<mystorageaccountname>.blob.core.windows.net/<mystorageaccountcontainername>')  
   CREATE CREDENTIAL [https://<mystorageaccountname>.blob.core.windows.net/<mystorageaccountcontainername>]
     WITH IDENTITY = 'SHARED ACCESS SIGNATURE',  
     SECRET = '<SAS_TOKEN>';  
   ```

1. Préparez et exécutez la commande ci-dessous pour vérifier que le fichier de sauvegarde est lisible et intact.

   ```console
   RESTORE FILELISTONLY FROM URL = 'https://<mystorageaccountname>.blob.core.windows.net/<mystorageaccountcontainername>/<file name>.bak'
   ```

1. Préparez et exécutez la commande **RESTORE DATABASE** comme suit pour restaurer le fichier de sauvegarde dans une base de données sur SQL Managed Instance - Azure Arc

   ```sql
   RESTORE DATABASE <database name> FROM URL = 'https://<mystorageaccountname>.blob.core.windows.net/<mystorageaccountcontainername>/<file name>'
   WITH MOVE 'Test' to '/var/opt/mssql/data/<file name>.mdf'
   ,MOVE 'Test_log' to '/var/opt/mssql/data/<file name>.ldf'
   ,RECOVERY  
   ,REPLACE  
   ,STATS = 5;  
   GO
   ```

Découvrez plus en détail la sauvegarde vers une URL ici :

[Documentation sur la sauvegarde vers une URL](/sql/relational-databases/backup-restore/sql-server-backup-to-url)

[Sauvegarder vers une URL en utilisant SQL Server Management Studio (SSMS)](/sql/relational-databases/tutorial-sql-server-backup-and-restore-to-azure-blob-storage-service)

-------

## <a name="method-2-copy-the-backup-file-into-an-azure-sql-managed-instance---azure-arc-pod-using-kubectl"></a>Méthode 2 : Copier le fichier de sauvegarde dans un pod Azure SQL Managed Instance - Azure Arc en utilisant kubectl

Cette méthode vous montre comment produire un fichier de sauvegarde que vous créez via n’importe quelle méthode, puis comment le copier dans le stockage local du pod Azure SQL Managed Instance afin de pouvoir effectuer une restauration à partir de là, comme vous le feriez sur un système de fichiers standard sur Windows ou Linux. Dans ce scénario, vous allez utiliser la commande `kubectl cp` pour copier le fichier depuis un emplacement donné dans le système de fichiers du pod.

### <a name="prerequisites"></a>Prérequis

- Installer et configurer kubectl pour le faire pointer vers votre cluster Kubernetes sur lequel les services de données Azure Arc sont déployés
- Avoir un outil comme Azure Data Studio ou SQL Server Management Server installé et connecté au serveur SQL Server où vous voulez créer le fichier de sauvegarde, OU avoir un fichier .bak existant déjà créé sur votre système de fichiers local.

### <a name="step-1-backup-the-database-if-you-havent-already"></a>Étape 1 : Sauvegarder la base de données si vous ne l’avez pas déjà fait

Sauvegardez la base de données SQL Server dans le chemin de votre fichier local, comme n’importe quelle sauvegarde standard SQL Server sur disque :

 ```sql
BACKUP DATABASE Test
TO DISK = 'c:\tmp\test.bak'
WITH FORMAT, MEDIANAME = 'Test’ ;
GO
```

### <a name="step-2-copy-the-backup-file-into-the-pods-file-system"></a>Étape 2 : Copier le fichier de sauvegarde dans le système de fichiers du pod

Recherchez le nom du pod où l’instance SQL est déployée. C’est généralement similaire à ceci : `pod/<sqlinstancename>-0`.

Obtenez la liste de tous les pods en exécutant :

 ```console
kubectl get pods -n <namespace of data controller>
```

Exemple :

Copiez le fichier de sauvegarde depuis le stockage local vers le pod SQL du cluster.

 ```console
kubectl cp <source file location> <pod name>:var/opt/mssql/data/<file name> -n <namespace name>

#Example:
kubectl cp C:\Backupfiles\test.bak sqlinstance1-0:var/opt/mssql/data/test.bak -n arc
```

### <a name="step-3-restore-the-database"></a>Étape 3 : Restaurer la base de données

Préparez et exécutez la commande RESTORE pour restaurer le fichier de sauvegarde dans Azure SQL Managed Instance - Azure Arc

```sql
RESTORE DATABASE test FROM DISK = '/var/opt/mssql/data/<file name>.bak'
WITH MOVE '<database name>' to '/var/opt/mssql/data/<file name>.mdf'  
,MOVE '<database name>' to '/var/opt/mssql/data/<file name>_log.ldf'  
,RECOVERY  
,REPLACE  
,STATS = 5;  
GO
```

Exemple :

```sql
RESTORE DATABASE test FROM DISK = '/var/opt/mssql/data/test.bak'
WITH MOVE 'test' to '/var/opt/mssql/data/test.mdf'  
,MOVE 'test' to '/var/opt/mssql/data/test_log.ldf'  
,RECOVERY  
,REPLACE  
,STATS = 5;  
GO
```


## <a name="next-steps"></a>Étapes suivantes

[En savoir plus sur les fonctionnalités et les capacités d’Azure Arc enabled SQL Managed Instance](managed-instance-features.md)

[Commencer en créant un contrôleur de données](create-data-controller.md)

[Vous avez déjà créé un contrôleur de données ? Créez une instance Azure Arc enabled SQL Managed Instance](create-sql-managed-instance.md)