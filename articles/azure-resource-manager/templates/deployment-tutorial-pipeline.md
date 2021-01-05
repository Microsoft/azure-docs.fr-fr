---
title: Intégration continue avec Azure Pipelines
description: Découvrez comment générer, tester et déployer en continu des modèles Azure Resource Manager (modèles ARM).
ms.date: 08/24/2020
ms.topic: tutorial
ms.author: jgao
ms.openlocfilehash: 8e9f047497f493752947d8115084dcfe86f5e040
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97588129"
---
# <a name="tutorial-continuous-integration-of-arm-templates-with-azure-pipelines"></a>Tutoriel : Intégration continue de modèles ARM avec Azure Pipelines

Dans le [tutoriel précédent](./deployment-tutorial-linked-template.md), vous avez déployé un modèle lié.  Dans ce tutoriel, vous allez apprendre à utiliser Azure Pipelines pour générer et déployer en continu des projets de modèle Azure Resource Manager (modèle ARM).

Azure DevOps fournit des services de développement pour prendre en charge les équipes afin de planifier le travail, collaborer sur le développement de code, mais aussi générer et déployer des applications. Les développeurs peuvent travailler dans le cloud à l’aide d’Azure DevOps Services. Azure DevOps fournit un ensemble de fonctionnalités auxquelles vous pouvez accéder via votre navigateur web ou un client IDE. Azure Pipelines est une de ces fonctionnalités. Azure Pipelines est un service complet d’intégration et de diffusion en continu. Il fonctionne avec votre fournisseur Git par défaut et peut être déployé sur la plupart des services cloud majeurs. Une fois déployé, il vous permet d’automatiser la génération, le test et le déploiement de votre code pour Microsoft Azure, Google Cloud Platform, ou Amazon Web Services.

> [!NOTE]
> Sélectionnez un nom de projet. Lorsque vous parcourez le didacticiel, remplacez chaque occurrence de **AzureRmPipeline** par le nom de votre projet.
> Ce nom de projet est utilisé pour générer des noms de ressource.  L’une des ressources est un compte de stockage. Ce nom doit comprendre entre 3 et 24 caractères, uniquement des lettres en minuscules et des nombres. Le nom doit être unique. Dans le modèle, le nom du compte de stockage est le nom du projet auquel **store** est ajouté, et le nom du projet doit comprendre entre 3 et 11 caractères. Le nom du projet doit donc respecter les exigences du nom du compte de stockage et comporter moins de 11 caractères.

Ce tutoriel décrit les tâches suivantes :

> [!div class="checklist"]
> * Préparer un dépôt GitHub
> * Créer un projet Azure DevOps
> * Créer un pipeline Azure
> * Vérifier le déploiement du pipeline
> * Mettre à jour le modèle et le redéployer
> * Nettoyer les ressources

Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour effectuer ce qui est décrit dans cet article, vous avez besoin des éléments suivants :

* **Un compte GitHub**, que vous utiliserez pour créer un référentiel pour vos modèles. Si vous n’en avez pas, vous pouvez en [créer un gratuitement](https://github.com). Pour plus d’informations sur l’utilisation de référentiels GitHub, consultez [Développer des référentiels GitHub](/azure/devops/pipelines/repos/github).
* **Installer Git**. Lors de ce didacticiel, nous utiliserons *Git Bash* ou *Git Shell*. Pour en savoir plus sur leur installation, veuillez consulter la page [Installer Git](https://www.atlassian.com/git/tutorials/install-git).
* **Une organisation Azure DevOps**. Si vous n’en avez pas, vous pouvez en créer une gratuitement. Pour en savoir plus, veuillez consulter [Créer une organisation ou une collection de projet](/azure/devops/organizations/accounts/create-organization?view=azure-devops).
* (facultatif) **Visual Studio Code avec l’extension Outils Resource Manager**. Consultez [Démarrage rapide : Créer des modèles ARM avec Visual Studio Code](quickstart-create-templates-use-visual-studio-code.md).

## <a name="prepare-a-github-repository"></a>Préparer un dépôt GitHub

GitHub est utilisé pour stocker le code source de votre projet, y compris les modèles Resource Manager. Pour connaître les autres référentiels pris en charge, veuillez consulter [Référentiels pris en charge par Azure DevOps](/azure/devops/pipelines/repos/?view=azure-devops).

### <a name="create-a-github-repository"></a>Créer un référentiel GitHub

Si vous n’avez pas de compte GitHub, consultez [Prérequis](#prerequisites).

1. Connectez-vous à [GitHub](https://github.com).
1. Sélectionnez votre image de compte dans le coin supérieur droit, puis **vos référentiels**.

    ![Azure Resource Manager, Azure DevOps Azure Pipelines, créer un référentiel GitHub](./media/deployment-tutorial-pipeline/azure-resource-manager-devops-pipelines-github-repository.png)

1. Sélectionnez le bouton vert **New** (Nouveau).
1. Dans **Repository name** (nom du référentiel), entrez un nom de référentiel.  Par exemple, **AzureRmPipeline-repo**. N’oubliez pas de remplacer chaque occurrence de **AzureRmPipeline** par le nom de votre projet. Vous pouvez sélectionner **Public** ou **Private** (Privé) pour ce didacticiel. Puis sélectionnez **Create repository** (Créer un référentiel).
1. Notez l’URL. L’URL du dépôt est au format suivant : `https://github.com/[YourAccountName]/[YourRepositoryName]` .

Ce référentiel est appelé *remote repository* (référentiel distant). Chaque développeur du même projet peut cloner son *dépôt local*, puis fusionner ses modifications dans le dépôt distant.

### <a name="clone-the-remote-repository"></a>Cloner le référentiel distant

1. Ouvrez Git Shell ou Git Bash. Consultez les [Conditions préalables](#prerequisites).
1. Vérifiez que votre dossier actif est **GitHub**.
1. Exécutez la commande suivante :

    ```bash
    git clone https://github.com/[YourAccountName]/[YourGitHubRepositoryName]
    cd [YourGitHubRepositoryName]
    mkdir CreateWebApp
    cd CreateWebApp
    pwd
    ```

    Remplacez `[YourAccountName]` (NomDeVotreCompte) par le nom de votre compte GitHub, puis remplacez `[YourGitHubRepositoryName]` (NomDeVotreRéférentielGitHub) par le nom de du référentiel que vous avez créé lors de la procédure précédente.

Le modèle est stocké dans le dossier _CreateWebApp_. La commande `pwd` affiche le chemin d’accès au dossier. C’est dans ce chemin d’accès que vous enregistrerez le modèle lors de la procédure suivante.

### <a name="download-a-quickstart-template"></a>Télécharger un modèle de démarrage rapide

Au lieu de créer les modèles, vous pouvez les télécharger et les enregistrer dans le dossier _CreateWebApp_.

* Modèle principal : https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/get-started-deployment/linked-template/azuredeploy.json
* Modèle lié : https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/get-started-deployment/linked-template/linkedStorageAccount.json

Le nom du dossier et les noms des fichiers sont utilisés tels quels dans le pipeline. Si vous modifiez ces noms, vous devez mettre à jour les noms utilisés dans le pipeline.

### <a name="push-the-template-to-the-remote-repository"></a>Transmettre le modèle vers le référentiel distant

Le fichier _azuredeploy.json_ a été ajouté au dépôt local. Ensuite, vous chargez le modèle dans le référentiel distant.

1. Si ce n’est pas déjà fait, ouvrez *Git Shell* ou *Git Bash*.
1. Changez de répertoire pour passer au dossier _CreateWebApp_ dans votre dépôt local.
1. Vérifiez que le fichier _azuredeploy.json_ se trouve dans le dossier.
1. Exécutez la commande suivante :

    ```bash
    git add .
    git commit -m "Add web app templates."
    git push origin main
    ```

    Vous pouvez obtenir un avertissement sur LF. Vous pouvez ignorer cet avertissement. **main** est la branche primaire.  Généralement, il faut créer une branche pour chaque mise à jour. Pour simplifier ce tutoriel, nous utilisons directement la branche primaire.

1. Accédez à votre référentiel GitHub à partir d’un navigateur. L’URL est `https://github.com/[YourAccountName]/[YourGitHubRepository]`. Vous devez voir le dossier _CreateWebApp_ et les trois fichiers à l’intérieur du dossier.
1. Sélectionnez _linkedStorageAccount.json_ pour ouvrir le modèle.
1. Sélectionnez le bouton **Raw**. L’URL commence par `https://raw.githubusercontent.com`.
1. Copiez l’URL. Vous devrez fournir cette valeur lors de la configuration du pipeline ultérieurement dans le tutoriel.

Pour le moment, vous avez créé un dépôt GitHub et chargé les modèles sur le dépôt.

## <a name="create-a-devops-project"></a>Créer un projet DevOps

Une organisation DevOps est nécessaire avant de procéder à la procédure suivante. Si vous n’en avez pas, consultez [Prérequis](#prerequisites).

1. Connectez-vous à [Azure DevOps](https://dev.azure.com).
1. Sélectionnez une organisation DevOps sur la gauche.

    ![Azure Resource Manager, Azure DevOps, Azure Pipelines, créer un projet Azure DevOps](./media/deployment-tutorial-pipeline/azure-resource-manager-devops-pipelines-create-devops-project.png)

1. Sélectionnez **Nouveau projet**. Si aucun projet ne s’affiche, la page Create project (Créer un projet) s’ouvre automatiquement.
1. Saisissez les valeurs suivantes :

    * **Project name** (Nom du projet) : nommez votre projet. Vous pouvez utiliser le nom du projet que vous avez choisi au tout début du didacticiel.
    * **Version control** (Contrôle de version) : Sélectionnez **Git**. Vous devrez peut-être développer **Advanced** (Avancé) pour voir **Version control** (Contrôle de version).

    Utilisez les valeurs par défaut pour les autres propriétés.
1. Sélectionnez **Create** (Créer).

Pour créer une connexion de service qui servira à déployer des projets vers Azure :

1. En bas du menu situé à gauche, sélectionnez **Project settings** (Paramètres du projet).
1. Sous **Pipelines**, sélectionnez **Service connections** (Connexions au service).
1. Sélectionnez **Nouvelle connexion de service**, **Azure Resource Manager**, puis **Suivant**.
1. Sélectionnez **Principal du service**, puis **Suivant**.
1. Saisissez les valeurs suivantes :

    * **Scope level** (Niveau d’étendue) : sélectionnez **Subscription** (Abonnement).
    * **Subscription** (Abonnement) : sélectionnez votre abonnement.
    * **Groupe de ressources** : Laissez cette valeur vide.
    * **Connection name** (Nom de la connexion) : nommez la connexion. Par exemple, **AzureRmPipeline-conn**. Notez ce nom : vous en aurez besoin lors de la création de votre pipeline.
    * **Accorder une autorisation d’accès à tous les pipelines**. (sélectionné)
1. Sélectionnez **Enregistrer**.

## <a name="create-a-pipeline"></a>Créer un pipeline

Pour le moment, vous avez mené à bien les tâches suivantes :  Si vous avez ignoré les sections précédentes, car vous êtes familiarisé avec GitHub et DevOps, vous devez effectuer ces tâches avant de continuer.

* Création d’un dépôt GitHub, puis enregistrement des modèles dans le dossier _CreateWebApp_ du dépôt.
* Création d’un projet DevOps et création d’une connexion au service Azure Resource Manager.

Pour créer un pipeline avec une étape pour déployer un modèle :

1. Dans le menu situé à gauche, sélectionnez **Pipelines**.
1. Sélectionnez **Nouveau pipeline**.
1. Sous l’onglet **Connect** (Connexion), sélectionnez **GitHub**. Si le système vous y invite, entrez vos informations d’identification de GitHub, puis suivez les instructions. Si vous voyez l’écran suivant, sélectionnez **Only select repositories** (Sélectionner uniquement les référentiels), puis vérifiez que votre référentiel se trouve dans la liste avant de sélectionner **Approve & Install** (Valider et installer).

    ![Azure Resource Manager, Azure DevOps, Azure Pipelines, sélectionner uniquement les référentiels](./media/deployment-tutorial-pipeline/azure-resource-manager-devops-pipelines-only-select-repositories.png)

1. Sous l’onglet **Select** (Sélection), sélectionnez votre référentiel. Le nom par défaut est `[YourAccountName]/[YourGitHubRepositoryName]`.
1. Sous l’onglet **Configure** (Configuration), sélectionnez **Starter pipeline** (Pipeline de démarrage). Il indique le fichier de pipeline _azure-pipelines.yml_ avec deux étapes de script.
1. Supprimez les deux étapes de script du fichier _.yml_.
1. Placez le curseur sur la ligne après **steps :** .
1. Sélectionnez **Afficher l’Assistant** à droite de l’écran pour ouvrir le volet **Tâches**.
1. Sélectionnez **Déploiement de modèle ARM**.
1. Saisissez les valeurs suivantes :

    * **deploymentScope** : sélectionnez **Groupe de ressources**. Pour en savoir plus sur les étendues, consultez [Étendues de déploiement](deploy-rest.md#deployment-scope).
    * **Connexion Azure Resource Manager** : sélectionnez le nom de connexion de service que vous avez créé précédemment.
    * **Abonnement**:  spécifiez l’ID de l’abonnement cible.
    * **Action** : sélectionnez l’action **Créer ou mettre à jour un groupe de ressources** qui effectue 2 actions : d’abord, il crée un groupe de ressources si un nouveau nom de groupe de ressources est fourni. Ensuite, il déploie le modèle spécifié.
    * **Groupe de ressources** : Entrez un nouveau nom de groupe de ressources. Par exemple, **AzureRmPipeline-rg**.
    * **Emplacement** : sélectionnez un emplacement pour le groupe de ressources, par exemple **USA Centre**.
    * **Emplacement du modèle** : sélectionnez **Artefact lié**, ce qui signifie que la tâche recherche le fichier de modèle directement à partir du dépôt connecté.
    * **Modèle** : entrez _CreateWebApp/azuredeploy.json_. Si vous avez changé le nom du dossier et le nom du fichier, vous devez modifier cette valeur.
    * **Paramètres du modèle** : laissez ce champ vide. Vous allez spécifier la valeur des paramètres dans **Remplacer les paramètres du modèle**.
    * **Remplacer les paramètres du modèle** : entrez `-projectName [EnterAProjectName] -linkedTemplateUri [EnterTheLinkedTemplateURL]` . Remplacez le nom du projet et l’URL du modèle lié. L’URL du modèle lié correspond à ce que vous avez écrit à la fin de [Création d’un dépôt GitHub](#create-a-github-repository). Elle commence par `https://raw.githubusercontent.com` .
    * **Mode de déploiement** : sélectionnez **Incrémentiel**.
    * **Nom du déploiement** : entrez **DeployPipelineTemplate**. Sélectionnez **Avancé** pour pouvoir voir **Nom du déploiement**.

    ![Capture d’écran montrant la page de déploiement de modèle ARM avec les valeurs obligatoires entrées.](./media/deployment-tutorial-pipeline/resource-manager-template-pipeline-configure.png)

1. Sélectionnez **Ajouter**.

    Pour plus d’informations sur la tâche, consultez [Tâche Déploiement de groupe de ressources Azure](/azure/devops/pipelines/tasks/deploy/azure-resource-group-deployment) et [Tâche Déploiement de modèle Azure Resource Manager](https://github.com/microsoft/azure-pipelines-tasks/blob/master/Tasks/AzureResourceManagerTemplateDeploymentV3/README.md)

    Le fichier _.yml_ doit ressembler à ce qui suit :

    ![Capture d’écran montrant la page de vérification avec le nouveau pipeline intitulé Passer en revue le fichier YAML de pipeline.](./media/deployment-tutorial-pipeline/azure-resource-manager-devops-pipelines-yml.png)

1. Sélectionnez **Enregistrer et exécuter**.
1. Dans le volet **Enregistrer et exécuter**, sélectionnez une nouvelle fois **Enregistrer et exécuter**. Une copie du fichier YAML est enregistrée dans le référentiel connecté. Vous pouvez voir le fichier YAML en accédant à votre référentiel.
1. Vérifiez que le pipeline est exécuté avec succès.

    ![Azure Resource Manager, Azure DevOps, Azure Pipelines, yaml](./media/deployment-tutorial-pipeline/azure-resource-manager-devops-pipelines-status.png)

## <a name="verify-the-deployment"></a>Vérifier le déploiement

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Ouvrez le groupe de ressources. Son nom correspond à ce que vous avez spécifié dans le fichier YAML du pipeline. Vous devriez voir un compte de stockage créé. Le nom du compte de stockage commence par **store** (stocker).
1. Sélectionnez le nom du compte de stockage pour l’ouvrir.
1. Sélectionner **Propriétés**. Notez que le paramètre **Réplication** a pour valeur **Stockage localement redondant (LRS)** .

## <a name="update-and-redeploy"></a>Mettre à jour et redéployer

Lorsque vous mettez à jour le modèle et transmettez les modifications dans le référentiel distant, le pipeline met automatiquement à jour les ressources (ici, il s’agit du compte de stockage).

1. Ouvrez _linkedStorageAccount.json_ à partir de votre dépôt local dans Visual Studio Code ou dans n’importe quel éditeur de texte.
1. Mettez à jour le **defaultValue** de **storageAccountType** sur **Standard_GRS**. Voir la capture d’écran suivante :

    ![Azure Resource Manager, Azure DevOps, Azure Pipelines, mettre à jour yaml](./media/deployment-tutorial-pipeline/azure-resource-manager-devops-pipelines-update-yml.png)

1. Enregistrez les modifications.
1. Transmettez les modifications vers le référentiel distant en exécutant les commandes suivantes à partir de Git Bash/Shell.

    ```bash
    git pull origin main
    git add .
    git commit -m "Update the storage account type."
    git push origin main
    ```

    La première commande (`pull`) synchronise le dépôt local avec le dépôt distant. Le fichier YAML du pipeline a été ajouté uniquement au dépôt distant. L’exécution de la commande `pull` télécharge une copie du fichier YAML sur la branche locale.

    La quatrième commande (`push`) charge le fichier _linkedStorageAccount.json_ révisé sur le dépôt distant. Une fois la branche primaire du dépôt distant mise à jour, le pipeline est redéclenché.

Pour vérifier les modifications, vous pouvez examiner la propriété Réplication du compte de stockage. Veuillez consulter la section [Vérifier le déploiement](#verify-the-deployment).

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, nettoyez les ressources Azure que vous avez déployées en supprimant le groupe de ressources.

1. Dans le portail Azure, sélectionnez **Groupe de ressources** dans le menu de gauche.
2. Entrez le nom du groupe de ressources dans le champ **Filtrer par nom**.
3. Sélectionnez le nom du groupe de ressources.
4. Sélectionnez **Supprimer le groupe de ressources** dans le menu supérieur.

Nous vous recommandons également de supprimer le référentiel GitHub et le projet Azure DevOps.

## <a name="next-steps"></a>Étapes suivantes

Félicitations, vous avez terminé ce tutoriel de déploiement de modèle Resource Manager. Faites-nous part de vos réflexions et de vos suggestions dans la section des commentaires. Merci !
Vous êtes prêt à passer à des concepts plus élaborés sur les modèles. Le tutoriel suivant détaille l’utilisation de la documentation de référence sur les modèles, afin de vous aider à définir les ressources à déployer.

> [!div class="nextstepaction"]
> [Utiliser la référence de modèle](./template-tutorial-use-template-reference.md)
