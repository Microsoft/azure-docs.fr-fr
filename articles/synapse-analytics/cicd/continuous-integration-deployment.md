---
title: Intégration et livraison continues pour l’espace de travail Synapse
description: Découvrez comment utiliser l’intégration et la livraison continues pour déployer les changements dans un espace de travail d’un environnement (développement, test, production) à un autre.
services: synapse-analytics
author: liud
ms.service: synapse-analytics
ms.topic: conceptual
ms.date: 11/20/2020
ms.author: liud
ms.reviewer: pimorano
ms.openlocfilehash: 5dbd49312b58dc656e2239e8a0a4acea614023de
ms.sourcegitcommit: e5f9126c1b04ffe55a2e0eb04b043e2c9e895e48
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96317134"
---
# <a name="continuous-integration-and-delivery-for-azure-synapse-workspace"></a>Intégration et livraison continues pour l’espace de travail Azure Synapse

## <a name="overview"></a>Vue d’ensemble

L’intégration continue (CI) est le processus d’automatisation de création et de test du code chaque fois qu’un membre de l’équipe valide les modifications apportées au contrôle de version. Le déploiement continu (CD) est le processus de création, de test, de configuration et de déploiement de plusieurs environnements de test ou intermédiaires dans un environnement de production.

Pour l’espace de travail Azure Synapse, l’intégration et la livraison continues (CI/CD) déplacent toutes les entités d’un environnement (développement, test, production) à un autre. La promotion de votre espace de travail vers un autre espace de travail se compose de deux parties : utiliser des [modèles Azure Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/templates/overview) pour créer ou mettre à jour des ressources d’espace de travail (pools et espace de travail), et migrer les artefacts (scripts SQL, bloc-notes, définition de travail Spark, pipelines, jeux de données, flux de données, etc.) avec des outils CI/CD Synapse dans Azure DevOps. 

Cet article décrit l’utilisation d’un pipeline de mise en production Azure pour automatiser le déploiement d’un espace de travail Synapse dans plusieurs environnements.

## <a name="prerequisites"></a>Prérequis

-   L’espace de travail utilisé pour le développement a été configuré avec un référentiel Git dans Studio. Pour plus d’informations, consultez [Contrôle de code source dans Synapse Studio](source-control.md).
-   Un projet Azure DevOps a été préparé pour exécuter le pipeline de mise en production.

## <a name="set-up-a-release-pipelines"></a>Configurer un pipeline de mise en production

1.  Dans [Azure DevOps](https://dev.azure.com/), ouvrez le projet créé pour la mise en production.

1.  Sur le côté gauche de la page, sélectionnez **Pipelines**, puis sélectionnez **Versions**.

    ![Sélectionner Pipelines, Versions](media/create-release-1.png)

1.  Sélectionnez **Nouveau pipeline** ou, si vous avez des pipelines existants, sélectionnez **Nouveau** puis **Nouveau pipeline de mise en production**.

1.  Sélectionnez le modèle **Tâche vide**.

    ![Sélectionner Projet vide](media/create-release-select-empty.png)

1.  Dans la zone **Nom de la phase**, entrez le nom de votre environnement.

1.  Sélectionnez **Ajouter un artefact**, puis choisissez le dépôt Git configuré avec votre instance Synapse Studio de développement. Sélectionnez le référentiel Git que vous avez utilisé pour gérer le modèle ARM de pools et l’espace de travail. Si vous utilisez GitHub en tant que source, vous devez créer une connexion de service pour votre compte GitHub et les référentiels d’extraction. Pour plus d’informations sur la [connexion de service](https://docs.microsoft.com/azure/devops/pipelines/library/service-endpoints) 

    ![Ajouter une branche de publication](media/release-creation-github.png)

1.  Sélectionnez la branche du modèle ARM pour la mise à jour des ressources. Pour **Version par défaut**, sélectionnez **La dernière de la branche par défaut**.

    ![Ajouter un modèle ARM](media/release-creation-arm-branch.png)

1.  Sélectionnez la [branche de publication](source-control.md#configure-publishing-settings) du dépôt comme **branche par défaut**. Par défaut, cette branche de publication est `workspace_publish`. Pour **Version par défaut**, sélectionnez **La dernière de la branche par défaut**.

    ![Ajouter un artefact](media/release-creation-publish-branch.png)

## <a name="set-up-a-stage-task-for-arm-resource-create-and-update"></a>Configurer une tâche intermédiaire pour la création et la mise à jour de ressources ARM 

Ajoutez une tâche de déploiement Azure Resource Manager pour créer ou mettre à jour des ressources, y compris l’espace de travail et les pools :

1. Dans la vue des phases, sélectionnez **Afficher les tâches de phase**.

    ![Vue des phases](media/release-creation-stage-view.png)

1. Créer une tâche. Recherchez **Déploiement de modèles ARM**, puis sélectionnez **Ajouter**.

1. Dans la tâche Déploiement, sélectionnez l’abonnement, le groupe de ressources et l’emplacement de l’espace de travail cible. Fournissez les informations d’identification si nécessaire.

1. Dans la liste **Actions**, sélectionnez **Créer ou mettre à jour un groupe de ressources**.

1. Sélectionnez le bouton de sélection ( **…** ) en regard de la zone **Modèle**. Rechercher le modèle Azure Resource Manager de votre espace de travail cible

1. Sélectionnez **…** en regard de la zone **Paramètres du modèle** pour choisir le fichier de paramètres.

1. Sélectionnez **…** en regard de la zone **Remplacer les paramètres du modèle** et entrez les valeurs de paramètre souhaitées pour l’espace de travail cible. 

1. Sélectionnez **Incrémentiel** comme **Mode de déploiement**.
    
    ![déploiement de l’espace de travail et des pools](media/pools-resource-deploy.png)

1. (Facultatif) Ajoutez **Azure PowerShell** pour l’octroi et la mise à jour de l’attribution de rôle de l’espace de travail. Si vous utilisez un pipeline de mise en production pour créer un espace de travail Synapse, le principal du service du pipeline est ajouté en tant qu’administrateur de l’espace de travail par défaut. Vous pouvez exécuter PowerShell pour accorder à d’autres comptes l’accès à l’espace de travail. 
    
    ![accorder l’autorisation](media/release-creation-grant-permission.png)

 > [!WARNING]
> En mode de déploiement complet, les ressources présentes dans le groupe de ressources mais pas spécifiées dans le modèle Resource Manager sont **supprimées**. Pour plus d’informations, consultez [Modes de déploiement Azure Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/templates/deployment-modes).

## <a name="set-up-a-stage-task-for-artifacts-deployment"></a>Configurer une tâche intermédiaire pour le déploiement d’artefacts 

Utilisez l’extension de [déploiement d’espace de travail Synapse](https://marketplace.visualstudio.com/items?itemName=AzureSynapseWorkspace.synapsecicd-deploy) pour déployer d’autres éléments dans l’espace de travail Synapse, comme un jeu de données, un script SQL, un notebook, une définition de travail Spark, un flux de données, un pipeline, un service lié, des informations d’identification et un runtime d'intégration.  

1. Recherchez et récupérez l’extension à partir de la **Place de marché Azure DevOps**(https://marketplace.visualstudio.com/azuredevops) 

     ![Obtenir une extension](media/get-extension-from-market.png)

1. Sélectionnez une organisation pour l’installation de l’extension. 

     ![Installer l’extension](media/install-extension.png)

1. Assurez-vous que le principal du service du pipeline Azure DevOps a reçu l’autorisation d’abonnement et a également été affecté en tant qu’administrateur d’espace de travail pour l’espace de travail cible. 

1. Créer une tâche. Recherchez **Déploiement d’espace de travail Synapse**, puis sélectionnez **Ajouter**.

     ![Ajouter une extension](media/add-extension-task.png)

1.  Dans la tâche, fournissez les informations du référentiel Git associées de **workspace_publish**, puis sélectionnez le groupe de ressources, la région, le nom et l’environnement cloud de l’espace de travail cible. Fournissez des paramètres et des valeurs si nécessaire.

    ![Déployer un espace de travail Synapse](media/create-release-artifacts-deployment.png)

> [!IMPORTANT]
> Dans les scénarios d’intégration et de livraison continues, le type de runtime d’intégration (IR) doit être le même dans les différents environnements. Par exemple, si vous avez un runtime d’intégration auto-hébergé dans l’environnement de développement, le même runtime d’intégration doit aussi être de type auto-hébergé dans les autres environnements (test, production). De même, si vous partagez des runtimes d’intégration dans plusieurs phases, vous devez les configurer comme étant liés et auto-hébergés dans tous les environnements (développement, test, production).

## <a name="create-release-for-deployment"></a>Créer une mise en production pour le déploiement 

Après avoir enregistré toutes les modifications, vous pouvez sélectionner **Créer une version** pour créer manuellement une mise en production. Pour automatiser la création des mises en production, consultez les [déclencheurs de mise en production Azure DevOps](https://docs.microsoft.com/azure/devops/pipelines/release/triggers).

   ![Sélectionner Créer une mise en production](media/release-creation-manually.png)

## <a name="best-practices-for-cicd"></a>Meilleures pratiques pour CI/CD

Si vous utilisez une intégration Git avec votre espace de travail Synapse, et disposez d’un pipeline CI/CD qui déplace vos modifications du développement aux tests, puis en production, nous vous recommandons les bonnes pratiques suivantes :

-   **Intégration Git**. Configurez uniquement votre espace de travail Synapse de développement avec l’intégration Git. Les modifications au niveau des espaces de travail de test et de production sont déployées via CI/CD et ne nécessitent pas d’intégration Git.
-   **Préparez les pools avant la migration des artefacts**. Si vous disposez d’un script SQL ou d’un notebook attaché à des pools dans l’espace de travail de développement, le même nom de pools dans différents environnements est attendu. 
-   **Infrastructure en tant que code (IaC)** . La gestion de l'infrastructure (réseaux, machines virtuelles, équilibreurs de charge et topologie de connexion) dans un modèle descriptif utilise le même contrôle de version que celui utilisé par l'équipe DevOps pour le code source. 
-   **Autres**. Consultez les [meilleures pratiques pour les artefacts ADF](/azure/data-factory/continuous-integration-deployment#best-practices-for-cicd)


