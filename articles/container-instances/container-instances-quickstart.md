---
title: Guide de démarrage rapide – Déployer un conteneur Docker dans une instance de conteneur – Azure CLI
description: Dans ce guide de démarrage rapide, vous utilisez le Azure CLI pour déployer rapidement une application web conteneurisée qui s’exécute dans une instance de conteneur Azure isolé.
ms.topic: quickstart
ms.date: 03/21/2019
ms.custom:
- seo-python-october2019
- seodec18
- mvc
- devx-track-js
- devx-track-azurecli
ms.openlocfilehash: 1c327fc7fc067948b5022f989e6c86f99573bd1a
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93100182"
---
# <a name="quickstart-deploy-a-container-instance-in-azure-using-the-azure-cli"></a>Démarrage rapide : Déployer un instance de conteneur dans Azure à l’aide d’Azure CLI

Utilisez Azure Container Instances pour exécuter, facilement et rapidement, des conteneurs Docker serverless dans Azure. Déployez une application sur une instance de conteneur à la demande lorsque vous n’avez pas besoin d’une plateforme d’orchestration de conteneur complète telle qu’Azure Kubernetes Service.

Dans cette procédure de démarrage rapide, vous utilisez Azure CLI pour déployer un conteneur Docker isolé et mettre son application à disposition avec un nom de domaine complet (FQDN). Quelques secondes après l’exécution d’une seule commande de déploiement, vous pouvez accéder à l’application en cours d’exécution dans le conteneur :

![Afficher une application déployée sur Azure Container Instances dans le navigateur][aci-app-browser]

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Pour ce guide de démarrage rapide, vous devez avoir la version 2.0.55 ou ultérieure de l’interface Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Les instances de conteneur Azure, comme toutes les ressources Azure, doivent être déployées dans un groupe de ressources. Les groupes de ressources vous permettent d’organiser et de gérer les ressources Azure connexes.

Commencez par créer un groupe de ressources nommé *myResourceGroup* à l’emplacement *eastus* à l’aide de la commande [az group create][az-group-create] suivante :

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

## <a name="create-a-container"></a>Créer un conteneur

Maintenant que vous avez un groupe de ressources, vous pouvez exécuter un conteneur dans Azure. Pour créer une instance de conteneur avec Azure CLI, fournissez un nom de groupe de ressources, un nom d’instance de conteneur et l’image conteneur Docker à la commande [az container create][az-container-create]. Dans ce démarrage rapide, vous utilisez l’image `mcr.microsoft.com/azuredocs/aci-helloworld` publique. Cette image contient une petite application web écrite en Node.js qui sert une page HTML statique.

Vous pouvez exposer vos conteneurs sur Internet en spécifiant un ou plusieurs ports à ouvrir, une étiquette de nom DNS ou les deux. Dans ce guide de démarrage rapide, vous déployez un conteneur avec une étiquette de nom DNS pour que l’application web soit publiquement accessible.

Pour démarrer une instance de conteneur, exécutez une commande similaire à la suivante. Définissez une valeur `--dns-name-label` être unique au sein de la région Azure dans laquelle vous créez l’instance. Si vous recevez un message d’erreur « Étiquette de nom DNS indisponible », essayez d’utiliser une autre étiquette de nom DNS.

```azurecli-interactive
az container create --resource-group myResourceGroup --name mycontainer --image mcr.microsoft.com/azuredocs/aci-helloworld --dns-name-label aci-demo --ports 80
```

Après quelques secondes, vous devez recevoir une réponse d’Azure CLI indiquant que le déploiement est terminé. Vérifiez son état à l’aide de la commande [az container show][az-container-show] :

```azurecli-interactive
az container show --resource-group myResourceGroup --name mycontainer --query "{FQDN:ipAddress.fqdn,ProvisioningState:provisioningState}" --out table
```

Lorsque vous exécutez la commande, le nom de domaine complet du conteneur et son état d’approvisionnement s’affichent.

```output
FQDN                               ProvisioningState
---------------------------------  -------------------
aci-demo.eastus.azurecontainer.io  Succeeded
```

Si l’élément `ProvisioningState` du conteneur est défini sur **Réussite** , accédez à son nom de domaine complet dans votre navigateur. Si vous voyez une page web similaire à l’image suivante, félicitations ! Cela signifie que vous avez réussi à déployer une application s’exécutant dans un conteneur Docker sur Azure.

![Afficher une application déployée sur Azure Container Instances dans le navigateur][aci-app-browser]

Si l’application n’apparaît pas dans un premier temps, vous devrez peut-être patienter quelques secondes, le temps que le DNS se propage, puis essayez d’actualiser votre navigateur.

## <a name="pull-the-container-logs"></a>Extraire les journaux d’activité de conteneur

Lorsque vous avez besoin de résoudre les problèmes relatifs à un conteneur ou à l’application qu’il exécute (ou simplement afficher son résultat), commencez par consulter les journaux d’activité de l’instance de conteneur.

Extrayez les journaux d’instance de conteneur avec la commande [az container logs][az-container-logs] :

```azurecli-interactive
az container logs --resource-group myResourceGroup --name mycontainer
```

La sortie affiche les journaux d’activité du conteneur et, normalement, les requêtes HTTP GET générées lorsque vous avez affiché l’application dans votre navigateur.

```output
listening on port 80
::ffff:10.240.255.55 - - [21/Mar/2019:17:43:53 +0000] "GET / HTTP/1.1" 304 - "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36"
::ffff:10.240.255.55 - - [21/Mar/2019:17:44:36 +0000] "GET / HTTP/1.1" 304 - "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36"
::ffff:10.240.255.55 - - [21/Mar/2019:17:44:36 +0000] "GET / HTTP/1.1" 304 - "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36"
```

## <a name="attach-output-streams"></a>Joindre des flux de sortie

En plus de consulter les journaux d’activité, vous pouvez joindre vos flux de sortie locale standard et d’erreur standard à ceux du conteneur.

Exécutez d’abord la commande [az container attach][az-container-attach] pour joindre votre console locale aux flux de sortie du conteneur :

```azurecli-interactive
az container attach --resource-group myResourceGroup --name mycontainer
```

Une fois joints, actualisez votre navigateur quelques fois pour générer des sorties supplémentaires. Ensuite, détachez votre console avec la commande `Control+C`. Vous devez obtenir une sortie similaire à la suivante :

```output
Container 'mycontainer' is in state 'Running'...
(count: 1) (last timestamp: 2019-03-21 17:27:20+00:00) pulling image "mcr.microsoft.com/azuredocs/aci-helloworld"
(count: 1) (last timestamp: 2019-03-21 17:27:24+00:00) Successfully pulled image "mcr.microsoft.com/azuredocs/aci-helloworld"
(count: 1) (last timestamp: 2019-03-21 17:27:27+00:00) Created container
(count: 1) (last timestamp: 2019-03-21 17:27:27+00:00) Started container

Start streaming logs:
listening on port 80

::ffff:10.240.255.55 - - [21/Mar/2019:17:43:53 +0000] "GET / HTTP/1.1" 304 - "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36"
::ffff:10.240.255.55 - - [21/Mar/2019:17:44:36 +0000] "GET / HTTP/1.1" 304 - "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36"
::ffff:10.240.255.55 - - [21/Mar/2019:17:44:36 +0000] "GET / HTTP/1.1" 304 - "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36"
::ffff:10.240.255.55 - - [21/Mar/2019:17:47:01 +0000] "GET / HTTP/1.1" 304 - "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36"
::ffff:10.240.255.56 - - [21/Mar/2019:17:47:12 +0000] "GET / HTTP/1.1" 304 - "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36"
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Dès que vous avez fini d’utiliser le conteneur, supprimez-le à l’aide de la commande [az container delete][az-container-delete] :

```azurecli-interactive
az container delete --resource-group myResourceGroup --name mycontainer
```

Pour vérifier que le conteneur a été supprimé, exécutez la commande [az container list](/cli/azure/container#az-container-list) :

```azurecli-interactive
az container list --resource-group myResourceGroup --output table
```

Le conteneur **mycontainer** ne doit pas apparaître dans la sortie de la commande. Si vous ne disposez d’aucun autre conteneur dans le groupe de ressources, aucune sortie ne s’affiche.

Si vous en avez terminé avec le groupe de ressources *myResourceGroup* et toutes les ressources qu’il contient, supprimez-le avec la commande [az group delete][az-group-delete] :

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé une instance de conteneur Azure à l’aide d’une image Microsoft publique. Si vous voulez créer une image conteneur et la déployer à partir d’un registre de conteneurs Azure privé, passez au didacticiel Azure Container Instances.

> [!div class="nextstepaction"]
> [Didacticiel Azure Container Instances](./container-instances-tutorial-prepare-app.md)

Pour tester les options d’exécution des conteneurs dans un système d’orchestration sur Azure, consultez les guides de démarrage rapide [Azure Kubernetes Service (AKS)][container-service].

<!-- IMAGES -->
[aci-app-browser]: ./media/container-instances-quickstart/view-an-application-running-in-an-azure-container-instance.png

<!-- LINKS - External -->
[app-github-repo]: https://github.com/Azure-Samples/aci-helloworld.git
[azure-account]: https://azure.microsoft.com/free/
[node-js]: https://nodejs.org

<!-- LINKS - Internal -->
[az-container-attach]: /cli/azure/container#az-container-attach
[az-container-create]: /cli/azure/container#az-container-create
[az-container-delete]: /cli/azure/container#az-container-delete
[az-container-list]: /cli/azure/container#az-container-list
[az-container-logs]: /cli/azure/container#az-container-logs
[az-container-show]: /cli/azure/container#az-container-show
[az-group-create]: /cli/azure/group#az-group-create
[az-group-delete]: /cli/azure/group#az-group-delete
[azure-cli-install]: /cli/azure/install-azure-cli
[container-service]: ../aks/kubernetes-walkthrough.md
