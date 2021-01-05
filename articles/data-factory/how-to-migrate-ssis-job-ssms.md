---
title: Migrer des travaux SQL Server Integration Services (SSIS) locaux vers Azure Data Factory
description: Cet article explique comment migrer des travaux SQL Server Integration Services (SSIS) vers des pipelines/activités/déclencheurs Azure Data Factory à l’aide de SQL Server Management Studio.
services: data-factory
documentationcenter: ''
author: chugugrace
ms.author: chugu
ms.reviewer: ''
manager: ''
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 4/7/2020
ms.openlocfilehash: 5566717387f6da375129a0e70c9ad825198d66b7
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96005705"
---
# <a name="migrate-sql-server-agent-jobs-to-adf-with-ssms"></a>Migrer des travaux SQL Server Agent vers ADF avec SSMS

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Quand vous [migrez des charges de travail SQL Server Integration Services (SSIS) locales vers SSIS dans ADF](scenario-ssis-migration-overview.md), une fois les packages SSIS migrés, vous pouvez effectuer la migration par lots des travaux SQL Server Agent avec le type d’étape de travail Package SQL Server Integration Services vers des pipelines/activités/déclencheurs de planification Azure Data Factory (ADF) par le biais de l’**Assistant Migration des travaux SSIS** de SQL Server Management Studio (SSMS).

En général, pour les travaux SQL Agent sélectionnés avec les types d’étapes de travail applicables, l’**Assistant Migration des travaux SSIS** peut :

- Mapper l’emplacement du package SSIS local à l’emplacement où les packages sont migrés, qui sont accessibles par SSIS dans ADF.
    > [!NOTE]
    > L’emplacement du package de système de fichiers est pris en charge uniquement.
- Migrer les travaux applicables avec les étapes de travail applicables aux ressources ADF correspondantes, comme indiqué ci-dessous :

|Objet de travail SQL Agent  |Ressource ADF  |Notes|
|---------|---------|---------|
|Travail SQL Agent|pipeline     |Le nom du pipeline sera *généré pour \<job name>* . <br> <br> Les travaux d’agent intégrés ne sont pas applicables : <li> Travail de maintenance de serveur SSIS <li> syspolicy_purge_history <li> collection_set_* <li> mdw_purge_data_* <li> sysutility_*|
|Étape de travail SSIS|Activité Exécuter le package SSIS|<li> Le nom de l’activité sera \<step name>. <li> Le compte proxy utilisé dans l’étape de travail sera migré en tant qu’authentification Windows de cette activité. <li> Les *options d’exécution* (sauf *Utiliser le runtime 32 bits*) définies dans l’étape de travail seront ignorées lors de la migration. <li> La *vérification* définie dans l’étape de travail sera ignorée lors de la migration.|
|schedule      |déclencheur de planification        |Le nom du déclencheur de planification sera *généré pour \<schedule name>* . <br> <br> Les options ci-dessous de planification du travail SQL Agent seront ignorées lors de la migration : <li> Intervalle de second niveau. <li> *Lancer automatiquement au démarrage de SQL Server Agent* <li> *Démarrer dès que les processeurs sont inactifs* <li> *jour ouvrable* et *week-end* <time zone> <br> Vous trouverez ci-dessous les différences après la migration de la planification du travail SQL Agent vers le déclencheur de planification ADF : <li> L’exécution suivante du déclencheur de planification ADF est indépendante de l’état de l’exécution déclenchée précédemment. <li> La configuration de la périodicité du déclencheur de planification ADF diffère de la fréquence quotidienne dans le travail SQL Agent.|

- Générer des modèles Azure Resource Manager (ARM) dans le dossier de sortie local, et les déployer directement ou ultérieurement dans la fabrique de données. Pour plus d’informations sur les modèles ADF Resource Manager, consultez [Types de ressources Microsoft.DataFactory](/azure/templates/microsoft.datafactory/allversions).

## <a name="prerequisites"></a>Prérequis

La fonctionnalité décrite dans cet article nécessite SQL Server Management Studio version 18.5 ou ultérieure. Pour obtenir la dernière version de SSMS, consultez [Télécharger SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-ver15).

## <a name="migrate-ssis-jobs-to-adf"></a>Migrer des travaux SSIS vers ADF

1. Dans SSMS, dans l’Explorateur d’objets, sélectionnez SQL Server Agent, Travaux, puis cliquez avec le bouton droit et sélectionnez **Migrer des travaux SSIS vers ADF**.
![La capture d’écran montre l’Explorateur d’objets de SQL Server Management Studio, dans lequel vous pouvez sélectionner des travaux, puis migrer des travaux SSIS vers ADF.](media/how-to-migrate-ssis-job-ssms/menu.png)

1. Connectez-vous à Azure, sélectionnez Abonnement Azure, Data Factory et Integration Runtime. Stockage Azure est facultatif. Il est utilisé à l’étape de mappage d’emplacement de package si les travaux SSIS à migrer ont des packages de système de fichiers SSIS.
![menu](media/how-to-migrate-ssis-job-ssms/step1.png)

1. Mappe les chemins des packages SSIS et fichiers de configuration dans les travaux SSIS à des chemins de destination accessibles aux pipelines migrés. Lors de cette étape de mappage, vous pouvez :

    1. Sélectionner un dossier source, puis **Ajouter un mappage**.
    1. Mettre à jour un chemin de dossier source. Les chemins valides sont les chemins de dossier ou les chemins de dossier parent des packages.
    1. Mettre à jour un chemin de dossier de destination. La valeur par défaut est le chemin relatif au compte de stockage par défaut, qui est sélectionné à l’étape 1.
    1. Supprimer un mappage sélectionné à l’aide de **Supprimer le mappage**.
![Capture d’écran montrant la page Mapper les chemins des packages SSIS et des fichiers de configuration, dans laquelle vous pouvez ajouter un mappage.](media/how-to-migrate-ssis-job-ssms/step2.png)
![Capture d’écran montrant la page Mapper les chemins des packages SSIS et des fichiers de configuration, dans laquelle vous pouvez mettre à jour les chemins d’accès aux dossiers source et de destination.](media/how-to-migrate-ssis-job-ssms/step2-1.png)

1. Sélectionnez les travaux applicables à migrer et configurez les paramètres de l’*activité Exécuter le package SSIS*.

    - *Paramètres par défaut* s’applique à toutes les étapes sélectionnées par défaut. Pour plus d’informations sur chaque propriété, consultez l’*onglet Paramètres* de l’activité [Exécuter le package SSIS](how-to-invoke-ssis-package-ssis-activity.md) quand l’emplacement du package est *Système de fichiers (package)* .
    ![Capture d’écran montrant la page Sélectionner des travaux SSIS, dans laquelle vous pouvez configurer les paramètres de l’activité de package SSIS exécuté correspondante.](media/how-to-migrate-ssis-job-ssms/step3-1.png)
    - *Paramètre de l’étape* : configurez le paramètre pour une étape sélectionnée.
        
        **Appliquer les paramètres par défaut** : par défaut, cette case est cochée. Décochez-la pour configurer le paramètre pour l’étape sélectionnée uniquement.  
        Pour plus d’informations les autres propriétés, consultez l’*onglet Paramètres* de l’activité [Exécuter le package SSIS](how-to-invoke-ssis-package-ssis-activity.md) quand l’emplacement du package est *Système de fichiers (package)* .
    ![Capture d’écran montrant la page Sélectionner des travaux SSIS, dans laquelle vous pouvez appliquer les paramètres par défaut.](media/how-to-migrate-ssis-job-ssms/step3-2.png)

1. Générez et déployez le modèle ARM.
    1. Sélectionnez ou entrez le chemin de sortie des modèles ARM des pipelines ADF migrés. Le dossier sera créé automatiquement s’il n’existe pas.
    2. Sélectionnez l’option de **Déployer des modèles ARM dans votre fabrique de données** :
        - Par défaut, cette case est décochée. Vous pourrez déployer des modèles ARM générés plus tard manuellement.
        - Sélectionnez cette option pour déployer les modèles ARM générés directement dans la fabrique de données.
    ![Capture d’écran montrant la page Configurer la migration, dans laquelle vous pouvez sélectionner ou entrer le chemin de sortie des modèles ARM des pipelines ADF migrés, et sélectionner l’option Déployer des modèles ARM dans votre fabrique de données.](media/how-to-migrate-ssis-job-ssms/step4.png)

1. Procédez à la migration, puis vérifiez les résultats.
![Capture d’écran montrant la page Résultat de la migration, qui affiche la progression de la migration.](media/how-to-migrate-ssis-job-ssms/step5.png)

## <a name="next-steps"></a>Étapes suivantes

[Exécuter et superviser le pipeline](how-to-invoke-ssis-package-ssis-activity.md)