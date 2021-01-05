---
title: Configurer l’interface de transmission de messages pour HPC - Machines virtuelles Azure | Microsoft Docs
description: Découvrez comment configurer MPI pour HPC sur Azure.
author: vermagit
ms.service: virtual-machines
ms.subservice: workloads
ms.topic: article
ms.date: 08/06/2020
ms.author: amverma
ms.reviewer: cynthn
ms.openlocfilehash: 08d952738a085aa6ed814668ece898f7460b4f33
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94963740"
---
# <a name="set-up-message-passing-interface-for-hpc"></a>Configurer l’interface de transmission de messages pour HPC

[L’interface MPI (Message Passing Interface)](https://en.wikipedia.org/wiki/Message_Passing_Interface) est une bibliothèque ouverte et une norme de facto pour la parallélisation de la mémoire distribuée. Elle est couramment utilisée dans de nombreuses charges de travail HPC. Les charges de travail HPC sur les machines virtuelles [RDMA](../../sizes-hpc.md#rdma-capable-instances) [Série H](../../sizes-hpc.md) et [Série N](../../sizes-gpu.md) peuvent utiliser MPI pour communiquer sur le réseau InfiniBand à faible latence et à bande passante élevée.

Les tailles des machines virtuelles compatibles SR-IOV sur Azure (HBv2, HB, HC, NCv3 et NDv2) permettent d’utiliser presque toutes les versions de MPI avec Mellanox OFED. Sur les machines virtuelles non compatibles SR-IOV, les implémentations MPI prises en charge utilisent l’interface Microsoft Network Direct (ND) pour la communication entre les machines virtuelles. Par conséquent, seules les versions Microsoft MPI (MS-MPI) 2012 R2 ou ultérieures et Intel MPI 5.x sont supportées. Les versions ultérieures (2017, 2018) de la bibliothèque runtime MPI Intel peuvent ne pas être compatibles avec les pilotes Azure RDMA.

Pour les [machines virtuelles RDMA](../../sizes-hpc.md#rdma-capable-instances) compatibles avec SR-IOV, les images de machines virtuelles [CentOS-HPC version 7.6 ou ultérieure](https://techcommunity.microsoft.com/t5/Azure-Compute/CentOS-HPC-VM-Image-for-SR-IOV-enabled-Azure-HPC-VMs/ba-p/665557) dans la Place de marché sont optimisées et préchargées avec les pilotes OFED pour RDMA et plusieurs bibliothèques MPI et packages de calcul scientifique couramment utilisés. Cela constitue le moyen le plus simple pour se lancer.

Les exemples de cette procédure illustrent la syntaxe RHEL/CentOS, mais les étapes sont générales et peuvent être utilisées pour tout système d’exploitation compatible, par exemple Ubuntu (16.04, 18.04, 19.04, 20.04) et SLES (12 SP4 et 15). Vous trouverez plus d’exemples sur la configuration d’autres implémentations MPI sur d’autres distributions dans le [référentiel azhpc-images](https://github.com/Azure/azhpc-images/blob/master/ubuntu/ubuntu-18.x/ubuntu-18.04-hpc/install_mpis.sh).

> [!NOTE]
> L’exécution de travaux MPI sur des machines virtuelles compatibles avec SR-IOV nécessite la configuration de clés de partition (p-keys) sur un locataire à des fins d’isolement et de sécurité. Suivez les étapes de la section [Découvrir les clés de partition](#discover-partition-keys) pour en savoir plus sur la détermination des valeurs des p-keys et leur configuration correcte pour un travail MPI.

## <a name="ucx"></a>UCX

[Unified Communication X (UCX)](https://github.com/openucx/ucx) est un framework d’API de communication pour HPC. Il est optimisé pour la communication MPI sur InfiniBand et fonctionne avec de nombreuses implémentations MPI telles que OpenMPI et MPICH.

```bash
wget https://github.com/openucx/ucx/releases/download/v1.4.0/ucx-1.4.0.tar.gz
tar -xvf ucx-1.4.0.tar.gz
cd ucx-1.4.0
./configure --prefix=<ucx-install-path>
make -j 8 && make install
```

## <a name="hpc-x"></a>HPC-X

Le [kit d’outils logiciels HPC-X](https://www.mellanox.com/products/hpc-x-toolkit) contient UCX et HCOLL.

```bash
HPCX_VERSION="v2.6.0"
HPCX_DOWNLOAD_URL=https://azhpcstor.blob.core.windows.net/azhpc-images-store/hpcx-v2.6.0-gcc-MLNX_OFED_LINUX-5.0-1.0.0.0-redhat7.7-x86_64.tbz
get --retry-connrefused --tries=3 --waitretry=5 $HPCX_DOWNLOAD_URL
tar -xvf hpcx-${HPCX_VERSION}-gcc-MLNX_OFED_LINUX-5.0-1.0.0.0-redhat7.7-x86_64.tbz
mv hpcx-${HPCX_VERSION}-gcc-MLNX_OFED_LINUX-5.0-1.0.0.0-redhat7.7-x86_64 ${INSTALL_PREFIX}
HPCX_PATH=${INSTALL_PREFIX}/hpcx-${HPCX_VERSION}-gcc-MLNX_OFED_LINUX-5.0-1.0.0.0-redhat7.7-x86_64
```

Exécuter HPC-X

```bash
${HPCX_PATH}mpirun -np 2 --map-by ppr:2:node -x UCX_TLS=rc ${HPCX_PATH}/ompi/tests/osu-micro-benchmarks-5.3.2/osu_latency
```

## <a name="openmpi"></a>OpenMPI

Installez UCX comme décrit ci-dessus. HCOLL fait partie du [kit d’outils logiciels HPC-X](https://www.mellanox.com/products/hpc-x-toolkit) et ne nécessite pas d’installation spéciale.

Installez OpenMPI à partir des packages disponibles dans le référentiel.

```bash
sudo yum install –y openmpi
```

Compilez OpenMPI.

```bash
OMPI_VERSION="4.0.3"
OMPI_DOWNLOAD_URL=https://download.open-mpi.org/release/open-mpi/v4.0/openmpi-${OMPI_VERSION}.tar.gz
wget --retry-connrefused --tries=3 --waitretry=5 $OMPI_DOWNLOAD_URL
tar -xvf openmpi-${OMPI_VERSION}.tar.gz
cd openmpi-${OMPI_VERSION}
./configure --prefix=${INSTALL_PREFIX}/openmpi-${OMPI_VERSION} --with-ucx=${UCX_PATH} --with-hcoll=${HCOLL_PATH} --enable-mpirun-prefix-by-default --with-platform=contrib/platform/mellanox/optimized && make -j$(nproc) && make install
```

Exécutez OpenMPI.

```bash
${INSTALL_PREFIX}/bin/mpirun -np 2 --map-by node --hostfile ~/hostfile -mca pml ucx --mca btl ^vader,tcp,openib -x UCX_NET_DEVICES=mlx5_0:1  -x UCX_IB_PKEY=0x0003  ./osu_latency
```

Vérifiez votre clé de partition comme indiqué ci-dessus.

## <a name="intel-mpi"></a>Intel MPI

Téléchargez la version [Intel MPI](https://software.intel.com/mpi-library/choose-download) de votre choix. Modifiez la variable d’environnement I_MPI_FABRICS selon la version. Utilisez `I_MPI_FABRICS=shm:ofa` pour Intel MPI 2018 et `I_MPI_FABRICS=shm:ofi` pour 2019.

### <a name="non-sr-iov-vms"></a>Machines virtuelles non SR-IOV
Pour les machines virtuelles non SR-IOV, voici un exemple de téléchargement de la [version d’évaluation gratuite](https://registrationcenter.intel.com/en/forms/?productid=1740) du runtime 5.x :
```bash
wget http://registrationcenter-download.intel.com/akdlm/irc_nas/tec/9278/l_mpi_p_5.1.3.223.tgz
```
Pour les étapes d’installation, consultez le [Guide d’installation de la bibliothèque Intel MPI](https://registrationcenter-download.intel.com/akdlm/irc_nas/1718/INSTALL.html?lang=en&fileExt=.html).
Vous pouvez aussi activer ptrace pour les processus non-racine et non-débogueur (nécessaire pour les versions les plus récentes d’Intel MPI).
```bash
echo 0 | sudo tee /proc/sys/kernel/yama/ptrace_scope
```

### <a name="suse-linux"></a>SUSE Linux
Pour les versions d’images de machines virtuelles SUSE Linux Enterprise Server - SLES 12 SP3 pour HPC, SLES 12 SP3 pour HPC (Premium), SLES 12 SP1 pour HPC, SLES 12 SP1 pour HPC (Premium), SLES 12 SP4 et SLES 15, les pilotes RDMA sont installés et les packages Intel MPI sont distribués sur la machine virtuelle. Installez Intel MPI en exécutant la commande suivante :
```bash
sudo rpm -v -i --nodeps /opt/intelMPI/intel_mpi_packages/*.rpm
```

## <a name="mpich"></a>MPICH

Installez UCX comme décrit ci-dessus. Créez MPICH.

```bash
wget https://www.mpich.org/static/downloads/3.3/mpich-3.3.tar.gz
tar -xvf mpich-3.3.tar.gz
cd mpich-3.3
./configure --with-ucx=${UCX_PATH} --prefix=${INSTALL_PREFIX} --with-device=ch4:ucx
make -j 8 && make install
```

Exécution de MPICH.

```bash
${INSTALL_PREFIX}/bin/mpiexec -n 2 -hostfile ~/hostfile -env UCX_IB_PKEY=0x0003 -bind-to hwthread ./osu_latency
```

Vérifiez votre clé de partition comme indiqué ci-dessus.

## <a name="mvapich2"></a>MVAPICH2

Créer MVAPICH2.

```bash
wget http://mvapich.cse.ohio-state.edu/download/mvapich/mv2/mvapich2-2.3.tar.gz
tar -xv mvapich2-2.3.tar.gz
cd mvapich2-2.3
./configure --prefix=${INSTALL_PREFIX}
make -j 8 && make install
```

Exécution de MVAPICH2.

```bash
${INSTALL_PREFIX}/bin/mpirun_rsh -np 2 -hostfile ~/hostfile MV2_CPU_MAPPING=48 ./osu_latency
```

## <a name="platform-mpi-community-edition"></a>Platform MPI Community Edition

Installez les packages requis pour Platform MPI.

```bash
sudo yum install libstdc++.i686
sudo yum install glibc.i686
Download platform MPI at https://www.ibm.com/developerworks/downloads/im/mpi/index.html 
sudo ./platform_mpi-09.01.04.03r-ce.bin
```

Suivez le processus d’installation.

## <a name="osu-mpi-benchmarks"></a>OSU MPI Benchmarks

Téléchargez [OSU MPI Benchmarks](http://mvapich.cse.ohio-state.edu/benchmarks/) et décompressez le fichier.

```bash
wget http://mvapich.cse.ohio-state.edu/download/mvapich/osu-micro-benchmarks-5.5.tar.gz
tar –xvf osu-micro-benchmarks-5.5.tar.gz
cd osu-micro-benchmarks-5.5
```

Créez des bancs d’essai à l’aide d’une bibliothèque MPI particulière :

```bash
CC=<mpi-install-path/bin/mpicc>CXX=<mpi-install-path/bin/mpicxx> ./configure 
make
```

MPI Benchmarks se trouve dans le dossier `mpi/`.


## <a name="discover-partition-keys"></a>Découvrir les clés de partition

Découvrez les clés de partition (p-keys) pour communiquer avec d’autres machines virtuelles au sein du même locataire (groupe à haute disponibilité ou groupe de machines virtuelles identiques).

```bash
/sys/class/infiniband/mlx5_0/ports/1/pkeys/0
/sys/class/infiniband/mlx5_0/ports/1/pkeys/1
```

La plus grande des deux est la clé de locataire, qui doit être utilisée avec MPI. Exemple : Si les éléments suivants sont les p-keys, 0x800b doit être utilisé avec MPI.

```bash
cat /sys/class/infiniband/mlx5_0/ports/1/pkeys/0
0x800b
cat /sys/class/infiniband/mlx5_0/ports/1/pkeys/1
0x7fff
```

Utiliser une partition autre que de la clé de partition par défaut (0x7fff). UCX requiert l’effacement de l’octet le plus significatif de la p-key. Par exemple, définissez UCX_IB_PKEY comme étant 0x000b pour 0x800b.

Notez également que tant que le locataire (groupe à haute disponibilité ou groupe de machines virtuelles identiques) existe, les PKEY restent les mêmes. Cela est vrai même lorsque les nœuds sont ajoutés ou supprimés. Les nouveaux locataires obtiennent des PKEYs différents.


## <a name="set-up-user-limits-for-mpi"></a>Configurez un nombre limite d’utilisateurs pour MPI

Configurez un nombre limite d’utilisateurs pour MPI.

```bash
cat << EOF | sudo tee -a /etc/security/limits.conf
*               hard    memlock         unlimited
*               soft    memlock         unlimited
*               hard    nofile          65535
*               soft    nofile          65535
EOF
```

## <a name="set-up-ssh-keys-for-mpi"></a>Configurez les clés SSH pour MPI

Configurer les clés SSH pour les types de MPI qui en ont besoin.

```bash
ssh-keygen -f /home/$USER/.ssh/id_rsa -t rsa -N ''
cat << EOF > /home/$USER/.ssh/config
Host *
    StrictHostKeyChecking no
EOF
cat /home/$USER/.ssh/id_rsa.pub >> /home/$USER/.ssh/authorized_keys
chmod 600 /home/$USER/.ssh/authorized_keys
chmod 644 /home/$USER/.ssh/config
```

La syntaxe ci-dessus suppose un répertoire de base partagé, sinon le répertoire .ssh doit être copié dans chaque nœud.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur les machines virtuelles [Série H](../../sizes-hpc.md) et [Série N](../../sizes-gpu.md) avec [InfiniBand](../../sizes-hpc.md#rdma-capable-instances)
- Consultez [Vue d’ensemble de la série HB](hb-series-overview.md) et [Vue d’ensemble de la série HC](hc-series-overview.md) pour en savoir plus sur la configuration optimale des charges de travail pour les performances et la scalabilité.
- Découvrez des informations sur les dernières annonces et des exemples et des résultats HPC sur les [blogs de la communauté Azure Compute Tech](https://techcommunity.microsoft.com/t5/azure-compute/bg-p/AzureCompute).
- Pour une vision plus globale de l’architecture d’exécution des charges de travail HPC, consultez [Calcul haute performance (HPC) sur Azure](/azure/architecture/topics/high-performance-computing/).
