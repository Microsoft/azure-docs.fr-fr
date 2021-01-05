---
title: Copier des données avec l’outil d’Azure Copier des données
description: Créez une fabrique de données Azure, puis utilisez l’outil Copier des données pour copier les données d’un emplacement situé dans le Stockage Blob Azure vers un autre emplacement.
services: data-factory
documentationcenter: ''
author: dearandyxu
ms.author: yexu
manager: anandsub
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: quickstart
ms.date: 11/09/2020
ms.openlocfilehash: 282917a6bbf7edb962a87ad87810adde56206d97
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96013457"
---
# <a name="quickstart-use-the-copy-data-tool-to-copy-data"></a>Démarrage rapide : Utiliser l’outil Copier des données pour copier des données

> [!div class="op_single_selector" title1="Sélectionnez la version du service Data Factory que vous utilisez :"]
> * [Version 1](v1/data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
> * [Version actuelle](quickstart-create-data-factory-copy-data-tool.md)

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Dans ce guide de démarrage rapide, vous utilisez le portail Azure pour créer une fabrique de données. Vous utilisez ensuite l’outil Copier des données pour créer un pipeline qui copie des données depuis un dossier dans un stockage d’objets Blob Azure vers un autre dossier. 

> [!NOTE]
> Si vous débutez avec Azure Data Factory, consultez [Présentation d’Azure Data Factory](introduction.md) avant de commencer ce guide de démarrage rapide. 

[!INCLUDE [data-factory-quickstart-prerequisites](../../includes/data-factory-quickstart-prerequisites.md)] 

## <a name="create-a-data-factory"></a>Créer une fabrique de données

1. Lancez le navigateur web **Microsoft Edge** ou **Google Chrome**. L’interface utilisateur de Data Factory n’est actuellement prise en charge que par les navigateurs web Microsoft Edge et Google Chrome.
1. Accédez au [portail Azure](https://portal.azure.com). 
1. Dans le menu du portail Azure, sélectionnez **Créer une ressource** > **Intégration** > **Data Factory** :

    ![Création d’une fabrique de données](./media/doc-common-process/new-azure-data-factory-menu.png)

1. Dans la page **Nouvelle fabrique de données**, entrez **ADFTutorialDataFactory** comme **nom**. 
 
   Le nom de la fabrique de données Azure doit être un nom *global unique*. Si l’erreur suivante s’affiche, changez le nom de la fabrique de données (par exemple, **&lt;votrenom&gt;ADFTutorialDataFactory**), puis tentez de la recréer. Consultez l’article [Data Factory - Règles d’affectation des noms](naming-rules.md) pour en savoir plus sur les règles d’affectation des noms d’artefacts Data Factory.
  
   ![Erreur quand le nom n’est pas disponible](./media/doc-common-process/name-not-available-error.png)
1. Pour **Abonnement**, sélectionnez l’abonnement Azure dans lequel vous voulez créer la fabrique de données. 
1. Pour **Groupe de ressources**, réalisez l’une des opérations suivantes :
     
   - Sélectionnez **Utiliser l’existant**, puis sélectionnez un groupe de ressources existant dans la liste. 
   - Sélectionnez **Créer**, puis entrez le nom d’un groupe de ressources.   
         
   Pour plus d’informations sur les groupes de ressources, consultez [Utilisation des groupes de ressources pour gérer vos ressources Azure](../azure-resource-manager/management/overview.md).  
1. Pour **Version**, sélectionnez **V2**.
1. Pour **Emplacement**, sélectionnez l’emplacement de la fabrique de données.

   La liste n’affiche que les emplacements pris en charge par Data Factory et où vos métadonnées Azure Data Factory sont stockées. Les magasins de données associés (tels que Stockage Azure et Azure SQL Database) et les services de calcul (comme Azure HDInsight) utilisés par Data Factory peuvent s’exécuter dans d’autres régions.

1. Sélectionnez **Create** (Créer).

1. Une fois la création terminée, la page **Data Factory** s’affiche. Sélectionnez la vignette **Créer et surveiller** pour démarrer l’application d’interface utilisateur (IU) d’Azure Data Factory dans un onglet séparé.
   
   ![Page d’accueil de la fabrique de données, avec la vignette « Créer et surveiller »](./media/doc-common-process/data-factory-home-page.png)

## <a name="start-the-copy-data-tool"></a>Démarrer l’outil Copier des données

1. Sur la page **Prise en main**, sélectionnez la vignette **Copier des données** pour démarrer l’outil Copier des données. 

   ![Vignette « Copier des données »](./media/doc-common-process/get-started-page.png)

1. Sur la page **Propriétés** de l’outil de copie des données, vous pouvez spécifier un nom pour le pipeline et sa description. Sélectionnez ensuite **Suivant**. 

   ![Page « Propriétés »](./media/quickstart-create-data-factory-copy-data-tool/copy-data-tool-properties-page.png)
1. Sur la page **Banque de données source**, procédez comme suit :

    a. Cliquez sur **+Créer une connexion** pour ajouter une connexion.

    b. Sélectionnez le type de service lié à créer pour la connexion source. Dans ce tutoriel, nous utilisons le **Stockage Blob Azure**. Sélectionnez-le dans la galerie, puis sélectionnez **Continuer**.
    
    ![Sélectionner l’objet blob](./media/quickstart-create-data-factory-copy-data-tool/select-blob-source.png)

    c. Dans la page **Nouveau service lié (Stockage Blob Azure)** , spécifiez un nom pour votre service lié. Sélectionnez votre compte de stockage dans la liste **Nom du compte de stockage**, testez la connexion, puis sélectionnez **Créer**. 

    ![Configurer le compte de stockage Blob Azure](./media/quickstart-create-data-factory-copy-data-tool/configure-blob-storage.png)

    d. Sélectionnez le service lié nouvellement créé en tant que source, puis cliquez sur **Suivant**.


1. Sur la page **Choisir le fichier ou le dossier de sortie**, procédez comme suit :

   a. Cliquez sur **Parcourir** pour accédez au dossier **adftutorial/input**, sélectionnez le fichier **emp.txt**, puis cliquez sur **Choisir**. 

   d. Cochez la case **Copie binaire** pour copier le fichier tel quel, puis sélectionnez **Suivant**. 

   ![Page « Choisir le fichier ou le dossier d’entrée »](./media/quickstart-create-data-factory-copy-data-tool/select-binary-copy.png)


1. Dans la page **Banque de données de destination**, sélectionnez le service lié **Stockage Blob Azure** que vous avez créé, puis choisissez **Suivant**. 

1. Dans la page **Choisir le fichier ou le dossier de sortie**, entrez **adftutorial/output** comme chemin du dossier, puis sélectionnez **Suivant**. 

   ![Page « Choisir le fichier ou le dossier de sortie »](./media/quickstart-create-data-factory-copy-data-tool/configure-sink-path.png) 

1. Sur la page **Paramètres**, cliquez sur **Suivant** pour utiliser les configurations par défaut. 

1. Dans la page **Résumé**, passez en revue tous les paramètres, puis cliquez sur **Suivant**. 

1. Sur la page **Déploiement terminé**, sélectionnez **Surveiller** pour surveiller le pipeline que vous avez créé. 

    ![Page « Déploiement terminé »](./media/quickstart-create-data-factory-copy-data-tool/deployment-page.png)

1. L’application bascule vers l’onglet **Surveiller**. Vous voyez l’état du pipeline dans cet onglet. Sélectionnez **Actualiser** pour actualiser la liste. Cliquez sur le lien en dessous de **Nom du pipeline** pour afficher les détails de l’exécution de l’activité ou réexécuter le pipeline. 
   
    ![Actualiser le pipeline](./media/quickstart-create-data-factory-copy-data-tool/refresh-pipeline.png)

1. Dans la page Exécutions de l’activité, sélectionnez le lien **Détails** (icône de lunettes) en dessous de la colonne **Nom de l’activité** pour plus d’informations sur l’opération de copie. Pour plus d’informations sur les propriétés, consultez [Vue d’ensemble de l’activité de copie](copy-activity-overview.md). 

1. Pour revenir à l’affichage Exécutions de pipeline, sélectionnez le lien **Toutes les exécutions de pipelines** dans le menu de navigation. Sélectionnez **Actualiser** pour actualiser l’affichage. 

1. Vérifiez que le fichier **emp.txt** est créé dans le dossier **de sortie** du conteneur **adftutorial**. Si le dossier de sortie n’existe pas, le service Data Factory le crée automatiquement. 

1. Passez sur l’onglet **Auteur** au-dessus de l’onglet **Surveiller** du volet gauche afin de pouvoir modifier les services liés, jeux de données et pipelines. Pour apprendre à les modifier dans l’interface utilisateur de Data Factory, consultez [Créer une fabrique de données à l’aide du portail Azure](quickstart-create-data-factory-portal.md).

    ![Sélectionner l’onglet Auteur](./media/quickstart-create-data-factory-copy-data-tool/select-author.png)

## <a name="next-steps"></a>Étapes suivantes
Dans cet exemple, le pipeline copie les données d’un emplacement vers un autre dans un stockage Blob Azure. Pour en savoir plus sur l’utilisation de Data Factory dans d’autres scénarios, consultez les [didacticiels](tutorial-copy-data-portal.md). 
