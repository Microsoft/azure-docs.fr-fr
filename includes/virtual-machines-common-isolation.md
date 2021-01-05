---
title: Fichier include
description: Fichier include
services: virtual-machines
author: styli365
ms.service: virtual-machines
ms.topic: include
ms.date: 11/05/2020
ms.author: sttsinar
ms.custom: include file
ms.openlocfilehash: bd31152c6742271658cf4fd7ac2a5e2957651c84
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96477521"
---
Le Calcul Azure propose des tailles de machines virtuelles qui sont isolées pour un type de matériel spécifique et dédiées à un seul et même client. Les tailles isolées vivent et fonctionnent sur une génération de matériel spécifique et seront déconseillées lors de sa mise hors service.

Les tailles de machines virtuelles sont les mieux adaptées aux charges de travail qui nécessitent un niveau élevé d’isolation par rapport aux charges de travail des autres clients pour des raisons de conformité et d’exigences réglementaires.  L’utilisation d’une taille isolée garantit que votre machine virtuelle sera la seule en cours d’exécution sur cette instance de serveur spécifique. 


En outre, comme les machines virtuelles de taille isolée sont volumineuses, les clients peuvent choisir de subdiviser les ressources de ces machines virtuelles à l’aide de la [prise en charge des machines virtuelles imbriquées par Azure](https://azure.microsoft.com/blog/nested-virtualization-in-azure/).

Les offres actuelles de machines virtuelles isolées sont les suivantes :
* Standard_E64is_v3
* Standard_E64i_v3
* Standard_E80ids_v4
* Standard_E80is_v4
* Standard_M128ms
* Standard_GS5
* Standard_G5
* Standard_F72s_v2


> [!NOTE]
> Les tailles de machines virtuelles isolées ont une durée de vie matérielle limitée. Pour plus d’informations, voir ci-dessous.

## <a name="deprecation-of-isolated-vm-sizes"></a>Dépréciation des tailles de machines virtuelles isolées

Étant donné que les tailles de machines virtuelles isolées sont liées au matériel, Azure enverra des rappels 12 mois avant la dépréciation officielle des tailles.  Azure proposera également une taille isolée mise à jour sur la prochaine version matérielle sur laquelle le client pourrait envisager de déplacer sa charge de travail.

| Taille | Date de mise hors service de l’isolation | 
| --- | --- |
| Standard_DS15_v2<sup>1</sup> | 15 mai 2020 |
| Standard_D15_v2<sup>1</sup>  | 15 mai 2020 |

<sup>1</sup> Pour plus d’informations sur le programme de mise hors service de l’isolation Standard_DS15_v2 et Standard_D15_v2, consultez les FAQ.


## <a name="faq"></a>Questions fréquentes (FAQ)
### <a name="q-is-the-size-going-to-get-retired-or-only-isolation-feature-is"></a>Q : La taille sera-t-elle mise hors service ou cela concerne-t-il seulement la fonctionnalité « isolation » ?
**R** : Si la taille de la machine virtuelle ne porte pas l’indice « i », seule la fonctionnalité « isolation » sera supprimée. Si l’isolation n’est pas nécessaire, aucune action n’est requise et la machine virtuelle continue de fonctionner comme prévu. Exemples : Standard_DS15_v2, Standard_D15_v2, Standard_M128ms, etc. Si la taille de la machine virtuelle comprend l’indice « i », la taille sera bientôt mise hors service.

### <a name="q-is-there-a-downtime-when-my-vm-lands-on-a-non-isolated-hardware"></a>Q : Y a-t-il un temps d’arrêt lorsque ma machine virtuelle arrive sur un matériel non isolé ?
**R** : Si l’isolation n’est pas nécessaire, aucune action n’est requise et il n’y aura pas de temps d’arrêt.

### <a name="q-is-there-any-cost-delta-for-moving-to-a-non-isolated-virtual-machine"></a>Q : Existe-t-il un écart de coût pour le déplacement vers une machine virtuelle non isolée ?
**R** : Non

### <a name="q-when-are-the-other-isolated-sizes-going-to-retire"></a>Q : Quand les autres tailles isolées vont-elles être mises hors service ?
**R** : Nous enverrons des rappels 12 mois avant la dépréciation officielle de la taille isolée.

### <a name="q-im-an-azure-service-fabric-customer-relying-on-the-silver-or-gold-durability-tiers-does-this-change-impact-me"></a>Q : Je suis un client Azure Service Fabric qui s’appuie sur les niveaux de durabilité Argent ou Or. Ce changement a-t-il un impact sur moi ?
**R** : Non. Les garanties fournies par les [niveaux de durabilité](../articles/service-fabric/service-fabric-cluster-capacity.md#durability-characteristics-of-the-cluster) de Service Fabric continuent de fonctionner même après ce changement. Si vous avez besoin d’une isolation matérielle physique pour d’autres raisons, vous devrez peut-être effectuer l’une des actions décrites ci-dessus. 
 
### <a name="q-what-are-the-milestones-for-d15_v2-or-ds15_v2-isolation-retirement"></a>Q : Quelles sont les étapes majeures de la mise hors service de l’isolation D15_v2 ou DS15_v2 ? 
**R** : 
 
| Date | Action |
|---|---| 
| 18 novembre 2019 | Disponibilité de D/DS15i_v2 (paiement à l’utilisation, instance réservée 1 an) | 
| 14 mai 2020 | Dernier jour pour acheter l’instance réservée 1 an D/DS15i_v2 | 
| 15 mai 2020 | Garantie d’isolation D/DS15_v2 retirée | 
| 15 mai 2021 | Mise hors service de D/DS15i_v2 (tous les clients, sauf ceux qui ont acheté une instance réservée 3 ans de D/DS15_v2 avant le 18 novembre 2019)| 
| 17 novembre 2022 | Mise hors service de D/DS15i_v2 quand les instances réservées 3 ans arrivent à leur terme (pour les clients qui ont acheté une instance réservée 3 ans de D/DS15_v2 avant le 18 novembre 2019) |

## <a name="next-steps"></a>Étapes suivantes

Les clients peuvent également choisir de subdiviser les ressources de ces machines virtuelles isolées à l’aide de la [prise en charge d’Azure pour les machines virtuelles imbriquées](https://azure.microsoft.com/blog/nested-virtualization-in-azure/).
