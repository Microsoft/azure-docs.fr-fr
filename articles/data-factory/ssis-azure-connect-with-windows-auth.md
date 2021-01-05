---
title: Accéder à des magasins de données et à des partages de fichiers avec l’Authentification Windows
description: Découvrez comment configurer le catalogue SSIS dans Azure SQL Database et Azure-SSIS Integration Runtime dans Azure Data Factory pour exécuter des packages qui accèdent à des magasins de données et à des partages de fichiers avec l’authentification Windows.
ms.date: 10/27/2020
ms.topic: conceptual
ms.prod: sql
ms.prod_service: integration-services
ms.custom: ''
ms.technology: integration-services
author: swinarko
ms.author: sawinark
ms.reviewer: maghan
ms.openlocfilehash: 545f698f444e99d3f3807f22b308963172018fcb
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92746667"
---
# <a name="access-data-stores-and-file-shares-with-windows-authentication-from-ssis-packages-in-azure"></a>Accéder à des magasins de données et à des partages de fichiers avec l’authentification Windows à partir de packages SSIS sur Azure

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Vous pouvez utiliser l’authentification Windows pour accéder à des magasins de données, par exemple des serveurs SQL, des partages de fichiers, Azure Files, etc. à partir de packages SSIS qui s’exécutent sur Azure-SSIS Integration Runtime (IR) dans Azure Data Factory (ADF). Il peut s’agir de magasins de données sur site, hébergés sur les Machines virtuelles Azure ou qui s’exécutent dans Azure en tant que services gérés. S’ils sont en local, vous devez joindre votre runtime Azure-SSIS IR à un Réseau virtuel (Réseau virtuel Microsoft Azure) connecté à votre réseau local : voir [Joindre un runtime Azure-SSIS IR à un Réseau virtuel Microsoft Azure](./join-azure-ssis-integration-runtime-virtual-network.md). Il existe quatre moyens d’accéder à des magasins de données avec l’authentification Windows à partir de packages SSIS qui s’exécutent sur un runtime Azure-SSIS IR :

| Méthode de connexion | Portée effective | Étape de configuration | Méthode d’accès dans des packages | Nombre de jeux d’informations d’identification et ressources connectées | Type de ressources connectées | 
|---|---|---|---|---|---|
| Configurer un contexte d’exécution au niveau de l’activité | Selon l’activité Exécuter un package SSIS | Configurez la propriété **Authentification Windows** pour définir un contexte « Exécution/Exécuter en tant que » pour l’exécution de packages SSIS en tant qu’activités Exécuter un package SSIS dans des pipelines ADF.<br/><br/> Pour plus d’informations, voir [Configurer l’activité Exécuter le package SSIS](./how-to-invoke-ssis-package-ssis-activity.md). | Accédez aux ressources directement dans les packages. Par exemple, utilisez un chemin d’accès UNC pour accéder aux partages de fichiers ou au service Azure Files : `\\YourFileShareServerName\YourFolderName` ou `\\YourAzureStorageAccountName.file.core.windows.net\YourFolderName`. | Prise en charge d’un seul jeu d’informations d’identification pour toutes les ressources connectées | - Partages de fichiers localement/sur des machines virtuelles Azure<br/><br/> - Azure Files, voir [Utiliser un partage de fichiers Azure](../storage/files/storage-how-to-use-files-windows.md)<br/><br/> - Serveurs SQL sur site/Machines virtuelles Azure avec authentification Windows<br/><br/> - Autres ressources avec authentification Windows |
| Configurer un contexte d’exécution au niveau du catalogue | Selon Azure-SSIS IR, mais est remplacé lors la configuration d’un contexte d’exécution au niveau de l’activité (voir plus haut) | Exécutez la procédure stockée `catalog.set_execution_credential` SSISDB pour définir un contexte « Exécution/Exécuter en tant que ».<br/><br/> Pour plus d’informations, voir la suite de cet article. | Accédez aux ressources directement dans les packages. Par exemple, utilisez un chemin d’accès UNC pour accéder aux partages de fichiers ou au service Azure Files : `\\YourFileShareServerName\YourFolderName` ou `\\YourAzureStorageAccountName.file.core.windows.net\YourFolderName`. | Prise en charge d’un seul jeu d’informations d’identification pour toutes les ressources connectées | - Partages de fichiers localement/sur des machines virtuelles Azure<br/><br/> - Azure Files, voir [Utiliser un partage de fichiers Azure](../storage/files/storage-how-to-use-files-windows.md)<br/><br/> - Serveurs SQL sur site/Machines virtuelles Azure avec authentification Windows<br/><br/> - Autres ressources avec authentification Windows |
| Conserver les informations d’identification avec la commande `cmdkey` | Selon Azure-SSIS IR, mais est remplacé lors la configuration d’un contexte d’exécution au niveau de l’activité ou du catalogue (voir plus haut) | Exécutez la commande `cmdkey` dans un script d’installation personnalisé (`main.cmd`) lors de l’approvisionnement de votre Azure-SSIS IR, par exemple, si vous utilisez des partages de fichiers, Azure Files ou SQL Server :<br/><br/> `cmdkey /add:YourFileShareServerName /user:YourDomainName\YourUsername /pass:YourPassword`,<br/><br/> `cmdkey /add:YourAzureStorageAccountName.file.core.windows.net /user:azure\YourAzureStorageAccountName /pass:YourAccessKey` ou<br/><br/> `cmdkey /add:YourSQLServerFullyQualifiedDomainNameOrIPAddress:YorSQLServerPort /user:YourDomainName\YourUsername /pass:YourPassword`.<br/><br/> Pour plus d’informations, voir [Personnaliser l’installation du runtime Azure-SSIS IR](./how-to-configure-azure-ssis-ir-custom-setup.md). | Accédez aux ressources directement dans les packages. Par exemple, utilisez un chemin d’accès UNC pour accéder aux partages de fichiers ou au service Azure Files : `\\YourFileShareServerName\YourFolderName` ou `\\YourAzureStorageAccountName.file.core.windows.net\YourFolderName`. | Prise en charge de plusieurs jeux d’informations d’identification pour différentes ressources connectées | - Partages de fichiers localement/sur des machines virtuelles Azure<br/><br/> - Azure Files, voir [Utiliser un partage de fichiers Azure](../storage/files/storage-how-to-use-files-windows.md)<br/><br/> - Serveurs SQL sur site/Machines virtuelles Azure avec authentification Windows<br/><br/> - Autres ressources avec authentification Windows |
| Monter des lecteurs à l’exécution du package (non persistant) | Par package | Exécutez la commande `net use` dans la tâche Exécuter le processus ajoutée au début du flux de contrôle de vos packages, par exemple, `net use D: \\YourFileShareServerName\YourFolderName`. | Accédez aux partages de fichiers par le biais de lecteurs mappés. | Prise en charge de plusieurs lecteurs pour différents partages de fichiers | - Partages de fichiers localement/sur des machines virtuelles Azure<br/><br/> - Azure Files, voir [Utiliser un partage de fichiers Azure](../storage/files/storage-how-to-use-files-windows.md) |
|||||||

> [!WARNING]
> Si vous n’utilisez aucune des méthodes ci-dessus pour accéder à des magasins de données avec l’Authentification Windows, les packages qui dépendent de l’Authentification Windows ne peuvent pas y accéder et échouent au moment de l’exécution. 

La suite de cet article explique comment configurer le catalogue SSIS (SSISDB) hébergé dans une base de données SQL/une instance managée SQL pour exécuter des packages sur le runtime d’intégration Azure-SSIS utilisant l’authentification Windows pour accéder aux magasins de données. 

## <a name="you-can-only-use-one-set-of-credentials"></a>Vous ne pouvez utiliser qu’un seul jeu d’informations d’identification

Si vous utilisez l’authentification Windows dans un package SSIS, vous ne pouvez recourir qu’à un seul jeu d’informations d’identification. Tant que vous ne les changez et ne les supprimez pas, les informations d’identification de domaine que vous indiquez en suivant les étapes de cet article s’appliquent à toutes les exécutions de packages, interactives ou planifiées, sur votre runtime Azure-SSIS IR. Si votre package doit se connecter à plusieurs magasins de données avec différents jeux d’informations d’identification, tournez-vous vers les autres méthodes décrites plus haut.

## <a name="provide-domain-credentials-for-windows-authentication"></a>Fournir des informations d’identification de domaine pour l’authentification Windows

Suivez les étapes ci-dessous pour fournir des informations d’identification de domaine permettant aux packages d’utiliser l’authentification Windows afin d’accéder à des magasins de données sur site :

1. Avec SSMS (SQL Server Management Studio) ou un autre outil, connectez-vous à la base de données SQL/l’instance managée SQL qui héberge SSISDB. Pour plus d’informations, voir [Se connecter à SSISDB sur Azure](/sql/integration-services/lift-shift/ssis-azure-connect-to-catalog-database).

2. La base de données SSISDB étant la base de données active, ouvrez une fenêtre de requête.

3. Exécutez la procédure stockée suivante en indiquant les informations d’identification de domaine nécessaires :

   ```sql
   catalog.set_execution_credential @user='<your user name>', @domain='<your domain name>', @password='<your password>'
   ```

4. Exécutez vos packages SSIS. Les packages utilisent les informations d’identification fournies pour accéder aux magasins de données sur site avec l’Authentification Windows.

### <a name="view-domain-credentials"></a>Afficher les informations d’identification de domaine

Pour afficher les informations d’identification de domaine actives, effectuez les actions suivantes :

1. Avec SSMS ou un autre outil, connectez-vous à la base de données SQL/l’instance managée SQL qui héberge SSISDB. Pour plus d’informations, voir [Se connecter à SSISDB sur Azure](/sql/integration-services/lift-shift/ssis-azure-connect-to-catalog-database).

2. La base de données SSISDB étant la base de données active, ouvrez une fenêtre de requête.

3. Exécutez la procédure stockée suivante et vérifiez la sortie :

   ```sql
   SELECT * 
   FROM catalog.master_properties
   WHERE property_name = 'EXECUTION_DOMAIN' OR property_name = 'EXECUTION_USER'
   ```

### <a name="clear-domain-credentials"></a>Effacer les informations d’identification de domaine
Pour effacer et supprimer les informations d’identification que vous avez fournies, comme indiqué dans cet article, effectuez les actions suivantes :

1. Avec SSMS ou un autre outil, connectez-vous à la base de données SQL/l’instance managée SQL qui héberge SSISDB. Pour plus d’informations, voir [Se connecter à SSISDB sur Azure](/sql/integration-services/lift-shift/ssis-azure-connect-to-catalog-database).

2. La base de données SSISDB étant la base de données active, ouvrez une fenêtre de requête.

3. Exécutez la procédure stockée suivante :

   ```sql
   catalog.set_execution_credential @user='', @domain='', @password=''
   ```

## <a name="connect-to-a-sql-server-on-premises"></a>Se connecter à un serveur SQL Server sur site

Suivez les étapes ci-dessous pour vérifier que vous pouvez vous connecter à un serveur SQL Server sur site :

1. Pour exécuter ce test, recherchez un ordinateur non membre d’un domaine.

2. Sur l’ordinateur non membre d’un domaine, exécutez la commande suivante pour lancer SSMS avec les informations d’identification de domaine souhaitées :

   ```cmd
   runas.exe /netonly /user:<domain>\<username> SSMS.exe
   ```

3. À partir de SSMS, vérifiez que vous pouvez vous connecter au serveur SQL Server sur site.

### <a name="prerequisites"></a>Prérequis

Suivez les étapes ci-dessous pour accéder à un serveur SQL Server sur site à partir de packages qui s’exécutent sur Azure :

1.  Dans le Gestionnaire de configuration SQL Server, activez le protocole TCP/IP.

2. Autorisez l’accès à travers le Pare-feu Windows. Pour plus d’informations, voir [Configurer le Pare-feu Windows pour accéder à SQL Server](/sql/sql-server/install/configure-the-windows-firewall-to-allow-sql-server-access).

3. Joignez votre runtime Azure-SSIS IR à un Réseau virtuel Microsoft Azure connecté au SQL Server local.  Pour plus d’informations, consultez [Joindre le runtime Azure-SSIS IR à un Réseau virtuel Microsoft Azure](./join-azure-ssis-integration-runtime-virtual-network.md).

4. Utilisez la procédure stockée `catalog.set_execution_credential` SSISDB pour indiquer les informations d’identification comme le décrit cet article.

## <a name="connect-to-a-file-share-on-premises"></a>Se connecter à un partage de fichiers sur site

Suivez les étapes ci-dessous pour vérifier que vous pouvez vous connecter à un partage de fichiers sur site :

1. Pour exécuter ce test, recherchez un ordinateur non membre d’un domaine.

2. Sur l’ordinateur non membre d’un domaine, exécutez les commandes suivantes. Elles ouvrent une fenêtre d’invite de commandes avec les informations d’identification de domaine à utiliser, puis testent la connectivité au partage de fichiers sur site en récupérant une liste de répertoires.

   ```cmd
   runas.exe /netonly /user:<domain>\<username> cmd.exe
   dir \\fileshare
   ```

3. Vérifiez que la liste de répertoires est retournée pour le partage de fichiers sur site.

### <a name="prerequisites"></a>Prérequis

Suivez les étapes ci-dessous pour accéder à un partage de fichiers sur site à partir de packages qui s’exécutent sur Azure :

1. Autorisez l’accès à travers le Pare-feu Windows.

2. Joignez votre runtime Azure-SSIS IR à un Réseau virtuel Microsoft Azure connecté au partage de fichiers local.  Pour plus d’informations, consultez [Joindre le runtime Azure-SSIS IR à un Réseau virtuel Microsoft Azure](./join-azure-ssis-integration-runtime-virtual-network.md).

3. Utilisez la procédure stockée `catalog.set_execution_credential` SSISDB pour indiquer les informations d’identification comme le décrit cet article.

## <a name="connect-to-a-file-share-on-azure-vm"></a>Se connecter à un partage de fichiers sur les Machines virtuelles Azure

Suivez les étapes ci-dessous pour accéder à un partage de fichiers sur les Machines virtuelles Azure à partir de packages qui s’exécutent sur Azure :

1. Avec SSMS ou un autre outil, connectez-vous à la base de données SQL/l’instance managée SQL qui héberge SSISDB. Pour plus d’informations, voir [Se connecter à SSISDB sur Azure](/sql/integration-services/lift-shift/ssis-azure-connect-to-catalog-database).

2. La base de données SSISDB étant la base de données active, ouvrez une fenêtre de requête.

3. Exécutez la procédure stockée suivante en indiquant les informations d’identification de domaine nécessaires :

   ```sql
   catalog.set_execution_credential @domain = N'.', @user = N'username of local account on Azure virtual machine', @password = N'password'
   ```

## <a name="connect-to-a-file-share-in-azure-files"></a>Se connecter à un partage de fichiers dans Azure Files

Pour plus d’informations sur Azure Files, consultez [Azure Files](https://azure.microsoft.com/services/storage/files/).

Suivez les étapes ci-dessous pour accéder à un partage de fichiers sur Azure Files à partir de packages qui s’exécutent sur Azure :

1. Avec SSMS ou un autre outil, connectez-vous à la base de données SQL/l’instance managée SQL qui héberge SSISDB. Pour plus d’informations, voir [Se connecter à SSISDB sur Azure](/sql/integration-services/lift-shift/ssis-azure-connect-to-catalog-database).

2. La base de données SSISDB étant la base de données active, ouvrez une fenêtre de requête.

3. Exécutez la procédure stockée suivante en indiquant les informations d’identification de domaine nécessaires :

   ```sql
   catalog.set_execution_credential @domain = N'Azure', @user = N'<storage-account-name>', @password = N'<storage-account-key>'
   ```

## <a name="next-steps"></a>Étapes suivantes

- Déployez vos packages. Pour plus d’informations, voir [Déployer un projet SSIS dans Azure avec SSMS](/sql/integration-services/ssis-quickstart-deploy-ssms).
- Exécutez vos packages. Pour plus d’informations, voir [Exécuter des packages SSIS sur Azure avec SSMS](/sql/integration-services/ssis-quickstart-run-ssms).
- Planifiez vos packages. Pour plus d’informations, consultez [Planifier des packages SSIS dans Azure](/sql/integration-services/lift-shift/ssis-azure-schedule-packages-ssms?view=sql-server-ver15).
