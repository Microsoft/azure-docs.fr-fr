---
title: S’authentifier à partir d’un cluster Kubernetes
description: Découvrez comment fournir à un cluster Kubernetes un accès aux images de votre registre de conteneurs Azure en créant un secret d’extraction (pull secret) et en utilisant un principal de service
ms.topic: article
author: karolz-ms
ms.author: karolz
ms.reviewer: danlep
ms.date: 05/28/2020
ms.openlocfilehash: fbf5dfd68b823b600b11cad3643e5d4004b85ff5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84309813"
---
# <a name="pull-images-from-an-azure-container-registry-to-a-kubernetes-cluster"></a>Extraire des images depuis un registre de conteneurs Azure vers un cluster Kubernetes

Vous pouvez utiliser un registre de conteneurs Azure comme source d’images conteneur avec n’importe quel cluster Kubernetes, notamment les clusters Kubernetes locaux, comme [minikube](https://minikube.sigs.k8s.io/) et [kind](https://kind.sigs.k8s.io/). Cet article explique comment créer un secret d’extraction Kubernetes basé sur un principal de service Azure Active Directory. Ensuite, utilisez le secret pour extraire des images à partir d’un registre de conteneurs Azure dans un déploiement Kubernetes.

> [!TIP]
> Si vous utilisez le [service Azure Kubernetes](../aks/intro-kubernetes.md) managé, vous pouvez également [intégrer votre cluster](../aks/cluster-container-registry-integration.md?toc=/azure/container-registry/toc.json&bc=/azure/container-registry/breadcrumb/toc.json) à un registre de conteneurs Azure cible pour les extractions d’images. 

Cet article suppose que vous avez déjà créé un registre de conteneurs Azure privé. Vous devez également disposer d’un cluster Kubernetes en cours d’exécution et accessible via l’outil en ligne de commande `kubectl`.

[!INCLUDE [container-registry-service-principal](../../includes/container-registry-service-principal.md)]

Si vous n’enregistrez pas ou que vous ne vous rappelez pas du mot de passe du principal de service, vous pouvez le réinitialiser avec la commande [az ad sp credential reset][az-ad-sp-credential-reset] :

```azurecli
az ad sp credential reset  --name http://<service-principal-name> --query password --output tsv
```

Cette commande retourne un nouveau mot de passe valide pour votre principal de service.

## <a name="create-an-image-pull-secret"></a>Créer un secret d’extraction d’image

Kubernetes utilise un *secret d’extraction d’image* pour stocker les informations nécessaires pour s’authentifier auprès de votre registre. Pour créer le secret d’extraction pour un registre de conteneurs Azure, vous fournissez l’ID du principal de service, le mot de passe et l’URL du registre. 

Créez un secret d’extraction d’image avec la commande `kubectl` suivante :

```console
kubectl create secret docker-registry <secret-name> \
    --namespace <namespace> \
    --docker-server=<container-registry-name>.azurecr.io \
    --docker-username=<service-principal-ID> \
    --docker-password=<service-principal-password>
```
où :

| Valeur | Description |
| :--- | :--- |
| `secret-name` | Nom du secret d’extraction d’image, par exemple, *acr-secret* |
| `namespace` | Espace de noms Kubernetes où placer le secret <br/> Nécessaire seulement si vous voulez placer le secret dans un espace de noms autre que l’espace de noms par défaut |
| `container-registry-name` | Nom de votre registre de conteneurs Azure, par exemple *myregistry*<br/><br/>`--docker-server` est le nom complet du serveur de connexion au registre.  |
| `service-principal-ID` | ID du principal de service qui sera utilisé par Kubernetes pour accéder à votre registre |
| `service-principal-password` | Mot de passe de principal de service |

## <a name="use-the-image-pull-secret"></a>Utiliser le secret d’extraction d’image

Une fois que vous avez créé le secret d’extraction d’image, vous pouvez l’utiliser pour créer des pods et des déploiements Kubernetes. Spécifiez le nom du secret sous `imagePullSecrets` dans le fichier de déploiement. Par exemple :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-awesome-app-pod
  namespace: awesomeapps
spec:
  containers:
    - name: main-app-container
      image: myregistry.azurecr.io/my-awesome-app:v1
      imagePullPolicy: IfNotPresent
  imagePullSecrets:
    - name: acr-secret
```

Dans l’exemple précédent, `my-awesome-app:v1` est le nom de l’image à extraire du registre de conteneurs Azure, et `acr-secret` est le nom du secret d’extraction que vous avez créé pour accéder au registre. Quand vous déployez le pod, Kubernetes extrait automatiquement l’image de votre registre, si elle n’est pas déjà présente sur le cluster.


## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur l’utilisation des principaux de service et d’Azure Container Registry, consultez [Authentification Azure Container Registry avec des principaux de service](container-registry-auth-service-principal.md)
* Découvrez plus d’informations sur les secrets d’extraction d’image dans la [documentation Kubernetes](https://kubernetes.io/docs/concepts/containers/images/#specifying-imagepullsecrets-on-a-pod)


<!-- IMAGES -->

<!-- LINKS - External -->
[acr-scripts-cli]: https://github.com/Azure/azure-docs-cli-python-samples/tree/master/container-registry
[acr-scripts-psh]: https://github.com/Azure/azure-docs-powershell-samples/tree/master/container-registry

<!-- LINKS - Internal -->
[az-ad-sp-credential-reset]: /cli/azure/ad/sp/credential#az-ad-sp-credential-reset