---
title: Créer des nœuds virtuels à l’aide du portail dans Azure Kubernetes Service (AKS)
description: Découvrez comment utiliser le Portail Azure pour créer un cluster Azure Kubernetes Service (AKS) qui s’appuie sur des nœuds virtuels pour exécuter des pods.
services: container-service
ms.topic: conceptual
ms.date: 05/06/2019
ms.custom: references_regions, devx-track-azurecli
ms.openlocfilehash: 7d49499b39c562aeff20d163fc86401d8c1f4a06
ms.sourcegitcommit: 1d6ec4b6f60b7d9759269ce55b00c5ac5fb57d32
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2020
ms.locfileid: "94579162"
---
# <a name="create-and-configure-an-azure-kubernetes-services-aks-cluster-to-use-virtual-nodes-in-the-azure-portal"></a>Créer et configurer un cluster Azure Kubernetes Service (AKS) pour utiliser des nœuds virtuels sur le Portail Azure

Cet article explique comment utiliser le portail Azure pour créer et configurer les ressources de réseau virtuel et un cluster AKS avec des nœuds virtuels activés.

> [!NOTE]
> [Cet article](virtual-nodes.md) vous donne une vue d’ensemble de la disponibilité par région et des limitations pour utiliser des nœuds virtuels.

## <a name="before-you-begin"></a>Avant de commencer

Les nœuds virtuels permettent la communication réseau entre les pods qui s’exécutent dans Azure Container Instances (ACI) et le cluster AKS. Pour que cette communication ait lieu, un sous-réseau de réseau virtuel est créé et des permissions déléguées sont assignées. Les nœuds virtuels ne fonctionnent qu’avec des clusters AKS créés avec un réseau *avancé* (Azure CNI). Par défaut, les clusters AKS sont créés avec un réseau *simple* (kubenet). Cet article vous montre comment créer un réseau virtuel et des sous-réseaux, puis déployer un cluster AKS qui utilise un réseau avancé.

Si vous n’avez pas encore utilisé ACI, inscrivez le fournisseur de services avec votre abonnement. Vous pouvez vérifier l’état d’inscription du fournisseur d’ACI à l’aide de la commande [az provider list][az-provider-list], comme dans l’exemple suivant :

```azurecli-interactive
az provider list --query "[?contains(namespace,'Microsoft.ContainerInstance')]" -o table
```

Le fournisseur de *Microsoft.ContainerInstance* doivent être signalé comme *Registered* (Inscrit), comme dans l’exemple de sortie suivant :

```output
Namespace                    RegistrationState    RegistrationPolicy
---------------------------  -------------------  --------------------
Microsoft.ContainerInstance  Registered           RegistrationRequired
```

Si le fournisseur apparaît *NotRegistered* (Non-inscrit), inscrivez-le à l’aide de la commande [az provider register][az-provider-register] comme dans l’exemple suivant :

```azurecli-interactive
az provider register --namespace Microsoft.ContainerInstance
```

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au portail Azure sur https://portal.azure.com.

## <a name="create-an-aks-cluster"></a>Créer un cluster AKS

Sélectionnez **Créer une ressource** > **Kubernetes Service** dans le coin supérieur gauche du portail Azure.

Sur la page **Bases**, configurez les options suivantes :

- *DÉTAILS DU PROJET* : Choisissez un abonnement Azure, puis sélectionnez ou créez un groupe de ressources Azure, comme *myResourceGroup*. Entrez un **nom du cluster Kubernetes**, tel que *myAKSCluster*.
- *DÉTAILS DU CLUSTER* : Sélectionnez une région, une version de Kubernetes et le préfixe du nom DNS du cluster AKS.
- *POOL DE NŒUDS PRINCIPAL* : Sélectionnez une taille de machine virtuelle pour les nœuds AKS. Elle ne sera **pas modifiable** une fois le cluster AKS déployé.
     - Sélectionnez également le nombre de nœuds à déployer dans le cluster. Dans le cadre de cet article, définissez **Nombre de nœuds** sur *1*. Le nombre de nœuds est **modifiable** après le déploiement du cluster.

Cliquez sur **Suivant : Mise à l'échelle**.

Sur la page **Mise à l’échelle**, sélectionnez *Activé* sous **Nœuds virtuels**.

![Créer un cluster AKS et activer les nœuds virtuels](media/virtual-nodes-portal/enable-virtual-nodes.png)

Par défaut, un principal de service Azure Active Directory est créé. Il a pour fonctions la communication entre les clusters et la compatibilité avec d’autres services Azure. Pour les autorisations, vous pouvez également utiliser une identité managée au lieu d’un principal de service. Pour plus d’informations, consultez [Utiliser des identités managées](use-managed-identity.md).

Le cluster est également configuré pour offrir une mise en réseau avancée. Les nœuds virtuels sont paramétrés de façon à utiliser leur propre sous-réseau de réseau virtuel Azure. Ce sous-réseau possède des permissions déléguées pour se connecter à des ressources Azure entre le cluster AKS. Si vous ne disposez pas de sous-réseau délégué, le Portail Azure crée et configure le réseau virtuel Azure et le sous-réseau d’une manière compatible avec les nœuds virtuels.

Sélectionnez **Revoir + créer**. Une fois la validation terminée, sélectionnez **Créer**.

Il ne faut que quelques minutes pour créer le cluster AKS et que celui-ci soit prêt à être utilisé.

## <a name="connect-to-the-cluster"></a>Se connecter au cluster

Azure Cloud Shell est un interpréteur de commandes interactif et gratuit que vous pouvez utiliser pour exécuter les étapes de cet article. Il contient des outils Azure courants préinstallés et configurés pour être utilisés avec votre compte. Pour gérer un cluster Kubernetes, utilisez [kubectl][kubectl], le client de ligne de commande Kubernetes. Le client `kubectl` est préinstallé dans Azure Cloud Shell.

Pour ouvrir Cloud Shell, sélectionnez **Essayer** en haut à droite d’un bloc de code. Vous pouvez également lancer Cloud Shell dans un onglet distinct du navigateur en accédant à [https://shell.azure.com/bash](https://shell.azure.com/bash). Sélectionnez **Copier** pour copier les blocs de code, collez-les dans Cloud Shell, puis appuyez sur Entrée pour les exécuter.

Exécutez la commande [az aks get-credentials][az-aks-get-credentials] pour configurer `kubectl` afin de vous connecter à votre cluster Kubernetes. L’exemple suivant obtient les informations d’identification du nom du cluster *myAKSCluster* dans le groupe de ressources nommé *myResourceGroup* :

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

Pour vérifier la connexion à votre cluster, utilisez la commande [kubectl get][kubectl-get] pour retourner une liste des nœuds du cluster.

```console
kubectl get nodes
```

L’exemple de sortie suivant illustre l’unique nœud de machine virtuelle créé, puis le nœud virtuel Linux, *virtual-node-aci-linux* :

```output
NAME                           STATUS    ROLES     AGE       VERSION
virtual-node-aci-linux         Ready     agent     28m       v1.11.2
aks-agentpool-14693408-0       Ready     agent     32m       v1.11.2
```

## <a name="deploy-a-sample-app"></a>Déployer un exemple d’application

Dans Azure Cloud Shell, créez un fichier nommé `virtual-node.yaml` et copiez-y le code YAML suivant. Pour planifier le conteneur au niveau du nœud, [nodeSelector][node-selector] et [toleration][toleration] sont définis. Ces paramètres permettent de planifier le pod sur le nœud virtuel et de vérifier que la fonctionnalité est activée.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: aci-helloworld
spec:
  replicas: 1
  selector:
    matchLabels:
      app: aci-helloworld
  template:
    metadata:
      labels:
        app: aci-helloworld
    spec:
      containers:
      - name: aci-helloworld
        image: mcr.microsoft.com/azuredocs/aci-helloworld
        ports:
        - containerPort: 80
      nodeSelector:
        kubernetes.io/role: agent
        beta.kubernetes.io/os: linux
        type: virtual-kubelet
      tolerations:
      - key: virtual-kubelet.io/provider
        operator: Exists
```

Exécutez l’application avec la commande [kubectl apply][kubectl-apply].

```azurecli-interactive
kubectl apply -f virtual-node.yaml
```

Utilisez la commande [kubectl get pods][kubectl-get] avec l’argument `-o wide` pour obtenir la liste des pods et le nœud planifié. Remarquez que le pod `virtual-node-helloworld` a été planifié sur le nœud `virtual-node-linux`.

```console
kubectl get pods -o wide
```

```output
NAME                                     READY     STATUS    RESTARTS   AGE       IP           NODE
virtual-node-helloworld-9b55975f-bnmfl   1/1       Running   0          4m        10.241.0.4   virtual-node-aci-linux
```

Le pod reçoit une adresse IP interne du sous-réseau de réseau virtuel Azure délégué de façon à être utilisé avec des nœuds virtuels.

> [!NOTE]
> Si vous utilisez des images stockées dans Azure Container Registry, [configurez et utilisez un secret Kubernetes][acr-aks-secrets]. Compte tenu des limitations imposées pour les nœuds virtuels, vous ne pouvez pas utiliser l'authentification de principal de service Azure AD intégrée. Si vous n'utilisez pas de secret, les pods planifiés sur les nœuds virtuels ne parviennent pas à démarrer et renvoient l'erreur `HTTP response status code 400 error code "InaccessibleImage"`.

## <a name="test-the-virtual-node-pod"></a>Tester le pod de nœud virtuel

Pour tester le pod en cours d’exécution sur le nœud virtuel, accédez à l’application de démonstration avec un client web. Comme le pod possède une adresse IP interne, vous pouvez tester facilement cette connectivité à partir d’un autre pod sur le cluster AKS. Créez un pod de test et attachez-y une session Terminal :

```console
kubectl run -it --rm virtual-node-test --image=debian
```

Installez `curl` dans le pod en utilisant `apt-get` :

```console
apt-get update && apt-get install -y curl
```

Maintenant, accédez à l’adresse de votre pod avec `curl` (par exemple, *http://10.241.0.4* ). Indiquez votre propre adresse IP interne, obtenue avec la commande `kubectl get pods` précédente :

```console
curl -L http://10.241.0.4
```

L’application de démonstration s’affiche, comme l’illustre l’exemple de sortie condensé suivant :

```output
<html>
<head>
  <title>Welcome to Azure Container Instances!</title>
</head>
[...]
```

Fermez la session de terminal sur votre pod de test avec `exit`. Une fois la session terminée, le pod est supprimé.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, un pod a été planifié sur le nœud virtuel et a reçu une adresse IP interne privée. Vous auriez tout aussi bien pu créer un déploiement de service et router le trafic vers votre pod au moyen d’un équilibreur de charge ou d’un contrôleur d’entrée. Pour plus d’informations, consultez [Créer un contrôleur d’entrée de base dans AKS][aks-basic-ingress].

Les nœuds virtuels constituent l’un des composants d’une solution de mise à l’échelle dans AKS. Pour plus d’informations sur ces solutions, consultez les articles suivants :

- [Utiliser l’autoscaler de pods élastique Kubernetes][aks-hpa]
- [Utiliser l’autoscaler de cluster Kubernetes][aks-cluster-autoscaler]
- [Consultez l’exemple de mise à l’échelle automatique pour les nœuds virtuels][virtual-node-autoscale]
- [Découvrez-en plus sur la bibliothèque open source de Virtual Kubelet][virtual-kubelet-repo]

<!-- LINKS - external -->
[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[node-selector]:https://kubernetes.io/docs/concepts/configuration/assign-pod-node/
[toleration]: https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/
[azure-cni]: https://github.com/Azure/azure-container-networking/blob/master/docs/cni.md
[aks-github]: https://github.com/azure/aks/issues]
[virtual-node-autoscale]: https://github.com/Azure-Samples/virtual-node-autoscale
[virtual-kubelet-repo]: https://github.com/virtual-kubelet/virtual-kubelet
[acr-aks-secrets]: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/

<!-- LINKS - internal -->
[aks-network]: ./configure-azure-cni.md
[az-aks-get-credentials]: /cli/azure/aks?view=azure-cli-latest#az-aks-get-credentials
[aks-hpa]: tutorial-kubernetes-scale.md
[aks-cluster-autoscaler]: cluster-autoscaler.md
[aks-basic-ingress]: ingress-basic.md
[az-provider-list]: /cli/azure/provider?view=azure-cli-latest#az-provider-list
[az-provider-register]: /cli/azure/provider?view=azure-cli-latest&preserve-view=true#az-provider-register
