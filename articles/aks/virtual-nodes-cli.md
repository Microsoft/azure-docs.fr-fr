---
title: Créer des nœuds virtuels à l’aide d’Azure CLI
titleSuffix: Azure Kubernetes Service
description: Découvrez comment utiliser l’interface de ligne de commande Azure pour créer un cluster Azure Kubernetes Service (AKS) qui s’appuie sur des nœuds virtuels pour exécuter des pods.
services: container-service
ms.topic: conceptual
ms.date: 05/06/2019
ms.custom: references_regions, devx-track-azurecli
ms.openlocfilehash: a655c8c145b4f3812dae9f1a4ec1e5eebbe44809
ms.sourcegitcommit: 99955130348f9d2db7d4fb5032fad89dad3185e7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93348472"
---
# <a name="create-and-configure-an-azure-kubernetes-services-aks-cluster-to-use-virtual-nodes-using-the-azure-cli"></a>Créer et configurer un cluster Azure Kubernetes Service (AKS) pour utiliser des nœuds virtuels à l’aide de l’interface de ligne de commande Azure

Cet article explique comment utiliser l’interface Azure CLI pour créer et configurer les ressources de réseau virtuel et un cluster AKS, puis activer des nœuds virtuels.

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

## <a name="launch-azure-cloud-shell"></a>Lancement d’Azure Cloud Shell

Azure Cloud Shell est un interpréteur de commandes interactif et gratuit que vous pouvez utiliser pour exécuter les étapes de cet article. Il contient des outils Azure courants préinstallés et configurés pour être utilisés avec votre compte.

Pour ouvrir Cloud Shell, sélectionnez **Essayer** en haut à droite d’un bloc de code. Vous pouvez également lancer Cloud Shell dans un onglet distinct du navigateur en accédant à [https://shell.azure.com/bash](https://shell.azure.com/bash). Sélectionnez **Copier** pour copier les blocs de code, collez-les dans Cloud Shell, puis appuyez sur Entrée pour les exécuter.

Si vous préférez installer et utiliser l’interface de ligne de commande en local, cet article nécessite Azure CLI version 2.0.49 ou ultérieure. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI]( /cli/azure/install-azure-cli).

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Un groupe de ressources Azure est un groupe logique dans lequel des ressources Azure sont déployées et gérées. Créez un groupe de ressources avec la commande [az group create][az-group-create]. L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *westus*.

```azurecli-interactive
az group create --name myResourceGroup --location westus
```

## <a name="create-a-virtual-network"></a>Créez un réseau virtuel

Utilisez la commande [az network vnet create][az-network-vnet-create] pour créer un réseau virtuel. L’exemple suivant crée un réseau virtuel nommé *myVnet* avec un préfixe d’adresse de *10.0.0.0/8* et un sous-réseau nommé *myAKSSubnet*. Par défaut, le préfixe d’adresse de ce sous-réseau est *10.240.0.0/16*  :

```azurecli-interactive
az network vnet create \
    --resource-group myResourceGroup \
    --name myVnet \
    --address-prefixes 10.0.0.0/8 \
    --subnet-name myAKSSubnet \
    --subnet-prefix 10.240.0.0/16
```

Maintenant, créez un sous-réseau supplémentaire pour les nœuds virtuels à l’aide de la commande [az network vnet subnet create][az-network-vnet-subnet-create]. L’exemple suivant crée un sous-réseau nommé *myVirtualNodeSubnet* avec le préfixe d’adresse *10.241.0.0/16*.

```azurecli-interactive
az network vnet subnet create \
    --resource-group myResourceGroup \
    --vnet-name myVnet \
    --name myVirtualNodeSubnet \
    --address-prefixes 10.241.0.0/16
```

## <a name="create-a-service-principal-or-use-a-managed-identity"></a>Créer un principal de service ou utiliser une identité managée

Pour permettre à un cluster AKS d’interagir avec d’autres ressources Azure, un principal du service Azure Active Directory est utilisé. Ce principal du service peut être créé automatiquement par Azure CLI ou le portail, ou vous pouvez en précréer un et lui affecter des autorisations supplémentaires. Pour les autorisations, vous pouvez également utiliser une identité managée au lieu d’un principal de service. Pour plus d’informations, consultez [Utiliser des identités managées](use-managed-identity.md).

Créez un principal du service à l’aide de la commande [az ad sp create-for-rbac][az-ad-sp-create-for-rbac]. Le paramètre `--skip-assignment` limite l’affectation d’autorisations supplémentaires.

```azurecli-interactive
az ad sp create-for-rbac --skip-assignment
```

Le résultat ressemble à l’exemple suivant :

```output
{
  "appId": "bef76eb3-d743-4a97-9534-03e9388811fc",
  "displayName": "azure-cli-2018-11-21-18-42-00",
  "name": "http://azure-cli-2018-11-21-18-42-00",
  "password": "1d257915-8714-4ce7-a7fb-0e5a5411df7f",
  "tenant": "72f988bf-86f1-41af-91ab-2d7cd011db48"
}
```

Prenez note des valeurs de *appId* et de *password*. Ces valeurs sont utilisées dans les étapes suivantes.

## <a name="assign-permissions-to-the-virtual-network"></a>Affecter des autorisations au réseau virtuel

Pour permettre à votre cluster d’utiliser et de gérer le réseau virtuel, vous devez accorder au principal de service AKS les droits appropriés pour utiliser les ressources réseau.

Tout d’abord, obtenez l’ID de ressource de réseau virtuel à l’aide de la commande [az network vnet show][az-network-vnet-show] :

```azurecli-interactive
az network vnet show --resource-group myResourceGroup --name myVnet --query id -o tsv
```

Pour accorder l’accès correct afin que le cluster AKS puisse utiliser le réseau virtuel, créez une attribution de rôle à l’aide de la commande [az role assignment create][az-role-assignment-create]. Remplacez `<appId`> et `<vnetId>` par les valeurs recueillies au cours des deux étapes précédentes.

```azurecli-interactive
az role assignment create --assignee <appId> --scope <vnetId> --role Contributor
```

## <a name="create-an-aks-cluster"></a>Créer un cluster AKS

Vous déployez un cluster AKS sur un sous-réseau AKS créé à l’étape précédente. Obtenez l’ID de ce sous-réseau à l’aide de la commande [az network vnet subnet show][az-network-vnet-subnet-show] :

```azurecli-interactive
az network vnet subnet show --resource-group myResourceGroup --vnet-name myVnet --name myAKSSubnet --query id -o tsv
```

Utilisez la commande [az aks create][az-aks-create] pour créer un cluster AKS. L’exemple suivant crée un cluster à un nœud nommé *myAKSCluster*. Remplacez `<subnetId>` par l’ID obtenu à l’étape précédente, puis `<appId>` et `<password>` par les valeurs collectées dans la section précédente.

```azurecli-interactive
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --node-count 1 \
    --network-plugin azure \
    --service-cidr 10.0.0.0/16 \
    --dns-service-ip 10.0.0.10 \
    --docker-bridge-address 172.17.0.1/16 \
    --vnet-subnet-id <subnetId> \
    --service-principal <appId> \
    --client-secret <password>
```

Au bout de quelques minutes, la commande se termine et retourne des informations au format JSON sur le cluster.

## <a name="enable-virtual-nodes-addon"></a>Activer le module complémentaire de nœuds virtuels

Pour activer les nœuds virtuels, utilisez à présent la commande [az aks enable-addons][az-aks-enable-addons]. L’exemple suivant utilise le sous-réseau nommé *myVirtualNodeSubnet* créé à l’étape précédente :

```azurecli-interactive
az aks enable-addons \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --addons virtual-node \
    --subnet-name myVirtualNodeSubnet
```

## <a name="connect-to-the-cluster"></a>Se connecter au cluster

Pour configurer `kubectl` afin de vous connecter à votre cluster Kubernetes, exécutez la commande [az aks get-credentials][az-aks-get-credentials]. Cette étape télécharge les informations d’identification et configure l’interface de ligne de commande Kubernetes pour leur utilisation.

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

Pour vérifier la connexion à votre cluster, utilisez la commande [kubectl get][kubectl-get] pour retourner une liste des nœuds du cluster.

```console
kubectl get nodes
```

L’exemple de sortie suivant illustre l’unique nœud de machine virtuelle créé, puis le nœud virtuel Linux, *virtual-node-aci-linux*  :

```output
NAME                          STATUS    ROLES     AGE       VERSION
virtual-node-aci-linux        Ready     agent     28m       v1.11.2
aks-agentpool-14693408-0      Ready     agent     32m       v1.11.2
```

## <a name="deploy-a-sample-app"></a>Déployer un exemple d’application

Créez un fichier nommé `virtual-node.yaml` et copiez-y le YAML suivant. Pour planifier le conteneur au niveau du nœud, [nodeSelector][node-selector] et [toleration][toleration] sont définis.

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
      - key: azure.com/aci
        effect: NoSchedule
```

Exécutez l’application avec la commande [kubectl apply][kubectl-apply].

```console
kubectl apply -f virtual-node.yaml
```

Utilisez la commande [kubectl get pods][kubectl-get] avec l’argument `-o wide` pour obtenir la liste des pods et le nœud planifié. Remarquez que le pod `aci-helloworld` a été planifié sur le nœud `virtual-node-aci-linux`.

```console
kubectl get pods -o wide
```

```output
NAME                            READY     STATUS    RESTARTS   AGE       IP           NODE
aci-helloworld-9b55975f-bnmfl   1/1       Running   0          4m        10.241.0.4   virtual-node-aci-linux
```

Le pod reçoit une adresse IP interne du sous-réseau de réseau virtuel Azure délégué de façon à être utilisé avec des nœuds virtuels.

> [!NOTE]
> Si vous utilisez des images stockées dans Azure Container Registry, [configurez et utilisez un secret Kubernetes][acr-aks-secrets]. Compte tenu des limitations imposées pour les nœuds virtuels, vous ne pouvez pas utiliser l'authentification de principal de service Azure AD intégrée. Si vous n'utilisez pas de secret, les pods planifiés sur les nœuds virtuels ne parviennent pas à démarrer et renvoient l'erreur `HTTP response status code 400 error code "InaccessibleImage"`.

## <a name="test-the-virtual-node-pod"></a>Tester le pod de nœud virtuel

Pour tester le pod en cours d’exécution sur le nœud virtuel, accédez à l’application de démonstration avec un client web. Comme le pod possède une adresse IP interne, vous pouvez tester facilement cette connectivité à partir d’un autre pod sur le cluster AKS. Créez un pod de test et attachez-y une session Terminal :

```console
kubectl run -it --rm testvk --image=debian
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

## <a name="remove-virtual-nodes"></a>Supprimer des nœuds virtuels

Si vous ne souhaitez plus utiliser des nœuds virtuels, vous pouvez les désactiver à l’aide de la commande [az aks disable-addons][az aks disable-addons]. 

Si nécessaire, accédez à [https://shell.azure.com](https://shell.azure.com) pour ouvrir Azure Cloud Shell dans votre navigateur.

Tout d’abord, supprimez le pod `aci-helloworld` qui s’exécute sur le nœud virtuel :

```console
kubectl delete -f virtual-node.yaml
```

L’exemple de commande suivant désactive les nœuds virtuels Linux :

```azurecli-interactive
az aks disable-addons --resource-group myResourceGroup --name myAKSCluster --addons virtual-node
```

À présent, supprimez les ressources réseau virtuelles et le groupe de ressources :

```azurecli-interactive
# Change the name of your resource group, cluster and network resources as needed
RES_GROUP=myResourceGroup
AKS_CLUSTER=myAKScluster
AKS_VNET=myVnet
AKS_SUBNET=myVirtualNodeSubnet

# Get AKS node resource group
NODE_RES_GROUP=$(az aks show --resource-group $RES_GROUP --name $AKS_CLUSTER --query nodeResourceGroup --output tsv)

# Get network profile ID
NETWORK_PROFILE_ID=$(az network profile list --resource-group $NODE_RES_GROUP --query [0].id --output tsv)

# Delete the network profile
az network profile delete --id $NETWORK_PROFILE_ID -y

# Delete the subnet delegation to Azure Container Instances
az network vnet subnet update --resource-group $RES_GROUP --vnet-name $AKS_VNET --name $AKS_SUBNET --remove delegations 0
```

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, un pod a été planifié sur le nœud virtuel et a reçu une adresse IP interne privée. Vous auriez tout aussi bien pu créer un déploiement de service et router le trafic vers votre pod au moyen d’un équilibreur de charge ou d’un contrôleur d’entrée. Pour plus d’informations, consultez [Créer un contrôleur d’entrée de base dans AKS][aks-basic-ingress].

Les nœuds virtuels constituent souvent l’un des composants d’une solution de mise à l’échelle dans AKS. Pour plus d’informations sur ces solutions, consultez les articles suivants :

- [Utiliser l’autoscaler de pods élastique Kubernetes][aks-hpa]
- [Utiliser l’autoscaler de cluster Kubernetes][aks-cluster-autoscaler]
- [Consultez l’exemple de mise à l’échelle automatique pour les nœuds virtuels][virtual-node-autoscale]
- [Découvrez-en plus sur la bibliothèque open source de Virtual Kubelet][virtual-kubelet-repo]

<!-- LINKS - external -->
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[node-selector]:https://kubernetes.io/docs/concepts/configuration/assign-pod-node/
[toleration]: https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/
[aks-github]: https://github.com/azure/aks/issues
[virtual-node-autoscale]: https://github.com/Azure-Samples/virtual-node-autoscale
[virtual-kubelet-repo]: https://github.com/virtual-kubelet/virtual-kubelet
[acr-aks-secrets]: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/

<!-- LINKS - internal -->
[azure-cli-install]: /cli/azure/install-azure-cli
[az-group-create]: /cli/azure/group#az-group-create
[az-network-vnet-create]: /cli/azure/network/vnet#az-network-vnet-create
[az-network-vnet-subnet-create]: /cli/azure/network/vnet/subnet#az-network-vnet-subnet-create
[az-ad-sp-create-for-rbac]: /cli/azure/ad/sp#az-ad-sp-create-for-rbac
[az-network-vnet-show]: /cli/azure/network/vnet#az-network-vnet-show
[az-role-assignment-create]: /cli/azure/role/assignment#az-role-assignment-create
[az-network-vnet-subnet-show]: /cli/azure/network/vnet/subnet#az-network-vnet-subnet-show
[az-aks-create]: /cli/azure/aks#az-aks-create
[az-aks-enable-addons]: /cli/azure/aks#az-aks-enable-addons
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-aks-get-credentials]: /cli/azure/aks#az-aks-get-credentials
[az aks disable-addons]: /cli/azure/aks#az-aks-disable-addons
[aks-hpa]: tutorial-kubernetes-scale.md
[aks-cluster-autoscaler]: ./cluster-autoscaler.md
[aks-basic-ingress]: ingress-basic.md
[az-provider-list]: /cli/azure/provider#az-provider-list
[az-provider-register]: /cli/azure/provider#az-provider-register
