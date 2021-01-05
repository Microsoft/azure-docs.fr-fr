---
title: Remises des plans de réservation Red Hat - Azure
description: Découvrez comment les remises des plans Red Hat sont appliquées aux logiciels Red Hat sur les machines virtuelles.
author: yashesvi
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: conceptual
ms.date: 06/11/2020
ms.author: banders
ms.openlocfilehash: 8e735d623cc991d2e3720e0ec418eee77aa8c46c
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96350855"
---
# <a name="understand-how-the-red-hat-linux-enterprise-software-reservation-plan-discount-is-applied-for-azure"></a>Comprendre l'application des remises aux plans de réservation de logiciels Red Hat Linux Enterprise pour Azure

Lorsque vous achetez un plan Red Hat Linux, la remise est automatiquement appliquée aux machines virtuelles Red Hat correspondant à la réservation. Un plan Red Hat Linux couvre le coût d'exécution du logiciel Red Hat sur une machine virtuelle Azure.

Pour acheter le plan Red Hat Linux approprié, vous devez savoir quelles machines virtuelles Red Hat vous exécutez et combien de processeurs virtuels elles contiennent. Utilisez les sections suivantes pour vous aider à identifier l’abonnement à acheter à partir de votre fichier CSV d’utilisation.

## <a name="discount-applies-to-different-vm-sizes"></a>La remise s'applique à différentes tailles de machines virtuelles

Comme pour les instances de machines virtuelles réservées, l'achat d'un plan Red Hat permet de bénéficier d'une certaine flexibilité en termes de taille d'instance. Cela signifie que votre remise s’applique même quand vous déployez une machine virtuelle avec un nombre différent de processeurs virtuels. La remise s’applique aux différentes tailles de machine virtuelle au sein de l’abonnement logiciel.

Le montant de remise dépend du ratio indiqué dans les tableaux suivants. Le ratio évalue l’encombrement relatif de chaque taille de machine virtuelle du groupe. Le ratio dépend des processeurs virtuels de la machine virtuelle. Utilisez la valeur du ratio pour calculer le nombre d'instances de machines virtuelles qui bénéficient de la remise du plan Red Hat Linux.

Par exemple, si vous achetez un plan Red Hat Linux Enterprise Server pour une machine virtuelle dotée de 1 à 4 processeurs virtuels, le ratio de cette réservation est de 1. La remise couvre le coût du logiciel Red Hat pour les configurations suivantes :

- 1 machine virtuelle déployée dotée de 1 à 4 processeurs virtuels,
- ou 0,46 (soit environ 46 %) des coûts de Red Hat Enterprise Linux pour une machine virtuelle dotée d’au moins 5 processeurs virtuels.

### <a name="red-hat-enterprise-linux"></a>Red Hat Enterprise Linux

Noms dans la place de marché du portail Azure :
- Red Hat Enterprise Linux 6.7
- Red Hat Enterprise Linux 6.8
- Red Hat Enterprise Linux 6.9
- Red Hat Enterprise Linux 6.10
- Red Hat Enterprise Linux 7
- Red Hat Enterprise Linux 7.2
- Red Hat Enterprise Linux 7.3
- Red Hat Enterprise Linux 7.4
- Red Hat Enterprise Linux 7.5
- Red Hat Enterprise Linux 7.6
- Red Hat Enterprise Linux 8.2

[Vérifier les compteurs Red Hat Enterprise Linux auxquels le plan s’applique](https://isfratio.blob.core.windows.net/isfratio/RHELRatios.csv)

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les réservations, consultez les articles suivants :

- [Qu’est-ce qu’une réservation Azure ?](save-compute-costs-reservations.md)
- [Prépayer des plans logiciels Red Hat avec réservations Azure](../../virtual-machines/linux/prepay-suse-software-charges.md)
- [Prépayer des machines virtuelles avec des instances de machines virtuelles réservées Azure](../../virtual-machines/prepay-reserved-vm-instances.md)
- [Gérer les réservations pour Azure](manage-reserved-vm-instance.md)
- [Comprendre l’utilisation d’une réservation pour votre abonnement avec paiement à l’utilisation](understand-reserved-instance-usage.md)
- [Comprendre l’utilisation d’une réservation pour votre Accord de Mise en Œuvre Entreprise](understand-reserved-instance-usage-ea.md)

## <a name="need-help-contact-us"></a>Vous avez besoin d’aide ? Nous contacter

Si vous avez des questions ou besoin d’aide, [créez une demande de support](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).