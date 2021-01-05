---
title: Activer InifinBand avec des machines virtuelles HPC - Machines virtuelles Azure | Microsoft Docs
description: Découvrez comment activer InfiniBand avec des machines virtuelles Azure HPC.
author: vermagit
ms.service: virtual-machines
ms.subservice: workloads
ms.topic: article
ms.date: 11/06/2020
ms.author: amverma
ms.reviewer: cynthn
ms.openlocfilehash: 2b2b9a8188bc83bba029755ffbbc590999cf9b3d
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94966987"
---
# <a name="enable-infiniband"></a>Activer InfiniBand

Les machines virtuelles [Série H](../../sizes-hpc.md) et [Série N](../../sizes-gpu.md) avec [RDMA](../../sizes-hpc.md#rdma-capable-instances) communiquent sur le réseau InfiniBand à faible latence et à bande passante élevée. La fonctionnalité RDMA sur une telle interconnexion est critique pour améliorer la scalabilité et les performances des charges de travail HPC et IA sur les nœuds distribués. Les machines virtuelles des séries H et N avec InfiniBand sont connectées dans un réseau fat-tree avec une conception à faible diamètre pour des performances RDMA optimisées et cohérentes.

Il existe plusieurs façons d’activer InfiniBand sur les tailles de machines virtuelles compatibles.

## <a name="vm-images-with-infiniband-drivers"></a>Images de machine virtuelle avec des pilotes InfiniBand
Pour obtenir la liste des images de machines virtuelles prises en charge sur la place de marché, consultez les[Images de machine virtuelle](configure.md#vm-images), qui sont préchargées avec des pilotes InfiniBand (pour les machines virtuelles SR-IOV ou non-SR-IOV) ou peuvent être configurées avec les pilotes appropriés.
Pour la SR-IOV activée [les machines virtuelles compatibles RDMA](../../sizes-hpc.md#rdma-capable-instances), [CentOS-HPC version 7.6 ou une version ultérieure de](https://techcommunity.microsoft.com/t5/Azure-Compute/CentOS-HPC-VM-Image-for-SR-IOV-enabled-Azure-HPC-VMs/ba-p/665557) les images de machine virtuelle de la place de marché sont le moyen le plus simple de commencer.
Les images de machine virtuelle Ubuntu peuvent être configurées avec les bons pilotes suivant ces [instructions](https://techcommunity.microsoft.com/t5/azure-compute/configuring-infiniband-for-ubuntu-hpc-and-gpu-vms/ba-p/1221351), sur les machines virtuelles SR-IOV ou non-SR-IOV.

## <a name="infiniband-driver-vm-extensions"></a>Extensions de machine virtuelle du pilote InfiniBand
Sur Linux, pour les machines virtuelles des séries H et N prenant en charge SR-IOV, [l’extension de machine virtuelle InfiniBandDriverLinux](../../extensions/hpc-compute-infiniband-linux.md) peut être utilisée pour installer les pilotes OFED Mellanox et activer InfiniBand.

Sur Windows, [l’extension de machine virtuelle InfiniBandDriverWindows](../../extensions/hpc-compute-infiniband-windows.md) installe des pilotes Windows Network Direct (sur des machines virtuelles non compatibles SR-IOV) ou des pilotes Mellanox OFED (sur des machines virtuelles compatibles SR-IOV) pour la connectivité RDMA. Dans certains déploiements des instances A8 et A9, l’extension HpcVmDrivers est ajoutée automatiquement. Notez que l’extension de machine virtuelle HpcVmDrivers est déconseillée ; elle ne sera pas mise à jour.

Pour ajouter l’extension de machine virtuelle sur une machine virtuelle, vous pouvez utiliser les cmdlets [Azure PowerShell](/powershell/azure/). Pour plus d’informations, consultez [Extensions et fonctionnalités de la machine virtuelle](../../extensions/overview.md). Vous pouvez également utiliser les extensions pour les machines virtuelles déployées dans le [modèle de déploiement classique](/previous-versions/azure/virtual-machines/windows/classic/agents-and-extensions-classic).

## <a name="manual-installation"></a>Installation manuelle
[Les pilotes Mellanox OpenFabrics (OFED)](https://www.mellanox.com/products/InfiniBand-VPI-Software) peuvent être installés manuellement sur les machines virtuelles [compatibles SR-IOV](../../sizes-hpc.md#rdma-capable-instances) [de série H](../../sizes-hpc.md) et les machines virtuelles [de série N](../../sizes-gpu.md).

### <a name="linux"></a>Linux
Les pilotes [OFED pour Linux](https://www.mellanox.com/products/infiniband-drivers/linux/mlnx_ofed) peuvent être installés avec l’exemple ci-dessous. L’exemple de cette procédure illustre la syntaxe RHEL/CentOS, mais les étapes sont générales et peuvent être utilisées pour tout système d’exploitation compatible, par exemple Ubuntu (16.04, 18.04, 19.04, 20.04) et SLES (12 SP4 et 15). D’autres exemples pour les autres distributions se trouvent sur le [référentiel azhpc-images](https://github.com/Azure/azhpc-images/blob/master/ubuntu/ubuntu-18.x/ubuntu-18.04-hpc/install_mellanoxofed.sh). Les pilotes de la boîte de réception fonctionnent également, mais les pilotes Mellanox OpenFabrics fournissent davantage de fonctionnalités.

```bash
MLNX_OFED_DOWNLOAD_URL=http://content.mellanox.com/ofed/MLNX_OFED-5.0-2.1.8.0/MLNX_OFED_LINUX-5.0-2.1.8.0-rhel7.7-x86_64.tgz
# Optionally verify checksum
wget --retry-connrefused --tries=3 --waitretry=5 $MLNX_OFED_DOWNLOAD_URL
tar zxvf MLNX_OFED_LINUX-5.0-2.1.8.0-rhel7.7-x86_64.tgz

KERNEL=( $(rpm -q kernel | sed 's/kernel\-//g') )
KERNEL=${KERNEL[-1]}
# Uncomment the lines below if you are running this on a VM
#RELEASE=( $(cat /etc/centos-release | awk '{print $4}') )
#yum -y install http://olcentgbl.trafficmanager.net/centos/${RELEASE}/updates/x86_64/kernel-devel-${KERNEL}.rpm
yum install -y kernel-devel-${KERNEL}
./MLNX_OFED_LINUX-5.0-2.1.8.0-rhel7.7-x86_64/mlnxofedinstall --kernel $KERNEL --kernel-sources /usr/src/kernels/${KERNEL} --add-kernel-support --skip-repo
```

### <a name="windows"></a>Windows
Pour Windows, téléchargez et installez les [pilotes Mellanox OFED pour Windows](https://www.mellanox.com/products/adapter-software/ethernet/windows/winof-2).

## <a name="enable-ip-over-infiniband-ib"></a>Activer IP sur InfiniBand (IB)
Si vous envisagez d’exécuter des travaux MPI, vous n’avez généralement pas besoin de l’IPoIB. La bibliothèque MPI utilise l’interface de verbes pour la communication IB (à moins que vous n’utilisiez explicitement le canal TCP/IP de la bibliothèque MPI). Toutefois, si vous disposez d’une application qui utilise TCP/IP pour la communication et que vous souhaitez exécuter sur IB, vous pouvez utiliser IPoIB sur l’interface IB. Utilisez les commandes suivantes (pour RHEL/CentOS) pour activer IP sur InfiniBand.

```bash
sudo sed -i -e 's/# OS.EnableRDMA=n/OS.EnableRDMA=y/g' /etc/waagent.conf
sudo systemctl restart waagent
```

## <a name="next-steps"></a>Étapes suivantes

- Découvrez-en davantage sur l’installation des [différentes bibliothèques MPI prises en charge](setup-mpi.md) et leur configuration optimale sur les machines virtuelles.
- Consultez [Vue d’ensemble de la série HB](hb-series-overview.md) et [Vue d’ensemble de la série HC](hc-series-overview.md) pour en savoir plus sur la configuration optimale des charges de travail pour les performances et la scalabilité.
- Découvrez des informations sur les dernières annonces et des exemples et des résultats HPC sur les [blogs de la communauté Azure Compute Tech](https://techcommunity.microsoft.com/t5/azure-compute/bg-p/AzureCompute).
- Pour une vision plus globale de l’architecture d’exécution des charges de travail HPC, consultez [Calcul haute performance (HPC) sur Azure](/azure/architecture/topics/high-performance-computing/).
