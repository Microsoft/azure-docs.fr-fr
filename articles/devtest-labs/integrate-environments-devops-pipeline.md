---
title: Intégrer des environnements à Azure Pipelines dans Azure DevTest Labs
description: Découvrez comment intégrer des environnements Azure DevTest Labs à vos pipelines d’intégration continue et de livraison continue (CI/CD) Azure DevOps.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: be726b2a3f67fd3dada4fdc3cf794922a3c18d06
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85483021"
---
# <a name="integrate-environments-into-your-azure-devops-cicd-pipelines"></a>Intégrer des environnements à vos pipelines CI/CD Azure DevOps
Vous pouvez utiliser l’extension Azure DevTest Labs Tasks installée dans Azure DevOps Services (anciennement Visual Studio Team Services) pour intégrer facilement votre pipeline de build et mise en production d’intégration continue (CI) et de livraison continue (CD) dans Azure DevTest Labs. Ces extensions facilitent le déploiement rapide d’un [environnement](devtest-lab-test-env.md) pour une tâche de test spécifique, ainsi que la suppression du déploiement une fois le test terminé. 

Cet article explique comment créer et déployer un environnement, puis supprimer l’environnement, le tout dans un seul pipeline complet. Normalement, vous effectuez chacune de ces tâches individuellement dans votre pipeline personnalisé de build-test-déploiement. Les extensions utilisées dans cet article s’ajoutent à ces [tâches de machine virtuelle DTL créer/supprimer](devtest-lab-integrate-ci-cd.md) :

- Créer un environnement
- Supprimer un environnement

## <a name="before-you-begin"></a>Avant de commencer
Pour pouvoir intégrer votre pipeline CI/CD à Azure DevTest Labs, installez l’extension [Azure DevTest Labs Tasks](https://marketplace.visualstudio.com/items?itemName=ms-azuredevtestlabs.tasks) à partir de Visual Studio Marketplace. 

## <a name="create-and-configure-the-lab-for-environments"></a>Créer et configurer le lab pour les environnements
Cette section décrit comment créer et configurer un lab dans lequel déployer l’environnement Azure.

1. [Créez un lab](devtest-lab-create-lab.md) si vous n’en avez pas déjà. 
2. Configurez le lab et créez un modèle d’environnement en suivant les instructions de cet article : [Créer des environnements de plusieurs machines virtuelles et des ressources PaaS avec les modèles Azure Resource Manager](devtest-lab-create-environment-from-arm.md).
3. Pour cet exemple, utilisez un modèle de démarrage rapide Azure existant : [https://azure.microsoft.com/resources/templates/201-web-app-redis-cache-sql-database/](https://azure.microsoft.com/resources/templates/201-web-app-redis-cache-sql-database/).
4. Copiez le dossier **201-web-app-redis-cache-sql-database** dans le dossier **ArmTemplate** du dépôt configuré à l’étape 2.

## <a name="create-a-release-definition"></a>Création d’une définition de version
Pour créer la définition de mise en production, effectuez les étapes suivantes :

1.  Sous l’onglet **Mises en production** du **hub Build et mise en production**, sélectionnez le bouton **+ (signe plus)** .
2.  Dans la fenêtre **Créer une définition de mise en production**, sélectionnez le modèle **Vide**, puis sélectionnez **Suivant**.
3.  Sélectionnez **Choisir plus tard**, puis sélectionnez **Créer** pour créer une définition de mise en production avec un environnement par défaut et aucun artefact lié.
4.  Pour ouvrir le menu contextuel, dans la nouvelle définition de mise en production, sélectionnez les **points de suspension (...)** à côté du nom de l’environnement, puis sélectionnez **Configurer les variables**.
5.  Dans la fenêtre **Configurer l’environnement**, pour les variables que vous utilisez dans les tâches de définition de mise en production, entrez les valeurs suivantes :
1.  Pour **administratorLogin**, entrez le nom de connexion de l’administrateur SQL.
2.  Pour **administratorLoginPassword**, entrez le mot de passe à utiliser avec la connexion de l’administrateur SQL. Utilisez l’icône en forme de cadenas pour masquer et sécuriser le mot de passe.
3.  Pour **databaseName**, entrez le nom de la base de données SQL.
4.  Ces variables s’appliquent aux exemples d’environnements de cet article. Elles peuvent être différentes dans d’autres environnements.

## <a name="create-an-environment"></a>Créer un environnement
L’étape suivante du déploiement consiste à créer l’environnement nécessaire pour le développement ou les tests.

1. Dans la définition de mise en production, sélectionnez **Ajouter des tâches**.
2. Sous l’onglet **Tâches**, ajoutez une tâche Azure DevTest Labs - Créer un environnement. Configurez la tâche comme indiqué ci-dessous :
    1. Pour **Abonnement RM Azure**, sélectionnez une connexion dans la liste **Connexions au service Azure disponibles**, ou créez une connexion d’autorisations plus limitée à votre abonnement Azure. Pour plus d’informations, consultez [Point de terminaison de service Azure Resource Manager](/azure/devops/pipelines/library/service-endpoints).
2. Pour **Nom du lab**, sélectionnez le nom de l’instance que vous avez créée précédemment*.
3. Pour **Nom du dépôt**, sélectionnez le dépôt dans lequel a été mis le modèle Resource Manager (201)*.
4. Pour **Nom du modèle**, sélectionnez le nom de l’environnement que vous avez enregistré dans votre dépôt de code source*. 
5. **Nom du lab**, **Nom du dépôt** et **Nom du modèle** sont les représentations conviviales des ID de ressources Azure. Pour éviter les erreurs provoquées par la saisie manuelle des noms conviviaux, sélectionnez les noms dans les listes déroulantes.
6. Pour **Nom de l’environnement**, entrez un nom qui identifie de façon unique l’instance de l’environnement dans le lab.  Ce nom doit être unique dans le lab.
7. Avec les variables **Fichier de paramètres** et **Paramètres**, vous pouvez passer des paramètres personnalisés à l’environnement. Utilisez-en une, ou les deux, pour définir les valeurs des paramètres. Cet exemple utilise la section Paramètres. Utilisez les noms des variables que vous avez définies dans l’environnement, par exemple : `-administratorLogin "$(administratorLogin)" -administratorLoginPassword "$(administratorLoginPassword)" -databaseName "$(databaseName)" -cacheSKUCapacity 1`
8. Les informations contenues dans le modèle d’environnement peuvent être passées à la section de sortie du modèle. Cochez **Créer les variables de sortie à partir de la sortie du modèle d’environnement** afin que d’autres tâches puissent utiliser les données. Suivez ce format : `$(Reference name.Output Name)`. Par exemple, si le nom de la référence était DTL et que le nom de la sortie dans le modèle était location, nous aurions la variable `$(DTL.location)`.

## <a name="delete-the-environment"></a>Supprimer l’environnement
La dernière étape consiste à supprimer l’environnement que vous avez déployé dans votre instance Azure DevTest Labs. Normalement, vous supprimez l’environnement après avoir exécuté les tâches de développement ou de test nécessaires sur les ressources déployées.

Dans la définition de mise en production, sélectionnez **Ajouter des tâches** et, sous l’onglet **Déployer**, ajoutez une tâche **Azure DevTest Labs - Supprimer l’environnement**. Configurez-le comme suit :

1. Pour supprimer la machine virtuelle, consultez [Azure DevTest Labs Tasks](https://marketplace.visualstudio.com/items?itemName=ms-azuredevtestlabs.tasks).
    1. Pour **Abonnement RM Azure**, sélectionnez une connexion dans la liste **Connexions au service Azure disponibles**, ou créez une connexion d’autorisations plus limitée à votre abonnement Azure. Pour plus d’informations, consultez [Point de terminaison de service Azure Resource Manager](/azure/devops/pipelines/library/service-endpoints).
    2. Pour **Nom du lab**, sélectionnez le lab dont fait partie l’environnement.
    3. Pour **Nom de l’environnement**, entrez le nom de l’environnement à supprimer.
2. Entrez un nom pour la définition de mise en production, puis enregistrez-le.

## <a name="next-steps"></a>Étapes suivantes
Voir les articles suivants : 
- [Créer des environnements de plusieurs machines virtuelles avec les modèles Azure Resource Manager](devtest-lab-create-environment-from-arm.md).
- Modèles Resource Manager de démarrage rapide pour l’automatisation de DevTest Labs, disponibles dans le [dépôt DevTest Labs sur GitHub](https://github.com/Azure/azure-quickstart-templates).
- [Page sur la résolution des problèmes avec VSTS](/azure/devops/pipelines/troubleshooting)

