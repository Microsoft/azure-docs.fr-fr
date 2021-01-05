---
title: Questions fréquentes (FAQ) sur le déploiement- Azure App Service | Microsoft Docs
description: Obtenez des réponses aux questions fréquemment posées sur le déploiement de la fonctionnalité Azure Web Apps d’Azure App Service.
author: genlin
manager: dcscontentpm
tags: top-support-issue
ms.assetid: 2fa5ee6b-51a6-4237-805f-518e6c57d11b
ms.topic: article
ms.date: 11/01/2018
ms.author: genli
ms.custom: seodec18
ms.openlocfilehash: 163a6940e50d1f8beacc23855fd1e6f9daad0085
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88080471"
---
# <a name="deployment-faqs-for-web-apps-in-azure"></a>Forum aux questions sur le déploiement de Web Apps dans Azure

Cet article contient des réponses aux questions fréquemment posées sur les problèmes de déploiement de la [fonctionnalité Web Apps d’Azure App Service](https://azure.microsoft.com/services/app-service/web/).

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="i-am-just-getting-started-with-app-service-web-apps-how-do-i-publish-my-code"></a>Je découvre App Service Web Apps. Comment publier mon code ?

Voici quelques options pour la publication de votre code d’application web :

*   Effectuer un déploiement à l’aide de Visual Studio. Si vous disposez de la solution Visual Studio, cliquez avec le bouton droit sur le projet d’application web, puis sélectionnez **Publier**.
*   Déployez à l’aide d’un client FTP. Dans le portail Azure, téléchargez le profil de publication de l’application web vers laquelle vous souhaitez déployer votre code. Ensuite, chargez les fichiers dans \site\wwwroot en utilisant les informations d’identification FTP du profil de publication.

Pour plus d’informations, voir [Déploiement de votre application dans Azure App Service](deploy-local-git.md).

## <a name="i-see-an-error-message-when-i-try-to-deploy-from-visual-studio-how-do-i-resolve-this-error"></a>Un message d’erreur s’affiche quand je tente de déployer à partir de Visual Studio. Comment faire pour résoudre cette erreur ?

Un message peut indiquer que vous utilisez une ancienne version du Kit de développement logiciel (SDK).  Ce message est le suivant : « Erreur lors du déploiement pour la ressource ’NomDeVotreRessource’ du groupe de ressources ’NomDeVotreGroupeDeRessources’ : InscriptionManquantePourEmplacement : L’abonnement n’est pas inscrit pour le type de ressource « composants » à l’emplacement « USA Centre ». Renouvelez l’inscription pour ce fournisseur afin d’avoir accès à cet emplacement. » 

Pour résoudre cette erreur, mettez à niveau vers la [dernière version du Kit de développement logiciel (SDK)](https://azure.microsoft.com/downloads/). Si vous voyez ce message et disposez de la dernière version du Kit de développement logiciel (SDK), envoyez une demande de support.

## <a name="how-do-i-deploy-an-aspnet-application-from-visual-studio-to-app-service"></a>Comment déployer une application ASP.NET à partir de Visual Studio vers App Service ?
<a id="deployasp"></a>

Le didacticiel [Création d’une application web ASP.NET dans Azure](quickstart-dotnetcore.md) explique comment déployer une application web ASP.NET vers une application web dans App Service à l’aide de Visual Studio 2019.

## <a name="what-are-the-different-types-of-deployment-credentials"></a>Quelles sont les différents types d’informations d’identification de déploiement ?

App Service prend en charge deux types d’informations d’identification pour le déploiement Git local et le déploiement FTP/S. Pour plus d’informations sur la façon de configurer les informations d’identification de déploiement, voir [Configurer les informations d’identification de déploiement pour Azure App Service](deploy-configure-credentials.md).

## <a name="what-is-the-file-or-directory-structure-of-my-app-service-web-app"></a>Qu’est la structure de fichiers ou de répertoires de mon application web App Service ?

Pour plus d’informations sur la structure de fichiers de votre application App Service, voir [Structure de fichiers dans Azure](https://github.com/projectkudu/kudu/wiki/File-structure-on-azure).

## <a name="how-do-i-resolve-ftp-error-550---there-is-not-enough-space-on-the-disk-when-i-try-to-ftp-my-files"></a>Comment résoudre l’« Erreur FTP 550 - Espace insuffisant sur le disque » lorsque j’essaie de transférer mes fichiers via FTP ?

Si vous voyez ce message, il est probable que vous atteignez un quota de disque dans le plan de service pour votre application web. Vous devez peut-être monter en puissance en passant à un niveau de service supérieur correspondant à vos besoins d’espace disque. Pour plus d’informations sur les plans de tarification et les limites de ressources, voir [Tarification de App Service](https://azure.microsoft.com/pricing/details/app-service/).

## <a name="how-do-i-set-up-continuous-deployment-for-my-app-service-web-app"></a>Comment configurer un déploiement continu pour mon application web App Service ?

Vous pouvez configurer un déploiement continu à partir de plusieurs ressources, dont Azure DevOps, OneDrive, GitHub, Bitbucket, Dropbox et d’autres référentiels Git. Ces options sont disponibles dans le portail. Le didacticiel [Déploiement continu vers App Service](deploy-continuous-deployment.md) explique comment configurer un déploiement continu.

## <a name="how-do-i-troubleshoot-issues-with-continuous-deployment-from-github-and-bitbucket"></a>Comment résoudre les problèmes de déploiement continu à partir de GitHub et de Bitbucket ?

Pour approfondir les problèmes liés à un déploiement continu à partir de GitHub ou de Bitbucket, voir [Analyse du déploiement continu](https://github.com/projectkudu/kudu/wiki/Investigating-continuous-deployment).

## <a name="i-cant-ftp-to-my-site-and-publish-my-code-how-do-i-resolve-this-issue"></a>Je ne peux pas effectuer de transfert FTP vers mon site et publier mon code. Comment faire pour résoudre ce problème ?

Pour résoudre les problèmes de FTP :

1. Vérifiez que vous avez entré le nom d’hôte et les informations d’identification corrects. Pour plus d’informations sur les différents types d’informations d’identification et leur utilisation, voir [Informations d’identification pour le déploiement](https://github.com/projectkudu/kudu/wiki/Deployment-credentials).
2. Vérifiez que les ports ne sont pas bloqués par un pare-feu. Les ports doivent être paramétrés comme suit :
    * Port de connexion de contrôle FTP : 21
    * Port de connexion de contrôle FTP : 989, 10001-10300

## <a name="how-do-i-publish-my-code-to-app-service"></a>Comment publier mon code sur App Service ?

Le démarrage rapide Azure est conçu pour vous aider à déployer votre application à l’aide de la pile de déploiement et de la méthode de votre choix. Pour utiliser le démarrage rapide, dans le portail Azure, accédez à votre App Service, sous **Déploiement**, sélectionnez **Démarrage rapide**.

## <a name="why-does-my-app-sometimes-restart-after-deployment-to-app-service"></a>Pourquoi mon application redémarre-t-elle parfois après un déploiement sur App Service ?

Pour en savoir plus sur les circonstances dans lesquelles un déploiement d’application peut entraîner un redémarrage, voir [Problèmes de déploiement et d’exécution](https://github.com/projectkudu/kudu/wiki/Deployment-vs-runtime-issues#deployments-and-web-app-restarts"). Comme décrit dans l’article, App Service déploie les fichiers dans le dossier wwwroot. Il ne redémarre jamais directement votre application.

## <a name="how-do-i-integrate-azure-devops-code-with-app-service"></a>Comment intégrer le code d’Azure DevOps avec App Service ?

Vous disposez de deux options pour utiliser un déploiement continu avec Azure DevOps :

*   Utiliser un projet Git. Connectez-vous via App Service en utilisant le Centre de déploiement.
*   Utiliser un projet Team Foundation Version Control (TFVC). Déployez à l’aide de l’agent de build pour App Service.

Le déploiement de code en continu pour ces deux options dépend des flux de travail de développeur et des procédures de vérification existants. Pour plus d’informations, voir les articles suivants : 

*   [Implémenter un déploiement continu de votre application sur un site web Azure](https://www.visualstudio.com/docs/release/examples/azure/azure-web-apps-from-build-and-release-hubs)
*   [Configurer une organisation Azure DevOps de façon à ce qu’elle puisse opérer un déploiement vers une application web](https://github.com/projectkudu/kudu/wiki/Setting-up-a-VSTS-account-so-it-can-deploy-to-a-Web-App)

## <a name="how-do-i-use-ftp-or-ftps-to-deploy-my-app-to-app-service"></a>Comment utiliser FTP ou FTPS pour déployer mon application vers App Service ?

Pour plus d’informations sur l’utilisation de FTP ou FTPS pour déployer votre application web vers App Service, voir [Déployer votre application dans Azure App Service avec FTP/S](deploy-ftp.md).
