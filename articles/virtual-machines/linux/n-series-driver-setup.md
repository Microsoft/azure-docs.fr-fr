---
title: Configuration des pilotes GPU de série N Azure pour Linux
description: Procédure de configuration des pilotes GPU NVIDIA pour les machines virtuelles série N exécutant Linux dans Azure
services: virtual-machines-linux
author: vikancha-MSFT
ms.service: virtual-machines-linux
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 01/09/2019
ms.author: vikancha
ms.openlocfilehash: 716a8853a2e2e0988cc50f5289f448d7a4adc9be
ms.sourcegitcommit: 4c89d9ea4b834d1963c4818a965eaaaa288194eb
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96608705"
---
# <a name="install-nvidia-gpu-drivers-on-n-series-vms-running-linux"></a>Installer les pilotes GPU NVIDIA sur les machines virtuelles série N exécutant Linux

Pour tirer parti des fonctionnalités GPU de machines virtuelles de la série N Azure reposant sur les GPU NVIDIA, vous devez installer des pilotes GPU NVIDIA. [L’extension du pilote GPU NVIDIA](../extensions/hpccompute-gpu-linux.md) installe les pilotes CUDA ou GRID NVIDIA appropriés sur une machine virtuelle de série N. Installez ou gérez l’extension à l’aide du portail Azure ou d’outils tels qu’Azure CLI ou les modèles Azure Resource Manager. Consultez la [documentation sur l’extension du pilote GPU NVIDIA](../extensions/hpccompute-gpu-linux.md) pour connaître les distributions prises en charge et les étapes de déploiement.

Si vous choisissez d’installer manuellement les pilotes GPU NVIDIA, cet article indique les distributions prises en charge, les pilotes et les étapes d’installation et de vérification. Des informations de configuration manuelle du pilote sont également disponibles pour [les machines virtuelles Windows](../windows/n-series-driver-setup.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

Pour plus d’informations sur les spécifications, les capacités de stockage et les disques des machines virtuelles série N, consultez l’article [Tailles de machine virtuelle Linux GPU](../sizes-gpu.md?toc=/azure/virtual-machines/linux/toc.json). 

> [!NOTE]
> Cet article contient des références au terme *liste noire*, un terme que Microsoft n’utilise plus. Lorsque le terme sera supprimé du logiciel, nous le supprimerons de cet article.

[!INCLUDE [virtual-machines-n-series-linux-support](../../../includes/virtual-machines-n-series-linux-support.md)]

## <a name="install-cuda-drivers-on-n-series-vms"></a>Installer les pilotes CUDA sur les machines virtuelles de série N

Voici les étapes à suivre pour installer les pilotes CUDA de la boîte à outils CUDA NVIDIA sur des machines virtuelles de série N. 


Les développeurs C et C++ peuvent éventuellement installer le kit d’outils complet pour créer des applications avec accélération GPU. Pour plus d’informations, consultez le [Guide d’installation de CUDA](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html).

Pour installer les pilotes CUDA, établissez une connexion SSH sur chaque machine virtuelle. Pour vérifier que le système dispose d’un GPU compatible CUDA, exécutez la commande suivante :

```bash
lspci | grep -i NVIDIA
```
Vous verrez une sortie similaire à l’exemple suivant (illustrant une carte K80 Tesla NVIDIA) :

![Sortie de commande Ispci](./media/n-series-driver-setup/lspci.png)

Ensuite, exécutez les commandes d’installation spécifiques de votre distribution.

### <a name="ubuntu"></a>Ubuntu 

1. Téléchargez et installez les pilotes CUDA à partir du site web NVIDIA. Par exemple, pour Ubuntu 16.04 LTS :
   ```bash
   CUDA_REPO_PKG=cuda-repo-ubuntu1604_10.0.130-1_amd64.deb

   wget -O /tmp/${CUDA_REPO_PKG} http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/${CUDA_REPO_PKG} 

   sudo dpkg -i /tmp/${CUDA_REPO_PKG}

   sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/7fa2af80.pub 

   rm -f /tmp/${CUDA_REPO_PKG}

   sudo apt-get update

   sudo apt-get install cuda-drivers

   ```

   L’installation peut prendre plusieurs minutes.

2. Pour éventuellement installer le kit d’outils CUDA complet, saisissez :

   ```bash
   sudo apt-get install cuda
   ```

3. Redémarrez la machine virtuelle et vérifiez l’installation.

#### <a name="cuda-driver-updates"></a>Mises à jour de pilote CUDA

Nous vous recommandons de régulièrement mettre à jour les pilotes CUDA après le déploiement.

```bash
sudo apt-get update

sudo apt-get upgrade -y

sudo apt-get dist-upgrade -y

sudo apt-get install cuda-drivers

sudo reboot
```

### <a name="centos-or-red-hat-enterprise-linux"></a>CentOS ou Red Hat Enterprise Linux

1. Mettez à jour le noyau (recommandé). Si vous choisissez de ne pas mettre à jour le noyau, vérifiez que les versions de `kernel-devel` et `dkms` sont appropriées pour votre noyau.

   ```
   sudo yum install kernel kernel-tools kernel-headers kernel-devel
  
   sudo reboot

2. Install the latest [Linux Integration Services for Hyper-V and Azure](https://www.microsoft.com/download/details.aspx?id=55106). Check if LIS is required by verifying the results of lspci. If all GPU devices are listed as expected, installing LIS is not required.

Skip this step if you plan to use CentOS 7.8(or higher) as LIS is no longer required for these versions.

   ```bash
   wget https://aka.ms/lis
 
   tar xvzf lis
 
   cd LISISO
 
   sudo ./install.sh
 
   sudo reboot
   ```
 
3. Reconnectez-vous à la machine virtuelle et continuez l’installation avec les commandes suivantes :

   ```bash
   sudo rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

   sudo yum install dkms

   CUDA_REPO_PKG=cuda-repo-rhel7-10.0.130-1.x86_64.rpm

   wget http://developer.download.nvidia.com/compute/cuda/repos/rhel7/x86_64/${CUDA_REPO_PKG} -O /tmp/${CUDA_REPO_PKG}

   sudo rpm -ivh /tmp/${CUDA_REPO_PKG}

   rm -f /tmp/${CUDA_REPO_PKG}

   sudo yum install cuda-drivers
   ```

   L’installation peut prendre plusieurs minutes. 

4. Pour éventuellement installer le kit d’outils CUDA complet, saisissez :

   ```bash
   sudo yum install cuda
   ```

5. Redémarrez la machine virtuelle et vérifiez l’installation.

### <a name="verify-driver-installation"></a>Vérification de l’installation du pilote

Pour interroger l’état de l’appareil GPU, connectez-vous par SSH à la machine virtuelle et exécutez l’utilitaire de ligne de commande [nvidia-smi](https://developer.nvidia.com/nvidia-system-management-interface) installé avec le pilote. 

Si le pilote est installé, vous obtenez un résultat qui ressemble à celui indiqué. Notez que **GPU-Util** affiche 0 %, sauf si vous exécutez actuellement une charge de travail GPU sur la machine virtuelle. La version de votre pilote et vos détails de GPU peuvent différer de ceux indiqués.

![État de l’appareil NVIDIA](./media/n-series-driver-setup/smi.png)

## <a name="rdma-network-connectivity"></a>Connectivité réseau RDMA

La connectivité réseau RDMA peut être activée sur des machines virtuelles de série N compatibles RDMA, comme les machines NC24r déployées dans le même groupe à haute disponibilité ou dans un seul groupe de placement au sein d’un groupe de machines virtuelles identiques. Le réseau RDMA prend en charge le trafic MPI (Message Passing Interface) pour les applications exécutées avec Intel MPI 5.x ou une version ultérieure. Des conditions supplémentaires suivent :

### <a name="distributions"></a>Distributions

Déployez des machines virtuelles de série N compatibles RDMA à partir de l’une des images suivantes de la Place de marché Azure qui prend en charge la connectivité RDMA sur des machines virtuelles de série N :
  
* **Ubuntu 16.04 LTS** - Configurez les pilotes RDMA sur la machine virtuelle et inscrivez-vous auprès d’Intel pour télécharger Intel MPI :

  [!INCLUDE [virtual-machines-common-ubuntu-rdma](../../../includes/virtual-machines-common-ubuntu-rdma.md)]

* **CentOS 7.4 HPC** - Les pilotes RDMA et Intel MPI 5.1 sont installés sur la machine virtuelle.

* **HPC basé sur CentOS** - CentOS-HPC 7.6 et versions ultérieures (pour les références SKU où InfiniBand est pris en charge sur SR-IOV). Ces images ont des bibliothèques Mellanox OFED et MPI préinstallées.

> [!NOTE]
> Les cartes CX3 Pro sont prises en charge uniquement par le biais des versions LTS de Mellanox OFED. Utilisez la version LTS de Mellanox OFED (4.9-0.1.7.0) sur les machines virtuelles de la série N avec des cartes ConnectX3-Pro. Pour plus d’informations, consultez [Pilotes Linux](https://www.mellanox.com/products/infiniband-drivers/linux/mlnx_ofed).
>
> En outre, certaines des dernières images HPC de la Place de marché Azure ont Mellanox OFED 5.1 ou des versions ultérieures ; ces versions ne prennent pas en charge les cartes ConnectX3-Pro. Vérifiez la version de Mellanox OFED dans l’image HPC avant de l’utiliser sur des machines virtuelles avec des cartes ConnectX3-Pro.
>
> Les images suivantes sont les dernières images CentOS-HPC qui prennent en charge les cartes ConnectX3-Pro :
>
> - OpenLogic:CentOS-HPC:7.6:7.6.2020062900
> - OpenLogic:CentOS-HPC:7_6gen2:7.6.2020062901
> - OpenLogic:CentOS-HPC:7.7:7.7.2020062600
> - OpenLogic:CentOS-HPC:7_7-gen2:7.7.2020062601
> - OpenLogic:CentOS-HPC:8_1:8.1.2020062400
> - OpenLogic:CentOS-HPC:8_1-gen2:8.1.2020062401
>

## <a name="install-grid-drivers-on-nv-or-nvv3-series-vms"></a>Installer les pilotes GRID sur les machines virtuelles de série NV ou NVv3

Pour installer les pilotes GRID NVIDIA sur les machines virtuelles de série NV ou NVv3, établissez une connexion SSH avec chaque machine virtuelle et suivez les étapes correspondant à votre distribution Linux. 

### <a name="ubuntu"></a>Ubuntu 

1. Exécutez la commande `lspci`. Vérifiez que la ou les cartes NVIDIA M60 sont visibles en tant que périphériques PCI.

2. Installez les mises à jour.

   ```bash
   sudo apt-get update

   sudo apt-get upgrade -y

   sudo apt-get dist-upgrade -y

   sudo apt-get install build-essential ubuntu-desktop -y
   
   sudo apt-get install linux-azure -y
   ```
3. Désactivez le pilote du noyau Nouveau, qui n’est pas compatible avec le pilote NVIDIA. (Utilisez uniquement le pilote NVIDIA sur les machines virtuelles NV ou NVv2.) Pour ce faire, créez un fichier `/etc/modprobe.d` nommé `nouveau.conf` avec le contenu suivant :

   ```
   blacklist nouveau

   blacklist lbm-nouveau
   ```


4. Redémarrez la machine virtuelle et reconnectez-vous. Quittez le serveur X :

   ```bash
   sudo systemctl stop lightdm.service
   ```

5. Téléchargez et installez le pilote GRID :

   ```bash
   wget -O NVIDIA-Linux-x86_64-grid.run https://go.microsoft.com/fwlink/?linkid=874272  

   chmod +x NVIDIA-Linux-x86_64-grid.run

   sudo ./NVIDIA-Linux-x86_64-grid.run
   ``` 

6. Lorsque vous êtes invité à indiquer si vous souhaitez exécuter l’utilitaire nvidia-xconfig pour mettre à jour votre fichier de configuration X, sélectionnez **Oui**.

7. Une fois l’installation terminée, copiez /etc/nvidia/gridd.conf.template sur un nouveau fichier gridd.conf dà l’emplacement etc/nvidia /

   ```bash
   sudo cp /etc/nvidia/gridd.conf.template /etc/nvidia/gridd.conf
   ```

8. Ajoutez la ligne suivante à `/etc/nvidia/gridd.conf` :
 
   ```
   IgnoreSP=FALSE
   EnableUI=FALSE
   ```
   
9. Le cas échéant, supprimez ce qui suit de `/etc/nvidia/gridd.conf` :
 
   ```
   FeatureType=0
   ```
10. Redémarrez la machine virtuelle et vérifiez l’installation.


### <a name="centos-or-red-hat-enterprise-linux"></a>CentOS ou Red Hat Enterprise Linux 

1. Mettez à jour le noyau et DKMS (recommandé). Si vous choisissez de ne pas mettre à jour le noyau, vérifiez que les versions de `kernel-devel` et `dkms` sont appropriées pour votre noyau.
 
   ```bash  
   sudo yum update
 
   sudo yum install kernel-devel
 
   sudo rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
 
   sudo yum install dkms
   
   sudo yum install hyperv-daemons
   ```

2. Désactivez le pilote du noyau Nouveau, qui n’est pas compatible avec le pilote NVIDIA. (Utilisez uniquement le pilote NVIDIA sur les machines virtuelles NV ou NV3.) Pour ce faire, créez un fichier `/etc/modprobe.d` nommé `nouveau.conf` avec le contenu suivant :

   ```
   blacklist nouveau

   blacklist lbm-nouveau
   ```
 
3. Redémarrez la machine virtuelle, reconnectez-vous et installez les derniers composants [Linux Integration Services pour Hyper-V et Azure](https://www.microsoft.com/download/details.aspx?id=55106). Vérifiez si LIS est requis en vérifiant les résultats d’lspci. Si tous les périphériques GPU sont répertoriés comme prévu, il n’est pas nécessaire d’installer LIS. 

Ignorez cette étape si vous utilisez CentOS/RHEL 7.8 et une version ultérieure.
 
   ```bash
   wget https://aka.ms/lis

   tar xvzf lis

   cd LISISO

   sudo ./install.sh

   sudo reboot

   ```
 
4. Reconnectez-vous à la machine virtuelle et exécutez la commande `lspci`. Vérifiez que la ou les cartes NVIDIA M60 sont visibles en tant que périphériques PCI.
 
5. Téléchargez et installez le pilote GRID :

   ```bash
   wget -O NVIDIA-Linux-x86_64-grid.run https://go.microsoft.com/fwlink/?linkid=874272  

   chmod +x NVIDIA-Linux-x86_64-grid.run

   sudo ./NVIDIA-Linux-x86_64-grid.run
   ``` 
6. Lorsque vous êtes invité à indiquer si vous souhaitez exécuter l’utilitaire nvidia-xconfig pour mettre à jour votre fichier de configuration X, sélectionnez **Oui**.

7. Une fois l’installation terminée, copiez /etc/nvidia/gridd.conf.template sur un nouveau fichier gridd.conf dà l’emplacement etc/nvidia /
  
   ```bash
   sudo cp /etc/nvidia/gridd.conf.template /etc/nvidia/gridd.conf
   ```
  
8. Ajoutez la ligne suivante à `/etc/nvidia/gridd.conf` :
 
   ```
   IgnoreSP=FALSE
   EnableUI=FALSE 
   ```
9. Le cas échéant, supprimez ce qui suit de `/etc/nvidia/gridd.conf` :
 
   ```
   FeatureType=0
   ```
10. Redémarrez la machine virtuelle et vérifiez l’installation.


### <a name="verify-driver-installation"></a>Vérification de l’installation du pilote


Pour interroger l’état de l’appareil GPU, connectez-vous par SSH à la machine virtuelle et exécutez l’utilitaire de ligne de commande [nvidia-smi](https://developer.nvidia.com/nvidia-system-management-interface) installé avec le pilote. 

Si le pilote est installé, vous obtenez un résultat qui ressemble à celui indiqué. Notez que **GPU-Util** affiche 0 %, sauf si vous exécutez actuellement une charge de travail GPU sur la machine virtuelle. La version de votre pilote et vos détails de GPU peuvent différer de ceux indiqués.

![Capture d’écran montrant la sortie lors de l’interrogation de l’état de l’appareil GPU.](./media/n-series-driver-setup/smi-nv.png)
 

### <a name="x11-server"></a>Serveur X11
Si vous avez besoin d’un serveur X11 pour les connexions à distance à une machine virtuelle NV ou NVv2, [x11vnc](http://www.karlrunge.com/x11vnc/) est recommandé, car il permet l’accélération matérielle des graphiques. Le BusID de l’appareil M60 doit être ajouté manuellement au fichier de configuration de X11 (en règle générale, `etc/X11/xorg.conf`). Ajoutez une section `"Device"` similaire à la suivante :
 
```
Section "Device"
    Identifier     "Device0"
    Driver         "nvidia"
    VendorName     "NVIDIA Corporation"
    BoardName      "Tesla M60"
    BusID          "PCI:0@your-BusID:0:0"
EndSection
```
 
En outre, mettez à jour votre section `"Screen"` pour utiliser cet appareil.
 
Vous trouverez le BusID décimal en exécutant :

```bash
nvidia-xconfig --query-gpu-info | awk '/PCI BusID/{print $4}'
```
 
Le BusID peut changer lorsqu’une machine virtuelle est réaffectée ou redémarrée. Par conséquent, il peut être judicieux de créer un script pour mettre à jour le BusID dans la configuration X11 lors du redémarrage d’une machine virtuelle. Par exemple, créez un script nommé `busidupdate.sh` (ou un autre nom de votre choix) avec un contenu similaire à ce qui suit :

```bash 
#!/bin/bash
XCONFIG="/etc/X11/xorg.conf"
OLDBUSID=`awk '/BusID/{gsub(/"/, "", $2); print $2}' ${XCONFIG}`
NEWBUSID=`nvidia-xconfig --query-gpu-info | awk '/PCI BusID/{print $4}'`

if [[ "${OLDBUSID}" == "${NEWBUSID}" ]] ; then
        echo "NVIDIA BUSID not changed - nothing to do"
else
        echo "NVIDIA BUSID changed from \"${OLDBUSID}\" to \"${NEWBUSID}\": Updating ${XCONFIG}" 
        sed -e 's|BusID.*|BusID          '\"${NEWBUSID}\"'|' -i ${XCONFIG}
fi
```

Ensuite, créez une entrée pour votre script de mise à jour dans `/etc/rc.d/rc3.d` afin qu’il soit appelé en tant que racine au démarrage.

## <a name="troubleshooting"></a>Dépannage

* Vous pouvez définir le mode de persistance à l’aide de `nvidia-smi`. De cette façon, la sortie de la commande est plus rapide quand vous avez besoin d’effectuer une requête sur les cartes. Pour définir le mode de persistance, exécutez `nvidia-smi -pm 1`. Notez que si la machine virtuelle est redémarrée, le paramètre du mode n’est pas conservé. Vous pouvez toujours définir le paramètre du mode dans un script à exécuter au démarrage.
* Si vous avez mis à jour les pilotes NVIDIA CUDA vers la dernière version et constatez que le connectivité RDMA ne fonctionne plus, [réinstallez les pilotes RDMA](#rdma-network-connectivity) pour rétablir cette connectivité. 
* Si une certaine version du système d’exploitation CentOS/RHEL (ou du noyau) n’est pas prise en charge pour LIS, une erreur « Version de noyau non prise en charge » s’affiche. Signalez cette erreur en même temps que les versions du système d’exploitation et du noyau.

## <a name="next-steps"></a>Étapes suivantes

* Pour capturer une image Linux VM avec vos pilotes NVIDIA installés, consultez [Généraliser et capturer une machine virtuelle Linux](capture-image.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).
