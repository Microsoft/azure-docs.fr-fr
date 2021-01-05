---
title: Configurer un runtime d’intégration auto-hébergé en tant que proxy pour SSIS
description: Découvrez comment configurer un runtime d’intégration auto-hébergé en tant que proxy pour Azure-SSIS Integration Runtime.
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
ms.date: 11/19/2020
ms.openlocfilehash: 82cc58d46061ec7b623d062ab0b0e5a1fdae7ddd
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96352216"
---
# <a name="configure-a-self-hosted-ir-as-a-proxy-for-an-azure-ssis-ir-in-azure-data-factory"></a>Configurer un IR auto-hébergé en tant que proxy pour Azure-SSIS IR dans Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Cet article explique comment exécuter des packages SQL Server Integration Services (SSIS) sur Azure-SSIS Integration Runtime (Azure-SSIS IR) dans Azure Data Factory avec un runtime d’intégration auto-hébergé (IR auto-hébergé) configuré en tant que proxy. 

Avec cette fonctionnalité, vous pouvez accéder aux données localement sans avoir à [joindre votre instance d’Azure-SSIS IR à un réseau virtuel](./join-azure-ssis-integration-runtime-virtual-network.md). Cette fonctionnalité est utile quand votre réseau d’entreprise a une configuration trop complexe ou une stratégie trop restrictive pour pouvoir y injecter votre instance Azure-SSIS IR.

Cette fonctionnalité décompose votre tâche de flux de données SSIS en deux tâches intermédiaires, le cas échéant : 
* **Tâche intermédiaire locale** : Cette tâche exécute votre composant de flux de données qui se connecte à un magasin de données local sur votre IR auto-hébergé. Elle déplace les données du magasin de données local vers une zone de transit dans votre stockage d’objets blob Azure, ou vice versa.
* **Tâche intermédiaire cloud** : Cette tâche exécute votre composant de flux de données qui ne se connecte pas à un magasin de données local sur votre Azure-SSIS IR. Elle déplace les données de la zone de transit dans votre stockage d’objets blob Azure vers un magasin de données cloud, ou vice versa.

Si votre tâche de flux de données déplace des données de l’environnement local vers le cloud, la première et la deuxième tâches intermédiaires sont des tâches locales et cloud, respectivement. Si votre tâche de flux de données déplace des données du cloud vers l’environnement local, la première et la deuxième tâches intermédiaires sont des tâches cloud et locales, respectivement. Si votre tâche de flux de données déplace des données sur l’environnement local, la première et la deuxième tâches intermédiaires sont toutes deux des tâches locales. Si votre tâche de flux de données déplace des données du cloud vers le cloud, cette fonctionnalité n’est pas applicable.

Cette fonctionnalité vous permet aussi, entre autres, de configurer votre IR auto-hébergé dans des régions qui ne sont pas encore prises en charge par Azure-SSIS IR, et d’autoriser l’adresse IP statique publique de votre IR auto-hébergé sur le pare-feu de vos sources de données.

## <a name="prepare-the-self-hosted-ir"></a>Préparer l’IR auto-hébergé

Pour utiliser cette fonctionnalité, vous devez d’abord créer une fabrique de données et y configurer une instance d’Azure-SSIS IR. Si ce n’est déjà fait, suivez les instructions fournies dans [Configurer un runtime d’intégration Azure-SSIS IR](./tutorial-deploy-ssis-packages-azure.md).

Vous configurez ensuite votre IR auto-hébergé dans la fabrique de données où votre instance Azure-SSIS IR est configurée. Pour ce faire, consultez [Créer un IR auto-hébergé](./create-self-hosted-integration-runtime.md).

Pour finir, vous téléchargez et installez la dernière version de l’IR auto-hébergé, ainsi que les pilotes et runtime supplémentaires, sur votre ordinateur local ou machine virtuelle Azure en effectuant les étapes suivantes :
- Téléchargez et installez la dernière version de l’[IR auto-hébergé](https://www.microsoft.com/download/details.aspx?id=39717).
- Si vous utilisez des connecteurs OLEDB (Object Linking and Embedding Database), ODBC (Open Database Connectivity) ou ADO.NET dans vos packages, téléchargez et installez les pilotes appropriés sur la machine où votre runtime d’intégration auto-hébergé est installé, si ce n’est pas déjà fait.  

  Si vous utilisez la version antérieure du pilote OLEDB pour SQL Server (SQL Server Native Client [SQLNCLI]), [téléchargez la version 64 bits](https://www.microsoft.com/download/details.aspx?id=50402).  

  Si vous utilisez la dernière version du pilote OLEDB pour SQL Server (MSOLEDBSQL), [téléchargez la version 64 bits](https://www.microsoft.com/download/details.aspx?id=56730).  
  
  Si vous utilisez des pilotes OLEDB/ODBC/ADO.NET pour d’autres systèmes de base de données, comme PostgreSQL, MySQL, Oracle, etc., vous pouvez télécharger les versions 64 bits à partir de leur site web.
- Si ce n’est pas déjà fait, [téléchargez et installez la version 64 bits du runtime Visual C++ (VC)](https://www.microsoft.com/download/details.aspx?id=40784) sur l’ordinateur où votre IR auto-hébergé est installé.

### <a name="enable-windows-authentication-for-on-premises-staging-tasks"></a>Utiliser l’authentification Windows pour des tâches de préproduction locales

Si vos tâches intermédiaires locales sur votre IR auto-hébergé nécessitent l’authentification Windows, [configurez vos packages SSIS pour qu’ils utilisent la même authentification Windows](/sql/integration-services/lift-shift/ssis-azure-connect-with-windows-auth?view=sql-server-ver15). 

Vos tâches intermédiaires locales seront appelées avec le compte de service de l’IR auto-hébergé (*NT SERVICE\DIAHostService* par défaut), et vos magasins de données seront accessibles avec le compte d’authentification Windows. Les deux comptes nécessitent que certaines stratégies de sécurité leur soient attribuées. Sur l’ordinateur IR auto-hébergé, accédez à **Stratégie de sécurité locale** > **Stratégies locales** > **Attribution des droits utilisateur**, puis effectuez les étapes suivantes :

1. Attribuez les stratégies *Ajuster les quotas de mémoire pour un processus* et *Remplacer un jeton de niveau processus* au compte de service de l’IR auto-hébergé. Ceci doit se produire automatiquement quand vous installez votre IR auto-hébergé avec le compte de service par défaut. Si ce n’est pas le cas, attribuez ces stratégies manuellement. Si vous utilisez un autre compte de service, attribuez-lui les mêmes stratégies.

1. Attribuez la stratégie *Se connecter en tant que service* au compte d’authentification Windows.

## <a name="prepare-the-azure-blob-storage-linked-service-for-staging"></a>Préparer le service lié Stockage Blob Azure pour la préproduction

Si ce n’est pas déjà fait, créez un service lié Stockage Blob Azure dans la fabrique de données où votre Azure-SSIS IR est configuré. Pour ce faire, consultez [Créer un service lié Azure Data Factory](./quickstart-create-data-factory-portal.md#create-a-linked-service). Veillez à effectuer les opérations suivantes :
- Pour **Magasin de données**, sélectionnez **Stockage Blob Azure**.  
- Pour **Se connecter via le runtime d'intégration**, sélectionnez **AutoResolveIntegrationRuntime** (et non votre Azure-SSIS IR ou votre IR auto-hébergé), car nous utilisons le runtime d'intégration Azure par défaut pour récupérer les informations d'accès à votre Stockage Blob Azure.
- Pour **Méthode d’authentification**, sélectionnez **Clé de compte**, **URI SAS**, **Principal du service** ou **Identité managée**.  

>[!TIP]
>Si vous sélectionnez la méthode du **Principal de service**, accordez au minimum à votre principal de service le rôle de *Contributeur aux données Blob du stockage*. Pour plus d’informations, consultez [Connecteur Stockage Blob Azure](connector-azure-blob-storage.md#linked-service-properties). Si vous sélectionnez la méthode **Identité managée**, accordez à votre identité managée par ADF des rôles appropriés pour accéder à Stockage Blob Azure. Pour plus d’informations, consultez [Accéder à Stockage Blob Azure en utilisant l’authentification Azure Active Directory avec l’identité managée par ADF](/sql/integration-services/connection-manager/azure-storage-connection-manager?view=sql-server-ver15#managed-identities-for-azure-resources-authentication).

![Préparer le service lié de stockage d’objets blob Azure pour la préproduction](media/self-hosted-integration-runtime-proxy-ssis/shir-azure-blob-storage-linked-service.png)

## <a name="configure-an-azure-ssis-ir-with-your-self-hosted-ir-as-a-proxy"></a>Configurer une instance Azure-SSIS IR avec votre IR auto-hébergé en tant que proxy

Une fois votre IR auto-hébergé et votre service lié de stockage d’objets blob Azure prêts pour la préproduction, vous pouvez configurer votre instance Azure-SSIS IR nouvelle ou existante avec l’IR auto-hébergé en tant que proxy dans votre application ou portail de fabrique de données. Avant cela, cependant, si votre instance Azure-SSIS IR existante est déjà en cours d’exécution, arrêtez-la, puis redémarrez-la.

1. Dans le volet **Installation d’Integration Runtime**, ignorez les sections **Paramètres généraux** et **Paramètres SQL** en sélectionnant **Suivant**. 

1. Dans la boîte de dialogue **Paramètres avancés**, effectuez les étapes suivantes :

   1. Cochez la case **Configurer un runtime d’intégration auto-hébergé en tant que proxy pour votre Azure-SSIS Integration Runtime**. 

   1. Dans la liste déroulante **Runtime d’intégration auto-hébergé**, sélectionnez votre IR auto-hébergé existant en tant que proxy pour l’instance Azure-SSIS IR.

   1. Dans la liste déroulante **Service lié de stockage de préproduction**, sélectionnez votre service lié de stockage d’objets blob Azure existant ou créez-en un pour la préproduction.

   1. Dans la zone **Chemin d’accès intermédiaire**, spécifiez un conteneur d’objets blob dans le compte de stockage d’objets blob Azure sélectionné, ou laissez ce champ vide afin d’utiliser un conteneur par défaut pour la préproduction.

   1. Sélectionnez **Continuer**.

   ![Paramètres avancés avec un IR auto-hébergé](./media/tutorial-create-azure-ssis-runtime-portal/advanced-settings-shir.png)

Vous pouvez également configurer votre instance Azure-SSIS IR, qu’elle soit nouvelle ou existante, avec l’IR auto-hébergé en tant que proxy à l’aide de PowerShell.

```powershell
$ResourceGroupName = "[your Azure resource group name]"
$DataFactoryName = "[your data factory name]"
$AzureSSISName = "[your Azure-SSIS IR name]"
# Self-hosted integration runtime info - This can be configured as a proxy for on-premises data access 
$DataProxyIntegrationRuntimeName = "" # OPTIONAL to configure a proxy for on-premises data access 
$DataProxyStagingLinkedServiceName = "" # OPTIONAL to configure a proxy for on-premises data access 
$DataProxyStagingPath = "" # OPTIONAL to configure a proxy for on-premises data access 

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
Start-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
    -DataFactoryName $DataFactoryName `
    -Name $AzureSSISName `
    -Force
```

## <a name="enable-ssis-packages-to-connect-by-proxy"></a>Activer les packages SSIS pour la connexion par proxy

En utilisant la version la plus récente de l’extension SSDT with SSIS Projects pour Visual Studio ou un programme d’installation autonome, vous trouverez une nouvelle propriété `ConnectByProxy` qui a été ajoutée dans les gestionnaires de connexions pour les composants de flux de données pris en charge.
* [Télécharger l’extension SSIS Projects pour Visual Studio](https://marketplace.visualstudio.com/items?itemName=SSIS.SqlServerIntegrationServicesProjects)
* [Télécharger le programme d’installation autonome](/sql/ssdt/download-sql-server-data-tools-ssdt?view=sql-server-2017#ssdt-for-vs-2017-standalone-installer)   

Lors de la conception de nouveaux packages qui contiennent des tâches de flux de données avec des composants qui accèdent aux données localement, vous pouvez activer cette propriété en lui affectant la valeur *True* dans le volet **Propriétés** des gestionnaires de connexions appropriés.

![Activer la propriété ConnectByProxy](media/self-hosted-integration-runtime-proxy-ssis/shir-connection-manager-properties.png)

Vous pouvez également activer cette propriété quand vous exécutez des packages existants, sans qu’il soit nécessaire de les modifier manuellement un par un.  Nous avons deux options :
- **Option A** : Ouvrir, regénérer et redéployer le projet contenant ces packages avec la dernière version de SSDT pour une exécution sur votre instance Azure-SSIS IR. Vous pouvez ensuite activer la propriété en lui affectant la valeur *true* pour les gestionnaires de connexions appropriés. Quand vous exécutez des packages à partir de SSMS, ces gestionnaires de connexions apparaissent sous l’onglet **Gestionnaires de connexions** de la fenêtre contextuelle **Exécuter le package**.

  ![Activer la propriété ConnectByProxy 2](media/self-hosted-integration-runtime-proxy-ssis/shir-connection-managers-tab-ssms.png)

  Vous pouvez aussi activer la propriété en lui affectant la valeur *True* pour les gestionnaires de connexions appropriés qui apparaissent sous l’onglet **Gestionnaires de connexions** de l’activité [Exécuter le package SSIS](./how-to-invoke-ssis-package-ssis-activity.md) quand vous exécutez des packages dans des pipelines Data Factory.
  
  ![Activer la propriété ConnectByProxy 3](media/self-hosted-integration-runtime-proxy-ssis/shir-connection-managers-tab-ssis-activity.png)

- **Option B** : Redéployer le projet contenant ces packages pour une exécution sur votre instance de SSIS IR. Vous pouvez ensuite activer la propriété en spécifiant son chemin de propriété, `\Package.Connections[YourConnectionManagerName].Properties[ConnectByProxy]`, et en lui affectant la valeur *True* comme substitution de propriété sous l’onglet **Avancé** de la fenêtre contextuelle **Exécuter le package** quand vous exécutez des packages à partir de SSMS.

  ![Activer la propriété ConnectByProxy 4](media/self-hosted-integration-runtime-proxy-ssis/shir-advanced-tab-ssms.png)

  Vous pouvez aussi activer la propriété en spécifiant son chemin de propriété, `\Package.Connections[YourConnectionManagerName].Properties[ConnectByProxy]`, et en lui affectant la valeur *True* comme substitution de propriété sous l’onglet **Substitutions de propriété** de l’activité [Exécuter le package SSIS](./how-to-invoke-ssis-package-ssis-activity.md) quand vous exécutez des packages dans des pipelines Data Factory.
  
  ![Activer la propriété ConnectByProxy 5](media/self-hosted-integration-runtime-proxy-ssis/shir-property-overrides-tab-ssis-activity.png)

## <a name="debug-the-on-premises-and-cloud-staging-tasks"></a>Déboguer les tâches intermédiaires locales et cloud

Sur votre IR auto-hébergé, vous trouverez les journaux du runtime dans le dossier *C:\ProgramData\SSISTelemetry* et les journaux d’exécution des tâches intermédiaires locales dans le dossier *C:\ProgramData\SSISTelemetry\ExecutionLog*.  Vous trouverez les journaux d’exécution des tâches de préproduction cloud dans votre SSISDB, dans les chemins de journalisation spécifiés ou dans Azure Monitor, selon que vous stockez vos packages dans SSISDB, que vous activez l’[intégration d’Azure Monitor](./monitor-using-azure-monitor.md#monitor-ssis-operations-with-azure-monitor), etc. Vous trouverez également les ID uniques des tâches intermédiaires locales dans les journaux d’exécution des tâches intermédiaires dans le cloud. 

![ID unique de la première tâche intermédiaire](media/self-hosted-integration-runtime-proxy-ssis/shir-first-staging-task-guid.png)

Si vous avez créé des tickets de support client, vous pouvez sélectionner le bouton **Envoyer des journaux** sous l’onglet **Diagnostics** de **Microsoft Integration Runtime Configuration Manager**, qui est installé sur votre runtime d’intégration auto-hébergé pour envoyer les journaux récents d’opérations/exécution que nous allons examiner.

## <a name="billing-for-the-on-premises-and-cloud-staging-tasks"></a>Facturation des tâches intermédiaires locales et cloud

Les tâches intermédiaires locales qui s’exécutent sur votre IR auto-hébergé sont facturées séparément, tout comme les activités de déplacement de données qui s’exécutent sur un IR auto-hébergé. Cela est indiqué dans l’article [Tarification Pipeline de données Azure Data Factory](https://azure.microsoft.com/pricing/details/data-factory/data-pipeline/).

Les tâches intermédiaires dans le cloud qui s’exécutent sur votre instance d’Azure-SSIS IR ne sont pas facturées séparément, mais votre instance d’Azure-SSIS IR en cours d’exécution est facturée comme décrit dans l’article [Tarification d’Azure-SSIS IR](https://azure.microsoft.com/pricing/details/data-factory/ssis/).

## <a name="enable-custom3rd-party-components"></a>Activer des composants personnalisés/tiers 

Pour permettre à vos composants personnalisés/tiers d’accéder aux données locales en utilisant le runtime d’intégration auto-hébergé en tant que proxy pour Azure-SSIS IR, suivez ces instructions :

1. Installez vos composants personnalisés/tiers ciblant SQL Server 2017 sur Azure-SSIS IR via des [configurations personnalisées standard/express](./how-to-configure-azure-ssis-ir-custom-setup.md).

1. Créez les clés de Registre DTSPath suivantes sur le runtime d'intégration auto-hébergé, si elles n'existent pas encore :
   1. `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Microsoft SQL Server\140\SSIS\Setup\DTSPath` défini sur `C:\Program Files\Microsoft SQL Server\140\DTS\`
   1. `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\Microsoft SQL Server\140\SSIS\Setup\DTSPath` défini sur `C:\Program Files (x86)\Microsoft SQL Server\140\DTS\`
   
1. Installez vos composants personnalisés/tiers ciblant SQL Server 2017 sur le runtime d’intégration auto-hébergé sous le DTSPath ci-dessus et vérifiez que votre processus d’installation :

   1. Crée les dossiers `<DTSPath>`, `<DTSPath>/Connections`, `<DTSPath>/PipelineComponents` et `<DTSPath>/UpgradeMappings` s’ils n’existent pas déjà.
   
   1. Crée votre propre fichier XML pour les mappages d’extension dans le dossier `<DTSPath>/UpgradeMappings`.
   
   1. Installe tous les assemblys référencés par vos assemblys de composants personnalisés/tiers dans le Global Assembly Cache (GAC).

Voici des exemples de nos partenaires, [Theobald Software](https://kb.theobald-software.com/xtract-is/XIS-for-Azure-SHIR) et [Aecorsoft](https://www.aecorsoft.com/blog/2020/11/8/using-azure-data-factory-to-bring-sap-data-to-azure-via-self-hosted-ir-and-ssis-ir), qui ont adapté leurs composants pour utiliser notre configuration personnalisée express et notre IR auto-hébergé comme proxy pour Azure-SSIS IR.

## <a name="enforce-tls-12"></a>Appliquer le protocole TLS 1.2

Si vous devez utiliser un chiffrement renforcé ou un protocole réseau plus sécurisé (TLS 1.2) et désactiver les anciennes versions SSL/TLS sur votre IR auto-hébergé, vous pouvez télécharger et exécuter le script *main.cmd* qui se trouve dans le dossier *CustomSetupScript/UserScenarios/TLS 1.2* de notre conteneur de préversions publiques.  À l’aide d’[Explorateur Stockage Azure](https://storageexplorer.com/), vous pouvez vous connecter à notre conteneur de préversions publiques en entrant l’URI SAS suivant :

`https://ssisazurefileshare.blob.core.windows.net/publicpreview?sp=rl&st=2020-03-25T04:00:00Z&se=2025-03-25T04:00:00Z&sv=2019-02-02&sr=c&sig=WAD3DATezJjhBCO3ezrQ7TUZ8syEUxZZtGIhhP6Pt4I%3D`

## <a name="current-limitations"></a>Limites actuelles

- Seuls les composants de flux de données intégrés/préinstallés sur Azure-SSIS IR Édition Standard, à l’exception des composants Hadoop/HDFS/DQS, sont actuellement pris en charge ; consultez [tous les composants intégrés/préinstallés sur Azure-SSIS IR](./built-in-preinstalled-components-ssis-integration-runtime.md).
- Seuls les composants de flux de données personnalisés/tiers écrits en code managé (.NET Framework) sont actuellement pris en charge ; ceux qui sont écrits en code natif (C++ ) ne sont actuellement pas pris en charge.
- Le changement des valeurs des variables dans les tâches de préproduction locales et cloud n’est actuellement pas pris en charge.
- Le changement des valeurs des variables de type « object » dans des tâches de préproduction locales ne sera pas reflété dans les autres tâches.
- *ParameterMapping* n'est actuellement pas pris en charge dans la source OLEDB. Pour résoudre ce problème, utilisez *Commande SQL à partir d'une variable* en tant que mode d'accès (*AccessMode*) et utilisez *Expression* pour insérer vos variables/paramètres dans une commande SQL. À titre d’illustration, consultez le package *ParameterMappingSample.dtsx* qui se trouve dans le dossier *SelfHostedIRProxy/Limitations* de notre conteneur de préversions publiques. À l’aide d’Explorateur Stockage Azure, vous pouvez vous connecter à notre conteneur de préversions publiques en entrant l’URI SAS ci-dessus.

## <a name="next-steps"></a>Étapes suivantes

Après avoir configuré votre IR auto-hébergé en tant que proxy pour votre instance Azure-SSIS IR, vous pouvez déployer et exécuter vos packages pour accéder aux données locales sous forme d’activités Exécuter le package SSIS dans des pipelines ADF. Pour découvrir comment procéder, consultez [Exécuter un package SSIS avec l’activité Exécuter le Package SSIS dans Azure Data Factory](./how-to-invoke-ssis-package-ssis-activity.md).