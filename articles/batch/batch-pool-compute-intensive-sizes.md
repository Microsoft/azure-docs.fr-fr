---
title: Utiliser des machines virtuelles Azure nécessitant beaucoup de ressources système avec Batch
description: Découvrez comment tirer parti des tailles de machines virtuelles HPC et GPU dans des pools Azure Batch Apprenez-en davantage sur les dépendances du système d’exploitation et découvrez plusieurs exemples de scénarios.
ms.topic: how-to
ms.date: 12/17/2018
ms.openlocfilehash: 016da7669c9e6a6586a53d379f9665c9ea048b64
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86147330"
---
# <a name="use-rdma-or-gpu-instances-in-batch-pools"></a>Utiliser des instances RDMA ou GPU dans des pools Batch

Pour exécuter certains travaux Batch, vous pouvez tirer parti des tailles de machines virtuelles Azure conçues pour le calcul à grande échelle. Par exemple :

* Pour exécuter des [charges de travail MPI](batch-mpi.md) multi-instances, choisissez des tailles de série H ou autres qui ont une interface réseau pour l’accès direct à la mémoire à distance (RDMA). Ces tailles se connectent à un réseau InfiniBand pour la communication entre les nœuds, ce qui peut accélérer les applications MPI. 

* Pour les applications CUDA, choisissez des tailles de série N qui incluent des cartes d’unité de traitement graphique (GPU) NVIDIA Tesla.

Cet article fournit des conseils et des exemples pour utiliser certaines des tailles spécialisées d’Azure dans les pools Batch. Pour obtenir des informations générales et en savoir plus sur les caractéristiques techniques, consultez :

* Tailles de machines virtuelles de calcul haute performance ([Linux](../virtual-machines/sizes-hpc.md), [Windows](../virtual-machines/sizes-hpc.md)) 

* Tailles de machines virtuelles Linux compatibles GPU ([Linux](../virtual-machines/sizes-gpu.md), [Windows](../virtual-machines/sizes-gpu.md)) 

> [!NOTE]
> Certaines tailles de machines virtuelles peuvent ne pas être disponibles dans les régions où vous créez vos comptes Batch. Pour vérifier la disponibilité d’une taille, consultez [Disponibilité des produits par région](https://azure.microsoft.com/regions/services/) et [Choisir une taille de machine virtuelle pour un pool Batch](batch-pool-vm-sizes.md).

## <a name="dependencies"></a>Les dépendances

Les fonctionnalités RDMA ou GPU des tailles nécessitant beaucoup de ressources système dans Batch sont prises en charge uniquement sur certains systèmes d’exploitation. (La liste des systèmes d’exploitation pris en charge est un sous-ensemble de ceux pris en charge pour les machines virtuelles créées dans ces tailles.) Selon la façon dont vous créez votre pool Batch, vous devrez peut-être installer ou configurer des pilotes supplémentaires ou d’autres logiciels sur les nœuds. Les tableaux suivants listent ces dépendances. Pour plus de détails, consultez les articles associés. Pour connaître les options de configuration des pools Batch, voir plus loin dans cet article.

### <a name="linux-pools---virtual-machine-configuration"></a>Pools Linux - Configuration de machine virtuelle

| Taille | Fonctionnalité | Systèmes d’exploitation | Logiciels requis | Paramètres de pool |
| -------- | -------- | ----- |  -------- | ----- |
| [H16r, H16mr, A8, A9](../virtual-machines/sizes-hpc.md)<br/>[NC24r, NC24rs_v2, NC24rs_v3, ND24rs<sup>*</sup>](../virtual-machines/linux/n-series-driver-setup.md#rdma-network-connectivity) | RDMA | Ubuntu 16.04 LTS ou<br/>HPC basé sur CentOS<br/>(Place de marché Azure) | Intel MPI 5<br/><br/>Pilotes RDMA Linux | Activer la communication entre les nœuds, désactiver l’exécution simultanée des tâches |
| [Séries NC, NCv2, NCv3, NDv2](../virtual-machines/linux/n-series-driver-setup.md) | GPU NVIDIA Tesla (varie selon la série) | Ubuntu 16.04 LTS ou<br/>CentOS 7.3 ou 7.4<br/>(Place de marché Azure) | Pilotes NVIDIA CUDA ou CUDA Toolkit | N/A | 
| [Séries NV, NVv2](../virtual-machines/linux/n-series-driver-setup.md) | GPU NVIDIA Tesla M60 | Ubuntu 16.04 LTS ou<br/>CentOS 7.3<br/>(Place de marché Azure) | Pilotes NVIDIA GRID | N/A |

<sup>*</sup>Les tailles de série N compatibles RDMA incluent également des GPU NVIDIA Tesla.

### <a name="windows-pools---virtual-machine-configuration"></a>Pools Windows - Configuration de machine virtuelle

| Taille | Fonctionnalité | Systèmes d’exploitation | Logiciels requis | Paramètres de pool |
| -------- | ------ | -------- | -------- | ----- |
| [H16r, H16mr, A8, A9](../virtual-machines/sizes-hpc.md)<br/>[NC24r, NC24rs_v2, NC24rs_v3, ND24rs<sup>*</sup>](../virtual-machines/windows/n-series-driver-setup.md#rdma-network-connectivity) | RDMA | Windows Server 2016, 2012 R2, ou<br/>2012 (Place de Marché Azure) | Microsoft MPI 2012 R2 ou ultérieur, ou<br/> Intel MPI 5<br/><br/>Pilotes RDMA Windows | Activer la communication entre les nœuds, désactiver l’exécution simultanée des tâches |
| [Séries NC, NCv2, NCv3, ND, NDv2](../virtual-machines/windows/n-series-driver-setup.md) | GPU NVIDIA Tesla (varie selon la série) | Windows Server 2016 ou <br/>2012 R2 (Place de Marché Azure) | Pilotes NVIDIA CUDA ou CUDA Toolkit| N/A | 
| [Séries NV, NVv2](../virtual-machines/windows/n-series-driver-setup.md) | GPU NVIDIA Tesla M60 | Windows Server 2016 ou<br/>2012 R2 (Place de Marché Azure) | Pilotes NVIDIA GRID | N/A |

<sup>*</sup>Les tailles de série N compatibles RDMA incluent également des GPU NVIDIA Tesla.

### <a name="windows-pools---cloud-services-configuration"></a>Pools Windows - Configuration des services cloud

> [!NOTE]
> Les tailles de série N ne sont pas prises en charge dans les pools Batch avec la configuration de service cloud.
>

| Taille | Fonctionnalité | Systèmes d’exploitation | Logiciels requis | Paramètres de pool |
| -------- | ------- | -------- | -------- | ----- |
| [H16r, H16mr, A8, A9](../virtual-machines/sizes-hpc.md) | RDMA | Windows Server 2016, 2012 R2, 2012, ou<br/>2008 R2 (Famille de système d’exploitation invité) | Microsoft MPI 2012 R2 ou ultérieur, ou<br/>Intel MPI 5<br/><br/>Pilotes RDMA Windows | Activer la communication entre les nœuds,<br/> désactiver l’exécution simultanée des tâches |

## <a name="pool-configuration-options"></a>Options de configuration de pool

Pour configurer une taille de machine virtuelle spécialisée pour votre pool Batch, plusieurs options s’offrent à vous pour installer les logiciels ou pilotes nécessaires :

* Pour les pools inclus dans la configuration de machine virtuelle, choisissez une image de machine virtuelle [Place de marché Azure](https://azuremarketplace.microsoft.com/marketplace/) préconfigurée dotée de pilotes et logiciels préinstallés. Exemples : 

  * [HPC basé sur CentOS 7.4](https://azuremarketplace.microsoft.com/marketplace/apps/openlogic.centos-hpc?tab=Overview) (inclut des pilotes RDMA et Intel MPI 5.1)

  * [Data Science Virtual Machine](../machine-learning/data-science-virtual-machine/overview.md) pour Linux ou Windows (inclut des pilotes NVIDIA CUDA)

  * Images Linux pour les charges de travail de conteneur Batch qui incluent également des pilotes GPU et RDMA :

    * [CentOS (avec pilotes GPU et RDMA) pour les pools de conteneur Azure Batch](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-azure-batch.centos-container-rdma?tab=Overview)

    * [Ubuntu Server (avec pilotes GPU et RDMA) pour les pools de conteneur Azure Batch](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-azure-batch.ubuntu-server-container-rdma?tab=Overview)

* Créez une [image de machine virtuelle Windows ou Linux personnalisée](batch-sig-images.md) sur laquelle vous avez installé des pilotes, des logiciels ou d’autres paramètres nécessaires à la taille de machine virtuelle. 

* Créez un [package d’application](batch-application-packages.md) Batch à partir d’un pilote compressé ou d’un programme d’installation d’application, puis configurez Batch pour déployer le package sur les nœuds du pool et l’installer une fois au moment de la création de chaque nœud. Par exemple, si le package d’application est un programme d’installation, créez une ligne de commande de [tâche de démarrage](jobs-and-tasks.md#start-task) pour effectuer une installation de l’application en mode silencieux sur tous les nœuds du pool. Envisagez d’utiliser un package d’application et une tâche de démarrage de pool si votre charge de travail dépend d’une version de pilote particulière.

  > [!NOTE] 
  > La tâche de démarrage doit s’exécuter avec des autorisations élevées (d’administrateur) et doit attendre de réussir. Les tâches à exécution longue augmentent le temps de provisionnement d’un pool Batch.
  >

* [Batch Shipyard](https://github.com/Azure/batch-shipyard) configure automatiquement les pilotes GPU et RDMA pour fonctionner de façon fluide avec les charges de travail en conteneur sur Azure Batch. Batch Shipyard est entièrement piloté par les fichiers de configuration. Il existe de nombreux exemples de recettes de configurations disponibles qui activent les charges de travail GPU et RDMA, comme la [recette CNTK GPU](https://github.com/Azure/batch-shipyard/tree/master/recipes/CNTK-GPU-OpenMPI) qui préconfigure les pilotes GPU sur les machines virtuelles de série N et charge le logiciel Microsoft Cognitive Toolkit en tant qu’image Docker.


## <a name="example-nvidia-gpu-drivers-on-windows-nc-vm-pool"></a>Exemple : Pilotes NVIDIA GPU sur un pool de machines virtuelles de contrôleur de réseau Windows

Pour exécuter des applications CUDA sur un pool de nœuds de contrôleur de réseau Windows, vous devez installer des pilotes NVDIA GPU. Les exemples d’étapes suivants utilisent un package d’application pour installer les pilotes NVIDIA GPU. Vous pouvez choisir cette option si votre charge de travail dépend d’une version de pilote GPU spécifique.

1. Téléchargez un package d’installation pour les pilotes GPU sur Windows Server 2016 à partir du [site web NVIDIA](https://www.nvidia.com/Download/index.aspx), par exemple, la [version 411.82](https://us.download.nvidia.com/Windows/Quadro_Certified/411.82/411.82-tesla-desktop-winserver2016-international.exe). Enregistrez le fichier localement en utilisant un nom court, comme *GPUDriverSetup.exe*.
2. Créez un fichier zip du package.
3. Chargez le package sur votre compte Batch. Pour connaître les étapes nécessaires, consultez les instructions relatives aux [packages d’applications](batch-application-packages.md). Spécifiez un ID d’application (par exemple, *GPUDriver*) et une version (par exemple, *411.82*).
1. À l’aide des API Batch ou du portail Azure, créez un pool dans la configuration de machine virtuelle avec le nombre de nœuds et l’échelle souhaités. Le tableau suivant présente des exemples de paramètres pour installer les pilotes NVIDIA GPU en mode silencieux à l’aide d’une tâche de démarrage :

| Paramètre | Valeur |
| ---- | ----- | 
| **Type d’image** | Place de marché (Linux/Windows) |
| **Publisher** | MicrosoftWindowsServer |
| **Offer** | WindowsServer |
| **Sku** | 2016-centre-de-données |
| **Taille du nœud** | NC6 Standard |
| **Références du package d’application** | GPUDriver, version 411.82 |
| **Tâche de démarrage activée** | True<br>**Ligne de commande** - `cmd /c "%AZ_BATCH_APP_PACKAGE_GPUDriver#411.82%\\GPUDriverSetup.exe /s"`<br/>**Identité de l’utilisateur** - Pool autouser, admin<br/>**Attente de la réussite** - True

## <a name="example-nvidia-gpu-drivers-on-a-linux-nc-vm-pool"></a>Exemple : Pilotes NVIDIA GPU sur un pool de machines virtuelles de contrôleur de réseau Linux

Pour exécuter des applications CUDA sur un pool de nœuds de contrôleur de réseau Linux, vous devez installer les pilotes NVIDIA Tesla GPU nécessaires à partir de CUDA Toolkit. Les exemples d’étapes suivants créent et déploient une image Ubuntu 16.04 LTS personnalisée avec les pilotes GPU :

1. Déployez une machine virtuelle de série NC Azure exécutant Ubuntu 16.04 LTS. Par exemple, créez la machine virtuelle dans la région USA Centre Sud. 
2. Ajoutez l’[extension NVIDIA GPU Drivers](../virtual-machines/extensions/hpccompute-gpu-linux.md
) à la machine virtuelle à l’aide du portail Azure, d’un ordinateur client qui se connecte à l’abonnement Azure ou d’Azure Cloud Shell. Vous pouvez également suivre les étapes permettant de se connecter à la machine virtuelle et d’[installer les pilotes CUDA](../virtual-machines/linux/n-series-driver-setup.md) manuellement.
3. Suivez les étapes pour créer une [image de Shared Image Gallery](batch-sig-images.md) pour Batch.
4. Créez un compte Batch dans une région qui prend en charge des machines virtuelles NC.
5. En utilisant les API Batch ou le portail Azure, créez un pool [avec l’image personnalisée](batch-sig-images.md), et le nombre de nœuds et l’échelle souhaités. Le tableau suivant contient des exemples de paramètres de pool pour l’image :

| Paramètre | Valeur |
| ---- | ---- |
| **Type d’image** | Image personnalisée |
| **Image personnalisée** | *Nom de l’image* |
| **Référence de l’agent de nœud** | batch.node.ubuntu 16.04 |
| **Taille du nœud** | NC6 Standard |

## <a name="example-microsoft-mpi-on-a-windows-h16r-vm-pool"></a>Exemple : Microsoft MPI sur un pool de machines virtuelles H16r Windows

Pour exécuter des applications Windows MPI sur un pool de nœuds de machine virtuelle H16r Azure, vous devez configurer l’extension HpcVmDrivers et installer [Microsoft MPI](/message-passing-interface/microsoft-mpi). Voici des exemples d’étapes permettant de déployer une image Windows Server 2016 personnalisée avec les pilotes et logiciels nécessaires :

1. Déployez une machine virtuelle H16r Azure exécutant Windows Server 2016. Par exemple, créez la machine virtuelle dans la région USA Ouest. 
2. Ajoutez l’extension HpcVmDrivers à la machine virtuelle en [exécutant une commande Azure PowerShell](../virtual-machines/sizes-hpc.md) à partir d’un ordinateur client qui se connecte à votre abonnement Azure, ou bien à l’aide d’Azure Cloud Shell. 
1. Établissez une connexion Bureau à distance à la machine virtuelle.
1. Téléchargez le [package d’installation](https://www.microsoft.com/download/details.aspx?id=57467) (MSMpiSetup.exe) pour obtenir la dernière version de Microsoft MPI, puis installez Microsoft MPI.
1. Suivez les étapes pour créer une [image de Shared Image Gallery](batch-sig-images.md) pour Batch.
1. À l’aide des API Batch ou via le portail Azure, créez un pool [utilisant Shared Image Gallery](batch-sig-images.md) et offrant le nombre de nœuds ainsi que l’échelle souhaités. Le tableau suivant contient des exemples de paramètres de pool pour l’image :

| Paramètre | Valeur |
| ---- | ---- |
| **Type d’image** | Image personnalisée |
| **Image personnalisée** | *Nom de l’image* |
| **Référence de l’agent de nœud** | batch.node.windows amd64 |
| **Taille du nœud** | H16r Standard |
| **Communication entre les nœuds activée** | True |
| **Nombre maximal de tâches par nœud** | 1 |

## <a name="example-intel-mpi-on-a-linux-h16r-vm-pool"></a>Exemple : Intel MPI sur un pool de machines virtuelles H16r Linux

Pour exécuter des applications MPI sur un pool de nœuds de série H Linux, une option consiste à utiliser l’image [HPC basée sur CentOS 7.4 ](https://azuremarketplace.microsoft.com/marketplace/apps/openlogic.centos-hpc?tab=Overview) de la Place de marché Azure. Les pilotes RDMA et Intel MPI Linux sont préinstallés. Cette image prend également en charge les charges de travail de conteneur Docker.

En utilisant les API Batch ou le portail Azure, créez un pool à l’aide de cette image avec le nombre de nœuds et l’échelle souhaités. Le tableau suivant contient des exemples de paramètres de pool :

| Paramètre | Valeur |
| ---- | ---- |
| **Type d’image** | Place de marché (Linux/Windows) |
| **Publisher** | OpenLogic |
| **Offer** | CentOS-HPC |
| **Sku** | 7.4 |
| **Taille du nœud** | H16r Standard |
| **Communication entre les nœuds activée** | True |
| **Nombre maximal de tâches par nœud** | 1 |

## <a name="next-steps"></a>Étapes suivantes

* Pour exécuter des travaux MPI sur un pool Azure Batch, consultez les exemples [Windows](batch-mpi.md) ou [Linux](/archive/blogs/windowshpc/introducing-mpi-support-for-linux-on-azure-batch).

* Pour obtenir des exemples de charges de travail GPU sur Batch, consultez les recettes [Batch Shipyard](https://github.com/Azure/batch-shipyard/).
