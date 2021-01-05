---
title: Utiliser des pilotes CSI (Container Storage interface) pour Azure Files sur Azure Kubernetes Service (AKS)
description: Découvrez comment utiliser des pilotes CSI (Container Storage interface) pour Azure Files sur un cluster Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 08/27/2020
author: palma21
ms.openlocfilehash: b29f4034b12ce43e6c051e454601f196365469f3
ms.sourcegitcommit: 295db318df10f20ae4aa71b5b03f7fb6cba15fc3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/15/2020
ms.locfileid: "94636978"
---
# <a name="use-azure-files-container-storage-interface-csi-drivers-in-azure-kubernetes-service-aks-preview"></a>Utiliser des pilotes CSI (Container Storage interface) pour Azure Files dans Azure Kubernetes Service (AKS) (préversion)

Le pilote CSI (Container Storage interface) pour Azure Files est un pilote conforme à la [spécification CSI](https://github.com/container-storage-interface/spec/blob/master/spec.md) utilisé par Azure Kubernetes Service (AKS) pour gérer le cycle de vie des partages Azure Files.

CSI est une norme pour exposer des systèmes de stockage de blocs et de fichiers arbitraires à des charges de travail en conteneur sur Kubernetes. En adoptant et en utilisant CSI, AKS peut désormais écrire, déployer et itérer les plug-ins afin d’exposer de nouveaux systèmes de stockage ou améliorer ceux existants dans Kubernetes sans avoir à toucher au code Kubernetes principal et attendre ses cycles de publication.

Pour créer un cluster AKS avec prise en charge du pilote CSI, consultez [Activer les pilotes CSI pour les disques Azure et Azure Files sur AKS](csi-storage-drivers.md).

>[!NOTE]
> *Les pilotes dans l’arborescence* sont les pilotes de stockage actuels, qui font partie du code Kubernetes principal, et les nouveaux pilotes CSI, qui sont des plug-ins.

## <a name="use-a-persistent-volume-with-azure-files"></a>Utiliser un volume persistant avec Azure Files

Un [volume persistant](concepts-storage.md#persistent-volumes) représente un élément de stockage provisionné pour une utilisation dans des pods Kubernetes. Un volume persistant peut être utilisé par un ou plusieurs pods et être provisionné de façon statique ou dynamique. Si plusieurs pods doivent accéder simultanément au même volume de stockage, vous pouvez utiliser Azure Files pour vous connecter à l’aide du [protocole SMB (Server Message Block)][smb-overview]. Cet article vous montre comment créer un partage Azure Files de manière dynamique utilisé par plusieurs pods dans un cluster AKS. Pour l’approvisionnement statique, consultez [Créer et utiliser manuellement un volume avec un partage Azure Files](azure-files-volume.md).

Pour plus d’informations sur les volumes Kubernetes, consultez [Options de stockage pour les applications dans AKS][concepts-storage].

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

## <a name="dynamically-create-azure-files-pvs-by-using-the-built-in-storage-classes"></a>Créer dynamiquement des PV Azure Files à l’aide des classes de stockage intégrées

Une classe de stockage permet de définir la façon dont un partage Azure Files est créé. Un compte de stockage est automatiquement créé dans le [groupe de ressources de nœud][node-resource-group] pour être utilisé avec la classe de stockage afin de contenir les partages Azure Files. Faites votre choix parmi les [références SKU de redondance de stockage Azure][storage-skus] suivantes pour *skuName* :

* **Standard_LRS** : Stockage localement redondant standard
* **Standard_GRS** : Stockage géo-redondant standard
* **Standard_ZRS** : Stockage redondant dans une zone standard
* **Standard_RAGRS** : Stockage géo-redondant avec accès en lecture standard
* **Premium_LRS** : Stockage Premium localement redondant

> [!NOTE]
> Azure Files prend en charge le stockage Premium Azure. Le partage de fichiers Premium minimal est de 100 Go.

Lorsque vous utilisez des pilotes de stockage CSI sur AKS, il existe deux `StorageClasses` supplémentaires intégrées qui utilisent les pilotes de stockage CSI Azure Files. Les classes de stockage CSI supplémentaires sont créées avec le cluster en même temps que les classes de stockage par défaut dans l’arborescence.

- `azurefile-csi`: Utilise le stockage Azure Standard pour créer un partage Azure Files.
- `azurefile-csi-premium`: Utilise le stockage Azure Premium pour créer un partage Azure Files.

La stratégie de récupération sur les deux classes de stockage garantit que le partage Azure Files sous-jacent est supprimé lorsque le PV respectif est supprimé. Les classes de stockage configurent également les partages de fichiers pour qu’ils soient extensibles. Il vous suffit de modifier la revendication de volume persistant (PVC) avec la nouvelle taille.

Pour utiliser ces classes de stockage, créez un [PVC](concepts-storage.md#persistent-volume-claims) et un pod respectif qui les référence et les utilise. Une revendication de volume persistant est utilisée pour configurer automatiquement le stockage basé sur une classe de stockage. Un PVC peut utiliser l’une des classes de stockage précréées ou une classe de stockage définie par l’utilisateur pour créer un partage Azure Files pour la référence SKU et la taille de votre choix. Quand vous créez une définition de pod, la revendication de volume persistant est spécifiée pour demander le stockage souhaité.

Créez un [exemple de PVC et de pod qui imprime la date actuelle dans un `outfile`](https://github.com/kubernetes-sigs/azurefile-csi-driver/blob/master/deploy/example/statefulset.yaml) avec la commande [kubectl apply][kubectl-apply] :

```console
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/azurefile-csi-driver/master/deploy/example/pvc-azurefile-csi.yaml
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/azurefile-csi-driver/master/deploy/example/nginx-pod-azurefile.yaml

persistentvolumeclaim/pvc-azurefile created
pod/nginx-azurefile created
```

Une fois que le pod est en cours d’exécution, vous pouvez vérifier que le partage de fichiers est correctement monté en exécutant la commande suivante et en vérifiant que la sortie contient `outfile` :

```console
$ kubectl exec nginx-azurefile -- ls -l /mnt/azurefile

total 29
-rwxrwxrwx 1 root root 29348 Aug 31 21:59 outfile
```

## <a name="create-a-custom-storage-class"></a>Créer une classe de stockage personnalisée

Les classes de stockage par défaut sont adaptées aux scénarios les plus courants, mais pas à tous. Dans certains cas, vous souhaiterez peut-être personnaliser votre propre classe de stockage avec vos propres paramètres. Par exemple, utilisez le manifeste suivant pour configurer les `mountOptions` du partage de fichiers.

La valeur par défaut de *fileMode* et *dirMode* est *0777* pour les partages de fichiers montés Kubernetes. Vous pouvez spécifier les différentes options de montage sur l’objet de classe de stockage.

Créez un fichier nommé `azure-file-sc.yaml` et copiez-y l’exemple de manifeste suivant :

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: my-azurefile
provisioner: file.csi.azure.com
reclaimPolicy: Delete
volumeBindingMode: Immediate
allowVolumeExpansion: true
mountOptions:
  - dir_mode=0640
  - file_mode=0640
  - uid=0
  - gid=0
  - mfsymlinks
  - cache=strict # https://linux.die.net/man/8/mount.cifs
  - nosharesock
parameters:
  skuName: Standard_LRS
```

Créez la classe de stockage avec la commande [kubectl apply][kubectl-apply] :

```console
kubectl apply -f azure-file-sc.yaml

storageclass.storage.k8s.io/my-azurefile created
```

Le pilote CSI pour Azure Files prend en charge la création de [captures instantanées de volumes persistants](https://kubernetes-csi.github.io/docs/snapshot-restore-feature.html) et des partages de fichiers sous-jacents.

Créez une [classe d’instantané de volume](https://github.com/kubernetes-sigs/azurefile-csi-driver/blob/master/deploy/example/snapshot/volumesnapshotclass-azurefile.yaml) avec la commande [kubectl apply][kubectl-apply] :

```console
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/azurefile-csi-driver/master/deploy/example/snapshot/volumesnapshotclass-azurefile.yaml

volumesnapshotclass.snapshot.storage.k8s.io/csi-azurefile-vsc created
```

Créez un [instantané de volume](https://github.com/kubernetes-sigs/azurefile-csi-driver/blob/master/deploy/example/snapshot/volumesnapshot-azurefile.yaml) à partir du PVC [que nous avons créé de façon dynamique au début de ce tutoriel](#dynamically-create-azure-files-pvs-by-using-the-built-in-storage-classes), `pvc-azurefile`.


```bash
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/azurefile-csi-driver/master/deploy/example/snapshot/volumesnapshot-azurefile.yaml


volumesnapshot.snapshot.storage.k8s.io/azurefile-volume-snapshot created
```

Vérifiez que l’instantané a été créé correctement :

```bash
$ kubectl describe volumesnapshot azurefile-volume-snapshot

Name:         azurefile-volume-snapshot
Namespace:    default
Labels:       <none>
Annotations:  API Version:  snapshot.storage.k8s.io/v1beta1
Kind:         VolumeSnapshot
Metadata:
  Creation Timestamp:  2020-08-27T22:37:41Z
  Finalizers:
    snapshot.storage.kubernetes.io/volumesnapshot-as-source-protection
    snapshot.storage.kubernetes.io/volumesnapshot-bound-protection
  Generation:        1
  Resource Version:  955091
  Self Link:         /apis/snapshot.storage.k8s.io/v1beta1/namespaces/default/volumesnapshots/azurefile-volume-snapshot
  UID:               c359a38f-35c1-4fb1-9da9-2c06d35ca0f4
Spec:
  Source:
    Persistent Volume Claim Name:  pvc-azurefile
  Volume Snapshot Class Name:      csi-azurefile-vsc
Status:
  Bound Volume Snapshot Content Name:  snapcontent-c359a38f-35c1-4fb1-9da9-2c06d35ca0f4
  Ready To Use:                        false
Events:                                <none>
```

## <a name="resize-a-persistent-volume"></a>Redimensionner un volume persistant

Vous pouvez demander un volume plus important pour un PVC. Modifiez l’objet PVC et spécifiez une taille supérieure. Cette modification déclenche l’expansion du volume sous-jacent qui stocke le PV.

> [!NOTE]
> Un nouveau PV n’est jamais créé pour satisfaire la revendication. Au lieu de cela, un volume existant est redimensionné.

Dans AKS, la classe de stockage intégrée `azurefile-csi` prend déjà en charge l’expansion, vous pouvez donc utiliser le [PVC créé précédemment avec cette classe de stockage](#dynamically-create-azure-files-pvs-by-using-the-built-in-storage-classes). Le PVC a demandé un partage de fichiers de 100 Gi. Nous pouvons confirmer cela en exécutant :

```console 
$ kubectl exec -it nginx-azurefile -- df -h /mnt/azurefile

Filesystem                                                                                Size  Used Avail Use% Mounted on
//f149b5a219bd34caeb07de9.file.core.windows.net/pvc-5e5d9980-da38-492b-8581-17e3cad01770  100G  128K  100G   1% /mnt/azurefile
```

Développez le PVC en augmentant le champ `spec.resources.requests.storage` :

```console
$ kubectl patch pvc pvc-azurefile --type merge --patch '{"spec": {"resources": {"requests": {"storage": "200Gi"}}}}'

persistentvolumeclaim/pvc-azurefile patched
```

Vérifiez que le PVC et le système de fichiers à l’intérieur du pod affichent la nouvelle taille :

```console
$ kubectl get pvc pvc-azurefile
NAME            STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS    AGE
pvc-azurefile   Bound    pvc-5e5d9980-da38-492b-8581-17e3cad01770   200Gi      RWX            azurefile-csi   64m

$ kubectl exec -it nginx-azurefile -- df -h /mnt/azurefile
Filesystem                                                                                Size  Used Avail Use% Mounted on
//f149b5a219bd34caeb07de9.file.core.windows.net/pvc-5e5d9980-da38-492b-8581-17e3cad01770  200G  128K  200G   1% /mnt/azurefile
```


## <a name="nfs-file-shares"></a>Partages de fichiers NFS
[Azure Files prend désormais en charge le protocole NFS v4.1](../storage/files/storage-files-how-to-create-nfs-shares.md). La prise en charge du protocole NFS v4.1 pour Azure Files vous offre un système de fichiers NFS complètement managé en tant que service reposant sur une plateforme de stockage résiliente et distribuée hautement disponible et durable.

 Cette option est optimisée pour les charges de travail à accès aléatoire avec des mises à jour de données sur place et fournit une prise en charge complète du système de fichiers POSIX. Cette section vous montre comment utiliser des partages NFS avec le pilote CSI d’Azure Files sur un cluster AKS.

Veillez à vérifier les [limitations](../storage/files/storage-files-compare-protocols.md#limitations) et la [disponibilité des régions](../storage/files/storage-files-compare-protocols.md#regional-availability) pendant la phase de préversion.

### <a name="register-the-allownfsfileshares-preview-feature"></a>Inscrire la fonctionnalité d’évaluation `AllowNfsFileShares`

Pour créer un partage de fichiers qui utilise NFS 4.1, vous devez activer l’indicateur de fonctionnalité `AllowNfsFileShares` sur votre abonnement.

Inscrivez l’indicateur de fonctionnalité `AllowNfsFileShares` à l’aide de la commande [az feature register][az-feature-register], comme indiqué dans l’exemple suivant :

```azurecli-interactive
az feature register --namespace "Microsoft.Storage" --name "AllowNfsFileShares"
```

Quelques minutes sont nécessaires pour que l’état s’affiche *Registered* (Inscrit). Vérifiez l’état de l’inscription à l’aide de la commande [az feature list][az-feature-list] :

```azurecli-interactive
az feature list -o table --query "[?contains(name, 'Microsoft.Storage/AllowNfsFileShares')].{Name:name,State:properties.state}"
```

Lorsque vous êtes prêt, actualisez l’inscription du fournisseur de ressources *Microsoft.Storage* à l’aide de la commande [az provider register][az-provider-register] :

```azurecli-interactive
az provider register --namespace Microsoft.Storage
```

### <a name="create-a-storage-account-for-the-nfs-file-share"></a>Créer un compte de stockage pour le partage de fichiers NFS

[Créez un compte de stockage Azure `Premium_LRS`](../storage/files/storage-how-to-create-premium-fileshare.md) avec les configurations suivantes pour prendre en charge les partages NFS :
- type de compte : FileStorage
- transfert sécurisé obligatoire (activer le trafic HTTPS uniquement) : false
- sélectionnez le réseau virtuel de vos nœuds d’agent dans Pare-feu et réseaux virtuels - vous pouvez donc préférer créer le compte de stockage dans le groupe de ressources MC_.

### <a name="create-nfs-file-share-storage-class"></a>Créer une classe de stockage pour les partages de fichiers NFS

Enregistrez un fichier `nfs-sc.yaml` avec le manifeste ci-dessous en modifiant les espaces réservés respectifs.

```yml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: azurefile-csi-nfs
provisioner: file.csi.azure.com
parameters:
  resourceGroup: EXISTING_RESOURCE_GROUP_NAME  # optional, required only when storage account is not in the same resource group as your agent nodes
  storageAccount: EXISTING_STORAGE_ACCOUNT_NAME
  protocol: nfs
```

Après avoir modifié et enregistré le fichier, créez la classe de stockage à l’aide de la commande [kubectl apply][kubectl-apply] :

```console
$ kubectl apply -f nfs-sc.yaml

storageclass.storage.k8s.io/azurefile-csi created
```

### <a name="create-a-deployment-with-an-nfs-backed-file-share"></a>Créer un déploiement avec un partage de fichiers sauvegardé sur NFS
Vous pouvez déployer un exemple [d’ensemble avec état](https://github.com/kubernetes-sigs/azurefile-csi-driver/blob/master/deploy/example/statefulset.yaml) qui enregistre les timestamps dans un fichier `data.txt` en déployant la commande suivante avec la commande [kubectl apply][kubectl-apply] :

 ```console
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/azurefile-csi-driver/master/deploy/example/statefulset.yaml

statefulset.apps/statefulset-azurefile created
```

Validez le contenu du volume en exécutant :

```console
$ kubectl exec -it statefulset-azurefile-0 -- df -h

Filesystem      Size  Used Avail Use% Mounted on
...
/dev/sda1                                                                                 29G   11G   19G  37% /etc/hosts
accountname.file.core.windows.net:/accountname/pvc-fa72ec43-ae64-42e4-a8a2-556606f5da38  100G     0  100G   0% /mnt/azurefile
...
```

>[!NOTE]
> Notez que dans la mesure où le partage de fichiers NFS s’effectue dans le cadre d’un compte Premium, la taille de partage de fichiers minimale est de 100 Go. Si vous créez un PVC avec une petite taille de stockage, vous risquez de rencontrer une erreur « failed to create file share ... size (5)... » (« échec de la création du partage de fichiers ... taille(5)... »).


## <a name="windows-containers"></a>Conteneurs Windows

Le pilote CSI pour Azure Files prend également en charge les nœuds et conteneurs Windows. Si vous souhaitez utiliser des conteneurs Windows, suivez le [Tutoriel sur les conteneurs Windows](windows-container-cli.md) pour ajouter un pool de nœuds Windows.

Une fois que vous disposez d’un pool de nœuds Windows, utilisez les classes de stockage intégrées comme `azurefile-csi` ou créez des classes personnalisées. Vous pouvez déployer un exemple [d’ensemble avec état basé sur Windows](https://github.com/kubernetes-sigs/azurefile-csi-driver/blob/master/deploy/example/windows/statefulset.yaml) qui enregistre les horodatages dans un fichier `data.txt` en déployant la commande suivante avec la commande [kubectl apply][kubectl-apply] :

 ```console
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/azurefile-csi-driver/master/deploy/example/windows/statefulset.yaml

statefulset.apps/busybox-azurefile created
```

Validez le contenu du volume en exécutant :

```console
$ kubectl exec -it busybox-azurefile-0 -- cat c:\\mnt\\azurefile\\data.txt # on Linux/MacOS Bash
$ kubectl exec -it busybox-azurefile-0 -- cat c:\mnt\azurefile\data.txt # on Windows Powershell/CMD

2020-08-27 22:11:01Z
2020-08-27 22:11:02Z
2020-08-27 22:11:04Z
(...)
```

## <a name="next-steps"></a>Étapes suivantes

- Pour savoir comment utiliser les pilotes CSI pour les disques Azure, consultez [Utiliser des disques Azure avec des pilotes CSI](azure-disk-csi.md).
- Pour plus d’informations sur les meilleures pratiques relatives au stockage, consultez [Meilleures pratiques relatives au stockage et aux sauvegardes dans Azure Kubernetes Service][operator-best-practices-storage].


<!-- LINKS - external -->
[access-modes]: https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubernetes-storage-classes]: https://kubernetes.io/docs/concepts/storage/storage-classes/
[kubernetes-volumes]: https://kubernetes.io/docs/concepts/storage/persistent-volumes/
[managed-disk-pricing-performance]: https://azure.microsoft.com/pricing/details/managed-disks/
[smb-overview]: /windows/desktop/FileIO/microsoft-smb-protocol-and-cifs-protocol-overview


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
[az-extension-add]: /cli/azure/extension?view=azure-cli-latest#az-extension-add&preserve-view=true
[az-extension-update]: /cli/azure/extension?view=azure-cli-latest#az-extension-update&preserve-view=true
[az-feature-register]: /cli/azure/feature?view=azure-cli-latest#az-feature-register&preserve-view=true
[az-feature-list]: /cli/azure/feature?view=azure-cli-latest#az-feature-list&preserve-view=true
[az-provider-register]: /cli/azure/provider?view=azure-cli-latest#az-provider-register&preserve-view=true
[node-resource-group]: faq.md#why-are-two-resource-groups-created-with-aks
[storage-skus]: ../storage/common/storage-redundancy.md
