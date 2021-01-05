---
title: Sécuriser le trafic de pods avec une stratégie réseau
titleSuffix: Azure Kubernetes Service
description: Découvrez comment sécuriser le trafic qui transite par des pods à l’aide de stratégies réseau Kubernetes dans Azure Kubernetes Service (AKS)
services: container-service
ms.topic: article
ms.date: 05/06/2019
ms.openlocfilehash: 598747c0d64db2ae62f740dca4c3e4141f2562f2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87050486"
---
# <a name="secure-traffic-between-pods-using-network-policies-in-azure-kubernetes-service-aks"></a>Sécuriser le trafic entre les pods avec des stratégies réseau dans Azure Kubernetes Service (AKS)

Lorsque vous exécutez des applications modernes, basées sur des microservices dans Kubernetes, vous souhaitez la plupart du temps décider des composants qui peuvent communiquer entre eux. Le principe des privilèges minimum doit être appliqué à la manière dont le trafic peut transiter entre les pods dans un cluster Azure Kubernetes Service (AKS). Supposons que vous souhaitiez bloquer le trafic directement vers les applications principales. La fonctionnalité *Network Policy* (Stratégie réseau) de Kubernetes vous permet de définir des règles de trafic d’entrée et de sortie entre les pods d’un cluster.

Cet article vous explique comment installer le moteur de stratégie réseau et créer des stratégies réseau Kubernetes pour contrôler le flux du trafic entre les pods dans AKS. Une stratégie réseau doit uniquement être utilisée pour les nœuds et pods Linux dans AKS.

## <a name="before-you-begin"></a>Avant de commencer

L’interface de ligne de commande Azure (Azure CLI) version 2.0.61 ou une version ultérieure doit avoir été installée et configurée. Exécutez  `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, consultez  [Installation d’Azure CLI][install-azure-cli].

> [!TIP]
> Si vous utilisiez la fonctionnalité de stratégie réseau disponible dans la préversion, nous vous recommandons de [créer un autre cluster](#create-an-aks-cluster-and-enable-network-policy).
> 
> Si vous souhaitez continuer à utiliser les clusters de test existants qui utilisaient une stratégie réseau de la préversion, mettez à niveau votre cluster vers une nouvelle version de Kubernetes pour la dernière version en disponibilité générale, puis déployez le manifeste YAML ci-après pour corriger les incidents du server de métriques et du tableau de bord Kubernetes. Ce correctif est uniquement requis pour les clusters qui utilisaient le moteur de stratégie réseau Calico.
>
> Afin d’optimiser la sécurité, [examinez le contenu de ce manifeste YAML][calico-aks-cleanup] pour connaître précisément les éléments qui sont déployés dans le cluster AKS.
>
> `kubectl delete -f https://raw.githubusercontent.com/Azure/aks-engine/master/docs/topics/calico-3.3.1-cleanup-after-upgrade.yaml`

## <a name="overview-of-network-policy"></a>Présentation de la stratégie réseau

Par défaut, tous les pods d’un cluster AKS peuvent envoyer et recevoir du trafic sans aucune limite. Pour améliorer la sécurité, vous pouvez définir des règles qui contrôlent le flux du trafic. Par exemple, les applications principales ne sont généralement exposées qu’aux services frontaux requis. De même, les composants de base de données sont uniquement accessibles aux couches Application qui s’y connectent.

Une stratégie réseau est une spécification Kubernetes qui définit les stratégies d’accès concernant la communication entre les pods. Vous utilisez les stratégies réseau pour définir un jeu ordonné de règles régissant l’envoi et la réception du trafic, puis pour appliquer ces règles à une collection de pods qui correspondent à un ou plusieurs sélecteurs d’étiquette.

Ces règles de stratégie réseau sont définies sous la forme de manifestes YAML. Les stratégies réseau peuvent être intégrées à un manifeste plus vaste qui crée également un déploiement ou un service.

### <a name="network-policy-options-in-aks"></a>Options de stratégie réseau dans AKS

Azure vous permet d’implémenter une stratégie réseau de deux manières. Vous choisissez une option de stratégie réseau lorsque vous créez un cluster AKS. Une fois le cluster créé, l’option de stratégie n’est plus modifiable :

* implémentation propre à Azure, appelée *stratégies réseau Azure*
* *stratégies réseau Calico*, une solution de réseau et de sécurité réseau open source conçue par [Tigera][tigera].

Ces deux implémentations appliquent les stratégies spécifiées à l’aide du logiciel Linux *IPTables*. Les stratégies sont converties en ensembles de paires d’adresses IP autorisées et interdites. Ces paires sont ensuite programmées sous la forme de règles de filtre IPTable.

### <a name="differences-between-azure-and-calico-policies-and-their-capabilities"></a>Différences entre les stratégies Azure et Calico et leurs fonctionnalités

| Fonctionnalité                               | Azure                      | Calico                      |
|------------------------------------------|----------------------------|-----------------------------|
| Plateformes prises en charge                      | Linux                      | Linux                       |
| Options de mise en réseau prises en charge             | Azure CNI                  | Azure CNI et kubenet       |
| Conformité à la spécification Kubernetes | Prise en charge de tous les types de stratégies |  Prise en charge de tous les types de stratégies |
| Fonctionnalités supplémentaires                      | None                       | Modèle de stratégie étendu composé d’une stratégie réseau globale, d’un ensemble réseau global et d’un point de terminaison d’hôte. Pour plus d’informations sur l’utilisation de la CLI `calicoctl` pour gérer ces fonctionnalités étendues, consultez les [informations de référence utilisateur concernant calicoctl][calicoctl]. |
| Support                                  | Pris en charge par l’équipe d’ingénierie et de support Azure | Support de la communauté Calico Pour plus d’informations sur les offres de support payantes supplémentaires, consultez l’article présentant les [options de support de Project Calico][calico-support]. |
| Journalisation                                  | Les règles ajoutées/supprimées dans IPTables sont journalisées sur chaque hôte sous */var/log/azure-npm.log* | Pour plus d’informations, consultez les [journaux de composant Calico][calico-logs]. |

## <a name="create-an-aks-cluster-and-enable-network-policy"></a>Créer un cluster AKS et activer la stratégie réseau

Pour voir les stratégies réseau en action, nous allons créer, puis développer une stratégie qui définit le flux de trafic :

* Refuser tout trafic sur un pod.
* Autoriser le trafic en fonction des étiquettes de pod.
* Autoriser le trafic en fonction de l’espace de noms.

Commençons par créer un cluster AKS qui prend en charge les stratégies réseau. 

> [!IMPORTANT]
>
> La fonctionnalité de stratégie réseau n’est activable qu’une fois le cluster créé. Vous ne pouvez pas activer une stratégie réseau sur un cluster AKS existant.

Pour utiliser une stratégie réseau Azure, vous devez utiliser le [plug-in Azure CNI][azure-cni] et définir vos propre réseau et sous-réseaux virtuels. Pour de plus amples informations sur la façon de planifier les plages de sous-réseau nécessaires, consultez [Configurer le réseau avancé][use-advanced-networking]. La stratégie réseau Calico peut être utilisée avec ce même plug-in Azure CNI ou avec le plug-in Kubenet CNI.

L’exemple de script suivant :

* Crée un réseau virtuel et un sous-réseau.
* Crée un principal de service Azure Active Directory (Azure AD) pour une utilisation avec le cluster AKS.
* Assigne des autorisations *Contributeur* pour le principal du service du cluster AKS sur le réseau virtuel.
* Crée un cluster AKS dans le réseau virtuel défini et active la stratégie réseau.
    * L’option de stratégie _Réseau Azure_ est utilisée. Pour utiliser Calico en tant qu’option de stratégie réseau à la place, utilisez le paramètre `--network-policy calico`. Remarque : Calico peut être utilisé avec `--network-plugin azure` ou `--network-plugin kubenet`.

Notez qu’au lieu d’utiliser un principal de service, vous pouvez utiliser une identité managée pour les autorisations. Pour plus d’informations, consultez [Utiliser des identités managées](use-managed-identity.md).

Fournissez votre propre *SP_PASSWORD* sécurisé. Vous pouvez remplacer les variables *RESOURCE_GROUP_NAME* et *CLUSTER_NAME* :

```azurecli-interactive
RESOURCE_GROUP_NAME=myResourceGroup-NP
CLUSTER_NAME=myAKSCluster
LOCATION=canadaeast

# Create a resource group
az group create --name $RESOURCE_GROUP_NAME --location $LOCATION

# Create a virtual network and subnet
az network vnet create \
    --resource-group $RESOURCE_GROUP_NAME \
    --name myVnet \
    --address-prefixes 10.0.0.0/8 \
    --subnet-name myAKSSubnet \
    --subnet-prefix 10.240.0.0/16

# Create a service principal and read in the application ID
SP=$(az ad sp create-for-rbac --output json)
SP_ID=$(echo $SP | jq -r .appId)
SP_PASSWORD=$(echo $SP | jq -r .password)

# Wait 15 seconds to make sure that service principal has propagated
echo "Waiting for service principal to propagate..."
sleep 15

# Get the virtual network resource ID
VNET_ID=$(az network vnet show --resource-group $RESOURCE_GROUP_NAME --name myVnet --query id -o tsv)

# Assign the service principal Contributor permissions to the virtual network resource
az role assignment create --assignee $SP_ID --scope $VNET_ID --role Contributor

# Get the virtual network subnet resource ID
SUBNET_ID=$(az network vnet subnet show --resource-group $RESOURCE_GROUP_NAME --vnet-name myVnet --name myAKSSubnet --query id -o tsv)

# Create the AKS cluster and specify the virtual network and service principal information
# Enable network policy by using the `--network-policy` parameter
az aks create \
    --resource-group $RESOURCE_GROUP_NAME \
    --name $CLUSTER_NAME \
    --node-count 1 \
    --generate-ssh-keys \
    --network-plugin azure \
    --service-cidr 10.0.0.0/16 \
    --dns-service-ip 10.0.0.10 \
    --docker-bridge-address 172.17.0.1/16 \
    --vnet-subnet-id $SUBNET_ID \
    --service-principal $SP_ID \
    --client-secret $SP_PASSWORD \
    --network-policy azure
```

La création du cluster ne prend que quelques minutes. Quand le cluster est prêt, configurez `kubectl` pour vous connecter à votre cluster Kubernetes au moyen de la commande [az aks get-credentials][az-aks-get-credentials]. Cette commande télécharge les informations d’identification et configure l’interface CLI Kubernetes pour les utiliser :

```azurecli-interactive
az aks get-credentials --resource-group $RESOURCE_GROUP_NAME --name $CLUSTER_NAME
```

## <a name="deny-all-inbound-traffic-to-a-pod"></a>Refuser tout trafic entrant sur un pod

Avant de définir des règles autorisant un trafic réseau particulier, commencez par créer une stratégie réseau pour refuser tout trafic. Cette stratégie vous donne un point de départ pour commencer à créer une liste verte uniquement pour le trafic souhaité. Vous pouvez aussi constater aisément que le trafic est ignoré lorsque la stratégie réseau est appliquée.

Pour l’exemple d’environnement d’application et de règles de trafic, commençons par créer un espace de noms nommé *development* (développement) pour l’exécution des exemples de pods :

```console
kubectl create namespace development
kubectl label namespace/development purpose=development
```

Créez un exemple de pod principal qui exécute NGINX. Ce pod principal peut être utilisé pour simuler un exemple d’application web principale. Créez ce pod dans l’espace de noms *development* et ouvrez le port *80* pour gérer le trafic web. Étiquetez le pod avec *app=webapp,role=backend* de façon à pouvoir le cibler avec une stratégie réseau dans la section suivante :

```console
kubectl run backend --image=nginx --labels app=webapp,role=backend --namespace development --expose --port 80
```

Créez un autre pod et attachez une session de terminal pour vérifier que vous accédez correctement à la page web NGINX par défaut :

```console
kubectl run --rm -it --image=alpine network-policy --namespace development
```

À l’invite de l’interpréteur de commandes, utilisez `wget` pour vérifier que vous pouvez accéder à la page web NGINX par défaut :

```console
wget -qO- http://backend
```

L’exemple de sortie ci-après affiche la page web NGINX par défaut renvoyée :

```output
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
[...]
```

Quittez la session de terminal attachée. Le pod de test est automatiquement supprimé.

```console
exit
```

### <a name="create-and-apply-a-network-policy"></a>Créer et appliquer une stratégie réseau

Maintenant que vous avez vérifié que vous pouvez utiliser la page web NGINX de base dans l’exemple de pod principal, créez une stratégie réseau destinée à refuser la totalité du trafic. Créez un fichier nommé `backend-policy.yaml` et collez le manifeste YAML suivant. Ce manifeste utilise un élément *podSelector* pour attacher la stratégie aux pods dotés de l’étiquette *app:webapp,role:backend*, comme votre exemple de pod NGINX. Aucune règle n’est définie sous *ingress* (entrée), tout trafic entrant sur le pod est donc refusé :

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: backend-policy
  namespace: development
spec:
  podSelector:
    matchLabels:
      app: webapp
      role: backend
  ingress: []
```

Accédez à [https://shell.azure.com](https://shell.azure.com) pour ouvrir Azure Cloud Shell dans votre navigateur.

Appliquez la stratégie réseau à l’aide de la commande [kubectl apply][kubectl-apply] et précisez le nom de votre manifeste YAML :

```console
kubectl apply -f backend-policy.yaml
```

### <a name="test-the-network-policy"></a>Tester la stratégie réseau

Voyons si vous pouvez réutiliser la page web NGINX sur le pod principal. Créez un autre pod de test et attachez une session de terminal :

```console
kubectl run --rm -it --image=alpine network-policy --namespace development
```

À l’invite de l’interpréteur de commandes, utilisez `wget` pour voir si vous pouvez accéder à la page web NGINX par défaut. Cette fois, définissez une valeur de délai d’attente sur *2* secondes. La stratégie réseau bloque à présent tout le trafic entrant, de sorte que la page ne peut pas être chargée, comme illustré dans l’exemple suivant :

```console
wget -qO- --timeout=2 http://backend
```

```output
wget: download timed out
```

Quittez la session de terminal attachée. Le pod de test est automatiquement supprimé.

```console
exit
```

## <a name="allow-inbound-traffic-based-on-a-pod-label"></a>Autoriser le trafic entrant en fonction d’une étiquette de pod

À la section précédente, nous avons planifié un pod NGINX principal et créé une stratégie réseau destinée à refuser la totalité du trafic. Créons à présent un pod frontal et mettons à jour la stratégie réseau de manière à autoriser le trafic provenant de pods frontaux.

Mettez à jour la stratégie réseau pour autoriser le trafic provenant des pods étiquetés *app:webapp,role:frontend* et dans n’importe quel espace de noms. Modifiez le précédent fichier *backend-policy.yaml* en y ajoutant des règles d’entrée *matchLabels* afin que votre manifeste ressemble à l’exemple suivant :

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: backend-policy
  namespace: development
spec:
  podSelector:
    matchLabels:
      app: webapp
      role: backend
  ingress:
  - from:
    - namespaceSelector: {}
      podSelector:
        matchLabels:
          app: webapp
          role: frontend
```

> [!NOTE]
> Cette stratégie de réseau utilise un élément *namespaceSelector* et un élément *podSelector* pour la règle d’entrée. Pour que les règles d’entrée soient additives, la syntaxe YAML a une grande importance. Dans cet exemple, les deux éléments doivent correspondre pour que la règle d’entrée soit appliquée. Les versions de Kubernetes antérieures à *1.12* risquent de ne pas interpréter correctement ces éléments et de ne pas limiter le trafic réseau comme vous le souhaitez. Pour plus d’informations sur ce comportement, consultez l’article [Behavior of to and from selectors][policy-rules] (Comportement des sélecteurs to et from).

Appliquez la stratégie réseau mise à jour à l’aide de la commande [kubectl apply][kubectl-apply] et spécifiez le nom de votre manifeste YAML :

```console
kubectl apply -f backend-policy.yaml
```

Planifiez un pod portant l’étiquette *app=webapp,role=frontend* et attachez une session de terminal :

```console
kubectl run --rm -it frontend --image=alpine --labels app=webapp,role=frontend --namespace development
```

À l’invite de l’interpréteur de commandes, utilisez `wget` pour voir si vous pouvez accéder à la page web NGINX par défaut :

```console
wget -qO- http://backend
```

Étant donné que la règle d’entrée autorise le trafic avec les pods étiquetés *app: webapp,role: frontend*, le trafic provenant du pod frontal est autorisé. L’exemple de sortie ci-après affiche la page web NGINX par défaut renvoyée :

```output
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
[...]
```

Quittez la session de terminal attachée. Le pod est automatiquement supprimé.

```console
exit
```

### <a name="test-a-pod-without-a-matching-label"></a>Tester un pod sans étiquette correspondante

La stratégie réseau autorise le trafic provenant des pods étiquetés *app: webapp,role: frontend*, mais doit refuser tout autre trafic. Vérifions si un autre pod dépourvu de cette étiquette peut ou non accéder au pod NGINX principal. Créez un autre pod de test et attachez une session de terminal :

```console
kubectl run --rm -it --image=alpine network-policy --namespace development
```

À l’invite de l’interpréteur de commandes, utilisez `wget` pour voir si vous pouvez accéder à la page web NGINX par défaut. La stratégie réseau bloque le trafic entrant, de sorte que la page ne peut pas être chargée, comme indiqué dans l’exemple suivant :

```console
wget -qO- --timeout=2 http://backend
```

```output
wget: download timed out
```

Quittez la session de terminal attachée. Le pod de test est automatiquement supprimé.

```console
exit
```

## <a name="allow-traffic-only-from-within-a-defined-namespace"></a>Autoriser le trafic uniquement à partir d’un espace de noms défini

Dans les exemples précédents, vous avez créé une stratégie réseau qui refusait tout trafic, puis vous avez mis à jour cette stratégie afin d’autoriser le trafic provenant de pods dotés d’une étiquette spécifique. Par ailleurs, il se révèle souvent utile de limiter le trafic à un seul espace de noms. Si les précédents exemples se rapportaient au trafic dans un espace de noms *development*, vous allez à présent créer une stratégie réseau qui empêche le trafic provenant d’un autre espace de noms, tel que *production*, d’atteindre les pods.

En premier lieu, créez un espace de noms pour simuler un espace de noms de production :

```console
kubectl create namespace production
kubectl label namespace/production purpose=production
```

Planifiez un pod de test dans l’espace de noms *production* qui est étiqueté en tant que *app=webapp,role=frontend*. Attachez une session de terminal :

```console
kubectl run --rm -it frontend --image=alpine --labels app=webapp,role=frontend --namespace production
```

À l’invite de l’interpréteur de commandes, utilisez `wget` pour vérifier que vous pouvez accéder à la page web NGINX par défaut :

```console
wget -qO- http://backend.development
```

Dans la mesure où l’étiquette du pod correspond à ce qui est actuellement accordé dans la stratégie réseau, le trafic est autorisé. La stratégie réseau n’examine pas les espaces de noms, elle ne tient compte que des étiquettes de pod. L’exemple de sortie ci-après affiche la page web NGINX par défaut renvoyée :

```output
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
[...]
```

Quittez la session de terminal attachée. Le pod de test est automatiquement supprimé.

```console
exit
```

### <a name="update-the-network-policy"></a>Mettre à jour la stratégie réseau

Mettons à jour la section *namespaceSelector* de la règle d’entrée de manière à n’autoriser que le trafic provenant de l’espace de noms *development*. Modifiez le fichier manifeste *backend-policy.yaml*, comme indiqué dans l’exemple suivant :

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: backend-policy
  namespace: development
spec:
  podSelector:
    matchLabels:
      app: webapp
      role: backend
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          purpose: development
      podSelector:
        matchLabels:
          app: webapp
          role: frontend
```

Dans des exemples plus complexes, vous pourriez définir plusieurs règles d’entrée, par exemple un élément *namespaceSelector*, puis un élément *podSelector*.

Appliquez la stratégie réseau mise à jour à l’aide de la commande [kubectl apply][kubectl-apply] et spécifiez le nom de votre manifeste YAML :

```console
kubectl apply -f backend-policy.yaml
```

### <a name="test-the-updated-network-policy"></a>Tester la stratégie réseau mise à jour

Planifiez un autre pod dans l’espace de noms *production* et attachez une session de terminal :

```console
kubectl run --rm -it frontend --image=alpine --labels app=webapp,role=frontend --namespace production
```

À l’invite de l’interpréteur de commandes, utilisez `wget` pour vérifier que la stratégie réseau refuse désormais le trafic :

```console
wget -qO- --timeout=2 http://backend.development
```

```output
wget: download timed out
```

Quittez le pod de test :

```console
exit
```

En maintenant le refus sur le trafic provenant de l’espace de noms *production*, replanifiez un pod de test dans l’espace de noms *development* et attachez une session de terminal :

```console
kubectl run --rm -it frontend --image=alpine --labels app=webapp,role=frontend --namespace development
```

À l’invite de l’interpréteur de commandes, utilisez `wget` pour vérifier que la stratégie réseau autorise le trafic :

```console
wget -qO- http://backend
```

Le trafic est autorisé, car le pod est planifié dans l’espace de noms qui correspond à ce qui est autorisé dans la stratégie réseau. L’exemple de sortie ci-après affiche la page web NGINX par défaut renvoyée :

```output
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
[...]
```

Quittez la session de terminal attachée. Le pod de test est automatiquement supprimé.

```console
exit
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Dans cet article, nous avons créé deux espaces de noms et appliqué une stratégie réseau. Pour nettoyer ces ressources, utilisez la commande [kubectl delete][kubectl-delete] et spécifiez les noms des ressources :

```console
kubectl delete namespace production
kubectl delete namespace development
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les ressources réseau, consultez l’article [Concepts de réseau pour les applications dans AKS (Azure Kubernetes Service)][concepts-network].

Pour plus d’informations sur les stratégies, consultez l’article [Kubernetes network policies][kubernetes-network-policies] (Stratégies réseau Kubernetes).

<!-- LINKS - external -->
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-delete]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#delete
[kubernetes-network-policies]: https://kubernetes.io/docs/concepts/services-networking/network-policies/
[azure-cni]: https://github.com/Azure/azure-container-networking/blob/master/docs/cni.md
[policy-rules]: https://kubernetes.io/docs/concepts/services-networking/network-policies/#behavior-of-to-and-from-selectors
[aks-github]: https://github.com/azure/aks/issues
[tigera]: https://www.tigera.io/
[calicoctl]: https://docs.projectcalico.org/reference/calicoctl/
[calico-support]: https://www.tigera.io/tigera-products/calico/
[calico-logs]: https://docs.projectcalico.org/maintenance/troubleshoot/component-logs
[calico-aks-cleanup]: https://github.com/Azure/aks-engine/blob/master/docs/topics/calico-3.3.1-cleanup-after-upgrade.yaml

<!-- LINKS - internal -->
[install-azure-cli]: /cli/azure/install-azure-cli
[use-advanced-networking]: configure-azure-cni.md
[az-aks-get-credentials]: /cli/azure/aks?view=azure-cli-latest#az-aks-get-credentials
[concepts-network]: concepts-network.md
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-feature-list]: /cli/azure/feature#az-feature-list
[az-provider-register]: /cli/azure/provider#az-provider-register
