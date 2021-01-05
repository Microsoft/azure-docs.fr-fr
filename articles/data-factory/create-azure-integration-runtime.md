---
title: Créer un runtime d’intégration Azure dans Azure Data Factory
description: Apprenez à créer un runtime d’intégration Azure dans Azure Data Factory, qui est utilisé pour copier des données et répartir des activités de transformation.
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 06/09/2020
author: nabhishek
ms.author: abnarain
manager: anandsub
ms.openlocfilehash: 97f3a8f79e503b8870baf67ee982b0091d926b1e
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92895339"
---
# <a name="how-to-create-and-configure-azure-integration-runtime"></a>Comment créer et configurer un runtime d’intégration Azure
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Le runtime d’intégration (IR) représente l’infrastructure de calcul utilisée par Azure Data Factory pour fournir des capacités d’intégration de données entre différents environnements réseau. Pour plus d’informations sur le runtime d’intégration, voir [Runtime d’intégration](concepts-integration-runtime.md).

Un runtime d’intégration Azure fournit un calcul entièrement géré pour effectuer en mode natif un déplacement de données et distribuer des activités de transformation de données à des services de calcul tels que HDInsight. Il est hébergé dans l’environnement Azure et prend en charge la connexion à des ressources dans un environnement de réseau public avec des points de terminaison accessibles au public.

Ce document décrit comment créer et configurer un runtime d’intégration Azure. 

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="default-azure-ir"></a>Runtime d’intégration Azure par défaut
Par défaut, chaque fabrique de données a un runtime d’intégration Azure dans le service principal, qui prend en charge les opérations sur les banques de données et les services de calcul en ligne dans un réseau public. L’emplacement de ce runtime d’intégration Azure est résolu automatiquement. Si la propriété **connectVia** n’est pas spécifiée dans la définition du service lié, le runtime d’intégration Azure par défaut est utilisé. Vous devez uniquement créer explicitement un runtime d’intégration Azure lorsque vous souhaitez définir explicitement l’emplacement du runtime d’intégration, ou si vous souhaitez grouper virtuellement les exécutions d’activité sur différents runtimes d’intégration à des fins de gestion. 

## <a name="create-azure-ir"></a>Créer un runtime d’intégration Azure

Pour créer et configurer un Azure IR, vous pouvez suivre les procédures suivantes.

### <a name="create-an-azure-ir-via-azure-powershell"></a>Créer un Azure IR via Azure PowerShell
Vous pouvez créer un runtime d’intégration à l’aide de la cmdlet PowerShell **Set-AzDataFactoryV2IntegrationRuntime**. Pour créer un Azure IR, vous spécifiez le nom, l’emplacement et le type de la commande. Voici un exemple de commande pour créer un runtime d’intégration Azure avec l’emplacement défini sur « West Europe » (Europe Ouest) :

```powershell
Set-AzDataFactoryV2IntegrationRuntime -DataFactoryName "SampleV2DataFactory1" -Name "MySampleAzureIR" -ResourceGroupName "ADFV2SampleRG" -Type Managed -Location "West Europe"
```  
Pour un runtime d’intégration Azure, le type doit être défini sur **Géré**. Vous n’avez pas besoin spécifier les détails du calcul, car il est entièrement géré de façon élastique dans le cloud. Spécifiez des détails de calcul tels que la taille de nœud et le nombre de nœuds lorsque vous souhaitez créer un runtime d’intégration Azure-SSIS. Pour plus d’informations, voir [Créer et configurer un runtime d’intégration Azure-SSIS](create-azure-ssis-integration-runtime.md).

Vous pouvez configurer un runtime d’intégration Azure existant pour modifier son emplacement à l’aide de la cmdlet PowerShell Set-AzDataFactoryV2IntegrationRuntime. Pour plus d’informations sur l’emplacement d’un runtime d’intégration Azure, voir [Présentation du runtime d’intégration](concepts-integration-runtime.md).

### <a name="create-an-azure-ir-via-azure-data-factory-ui"></a>Créer une Azure IR via l’interface utilisateur Azure Data Factory
Pour créer un Azure IR à l’aide de l’interface utilisateur Azure Data Factory, procédez comme suit.

1. Dans la page **Prise en main** de l’interface utilisateur Azure Data Factory, sélectionnez l’ [onglet Gérer](./author-management-hub.md) dans le volet le plus à gauche.

   ![Bouton Gérer de la page d’accueil](media/doc-common-process/get-started-page-manage-button.png)

1. Sélectionnez **Runtimes d’intégration** dans le volet gauche, puis **+ Nouveau**.

   ![Capture d’écran qui met en surbrillance les runtimes d’intégration dans le volet de gauche et le bouton + Nouveau.](media/doc-common-process/manage-new-integration-runtime.png)

1. Sur la page **Configuration du runtime d’intégration** , sélectionnez **Azure, auto-hébergé** , puis **Continuer**. 

1. Sur la page suivante, sélectionnez **Azure** pour créer un Azure IR, puis choisissez **Continuer**.
   ![Créer un runtime d’intégration](media/create-azure-integration-runtime/new-azure-integration-runtime.png)

1. Entrez un nom pour votre Azure IR, puis sélectionnez **Créer**.
   ![Créer un Azure IR](media/create-azure-integration-runtime/create-azure-integration-runtime.png)

1. Une notification contextuelle s’affiche lorsque la création est terminée. Dans la page **Runtimes d’intégration** , assurez-vous que le runtime d’intégration nouvellement créé est répertorié.

## <a name="use-azure-ir"></a>Utiliser un runtime d’intégration Azure

Une fois un runtime d’intégration Azure créé, vous pouvez le référencer dans votre définition de service lié. Voici un exemple de la façon dont vous pouvez référencer le runtime d’intégration Azure créé ci-dessus à partir d’un service lié de Stockage Azure :

```json
{
    "name": "MyStorageLinkedService",
    "properties": {
      "type": "AzureStorage",
      "typeProperties": {
        "connectionString": "DefaultEndpointsProtocol=https;AccountName=myaccountname;AccountKey=..."
      },
      "connectVia": {
        "referenceName": "MySampleAzureIR",
        "type": "IntegrationRuntimeReference"
      }   
    }
}

```

## <a name="next-steps"></a>Étapes suivantes
Lisez les articles suivants concernant la création d’autres types de runtimes d’intégration :

- [Créer un runtime d’intégration auto-hébergé](create-self-hosted-integration-runtime.md)
- [Créer un runtime d’intégration Azure-SSIS](create-azure-ssis-integration-runtime.md)
