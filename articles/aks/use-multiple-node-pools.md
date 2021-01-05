---
title: Utiliser plusieurs pools de nœuds dans Azure Kubernetes Service (AKS)
description: Découvrez comment créer et gérer plusieurs pools de nœuds pour un cluster dans Azure Kubernetes Service (AKS)
services: container-service
ms.topic: article
ms.date: 04/08/2020
ms.openlocfilehash: 39c2fe177d0a6d913d7bf2b2baf44af3c69c0868
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96006931"
---
# <a name="create-and-manage-multiple-node-pools-for-a-cluster-in-azure-kubernetes-service-aks"></a>Créer et gérer plusieurs pools de nœuds pour un cluster dans Azure Kubernetes Service (AKS)

Dans Azure Kubernetes Service (AKS), les nœuds d’une même configuration sont regroupés dans des *pools de nœuds*. Ces pools de nœuds contiennent les machines virtuelles sous-jacentes qui exécutent vos applications. Le nombre initial de nœuds et leur taille (référence SKU) sont définis lorsque vous créez un cluster AKS, opération qui crée un [pool de nœuds système][use-system-pool]. Pour prendre en charge les applications qui ont des exigences de calcul ou de stockage différentes, vous pouvez créer des *pools de nœuds utilisateur* supplémentaires. Les pools de nœuds système sont principalement utilisés pour héberger des pods système critiques, tels que CoreDNS et tunnelfront. Les pools de nœuds utilisateur sont principalement utilisés pour héberger vos pods d'application. Cela étant, les pods d’application peuvent être planifiés sur des pools de nœuds système si vous souhaitez n'avoir qu’un pool dans votre cluster AKS. Les pools de nœuds utilisateur sont l’endroit où vous placez les pods propres à votre application. Par exemple, utilisez ces pools de nœuds utilisateur supplémentaires pour fournir des GPU pour les applications nécessitant beaucoup de ressources système ou pour accéder à un stockage SSD hautes performances.

> [!NOTE]
> Cette fonctionnalité permet de mieux contrôler la création et la gestion de pools de nœuds multiples. Par conséquent, des commandes distinctes sont requises pour créer/mettre à jour/supprimer. Auparavant, les opérations de cluster via `az aks create` ou `az aks update` utilisaient l’API managedCluster et étaient la seule option pour modifier votre plan de contrôle et un pool de nœuds unique. Cette fonctionnalité expose une opération distincte définie pour les pools d’agents via l’API agentPool et requiert l’utilisation du jeu de commandes `az aks nodepool` pour exécuter des opérations sur un pool de nœuds individuel.

Cet article vous montre comment créer et gérer plusieurs pools de nœuds dans un cluster AKS.

## <a name="before-you-begin"></a>Avant de commencer

La version 2.2.0 (ou ultérieure) d’Azure CLI doit être installée et configurée. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI][install-azure-cli].

## <a name="limitations"></a>Limites

Les limitations suivantes s’appliquent lorsque vous créez et gérez les clusters AKS prenant en charge plusieurs pools de nœuds :

* Voir [Quotas, restrictions de taille de machine virtuelle et disponibilité des régions dans Azure Kubernetes Service (AKS)][quotas-skus-regions].
* Vous pouvez supprimer un pool de nœuds système à condition de disposer d’un autre pool de nœuds système pour prendre sa place dans le cluster AKS.
* Les pools de nœuds système doivent contenir au moins un nœud, et les pools de nœuds utilisateur peuvent contenir zéro, un ou plusieurs nœuds.
* Le cluster AKS doit utiliser l’équilibreur de charge de la référence SKU Standard pour utiliser plusieurs pools de nœuds, la fonctionnalité n’est pas prise en charge avec les équilibreurs de charge de la référence SKU De base.
* Le cluster AKS doit utiliser des groupes de machines virtuelles identiques pour les nœuds.
* Le nom d’un pool de nœuds ne peut contenir que des caractères alphanumériques minuscules et doit commencer par une lettre minuscule. Pour les pools de nœuds Linux, la longueur doit être comprise entre 1 et 12 caractères. Pour les pools de nœuds Windows, elle doit être comprise entre 1 et 6 caractères.
* Tous les pools de nœuds doivent résider dans le même réseau virtuel.
* Lors de la création de plusieurs pools de nœuds au moment de la création du cluster, toutes les versions de Kubernetes utilisées par les pools de nœuds doivent correspondre à la version définie pour le plan de contrôle. Cela peut être mis à jour une fois que le cluster a été provisionné, en utilisant des opérations par pool de nœuds.

## <a name="create-an-aks-cluster"></a>Créer un cluster AKS

> [!Important]
> Si vous exécutez un pool de nœuds système unique pour votre cluster AKS dans un environnement de production, nous vous recommandons d’utiliser au moins trois nœuds pour le pool de nœuds.

Pour commencer, créez un cluster AKS avec un pool de nœuds unique. L’exemple suivant utilise la commande [az group create][az-group-create] pour créer un groupe de ressources nommé *myResourceGroup* dans la région *eastus*. Un cluster AKS nommé *myAKSCluster* est alors créé à l’aide de la commande [az aks create][az-aks-create].

> [!NOTE]
> La référence SKU d’équilibreur de charge *De base* **n’est pas prise en charge** en cas d’utilisation de plusieurs pools de nœuds. Par défaut, les clusters AKS sont créés avec la référence SKU d'équilibreur de charge *Standard* à partir d'Azure CLI et du portail Azure.

```azurecli-interactive
# Create a resource group in East US
az group create --name myResourceGroup --location eastus

# Create a basic single-node AKS cluster
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --vm-set-type VirtualMachineScaleSets \
    --node-count 2 \
    --generate-ssh-keys \
    --load-balancer-sku standard
```

La création du cluster ne prend que quelques minutes.

> [!NOTE]
> Pour vous assurer de la fiabilité de votre cluster, vous devez exécuter au moins 2 (deux) nœuds dans le pool de nœuds par défaut, car les services système essentiels sont exécutés sur ce pool de nœuds.

Lorsque le cluster est prêt, utilisez la commande [az aks get-credentials][az-aks-get-credentials] pour obtenir les informations d’identification du cluster à utiliser avec `kubectl` :

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

## <a name="add-a-node-pool"></a>Ajouter un pool de nœuds

Le cluster créé à l’étape précédente dispose d’un pool de nœuds unique. Nous allons ajouter un deuxième pool de nœuds à l’aide de la commande [az aks nodepool add][az-aks-nodepool-add]. L’exemple suivant crée un pool de nœuds nommé *mynodepool* qui exécute *3* nœuds :

```azurecli-interactive
az aks nodepool add \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name mynodepool \
    --node-count 3
```

> [!NOTE]
> Le nom d’un pool de nœuds doit commencer par une lettre minuscule et ne peut contenir que des caractères alphanumériques. Pour les pools de nœuds Linux, la longueur doit être comprise entre 1 et 12 caractères. Pour les pools de nœuds Windows, elle doit être comprise entre 1 et 6 caractères.

Pour afficher l’état de vos pools de nœuds, utilisez la commande [az aks node pool list][az-aks-nodepool-list] et spécifiez votre groupe de ressources et le nom du cluster :

```azurecli-interactive
az aks nodepool list --resource-group myResourceGroup --cluster-name myAKSCluster
```

L’exemple de sortie suivant montre que *mynodepool* a été créé avec trois nœuds dans le pool de nœuds. Lorsque le cluster AKS a été créé à l'étape précédente, un pool *nodepool1* par défaut a été généré avec *2* nœuds.

```output
[
  {
    ...
    "count": 3,
    ...
    "name": "mynodepool",
    "orchestratorVersion": "1.15.7",
    ...
    "vmSize": "Standard_DS2_v2",
    ...
  },
  {
    ...
    "count": 2,
    ...
    "name": "nodepool1",
    "orchestratorVersion": "1.15.7",
    ...
    "vmSize": "Standard_DS2_v2",
    ...
  }
]
```

> [!TIP]
> Si aucune taille *VmSize* n’est spécifiée lorsque vous ajoutez un pool de nœuds, la taille par défaut est *Standard_D2s_v3* pour les pools de nœuds Windows et *Standard_DS2_v2* pour les pools de nœuds Linux. Si aucune version *OrchestratorVersion* n’est spécifiée, la version par défaut est identique à celle du plan de contrôle.

### <a name="add-a-node-pool-with-a-unique-subnet-preview"></a>Ajouter un pool de nœuds avec un sous-réseau unique (préversion)

Une charge de travail peut nécessiter le fractionnement des nœuds d'un cluster en pools distincts à des fins d'isolation logique. Cette isolation peut être prise en charge par des sous-réseaux distincts dédiés à chaque pool de nœuds du cluster. Cela peut répondre à des exigences telles que l'utilisation d'un espace d'adressage de réseau virtuel non contigu à fractionner entre les pools de nœuds.

#### <a name="limitations"></a>Limites

* Tous les sous-réseaux attribués aux pools de nœuds doivent appartenir au même réseau virtuel.
* Les pods système doivent avoir accès à tous les nœuds du cluster pour fournir les fonctionnalités critiques telles que la résolution DNS via coreDNS.
* Dans le cadre de la préversion, l'attribution d'un sous-réseau unique par pool de nœuds est limitée à Azure CNI.
* L'utilisation de stratégies réseau avec un sous-réseau unique par pool de nœuds n'est pas prise en charge dans le cadre de la préversion.

Pour créer un pool de nœuds avec un sous-réseau dédié, transmettez l'ID de ressource du sous-réseau en tant que paramètre supplémentaire lors de la création du pool de nœuds.

```azurecli-interactive
az aks nodepool add \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name mynodepool \
    --node-count 3 \
    --vnet-subnet-id <YOUR_SUBNET_RESOURCE_ID>
```

## <a name="upgrade-a-node-pool"></a>Mettre à niveau un pool de nœuds

> [!NOTE]
> Les opérations de mise à niveau et de mise à l’échelle sur un cluster ou un pool de nœuds ne peuvent pas se produire simultanément. Toute tentative de les exécuter simultanément retourne une erreur. En effet, chaque opération doit être terminée sur la ressource cible avant l’exécution de la demande suivante sur cette même ressource. Pour en savoir plus, voir notre [Guide de résolution des problèmes](./troubleshooting.md#im-receiving-errors-when-trying-to-upgrade-or-scale-that-state-my-cluster-is-being-upgraded-or-has-failed-upgrade).

Les commandes de cette section expliquent comment mettre à niveau un seul pool de nœuds. La relation entre la mise à niveau de la version Kubernetes du plan de contrôle et le pool de est décrite [ci-dessous](#upgrade-a-cluster-control-plane-with-multiple-node-pools).

> [!NOTE]
> La version de l'image de système d'exploitation du pool de nœuds est liée à la version Kubernetes du cluster. Vous n'obtiendrez les mises à niveau de l'image de système d'exploitation qu'après une mise à niveau du cluster.

Comme il existe deux pools de nœuds dans cet exemple, nous devons utiliser la commande [az aks nodepool upgrade][az-aks-nodepool-upgrade] pour mettre à niveau un pool de nœuds. Pour voir les mises à niveau disponibles, utilisez [az aks get-upgrades][az-aks-get-upgrades]

```azurecli-interactive
az aks get-upgrades --resource-group myResourceGroup --name myAKSCluster
```

Nous allons mettre à niveau *mynodepool*. Utilisez la commande [az aks nodepool upgrade][az-aks-nodepool-upgrade] pour mettre à niveau le pool de nœuds, comme dans l’exemple suivant :

```azurecli-interactive
az aks nodepool upgrade \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name mynodepool \
    --kubernetes-version KUBERNETES_VERSION \
    --no-wait
```

Listez de nouveau l’état de vos pools de nœuds à l’aide de la commande [az aks node pool list][az-aks-nodepool-list]. L’exemple suivant montre que *mynodepool* est en cours de *mise à niveau* (état Upgrading) vers *KUBERNETES_VERSION* :

```azurecli
az aks nodepool list -g myResourceGroup --cluster-name myAKSCluster
```

```output
[
  {
    ...
    "count": 3,
    ...
    "name": "mynodepool",
    "orchestratorVersion": "KUBERNETES_VERSION",
    ...
    "provisioningState": "Upgrading",
    ...
    "vmSize": "Standard_DS2_v2",
    ...
  },
  {
    ...
    "count": 2,
    ...
    "name": "nodepool1",
    "orchestratorVersion": "1.15.7",
    ...
    "provisioningState": "Succeeded",
    ...
    "vmSize": "Standard_DS2_v2",
    ...
  }
]
```

Il faut quelques minutes pour mettre à niveau les nœuds vers la version spécifiée.

En guise de bonne pratique, vous devez mettre à niveau tous les pools de nœuds dans un cluster AKS vers la même version de Kubernetes. Le comportement par défaut de `az aks upgrade` est de mettre à niveau tous les pools de nœuds en même temps que le plan de contrôle pour obtenir cet alignement. La possibilité de mettre à niveau les pools de nœuds individuels vous permet d’effectuer une mise à niveau propagée et de planifier des pods entre les pools de nœuds pour maintenir la disponibilité des applications au sein des contraintes susmentionnées.

## <a name="upgrade-a-cluster-control-plane-with-multiple-node-pools"></a>Mettre à niveau un plan de contrôle de cluster avec plusieurs pools de nœuds

> [!NOTE]
> Kubernetes utilise le schéma de contrôle de version standard [Semantic Versioning](https://semver.org/). Le numéro de version est exprimé par *x.y.z*, où *x* est la version principale, *y* est la version secondaire et *z* est la version du correctif. Par exemple, dans la version *1.12.6*, 1 est la version principale, 12 est la version secondaire, et 6 est la version du correctif. La version Kubernetes du plan de contrôle et le pool de nœuds initial sont définis lors de la création du cluster. Tous les pools de nœuds supplémentaires ont leur version Kubernetes définie lorsqu'ils sont ajoutés au cluster. Les versions Kubernetes peuvent différer entre les pools de nœuds ainsi qu’entre un pool de nœuds et le plan de contrôle.

Un cluster AKS possède deux objets de ressource de cluster avec des versions Kubernetes associées.

1. Un plan de contrôle de cluster est une version Kubernetes du plan de contrôle.
2. Pool de nœuds avec une version de Kubernetes.

Un plan de contrôle est mappé à un ou plusieurs pools de nœuds. Le comportement d’une opération de mise à niveau dépend de la commande Azure CLI utilisée.

La mise à niveau d’un plan de contrôle AKS requiert l'utilisation de `az aks upgrade`. Cette commande met à niveau la version du plan de contrôle et tous les pools de nœuds du cluster.

L’émission de la commande `az aks upgrade` avec l’indicateur `--control-plane-only` ne met à niveau que le plan de contrôle du cluster. Aucun des pools de nœuds associés dans le cluster n’est modifié.

La mise à niveau de pools de nœuds individuels requiert l’utilisation de `az aks nodepool upgrade`. Cette commande met uniquement à niveau le pool de nœuds cible avec la version de Kubernetes spécifiée.

### <a name="validation-rules-for-upgrades"></a>Règles de validation pour les mises à niveau

Les mises à niveau valides de Kubernetes du plan de contrôle ou des pools de nœuds d’un cluster sont validées par les ensembles de règles suivants.

* Règles pour la mise à niveau des pools de nœuds par les versions valides :
   * La version du pool de nœuds doit avoir la même version *principale* que le plan de contrôle.
   * La version *secondaire* du pool de nœuds doit être située à deux versions *secondaires* de la version du plan de contrôle.
   * La version du pool de nœuds ne peut pas être supérieure à la version du `major.minor.patch` de contrôle.

* Règles pour l’envoi d’une opération de mise à niveau :
   * Vous ne pouvez pas passer à une version de Kubernetes antérieure pour le plan de contrôle ou le pool de nœuds.
   * Si une version de Kubernetes de pool de nœuds n’est pas spécifiée, le comportement dépend du client utilisé. Dans les modèles Resource Manager, la déclaration revient à la version existante définie pour le pool de nœuds si elle est utilisée. Si aucune valeur n’est définie, la version du plan de contrôle est utilisée dans le cadre du rétablissement de la version antérieure.
   * Vous pouvez procéder à la mise à niveau ou à la mise à l'échelle d'un plan de contrôle ou d'un pool de nœuds à un moment donné, mais vous ne pouvez pas soumettre plusieurs opérations sur un plan de contrôle ou une ressource de pools simultanément.

## <a name="scale-a-node-pool-manually"></a>Mettre manuellement à l'échelle un pool de nœuds

Lorsque vos exigences de charge de travail d’application changent, vous pouvez être amené à mettre à l’échelle le nombre de nœuds dans un pool de nœuds. Le nombre de nœuds peut être augmenté ou diminué.

<!--If you scale down, nodes are carefully [cordoned and drained][kubernetes-drain] to minimize disruption to running applications.-->

Pour mettre à l’échelle le nombre de nœuds dans un pool de nœuds, utilisez la commande [az aks node pool scale][az-aks-nodepool-scale]. L’exemple suivant met à l’échelle le nombre de nœuds dans *mynodepool* sur *5* :

```azurecli-interactive
az aks nodepool scale \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name mynodepool \
    --node-count 5 \
    --no-wait
```

Listez de nouveau l’état de vos pools de nœuds à l’aide de la commande [az aks node pool list][az-aks-nodepool-list]. L’exemple suivant montre que *mynodepool* est dans l’état *Upgrading (Mise à niveau)* avec une nouvelle valeur de *5* nœuds :

```azurecli
az aks nodepool list -g myResourceGroup --cluster-name myAKSCluster
```

```output
[
  {
    ...
    "count": 5,
    ...
    "name": "mynodepool",
    "orchestratorVersion": "1.15.7",
    ...
    "provisioningState": "Scaling",
    ...
    "vmSize": "Standard_DS2_v2",
    ...
  },
  {
    ...
    "count": 2,
    ...
    "name": "nodepool1",
    "orchestratorVersion": "1.15.7",
    ...
    "provisioningState": "Succeeded",
    ...
    "vmSize": "Standard_DS2_v2",
    ...
  }
]
```

L’opération de mise à l’échelle peut prendre quelques minutes.

## <a name="scale-a-specific-node-pool-automatically-by-enabling-the-cluster-autoscaler"></a>Mettre automatiquement à l’échelle un pool de nœuds spécifique en activant l’option de mise à l’échelle automatique de cluster

AKS offre une fonctionnalité distincte pour mettre automatiquement à l’échelle les pools de nœuds avec une fonctionnalité appelée [mise à l’échelle automatique du cluster](cluster-autoscaler.md). Cette fonctionnalité peut être activée pour chaque pool de nœuds avec un nombre minimal et maximal de mise à l’échelle par pool de nœuds. Découvrez comment [utiliser la mise à l’échelle automatique de cluster par pool de nœuds](cluster-autoscaler.md#use-the-cluster-autoscaler-with-multiple-node-pools-enabled).

## <a name="delete-a-node-pool"></a>Supprimer un pool de nœuds

Si vous n’avez plus besoin d’un pool, vous pouvez le supprimer et supprimer les nœuds des machines virtuelles sous-jacentes. Pour supprimer un pool de nœuds, utilisez la commande [az aks node pool delete][az-aks-nodepool-delete] et spécifiez le nom du pool de nœuds. L’exemple suivant supprime le pool *mynodepool* créé au cours des étapes précédentes :

> [!CAUTION]
> Il n’existe aucune option de récupération pour les pertes de données qui peuvent se produire lorsque vous supprimez un pool de nœuds. Si des pods ne peuvent pas être planifiés sur d’autres pools de nœuds, ces applications ne sont pas disponibles. Veillez à ne pas supprimer un pool de nœuds lorsque des applications en cours d’utilisation n’ont pas de sauvegardes de données ou la possibilité de s’exécuter sur d’autres pools de nœuds dans votre cluster.

```azurecli-interactive
az aks nodepool delete -g myResourceGroup --cluster-name myAKSCluster --name mynodepool --no-wait
```

L’exemple de sortie suivant de la commande [az aks node pool list][az-aks-nodepool-list] montre que *mynodepool* est dans l’état *Deleting (Suppression)*  :

```azurecli
az aks nodepool list -g myResourceGroup --cluster-name myAKSCluster
```

```output
[
  {
    ...
    "count": 5,
    ...
    "name": "mynodepool",
    "orchestratorVersion": "1.15.7",
    ...
    "provisioningState": "Deleting",
    ...
    "vmSize": "Standard_DS2_v2",
    ...
  },
  {
    ...
    "count": 2,
    ...
    "name": "nodepool1",
    "orchestratorVersion": "1.15.7",
    ...
    "provisioningState": "Succeeded",
    ...
    "vmSize": "Standard_DS2_v2",
    ...
  }
]
```

La suppression des nœuds et du pool de nœuds prend quelques minutes.

## <a name="specify-a-vm-size-for-a-node-pool"></a>Spécifier une taille de machine virtuelle pour un pool de nœuds

Dans les exemples précédents de création d’un pool de nœuds, une taille de machine virtuelle par défaut a été utilisée pour les nœuds créés dans le cluster. Un scénario plus courant consiste à créer des pools de nœuds avec des fonctionnalités et des tailles de machine virtuelle différentes. Par exemple, vous pouvez créer un pool de nœuds contenant des nœuds avec de grandes quantités de mémoire ou de processeur, ou un pool de nœuds fournissant une prise en charge GPU. Dans l’étape suivante, vous allez [utiliser des teintes et des tolérances](#setting-nodepool-taints) pour indiquer au planificateur Kubernetes comment limiter l’accès aux pods qui peuvent s’exécuter sur ces nœuds.

Dans l’exemple suivant, créez un pool de nœuds basé sur GPU qui utilise la taille de machine virtuelle *Standard_NC6*. Ces machines virtuelles sont alimentées par la carte NVIDIA Tesla K80. Pour obtenir des informations sur les tailles de machine virtuelle disponibles, consultez [Tailles des machines virtuelles Linux dans Azure][vm-sizes].

Créez un pool de nœuds de nouveau à l’aide de la commande [az aks node pool add][az-aks-nodepool-add]. Cette fois, spécifiez le nom *gpunodepool* et utilisez le paramètre `--node-vm-size` pour spécifier la taille *Standard_NC6* :

```azurecli-interactive
az aks nodepool add \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name gpunodepool \
    --node-count 1 \
    --node-vm-size Standard_NC6 \
    --no-wait
```

L’exemple de sortie suivant de la commande [az aks node pool list][az-aks-nodepool-list] montre que *gpunodepool* crée (*Creating*) des nœuds avec la taille *VmSize* spécifiée :

```azurecli
az aks nodepool list -g myResourceGroup --cluster-name myAKSCluster
```

```output
[
  {
    ...
    "count": 1,
    ...
    "name": "gpunodepool",
    "orchestratorVersion": "1.15.7",
    ...
    "provisioningState": "Creating",
    ...
    "vmSize": "Standard_NC6",
    ...
  },
  {
    ...
    "count": 2,
    ...
    "name": "nodepool1",
    "orchestratorVersion": "1.15.7",
    ...
    "provisioningState": "Succeeded",
    ...
    "vmSize": "Standard_DS2_v2",
    ...
  }
]
```

Il faut quelques minutes pour que *gpunodepool* soit créé avec succès.

## <a name="specify-a-taint-label-or-tag-for-a-node-pool"></a>Spécifier une teinte, un intitulé ou une étiquette pour un pool de nœuds

### <a name="setting-nodepool-taints"></a>Définition de teintes de pool de nœud

Lorsque vous créez un pool de nœuds, vous pouvez lui ajouter des teintes, des intitulés ou des étiquettes. Lorsque vous ajoutez une teinte, un intitulé ou une étiquette, tous les nœuds du pool reçoivent cette teinte, cet intitulé ou cette étiquette.

Pour créer un pool de nœuds à l’aide d’une teinte, utilisez la commande [az aks nodepool add][az-aks-nodepool-add]. Spécifiez le nom *taintnp* et utilisez le paramètre `--node-taints` afin de spécifier *sku=gpu:NoSchedule* pour la teinte.

```azurecli-interactive
az aks nodepool add \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name taintnp \
    --node-count 1 \
    --node-taints sku=gpu:NoSchedule \
    --no-wait
```

> [!NOTE]
> Une teinte ne peut être définie pour un pool de nœuds que pendant la création de celui-ci.

L’exemple de sortie suivant de la commande [az aks node pool list][az-aks-nodepool-list] montre que *taintnp* crée (*Creating*) des nœuds avec les teintes (*nodeTaints*) spécifiées :

```console
$ az aks nodepool list -g myResourceGroup --cluster-name myAKSCluster

[
  {
    ...
    "count": 1,
    ...
    "name": "taintnp",
    "orchestratorVersion": "1.15.7",
    ...
    "provisioningState": "Creating",
    ...
    "nodeTaints":  [
      "sku=gpu:NoSchedule"
    ],
    ...
  },
 ...
]
```

Les informations concernant la teinte sont visibles dans Kubernetes pour la gestion des règles de planification des nœuds. Le planificateur Kubernetes peut utiliser des teintes et des tolérances pour restreindre les charges de travail qui peuvent s’exécuter sur des nœuds.

* Quand une **teinte** est appliquée à un nœud, seuls des pods spécifiques peuvent être planifiés sur le nœud.
* Une **tolérance** est ensuite appliquée à un pod pour lui permettre de *tolérer* la teinte d’un nœud.

Pour plus d’informations sur la façon d’utiliser les fonctionnalités de Kubernetes planifiées avancées, consultez [Bonnes pratiques relatives aux fonctionnalités avancées du planificateur dans AKS][taints-tolerations].

À l’étape précédente, vous avez appliqué la teinte *sku=gpu:NoSchedule* lorsque vous avez créé votre pool de nœuds. L’exemple de manifeste YAML de base suivant utilise une tolérance pour autoriser le planificateur Kubernetes à exécuter un pod NGINX sur un nœud de ce pool de nœuds.

Créez un fichier nommé `nginx-toleration.yaml` et copiez-y l’exemple de code YAML suivant :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - image: mcr.microsoft.com/oss/nginx/nginx:1.15.9-alpine
    name: mypod
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 1
        memory: 2G
  tolerations:
  - key: "sku"
    operator: "Equal"
    value: "gpu"
    effect: "NoSchedule"
```

Planifiez le pod à l’aide de la commande `kubectl apply -f nginx-toleration.yaml` :

```console
kubectl apply -f nginx-toleration.yaml
```

Il faut quelques secondes pour planifier le pod et extraire l’image NGINX. Utilisez la commande [kubectl describe pod][kubectl-describe] pour voir l’état du pod. L’exemple de sortie condensée suivant montre que la tolérance *sku=gpu:NoSchedule* est appliquée. Dans la section des événements, le planificateur a affecté le pod au nœud *aks-taintnp-28993262-vmss000000* :

```console
kubectl describe pod mypod
```

```output
[...]
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
                 sku=gpu:NoSchedule
Events:
  Type    Reason     Age    From                Message
  ----    ------     ----   ----                -------
  Normal  Scheduled  4m48s  default-scheduler   Successfully assigned default/mypod to aks-taintnp-28993262-vmss000000
  Normal  Pulling    4m47s  kubelet             pulling image "mcr.microsoft.com/oss/nginx/nginx:1.15.9-alpine"
  Normal  Pulled     4m43s  kubelet             Successfully pulled image "mcr.microsoft.com/oss/nginx/nginx:1.15.9-alpine"
  Normal  Created    4m40s  kubelet             Created container
  Normal  Started    4m40s  kubelet             Started container
```

Seuls les pods auxquels cette tolérance est appliquée peuvent être planifiés sur les nœuds dans *taintnp*. N’importe quel autre pod sera planifié dans le pool de nœuds *nodepool1*. Si vous créez des pools de nœuds supplémentaires, vous pouvez utiliser des teintes et des tolérances supplémentaires pour limiter les pods qui peuvent être planifiés sur ces ressources de nœud.

### <a name="setting-nodepool-labels"></a>Définition d’étiquettes de pool de nœuds

Vous pouvez également ajouter des intitulés à un pool de nœuds pendant la création de celui-ci. Les intitulés définis au niveau du pool de nœuds sont ajoutés à chaque nœud du pool. Ces [intitulés sont visibles dans Kubernetes][kubernetes-labels] pour la gestion des règles de planification des nœuds.

Pour créer un pool de nœuds à l’aide d’un intitulé, utilisez la commande [az aks nodepool add][az-aks-nodepool-add]. Spécifiez le nom *labelnp* et utilisez le paramètre `--labels` afin de spécifier *dept=IT* et *costcenter=9999* comme intitulés.

```azurecli-interactive
az aks nodepool add \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name labelnp \
    --node-count 1 \
    --labels dept=IT costcenter=9999 \
    --no-wait
```

> [!NOTE]
> L’intitulé d’un pool de nœuds ne peut être défini que pendant la création de celui-ci. Les intitulés doivent également constituer une paire clé/valeur et avoir une [syntaxe valide][kubernetes-label-syntax].

L’exemple de sortie suivant de la commande [az aks node pool list][az-aks-nodepool-list] montre que *labelnp* crée (*Creating*) des nœuds avec les intitulés (*nodeLabels*) spécifiés :

```console
$ az aks nodepool list -g myResourceGroup --cluster-name myAKSCluster

[
  {
    ...
    "count": 1,
    ...
    "name": "labelnp",
    "orchestratorVersion": "1.15.7",
    ...
    "provisioningState": "Creating",
    ...
    "nodeLabels":  {
      "dept": "IT",
      "costcenter": "9999"
    },
    ...
  },
 ...
]
```

### <a name="setting-nodepool-azure-tags"></a>Définition de balises Azure de pool de nœuds

Vous pouvez appliquer une balise Azure à des pools de nœuds de votre cluster AKS. Les balises appliquées à un pool de nœuds sont appliquées à chaque nœud du pool et sont conservées lors des mises à niveau. Des étiquettes sont également appliquées aux nouveaux nœuds qui sont ajoutés à un pool lors des opérations de scale-out. L'ajout d'une balise peut être utile pour les tâches telles que le suivi des stratégies ou l'estimation des coûts.

Les balises Azure ont des clés qui ne sont respectent pas la casse pour des opérations telles que la récupération d’une balise en recherchant la clé. Dans ce cas, une balise avec la clé donnée est mise à jour ou récupérée, quelle que soit la casse. Les valeurs des étiquettes respectent la casse.

Dans AKS, si plusieurs balises sont définies avec des clés identiques, mais une casse différente, la balise utilisée est la première dans l’ordre alphabétique. Par exemple, `{"Key1": "val1", "kEy1": "val2", "key1": "val3"}` entraîne la définition de `Key1` et de `val1`.

Créez un pool de nœuds à l’aide de la commande [az aks nodepool add][az-aks-nodepool-add]. Spécifiez le nom *tagnodepool*, et utilisez le paramètre `--tag` afin de spécifier *dept = IT* et *costcenter = 9999* pour les balises.

```azurecli-interactive
az aks nodepool add \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name tagnodepool \
    --node-count 1 \
    --tags dept=IT costcenter=9999 \
    --no-wait
```

> [!NOTE]
> Vous pouvez également utiliser le paramètre `--tags` lors de l'utilisation de la commande [az aks nodepool update][az-aks-nodepool-update] ainsi que lors de la création du cluster. Lors de la création du cluster, le paramètre `--tags` applique la balise au pool de nœuds initial créé avec le cluster. Tous les noms de balise doivent respecter les limites présentées dans l'article [Utiliser des balises pour organiser vos ressources Azure][tag-limitation]. La mise à jour d'un pool de nœuds à l'aide du paramètre `--tags` entraîne la mise à jour de toutes les valeurs de balise existantes et l'ajout de toutes les nouvelles balises. Par exemple, si vous remplacez les valeurs de balise *dept=IT* et *costcenter=9999* d'un pool de nœuds par *team=dev* et *costcenter=111*, les valeurs de balise du pool de nœuds seront les suivantes : *dept=IT*, *costcenter=111* et *team=dev*.

L'exemple de sortie suivant de la commande [az aks node pool list][az-aks-nodepool-list] montre que *tagnodepool* crée (*Creating*) des nœuds avec la balise (*tag*) spécifiée :

```azurecli
az aks nodepool list -g myResourceGroup --cluster-name myAKSCluster
```

```output
[
  {
    ...
    "count": 1,
    ...
    "name": "tagnodepool",
    "orchestratorVersion": "1.15.7",
    ...
    "provisioningState": "Creating",
    ...
    "tags": {
      "dept": "IT",
      "costcenter": "9999"
    },
    ...
  },
 ...
]
```

## <a name="manage-node-pools-using-a-resource-manager-template"></a>Gérer les pools de nœuds à l’aide d’un modèle Resource Manager

Lorsque vous utilisez un modèle Azure Resource Manager pour créer et gérer des ressources, vous pouvez généralement mettre à jour les paramètres dans votre modèle et le redéployer pour mettre à jour la ressource. Avec les pools de nœuds dans AKS, le profil de pool de nœuds initial ne peut pas être mis à jour une fois que le cluster AKS a été créé. Ce comportement signifie que vous ne pouvez pas mettre à jour un modèle Resource Manager existant, apporter un changement aux pools de nœuds et redéployer. Au lieu de cela, vous devez créer un modèle Resource Manager distinct qui mette à jour uniquement les pools de nœuds pour un cluster AKS existant.

Créez un modèle tel que `aks-agentpools.json` et collez l’exemple de manifeste suivant. Cet exemple de modèle configure les paramètres suivants :

* Met à jour le pool de nœuds *Linux* nommé *myagentpool* pour exécuter trois nœuds.
* Définit les nœuds dans le pool de nœuds pour exécuter Kubernetes version *1.15.7*.
* Définit la taille de nœud en tant que *Standard_DS2_v2*.

Modifiez ces valeurs en fonction des besoins pour mettre à jour, ajouter ou supprimer des pools de nœuds :

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "clusterName": {
            "type": "string",
            "metadata": {
                "description": "The name of your existing AKS cluster."
            }
        },
        "location": {
            "type": "string",
            "metadata": {
                "description": "The location of your existing AKS cluster."
            }
        },
        "agentPoolName": {
            "type": "string",
            "defaultValue": "myagentpool",
            "metadata": {
                "description": "The name of the agent pool to create or update."
            }
        },
        "vnetSubnetId": {
            "type": "string",
            "defaultValue": "",
            "metadata": {
                "description": "The Vnet subnet resource ID for your existing AKS cluster."
            }
        }
    },
    "variables": {
        "apiVersion": {
            "aks": "2020-01-01"
        },
        "agentPoolProfiles": {
            "maxPods": 30,
            "osDiskSizeGB": 0,
            "agentCount": 3,
            "agentVmSize": "Standard_DS2_v2",
            "osType": "Linux",
            "vnetSubnetId": "[parameters('vnetSubnetId')]"
        }
    },
    "resources": [
        {
            "apiVersion": "2020-01-01",
            "type": "Microsoft.ContainerService/managedClusters/agentPools",
            "name": "[concat(parameters('clusterName'),'/', parameters('agentPoolName'))]",
            "location": "[parameters('location')]",
            "properties": {
                "maxPods": "[variables('agentPoolProfiles').maxPods]",
                "osDiskSizeGB": "[variables('agentPoolProfiles').osDiskSizeGB]",
                "count": "[variables('agentPoolProfiles').agentCount]",
                "vmSize": "[variables('agentPoolProfiles').agentVmSize]",
                "osType": "[variables('agentPoolProfiles').osType]",
                "storageProfile": "ManagedDisks",
                "type": "VirtualMachineScaleSets",
                "vnetSubnetID": "[variables('agentPoolProfiles').vnetSubnetId]",
                "orchestratorVersion": "1.15.7"
            }
        }
    ]
}
```

Déployez ce modèle à l’aide de la commande [az group deployment create][az-group-deployment-create], comme indiqué dans l’exemple suivant. Vous êtes invité à fournir le nom et l’emplacement du cluster AKS existant :

```azurecli-interactive
az group deployment create \
    --resource-group myResourceGroup \
    --template-file aks-agentpools.json
```

> [!TIP]
> Vous pouvez ajouter une balise à votre pool de nœuds en ajoutant la propriété *tag* au modèle, comme dans l’exemple suivant.
> 
> ```json
> ...
> "resources": [
> {
>   ...
>   "properties": {
>     ...
>     "tags": {
>       "name1": "val1"
>     },
>     ...
>   }
> }
> ...
> ```

La mise à jour de votre cluster AKS peut prendre quelques minutes selon les paramètres de pool de nœuds et les opérations que vous définissez dans votre modèle Resource Manager.

## <a name="assign-a-public-ip-per-node-for-your-node-pools-preview"></a>Attribuer une adresse IP publique par nœud dans un pool de nœuds (préversion)

> [!WARNING]
> Vous devez installer l’extension CLI 0.4.43 en préversion ou une version ultérieure pour utiliser la fonctionnalité d’adresse IP publique par nœud.

Les nœuds AKS n’ont pas besoin de leurs propres adresses IP publiques pour communiquer. Toutefois, certains scénarios peuvent exiger que les nœuds d'un pool de nœuds reçoivent leurs propres adresses IP publiques dédiées. C’est par exemple le cas pour les charges de travail de gaming, où une console doit être directement connectée à une machine virtuelle du cloud afin de réduire les tronçons. Vous devez alors vous inscrire sur AKS pour accéder à la fonctionnalité d'évaluation « Node Public IP » (disponible en préversion).

Pour installer ou mettre à jour l’extension aks-preview vers la version la plus récente, utilisez les commandes Azure CLI suivantes :

```azurecli
az extension add --name aks-preview
az extension update --name aks-preview
az extension list
```

Pour vous inscrire à la fonctionnalité « Node Public IP », utilisez la commande Azure CLI suivante :

```azurecli-interactive
az feature register --name NodePublicIPPreview --namespace Microsoft.ContainerService
```
L’inscription peut prendre plusieurs minutes.  Vous pouvez vérifier l’état de l’inscription avec la commande suivante :

```azurecli-interactive
 az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/NodePublicIPPreview')].{Name:name,State:properties.state}"
```

Une fois l’inscription terminée, créez un groupe de ressources.

```azurecli-interactive
az group create --name myResourceGroup2 --location eastus
```

Créez un cluster AKS et attachez une adresse IP publique pour vos nœuds. Chacun des nœuds du pool reçoit une adresse IP publique unique. Pour le vérifier, il suffit de regarder les instances du groupe de machines virtuelles identiques.

```azurecli-interactive
az aks create -g MyResourceGroup2 -n MyManagedCluster -l eastus  --enable-node-public-ip
```

Pour les clusters AKS existants, vous pouvez également ajouter un nouveau pool de nœuds et attacher une adresse IP publique pour vos nœuds.

```azurecli-interactive
az aks nodepool add -g MyResourceGroup2 --cluster-name MyManagedCluster -n nodepool2 --enable-node-public-ip
```

> [!Important]
> Dans la préversion, Azure Instance Metadata Service ne prend pas en charge la récupération des adresses IP publiques pour la référence SKU de machine virtuelle de niveau standard. En raison de cette limitation, vous ne pouvez pas utiliser les commandes kubectl pour afficher les adresses IP publiques qui sont attribuées aux nœuds. Toutefois, les adresses IP sont affectées et fonctionnent comme prévu. Les adresses IP publiques de vos nœuds sont attachées aux instances de votre groupe de machines virtuelles identiques.

Vous pouvez localiser les adresses IP publiques de vos nœuds de différentes manières :

* Utilisez la commande Azure CLI [az vmss list-instance-public-ips][az-list-ips].
* Utilisez les [commandes PowerShell ou Bash][vmss-commands]. 
* Vous pouvez également voir les adresses IP publiques dans le portail Azure en affichant les instances du groupe de machines virtuelles identiques.

> [!Important]
> Le [groupe de ressources du nœud][node-resource-group] contient les nœuds et leurs adresses IP publiques. Utilisez le groupe de ressources du nœud lorsque vous exécutez des commandes pour rechercher les adresses IP publiques de vos nœuds.

```azurecli
az vmss list-instance-public-ips -g MC_MyResourceGroup2_MyManagedCluster_eastus -n YourVirtualMachineScaleSetName
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Dans cet article, vous avez créé un cluster AKS qui inclut des nœuds basés sur GPU. Pour réduire les coûts inutiles, vous voudrez peut-être supprimer le pool *gpunodepool* ou le cluster AKS tout entier.

Pour supprimer le pool de nœuds basé sur GPU, utilisez la commande [az aks nodepool delete][az-aks-nodepool-delete] comme indiqué dans l’exemple suivant :

```azurecli-interactive
az aks nodepool delete -g myResourceGroup --cluster-name myAKSCluster --name gpunodepool
```

Pour supprimer le cluster lui-même, utilisez la commande [az group delete][az-group-delete] pour supprimer le groupe de ressources AKS :

```azurecli-interactive
az group delete --name myResourceGroup --yes --no-wait
```

Vous pouvez également supprimer le cluster supplémentaire que vous avez créé dans le cadre du scénario d’adresse IP publique pour les pools de nœuds.

```azurecli-interactive
az group delete --name myResourceGroup2 --yes --no-wait
```

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage sur les [pools de nœuds système][use-system-pool].

Dans cet article, vous avez appris comment créer et gérer plusieurs pools de nœuds dans un cluster AKS. Pour plus d’informations sur la façon de contrôler les pods entre les pools de nœuds, consultez [Bonnes pratiques relatives aux fonctionnalités avancées du planificateur dans AKS][operator-best-practices-advanced-scheduler].

Pour créer et utiliser des pools de nœuds de conteneur Windows Server, consultez [Créer un conteneur Windows Server dans AKS][aks-windows].

Utilisez les [groupes de placement de proximité][reduce-latency-ppg] pour réduire la latence dans vos applications AKS.

<!-- EXTERNAL LINKS -->
[kubernetes-drain]: https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-taint]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#taint
[kubectl-describe]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe
[kubernetes-labels]: https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/
[kubernetes-label-syntax]: https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#syntax-and-character-set

<!-- INTERNAL LINKS -->
[aks-windows]: windows-container-cli.md
[az-aks-get-credentials]: /cli/azure/aks#az-aks-get-credentials
[az-aks-create]: /cli/azure/aks#az-aks-create
[az-aks-get-upgrades]: /cli/azure/aks?view=azure-cli-latest#az-aks-get-upgrades
[az-aks-nodepool-add]: /cli/azure/aks/nodepool?view=azure-cli-latest#az-aks-nodepool-add
[az-aks-nodepool-list]: /cli/azure/aks/nodepool?view=azure-cli-latest#az-aks-nodepool-list
[az-aks-nodepool-update]: /cli/azure/aks/nodepool?view=azure-cli-latest#az-aks-nodepool-update
[az-aks-nodepool-upgrade]: /cli/azure/aks/nodepool?view=azure-cli-latest#az-aks-nodepool-upgrade
[az-aks-nodepool-scale]: /cli/azure/aks/nodepool?view=azure-cli-latest#az-aks-nodepool-scale
[az-aks-nodepool-delete]: /cli/azure/aks/nodepool?view=azure-cli-latest#az-aks-nodepool-delete
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[az-group-create]: /cli/azure/group#az-group-create
[az-group-delete]: /cli/azure/group#az-group-delete
[az-group-deployment-create]: /cli/azure/group/deployment#az-group-deployment-create
[gpu-cluster]: gpu-cluster.md
[install-azure-cli]: /cli/azure/install-azure-cli
[operator-best-practices-advanced-scheduler]: operator-best-practices-advanced-scheduler.md
[quotas-skus-regions]: quotas-skus-regions.md
[supported-versions]: supported-kubernetes-versions.md
[tag-limitation]: ../azure-resource-manager/management/tag-resources.md
[taints-tolerations]: operator-best-practices-advanced-scheduler.md#provide-dedicated-nodes-using-taints-and-tolerations
[vm-sizes]: ../virtual-machines/sizes.md
[use-system-pool]: use-system-pools.md
[ip-limitations]: ../virtual-network/virtual-network-ip-addresses-overview-arm#standard
[node-resource-group]: faq.md#why-are-two-resource-groups-created-with-aks
[vmss-commands]: ../virtual-machine-scale-sets/virtual-machine-scale-sets-networking.md#public-ipv4-per-virtual-machine
[az-list-ips]: /cli/azure/vmss?view=azure-cli-latest.md#az-vmss-list-instance-public-ips
[reduce-latency-ppg]: reduce-latency-ppg.md