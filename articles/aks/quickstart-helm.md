---
title: Développer sur Azure Kubernetes Service (AKS) avec Helm
description: Utilisez Helm avec AKS et Azure Container Registry pour empaqueter et exécuter des conteneurs d’applications dans un cluster.
services: container-service
author: zr-msft
ms.topic: article
ms.date: 07/28/2020
ms.author: zarhoads
ms.openlocfilehash: 2c4e39ae664c89e4f6d49f2941115e5a7d44cf7b
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96929171"
---
# <a name="quickstart-develop-on-azure-kubernetes-service-aks-with-helm"></a>Démarrage rapide : Développer sur Azure Kubernetes Service (AKS) avec Helm

[Helm][helm] est un outil d’empaquetage open source qui vous aide à installer et à gérer le cycle de vie d’applications Kubernetes. À l’instar de gestionnaires de package Linux tels que *APT* et *Yum*, Helm sert à gérer les graphiques Kubernetes, qui sont des packages de ressources Kubernetes préconfigurés.

Cet article montre comment utiliser Helm pour empaqueter et exécuter une application sur AKS. Pour plus d’informations sur l’installation d’une application existante à l’aide de Helm, consultez [Installer des applications existantes avec Helm dans AKS][helm-existing].

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, vous pouvez créer un [compte gratuit](https://azure.microsoft.com/free).
* [Azure CLI](/cli/azure/install-azure-cli).
* [Helm v3 installé][helm-install].

## <a name="create-an-azure-container-registry"></a>Création d’un Azure Container Registry
Pour utiliser Helm afin d’exécuter votre application dans votre cluster AKS, vous avez besoin d’un registre Azure Container Registry pour stocker vos images conteneur. L’exemple ci-dessous utilise [az acr create][az-acr-create] pour créer un ACR nommé *MyHelmACR* dans le groupe de ressources *MyResourceGroup* avec la référence SKU *De base*. Vous devez indiquer votre nom de registre unique. Le nom du registre doit être unique dans Azure et contenir entre 5 et 50 caractères alphanumériques. La référence SKU *De base* est un point d’entrée au coût optimisé fourni à des fins de développement qui offre un bon équilibre entre stockage et débit.

```azurecli
az group create --name MyResourceGroup --location eastus
az acr create --resource-group MyResourceGroup --name MyHelmACR --sku Basic
```

Le résultat ressemble à l’exemple qui suit. Notez la valeur *loginServer* pour votre ACR, car elle sera utilisée dans une étape ultérieure. Dans l’exemple ci-dessous, *myhelmacr.azurecr.io* est la valeur *loginServer* pour *MyHelmACR*.

```console
{
  "adminUserEnabled": false,
  "creationDate": "2019-06-11T13:35:17.998425+00:00",
  "id": "/subscriptions/<ID>/resourceGroups/MyResourceGroup/providers/Microsoft.ContainerRegistry/registries/MyHelmACR",
  "location": "eastus",
  "loginServer": "myhelmacr.azurecr.io",
  "name": "MyHelmACR",
  "networkRuleSet": null,
  "provisioningState": "Succeeded",
  "resourceGroup": "MyResourceGroup",
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

## <a name="create-an-azure-kubernetes-service-cluster"></a>Créer un cluster Azure Kubernetes Service

Créez un cluster AKS. La commande ci-dessous crée un cluster AKS appelé MyAKS et y joint MyHelmACR.

```azurecli
az aks create -g MyResourceGroup -n MyAKS --location eastus  --attach-acr MyHelmACR --generate-ssh-keys
```

Votre cluster AKS doit accéder à votre ACR pour extraire les images conteneur et les exécuter. La commande ci-dessus accorde également au cluster *MyAKS* l’accès à votre ACR *MyHelmACR*.

## <a name="connect-to-your-aks-cluster"></a>Se connecter à votre cluster AKS

Pour vous connecter au cluster Kubernetes à partir de votre ordinateur local, utilisez [kubectl][kubectl], le client de ligne de commande Kubernetes.

Si vous utilisez Azure Cloud Shell, `kubectl` est déjà installé. Vous pouvez également l’installer en local à l’aide de la commande [az aks install-cli][] :

```azurecli
az aks install-cli
```

Pour configurer `kubectl` afin de vous connecter à votre cluster Kubernetes, exécutez la commande [az aks get-credentials][]. L’exemple suivant obtient les informations d’identification du cluster AKS nommé *myAKSCluster* dans *MyResourceGroup* :

```azurecli
az aks get-credentials --resource-group MyResourceGroup --name MyAKS
```

## <a name="download-the-sample-application"></a>Téléchargement de l'exemple d'application

Ce guide de démarrage rapide utilise [un exemple d’application Node.js à partir de l’exemple de référentiel Azure Dev Spaces][example-nodejs]. Clonez l’application à partir de GitHub, puis accédez au répertoire `dev-spaces/samples/nodejs/getting-started/webfrontend`.

```console
git clone https://github.com/Azure/dev-spaces
cd dev-spaces/samples/nodejs/getting-started/webfrontend
```

## <a name="create-a-dockerfile"></a>Créer un Dockerfile

Créez un fichier *Dockerfile* à l’aide des éléments suivants :

```dockerfile
FROM node:latest

WORKDIR /webfrontend

COPY package.json ./

RUN npm install

COPY . .

EXPOSE 80
CMD ["node","server.js"]
```

## <a name="build-and-push-the-sample-application-to-the-acr"></a>Créer et envoyer (push) l’exemple d’application vers l’ACR

Utilisez la commande [az acr build][az-acr-build] pour générer et envoyer (push) une image au registre en utilisant le fichier Dockerfile précédent. Le `.` à la fin de la commande définit l’emplacement du fichier Dockerfile, dans ce cas, le répertoire actif.

```azurecli
az acr build --image webfrontend:v1 \
  --registry MyHelmACR \
  --file Dockerfile .
```

## <a name="create-your-helm-chart"></a>Créer votre chart Helm

Générez votre chart Helm à l’aide de la commande `helm create`.

```console
helm create webfrontend
```

Effectuez les mises à jour suivantes de *webfrontend/values.yaml*. Remplacez la propriété loginServer de votre registre que vous avez notée à l’étape précédente, par exemple *myhelmacr.azurecr.io* :

* Remplacez `image.repository` par `<loginServer>/webfrontend` :
* Remplacez `service.type` par `LoadBalancer` :

Par exemple :

```yml
# Default values for webfrontend.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

replicaCount: 1

image:
  repository: *myhelmacr.azurecr.io*/webfrontend
  pullPolicy: IfNotPresent
...
service:
  type: LoadBalancer
  port: 80
...
```

Mettez à jour `appVersion` vers `v1` dans *webfrontend/Chart.yaml*. Par exemple

```yml
apiVersion: v2
name: webfrontend
...
# This is the version number of the application being deployed. This version number should be
# incremented each time you make changes to the application.
appVersion: v1
```

## <a name="run-your-helm-chart"></a>Exécuter votre chart Helm

Utilisez la commande `helm install` pour installer votre application à l’aide de votre chart Helm.

```console
helm install webfrontend webfrontend/
```

Le renvoi d’une adresse IP publique par le service prend plusieurs minutes. Pour surveiller la progression, utilisez la commande `kubectl get service` avec le paramètre *watch* :

```console
$ kubectl get service --watch

NAME                TYPE          CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
webfrontend         LoadBalancer  10.0.141.72   <pending>     80:32150/TCP   2m
...
webfrontend         LoadBalancer  10.0.141.72   <EXTERNAL-IP> 80:32150/TCP   7m
```

Accédez à l’équilibreur de charge de votre application dans un navigateur en utilisant `<EXTERNAL-IP>` pour voir l’exemple d’application.

## <a name="delete-the-cluster"></a>Supprimer le cluster

Quand vous n’avez plus besoin du cluster, utilisez la commande [az group delete][az-group-delete] pour supprimer le groupe de ressources, le cluster AKS, le registre de conteneurs, les images conteneur qui y sont stockées et toutes les ressources associées.

```azurecli-interactive
az group delete --name MyResourceGroup --yes --no-wait
```

> [!NOTE]
> Lorsque vous supprimez le cluster, le principal de service Azure Active Directory utilisé par le cluster AKS n’est pas supprimé. Pour obtenir des instructions sur la façon de supprimer le principal de service, consultez [Considérations et suppression du principal de service AKS][sp-delete]. Si vous avez utilisé une identité managée, l’identité est managée par la plateforme et n’a pas besoin d’être supprimée.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’utilisation de Helm, consultez la documentation de Helm.

> [!div class="nextstepaction"]
> [Documentation de Helm][helm-documentation]

[az-acr-create]: /cli/azure/acr#az-acr-create
[az-acr-build]: /cli/azure/acr#az-acr-build
[az-group-delete]: /cli/azure/group#az-group-delete
[az aks get-credentials]: /cli/azure/aks#az-aks-get-credentials
[az aks install-cli]: /cli/azure/aks#az-aks-install-cli
[example-nodejs]: https://github.com/Azure/dev-spaces/tree/master/samples/nodejs/getting-started/webfrontend
[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[helm]: https://helm.sh/
[helm-documentation]: https://helm.sh/docs/
[helm-existing]: kubernetes-helm.md
[helm-install]: https://helm.sh/docs/intro/install/
[sp-delete]: kubernetes-service-principal.md#additional-considerations
