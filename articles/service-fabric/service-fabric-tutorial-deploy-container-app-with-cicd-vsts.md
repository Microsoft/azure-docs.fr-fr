---
title: Déployer une application conteneur avec CI/CD
description: Dans ce tutoriel, vous allez apprendre à configurer l’intégration et le déploiement continus pour une application conteneur Azure Service Fabric à l’aide de Visual Studio et d’Azure DevOps.
ms.topic: tutorial
ms.date: 08/29/2018
ms.custom: mvc
ms.openlocfilehash: bb0eb9226a99f139ff10a8da12a1e22017536c67
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96018849"
---
# <a name="tutorial-deploy-a-container-application-with-cicd-to-a-service-fabric-cluster"></a>Tutoriel : Déployer une application conteneur avec intégration et déploiement continus sur un cluster Service Fabric

Deuxième d’une série, ce tutoriel explique comment configurer l’intégration et le déploiement continus d’une application conteneur Azure Service Fabric à l’aide de Visual Studio et d’Azure DevOps.  Une application Service Fabric existante est requise. L’application créée dans [Déployer une application .NET dans un conteneur Windows sur Azure Service Fabric](service-fabric-host-app-in-a-container.md) est utilisée à titre d’exemple.

Dans ce deuxième volet, vous apprenez à :

> [!div class="checklist"]
> * Ajouter le contrôle de code source à votre projet
> * Créer une définition de build dans Visual Studio Team Explorer
> * Créer une définition de mise en production dans Visual Studio Team Explorer
> * Déployer et mettre à niveau une application automatiquement

## <a name="prerequisites"></a>Prérequis

Avant de commencer ce tutoriel :

* disposer d’un cluster sur Azure, ou en [créer un avec ce didacticiel](service-fabric-tutorial-create-vnet-and-windows-cluster.md) ;
* [y déployer une application en conteneur](service-fabric-host-app-in-a-container.md).

## <a name="prepare-a-publish-profile"></a>Préparer un profil de publication

Maintenant que vous avez [déployé une application conteneur](service-fabric-host-app-in-a-container.md), vous pouvez configurer l’intégration continue.  Tout d’abord, préparez au sein de votre application un profil de publication destiné au processus de déploiement qui s’exécute dans Azure DevOps.  Le profil de publication doit être configuré pour cibler le cluster que vous avez précédemment créé.  Démarrez Visual Studio et ouvrez un projet d’application Service Fabric existant.  Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur l’application, puis sélectionnez **Publier...** .

Choisissez au sein de votre projet d’application un profil cible à utiliser pour votre flux de travail d’intégration continue (Cloud, par exemple).  Spécifiez le point de terminaison de connexion du cluster.  Cochez la case **Mettre à niveau l’application** pour que votre application soit mise à niveau pour chaque déploiement dans Azure DevOps.  Cliquez sur le lien hypertexte **Enregistrer** pour enregistrer les paramètres dans le profil de publication, puis cliquez sur **Annuler** pour fermer la boîte de dialogue.

![Profil d’envoi (push)][publish-app-profile]

## <a name="share-your-visual-studio-solution-to-a-new-azure-devops-git-repo"></a>Partager votre solution Visual Studio dans un nouveau référentiel Git Azure DevOps

Partagez les fichiers sources de votre application dans un projet d’équipe Azure DevOps afin de pouvoir générer des builds.

Créez un dépôt Git local pour votre projet en sélectionnant **Ajouter au contrôle de code source** -> **Git** dans la barre d’état située dans le coin inférieur droit de Visual Studio.

Dans la vue **Envoi (push)** de **Team Explorer**, sélectionnez le bouton **Publier le référentiel Git** sous **Push sur Azure DevOps**.

![Capture d’écran de la fenêtre de synchronisation de Team Explorer dans Visual Studio. Sous Transmettre à Azure DevOps, le bouton Publier le dépôt Git est mis en surbrillance.][push-git-repo]

Vérifiez votre adresse e-mail et sélectionnez votre organisation dans la liste déroulante **Compte**. Vous aurez peut-être à configurer une organisation si vous n’en avez pas déjà une. Entrez le nom de votre dépôt et sélectionnez **Publier le dépôt**.

![Capture d’écran de la fenêtre Push to Azure DevOps (Transmettre à Azure DevOps). Les paramètres d’e-mail, de compte, de nom de référentiel et le bouton de publication du référentiel sont mis en surbrillance.][publish-code]

La publication du référentiel entraîne la création d’un projet d’équipe portant le même nom que le référentiel local dans votre compte. Pour créer le référentiel dans un projet d’équipe existant, cliquez sur **Avancé** à côté du **Nom du référentiel** et sélectionnez un projet d’équipe. Vous pouvez afficher votre code sur le web en sélectionnant **See it on the web** (Visualiser sur le web).

## <a name="configure-continuous-delivery-with-azure-pipelines"></a>Configurer la livraison continue avec Azure Pipelines

Une définition de build Azure DevOps décrit un workflow qui se compose d’un ensemble d’étapes de génération exécutées séquentiellement. Créez une définition de build qui produit un package d’application Service Fabric et les autres artefacts à déployer sur un cluster Service Fabric. En savoir plus sur les [définitions de build](https://www.visualstudio.com/docs/build/define/create) Azure DevOps 

Une définition de mise en production Azure DevOps décrit un workflow qui déploie un package d’application sur un cluster. Lorsqu’elles sont utilisées ensemble, la définition de version et la définition de génération exécutent le flux de travail dans son ensemble, depuis le déploiement des fichiers source jusqu’à l’exécution de l’application dans votre cluster. En savoir plus sur les [définitions de mise en production](https://www.visualstudio.com/docs/release/author-release-definition/more-release-definition) Azure DevOps

### <a name="create-a-build-definition"></a>Créer une définition de build

Pour ouvrir votre nouveau projet d’équipe, accédez à https://dev.azure.com dans un navigateur web, puis sélectionnez votre organisation et votre nouveau projet. 

Sélectionnez l’option **Pipelines** dans le volet gauche, puis cliquez sur **Nouveau pipeline**.

>[!NOTE]
>Si vous ne voyez pas le modèle de définition de build, assurez-vous que la fonctionnalité **Nouvelle expérience de création de pipeline YAML** soit désactivée. Cette fonctionnalité est configurée dans la section **Fonctionnalités préliminaires** de votre compte de DevOps.

![Nouveau pipeline][new-pipeline]

Sélectionnez la source **Azure Repos Git**, votre projet d’équipe, le dépôt de votre projet, puis la branche par défaut **master** pour les builds manuelles et planifiées.  Puis, cliquez sur **Continuer**.

Dans **Sélectionner un modèle**, sélectionnez le modèle **Application Azure Service Fabric avec prise en charge Docker** et cliquez sur **Appliquer**.

![Choisir le modèle de build][select-build-template]

Dans **Tâches**, sélectionnez **Hébergée VS2017** pour **Pool d’agents**.

![Sélectionner les tâches][task-agent-pool]

Cliquez sur **Images Tag**.

Dans **Type de registre de conteneurs**, sélectionnez **Azure Container Registry**. Sélectionnez un **Abonnement Azure**, puis cliquez sur **Autoriser**. Sélectionnez un service **Azure Container Registry**.

![Sélectionner des images Docker Tag][select-tag-images]

Cliquez sur **Images Push**.

Dans **Type de registre de conteneurs**, sélectionnez **Azure Container Registry**. Sélectionnez un **Abonnement Azure**, puis cliquez sur **Autoriser**. Sélectionnez un service **Azure Container Registry**.

![Sélectionner des images Docker Push][select-push-images]

Sous l’onglet **Déclencheurs**, activez l’intégration continue en cochant la case **Activer l’intégration continue**. Dans la zone **Filtres de branche**, cliquez sur **+ Ajouter**, et la zone **Spécification de branche** est alors définie par défaut sur la valeur **maître**.

Dans la boîte de dialogue **Enregistrer le pipeline de build et mettre en file d’attente**, cliquez sur **Enregistrer et mettre en file d’attente** pour lancer un build manuellement.

![Sélectionnez les déclencheurs][save-and-queue]

Les builds sont également déclenchées par envoi (push) ou par archivage. Pour vérifier la progression de votre build, basculez vers l’onglet **Builds**.  Une fois que vous avez vérifié que la build s’exécute correctement, spécifiez une définition de mise en production assurant le déploiement de votre application sur un cluster.

### <a name="create-a-release-definition"></a>Création d’une définition de version

Sélectionnez l’option **Pipelines** dans le volet gauche, cliquez sur **Versions**, puis sur **+Nouveau pipeline**.  Sous **Sélectionner un modèle**, sélectionnez le modèle **Déploiement Azure Service Fabric** dans la liste, puis **Appliquer**.

![Choisissez le modèle de template][select-release-template]

Sélectionnez **Tâches**, puis **Environnement 1** et **+ Nouveau** pour ajouter une nouvelle connexion de cluster.

![Ajouter une connexion au cluster][add-cluster-connection]

Sur la vue **Ajouter une nouvelle connexion Service Fabric**, sélectionnez l’authentification **Basée sur les certificats** ou **Azure Active Directory**.  Spécifiez le nom de connexion « mysftestcluster » et le point de terminaison de cluster « tcp://mysftestcluster.southcentralus.cloudapp.azure.com:19000 » (ou celui du cluster sur lequel porte le déploiement).

Pour l’authentification basée sur les certificats, ajoutez **l’empreinte du certificat de serveur** utilisé pour créer le cluster.  Dans **Certificat client**, ajoutez le codage en base 64 du fichier de certificat client. Consultez la fenêtre contextuelle d’aide de ce champ pour savoir comment obtenir cette représentation codée en base 64 du certificat. Ajoutez également le **Mot de passe** du certificat.  Vous pouvez utiliser le certificat de serveur ou de cluster si vous ne disposez pas d’un certificat client distinct.

Dans les informations d’identification d’Azure Active Directory, ajoutez **l’empreinte du certificat de serveur** utilisé pour créer le cluster et les informations d’identification à utiliser pour se connecter au cluster dans les champs **Nom d’utilisateur** et **Mot de passe**.

Cliquez sur **Ajouter** pour enregistrer la connexion du cluster.



Sous Phase de l’agent, cliquez sur **Déployer une application Service Fabric**.
Cliquez sur **Paramètres Docker**, puis sur **Configurer les paramètres Docker**. Dans **Source d’informations d’identification du Registre**, sélectionnez **Connexion au service Azure Resource Manager**. Ensuite, sélectionnez votre **abonnement Azure**.

![Agent du pipeline de mise en production][release-pipeline-agent]

Ensuite, ajoutez un artefact de build au pipeline afin de permettre à la définition de mise en production de trouver la sortie de la build. Sélectionnez **Pipeline** et **Artefacts**->**+ Ajouter**.  Dans **Source (définition de build)**, sélectionnez la définition de build que vous avez créée précédemment.  Cliquez sur **Ajouter** pour enregistrer l’artefact de la build.

![Ajoutez un artefact][add-artifact]

Activez un déclencheur de déploiement continu afin qu’une version soit créée automatiquement à la fin de la génération. Cliquez sur l’icône en forme d’éclair de l’artefact, activez le déclencheur, puis cliquez sur **Enregistrer** pour enregistrer la définition de mise en production.

![Activez le déclencheur][enable-trigger]

Sélectionnez **+ Mise en production** -> **Créer une mise en production** -> **Créer** pour créer manuellement une mise en production. Vous pouvez surveiller la progression de la mise en production dans l’onglet **Mises en production**.

Vérifiez que le déploiement a réussi et que l’application est en cours d’exécution dans le cluster.  Ouvrez un navigateur web et accédez à `http://mysftestcluster.southcentralus.cloudapp.azure.com:19080/Explorer/`.  Notez la version de l’application. Dans cet exemple, il s’agit de la version 1.0.0.20170616.3.

## <a name="commit-and-push-changes-trigger-a-release"></a>Valider et envoyer les modifications, déclencher une mise en production

Vérifiez que le pipeline d’intégration continue fonctionne en archivant certaines modifications de code dans Azure DevOps.

Lorsque vous écrivez votre code, vos modifications sont suivies automatiquement par Visual Studio. Validez les modifications dans votre dépôt Git local en sélectionnant l’icône de modifications en attente (![L’icône des modifications en attente affiche un crayon et un chiffre.][pending]) dans la barre d’état en bas à droite.

Dans la vue **Modifications** de Team Explorer, ajoutez un message décrivant votre mise à jour, puis validez vos modifications.

![Valider tout][changes]

Sélectionnez l’icône de modifications non publiées (![Modifications non publiées][unpublished-changes]) dans la barre d’état ou la vue Synchronisation de Team Explorer. Sélectionnez **Envoi (push)** pour mettre à jour votre code dans Azure DevOps.

![Envoi (push) des modifications][push]

L’envoi (push) des modifications à Azure DevOps déclenche automatiquement une build.  Une fois la définition de build terminée, une mise en production est créée automatiquement et commence la mise à niveau de l’application sur le cluster.

Pour vérifier la progression de votre build, basculez vers l’onglet **Builds** de **Team Explorer** dans Visual Studio.  Une fois que vous avez vérifié que la build s’exécute correctement, spécifiez une définition de mise en production assurant le déploiement de votre application sur un cluster.

Vérifiez que le déploiement a réussi et que l’application est en cours d’exécution dans le cluster.  Ouvrez un navigateur web et accédez à `http://mysftestcluster.southcentralus.cloudapp.azure.com:19080/Explorer/`.  Notez la version de l’application. Dans cet exemple, il s’agit de la version 1.0.0.20170815.3.

![Capture d’écran de l’application de vote dans Service Fabric Explorer. Dans l’onglet Essentials, la version de l’application « 1.0.0.20170815.3 » est mise en surbrillance.][sfx1]

## <a name="update-the-application"></a>Mettre à jour l’application

Apportez des modifications au code de l’application.  Enregistrez et validez les modifications en suivant les étapes décrites précédemment.

Une fois que la mise à niveau de l’application a commencé, vous pouvez suivre la progression dans Service Fabric Explorer :

![Capture d’écran de l’application de vote dans Service Fabric Explorer. Un message « Mise à niveau en cours » est mis en surbrillance et l’état de l’application est « Mise à niveau ».][sfx2]

La mise à niveau de l’application peut prendre plusieurs minutes. Une fois la mise à niveau terminée, l’application exécute la version suivante.  Dans cet exemple, 1.0.0.20170815.4.

![Capture d’écran de l’application de vote dans Service Fabric Explorer. Dans l’onglet Essentials, la version de l’application mise à jour « 1.0.0.20170815.4 » est mise en surbrillance.][sfx3]

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Ajouter le contrôle de code source à votre projet
> * Créer une définition de build
> * Création d’une définition de version
> * Déployer et mettre à niveau une application automatiquement

Dans la partie suivante du didacticiel, vous apprendrez à configurer [la surveillance pour votre conteneur](service-fabric-tutorial-monitoring-wincontainers.md).

<!-- Image References -->
[publish-app-profile]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/PublishAppProfile.png
[push-git-repo]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/PublishGitRepo.png
[publish-code]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/PublishCode.png
[new-pipeline]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/NewPipeline.png
[select-build-template]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/SelectBuildTemplate.png
[task-agent-pool]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/TaskAgentPool.png
[save-and-queue]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/SaveAndQueue.png
[select-tag-images]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/DockerTagImages.png
[select-push-images]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/DockerPushImages.png
[select-release-template]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/SelectReleaseTemplate.png
[set-continuous-integration]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/SetContinuousIntegration.png
[add-cluster-connection]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/AddClusterConnection.png
[release-pipeline-agent]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/ReleasePipelineAgent.png
[add-artifact]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/AddArtifact.png
[enable-trigger]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/EnableTrigger.png
[sfx1]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/SFX1.png
[sfx2]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/SFX2.png
[sfx3]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/SFX3.png
[pending]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/Pending.png
[changes]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/Changes.png
[unpublished-changes]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/UnpublishedChanges.png
[push]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/Push.png
[continuous-delivery-with-VSTS]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/VSTS-Dialog.png
[new-service-endpoint]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/NewServiceEndpoint.png
[new-service-endpoint-dialog]: ./media/service-fabric-tutorial-deploy-container-app-with-cicd-vsts/NewServiceEndpointDialog.png
