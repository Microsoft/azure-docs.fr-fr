---
title: Guide de démarrage rapide - Créer un registre - Azure CLI
description: Apprenez rapidement à créer un registre de conteneurs Docker privé avec l’interface de ligne de commande Azure.
ms.topic: quickstart
ms.date: 06/12/2020
ms.custom: seodec18, H1Hack27Feb2017, mvc, devx-track-azurecli
ms.openlocfilehash: 226e50aec8f7c76a1b4c81d1a07d57583059ef0e
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96020073"
---
# <a name="quickstart-create-a-private-container-registry-using-the-azure-cli"></a>Démarrage rapide : Créer un registre de conteneurs privé avec Azure CLI

Azure Container Registry est un service de registre de conteneurs Docker géré utilisé pour stocker des images de conteneurs Docker privés. Ce guide décrit en détail la création d’une instance Azure Container Registry à l’aide de l’interface de ligne de commande Azure. Vous allez ensuite utiliser des commandes Docker pour envoyer (push) une image conteneur dans le registre, puis tirer (pull) et exécuter l’image à partir de votre registre.

Ce guide de démarrage rapide vous demande d’exécuter Azure CLI (version 2.0.55 ou ultérieure recommandée). Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI][azure-cli].

Docker doit également être installé en local. Docker fournit des packages qui le configurent facilement sur n’importe quel système [macOS][docker-mac], [Windows][docker-windows] ou [Linux][docker-linux].

Étant donné qu’Azure Cloud Shell n’inclut pas tous les composants Docker requis (par exemple, le démon `dockerd`), vous ne pouvez pas l’utiliser pour ce démarrage rapide.

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Créez un groupe de ressources avec la commande [az group create][az-group-create]. Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure sont déployées et gérées.

L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *eastus*.

```azurecli
az group create --name myResourceGroup --location eastus
```

## <a name="create-a-container-registry"></a>Créer un registre de conteneur

Dans ce guide de démarrage rapide, vous allez créer un registre *De base*. Il s’agit d’une option à coût optimisé pour les développeurs qui apprennent à se servir d’Azure Container Registry. Pour plus d’informations sur les niveaux de service disponibles, consultez [Niveaux de service des registres de conteneurs][container-registry-skus].

Créez une instance ACR à l’aide de la commande [az acr create][az-acr-create]. Le nom du registre doit être unique dans Azure et contenir entre 5 et 50 caractères alphanumériques. Dans l’exemple suivant, nous utilisons le nom *myContainerRegistry007*. Mettez à jour le nom de façon à utiliser une valeur unique.

```azurecli
az acr create --resource-group myResourceGroup \
  --name myContainerRegistry007 --sku Basic
```

Une fois le registre créé, la sortie ressemble à ce qui suit :

```json
{
  "adminUserEnabled": false,
  "creationDate": "2019-01-08T22:32:13.175925+00:00",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.ContainerRegistry/registries/myContainerRegistry007",
  "location": "eastus",
  "loginServer": "mycontainerregistry007.azurecr.io",
  "name": "myContainerRegistry007",
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
  "sku": {
    "name": "Basic",
    "tier": "Basic"
  },
  "status": null,
  "storageAccount": null,
  "tags": {},
  "type": "Microsoft.ContainerRegistry/registries"
}
```

Notez `loginServer` dans la sortie, qui est le nom complet du registre (tout en minuscules). Dans la suite et fin du présent guide de démarrage rapide, `<registry-name>` est un espace réservé pour le nom du registre de conteneurs et `<login-server>` est un espace réservé pour le nom du serveur de connexion du registre.

## <a name="log-in-to-registry"></a>Se connecter au registre

Avant d’envoyer (push) et de tirer (pull) des images conteneur, vous devez vous connecter au registre. Pour ce faire, utilisez la commande [az acr login][az-acr-login]. Spécifiez uniquement le nom du registre au moment de la connexion à Azure CLI. N’utilisez pas le nom du serveur de connexion, qui comprend un suffixe de domaine comme `azurecr.io`. 

```azurecli
az acr login --name <registry-name>
```

Exemple :

```azurecli
az acr login --name mycontainerregistry
```

Une fois l’opération terminée, la commande retourne un message `Login Succeeded`.

[!INCLUDE [container-registry-quickstart-docker-push](../../includes/container-registry-quickstart-docker-push.md)]

## <a name="list-container-images"></a>Répertorier les images conteneur

L’exemple suivant liste les référentiels de votre registre :

```azurecli
az acr repository list --name <registry-name> --output table
```

Sortie :

```
Result
----------------
hello-world
```

L’exemple suivant liste les balises existantes sur le dépôt **hello-world**.

```azurecli
az acr repository show-tags --name <registry-name> --repository hello-world --output table
```

Sortie :

```
Result
--------
v1
```

[!INCLUDE [container-registry-quickstart-docker-pull](../../includes/container-registry-quickstart-docker-pull.md)]

## <a name="clean-up-resources"></a>Nettoyer les ressources

Vous pouvez utiliser la commande [az group delete][az-group-delete] pour supprimer le groupe de ressources, le registre de conteneurs et les images conteneur stockées à cet endroit quand vous n’en avez plus besoin.

```azurecli
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé un registre de conteneurs Azure avec Azure CLI, envoyé (push) une image conteneur au registre, puis tiré (pull) et exécuté l’image à partir du registre. Passez à présent au tutoriel sur Azure Container Registry (ACR) pour approfondir vos connaissances.

> [!div class="nextstepaction"]
> [Tutoriels sur Azure Container Registry][container-registry-tutorial-prepare-registry]

> [!div class="nextstepaction"]
> [Tutoriels Azure Container Registry Tasks][container-registry-tutorial-quick-task]

<!-- LINKS - external -->
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-pull]: https://docs.docker.com/engine/reference/commandline/pull/
[docker-rmi]: https://docs.docker.com/engine/reference/commandline/rmi/
[docker-run]: https://docs.docker.com/engine/reference/commandline/run/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[docker-windows]: https://docs.docker.com/docker-for-windows/

<!-- LINKS - internal -->
[az-acr-create]: /cli/azure/acr#az-acr-create
[az-acr-login]: /cli/azure/acr#az-acr-login
[az-group-create]: /cli/azure/group#az-group-create
[az-group-delete]: /cli/azure/group#az-group-delete
[azure-cli]: /cli/azure/install-azure-cli
[container-registry-tutorial-quick-task]: container-registry-tutorial-quick-task.md
[container-registry-skus]: container-registry-skus.md
[container-registry-tutorial-prepare-registry]: container-registry-tutorial-prepare-registry.md
