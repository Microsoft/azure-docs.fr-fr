---
title: Envoyer les données de traçabilité Data Factory à Azure Purview
description: En savoir plus sur l’envoi (push) de données de traçabilité Data Factory à Azure Purview
services: data-factory
ms.author: lle
author: lrtoyou1223
manager: shwang
ms.reviewer: craigg
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom:
- seo-lt-2019
- references_regions
ms.date: 12/3/2020
ms.openlocfilehash: 6b50c9440a958bc1398e79ddf3c5a0984816ec45
ms.sourcegitcommit: c4246c2b986c6f53b20b94d4e75ccc49ec768a9a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96602954"
---
# <a name="push-data-factory-lineage-data-to-azure-purview-preview"></a>Envoyer les données de traçabilité Data Factory à Azure Purview (préversion)

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Dans ce tutoriel, vous allez utiliser l’interface utilisateur Data Factory pour créer un pipeline qui exécute des activités et signale des données de traçabilité à un compte Azure Purview. Vous pouvez ensuite afficher toutes les informations de traçabilité dans votre compte Azure Purview.

## <a name="prerequisites"></a>Prérequis
* **Abonnement Azure**. Si vous n’avez pas d’abonnement Azure, créez un [compte Azure gratuit](https://azure.microsoft.com/free/) avant de commencer.
* **Azure Data Factory**. Si vous n’avez pas de fabrique de données Azure, consultez [Créer une fabrique de données Azure](./quickstart-create-data-factory-portal.md).
* **Compte Azure Purview**. Le compte Purview capture toutes les données de traçabilité générées par la fabrique de données. Si vous ne possédez pas de compte Azure Purview, consultez [Créer un compte Azure Purview](https://docs.microsoft.com/azure/purview/create-catalog-portal).


## <a name="run-data-factory-activities-and-push-lineage-data-to-azure-purview"></a>Exécuter des activités Data Factory et envoyer (push) des données de traçabilité à Azure Purview
### <a name="step-1--connect-data-factory-to-your-purview-account"></a>Étape 1 :  Connecter Data Factory à votre compte Purview
Connectez-vous à votre compte Purview dans le portail Purview, puis accédez à **Centre de gestion**. Choisissez **Data Factory** dans **Connexions externes**, puis cliquez sur le bouton **Nouveau** pour créer une connexion à une nouvelle fabrique de données. 
[![Capture d’écran illustrant la création d’une connexion entre Data Factory et un compte Purview](./media/data-factory-purview/connect-adf-to-purview.png) ](./media/data-factory-purview/connect-adf-to-purview.png#lightbox)

Dans la page contextuelle, vous pouvez choisir la fabrique de données que vous souhaitez vous connecter à ce compte Purview. 
![Capture d’écran d’une nouvelle connexion](./media/data-factory-purview/new-adf-purview-connection.png)

Vous pouvez vérifier l’état après avoir créé la connexion. **Connecté** signifie que la connexion entre Data Factory et ce compte Purview est réussie. 
> [!NOTE]
> Vous devez être affecté à l’un des rôles ci-dessous dans le compte Purview et au rôle Data Factory **Contributeur** pour créer la connexion entre Data Factory et Azure Purview.
> - Propriétaire
> - Administrateur de l'accès utilisateur

### <a name="step-2-run-copy-and-dataflow-activities-in-data-factory"></a>Étape 2 : Exécuter des activités Copy et Dataflow dans Data Factory
Vous pouvez créer des pipelines, des activités Copy et des activités Dataflow dans Data Factory. Vous n’avez pas besoin de configuration supplémentaire pour la capture de données de traçabilité. Les données de traçabilité sont automatiquement capturées lors de l’exécution des activités.
![Capture d’écran des activités Copy et Dataflow](./media/data-factory-purview/adf-activities-for-lineage.png) Si vous ne savez pas comment créer des activités Copy et Dataflow, consultez [Copier des données à partir d’un stockage Blob Azure vers une base de données dans Azure SQL Database en utilisant Azure Data Factory](./tutorial-copy-data-portal.md) et [Transformer des données avec des flux de données de mappage](./tutorial-data-flow.md).

### <a name="step-3-run-execute-ssis-package-activities-in-data-factory"></a>Étape 3 : Exécuter des activités Exécuter le package SSIS dans Data Factory
Vous pouvez créer des pipelines et des activités Exécuter le package SSIS dans Data Factory. Vous n’avez pas besoin de configuration supplémentaire pour la capture de données de traçabilité. Les données de traçabilité sont automatiquement capturées lors de l’exécution des activités.
![Capture d’écran de l’activité Exécuter le package SSIS](./media/data-factory-purview/ssis-activities-for-lineage.png)

Si vous ne savez pas comment créer des activités Exécuter le package SSIS, consultez [Exécuter des packages SSIS dans Azure](./tutorial-deploy-ssis-packages-azure.md).

### <a name="step-4-view-lineage-information-in-your-purview-account"></a>Étape 4 : Afficher les informations de traçabilité dans votre compte Purview
Revenez à votre compte Purview. Dans la page d’accueil, sélectionnez **Parcourir les ressources**. Sélectionnez la ressource souhaitée, puis cliquez sur l’onglet Traçabilité. Toutes les informations de traçabilité s’affichent.
[![Capture d’écran du compte Purview](./media/data-factory-purview/view-dataset.png) ](./media/data-factory-purview/view-dataset.png#lightbox)

Vous pouvez voir les données de traçabilité de l’activité Copy.
[![Capture d’écran de la traçabilité de Copy](./media/data-factory-purview/copy-lineage.png) ](./media/data-factory-purview/copy-lineage.png#lightbox)

Vous pouvez également voir les données de traçabilité de l’activité Dataflow.
[![Capture d’écran de la traçabilité de Dataflow](./media/data-factory-purview/dataflow-lineage.png) ](./media/data-factory-purview/dataflow-lineage.png#lightbox)

> [!NOTE] 
> En ce qui concerne la traçabilité de l’activité Dataflow, nous ne prenons en charge que la source et le récepteur. La traçabilité de la transformation Dataflow n’est pas encore prise en charge.

Vous pouvez également voir les données de traçabilité pour l’activité Exécuter le package SSIS.
[![Capture d’écran de la traçabilité de SSIS](./media/data-factory-purview/ssis-lineage.png) ](./media/data-factory-purview/ssis-lineage.png#lightbox)

> [!NOTE] 
> En ce qui concerne la traçabilité de l’activité Exécuter le package SSIS, nous ne prenons en charge que la source et la destination. La traçabilité de la transformation n’est pas encore prise en charge.

## <a name="next-steps"></a>Étapes suivantes
[Guide de l’utilisateur sur la traçabilité du catalogue](https://docs.microsoft.com/azure/purview/catalog-lineage-user-guide)

[Connecter Data Factory à Azure Purview](connect-data-factory-to-azure-purview.md)