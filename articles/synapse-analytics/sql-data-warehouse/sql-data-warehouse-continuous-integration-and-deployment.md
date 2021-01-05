---
title: Intégration et déploiement continus pour un pool SQL dédié
description: Expérience DevOps de base de données de classe Entreprise destinée aux pools SQL dédiés dans Azure Synapse Analytics avec prise en charge intégrée de l’intégration et du déploiement continus à l’aide d’Azure Pipelines.
services: synapse-analytics
author: kevinvngo
manager: craigg
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: sql-dw
ms.date: 02/04/2020
ms.author: kevin
ms.reviewer: igorstan
ms.custom: azure-synapse
ms.openlocfilehash: 35f503e7214fa91962c91c35611fc9447302f9fc
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96462818"
---
# <a name="continuous-integration-and-deployment-for-dedicated-sql-pool-in-azure-synapse-analytics"></a>Intégration et déploiement continus pour un pool SQL dédié dans Azure Synapse Analytics

Ce tutoriel simple décrit comment intégrer votre projet de base de données SQL Server Data Tools (SSDT) à Azure DevOps et comment tirer parti d’Azure Pipelines pour configurer l’intégration et le déploiement continus. Ce tutoriel constitue la deuxième étape de la génération de votre pipeline d’intégration et de déploiement continus pour l’entreposage de données.

## <a name="before-you-begin"></a>Avant de commencer

- Parcourez le [tutoriel sur l’intégration du contrôle de code source](sql-data-warehouse-source-control-integration.md)

- Configurer Azure DevOps et s’y connecter

## <a name="continuous-integration-with-visual-studio-build"></a>Intégration continue avec la build Visual Studio

1. Accédez à Azure Pipelines et créez un pipeline de build.

      ![Nouveau pipeline](./media/sql-data-warehouse-continuous-integration-and-deployment/1-new-build-pipeline.png "Nouveau pipeline")

2. Sélectionnez votre référentiel de code source (Azure Repos Git) et sélectionnez le modèle d’application de bureau .NET.

      ![Configuration du pipeline](./media/sql-data-warehouse-continuous-integration-and-deployment/2-pipeline-setup.png "Configuration du pipeline")

3. Modifiez votre fichier YAML pour utiliser le pool approprié de votre agent. Votre fichier YAML doit ressembler à ce qui suit :

      ![YAML](./media/sql-data-warehouse-continuous-integration-and-deployment/3-yaml-file.png "YAML")

À ce stade, vous disposez d’un environnement simple dans lequel tout enregistrement dans la branche principale de votre référentiel de contrôle de code source doit déclencher automatiquement une build Visual Studio réussie de votre projet de base de données. Vérifiez que l’automatisation fonctionne de bout en bout en apportant une modification dans votre projet de base de données locale et en enregistrant cette modification dans votre branche principale.

## <a name="continuous-deployment-with-the-azure-synapse-analytics-or-database-deployment-task"></a>Déploiement continu avec la tâche de déploiement Azure Synapse Analytics (ou SQL Database)

1. Ajoutez une nouvelle tâche à l’aide de la [tâche de déploiement Azure SQL Database](/azure/devops/pipelines/targets/azure-sqldb) et renseignez les champs obligatoires pour vous connecter à votre entrepôt de données cible. Pendant l’exécution de cette tâche, le DACPAC généré à partir du processus de génération précédent est déployé dans l’entrepôt de données cible. Vous pouvez aussi utiliser la [tâche de déploiement Azure Synapse Analytics](https://marketplace.visualstudio.com/items?itemName=ms-sql-dw.SQLDWDeployment).

      ![Tâche de déploiement](./media/sql-data-warehouse-continuous-integration-and-deployment/4-deployment-task.png "Tâche de déploiement")

2. Si vous utilisez un agent autohébergé, veillez à définir votre variable d’environnement de façon à utiliser le bon fichier SqlPackage.exe pour Azure Synapse Analytics. Le chemin doit être similaire à celui-ci :

      ![Variable d’environnement](./media/sql-data-warehouse-continuous-integration-and-deployment/5-environment-variable-preview.png "Variable d’environnement")

   C:\Program Files (x86)\Microsoft Visual Studio\2019\Preview\Common7\IDE\Extensions\Microsoft\SQLDB\DAC\150  

   Exécutez et validez votre pipeline. Vous pouvez apporter des modifications localement et les enregistrer dans votre contrôle de code source qui devraient générer une build et un déploiement automatiques.

## <a name="next-steps"></a>Étapes suivantes

- Explorer l’[Architecture du pool SQL dédié (anciennement SQL DW)](massively-parallel-processing-mpp-architecture.md)
- Créer rapidement un [pool SQL dédié (anciennement SQL DW)](create-data-warehouse-portal.md)
- [Charger des exemples de données](load-data-from-azure-blob-storage-using-polybase.md)
- Explorer les [vidéos](sql-data-warehouse-videos.md)
