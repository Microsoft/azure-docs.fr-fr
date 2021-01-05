---
title: Utilisation de Windows Virtual Desktop avec Azure NetApp Files | Microsoft Docs
description: Fournit des conseils sur les meilleures pratiques et des exemples de blueprints pour le déploiement de Windows Virtual Desktop avec Azure NetApp Files.
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
ms.date: 08/13/2020
ms.author: b-juche
ms.openlocfilehash: 0cd1f6210fbdb74e3fd511150157dccca3e92dda
ms.sourcegitcommit: 50802bffd56155f3b01bfb4ed009b70045131750
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91932462"
---
# <a name="benefits-of-using-azure-netapp-files-with-windows-virtual-desktop"></a>Avantages d’utiliser Azure NetApp Files avec Windows Virtual Desktop 

Cet article fournit des conseils sur les meilleures pratiques pour le déploiement de Windows Virtual Desktop (WVD) avec Azure NetApp Files.

Azure NetApp Files est un service de stockage de fichiers très performant proposé par Azure. Il peut fournir jusqu’à 450 000 IOPS et une latence inférieure à la milliseconde, et il est capable de prendre en charge des déploiements de Windows Virtual Desktop à très grande échelle. Vous pouvez ajuster la bande passante et modifier le niveau de service de vos volumes Azure NetApp Files à la demande et presque instantanément sans interrompre les E/S et tout en conservant l’accès au plan de données. Cette capacité vous permet d’optimiser facilement la mise à l’échelle de votre déploiement WVD en fonction du coût. Vous pouvez également créer des instantanés de volume à faible encombrement et à un moment donné sans perturber le niveau de performance du volume. Cette capacité vous permet de restaurer des [conteneurs de profil utilisateur FSLogix](../virtual-desktop/store-fslogix-profile.md) individuels via une copie du répertoire `~snapshot` ou de restaurer instantanément l’intégralité du volume en une fois via la capacité de restauration de volume.  Avec jusqu’à 255 instantanés (rotatifs) en place pour protéger un volume contre la perte ou l’altération des données, les administrateurs ont de nombreuses chances d’annuler ce qui a été fait.

## <a name="sample-blueprints"></a>Exemples de blueprints

Les exemples de blueprints suivants illustrent l’intégration de Windows Virtual Desktop à Azure NetApp Files. Dans un scénario de bureau groupé, les utilisateurs sont dirigés vers la meilleure session disponible (le [mode d’équilibrage de charge en largeur d’abord](../virtual-desktop/host-pool-load-balancing.md#breadth-first-load-balancing-method)) hébergée dans le pool, en utilisant des [machines virtuelles multisessions](../virtual-desktop/windows-10-multisession-faq.md#what-is-windows-10-enterprise-multi-session). En revanche, les bureaux personnels sont réservés aux scénarios dans lesquels chaque utilisateur dispose de sa propre machine virtuelle.

### <a name="pooled-desktop-scenario"></a>Scénario de bureau groupé

Pour le scénario groupé, l’équipe de Windows Virtual Desktop [recommande](/windows-server/remote/remote-desktop-services/virtual-machine-recs#multi-session-recommendations) les conseils suivants en ce qui concerne le nombre d’utilisateurs par processeur virtuel. Notez qu’aucune taille de machine virtuelle n’est spécifiée dans cette recommandation.

|     Type de charge de travail     |     Clair    |     Moyenne    |     Intensif    |
|-----------------------|--------------|---------------|--------------|
|     Utilisateurs par processeur virtuel    |     6        |     4         |     2        |


Cette recommandation est confirmée par un test LoginVSI de 500 utilisateurs, enregistrant environ 62 « utilisateurs confirmés/moyens » sur chaque machine virtuelle D16as_V4. 

Par exemple, à 62 utilisateurs par machine virtuelle D16as_V4, Azure NetApp Files peut facilement prendre en charge 60 000 utilisateurs par environnement. Le test d’évaluation de la limite supérieure de la machine virtuelle D32as_v4 est en cours. Si la recommandations relative au nombre d’utilisateurs de WVD par processeur virtuel s’applique à la D32as_v4, plus de 120 000 utilisateurs pourraient être répartis dans 1 000 machines virtuelles avant de dépasser [la limite du réseau virtuel des 1 000 adresses IP](./azure-netapp-files-network-topologies.md), comme illustré dans la figure suivante.  

![Windows Virtual Desktop Scénario de bureau groupé](../media/azure-netapp-files/solutions-pooled-desktop-scenario.png)   

### <a name="personal-desktop-scenario"></a>Scénario de bureau personnel 

Dans un scénario de bureau personnel, la figure ci-dessous illustre la recommandation architecturale à usage général. Les utilisateurs sont mappés à des pods de bureau spécifiques et chaque pod compte un peu moins de 1 000 machines virtuelles, ce qui laisse de la place pour les adresses IP se propageant à partir du réseau virtuel de gestion. Azure NetApp Files peut facilement traiter plus de 900 bureaux personnels par réseau virtuel de pools d’hôtes à session unique, le nombre réel de machines virtuelles étant égal à 1 000 moins le nombre d’hôtes de gestion trouvés dans le réseau virtuel du hub. Si vous avez besoin de bureaux personnels supplémentaires, il est facile d’ajouter d’autres pods (pools d’hôtes et réseaux virtuels), comme illustré dans la figure suivante. 

![Windows Virtual Desktop Scénario de bureau personnel](../media/azure-netapp-files/solutions-personal-desktop-scenario.png)  

Lors de la création d’une architecture basée sur des pods comme celle-ci, il est important d’affecter les utilisateurs au pod approprié lors de la connexion afin de s’assurer que les utilisateurs trouveront toujours leur profil utilisateur. 

## <a name="next-steps"></a>Étapes suivantes

- [Architectures de solution avec Azure NetApp Files](azure-netapp-files-solution-architectures.md)