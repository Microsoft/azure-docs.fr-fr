---
title: Synchroniser du contenu à partir d’un dossier cloud
description: Découvrez comment déployer votre application dans Azure App Service via la synchronisation de contenu à partir d’un dossier cloud OneDrive ou Dropbox, par exemple.
ms.assetid: 88d3a670-303a-4fa2-9de9-715cc904acec
ms.topic: article
ms.date: 12/03/2018
ms.reviewer: dariac
ms.custom: seodec18
ms.openlocfilehash: 880edff95bb548ec5328c543a542ea5dfcfc362f
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92150289"
---
# <a name="sync-content-from-a-cloud-folder-to-azure-app-service"></a>Synchronisation de contenu à partir d’un dossier cloud dans Azure App Service
Cet article vous explique comment synchroniser votre contenu avec [Azure App Service](./overview.md) à partir de Dropbox et OneDrive. 

Le déploiement à la demande de synchronisation de contenu est généré par le [moteur de déploiement Kudu](https://github.com/projectkudu/kudu/wiki) App Service. Vous pouvez utiliser le code et le contenu de votre application dans un dossier cloud désigné, puis synchroniser sur App Service d’un seul clic. La synchronisation de contenu utilise le serveur de build Kudu. 

## <a name="enable-content-sync-deployment"></a>Activer le déploiement de la synchronisation de contenu

Pour activer la synchronisation de contenu, accédez à votre page d’application App Service dans le [portail Azure](https://portal.azure.com).

Dans le menu de gauche, cliquez sur **Centre de déploiement** > **OneDrive** ou **Dropbox** > **Autoriser**. Suivez les instructions d’autorisation. 

![Montre comment autoriser OneDrive ou Dropbox dans le Centre de déploiement du Portail Azure.](media/app-service-deploy-content-sync/choose-source.png)

Vous ne devez autoriser qu’une seule fois avec OneDrive ou Dropbox. Si vous êtes déjà autorisé, cliquez simplement sur **Continuer**. Vous pouvez modifier le compte OneDrive ou Dropbox autorisé en cliquant sur **Changer de compte**.

![Montre comment changer le compte OneDrive ou Dropbox autorisé dans le Centre de déploiement du Portail Azure.](media/app-service-deploy-content-sync/continue.png)

Sur la page **Configurer**, sélectionnez le dossier que vous voulez synchroniser. Ce dossier est créé sous le chemin de contenu désigné suivant dans OneDrive ou Dropbox. 
   
* **OneDrive** : `Apps\Azure Web Apps`
* **Dropbox** : `Apps\Azure`

Lorsque vous avez terminé, cliquez sur **Continuer**.

Sur la page **Résumé**, vérifiez les options, puis cliquez sur **Terminer**.

## <a name="synchronize-content"></a>Synchroniser le contenu

Lorsque vous voulez synchroniser le contenu dans votre dossier cloud avec App Service, revenez à la page **Centre de déploiement**, puis cliquez sur **Synchroniser**.

![Montre comment synchroniser votre dossier cloud avec App Service.](media/app-service-deploy-content-sync/synchronize.png)
   
   > [!NOTE]
   > En raison de différences sous-jacentes entre les API, **OneDrive Entreprise** n’est pas pris en charge pour l’instant. 
   > 
   > 

## <a name="disable-content-sync-deployment"></a>Désactiver le déploiement de la synchronisation de contenu

Pour désactiver la synchronisation de contenu, accédez à votre page d’application App Service dans le [portail Azure](https://portal.azure.com).

Dans le menu de gauche, cliquez sur **Centre de déploiement** > **Déconnecter**.

![Montre comment déconnecter la synchronisation de votre dossier cloud avec votre application App Service dans le Portail Azure.](media/app-service-deploy-content-sync/disable.png)

[!INCLUDE [What happens to my app during deployment?](../../includes/app-service-deploy-atomicity.md)]

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Déployer à partir du dépôt Git local](deploy-local-git.md)