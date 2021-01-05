---
title: Tutoriel - Déployer une application de conteneur sur une instance de conteneur
description: Tutoriel Azure Container Instances (partie 3 sur 3) - Déployer une application conteneur sur Azure Container Instances
ms.topic: tutorial
ms.date: 03/21/2018
ms.custom: seodec18, mvc, devx-track-azurecli
ms.openlocfilehash: 2ea3d285f00d38df84587d9a7c15242fff38453b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87500570"
---
# <a name="tutorial-deploy-a-container-application-to-azure-container-instances"></a>Tutoriel : Déployer une application conteneur sur Azure Container Instances

Il s’agit du didacticiel final d’une série en trois parties. Dans les deux premières parties, [nous avons créé une image conteneur](container-instances-tutorial-prepare-app.md) et [nous l’avons envoyée à Azure Container Registry](container-instances-tutorial-prepare-acr.md). Cet article termine la série en déployant le conteneur sur Azure Container Instances.

Dans ce tutoriel, vous allez :

> [!div class="checklist"]
> * Déployer le conteneur à partir d’Azure Container Registry vers Azure Container Instances
> * Afficher l’application en cours d’exécution dans le navigateur
> * Afficher les journaux d’activité du conteneur

## <a name="before-you-begin"></a>Avant de commencer

[!INCLUDE [container-instances-tutorial-prerequisites](../../includes/container-instances-tutorial-prerequisites.md)]

## <a name="deploy-the-container-using-the-azure-cli"></a>Déployer le conteneur à l’aide de l’interface CLI Azure

Dans cette section, vous utilisez l’interface Azure CLI pour déployer l’image générée dans le [premier didacticiel](container-instances-tutorial-prepare-app.md) et envoyée à Azure Container Registry dans le [deuxième didacticiel](container-instances-tutorial-prepare-acr.md). Assurez-vous d’avoir terminé ces didacticiels avant de continuer.

### <a name="get-registry-credentials"></a>Obtenir les informations d’identification du registre

Lorsque vous déployez une image qui est hébergée dans un registre de conteneurs Azure privé, tel que celui créé dans le [deuxième tutoriel](container-instances-tutorial-prepare-acr.md), vous devez fournir les informations d’identification pour accéder au registre. 

Une bonne pratique à suivre dans de nombreux scénarios consiste à créer et à configurer un principal de service Azure Active Directory avec des autorisations d’accès en *extraction* (pull) à votre registre. Consultez [S’authentifier avec Azure Container Registry à partir d’Azure Container Instances](../container-registry/container-registry-auth-aci.md) pour obtenir des exemples de scripts afin de créer un principal de service doté des autorisations nécessaires. Notez l’*ID du principal de service* et le *mot de passe du principal de service*. Vous utilisez ces informations d’identification pour accéder au registre lorsque vous déployez le conteneur.

Vous avez également besoin du nom complet du serveur de connexion au registre de conteneurs (remplacez `<acrName>` par le nom de votre registre) :

```azurecli
az acr show --name <acrName> --query loginServer
```

### <a name="deploy-container"></a>Déployer le conteneur

Maintenant, utilisez la commande [az container create][az-container-create] pour déployer le conteneur. Remplacez `<acrLoginServer>` par les valeurs obtenues à partir de la commande précédente. Remplacez `<service-principal-ID>` et `<service-principal-password>` par l’ID et le mot de passe de principal de service que vous avez créés pour accéder au registre. Remplacez `<aciDnsLabel>` par un nom DNS souhaité.

```azurecli
az container create --resource-group myResourceGroup --name aci-tutorial-app --image <acrLoginServer>/aci-tutorial-app:v1 --cpu 1 --memory 1 --registry-login-server <acrLoginServer> --registry-username <service-principal-ID> --registry-password <service-principal-password> --dns-name-label <aciDnsLabel> --ports 80
```

Après quelques secondes, vous devriez recevoir une réponse initiale d’Azure. La valeur `--dns-name-label` doit être unique au sein de la région Azure dans laquelle vous créez l’instance de conteneur. Modifiez la valeur de la commande précédente si vous recevez un message d’erreur **Étiquette de nom DNS** lorsque vous exécutez la commande.

### <a name="verify-deployment-progress"></a>Vérifier la progression du déploiement

Pour afficher l’état du déploiement, utilisez la commande [az container show][az-container-show] :

```azurecli
az container show --resource-group myResourceGroup --name aci-tutorial-app --query instanceView.state
```

Répétez la commande [az container show][az-container-show] jusqu’à ce que l’état passe de *En attente* à *En cours d’exécution*, ce qui prend normalement moins d’une minute. Lorsque le conteneur est *En cours d’exécution*, passez à l’étape suivante.

## <a name="view-the-application-and-container-logs"></a>Afficher les journaux d’activité d’applications et de conteneurs

Une fois le déploiement réussi, affichez le nom de domaine complet du conteneur avec la commande [az container show][az-container-show] :

```azurecli
az container show --resource-group myResourceGroup --name aci-tutorial-app --query ipAddress.fqdn
```

Par exemple :
```output
"aci-demo.eastus.azurecontainer.io"
```

Pour afficher l’application en cours d’exécution, accédez au nom DNS affiché dans votre navigateur favori :

![Application Hello world dans le navigateur][aci-app-browser]

Vous pouvez également afficher la sortie du journal du conteneur :

```azurecli
az container logs --resource-group myResourceGroup --name aci-tutorial-app
```

Exemple de sortie :

```output
listening on port 80
::ffff:10.240.0.4 - - [21/Jul/2017:06:00:02 +0000] "GET / HTTP/1.1" 200 1663 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.115 Safari/537.36"
::ffff:10.240.0.4 - - [21/Jul/2017:06:00:02 +0000] "GET /favicon.ico HTTP/1.1" 404 150 "http://aci-demo.eastus.azurecontainer.io/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.115 Safari/537.36"
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous n’avez plus besoin des ressources que vous avez créées dans cette série de didacticiels, vous pouvez exécuter la commande [az group delete][az-group-delete] pour supprimer le groupe de ressources et toutes les ressources qu’il contient. Cette commande supprime le registre de conteneurs que vous avez créé, ainsi que le conteneur en cours d’exécution et toutes les ressources associées.

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez terminé le processus de déploiement de votre conteneur sur Azure Container Instances. Les étapes suivantes ont été effectuées :

> [!div class="checklist"]
> * Déployer le conteneur à partir d’Azure Container Registry à l’aide d’Azure CLI
> * afficher l’application dans le navigateur ;
> * afficher les journaux d’activité du conteneur.

Maintenant que vous avez les bases, continuez pour en savoir plus sur Azure Container Instances, par exemple sur le fonctionnement des groupes de conteneurs :

> [!div class="nextstepaction"]
> [Groupes de conteneurs dans Azure Container Instances](container-instances-container-groups.md)

<!-- IMAGES -->
[aci-app-browser]: ./media/container-instances-quickstart/aci-app-browser.png

<!-- LINKS - external -->
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-login]: https://docs.docker.com/engine/reference/commandline/login/
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[docker-windows]: https://docs.docker.com/docker-for-windows/

<!-- LINKS - internal -->
[az-container-create]: /cli/azure/container#az-container-create
[az-container-show]: /cli/azure/container#az-container-show
[az-group-delete]: /cli/azure/group#az-group-delete
[azure-cli-install]: /cli/azure/install-azure-cli
[prepare-app]: ./container-instances-tutorial-prepare-app.md
