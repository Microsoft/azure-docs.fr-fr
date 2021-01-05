---
title: Provisionner le runtime d’intégration Azure-SSIS
description: Découvrez comment provisionner le runtime d’intégration Azure-SSIS dans Azure Data Factory afin de pouvoir déployer et exécuter des packages SSIS dans Azure.
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: ''
ms.devlang: ''
ms.topic: tutorial
ms.custom: seo-lt-2019
ms.date: 10/13/2020
author: swinarko
ms.author: sawinark
ms.reviewer: douglasl
manager: mflasko
ms.openlocfilehash: 33f3f599eaf95852b52b5bd3301e44316d18cce5
ms.sourcegitcommit: fb3c846de147cc2e3515cd8219d8c84790e3a442
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92637018"
---
# <a name="provision-the-azure-ssis-integration-runtime-in-azure-data-factory"></a>Provisionner le runtime d’intégration Azure-SSIS dans Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Ce tutoriel décrit les différentes étapes d’utilisation du portail Azure pour provisionner un runtime d’intégration (IR) Azure-SSIS (SQL Server Integration Services) dans Azure Data Factory. Un IR Azure-SSIS prend en charge :

- L’exécution de packages déployés dans le catalogue SSIS (SSISDB) hébergé par un serveur Azure SQL Database/Managed Instance (modèle de déploiement de projet)
- L’exécution de packages déployés dans le système de fichiers, Azure Files ou une base de données SQL Server (MSDB) hébergée par Azure SQL Managed Instance (modèle de déploiement de package)

Une fois qu’un IR Azure-SSIS est provisionné, vous pouvez utiliser des outils familiers pour déployer et exécuter vos packages dans Azure. Ces outils sont déjà compatibles avec Azure et incluent SQL Server Data Tools (SSDT), SQL Server Management Studio (SSMS) et des utilitaires en ligne de commande comme [dtutil](/sql/integration-services/dtutil-utility?view=sql-server-2017) et [AzureDTExec](./how-to-invoke-ssis-package-azure-enabled-dtexec.md).

Pour obtenir des informations conceptuelles sur les runtimes d’intégration (IR) Azure-SSIS, consultez [Vue d’ensemble du runtime d’intégration Azure-SSIS](concepts-integration-runtime.md#azure-ssis-integration-runtime).

Dans ce tutoriel, vous allez effectuer les étapes suivantes :

> [!div class="checklist"]
> * Créer une fabrique de données.
> * Approvisionner un runtime d’intégration Azure-SSIS.

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

- **Abonnement Azure** . Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

- **Serveur SQL Database (facultatif)** . Si vous n’avez pas encore de serveur de base de données, créez-en un dans le portail Azure avant de commencer. Data Factory crée à son tour une instance SSISDB sur ce serveur de base de données. 

  Nous vous recommandons de créer le serveur de base de données dans la même région Azure que le runtime d’intégration. Cette configuration permet au runtime d’intégration d’écrire des journaux des exécutions dans SSISDB sans dépasser les régions Azure.

  Gardez à l’esprit les points suivants :

  - En fonction du serveur de base de données sélectionné, l’instance SSISDB peut être créée à pour vous en tant que base de données unique, dans le cadre d’un pool élastique ou dans une instance managée. Elle peut être accessible dans un réseau public ou en la joignant à un réseau virtuel. Pour obtenir des conseils sur le choix du type de serveur de base de données pour héberger SSISDB, consultez [Comparer SQL Database et SQL Managed Instance](../data-factory/create-azure-ssis-integration-runtime.md#comparison-of-sql-database-and-sql-managed-instance). 
  
    Si vous utilisez un serveur Azure SQL Database avec des règles de pare-feu IP/points de terminaison de service de réseau virtuel ou une instance managée avec un point de terminaison privé pour héberger le catalogue SSISDB, ou si vous avez besoin d’accéder à des données locales sans configurer un runtime d’intégration (IR) auto-hébergé, vous devez joindre votre Azure-SSIS IR à un réseau virtuel. Pour plus d’informations, consultez [Créer un runtime Azure-SSIS IR dans un réseau virtuel](./create-azure-ssis-integration-runtime.md).

  - Vérifiez que le paramètre **Autoriser l’accès aux services Azure** est activé pour votre serveur de base de données. Ce paramètre ne s’applique pas quand vous utilisez un serveur Azure SQL Database avec des règles de pare-feu IP/points de terminaison de service de réseau virtuel ou une instance managée avec un point de terminaison privé pour héberger le catalogue SSISDB. Pour plus d’informations, consultez [Sécuriser Azure SQL Database](../azure-sql/database/secure-database-tutorial.md#create-firewall-rules). Pour savoir comment activer ce paramètre à l’aide de PowerShell, consultez [New-AzSqlServerFirewallRule](/powershell/module/az.sql/new-azsqlserverfirewallrule).

  - Ajoutez l’adresse IP de l’ordinateur client ou une plage d’adresses IP qui inclut l’adresse IP de l’ordinateur client à la liste d’adresses IP client dans les paramètres de pare-feu du serveur de base de données. Pour plus d’informations, consultez [Règles de pare-feu au niveau du serveur et de la base de données Azure SQL Database](../azure-sql/database/firewall-configure.md).

  - Vous pouvez vous connecter au serveur de base de données à l’aide de l’authentification SQL avec vos informations d’identification d’administrateur de serveur ou de l’authentification Azure AD avec l’identité managée de votre fabrique de données. Pour cette dernière, vous devez ajouter l’identité managée de votre fabrique de données à un groupe Azure AD avec autorisations d’accès au serveur de base de données. Pour plus d’informations, consultez [Créer un runtime Azure-SSIS IR avec l’authentification AAD](./create-azure-ssis-integration-runtime.md).

  - Vérifiez que votre serveur de base de données ne dispose pas déjà d’une instance SSISDB. Le provisionnement d’un IR Azure-SSIS ne prend pas en charge l’utilisation d’une instance SSISDB existante.

> [!NOTE]
> Pour obtenir la liste des régions Azure dans lesquelles Data Factory et un runtime Azure-SSIS IR sont actuellement disponibles, consultez [Disponibilité d’Azure Data Factory et du runtime Azure-SSIS IR par région](https://azure.microsoft.com/global-infrastructure/services/?products=data-factory&regions=all). 

## <a name="create-a-data-factory"></a>Créer une fabrique de données

Pour créer votre fabrique de données à l’aide du portail Azure, suivez les instructions pas à pas dans [Créer une fabrique de données à l’aide de l’interface utilisateur](./quickstart-create-data-factory-portal.md#create-a-data-factory). Durant la procédure, sélectionnez **Épingler au tableau de bord** pour y accéder rapidement après sa création. 

Une fois la fabrique de données créée, ouvrez la page de vue d’ensemble dans le portail Azure. Sélectionnez la vignette **Créer et surveiller** pour ouvrir la page **Prise en main** dans un onglet séparé. Vous pourrez continuer à créer votre IR Azure-SSIS à partir de cette page.

## <a name="create-an-azure-ssis-integration-runtime"></a>Créer un runtime d’intégration Azure-SSIS

### <a name="from-the-data-factory-overview"></a>À partir de la vue d’ensemble de Data Factory

1. Dans la page de **prise en main** , cliquez sur la vignette **Configurer un runtime d’intégration SSIS** . 

   ![Vignette Configurer un runtime d’intégration SSIS](./media/tutorial-create-azure-ssis-runtime-portal/configure-ssis-integration-runtime-tile.png)

1. Consultez la section [Approvisionner un runtime d’intégration Azure-SSIS](#provision-an-azure-ssis-integration-runtime) pour connaître les autres étapes de configuration d’un runtime d’intégration Azure-SSIS. 

### <a name="from-the-authoring-ui"></a>À partir de l’interface utilisateur de création

1. Dans l’interface utilisateur Azure Data Factory, basculez vers l’onglet **Modifier** et sélectionnez **Connexions** . Basculez ensuite vers l’onglet **Runtimes d’intégration** pour voir les runtimes d’intégration existants dans votre fabrique de données. 

   ![Sélections pour l’affichage des runtimes d’intégration existants](./media/tutorial-create-azure-ssis-runtime-portal/view-azure-ssis-integration-runtimes.png)

1. Sélectionnez **Nouveau** pour créer un runtime d’intégration Azure-SSIS et ouvrez le volet **Configuration du runtime d’intégration** . 

   ![Runtime d’intégration via le menu](./media/tutorial-create-azure-ssis-runtime-portal/edit-connections-new-integration-runtime-button.png)

1. Dans le volet **Configuration du runtime d’intégration** , sélectionnez la vignette **Faire une migration lift-and-shift des packages SSIS existants à exécuter dans Azure** , puis sélectionnez **Suivant** .

   ![Spécifier le type de runtime d’intégration](./media/tutorial-create-azure-ssis-runtime-portal/integration-runtime-setup-options.png)

1. Consultez la section [Approvisionner un runtime d’intégration Azure-SSIS](#provision-an-azure-ssis-integration-runtime) pour connaître les autres étapes de configuration d’un runtime d’intégration Azure-SSIS. 

## <a name="provision-an-azure-ssis-integration-runtime"></a>Approvisionner un runtime d’intégration Azure-SSIS

Le volet **Configuration du runtime d’intégration** comporte trois pages dans lesquelles vous pouvez configurer successivement les paramètres généraux, de déploiement et avancés.

### <a name="general-settings-page"></a>Page Paramètres généraux

Dans la page **Paramètres généraux** du volet **Configuration du runtime d’intégration** , effectuez les étapes suivantes. 

   ![Paramètres généraux :](./media/tutorial-create-azure-ssis-runtime-portal/general-settings.png)

   1. Dans **Nom** , saisissez le nom de votre runtime d’intégration. 

   1. Dans **Description** , saisissez la description de votre runtime d’intégration. 

   1. Pour **Emplacement** , sélectionnez l’emplacement de votre runtime d’intégration. Seuls les emplacements pris en charge sont affichés. Nous vous conseillons de sélectionner le même emplacement que votre serveur de base de données pour héberger SSISDB. 

   1. Pour **Taille du nœud** , sélectionnez la taille du nœud du cluster de votre runtime d’intégration. Seules les tailles de nœuds prises en charge sont affichées. Sélectionnez une taille de nœud importante (scale-up) si vous voulez exécuter de nombreux packages nécessitant beaucoup de calcul ou mémoire. 

   1. Pour **Nombre de nœud** , sélectionnez le nombre de nœuds du cluster de votre runtime d’intégration. Seuls les nombres de nœuds pris en charge sont affichés. Sélectionnez un grand cluster avec de nombreux nœuds (scale-out) si vous voulez exécuter de nombreux packages en parallèle. 

   1. Pour **Édition/Licence** , sélectionnez l’édition SQL Server pour votre runtime d’intégration : Standard ou Entreprise. Sélectionnez Entreprise si vous voulez utiliser les fonctionnalités avancées de votre runtime d’intégration. 

   1. Pour **Économiser de l’argent** , sélectionnez l’option Azure Hybrid Benefit pour votre runtime d’intégration : **Oui** ou **Non** . Sélectionnez **Oui** si vous souhaitez utiliser votre propre licence SQL Server avec Software Assurance pour faire des économies grâce à une utilisation hybride. 

   1. Sélectionnez **Suivant** . 

### <a name="deployment-settings-page"></a>Page Paramètres de déploiement

Dans la page **Paramètres de déploiement** du volet **Installation du runtime d’intégration** , vous disposez des options permettant de créer des magasins de packages Azure-SSIS IR ou SSISDB.

#### <a name="creating-ssisdb"></a>Création de SSISDB

Dans la page **Paramètres de déploiement** du volet **Installation du runtime d’intégration** , si vous souhaitez déployer vos packages dans SSISDB (Modèle de déploiement de projet), cochez la case **Créer un catalogue SSIS (SSISDB) hébergé par un serveur Azure SQL Database/Managed Instance pour stocker vos projets/packages/environnements/journaux d’exécution** . Autrement, si vous souhaitez déployer vos packages dans le système de fichiers, Azure Files ou une base de données SQL Server (MSDB) hébergée par Azure SQL Managed Instance (modèle de déploiement de package), il n’est pas nécessaire de créer de SSISDB ni de cocher la case.

Quel que soit votre modèle de déploiement, si vous souhaitez utiliser SQL Server Agent hébergé par Azure SQL Managed Instance afin d’orchestrer ou de planifier les exécutions de vos packages, cela est activé par SSISDB ; vous pouvez donc cocher quand même cette case. Pour plus d’informations, consultez [Planifier des exécutions de packages SSIS par le biais de l’agent Azure SQL Managed Instance](./how-to-invoke-ssis-package-managed-instance-agent.md).
   
Si vous cochez la case, effectuez les étapes suivantes pour apporter votre propre serveur de base de données afin d’héberger la base de données SSISDB que nous allons créer et gérer en votre nom.

   ![Paramètres de déploiement pour SSISDB](./media/tutorial-create-azure-ssis-runtime-portal/deployment-settings.png)
   
   1. Pour **Abonnement** , sélectionnez l’abonnement Azure comprenant votre serveur de base de données pour héberger la base de données SSISDB. 

   1. Pour **Emplacement** , sélectionnez l’emplacement de votre serveur de base de données pour héberger la base de données SSISDB. Nous vous recommandons de sélectionner le même emplacement que celui de runtime d’intégration.

   1. Pour **Point de terminaison du serveur de base de données du catalogue** , sélectionnez le point de terminaison de votre serveur de base de données pour héberger SSISDB. 
   
      En fonction du serveur de base de données sélectionné, l’instance SSISDB peut être créée à pour vous en tant que base de données unique, dans le cadre d’un pool élastique ou dans une instance managée. Elle peut être accessible dans un réseau public ou en la joignant à un réseau virtuel. Pour obtenir des conseils sur le choix du type de serveur de base de données pour héberger SSISDB, consultez [Comparer SQL Database et SQL Managed Instance](../data-factory/create-azure-ssis-integration-runtime.md#comparison-of-sql-database-and-sql-managed-instance).   

      Si vous sélectionnez un serveur Azure SQL Database avec des règles de pare-feu IP/points de terminaison de service de réseau virtuel ou une instance managée avec un point de terminaison privé pour héberger le catalogue SSISDB, ou si vous avez besoin d’accéder à des données locales sans configurer un runtime d’intégration (IR) auto-hébergé, vous devez joindre votre Azure-SSIS IR à un réseau virtuel. Pour plus d’informations, consultez [Créer un runtime Azure-SSIS IR dans un réseau virtuel](./create-azure-ssis-integration-runtime.md).

   1. Cochez la case **Utiliser l’authentification Azure AD avec l’identité managée pour votre ADF** afin de choisir la méthode d’authentification de votre serveur de base de données pour héberger le catalogue SSISDB. Vous choisissez l’authentification SQL ou l’authentification Azure AD avec l’identité managée de votre fabrique de données.

      Si vous cochez la case, vous devez ajouter l’identité managée de votre fabrique de données à un groupe Azure AD avec autorisations d’accès à votre serveur de base de données. Pour plus d’informations, consultez [Créer un runtime Azure-SSIS IR avec l’authentification AAD](./create-azure-ssis-integration-runtime.md).
   
   1. Pour **Nom d’utilisateur administrateur** , entrez le nom d’utilisateur d’authentification SQL de votre serveur de base de données pour héberger le catalogue SSISDB. 

   1. Pour **Mot de passe administrateur** , entrez le mot de passe d’authentification SQL de votre serveur de base de données pour héberger le catalogue SSISDB. 

   1. Pour **Niveau de service de base de données de catalogue** , sélectionnez le niveau de service de votre serveur de base de données pour héberger le catalogue SSISDB. Sélectionnez le niveau De base, Standard ou Premium, ou sélectionnez un nom de pool élastique.

Sélectionnez **Tester la connexion** le cas échéant et, si l’opération réussit, sélectionnez **Suivant** .

#### <a name="creating-azure-ssis-ir-package-stores"></a>Création de magasins de packages Azure-SSIS IR

Dans la page **Paramètres de déploiement** du volet **Installation du runtime d’intégration** , si vous souhaitez gérer vos packages qui sont déployés dans MSDB, le système de fichiers ou Azure Files (modèle de déploiement de package) avec des magasins de packages Azure-SSIS IR, cochez la case **Créer des magasins de packages pour gérer les packages qui sont déployés dans le système de fichiers/Azure Files/une base de données SQL Server (MSDB) hébergée par Azure SQL Managed Instance** .
   
Le magasin de packages Azure-SSIS IR vous permet d’importer/d’exporter/de supprimer/d’exécuter des packages et de superviser/d’arrêter l’exécution de packages par le biais de SSMS, à l’image du [magasin de packages SSIS hérités](/sql/integration-services/service/package-management-ssis-service?view=sql-server-2017). Pour plus d’informations, consultez [Gérer les packages SSIS avec des magasins de packages Azure-SSIS IR](./azure-ssis-integration-runtime-package-store.md).
   
Si vous cochez cette case, vous pouvez ajouter plusieurs magasins de packages à votre runtime d’intégration Azure-SSIS en sélectionnant **Nouveau** . À l’inverse, un magasin de packages peut être partagé par plusieurs runtimes d’intégration Azure-SSIS.

![Paramètres de déploiement pour MSDB/le système de fichiers/Azure Files](./media/tutorial-create-azure-ssis-runtime-portal/deployment-settings2.png)

Dans le volet **Ajouter un magasin de packages** , effectuez les étapes suivantes.
   
   1. Dans **Nom du magasin de packages** , entrez le nom de votre magasin de packages. 

   1. Dans **Service lié du magasin de packages** , sélectionnez votre service lié existant qui stocke les informations d’accès pour le système de fichiers/Azure Files/Azure SQL Managed Instance où vos packages sont déployés ou créez-en un en sélectionnant **Nouveau** . Dans le volet **Nouveau service lié** , effectuez les étapes suivantes. 

      > [!NOTE]
      > Vous pouvez utiliser les services liés **Stockage Fichier Azure** ou **Système de fichiers** pour accéder à Azure Files. Si vous utilisez le service lié **Stockage Fichier Azure** , le magasin de packages Azure-SSIS IR prend en charge uniquement la méthode d’authentification **De base** (pas les méthodes **Clé de compte** et **URI SAS** ). Pour utiliser l’authentification **De base** sur le service lié **Stockage Fichier Azure** , ajoutez `?feature.upgradeAzureFileStorage=false` à l’URL du portail ADF dans votre navigateur. Vous pouvez aussi utiliser le service lié **Système de fichiers** pour accéder à Azure Files à la place. 

      ![Paramètres de déploiement pour les services liés](./media/tutorial-create-azure-ssis-runtime-portal/deployment-settings-linked-service.png)

      1. Dans **Nom** , entrez le nom de votre service lié. 
         
      1. Dans **Description** , entrez la description de votre service lié. 
         
      1. Dans **Type** , sélectionnez **Stockage Fichier Azure** , **Azure SQL Managed Instance** ou **Système de fichiers** .

      1. Vous pouvez ignorer **Se connecter via le runtime d’intégration** , car nous utilisons toujours votre runtime d’intégration Azure-SSIS pour extraire les informations d’accès des magasins de packages.

      1. Si vous sélectionnez **Stockage Fichier Azure** , effectuez les étapes suivantes. 

         1. Pour **Méthode de sélection de compte** , sélectionnez **À partir d’un abonnement Azure** ou **Entrer manuellement** .
         
         1. Si vous sélectionnez **À partir d’un abonnement Azure** , sélectionnez l’ **abonnement Azure** , le **nom du compte de stockage** et le **partage de fichiers** appropriés.
            
         1. Si vous sélectionnez **Entrer manuellement** , entrez `\\<storage account name>.file.core.windows.net\<file share name>` pour **Hôte** , `Azure\<storage account name>` pour **Nom d’utilisateur** et `<storage account key>` pour **Mot de passe** ou sélectionnez le **coffre de clés Azure** où le mot de passe est stocké en tant que secret.

      1. Si vous sélectionnez **Azure SQL Managed Instance** , effectuez les étapes suivantes. 

         1. Sélectionnez **Chaîne de connexion** pour entrer la chaîne manuellement ou le **coffre de clés Azure** où elle est stockée en tant que secret.
         
         1. Si vous sélectionnez **Chaîne de connexion** , effectuez les étapes suivantes. 

            1. Pour **Nom de domaine complet** , entrez `<server name>.<dns prefix>.database.windows.net` ou `<server name>.public.<dns prefix>.database.windows.net,3342` comme point de terminaison privé ou public de votre instance gérée SQL Azure, respectivement. Si vous entrez le point de terminaison privé, **Tester la connexion** n’est pas applicable, car l’interface utilisateur d’ADF ne peut pas l’atteindre.

            1. Pour **Nom de la base de données** , entrez `msdb`.
               
            1. Pour **Type d’authentification** , sélectionnez **Authentification SQL** , **Identité gérée** ou **Principal du service** .

            1. Si vous sélectionnez **Authentification SQL** , entrez les **nom d’utilisateur** et **mot de passe** appropriés ou sélectionnez le **coffre de clés Azure** où le mot de passe est stocké en tant que secret.

            1. Si vous sélectionnez **Identité gérée** , accordez à votre instance gérée ADF l’accès à votre instance gérée SQL Azure.

            1. Si vous sélectionnez **Principal du service** , entrez les **ID principal de service** et la **clé du principal de service** appropriés ou sélectionnez le **coffre de clés Azure** où la clé est stockée en tant que secret.

      1. Si vous sélectionnez **Système de fichiers** , entrez le chemin d’accès UNC du dossier dans lequel vos packages sont déployés pour **Hôte** ainsi que les **nom d’utilisateur** et **mot de passe** appropriés, ou sélectionnez le **coffre de clés Azure** où le mot de passe est stocké en tant que secret.

      1. Sélectionnez **Tester la connexion** le cas échéant et, si l’opération réussit, sélectionnez **Créer** .

   1. Vos magasins de packages ajoutés s’affichent dans la page **Paramètres de déploiement** . Pour les supprimer, activez leurs cases à cocher, puis sélectionnez **Supprimer** .

Sélectionnez **Tester la connexion** le cas échéant et, si l’opération réussit, sélectionnez **Suivant** .

### <a name="advanced-settings-page"></a>Page Paramètres avancés

Dans la page **Paramètres avancés** du volet **Configuration du runtime d’intégration** , effectuez les étapes suivantes. 

   ![Paramètres avancés](./media/tutorial-create-azure-ssis-runtime-portal/advanced-settings.png)

   1. Pour **Exécutions parallèles maximales par nœud** , sélectionnez le nombre maximal de packages exécutables simultanément par nœud au sein de votre cluster de runtime d’intégration. Seuls les numéros des packages pris en charge sont affichés. Sélectionnez un petit nombre si vous souhaitez utiliser plusieurs cœurs pour exécuter un seul package volumineux nécessitant beaucoup de calcul ou de mémoire. Sélectionnez un nombre élevé si vous souhaitez exécuter un ou plusieurs petits packages dans un seul cœur. 

   1. Cochez la case **Personnaliser votre Azure-SSIS Integration Runtime avec des installations de composants/configurations système supplémentaires** pour indiquer si vous souhaitez ajouter des configurations personnalisées standard/Express sur votre Azure-SSIS IR. Pour plus d’informations, consultez [Personnalisation de l’installation du runtime d’intégration Azure-SSIS](./how-to-configure-azure-ssis-ir-custom-setup.md).
   
   1. Cochez la case **Sélectionner un réseau virtuel à joindre à votre Azure-SSIS Integration Runtime, autoriser ADF à créer certaines ressources réseau et apporter éventuellement vos propres adresses IP publiques statiques** pour indiquer si vous souhaitez joindre votre Azure-SSIS IR à un réseau virtuel.

      Cochez cette case si vous utilisez un serveur Azure SQL Database avec des règles de pare-feu IP/points de terminaison de service de réseau virtuel ou une instance managée avec un point de terminaison privé pour héberger SSISDB, ou si vous avez besoin d’accéder à des données locales sans configurer un runtime d’intégration auto-hébergé. Pour plus d’informations, consultez [Créer un runtime Azure-SSIS IR dans un réseau virtuel](./create-azure-ssis-integration-runtime.md). 
   
   1. Cochez la case **Configurer le runtime d’intégration auto-hébergé en tant que proxy pour votre Azure-SSIS Integration Runtime** pour indiquer si vous voulez configurer un runtime d’intégration auto-hébergé en tant que proxy pour votre Azure-SSIS IR. Pour plus d’informations, consultez [Configurer un runtime d’intégration auto-hébergé en tant que proxy](./self-hosted-integration-runtime-proxy-ssis.md).   

   1. Sélectionnez **Continuer** . 

Dans la page **Résumé** du volet **Configuration du runtime d’intégration** , passez en revue tous les paramètres de provisionnement, ajoutez les liens de documentation recommandés aux favoris, puis sélectionnez **Terminer** pour lancer la création de votre runtime d’intégration. 

   > [!NOTE]
   > En dehors des réglages personnalisés, ce processus devrait se terminer dans les 5 minutes.
   >
   > Si vous utilisez le catalogue SSISDB, le service Data Factory se connecte à votre serveur de base de données pour le préparer. 
   > 
   > Quand vous provisionnez un runtime d’intégration Azure-SSIS, les composants Access Redistributable et Azure Feature Pack pour SSIS sont également installés. Ces composants assurent la connectivité aux fichiers Excel et Access ainsi qu’à diverses sources de données Azure, en plus des sources de données déjà prises en charge par les composants intégrés. Pour plus d’informations sur les composants intégrés/préinstallés, consultez [Composants intégrés/préinstallés sur Azure-SSIS IR](./built-in-preinstalled-components-ssis-integration-runtime.md). Pour plus d’informations sur les composants supplémentaires que vous pouvez installer, consultez [Personnalisation des configurations du runtime d’intégration Azure-SSIS](./how-to-configure-azure-ssis-ir-custom-setup.md).

### <a name="connections-pane"></a>Volet Connexions

Dans le volet **Connexions** du hub **Gestion** , basculez sur la page **Runtimes d’intégration** et sélectionnez **Actualiser** . 

   ![Volet Connexions](./media/tutorial-create-azure-ssis-runtime-portal/connections-pane.png)

   Vous pouvez modifier/reconfigurer votre runtime d’intégration Azure-SSIS en sélectionnant son nom. Vous pouvez également sélectionner les boutons appropriés pour superviser/démarrer/arrêter/supprimer votre runtime d’intégration Azure-SSIS, générer automatiquement un pipeline ADF avec l’activité Exécuter le package SSIS à exécuter sur votre runtime d’intégration Azure-SSIS et afficher le code JSON/la charge utile de votre runtime d’intégration Azure-SSIS.  Vous ne pouvez modifier ou supprimer votre runtime d’intégration Azure-SSIS que quand il est arrêté.

## <a name="deploy-ssis-packages"></a>Déployer des packages SSIS

Si vous utilisez le catalogue SSISDB, vous pouvez y déployer vos packages et les exécuter sur votre runtime d’intégration Azure-SSIS à l’aide des outils SSDT ou SSMS compatibles avec Azure. Ces outils se connectent à votre serveur de base de données via son point de terminaison : 

- Pour un serveur Azure SQL Database, le format du point de terminaison de serveur est `<server name>.database.windows.net`.
- Pour une instance managée avec un point de terminaison privé, le format du point de terminaison de serveur est `<server name>.<dns prefix>.database.windows.net`.
- Pour une instance managée avec un point de terminaison public, le format du point de terminaison de serveur est `<server name>.public.<dns prefix>.database.windows.net,3342`. 

Si vous n’utilisez pas SSISDB, vous pouvez déployer vos packages sur un système de fichiers, Azure Files ou MSDB, dont l’hébergement est assuré par Azure SQL Managed Instance, puis les exécuter sur Azure-SSIS IR à l’aide des utilitaires en ligne de commande [dtutil](/sql/integration-services/dtutil-utility?view=sql-server-2017) et [AzureDTExec](./how-to-invoke-ssis-package-azure-enabled-dtexec.md). 

Pour plus d’informations, consultez [Déployer des projets ou des packages SSIS](/sql/integration-services/packages/deploy-integration-services-ssis-projects-and-packages?view=sql-server-ver15).

Dans les deux cas, vous pouvez également exécuter vos packages déployés sur Azure-SSIS IR à l’aide de l’activité Exécuter le package SSIS dans des pipelines Data Factory. Pour plus d’informations, consultez [Appeler une exécution de package SSIS comme activité Data Factory de première classe](./how-to-invoke-ssis-package-ssis-activity.md).

Consultez également la documentation SSIS suivante : 

- [Déployer, exécuter et surveiller des packages SSIS dans Azure](/sql/integration-services/lift-shift/ssis-azure-deploy-run-monitor-tutorial) 
- [Se connecter à SSISDB dans Azure](/sql/integration-services/lift-shift/ssis-azure-connect-to-catalog-database) 
- [Planifier les exécutions de packages sur Azure](/sql/integration-services/lift-shift/ssis-azure-schedule-packages) 
- [Se connecter aux sources de données locales avec l’authentification Windows](/sql/integration-services/lift-shift/ssis-azure-connect-with-windows-auth) 

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur la personnalisation de votre runtime d’intégration Azure-SSIS, passez à l’article suivant : 

> [!div class="nextstepaction"]
> [Personnaliser un runtime Azure-SSIS IR](./how-to-configure-azure-ssis-ir-custom-setup.md)