---
title: Configurer le déploiement continu
description: Découvrez comment activer CI/CD vers Azure App Service à partir de GitHub, BitBucket, Azure Repos ou d’autres dépôts. Sélectionnez le pipeline de build qui répond à vos besoins.
ms.assetid: 6adb5c84-6cf3-424e-a336-c554f23b4000
ms.topic: article
ms.date: 03/20/2020
ms.reviewer: dariac
ms.custom: seodec18
ms.openlocfilehash: b7730558e2a660b0cf00a5b6962d1e2275dd472c
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96004339"
---
# <a name="continuous-deployment-to-azure-app-service"></a>Déploiement continu vers Azure App Service

[Azure App Service](overview.md) permet un déploiement continu à partir des référentiels GitHub, BitBucket et [Azure Repos](https://azure.microsoft.com/services/devops/repos/) en extrayant les dernières mises à jour. Cet article explique comment utiliser le Portail Azure pour procéder à un déploiement continu de votre application via le service de builds Kudu ou [Azure Pipelines](https://azure.microsoft.com/services/devops/pipelines/). 

Pour plus d'informations sur les services de contrôle de code source, consultez [Création d’un dépôt (GitHub)], [Création d’un dépôt (BitBucket)] ou [Créer un référentiel Git (Azure Repos)].

[!INCLUDE [Prepare repository](../../includes/app-service-deploy-prepare-repo.md)]

## <a name="authorize-azure-app-service"></a>Autoriser Azure App Service 

Pour utiliser Azure Repos, assurez-vous que votre organisation Azure DevOps est liée à votre abonnement Azure. Pour plus d'informations, consultez [Configurer un compte Azure DevOps Services afin de le déployer sur une application web](/azure/devops/pipelines/apps/cd/deploy-webdeploy-webapps?view=azure-devops&preserve-view=true).

Pour Bitbucket ou GitHub, autorisez Azure App Service à se connecter à votre référentiel. Un service de contrôle de source ne nécessite qu'une seule autorisation. 

1. Sur le [portail Azure](https://portal.azure.com), recherchez et sélectionnez **App Services**.

   ![Recherchez App Services.](media/app-service-continuous-deployment/search-for-app-services.png)

1. Sélectionnez le service App Service que vous souhaitez déployer.

   ![Sélectionnez votre application.](media/app-service-continuous-deployment/select-your-app.png)
   
1. Sur la page de l'application, sélectionnez **Centre de déploiement** dans le menu de gauche.
   
1. Sur la page **Centre de déploiement**, choisissez **GitHub** ou **Bitbucket**, puis sélectionnez **Autoriser**. 
   
   ![Sélectionnez le service de contrôle de code source, puis choisissez Autoriser.](media/app-service-continuous-deployment/github-choose-source.png)
   
1. Si nécessaire, connectez-vous au service et suivez les instructions d'autorisation. 

## <a name="enable-continuous-deployment"></a>Activer le déploiement continu 

Après avoir autorisé un service de contrôle de code source, configurez votre application pour un déploiement continu via le serveur de builds [Kudu App Service](#option-1-kudu-app-service) intégré ou via [Azure Pipelines](#option-2-azure-pipelines). 

### <a name="option-1-kudu-app-service"></a>Option 1 : Kudu App Service

Vous pouvez utiliser le serveur de builds Kudu App Service intégré pour procéder à des déploiements continus à partir de GitHub, Bitbucket ou Azure Repos. 

1. Sur le [portail Azure](https://portal.azure.com), recherchez **App Services**, puis sélectionnez le service App Service que vous souhaitez déployer. 
   
1. Sur la page de l'application, sélectionnez **Centre de déploiement** dans le menu de gauche.
   
1. Sélectionnez votre fournisseur de contrôle de code source autorisé sur la page **Centre de déploiement**, puis choisissez **Continuer**. Pour GitHub ou Bitbucket, vous pouvez également sélectionner **Modifier le compte** pour changer de compte autorisé. 
   
   > [!NOTE]
   > Pour utiliser Azure Repos, assurez-vous que votre organisation Azure DevOps Services est liée à votre abonnement Azure. Pour plus d'informations, consultez [Configurer un compte Azure DevOps Services afin de le déployer sur une application web](/azure/devops/pipelines/apps/cd/deploy-webdeploy-webapps?view=azure-devops&preserve-view=true).
   
1. Pour GitHub ou Azure Repos, sur la page **Fournisseur de builds**, sélectionnez **Service de builds App Service**, puis **Continuer**. Bitbucket utilise toujours le service de builds App Service.
   
   ![Sélectionnez Service de builds App Service, puis Continuer.](media/app-service-continuous-deployment/choose-kudu.png)
   
1. Sur la page **Configurer** :
   
   - Pour GitHub, sélectionnez l'**Organisation**, le **Référentiel** et la **Branche** que vous souhaitez déployer en continu.
     
     > [!NOTE]
     > En l'absence de référentiel, il peut être nécessaire d'autoriser Azure App Service dans GitHub. Accédez à votre référentiel GitHub, puis sélectionnez **Paramètres** > **Applications** > **Applications OAuth autorisées**. Sélectionnez **Azure App Service**, puis **Octroyer**. Pour les dépôts de l’organisation, vous devez être propriétaire de l’organisation pour accorder des autorisations.
     
   - Pour Bitbucket, sélectionnez l'**Équipe**, le **Référentiel** et la **Branche** que vous souhaitez déployer en continu.
     
   - Pour Azure Repos, sélectionnez l'**Organisation Azure DevOps**, le **Projet**, le **Référentiel** et la **Branche** que vous souhaitez déployer en continu.
     
     > [!NOTE]
     > Si votre organisation Azure DevOps n'est pas répertoriée, assurez-vous qu'elle est liée à votre abonnement Azure. Pour plus d'informations, consultez [Configurer un compte Azure DevOps Services afin de le déployer sur une application web](/azure/devops/pipelines/apps/cd/deploy-webdeploy-webapps?view=azure-devops&preserve-view=true).
     
1. Sélectionnez **Continuer**.
   
   ![Renseignez les informations relatives au référentiel, puis sélectionnez Continuer.](media/app-service-continuous-deployment/configure-kudu.png)
   
1. Une fois le fournisseur de builds configuré, vérifiez les paramètres sur la page **Résumé**, puis sélectionnez **Terminer**.
   
1. Les nouvelles validations correspondant au référentiel et à la branche sélectionnés sont déployées en continu dans votre application App Service. Vous pouvez suivre les validations et les déploiements sur la page **Centre de déploiement**.
   
   ![Suivre les validations et les déploiements dans le Centre de déploiement](media/app-service-continuous-deployment/github-finished.png)

### <a name="option-2-azure-pipelines"></a>Option n°2 : Azure Pipelines 

Si votre compte dispose des autorisations nécessaires, vous pouvez configurer Azure Pipelines pour un déploiement continu à partir de GitHub ou Azure Repos. Pour plus d'informations sur le déploiement via Azure Pipelines, consultez [Déployer une application web dans Azure App Services](/azure/devops/pipelines/apps/cd/deploy-webdeploy-webapps).

#### <a name="prerequisites"></a>Prérequis

Pour permettre à Azure App Service de créer une livraison continue via Azure Pipelines, votre organisation Azure DevOps doit disposer des autorisations suivantes : 

- Votre compte Azure doit être autorisé à écrire dans Azure Active Directory et à créer un service. 
  
- Votre compte Azure doit disposer du rôle **Propriétaire** dans votre abonnement Azure.

- Vous devez être administrateur dans le projet Azure DevOps que vous souhaitez utiliser.

#### <a name="github--azure-pipelines"></a>GitHub + Azure Pipelines

1. Sur le [portail Azure](https://portal.azure.com), recherchez **App Services**, puis sélectionnez le service App Service que vous souhaitez déployer. 
   
1. Sur la page de l'application, sélectionnez **Centre de déploiement** dans le menu de gauche.

1. Sélectionnez **GitHub** comme fournisseur de contrôle de code source dans la page **Centre de déploiement**, puis choisissez **Continuer**. Pour **GitHub**, vous pouvez sélectionner **Changer de compte** pour changer le compte autorisé.

    :::image type="content" source="media/app-service-continuous-deployment/deployment-center-src-control.png" alt-text="Capture d'écran de la page Centre de déploiement App Service.":::
   
1. Dans la page **Fournisseur de générations**, sélectionnez **Azure Pipelines (Préversion)** , puis **Continuer**.

    :::image type="content" source="media/app-service-continuous-deployment/select-build-provider.png" alt-text="Capture d'écran de la page Centre de déploiement, sur laquelle Azure Pipelines (préversion) est sélectionné.":::
   
1. Dans la page **Configurer**, dans la section **Code**, sélectionnez l’**organisation**, le **dépôt** et la **branche** que vous souhaitez déployer en continu, puis choisissez **Continuer**.
     
     > [!NOTE]
     > En l'absence de référentiel, il peut être nécessaire d'autoriser Azure App Service dans GitHub. Accédez à votre référentiel GitHub, puis sélectionnez **Paramètres** > **Applications** > **Applications OAuth autorisées**. Sélectionnez **Azure App Service**, puis **Octroyer**. Pour les dépôts de l’organisation, vous devez être propriétaire de l’organisation pour accorder des autorisations.
       
    Dans la section **Build**, spécifiez l’organisation Azure DevOps, le projet, le framework de langage que doit utiliser Azure Pipelines pour exécuter les tâches de build, puis sélectionnez **Continuer**.

   :::image type="content" source="media/app-service-continuous-deployment/build-configure.png" alt-text="Capture d'écran de la section Build, avec un exemple de texte dans les champs.":::

1. Une fois le fournisseur de builds configuré, vérifiez les paramètres sur la page **Résumé**, puis sélectionnez **Terminer**.

   :::image type="content" source="media/app-service-continuous-deployment/summary.png" alt-text="Capture d'écran de la page Centre de déploiement présentant les validations et les déploiements et sur laquelle le bouton Actualiser est en surbrillance.":::
   
1. Les nouvelles validations dans le dépôt et la branche sélectionnés sont maintenant déployées en continu dans votre service App Service. Vous pouvez suivre les validations et les déploiements sur la page **Centre de déploiement**.
   
   ![Suivre les validations et les déploiements dans le Centre de déploiement](media/app-service-continuous-deployment/github-finished.png)

#### <a name="azure-repos--azure-pipelines"></a>Azure Repos + Azure Pipelines

1. Sur le [portail Azure](https://portal.azure.com), recherchez **App Services**, puis sélectionnez le service App Service que vous souhaitez déployer. 
   
1. Sur la page de l'application, sélectionnez **Centre de déploiement** dans le menu de gauche.

1. Sélectionnez **Azure Repos** comme fournisseur de contrôle de code source dans la page **Centre de déploiement**, puis choisissez **Continuer**.

    :::image type="content" source="media/app-service-continuous-deployment/deployment-center-src-control.png" alt-text="Capture d'écran de la page Centre de déploiement présentant les sélections de Déploiements continus (CI/CD).":::

1. Dans la page **Fournisseur de générations**, sélectionnez **Azure Pipelines (Préversion)** , puis **Continuer**.

    :::image type="content" source="media/app-service-continuous-deployment/azure-pipelines.png" alt-text="Capture d'écran du Centre de déploiement présentant Azure Pipelines (préversion).":::

1. Dans la page **Configurer**, dans la section **Code**, sélectionnez l’**organisation**, le **dépôt** et la **branche** que vous souhaitez déployer en continu, puis choisissez **Continuer**.

   > [!NOTE]
   > Si votre organisation Azure DevOps existante n'est pas répertoriée, vous serez peut-être amené à devoir la lier à votre abonnement Azure. Pour plus d’informations, consultez [Définir votre pipeline de mise en production CD](/azure/devops/pipelines/apps/cd/deploy-webdeploy-webapps#cd).

   Dans la section **Build**, spécifiez l’organisation Azure DevOps, le projet, le framework de langage que doit utiliser Azure Pipelines pour exécuter les tâches de build, puis sélectionnez **Continuer**.

   :::image type="content" source="media/app-service-continuous-deployment/build-configure.png" alt-text="Capture d'écran de la section Build présentant les champs Organisation et Projet d'Azure DevOps avec des exemples.":::

1. Une fois le fournisseur de builds configuré, vérifiez les paramètres sur la page **Résumé**, puis sélectionnez **Terminer**.  
     
   :::image type="content" source="media/app-service-continuous-deployment/summary-azure-pipelines.png" alt-text="Capture d'écran des paramètres sélectionnés sur la page Résumé.":::

1. Les nouvelles validations dans le dépôt et la branche sélectionnés sont maintenant déployées en continu dans votre service App Service. Vous pouvez suivre les validations et les déploiements sur la page **Centre de déploiement**.

## <a name="disable-continuous-deployment"></a>Désactiver le déploiement continu

Pour désactiver le déploiement continu, sélectionnez **Déconnecter** en haut de la page **Centre de déploiement** de votre application.

![Désactiver le déploiement continu](media/app-service-continuous-deployment/disable.png)

[!INCLUDE [What happens to my app during deployment?](../../includes/app-service-deploy-atomicity.md)]

## <a name="use-unsupported-repos"></a>Utiliser des dépôts non pris en charge

Pour les applications Windows, vous pouvez configurer manuellement le déploiement continu à partir d’un dépôt Git ou Mercurial dans le cloud que le portail ne prend pas directement en charge, comme [GitLab](https://gitlab.com/). Pour cela, choisissez la case Externe dans la page **Centre de déploiement**. Pour plus d’informations, consultez [Configurer le déploiement continu avec des étapes manuelles](https://github.com/projectkudu/kudu/wiki/Continuous-deployment#setting-up-continuous-deployment-using-manual-steps).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Examiner les problèmes courants liés au déploiement continu](https://github.com/projectkudu/kudu/wiki/Investigating-continuous-deployment)
* [Utilisation d'Azure PowerShell](/powershell/azure/)
* [Documentation de Git](https://git-scm.com/documentation)
* [Project Kudu](https://github.com/projectkudu/kudu/wiki)

[Création d’un dépôt (GitHub)]: https://help.github.com/articles/create-a-repo
[Création d’un dépôt (BitBucket)]: https://confluence.atlassian.com/get-started-with-bitbucket/create-a-repository-861178559.html
[Créer un référentiel Git (Azure Repos)]: /azure/devops/repos/git/creatingrepo