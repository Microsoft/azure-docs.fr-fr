---
title: Créer un runtime d’intégration Azure-SSIS dans Azure Data Factory
description: Découvrez comment créer un runtime d’intégration Azure-SSIS dans Azure Data Factory afin de pouvoir déployer et exécuter des packages SSIS dans Azure.
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 10/13/2020
author: swinarko
ms.author: sawinark
ms.reviewer: douglasl
manager: mflasko
ms.openlocfilehash: 7936699832a09f535729c42b12fec2d5c49a11a0
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96350940"
---
# <a name="create-an-azure-ssis-integration-runtime-in-azure-data-factory"></a>Créer un runtime d’intégration Azure-SSIS dans Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Cet article décrit les différentes étapes d’approvisionnement d’un runtime d’intégration (IR) Azure-SQL Server Integration Services (SSIS) dans Azure Data Factory (ADF). Un IR Azure-SSIS prend en charge :

- L’exécution de packages déployés dans le catalogue SSIS (SSISDB) hébergé par un serveur Azure SQL Database/Managed Instance (modèle de déploiement de projet)
- L’exécution de packages déployés dans le système de fichiers, Azure Files ou une base de données SQL Server (MSDB) hébergée par Azure SQL Managed Instance (modèle de déploiement de package)

Une fois qu’un IR Azure-SSIS est provisionné, vous pouvez utiliser des outils familiers pour déployer et exécuter vos packages dans Azure. Ces outils sont déjà compatibles avec Azure et incluent SQL Server Data Tools (SSDT), SQL Server Management Studio (SSMS) et des utilitaires en ligne de commande comme [dtutil](/sql/integration-services/dtutil-utility?view=sql-server-2017) et [AzureDTExec](./how-to-invoke-ssis-package-azure-enabled-dtexec.md).

Le tutoriel [Provisionner le runtime d’intégration Azure-SSIS dans Azure Data Factory](./tutorial-deploy-ssis-packages-azure.md) montre comment créer un IR Azure-SSIS à l’aide du portail Azure ou de l’application Data Factory. Ce tutoriel montre également comment utiliser, si vous le souhaitez, un serveur Azure SQL Database ou une instance managée pour héberger le catalogue SSISDB. Cet article s’appuie sur le tutoriel et explique comment effectuer les tâches facultatives suivantes :

- Utiliser un serveur Azure SQL Database avec des règles de pare-feu IP/points de terminaison de service de réseau virtuel ou une instance managée avec un point de terminaison privé pour héberger SSISDB. Vous devez au préalable configurer les autorisations et paramètres de réseau virtuel pour que votre runtime d’intégration Azure-SSIS soit joint à un réseau virtuel.

- Utiliser l’authentification Azure Active Directory (Azure AD) avec l’identité managée pour que votre fabrique de données se connecte à un serveur Azure SQL Database ou une instance managée. Vous devez au préalable ajouter l’identité managée pour votre fabrique de données comme utilisateur de base de données capable de créer une instance SSISDB.

- Joindre votre IR Azure-SSIS à un réseau virtuel ou configurer un IR auto-hébergé en tant que proxy pour votre IR Azure-SSIS afin d’accéder aux données locales.

Cet article montre comment provisionner un IR Azure-SSIS à l’aide du portail Azure, d’Azure PowerShell et d’un modèle Azure Resource Manager.

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

- **Abonnement Azure**. Si vous n’avez pas encore d’abonnement, vous pouvez créer un compte d’[essai gratuit](https://azure.microsoft.com/pricing/free-trial/).

- **Serveur Azure SQL Database ou SQL Managed Instance (facultatif)** . Si vous n'avez pas encore de serveur de base de données ou d'instance gérée, créez-en un(e) sur le portail Azure avant de commencer. Data Factory crée à son tour une instance SSISDB sur ce serveur de base de données. 

  Nous vous recommandons de créer le serveur de base de données ou l'instance gérée dans la même région Azure que le runtime d'intégration. Cette configuration permet au runtime d’intégration d’écrire des journaux des exécutions dans SSISDB sans dépasser les régions Azure.

  Gardez à l’esprit les points suivants :

  - L'instance SSISDB peut être créée pour vous en tant que base de données unique, dans le cadre d’un pool élastique, ou dans une instance gérée. Elle peut être accessible dans un réseau public ou en la joignant à un réseau virtuel. Pour savoir comment choisir entre SQL Database et SQL Managed Instance pour héberger SSISDB, consultez la section [Comparer SQL Database et SQL Managed Instance](#comparison-of-sql-database-and-sql-managed-instance) de cet article. 
  
    Si vous utilisez un serveur Azure SQL Database avec des règles de pare-feu IP/points de terminaison de service de réseau virtuel ou SQL Managed Instance avec un point de terminaison privé pour héberger le catalogue SSISDB, ou si vous avez besoin d'accéder à des données locales sans configurer de runtime d'intégration (IR) auto-hébergé, vous devez joindre votre Azure-SSIS IR à un réseau virtuel. Pour plus d’informations, consultez [Joindre un runtime d’intégration Azure-SSIS à un réseau virtuel](./join-azure-ssis-integration-runtime-virtual-network.md).

  - Vérifiez que le paramètre **Autoriser l’accès aux services Azure** est activé pour votre serveur de base de données. Ce paramètre ne s'applique pas lorsque vous utilisez un serveur Azure SQL Database avec des règles de pare-feu IP/points de terminaison de service de réseau virtuel ou SQL Managed Instance avec un point de terminaison privé pour héberger le catalogue SSISDB. Pour plus d’informations, consultez [Sécuriser Azure SQL Database](../azure-sql/database/secure-database-tutorial.md#create-firewall-rules). Pour savoir comment activer ce paramètre à l’aide de PowerShell, consultez [New-AzSqlServerFirewallRule](/powershell/module/az.sql/new-azsqlserverfirewallrule).

  - Ajoutez l’adresse IP de l’ordinateur client ou une plage d’adresses IP qui inclut l’adresse IP de l’ordinateur client à la liste d’adresses IP client dans les paramètres de pare-feu du serveur de base de données. Pour plus d’informations, consultez [Règles de pare-feu au niveau du serveur et de la base de données Azure SQL Database](../azure-sql/database/firewall-configure.md).

  - Vous pouvez vous connecter au serveur de base de données à l’aide de l’authentification SQL avec vos informations d’identification d’administrateur de serveur ou de l’authentification Azure AD avec l’identité managée de votre fabrique de données. Pour cette dernière, vous devez ajouter l’identité managée de votre fabrique de données à un groupe Azure AD avec autorisations d’accès au serveur de base de données. Pour plus d’informations, consultez [Activer l’authentification Azure AD pour le runtime d’intégration (IR) Azure-SSIS](./enable-aad-authentication-azure-ssis-ir.md).

  - Vérifiez que votre serveur de base de données ne dispose pas déjà d’une instance SSISDB. Le provisionnement d’un IR Azure-SSIS ne prend pas en charge l’utilisation d’une instance SSISDB existante.

- **Réseau virtuel Azure Resource Manager (facultatif)** . Vous devez disposer d’un réseau virtuel Azure Resource Manager si au moins l’une des conditions suivantes est remplie :

  - Vous hébergez le catalogue SSISDB sur un serveur Azure SQL Database avec des règles de pare-feu IP/points de terminaison de service de réseau virtuel ou une instance managée avec un point de terminaison privé.

  - Vous souhaitez vous connecter à des magasins de données locaux à partir de packages SSIS en cours d’exécution sur votre IR Azure-SSIS sans configurer d’IR auto-hébergé.

- **Azure PowerShell (facultatif)** . Suivez les instructions de la rubrique [Guide pratique pour installer et configurer Azure PowerShell](/powershell/azure/install-az-ps) si vous souhaitez exécuter un script PowerShell pour provisionner votre runtime d’intégration Azure-SSIS.

### <a name="regional-support"></a>Prise en charge régionale

Pour obtenir la liste des régions Azure dans lesquelles Data Factory et un IR Azure-SSIS sont disponibles, consultez la page sur la [disponibilité d’Azure Data Factory et d’un IR Azure-SSIS par région](https://azure.microsoft.com/global-infrastructure/services/?products=data-factory&regions=all).

### <a name="comparison-of-sql-database-and-sql-managed-instance"></a>Comparaison entre SQL Database et SQL Managed Instance

Le tableau suivant compare certaines fonctionnalités d'un serveur Azure SQL Database et de SQL Managed Instance concernant l'IR Azure-SSIS :

| Fonctionnalité | SQL Database| SQL Managed Instance |
|---------|--------------|------------------|
| **Planification** | L’agent SQL Server n’est pas disponible.<br/><br/>Consultez [Planifier un package dans le cadre d’un pipeline Azure Data Factory](/sql/integration-services/lift-shift/ssis-azure-schedule-packages?view=sql-server-2017#activity).| L’agent Managed Instance est disponible. |
| **Authentification** | Vous pouvez créer une instance SSISDB avec un utilisateur de base de données autonome représentant un groupe Azure AD avec l’identité managée de votre fabrique de données en tant que membre du rôle **db_owner**.<br/><br/>Consultez [Activer l’authentification Azure AD pour créer un SSISDB sur le serveur Azure SQL Database](enable-aad-authentication-azure-ssis-ir.md#enable-azure-ad-on-azure-sql-database). | Vous pouvez créer une instance SSISDB avec un utilisateur de base de données autonome représentant l’identité managée de votre fabrique de données. <br/><br/>Consultez [Activer l’authentification Azure AD pour créer un SSISDB dans Azure SQL Managed Instance](enable-aad-authentication-azure-ssis-ir.md#enable-azure-ad-on-sql-managed-instance). |
| **Niveau de service** | Quand vous créez un IR Azure-SSIS avec votre serveur Azure SQL Database, vous pouvez sélectionner le niveau de service pour SSISDB. Il existe plusieurs niveaux de service. | Quand vous créez un IR Azure-SSIS avec votre instance managée, vous ne pouvez pas sélectionner le niveau de service pour SSISDB. Toutes les bases de données dans votre instance managée partagent la même ressource allouée à cette instance. |
| **Réseau virtuel** | Votre Azure-SSIS IR peut joindre un réseau virtuel Azure Resource Manager si vous utilisez un serveur Azure SQL Database avec des règles de pare-feu IP/points de terminaison de service de réseau virtuel. | Votre Azure-SSIS IR peut joindre un réseau virtuel Azure Resource Manager si vous utilisez une instance managée avec un point de terminaison privé. Le réseau virtuel est requis quand vous n’activez pas de point de terminaison public pour votre instance managée.<br/><br/>Si vous joignez votre IR Azure-SSIS au même réseau virtuel que votre instance managée, vérifiez qu’il est sur un sous-réseau différent de celui de votre instance managée. Si vous joignez votre IR Azure-SSIS à un réseau virtuel différent de celui de votre instance managée, nous recommandons un appairage de réseaux virtuels ou une connexion de réseau à réseau. Consultez [Connecter votre application à Azure SQL Database Managed Instance](../azure-sql/managed-instance/connect-application-instance.md). |
| **Transactions distribuées** | Cette fonctionnalité est prise en charge par le biais des transactions élastiques. Les transactions MSDTC (Microsoft Distributed Transaction Coordinator) ne sont pas prises en charge. Si vos packages SSIS utilisent MSDTC pour coordonner les transactions distribuées, envisagez la migration vers les transactions élastiques pour Azure SQL Database. Pour plus d’informations, consultez [Transactions distribuées entre bases de données cloud](../azure-sql/database/elastic-transactions-overview.md). | Non pris en charge. |
| | | |

## <a name="use-the-azure-portal-to-create-an-integration-runtime"></a>Utiliser le portail Azure pour créer un runtime d’intégration

Dans le cadre de cette section, vous allez utiliser le portail Azure (plus précisément, l’interface utilisateur ou l’application Data Factory) pour créer un runtime d’intégration Azure-SSIS.

### <a name="create-a-data-factory"></a>Créer une fabrique de données

Pour créer votre fabrique de données à l’aide du portail Azure, suivez les instructions pas à pas dans [Créer une fabrique de données à l’aide de l’interface utilisateur](./quickstart-create-data-factory-portal.md#create-a-data-factory). Durant la procédure, sélectionnez **Épingler au tableau de bord** pour y accéder rapidement après sa création. 

Une fois la fabrique de données créée, ouvrez la page de vue d’ensemble dans le portail Azure. Sélectionnez la vignette **Créer et surveiller** pour ouvrir la page **Commençons** dans un onglet séparé. Vous pourrez continuer à créer votre IR Azure-SSIS à partir de cette page.   

### <a name="provision-an-azure-ssis-integration-runtime"></a>Approvisionner un runtime d’intégration Azure-SSIS

Dans la page de **prise en main**, cliquez sur la vignette **Configurer un runtime d’intégration SSIS** pour ouvrir le volet **Configuration du runtime d’intégration**.

   ![Vignette Configurer un runtime d’intégration SSIS](./media/tutorial-create-azure-ssis-runtime-portal/configure-ssis-integration-runtime-tile.png)

   Le volet **Configuration du runtime d’intégration** comporte trois pages dans lesquelles vous pouvez configurer successivement les paramètres généraux, de déploiement et avancés.

#### <a name="general-settings-page"></a>Page Paramètres généraux

Dans la page **Paramètres généraux** du volet **Configuration du runtime d’intégration**, effectuez les étapes suivantes.

   ![Paramètres généraux :](./media/tutorial-create-azure-ssis-runtime-portal/general-settings.png)

   1. Dans **Nom**, saisissez le nom de votre runtime d’intégration.

   2. Dans **Description**, saisissez la description de votre runtime d’intégration.

   3. Pour **Emplacement**, sélectionnez l’emplacement de votre runtime d’intégration. Seuls les emplacements pris en charge sont affichés. Nous vous conseillons de sélectionner le même emplacement que votre serveur de base de données pour héberger SSISDB.

   4. Pour **Taille du nœud**, sélectionnez la taille du nœud du cluster de votre runtime d’intégration. Seules les tailles de nœuds prises en charge sont affichées. Sélectionnez une taille de nœud importante (scale-up) si vous voulez exécuter de nombreux packages nécessitant beaucoup de calcul ou mémoire.
   > [!NOTE]
   > Si vous avez besoin d’[isolation de calcul](../azure-government/azure-secure-isolation-guidance.md#compute-isolation), sélectionnez la taille de nœud **Standard_E64i_v3**. Cette taille de nœud représente des machines virtuelles isolées qui consomment la totalité de leur hôte physique et fournissent le niveau d’isolation requis pour certaines charges de travail, telles que les charges de travail Impact Level 5 (IL5) du ministère de la défense des États-Unis.
   
   5. Pour **Nombre de nœud**, sélectionnez le nombre de nœuds du cluster de votre runtime d’intégration. Seuls les nombres de nœuds pris en charge sont affichés. Sélectionnez un grand cluster avec de nombreux nœuds (scale-out) si vous voulez exécuter de nombreux packages en parallèle.

   6. Pour **Édition/Licence**, sélectionnez l’édition SQL Server pour votre runtime d’intégration : Standard ou Entreprise. Sélectionnez Entreprise si vous voulez utiliser les fonctionnalités avancées de votre runtime d’intégration.

   7. Pour **Économiser de l’argent**, sélectionnez l’option Azure Hybrid Benefit pour votre runtime d’intégration : **Oui** ou **Non**. Sélectionnez **Oui** si vous souhaitez utiliser votre propre licence SQL Server avec Software Assurance pour faire des économies grâce à une utilisation hybride.

   8. Sélectionnez **Suivant**.

#### <a name="deployment-settings-page"></a>Page Paramètres de déploiement

Dans la page **Paramètres de déploiement** du volet **Installation du runtime d’intégration**, vous disposez des options permettant de créer des magasins de packages Azure-SSIS IR ou SSISDB.

##### <a name="creating-ssisdb"></a>Création de SSISDB

Dans la page **Paramètres de déploiement** du volet **Installation du runtime d’intégration**, si vous souhaitez déployer vos packages dans SSISDB (Modèle de déploiement de projet), cochez la case **Créer un catalogue SSIS (SSISDB) hébergé par un serveur Azure SQL Database/Managed Instance pour stocker vos projets/packages/environnements/journaux d’exécution**. Autrement, si vous souhaitez déployer vos packages dans le système de fichiers, Azure Files ou une base de données SQL Server (MSDB) hébergée par Azure SQL Managed Instance (modèle de déploiement de package), il n’est pas nécessaire de créer de SSISDB ni de cocher la case.

Quel que soit votre modèle de déploiement, si vous souhaitez utiliser SQL Server Agent hébergé par Azure SQL Managed Instance afin d’orchestrer ou de planifier les exécutions de vos packages, cela est activé par SSISDB ; vous pouvez donc cocher quand même cette case. Pour plus d’informations, consultez [Planifier des exécutions de packages SSIS par le biais de l’agent Azure SQL Managed Instance](./how-to-invoke-ssis-package-managed-instance-agent.md).
   
Si vous cochez la case, effectuez les étapes suivantes pour apporter votre propre serveur de base de données afin d’héberger la base de données SSISDB que nous allons créer et gérer en votre nom.

   ![Paramètres de déploiement pour SSISDB](./media/tutorial-create-azure-ssis-runtime-portal/deployment-settings.png)
   
   1. Pour **Abonnement**, sélectionnez l’abonnement Azure comprenant votre serveur de base de données pour héberger la base de données SSISDB. 

   1. Pour **Emplacement**, sélectionnez l’emplacement de votre serveur de base de données pour héberger la base de données SSISDB. Nous vous recommandons de sélectionner le même emplacement que celui de runtime d’intégration.

   1. Pour **Point de terminaison du serveur de base de données du catalogue**, sélectionnez le point de terminaison de votre serveur de base de données pour héberger SSISDB. 
   
      En fonction du serveur de base de données sélectionné, l’instance SSISDB peut être créée à pour vous en tant que base de données unique, dans le cadre d’un pool élastique ou dans une instance managée. Elle peut être accessible dans un réseau public ou en la joignant à un réseau virtuel. Pour obtenir des conseils sur le choix du type de serveur de base de données pour héberger SSISDB, consultez [Comparer SQL Database et SQL Managed Instance](../data-factory/create-azure-ssis-integration-runtime.md#comparison-of-sql-database-and-sql-managed-instance).   

      Si vous sélectionnez un serveur Azure SQL Database avec des règles de pare-feu IP/points de terminaison de service de réseau virtuel ou une instance managée avec un point de terminaison privé pour héberger le catalogue SSISDB, ou si vous avez besoin d’accéder à des données locales sans configurer un runtime d’intégration (IR) auto-hébergé, vous devez joindre votre Azure-SSIS IR à un réseau virtuel. Pour plus d’informations, consultez [Créer un runtime Azure-SSIS IR dans un réseau virtuel]().

   1. Cochez la case **Utiliser l’authentification Azure AD avec l’identité managée pour votre ADF** afin de choisir la méthode d’authentification de votre serveur de base de données pour héberger le catalogue SSISDB. Vous choisissez l’authentification SQL ou l’authentification Azure AD avec l’identité managée de votre fabrique de données.

      Si vous cochez la case, vous devez ajouter l’identité managée de votre fabrique de données à un groupe Azure AD avec autorisations d’accès à votre serveur de base de données. Pour plus d’informations, consultez [Créer un runtime Azure-SSIS IR avec l’authentification AAD]().
   
   1. Pour **Nom d’utilisateur administrateur**, entrez le nom d’utilisateur d’authentification SQL de votre serveur de base de données pour héberger le catalogue SSISDB. 

   1. Pour **Mot de passe administrateur**, entrez le mot de passe d’authentification SQL de votre serveur de base de données pour héberger le catalogue SSISDB. 

   1. Pour **Niveau de service de base de données de catalogue**, sélectionnez le niveau de service de votre serveur de base de données pour héberger le catalogue SSISDB. Sélectionnez le niveau De base, Standard ou Premium, ou sélectionnez un nom de pool élastique.

Sélectionnez **Tester la connexion** le cas échéant et, si l’opération réussit, sélectionnez **Suivant**.

> [!NOTE]
   > Si vous utilisez Azure SQL Database Server pour héberger SSISDB, vos données seront stockées dans le stockage géoredondant pour les sauvegardes par défaut. Si vous ne souhaitez pas répliquer vos données dans d’autres régions, suivez les instructions pour [configurer la redondance du stockage de sauvegarde à l’aide de PowerShell](../azure-sql/database/automated-backups-overview.md?tabs=single-database#configure-backup-storage-redundancy-by-using-powershell).
   
##### <a name="creating-azure-ssis-ir-package-stores"></a>Création de magasins de packages Azure-SSIS IR

Dans la page **Paramètres de déploiement** du volet **Installation du runtime d’intégration**, si vous souhaitez gérer vos packages qui sont déployés dans MSDB, le système de fichiers ou Azure Files (modèle de déploiement de package) avec des magasins de packages Azure-SSIS IR, cochez la case **Créer des magasins de packages pour gérer les packages qui sont déployés dans le système de fichiers/Azure Files/une base de données SQL Server (MSDB) hébergée par Azure SQL Managed Instance**.
   
Le magasin de packages Azure-SSIS IR vous permet d’importer/d’exporter/de supprimer/d’exécuter des packages et de superviser/d’arrêter l’exécution de packages par le biais de SSMS, à l’image du [magasin de packages SSIS hérités](/sql/integration-services/service/package-management-ssis-service?view=sql-server-2017). Pour plus d’informations, consultez [Gérer les packages SSIS avec des magasins de packages Azure-SSIS IR](./azure-ssis-integration-runtime-package-store.md).
   
Si vous cochez cette case, vous pouvez ajouter plusieurs magasins de packages à votre runtime d’intégration Azure-SSIS en sélectionnant **Nouveau**. À l’inverse, un magasin de packages peut être partagé par plusieurs runtimes d’intégration Azure-SSIS.

![Paramètres de déploiement pour MSDB/le système de fichiers/Azure Files](./media/tutorial-create-azure-ssis-runtime-portal/deployment-settings2.png)

Dans le volet **Ajouter un magasin de packages**, effectuez les étapes suivantes.
   
   1. Dans **Nom du magasin de packages**, entrez le nom de votre magasin de packages. 

   1. Dans **Service lié du magasin de packages**, sélectionnez votre service lié existant qui stocke les informations d’accès pour le système de fichiers/Azure Files/Azure SQL Managed Instance où vos packages sont déployés ou créez-en un en sélectionnant **Nouveau**. Dans le volet **Nouveau service lié**, effectuez les étapes suivantes.
   
      > [!NOTE]
      > Vous pouvez utiliser les services liés **Stockage Fichier Azure** ou **Système de fichiers** pour accéder à Azure Files. Si vous utilisez le service lié **Stockage Fichier Azure**, le magasin de packages Azure-SSIS IR prend en charge uniquement la méthode d’authentification **De base** (pas les méthodes **Clé de compte** et **URI SAS**). Pour utiliser l’authentification **De base** sur le service lié **Stockage Fichier Azure**, ajoutez `?feature.upgradeAzureFileStorage=false` à l’URL du portail ADF dans votre navigateur. Vous pouvez aussi utiliser le service lié **Système de fichiers** pour accéder à Azure Files à la place. 

      ![Paramètres de déploiement pour les services liés](./media/tutorial-create-azure-ssis-runtime-portal/deployment-settings-linked-service.png)

      1. Dans **Nom**, entrez le nom de votre service lié. 
         
      1. Dans **Description**, entrez la description de votre service lié. 
         
      1. Dans **Type**, sélectionnez **Stockage Fichier Azure**, **Azure SQL Managed Instance** ou **Système de fichiers**.

      1. Vous pouvez ignorer **Se connecter via le runtime d’intégration**, car nous utilisons toujours votre runtime d’intégration Azure-SSIS pour extraire les informations d’accès des magasins de packages.

      1. Si vous sélectionnez **Stockage Fichier Azure**, effectuez les étapes suivantes. 

         1. Pour **Méthode de sélection de compte**, sélectionnez **À partir d’un abonnement Azure** ou **Entrer manuellement**.
         
         1. Si vous sélectionnez **À partir d’un abonnement Azure**, sélectionnez l’**abonnement Azure**, le **nom du compte de stockage** et le **partage de fichiers** appropriés.
            
         1. Si vous sélectionnez **Entrer manuellement**, entrez `\\<storage account name>.file.core.windows.net\<file share name>` pour **Hôte**, `Azure\<storage account name>` pour **Nom d’utilisateur** et `<storage account key>` pour **Mot de passe** ou sélectionnez le **coffre de clés Azure** où le mot de passe est stocké en tant que secret.

      1. Si vous sélectionnez **Azure SQL Managed Instance**, effectuez les étapes suivantes. 

         1. Sélectionnez **Chaîne de connexion** pour entrer la chaîne manuellement ou le **coffre de clés Azure** où elle est stockée en tant que secret.
         
         1. Si vous sélectionnez **Chaîne de connexion**, effectuez les étapes suivantes. 

            1. Pour **Nom de domaine complet**, entrez `<server name>.<dns prefix>.database.windows.net` ou `<server name>.public.<dns prefix>.database.windows.net,3342` comme point de terminaison privé ou public de votre instance gérée SQL Azure, respectivement. Si vous entrez le point de terminaison privé, **Tester la connexion** n’est pas applicable, car l’interface utilisateur d’ADF ne peut pas l’atteindre.

            1. Pour **Nom de la base de données**, entrez `msdb`.
               
            1. Pour **Type d’authentification**, sélectionnez **Authentification SQL**, **Identité gérée** ou **Principal du service**.

            1. Si vous sélectionnez **Authentification SQL**, entrez les **nom d’utilisateur** et **mot de passe** appropriés ou sélectionnez le **coffre de clés Azure** où le mot de passe est stocké en tant que secret.

            1. Si vous sélectionnez **Identité gérée**, accordez à votre instance gérée ADF l’accès à votre instance gérée SQL Azure.

            1. Si vous sélectionnez **Principal du service**, entrez les **ID principal de service** et la **clé du principal de service** appropriés ou sélectionnez le **coffre de clés Azure** où la clé est stockée en tant que secret.

      1. Si vous sélectionnez **Système de fichiers**, entrez le chemin d’accès UNC du dossier dans lequel vos packages sont déployés pour **Hôte** ainsi que les **nom d’utilisateur** et **mot de passe** appropriés, ou sélectionnez le **coffre de clés Azure** où le mot de passe est stocké en tant que secret.

      1. Sélectionnez **Tester la connexion** le cas échéant et, si l’opération réussit, sélectionnez **Créer**.

   1. Vos magasins de packages ajoutés s’affichent dans la page **Paramètres de déploiement**. Pour les supprimer, activez leurs cases à cocher, puis sélectionnez **Supprimer**.

Sélectionnez **Tester la connexion** le cas échéant et, si l’opération réussit, sélectionnez **Suivant**.

#### <a name="advanced-settings-page"></a>Page Paramètres avancés

Dans la page **Paramètres avancés** du volet **Configuration du runtime d’intégration**, effectuez les étapes suivantes.

   ![Paramètres avancés](./media/tutorial-create-azure-ssis-runtime-portal/advanced-settings.png)

   1. Pour **Exécutions parallèles maximales par nœud**, sélectionnez le nombre maximal de packages exécutables simultanément par nœud au sein de votre cluster de runtime d’intégration. Seuls les numéros des packages pris en charge sont affichés. Sélectionnez un petit nombre si vous souhaitez utiliser plusieurs cœurs pour exécuter un seul package volumineux nécessitant beaucoup de calcul ou de mémoire. Sélectionnez un nombre élevé si vous souhaitez exécuter un ou plusieurs petits packages dans un seul cœur.

   1. Cochez la case **Personnaliser votre Azure-SSIS Integration Runtime avec des installations de composants/configurations système supplémentaires** pour indiquer si vous souhaitez ajouter des configurations personnalisées standard/Express sur votre Azure-SSIS IR. Pour plus d’informations, consultez [Personnalisation de l’installation du runtime d’intégration Azure-SSIS](./how-to-configure-azure-ssis-ir-custom-setup.md).

      Si vous cochez la case, effectuez les étapes suivantes.

      ![Paramètres avancés avec des configurations personnalisées](./media/tutorial-create-azure-ssis-runtime-portal/advanced-settings-custom.png)
   
      1. Dans **Custom setup container SAS URI** (URI SAP du conteneur de configurations personnalisées), entrez l’URI SAP de votre conteneur dans lequel vous stockez les scripts et les fichiers associés pour les configurations personnalisées standard.

      1. Pour **Installation personnalisée rapide**, sélectionnez **Nouveau** afin d’ouvrir le panneau **Ajouter une installation personnalisée rapide**, puis sélectionnez les types de votre choix dans le menu déroulant **Type d’installation personnalisée rapide**, p. ex. **Exécuter la commande cmdkey**, **Ajouter une variable d'environnement**, **Installer un composant sous licence**, etc.

         Si vous sélectionnez le type **Installer le composant sous licence**, vous pouvez sélectionner tous les composants intégrés de nos partenaires ISV dans le menu déroulant **Nom du composant** et, si nécessaire, entrez la clé de licence du produit ou chargez le fichier de licence du produit que vous avez acheté auprès d’eux dans la case **Clé de licence**/**Fichier de licence**.
  
         Vos nouvelles installations personnalisées rapides s’affichent sur la page **Paramètres avancés**. Pour les supprimer, vous pouvez activer leurs cases à cocher, puis sélectionner **Supprimer**.

   1. Cochez la case **Sélectionner un réseau virtuel à joindre à votre Azure-SSIS Integration Runtime, autoriser ADF à créer certaines ressources réseau et apporter éventuellement vos propres adresses IP publiques statiques** pour indiquer si vous souhaitez joindre votre runtime d'intégration à un réseau virtuel. 

      Cochez cette case si vous utilisez un serveur Azure SQL Database avec des règles de pare-feu IP/des points de terminaison de service de réseau virtuel ou une instance managée avec un point de terminaison privé pour héberger SSISDB ou si vous avez besoin d’accéder à des données locales (si vous avez des sources/destinations de données locales dans vos packages SSIS) sans configurer d’IR auto-hébergé. Pour plus d’informations, consultez [Joindre le runtime d’intégration Azure-SSIS à un réseau virtuel](./join-azure-ssis-integration-runtime-virtual-network.md). 

      Si vous cochez la case, effectuez les étapes suivantes.

      ![Paramètres avancés avec un réseau virtuel](./media/tutorial-create-azure-ssis-runtime-portal/advanced-settings-vnet.png)

      1. Dans **Abonnement**, sélectionnez l’abonnement Azure possédant votre réseau virtuel.

      1. Pour **Emplacement**, sélectionnez le même emplacement que celui de votre runtime d’intégration.

      1. Pour **Type**, sélectionnez le type de votre réseau virtuel : classique ou Azure Resource Manager. Nous vous recommandons de sélectionner un réseau virtuel Azure Resource Manager, car les réseaux virtuels classiques seront bientôt dépréciés.

      1. Pour **Nom du réseau virtuel**, sélectionnez le nom de votre réseau virtuel. Il doit être identique à celui utilisé pour votre serveur Azure SQL Database avec des points de terminaison de service de réseau virtuel ou une instance managée avec un point de terminaison privé pour héberger le catalogue SSISDB. Il peut également s’agir de celui connecté à votre réseau local. Sinon, vous pouvez utiliser n’importe quel réseau virtuel pour apporter vos propres adresses IP publiques statiques pour Azure-SSIS IR.

      1. Pour **Nom du sous-réseau**, sélectionnez le nom du sous-réseau de votre réseau virtuel. Il doit être identique à celui utilisé pour votre serveur Azure SQL Database avec des points de terminaison de service de réseau virtuel pour héberger le catalogue SSISDB. Il peut également s’agir d’un autre sous-réseau que celui utilisé pour votre instance managée avec un point de terminaison privé pour héberger le catalogue SSISDB. Sinon, vous pouvez utiliser n’importe quel sous-réseau pour apporter vos propres adresses IP publiques statiques pour Azure-SSIS IR.

      1. Activez la case à cocher **Apporter des adresses IP publiques statiques pour votre Azure-SSIS Integration Runtime** pour choisir d’apporter vos propres adresses IP publiques statiques pour Azure-SSIS IR, de manière à pouvoir les autoriser sur le pare-feu pour vos sources de données.

         Si vous cochez la case, effectuez les étapes suivantes.

         1. Pour **Première adresse IP publique statique**, sélectionnez la première adresse IP publique statique qui répond aux exigences de votre Azure-SSIS IR. Si vous n’en avez pas, cliquez sur le lien **Créer** pour créer des adresses IP publiques statiques sur le Portail Azure, puis cliquez sur le bouton Actualiser du portail pour pouvoir les sélectionner.
      
         1. Pour **Deuxième adresse IP publique statique**, sélectionnez la deuxième adresse IP publique statique qui répond aux exigences de votre Azure-SSIS IR. Si vous n’en avez pas, cliquez sur le lien **Créer** pour créer des adresses IP publiques statiques sur le Portail Azure, puis cliquez sur le bouton Actualiser du portail pour pouvoir les sélectionner.

   1. Cochez la case **Configurer le runtime d’intégration auto-hébergé en tant que proxy pour votre Azure-SSIS Integration Runtime** pour indiquer si vous voulez configurer un runtime d’intégration auto-hébergé en tant que proxy pour votre Azure-SSIS IR. Pour plus d’informations, consultez [Configurer un runtime d’intégration auto-hébergé en tant que proxy](./self-hosted-integration-runtime-proxy-ssis.md). 

      Si vous cochez la case, effectuez les étapes suivantes.

      ![Paramètres avancés avec un IR auto-hébergé](./media/tutorial-create-azure-ssis-runtime-portal/advanced-settings-shir.png)

      1. Sous **Runtime d’intégration auto-hébergé**, sélectionnez votre IR auto-hébergé existant en tant que proxy pour Azure-SSIS IR.

      1. Sous **Service lié de stockage de préproduction**, sélectionnez votre service lié de stockage Blob Azure existant ou créez-en un pour la préproduction.

      1. Sous **Chemin d’accès intermédiaire**, spécifiez un conteneur d’objets blob dans le compte de stockage Blob Azure sélectionné ou laissez ce champ pour utiliser un conteneur par défaut pour la préproduction.

   1. Sélectionnez **Validation de réseau virtuel** > **Continuer**. 

Dans la section **Résumé**, passez en revue tous les paramètres de provisionnement, ajoutez les liens de documentation recommandés aux favoris, puis sélectionnez **Terminer** pour lancer la création de votre runtime d’intégration.

   > [!NOTE]
   > En dehors des réglages personnalisés, ce processus devrait se terminer dans les 5 minutes. Toutefois, l’IR Azure-SSIS peut prendre jusqu’à 20-30 minutes pour joindre un réseau virtuel.
   >
   > Si vous utilisez le catalogue SSISDB, le service Data Factory se connecte à votre serveur de base de données pour le préparer. Il configure également les autorisations et les paramètres de votre réseau virtuel, s’ils sont spécifiés, et joint votre runtime d’intégration Azure-SSIS au réseau virtuel.
   > 
   > Quand vous provisionnez un runtime d’intégration Azure-SSIS, les composants Access Redistributable et Azure Feature Pack pour SSIS sont également installés. Ces composants assurent la connectivité aux fichiers Excel et Access ainsi qu’à diverses sources de données Azure, en plus des sources de données déjà prises en charge par les composants intégrés. Pour plus d’informations sur les composants intégrés/préinstallés, consultez [Composants intégrés/préinstallés sur Azure-SSIS IR](./built-in-preinstalled-components-ssis-integration-runtime.md). Pour plus d’informations sur les composants supplémentaires que vous pouvez installer, consultez [Personnalisation des configurations du runtime d’intégration Azure-SSIS](./how-to-configure-azure-ssis-ir-custom-setup.md).

#### <a name="connections-pane"></a>Volet Connexions

Dans le volet **Connexions** du hub **Gestion**, basculez sur la page **Runtimes d’intégration** et sélectionnez **Actualiser**. 

   ![Volet Connexions](./media/tutorial-create-azure-ssis-runtime-portal/connections-pane.png)

   Vous pouvez modifier/reconfigurer votre runtime d’intégration Azure-SSIS en sélectionnant son nom. Vous pouvez également sélectionner les boutons appropriés pour superviser/démarrer/arrêter/supprimer votre runtime d’intégration Azure-SSIS, générer automatiquement un pipeline ADF avec l’activité Exécuter le package SSIS à exécuter sur votre runtime d’intégration Azure-SSIS et afficher le code JSON/la charge utile de votre runtime d’intégration Azure-SSIS.  Vous ne pouvez modifier ou supprimer votre runtime d’intégration Azure-SSIS que lorsqu’il est arrêté.

### <a name="azure-ssis-integration-runtimes-in-the-portal"></a>Runtimes d’intégration Azure SSIS sur le portail

1. Dans l’interface utilisateur Azure Data Factory, basculez vers l’onglet **Modifier** et sélectionnez **Connexions**. Basculez ensuite vers l’onglet **Runtimes d’intégration** pour voir les runtimes d’intégration existants dans votre fabrique de données.

   ![Afficher les runtimes d’intégration existants](./media/tutorial-create-azure-ssis-runtime-portal/view-azure-ssis-integration-runtimes.png)

1. Sélectionnez **Nouveau** pour créer un runtime d’intégration Azure-SSIS, puis ouvrez le volet **Configuration du runtime d’intégration**.

   ![Runtime d’intégration via le menu](./media/tutorial-create-azure-ssis-runtime-portal/edit-connections-new-integration-runtime-button.png)

1. Dans le volet **Configuration du runtime d’intégration**, sélectionnez la vignette **Faire une migration lift-and-shift des packages SSIS existants à exécuter dans Azure**, puis sélectionnez **Suivant**.

   ![Spécifier le type de runtime d’intégration](./media/tutorial-create-azure-ssis-runtime-portal/integration-runtime-setup-options.png)

1. Consultez la section [Provisionner un runtime d’intégration Azure-SSIS](#provision-an-azure-ssis-integration-runtime) pour connaître les autres étapes de configuration d’un runtime d’intégration Azure-SSIS.

## <a name="use-azure-powershell-to-create-an-integration-runtime"></a>Utiliser Azure PowerShell pour créer un runtime d’intégration

Dans le cadre de cette section, vous allez utiliser Azure PowerShell pour créer un runtime d’intégration Azure-SSIS.

### <a name="create-variables"></a>Créer des variables

Copiez et collez le script suivant. Spécifiez des valeurs pour les variables. 

```powershell
### Azure Data Factory info
# If your input contains a PSH special character like "$", precede it with the escape character "`" - for example, "`$"
$SubscriptionName = "[your Azure subscription name]"
$ResourceGroupName = "[your Azure resource group name]"
# Data factory name - must be globally unique
$DataFactoryName = "[your data factory name]"
# For supported regions, see https://azure.microsoft.com/global-infrastructure/services/?products=data-factory&regions=all
$DataFactoryLocation = "EastUS"

### Azure-SSIS integration runtime info - This is a Data Factory compute resource for running SSIS packages.
$AzureSSISName = "[your Azure-SSIS IR name]"
$AzureSSISDescription = "[your Azure-SSIS IR description]"
# For supported regions, see https://azure.microsoft.com/global-infrastructure/services/?products=data-factory&regions=all
$AzureSSISLocation = "EastUS"
# For supported node sizes, see https://azure.microsoft.com/pricing/details/data-factory/ssis/
$AzureSSISNodeSize = "Standard_D8_v3"
# 1-10 nodes are currently supported
$AzureSSISNodeNumber = 2
# Azure-SSIS IR edition/license info: Standard or Enterprise
$AzureSSISEdition = "Standard" # Standard by default, whereas Enterprise lets you use advanced features on your Azure-SSIS IR
# Azure-SSIS IR hybrid usage info: LicenseIncluded or BasePrice
$AzureSSISLicenseType = "LicenseIncluded" # LicenseIncluded by default, whereas BasePrice lets you bring your own on-premises SQL Server license with Software Assurance to earn cost savings from Azure Hybrid Benefit option
# For a Standard_D1_v2 node, up to 4 parallel executions per node are supported. For other nodes, up to (2 x number of cores) are currently supported.
$AzureSSISMaxParallelExecutionsPerNode = 8
# Custom setup info: Standard/express custom setups
$SetupScriptContainerSasUri = "" # OPTIONAL to provide a SAS URI of blob container for standard custom setup where your script and its associated files are stored
$ExpressCustomSetup = "[RunCmdkey|SetEnvironmentVariable|InstallAzurePowerShell|SentryOne.TaskFactory|oh22is.SQLPhonetics.NET|oh22is.HEDDA.IO|KingswaySoft.IntegrationToolkit|KingswaySoft.ProductivityPack|Theobald.XtractIS|AecorSoft.IntegrationService|CData.Standard|CData.Extended or leave it empty]" # OPTIONAL to configure an express custom setup without script
# Virtual network info: Classic or Azure Resource Manager
$VnetId = "[your virtual network resource ID or leave it empty]" # REQUIRED if you use an Azure SQL Database server with IP firewall rules/virtual network service endpoints or a managed instance with private endpoint to host SSISDB, or if you require access to on-premises data without configuring a self-hosted IR. We recommend an Azure Resource Manager virtual network, because classic virtual networks will be deprecated soon.
$SubnetName = "[your subnet name or leave it empty]" # WARNING: Use the same subnet as the one used for your Azure SQL Database server with virtual network service endpoints, or a different subnet from the one used for your managed instance with a private endpoint
# Public IP address info: OPTIONAL to provide two standard static public IP addresses with DNS name under the same subscription and in the same region as your virtual network
$FirstPublicIP = "[your first public IP address resource ID or leave it empty]"
$SecondPublicIP = "[your second public IP address resource ID or leave it empty]"

### SSISDB info
$SSISDBServerEndpoint = "[your Azure SQL Database server name.database.windows.net or managed instance name.DNS prefix.database.windows.net or managed instance name.public.DNS prefix.database.windows.net,3342 or leave it empty if you do not use SSISDB]" # WARNING: If you use SSISDB, ensure that there's no existing SSISDB on your database server, so we can prepare and manage one on your behalf
# Authentication info: SQL or Azure AD
$SSISDBServerAdminUserName = "[your server admin username for SQL authentication or leave it empty for Azure AD authentication]"
$SSISDBServerAdminPassword = "[your server admin password for SQL authentication or leave it empty for Azure AD authentication]"
# For the basic pricing tier, specify "Basic," not "B." For standard, premium, and elastic pool tiers, specify "S0," "S1," "S2," "S3," etc. See https://docs.microsoft.com/azure/sql-database/sql-database-resource-limits-database-server.
$SSISDBPricingTier = "[Basic|S0|S1|S2|S3|S4|S6|S7|S9|S12|P1|P2|P4|P6|P11|P15|…|ELASTIC_POOL(name = <elastic_pool_name>) for Azure SQL Database server or leave it empty for managed instance]"

### Self-hosted integration runtime info - This can be configured as a proxy for on-premises data access 
$DataProxyIntegrationRuntimeName = "" # OPTIONAL to configure a proxy for on-premises data access 
$DataProxyStagingLinkedServiceName = "" # OPTIONAL to configure a proxy for on-premises data access 
$DataProxyStagingPath = "" # OPTIONAL to configure a proxy for on-premises data access 
```

### <a name="sign-in-and-select-a-subscription"></a>Se connecter et sélectionner un abonnement

Ajoutez le script suivant pour vous connecter et sélectionner votre abonnement Azure.

```powershell
Connect-AzAccount
Select-AzSubscription -SubscriptionName $SubscriptionName
```

### <a name="validate-the-connection-to-database-server"></a>Valider la connexion au serveur de base de données

Ajoutez le script suivant pour valider votre serveur Azure SQL Database ou votre instance managée.

```powershell
# Validate only if you use SSISDB and you don't use virtual network or Azure AD authentication
if(![string]::IsNullOrEmpty($SSISDBServerEndpoint))
{
    if([string]::IsNullOrEmpty($VnetId) -and [string]::IsNullOrEmpty($SubnetName))
    {
        if(![string]::IsNullOrEmpty($SSISDBServerAdminUserName) -and ![string]::IsNullOrEmpty($SSISDBServerAdminPassword))
        {
            $SSISDBConnectionString = "Data Source=" + $SSISDBServerEndpoint + ";User ID=" + $SSISDBServerAdminUserName + ";Password=" + $SSISDBServerAdminPassword
            $sqlConnection = New-Object System.Data.SqlClient.SqlConnection $SSISDBConnectionString;
            Try
            {
                $sqlConnection.Open();
            }
            Catch [System.Data.SqlClient.SqlException]
            {
                Write-Warning "Cannot connect to your Azure SQL Database server, exception: $_";
                Write-Warning "Please make sure the server you specified has already been created. Do you want to proceed? [Y/N]"
                $yn = Read-Host
                if(!($yn -ieq "Y"))
                {
                    Return;
                }
            }
        }
    }
}
```

### <a name="configure-the-virtual-network"></a>Configurer le réseau virtuel

Ajoutez le script suivant afin de configurer automatiquement les autorisations et paramètres du réseau virtuel pour le runtime d’intégration Azure-SSIS à joindre.

```powershell
# Make sure to run this script against the subscription to which the virtual network belongs
if(![string]::IsNullOrEmpty($VnetId) -and ![string]::IsNullOrEmpty($SubnetName))
{
    # Register to the Azure Batch resource provider
    $BatchApplicationId = "ddbf3205-c6bd-46ae-8127-60eb93363864"
    $BatchObjectId = (Get-AzADServicePrincipal -ServicePrincipalName $BatchApplicationId).Id
    Register-AzResourceProvider -ProviderNamespace Microsoft.Batch
    while(!(Get-AzResourceProvider -ProviderNamespace "Microsoft.Batch").RegistrationState.Contains("Registered"))
    {
    Start-Sleep -s 10
    }
    if($VnetId -match "/providers/Microsoft.ClassicNetwork/")
    {
        # Assign the VM contributor role to Microsoft.Batch
        New-AzRoleAssignment -ObjectId $BatchObjectId -RoleDefinitionName "Classic Virtual Machine Contributor" -Scope $VnetId
    }
}
```

### <a name="create-a-resource-group"></a>Créer un groupe de ressources

Créez un [groupe de ressources Azure](../azure-resource-manager/management/overview.md) avec la commande [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Un groupe de ressources est un conteneur logique dans lequel les ressources Azure sont déployées et gérées en tant que groupe.

Si votre groupe de ressources existe déjà, ne copiez pas ce code dans votre script. 

```powershell
New-AzResourceGroup -Location $DataFactoryLocation -Name $ResourceGroupName
```

### <a name="create-a-data-factory"></a>Créer une fabrique de données

Exécutez la commande suivante pour créer la fabrique de données.

```powershell
Set-AzDataFactoryV2 -ResourceGroupName $ResourceGroupName `
    -Location $DataFactoryLocation `
    -Name $DataFactoryName
```

### <a name="create-an-integration-runtime"></a>Créer un runtime d’intégration

Exécutez les commandes suivantes pour créer un runtime d’intégration Azure-SSIS qui exécute des packages SSIS dans Azure.

Si vous n’utilisez pas de catalogue SSISDB, vous pouvez omettre les paramètres `CatalogServerEndpoint`, `CatalogPricingTier` et `CatalogAdminCredential`.

Si vous n’utilisez pas de serveur Azure SQL Database avec des règles de pare-feu IP/points de terminaison de service de réseau virtuel ou une instance managée avec un point de terminaison privé pour héberger le catalogue SSISDB, ou si vous avez besoin d’accéder à des données locales, vous pouvez omettre les paramètres `VNetId` et `Subnet` ou passer des valeurs vides pour ces paramètres. Vous pouvez également les omettre si vous configurez le runtime d’intégration auto-hébergé en tant que proxy pour votre Azure-SSIS IR afin d’accéder à des données locales. Dans le cas contraire, vous ne pouvez pas les omettre et vous devez passer des valeurs valides de votre configuration de réseau virtuel. Pour plus d’informations, consultez [Joindre un runtime d’intégration Azure-SSIS à un réseau virtuel](./join-azure-ssis-integration-runtime-virtual-network.md).

Si vous utilisez une instance managée pour héberger un catalogue SSISDB, vous pouvez omettre le paramètre `CatalogPricingTier` ou passer une valeur vide pour ce paramètre. Dans le cas contraire, vous ne pouvez pas l’omettre et vous devez passer une valeur valide à partir de la liste des niveaux tarifaires pris en charge pour Azure SQL Database. Pour en savoir plus, consultez [Limites de ressources SQL Database](../azure-sql/database/resource-limits-logical-server.md).

Si vous utilisez l’authentification Azure AD avec l’identité managée pour que votre fabrique de données se connecte au serveur de base de données, vous pouvez omettre le paramètre `CatalogAdminCredential`. Cependant, vous devez ajouter l’identité managée de votre fabrique de données à un groupe Azure AD avec autorisations d’accès au serveur de base de données. Pour plus d’informations, consultez [Activer l’authentification Azure AD pour le runtime d’intégration (IR) Azure-SSIS](./enable-aad-authentication-azure-ssis-ir.md). Sinon, vous ne pouvez pas l’omettre et vous devez passer un objet valide formé à partir de votre nom d’utilisateur et mot de passe d’administrateur de serveur pour l’authentification SQL.

```powershell
Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
    -DataFactoryName $DataFactoryName `
    -Name $AzureSSISName `
    -Description $AzureSSISDescription `
    -Type Managed `
    -Location $AzureSSISLocation `
    -NodeSize $AzureSSISNodeSize `
    -NodeCount $AzureSSISNodeNumber `
    -Edition $AzureSSISEdition `
    -LicenseType $AzureSSISLicenseType `
    -MaxParallelExecutionsPerNode $AzureSSISMaxParallelExecutionsPerNode `
    -VnetId $VnetId `
    -Subnet $SubnetName
       
# Add the CatalogServerEndpoint, CatalogPricingTier, and CatalogAdminCredential parameters if you use SSISDB
if(![string]::IsNullOrEmpty($SSISDBServerEndpoint))
{
    Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
        -DataFactoryName $DataFactoryName `
        -Name $AzureSSISName `
        -CatalogServerEndpoint $SSISDBServerEndpoint `
        -CatalogPricingTier $SSISDBPricingTier

    if(![string]::IsNullOrEmpty($SSISDBServerAdminUserName) –and ![string]::IsNullOrEmpty($SSISDBServerAdminPassword)) # Add the CatalogAdminCredential parameter if you don't use Azure AD authentication
    {
        $secpasswd = ConvertTo-SecureString $SSISDBServerAdminPassword -AsPlainText -Force
        $serverCreds = New-Object System.Management.Automation.PSCredential($SSISDBServerAdminUserName, $secpasswd)

        Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
            -DataFactoryName $DataFactoryName `
            -Name $AzureSSISName `
            -CatalogAdminCredential $serverCreds
    }
}

# Add custom setup parameters if you use standard/express custom setups
if(![string]::IsNullOrEmpty($SetupScriptContainerSasUri))
{
    Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
        -DataFactoryName $DataFactoryName `
        -Name $AzureSSISName `
        -SetupScriptContainerSasUri $SetupScriptContainerSasUri
}
if(![string]::IsNullOrEmpty($ExpressCustomSetup))
{
    if($ExpressCustomSetup -eq "RunCmdkey")
    {
        $addCmdkeyArgument = "YourFileShareServerName or YourAzureStorageAccountName.file.core.windows.net"
        $userCmdkeyArgument = "YourDomainName\YourUsername or azure\YourAzureStorageAccountName"
        $passCmdkeyArgument = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourPassword or YourAccessKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.CmdkeySetup($addCmdkeyArgument, $userCmdkeyArgument, $passCmdkeyArgument)
    }
    if($ExpressCustomSetup -eq "SetEnvironmentVariable")
    {
        $variableName = "YourVariableName"
        $variableValue = "YourVariableValue"
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.EnvironmentVariableSetup($variableName, $variableValue)
    }
    if($ExpressCustomSetup -eq "InstallAzurePowerShell")
    {
        $moduleVersion = "YourAzModuleVersion"
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.AzPowerShellSetup($moduleVersion)
    }
    if($ExpressCustomSetup -eq "SentryOne.TaskFactory")
    {
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }
    if($ExpressCustomSetup -eq "oh22is.SQLPhonetics.NET")
    {
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }
    if($ExpressCustomSetup -eq "oh22is.HEDDA.IO")
    {
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup)
    }
    if($ExpressCustomSetup -eq "KingswaySoft.IntegrationToolkit")
    {
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }
    if($ExpressCustomSetup -eq "KingswaySoft.ProductivityPack")
    {
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }    
    if($ExpressCustomSetup -eq "Theobald.XtractIS")
    {
        $jsonData = Get-Content -Raw -Path YourLicenseFile.json
        $jsonData = $jsonData -replace '\s',''
        $jsonData = $jsonData.replace('"','\"')
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString($jsonData)
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }
    if($ExpressCustomSetup -eq "AecorSoft.IntegrationService")
    {
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }
    if($ExpressCustomSetup -eq "CData.Standard")
    {
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }
    if($ExpressCustomSetup -eq "CData.Extended")
    {
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }    
    # Create an array of one or more express custom setups
    $setups = New-Object System.Collections.ArrayList
    $setups.Add($setup)

    Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
        -DataFactoryName $DataFactoryName `
        -Name $AzureSSISName `
        -ExpressCustomSetup $setups
}

# Add self-hosted integration runtime parameters if you configure a proxy for on-premises data accesss
if(![string]::IsNullOrEmpty($DataProxyIntegrationRuntimeName) -and ![string]::IsNullOrEmpty($DataProxyStagingLinkedServiceName))
{
    Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
        -DataFactoryName $DataFactoryName `
        -Name $AzureSSISName `
        -DataProxyIntegrationRuntimeName $DataProxyIntegrationRuntimeName `
        -DataProxyStagingLinkedServiceName $DataProxyStagingLinkedServiceName

    if(![string]::IsNullOrEmpty($DataProxyStagingPath))
    {
        Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
            -DataFactoryName $DataFactoryName `
            -Name $AzureSSISName `
            -DataProxyStagingPath $DataProxyStagingPath
    }
}

# Add public IP address parameters if you bring your own static public IP addresses
if(![string]::IsNullOrEmpty($FirstPublicIP) -and ![string]::IsNullOrEmpty($SecondPublicIP))
{
    $publicIPs = @($FirstPublicIP, $SecondPublicIP)
    Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
        -DataFactoryName $DataFactoryName `
        -Name $AzureSSISName `
        -PublicIPs $publicIPs
}
```

### <a name="start-the-integration-runtime"></a>Démarrer le runtime d’intégration

Exécutez les commandes suivantes pour démarrer le runtime d’intégration Azure-SSIS.

```powershell
write-host("##### Starting #####")
Start-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
    -DataFactoryName $DataFactoryName `
    -Name $AzureSSISName `
    -Force

write-host("##### Completed #####")
write-host("If any cmdlet is unsuccessful, please consider using -Debug option for diagnostics.")
```

> [!NOTE]
> En dehors des réglages personnalisés, ce processus devrait se terminer dans les 5 minutes. Toutefois, l’IR Azure-SSIS peut prendre jusqu’à 20-30 minutes pour joindre un réseau virtuel.
>
> Si vous utilisez le catalogue SSISDB, le service Data Factory se connecte à votre serveur de base de données pour le préparer. Il configure également les autorisations et les paramètres de votre réseau virtuel, s’ils sont spécifiés, et joint votre runtime d’intégration Azure-SSIS au réseau virtuel.
> 
> Quand vous provisionnez un runtime d’intégration Azure-SSIS, les composants Access Redistributable et Azure Feature Pack pour SSIS sont également installés. Ces composants assurent la connectivité aux fichiers Excel et Access ainsi qu’à diverses sources de données Azure, en plus des sources de données déjà prises en charge par les composants intégrés. Pour plus d’informations sur les composants intégrés/préinstallés, consultez [Composants intégrés/préinstallés sur Azure-SSIS IR](./built-in-preinstalled-components-ssis-integration-runtime.md). Pour plus d’informations sur les composants supplémentaires que vous pouvez installer, consultez [Personnalisation des configurations du runtime d’intégration Azure-SSIS](./how-to-configure-azure-ssis-ir-custom-setup.md).

### <a name="full-script"></a>Script complet

Voici le script complet qui crée un runtime d’intégration Azure-SSIS.

```powershell
### Azure Data Factory info
# If your input contains a PSH special character like "$", precede it with the escape character "`" - for example, "`$"
$SubscriptionName = "[your Azure subscription name]"
$ResourceGroupName = "[your Azure resource group name]"
# Data factory name - must be globally unique
$DataFactoryName = "[your data factory name]"
# For supported regions, see https://azure.microsoft.com/global-infrastructure/services/?products=data-factory&regions=all
$DataFactoryLocation = "EastUS"

### Azure-SSIS integration runtime info - This is a Data Factory compute resource for running SSIS packages.
$AzureSSISName = "[your Azure-SSIS IR name]"
$AzureSSISDescription = "[your Azure-SSIS IR description]"
# For supported regions, see https://azure.microsoft.com/global-infrastructure/services/?products=data-factory&regions=all
$AzureSSISLocation = "EastUS"
# For supported node sizes, see https://azure.microsoft.com/pricing/details/data-factory/ssis/
$AzureSSISNodeSize = "Standard_D8_v3"
# 1-10 nodes are currently supported
$AzureSSISNodeNumber = 2
# Azure-SSIS IR edition/license info: Standard or Enterprise
$AzureSSISEdition = "Standard" # Standard by default, whereas Enterprise lets you use advanced features on your Azure-SSIS IR
# Azure-SSIS IR hybrid usage info: LicenseIncluded or BasePrice
$AzureSSISLicenseType = "LicenseIncluded" # LicenseIncluded by default, whereas BasePrice lets you bring your own on-premises SQL Server license with Software Assurance to earn cost savings from the Azure Hybrid Benefit option
# For a Standard_D1_v2 node, up to four parallel executions per node are supported. For other nodes, up to (2 x number of cores) are currently supported.
$AzureSSISMaxParallelExecutionsPerNode = 8
# Custom setup info: Standard/express custom setups
$SetupScriptContainerSasUri = "" # OPTIONAL to provide a SAS URI of blob container for standard custom setup where your script and its associated files are stored
$ExpressCustomSetup = "[RunCmdkey|SetEnvironmentVariable|InstallAzurePowerShell|SentryOne.TaskFactory|oh22is.SQLPhonetics.NET|oh22is.HEDDA.IO|KingswaySoft.IntegrationToolkit|KingswaySoft.ProductivityPack|Theobald.XtractIS|AecorSoft.IntegrationService|CData.Standard|CData.Extended or leave it empty]" # OPTIONAL to configure an express custom setup without script
# Virtual network info: Classic or Azure Resource Manager
$VnetId = "[your virtual network resource ID or leave it empty]" # REQUIRED if you use an Azure SQL Database server with IP firewall rules/virtual network service endpoints or a managed instance with private endpoint to host SSISDB, or if you require access to on-premises data without configuring a self-hosted IR. We recommend an Azure Resource Manager virtual network, because classic virtual networks will be deprecated soon.
$SubnetName = "[your subnet name or leave it empty]" # WARNING: Use the same subnet as the one used for your Azure SQL Database server with virtual network service endpoints, or a different subnet from the one used for your managed instance with a private endpoint
# Public IP address info: OPTIONAL to provide two standard static public IP addresses with DNS name under the same subscription and in the same region as your virtual network
$FirstPublicIP = "[your first public IP address resource ID or leave it empty]"
$SecondPublicIP = "[your second public IP address resource ID or leave it empty]"

### SSISDB info
$SSISDBServerEndpoint = "[your Azure SQL Database server name.database.windows.net or managed instance name.DNS prefix.database.windows.net or managed instance name.public.DNS prefix.database.windows.net,3342 or leave it empty if you do not use SSISDB]" # WARNING: If you use SSISDB, ensure that there's no existing SSISDB on your database server, so we can prepare and manage one on your behalf
# Authentication info: SQL or Azure AD
$SSISDBServerAdminUserName = "[your server admin username for SQL authentication or leave it empty for Azure AD authentication]"
$SSISDBServerAdminPassword = "[your server admin password for SQL authentication or leave it empty for Azure AD authentication]"
# For the basic pricing tier, specify "Basic," not "B." For standard, premium, and elastic pool tiers, specify "S0," "S1," "S2," "S3," etc. See https://docs.microsoft.com/azure/sql-database/sql-database-resource-limits-database-server.
$SSISDBPricingTier = "[Basic|S0|S1|S2|S3|S4|S6|S7|S9|S12|P1|P2|P4|P6|P11|P15|…|ELASTIC_POOL(name = <elastic_pool_name>) for Azure SQL Database server or leave it empty for managed instance]"

### Self-hosted integration runtime info - This can be configured as a proxy for on-premises data access 
$DataProxyIntegrationRuntimeName = "" # OPTIONAL to configure a proxy for on-premises data access 
$DataProxyStagingLinkedServiceName = "" # OPTIONAL to configure a proxy for on-premises data access 
$DataProxyStagingPath = "" # OPTIONAL to configure a proxy for on-premises data access 

### Sign in and select a subscription
Connect-AzAccount
Select-AzSubscription -SubscriptionName $SubscriptionName

### Validate the connection to the database server
# Validate only if you use SSISDB and don't use a virtual network or Azure AD authentication
if(![string]::IsNullOrEmpty($SSISDBServerEndpoint))
{
    if([string]::IsNullOrEmpty($VnetId) -and [string]::IsNullOrEmpty($SubnetName))
    {
        if(![string]::IsNullOrEmpty($SSISDBServerAdminUserName) -and ![string]::IsNullOrEmpty($SSISDBServerAdminPassword))
        {
            $SSISDBConnectionString = "Data Source=" + $SSISDBServerEndpoint + ";User ID=" + $SSISDBServerAdminUserName + ";Password=" + $SSISDBServerAdminPassword
            $sqlConnection = New-Object System.Data.SqlClient.SqlConnection $SSISDBConnectionString;
            Try
            {
                $sqlConnection.Open();
            }
            Catch [System.Data.SqlClient.SqlException]
            {
                Write-Warning "Cannot connect to your Azure SQL Database server, exception: $_";
                Write-Warning "Please make sure the server you specified has already been created. Do you want to proceed? [Y/N]"
                $yn = Read-Host
                if(!($yn -ieq "Y"))
                {
                    Return;
                }
            }
        }
    }
}

### Configure a virtual network
# Make sure to run this script against the subscription to which the virtual network belongs
if(![string]::IsNullOrEmpty($VnetId) -and ![string]::IsNullOrEmpty($SubnetName))
{
    # Register to the Azure Batch resource provider
    $BatchApplicationId = "ddbf3205-c6bd-46ae-8127-60eb93363864"
    $BatchObjectId = (Get-AzADServicePrincipal -ServicePrincipalName $BatchApplicationId).Id
    Register-AzResourceProvider -ProviderNamespace Microsoft.Batch
    while(!(Get-AzResourceProvider -ProviderNamespace "Microsoft.Batch").RegistrationState.Contains("Registered"))
    {
    Start-Sleep -s 10
    }
    if($VnetId -match "/providers/Microsoft.ClassicNetwork/")
    {
        # Assign the VM contributor role to Microsoft.Batch
        New-AzRoleAssignment -ObjectId $BatchObjectId -RoleDefinitionName "Classic Virtual Machine Contributor" -Scope $VnetId
    }
}

### Create a data factory
Set-AzDataFactoryV2 -ResourceGroupName $ResourceGroupName `
    -Location $DataFactoryLocation `
    -Name $DataFactoryName

### Create an integration runtime
Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
    -DataFactoryName $DataFactoryName `
    -Name $AzureSSISName `
    -Description $AzureSSISDescription `
    -Type Managed `
    -Location $AzureSSISLocation `
    -NodeSize $AzureSSISNodeSize `
    -NodeCount $AzureSSISNodeNumber `
    -Edition $AzureSSISEdition `
    -LicenseType $AzureSSISLicenseType `
    -MaxParallelExecutionsPerNode $AzureSSISMaxParallelExecutionsPerNode `
    -VnetId $VnetId `
    -Subnet $SubnetName
       
# Add CatalogServerEndpoint, CatalogPricingTier, and CatalogAdminCredential parameters if you use SSISDB
if(![string]::IsNullOrEmpty($SSISDBServerEndpoint))
{
    Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
        -DataFactoryName $DataFactoryName `
        -Name $AzureSSISName `
        -CatalogServerEndpoint $SSISDBServerEndpoint `
        -CatalogPricingTier $SSISDBPricingTier

    if(![string]::IsNullOrEmpty($SSISDBServerAdminUserName) –and ![string]::IsNullOrEmpty($SSISDBServerAdminPassword)) # Add the CatalogAdminCredential parameter if you don't use Azure AD authentication
    {
        $secpasswd = ConvertTo-SecureString $SSISDBServerAdminPassword -AsPlainText -Force
        $serverCreds = New-Object System.Management.Automation.PSCredential($SSISDBServerAdminUserName, $secpasswd)

        Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
            -DataFactoryName $DataFactoryName `
            -Name $AzureSSISName `
            -CatalogAdminCredential $serverCreds
    }
}

# Add custom setup parameters if you use standard/express custom setups
if(![string]::IsNullOrEmpty($SetupScriptContainerSasUri))
{
    Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
        -DataFactoryName $DataFactoryName `
        -Name $AzureSSISName `
        -SetupScriptContainerSasUri $SetupScriptContainerSasUri
}
if(![string]::IsNullOrEmpty($ExpressCustomSetup))
{
    if($ExpressCustomSetup -eq "RunCmdkey")
    {
        $addCmdkeyArgument = "YourFileShareServerName or YourAzureStorageAccountName.file.core.windows.net"
        $userCmdkeyArgument = "YourDomainName\YourUsername or azure\YourAzureStorageAccountName"
        $passCmdkeyArgument = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourPassword or YourAccessKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.CmdkeySetup($addCmdkeyArgument, $userCmdkeyArgument, $passCmdkeyArgument)
    }
    if($ExpressCustomSetup -eq "SetEnvironmentVariable")
    {
        $variableName = "YourVariableName"
        $variableValue = "YourVariableValue"
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.EnvironmentVariableSetup($variableName, $variableValue)
    }
    if($ExpressCustomSetup -eq "InstallAzurePowerShell")
    {
        $moduleVersion = "YourAzModuleVersion"
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.AzPowerShellSetup($moduleVersion)
    }
    if($ExpressCustomSetup -eq "SentryOne.TaskFactory")
    {
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }
    if($ExpressCustomSetup -eq "oh22is.SQLPhonetics.NET")
    {
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }
    if($ExpressCustomSetup -eq "oh22is.HEDDA.IO")
    {
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup)
    }
    if($ExpressCustomSetup -eq "KingswaySoft.IntegrationToolkit")
    {
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }
    if($ExpressCustomSetup -eq "KingswaySoft.ProductivityPack")
    {
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }    
    if($ExpressCustomSetup -eq "Theobald.XtractIS")
    {
        $jsonData = Get-Content -Raw -Path YourLicenseFile.json
        $jsonData = $jsonData -replace '\s',''
        $jsonData = $jsonData.replace('"','\"')
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString($jsonData)
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }
    if($ExpressCustomSetup -eq "AecorSoft.IntegrationService")
    {
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }
    if($ExpressCustomSetup -eq "CData.Standard")
    {
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }
    if($ExpressCustomSetup -eq "CData.Extended")
    {
        $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
        $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
    }
    # Create an array of one or more express custom setups
    $setups = New-Object System.Collections.ArrayList
    $setups.Add($setup)

    Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
        -DataFactoryName $DataFactoryName `
        -Name $AzureSSISName `
        -ExpressCustomSetup $setups
}

# Add self-hosted integration runtime parameters if you configure a proxy for on-premises data accesss
if(![string]::IsNullOrEmpty($DataProxyIntegrationRuntimeName) -and ![string]::IsNullOrEmpty($DataProxyStagingLinkedServiceName))
{
    Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
        -DataFactoryName $DataFactoryName `
        -Name $AzureSSISName `
        -DataProxyIntegrationRuntimeName $DataProxyIntegrationRuntimeName `
        -DataProxyStagingLinkedServiceName $DataProxyStagingLinkedServiceName

    if(![string]::IsNullOrEmpty($DataProxyStagingPath))
    {
        Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
            -DataFactoryName $DataFactoryName `
            -Name $AzureSSISName `
            -DataProxyStagingPath $DataProxyStagingPath
    }
}

# Add public IP address parameters if you bring your own static public IP addresses
if(![string]::IsNullOrEmpty($FirstPublicIP) -and ![string]::IsNullOrEmpty($SecondPublicIP))
{
    $publicIPs = @($FirstPublicIP, $SecondPublicIP)
    Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
        -DataFactoryName $DataFactoryName `
        -Name $AzureSSISName `
        -PublicIPs $publicIPs
}

### Start the integration runtime
write-host("##### Starting #####")
Start-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
    -DataFactoryName $DataFactoryName `
    -Name $AzureSSISName `
    -Force

write-host("##### Completed #####")
write-host("If any cmdlet is unsuccessful, please consider using -Debug option for diagnostics.")
```

## <a name="use-an-azure-resource-manager-template-to-create-an-integration-runtime"></a>Utiliser un modèle Azure Resource Manager pour créer un runtime d’intégration

Dans le cadre de cette section, vous allez utiliser un modèle Azure Resource Manager pour créer le runtime d’intégration Azure-SSIS. Voici un exemple de procédure :

1. Créez un fichier JSON avec le modèle Azure Resource Manager suivant. Remplacez les valeurs entre chevrons (espaces réservés) par vos propres valeurs.

    ```json
    {
        "contentVersion": "1.0.0.0",
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "parameters": {},
        "variables": {},
        "resources": [{
            "name": "<Specify a name for your data factory>",
            "apiVersion": "2018-06-01",
            "type": "Microsoft.DataFactory/factories",
            "location": "East US",
            "properties": {},
            "resources": [{
                "type": "integrationruntimes",
                "name": "<Specify a name for your Azure-SSIS IR>",
                "dependsOn": [ "<The name of the data factory you specified at the beginning>" ],
                "apiVersion": "2018-06-01",
                "properties": {
                    "type": "Managed",
                    "typeProperties": {
                        "computeProperties": {
                            "location": "East US",
                            "nodeSize": "Standard_D8_v3",
                            "numberOfNodes": 1,
                            "maxParallelExecutionsPerNode": 8
                        },
                        "ssisProperties": {
                            "catalogInfo": {
                                "catalogServerEndpoint": "<Azure SQL Database server name>.database.windows.net",
                                "catalogAdminUserName": "<Azure SQL Database server admin username>",
                                "catalogAdminPassword": {
                                    "type": "SecureString",
                                    "value": "<Azure SQL Database server admin password>"
                                },
                                "catalogPricingTier": "Basic"
                            }
                        }
                    }
                }
            }]
        }]
    }
    ```

2. Pour déployer le modèle Azure Resource Manager, exécutez la commande `New-AzResourceGroupDeployment` comme indiqué dans l’exemple suivant. Dans l’exemple, `ADFTutorialResourceGroup` est le nom de votre groupe de ressources. `ADFTutorialARM.json` est le fichier qui contient la définition JSON pour votre fabrique de données et le runtime d’intégration Azure-SSIS.

    ```powershell
    New-AzResourceGroupDeployment -Name MyARMDeployment -ResourceGroupName ADFTutorialResourceGroup -TemplateFile ADFTutorialARM.json
    ```

    Cette commande crée votre fabrique de données et un runtime d’intégration Azure-SSIS dans cette fabrique, mais elle ne démarre pas le runtime d’intégration.

3. Pour démarrer l’IR Azure-SSIS, exécutez la commande `Start-AzDataFactoryV2IntegrationRuntime` :

    ```powershell
    Start-AzDataFactoryV2IntegrationRuntime -ResourceGroupName "<Resource Group Name>" `
        -DataFactoryName "<Data Factory Name>" `
        -Name "<Azure SSIS IR Name>" `
        -Force
    ```

> [!NOTE]
> En dehors des réglages personnalisés, ce processus devrait se terminer dans les 5 minutes. Toutefois, l’IR Azure-SSIS peut prendre jusqu’à 20-30 minutes pour joindre un réseau virtuel.
>
> Si vous utilisez le catalogue SSISDB, le service Data Factory se connecte à votre serveur de base de données pour le préparer. Il configure également les autorisations et les paramètres de votre réseau virtuel, s’ils sont spécifiés, et joint votre runtime d’intégration Azure-SSIS au réseau virtuel.
> 
> Quand vous provisionnez un runtime d’intégration Azure-SSIS, les composants Access Redistributable et Azure Feature Pack pour SSIS sont également installés. Ces composants assurent la connectivité aux fichiers Excel et Access ainsi qu’à diverses sources de données Azure, en plus des sources de données déjà prises en charge par les composants intégrés. Pour plus d’informations sur les composants intégrés/préinstallés, consultez [Composants intégrés/préinstallés sur Azure-SSIS IR](./built-in-preinstalled-components-ssis-integration-runtime.md). Pour plus d’informations sur les composants supplémentaires que vous pouvez installer, consultez [Personnalisation des configurations du runtime d’intégration Azure-SSIS](./how-to-configure-azure-ssis-ir-custom-setup.md).

## <a name="deploy-ssis-packages"></a>Déployer des packages SSIS

Si vous utilisez le catalogue SSISDB, vous pouvez y déployer vos packages et les exécuter sur votre runtime d’intégration Azure-SSIS à l’aide des outils SSDT ou SSMS compatibles avec Azure. Ces outils se connectent à votre serveur de base de données via son point de terminaison : 

- Pour un serveur Azure SQL Database, le format du point de terminaison de serveur est `<server name>.database.windows.net`.
- Pour une instance managée avec un point de terminaison privé, le format du point de terminaison de serveur est `<server name>.<dns prefix>.database.windows.net`.
- Pour une instance managée avec un point de terminaison public, le format du point de terminaison de serveur est `<server name>.public.<dns prefix>.database.windows.net,3342`. 

Si vous n’utilisez pas SSISDB, vous pouvez déployer vos packages sur un système de fichiers, Azure Files ou MSDB, dont l’hébergement est assuré par Azure SQL Managed Instance, puis les exécuter sur Azure-SSIS IR à l’aide des utilitaires en ligne de commande [dtutil](/sql/integration-services/dtutil-utility?view=sql-server-2017) et [AzureDTExec](./how-to-invoke-ssis-package-azure-enabled-dtexec.md). 

Pour plus d’informations, consultez [Déployer des projets ou des packages SSIS](/sql/integration-services/packages/deploy-integration-services-ssis-projects-and-packages?view=sql-server-ver15).

Dans les deux cas, vous pouvez également exécuter vos packages déployés sur Azure-SSIS IR à l’aide de l’activité Exécuter le package SSIS dans des pipelines Data Factory. Pour plus d’informations, consultez [Appeler une exécution de package SSIS comme activité Data Factory de première classe](./how-to-invoke-ssis-package-ssis-activity.md).

## <a name="next-steps"></a>Étapes suivantes

Consultez les autres rubriques relatives au runtime d’intégration Azure SSIS dans cette documentation :

- [Runtime d’intégration Azure SSIS](concepts-integration-runtime.md#azure-ssis-integration-runtime). Cet article fournit des informations sur les runtimes d’intégration en général, notamment sur le runtime d’intégration Azure-SSIS.
- [Monitor an Azure-SSIS IR](monitor-integration-runtime.md#azure-ssis-integration-runtime) (Surveiller le runtime d’intégration Azure-SSIS). Cet article vous montre comment récupérer et comprendre des informations sur votre runtime d’intégration Azure-SSIS.
- [Manage an Azure-SSIS IR](manage-azure-ssis-integration-runtime.md) (Gérer un runtime d’intégration Azure-SSIS). Cet article vous explique comment arrêter, démarrer ou supprimer votre Azure-SSIS IR. Il vous montre également comment effectuer un scale-out de votre IR Azure-SSIS en ajoutant des nœuds.
- [Déployer, exécuter et surveiller des packages SSIS dans Azure](/sql/integration-services/lift-shift/ssis-azure-deploy-run-monitor-tutorial)   
- [Se connecter à SSISDB dans Azure](/sql/integration-services/lift-shift/ssis-azure-connect-to-catalog-database)
- [Se connecter aux sources de données locales avec l’authentification Windows](/sql/integration-services/lift-shift/ssis-azure-connect-with-windows-auth) 
- [Planifier les exécutions de packages sur Azure](/sql/integration-services/lift-shift/ssis-azure-schedule-packages)