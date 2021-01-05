---
title: Intégrer Azure NetApp Files à Azure Kubernetes Service
description: Découvrez comment intégrer Azure NetApp Files à Azure Kubernetes Service
services: container-service
ms.topic: article
ms.date: 10/23/2020
ms.openlocfilehash: bc65c3dfad4c27c1650054c6836fbbbf07a7dbf2
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93126251"
---
# <a name="integrate-azure-netapp-files-with-azure-kubernetes-service"></a>Intégrer Azure NetApp Files à Azure Kubernetes Service

[Azure NetApp Files][anf] est un service de stockage de fichiers limité, hautes performances et de classe entreprise s’exécutant sur Azure. Cet article vous montre comment intégrer Azure NetApp Files à Azure Kubernetes Service (AKS).

## <a name="before-you-begin"></a>Avant de commencer
Cet article suppose que vous avez un cluster AKS existant. Si vous avez besoin d’un cluster AKS, consultez le guide de démarrage rapide d’AKS [avec Azure CLI][aks-quickstart-cli]ou avec le [Portail Azure][aks-quickstart-portal].

> [!IMPORTANT]
> Votre cluster AKS doit également être situé [dans une région qui prend en charge Azure NetApp Files][anf-regions].

Azure CLI 2.0.59 (ou une version ultérieure) doit également être installé et configuré. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI][install-azure-cli].

### <a name="limitations"></a>Limites

Les limitations suivantes s’appliquent lorsque vous utilisez Azure NetApp Files :

* Azure NetApp Files est uniquement disponible [dans les régions Azure sélectionnées][anf-regions].
* Avant de pouvoir utiliser Azure NetApp Files, vous devez avoir accès au service Azure NetApp Files. Pour demander l’accès, vous pouvez utiliser le [formulaire de soumission de liste d’attente Azure NetApp Files][anf-waitlist]. Vous ne pouvez pas accéder au service Azure NetApp Files tant que vous n’avez pas reçu l’e-mail de confirmation officiel de l’équipe Azure NetApp Files.
* Après le déploiement initial d’un cluster AKS, seul le provisionnement statique pour Azure NetApp Files est pris en charge.
* Pour utiliser le provisionnement dynamique avec Azure NetApp Files, installez et configurez [NetApp Trident](https://netapp-trident.readthedocs.io/) version 19.07 ou ultérieure.

## <a name="configure-azure-netapp-files"></a>Configurer Azure NetApp Files

> [!IMPORTANT]
> Avant de pouvoir inscrire le fournisseur de ressources *Microsoft.NetApp* , vous devez renseigner le [formulaire de soumission de liste d’attente Azure NetApp Files][anf-waitlist] pour votre abonnement. Vous ne pouvez pas inscrire le fournisseur de ressources tant que vous n’avez pas reçu l’e-mail de confirmation officiel de l’équipe Azure NetApp Files.

Inscrivez le fournisseur de ressources *Microsoft.NetApp*  :

```azurecli
az provider register --namespace Microsoft.NetApp --wait
```

> [!NOTE]
> Cela peut prendre du temps.

Quand vous créez un compte Azure NetApp pour une utilisation avec AKS, vous devez créer le compte dans le groupe de ressources de **nœud**. Tout d’abord, obtenez le nom du groupe de ressources avec la commande [az aks show][az-aks-show] et ajoutez le paramètre de requête `--query nodeResourceGroup`. L’exemple suivant obtient le groupe de ressources de nœud pour le cluster AKS nommé *myAKSCluster* dans le groupe de ressources nommé *myResourceGroup*  :

```azurecli-interactive
az aks show --resource-group myResourceGroup --name myAKSCluster --query nodeResourceGroup -o tsv
```

```output
MC_myResourceGroup_myAKSCluster_eastus
```

Créez un compte Azure NetApp Files dans le groupe de ressources de **nœud** et dans la même région que votre cluster AKS en utilisant [az netappfiles account create][az-netappfiles-account-create]. L’exemple suivant crée un compte nommé *myaccount1* dans le groupe de ressources *MC_myResourceGroup_myAKSCluster_eastus* et la région *eastus*  :

```azurecli
az netappfiles account create \
    --resource-group MC_myResourceGroup_myAKSCluster_eastus \
    --location eastus \
    --account-name myaccount1
```

Créez un pool de capacités à l’aide de la commande [az netappfiles pool create][az-netappfiles-pool-create]. L’exemple suivant crée un pool de capacités nommé *mypool1* avec une taille de 4 To et un niveau de service *Premium*  :

```azurecli
az netappfiles pool create \
    --resource-group MC_myResourceGroup_myAKSCluster_eastus \
    --location eastus \
    --account-name myaccount1 \
    --pool-name mypool1 \
    --size 4 \
    --service-level Premium
```

Créez un sous-réseau à [déléguer à Azure NetApp Files][anf-delegate-subnet] en utilisant [az network vnet subnet create][az-network-vnet-subnet-create]. *Ce sous-réseau doit se trouver dans le même réseau virtuel que votre cluster AKS.*

```azurecli
RESOURCE_GROUP=MC_myResourceGroup_myAKSCluster_eastus
VNET_NAME=$(az network vnet list --resource-group $RESOURCE_GROUP --query [].name -o tsv)
VNET_ID=$(az network vnet show --resource-group $RESOURCE_GROUP --name $VNET_NAME --query "id" -o tsv)
SUBNET_NAME=MyNetAppSubnet
az network vnet subnet create \
    --resource-group $RESOURCE_GROUP \
    --vnet-name $VNET_NAME \
    --name $SUBNET_NAME \
    --delegations "Microsoft.NetApp/volumes" \
    --address-prefixes 10.0.0.0/28
```

Créez un volume en utilisant [az netappfiles volume create][az-netappfiles-volume-create].

```azurecli
RESOURCE_GROUP=MC_myResourceGroup_myAKSCluster_eastus
LOCATION=eastus
ANF_ACCOUNT_NAME=myaccount1
POOL_NAME=mypool1
SERVICE_LEVEL=Premium
VNET_NAME=$(az network vnet list --resource-group $RESOURCE_GROUP --query [].name -o tsv)
VNET_ID=$(az network vnet show --resource-group $RESOURCE_GROUP --name $VNET_NAME --query "id" -o tsv)
SUBNET_NAME=MyNetAppSubnet
SUBNET_ID=$(az network vnet subnet show --resource-group $RESOURCE_GROUP --vnet-name $VNET_NAME --name $SUBNET_NAME --query "id" -o tsv)
VOLUME_SIZE_GiB=100 # 100 GiB
UNIQUE_FILE_PATH="myfilepath2" # Please note that file path needs to be unique within all ANF Accounts

az netappfiles volume create \
    --resource-group $RESOURCE_GROUP \
    --location $LOCATION \
    --account-name $ANF_ACCOUNT_NAME \
    --pool-name $POOL_NAME \
    --name "myvol1" \
    --service-level $SERVICE_LEVEL \
    --vnet $VNET_ID \
    --subnet $SUBNET_ID \
    --usage-threshold $VOLUME_SIZE_GiB \
    --file-path $UNIQUE_FILE_PATH \
    --protocol-types "NFSv3"
```

## <a name="create-the-persistentvolume"></a>Créer la ressource PersistentVolume

Listez les détails de votre volume en utilisant [az netappfiles volume show][az-netappfiles-volume-show]

```azurecli
az netappfiles volume show --resource-group $RESOURCE_GROUP --account-name $ANF_ACCOUNT_NAME --pool-name $POOL_NAME --volume-name "myvol1"
```

```output
{
  ...
  "creationToken": "myfilepath2",
  ...
  "mountTargets": [
    {
      ...
      "ipAddress": "10.0.0.4",
      ...
    }
  ],
  ...
}
```

Créez un fichier `pv-nfs.yaml` définissant une ressource PersistentVolume. Remplacez `path` par *creationToken* et `server` par *ipAddress* à partir de la commande précédente. Par exemple :

```yaml
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nfs
spec:
  capacity:
    storage: 100Gi
  accessModes:
    - ReadWriteMany
  nfs:
    server: 10.0.0.4
    path: /myfilepath2
```

Mettez à jour *server* et *path* avec les valeurs de votre volume NFS (Network File System) que vous avez créé à l’étape précédente. Créez la ressource PersistentVolume avec la commande [kubectl apply][kubectl-apply] :

```console
kubectl apply -f pv-nfs.yaml
```

Vérifiez que l’ *état* de la ressource PersistentVolume est *Disponible* à l’aide de la commande [kubectl describe][kubectl-describe] :

```console
kubectl describe pv pv-nfs
```

## <a name="create-the-persistentvolumeclaim"></a>Créer la ressource PersistentVolumeClaim

Créez un fichier `pvc-nfs.yaml` définissant une ressource PersistentVolume. Par exemple :

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-nfs
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: ""
  resources:
    requests:
      storage: 1Gi
```

Créez la ressource PersistentVolumeClaim avec la commande [kubectl apply][kubectl-apply] :

```console
kubectl apply -f pvc-nfs.yaml
```

Vérifiez que l’ *état* de la ressource PersistentVolumeClaim est *Bound* (Lié) à l’aide de la commande [kubectl describe][kubectl-describe] :

```console
kubectl describe pvc pvc-nfs
```

## <a name="mount-with-a-pod"></a>Monter avec un pod

Créez un fichier `nginx-nfs.yaml` définissant un pod qui utilise la ressource PersistentVolumeClaim. Par exemple :

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: nginx-nfs
spec:
  containers:
  - image: mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine
    name: nginx-nfs
    command:
    - "/bin/sh"
    - "-c"
    - while true; do echo $(date) >> /mnt/azure/outfile; sleep 1; done
    volumeMounts:
    - name: disk01
      mountPath: /mnt/azure
  volumes:
  - name: disk01
    persistentVolumeClaim:
      claimName: pvc-nfs
```

Créez le pod avec la commande [kubectl apply][kubectl-apply] :

```console
kubectl apply -f nginx-nfs.yaml
```

Vérifiez que le pod est *En cours d’exécution* à l’aide de la commande [kubectl describe][kubectl-describe] :

```console
kubectl describe pod nginx-nfs
```

Vérifiez que votre volume a été monté dans le pod en utilisant [kubectl exec][kubectl-exec] pour vous connecter au pod, puis `df -h` pour vérifier si le volume est monté.

```console
$ kubectl exec -it nginx-nfs -- sh
```

```output
/ # df -h
Filesystem             Size  Used Avail Use% Mounted on
...
10.0.0.4:/myfilepath2  100T  384K  100T   1% /mnt/azure
...
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur Azure NetApp Files, consultez [Présentation d’Azure NetApp Files][anf]. Pour plus d’informations sur l’utilisation de NFS avec AKS, consultez [Créer manuellement et utiliser un volume de serveur Linux NFS (Network File System) avec Azure Kubernetes Service (AKS)][aks-nfs].


[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[aks-nfs]: azure-nfs-volume.md
[anf]: ../azure-netapp-files/azure-netapp-files-introduction.md
[anf-delegate-subnet]: ../azure-netapp-files/azure-netapp-files-delegate-subnet.md
[anf-quickstart]: ../azure-netapp-files/
[anf-regions]: https://azure.microsoft.com/global-infrastructure/services/?products=netapp&regions=all
[anf-waitlist]: https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbR8cq17Xv9yVBtRCSlcD_gdVUNUpUWEpLNERIM1NOVzA5MzczQ0dQR1ZTSS4u
[az-aks-show]: /cli/azure/aks#az-aks-show
[az-netappfiles-account-create]: /cli/azure/netappfiles/account?view=azure-cli-latest#az-netappfiles-account-create
[az-netappfiles-pool-create]: /cli/azure/netappfiles/pool?view=azure-cli-latest#az-netappfiles-pool-create
[az-netappfiles-volume-create]: /cli/azure/netappfiles/volume?view=azure-cli-latest#az-netappfiles-volume-create
[az-netappfiles-volume-show]: /cli/azure/netappfiles/volume?view=azure-cli-latest#az-netappfiles-volume-show
[az-network-vnet-subnet-create]: /cli/azure/network/vnet/subnet?view=azure-cli-latest#az-network-vnet-subnet-create
[install-azure-cli]: /cli/azure/install-azure-cli
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-describe]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe
[kubectl-exec]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#exec
