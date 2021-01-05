---
title: Tutoriel - Envoyer par push une mise à jour à un registre géorépliqué
description: Envoyez (push) une image Docker mise à jour à votre registre de conteneurs Azure géorépliqué, puis déployez les modifications automatiquement sur les applications web qui s’exécutent dans plusieurs régions. Troisième partie d’une série en trois parties.
ms.topic: tutorial
ms.date: 04/30/2018
ms.custom: seodec18, mvc
ms.openlocfilehash: 9222ac31e067cba6a0ffa71143c90f906ba6ff7f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "74454696"
---
# <a name="tutorial-push-an-updated-container-image-to-a-geo-replicated-container-registry-for-regional-web-app-deployments"></a>Tutoriel : Envoyer une image conteneur mise à jour à un registre de conteneurs géorépliqué pour les déploiements d’applications web régionaux

Il s’agit de la troisième partie d’un didacticiel en trois parties. Dans le [didacticiel précédent](container-registry-tutorial-deploy-app.md), la géoréplication a été configurée pour deux déploiements d’application web régionaux différents. Dans ce didacticiel, vous allez tout d’abord modifier l’application, puis générer une image de conteneur et la distribuer à votre registre géorépliqué. Enfin, vous allez afficher la modification, déployée automatiquement par des webhooks Azure Container Registry dans les deux instances de l’application web.

Dans ce didacticiel, il s’agit de la dernière partie de la série :

> [!div class="checklist"]
> * Modification de l’application web HTML
> * Générer et marquer l’image Docker
> * Envoyer de la modification à Azure Container Registry
> * Afficher l’application mise à jour dans deux régions différentes

Si vous n’avez pas encore configuré les deux déploiements régionaux *Web App for Containers*, retournez au didacticiel précédent dans la série, [Déployer une application web à partir d’Azure Container Registry](container-registry-tutorial-deploy-app.md).

## <a name="modify-the-web-application"></a>Modification de l’application Web

Dans cette étape, apportez une modification à l’application web qui sera très visible une fois que vous enverrez l’image de conteneur mise à jour vers Azure Container Registry.

Recherchez le fichier `AcrHelloworld/Views/Home/Index.cshtml` dans la source de l’application que vous avez [clonée à partir de GitHub](container-registry-tutorial-prepare-registry.md#get-application-code) dans un didacticiel précédent et ouvrez-le dans l’éditeur de texte de votre choix. Ajoutez la ligne suivante sous la ligne `<h1>` existante :

```html
<h1>MODIFIED</h1>
```

Votre `Index.cshtml` modifié doit ressembler à ceci :

```html
@{
    ViewData["Title"] = "Azure Container Registry :: Geo-replication";
}
<style>
    body {
        background-image: url('images/azure-regions.png');
        background-size: cover;
    }
    .footer {
        position: fixed;
        bottom: 0px;
        width: 100%;
    }
</style>

<h1 style="text-align:center;color:blue">Hello World from:  @ViewData["REGION"]</h1>
<h1>MODIFIED</h1>
<div class="footer">
    <ul>
        <li>Registry URL: @ViewData["REGISTRYURL"]</li>
        <li>Registry IP: @ViewData["REGISTRYIP"]</li>
        <li>Registry Region: @ViewData["REGION"]</li>
    </ul>
</div>
```

## <a name="rebuild-the-image"></a>Régénérer l’image

Maintenant que vous avez mis à jour l’application web, régénérez son image de conteneur. Comme précédemment, utilisez le nom d’image complet, y compris le nom de domaine complet (FQDN) du serveur de connexion, pour la balise :

```bash
docker build . -f ./AcrHelloworld/Dockerfile -t <acrName>.azurecr.io/acr-helloworld:v1
```

## <a name="push-image-to-azure-container-registry"></a>Envoyer l’image à Azure Container Registry

Ensuite, envoyez l’image conteneur *acr-helloworld* mise à jour dans votre registre géorépliqué. Ici, vous exécutez une seule commande `docker push` pour déployer l’image mise à jour sur les réplicas de registre, à la fois dans les régions *USA Ouest* et *USA Est*.

```bash
docker push <acrName>.azurecr.io/acr-helloworld:v1
```

Le résultat de `docker push` doit ressembler à ce qui suit :

```console
$ docker push uniqueregistryname.azurecr.io/acr-helloworld:v1
The push refers to a repository [uniqueregistryname.azurecr.io/acr-helloworld]
5b9454e91555: Pushed
d6803756744a: Layer already exists
b7b1f3a15779: Layer already exists
a89567dff12d: Layer already exists
59c7b561ff56: Layer already exists
9a2f9413d9e4: Layer already exists
a75caa09eb1f: Layer already exists
v1: digest: sha256:4c3f2211569346fbe2d1006c18cbea2a4a9dcc1eb3a078608cef70d3a186ec7a size: 1792
```

## <a name="view-the-webhook-logs"></a>Afficher les journaux d’activité du webhook

Pendant que l’image est en cours de réplication, vous pouvez voir les webhooks Azure Container Registry déclenchés.

Pour voir les webhooks régionaux qui ont été créés lorsque vous avez déployé le conteneur sur *Web Apps pour les conteneurs* dans un didacticiel précédent, accédez à votre registre de conteneurs dans le portail Azure, puis sélectionnez **Webhooks** sous **SERVICES**.

![Webhooks de registre de conteneurs dans le portail Azure][tutorial-portal-01]

Sélectionnez chaque webhook pour consulter l’historique des appels et des réponses. Vous devez voir une ligne pour l’action **push** dans les journaux d’activité des deux webhooks. Ici, le journal pour le webhook qui se trouve dans la région *USA Ouest* montre l’action **push** déclenchée par le `docker push` à l’étape précédente :

![Journal des webhooks de registre de conteneurs dans le portail Azure (USA Ouest)][tutorial-portal-02]

## <a name="view-the-updated-web-app"></a>Afficher l’application web mise à jour

Les webhooks notifient les applications web qu’une nouvelle image a été envoyée au registre, qui déploie automatiquement le conteneur mis à jour pour les deux applications web régionales.

Vérifiez que l’application a été mise à jour dans les deux déploiements en accédant aux deux déploiements régionaux dans votre navigateur web. En guise de rappel, vous pouvez trouver l’URL de l’application web déployée dans le coin supérieur droit de chaque onglet de la vue d’ensemble d’App Service.

![Vue d’ensemble de App Service dans le portail Azure][tutorial-portal-03]

Pour voir l’application mise à jour, sélectionnez le lien dans la vue d’ensemble d’App Service. Voici une vue d’exemple de l’application en cours d’exécution dans la région *USA Ouest* :

![Affichage du navigateur de l’application web modifiée en cours d’exécution dans la région USA Ouest][deployed-app-westus-modified]

Vérifiez que l’image de conteneur mise à jour a également été déployée sur le déploiement *USA Est* en l’affichant dans votre navigateur.

![Affichage du navigateur de l’application web modifiée en cours d’exécution dans la région USA Est][deployed-app-eastus-modified]

Avec une seule commande `docker push`, vous avez automatiquement mis à jour l’application web exécutée dans les deux déploiements d’application web régionaux. De plus, Azure Container Registry a distribué les images conteneurs des référentiels les plus proches de chaque déploiement.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez mis à jour et envoyé une nouvelle version du conteneur d’application web dans le registre géorépliqué. Les webhooks dans Azure Container Registry ont informé Web App pour conteneurs de la mise à jour, ce qui a déclenché une extraction locale à partir du réplica de registre le plus proche.

### <a name="acr-build-automated-image-build-and-patch"></a>ACR Build : génération d’images et correctifs automatisés

Outre la géoréplication, ACR Build est une autre fonctionnalité d’Azure Container Registry vous permettant d’optimiser votre pipeline de déploiement de conteneurs. Commencez par consulter la vue d’ensemble d’ACR Build pour connaître ses fonctionnalités dans les grandes lignes :

[Automatiser les mises à jour correctives du système d’exploitation et de l’infrastructure avec ACR Build](container-registry-tasks-overview.md)

<!-- IMAGES -->
[deployed-app-eastus-modified]: ./media/container-registry-tutorial-deploy-update/deployed-app-eastus-modified.png
[deployed-app-westus-modified]: ./media/container-registry-tutorial-deploy-update/deployed-app-westus-modified.png
[local-container-01]: ./media/container-registry-tutorial-deploy-update/local-container-01.png
[tutorial-portal-01]: ./media/container-registry-tutorial-deploy-update/tutorial-portal-01.png
[tutorial-portal-02]: ./media/container-registry-tutorial-deploy-update/tutorial-portal-02.png
[tutorial-portal-03]: ./media/container-registry-tutorial-deploy-update/tutorial-portal-03.png