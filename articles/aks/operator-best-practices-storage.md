---
title: Meilleures pratiques relatives au stockage et aux sauvegardes
titleSuffix: Azure Kubernetes Service
description: Découvrir les meilleures pratiques relatives au stockage, au chiffrement des données et aux sauvegardes dans Azure Kubernetes Service (AKS) pour les opérateurs de clusters
services: container-service
ms.topic: conceptual
ms.date: 5/6/2019
ms.openlocfilehash: 722fe393ad7637be20360463a4c3b6234224a036
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88653968"
---
# <a name="best-practices-for-storage-and-backups-in-azure-kubernetes-service-aks"></a>Meilleures pratiques relatives au stockage et aux sauvegardes dans Azure Kubernetes Service (AKS)

Lorsque vous créez et gérez des clusters dans Azure Kubernetes Service (AKS), il est souvent nécessaire de stocker vos applications. Il est important de comprendre les besoins en matière de performances des pods et les méthodes pour y accéder afin de fournir le stockage approprié aux applications. La taille du nœud AKS peut avoir une incidence sur le choix du type de stockage. Vous devez également planifier des méthodes pour sauvegarder et tester le processus de restauration pour le stockage attaché.

Cet article traite des meilleures pratiques de stockage pour les opérateurs de clusters. Dans cet article, vous apprenez :

> [!div class="checklist"]
> * Les types de stockage disponibles.
> * Comment définir correctement la taille des nœuds AKS pour des performances de stockage optimales.
> * Les différences entre l’approvisionnement dynamique et statique des volumes.
> * Les méthodes de sauvegarde et de sécurisation de vos volumes de données.

## <a name="choose-the-appropriate-storage-type"></a>Le choix du type de stockage adapté.

**Conseils pour une utilisation optimale** : identifiez les besoins de votre application pour choisir le stockage approprié. Utilisez un stockage sur disque SSD haute performance pour les charges de travail de production. Optez pour un stockage basé sur un réseau si vous avez besoin de multiples connexions simultanées.

Les applications nécessitent souvent différents types et vitesses de stockage. Vos applications requièrent-elles un stockage qui se connecte à des pods individuels ou un stockage partagé entre plusieurs pods ? Le stockage est-il destiné à un accès aux données en lecture seule ou à l’écriture de volumes importants de données structurées ? Ces besoins de stockage déterminent le type de stockage le plus approprié.

Le tableau suivant présente les types de stockage disponibles et leurs fonctionnalités :

| Cas d’utilisation | Plug-in de volume | Lecture/écriture unique | Nombreuses lectures seules | Nombreuses lectures/écritures | Prise en charge de conteneur Windows Server |
|----------|---------------|-----------------|----------------|-----------------|--------------------|
| Configuration partagée       | Azure Files   | Oui | Oui | Oui | Oui |
| Données d’application structurées        | Disques Azure   | Oui | Non  | Non  | Oui |
| Données non structurées, opérations sur le système de fichiers | [BlobFuse][blobfuse] | Oui | Oui | Oui | Non |

Les deux principaux types de stockage fournis pour les volumes dans AKS reposent sur des disques Azure ou Azure Files. Pour améliorer la sécurité, ces deux types de stockage utilisent Azure Storage Service Encryption (SSE) par défaut pour chiffrer les données au repos. Les disques ne peuvent actuellement pas être chiffrés à l’aide d’Azure Disk Encryption au niveau des nœuds AKS.

Azure Files et les disques Azure sont disponibles pour les niveaux de performance Standard et Premium :

- Les disques *Premium* sont assortis de disques SSD hautes performances. Les disques Premium sont recommandés pour toutes les charges de travail de production.
- Les disques *Standard* sont assortis de disques rotatifs ordinaires (HDD) et sont adaptés en cas de données d’archivage ou rarement utilisées.

Identifiez les besoins de performances de votre application et les modèles d’accès pour choisir le niveau de stockage approprié. Pour plus d’informations sur les tailles et les niveaux de performance des disques managés, consultez [Vue d’ensemble d’Azure Disques managés][managed-disks].

### <a name="create-and-use-storage-classes-to-define-application-needs"></a>Créer et utiliser des classes de stockage pour définir les besoins des applications

Le type de stockage que vous utilisez est défini à l’aide de *classes de stockage* Kubernetes. La classe de stockage est ensuite référencée dans la spécification du pod ou du déploiement. Ces définitions fonctionnent ensemble pour créer le stockage approprié et le connecter aux pods. Pour plus d’informations, consultez [Classes de stockage dans AKS][aks-concepts-storage-classes].

## <a name="size-the-nodes-for-storage-needs"></a>Dimensionner les nœuds en fonction des besoins de stockage

**Conseils pour une utilisation optimale** :chaque taille de nœud prend en charge un nombre maximal de disques. Différentes tailles de nœud fournissent également des quantités différentes de stockage local et de bande passante de réseau. Planifiez les exigences de votre application pour déployer la taille appropriée de nœuds.

Les nœuds AKS sont exécutés en tant que machines virtuelles Azure. Différents types et tailles de machine virtuelle sont disponibles. Chaque taille de machine virtuelle fournit une quantité différente de ressources fondamentales telles que l’UC et la mémoire. Chaque taille de machine virtuelle comporte un nombre maximal de disques pouvant y être attachés. Selon les tailles de machine virtuelle, les performances de stockage varient également en matière d’IOPS (opérations d’entrée/sortie par seconde) maximal des disques locaux et attachés.

Si vos applications nécessitent des disques Azure comme solution de stockage, planifiez et choisissez une taille de machine virtuelle de nœud appropriée. La quantité d’UC et de mémoire n’est pas le seul facteur à prendre en compte lorsque vous choisissez une taille de machine virtuelle. Les capacités de stockage sont également importantes. Par exemple, les tailles de machine virtuelle *Standard_B2ms* et *Standard_DS2_v2* incluent une quantité similaire de ressources d’UC et de mémoire. Leurs performances de stockage potentielles sont toutefois différentes, comme indiqué dans le tableau suivant :

| Taille et type de nœud | Processeurs virtuels | Mémoire (Gio) | Disques de données max. | IOPS de disque non mis en cache max. | Débit non mis en cache max. (Mbits/s) |
|--------------------|------|--------------|----------------|------------------------|--------------------------------|
| Standard_B2ms      | 2    | 8            | 4              | 1 920                  | 22,5                           |
| Standard_DS2_v2    | 2    | 7            | 8              | 6 400                  | 96                             |

Ici, la taille *Standard_DS2_v2* permet de doubler le nombre de disques attachés et de fournir trois à quatre fois la quantité d’IOPS et de débit de disque. Si vous prenez uniquement en compte les principales ressources de calcul et que vous comparez les coûts, vous serez tenté de choisir la taille de machine virtuelle *Standard_B2ms*, mais les performances de stockage risquent d’être médiocres et de présenter des limitations. Consultez votre équipe de développement d’application pour comprendre ses besoins de capacité de stockage et de performance. Choisissez la taille de machine virtuelle appropriée pour les nœuds AKS permettant de satisfaire ou de dépasser ses besoins de performance. Évaluez régulièrement les applications afin d’ajuster la taille de machine virtuelle en fonction des besoins.

Pour plus d’informations sur les tailles de machine virtuelle disponibles, consultez [Tailles des machines virtuelles Linux dans Azure][vm-sizes].

## <a name="dynamically-provision-volumes"></a>Approvisionner les volumes de manière dynamique

**Conseils pour une utilisation optimale** : afin de réduire les frais de gestion et de vous permettre d’ajuster votre utilisation, ne créez et n’affectez pas de volumes persistants de manière statique. Optez pour un approvisionnement dynamique. Dans vos classes de stockage, définissez la stratégie de demande de récupération appropriée pour réduire les coûts de stockage inutiles une fois les pods supprimés.

Lorsque vous devez lier un stockage aux pods, vous utilisez des volumes persistants. Ces volumes persistants peuvent être créés manuellement ou dynamiquement. La création manuelle de volumes persistants ajoute une surcharge de gestion et limite votre capacité d’ajustement. Utilisez un approvisionnement de volume persistant dynamique pour simplifier la gestion du stockage et permettre à vos applications de se développer et de s’ajuster en fonction des besoins.

![Revendications de volumes persistants dans un cluster AKS (Azure Kubernetes Service)](media/concepts-storage/persistent-volume-claims.png)

Une revendication de volume persistant vous permet de créer des espaces de stockage de manière dynamique en fonction des besoins. Les disques Azure sous-jacents sont créés en fonction des requêtes des pods. Dans la définition de pod, vous demandez un volume à créer et à rattacher à un chemin d’accès de montage spécifique.

Pour en savoir plus sur les concepts relatifs à la création et à l’utilisation dynamique des volumes, consultez [Revendications de volume persistant][aks-concepts-storage-pvcs].

Pour voir ces volumes en action, découvrez comment créer et utiliser un volume persistant de manière dynamique avec des [disques Azure][dynamic-disks] ou [Azure Files][dynamic-files].

Dans le cadre de vos définitions de classe de stockage, définissez la propriété *reclaimPolicy* de manière appropriée. Cette stratégie de récupération contrôle le comportement de la ressource de stockage Azure sous-jacente quand le pod est supprimé et que le volume persistant n’est peut-être plus nécessaire. La ressource de stockage sous-jacente peut être supprimée ou conservée en vue de son utilisation par un pod futur. La valeur reclaimPolicy peut être définie sur *Conserver* ou *Supprimer*. Identifiez les besoins de votre application et implémentez des contrôles réguliers pour le stockage conservé afin de réduire la quantité de stockage non utilisée et facturée.

Pour plus d’informations sur les options de classe de stockage, consultez les [stratégies de récupération de stockage][reclaim-policy].

## <a name="secure-and-back-up-your-data"></a>Sécuriser et sauvegarder vos données

**Conseils sur les meilleures pratiques** : sauvegardez vos données à l’aide d’un outil adapté à votre type de stockage, tel que Velero ou Sauvegarde Azure. Vérifiez l’intégrité et la sécurité de ces sauvegardes.

Lorsque vos applications stockent et exploitent des données conservées sur des disques ou dans des fichiers, vous devez effectuer des sauvegardes ou des captures instantanées régulières de ces données. Les disques Azure peuvent utiliser des technologies de capture instantanée intégrées. Vous aurez peut-être besoin de rechercher vos applications pour vider les écritures sur le disque avant d’effectuer l’opération de capture instantanée. [Velero][velero] peut sauvegarder des volumes persistants avec des ressources de cluster et des configurations supplémentaires. Si vous ne pouvez pas [supprimer l’état de vos applications][remove-state], sauvegardez les données à partir de volumes persistants et testez régulièrement les opérations de restauration pour vérifier l’intégrité des données et les processus nécessaires.

Identifiez les limitations des différentes approches en matière de sauvegarde de données et déterminez si vous devez suspendre vos données avant la capture instantanée. Les sauvegardes de données ne vous permettent pas nécessairement de restaurer votre environnement d’application de déploiement de cluster. Pour plus d’informations sur ces scénarios, consultez [Best practices for business continuity and disaster recovery in AKS][best-practices-multi-region] (Meilleures pratiques pour la continuité d’activité et la récupération d’urgence dans AKS).

## <a name="next-steps"></a>Étapes suivantes

Cet article se concentre sur les meilleures pratiques de stockage dans AKS. Pour plus d’informations sur les principes fondamentaux du stockage dans Kubernetes, consultez [Options de stockage pour les applications dans AKS (Azure Kubernetes Service)][aks-concepts-storage].

<!-- LINKS - External -->
[velero]: https://github.com/heptio/velero
[blobfuse]: https://github.com/Azure/azure-storage-fuse

<!-- LINKS - Internal -->
[aks-concepts-storage]: concepts-storage.md
[vm-sizes]: ../virtual-machines/sizes.md
[dynamic-disks]: azure-disks-dynamic-pv.md
[dynamic-files]: azure-files-dynamic-pv.md
[reclaim-policy]: concepts-storage.md#storage-classes
[aks-concepts-storage-pvcs]: concepts-storage.md#persistent-volume-claims
[aks-concepts-storage-classes]: concepts-storage.md#storage-classes
[managed-disks]: ../virtual-machines/managed-disks-overview.md
[best-practices-multi-region]: operator-best-practices-multi-region.md
[remove-state]: operator-best-practices-multi-region.md#remove-service-state-from-inside-containers
