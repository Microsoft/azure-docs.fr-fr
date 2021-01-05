---
title: Tailles des machines virtuelles Azure – HPC | Microsoft Docs
description: Répertorie les différentes tailles disponibles pour les machines virtuelles de calcul haute performance dans Azure. Répertorie des informations sur le nombre de processeurs virtuels, de disques de données et de cartes réseau, ainsi que sur le débit de stockage et la bande passante réseau pour les tailles disponibles dans cette série.
author: vermagit
ms.service: virtual-machines
ms.subservice: sizes
ms.topic: conceptual
ms.workload: infrastructure-services
ms.date: 12/09/2020
ms.author: amverma
ms.reviewer: jushiman
ms.openlocfilehash: 5559e491be15f8f848329405736fc977267c70cc
ms.sourcegitcommit: dea56e0dd919ad4250dde03c11d5406530c21c28
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96938594"
---
# <a name="high-performance-computing-vm-sizes"></a>Tailles des machines virtuelles de calcul haute performance (HPC)

Les machines virtuelles Azure de la série H sont conçues pour offrir des performances, une extensibilité et une rentabilité exceptionnelles adaptées à un vaste éventail de charges de travail HPC réelles.

Les machines virtuelles de la [série HBv2](hbv2-series.md) sont optimisées pour des applications tributaires de la bande passante mémoire, par exemple la dynamique des fluides, l’analyse par éléments finis et la simulation de réservoir. Les machines virtuelles HBv2 disposent de 120 cœurs de processeur AMD EPYC 7742, de 4 Go de RAM par cœur de processeur, et d’aucun multithreading simultané. Chaque machine virtuelle HBv2 fournit jusqu’à 340 Go/s de bande passante de mémoire et jusqu’à 4 téraflops de calcul FP64.

Les machines virtuelles de la série HBv2 disposent d’un bus InfiniBand Mellanox HDR à 200 Gb/s, tandis que les machines virtuelles des séries HB et HC disposent d’un bus InfiniBand Mellanox EDR à 100 Gb/s. Chacun de ces types de machine virtuelle est connecté dans une arborescence FAT non bloquante pour des performances RDMA optimisées et cohérentes. Les machines virtuelles HBv2 prennent en charge le routage adaptatif et le transport connecté dynamique (DCT, en plus des transports RC et UD standards). Ces fonctionnalités améliorent les performances, l’extensibilité et la cohérence des applications, et leur utilisation est fortement recommandée.

Les machines virtuelles de la [série HB](hb-series.md) sont optimisées pour des applications tributaires de la bande passante mémoire, par exemple la dynamique des fluides, l’analyse explicite par éléments finis et la modélisation du climat. Les machines virtuelles HB disposent de 60 cœurs de processeur AMD EPYC 7551, de 4 Go de RAM par cœur de processeur, et d’aucun hyperthreading. La plateforme AMD EPYC fournit plus de 260 Go/s de bande passante mémoire.

Les machines virtuelles de la [série HC](hc-series.md) sont optimisées pour les applications tributaires d’un calcul dense, telles que l’analyse implicite par éléments finis, la dynamique moléculaire et la chimie numérique. Les machines virtuelles HC disposent de 44 cœurs de processeur Intel Xeon Platinum 8168, de 8 Go de RAM par cœur de processeur, et d’aucun hyperthreading. La plateforme Intel Xeon Platinum prend en charge le riche écosystème d’outils logiciels d’Intel, comme Intel Math Kernel Library.

Les machines virtuelles de la [série H](h-series.md) sont optimisées pour les applications tributaires de fréquences de processeur élevées ou ayant des besoins en mémoire importants par cœur. Les machines virtuelles de la série H disposent de 8 ou 16 cœurs de processeur Intel Xeon E5 2667 v3, de 7 ou 14 Go de RAM par cœur de processeur, et d’aucun hyperthreading. Les machines virtuelles de la série H disposent d’un Infiniband FDR Mellanox de 56 Go/s dans une configuration de gros arbres non bloquant pour des performances RDMA homogènes. Les machines virtuelles de la série H prennent en charge Intel MPI 5.x et MS-MPI.

> [!NOTE]
> Toutes les machines virtuelles de série HBv2, HB et HC ont un accès exclusif aux serveurs physiques. Il n’y a qu’une seule machine virtuelle par serveur physique et aucune multilocation partagée avec d’autres machines virtuelles pour ces tailles de machine virtuelle.

> [!NOTE]
> La mise hors service des [machines virtuelles A8-A11](./sizes-previous-gen.md#a-series---compute-intensive-instances) est planifiée pour le mois de mars 2021. Pour plus d’informations, consultez le [Guide de migration HPC](https://azure.microsoft.com/resources/hpc-migration-guide/).

## <a name="rdma-capable-instances"></a>Instances prenant en charge RDMA

La plupart des tailles de machines virtuelles HPC (HBv2, HB, HC, H16r, H16mr, A8 et A9) offrent une interface réseau pour la connectivité par accès direct à la mémoire à distance (RDMA). Les tailles de la [série N](./nc-series.md) désignées par « r » (ND40rs_v2, ND24rs, NC24rs_v3, NC24rs_v2 et NC24r) sont également compatibles RDMA. Cette interface s’ajoute à l’interface réseau Ethernet Azure standard disponible dans d’autres tailles de machine virtuelle.

Cette interface permet aux instances compatibles RDMA de communiquer sur un réseau InfiniBand (IB), opérant à des vitesses HDR pour les machines virtuelles HBv2, EDR pour les machines virtuelles HB, HC et NDv2, FDR pour les machines virtuelles H16r et H16mr et autres machines virtuelles de la série N compatibles RDMA, et QDR pour les machines virtuelles A8 et A9. Ces fonctionnalités RDMA peuvent améliorer l’extensibilité et les performances de certaines applications MPI (Message Passing Interface).

> [!NOTE]
> Dans Azure HPC, il existe deux classes de machines virtuelles, selon qu’elles sont compatibles SR-IOV pour InfiniBand. À l’heure actuelle, presque toutes les machines virtuelles les plus récentes compatibles avec RDMA ou InfiniBand sur Azure sont compatibles avec SR-IOV, à l’exception de H16r, H16mr, NC24r, A8, A9.
> RDMA est activé uniquement sur le réseau InfiniBand (IB) et est pris en charge pour toutes les machines virtuelles compatibles RDMA.
> IP over IB est uniquement pris en charge sur les machines virtuelles compatibles SR-IOV.
> RDMA n’est pas activé sur le réseau Ethernet.

- **Système d’exploitation** : Linux est très bien pris en charge pour les machines virtuelles HPC ; des distributions comme CentOS, RHEL, Ubuntu et SUSE sont couramment utilisées. En ce qui concerne la prise en charge Windows, Windows Server 2016 et les versions ultérieures sont pris en charge sur toutes les machines virtuelles de la série HPC. Windows Server 2012 R2 et Windows Server 2012 sont également pris en charge sur les machines virtuelles non compatibles SR-IOV (H16r, H16mr, A8 et A9). Notez que [Windows Server 2012 R2 n’est pas pris en charge sur les HBv2 et autres machines virtuelles ayant plus de 64 cœurs (virtuels ou physiques)](/windows-server/virtualization/hyper-v/supported-windows-guest-operating-systems-for-hyper-v-on-windows). Pour obtenir la liste des images de machines virtuelles prises en charge sur la place de marché et savoir comment les configurer de manière appropriée, consultez [Images de machine virtuelle](./workloads/hpc/configure.md).

- **InfiniBand et pilotes** : sur les machines virtuelles prenant en charge InfiniBand, certains pilotes sont requis pour activer la fonctionnalité RDMA. Sur Linux, les machines virtuelles SR-IOV et non-SR-IOV, les images de machines virtuelles CentOS-HPC de la Place de marché sont préconfigurées avec les pilotes appropriés. Les images de machine virtuelle Ubuntu peuvent être configurées avec les bons pilotes suivant ces [instructions](https://techcommunity.microsoft.com/t5/azure-compute/configuring-infiniband-for-ubuntu-hpc-and-gpu-vms/ba-p/1221351). Consultez [Configurer et optimiser des machines virtuelles pour le système d’exploitation Linux](./workloads/hpc/configure.md) pour plus d’informations sur les images de système d’exploitation Linux prêtes à l’emploi.

   Sur Linux, pour les machines virtuelles des séries H et N prenant en charge SR-IOV, [l’extension de machine virtuelle InfiniBandDriverLinux](./extensions/hpc-compute-infiniband-linux.md) peut être utilisée pour installer les pilotes OFED Mellanox et activer InfiniBand. Pour savoir comment activer InfiniBand sur une machine virtuelle compatible RDMA, consultez [Charges de travail HPC](./workloads/hpc/enable-infiniband.md).

   Sur Windows, [l’extension de machine virtuelle InfiniBandDriverWindows](./extensions/hpc-compute-infiniband-windows.md) installe des pilotes Windows Network Direct (sur des machines virtuelles non compatibles SR-IOV) ou des pilotes Mellanox OFED (sur des machines virtuelles compatibles SR-IOV) pour la connectivité RDMA. Dans certains déploiements des instances A8 et A9, l’extension HpcVmDrivers est ajoutée automatiquement. Notez que l’extension de machine virtuelle HpcVmDrivers est déconseillée ; elle ne sera pas mise à jour.

   Pour ajouter l’extension de machine virtuelle sur une machine virtuelle, vous pouvez utiliser les cmdlets [Azure PowerShell](/powershell/azure/). Pour plus d’informations, consultez [Extensions et fonctionnalités de la machine virtuelle](./extensions/overview.md). Vous pouvez également utiliser les extensions pour les machines virtuelles déployées dans le [modèle de déploiement classique](/previous-versions/azure/virtual-machines/windows/classic/agents-and-extensions-classic).

- **MPI** : les tailles des machines virtuelles SR-IOV activées sur Azure permettent d’utiliser presque tous les qualificateurs de MPI avec Mellanox OFED. Sur les machines virtuelles non compatibles SR-IOV, les implémentations MPI prises en charge utilisent l’interface Microsoft Network Direct (ND) pour la communication entre les machines virtuelles. Par conséquent, seules les versions Microsoft MPI (MS-MPI) 2012 R2 ou ultérieures et Intel MPI 5.x sont supportées. Les versions ultérieures (2017, 2018) de la bibliothèque runtime MPI Intel peuvent ne pas être compatibles avec les pilotes Azure RDMA. Pour plus d’informations sur la configuration de l’interface de passage de messages sur des machines virtuelles HPC sur Azure, consultez [Configurer MPI pour HPC](./workloads/hpc/setup-mpi.md).

- **Espace d’adressage réseau RDMA** : le réseau RDMA dans Azure réserve l’espace d’adressage 172.16.0.0/16. Si vous exécutez des applications MPI sur des instances déployées dans un réseau virtuel Azure, assurez-vous que l’espace d’adressage du réseau virtuel ne chevauche pas le réseau RDMA.

## <a name="cluster-configuration-options"></a>Options de configuration de cluster

Azure fournit plusieurs options pour créer des clusters de machines virtuelles HPC Windows pouvant communiquer via le réseau RDMA, notamment : 

- **Machines virtuelles** : déployez les machines virtuelles HPC compatibles RDMA dans le même groupe de machines virtuelles identiques ou groupe à haute disponibilité (quand vous utilisez le modèle de déploiement Azure Resource Manager). Si vous utilisez le modèle de déploiement classique, déployez les machines virtuelles dans le même service cloud.

- **Groupes de machines virtuelles identiques** : dans un groupe de machines virtuelles identiques, veillez à limiter le déploiement à un seul groupe de placements pour la communication InfiniBand au sein du groupe identique. Par exemple, dans un modèle Resource Manager, définissez la propriété `singlePlacementGroup` avec la valeur `true`. Notez que la taille d’un groupe identique qui peut être lancé avec la propriété `singlePlacementGroup` définie sur `true` est limitée par défaut à 100 machines virtuelles. Si vos besoins de mise à l’échelle des travaux HPC sont supérieurs à 100 machines virtuelles dans un seul locataire, vous pouvez demander une augmentation : [ouvrez une demande de support client en ligne](../azure-portal/supportability/how-to-create-azure-support-request.md) gratuitement. La limite du nombre de machines virtuelles dans un seul groupe identique peut être portée à 300. Notez que quand vous déployez des machines virtuelles à l’aide de groupes à haute disponibilité, la limite maximale est de 200 machines virtuelles par groupe à haute disponibilité.

- **MPI entre les machines virtuelles** : si RDMA (par exemple en utilisant la communication MPI) est nécessaire entre les machines virtuelles, assurez-vous que les machines virtuelles figurent dans le même groupe de machines virtuelles identiques ou groupe à haute disponibilité.

- **Azure CycleCloud** : Créez un cluster HPC dans [Azure CycleCloud](/azure/cyclecloud/) pour exécuter des travaux MPI.

- **Azure Batch** : Créez un pool [Azure Batch](../batch/index.yml) pour exécuter des charges de travail MPI. Pour utiliser des instances nécessitant beaucoup de ressources système lors de l’exécution d’applications MPI avec Azure Batch, consultez [Utiliser les tâches multi-instances pour exécuter des applications MPI (Message Passing Interface) dans Azure Batch](../batch/batch-mpi.md).

- **Microsoft HPC Pack** - [HPC Pack](/powershell/high-performance-computing/overview) comprend un environnement d’exécution pour MS-MPI qui utilise le réseau RDMA Azure en cas de déploiement sur des machines virtuelles Linux compatibles RDMA. Pour des exemples de déploiement, voir [Configuration d’un cluster RDMA Linux avec HPC Pack pour exécuter des applications MPI](/powershell/high-performance-computing/hpcpack-linux-openfoam).

## <a name="deployment-considerations"></a>Points à prendre en considération pour le déploiement

- **Abonnement Azure** : pour déployer un plus grand nombre d’instances de calcul intensif, envisagez de souscrire un abonnement de paiement à l’utilisation ou d’autres options d’achat. Si vous utilisez un [compte gratuit Azure](https://azure.microsoft.com/free/), vous pouvez seulement utiliser un nombre limité de cœurs de calcul Azure.

- **Tarification et disponibilité** : ces tailles de machines virtuelles sont proposées uniquement au niveau tarifaire Standard. Pour connaître la disponibilité dans les différentes régions Azure, voir [Disponibilité des produits par région](https://azure.microsoft.com/global-infrastructure/services/) .

- **Quota de cœurs** : vous devrez peut-être augmenter le quota de cœurs dans votre abonnement Azure à partir de la valeur par défaut. Votre abonnement peut également limiter le nombre de cœurs, que vous pouvez déployer dans certaines familles de taille de machine virtuelle, dont la série H. Pour demander une augmentation de quota, [ouvrez une demande de service clientèle en ligne](../azure-portal/supportability/how-to-create-azure-support-request.md) gratuitement. (Les limites par défaut peuvent varier en fonction de la catégorie de votre abonnement.)

  > [!NOTE]
  > Si vous avez des besoins de capacité à grande échelle, contactez le support Azure. Les quotas d’Azure sont des limites de crédit et non des garanties de capacité. Quel que soit votre quota, vous êtes facturé uniquement pour les cœurs que vous utilisez.
  
- **Réseau virtuel** : un [réseau virtuel](https://azure.microsoft.com/documentation/services/virtual-network/) Azure n’est pas requis pour utiliser les instances qui nécessitent beaucoup de ressources système. Cependant, pour bon nombre de scénarios de déploiement, vous avez besoin d’au moins un réseau virtuel Azure cloud ou d’une connexion de site à site si vous devez accéder à des ressources locales. Si nécessaire, créez un réseau virtuel avant de déployer les instances. L’ajout de machines virtuelles nécessitant beaucoup de ressources système à un réseau virtuel dans un groupe d’affinités n’est pas pris en charge.

- **Redimensionnement** : en raison de leur matériel spécialisé, seules les instances nécessitant beaucoup de ressources système qui appartiennent à la même famille de taille (série H ou N) peuvent être redimensionnées. Par exemple, vous pouvez redimensionner une machine virtuelle de la série H uniquement d’une seule taille en une autre de cette même série. Des considérations supplémentaires concernant la prise en charge des pilotes InfiniBand et les disques NVMe peuvent se révéler nécessaires pour certaines machines virtuelles.


## <a name="other-sizes"></a>Autres tailles

- [Usage général](sizes-general.md)
- [Optimisé pour le calcul](sizes-compute.md)
- [Mémoire optimisée](sizes-memory.md)
- [Optimisé pour le stockage](sizes-storage.md)
- [Optimisé pour le GPU](sizes-gpu.md)
- [Générations précédentes](sizes-previous-gen.md)

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur [la configuration de vos machines virtuelles](./workloads/hpc/configure.md), [l’activation d’InfiniBand](./workloads/hpc/enable-infiniband.md), [la configuration de MPI](./workloads/hpc/setup-mpi.md) et l’optimisation des applications HPC pour Azure dans l’article relatif aux [charges de travail HPC](./workloads/hpc/overview.md).
- Découvrez des informations sur les dernières annonces et des exemples et des résultats HPC sur les [blogs de la communauté Azure Compute Tech](https://techcommunity.microsoft.com/t5/azure-compute/bg-p/AzureCompute).
- Pour une vision plus globale de l’architecture d’exécution des charges de travail HPC, consultez [Calcul haute performance (HPC) sur Azure](/azure/architecture/topics/high-performance-computing/).
