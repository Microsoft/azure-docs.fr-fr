---
title: Utiliser Azure HPC Cache et Azure NetApp Files
description: Comment utiliser Azure HPC Cache pour améliorer l’accès aux données stockées avec Azure NetApp Files
author: ekpgh
ms.service: hpc-cache
ms.topic: how-to
ms.date: 10/30/2019
ms.author: v-erkel
ms.openlocfilehash: e955ddc14bb2b0a7abc0dc815c6955247568876b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86497010"
---
# <a name="use-azure-hpc-cache-with-azure-netapp-files"></a>Utiliser Azure HPC Cache avec Azure NetApp Files

Vous pouvez utiliser [Azure NetApp Files](https://azure.microsoft.com/services/netapp/) en tant que cible de stockage pour votre Azure HPC Cache. Cet article explique comment les deux services peuvent fonctionner ensemble et donne des conseils pour les configurer.

Azure NetApp Files associe leur système d’exploitation ONTAP à l’évolutivité et à la vitesse de Microsoft Azure. Cette combinaison permet aux utilisateurs de passer les workflows établis dans le cloud sans réécrire le code.

L’ajout d’un composant Azure HPC Cache peut améliorer l’accès aux fichiers en présentant plusieurs volumes Azure NetApp Files dans un espace de noms agrégé. Il peut fournir une mise en cache de périphérie pour les volumes situés dans une région de service différente. Cela peut également améliorer les performances à la demande pour les volumes qui ont été créés à des niveaux de service de niveau inférieur pour réduire les coûts.

## <a name="overview"></a>Vue d’ensemble

Pour utiliser un système Azure NetApp Files comme stockage principal avec Azure HPC Cache, suivez ce processus.

1. Créez le système et les volumes Azure NetApp Files conformément aux instructions de [Planifier votre système, ci-dessous](#plan-your-azure-netapp-files-system).
1. Créez le cache Azure HPC Cache dans la région où vous avez besoin d’un accès aux fichiers. (Suivez les instructions de [Créer un cache Azure HPC Cache](hpc-cache-create.md).)
1. [Définissez des cibles de stockage](#create-storage-targets-in-the-cache) dans le cache qui pointent vers les volumes Azure NetApp Files. Créez une cible de stockage de cache pour chaque adresse IP unique utilisée pour accéder aux volumes.
1. Faites monter le [cache Azure HPC Cache](#mount-storage-targets) aux clients au lieu de monter directement les volumes Azure NetApp Files.

## <a name="plan-your-azure-netapp-files-system"></a>Planifier votre système Azure NetApp Files

Lorsque vous planifiez votre système Azure NetApp Files, faites attention aux éléments de cette section pour vous assurer que vous pouvez l’intégrer facilement à Azure HPC Cache.

Lisez également la [Documentation Azure NetApp Files](../azure-netapp-files/index.yml) avant de créer des volumes à utiliser avec Azure HPC Cache.

### <a name="nfs-client-access-only"></a>Accès client NFS uniquement

Azure HPC Cache prend actuellement en charge l’accès NFS uniquement. Il ne peut pas être utilisé avec les volumes de bits de la liste de contrôle d’accès SMB ou du mode POSIX.

### <a name="exclusive-subnet-for-azure-netapp-files"></a>Sous-réseau exclusif pour Azure NetApp Files

Azure NetApp Files utilise un sous-réseau délégué unique pour ses volumes. Aucune autre ressource ne peut utiliser ce sous-réseau. En outre, un seul sous-réseau dans un réseau virtuel peut être utilisé pour Azure NetApp Files. Pour plus d’informations, consultez [Consignes pour planifier un réseau Azure NetApp Files](../azure-netapp-files/azure-netapp-files-network-topologies.md).

### <a name="delegated-subnet-size"></a>Taille du sous-réseau délégué

Utilisez la taille minimale du sous-réseau délégué lors de la création d’un système Azure NetApp Files pour une utilisation avec Azure HPC Cache.

La taille minimale, spécifiée avec le masque réseau /28, fournit 16 adresses IP. Dans la pratique, Azure NetApp Files utilise uniquement trois des adresses IP disponibles pour l’accès au volume. Cela signifie que vous n’avez qu’à créer trois cibles de stockage dans votre cache Azure HPC Cache pour couvrir tous les volumes.

Si le sous-réseau délégué est trop grand, il est possible que les volumes Azure NetApp Files utilisent davantage d’adresses IP que ce que peut gérer une seule instance de cache Azure HPC Cache. Un seul cache peut avoir au maximum 10 cibles de stockage.

L’exemple de démarrage rapide dans la Documentation Azure NetApp Files utilise 10.7.0.0/16 pour le sous-réseau délégué, ce qui donne un sous-réseau trop grand.

### <a name="capacity-pool-service-level"></a>Niveau de service du pool de capacité

Lorsque vous choisissez le niveau de service pour votre pool de capacité, pensez à votre flux de travail. Si vous écrivez fréquemment des données sur le volume Azure NetApp Files, les performances du cache peuvent être restreintes si le temps d’écriture différée est lent. Choisissez un niveau de service élevé pour les volumes qui auront des écritures fréquentes.

Les volumes avec des niveaux de service faibles peuvent également indiquer un décalage au début d’une tâche, tandis que le cache prérenseigne le contenu. Une fois que le cache est opérationnel et qu’il fonctionne avec un bon jeu de fichiers, le délai doit devenir invisible.

Il est important de planifier le niveau de service du pool de capacité à l’avance, car il ne peut pas être modifié après la création. Un nouveau volume doit être créé dans un pool de capacité différent et les données doivent être copiées.

Notez que vous pouvez modifier le quota de stockage d’un volume et la taille du pool de capacité sans en interrompre l’accès.

## <a name="create-storage-targets-in-the-cache"></a>Créer des cibles de stockage dans le cache

Une fois votre système Azure NetApp Files configuré et le cache Azure HPC Cachecréé, définissez des cibles de stockage dans le cache qui pointent vers les volumes du système de fichiers.

Créez une cible de stockage pour chaque adresse IP utilisée par vos volumes Azure NetApp Files. L’adresse IP est indiquée dans la page des instructions de montage du volume.

Si plusieurs volumes partagent la même adresse IP, vous pouvez utiliser une cible de stockage pour chacun d’eux.  

Suivez les [instructions de montage dans la Documentation Azure NetApp Files](../azure-netapp-files/azure-netapp-files-mount-unmount-volumes-for-virtual-machines.md) pour rechercher les adresses IP à utiliser.

Vous pouvez également trouver des adresses IP avec l’interface de ligne de commande Azure :

```azurecli
az netappfiles volume list -g ${RESOURCE_GROUP} --account-name ${ANF_ACCOUNT} --pool-name ${POOL} --query "[].mountTargets[].ipAddress" | grep -Ee '[0-9]+[.][0-9]+[.][0-9]+[.][0-9]+' | tr -d '"' | tr -d , | sort | uniq
```

Les noms d’exportation sur le système Azure NetApp Files ont un composant de chemin d’accès unique. N’essayez pas de créer une cible de stockage pour l’exportation racine ``/`` dans Azure NetApp Files, car cette exportation ne fournit pas d’accès aux fichiers.

Il n’existe aucune restriction spéciale sur les chemins d’accès de l’espace de noms virtuel pour ces cibles de stockage.

## <a name="mount-storage-targets"></a>Monter des cibles de stockage

Les ordinateurs clients doivent monter le cache au lieu de monter directement les volumes Azure NetApp Files. Suivez les instructions dans [Monter le cache Azure HPC Cache](hpc-cache-mount.md).

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur la configuration et l’utilisation de [Azure NetApp Files](../azure-netapp-files/index.yml)
* Pour plus d’informations sur la planification et la configuration de votre système Azure HPC Cache pour l’utilisation d’Azure NetApp Files, [contactez le support](hpc-cache-support-ticket.md).
