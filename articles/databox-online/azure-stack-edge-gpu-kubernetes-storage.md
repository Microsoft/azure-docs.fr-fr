---
title: Comprendre la gestion du stockage Kubernetes sur un appareil Azure Stack Edge Pro | Microsoft Docs
description: Décrit comment la gestion du stockage Kubernetes s’effectue sur un appareil Azure Stack Edge Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: conceptual
ms.date: 11/04/2020
ms.author: alkohli
ms.openlocfilehash: 34165071238ca3edf78ab9cca43639c23ce5ed2a
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96448701"
---
# <a name="kubernetes-storage-management-on-your-azure-stack-edge-pro-gpu-device"></a>Gestion du stockage Kubernetes sur votre appareil Azure Stack Edge Pro avec GPU

Sur votre appareil Azure Stack Edge Pro, un cluster Kubernetes est créé lorsque vous configurez le rôle de calcul. Une fois le cluster Kubernetes créé, les applications en conteneur peuvent être déployées sur le cluster Kubernetes dans des pods. Il existe différents moyens de fournir un stockage aux pods dans votre cluster Kubernetes. 

Cet article décrit les méthodes de provisionnement du stockage dans un cluster Kubernetes en général, et dans le contexte de votre appareil Azure Stack Edge Pro en particulier. 

## <a name="storage-requirements-for-kubernetes-pods"></a>Exigences en matière de stockage pour les pods Kubernetes

Les pods Kubernetes sont sans état, mais les applications qu’ils exécutent sont généralement avec état. Dans la mesure où les pods peuvent être éphémères et qu’ils redémarrent, basculent ou se déplacent entre les nœuds Kubernetes, les conditions suivantes doivent être remplies pour le stockage associé à un pod. 

Le stockage doit :

- Résider en dehors du pod.
- Être indépendant du cycle de vie du pod.
- Être accessible depuis tous les nœuds Kubernetes.

Comprendre comment le stockage est géré pour Kubernetes implique de comprendre deux ressources d’API : 

- **PersistentVolume (PV)**  : il s’agit d’un élément de stockage présent dans le cluster Kubernetes. Le stockage Kubernetes peut être provisionné de manière statique en tant que `PersistentVolume`. Il peut également être provisionné de manière dynamique en tant que `StorageClass`.

- **PersistentVolumeClaim (PVC)**  : il relève d’une demande de stockage effectuée par un utilisateur. Les PVC consomment des ressources PV. Les PVC peuvent exiger des modes d’accès et de dimensionnement spécifiques. 

    Comme les utilisateurs ont besoin de `PersistentVolumes` disposant de propriétés variées pour des problèmes différents, les administrateurs de cluster doivent être en mesure d’offrir toute une panoplie de `PersistentVolumes` qui se différencient autrement que par les simples modes d’accès et de dimensionnement. La ressource `StorageClass` vous permet de répondre à ces besoins.

Le provisionnement du stockage peut être statique ou dynamique. Chaque type de provisionnement est abordé dans les sections suivantes.

## <a name="static-provisioning"></a>Provisionnement statique

Les administrateurs de cluster Kubernetes peuvent provisionner le stockage de manière statique. Pour ce faire, ils peuvent utiliser un back-end de stockage basé sur des systèmes de fichiers SMB/NFS, ou se servir de disques iSCSI qui s’attachent localement sur le réseau dans un environnement local, voire même utiliser Azure Files ou des disques Azure dans le cloud. Ce type de stockage n’étant pas provisionné par défaut, les administrateurs de cluster doivent planifier et gérer ce provisionnement. 
 
Voici un schéma décrivant la façon dont le stockage provisionné statiquement est consommé dans Kubernetes : 

![Provisionnement statique via PersistentVolumes](./media/azure-stack-edge-gpu-kubernetes-storage/static-provisioning-persistent-volumes-1.png)

Les étapes suivantes se produisent : 

1. **Provisionnement du stockage** : l’administrateur de cluster provisionne le stockage. Dans cet exemple, l’administrateur de cluster crée un ou plusieurs partages SMB, qui créent automatiquement des objets de volume persistant dans le cluster Kubernetes correspondant à ces partages. 

1. **Revendication du stockage** : vous soumettez un déploiement PVC qui demande le stockage. Cette revendication du stockage est l’élément PersistentVolumeClaim (PVC). Si la taille et le mode d’accès du PV correspondent à ceux du PVC, le PVC est lié au PV. Le PVC et le PV sont mappés un-à-un.

1. **Montage du PVC sur le conteneur** : dès lors que le PVC est lié au PV, vous pouvez monter ce PVC sur un chemin dans votre conteneur. Lorsque la logique d’application dans le conteneur lit depuis ce chemin, ou écrit dans celui-ci, les données sont écrites dans le stockage SMB.
 

## <a name="dynamic-provisioning"></a>Approvisionnement dynamique

Voici un schéma décrivant la façon dont le stockage provisionné statiquement est consommé dans Kubernetes : 

![Provisionnement dynamique via StorageClasses](./media/azure-stack-edge-gpu-kubernetes-storage/dynamic-provisioning-storage-classes-1.png)

Les étapes suivantes se produisent : 

1. **Définition de la classe de stockage** : l’administrateur de cluster définit une classe de stockage en fonction de l’environnement d’exploitation de votre cluster Kubernetes. L’administrateur de cluster déploie également un approvisionneur, qui n’est autre qu’une application ou un pod déployé sur le cluster Kubernetes. L’approvisionneur dispose de tous les détails permettant de provisionner les partages de manière dynamique.  

1. **Revendication du stockage** : vous soumettez une application qui doit réclamer le stockage. Dès lors qu’un PVC est créé avec cette référence de classe de stockage, l’approvisionneur est appelé. 

1. **Provisionner le stockage de façon dynamique** : L’approvisionneur crée dynamiquement le partage associé au stockage sur disque local. Une fois la création du partage effectuée, il crée également un objet PV correspondant automatiquement à ce partage.

1. **Montage du PVC sur le conteneur** : lorsque le PVC est lié au PV, vous pouvez monter le PVC sur le conteneur, dans un chemin, de la même façon que pour le provisionnement statique, et lire à partir du partage ou écrire dans celui-ci.


## <a name="storage-provisioning-on-azure-stack-edge-pro"></a>Provisionnement du stockage sur Azure Stack Edge Pro

Sur l’appareil Azure Stack Edge Pro, les `PersistentVolumes` provisionnés de manière statique sont créés à l’aide des fonctionnalités de stockage de l’appareil. Lorsque vous provisionnez un partage et que l’option **Utiliser le partage avec Edge Computing** est activée, cette action crée automatiquement une ressource PV dans le cluster Kubernetes.

![Création d’un partage local dans le portail Azure pour le provisionnement statique](./media/azure-stack-edge-gpu-kubernetes-storage/static-provisioning-azure-portal-2.png)

Pour utiliser la hiérarchisation cloud, vous pouvez créer un partage cloud Edge avec l’option Utiliser le partage avec Edge Computing activée. Un PV est de nouveau créé automatiquement pour ce partage. Toutes les données d’application que vous écrivez dans le partage Edge sont hiérarchisées dans le cloud. 

![Création d’un partage cloud dans le portail Azure pour le provisionnement statique](./media/azure-stack-edge-gpu-kubernetes-storage/static-provisioning-azure-portal-1.png)

Vous pouvez créer des partages SMB et NFS pour provisionner de manière statique des PV sur un appareil Azure Stack Edge Pro. Une fois le PV provisionné, vous devez soumettre un PVC pour revendiquer ce stockage. Voici un exemple de déploiement PVC `yaml` qui réclame le stockage et utilise les partages que vous avez provisionnés.


```yml
kind: PersistentVolumeClaim 
apiVersion: v1 
metadata: 
  name: pvc-smb-flexvol 
spec: 
  accessModes: 
  - ReadWriteMany 
  resources: 
    requests: 
      storage: 10Gi 
  volumeName: <nfs-or-smb-share-name-here> 
  storageClassName: ""
```

Pour plus d’informations, consultez [Déployer une application avec état par le biais du provisionnement statique sur votre appareil Azure Stack Edge Pro via kubectl](azure-stack-edge-gpu-deploy-stateful-application-static-provision-kubernetes.md).

Pour accéder au même stockage approvisionné statistiquement, les options de montage de volume correspondantes des liaisons de stockage pour IoT sont les suivantes. `/home/input` correspond au chemin permettant d’accéder au volume dans le conteneur.

```
{
"HostConfig": {
"Mounts": [
{
"Target": "/home/input",
"Source": "<nfs-or-smb-share-name-here>",
"Type": "volume"
},
{
"Target": "/home/output",
"Source": "<nfs-or-smb-share-name-here>",
"Type": "volume"
}]
}
}
```

Azure Stack Edge Pro compte également une classe `StorageClass` intégrée, appelée `ase-node-local`, qui utilise un stockage sur disque de données attaché au nœud Kubernetes. Cette classe `StorageClass` prend en charge le provisionnement dynamique. Lorsque vous créez une référence `StorageClass` dans les applications pod, un PV est automatiquement créé pour vous. Pour plus d’informations, consultez l’article [Tableau de bord Kubernetes](azure-stack-edge-gpu-monitor-kubernetes-dashboard.md) pour interroger `ase-node-local StorageClass`.

![Classe de stockage intégrée dans le tableau de bord Kubernetes](./media/azure-stack-edge-gpu-kubernetes-storage/dynamic-provisioning-builtin-storage-class-1.png)

Pour plus d’informations, consultez [Déployer une application avec état par le biais du provisionnement dynamique sur votre appareil Azure Stack Edge Pro via kubectl](azure-stack-edge-gpu-deploy-stateful-application-dynamic-provision-kubernetes.md).

## <a name="choose-storage-type"></a>Choisir un type de stockage

Vous devrez peut-être choisir votre type de stockage en fonction de la charge de travail que vous déployez. 

- Si vous souhaitez un mode d’accès `ReadWriteMany` pour vos `PersistentVolumes`, dans lesquels les volumes sont montés en lecture-écriture par le déploiement de nombreux nœuds, utilisez le provisionnement statique pour les partages SMB/NFS.

- Si les applications que vous déployez présentent une exigence de conformité POSIX, par exemple des applications telles que MongoDB, PostgreSQL, MySQL ou Prometheus, utilisez la StorageClass intégrée. Les modes d’accès sont `ReadWriteOnce`, sinon le volume est monté en lecture-écriture par un nœud unique. 


Pour plus d’informations sur les modes d’accès, consultez [Mode d’accès des volumes Kubernetes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes).


## <a name="next-steps"></a>Étapes suivantes

Pour comprendre comment provisionner un `PersistentVolume` de manière statique, consultez :

- [Déployer une application avec état par le biais du provisionnement statique sur votre appareil Azure Stack Edge Pro via kubectl](azure-stack-edge-gpu-deploy-stateful-application-static-provision-kubernetes.md).

Pour savoir comment provisionner dynamiquement `StorageClass`, consultez :

- [Déployer une application avec état via un approvisionnement dynamique sur votre appareil Azure Stack Edge Pro via kubectl](azure-stack-edge-gpu-deploy-stateful-application-dynamic-provision-kubernetes.md).
