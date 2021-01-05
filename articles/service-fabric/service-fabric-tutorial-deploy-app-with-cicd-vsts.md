---
title: Déployer une application avec CI et Azure Pipelines
description: Dans ce tutoriel, vous allez découvrir comment configurer l’intégration et le déploiement continus pour une application Service Fabric à l’aide d’Azure Pipelines.
ms.topic: tutorial
ms.date: 07/22/2019
ms.custom: mvc
ms.openlocfilehash: a26cfaca466e01b154c65b27895f3004f6320e5d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91326335"
---
# <a name="tutorial-deploy-an-application-with-cicd-to-a-service-fabric-cluster"></a>Tutoriel : Déployer une application avec intégration et déploiement continus sur un cluster Service Fabric

Quatrième d’une série, ce tutoriel explique comment configurer l’intégration et le déploiement continus d’une application Azure Service Fabric à l’aide d’Azure Pipelines.  Une application Service Fabric existante est requise. L’application créée dans le didacticiel [Générer une application .NET](service-fabric-tutorial-create-dotnet-app.md) est utilisée à titre d’exemple.

Dans ce troisième volet, vous apprenez à :

> [!div class="checklist"]
> * Ajouter le contrôle de code source à votre projet
> * Créer un pipeline de build dans Azure Pipelines
> * Créer un pipeline de mise en production dans Azure Pipelines
> * Déployer et mettre à niveau une application automatiquement

Cette série de tutoriels vous montre comment effectuer les opérations suivantes :
> [!div class="checklist"]
> * [Créer une application .NET Service Fabric](service-fabric-tutorial-create-dotnet-app.md)
> * [Déployer l’application sur un cluster distant](service-fabric-tutorial-deploy-app-to-party-cluster.md)
> * [Ajouter un point de terminaison HTTPS à un service frontal ASP.NET Core](service-fabric-tutorial-dotnet-app-enable-https-endpoint.md)
> * Configurer CI/CD à l’aide d’Azure Pipelines
> * [Configurer la surveillance et les diagnostics pour l’application](service-fabric-tutorial-monitoring-aspnet.md)

## <a name="prerequisites"></a>Prérequis

Avant de commencer ce tutoriel :

* Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
* [Installez Visual Studio 2019](https://www.visualstudio.com/) et les charges de travail **Développement Azure** et **Développement web et ASP.NET**.
* [Installez le Kit de développement logiciel (SDK) Service Fabric](service-fabric-get-started.md)
* Créez un cluster Service Fabric Windows sur Azure, par exemple en suivant [ce tutoriel](service-fabric-tutorial-create-vnet-and-windows-cluster.md)
* Créez une [organisation Azure DevOps](/azure/devops/organizations/accounts/create-organization-msa-or-work-student). Cela vous permet de créer un projet dans Azure DevOps et d’utiliser Azure Pipelines.

## <a name="download-the-voting-sample-application"></a>Télécharger l’exemple d’application de vote

Si vous n’avez pas généré l’exemple d’application de vote lors de la [première partie de cette série de didacticiels](service-fabric-tutorial-create-dotnet-app.md), vous pouvez le télécharger. Dans une fenêtre Commande, exécutez la commande suivante pour cloner le référentiel de l’exemple d’application sur votre ordinateur local.

```git
git clone https://github.com/Azure-Samples/service-fabric-dotnet-quickstart
```

## <a name="prepare-a-publish-profile"></a>Préparer un profil de publication

Maintenant que vous avez [créé une application](service-fabric-tutorial-create-dotnet-app.md) et [déployé l’application dans Azure](service-fabric-tutorial-deploy-app-to-party-cluster.md), vous êtes prêt à configurer l’intégration continue.  Tout d’abord, préparez au sein de votre application un profil de publication destiné au processus de déploiement qui s’exécute dans Azure Pipelines.  Le profil de publication doit être configuré pour cibler le cluster que vous avez précédemment créé.  Démarrez Visual Studio et ouvrez un projet d’application Service Fabric existant.  Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur l’application, puis sélectionnez **Publier...** .

Choisissez au sein de votre projet d’application un profil cible à utiliser pour votre flux de travail d’intégration continue (Cloud, par exemple).  Spécifiez le point de terminaison de connexion du cluster.  Cochez la case **Mettre à niveau l’application** pour que votre application soit mise à niveau pour chaque déploiement dans Azure DevOps.  Cliquez sur le lien hypertexte **Enregistrer** pour enregistrer les paramètres dans le profil de publication, puis cliquez sur **Annuler** pour fermer la boîte de dialogue.

![Profil d’envoi (push)][publish-app-profile]

## <a name="share-your-visual-studio-solution-to-a-new-azure-devops-git-repo"></a>Partager votre solution Visual Studio dans un nouveau référentiel Git Azure DevOps

Partagez les fichiers source de votre application dans un projet Azure DevOps pour pouvoir générer des builds.

Créez un dépôt Git local pour votre projet en sélectionnant **Ajouter au contrôle de code source** -> **Git** dans la barre d’état située dans le coin inférieur droit de Visual Studio.

Dans la vue **Envoi (push)** de **Team Explorer**, sélectionnez le bouton **Publier le référentiel Git** sous **Push sur Azure DevOps**.

![Capture d’écran de la fenêtre de synchronisation de Team Explorer dans Visual Studio. Le bouton Publier le dépôt Git est mis en surbrillance sous Transmettre à Azure DevOps.][push-git-repo]

Vérifiez votre adresse e-mail et sélectionnez votre compte dans la liste déroulante **Domaine Azure DevOps**. Entrez le nom de votre dépôt et sélectionnez **Publier le dépôt**.

![Capture d’écran des paramètres d’envoi (push) vers Azure DevOps avec le bouton E-mail, Compte, Nom du référentiel et Publier le dépôt mis en surbrillance.][publish-code]

La publication du référentiel entraîne la création d’un projet portant le même nom que le référentiel local dans votre compte. Pour créer le référentiel dans un projet existant, cliquez sur **Avancé** en regard de Nom du **référentiel**, puis sélectionnez un projet. Vous pouvez afficher votre code sur le web en sélectionnant **See it on the web** (Visualiser sur le web).

## <a name="configure-continuous-delivery-with-azure-pipelines"></a>Configurer la livraison continue avec Azure Pipelines

Un pipeline de build Azure Pipelines décrit un flux de travail qui se compose d’un ensemble d’étapes de génération exécutées séquentiellement. Créez un pipeline de build qui produit un package d’application Service Fabric et les autres artefacts à déployer sur un cluster Service Fabric. En savoir plus sur [les pipelines de build Azure Pipelines](https://www.visualstudio.com/docs/build/define/create). 

Un pipeline de mise en production Azure Pipelines décrit un flux de travail qui déploie un package d’application sur un cluster. Lorsqu’ils sont utilisés ensemble, le pipeline de build et le pipeline de mise en production exécutent le flux de travail dans son ensemble, depuis les fichiers source jusqu’à l’exécution de l’application dans votre cluster. Apprenez-en davantage sur les [pipelines de mise en production Azure Pipelines](https://www.visualstudio.com/docs/release/author-release-definition/more-release-definition).

### <a name="create-a-build-pipeline"></a>Créer un pipeline de build

Ouvrez un navigateur web et accédez à votre nouveau projet à l’adresse : [https://&lt;moncompte&gt;.visualstudio.com/Voting/Voting%20Team/_git/Voting](https://myaccount.visualstudio.com/Voting/Voting%20Team/_git/Voting).

Sélectionnez l’onglet **Pipelines**, puis **Builds**, puis cliquez sur **Nouveau pipeline**.

![Nouveau pipeline][new-pipeline]

Sélectionnez la source **Azure Repos Git**, le projet d’équipe **Voting**, le dépôt **Voting** et la branche par défaut **maître** pour les builds manuelles et planifiées.  Puis, cliquez sur **Continuer**.

![Sélectionner un dépôt][select-repo]

Dans **Sélectionner un modèle**, sélectionnez le modèle **Application Azure Service Fabric**, puis cliquez sur **Appliquer**.

![Choisir le modèle de build][select-build-template]

Dans **Tâches**, entrez « VS 2017 hébergé » comme **Pool d’agents**.

![Sélectionner les tâches][save-and-queue]

Sous **Déclencheurs**, activez l’intégration continue en cochant la case **Activer l’intégration continue**. Dans la zone **Filtres de branche**, la **Spécification de branche** est définie par défaut sur la valeur **maître**. Sélectionnez **Enregistrer et mettre en file d’attente** pour lancer une build manuellement.

![Sélectionnez les déclencheurs][save-and-queue2]

Les builds sont également déclenchées par envoi (push) ou par archivage. Pour vérifier la progression de votre build, basculez vers l’onglet **Builds**.  Une fois que vous avez vérifié que la build s’exécute correctement, définissez un pipeline de mise en production assurant le déploiement de votre application sur un cluster.

### <a name="create-a-release-pipeline"></a>Créer un pipeline de mise en production

Sélectionnez l’onglet **Pipelines**, puis **Versions** et **+ Nouveau pipeline**.  Sous **Sélectionner un modèle**, sélectionnez le modèle **Déploiement Azure Service Fabric** dans la liste, puis **Appliquer**.

![Choisissez le modèle de template][select-release-template]

Sélectionnez **Tâches**->**Environnement 1**, puis **+ Nouveau** pour ajouter une nouvelle connexion de cluster.

![Ajouter une connexion au cluster][add-cluster-connection]

Sur la vue **Ajouter une nouvelle connexion Service Fabric**, sélectionnez l’authentification **Basée sur les certificats** ou **Azure Active Directory**.  Spécifiez le nom de connexion « mysftestcluster » et le point de terminaison de cluster « tcp://mysftestcluster.southcentralus.cloudapp.azure.com:19000 » (ou celui du cluster sur lequel porte le déploiement).

Pour l’authentification basée sur les certificats, ajoutez **l’empreinte du certificat de serveur** utilisé pour créer le cluster.  Dans **Certificat client**, ajoutez le codage en base 64 du fichier de certificat client. Consultez la fenêtre contextuelle d’aide de ce champ pour savoir comment obtenir cette représentation codée en base 64 du certificat. Ajoutez également le **Mot de passe** du certificat.  Vous pouvez utiliser le certificat de serveur ou de cluster si vous ne disposez pas d’un certificat client distinct.

Dans les informations d’identification d’Azure Active Directory, ajoutez **l’empreinte du certificat de serveur** utilisé pour créer le cluster et les informations d’identification à utiliser pour se connecter au cluster dans les champs **Nom d’utilisateur** et **Mot de passe**.

Cliquez sur **Ajouter** pour enregistrer la connexion du cluster.

Ensuite, ajoutez un artefact de build au pipeline afin de permettre au pipeline de mise en production de trouver la sortie de la build. Sélectionnez **Pipeline** et **Artefacts**->**+ Ajouter**.  Dans **Source (définition de build)**, sélectionnez le pipeline de build que vous avez créé précédemment.  Cliquez sur **Ajouter** pour enregistrer l’artefact de la build.

![Ajoutez un artefact][add-artifact]

Activez un déclencheur de déploiement continu afin qu’une version soit créée automatiquement à la fin de la génération. Cliquez sur l’icône en forme d’éclair de l’artefact, activez le déclencheur, puis cliquez sur **Enregistrer** pour enregistrer le pipeline de mise en production.

![Activez le déclencheur][enable-trigger]

Sélectionnez **+ Mise en production** -> **Créer une mise en production** -> **Créer** pour créer manuellement une mise en production. Vous pouvez surveiller la progression de la mise en production dans l’onglet **Mises en production**.

Vérifiez que le déploiement a réussi et que l’application est en cours d’exécution dans le cluster.  Ouvrez un navigateur web et accédez à `http://mysftestcluster.southcentralus.cloudapp.azure.com:19080/Explorer/`.  Notez la version de l’application. Dans cet exemple, il s’agit de la version 1.0.0.20170616.3.

## <a name="commit-and-push-changes-trigger-a-release"></a>Valider et envoyer les modifications, déclencher une mise en production

Vérifiez que le pipeline d’intégration continue fonctionne en archivant certaines modifications de code dans Azure DevOps.

Lorsque vous écrivez votre code, vos modifications sont suivies automatiquement par Visual Studio. Validez les modifications dans votre dépôt Git local en sélectionnant l’icône de modifications en attente (![L’icône des modifications en attente affiche un crayon et un chiffre.][pending]) dans la barre d’état en bas à droite.

Dans la vue **Modifications** de Team Explorer, ajoutez un message décrivant votre mise à jour, puis validez vos modifications.

![Valider tout][changes]

Sélectionnez l’icône de modifications non publiées (![Modifications non publiées][unpublished-changes]) dans la barre d’état ou la vue Synchronisation de Team Explorer. Sélectionnez **Envoi (push)** pour mettre à jour votre code dans Azure Pipelines.

![Envoi (push) des modifications][push]

L’envoi (push) des modifications à Azure Pipelines déclenche automatiquement une build.  Une fois le pipeline de build terminé, une mise en production est créée automatiquement et commence la mise à niveau de l’application sur le cluster.

Pour vérifier la progression de votre build, basculez vers l’onglet **Builds** de **Team Explorer** dans Visual Studio.  Une fois que vous avez vérifié que la build s’exécute correctement, définissez un pipeline de mise en production assurant le déploiement de votre application sur un cluster.

Vérifiez que le déploiement a réussi et que l’application est en cours d’exécution dans le cluster.  Ouvrez un navigateur web et accédez à `http://mysftestcluster.southcentralus.cloudapp.azure.com:19080/Explorer/`.  Notez la version de l’application. Dans cet exemple, il s’agit de la version 1.0.0.20170815.3.

![Capture d’écran de l’application de vote dans Service Fabric Explorer en cours d’exécution dans une fenêtre de navigateur. La version de l’application « 1.0.0.20170815.3 » est mise en surbrillance.][sfx1]

## <a name="update-the-application"></a>Mettre à jour l’application

Apportez des modifications au code de l’application.  Enregistrez et validez les modifications en suivant les étapes décrites précédemment.

Une fois que la mise à niveau de l’application a commencé, vous pouvez suivre la progression dans Service Fabric Explorer :

![Capture d’écran de l’application de vote dans Service Fabric Explorer. Le message d’état « Mise à niveau » et le message « Mise à niveau en cours » sont mis en surbrillance.][sfx2]

La mise à niveau de l’application peut prendre plusieurs minutes. Une fois la mise à niveau terminée, l’application exécute la version suivante.  Dans cet exemple, 1.0.0.20170815.4.

![Capture d’écran de l’application de vote dans Service Fabric Explorer en cours d’exécution dans une fenêtre de navigateur. La version de l’application mise à jour « 1.0.0.20170815.4 » est mise en surbrillance.][sfx3]

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Ajouter le contrôle de code source à votre projet
> * Créer un pipeline de build
> * Créer un pipeline de mise en production
> * Déployer et mettre à niveau une application automatiquement

Passez au tutoriel suivant :
> [!div class="nextstepaction"]
> [Configurer la surveillance et les diagnostics pour l’application](service-fabric-tutorial-monitoring-aspnet.md)

<!-- Image References -->
[publish-app-profile]: ./media/service-fabric-tutorial-deploy-app-with-cicd-vsts/PublishAppProfile.png
[push-git-repo]: ./media/service-fabric-tutorial-deploy-app-with-cicd-vsts/PublishGitRepo.png
[publish-code]: ./media/service-fabric-tutorial-deploy-app-with-cicd-vsts/PublishCode.png
[new-pipeline]: ./media/service-fabric-tutorial-deploy-app-with-cicd-vsts/NewPipeline.png
[select-repo]: ./media/service-fabric-tutorial-deploy-app-with-cicd-vsts/SelectRepo.png
[select-build-template]: ./media/service-fabric-tutorial-deploy-app-with-cicd-vsts/SelectBuildTemplate.png
[save-and-queue]: ./media/service-fabric-tutorial-deploy-app-with-cicd-vsts/SaveAndQueue.png
[save-and-queue2]: ./media/service-fabric-tutorial-deploy-app-with-cicd-vsts/SaveAndQueue2.png
[select-release-template]: ./media/service-fabric-tutorial-deploy-app-with-cicd-vsts/SelectReleaseTemplate.png
[set-continuous-integration]: ./media/service-fabric-tutorial-deploy-app-with-cicd-vsts/SetContinuousIntegration.png
[add-cluster-connection]: ./media/service-fabric-tutorial-deploy-app-with-cicd-vsts/AddClusterConnection.png
[add-artifact]: ./media/service-fabric-tutorial-deploy-app-with-cicd-vsts/AddArtifact.png
[enable-trigger]: ./media/service-fabric-tutorial-deploy-app-with-cicd-vsts/EnableTrigger.png
[sfx1]: ./media/service-fabric-tutorial-deploy-app-with-cicd-vsts/SFX1.png
[sfx2]: ./media/service-fabric-tutorial-deploy-app-with-cicd-vsts/SFX2.png
[sfx3]: ./media/service-fabric-tutorial-deploy-app-with-cicd-vsts/SFX3.png
[pending]: ./media/service-fabric-tutorial-deploy-app-with-cicd-vsts/Pending.png
[changes]: ./media/service-fabric-tutorial-deploy-app-with-cicd-vsts/Changes.png
[unpublished-changes]: ./media/service-fabric-tutorial-deploy-app-with-cicd-vsts/UnpublishedChanges.png
[push]: ./media/service-fabric-tutorial-deploy-app-with-cicd-vsts/Push.png
[continuous-delivery-with-AzureDevOpsServices]: ./media/service-fabric-tutorial-deploy-app-with-cicd-vsts/VSTS-Dialog.png
[new-service-endpoint]: ./media/service-fabric-tutorial-deploy-app-with-cicd-vsts/NewServiceEndpoint.png
[new-service-endpoint-dialog]: ./media/service-fabric-tutorial-deploy-app-with-cicd-vsts/NewServiceEndpointDialog.png
