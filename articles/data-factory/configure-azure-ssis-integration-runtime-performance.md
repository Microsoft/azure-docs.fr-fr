---
title: Configurer les performances pour Azure-SSIS Integration Runtime
description: Découvrez comment configurer les propriétés d’Azure-SSIS Integration Runtime pour de hautes performances
services: data-factory
ms.date: 01/10/2018
ms.topic: conceptual
ms.service: data-factory
ms.workload: data-services
author: swinarko
ms.author: sawinark
ms.reviewer: ''
manager: anandsub
ms.openlocfilehash: f0fcd61230d68d7b26017237e2b7e0465fcb1f07
ms.sourcegitcommit: fb3c846de147cc2e3515cd8219d8c84790e3a442
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92635318"
---
# <a name="configure-the-azure-ssis-integration-runtime-for-high-performance"></a>Configurer Azure-SSIS Integration Runtime pour de hautes performances

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]


Cet article décrit comment configurer Azure-SSIS Integration Runtime pour de hautes performances. Le Azure-SSIS Integration Runtime vous permet de déployer et d’exécuter des packages SQL Server Integration Services (SSIS) dans Azure. Pour en savoir plus sur Azure-SSIS Integration Runtime, consultez l’article [Runtime d’intégration](concepts-integration-runtime.md#azure-ssis-integration-runtime). Pour plus d’informations sur le déploiement et l’exécution de packages SSIS sur Azure, consultez [Migration « lift-and-shift » des charges de travail SQL Server Integration Services dans le cloud](/sql/integration-services/lift-shift/ssis-azure-lift-shift-ssis-packages-overview).

> [!IMPORTANT]
> Cet article contient des résultats et observations de performance suite à des tests interne effectués par des membres de l’équipe de développement SSIS. Vos résultats peuvent varier. Effectuez vos propres tests avant de finaliser vos paramètres de configuration, qui affectent les coûts et les performances.

## <a name="properties-to-configure"></a>Propriétés à configurer

La partie suivante d’un script de configuration montre les propriétés que vous pouvez configurer lorsque vous créez une infrastructureAzure-SSIS Integration Runtime. Pour le script PowerShell et la description complets, consultez [Déployer des packages SQL Server Integration Services pour Azure](tutorial-deploy-ssis-packages-azure-powershell.md).

```powershell
# If your input contains a PSH special character, e.g. "$", precede it with the escape character "`" like "`$"
$SubscriptionName = "[your Azure subscription name]"
$ResourceGroupName = "[your Azure resource group name]"
$DataFactoryName = "[your data factory name]"
# For supported regions, see https://azure.microsoft.com/global-infrastructure/services/?products=data-factory&regions=all
$DataFactoryLocation = "EastUS"

### Azure-SSIS integration runtime information - This is a Data Factory compute resource for running SSIS packages
$AzureSSISName = "[specify a name for your Azure-SSIS IR]"
$AzureSSISDescription = "[specify a description for your Azure-SSIS IR]"
# For supported regions, see https://azure.microsoft.com/global-infrastructure/services/?products=data-factory&regions=all
$AzureSSISLocation = "EastUS"
# For supported node sizes, see https://azure.microsoft.com/pricing/details/data-factory/ssis/
$AzureSSISNodeSize = "Standard_D8_v3"
# 1-10 nodes are currently supported
$AzureSSISNodeNumber = 2
# Azure-SSIS IR edition/license info: Standard or Enterprise
$AzureSSISEdition = "Standard" # Standard by default, while Enterprise lets you use advanced/premium features on your Azure-SSIS IR
# Azure-SSIS IR hybrid usage info: LicenseIncluded or BasePrice
$AzureSSISLicenseType = "LicenseIncluded" # LicenseIncluded by default, while BasePrice lets you bring your existing SQL Server license with Software Assurance to earn cost savings from Azure Hybrid Benefit (AHB) option
# For a Standard_D1_v2 node, up to 4 parallel executions per node are supported, but for other nodes, up to max(2 x number of cores, 8) are currently supported
$AzureSSISMaxParallelExecutionsPerNode = 8
# Custom setup info
$SetupScriptContainerSasUri = "" # OPTIONAL to provide SAS URI of blob container where your custom setup script and its associated files are stored
# Virtual network info: Classic or Azure Resource Manager
$VnetId = "[your virtual network resource ID or leave it empty]" # REQUIRED if you use Azure SQL Database with virtual network service endpoints/SQL Managed Instance/on-premises data, Azure Resource Manager virtual network is recommended, Classic virtual network will be deprecated soon
$SubnetName = "[your subnet name or leave it empty]" # WARNING: Please use the same subnet as the one used with your Azure SQL Database with virtual network service endpoints or a different subnet than the one used for your SQL Managed Instance

### SSISDB info
$SSISDBServerEndpoint = "[your server name or managed instance name.DNS prefix].database.windows.net" # WARNING: Please ensure that there is no existing SSISDB, so we can prepare and manage one on your behalf
# Authentication info: SQL or Azure Active Directory (AAD)
$SSISDBServerAdminUserName = "[your server admin username for SQL authentication or leave it empty for AAD authentication]"
$SSISDBServerAdminPassword = "[your server admin password for SQL authentication or leave it empty for AAD authentication]"
$SSISDBPricingTier = "[Basic|S0|S1|S2|S3|S4|S6|S7|S9|S12|P1|P2|P4|P6|P11|P15|…|ELASTIC_POOL(name = <elastic_pool_name>) for Azure SQL Database or leave it empty for SQL Managed Instance]"
```

## <a name="azuressislocation"></a>AzureSSISLocation
**AzureSSISLocation** est l’emplacement du nœud du rôle de travail du runtime d’intégration. Le nœud Worker maintient une connexion constante avec la base de données du catalogue SSIS (SSISDB) dans Azure SQL Database. Définissez **AzureSSISLocation** au même emplacement que le [serveur SQL logique](../azure-sql/database/logical-servers.md) qui héberge SSISDB. Ainsi, le runtime d’intégration fonctionnera le plus efficacement possible.

## <a name="azuressisnodesize"></a>AzureSSISNodeSize
Data Factory, y compris le runtime d’intégration Azure-SSIS, prend en charge les options suivantes :
-   Standard\_A4\_v2
-   Standard\_A8\_v2
-   Standard\_D1\_v2
-   Standard\_D2\_v2
-   Standard\_D3\_v2
-   Standard\_D4\_v2
-   Standard\_D2\_v3
-   Standard\_D4\_v3
-   Standard\_D8\_v3
-   Standard\_D16\_v3
-   Standard\_D32\_v3
-   Standard\_D64\_v3
-   Standard\_E2\_v3
-   Standard\_E4\_v3
-   Standard\_E8\_v3
-   Standard\_E16\_v3
-   Standard\_E32\_v3
-   Standard\_E64\_v3

Dans le test interne officiel effectué par l’équipe d’ingénierie SSIS, la série D semble mieux appropriée à l’exécution du package SSIS que la série A.

-   Le rapport performance/prix de la série D est supérieur à celui de la série A, et le rapport performance/prix de la série v3 est supérieur à celui de la série v2.
-   Le débit de la série D est supérieur à celui de la série A au même prix, et le débit de la série v3 est supérieur à celui de la série v2 au même prix.
-   Les nœuds de série v2 d’Azure-SSIS IR ne convenant pas pour une installation personnalisée, veuillez utiliser les nœuds de série v3 à la place. Si vous utilisez déjà les nœuds de série v2, optez dès que possible pour l’utilisation des nœuds de série v3.
-   La série E est une taille de machine virtuelle à mémoire optimisée qui fournit un rapport mémoire/U/C plus élevé que les autres machines. Si votre package requiert beaucoup de mémoire, vous pouvez envisager de choisir la machine virtuelle de série E.

### <a name="configure-for-execution-speed"></a>Configurer pour la vitesse d’exécution
Si vous n’avez pas de nombreux packages à exécuter et que vous voulez exécuter rapidement des packages, utilisez les informations données dans le tableau suivant pour choisir un type de machine virtuelle approprié à votre scénario.

Ces données représentent une seule exécution de package sur un seul nœud de rôle de travail. Le package charge 3 millions d’enregistrements avec des colonnes de prénoms et de noms depuis le stockage Blob Azure, génère une colonne avec des noms complets et écrit les enregistrements dont le nom complet a plus de 20 caractères dans le stockage Blob Azure.

L’axe des Y est le nombre de packages dont l’exécution s’est terminée en une heure. Notez qu’il ne s’agit que d’un résultat de test d’un package gourmand en mémoire. Si vous souhaitez connaître le débit de votre package, il est recommandé d’effectuer le test par vous-même.

![Vitesse d’exécution du package Azure-SSIS Integration Runtime](media/configure-azure-ssis-integration-runtime-performance/ssisir-execution-speedV2.png)

### <a name="configure-for-overall-throughput"></a>Configurer pour le débit global

Si vous avez beaucoup de packages à exécuter et que vous vous souciez surtout du débit global, utilisez les informations données dans le tableau suivant pour choisir un type de machine virtuelle approprié à votre scénario.

L’axe des Y est le nombre de packages dont l’exécution s’est terminée en une heure. Notez qu’il ne s’agit que d’un résultat de test d’un package gourmand en mémoire. Si vous souhaitez connaître le débit de votre package, il est recommandé d’effectuer le test par vous-même.

![Débit global maximal d’Azure-SSIS Integration Runtime](media/configure-azure-ssis-integration-runtime-performance/ssisir-overall-throughputV2.png)

## <a name="azuressisnodenumber"></a>AzureSSISNodeNumber

**AzureSSISNodeNumber** ajuste l’évolutivité du runtime d’intégration. Le débit du runtime d’intégration est proportionnel au **AzureSSISNodeNumber** . Définir le **AzureSSISNodeNumber** une petite valeur dans un premier temps, surveiller le débit du runtime d’intégration, puis modifiez la valeur pour votre scénario. Pour reconfigurer le nœud du rôle de travail, consultez [Gérer Azure-SSIS Integration Runtime](manage-azure-ssis-integration-runtime.md).

## <a name="azuressismaxparallelexecutionspernode"></a>AzureSSISMaxParallelExecutionsPerNode

Lorsque vous utilisez déjà un nœud de rôle de travail puissant pour exécuter des packages, l’augmentation de **AzureSSISMaxParallelExecutionsPerNode** peut augmenter le débit global du runtime d’intégration. Si vous souhaitez augmenter la valeur maximale, vous devez utiliser Azure PowerShell pour mettre à jour **AzureSSISMaxParallelExecutionsPerNode** . Vous pouvez estimer la valeur appropriée sur la base du coût de votre package et sur les configurations suivantes des nœuds de rôle de travail. Pour plus d’informations, consultez [Tailles de machines virtuelles à usage général](../virtual-machines/sizes-general.md).

| Taille             | Processeurs virtuels | Mémoire : Gio | Stockage temporaire (SSD) en Gio | Débit de stockage temporaire max. : IOPS / MBps en lecture / MBps en écriture | Disques de données max. / débit : E/S par seconde | Nombre max de cartes réseau / Performance réseau attendue (Mbits/s) |
|------------------|------|-------------|------------------------|------------------------------------------------------------|-----------------------------------|------------------------------------------------|
| Standard\_D1\_v2 | 1    | 3,5         | 50                     | 3000 / 46 / 23                                             | 2 / 2 x 500                         | 2 / 750                                        |
| Standard\_D2\_v2 | 2    | 7           | 100                    | 6000 / 93 / 46                                             | 4 / 4 x 500                         | 2 / 1 500                                       |
| Standard\_D3\_v2 | 4    | 14          | 200                    | 12000 / 187 / 93                                           | 8 / 8 x 500                         | 4 / 3 000                                       |
| Standard\_D4\_v2 | 8    | 28          | 400                    | 24000 / 375 / 187                                          | 16 / 16 x 500                       | 8 / 6 000                                       |
| Standard\_A4\_v2 | 4    | 8           | 40                     | 4000 / 80 / 40                                             | 8 / 8 x 500                         | 4 / 1 000                                       |
| Standard\_A8\_v2 | 8    | 16          | 80                     | 8000 / 160 / 80                                            | 16 / 16 x 500                       | 8 / 2 000                                       |
| Standard\_D2\_v3 | 2    | 8           | 50                     | 3000 / 46 / 23                                             | 4/6 x 500                         | 2 / 1 000                                       |
| Standard\_D4\_v3 | 4    | 16          | 100                    | 6000 / 93 / 46                                             | 8/12 x 500                        | 2 / 2 000                                       |
| Standard\_D8\_v3 | 8    | 32          | 200                    | 12000 / 187 / 93                                           | 16/24 x 500                       | 4/4000                                       |
| Standard\_D16\_v3| 16   | 64          | 400                    | 24000 / 375 / 187                                          | 32/48 x 500                        | 8 / 8000                                       |
| Standard\_D32\_v3| 32   | 128         | 800                    | 48000 / 750 / 375                                          | 32/96 x 500                       | 8 / 16 000                                      |
| Standard\_D64\_v3| 64   | 256         | 1 600                   | 96000 / 1000 / 500                                         | 32/192 x 500                      | 8 / 30000                                      |
| Standard\_E2\_v3 | 2    | 16          | 50                     | 3000 / 46 / 23                                             | 4/6 x 500                         | 2 / 1 000                                       |
| Standard\_E4\_v3 | 4    | 32          | 100                    | 6000 / 93 / 46                                             | 8/12 x 500                        | 2 / 2 000                                       |
| Standard\_E8\_v3 | 8    | 64          | 200                    | 12000 / 187 / 93                                           | 16/24 x 500                       | 4/4000                                       |
| Standard\_E16\_v3| 16   | 128         | 400                    | 24000 / 375 / 187                                          | 32/48 x 500                       | 8 / 8000                                       |
| Standard\_E32\_v3| 32   | 256         | 800                    | 48000 / 750 / 375                                          | 32/96 x 500                       | 8 / 16 000                                      |
| Standard\_E64\_v3| 64   | 432         | 1 600                   | 96000 / 1000 / 500                                         | 32/192 x 500                      | 8 / 30000                                      |

Voici les instructions à suivre pour définir la valeur correcte pour la propriété **AzureSSISMaxParallelExecutionsPerNode**  : 

1. Commencez par définir une petite valeur.
2. Augmentez-la par petites quantités pour vérifier si le débit global est amélioré.
3. Cessez d’augmenter la valeur lorsque le débit global atteint la valeur maximale.

## <a name="ssisdbpricingtier"></a>SSISDBPricingTier

**SSISDBPricingTier** est le niveau de tarification pour la base de données du catalogue SSIS (SSISDB) dans Azure SQL Database. Ce paramètre affecte le nombre maximal de rôles de travail dans l’instance IR, la vitesse de la file d’attente de l’exécution d’un package et la vitesse de chargement du journal d’exécution.

-   Si vous ne vous souciez pas de la vitesse d’exécution de la file d’attente du package et de chargement du journal d’exécution, vous pouvez choisir le niveau tarifaire le plus bas de la base de données. Azure SQL Database avec le tarif De base prend en charge 8 threads de travail dans une instance de runtime d’intégration.

-   Choisissez une base de données plus puissante que la base de données de base si le nombre de rôles de travail est supérieur à 8, ou si le nombre de cœurs est supérieur à 50. Dans le cas contraire, la base de données devient le goulot d’étranglement de l’instance du runtime d’intégration et les performances s’en ressentent.

-   Choisissez une base de données plus puissante, telles que s3, si le niveau de journalisation est mis sur Informations détaillées. Selon notre test interne non officiel, le niveau tarifaire s3 peut prendre en charge l’exécution du package SSIS avec 2 nœuds, 128 nombres parallèle et le niveau de journalisation détaillé.

Vous pouvez également ajuster le niveau tarifaire de la base de données en fonction des informations d’utilisation de l’[unité de transaction de base de données](../azure-sql/database/service-tiers-dtu.md) (DTU) disponibles sur le portail Azure.

## <a name="design-for-high-performance"></a>Conception pour de hautes performances
La conception d’un package SSIS à exécuter sur Azure diffère de la conception d’un package pour une exécution locale. Au lieu de combiner plusieurs tâches indépendantes dans le même package, séparez-les en plusieurs packages pour une exécution plus efficace dans Azure-SSIS Integration Runtime. Créez une exécution pour chaque package, afin qu’aucun package ne doive attendre que l’exécution des autres soit terminée. Cette approche tire parti de l’évolutivité d’Azure-SSIS Integration Runtime et améliore le débit global.

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur Azure-SSIS Integration Runtime. Consulter [Azure-SSIS Integration Runtime](concepts-integration-runtime.md#azure-ssis-integration-runtime).