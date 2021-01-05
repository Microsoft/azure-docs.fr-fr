---
title: Utiliser des pilotes CSI (Container Storage interface) pour les disques Azure sur Azure Kubernetes Service (AKS)
description: Découvrez comment utiliser des pilotes CSI (Container Storage interface) pour les disques Azure sur un cluster Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 08/27/2020
author: palma21
ms.openlocfilehash: 2dba9fbcbddbc7a66763636986f3d98f4f95332c
ms.sourcegitcommit: c157b830430f9937a7fa7a3a6666dcb66caa338b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94683129"
---
# <a name="use-the-azure-disk-container-storage-interface-csi-drivers-in-azure-kubernetes-service-aks-preview"></a>Utiliser le disque Azure CSI (Container Storage interface) pour Azure Files dans Azure Kubernetes Service (AKS) (préversion)
Le disque CSI (Container Storage interface) pour Azure Files est un pilote conforme à la [spécification CSI](https://github.com/container-storage-interface/spec/blob/master/spec.md) utilisé par Azure Kubernetes Service (AKS) pour gérer le cycle de vie des disques Azure.

CSI est une norme pour exposer des systèmes de stockage de blocs et de fichiers arbitraires à des charges de travail en conteneur sur Kubernetes. En adoptant et en utilisant CSI, AKS peut écrire, déployer et itérer les plug-ins afin d’exposer de nouveaux systèmes de stockage ou améliorer ceux existants dans Kubernetes sans avoir à toucher au code Kubernetes principal et attendre ses cycles de publication.

Pour créer un cluster AKS avec prise en charge du pilote CSI, consultez [Activer les pilotes CSI pour les disques Azure et Azure Files sur AKS](csi-storage-drivers.md).

>[!NOTE]
> *Les pilotes dans l’arborescence* sont les pilotes de stockage actuels, qui font partie du code Kubernetes principal, et les nouveaux pilotes CSI, qui sont des plug-ins.

## <a name="use-csi-persistent-volumes-with-azure-disks"></a>Utiliser des volumes persistants CSI avec les disques Azure

Un [volume persistant](concepts-storage.md#persistent-volumes) représente un élément de stockage provisionné pour une utilisation dans des pods Kubernetes. Un volume persistant peut être utilisé par un ou plusieurs pods et être provisionné de façon statique ou dynamique. Cet article vous montre comment créer des volumes persistants de manière dynamique avec des disques Azure pour permettre à un pod unique de les utiliser dans un cluster AKS. Pour l’approvisionnement statique, consultez [Créer et utiliser manuellement un volume avec des disques Azure](azure-disk-volume.md).

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

Pour plus d’informations sur les volumes Kubernetes, consultez [Options de stockage pour les applications dans AKS][concepts-storage].

## <a name="dynamically-create-azure-disk-pvs-by-using-the-built-in-storage-classes"></a>Créer dynamiquement des volumes persistants de disque Azure à l’aide des classes de stockage intégrées

Une classe de stockage permet de définir la création dynamique d’une unité de stockage avec un volume persistant. Pour plus d’informations sur les classes de stockage Kubernetes, consultez [Classes de stockage Kubernetes][kubernetes-storage-classes]. Lorsque vous utilisez des pilotes de stockage CSI sur AKS, il existe deux `StorageClasses` supplémentaires intégrées qui utilisent les pilotes de stockage CSI de disque Azure. Les classes de stockage CSI supplémentaires sont créées avec le cluster en même temps que les classes de stockage par défaut dans l’arborescence.

- `managed-csi`: Utilise un stockage localement redondant (LRS) Azure Standard SSD pour créer un disque managé.
- `managed-csi-premium`: Utilise un stockage localement redondant (LRS) Azure Premium pour créer un disque managé.

La stratégie de récupération des deux classes de stockage garantit que le disque Azure sous-jacent est supprimé lorsque le volume persistant respectif est supprimé. Les classes de stockage configurent également les volumes persistants pour qu’ils soient extensibles. Vous devez simplement modifier la revendication de volume persistant (PVC) avec la nouvelle taille.

Pour tirer parti de ces classes de stockage, créez un [PVC](concepts-storage.md#persistent-volume-claims) et un pod respectif qui les référence et les utilise. Une revendication de volume persistant est utilisée pour configurer automatiquement le stockage basé sur une classe de stockage. Un PVC peut utiliser l’une des classes de stockage précréées ou une classe de stockage définie par l’utilisateur pour créer un disque managé Azure pour la référence SKU et la taille de votre choix. Lorsque vous créez une définition de pod, la revendication de volume persistant est spécifiée pour demander le stockage souhaité.

Créez un exemple de pod et un PVC respectif avec la commande [kubectl apply][kubectl-apply] :

```console
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/azuredisk-csi-driver/master/deploy/example/pvc-azuredisk-csi.yaml
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/azuredisk-csi-driver/master/deploy/example/nginx-pod-azuredisk.yaml

persistentvolumeclaim/pvc-azuredisk created
pod/nginx-azuredisk created
```

Une fois que le pod en cours d’exécution, créez un fichier appelé `test.txt`.

```bash
$ kubectl exec nginx-azuredisk -- touch /mnt/azuredisk/test.txt
```

Vous pouvez maintenant vérifier que le disque est correctement monté en exécutant la commande suivante et en vérifiant la présence du fichier `test.txt` dans la sortie :

```console
$ kubectl exec nginx-azuredisk -- ls /mnt/azuredisk

lost+found
outfile
test.txt
```

## <a name="create-a-custom-storage-class"></a>Créer une classe de stockage personnalisée

Les classes de stockage par défaut sont adaptées aux scénarios les plus courants, mais pas à tous. Dans certains cas, vous souhaiterez peut-être personnaliser votre propre classe de stockage avec vos propres paramètres. Par exemple, dans le cadre d’un scénario, vous pouvez être amené à modifier la classe `volumeBindingMode`.

Les classes de stockage par défaut utilisent une classe `volumeBindingMode: Immediate` qui garantit que cela se produit immédiatement après la création du PVC. Si vos pools de nœuds sont limités en termes de topologie, par exemple à l’aide de zones de disponibilité, les volumes persistants sont liés ou approvisionnés sans tenir compte des exigences de planification du pod (dans ce cas, il se trouve dans une zone spécifique).

Dans un tel scénario, vous pouvez utiliser `volumeBindingMode: WaitForFirstConsumer`, qui retarde la liaison et l’approvisionnement d’un volume persistant jusqu’à ce qu’un pod utilisant le PVC soit créé. Ainsi, le volume persistant est conforme et approvisionné dans la zone de disponibilité (ou une autre topologie) spécifiée par les contraintes de planification du pod.

Créez un fichier nommé `sc-azuredisk-csi-waitforfirstconsumer.yaml` et collez le manifeste suivant.
La classe de stockage est identique à la classe de stockage `managed-csi`, mais avec une classe `volumeBindingMode` différente.

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: azuredisk-csi-waitforfirstconsumer
provisioner: disk.csi.azure.com
parameters:
  skuname: StandardSSD_LRS 
allowVolumeExpansion: true
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```

Créez la classe de stockage à l’aide de la commande [kubectl apply][kubectl-apply] et spécifiez votre fichier `sc-azuredisk-csi-waitforfirstconsumer.yaml` :

```console
$ kubectl apply -f sc-azuredisk-csi-waitforfirstconsumer.yaml

storageclass.storage.k8s.io/azuredisk-csi-waitforfirstconsumer created
```

## <a name="volume-snapshots"></a>Clichés instantanés de volume

Le pilote CSI de disque Azure prend en charge la création de [captures instantanées de volumes persistants](https://kubernetes-csi.github.io/docs/snapshot-restore-feature.html). Dans le cadre de cette fonctionnalité, le pilote peut effectuer des *instantanés* [*complets*  ou incrémentiels](../virtual-machines/windows/disks-incremental-snapshots.md) en fonction de la valeur définie dans le paramètre `incremental` (true, par défaut).

Pour plus d’informations sur tous les paramètres, consultez [Paramètres de classe des instantanés de volume](https://github.com/kubernetes-sigs/azuredisk-csi-driver/blob/master/docs/driver-parameters.md#volumesnapshotclass).

### <a name="create-a-volume-snapshot"></a>Créer un instantané de volume

Pour une illustration de cette fonctionnalité, créez une [classe d’instantané de volume](https://github.com/kubernetes-sigs/azuredisk-csi-driver/blob/master/deploy/example/snapshot/storageclass-azuredisk-snapshot.yaml) avec la commande [kubectl apply][kubectl-apply] :

```console
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/azuredisk-csi-driver/master/deploy/example/snapshot/storageclass-azuredisk-snapshot.yaml

volumesnapshotclass.snapshot.storage.k8s.io/csi-azuredisk-vsc created
```

Nous allons à présent créer un [instantané de volume](https://github.com/kubernetes-sigs/azuredisk-csi-driver/blob/master/deploy/example/snapshot/azuredisk-volume-snapshot.yaml) à partir du PVC [que nous avons créé de façon dynamique au début de ce tutoriel](#dynamically-create-azure-disk-pvs-by-using-the-built-in-storage-classes), `pvc-azuredisk`.


```bash
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/azuredisk-csi-driver/master/deploy/example/snapshot/azuredisk-volume-snapshot.yaml

volumesnapshot.snapshot.storage.k8s.io/azuredisk-volume-snapshot created
```

Vérifiez que l’instantané a été créé correctement :

```bash
$ kubectl describe volumesnapshot azuredisk-volume-snapshot

Name:         azuredisk-volume-snapshot
Namespace:    default
Labels:       <none>
Annotations:  API Version:  snapshot.storage.k8s.io/v1beta1
Kind:         VolumeSnapshot
Metadata:
  Creation Timestamp:  2020-08-27T05:27:58Z
  Finalizers:
    snapshot.storage.kubernetes.io/volumesnapshot-as-source-protection
    snapshot.storage.kubernetes.io/volumesnapshot-bound-protection
  Generation:        1
  Resource Version:  714582
  Self Link:         /apis/snapshot.storage.k8s.io/v1beta1/namespaces/default/volumesnapshots/azuredisk-volume-snapshot
  UID:               dd953ab5-6c24-42d4-ad4a-f33180e0ef87
Spec:
  Source:
    Persistent Volume Claim Name:  pvc-azuredisk
  Volume Snapshot Class Name:      csi-azuredisk-vsc
Status:
  Bound Volume Snapshot Content Name:  snapcontent-dd953ab5-6c24-42d4-ad4a-f33180e0ef87
  Creation Time:                       2020-08-31T05:27:59Z
  Ready To Use:                        true
  Restore Size:                        10Gi
Events:                                <none>
```

### <a name="create-a-new-pvc-based-on-a-volume-snapshot"></a>Créer un PVC basé sur un instantané de volume

Vous pouvez créer un PVC basé sur un instantané de volume. Utilisez l’instantané créé à l’étape précédente, puis créez un [PVC](https://github.com/kubernetes-sigs/azuredisk-csi-driver/blob/master/deploy/example/snapshot/pvc-azuredisk-snapshot-restored.yaml) et un [pod](https://github.com/kubernetes-sigs/azuredisk-csi-driver/blob/master/deploy/example/snapshot/nginx-pod-restored-snapshot.yaml) pour le consommer.

```console
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/azuredisk-csi-driver/master/deploy/example/snapshot/pvc-azuredisk-snapshot-restored.yaml

$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/azuredisk-csi-driver/master/deploy/example/snapshot/nginx-pod-restored-snapshot.yaml

persistentvolumeclaim/pvc-azuredisk-snapshot-restored created
pod/nginx-restored created
```

Enfin, assurez-vous qu’il s’agit du même PVC que celui créé précédemment en vérifiant son contenu.

```console
$ kubectl exec nginx-restored -- ls /mnt/azuredisk

lost+found
outfile
test.txt
```

Comme attendu, nous pouvons toujours voir le fichier `test.txt` créé précédemment.

## <a name="clone-volumes"></a>Cloner des volumes

Un volume cloné est défini comme un doublon d’un volume Kubernetes existant. Pour plus d’informations sur le clonage des volumes dans Kubernetes, consultez la documentation conceptuelle relative au [clonage de volume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#volume-cloning).

Le pilote CSI des disques Azure prend en charge le clonage de volume. Pour une illustration, créez un [volume cloné](https://github.com/kubernetes-sigs/azuredisk-csi-driver/blob/master/deploy/example/cloning/nginx-pod-restored-cloning.yaml) de celui [créé précédemment](#dynamically-create-azure-disk-pvs-by-using-the-built-in-storage-classes) `azuredisk-pvc` et [un pod pour le consommer](https://github.com/kubernetes-sigs/azuredisk-csi-driver/blob/master/deploy/example/cloning/nginx-pod-restored-cloning.yaml).


```console
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/azuredisk-csi-driver/master/deploy/example/cloning/pvc-azuredisk-cloning.yaml

$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/azuredisk-csi-driver/master/deploy/example/cloning/nginx-pod-restored-cloning.yaml

persistentvolumeclaim/pvc-azuredisk-cloning created
pod/nginx-restored-cloning created
```

Nous pouvons maintenant vérifier le contenu du volume cloné en exécutant la commande suivante et en vérifiant que nous voyons toujours notre fichier créé `test.txt`.

```console
$ kubectl exec nginx-restored-cloning -- ls /mnt/azuredisk

lost+found
outfile
test.txt
```

## <a name="resize-a-persistent-volume"></a>Redimensionner un volume persistant

Vous pouvez demander un volume plus important pour un PVC. Modifiez l’objet PVC et spécifiez une taille supérieure. Cette modification déclenche l’expansion du volume sous-jacent qui stocke le PV.

> [!NOTE]
> Un nouveau PV n’est jamais créé pour satisfaire la revendication. Au lieu de cela, un volume existant est redimensionné.

Dans AKS, la classe de stockage intégrée `managed-csi` permet l’expansion, vous pouvez donc utiliser le [PVC créé précédemment avec cette classe de stockage](#dynamically-create-azure-disk-pvs-by-using-the-built-in-storage-classes). Le PVC a demandé un volume persistant de 10 GI. Nous pouvons confirmer cela en exécutant :

```console 
$ kubectl exec -it nginx-azuredisk -- df -h /mnt/azuredisk

Filesystem      Size  Used Avail Use% Mounted on
/dev/sdc        9.8G   42M  9.8G   1% /mnt/azuredisk
```

> [!IMPORTANT]
> Actuellement, le pilote CSI de disque Azure prend uniquement en charge le redimensionnement des PVC sans pod associé (et sans volume monté sur un nœud spécifique).

Nous allons donc supprimer le pod que nous avons créé précédemment :

```console
$ kubectl delete -f https://raw.githubusercontent.com/kubernetes-sigs/azuredisk-csi-driver/master/deploy/example/nginx-pod-azuredisk.yaml

pod "nginx-azuredisk" deleted
```

Développons le PVC en augmentant le champ `spec.resources.requests.storage` :

```console
$ kubectl patch pvc pvc-azuredisk --type merge --patch '{"spec": {"resources": {"requests": {"storage": "15Gi"}}}}'

persistentvolumeclaim/pvc-azuredisk patched
```

Vérifions que le volume est désormais plus grand :

```console
$ kubectl get pv

NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                     STORAGECLASS   REASON   AGE
pvc-391ea1a6-0191-4022-b915-c8dc4216174a   15Gi       RWO            Delete           Bound    default/pvc-azuredisk                     managed-csi             2d2h
(...)
```

> [!NOTE]
> Le PVC ne reflète pas la nouvelle taille tant qu’il n’est pas associé à un pod.

Créons un nouveau pod :

```console
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/azuredisk-csi-driver/master/deploy/example/nginx-pod-azuredisk.yaml

pod/nginx-azuredisk created
```

Enfin, vérifions la taille du PVC et à l’intérieur du pod :
```console
$ kubectl get pvc pvc-azuredisk
NAME            STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc-azuredisk   Bound    pvc-391ea1a6-0191-4022-b915-c8dc4216174a   15Gi       RWO            managed-csi    2d2h

$ kubectl exec -it nginx-azuredisk -- df -h /mnt/azuredisk
Filesystem      Size  Used Avail Use% Mounted on
/dev/sdc         15G   46M   15G   1% /mnt/azuredisk
```

## <a name="shared-disk"></a>Disque partagé

[Disques partagés Azure](../virtual-machines/disks-shared.md) est une fonctionnalité de disques managés Azure qui permet d’attacher un disque Azure à des nœuds d’agent simultanément. Le fait d’attacher un disque managé à plusieurs nœuds d’agent vous permet, par exemple, de déployer de nouvelles applications en cluster ou de migrer des applications en cluster existantes vers Azure.

> [!IMPORTANT] 
> Actuellement, seul le fichier spécial en mode bloc brut (`volumeMode: Block`) est pris en charge par le pilote CSI du disque Azure. Les applications doivent gérer la coordination et le contrôle des écritures, des lectures, des verrous, des caches, des montages et des clôturages sur le disque partagé, qui est exposé en tant que fichier spécial en mode bloc brut.

Nous allons créer un fichier appelé `shared-disk.yaml` en copiant la commande suivante, qui contient la classe de stockage sur disque partagé et le PVC :

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: managed-csi-shared
provisioner: disk.csi.azure.com
parameters:
  skuname: Premium_LRS  # Currently shared disk is only available with premium SSD
  maxShares: "2"
  cachingMode: None  # ReadOnly cache is not available for premium SSD with maxShares>1
reclaimPolicy: Delete
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc-azuredisk-shared
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 256Gi  # minimum size of shared disk is 256GB (P15)
  volumeMode: Block
  storageClassName: managed-csi-shared
```

Créez la classe de stockage à l’aide de la commande [kubectl apply][kubectl-apply] et spécifiez votre fichier `shared-disk.yaml` :

```console
$ kubectl apply -f shared-disk.yaml

storageclass.storage.k8s.io/managed-csi-shared created
persistentvolumeclaim/pvc-azuredisk-shared created
``` 

Créons maintenant un fichier appelé `deployment-shared.yml` en copiant la commande suivante :

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: deployment-azuredisk
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
      name: deployment-azuredisk
    spec:
      containers:
        - name: deployment-azuredisk
          image: mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine
          volumeDevices:
            - name: azuredisk
              devicePath: /dev/sdx
      volumes:
        - name: azuredisk
          persistentVolumeClaim:
            claimName: pvc-azuredisk-shared
```

Créez le déploiement à l’aide de la commande [kubectl apply][kubectl-apply] et spécifiez votre fichier `deployment-shared.yml` :

```console
$ kubectl apply -f deployment-shared.yml

deployment/deployment-azuredisk created
```

Enfin, nous allons vérifier le fichier spécial en mode bloc à l’intérieur du pod :

```console
# kubectl exec -it deployment-sharedisk-7454978bc6-xh7jp sh
/ # dd if=/dev/zero of=/dev/sdx bs=1024k count=100
100+0 records in
100+0 records out/s
```

## <a name="windows-containers"></a>Conteneurs Windows

Le pilote CSI de disque Azure prend également en charge les nœuds et conteneurs Windows. Si vous souhaitez utiliser des conteneurs Windows, suivez le [Tutoriel sur les conteneurs Windows](windows-container-cli.md) pour ajouter un pool de nœuds Windows.

Lorsque vous disposez d’un pool de nœuds Windows, vous pouvez utiliser les classes de stockage intégrées comme `managed-csi`. Vous pouvez déployer un exemple [d’ensemble avec état basé sur Windows](https://github.com/kubernetes-sigs/azuredisk-csi-driver/blob/master/deploy/example/windows/statefulset.yaml) qui enregistre les horodatages dans le fichier `data.txt` en déployant la commande suivante avec la commande [kubectl apply][kubectl-apply] :

 ```console
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/azuredisk-csi-driver/master/deploy/example/windows/statefulset.yaml

statefulset.apps/busybox-azuredisk created
```

Vous pouvez valider le contenu du volume en exécutant :

```console
$ kubectl exec -it busybox-azuredisk-0 -- cat c:\\mnt\\azuredisk\\data.txt # on Linux/MacOS Bash
$ kubectl exec -it busybox-azuredisk-0 -- cat c:\mnt\azuredisk\data.txt # on Windows Powershell/CMD

2020-08-27 08:13:41Z
2020-08-27 08:13:42Z
2020-08-27 08:13:44Z
(...)
```

## <a name="next-steps"></a>Étapes suivantes

- Pour savoir comment utiliser les pilotes CSI pour Azure Files, consultez [Utiliser Azure Files avec des pilotes CSI](azure-files-csi.md).
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