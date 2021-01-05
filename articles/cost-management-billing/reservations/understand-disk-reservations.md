---
title: Comprendre la remise de réservation pour le stockage sur disque Azure
description: Découvrez comment une remise sur les disques réservés Azure s’applique à vos disques managés SSD Azure Premium.
author: roygara
ms.author: rogarana
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: conceptual
ms.date: 01/30/2020
ms.openlocfilehash: cd89c3ddc3c58de02f1104109ce7f243c4d1e6fd
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/20/2020
ms.locfileid: "88682448"
---
# <a name="understand-how-your-reservation-discount-is-applied-to-azure-disk-storage"></a>Comprendre comment votre remise de réservation s’applique au stockage disque Azure

Une fois que vous avez acheté une capacité réservée de disque Azure, une remise de réservation s’applique automatiquement aux ressources de disque qui correspondent aux conditions de votre réservation. La remise de réservation s’applique aux références SKU de disque uniquement. Les captures instantanées de disque sont facturées aux tarifs du paiement à l’utilisation.

Pour plus d’informations sur la réservation de disque Azure, consultez [Réduire les coûts avec la réservation de disque Azure](../../virtual-machines/disks-reserved-capacity.md). Pour plus d’informations sur les tarifs de la réservation de disque Azure, consultez [Tarifs Disque managés Azure](https://azure.microsoft.com/pricing/details/managed-disks/).

## <a name="how-the-reservation-discount-is-applied"></a>Comment la remise de réservation est-elle appliquée ?

Par principe, la remise de réservation de disque Azure qui n’est pas utilisée est perdue. Elle s’applique aux ressources de disque managé toutes les heures. Pour une heure donnée, si vous n’avez pas de ressources de disque managé qui remplissent les conditions de réservation, vous perdez une quantité de réservation pour cette heure. Les heures non utilisées ne sont pas reportées.

Lorsque vous supprimez une ressource, la remise de réservation s’applique automatiquement à une autre ressource correspondante dans l’étendue spécifiée. Si aucune ressource correspondante n’est trouvée, les heures réservées sont perdues.

## <a name="discount-examples"></a>Exemples de remises

Les exemples suivants montrent comment la remise de réservation de disque Azure s’applique en fonction de votre déploiement.

Supposons que vous achetez et réservez 100 disques P30 dans la région USA Ouest 2 pour une année. Chaque disque dispose d’environ 1 Tio de stockage. Considérons que le coût de cet exemple de réservation est de 140 100 USD. Vous pouvez choisir de payer le montant total à l’avance ou d’effectuer des versements mensuels fixes de 11 675 USD pendant les 12 prochains mois.

Les scénarios suivants décrivent ce qu’il se passe si vous sous-utilisez, sur-utilisez ou nivelez votre capacité réservée. Pour ces exemples, partons du principe que vous vous êtes inscrit à un plan de paiement de réservation mensuel.

### <a name="underusing-your-capacity"></a>Sous-utilisation de votre capacité

Supposons que vous déployez seulement 99 de vos 100 disques P30 SSD Azure Premium pendant une heure dans le cadre de la période de réservation. Le disque P30 restant n’est pas appliqué pendant cette heure. Il n’est pas non plus reporté.

### <a name="overusing-your-capacity"></a>Sur-utilisation de votre capacité

Supposons que vous utilisez 101 disques P30 SSD Premium pendant une heure dans le cadre de la période de réservation. La remise de réservation s’applique uniquement aux 100 disques P30. Le disque P30 restant est facturé aux tarifs du paiement à l’utilisation pour cette heure. Pour l’heure suivante, si votre utilisation descend à 100 disques P30, l’utilisation est entièrement couverte par la réservation.

### <a name="tiering-your-capacity"></a>Hiérarchisation de votre capacité

Supposons que vous souhaitez utiliser au total 200 disques P30 SSD Premium pendant une heure donnée dans le cadre de la période de réservation. Par ailleurs, vous en utilisez seulement 100 au cours des 30 premières minutes. Durant cette période, votre utilisation est entièrement couverte parce que la réservation que vous avez effectuée portait sur 100 disques P30. Si vous interrompez ensuite l’utilisation des 100 premiers disques (votre utilisation est donc de zéro), puis que vous commencez à utiliser les 100 autres disques pendant les 30 minutes restantes, cette utilisation est également couverte dans le cadre de votre réservation.

![Exemple de sous-utilisation, de sur-utilisation et de nivellement de la capacité](media/understand-disk-reservations/reserved-disks-example-scenarios.png)

## <a name="need-help-contact-us"></a>Vous avez besoin d’aide ? Nous contacter

Si vous avez des questions ou besoin d’aide, [créez une demande de support](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Étapes suivantes

- [Réduire les coûts avec la réservation de disques Azure](../../virtual-machines/disks-reserved-capacity.md)
- [Qu’est-ce qu’une réservation Azure ?](save-compute-costs-reservations.md)
