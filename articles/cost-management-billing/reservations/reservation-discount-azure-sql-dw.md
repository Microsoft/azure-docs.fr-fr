---
title: Guide pratique pour appliquer des remises de réservation Azure Synapse Analytics | Microsoft Docs
description: Découvrez comment les remises de réservation Azure Synapse Analytics peuvent vous permettre de faire des économies.
author: yashesvi
ms.reviewer: yashar
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: conceptual
ms.date: 04/30/2020
ms.author: banders
ms.openlocfilehash: f0bd57befc0ec30473065ac69026a77e21e30e5c
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/20/2020
ms.locfileid: "88682601"
---
# <a name="how-reservation-discounts-apply-to-azure-synapse-analytics"></a>Guide pratique pour appliquer des remises de réservation Azure Synapse Analytics

Une fois que vous avez acheté une capacité de réserve Azure Synapse Analytics, la remise de réservation est automatiquement appliquée à vos instances provisionnées qui se trouvent dans la région concernée. La remise de réservation s’applique à l’utilisation émise par le compteur de cDWU Azure Synapse Analytics. Les opérations liées au stockage et au réseau sont facturées au tarif de paiement à l’utilisation.

## <a name="reservation-discount-application"></a>Application de la remise de réservation

La remise de capacité de réserve Azure Synapse Analytics est appliquée aux entrepôts de données en cours d’exécution sur une base horaire. Si vous n’avez pas d’entrepôt déployé pendant une heure, la capacité réservée est perdue pour cette heure. Elle n’est pas reconduite.

Après l’achat, la réservation que vous achetez est associée à l’utilisation d’Azure Synapse Analytics qu’en font les entrepôts en cours d’exécution à n’importe quel point dans le temps. Si vous arrêtez certains entrepôts, les remises de réservation s’appliquent automatiquement à tous les autres entrepôts correspondants.

Pour les entrepôts qui ne s’exécutent pas pendant une heure complète, la réservation est automatiquement appliquée aux autres instances correspondantes dans cette heure.

## <a name="discount-examples"></a>Exemples de remises

Les exemples suivants montrent comment la remise de capacité de réserve d’Azure Synapse Analytics s’applique en fonction des déploiements.

- **Exemple 1** : vous achetez 5 unités de capacité réservée de 100 cDWU. Vous exécutez une instance Azure Synapse Analytics DW1500c pendant une heure. Dans ce cas, l’utilisation émise correspond à 15 unités de 100 cDWU. La remise de réservation s’applique aux 5 unités que vous avez utilisées. Vous êtes facturé au tarif de paiement à l’utilisation pour les 10 unités restantes de 100 cDWU que vous avez utilisées. En d'autres termes, une couverture partielle est possible pour les réservations multiples.

- **Exemple 2 :** vous achetez 5 unités de capacité réservée de 100 cDWU. Vous exécutez deux instances Azure Synapse Analytics DW100c pendant une heure. Dans ce cas, deux événements d’utilisation sont émis pour 1 unité de 100 cDWU. Les deux événements d’utilisation bénéficient de remises de capacité réservée. Les 3 unités restantes de capacité réservée de 100 cDWU sont perdues et ne sont pas reconduites. En d’autres termes, une seule réservation peut être associée plusieurs instances d’Azure Synapse Analytics.

- **Exemple 3** : vous achetez 1 unité de capacité réservée de 100 cDWU. Vous exécutez deux instances Azure Synapse Analytics DW100c. Chacune s’exécute pendant 30 minutes. Dans ce cas, les deux événements d’utilisation bénéficient de remises de capacité réservée. Aucune utilisation n’est facturée au tarif de paiement à l’utilisation.

## <a name="need-help-contact-us"></a>Vous avez besoin d’aide ? Nous contacter

- Si vous avez des questions ou besoin d’aide, [créez une demande de support](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les réservations Azure, consultez les articles suivants :

- [Qu’est-ce qu’une réservation Azure ?](save-compute-costs-reservations.md)
- [Afficher les transactions de réservation](view-reservations.md)
- [Obtenir des transactions de réservation et utilisation par le biais d’une API](reservation-apis.md)
- [Gérer les réservations](manage-reserved-vm-instance.md)
