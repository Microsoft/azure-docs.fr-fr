---
title: Gérer les packages avec le magasin de packages Azure-SSIS Integration Runtime
description: Découvrez comment gérer des packages avec le magasin de packages Azure-SSIS Integration Runtime.
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
author: swinarko
ms.author: sawinark
ms.reviewer: douglasl
manager: mflasko
ms.custom: seo-lt-2019
ms.date: 09/29/2020
ms.openlocfilehash: 29d072c513d9a75055d4bb486f44b17b00b7f0a9
ms.sourcegitcommit: fb3c846de147cc2e3515cd8219d8c84790e3a442
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92638344"
---
# <a name="manage-packages-with-azure-ssis-integration-runtime-package-store"></a>Gérer les packages avec le magasin de packages Azure-SSIS Integration Runtime

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Pour promouvoir et insérer des charges de travail SQL Server Integration Services (SSIS) locales dans le cloud, vous pouvez configurer Azure-SSIS Integration Runtime (IR) dans Azure Data Factory (ADF). Pour plus d’informations, consultez [Configurer un IR Azure-SSIS](./tutorial-deploy-ssis-packages-azure.md). Un IR Azure-SSIS prend en charge :

- L’exécution de packages déployés dans le catalogue SSIS (SSISDB) hébergé par un serveur Azure SQL Database/Managed Instance (modèle de déploiement de projet)
- L’exécution de packages déployés dans le système de fichiers, Azure Files ou une base de données SQL Server (MSDB) hébergée par Azure SQL Managed Instance (modèle de déploiement de package)

Lorsque vous utilisez un modèle de déploiement de package, vous pouvez choisir si vous souhaitez approvisionner votre runtime d’intégration Azure-SSIS avec des magasins de packages. Ils fournissent une couche de gestion des packages par-dessus le système de fichiers, Azure Files ou msdb hébergé par Azure SQL Managed Instance. Le magasin de packages Azure-SSIS IR vous permet d’importer/exporter/supprimer/exécuter des packages et superviser/arrêter l’exécution de packages par le biais de SQL Server Management Studio (SSMS), à l’image du [magasin de packages SSIS hérités](/sql/integration-services/service/package-management-ssis-service?view=sql-server-2017). 

## <a name="connect-to-azure-ssis-ir"></a>Se connecter à l’IR Azure-SSIS

Une fois votre IR Azure-SSIS configuré, vous pouvez vous y connecter et parcourir ses magasins de packages sur SSMS.

![Se connecter à l’IR Azure-SSIS](media/azure-ssis-integration-runtime-package-store/ssms-package-store-connect.png)

Dans la fenêtre **Explorateur d’objets** de SSMS, sélectionnez **Azure-SSIS Integration Runtime** dans le menu déroulant **Se connecter** . Connectez-vous ensuite à Azure et sélectionnez l’abonnement, l’ADF et le runtime d’intégration Azure-SSIS appropriés que vous avez approvisionnés avec des magasins de packages. Votre IR Azure-SSIS s’affiche au-dessus des nœuds **Exécution des packages** et **Packages stockés** . Développez le nœud **Packages stockés** pour afficher vos magasins de packages sous-jacents. Développez vos magasins de packages pour afficher les dossiers et les packages sous-jacents. Vous pouvez être invité à entrer les informations d’identification d’accès pour vos magasins de packages, si SSMS ne parvient pas à s’y connecter automatiquement. Par exemple, si vous développez un magasin de packages en plus de MSDB, vous pouvez être invité à vous connecter d’abord à votre Managed Instance Azure SQL.

![Se connecter à Azure SQL Managed Instance](media/azure-ssis-integration-runtime-package-store/ssms-package-store-connect2.png)

## <a name="manage-folders-and-packages"></a>Gérer des dossiers et des packages

Après vous être connecté à votre runtime d’intégration Azure-SSIS sur SSMS, vous pouvez cliquer avec le bouton droit sur n’importe quel magasin de packages, dossier ou package pour faire apparaître un menu et sélectionner **Nouveau dossier** , **Importer un package** , **Exporter un package** , **Supprimer** ou **Actualiser** .

   ![Gérer des dossiers et des packages](media/azure-ssis-integration-runtime-package-store/ssms-package-store-manage.png)

   *  Sélectionnez **Nouveau dossier** afin de créer un dossier pour les packages importés.

   *  Sélectionnez **Importer un package** pour importer dans votre magasin de packages des packages provenant du **système de fichiers** , de **SQL Server** (MSDB) ou du **magasin de packages SSIS** hérité.

      ![Importer un package](media/azure-ssis-integration-runtime-package-store/ssms-package-store-import.png)

      Selon **l’emplacement du package** d’importation, sélectionnez **Serveur**/**Type d’authentification** , entrez les informations d’identification si nécessaire, spécifiez le **chemin du package** , puis entrez le nouveau **nom du package** . Lorsque vous importez des packages, leur niveau de protection ne peut pas être modifié. Pour le modifier, utilisez SQL Server Data Tools (SSDT) ou l’utilitaire de ligne de commande `dtutil`.

      > [!NOTE]
      > L’importation de packages SSIS dans les magasins de packages Azure-SSIS IR peut uniquement s’effectuer un par un et les copiera simplement dans le système de fichiers/Azure Files/la msdb sous-jacents tout en conservant leur version SQL Server/SSIS. 
      >
      > Étant donné qu’Azure-SSIS IR est actuellement basé sur **SQL Server 2017** , l’exécution de packages de version antérieure sur le runtime d’intégration les met à niveau vers des packages SSIS 2017 au moment de l’exécution. L’exécution de packages de version ultérieure n’est pas prise en charge.
      >
      > En outre, étant donné que les magasins de packages SSIS hérités sont liés à une version spécifique de SQL Server et accessibles uniquement sur SSMS pour cette version, les packages de version antérieure dans les magasins de packages SSIS hérités doivent d’abord être exportés vers le système de fichiers à l’aide de la version désignée de SSMS avant de pouvoir être importés dans les magasins de packages Azure-SSIS IR à l’aide de SSMS 2019 ou versions ultérieures.
      >
      > Sinon, pour importer plusieurs packages SSIS dans les magasins de packages Azure-SSIS IR tout en basculant leur niveau de protection, vous pouvez utiliser l’utilitaire de ligne de commande [dtutil](/sql/integration-services/dtutil-utility?view=sql-server-2017). Voir [Déploiement de plusieurs packages à l’aide de dtutil](#deploying-multiple-packages-with-dtutil).

   *  Sélectionnez **Exporter un package** pour exporter des packages de votre magasin de packages vers le **système de fichiers** , **SQL Server** (MSDB) ou le **magasin de packages SSIS** hérité.

      ![Exporter un package](media/azure-ssis-integration-runtime-package-store/ssms-package-store-export.png)

      Selon **l’emplacement du package** d’exportation, sélectionnez **Serveur**/**Type d’authentification** , entrez les informations d'identification si nécessaire, puis sélectionnez le **chemin du package** . Lorsque vous exportez des packages, s’ils sont chiffrés, entrez d’abord les mots de passe pour les déchiffrer, puis modifiez le cas échéant leur niveau de protection, par exemple, pour ne pas stocker de données sensibles ou pour chiffrer ces dernières ou toutes les données avec la clé ou le mot de passe de l’utilisateur.

      > [!NOTE]
      > L’exportation de packages SSIS à partir de magasins de packages Azure-SSIS IR peut uniquement s’effectuer un par un. Le faire sans changer leur niveau de protection ne fera que les copier tout en conservant leur version SQL Server/SSIS. Sinon, cela les mettra à niveau vers des packages SSIS 2019 ou versions ultérieures.
      >
      > Étant donné qu’Azure-SSIS IR est actuellement basé sur **SQL Server 2017** , l’exécution de packages de version antérieure sur le runtime d’intégration les met à niveau vers des packages SSIS 2017 au moment de l’exécution. L’exécution de packages de version ultérieure n’est pas prise en charge.
      >
      > Sinon, pour exporter plusieurs packages SSIS à partir des magasins de packages Azure-SSIS IR tout en basculant leur niveau de protection, vous pouvez utiliser l’utilitaire de ligne de commande [dtutil](/sql/integration-services/dtutil-utility?view=sql-server-2017). Voir [Déploiement de plusieurs packages à l’aide de dtutil](#deploying-multiple-packages-with-dtutil).

   *  Sélectionnez **Supprimer** pour supprimer les dossiers/packages existants de votre magasin de packages.

   *  Sélectionnez **Actualiser** pour afficher les dossiers/packages récemment ajoutés à votre magasin de packages.

## <a name="execute-packages"></a>Exécuter des packages

Après vous être connecté à votre runtime d’intégration Azure-SSIS sur SSMS, vous pouvez cliquer avec le bouton droit sur n’importe quel package stocké pour afficher un menu et sélectionner **Exécuter le package** .  Cette opération ouvre la boîte de dialogue **Utilitaire d’exécution de package** dans laquelle vous pouvez configurer les exécutions de packages sur l’IR Azure-SSIS en tant qu’activités Exécuter le Package SSIS dans des pipelines ADF.

![Utilitaire d'exécution de package, pages 1 et 2](media/azure-ssis-integration-runtime-package-store/ssms-package-store-execute.png)

![Utilitaire d'exécution de package, pages 3 et 4](media/azure-ssis-integration-runtime-package-store/ssms-package-store-execute2.png)

Les pages **Général** , **Configurations** , **Options d’exécution** et **Journalisation** de la boîte de dialogue **Utilitaire d’exécution de package** correspondent à l’onglet **Paramètres** de l’activité Exécuter le package SSIS. Dans ces pages, vous pouvez entrer le mot de passe de chiffrement de votre package et accéder aux informations du fichier config de votre package. Vous pouvez également entrer les informations d’identification et les propriétés d’exécution de votre package, ainsi que les informations d’accès à votre dossier de journal.  La page **Valeurs définies** de la boîte de dialogue **Utilitaire d'exécution de package** correspond à l’onglet **Substitutions de propriété** de l’activité Execute SSIS Package, où vous pouvez entrer les propriétés de votre package existant à substituer. Pour plus d’informations, consultez [Exécuter des packages SSIS avec des activités Exécuter le Package SSIS dans des pipelines ADF](./how-to-invoke-ssis-package-ssis-activity.md).

Lorsque vous sélectionnez le bouton **Exécuter** , un nouveau pipeline ADF avec l’activité Exécuter le package SSIS sera automatiquement généré puis déclenché. Si un pipeline ADF avec les mêmes paramètres existe déjà, il sera réexécuté et aucun nouveau pipeline ne sera généré. Le pipeline ADF et l’activité Exécuter le Package SSIS seront nommés `Pipeline_SSMS_YourPackageName_HashString` et `Activity_SSMS_YourPackageName`, respectivement.

![Utilitaire d'exécution de package, bouton](media/azure-ssis-integration-runtime-package-store/ssms-package-store-execute3.png)

![Activité Exécuter le package SSIS](media/azure-ssis-integration-runtime-package-store/ssis-activity-package-store.png)

## <a name="monitor-and-stop-running-packages"></a>Analyser et arrêter des packages en cours d'exécution

Après vous être connecté à votre runtime d’intégration Azure-SSIS sur SSMS, vous pouvez développer le nœud **Exécution des packages** pour afficher les packages en cours d’exécution sous-jacents.  Cliquez avec le bouton droit sur l’un d’eux pour afficher un menu, puis sélectionnez **Arrêter** ou **Actualiser** .

   ![Analyser et arrêter des packages en cours d'exécution](media/azure-ssis-integration-runtime-package-store/ssms-package-store-monitor.png)

   *  Sélectionnez **Arrêter** pour annuler le pipeline ADF en cours d’exécution qui exécute le package en tant qu’activité Exécuter le Package SSIS.

   *  Sélectionnez **Actualiser** pour afficher les packages qui viennent d’être exécutés à partir de vos magasins de packages.

## <a name="monitor-azure-ssis-ir-and-edit-package-stores"></a>Analyser l’IR Azure-SSIS et modifier des magasins de packages

Après vous être connecté à votre runtime d’intégration Azure-SSIS sur SSMS, vous pouvez cliquer dessus avec le bouton droit pour afficher un menu et sélectionner **Accéder au portail Azure Data Factory** ou **Actualiser** .

   ![Accéder au portail ADF](media/azure-ssis-integration-runtime-package-store/ssms-package-store-monitor2.png)

   *  Sélectionnez **Accédez au portail Azure Data Factory** pour ouvrir la page **Runtimes d’intégration** du hub d’analyse ADF, où vous pouvez analyser votre IR Azure-SSIS. Dans la vignette **Magasins de packages** , vous pouvez voir le nombre de magasins de packages attachés à votre IR Azure-SSIS.  Si vous sélectionnez ce nombre, une fenêtre s’affiche dans laquelle vous pouvez modifier les services liés ADF qui stockent les informations d’accès de vos magasins de packages.

      ![Modifier des magasins de packages](media/azure-ssis-integration-runtime-package-store/ssms-package-store-monitor3.png)

   *  Sélectionnez **Actualiser** pour afficher les dossiers/packages récemment ajoutés dans vos magasins de packages et les packages en cours d’exécution dans vos magasins de packages.

## <a name="deploying-multiple-packages-with-dtutil"></a>Déploiement de plusieurs packages à l’aide de dtutil

Pour élever et déplacer vos charges de travail SSIS locales vers SSIS dans ADF tout en conservant le modèle de déploiement de package hérité, vous devez déployer vos packages à partir du système de fichiers, de la msdb hébergée par SQL Server ou de magasins de packages SSIS hérités dans Azure Files, la msdb hébergée par Azure SQL Managed Instance ou les magasins de packages Azure-SSIS IR. Dans le même temps et si vous ne l’avez pas déjà fait, vous devez également changer leur niveau de protection du chiffrement par clé utilisateur par un chiffrement en clair ou un chiffrement par mot de passe.

Vous pouvez utiliser l’utilitaire de ligne de commande [dtutil](/sql/integration-services/dtutil-utility?view=sql-server-2017) fourni avec l’installation de SQL Server/SSIS pour déployer plusieurs packages par lots. Il est lié à une version spécifique de SSIS. Par conséquent, si vous l’utilisez pour déployer des packages de version antérieure sans changer leur niveau de protection, il les copie simplement tout en conservant leur version SSIS. Si vous l’utilisez pour les déployer et changez leur niveau de protection en même temps, il les mettra à niveau vers sa version SSIS.

 Étant donné qu’Azure-SSIS IR est actuellement basé sur **SQL Server 2017** , l’exécution de packages de version antérieure sur le runtime d’intégration les met à niveau vers des packages SSIS 2017 au moment de l’exécution. L’exécution de packages de version ultérieure n’est pas prise en charge.

Ainsi, pour éviter les mises à niveau au moment de l’exécution, le déploiement de packages à exécuter sur Azure-SSIS IR dans le modèle de déploiement de package doit utiliser dtutil 2017 fourni avec l’installation de SQL Server/SSIS 2017. Pour ce faire, vous pouvez télécharger et installer gratuitement [SQL Server/SSIS 2017 Developer Edition](https://go.microsoft.com/fwlink/?linkid=853016). Une fois l’installation terminée, vous pouvez trouver dtutil 2017 sous ce dossier : `YourLocalDrive:\Program Files\Microsoft SQL Server\140\DTS\Binn`.

### <a name="deploying-multiple-packages-from-file-system-on-premises-into-azure-files-with-dtutil"></a>Déploiement de plusieurs packages dans Azure Files à partir du système de fichiers local à l’aide de dtutil

 Pour déployer plusieurs packages du système de fichiers vers Azure Files et changer leur niveau de protection dans le même temps, vous pouvez exécuter les commandes suivantes à partir d’une invite de commandes. Remplacez toutes les chaînes qui sont spécifiques à votre cas.
  
```dos
REM Persist the access credentials for Azure Files on your local machine
cmdkey /ADD:YourStorageAccountName.file.core.windows.net /USER:azure\YourStorageAccountName /PASS:YourStorageAccountKey

REM Connect Azure Files to a drive on your local machine
net use Z: \\YourStorageAccountName.file.core.windows.net\YourFileShare /PERSISTENT:Yes

REM Go to a local folder where you store your packages
cd YourLocalDrive:\...\YourPackageFolder

REM Run dtutil in a loop to deploy your packages from the local folder into Azure Files while switching their protection level
for %f in (*.dtsx) do dtutil.exe /FILE %f /ENCRYPT FILE;Z:\%f;2;YourEncryptionPassword
```

Pour exécuter les commandes ci-dessus dans un fichier Batch, remplacez `%f` par `%%f`.

Pour déployer plusieurs packages à partir de magasins de packages SSIS hérités par-dessus le système de fichiers dans Azure Files et basculer leur niveau de protection dans le même temps, vous pouvez utiliser les mêmes commandes, mais remplacez `YourLocalDrive:\...\YourPackageFolder` par un dossier local utilisé par les magasins de packages SSIS hérités : `YourLocalDrive:\Program Files\Microsoft SQL Server\YourSQLServerDefaultCompatibilityLevel\DTS\Packages\YourPackageFolder`. Par exemple, si votre magasin de packages SSIS hérité est lié à SQL Server 2016, accédez à `YourLocalDrive:\Program Files\Microsoft SQL Server\130\DTS\Packages\YourPackageFolder`.  Vous pouvez trouver la valeur de `YourSQLServerDefaultCompatibilityLevel` dans une [liste des niveaux de compatibilité par défaut de SQL Server](/sql/t-sql/statements/alter-database-transact-sql-compatibility-level?view=sql-server-ver15#arguments).

Si vous avez configuré des magasins de packages Azure-SSIS IR par-dessus Azure Files, vos packages déployés s’y affichent lorsque vous vous connectez à votre runtime d’intégration Azure-SSIS sur SSMS 2019 ou versions ultérieures.

### <a name="deploying-multiple-packages-from-msdb-on-premises-into-msdb-in-azure-with-dtutil"></a>Déploiement de plusieurs packages d’une msdb locale vers une msdb dans Azure à l’aide de dtutil

 Pour déployer plusieurs packages à partir d’une msdb hébergée par SQL Server ou des magasins de packages SSIS hérités par-dessus une msdb vers une msdb hébergée par Azure SQL Managed Instance et changer leur niveau de protection dans le même temps, vous pouvez vous connecter à votre serveur SQL sur SSMS, cliquer avec le bouton droit sur le nœud `Databases->System Databases->msdb` dans **Explorateur d’objets** de SSMS pour ouvrir une fenêtre **Nouvelle requête** et exécuter le script T-SQL suivant. Remplacez toutes les chaînes qui sont spécifiques à votre cas :  
  
```sql
BEGIN
  SELECT 'dtutil /SQL '+f.foldername+'\'+NAME+' /ENCRYPT SQL;'+f.foldername+'\'+NAME+';2;YourEncryptionPassword /DestServer YourSQLManagedInstanceEndpoint /DestUser YourSQLAuthUsername /DestPassword YourSQLAuthPassword'
  FROM msdb.dbo.sysssispackages p
  inner join msdb.dbo.sysssispackagefolders f
  ON p.folderid = f.folderid
END
```

Pour utiliser le point de terminaison privé/public de votre instance managée Azure SQL, remplacez `YourSQLManagedInstanceEndpoint` par `YourSQLMIName.YourDNSPrefix.database.windows.net`/`YourSQLMIName.public.YourDNSPrefix.database.windows.net,3342`, respectivement.

Le script génère des lignes de commande dtutil pour tous les packages de la msdb que vous pouvez sélectionner, copier-coller et exécuter à partir d’une invite de commandes.

![Générer des lignes de commande dtutil](media/azure-ssis-integration-runtime-package-store/sql-server-msdb-to-sql-mi-msdb.png)

```dos
dtutil /SQL YourFolder\YourPackage1 /ENCRYPT SQL;YourFolder\YourPackage1;2;YourEncryptionPassword /DestServer YourSQLManagedInstanceEndpoint /DestUser YourUserName /DestPassword YourPassword
dtutil /SQL YourFolder\YourPackage2 /ENCRYPT SQL;YourFolder\YourPackage2;2;YourEncryptionPassword /DestServer YourSQLManagedInstanceEndpoint /DestUser YourUserName /DestPassword YourPassword
dtutil /SQL YourFolder\YourPackage3 /ENCRYPT SQL;YourFolder\YourPackage3;2;YourEncryptionPassword /DestServer YourSQLManagedInstanceEndpoint /DestUser YourUserName /DestPassword YourPassword
```

Si vous avez configuré des magasins de packages Azure-SSIS IR par-dessus la msdb, vos packages déployés s’y affichent lorsque vous vous connectez à votre runtime d’intégration Azure-SSIS sur SSMS 2019 ou versions ultérieures.

### <a name="deploying-multiple-packages-from-msdb-on-premises-into-azure-files-with-dtutil"></a>Déploiement de plusieurs packages dans Azure Files à partir de la msdb locale à l’aide de dtutil

 Pour déployer plusieurs packages à partir d’une msdb hébergée par SQL Server ou des magasins de packages SSIS hérités par-dessus une msdb vers Azure Files et changer leur niveau de protection dans le même temps, vous pouvez vous connecter à votre serveur SQL sur SSMS, cliquer avec le bouton droit sur le nœud `Databases->System Databases->msdb` dans **Explorateur d’objets** de SSMS pour ouvrir une fenêtre **Nouvelle requête** et exécuter le script T-SQL suivant. Remplacez toutes les chaînes qui sont spécifiques à votre cas :  
  
```sql
BEGIN
  SELECT 'dtutil /SQL '+f.foldername+'\'+NAME+' /ENCRYPT FILE;Z:\'+f.foldername+'\'+NAME+'.dtsx;2;YourEncryptionPassword' 
  FROM msdb.dbo.sysssispackages p
  inner join msdb.dbo.sysssispackagefolders f
  ON p.folderid = f.folderid
END
```

Le script génère des lignes de commande dtutil pour tous les packages de la msdb que vous pouvez sélectionner, copier-coller et exécuter à partir d’une invite de commandes.

```dos
REM Persist the access credentials for Azure Files on your local machine
cmdkey /ADD:YourStorageAccountName.file.core.windows.net /USER:azure\YourStorageAccountName /PASS:YourStorageAccountKey

REM Connect Azure Files to a drive on your local machine
net use Z: \\YourStorageAccountName.file.core.windows.net\YourFileShare /PERSISTENT:Yes

REM Multiselect, copy & paste, and run the T-SQL-generated dtutil command lines to deploy your packages from MSDB on premises into Azure Files while switching their protection level
dtutil /SQL YourFolder\YourPackage1 /ENCRYPT FILE;Z:\YourFolder\YourPackage1.dtsx;2;YourEncryptionPassword
dtutil /SQL YourFolder\YourPackage2 /ENCRYPT FILE;Z:\YourFolder\YourPackage2.dtsx;2;YourEncryptionPassword
dtutil /SQL YourFolder\YourPackage3 /ENCRYPT FILE;Z:\YourFolder\YourPackage3.dtsx;2;YourEncryptionPassword
```

Si vous avez configuré des magasins de packages Azure-SSIS IR par-dessus Azure Files, vos packages déployés s’y affichent lorsque vous vous connectez à votre runtime d’intégration Azure-SSIS sur SSMS 2019 ou versions ultérieures.

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez réexécuter/modifier les pipelines ADF générés automatiquement avec des activités Exécuter le Package SSIS, ou en créer d’autres sur le portail ADF. Pour plus d’informations, consultez [Exécuter des packages SSIS avec des activités Exécuter le Package SSIS dans des pipelines ADF](./how-to-invoke-ssis-package-ssis-activity.md).