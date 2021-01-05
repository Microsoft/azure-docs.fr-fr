---
title: 'Concepts : stockage dans AKS (Azure Kubernetes Service)'
description: Découvrez le stockage dans AKS (Azure Kubernetes Service), notamment les volumes, les volumes persistants, les classes de stockage et les revendications
services: container-service
ms.topic: conceptual
ms.date: 08/17/2020
ms.openlocfilehash: 0ed38625703397c9ba5021e84cd3118f30fa83c7
ms.sourcegitcommit: 693df7d78dfd5393a28bf1508e3e7487e2132293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92900944"
---
# <a name="storage-options-for-applications-in-azure-kubernetes-service-aks"></a>Options de stockage pour les applications dans AKS (Azure Kubernetes Service)

Les applications qui s’exécutent dans AKS (Azure Kubernetes Service) peuvent avoir besoin de stocker et de récupérer des données. Pour certaines charges de travail d’applications, ce stockage de données peut utiliser un stockage local rapide sur le nœud qui n’est plus nécessaire quand les pods sont supprimés. D’autres charges de travail d’applications peuvent nécessiter un stockage persistant sur des volumes de données plus réguliers au sein de la plateforme Azure. Plusieurs pods peuvent avoir besoin de partager les mêmes volumes de données ou de rattacher des volumes de données si le pod est replanifié sur un autre nœud. Enfin, vous pouvez être amené à injecter des données sensibles ou des informations de configuration d’application dans des pods.

![Options de stockage pour les applications dans un cluster AKS (Azure Kubernetes Service)](media/concepts-storage/aks-storage-options.png)

Cet article présente les concepts fondamentaux qui fournissent le stockage à vos applications dans AKS :

- [Volumes](#volumes)
- [Volumes persistants](#persistent-volumes)
- [Classes de stockage](#storage-classes)
- [Revendications de volume persistant](#persistent-volume-claims)

## <a name="volumes"></a>Volumes

Les applications doivent souvent être en mesure de stocker et de récupérer des données. Comme Kubernetes traite normalement les pods individuels en tant que ressources éphémères jetables, les applications disposent de différentes méthodes pour utiliser et conserver les données en fonction des besoins. Un *volume* représente un moyen de stocker, de récupérer et de conserver les données entre les pods et tout au long du cycle de vie des applications.

Les volumes traditionnels pour stocker et récupérer les données sont créés en tant que ressources Kubernetes soutenues par Stockage Azure. Vous pouvez créer manuellement ces volumes de données en vue de les assigner à des pods directement, ou laisser Kubernetes les créer automatiquement. Ces volumes de données peuvent utiliser des disques Azure ou Azure Files :

- Les *disques Azure* permettent de créer une ressource *DataDisk* Kubernetes. Les disques peuvent utiliser un stockage Azure Premium, assorti de disques SSD hautes performances, ou le stockage Azure Standard, assorti de disques HDD standards. Pour la plupart des charges de travail de production et de développement, utilisez le stockage Premium. Les disques Azure étant montés avec le mode d’accès *ReadWriteOnce* , ils ne sont disponibles que pour un seul pod. Pour les volumes de stockage accessibles par plusieurs pods simultanément, utilisez Azure Files.
- *Azure Files* permet de monter un partage SMB 3.0 assorti d’un compte Stockage Azure sur des pods. Avec Azure Files, vous pouvez partager des données entre plusieurs nœuds et pods. Les fichiers peuvent utiliser un stockage Azure Standard, assorti de disques HDD standard, ou un stockage Azure Premium, assorti de disques SSD hautes performances.

Dans Kubernetes, les volumes ne se limitent pas forcément à un disque classique sur lequel des informations peuvent être stockées et récupérées. Les volumes Kubernetes peuvent également servir à injecter des données dans un pod en vue de leur utilisation par les conteneurs. Voici certains types de volume supplémentaires courants dans Kubernetes :

- *emptyDir*  : ce volume est couramment utilisé comme espace temporaire pour un pod. Tous les conteneurs au sein d’un pod peuvent accéder aux données sur le volume. Les données écrites sur ce type de volume sont conservées jusqu’à la fin de la durée de vie du pod ; quand le pod est supprimé, le volume est également supprimé. Ce volume utilise généralement le stockage sur disque du nœud local sous-jacent, bien qu’il puisse aussi se trouver uniquement dans la mémoire du nœud.
- *secret*  : ce volume est utilisé pour injecter des données sensibles dans les pods, telles que les mots de passe. Tout d’abord, vous créez un secret à l’aide de l’API Kubernetes. Quand vous définissez votre pod ou déploiement, un secret spécifique peut être demandé. Les secrets sont fournis uniquement aux nœuds dont un pod planifié a besoin d’un secret. En outre, le secret est stocké dans un volume *tmpfs* , au lieu d’être écrit sur le disque. Quand le dernier pod sur un nœud qui requiert un secret est supprimé, ce dernier est supprimé du volume tmpfs du nœud. Les secrets sont stockés dans un espace de noms donné et ne sont accessibles qu’aux pods se trouvant dans cet espace de noms.
- *configMap*  : ce type de volume est utilisé pour injecter des propriétés de paires clé-valeur dans des pods, telles que des informations de configuration d’application. Au lieu de définir des informations de configuration d’application au sein d’une image conteneur, vous pouvez les définir en tant que ressource Kubernetes pouvant être facilement mise à jour et appliquée à de nouvelles instances de pods au fur et à mesure de leur déploiement. Comme dans le cas d’un secret, vous créez tout d’abord un volume ConfigMap à l’aide de l’API Kubernetes. Ce volume ConfigMap peut ensuite être demandé quand vous définissez un pod ou un déploiement. Les volumes ConfigMap sont stockés dans un espace de noms donné et ne sont accessibles qu’aux pods se trouvant dans cet espace de noms.

## <a name="persistent-volumes"></a>Volumes persistants

Les volumes qui sont définis et créés dans le cadre du cycle de vie d’un pod sont conservés jusqu’à ce que le pod soit supprimé. Le stockage d’un pod est censé être conservé si le pod est replanifié sur un autre hôte pendant un événement de maintenance, en particulier dans les ressources StatefulSet. Un *volume persistant* est une ressource de stockage créée et gérée par l’API Kubernetes, et qui peut exister au-delà de la durée de vie d’un pod donné.

Des disques Azure ou Azure Files sont utilisés pour fournir le volume persistant (PersistentVolume). Comme indiqué dans la section précédente sur les volumes, le choix de disques Azure ou d’Azure Files est souvent déterminé par le niveau de performance ou la nécessité d’un accès simultané aux données.

![Volumes persistants dans un cluster AKS (Azure Kubernetes Service)](media/concepts-storage/persistent-volumes.png)

Un volume persistant peut être créé *statiquement* par un administrateur de cluster, ou *dynamiquement* par le serveur d’API Kubernetes. Si un pod est planifié et demande un stockage qui n’est pas disponible, Kubernetes peut créer le stockage sur disques Azure ou Azure Files sous-jacent et l’attacher au pod. Le provisionnement dynamique utilise une classe de stockage ( *StorageClass* ) pour identifier le type de stockage Azure à créer.

## <a name="storage-classes"></a>Classes de stockage

Pour définir différents niveaux de stockage, tels que Premium et Standard, vous pouvez créer une classe de stockage ( *StorageClass* ). La classe de stockage définit également la stratégie de récupération ( *reclaimPolicy* ). Cette stratégie de récupération contrôle le comportement de la ressource de stockage Azure sous-jacente quand le pod est supprimé et que le volume persistant n’est peut-être plus nécessaire. La ressource de stockage sous-jacente peut être supprimée ou conservée en vue de son utilisation par un pod futur.

Dans AKS, quatre `StorageClasses` initiales sont créées pour le cluster à l’aide des plug-ins de stockage dans l’arborescence :

- `default` : utilise le stockage Azure StandardSSD pour créer un disque managé. La stratégie de récupération veille à ce que le disque Azure sous-jacent soit supprimé quand le volume persistant qui l’a utilisé est supprimé.
- `managed-premium` : utilise un stockage Premium Azure pour créer un disque managé. La stratégie de récupération veille à nouveau à ce que le disque Azure sous-jacent soit supprimé quand le volume persistant qui l’a utilisé est supprimé.
- `azurefile` : utilise le stockage Azure Standard pour créer un partage de fichiers Azure. La stratégie de récupération veille à ce que le partage de fichiers Azure sous-jacent soit supprimé quand le volume persistant qui l’a utilisé est supprimé.
- `azurefile-premium` : utilise le stockage Azure Premium pour créer un partage de fichiers Azure. La stratégie de récupération veille à ce que le partage de fichiers Azure sous-jacent soit supprimé quand le volume persistant qui l’a utilisé est supprimé.

Pour les clusters qui utilisent les nouveaux plug-ins externes d’interface de stockage de conteneur (Container Storage interface CSI) (préversion), les `StorageClasses` supplémentaires suivantes sont créées :
- `managed-csi` : utilise un stockage localement redondant (LRS) Azure StandardSSD pour créer un disque managé. La stratégie de récupération veille à ce que le disque Azure sous-jacent soit supprimé quand le volume persistant qui l’a utilisé est supprimé. La classe de stockage configure également les volumes persistants pour qu’ils soient extensibles. Il vous suffit de modifier la revendication de volume persistant avec la nouvelle taille.
- `managed-csi-premium` : utilise un stockage localement redondant (LRS) Azure Premium pour créer un disque managé. La stratégie de récupération veille à nouveau à ce que le disque Azure sous-jacent soit supprimé quand le volume persistant qui l’a utilisé est supprimé. De même, cette classe de stockage permet l’extension des volumes persistants.
- `azurefile-csi` : utilise le stockage Azure Standard pour créer un partage de fichiers Azure. La stratégie de récupération veille à ce que le partage de fichiers Azure sous-jacent soit supprimé quand le volume persistant qui l’a utilisé est supprimé.
- `azurefile-csi-premium` : utilise le stockage Azure Premium pour créer un partage de fichiers Azure. La stratégie de récupération veille à ce que le partage de fichiers Azure sous-jacent soit supprimé quand le volume persistant qui l’a utilisé est supprimé.

Si aucune classe de stockage n’est spécifiée pour un volume persistant, la classe de stockage par défaut est utilisée. Quand vous demandez des volumes persistants, veillez à ce qu’ils utilisent le stockage dont vous avez besoin. Vous pouvez créer une classe de stockage pour des besoins supplémentaires à l’aide de `kubectl`. L’exemple suivant utilise la fonctionnalité Disques managés Premium et spécifie que le disque Azure sous-jacent doit être *conservé* quand le pod est supprimé :

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: managed-premium-retain
provisioner: kubernetes.io/azure-disk
reclaimPolicy: Retain
parameters:
  storageaccounttype: Premium_LRS
  kind: Managed
```

> [!NOTE]
> AKS rapproche les classes de stockage par défaut et remplace toutes les modifications que vous apportez à ces classes de stockage.

## <a name="persistent-volume-claims"></a>Revendications de volume persistant

Une revendication de volume persistant (PersistentVolumeClaim) demande un stockage sur disque Azure ou Azure Files selon une classe de stockage, un mode d’accès et une taille spécifiques. Le serveur d’API Kubernetes peut provisionner dynamiquement la ressource de stockage sous-jacente dans Azure si aucune ressource existante ne satisfait à la revendication selon la classe de stockage définie. La définition du pod inclut le montage du volume une fois que ce dernier a été connecté au pod.

![Revendications de volumes persistants dans un cluster AKS (Azure Kubernetes Service)](media/concepts-storage/persistent-volume-claims.png)

Un volume persistant est *lié* à une revendication de volume persistant une fois qu’une ressource de stockage disponible a été assignée au pod qui la demande. Les volumes persistants sont liés aux revendications par un mappage 1 à 1.

L’exemple de manifeste YAML suivant montre une revendication de volume persistant qui utilise la classe de stockage *managed-premium* et demande un disque ayant une taille de *5Gi*  :

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: azure-managed-disk
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: managed-premium
  resources:
    requests:
      storage: 5Gi
```

Quand vous créez une définition de pod, la revendication de volume persistant est spécifiée pour demander le stockage souhaité. Vous spécifiez également le montage de volume ( *volumeMount* ) permettant à vos applications de lire et d’écrire des données. L’exemple de manifeste YAML suivant illustre l’utilisation de la revendication de volume persistant précédente pour monter un volume sur */mnt/azure*  :

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: nginx
spec:
  containers:
    - name: myfrontend
      image: mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine
      volumeMounts:
      - mountPath: "/mnt/azure"
        name: volume
  volumes:
    - name: volume
      persistentVolumeClaim:
        claimName: azure-managed-disk
```

## <a name="next-steps"></a>Étapes suivantes

Pour connaître les meilleures pratiques associées, consultez [Meilleures pratiques relatives au stockage et aux sauvegardes dans Azure Kubernetes Service (AKS)][operator-best-practices-storage].

Pour voir comment créer des volumes dynamiques et statiques qui utilisent des disques Azure ou Azure Files, consultez les articles de procédures suivants :

- [Créer un volume statique à l’aide de disques Azure][aks-static-disks]
- [Créer un volume statique à l’aide d’Azure Files][aks-static-files]
- [Créer un volume dynamique à l’aide de disques Azure][aks-dynamic-disks]
- [Créer un volume dynamique à l’aide d’Azure Files][aks-dynamic-files]

Pour plus d’informations sur les concepts fondamentaux de Kubernetes et d’AKS, consultez les articles suivants :

- [Clusters et charges de travail Kubernetes/AKS][aks-concepts-clusters-workloads]
- [Identité Kubernetes/AKS][aks-concepts-identity]
- [Sécurité Kubernetes/AKS][aks-concepts-security]
- [Réseaux virtuels Kubernetes/AKS][aks-concepts-network]
- [Mise à l’échelle Kubernetes/AKS][aks-concepts-scale]

<!-- EXTERNAL LINKS -->

<!-- INTERNAL LINKS -->
[aks-static-disks]: azure-disk-volume.md
[aks-static-files]: azure-files-volume.md
[aks-dynamic-disks]: azure-disks-dynamic-pv.md
[aks-dynamic-files]: azure-files-dynamic-pv.md
[aks-concepts-clusters-workloads]: concepts-clusters-workloads.md
[aks-concepts-identity]: concepts-identity.md
[aks-concepts-scale]: concepts-scale.md
[aks-concepts-security]: concepts-security.md
[aks-concepts-network]: concepts-network.md
[operator-best-practices-storage]: operator-best-practices-storage.md
