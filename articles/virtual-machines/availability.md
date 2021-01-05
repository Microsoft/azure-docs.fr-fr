---
title: Options de disponibilité
description: En savoir plus sur les fonctionnalités de disponibilité pour l’exécution de machines virtuelles dans Azure
author: cynthn
ms.author: cynthn
ms.service: virtual-machines
ms.topic: conceptual
ms.date: 05/10/2019
ms.openlocfilehash: a0ea9a0871dd57088473999b28553258ff210038
ms.sourcegitcommit: 9826fb9575dcc1d49f16dd8c7794c7b471bd3109
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2020
ms.locfileid: "94628762"
---
# <a name="availability-options-for-virtual-machines-in-azure"></a>Options de disponibilité pour les machines virtuelles dans Azure

Cet article fournit une vue d’ensemble des fonctionnalités de disponibilité des machines virtuelles Azure.

## <a name="high-availability"></a>Haute disponibilité

Les charges de travail sont généralement réparties sur différentes machines virtuelles afin d’obtenir un débit et des performances élevées, et pour créer une redondance au cas où une machine virtuelle serait affectée en raison d’une mise à jour ou d’un autre événement. 

Azure propose de nombreuses options pour obtenir une haute disponibilité. Commençons par parler des constructions de base. 

### <a name="availability-zones"></a>Zones de disponibilité

Les [zones de disponibilité](../availability-zones/az-overview.md) élargissent le niveau de contrôle dont vous disposez pour maintenir la disponibilité des applications et des données sur vos machines virtuelles. Une zone de disponibilité est une zone physiquement séparée au sein d’une région Azure. On distingue trois zones de disponibilité par région Azure prise en charge. 

Chaque zone de disponibilité possède une source d’alimentation, un réseau et un système de refroidissement propres. En concevant vos solutions pour répliquer des machines virtuelles dans des zones, vous pouvez protéger vos applications et vos données de la perte d’un centre de données. Si une zone est compromise, les applications et données répliquées sont instantanément rendues disponibles dans une autre zone. 

![Zones de disponibilité](./media/virtual-machines-common-regions-and-availability/three-zones-per-region.png)

Apprenez-en davantage sur le déploiement d’une machine virtuelle [Windows](./windows/create-powershell-availability-zone.md) ou [Linux](./linux/create-cli-availability-zone.md) dans une zone de disponibilité.


### <a name="fault-domains"></a>Domaines d’erreur

Un domaine d’erreur est un groupe logique de matériels sous-jacents qui partagent la même source d’alimentation et le même commutateur réseau, par exemple un rack dans un centre de données local. 

### <a name="update-domains"></a>Domaines de mise à jour

Un domaine de mise à jour est un groupe logique de matériels sous-jacents qui peuvent faire l’objet d’une opération de maintenance ou être redémarrés en même temps. 

Cette approche garantit qu’au moins une instance de votre application reste toujours en cours d’exécution, car la plateforme Azure fait l’objet d’une maintenance périodique. Le redémarrage des domaines de mise à jour peut ne pas suivre un ordre séquentiel au cours de la maintenance, mais un seul domaine de mise à jour peut être redémarré à la fois.


## <a name="virtual-machines-scale-sets"></a>Groupes de machines virtuelles identiques 

Les groupes identiques de machines virtuelles Azure vous permettent de créer et de gérer un groupe de machines virtuelles et disposant d’une charge équilibrée. Le nombre d’instances de machine virtuelle peut augmenter ou diminuer automatiquement en fonction d’une demande ou d’un calendrier défini. Les groupes identiques offrent une haute disponibilité à vos applications, et vous permettent de gérer, configurer et mettre à jour de manière centralisée de nombreuses machines virtuelles. Il est recommandé de créer au moins deux machines virtuelles dans un groupe identique, de manière à fournir une application hautement disponible et à répondre aux exigences du [niveau de 99,95 % inscrit dans les contrats de niveau de service Azure](https://azure.microsoft.com/support/legal/sla/virtual-machines/). Le groupe identique proprement dit ne vous coûte rien ; vous payez uniquement pour chaque instance de machine virtuelle que vous créez. Lorsqu’une seule machine virtuelle utilise des [disques SSD Azure Premium](./disks-types.md#premium-ssd), le contrat SLA Azure s’applique pour les événements de maintenance non planifiés. Les machines virtuelles d’un groupe identique peuvent être déployées sur plusieurs domaines de mise à jour et domaines d’erreur pour optimiser la disponibilité et la résilience aux pannes dues à des pannes de centre de données ou à des événements de maintenance planifiés ou non. Les machines virtuelles d’un groupe identique peuvent également être déployées dans une seule zone de disponibilité, ou dans une région. Les options de déploiement de la zone de disponibilité peuvent varier en fonction du mode d’orchestration.

**Domaines d’erreur et domaines de mise à jour**

Les groupes de machines virtuelles identiques simplifient la conception pour la haute disponibilité en alignant les domaines d’erreur et les domaines de mise à jour. Vous avez uniquement à définir le nombre de domaines d’erreur pour le groupe identique. Le nombre de domaines d’erreur disponibles pour les groupes identiques peut varier selon la région. Consultez [Gérer la disponibilité des machines virtuelles dans Azure](./manage-availability.md).


## <a name="availability-sets"></a>Groupes à haute disponibilité
Un groupe à haute disponibilité est un regroupement logique de machines virtuelles qui permet à Azure de comprendre comment votre application est conçue, afin de garantir la redondance et la disponibilité. Il est recommandé de créer au moins deux machines virtuelles dans un groupe à haute disponibilité, de manière à fournir une application hautement disponible et à répondre aux exigences du [niveau de 99,95 % inscrit dans les contrats de niveau de service Azure](https://azure.microsoft.com/support/legal/sla/virtual-machines/). Le groupe à haute disponibilité proprement dit ne vous coûte rien ; vous payez uniquement pour chaque instance de machine virtuelle que vous créez. Lorsqu’une seule machine virtuelle utilise des [disques SSD Azure Premium](./disks-types.md#premium-ssd), le contrat SLA Azure s’applique pour les événements de maintenance non planifiés.

Dans un groupe à haute disponibilité, les machines virtuelles sont automatiquement réparties sur ces domaines d’erreur. Cette approche limite l’impact des défaillances du matériel physique, des pannes de réseau ou des coupures de courant susceptibles de survenir.

Les machines virtuelles faisant appel à [Azure Disques managés](./faq-for-disks.md) sont alignées sur les domaines d’erreur des disques managés lorsqu’un groupe à haute disponibilité géré est utilisé. Cet alignement garantit que tous les disques managés attachés à une machine virtuelle se trouvent dans le même domaine d’erreur de disques managés. 

Seules des machines virtuelles avec des disques managés peuvent être créées dans un groupe à haute disponibilité géré. Le nombre de domaines d’erreur de disques gérés varie en fonction de la région (deux ou trois par région). Vous pouvez en savoir plus sur ces domaines d’erreur de disque géré pour [les machines virtuelles Linux](./manage-availability.md#use-managed-disks-for-vms-in-an-availability-set) ou [les machines virtuelles Windows](./manage-availability.md#use-managed-disks-for-vms-in-an-availability-set).

![Groupe à haute disponibilité géré](./media/virtual-machines-common-manage-availability/md-fd-updated.png)


Les machines virtuelles d’un groupe à haute disponibilité sont également automatiquement réparties entre les domaines de mise à jour. 

![Groupes à haute disponibilité](./media/virtual-machines-common-manage-availability/ud-fd-configuration.png)

## <a name="next-steps"></a>Étapes suivantes
Vous pouvez maintenant commencer à utiliser ces fonctionnalités de gestion de la redondance et de la disponibilité pour créer votre environnement Azure. Pour en savoir plus sur les meilleures pratiques, consultez la rubrique relative aux [meilleures pratiques Azure en matière de disponibilité](/azure/architecture/checklist/resiliency-per-service).
