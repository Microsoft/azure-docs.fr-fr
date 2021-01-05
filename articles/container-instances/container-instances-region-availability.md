---
title: Disponibilité des ressources par région
description: Disponibilité des ressources de calcul et de mémoire pour le service Azure Container Instances dans différentes régions Azure.
ms.topic: article
ms.date: 04/27/2020
ms.custom: references_regions
ms.openlocfilehash: 8651a1cc2c2ba41e2c28e9f63dc78c091a6af61d
ms.sourcegitcommit: d6e92295e1f161a547da33999ad66c94cf334563
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96763891"
---
# <a name="resource-availability-for-azure-container-instances-in-azure-regions"></a>Disponibilité des ressources pour Azure Container Instances dans les régions Azure

Cet article décrit en détail la disponibilité des ressources de calcul, de mémoire et de stockage d’Azure Container Instances dans les régions Azure et du système d’exploitation cible. Pour obtenir la liste générale des régions disponibles pour Azure Container Instances, consultez [Régions disponibles](https://azure.microsoft.com/regions/services/).

Les valeurs présentées sont les ressources maximales disponibles par déploiement d’un [groupe de conteneurs](container-instances-container-groups.md). Les valeurs sont à jour au moment de la publication.

> [!NOTE]
> Les groupes de conteneurs créés dans les limites de ces ressources sont soumis à la disponibilité dans la région de déploiement. Quand une région a une charge importante, vous pouvez rencontrer un échec durant le déploiement des instances. Pour atténuer ce type d'échec de déploiement, essayez de déployer des instances comportant des paramètres de ressources inférieurs, ou bien retentez le déploiement ultérieurement ou dans une autre région avec les ressources disponibles.

Pour plus d’informations sur les quotas et autres limites de vos déploiements, voir [Quotas et limites pour Azure Container Instances](container-instances-quotas.md).

## <a name="linux-container-groups"></a>Groupes de conteneurs Linux

Les régions et les ressources maximales suivantes sont disponibles pour les groupes de conteneurs avec des conteneurs Linux dans des déploiements généraux, déploiements de [réseau virtuel Azure](container-instances-vnet.md) et déploiements avec des [ressources GPU](container-instances-gpu.md) (préversion).

> [!IMPORTANT]
> Le nombre maximal de ressources dans une région est différent selon votre déploiement. Par exemple, une région peut avoir une taille de processeur et de mémoire maximale différente dans un déploiement de réseau virtuel Azure par rapport à un déploiement général. Cette même région peut également avoir un ensemble différent de valeurs maximales pour un déploiement avec des ressources GPU. Vérifiez votre type de déploiement avant de consulter les tableaux ci-dessous pour obtenir les valeurs maximales de votre région.

| Région | Utilisation maximale du processeur | Mémoire max. (GB) | Utilisation maximale du processeur du réseau virtuel | Mémoire maximale du réseau virtuel (Go) | Stockage (Go) | Références SKU de GPU (préversion) |
| -------- | :---: | :---: | :----: | :-----: | :-------: | :----: |
| Australie Est | 4 | 16 | 4 | 16 | 50 | N/A |
| Brésil Sud | 4 | 16 | 2 | 8 | 50 | N/A |
| Centre du Canada | 4 | 16 | 4 | 16 | 50 | N/A |
| Inde centrale | 4 | 16 | N/A | N/A | 50 | V100 |
| USA Centre | 4 | 16 | 4 | 16 | 50 | N/A |
| Asie Est | 4 | 16 | 4 | 16 | 50 | N/A |
| USA Est | 4 | 16 | 4 | 16 | 50 | K80, P100, V100 |
| USA Est 2 | 4 | 16 | 4 | 16 | 50 | N/A |
| France Centre | 4 | 16 | 4 | 16 | 50 | N/A |
| Japon Est | 2 | 8 | 4 | 16 | 50 | N/A |
| Centre de la Corée | 4 | 16 | N/A | N/A | 50 | N/A |
| Centre-Nord des États-Unis | 2 | 3,5 | 4 | 16 | 50 | K80, P100, V100 |
| Europe Nord | 4 | 16 | 4 | 16 | 50 | K80 |
| États-Unis - partie centrale méridionale | 4 | 16 | 4 | 16 | 50 | N/A |
| Asie Sud-Est | 4 | 16 | 4 | 16 | 50 | P100, V100 |
| Inde Sud | 4 | 16 | N/A | N/A | 50 | N/A |
| Sud du Royaume-Uni | 4 | 16 | 4 | 16 | 50 | N/A |
| Centre-USA Ouest| 4 | 16 | 4 | 16 | 50 | N/A |
| Europe Ouest | 4 | 16 | 4 | 16 | 50 | K80, P100, V100 |
| USA Ouest | 4 | 16 | 4 | 16 | 50 | N/A |
| USA Ouest 2 | 4 | 16 | 4 | 16 | 50 | K80, P100, V100 |

Les ressources maximales suivantes sont accessibles à un groupe de conteneurs déployé avec [Ressources GPU](container-instances-gpu.md) (préversion).

> [!IMPORTANT]
> À ce stade, les déploiements avec des ressources GPU ne sont pas pris en charge dans le déploiement d’un réseau virtuel Azure et ne sont disponibles que sur les groupes de conteneurs Linux.

| Références SKU de GPU | Nombre de GPU | Utilisation maximale du processeur | Mémoire max. (GB) | Stockage (Go) |
| --- | --- | --- | --- | --- |
| K80 | 1 | 6 | 56 | 50 |
| K80 | 2 | 12 | 112 | 50 |
| K80 | 4 | 24 | 224 | 50 |
| P100, V100 | 1 | 6 | 112 | 50 |
| P100, V100 | 2 | 12 | 224 | 50 |
| P100, V100 | 4 | 24 | 448 | 50 |

## <a name="windows-container-groups"></a>Groupes de conteneurs Windows

Les régions et les ressources maximales suivantes sont accessibles aux groupes de conteneurs Windows Server 2019 (préversion) avec les conteneurs Windows Server [pris en charge ou en préversion](container-instances-faq.md#what-windows-base-os-images-are-supported).

> [!IMPORTANT]
> À ce stade, les déploiements avec des groupes de conteneurs Windows ne sont pas pris en charge dans le déploiement d’un réseau virtuel Azure.

###  <a name="windows-server-2016"></a>Windows Server 2016

> [!NOTE]
> Pour plus d’informations sur les hôtes 1B, 2B et 3B, consultez [Compatibilité des versions d’hôte et de conteneur](/virtualization/windowscontainers/deploy-containers/update-containers#host-and-container-version-compatibility).

| Région | Utilisation maximale de l’UC pour 1B/2B | Mémoire maximale (Go) pour 1B/2B |Utilisation maximale de l’UC pour 3B | Mémoire maximale (Go) pour 3B | Stockage (Go) |
| -------- | :---: | :---: | :----: | :-----: | :-------: |
| Australie Est | 2 | 8 | 2 | 8 | 20 |
| Brésil Sud | 4 | 16 | 4 | 16 | 20 |
| Centre du Canada | 2 | 8 | 2 | 3,5 | 20 |
| Inde centrale | 2 | 3,5 | 2 | 3,5 | 20 |
| USA Centre | 2 | 3,5 | 2 | 3,5 | 20 |
| Asie Est | 2 | 3,5 | 2 | 3,5 | 20 |
| USA Est | 4 | 16 | 2 | 8 | 20 |
| USA Est 2 | 2 | 3,5 | 4 | 16 | 20 |
| Japon Est | 4 | 16 | 4 | 16 | 20 |
| Centre de la Corée | 4 | 16 | 4 | 16 | 20 |
| Centre-Nord des États-Unis | 4 | 16 | 4 | 16 | 20 |
| Europe Nord | 2 | 8 | 2 | 8 | 20 |
| États-Unis - partie centrale méridionale | 2 | 3,5 | 2 | 8 | 20 |
| Asie Sud-Est | N/A | N/A | 2 | 3,5 | 20 |
| Inde Sud | 2 | 3,5 | 2 | 3,5 | 20 |
| Sud du Royaume-Uni | 2 | 8 | 2 | 3,5 | 20 |
| Centre-USA Ouest | 4 | 16 | 2 | 8 | 20 |
| Europe Ouest | 4 | 16 | 4 | 16 | 20 |
| USA Ouest | 4 | 16 | 2 | 8 | 20 |
| USA Ouest 2 | 2 | 8 | 2 | 3,5 | 20 |


### <a name="windows-server-2019-ltsc"></a>Windows Server 2019 LTSC

> [!NOTE]
> Pour plus d’informations sur les hôtes 1B, 2B et 3B, consultez [Compatibilité des versions d’hôte et de conteneur](/virtualization/windowscontainers/deploy-containers/update-containers#host-and-container-version-compatibility).

| Région | Utilisation maximale de l’UC pour 1B/2B | Mémoire maximale (Go) pour 1B/2B |Utilisation maximale de l’UC pour 3B | Mémoire maximale (Go) pour 3B | Stockage (Go) |
| -------- | :---: | :---: | :----: | :-----: | :-------: |
| Australie Est | 4 | 16 | 4 | 16 | 20 |
| Brésil Sud | 4 | 16 | 4 | 16 | 20 |
| Centre du Canada | 4 | 16 | 4 | 16 | 20 |
| Inde centrale | 4 | 16 | 4 | 16 | 20 |
| USA Centre | 4 | 16 | 4 | 16 | 20 |
| Asie Est | 4 | 16 | 4 | 16 | 20 |
| USA Est | 4 | 16 | 4 | 16 | 20 |
| USA Est 2 | 2 | 3,5 | 2 | 3,5 | 20 |
| France Centre | 4 | 16 | 4 | 16 | 20 |
| Japon Est | N/A | N/A | 4 | 16 | 20 |
| Centre de la Corée | 4 | 16 | 4 | 16 | 20 |
| Centre-Nord des États-Unis | 4 | 16 | 4 | 16 | 20 |
| Europe Nord | 4 | 16 | 4 | 16 | 20 |
| États-Unis - partie centrale méridionale | 4 | 16 | 4 | 16 | 20 |
| Asie Sud-Est | 4 | 16 | 4 | 16 | 20 |
| Inde Sud | 4 | 16 | 4 | 16 | 20 |
| Sud du Royaume-Uni | 4 | 16 | 4 | 16 | 20 |
| Centre-USA Ouest | 4 | 16 | 4 | 16 | 20 |
| Europe Ouest | 4 | 16 | 4 | 16 | 20 |
| USA Ouest | 4 | 16 | 4 | 16 | 20 |
| USA Ouest 2 | 2 | 8 | 4 | 16 | 20 |

## <a name="next-steps"></a>Étapes suivantes

Informez l’équipe si vous souhaitez voir des régions supplémentaires ou une disponibilité accrue des ressources sur le site [aka.ms/aci/feedback](https://aka.ms/aci/feedback).

Pour plus d’informations sur la résolution des problèmes de déploiement d’instances de conteneur, consultez [Résoudre les problèmes de déploiement avec Azure Container Instances](container-instances-troubleshooting.md).


[azure-support]: https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest
