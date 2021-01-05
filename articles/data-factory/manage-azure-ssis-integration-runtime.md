---
title: Reconfigurer le runtime d’intégration Azure-SSIS
description: Découvrez comment reconfigurer le runtime d’intégration Azure-SSIS dans Azure Data Factory après l’avoir déjà configuré.
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 06/03/2020
author: swinarko
ms.author: sawinark
ms.reviewer: douglasl
manager: anandsub
ms.openlocfilehash: 7545f593aea2aca8c0aa240d44bcef3b7a1ca7c6
ms.sourcegitcommit: fb3c846de147cc2e3515cd8219d8c84790e3a442
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92636270"
---
# <a name="reconfigure-the-azure-ssis-integration-runtime"></a>Reconfigurer le runtime d’intégration Azure-SSIS

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Cet article décrit comment reconfigurer un runtime d’intégration Azure-SSIS existant. Consultez [Créer un runtime d’intégration Azure-SSIS](create-azure-ssis-integration-runtime.md) pour créer un runtime d’intégration (IR) Azure-SSIS dans Azure Data Factory.  

## <a name="data-factory-ui"></a>IU de la fabrique de données 
Vous pouvez utiliser l’interface utilisateur de Data Factory pour arrêter, modifier/reconfigurer ou supprimer un runtime d’intégration Azure-SSIS. 

1. Ouvrez l’interface utilisateur de Data Factory en sélectionnant la vignette **Créer et surveiller** dans la page d’accueil de votre fabrique de données.
2. Sélectionnez le hub **Gérer** sous les hubs **Accueil** , **Modifier** et **Surveiller** pour afficher le volet **Connexions** .

### <a name="to-reconfigure-an-azure-ssis-ir"></a>Pour reconfigurer un runtime d’intégration Azure-SSIS
Dans le volet **Connexions** du hub **Gestion** , basculez sur la page **Runtimes d’intégration** et sélectionnez **Actualiser** . 

   ![Volet Connexions](./media/tutorial-create-azure-ssis-runtime-portal/connections-pane.png)

   Vous pouvez modifier/reconfigurer votre runtime d’intégration Azure-SSIS en sélectionnant son nom. Vous pouvez également sélectionner les boutons appropriés pour superviser/démarrer/arrêter/supprimer votre runtime d’intégration Azure-SSIS, générer automatiquement un pipeline ADF avec l’activité Exécuter le package SSIS à exécuter sur votre runtime d’intégration Azure-SSIS et afficher le code JSON/la charge utile de votre runtime d’intégration Azure-SSIS.  Vous ne pouvez modifier ou supprimer votre runtime d’intégration Azure-SSIS que lorsqu’il est arrêté.

## <a name="azure-powershell"></a>Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Une fois que vous avez configuré et démarré une instance du runtime d’intégration Azure-SSIS, vous pouvez la reconfigurer en exécutant plusieurs cmdlets PowerShell `Stop` - `Set` - `Start`, les unes à la suite des autres. Par exemple, le script PowerShell suivant modifie le nombre de nœuds alloués à l’instance du runtime d’intégration Azure-SSIS et lui en attribue cinq.

### <a name="reconfigure-an-azure-ssis-ir"></a>Reconfigurer un runtime d’intégration Azure-SSIS

1. Tout d’abord, arrêtez le runtime d’intégration Azure-SSIS à l’aide de l’applet de commande [Stop-AzDataFactoryV2IntegrationRuntime](/powershell/module/az.datafactory/stop-Azdatafactoryv2integrationruntime). Cette commande libère tous ses nœuds et arrête la facturation.

   ```powershell
   Stop-AzDataFactoryV2IntegrationRuntime -DataFactoryName $DataFactoryName -Name $AzureSSISName -ResourceGroupName $ResourceGroupName 
   ```
2. Ensuite, reconfigurez Azure-SSIS Integration Runtime à l’aide de l’applet de commande [Set-AzDataFactoryV2IntegrationRuntime](/powershell/module/az.datafactory/set-Azdatafactoryv2integrationruntime). L’exemple de commande suivant augmente la taille du runtime d’intégration Azure-SSIS en le faisant passer à cinq nœuds.

   ```powershell
   Set-AzDataFactoryV2IntegrationRuntime -DataFactoryName $DataFactoryName -Name $AzureSSISName -ResourceGroupName $ResourceGroupName -NodeCount 5
   ```  
3. Démarrez ensuite le runtime d’intégration Azure-SSIS à l’aide de l’applet de commande [Start-AzDataFactoryV2IntegrationRuntime](/powershell/module/az.datafactory/start-Azdatafactoryv2integrationruntime). Cette commande a pour effet d’allouer tous ses nœuds à l’exécution des packages SSIS.   

   ```powershell
   Start-AzDataFactoryV2IntegrationRuntime -DataFactoryName $DataFactoryName -Name $AzureSSISName -ResourceGroupName $ResourceGroupName
   ```

### <a name="delete-an-azure-ssis-ir"></a>Supprimer un runtime d’intégration Azure-SSIS
1. Tout d’abord, répertoriez tous les runtimes d’intégration Azure-SSIS dans votre fabrique de données.

   ```powershell
   Get-AzDataFactoryV2IntegrationRuntime -DataFactoryName $DataFactoryName -ResourceGroupName $ResourceGroupName -Status
   ```
2. Puis, arrêtez tous les runtimes d’intégration Azure-SSIS dans votre fabrique de données.

   ```powershell
   Stop-AzDataFactoryV2IntegrationRuntime -DataFactoryName $DataFactoryName -Name $AzureSSISName -ResourceGroupName $ResourceGroupName -Force
   ```
3. Ensuite, supprimez un à un tous les runtimes d’intégration Azure-SSIS dans votre fabrique de données.

   ```powershell
   Remove-AzDataFactoryV2IntegrationRuntime -DataFactoryName $DataFactoryName -Name $AzureSSISName -ResourceGroupName $ResourceGroupName -Force
   ```
4. Enfin, supprimez votre fabrique de données.

   ```powershell
   Remove-AzDataFactoryV2 -Name $DataFactoryName -ResourceGroupName $ResourceGroupName -Force
   ```
5. Si vous aviez créé un groupe de ressources, supprimez-le.

   ```powershell
   Remove-AzResourceGroup -Name $ResourceGroupName -Force 
   ```

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur le runtime Azure-SSIS, voir les rubriques suivantes : 

- [Azure-SSIS Integration Runtime](concepts-integration-runtime.md#azure-ssis-integration-runtime) (Runtime d’intégration Azure-SSIS). Cet article fournit des informations conceptuelles sur les runtimes d’intégration en général, y compris sur le runtime d’intégration Azure-SSIS. 
- [Didacticiel : deploy SSIS packages to Azure](./tutorial-deploy-ssis-packages-azure.md) (Déployer des packages SSIS vers Azure). Cet article fournit des instructions détaillées pour créer un runtime d’intégration Azure-SSIS. Dans le cadre de cet article, une instance Azure SQL Database est utilisée pour héberger le catalogue SSIS. 
- [Procédure : Créer un runtime d’intégration Azure-SSIS](create-azure-ssis-integration-runtime.md). Cet article s’appuie sur le tutoriel et fournit des instructions sur la façon d’utiliser une instance managée SQL Azure et de joindre le runtime d’intégration à un réseau virtuel. 
- [Joindre un runtime d’intégration Azure-SSIS à un réseau virtuel](join-azure-ssis-integration-runtime-virtual-network.md). Cet article fournit des informations conceptuelles sur la façon d’attacher un runtime d’intégration Azure-SSIS à un réseau virtuel Azure. Il décrit également les étapes nécessaires pour utiliser le portail Azure afin de configurer le réseau virtuel de sorte que le runtime d’intégration Azure-SSIS puisse le rejoindre. 
- [Monitor an Azure-SSIS IR](monitor-integration-runtime.md#azure-ssis-integration-runtime) (Surveiller le runtime d’intégration Azure-SSIS). Cet article explique comment récupérer des informations sur un runtime d’intégration Azure-SSIS ainsi que des descriptions d’état dans les informations renvoyées.