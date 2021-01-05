---
title: Didacticiel Kubernetes sur Azure - Déployer une application
description: Dans le cadre de ce didacticiel Azure Kubernetes Service (AKS), vous allez déployer une application multiconteneur dans votre cluster à l’aide d’une image personnalisée stockée dans Azure Container Registry.
services: container-service
ms.topic: tutorial
ms.date: 09/30/2020
ms.custom: mvc
ms.openlocfilehash: 8114aa0b6c2483d543376727a44d14041ed02b37
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91576487"
---
# <a name="tutorial-run-applications-in-azure-kubernetes-service-aks"></a>Didacticiel : Exécuter des applications dans Azure Kubernetes Service (ACS)

Kubernetes fournit une plateforme distribuée destinée aux applications en conteneur. Vous générez et déployez vos propres applications et services dans un cluster Kubernetes, et vous laissez le cluster gérer la disponibilité et la connectivité. Dans ce didacticiel (le quatrième d’une série de sept), un exemple d’application est déployé dans un cluster Kubernetes. Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Mettre à jour un fichier manifeste Kubernetes
> * Exécution d’une application dans Kubernetes
> * Test de l’application

Dans les tutoriels suivants, cette application fait l’objet d’un scale-out et est mise à jour.

Ce guide de démarrage rapide suppose une compréhension élémentaire des concepts liés à Kubernetes. Pour plus d’informations, consultez [Concepts de base de Kubernetes pour AKS (Azure Kubernetes Service)][kubernetes-concepts].

## <a name="before-you-begin"></a>Avant de commencer

Dans les didacticiels précédents, une application a été empaquetée dans une image conteneur, l’image a été chargée dans Azure Container Registry et un cluster Kubernetes a été créé.

Pour effectuer ce didacticiel, vous avez besoin du fichier manifeste Kubernetes `azure-vote-all-in-one-redis.yaml`. Ce fichier a été téléchargé avec le code source de l’application dans un didacticiel précédent. Vérifiez que vous avez cloné le référentiel et changé des répertoires dans le référentiel cloné. Si vous n’avez pas effectué ces étapes et si vous souhaitez suivre cette procédure, commencez par [Tutoriel 1 : Créer des images conteneur][aks-tutorial-prepare-app].

Ce didacticiel nécessite l’exécution de l’interface de ligne de commande Azure CLI version 2.0.53 ou ultérieure. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI][azure-cli-install].

## <a name="update-the-manifest-file"></a>Mettre à jour le fichier manifeste

Dans ces didacticiels, une instance Azure Container Registry (ACR) stocke l’image conteneur de l’exemple d’application. Pour déployer l’application, vous devez mettre à jour le nom de l’image dans le fichier manifeste Kubernetes afin d’inclure le nom du serveur de connexion ACR.

Obtenez le nom du serveur de connexion ACR à l’aide de la commande [az acr list][az-acr-list], comme suit :

```azurecli
az acr list --resource-group myResourceGroup --query "[].{acrLoginServer:loginServer}" --output table
```

L’exemple de fichier manifeste provenant du référentiel git cloné dans le premier didacticiel utilise le nom de serveur de connexion *microsoft*. Veillez à être dans le répertoire *azure-voting-app-redis* cloné, puis ouvrez le fichier manifeste avec un éditeur de texte, comme `vi` :

```console
vi azure-vote-all-in-one-redis.yaml
```

Remplacez *microsoft* par le nom de votre serveur de connexion ACR. Le nom de l’image figure à la ligne 51 du fichier manifeste. L’exemple ci-après indique le nom de l’image par défaut :

```yaml
containers:
- name: azure-vote-front
  image: mcr.microsoft.com/azuredocs/azure-vote-front:v1
```

Fournissez le nom de votre propre serveur de connexion ACR afin que votre fichier manifeste ressemble à ceci :

```yaml
containers:
- name: azure-vote-front
  image: <acrName>.azurecr.io/azure-vote-front:v1
```

Enregistrez et fermez le fichier. Dans `vi`, utilisez `:wq`.

## <a name="deploy-the-application"></a>Déployer l’application

Pour déployer votre application, utilisez la commande [kubectl apply][kubectl-apply]. Cette commande analyse le fichier manifeste et crée les objets Kubernetes définis. Spécifiez l’exemple de fichier manifeste, comme indiqué dans l’exemple suivant :

```console
kubectl apply -f azure-vote-all-in-one-redis.yaml
```

L’exemple de sortie suivant montre les ressources correctement créées dans le cluster AKS :

```
$ kubectl apply -f azure-vote-all-in-one-redis.yaml

deployment "azure-vote-back" created
service "azure-vote-back" created
deployment "azure-vote-front" created
service "azure-vote-front" created
```

## <a name="test-the-application"></a>Test de l’application

Quand l’application s’exécute, un service Kubernetes expose le front-end de l’application sur Internet. L’exécution de ce processus peut prendre plusieurs minutes.

Pour surveiller la progression, utilisez la commande [kubectl get service][kubectl-get] avec l’argument `--watch`.

```console
kubectl get service azure-vote-front --watch
```

Dans un premier temps, la valeur *EXTERNAL-IP* du service *azure-vote-front* est indiqué comme étant en attente (*pending*) :

```
azure-vote-front   LoadBalancer   10.0.34.242   <pending>     80:30676/TCP   5s
```

Quand l’adresse *EXTERNAL-IP* passe de l’état *pending* à une adresse IP publique réelle, utilisez `CTRL-C` pour arrêter le processus de surveillance `kubectl`. L’exemple de sortie suivant montre une adresse IP publique valide affectée au service :

```
azure-vote-front   LoadBalancer   10.0.34.242   52.179.23.131   80:30676/TCP   67s
```

Pour voir l’application en action, ouvrez un navigateur web en utilisant l’adresse IP externe de votre service :

![Image du cluster Kubernetes sur Azure](media/container-service-kubernetes-tutorials/azure-vote.png)

Si l’application ne s’est pas chargée, il y a peut-être un problème d’autorisation avec votre registre d’images. Pour visualiser l’état de vos conteneurs, utilisez la commande `kubectl get pods`. Si les images conteneur ne peuvent pas être extraites, consultez [S’authentifier auprès d’Azure Container Registry à partir d’Azure Kubernetes Service](cluster-container-registry-integration.md).

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, l’exemple d’application de vote Azure a été déployée sur un cluster Kubernetes dans AKS. Vous avez appris à :

> [!div class="checklist"]
> * Mise à jour d’un fichier manifeste Kubernetes
> * Exécution d’une application dans Kubernetes
> * Test de l’application

Passez au didacticiel suivant pour découvrir comment mettre à l’échelle une application Kubernetes et l’infrastructure Kubernetes sous-jacente.

> [!div class="nextstepaction"]
> [Mettre à l’échelle l’application et l’infrastructure Kubernetes][aks-tutorial-scale]

<!-- LINKS - external -->
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-create]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#create
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get

<!-- LINKS - internal -->
[aks-tutorial-prepare-app]: ./tutorial-kubernetes-prepare-app.md
[aks-tutorial-scale]: ./tutorial-kubernetes-scale.md
[az-acr-list]: /cli/azure/acr
[azure-cli-install]: /cli/azure/install-azure-cli
[kubernetes-concepts]: concepts-clusters-workloads.md
[kubernetes-service]: concepts-network.md#services
