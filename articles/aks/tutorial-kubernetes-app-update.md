---
title: Didacticiel Kubernetes sur Azure - Mettre à jour une application
description: Dans le cadre de ce didacticiel Azure Kubernetes Service (AKS), vous allez apprendre à mettre à jour un déploiement d’application existant vers ACS avec une nouvelle version du code d’application.
services: container-service
ms.topic: tutorial
ms.date: 09/30/2020
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: c8401a81a36d86b871df9fc428c393007b97c400
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94833909"
---
# <a name="tutorial-update-an-application-in-azure-kubernetes-service-aks"></a>Didacticiel : mettre à jour une application dans Azure Kubernetes Service (AKS)

Après avoir déployé une application dans Kubernetes, vous pouvez la mettre à jour en spécifiant une nouvelle image conteneur ou une nouvelle version de l’image. Une mise à jour se fait alors étape par étape, afin que seule une partie du déploiement soit mise à jour en même temps. Cette mise à jour progressive permet à l’application de poursuivre son exécution pendant la mise à jour. Elle fournit également un mécanisme de restauration en cas d’échec du déploiement.

Dans ce didacticiel (le sixième d’une série de sept), l’exemple de l’application Azure Vote est mis à jour. Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Mise à jour du code de l’application frontale
> * Création d’une image conteneur mise à jour
> * Envoi de l’image conteneur à Azure Container Registry
> * Déploiement de l’image conteneur mise à jour

## <a name="before-you-begin"></a>Avant de commencer

Dans les tutoriels précédents, une application a été empaquetée dans une image conteneur. Cette image a été chargée dans Azure Container Registry et vous avez créé un cluster AKS. L’application a ensuite été déployée sur le cluster AKS.

Un référentiel d’application a également été cloné, ce qui inclut le code source de l’application et un fichier Docker Compose préalablement créé utilisé dans ce didacticiel. Vérifiez que vous avez cloné le référentiel et changé des répertoires dans le répertoire cloné. Si vous n’avez pas effectué ces étapes et si vous souhaitez suivre cette procédure, commencez par [Tutoriel 1 : Créer des images conteneur][aks-tutorial-prepare-app].

Ce didacticiel nécessite l’exécution de l’interface de ligne de commande Azure CLI version 2.0.53 ou ultérieure. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI][azure-cli-install].

## <a name="update-an-application"></a>Mettre à jour une application

Apportons une modification à l’exemple d’application, puis mettons à jour la version déjà déployée dans votre cluster AKS. Veillez à être dans le répertoire *azure-voting-app-redis* cloné. Le code source de l’exemple d’application se trouve alors dans le répertoire *azure-vote*. Ouvrez le fichier *config_file.cfg* avec un éditeur tel que `vi` :

```console
vi azure-vote/azure-vote/config_file.cfg
```

Remplacez les valeurs de *VOTE1VALUE* et *VOTE2VALUE* par d’autres valeurs, comme des couleurs. L’exemple suivant présente les valeurs mise à jour :

```
# UI Configurations
TITLE = 'Azure Voting App'
VOTE1VALUE = 'Blue'
VOTE2VALUE = 'Purple'
SHOWHOST = 'false'
```

Enregistrez et fermez le fichier. Dans `vi`, utilisez `:wq`.

## <a name="update-the-container-image"></a>Mettre à jour l’image conteneur

Pour recréer l’image front-end et tester l’application mise à jour, utilisez [docker-compose][docker-compose]. L’argument `--build` est utilisé pour indiquer à Docker Compose de recréer l’image d’application :

```console
docker-compose up --build -d
```

## <a name="test-the-application-locally"></a>Tester localement l’application

Pour vous assurer que l’image conteneur mise à jour intègre vos modifications, ouvrez un navigateur web local et accédez à l’adresse `http://localhost:8080`.

:::image type="content" source="media/container-service-kubernetes-tutorials/vote-app-updated.png" alt-text="Capture d’écran montrant un exemple d’image conteneur mise à jour - Application de vote Azure ouverte avec un navigateur web local et un hôte local.":::

Les valeurs mises à jour fournies dans le fichier *config_file.cfg* s’affichent dans votre application en cours d’exécution.

## <a name="tag-and-push-the-image"></a>Marquer et envoyer l’image

Pour utiliser correctement l’image mise à jour, balisez l’image *azure-vote-front* avec le nom du serveur de connexion de votre registre ACR. Obtenez le nom du serveur de connexion à l’aide de la commande [az acr list](/cli/azure/acr) :

```azurecli
az acr list --resource-group myResourceGroup --query "[].{acrLoginServer:loginServer}" --output table
```

Utilisez [docker tag][docker-tag] pour ajouter une balise à l’image. Remplacez `<acrLoginServer>` par le nom de votre serveur de connexion ACR ou par votre nom d’hôte du registre public, puis mettez à jour la version de l’image vers *:v2*, comme suit :

```console
docker tag mcr.microsoft.com/azuredocs/azure-vote-front:v1 <acrLoginServer>/azure-vote-front:v2
```

À présent, utilisez [docker push][docker-push] pour charger l’image dans votre registre. Remplacez `<acrLoginServer>` par le nom de votre serveur de connexion ACR.

> [!NOTE]
> Si vous rencontrez des problèmes en exécutant un push vers votre registre ACR, vérifiez que vous êtes toujours connecté. Exécutez la commande [az acr login][az-acr-login] en utilisant le nom du registre Azure Container Registry que vous avez créé à l’étape [Créer un registre Azure Container Registry](tutorial-kubernetes-prepare-acr.md#create-an-azure-container-registry). Par exemple : `az acr login --name <azure container registry name>`.

```console
docker push <acrLoginServer>/azure-vote-front:v2
```

## <a name="deploy-the-updated-application"></a>Déployer l’application mise à jour

Pour fournir une disponibilité maximale, vous devez exécuter plusieurs instances du pod d’application. Vérifiez le nombre d’instances front-end en cours d’exécution avec la commande [kubectl get pods][kubectl-get] :

```
$ kubectl get pods

NAME                               READY     STATUS    RESTARTS   AGE
azure-vote-back-217588096-5w632    1/1       Running   0          10m
azure-vote-front-233282510-b5pkz   1/1       Running   0          10m
azure-vote-front-233282510-dhrtr   1/1       Running   0          10m
azure-vote-front-233282510-pqbfk   1/1       Running   0          10m
```

Si vous ne disposez pas de plusieurs pods de front-end, mettez à l’échelle le déploiement *azure-vote-front* comme suit :

```console
kubectl scale --replicas=3 deployment/azure-vote-front
```

Pour mettre à jour l’application, utilisez la commande [kubectl set][kubectl-set]. Mettez à jour `<acrLoginServer>` avec le nom du serveur de connexion ou le nom d’hôte de votre registre de conteneurs, puis spécifiez la version d’application *v2* :

```console
kubectl set image deployment azure-vote-front azure-vote-front=<acrLoginServer>/azure-vote-front:v2
```

Pour surveiller le déploiement, utilisez la commande [kubectl get pod][kubectl-get]. À mesure que l’application mise à jour est déployée, vos pods sont terminés et recréés avec la nouvelle image conteneur.

```console
kubectl get pods
```

L’exemple de sortie ci-après présente la terminaison des pods et les nouvelles instances en cours d’exécution à mesure que le déploiement progresse :

```
$ kubectl get pods

NAME                               READY     STATUS        RESTARTS   AGE
azure-vote-back-2978095810-gq9g0   1/1       Running       0          5m
azure-vote-front-1297194256-tpjlg  1/1       Running       0          1m
azure-vote-front-1297194256-tptnx  1/1       Running       0          5m
azure-vote-front-1297194256-zktw9  1/1       Terminating   0          1m
```

## <a name="test-the-updated-application"></a>Tester l’application mise à jour

Pour visualiser l’application mise à jour, commencez par obtenir l’adresse IP externe du service `azure-vote-front` :

```console
kubectl get service azure-vote-front
```

À présent, ouvrez un navigateur web local et accédez à l’adresse IP de votre service :

:::image type="content" source="media/container-service-kubernetes-tutorials/vote-app-updated-external.png" alt-text="Capture d’écran montrant un exemple de l’application mise à jour - Application de vote Azure ouverte dans un navigateur web local.":::

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, nous avons mis à jour une application et nous avons déployé cette mise à jour sur votre cluster AKS. Vous avez appris à :

> [!div class="checklist"]
> * Mise à jour du code de l’application frontale
> * Création d’une image conteneur mise à jour
> * Envoi de l’image conteneur à Azure Container Registry
> * Déploiement de l’image conteneur mise à jour

Passez au didacticiel suivant pour découvrir comment mettre à niveau un cluster AKS vers une nouvelle version de Kubernetes.

> [!div class="nextstepaction"]
> [Mettre à niveau Kubernetes][aks-tutorial-upgrade]

<!-- LINKS - external -->
[docker-compose]: https://docs.docker.com/compose/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-set]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#set

<!-- LINKS - internal -->
[aks-tutorial-prepare-app]: ./tutorial-kubernetes-prepare-app.md
[aks-tutorial-upgrade]: ./tutorial-kubernetes-upgrade-cluster.md
[az-acr-login]: /cli/azure/acr
[azure-cli-install]: /cli/azure/install-azure-cli
