---
author: cynthn
ms.service: virtual-machines
ms.topic: include
ms.date: 10/26/2018
ms.author: cynthn
ms.openlocfilehash: d41b86b902d9a58b144e251e6922fbd95d459031
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96016122"
---
1. Installer dapl, rdmacm, ibverbs et mlx4

   ```bash
   sudo apt-get update

   sudo apt-get install libdapl2 libmlx4-1

   ```

2. Dans /etc/waagent.conf, activez RDMA en supprimant les marques de commentaire dans les lignes de configuration suivantes. Un accès racine est requis pour modifier ce fichier.
  
   ```
   OS.EnableRDMA=y

   OS.UpdateRdmaDriver=y
   ```

3. Ajoutez ou modifiez les paramètres de mémoire suivants en Ko dans le fichier /etc/security/limits.conf. Un accès racine est requis pour modifier ce fichier. À des fins de test, vous pouvez définir pour memlock une valeur illimitée. Par exemple : `<User or group name>   hard    memlock   unlimited`.

   ```
   <User or group name> hard    memlock <memory required for your application in KB>

   <User or group name> soft    memlock <memory required for your application in KB>
   ```
  
4. Installez la bibliothèque Intel MPI. [Achetez et téléchargez](https://software.intel.com/intel-mpi-library/) la bibliothèque auprès d’Intel ou téléchargez la [version d’évaluation gratuite](https://registrationcenter.intel.com/en/forms/?productid=1740).

   ```bash
   wget http://registrationcenter-download.intel.com/akdlm/irc_nas/tec/9278/l_mpi_p_5.1.3.223.tgz
   ```
 
   Seuls les runtimes Intel MPI 5.x sont pris en charge.
 
   Pour les étapes d’installation, consultez le [Guide d’installation de la bibliothèque Intel MPI](https://registrationcenter-download.intel.com/akdlm/irc_nas/1718/INSTALL.html?lang=en&fileExt=.html).

5. Activez ptrace pour les processus non-racine et non-débogueur (nécessaire pour les versions les plus récentes d’Intel MPI).
 
   ```bash
   echo 0 | sudo tee /proc/sys/kernel/yama/ptrace_scope
   ```
