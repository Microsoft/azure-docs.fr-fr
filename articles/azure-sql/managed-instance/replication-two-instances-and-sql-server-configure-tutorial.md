---
title: Configurer la réplication transactionnelle entre Azure SQL Managed Instance et SQL Server
description: Tutoriel expliquant comment configurer la réplication entre une instance managée de serveur de publication, une instance managée de serveur de distribution et un abonné SQL Server sur une machine virtuelle Azure, ainsi que les composants réseau nécessaires, tels que la zone DNS privée et VNET Peering.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: security
ms.custom: sqldbrb=1
ms.topic: tutorial
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 11/21/2019
ms.openlocfilehash: d2b45f5b51f4656294632aa46f679a7a09c06ed3
ms.sourcegitcommit: 1cf157f9a57850739adef72219e79d76ed89e264
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2020
ms.locfileid: "94593924"
---
# <a name="tutorial-configure-transactional-replication-between-azure-sql-managed-instance-and-sql-server"></a>Tutoriel : Configurer la réplication transactionnelle entre Azure SQL Managed Instance et SQL Server
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

La réplication transactionnelle vous permet de répliquer des données d'une base de données vers une autre hébergée sur SQL Server ou [Azure SQL Managed Instance](sql-managed-instance-paas-overview.md). SQL Managed Instance peut être une base de données du serveur de publication, une base de données du serveur de distribution ou un abonné dans la topologie de réplication. Consultez [Configurations de réplication transactionnelle](replication-transactional-overview.md#common-configurations) pour connaître les configurations disponibles. 

La réplication transactionnelle est actuellement disponible en préversion publique pour SQL Managed Instance. 

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> - Configurer une instance managée en tant que serveur de publication de réplication
> - Configurer une instance managée en tant que serveur de distribution de réplication
> - Configurer SQL Server en tant qu’abonné

![Réplication entre un serveur de publication d’instance managée, un serveur de distribution d’instance managée et un abonné SQL Server](./media/replication-two-instances-and-sql-server-configure-tutorial/sqlmi-to-sql-replication.png)

Ce tutoriel s’adresse à un public expérimenté et suppose que l’utilisateur est familiarisé avec le déploiement ainsi qu’avec la connexion aux instances managées et aux machines virtuelles SQL Server dans Azure. 


> [!NOTE]
> Cet article décrit l’utilisation de la [réplication transactionnelle](/sql/relational-databases/replication/transactional/transactional-replication) dans SQL Database Managed Instance. Celle-ci n’est pas liée aux [groupes de basculement](../database/auto-failover-group-overview.md), une fonctionnalité d’Azure SQL Managed Instance qui vous permet de créer des réplicas lisibles complets d’instances individuelles. Des considérations supplémentaires sont à prendre en compte pour la configuration de la [réplication transactionnelle avec des groupes de basculement](replication-transactional-overview.md#with-failover-groups).

## <a name="prerequisites"></a>Prérequis

Pour suivre le tutoriel, vérifiez que les prérequis ci-dessous sont remplis :

- Un [abonnement Azure](https://azure.microsoft.com/free/).
- Savoir comment déployer deux instances managées au sein d’un même réseau virtuel
- Disposer d’un abonné SQL Server, localement ou sur une machine virtuelle Azure. Ce tutoriel utilise une machine virtuelle Azure  
- [SQL Server Management Studio (SSMS) 18.0 ou version ultérieure](/sql/ssms/download-sql-server-management-studio-ssms)
- La version la plus récente [d’Azure PowerShell](/powershell/azure/install-az-ps).
- Les ports 445 et 1433 autorisent le trafic SQL via le pare-feu Azure et le pare-feu Windows.

## <a name="create-the-resource-group"></a>Créer le groupe de ressources

Utilisez l’extrait de code PowerShell suivant pour créer un groupe de ressources :

```powershell-interactive
# set variables
$ResourceGroupName = "SQLMI-Repl"
$Location = "East US 2"

# Create a new resource group
New-AzResourceGroup -Name  $ResourceGroupName -Location $Location
```

## <a name="create-two-managed-instances"></a>Créer deux instances managées

Créez deux instances managées dans ce nouveau groupe de ressources à l’aide du [portail Azure](https://portal.azure.com).

- Le nom de l’instance managée du serveur de publication doit être `sql-mi-publisher` (avec quelques caractères en plus pour la randomisation), et le nom du réseau virtuel doit être `vnet-sql-mi-publisher`.
- Le nom de l’instance managée du serveur de distribution doit être `sql-mi-distributor` (avec quelques caractères en plus pour la randomisation), et il doit se trouver _dans le même réseau virtuel que l’instance managée du serveur de publication_.

   ![Utiliser le réseau virtuel du serveur de publication pour le serveur de distribution](./media/replication-two-instances-and-sql-server-configure-tutorial/use-same-vnet-for-distributor.png)

Pour plus d’informations sur la création d’une instance managée, consultez [Créer une instance managée dans le portail](instance-create-quickstart.md).

  > [!NOTE]
  > Par souci de simplicité, et parce qu’il s’agit de la configuration la plus courante, ce tutoriel suggère de placer l’instance managée du serveur de distribution dans le même réseau virtuel que le serveur de publication. Toutefois, il est possible de créer le serveur de distribution dans un autre réseau virtuel. Pour ce faire, vous devez configurer VNET Peering entre les réseaux virtuels du serveur de publication et du serveur de distribution, puis configurer VNET Peering entre les réseaux virtuels du serveur de distribution et de l’abonné.

## <a name="create-a-sql-server-vm"></a>Créer une machine virtuelle SQL Server

Créez une machine virtuelle SQL Server dans le [portail Azure](https://portal.azure.com). La machine virtuelle SQL Server doit présenter les caractéristiques suivantes :

- Nom : `sql-vm-sub`
- Image : SQL Server 2016 ou version ultérieure
- Groupe de ressources : identique à celui de l’instance managée
- Réseau virtuel : `sql-vm-sub-vnet`

Pour plus d’informations sur le déploiement d’une machine virtuelle SQL Server sur Azure, consultez [Guide de démarrage rapide : Créer une machine virtuelle SQL Server](../virtual-machines/windows/sql-vm-create-portal-quickstart.md).

## <a name="configure-vnet-peering"></a>Configurer VNET Peering

Configurez VNET Peering pour permettre la communication entre le réseau virtuel des deux instances managées et le réseau virtuel de SQL Server. Pour cela, utilisez cet extrait de code PowerShell :

```powershell-interactive
# Set variables
$SubscriptionId = '<SubscriptionID>'
$resourceGroup = 'SQLMI-Repl'
$pubvNet = 'sql-mi-publisher-vnet'
$subvNet = 'sql-vm-sub-vnet'
$pubsubName = 'Pub-to-Sub-Peer'
$subpubName = 'Sub-to-Pub-Peer'

$virtualNetwork1 = Get-AzVirtualNetwork `
  -ResourceGroupName $resourceGroup `
  -Name $pubvNet

 $virtualNetwork2 = Get-AzVirtualNetwork `
  -ResourceGroupName $resourceGroup `
  -Name $subvNet  

# Configure VNet peering from publisher to subscriber
Add-AzVirtualNetworkPeering `
  -Name $pubsubName `
  -VirtualNetwork $virtualNetwork1 `
  -RemoteVirtualNetworkId $virtualNetwork2.Id

# Configure VNet peering from subscriber to publisher
Add-AzVirtualNetworkPeering `
  -Name $subpubName `
  -VirtualNetwork $virtualNetwork2 `
  -RemoteVirtualNetworkId $virtualNetwork1.Id

# Check status of peering on the publisher VNet; should say connected
Get-AzVirtualNetworkPeering `
 -ResourceGroupName $resourceGroup `
 -VirtualNetworkName $pubvNet `
 | Select PeeringState

# Check status of peering on the subscriber VNet; should say connected
Get-AzVirtualNetworkPeering `
 -ResourceGroupName $resourceGroup `
 -VirtualNetworkName $subvNet `
 | Select PeeringState

```

Une fois l’appairage VNet établi, testez la connectivité en lançant SQL Server Management Studio (SSMS) sur SQL Server et en vous connectant aux deux instances managées. Pour plus d’informations sur la connexion à une instance managée à l’aide de SSMS, consultez [Utiliser SSMS pour se connecter à SQL Managed Instance](point-to-site-p2s-configure.md#connect-with-ssms).

![Tester la connectivité aux instances managées](./media/replication-two-instances-and-sql-server-configure-tutorial/test-connectivity-to-mi.png)

## <a name="create-a-private-dns-zone"></a>Créer une zone DNS privée

Une zone DNS privée autorise le routage DNS entre les instances managées et SQL Server.

### <a name="create-a-private-dns-zone"></a>Créer une zone DNS privée

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sélectionnez **Créer une ressource** pour créer une ressource Azure.
1. Recherchez `private dns zone` sur la Place de marché Microsoft Azure.
1. Sélectionnez la ressource **Zone DNS privée** publiée par Microsoft, puis sélectionnez **Créer** pour créer la zone DNS.
1. Sélectionnez l’abonnement et le groupe de ressources dans la liste déroulante.
1. Fournissez un nom arbitraire pour votre zone DNS, par exemple `repldns.com`.

   ![Créer une zone DNS privée](./media/replication-two-instances-and-sql-server-configure-tutorial/create-private-dns-zone.png)

1. Sélectionnez **Revoir + créer**. Passez en revue les paramètres de votre zone DNS privée, puis sélectionnez **Créer** pour créer votre ressource.

### <a name="create-an-a-record"></a>Créer un enregistrement A

1. Accédez à votre nouvelle **zone DNS privée**, puis sélectionnez **Vue d’ensemble**.
1. Sélectionnez **+ Jeu d’enregistrements** pour créer un enregistrement A.
1. Indiquez le nom de votre machine virtuelle SQL Server, ainsi que l’adresse IP interne privée.

   ![Configurer un enregistrement A](./media/replication-two-instances-and-sql-server-configure-tutorial/configure-a-record.png)

1. Sélectionnez **OK** pour créer l’enregistrement A.

### <a name="link-the-virtual-network"></a>Lier le réseau virtuel

1. Accédez à votre nouvelle **zone DNS privée**, puis sélectionnez **Liaisons de réseau virtuel**.
1. Sélectionnez **Ajouter**.
1. Fournissez un nom pour le lien, tel que `Pub-link`.
1. Sélectionnez votre abonnement dans la liste déroulante, puis sélectionnez le réseau virtuel de votre instance managée de serveur de publication.
1. Cochez la case en regard de l’option **Activer l’inscription automatique**.

   ![Créer une liaison de réseau virtuel](./media/replication-two-instances-and-sql-server-configure-tutorial/configure-vnet-link.png)

1. Sélectionnez **OK** pour créer une liaison de réseau virtuel.
1. Répétez ces étapes afin d’ajouter une liaison pour le réseau virtuel de l’abonné, avec un nom tel que `Sub-link`.

## <a name="create-an-azure-storage-account"></a>Créer un compte de stockage Azure

[Créez un compte de stockage Azure](../../storage/common/storage-account-create.md#create-a-storage-account) pour le répertoire de travail, puis créez un [partage de fichiers](../../storage/files/storage-how-to-create-file-share.md) au sein du compte de stockage.

Copiez le chemin d’accès au partage de fichier au format : `\\storage-account-name.file.core.windows.net\file-share-name`

Exemple : `\\replstorage.file.core.windows.net\replshare`

Copiez la chaîne de connexion de la clé d’accès de stockage au format suivant : `DefaultEndpointsProtocol=https;AccountName=<Storage-Account-Name>;AccountKey=****;EndpointSuffix=core.windows.net`

Exemple : `DefaultEndpointsProtocol=https;AccountName=replstorage;AccountKey=dYT5hHZVu9aTgIteGfpYE64cfis0mpKTmmc8+EP53GxuRg6TCwe5eTYWrQM4AmQSG5lb3OBskhg==;EndpointSuffix=core.windows.net`

Pour plus d’informations, consultez [Gérer les clés d’accès au compte de stockage](../../storage/common/storage-account-keys-manage.md).

## <a name="create-a-database"></a>Création d'une base de données

Créez une base de données sur l’instance managée de serveur de publication. Pour ce faire, procédez comme suit :

1. Lancez SQL Server Management Studio sur SQL Server.
1. Connectez-vous à l’instance managée `sql-mi-publisher`.
1. Ouvrez une fenêtre **Nouvelle requête**, puis exécutez la requête T-SQL suivante pour créer la base de données.

```sql
-- Create the databases
USE [master]
GO

-- Drop database if it exists
IF EXISTS (SELECT * FROM sys.sysdatabases WHERE name = 'ReplTutorial')
BEGIN
    DROP DATABASE ReplTutorial
END
GO

-- Create new database
CREATE DATABASE [ReplTutorial]
GO

-- Create table
USE [ReplTutorial]
GO
CREATE TABLE ReplTest (
   ID INT NOT NULL PRIMARY KEY,
   c1 VARCHAR(100) NOT NULL,
   dt1 DATETIME NOT NULL DEFAULT getdate()
)
GO

-- Populate table with data
USE [ReplTutorial]
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

## <a name="configure-distribution"></a>Configurer la distribution

Une fois que la connectivité est établie et que vous disposez d’un exemple de base de données, vous pouvez configurer la distribution sur votre instance managée `sql-mi-distributor`. Pour ce faire, procédez comme suit :

1. Lancez SQL Server Management Studio sur SQL Server.
1. Connectez-vous à l’instance managée `sql-mi-distributor`.
1. Ouvrez une fenêtre **Nouvelle requête**, puis exécutez le code Transact-SQL suivant pour configurer la distribution sur l’instance managée de serveur de distribution :

   ```sql
   EXEC sp_adddistributor @distributor = 'sql-mi-distributor.b6bf57.database.windows.net', @password = '<distributor_admin_password>'
   
   EXEC sp_adddistributiondb @database = N'distribution'
   
   EXEC sp_adddistpublisher @publisher = 'sql-mi-publisher.b6bf57.database.windows.net', -- primary publisher
        @distribution_db = N'distribution',
        @security_mode = 0,
        @login = N'azureuser',
        @password = N'<publisher_password>',
        @working_directory = N'\\replstorage.file.core.windows.net\replshare',
        @storage_connection_string = N'<storage_connection_string>'
        -- example: @storage_connection_string = N'DefaultEndpointsProtocol=https;AccountName=replstorage;AccountKey=dYT5hHZVu9aTgIteGfpYE64cfis0mpKTmmc8+EP53GxuRg6TCwe5eTYWrQM4AmQSG5lb3OBskhg==;EndpointSuffix=core.windows.net'

   ```

   > [!NOTE]
   > Utilisez uniquement des barres obliques inverses (`\`) pour le paramètre @working_directory. L’utilisation d’une barre oblique (`/`) peut provoquer une erreur lors de la connexion au partage de fichiers.

1. Connectez-vous à l’instance managée `sql-mi-publisher`.
1. Ouvrez une fenêtre **Nouvelle requête**, puis exécutez le code Transact-SQL suivant pour inscrire le serveur de distribution auprès du serveur de publication :

   ```sql
   Use MASTER
   EXEC sys.sp_adddistributor @distributor = 'sql-mi-distributor.b6bf57.database.windows.net', @password = '<distributor_admin_password>'
   ```

## <a name="create-the-publication"></a>Créer la publication

Une fois la distribution configurée, vous pouvez créer la publication. Pour ce faire, procédez comme suit :

1. Lancez SQL Server Management Studio sur SQL Server.
1. Connectez-vous à l’instance managée `sql-mi-publisher`.
1. Dans l’**Explorateur d’objets**, développez le nœud **Réplication**, puis cliquez avec le bouton droit sur le dossier **Publications locales**. Sélectionnez **Nouvelle publication...** .
1. Sélectionnez **Suivant** pour quitter la page d’accueil.
1. Dans la page **Base de données de publication**, sélectionnez la base de données `ReplTutorial` que vous avez créée précédemment. Sélectionnez **Suivant**.
1. Dans la page **Type de publication**, sélectionnez **Publication transactionnelle**. Sélectionnez **Suivant**.
1. Dans la page **Articles**, cochez la case en regard de **Tables**. Sélectionnez **Suivant**.
1. Dans la page **Filtrer les lignes de la table**, sélectionnez **Suivant** sans ajouter de filtres.
1. Dans la page **Agent d’instantané**, cochez la case en regard de l’option **Créer une capture instantanée immédiatement et garder cette dernière disponible pour l’initialisation des abonnements**. Sélectionnez **Suivant**.
1. Dans la page **Sécurité de l’agent**, sélectionnez **Paramètres de sécurité...** . Fournissez les informations de connexion SQL Server qui doivent être utilisées pour l’Agent d’instantané et pour vous connecter au serveur de publication. Sélectionnez **OK** pour fermer la page **Sécurité de l’Agent d’instantané**. Sélectionnez **Suivant**.

   ![Configurer la sécurité de l’Agent d’instantané](./media/replication-two-instances-and-sql-server-configure-tutorial/snapshot-agent-security.png)

1. Dans la page **Actions de l’Assistant**, choisissez **Créer la publication** et si vous le souhaitez, vous pouvez choisir **Générer un fichier de script comportant les étapes de création de la publication** pour enregistrer ce script ultérieurement.
1. Dans la page **Terminer l’Assistant**, nommez votre publication `ReplTest`, puis sélectionnez **Suivant** pour créer votre publication.
1. Une fois votre publication créée, actualisez le nœud **Réplication** dans l’**Explorateur d’objets**, puis développez **Publications locales** pour voir votre nouvelle publication.

## <a name="create-the-subscription"></a>Créer l’abonnement

Une fois la publication créée, vous pouvez créer l’abonnement. Pour ce faire, procédez comme suit :

1. Lancez SQL Server Management Studio sur SQL Server.
1. Connectez-vous à l’instance managée `sql-mi-publisher`.
1. Ouvrez une fenêtre **Nouvelle requête**, puis exécutez le code Transact-SQL suivant pour ajouter l’agent d’abonnement et l’agent de distribution. Utilisez le DNS dans le nom de l’abonné.

```sql
use [ReplTutorial]
exec sp_addsubscription
@publication = N'ReplTest',
@subscriber = N'sql-vm-sub.repldns.com', -- include the DNS configured in the private DNS zone
@destination_db = N'ReplSub',
@subscription_type = N'Push',
@sync_type = N'automatic',
@article = N'all',
@update_mode = N'read only',
@subscriber_type = 0

exec sp_addpushsubscription_agent
@publication = N'ReplTest',
@subscriber = N'sql-vm-sub.repldns.com', -- include the DNS configured in the private DNS zone
@subscriber_db = N'ReplSub',
@job_login = N'azureuser',
@job_password = '<Complex Password>',
@subscriber_security_mode = 0,
@subscriber_login = N'azureuser',
@subscriber_password = '<Complex Password>',
@dts_package_location = N'Distributor'
GO
```

## <a name="test-replication"></a>Tester la réplication

Une fois que la réplication a été configurée, vous pouvez la tester en insérant de nouveaux éléments sur la base de données du serveur de publication et en observant les modifications se propager vers la base de données de l’abonné.

Exécutez l’extrait de code T-SQL suivant pour afficher les lignes sur la base de données de l’abonné :

```sql
Use ReplSub
select * from dbo.ReplTest
```

Exécutez l’extrait de code T-SQL suivant pour insérer des lignes supplémentaires sur la base de données du serveur de publication, puis vérifiez de nouveau les lignes sur la base de données de l’abonné.

```sql
Use ReplTutorial
INSERT INTO ReplTest (ID, c1) VALUES (15, 'pub')
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

1. Accédez à votre groupe de ressources sur le [portail Azure](https://portal.azure.com).
1. Sélectionnez la ou les instances gérées, puis choisissez **Supprimer**. Saisissez `yes` dans la zone de texte pour confirmer que vous souhaitez supprimer la ressource, puis sélectionnez **Supprimer**. Ce processus peut prendre un certain temps en arrière-plan et tant qu’il n’est pas terminé, vous ne pourrez pas supprimer le *cluster virtuel* ou d’autres ressources dépendantes. Supervisez la suppression dans l’onglet **Activité** pour confirmer que votre instance managée a été supprimée.
1. Une fois l’instance managée supprimée, supprimez le *cluster virtuel* en le sélectionnant dans votre groupe de ressources, puis en choisissant **Supprimer**. Saisissez `yes` dans la zone de texte pour confirmer que vous souhaitez supprimer la ressource, puis sélectionnez **Supprimer**.
1. Supprimez toutes les ressources restantes. Saisissez `yes` dans la zone de texte pour confirmer que vous souhaitez supprimer la ressource, puis sélectionnez **Supprimer**.
1. Supprimez le groupe de ressources en sélectionnant **Supprimer le groupe de ressources**, saisissez le nom du groupe de ressources, `myResourceGroup`, puis sélectionnez **Supprimer**.

## <a name="known-errors"></a>Erreurs connues

### <a name="windows-logins-are-not-supported"></a>Comptes de connexion Windows non pris en charge

`Exception Message: Windows logins are not supported in this version of SQL Server.`

L’agent a été configuré avec un compte de connexion Windows au lieu d’un compte de connexion SQL Server. Utilisez la page **Sécurité de l’agent** dans **Propriétés de la publication** pour remplacer le compte de connexion Windows par un compte de connexion SQL Server.

### <a name="failed-to-connect-to-azure-storage"></a>Échec de la connexion au stockage Azure

`Connecting to Azure Files Storage '\\replstorage.file.core.windows.net\replshare' Failed to connect to Azure Storage '' with OS error: 53.`

2019-11-19 02:21:05.07 Obtention de la chaîne de connexion du stockage Azure pour replstorage 2019-11-19 02:21:05.07 Connexion à Azure Files Storage '\\replstorage.file.core.windows.net\replshare' 2019-11-19 02:21:31.21 Échec de la connexion au stockage Azure avec l’erreur de système d’exploitation : 53.

Cette erreur est probablement due au fait que le port 445 est fermé dans le pare-feu Azure, dans le pare-feu Windows ou dans les deux.

`Connecting to Azure Files Storage '\\replstorage.file.core.windows.net\replshare' Failed to connect to Azure Storage '' with OS error: 55.`

L’utilisation d’une barre oblique au lieu d’une barre oblique inverse dans le chemin de fichier du partage de fichiers peut provoquer cette erreur.
  
  - Le chemin suivant est correct : `\\replstorage.file.core.windows.net\replshare`
  - Ce chemin peut provoquer l’erreur de système d’exploitation 55 : `'\\replstorage.file.core.windows.net/replshare'`

### <a name="could-not-connect-to-subscriber"></a>Impossible de se connecter à l’abonné

`The process could not connect to Subscriber 'SQL-VM-SUB`
`Could not open a connection to SQL Server [53].`
`A network-related or instance-specific error has occurred while establishing a connection to SQL Server. Server is not found or not accessible. Check if instance name is correct and if SQL Server is configured to allow remote connections.`

Solutions possibles :

- Vérifiez que le port 1433 est ouvert.
- Vérifiez que le protocole TCP/IP est activé sur l’abonné.
- Vérifiez que le nom DNS a été utilisé lors de la création de l’abonné.
- Vérifiez que vos réseaux virtuels sont correctement liés dans la zone DNS privée.
- Vérifiez que votre enregistrement A est configuré correctement.
- Vérifiez que VNET Peering est configuré correctement.

### <a name="no-publications-to-which-you-can-subscribe"></a>Il n’existe aucune publication à laquelle vous pouvez vous abonner

Quand vous ajoutez un nouvel abonnement à l’aide de l’Assistant **Nouvel abonnement**, la page **Publication** peut ne contenir aucune base de données ou publication, et le message d’erreur suivant peut s’afficher :

`There are no publications to which you can subscribe, either because this server has no publications or because you do not have sufficient privileges to access the publications.`

Il est possible que ce message d’erreur soit réellement dû au fait qu’il n’existe aucune publication sur le serveur de publication auquel vous êtes connecté ou au fait que vous ne disposez pas d’autorisations suffisantes. Toutefois, cette erreur peut également être causée par une ancienne version de SQL Server Management Studio. Essayez d’effectuer une mise à niveau vers SQL Server Management Studio 18.0 ou vers une version ultérieure pour être sûr que la cause racine n’est pas liée à la version utilisée.

## <a name="next-steps"></a>Étapes suivantes

### <a name="enable-security-features"></a>Activer les fonctionnalités de sécurité

Consultez l’article [Qu’est-ce qu’Azure SQL Managed Instance ?](sql-managed-instance-paas-overview.md#advanced-security-and-compliance) pour obtenir la liste complète des moyens de sécuriser votre base de données. Les fonctionnalités de sécurité suivantes sont présentées :

- [Audit d’une instance managée SQL](auditing-configure.md)
- [Always Encrypted](/sql/relational-databases/security/encryption/always-encrypted-database-engine)
- [Détection de menaces](threat-detection-configure.md)
- [Masquage des données dynamiques](/sql/relational-databases/security/dynamic-data-masking)
- [Sécurité au niveau des lignes](/sql/relational-databases/security/row-level-security)
- [Chiffrement transparent des données (TDE)](/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql)

### <a name="sql-managed-instance-capabilities"></a>Fonctionnalités de SQL Managed Instance

Pour obtenir une vue d’ensemble des fonctionnalités d’une instance managée, consultez :

> [!div class="nextstepaction"]
> [Fonctionnalités de SQL Managed Instance](sql-managed-instance-paas-overview.md)