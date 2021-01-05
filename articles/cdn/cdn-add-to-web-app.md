---
title: 'Didacticiel : Ajouter Azure CDN à une application web Azure App Service | Microsoft Docs'
description: Dans ce didacticiel, vous allez apprendre à ajouter un réseau de distribution de contenu (CDN) à une application web Azure App Service pour mettre en cache et distribuer vos fichiers statiques à partir des serveurs situés à proximité de vos clients, partout dans le monde.
services: cdn
documentationcenter: ''
author: asudbring
manager: danielgi
editor: ''
ms.assetid: ''
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 05/14/2018
ms.author: allensu
ms.custom: mvc
ms.openlocfilehash: 3f1dacb57931913edfb181f023bdf98717777d50
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97562240"
---
# <a name="tutorial-add-azure-cdn-to-an-azure-app-service-web-app"></a>Tutoriel : Ajouter Azure CDN à une application web Azure App Service

Ce didacticiel montre comment ajouter [Azure Content Delivery Network (CDN)](cdn-overview.md) à une [application web dans Azure App Service](../app-service/overview.md). Web Apps est un service pour l’hébergement d’applications web, d’API REST et de backends mobiles. 

Voici la page d’accueil de l’exemple de site HTML statique que vous allez utiliser :

![Page d’accueil de l’exemple d’application](media/cdn-add-to-web-app/sample-app-home-page.png)

Ce que vous allez apprendre :

> [!div class="checklist"]
> * Créer un point de terminaison CDN.
> * Actualiser les ressources mises en cache.
> * Utiliser des chaînes de requête pour contrôler les versions mises en cache.


## <a name="prerequisites"></a>Prérequis

Pour suivre ce tutoriel :

- [Installez Git](https://git-scm.com/)
- [Installer l’interface de ligne de commande Microsoft Azure](/cli/azure/install-azure-cli)

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="create-the-web-app"></a>Créer l’application web

Pour créer l’application web que vous allez utiliser, suivez les instructions de l’article [Créer une application web HTML statique dans Azure](../app-service/quickstart-html.md) jusqu’à la fin de l’étape **Accéder à l’application**.

## <a name="log-in-to-the-azure-portal"></a>Se connecter au portail Azure.

Ouvrez un navigateur et accédez au [portail Azure](https://portal.azure.com).

### <a name="dynamic-site-acceleration-optimization"></a>Optimisation de l’accélération de site dynamique
Si vous souhaitez optimiser votre point de terminaison CDN pour l’accélération de site dynamique (DSA), utilisez le [portail CDN](cdn-create-new-endpoint.md) pour créer votre profil et le point de terminaison. Grâce à [l’optimisation DSA](cdn-dynamic-site-acceleration.md), le niveau de performance des pages web comprenant du contenu dynamique est sensiblement amélioré. Pour obtenir des instructions sur l’optimisation d’un point de terminaison CDN pour DSA à partir du portail CDN, consultez [Configuration d’un point de terminaison CDN pour accélérer la distribution de fichiers dynamiques](cdn-dynamic-site-acceleration.md#cdn-endpoint-configuration-to-accelerate-delivery-of-dynamic-files). Sinon, si vous ne souhaitez pas optimiser votre nouveau point de terminaison, vous pouvez utiliser le portail d’applications web pour le créer en suivant les étapes décrites dans la section suivante. Remarque : pour les profils **CDN Azure fournis par Verizon**, il n’est pas possible de modifier l’optimisation d’un point de terminaison CDN après sa création.

## <a name="create-a-cdn-profile-and-endpoint"></a>Création d’un profil CDN et d’un point de terminaison

Dans le volet de navigation gauche, sélectionnez **App Services**, puis sélectionnez l’application que vous avez créée à la rubrique [Créer une application web HTML statique dans Azure en 5 minutes](../app-service/quickstart-html.md).

![Sélection de l’application App Service dans le portail](media/cdn-add-to-web-app/portal-select-app-services.png)

Dans la page **App Service**, dans la section **Paramètres**, sélectionnez **Mise en réseau > Configurer Azure CDN pour votre application**.

![Sélection de CDN dans le portail](media/cdn-add-to-web-app/portal-select-cdn.png)

Dans la page **Azure Content Delivery Network**, fournissez les paramètres du **Nouveau point de terminaison**, comme spécifié dans le tableau.

![Création d’un profil et d’un point de terminaison dans le portail](media/cdn-add-to-web-app/portal-new-endpoint.png)

| Paramètre | Valeur suggérée | Description |
| ------- | --------------- | ----------- |
| **Profil CDN** | myCDNProfile | Un profil CDN est une collection de points de terminaison CDN possédant le même niveau tarifaire. |
| **Niveau tarifaire** | Standard Akamai | Le [niveau tarifaire](cdn-features.md) spécifie le fournisseur et les fonctionnalités disponibles. Ce didacticiel utilise *Akamai standard*. |
| **Nom du point de terminaison CDN** | N’importe quel nom qui est unique dans le domaine azureedge.net | Vous accédez à vos ressources mises en cache au niveau du *&lt;Nom_Point_Terminaison&gt;* .azureedge.net. du domaine.

Sélectionnez **Créer** pour créer un profil CDN.

Azure crée le profil et le point de terminaison. Le nouveau point de terminaison s’affiche dans la liste **Points de terminaison**, et lorsqu’il est configuré, son état est **Exécution en cours**.

![Nouveau point de terminaison dans la liste](media/cdn-add-to-web-app/portal-new-endpoint-in-list.png)

### <a name="test-the-cdn-endpoint"></a>Test du point de terminaison CDN

 Le point de terminaison n’est pas disponible immédiatement, car la propagation de l’enregistrement peut prendre du temps : 
   - Pour les profils du **CDN Azure Standard fourni par Microsoft**, la propagation s’effectue généralement dans un délai de 10 minutes. 
   - Pour les profils du **CDN Azure Standard fourni par Akamai**, la propagation s’effectue généralement dans un délai d’une minute. 
   - Dans le cas des profils du **CDN Azure Standard fourni par Verizon** et du **CDN Azure Premium fourni par Verizon**, la propagation s’effectue généralement dans un délai de 90 minutes. 

L’application fournie à titre d’exemple comprend un fichier *index.html* et des dossiers *css*, *img* et *js* contenant d’autres ressources statiques. Les chemins d’accès de contenu pour tous ces fichiers sont identiques au point de terminaison CDN. Par exemple, les deux URL suivantes accèdent au fichier *bootstrap.css* dans le dossier *css* :

```
http://<appname>.azurewebsites.net/css/bootstrap.css
```

```
http://<endpointname>.azureedge.net/css/bootstrap.css
```

Dans un navigateur, accédez à l’URL suivante :

```
http://<endpointname>.azureedge.net/index.html
```

![Page d’accueil de l’exemple d’application traitée à partir du CDN](media/cdn-add-to-web-app/sample-app-home-page-cdn.png)

 Vous consultez la page que vous avez exécutée précédemment dans une application web Azure. Azure CDN a récupéré les ressources de l’application web d’origine et les traite à partir du point de terminaison CDN.

Pour vous assurer que cette page est mise en cache dans le CDN, actualisez la page. Deux demandes pour la même ressource sont parfois nécessaires pour que le CDN mette en cache le contenu demandé.

Pour plus d’informations sur la création de profils et de points de terminaison Azure CDN, consultez [Prise en main d’Azure CDN](cdn-create-new-endpoint.md).

## <a name="purge-the-cdn"></a>Vidage du CDN

Le CDN actualise régulièrement ses ressources à partir de l’application web d’origine en fonction de la configuration de la durée de vie (TTL). La valeur TTL par défaut est de sept jours.

Dans certains cas, vous devrez peut-être actualiser le CDN avant l’expiration de la durée de vie. C’est par exemple le cas lorsque vous déployez un contenu mis à jour dans l’application web. Pour déclencher une actualisation, videz manuellement les ressources du CDN. 

Dans cette section du didacticiel, vous déployez une modification dans l’application web et videz le CDN pour le forcer à actualiser son cache.

### <a name="deploy-a-change-to-the-web-app"></a>Déploiement d’une modification dans l’application web

Ouvrez le fichier *index.html* et ajoutez *- V2* au titre H1, comme illustré dans l’exemple suivant : 

```
<h1>Azure App Service - Sample Static HTML Site - V2</h1>
```

Validez votre modification et déployez-la dans l’application web.

```bash
git commit -am "version 2"
git push azure main
```

Une fois le déploiement terminé, accédez à l’URL de l’application web pour constater la modification.

```
http://<appname>.azurewebsites.net/index.html
```

![V2 dans le titre de l’application web](media/cdn-add-to-web-app/v2-in-web-app-title.png)

Si vous ouvrez la page d’accueil à partir de l’URL du point de terminaison CDN, vous ne pourrez pas visualiser cette modification, car la version mise en cache dans le CDN n’a pas encore expiré. 

```
http://<endpointname>.azureedge.net/index.html
```

![Pas de V2 dans le titre du CDN](media/cdn-add-to-web-app/no-v2-in-cdn-title.png)

### <a name="purge-the-cdn-in-the-portal"></a>Vidage du CDN dans le portail

Pour forcer le CDN à mettre à jour sa version mise en cache, videz le CDN.

Dans le portail de navigation gauche, sélectionnez **Groupes de ressources**, puis sélectionnez le groupe de ressources que vous avez créé pour votre application web (myResourceGroup).

![Sélection du groupe de ressources](media/cdn-add-to-web-app/portal-select-group.png)

Dans la liste des ressources, sélectionnez votre point de terminaison CDN.

![Sélection du point de terminaison](media/cdn-add-to-web-app/portal-select-endpoint.png)

En haut de la page **Point de terminaison**, sélectionnez **Vider**.

![Sélection de l’option Vider](media/cdn-add-to-web-app/portal-select-purge.png)

Entrez les chemins d’accès au contenu que vous souhaitez vider. Vous pouvez transmettre un chemin d’accès complet pour vider un fichier individuel ou un segment de chemin d’accès pour vider et actualiser tout le contenu d’un dossier. Étant donné que vous avez modifié *index.html*, assurez-vous que ce fichier se trouve dans l’un des chemins d’accès.

En bas de la page, sélectionnez **Vider**.

![Vidage de la page](media/cdn-add-to-web-app/app-service-web-purge-cdn.png)

### <a name="verify-that-the-cdn-is-updated"></a>Vérification de la mise à jour du CDN

Patientez jusqu’à la fin du traitement de la demande de vidage. Quelques minutes sont habituellement nécessaires. Pour afficher l’état actuel, sélectionnez l’icône représentant une cloche en haut de la page. 

![Notification de vidage](media/cdn-add-to-web-app/portal-purge-notification.png)

Si vous accédez au fichier *index.html* à partir de l’URL du point de terminaison CDN, vous pourrez voir la valeur *V2* que vous avez ajoutée au titre sur la page d’accueil, ce qui indique que le cache du CDN a bien été actualisé.

```
http://<endpointname>.azureedge.net/index.html
```

![V2 dans le titre du CDN](media/cdn-add-to-web-app/v2-in-cdn-title.png)

Pour plus d’informations, consultez [Purger un point de terminaison CDN Azure](../cdn/cdn-purge-endpoint.md). 

## <a name="use-query-strings-to-version-content"></a>Utiliser des chaînes de requête pour contrôler la version du contenu

Azure CDN offre les options de comportement de mise en cache suivantes :

* Ignorer les chaînes de requête
* Contourner la mise en cache pour les chaînes de requête
* Mettre en cache chaque URL unique 

La première option est celle par défaut, ce qui signifie qu’il n’existe qu’une seule version mise en cache d’une ressource, quelle que soit la chaîne de requête dans l’URL. 

Dans cette section du didacticiel, vous modifiez le comportement de mise en cache pour mettre en cache chaque URL unique.

### <a name="change-the-cache-behavior"></a>Modification du comportement de mise en cache

Dans la page **Point de terminaison CDN** du portail Azure, sélectionnez **Cache**.

Sélectionnez **Mettre en cache chaque URL unique** dans la liste déroulante **Comportement de mise en cache des chaînes de requête**.

Sélectionnez **Enregistrer**.

![Sélection du comportement de mise en cache de chaîne de requête](media/cdn-add-to-web-app/portal-select-caching-behavior.png)

### <a name="verify-that-unique-urls-are-cached-separately"></a>Vérification de la mise en cache séparée des URL uniques

Dans un navigateur, accédez à la page d’accueil au niveau du point de terminaison CDN, puis insérez une chaîne de requête : 

```
http://<endpointname>.azureedge.net/index.html?q=1
```

Azure CDN renvoie le contenu de l’application web actuelle, qui inclut *V2* dans le titre. 

Pour vous assurer que cette page est mise en cache dans le CDN, actualisez la page. 

Ouvrez *index.html*, changez *V2* en *V3*, puis déployez la modification. 

```bash
git commit -am "version 3"
git push azure main
```

Dans un navigateur, accédez à l’URL du point de terminaison CDN avec une nouvelle chaîne de requête, par exemple `q=2`. Azure CDN obtient le fichier *index.html* actuel et affiche *V3*. Toutefois, si vous naviguez vers le point de terminaison CDN avec la chaîne de requête `q=1`, c’est la valeur *V2* qui s’affiche.

```
http://<endpointname>.azureedge.net/index.html?q=2
```

![V3 dans le titre dans le CDN, chaîne de requête 2](media/cdn-add-to-web-app/v3-in-cdn-title-qs2.png)

```
http://<endpointname>.azureedge.net/index.html?q=1
```

![V2 dans le titre dans le CDN, chaîne de requête 1](media/cdn-add-to-web-app/v2-in-cdn-title-qs1.png)

Cette sortie montre que chaque chaîne de requête est traitée différemment :

* Comme la chaîne de requête q=1 a été utilisée précédemment, le contenu mis en cache est retourné (V2).
* Comme la chaîne de requête q=2 est nouvelle, le contenu le plus récent de l’application web est récupéré et retourné (V3).

Pour plus d’informations, consultez [Contrôle du comportement de mise en cache du CDN Azure avec des chaînes de requête](../cdn/cdn-query-string.md).

[!INCLUDE [cli-samples-clean-up](../../includes/cli-samples-clean-up.md)]

## <a name="next-steps"></a>Étapes suivantes

Vous avez appris à effectuer les opérations suivantes :

> [!div class="checklist"]
> * Créer un point de terminaison CDN.
> * Actualiser les ressources mises en cache.
> * Utiliser des chaînes de requête pour contrôler les versions mises en cache.

Découvrez comment optimiser les performances du réseau CDN dans les articles suivants :

> [!div class="nextstepaction"]
> [Tutoriel : Ajouter un domaine personnalisé à votre point de terminaison Azure CDN](cdn-map-content-to-custom-domain.md)