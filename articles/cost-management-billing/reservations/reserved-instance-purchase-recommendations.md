---
title: Recommandations de réservation Azure
description: Apprenez-en davantage sur les recommandations de réservation Azure.
author: banders
ms.author: banders
ms.reviewer: yashar
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: conceptual
ms.date: 09/02/2020
ms.openlocfilehash: 869009d7dd26685842da3c948fbdc058a38feb21
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/02/2020
ms.locfileid: "89398424"
---
# <a name="reservation-recommendations"></a>Reservation recommendations

Les recommandations d’achat d’instances réservées Azure sont fournies via l’[API de recommandation de réservation](/rest/api/consumption/reservationrecommendations) d’Azure Consumption, via [Azure Advisor](../../advisor/advisor-cost-recommendations.md#buy-reserved-virtual-machine-instances-to-save-money-over-pay-as-you-go-costs) et via de l’expérience d’achat de réservation dans le portail Azure.

Les étapes suivantes définissent comment les recommandations sont calculées :

1. Le moteur de recommandation évalue l’utilisation horaire de vos ressources dans l’étendue donnée au cours des 7, 30 et 60 derniers jours.
2. En fonction des données d’utilisation, le moteur simule vos coûts avec et sans réservations.
3. Les coûts sont simulés pour différentes quantités, et la quantité qui optimise les économies est recommandée.
4. Si vos ressources sont arrêtées régulièrement, la simulation ne trouve aucune économie et aucune recommandation d’achat n’est fournie.
5. Les recommandations tiennent compte des remises spéciales qui peuvent s’appliquer à vos taux d’utilisation à la demande.

## <a name="recommendations-in-the-azure-portal"></a>Recommandations dans le portail Azure

Les recommandations d’achat de réservation sont également indiquées dans le portail Azure dans l’expérience d’achat. Les recommandations sont indiquées avec la **quantité recommandée**. Une fois achetée, la quantité recommandée par Azure vous permettra d’économiser au maximum. Bien que vous puissiez acheter la quantité de votre choix, si vous achetez une quantité différente, vos économies ne sont pas optimales.

Prenons quelques exemples qui expliquent pourquoi.

Dans l’exemple d’image suivant pour la recommandation sélectionnée, Azure recommande une quantité d’achat de 6.

:::image type="content" source="./media/reserved-instance-purchase-recommendations/recommended-quantity.png" alt-text="Exemple présentant une recommandation d’achat de réservation" lightbox="./media/reserved-instance-purchase-recommendations/recommended-quantity.png" :::

Des informations supplémentaires sur la recommandation s’affichent lorsque vous sélectionnez le lien **Voir les détails**. L’image suivante montre des détails sur la recommandation. La quantité recommandée est calculée pour l’utilisation la plus élevée possible, en fonction de votre historique d’utilisation. Votre recommandation peut ne pas cibler une utilisation de 100 % si vous avez une utilisation incohérente. Dans l’exemple, notez que l’utilisation a fluctué dans le temps. Le coût de la réservation, les économies possibles et le pourcentage d’utilisation sont affichés.

:::image type="content" source="./media/reserved-instance-purchase-recommendations/recommended-quantity-details.png" alt-text="Exemple présentant les détails d’une recommandation d’achat de réservation" :::

Quand vous augmentez ou réduisez la quantité de réservation au-delà de la recommandation, le graphique et les valeurs estimées changent. En augmentant la quantité de réservation, vos économies sont réduites, car vous vous retrouvez avec une utilisation réduite de la réservation. En d’autres termes, vous payez pour des réservations qui ne sont pas entièrement utilisées.

Si vous réduisez la quantité de réservation, vos économies sont également réduites. Bien que vous ayez une utilisation accrue, il y aura probablement des périodes où vos réservations ne couvriront pas pleinement votre utilisation. Une utilisation au-delà de la quantité de réservation sera employée par des ressources de paiement à l’utilisation plus coûteuses. L’exemple d’image suivant illustre le point. Nous avons réduit manuellement la quantité de réservation à 4. L’utilisation de la réservation est augmentée, mais les économies globales sont réduites, car les coûts de paiement à l’utilisation sont présents.

:::image type="content" source="./media/reserved-instance-purchase-recommendations/recommended-quantity-details-changed.png" alt-text="Exemple présentant les détails d’une recommandation d’achat de réservation modifiée" :::

Pour optimiser les économies avec les réservations, essayez d’acheter des réservations aussi près que possible de la recommandation.

## <a name="recommendations-in-azure-advisor"></a>Recommandations dans Azure Advisor

Les recommandations d’achat de réservation sont disponibles dans Azure Advisor. Gardez à l’esprit les points suivants :

- Advisor produit des recommandations dont l’étendue est limitée à un seul abonnement. Si vous voulez voir les recommandations pour l’ensemble de l’étendue de facturation (compte de facturation ou profil de facturation), accédez au Portail Azure > Réservations > Ajoutez et sélectionnez le type pour lequel vous voulez voir les recommandations.
- Les recommandations disponibles dans Advisor considèrent la tendance d’utilisation sur 30 jours.
- La quantité et les économies relatives aux recommandations concernent une réservation de 3 ans, le cas échéant. Si la réservation de 3 ans n’est pas vendue pour le service, la recommandation est calculée à l’aide d’un prix de réservation de 1 an.
- Les recommandations tiennent compte des remises spéciales qui peuvent s’appliquer à vos taux d’utilisation à la demande.
- Si vous achetez une réservation d’étendue partagée, les recommandations d’achat de réservations d’Advisor peuvent mettre jusqu’à cinq jours avant de disparaître.

## <a name="other-expected-api-behavior"></a>Autre comportement attendu de l’API

- Si vous utilisez une période de recherche de 7 jours, vous risquez de ne pas obtenir de recommandations quand les machines virtuelles sont arrêtées pendant plus d’une journée.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez [comment la remise de réservation Azure est appliquée aux machines virtuelles](../manage/understand-vm-reservation-charges.md).
