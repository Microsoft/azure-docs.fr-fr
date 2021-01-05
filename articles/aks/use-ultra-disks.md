---
title: Activer la prise en charge de disques Ultra dans Azure Kubernetes Service (AKS)
description: Découvrez les procédure d’activation et de configuration de disques Ultra dans un cluster Azure Kubernetes Service (AKS)
services: container-service
ms.topic: article
ms.date: 07/10/2020
ms.openlocfilehash: 049c2682a8f61bb658083b0418a4fcf99dc477a5
ms.sourcegitcommit: 693df7d78dfd5393a28bf1508e3e7487e2132293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92900044"
---
# <a name="use-azure-ultra-disks-on-azure-kubernetes-service-preview"></a>Utiliser des disques Ultra Azure dans Azure Kubernetes Service (version préliminaire)

Les [disques Ultra Azure](../virtual-machines/disks-enable-ultra-ssd.md) permettent un stockage de disque à haut débit avec un nombre élevé d’IOPS et une faible latence homogène pour vos applications avec état. Entre autres avantages, les disques Ultra permettent de changer dynamiquement les performances des disques SSD en fonction de vos charges de travail, sans devoir redémarrer vos nœuds d’agent. Les disques Ultra sont adaptés aux charges de travail à forte intensité de données.

## <a name="before-you-begin"></a>Avant de commencer

Cette fonctionnalité ne peut être définie qu’au moment de la création du cluster ou de la création du pool de nœuds.

> [!IMPORTANT]
> Les disques Ultra Azure requièrent des pools de nœuds déployés dans des zones de disponibilité et des régions qui prennent en charge ces disques, ainsi que des séries de machines virtuelles spécifiques uniquement. Consultez [**Étendue et limitations de la version en disponibilité générale des disques Ultra**](../virtual-machines/disks-enable-ultra-ssd.md#ga-scope-and-limitations).

### <a name="register-the-enableultrassd-preview-feature"></a>Inscrire la fonctionnalité d’évaluation `EnableUltraSSD`

Pour créer un cluster AKS qui utilise un pool de nœuds pouvant exploiter les disques Ultra, vous devez activer l’indicateur de fonctionnalité `EnableUltraSSD` sur votre abonnement.

Inscrivez l’indicateur de fonctionnalité `EnableUltraSSD` à l’aide de la commande [az feature register][az-feature-register], comme indiqué dans l’exemple suivant :

```azurecli-interactive
az feature register --namespace "Microsoft.ContainerService" --name "EnableUltraSSD"
```

Quelques minutes sont nécessaires pour que l’état s’affiche *Registered* (Inscrit). Vous pouvez vérifier l’état de l’enregistrement à l’aide de la commande [az feature list][az-feature-list] :

```azurecli-interactive
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/EnableUltraSSD')].{Name:name,State:properties.state}"
```

Lorsque vous êtes prêt, actualisez l’inscription du fournisseur de ressources *Microsoft.ContainerService* à l’aide de la commande [az provider register][az-provider-register] :

```azurecli-interactive
az provider register --namespace Microsoft.ContainerService
```

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

### <a name="install-aks-preview-cli-extension"></a>Installer l’extension CLI de préversion d’aks

Pour créer un cluster AKS ou un pool de nœuds pouvant utiliser les disques Ultra, vous avez besoin de la dernière extension CLI *aks-preview* . Installez l’extension Azure CLI *aks-preview* à l’aide de la commande [az extension add][az-extension-add] ou installez toutes les mises à jour disponibles à l’aide de la commande [az extension update][az-extension-update] :

```azurecli-interactive
# Install the aks-preview extension
az extension add --name aks-preview

# Update the extension to make sure you have the latest version installed
az extension update --name aks-preview
``` 

### <a name="limitations"></a>Limites
- Consultez [**Étendue et limitations de la version en disponibilité générale des disques Ultra**](../virtual-machines/disks-enable-ultra-ssd.md#ga-scope-and-limitations)
- La plage de tailles prise en charge pour les disques Ultra est comprise entre 100 et 1 500

## <a name="create-a-new-cluster-that-can-use-ultra-disks"></a>Créer un cluster qui peut utiliser des disques Ultra

Créez un cluster AKS capable d’exploiter des disques Ultra à l’aide des commandes CLI suivantes. Utilisez l’indicateur `--aks-custom-headers` pour définir la fonctionnalité `EnableUltraSSD`.

Créez un groupe de ressources Azure :

```azurecli-interactive
# Create an Azure resource group
az group create --name myResourceGroup --location westus2
```

Créez le cluster AKS avec prise en charge des Disques Ultra.

```azurecli-interactive
# Create an AKS-managed Azure AD cluster
az aks create -g MyResourceGroup -n MyManagedCluster -l westus2 --node-vm-size Standard_L8s_v2 --zones 1 2 --node-count 2 --aks-custom-headers EnableUltraSSD=true
```

Si vous souhaitez créer des clusters sans prise en charge des disques Ultra, vous pouvez le faire en omettant le paramètre `--aks-custom-headers` personnalisé.

## <a name="enable-ultra-disks-on-an-existing-cluster"></a>Activer les disques Ultra sur un cluster existant

Vous pouvez activer les disques Ultra sur les clusters existants en ajoutant un nouveau pool de nœuds à votre cluster qui prend en charge les disques Ultra. Configurez un nouveau pool de nœuds pour utiliser le chiffrement basé sur l’hôte à l’aide de l’indicateur `--aks-custom-headers`.

```azurecli
az aks nodepool add --name ultradisk --cluster-name myAKSCluster --resource-group myResourceGroup --node-vm-size Standard_L8s_v2 --zones 1 2 --node-count 2 --aks-custom-headers EnableUltraSSD=true
```

Si vous souhaitez créer des pools de nœuds sans la prise en charge des disques Ultra, vous pouvez le faire en omettant le paramètre `--aks-custom-headers` personnalisé.

## <a name="use-ultra-disks-dynamically-with-a-storage-class"></a>Utiliser des disques Ultra de manière dynamique avec une classe de stockage

Pour utiliser des disques Ultra dans nos déploiements ou groupes avec état, vous pouvez utiliser une [classe de stockage pour l’approvisionnement dynamique](azure-disks-dynamic-pv.md).

### <a name="create-the-storage-class"></a>Créer la classe de stockage

Une classe de stockage permet de définir la création dynamique d’une unité de stockage avec un volume persistant. Pour plus d’informations sur les classes de stockage Kubernetes, consultez [Classes de stockage Kubernetes][kubernetes-storage-classes].

Dans ce cas, nous allons créer une classe de stockage qui fait référence à des disques Ultra. Créez un fichier nommé `azure-ultra-disk-sc.yaml` et copiez-y le manifeste suivant.

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: ultra-disk-sc
provisioner: kubernetes.io/azure-disk
volumeBindingMode: WaitForFirstConsumer # optional, but recommended if you want to wait until the pod that will use this disk is created 
parameters:
  skuname: UltraSSD_LRS
  kind: managed
  cachingmode: None
  diskIopsReadWrite: "2000"  # minimum value: 2 IOPS/GiB 
  diskMbpsReadWrite: "320"   # minimum value: 0.032/GiB
```

Créez la classe de stockage avec la commande [kubectl apply][kubectl-apply] et spécifiez votre fichier *azure-ultra-disk-sc.yaml*  :

```console
$ kubectl apply -f azure-ultra-disk-sc.yaml


storageclass.storage.k8s.io/ultra-disk-sc created
```

## <a name="create-a-persistent-volume-claim"></a>Créer une revendication de volume persistant

Une revendication de volume persistant (PVC) est utilisée pour configurer automatiquement le stockage basé sur une classe de stockage. Dans ce cas, un PVC peut utiliser la classe de stockage créée précédemment pour créer un disque ultra.

Créez un fichier nommé `azure-ultra-disk-pvc.yaml` et copiez-y le manifeste suivant. La revendication demande un disque nommé `ultra-disk`, d’une taille de *1 000 Go* avec un accès *ReadWriteOnce* . La classe de stockage *ultra-disk-sc* est spécifiée en tant que classe de stockage.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ultra-disk
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: ultra-disk-sc
  resources:
    requests:
      storage: 1000Gi
```

Créez la revendication de volume persistant avec la commande [kubectl apply][kubectl-apply] et spécifiez votre fichier *azure-ultra-disk-pvc.yaml*  :

```console
$ kubectl apply -f azure-ultra-disk-pvc.yaml

persistentvolumeclaim/ultra-disk created
```

## <a name="use-the-persistent-volume"></a>Utiliser le volume persistant

Une fois la revendication de volume persistant créée, et le disque provisionné convenablement, un pod peut être créé avec un accès au disque. Le manifeste suivant crée un pod NGINX de base qui utilise la revendication de volume persistant nommé *ultra-disk* pour monter le disque Azure à l’emplacement `/mnt/azure`.

Créez un fichier nommé `nginx-ultra.yaml` et copiez-y le manifeste suivant.

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: nginx-ultra
spec:
  containers:
  - name: nginx-ultra
    image: mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 250m
        memory: 256Mi
    volumeMounts:
    - mountPath: "/mnt/azure"
      name: volume
  volumes:
    - name: volume
      persistentVolumeClaim:
        claimName: ultra-disk
```

Créez le pod avec la commande [kubectl apply][kubectl-apply], comme indiqué dans l’exemple suivant :

```console
$ kubectl apply -f nginx-ultra.yaml

pod/nginx-ultra created
```

Vous disposez maintenant d’un pod en cours d’exécution avec le disque Azure monté dans le répertoire `/mnt/azure`. Cette configuration peut être consultée en inspectant votre pod via `kubectl describe pod nginx-ultra`, comme illustré dans l’exemple condensé suivant :

```console
$ kubectl describe pod nginx-ultra

[...]
Volumes:
  volume:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  azure-managed-disk
    ReadOnly:   false
  default-token-smm2n:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-smm2n
    Optional:    false
[...]
Events:
  Type    Reason                 Age   From                               Message
  ----    ------                 ----  ----                               -------
  Normal  Scheduled              2m    default-scheduler                  Successfully assigned mypod to aks-nodepool1-79590246-0
  Normal  SuccessfulMountVolume  2m    kubelet, aks-nodepool1-79590246-0  MountVolume.SetUp succeeded for volume "default-token-smm2n"
  Normal  SuccessfulMountVolume  1m    kubelet, aks-nodepool1-79590246-0  MountVolume.SetUp succeeded for volume "pvc-faf0f176-8b8d-11e8-923b-deb28c58d242"
[...]
```


## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur les disques Ultra, consultez [Utilisation de disques Ultra Azure](../virtual-machines/disks-enable-ultra-ssd.md).
- Pour plus d’informations sur les meilleures pratiques relatives au stockage, consultez [Meilleures pratiques relatives au stockage et aux sauvegardes dans Azure Kubernetes Service (AKS)][operator-best-practices-storage]

<!-- LINKS - external -->
[access-modes]: https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubernetes-storage-classes]: https://kubernetes.io/docs/concepts/storage/storage-classes/
[kubernetes-volumes]: https://kubernetes.io/docs/concepts/storage/persistent-volumes/
[managed-disk-pricing-performance]: https://azure.microsoft.com/pricing/details/managed-disks/

<!-- LINKS - internal -->
[azure-disk-volume]: azure-disk-volume.md
[azure-files-pvc]: azure-files-dynamic-pv.md
[premium-storage]: ../virtual-machines/disks-types.md
[az-disk-list]: /cli/azure/disk#az-disk-list
[az-snapshot-create]: /cli/azure/snapshot#az-snapshot-create
[az-disk-create]: /cli/azure/disk#az-disk-create
[az-disk-show]: /cli/azure/disk#az-disk-show
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
[operator-best-practices-storage]: operator-best-practices-storage.md
[concepts-storage]: concepts-storage.md
[storage-class-concepts]: concepts-storage.md#storage-classes
[az-extension-add]: /cli/azure/extension?view=azure-cli-latest#az-extension-add
[az-extension-update]: /cli/azure/extension?view=azure-cli-latest#az-extension-update
[az-feature-register]: /cli/azure/feature?view=azure-cli-latest#az-feature-register
[az-feature-list]: /cli/azure/feature?view=azure-cli-latest#az-feature-list
[az-provider-register]: /cli/azure/provider?view=azure-cli-latest#az-provider-register
