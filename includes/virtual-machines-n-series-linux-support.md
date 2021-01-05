---
title: Fichier include
description: Fichier include
services: virtual-machines-linux
author: cynthn
ms.service: virtual-machines-linux
ms.topic: include
ms.date: 02/11/2019
ms.author: cynthn
ms.custom: include file
ms.openlocfilehash: 5bfe077a99a3e89efe5fe4257a646484fb3810da
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96537021"
---
## <a name="supported-distributions-and-drivers"></a>Distributions et pilotes pris en charge

### <a name="nvidia-cuda-drivers"></a>Pilotes CUDA NVIDIA

Les pilotes CUDA NVIDIA pour les machines virtuelles des séries NC, NCv2, NCv3, ND et NDv2 (facultatif pour la série NV) sont pris en charge uniquement dans les distributions Linux figurant dans le tableau suivant. Les informations relatives aux pilotes CUDA sont à jour au moment de la publication. Pour les pilotes CUDA les plus récents et les systèmes d’exploitation pris en charge, rendez-vous sur le site web [NVIDIA](https://developer.nvidia.com/cuda-zone). Assurez-vous que vous installez ou mettez à niveau en utilisant les derniers pilotes CUDA pour votre distribution. 

> [!TIP]
> Vous pouvez déployez une image de [machine virtuelle de science des données](../articles/machine-learning/data-science-virtual-machine/overview.md) Azure, comme alternative à une installation manuelle du pilote CUDA sur une machine virtuelle Linux. Les éditions des machines virtuelles Science des données pour Ubuntu 16.04 LTS ou CentOS 7.4 préinstallent les pilotes NVIDIA CUDA, la bibliothèque de réseau neuronal profond et d’autres outils.


### <a name="nvidia-grid-drivers"></a>Pilotes NVIDIA GRID

Microsoft redistribue les programmes d’installation du pilote GRID NVIDIA pour les machines virtuelles de série NV et NVv3 utilisées comme stations de travail virtuelles ou pour les applications virtuelles. Installez uniquement ces pilotes GRID sur des machines virtuelles Azure NV, et uniquement sur les systèmes d’exploitation répertoriés dans le tableau suivant. Ces pilotes incluent les licences des logiciels GRID Virtual GPU dans Azure. Vous n’avez pas besoin de configurer un serveur de licences logicielles NVIDIA vGPU.

Les pilotes GRID redistribués par Azure ne fonctionnent pas sur les machines virtuelles de série autre que NV, comme les machines virtuelles de série NC, NCv2, NCv3, ND et NDv2.

|Distribution|Pilote|
| --- | -- |
|Ubuntu 18.04 LTS<br/><br/>Ubuntu 16.04 LTS<br/><br/>Red Hat Enterprise Linux 7.6 à 7.8, 8.0, 8.1<br/><br/>CentOS 7.6, 7.7, 8.0, 8(1911)<br/><br/>SUSE Linux Enterprise Server 12 SP2 | NVIDIA GRID 11.2, branche pilote [R450](https://go.microsoft.com/fwlink/?linkid=874272) (.exe)|

Visitez [GitHub](https://github.com/Azure/azhpc-extensions/blob/master/NvidiaGPU/resources.json) pour obtenir la liste complète de tous les liens précédents du pilote Nvidia GRID.

> [!WARNING] 
> L’installation de logiciels tiers sur des produits Red Hat peut affecter les conditions de prise en charge de Red Hat. Consultez l’[article de la Base de connaissances Red Hat](https://access.redhat.com/articles/1067).
>
