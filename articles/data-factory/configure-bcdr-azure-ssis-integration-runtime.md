---
title: Configurer Azure-SSIS Integration Runtime pour le basculement SQL Database
description: Cet article explique comment configurer Azure-SSIS Integration Runtime avec la géoréplication et le basculement Azure SQL Database pour la base de données SSISDB.
services: data-factory
ms.service: data-factory
ms.workload: data-services
ms.devlang: powershell
author: swinarko
ms.author: sawinark
manager: mflasko
ms.reviewer: douglasl
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 11/06/2020
ms.openlocfilehash: 6b37a0df994546762abbcf3452d8e7b52dec6847
ms.sourcegitcommit: 46c5ffd69fa7bc71102737d1fab4338ca782b6f1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "94331411"
---
# <a name="configure-the-azure-ssis-integration-runtime-with-sql-database-geo-replication-and-failover"></a>Configurer Azure-SSIS Integration Runtime avec la géoréplication et le basculement SQL Database

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-xxx-md.md)]

Cet article explique comment configurer Azure-SSIS Integration Runtime avec la géoréplication Azure SQL Database pour la base de données SSISDB. Lorsqu’un basculement se produit, vous pouvez garantir le bon fonctionnement d’Azure-SSIS Integration Runtime avec la base de données secondaire.

Pour plus d’informations sur la géoréplication et le basculement de SQL Database, consultez [Vue d’ensemble : géoréplication active et groupes de basculement automatique](../azure-sql/database/auto-failover-group-overview.md).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="azure-ssis-ir-failover-with-a-sql-managed-instance"></a>Basculement du runtime Azure-SSIS IR avec Azure SQL Managed Instance

### <a name="prerequisites"></a>Prérequis

Azure SQL Managed Instance utilise une clé *DMK (clé principale de base de données)* pour sécuriser les données, les informations d’identification et les informations de connexion stockées dans une base de données. Pour activer le déchiffrement automatique de la clé DMK, une copie de la clé est chiffrée à l’aide de la *clé SMK (clé principale du serveur)* . 

La SMK n’est pas répliquée dans le groupe de basculement. Vous devez ajouter un mot de passe à la fois sur les instances principale et secondaire pour le déchiffrement DMK après le basculement.

1. Exécutez la commande suivante pour SSISDB sur l’instance principale. Cette étape ajoute un nouveau mot de passe de chiffrement.

   ```sql
   ALTER MASTER KEY ADD ENCRYPTION BY PASSWORD = 'password'
   ```

2. Créez un groupe de basculement sur SQL Managed Instance.

3. Exécutez **sp_control_dbmasterkey_password** sur l’instance secondaire avec le nouveau mot de passe de chiffrement.

   ```sql
   EXEC sp_control_dbmasterkey_password @db_name = N'SSISDB', @password = N'<password>', @action = N'add';  
   GO
   ```

### <a name="scenario-1-azure-ssis-ir-is-pointing-to-a-readwrite-listener-endpoint"></a>Scénario 1 : Azure-SSIS IR pointe vers un point de terminaison de l’écouteur en lecture/écriture

Si vous souhaitez qu’Azure-SSIS IR pointe vers un point de terminaison de l’écouteur en lecture/écriture, vous devez d’abord pointer vers le point de terminaison du serveur principal. Après avoir placé SSISDB dans un groupe de basculement, vous pouvez arrêter votre Azure-SSIS IR, le modifier pour qu’il pointe vers le point de terminaison de l’écouteur en lecture/écriture à l’aide d’Azure PowerShell, puis le redémarrer.

```powershell
Set-AzDataFactoryV2IntegrationRuntime -CatalogServerEndpoint "Azure SQL Managed Instance read/write listener endpoint"
```

#### <a name="solution"></a>Solution

Lorsque le basculement se produit, procédez comme suit :

1. Arrêtez Azure-SSIS IR dans la région primaire.

2. Modifiez Azure-SSIS IR avec les informations d’URI sur la région, le réseau virtuel et la signature d’accès partagé (SAP) pour une installation personnalisée sur l’instance secondaire. Comme Azure-SSIS IR pointe vers l’écouteur en lecture et en écriture et que le point de terminaison est transparent pour Azure-SSIS IR, vous n’avez pas besoin de modifier le point de terminaison.

   ```powershell
   Set-AzDataFactoryV2IntegrationRuntime -Location "new region" `
      -VNetId "new VNet" `
      -Subnet "new subnet" `
      -SetupScriptContainerSasUri "new custom setup SAS URI"
   ```

3. Redémarrez Azure-SSIS IR.

### <a name="scenario-2-azure-ssis-ir-is-pointing-to-a-primary-server-endpoint"></a>Scénario 2 : Azure-SSIS IR pointe vers un point de terminaison du serveur principal

Ce scénario convient si Azure-SSIS IR pointe vers un point de terminaison du serveur principal.

#### <a name="solution"></a>Solution

Lorsque le basculement se produit, procédez comme suit :

1. Arrêtez Azure-SSIS IR dans la région primaire.

2. Modifiez le runtime Azure-SSIS IR avec les nouvelles informations sur la région, le point de terminaison et le réseau virtuel de l’instance secondaire.

   ```powershell
   Set-AzDataFactoryV2IntegrationRuntime -Location "new region" `
      -CatalogServerEndpoint "Azure SQL Database endpoint" `
      -CatalogAdminCredential "Azure SQL Database admin credentials" `
      -VNetId "new VNet" `
      -Subnet "new subnet" `
      -SetupScriptContainerSasUri "new custom setup SAS URI"
   ```

3. Redémarrez Azure-SSIS IR.

### <a name="scenario-3-azure-ssis-ir-is-pointing-to-a-public-endpoint-of-a-sql-managed-instance"></a>Scénario 3 : Azure-SSIS IR pointe vers un point de terminaison public d’une SQL Managed Instance

Ce scénario convient si Azure-SSIS IR pointe vers le point de terminaison public d’une Azure SQL Managed Instance et n’est pas joint à un réseau virtuel. La seule différence avec le scénario 2 vient du fait que vous n’avez pas besoin de modifier les informations relatives au réseau virtuel d’Azure-SSIS IR après le basculement.

#### <a name="solution"></a>Solution

Lorsque le basculement se produit, procédez comme suit :

1. Arrêtez Azure-SSIS IR dans la région primaire.

2. Modifiez Azure-SSIS IR avec les nouvelles informations relatives à la région et au point de terminaison pour l’instance secondaire.

   ```powershell
   Set-AzDataFactoryV2IntegrationRuntime -Location "new region" `
      -CatalogServerEndpoint "Azure SQL Database server endpoint" `
      -CatalogAdminCredential "Azure SQL Database server admin credentials" `
      -SetupScriptContainerSasUri "new custom setup SAS URI"
   ```

3. Redémarrez Azure-SSIS IR.

### <a name="scenario-4-attach-an-existing-ssisdb-instance-ssis-catalog-to-a-new-azure-ssis-ir"></a>Scénario 4 : Attachement d’une instance SSISDB (catalogue SSIS) existante à un nouveau runtime d’intégration Azure-SSIS IR

Ce scénario convient si vous souhaitez que SSISDB fonctionne avec un nouveau Azure-SSIS IR dans une nouvelle région lorsqu’un incident Azure Data Factory ou Azure-SSIS IR se produit dans la région actuelle.

#### <a name="solution"></a>Solution

Lorsque le basculement se produit, procédez comme suit.

> [!NOTE]
> Utilisez PowerShell pour l’étape 4 (création de l’IR). Si vous ne le faites pas, le portail Azure signalera une erreur indiquant que SSISDB existe déjà.

1. Arrêtez Azure-SSIS IR dans la région primaire.

2. Exécutez une procédure stockée pour mettre à jour les métadonnées dans SSISDB afin d’accepter les connexions provenant de **\<new_data_factory_name\>** et **\<new_integration_runtime_name\>** .
   
   ```sql
   EXEC [catalog].[failover_integration_runtime] @data_factory_name='<new_data_factory_name>', @integration_runtime_name='<new_integration_runtime_name>'
   ```

3. Créez une fabrique de données nommée **\<new_data_factory_name\>** dans la nouvelle région.

   ```powershell
   Set-AzDataFactoryV2 -ResourceGroupName "new resource group name" `
      -Location "new region"`
      -Name "<new_data_factory_name>"
   ```
   
   Pour plus d’informations sur cette commande PowerShell, consultez [Créer une fabrique de données Azure à l’aide de PowerShell](quickstart-create-data-factory-powershell.md).

4. Créez un runtime d’intégration Azure-SSIS appelé **\<new_integration_runtime_name\>** dans la nouvelle région à l’aide d’Azure PowerShell.

   ```powershell
   Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName "new resource group name" `
      -DataFactoryName "new data factory name" `
      -Name "<new_integration_runtime_name>" `
      -Description $AzureSSISDescription `
      -Type Managed `
      -Location $AzureSSISLocation `
      -NodeSize $AzureSSISNodeSize `
      -NodeCount $AzureSSISNodeNumber `
      -Edition $AzureSSISEdition `
      -LicenseType $AzureSSISLicenseType `
      -MaxParallelExecutionsPerNode $AzureSSISMaxParallelExecutionsPerNode `
      -VnetId "new vnet" `
      -Subnet "new subnet" `
      -CatalogServerEndpoint $SSISDBServerEndpoint `
      -CatalogPricingTier $SSISDBPricingTier
   ```
   
   Pour plus d’informations sur cette commande PowerShell, consultez [Créer le runtime d’intégration Azure-SSIS dans Azure Data Factory](create-azure-ssis-integration-runtime.md).

## <a name="azure-ssis-ir-failover-with-sql-database"></a>Basculement du runtime Azure-SSIS IR avec SQL Database

### <a name="scenario-1-azure-ssis-ir-is-pointing-to-a-readwrite-listener-endpoint"></a>Scénario 1 : Azure-SSIS IR pointe vers un point de terminaison de l’écouteur en lecture/écriture

Ce scénario convient si :

- Azure-SSIS Integration Runtime pointe vers le point de terminaison de l’écouteur en lecture-écriture du groupe de basculement.
- Le serveur SQL Database n’est *pas* configuré avec la règle de point de terminaison de service de réseau virtuel.

Si vous souhaitez qu’Azure-SSIS IR pointe vers un point de terminaison de l’écouteur en lecture/écriture, vous devez d’abord pointer vers le point de terminaison du serveur principal. Après avoir placé SSISDB dans un groupe de basculement, vous pouvez arrêter votre Azure-SSIS IR, le modifier pour qu’il pointe vers le point de terminaison de l’écouteur en lecture/écriture à l’aide d’Azure PowerShell, puis le redémarrer.

```powershell
Set-AzDataFactoryV2IntegrationRuntime -CatalogServerEndpoint "Azure SQL Database read/write listener endpoint"
```

#### <a name="solution"></a>Solution

En cas de basculement, il est transparent pour Azure-SSIS Integration Runtime. Azure-SSIS Integration Runtime se connecte automatiquement au nouveau point de terminaison principal du groupe de basculement. 

Si vous souhaitez mettre à jour la région ou d’autres informations dans Azure-SSIS IR, vous pouvez l’arrêter, le modifier et le redémarrer.


### <a name="scenario-2-azure-ssis-ir-is-pointing-to-a-primary-server-endpoint"></a>Scénario 2 : Azure-SSIS IR pointe vers un point de terminaison du serveur principal

Ce scénario convient si Azure-SSIS IR pointe vers un point de terminaison du serveur principal.

#### <a name="solution"></a>Solution

Lorsque le basculement se produit, procédez comme suit :

1. Arrêtez Azure-SSIS IR dans la région primaire.

2. Modifiez le runtime Azure-SSIS IR avec les nouvelles informations sur la région, le point de terminaison et le réseau virtuel de l’instance secondaire.

   ```powershell
   Set-AzDataFactoryV2IntegrationRuntime -Location "new region" `
      -CatalogServerEndpoint "Azure SQL Database endpoint" `
      -CatalogAdminCredential "Azure SQL Database admin credentials" `
      -VNetId "new VNet" `
      -Subnet "new subnet" `
      -SetupScriptContainerSasUri "new custom setup SAS URI"
   ```

3. Redémarrez Azure-SSIS IR.

### <a name="scenario-3-attach-an-existing-ssisdb-ssis-catalog-to-a-new-azure-ssis-ir"></a>Scénario 3 : Attachement d’une instance SSISDB (catalogue SSIS) existante à un nouveau runtime d’intégration Azure-SSIS IR

Ce scénario convient si vous souhaitez approvisionner un nouveau runtime Azure-SSIS IR dans une région secondaire. Il convient également si vous souhaitez que votre SSISDB continue de fonctionner avec un nouveau Azure-SSIS IR dans une nouvelle région lorsqu’un incident Azure Data Factory ou Azure-SSIS IR se produit dans la région actuelle.

#### <a name="solution"></a>Solution

Lorsque le basculement se produit, procédez comme suit.

> [!NOTE]
> Utilisez PowerShell pour l’étape 4 (création de l’IR). Si vous ne le faites pas, le portail Azure signalera une erreur indiquant que SSISDB existe déjà.

1. Arrêtez Azure-SSIS IR dans la région primaire.

2. Exécutez une procédure stockée pour mettre à jour les métadonnées dans SSISDB afin d’accepter les connexions provenant de **\<new_data_factory_name\>** et **\<new_integration_runtime_name\>** .
   
   ```sql
   EXEC [catalog].[failover_integration_runtime] @data_factory_name='<new_data_factory_name>', @integration_runtime_name='<new_integration_runtime_name>'
   ```

3. Créez une fabrique de données nommée **\<new_data_factory_name\>** dans la nouvelle région.

   ```powershell
   Set-AzDataFactoryV2 -ResourceGroupName "new resource group name" `
      -Location "new region"`
      -Name "<new_data_factory_name>"
   ```
   
   Pour plus d’informations sur cette commande PowerShell, consultez [Créer une fabrique de données Azure à l’aide de PowerShell](quickstart-create-data-factory-powershell.md).

4. Créez un runtime d’intégration Azure-SSIS appelé **\<new_integration_runtime_name\>** dans la nouvelle région à l’aide d’Azure PowerShell.

   ```powershell
   Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName "new resource group name" `
      -DataFactoryName "new data factory name" `
      -Name "<new_integration_runtime_name>" `
      -Description $AzureSSISDescription `
      -Type Managed `
      -Location $AzureSSISLocation `
      -NodeSize $AzureSSISNodeSize `
      -NodeCount $AzureSSISNodeNumber `
      -Edition $AzureSSISEdition `
      -LicenseType $AzureSSISLicenseType `
      -MaxParallelExecutionsPerNode $AzureSSISMaxParallelExecutionsPerNode `
      -VnetId "new vnet" `
      -Subnet "new subnet" `
      -CatalogServerEndpoint $SSISDBServerEndpoint `
      -CatalogPricingTier $SSISDBPricingTier
   ```

   Pour plus d’informations sur cette commande PowerShell, consultez [Créer le runtime d’intégration Azure-SSIS dans Azure Data Factory](create-azure-ssis-integration-runtime.md).

## <a name="next-steps"></a>Étapes suivantes

Il existe d’autres options de configuration pour le runtime d’intégration Azure-SSIS :

- [Configurer Azure-SSIS Integration Runtime pour obtenir de hautes performances](configure-azure-ssis-integration-runtime-performance.md)

- [Personnaliser la configuration du runtime d’intégration Azure-SSIS](how-to-configure-azure-ssis-ir-custom-setup.md)

- [Approvisionner Enterprise Edition pour Azure-SSIS Integration Runtime](how-to-configure-azure-ssis-ir-enterprise-edition.md)
