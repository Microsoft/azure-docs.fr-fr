---
title: Benchmarks de performances Azure NetApp Files pour Linux | Microsoft Docs
description: Décrit les benchmarks de performances relatifs à Azure NetApp Files pour Linux.
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 04/29/2020
ms.author: b-juche
ms.openlocfilehash: b763a734866dd5fed5bf0500d4d52b9324c92a79
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "82614010"
---
# <a name="azure-netapp-files-performance-benchmarks-for-linux"></a>Benchmarks de performances Azure NetApp Files pour Linux

Cet article décrit les benchmarks de performances relatifs à Azure NetApp Files pour Linux.

## <a name="linux-scale-out"></a>Scale-out Linux

Cette section décrit les benchmarks de performances du débit de charge de travail Linux et des IOPS de charge de travail.

### <a name="linux-workload-throughput"></a>Débit de la charge de travail Linux  

Le graphique ci-dessous représente une charge de travail séquentielle de 64 Kio et une plage de travail de 1 Tio. Il indique qu’un seul volume d’Azure NetApp Files peut gérer entre environ 1 600 Mio/s de pures écritures séquentielles et environ 4 500 Mio/s de pures lectures séquentielles.  

Le graphique illustre des diminutions de 10 % à la fois, de la lecture pure à l’écriture pure. Il montre ce que vous pouvez attendre quand vous utilisez des ratios lecture/écriture variables (100%:0%, 90%:10%, 80%:20%, etc.).

![Débit de la charge de travail Linux](../media/azure-netapp-files/performance-benchmarks-linux-workload-throughput.png)  

### <a name="linux-workload-iops"></a>IOPS de la charge de travail Linux  

Le graphique ci-dessous représente une charge de travail aléatoire de 4 Kio et une plage de travail de 1 Tio. Le graphique montre qu’un volume d’Azure NetApp Files peut gérer entre 130 000 écritures aléatoires pures et 460 000 lectures aléatoires pures.  

Ce graphique illustre des diminutions de 10 % à la fois, de la lecture pure à l’écriture pure. Il montre ce que vous pouvez attendre quand vous utilisez des ratios lecture/écriture variables (100%:0%, 90%:10%, 80%:20%, etc.).

![IOPS de la charge de travail Linux](../media/azure-netapp-files/performance-benchmarks-linux-workload-iops.png)  

## <a name="linux-scale-up"></a>Scale-up Linux  

Le noyau Linux 5.3 permet la mise en réseau avec scale-out à un seul client pour NFS-`nconnect`. Les graphiques de cette section montrent les résultats des tests de validation de l’option de montage côté client avec NFSv3. La fonctionnalité est disponible sur SUSE (à partir de SLES12SP4) et Ubuntu (à partir de la version 19.10). C’est un concept similaire à la fois pour SMB Multichannel et Oracle Direct NFS.

Les graphiques comparent les avantages de `nconnect` par rapport à un volume monté non connecté. Dans les graphiques, FIO a généré la charge de travail à partir d’une instance D32s_v3 unique dans la région Azure us-west2.

### <a name="linux-read-throughput"></a>Débit de lecture Linux  

Les graphiques suivants montrent des lectures séquentielles d’environ 3 500 Mio/s de lectures avec `nconnect`, approximativement 2,3 fois plus qu’avec l’option sans `nconnect`.

![Débit de lecture Linux](../media/azure-netapp-files/performance-benchmarks-linux-read-throughput.png)  

### <a name="linux-write-throughput"></a>Débit d’écriture Linux  

Les graphiques suivants montrent des écritures séquentielles. Ils indiquent que `nconnect` n’a pas d’avantage notable pour les écritures séquentielles. 1 500 Mio/s correspond à peu près à la limite supérieure du volume en écriture séquentielle et à la limite de sortie de l’instance D32s_v3.

![Débit d’écriture Linux](../media/azure-netapp-files/performance-benchmarks-linux-write-throughput.png)  

### <a name="linux-read-iops"></a>E/S de lecture Linux  

Les graphiques suivants montrent des lectures aléatoires d’environ 200 000 IOPS de lecture avec `nconnect`, environ 3 fois plus qu’avec l’option sans `nconnect`.

![IOPS de lecture Linux](../media/azure-netapp-files/performance-benchmarks-linux-read-iops.png)  

### <a name="linux-write-iops"></a>IOPS d’écriture Linux  

Les graphiques suivants montrent des écritures aléatoires d’environ 135 000 IOPS d’écriture avec `nconnect`, environ 3 fois plus qu’avec l’option sans `nconnect`.

![IOPS d’écriture Linux](../media/azure-netapp-files/performance-benchmarks-linux-write-iops.png)  

## <a name="next-steps"></a>Étapes suivantes

- [Azure NetApp Files : Tirer le meilleur parti de votre stockage cloud](https://cloud.netapp.com/hubfs/Resources/ANF%20PERFORMANCE%20TESTING%20IN%20TEMPLATE.pdf?hsCtaTracking=f2f560e9-9d13-4814-852d-cfc9bf736c6a%7C764e9d9c-9e6b-4549-97ec-af930247f22f)
