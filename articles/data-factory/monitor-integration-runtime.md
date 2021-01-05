---
title: Superviser un runtime d'intégration dans Azure Data Factory
description: Découvrez comment surveiller plusieurs types de runtime d’intégration dans Azure Data Factory.
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 08/11/2020
author: dcstwh
ms.author: weetok
manager: anandsub
ms.openlocfilehash: fa71dc1e6b3a09827f2ad3d9f714622da5a36222
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96862443"
---
# <a name="monitor-an-integration-runtime-in-azure-data-factory"></a>Surveiller un runtime d’intégration dans Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]
  
**Integration Runtime** est l’infrastructure de calcul utilisée par Azure Data Factory (ADF) pour fournir diverses capacités d’intégration de données sur différents environnements réseau. Azure Data Factory propose trois types de runtimes d’intégration :

- Runtime d’intégration Azure
- Runtime d’intégration auto-hébergé
- Azure-SQL Server Integration Services (SSIS) Integration Runtime

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Pour obtenir l’état d’une instance de runtime d’intégration (IR), exécutez la commande PowerShell suivante : 

```powershell
Get-AzDataFactoryV2IntegrationRuntime -DataFactoryName MyDataFactory -ResourceGroupName MyResourceGroup -Name MyAzureIR -Status
``` 

Cette cmdlet renvoie des informations différentes en fonction du type de runtime d’intégration. Cet article explique les propriétés et les états de chacun des types de runtime d’intégration.  

## <a name="azure-integration-runtime"></a>Runtime d’intégration Azure

La ressource de calcul d’un runtime d’intégration Azure est entièrement gérée dans Azure, en toute flexibilité. Le tableau suivant fournit des descriptions des propriétés renvoyées par la commande **Get-AzDataFactoryV2IntegrationRuntime** :

### <a name="properties"></a>Propriétés

Le tableau suivant fournit des descriptions des propriétés renvoyées par la cmdlet pour un runtime d’intégration Azure :

| Propriété | Description |
-------- | ------------- | 
| Nom | Nom du runtime d’intégration Azure. |  
| State | État du runtime d’intégration Azure. | 
| Emplacement | Emplacement du runtime d’intégration Azure. Pour plus d’informations sur l’emplacement d’un runtime d’intégration d’Azure, voir [Introduction au runtime d’intégration](concepts-integration-runtime.md). |
| DataFactoryName | Nom de la fabrique de données à laquelle le runtime d’intégration Azure appartient. | 
| ResourceGroupName | Nom du groupe de ressources auquel la fabrique de données appartient.  |
| Description | Description du runtime d’intégration Azure.  |

### <a name="status"></a>Statut

Le tableau suivant indique les états possibles d’un runtime d’intégration Azure :

| Statut | Commentaires/Scénarios | 
| ------ | ------------------ |
| En ligne | Le runtime d’intégration Azure est en ligne et prêt à être utilisé. | 
| Hors connexion | Le runtime d’intégration Azure est hors connexion en raison d’une erreur interne. |

## <a name="self-hosted-integration-runtime"></a>Runtime d’intégration auto-hébergé

Le tableau suivant fournit des descriptions des propriétés renvoyées par la cmdlet Get-AzDataFactoryV2IntegrationRuntime. 

> [!NOTE] 
> Les propriétés renvoyées et l’état contiennent des informations sur le runtime d’intégration auto-hébergé dans son ensemble et sur chacun des nœuds du runtime.  

### <a name="properties"></a>Propriétés

Le tableau suivant fournit des descriptions des propriétés de surveillance pour **chaque nœud** :

| Propriété | Description | 
| -------- | ----------- | 
| Nom | Nom du runtime d’intégration auto-hébergé et des nœuds qui lui sont associés. Le nœud est un ordinateur Windows local sur lequel le runtime d’intégration auto-hébergé est installé. |  
| Statut | État du runtime d’intégration auto-hébergé dans son ensemble et de chacun des nœuds. Exemple : En ligne/Hors connexion/Limité/etc. Pour plus d’informations sur ces états, voir la section suivante. | 
| Version | Version du runtime d’intégration auto-hébergé et de chacun des nœuds. La version du runtime d’intégration auto-hébergé est déterminée d’après la version de la majorité des nœuds dans le groupe. S’il existe des nœuds de différentes versions dans la configuration du runtime d’intégration auto-hébergé, seuls les nœuds dont le numéro de version est identique à celui du runtime d’intégration auto-hébergé logique fonctionnent correctement. Les autres sont en mode limité et ont besoin d’une mise à jour manuelle (uniquement si la mise à jour automatique échoue). | 
| Mémoire disponible | Mémoire disponible dans un nœud de runtime d’intégration auto-hébergé. Cette valeur est un instantané en quasi temps réel. | 
| Utilisation du processeur | Utilisation du processeur dans un nœud de runtime d’intégration auto-hébergé. Cette valeur est un instantané en quasi temps réel. |
| Réseau (entrée/sortie) | Utilisation du réseau dans un nœud de runtime d’intégration auto-hébergé. Cette valeur est un instantané en quasi temps réel. | 
| Tâches simultanées (en cours d’exécution/limite) | **Exécution en cours**. Nombre de travaux ou tâches qui s’exécutent sur chaque nœud. Cette valeur est un instantané en quasi temps réel. <br/><br/>**Limite**. La limite correspond au nombre maximal de travaux simultanés pour chaque nœud. Cette valeur est définie selon la taille de l’ordinateur. Vous pouvez augmenter la limite pour monter en puissance l’exécution de tâches simultanées dans les scénarios avancés, quand des activités expirent alors même que le processeur, la mémoire ou le réseau est sous-utilisé. Cette fonctionnalité est également disponible avec un runtime d’intégration d’auto-hébergé à nœud unique. |
| Role | Il existe deux types de rôles dans un runtime d’intégration auto-hébergé à nœuds multiples : répartiteur et rôle de travail. Tous les nœuds sont des rôles de travail, ce qui signifie qu’ils peuvent tous être utilisés pour exécuter des tâches. Il n’existe qu’un seul nœud répartiteur. Il est utilisé pour extraire des tâches ou des travaux auprès de services cloud et pour les répartir entre différents nœuds rôles de travail. Le nœud répartiteur est également un nœud rôle de travail. |

Certains paramètres de ces propriétés semblent plus logiques dans une configuration d’au moins deux nœuds dans le runtime d’intégration auto-hébergé (autrement dit, dans un scénario de scale-out).

#### <a name="concurrent-jobs-limit"></a>Limite de tâches simultanées

La valeur par défaut de la limite de tâches simultanées est définie selon la taille de la machine. Les facteurs utilisés pour calculer cette valeur dépendent de la quantité de RAM et du nombre de cœurs de processeur de la machine. Ainsi, plus il y a de cœurs et de mémoire, plus la limite par défaut de tâches simultanées est élevée.

Vous effectuez un scale-out en augmentant le nombre de nœuds. Lorsque vous augmentez le nombre de nœuds, la limite de tâches simultanées est constituée par la somme des valeurs de limite de tâches simultanées de tous les nœuds disponibles.  Par exemple, si un seul nœud vous permet d’exécuter un maximum de douze tâches simultanées, l’ajout de trois nœuds similaires supplémentaires vous permet d’exécuter un maximum de 48 tâches simultanées (c’est-à-dire 4 x 12). Nous vous recommandons d’augmenter la limite de tâches simultanées uniquement lorsque vous constatez une faible utilisation des ressources avec des valeurs par défaut sur chaque nœud.

Vous pouvez remplacer la valeur par défaut calculée dans le portail Azure. Sélectionnez Auteur > Connexions > Runtimes d’intégration > Modifier > Nœuds > Modifier la valeur de tâches simultanées par nœud. Vous pouvez également utiliser la commande PowerShell [update-Azdatafactoryv2integrationruntimenode](/powershell/module/az.datafactory/update-Azdatafactoryv2integrationruntimenode#examples).
  
### <a name="status-per-node"></a>État (par nœud)

Le tableau suivant indique les états possibles d’un nœud de runtime d’intégration Azure auto-hébergé :

| Statut | Description |
| ------ | ------------------ | 
| En ligne | Le nœud est connecté au service Data Factory. |
| Hors connexion | Le nœud est hors connexion. |
| Mise à niveau | Le nœud est en cours de mise à jour automatique. |
| Limité | En raison d’un problème de connectivité. Éventuellement dû à un problème de port HTTP 8060, à un problème de connectivité du bus de service ou à un problème de synchronisation des informations d’identification. |
| Inactif | La configuration du nœud est différente de celle de la majorité des autres nœuds. |

Un nœud peut être inactif quand il ne parvient pas à se connecter à d’autres nœuds.

### <a name="status-overall-self-hosted-integration-runtime"></a>État (runtime d’intégration auto-hébergé dans son ensemble)

Le tableau suivant indique les états possibles d’un runtime d’intégration Azure auto-hébergé. Cet état dépend des états de tous les nœuds appartenant au runtime. 

| Statut | Description |
| ------ | ----------- | 
| Inscription requise | Aucun nœud n’est encore inscrit dans ce runtime d’intégration auto-hébergé. |
| En ligne | Tous les nœuds sont en ligne. |
| Hors connexion | Aucun nœud n’est en ligne. |
| Limité | L’état de tous les nœuds de ce runtime d’intégration auto-hébergé n’est pas sain. Cet état sert d’avertissement. Il peut indiquer que certains nœuds sont en panne. Cela peut être dû à un problème de synchronisation des informations d’identification sur le nœud répartiteur ou le nœud rôle de travail. |

Utilisez la cmdlet **Get-AzDataFactoryV2IntegrationRuntimeMetric**. Il s’agit d’extraire la charge utile JSON qui contient les propriétés détaillées du runtime d’intégration auto-hébergé et les valeurs des instantanés pendant toute la durée d’exécution de la cmdlet.

```powershell
Get-AzDataFactoryV2IntegrationRuntimeMetric -name $integrationRuntimeName -ResourceGroupName $resourceGroupName -DataFactoryName $dataFactoryName | ConvertTo-Json 
```

Exemple de sortie (suppose que deux nœuds sont associés à ce runtime d’intégration auto-hébergé) :

```json
{
    "IntegrationRuntimeName":  "<Name of your integration runtime>",
    "ResourceGroupName":  "<Resource Group Name>",
    "DataFactoryName":  "<Data Factory Name>",
    "Nodes":  [
        {
            "NodeName":  "<Node Name>",
            "AvailableMemoryInMB":  <Value>,
            "CpuUtilization":  <Value>,
            "ConcurrentJobsLimit":  <Value>,
            "ConcurrentJobsRunning":  <Value>,
            "MaxConcurrentJobs":  <Value>,
            "SentBytes":  <Value>,
            "ReceivedBytes":  <Value>
        },
        {
            "NodeName":  "<Node Name>",
            "AvailableMemoryInMB":  <Value>,
            "CpuUtilization":  <Value>,
            "ConcurrentJobsLimit":  <Value>,
            "ConcurrentJobsRunning":  <Value>,
            "MaxConcurrentJobs":  <Value>,
            "SentBytes":  <Value>,
            "ReceivedBytes":  <Value>
        }

    ]
} 
```

## <a name="azure-ssis-integration-runtime"></a>Runtime d’intégration Azure SSIS

Azure-SSIS IR est un cluster complètement managé de machines virtuelles Azure (VM ou nœuds), dédié à l’exécution de packages SSIS. Il existe différentes méthodes permettant d’appeler des exécutions de packages SSIS sur Azure-SSIS IR : par exemple, avec SQL Server Data Tools (SSDT) compatible Azure, l’utilitaire de ligne de commande AzureDTExec, T-SQL sur SQL Server Management Studio (SSMS)/SQL Server Agent et les activités Exécuter un package SSIS des pipelines ADF. Azure-SSIS IR n’exécute aucune autre activité ADF. Après le provisionnement, vous pouvez superviser ses propriétés et états globaux et propres aux nœuds à l’aide d’Azure PowerShell, du Portail Azure et d’Azure Monitor.

### <a name="monitor-the-azure-ssis-integration-runtime-with-azure-powershell"></a>Supervision d’Azure-SSIS Integration Runtime avec Azure PowerShell

Utilisez la cmdlet Azure PowerShell suivante pour superviser les propriétés et états globaux et propres aux nœuds d’Azure-SSIS IR.

```powershell
Get-AzDataFactoryV2IntegrationRuntime -DataFactoryName $DataFactoryName -Name $AzureSSISName -ResourceGroupName $ResourceGroupName -Status
```

#### <a name="properties"></a>Propriétés

Le tableau suivant donne une description des propriétés retournées par la cmdlet ci-dessus pour un runtime Azure-SSIS IR.

| Propriété/état              | Description                  |
| ---------------------------- | ---------------------------- |
| CreateTime                   | Heure UTC de création du runtime Azure-SSIS IR. |
| Nœuds                        | Nœuds alloués/disponibles du runtime Azure-SSIS IR avec états propres aux nœuds (en cours de démarrage/disponible/en cours de recyclage/indisponible) et erreurs actionnables. |
| OtherErrors                  | Erreurs actionnables non propres aux nœuds du runtime Azure-SSIS IR. |
| LastOperation                | Résultat de la dernière opération de démarrage/d’arrêt du runtime Azure-SSIS IR avec erreur(s) actionnables(s) en cas d’échec. |
| State                        | État global (initial/en cours de démarrage/démarré/en cours d’arrêt/arrêté) du runtime Azure-SSIS IR. |
| Emplacement                     | Emplacement du runtime Azure-SSIS IR. |
| NodeSize                     | Taille de chacun des nœuds du runtime Azure-SSIS IR. |
| NodeCount                    | Nombre de nœuds du runtime Azure-SSIS IR. |
| MaxParallelExecutionsPerNode | Nombre maximal d’exécutions parallèles par nœud dans le runtime Azure-SSIS IR. |
| CatalogServerEndpoint        | Point de terminaison du serveur Azure SQL Database ou de l’instance gérée existant permettant d’héberger le catalogue SSIS (SSISDB). |
| CatalogAdminUserName         | Nom d’utilisateur administrateur du serveur Azure SQL Database ou de l’instance gérée existant. ADF utilise ces informations pour préparer et gérer automatiquement la base de données SSISDB. |
| CatalogAdminPassword         | Mot de passe d’administrateur du serveur Azure SQL Database ou de l’instance gérée existant. |
| CatalogPricingTier           | Niveau tarifaire de la base de données SSISDB hébergée par le serveur Azure SQL Database.  Ne s’applique pas à Azure SQL Managed Instance hébergeant la base de données SSISDB. |
| VNetId                       | ID de la ressource de réseau virtuel que le runtime Azure-SSIS IR doit rejoindre. |
| Subnet                       | Nom du sous-réseau que le runtime Azure-SSIS IR doit rejoindre. |
| id                           | ID de la ressource du runtime Azure-SSIS IR. |
| Type                         | Type (managé/auto-hébergé) du runtime Azure-SSIS IR. |
| ResourceGroupName            | Nom du groupe de ressources Azure dans lequel la fabrique ADF et le runtime Azure-SSIS IR ont été créés. |
| DataFactoryName              | Nom de la fabrique ADF. |
| Nom                         | Nom du runtime Azure-SSIS IR. |
| Description                  | Description du runtime Azure-SSIS IR. |
  
#### <a name="status-per-azure-ssis-ir-node"></a>État (par nœud Azure-SSIS IR)

Le tableau suivant indique les états possibles d’un nœud Azure-SSIS IR :

| État propre au nœud | Description |
| -------------------- | ----------- | 
| Démarrage en cours             | Ce nœud est en cours de préparation. |
| Disponible            | Ce nœud est prêt pour vous permettre de déployer ou d’exécuter des packages SSIS. |
| Recyclage            | Ce nœud est en cours de réparation ou de redémarrage. |
| Non disponible          | Ce nœud n’est pas prêt pour vous permettre de déployer/d’exécuter des packages SSIS. Il présente des erreurs/problèmes actionnables que vous pouvez résoudre. |

#### <a name="status-overall-azure-ssis-ir"></a>État (runtime Azure-SSIS IR global)

Le tableau suivant indique les états globaux possibles d’un runtime Azure-SSIS IR. L’état global dépend des états combinés de tous les nœuds appartenant au runtime Azure-SSIS IR. 

| État global | Description | 
| -------------- | ----------- | 
| Initial        | Les nœuds du runtime Azure-SSIS IR n’ont pas été alloués/préparés. | 
| Démarrage en cours       | Les nœuds du runtime Azure-SSIS IR sont en cours d’allocation/préparation et la facturation a commencé. |
| Démarré        | Les nœuds du runtime Azure-SSIS IR ont été alloués/préparés et sont prêts pour vous permettre de déployer/d’exécuter des packages SSIS. |
| En cours d’arrêt       | Les nœuds du runtime Azure-SSIS IR sont en cours de libération. |
| Arrêté        | Les nœuds du runtime Azure-SSIS IR ont été libérés et la facturation a été arrêtée. |

### <a name="monitor-the-azure-ssis-integration-runtime-in-azure-portal"></a>Supervision du runtime Azure-SSIS IR sur le Portail Azure

Pour superviser votre runtime Azure-SSIS IR sur le Portail Azure, accédez à la page **Runtimes d’intégration** du hub **Superviser** sur l’interface utilisateur ADF, où vous pouvez voir tous vos runtimes d’intégration.

![Supervision de tous les runtimes d’intégration](media/monitor-integration-runtime/monitor-integration-runtimes.png)

Ensuite, sélectionnez le nom de votre runtime Azure-SSIS IR pour ouvrir sa page de supervision, sur laquelle vous pouvez voir ses propriétés et états globaux/propres aux nœuds. Sur cette page, en fonction de la façon dont vous configurez les paramètres généraux, de déploiement et avancés de votre Azure-SSIS IR, vous trouverez différentes vignettes d’informations/fonctionnelles.  Les vignettes d’information **TYPE** et **RÉGION** affichent respectivement le type et la région de votre Azure-SSIS IR. La vignette d’information **TAILLE DE NŒUD** affiche la référence SKU (série edition_VM tier_VM SSIS), le nombre de cœurs de processeur et la taille de RAM par nœud pour votre Azure-SSIS IR. La vignette d’information **NŒUD(S) EN COURS D’EXÉCUTION / DEMANDÉ(S)** compare le nombre de nœuds en cours d’exécution au nombre total de nœuds précédemment demandés pour votre Azure-SSIS IR. Les vignettes fonctionnelles sont décrites plus en détail ci-dessous.

![Supervision du runtime Azure-SSIS IR](media/monitor-integration-runtime/monitor-azure-ssis-integration-runtime.png)

#### <a name="status-tile"></a>Vignette ÉTAT

Sur la vignette **ÉTAT** de la page de supervision de votre runtime Azure-SSIS IR, vous pouvez voir son état global, par exemple, **En cours d’exécution** ou **Arrêté**. Si vous sélectionnez l’état **En cours d’exécution**, la fenêtre qui s’ouvre comporte un bouton **Arrêter** en direct permettant d’arrêter le runtime Azure-SSIS IR. Si vous sélectionnez l’état **Arrêté**, la fenêtre qui s’ouvre comporte un bouton **Démarrer** en direct permettant de démarrer le runtime Azure-SSIS IR. La fenêtre contextuelle comporte également un bouton **Exécuter le package SSIS** pour générer automatiquement un pipeline ADF avec l’activité Exécuter le package SSIS qui s’exécute sur votre Azure-SSIS IR (consultez [Exécuter un package SSIS avec l’activité Exécuter le Package SSIS dans Azure Data Factory](./how-to-invoke-ssis-package-ssis-activity.md)) et une zone de texte **ID de ressource**, à partir de laquelle vous pouvez copier votre ID de ressource Azure-SSIS IR (`/subscriptions/YourAzureSubscripton/resourcegroups/YourResourceGroup/providers/Microsoft.DataFactory/factories/YourADF/integrationruntimes/YourAzureSSISIR`). Le suffixe de votre ID de ressource Azure-SSIS IR qui contient vos noms ADF et Azure-SSIS IR forme un ID de cluster qui peut être utilisé pour acheter des composants SSIS/sous licence supplémentaires auprès d’éditeurs de logiciels indépendants (ISV) et les associer à votre Azure-SSIS IR (voir [Installer des composants personnalisés payants, ou sous licence, pour le runtime d’intégration Azure-SSIS](./how-to-develop-azure-ssis-ir-licensed-components.md)).

![Supervision du runtime Azure-SSIS IR – Vignette ÉTAT](media/monitor-integration-runtime/monitor-azure-ssis-integration-runtime-status.png)

#### <a name="ssisdb-server-endpoint-tile"></a>Vignette POINT DE TERMINAISON DE SERVEUR SSISDB

Si vous utilisez un modèle de déploiement de projet selon lequel les packages sont stockés dans une base de données SSISDB hébergée par votre serveur Azure SQL Database ou votre instance gérée, la vignette **POINT DE TERMINAISON DE SERVEUR SSISDB** apparaît sur la page de supervision de votre runtime Azure-SSIS IR (consultez [Configuration des paramètres de déploiement d’un runtime Azure-SSIS IR](./tutorial-deploy-ssis-packages-azure.md#deployment-settings-page)). Sur cette vignette, vous pouvez sélectionner un lien désignant votre serveur Azure SQL Database ou votre instance gérée pour afficher une fenêtre vous permettant de copier le point de terminaison de serveur à partir d’une zone de texte et de l’utiliser en cas de connexion à partir de SSMS pour déployer, configurer, exécuter et gérer vos packages. Dans la fenêtre indépendante, vous pouvez également sélectionner le lien **Afficher les paramètres de la base de données Azure SQL Database ou de l’instance gérée** pour reconfigurer/redimensionner votre base de données SSISDB sur le Portail Azure.

![Supervision du runtime Azure-SSIS IR – Vignette SSISDB](media/monitor-integration-runtime/monitor-azure-ssis-integration-runtime-ssisdb.png)

#### <a name="proxy--staging-tile"></a>Vignette PROXY / STAGING

Si vous téléchargez, installez et configurez le runtime d’intégration auto-hébergé (SHIR) en tant que proxy pour votre Azure-SSIS IR afin d’accéder aux données localement, vous verrez la vignette **PROXY / STAGING** sur votre page d’analyse Azure-SSIS IR (voir [Configurer un IR auto-hébergé en tant que proxy pour Azure-SSIS IR dans Azure Data Factory](./self-hosted-integration-runtime-proxy-ssis.md)). Sur cette vignette, vous pouvez sélectionner un lien désignant votre SHIR pour ouvrir sa page d’analyse. Vous pouvez également sélectionner un autre lien désignant votre Stockage Blob Azure pour la mise en lots afin de reconfigurer son service lié.

#### <a name="validate-vnet--subnet-tile"></a>Vignette VALIDER LE RÉSEAU VIRTUEL/SOUS-RÉSEAU

Si vous joignez votre runtime Azure-SSIS IR à un réseau virtuel, la vignette **VALIDER LE RÉSEAU VIRTUEL/SOUS-RÉSEAU** apparaît sur la page de supervision de votre runtime Azure-SSIS IR (consultez [Jonction d’un runtime Azure-SSIS IR à un réseau virtuel](./join-azure-ssis-integration-runtime-virtual-network.md)). Sur cette vignette, vous pouvez sélectionner un lien désignant votre réseau virtuel et votre sous-réseau pour afficher une fenêtre vous permettant de copier l’ID de votre ressource de réseau virtuel (`/subscriptions/YourAzureSubscripton/resourceGroups/YourResourceGroup/providers/Microsoft.Network/virtualNetworks/YourARMVNet`) et le nom de votre sous-réseau à partir de zones de texte, ainsi que de valider la configuration de votre réseau virtuel et de votre sous-réseau pour vérifier que les trafics réseau entrants/sortants et la gestion de votre runtime Azure-SSIS IR ne sont pas entravés.

![Supervision du runtime Azure-SSIS IR – Vignette VALIDER](media/monitor-integration-runtime/monitor-azure-ssis-integration-runtime-validate.png)

#### <a name="diagnose-connectivity-tile"></a>Vignette DIAGNOSTIQUER LA CONNECTIVITÉ

Sur la vignette **DIAGNOSTIQUER LA CONNECTIVITÉ** de la page de supervision de votre runtime Azure-SSIS IR, vous pouvez sélectionner le lien **Tester la connexion** pour ouvrir une fenêtre vous permettant de vérifier les connexions entre votre runtime Azure-SSIS IR et les magasins de packages/configuration/données correspondants, ainsi que les services de gestion, au moyen de leur nom de domaine complet (FQDN)/adresse IP et de leur port désigné (consultez [Test des connexions à partir d’un runtime Azure-SSIS IR](./ssis-integration-runtime-diagnose-connectivity-faq.md)).

![Capture d’écran montrant où tester les connexions entre Azure-SSIS IR et les magasins de données, configurations et packages concernés.](media/monitor-integration-runtime/monitor-azure-ssis-integration-runtime-diagnose.png)

#### <a name="static-public-ip-addresses-tile"></a>Vignette ADRESSES IP PUBLIQUES STATIQUES

Si vous apportez vos propres adresses IP publiques statiques pour Azure-SSIS IR, la vignette **ADRESSES IP PUBLIQUES STATIQUES** apparaît sur votre page d’analyse de Azure-SSIS IR (voir [Intégration de vos propres adresses IP publiques statiques pour Azure-SSIS IR](./join-azure-ssis-integration-runtime-virtual-network.md#publicIP)). Sur cette vignette, vous pouvez sélectionner des liens désignant vos premières/deuxième adresses IP publiques statiques pour Azure-SSIS IR pour afficher une fenêtre contextuelle, dans laquelle vous pouvez copier leur ID de ressource (`/subscriptions/YourAzureSubscripton/resourceGroups/YourResourceGroup/providers/Microsoft.Network/publicIPAddresses/YourPublicIPAddress`) à partir d’une zone de texte. Dans la fenêtre contextuelle, vous pouvez également sélectionner le lien **Afficher votre premier/deuxième paramètre d’adresse IP publique statique** pour gérer votre première/deuxième adresse IP publique statique dans le Portail Azure.

![Capture d’écran montrant où désigner votre première ou votre deuxième adresse IP publique statique.](media/monitor-integration-runtime/monitor-azure-ssis-integration-runtime-static.png)

#### <a name="package-stores-tile"></a>Vignette MAGASINS DE PACKAGES

Si vous utilisez un modèle de déploiement de package selon lequel les packages sont stockés un système de fichiers/Azure Files/une base de données SQL Server (MSDB) hébergé par votre instance Azure SQL Managed Instance et géré au moyen de magasins de packages Azure-SSIS IR, la vignette **MAGASINS DE PACKAGES** apparaît sur la page de supervision de votre runtime Azure-SSIS IR (consultez [Configuration des paramètres de déploiement d’un runtime Azure-SSIS IR](./tutorial-deploy-ssis-packages-azure.md#deployment-settings-page)). Sur cette vignette, vous pouvez sélectionner un lien désignant le nombre de magasins de packages attachés à votre runtime Azure-SSIS IR pour afficher une fenêtre vous permettant de reconfigurer les services liés correspondant à vos magasins de packages Azure-SSIS IR qui se trouvent au-dessus du système de fichiers/d’Azure Files/de la base de données MSDB hébergé par votre instance Azure SQL Managed Instance.

![Supervision du runtime Azure-SSIS IR – Vignette PACKAGE](media/monitor-integration-runtime/monitor-azure-ssis-integration-runtime-package.png)

#### <a name="errors-tile"></a>Vignette ERREUR(S)

En cas de problèmes liés au démarrage/à l’arrêt/à la maintenance/à la mise à niveau de votre runtime Azure-SSIS IR, une vignette **ERREUR(S)** supplémentaire apparaît sur la page de supervision de votre runtime Azure-SSIS IR. Sur cette vignette, vous pouvez sélectionner un lien désignant le nombre d’erreurs générées par votre runtime Azure-SSIS IR pour afficher une fenêtre vous permettant de voir ces erreurs plus en détail et de les copier pour trouver les solutions recommandées dans notre guide de dépannage (consultez [Résolution des problèmes d’un runtime Azure-SSIS IR](./ssis-integration-runtime-management-troubleshoot.md)).

![Supervision du runtime Azure-SSIS IR – Vignette DIAGNOSTIQUER](media/monitor-integration-runtime/monitor-azure-ssis-integration-runtime-error.png)

### <a name="monitor-the-azure-ssis-integration-runtime-with-azure-monitor"></a>Supervision d’Azure-SSIS Integration Runtime avec Azure Monitor

Pour superviser votre runtime Azure-SSIS IR avec Azure Monitor, consultez [Supervision des opérations SSIS avec Azure Monitor](./monitor-using-azure-monitor.md#monitor-ssis-operations-with-azure-monitor).

### <a name="more-info-about-the-azure-ssis-integration-runtime"></a>Plus d’informations sur le runtime d’intégration Azure-SSIS

Pour plus d’informations sur le runtime d’intégration Azure-SSIS, voir les articles suivants :

- [Azure-SSIS Integration Runtime](concepts-integration-runtime.md#azure-ssis-integration-runtime) (Runtime d’intégration Azure-SSIS). Cet article fournit des informations conceptuelles sur les runtimes d’intégration en général, notamment sur le runtime d’intégration Azure-SSIS. 
- [Didacticiel : deploy SSIS packages to Azure](./tutorial-deploy-ssis-packages-azure.md) (Déployer des packages SSIS vers Azure). Cet article fournit des instructions détaillées pour créer votre Azure-SSIS IR et utiliser Azure SQL Database pour héberger le catalogue SSIS (SSISDB). 
- [Procédure : Créer un runtime d’intégration Azure-SSIS](create-azure-ssis-integration-runtime.md). Cet article s’appuie sur le didacticiel et fournit des instructions sur la façon d’utiliser Azure SQL Managed Instance pour héberger SSISDB. 
- [Manage an Azure-SSIS IR](manage-azure-ssis-integration-runtime.md) (Gérer un runtime d’intégration Azure-SSIS). Cet article vous explique comment démarrer, arrêter ou supprimer votre Azure-SSIS IR. Il montre également comment procéder à une mise à l’échelle en ajoutant des nœuds supplémentaires. 
- [Joindre un runtime d’intégration Azure-SSIS à un réseau virtuel](join-azure-ssis-integration-runtime-virtual-network.md). Cet article fournit des instructions pour joindre votre Azure-SSIS IR à un réseau virtuel.

## <a name="next-steps"></a>Étapes suivantes
Consultez les articles suivants pour surveiller les pipelines de différentes façons : 

- [Démarrages rapides : create a data factory](quickstart-create-data-factory-dot-net.md) (Créer une fabrique de données)
- [Use Azure Monitor to monitor Data Factory pipelines](monitor-using-azure-monitor.md) (Utiliser Azure Monitor pour surveiller les pipelines Data Factory)
