---
title: Déployer à partir d’un dépôt Git local
description: Découvrez comment activer le déploiement Git local vers Azure App Service L’une des façons les plus simples de déployer du code à partir de votre ordinateur local.
ms.assetid: ac50a623-c4b8-4dfd-96b2-a09420770063
ms.topic: article
ms.date: 06/18/2019
ms.reviewer: dariac
ms.custom: seodec18, devx-track-azurecli
ms.openlocfilehash: 26fd8bc73fad3ea313641fc4b1e0f454ee2c0813
ms.sourcegitcommit: fa807e40d729bf066b9b81c76a0e8c5b1c03b536
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97347776"
---
# <a name="local-git-deployment-to-azure-app-service"></a>Déploiement Git local vers Azure App Service

Ce guide pratique vous montre comment déployer votre application sur [Azure App Service](overview.md) depuis un référentiel Git sur votre ordinateur local.

## <a name="prerequisites"></a>Prérequis

Pour suivre les étapes décrites dans ce guide de procédures :

- [!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]
  
- [Installer Git](https://www.git-scm.com/downloads).
  
- Obtenez un référentiel Git local comprenant le code que vous souhaitez déployer. Pour télécharger un exemple de référentiel, exécutez la commande suivante dans la fenêtre de terminal locale :
  
  ```bash
  git clone https://github.com/Azure-Samples/nodejs-docs-hello-world.git
  ```

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

[!INCLUDE [Prepare repository](../../includes/app-service-deploy-prepare-repo.md)]

## <a name="deploy-with-kudu-build-server"></a>Déployer avec un serveur de builds Kudu

Pour permettre un déploiement Git local pour votre application avec le serveur de builds App Service Kudu, le plus simple consiste à utiliser Azure Cloud Shell. 

### <a name="configure-a-deployment-user"></a>Configuration d’un utilisateur de déploiement

[!INCLUDE [Configure a deployment user](../../includes/configure-deployment-user-no-h.md)]

### <a name="get-the-deployment-url"></a>Obtenir l'URL de déploiement

Pour obtenir l'URL permettant d'activer le déploiement Git local pour une application existante, exécutez [`az webapp deployment source config-local-git`](/cli/azure/webapp/deployment/source#az-webapp-deployment-source-config-local-git) dans Cloud Shell. Remplacez \<app-name> et \<group-name> par les noms de votre application et de son groupe de ressources Azure.

```azurecli-interactive
az webapp deployment source config-local-git --name <app-name> --resource-group <group-name>
```
> [!NOTE]
> Si vous utilisez un plan App Service Linux, vous devez ajouter ce paramètre : --runtime python|3.7


Pour créer une application Git, vous pouvez également exécuter [`az webapp create`](/cli/azure/webapp#az-webapp-create) dans Cloud Shell avec le paramètre `--deployment-local-git`. Remplacez \<app-name>, \<group-name> et \<plan-name> par les noms de votre nouvelle application Git, de son groupe de ressources Azure et de son plan Azure App Service.

```azurecli-interactive
az webapp create --name <app-name> --resource-group <group-name> --plan <plan-name> --deployment-local-git
```

L’une ou l’autre des commandes renvoie une URL comme : `https://<deployment-username>@<app-name>.scm.azurewebsites.net/<app-name>.git`. Utilisez cette URL pour déployer votre application à l’étape suivante.

Plutôt que d'utiliser cette URL au niveau du compte, vous pouvez également activer Git local à l’aide des informations d’identification au niveau de l’application. Azure App Service génère automatiquement ces informations d’identification pour chaque application. 

Pour obtenir les informations d'identification, exécutez la commande suivante dans Cloud Shell. Remplacez \<app-name> et \<group-name> par le nom de votre application et le nom du groupe de ressources Azure.

```azurecli-interactive
az webapp deployment list-publishing-credentials --name <app-name> --resource-group <group-name> --query scmUri --output tsv
```

Utilisez l’URL renvoyée pour déployer votre application à l’étape suivante.

### <a name="deploy-the-web-app"></a>Déployer l’application web

1. Ouvrez une fenêtre de terminal local vers votre référentiel Git local et ajoutez un référentiel distant Azure. Dans la commande suivante, remplacez \<url> par l’URL de déploiement spécifique de l’utilisateur ou l’URL spécifique de l’application que vous avez obtenue à l’étape précédente.
   
   ```bash
   git remote add azure <url>
   ```
   
1. Effectuez une transmission de type push vers Azure avec `git push azure master`. 
   
1. Dans la fenêtre **Git Credential Manager**, entrez votre [mot de passe d'utilisateur de déploiement](#configure-a-deployment-user), et non votre mot de passe de connexion Azure.
   
1. Passez en revue la sortie. Vous pouvez voir une automation spécifique au runtime, comme MSBuild pour ASP.NET, `npm install` pour Node.js et `pip install` pour Python. 
   
1. Dans le portail Azure, accédez à votre application pour vérifier que le contenu a été déployé.

## <a name="deploy-with-azure-pipelines-builds"></a>Déployer avec des builds Azure Pipelines

Si votre compte dispose des autorisations nécessaires, vous pouvez configurer Azure Pipelines (préversion) afin d'activer le déploiement Git local pour votre application. 

- Votre compte Azure doit être autorisé à écrire dans Azure Active Directory et à créer un service. 
  
- Votre compte Azure doit disposer du rôle **Propriétaire** dans votre abonnement Azure.

- Vous devez être administrateur dans le projet Azure DevOps que vous souhaitez utiliser.

Pour activer le déploiement Git local pour votre application avec Azure Pipelines (préversion) :

1. Dans le [portail Azure](https://portal.azure.com), recherchez et sélectionnez **App Services**. 

1. Sélectionnez votre application Azure App Service, puis sélectionnez **Centre de déploiement** dans le menu de gauche.
   
1. Dans la page **Centre de déploiement**, sélectionnez **Git local**, puis **Continuer**. 
   
   ![Sélectionner Git local, puis Continuer](media/app-service-deploy-local-git/portal-enable.png)
   
1. Dans la page **Fournisseur de générations**, sélectionnez **Azure pipelines (préversion)** , puis **Continuer**. 
   
   ![Sélectionnez Azure Pipelines (préversion), puis Continuer.](media/app-service-deploy-local-git/pipeline-builds.png)

1. Dans la page **Configurer**, configurez une nouvelle organisation Azure DevOps ou spécifiez une organisation existante, puis sélectionnez **Continuer**.
   
   > [!NOTE]
   > Si votre organisation Azure DevOps existante n'est pas répertoriée, vous serez peut-être amené à devoir la lier à votre abonnement Azure. Pour plus d’informations, consultez [Définir votre pipeline de mise en production CD](/azure/devops/pipelines/apps/cd/deploy-webdeploy-webapps#cd).
   
1. En fonction du [niveau tarifaire](https://azure.microsoft.com/pricing/details/app-service/plans/) de votre plan App Service, il peut arriver qu’une page **Déployer en préproduction** s'affiche. Choisissez si vous souhaitez [activer des emplacements de déploiement](deploy-staging-slots.md), puis sélectionnez **Continuer**.
   
1. Dans la page **Résumé**, vérifiez les paramètres, puis sélectionnez **Terminer**.
   
1. Une fois le pipeline Azure prêt, copiez l’URL du référentiel Git à partir de la page **Centre de déploiement** pour l'utiliser à l'étape suivante. 
   
   ![Copier l’URL du référentiel Git](media/app-service-deploy-local-git/vsts-repo-ready.png)

1. Dans votre fenêtre de terminal local, ajoutez un référentiel distant Azure à votre référentiel Git local. Dans la commande, remplacez \<url> par l’URL du référentiel Git que vous avez obtenue à l’étape précédente.
   
   ```bash
   git remote add azure <url>
   ```
   
1. Effectuez une transmission de type push vers Azure avec `git push azure master`. 
   
1. Dans la page **Git Credential Manager**, connectez-vous avec votre nom d'utilisateur visualstudio.com. Pour découvrir d’autres méthodes d’authentification, consultez [Vue d’ensemble de l’authentification Azure DevOps Services](/vsts/git/auth-overview?view=vsts).
   
1. Une fois le déploiement terminé, vous pouvez afficher la progression de la génération sur `https://<azure_devops_account>.visualstudio.com/<project_name>/_build` et la progression du déploiement sur `https://<azure_devops_account>.visualstudio.com/<project_name>/_release`.
   
1. Dans le portail Azure, accédez à votre application pour vérifier que le contenu a été déployé.

[!INCLUDE [What happens to my app during deployment?](../../includes/app-service-deploy-atomicity.md)]

## <a name="troubleshoot-deployment"></a>Résoudre les problèmes de déploiement

Les messages d'erreur suivants peuvent s'afficher lorsque vous utilisez Git pour publier une application App Service dans Azure :

|Message|Cause|Résolution
---|---|---|
|`Unable to access '[siteURL]': Failed to connect to [scmAddress]`|L’application n’est pas opérationnelle.|Démarrez l’application dans le portail Azure. Le déploiement Git n'est pas disponible lorsque l’application web est arrêtée.|
|`Couldn't resolve host 'hostname'`|Les informations d’adresse du référentiel «Azure» distant ne sont pas correctes.|Utilisez la commande `git remote -v` pour répertorier tous les référentiels distants avec l’URL associée. Vérifiez que l'URL du référentiel distant « azure » est correcte. Si nécessaire, supprimez et recréez ce référentiel distant au moyen de l’URL correcte.|
|`No refs in common and none specified; doing nothing. Perhaps you should specify a branch such as 'main'.`|Vous n’avez pas spécifié de branche pendant `git push` ou vous n'avez pas défini la valeur `push.default` dans `.gitconfig`.|Réexécutez `git push`, en spécifiant la branche primaire : `git push azure master`.|
|`src refspec [branchname] does not match any.`|Vous avez tenté d’effectuer une transmission de type push sur une autre branche que la branche primaire du référentiel distant « azure ».|Réexécutez `git push`, en spécifiant la branche maîtresse : `git push azure master`.|
|`RPC failed; result=22, HTTP code = 5xx.`|Cette erreur peut se produire si vous essayez d’envoyer (push) un dépôt Git volumineux via HTTPS.|Modifiez la configuration Git sur l’ordinateur local pour agrandir le `postBuffer`. Par exemple : `git config --global http.postBuffer 524288000`.|
|`Error - Changes committed to remote repository but your web app not updated.`|Vous avez déployé une application Node.js contenant un fichier _package.json_ spécifiant des modules obligatoires supplémentaires.|Examinez les messages d'erreur `npm ERR!` préalables à cette erreur pour plus de contexte sur l’échec. Voici les causes connues de cette erreur et les messages `npm ERR!` correspondants :<br /><br />**Fichier package.json incorrect**: `npm ERR! Couldn't read dependencies.`<br /><br />**Un module natif n’a pas de distribution binaire pour Windows** :<br />`npm ERR! \cmd "/c" "node-gyp rebuild"\ failed with 1` <br />or <br />`npm ERR! [modulename@version] preinstall: \make || gmake\ `|

## <a name="additional-resources"></a>Ressources supplémentaires

- [Documentation du projet Kudu](https://github.com/projectkudu/kudu/wiki)
- [Déploiement continu vers Azure App Service](deploy-continuous-deployment.md)
- [Exemple : Créer une application web et déployer du code à partir d’un référentiel Git local (Azure CLI)](./scripts/cli-deploy-local-git.md?toc=%2fcli%2fazure%2ftoc.json)
- [Exemple : Créer une application web et déployer du code à partir d’un référentiel Git local (PowerShell)](./scripts/powershell-deploy-local-git.md?toc=%2fpowershell%2fmodule%2ftoc.json)
