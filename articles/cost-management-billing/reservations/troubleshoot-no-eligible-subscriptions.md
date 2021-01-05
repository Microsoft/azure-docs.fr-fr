---
title: Résoudre les problèmes liés à l’absence d’abonnements éligibles dans le portail Azure
description: Cet article vous aide à résoudre les problèmes liés au message d’erreur indiquant l’absence d’abonnements éligibles dans le portail Azure lorsque vous essayez d’acheter une réservation.
author: bandersmsft
ms.service: cost-management-billing
ms.subservice: reservations
ms.author: banders
ms.reviewer: yashar
ms.topic: troubleshooting
ms.date: 12/15/2020
ms.openlocfilehash: ad85bd278b5dff1532f218acc0b8e88515d96070
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97561203"
---
# <a name="troubleshoot-no-eligible-subscriptions"></a>Résoudre les problèmes liés à l’absence d’abonnements éligibles

Cet article vous aide à résoudre les problèmes liés au message d’erreur *Aucun abonnement éligible* dans le portail Azure lorsque vous essayez d’acheter une réservation.

## <a name="symptoms"></a>Symptômes

1. Connectez-vous au [portail Azure](https://portal.azure.com) et accédez à **Réservations**.
1. Sélectionnez **Ajouter**, puis sélectionnez un service.
1. Le message d’erreur suivant s’affiche :
   ```
    No eligible subscriptions
    
    You do not have any eligible subscriptions to purchase reservations. To purchase a reservation, you should have owner or reservation purchaser permission on at least one subscription of the following type: Pay-as-you-go, CSP, Microsoft Enterprise or Microsoft Customer Agreement.
    ```
1. Dans la zone **Sélectionner le produit que vous voulez acheter**, développez la liste **Abonnement de facturation** pour voir la raison pour laquelle un abonnement spécifique n’est pas éligible pour l’achat d’une instance réservée. L’image suivante montre des exemples expliquant pourquoi une réservation ne peut pas être achetée.  
    :::image type="content" source="./media/troubleshoot-no-eligible-subscriptions/select-product-to-purchase.png" alt-text="Exemple indiquant pourquoi une réservation ne peut pas être achetée" lightbox="./media/troubleshoot-no-eligible-subscriptions/select-product-to-purchase.png" :::

## <a name="cause"></a>Cause

Pour acheter une instance réservée Azure, vous devez disposer d’au moins un abonnement répondant aux conditions suivantes :

- L’abonnement doit être un type d’offre pris en charge. Les types d’offre pris en charge sont : paiement à l’utilisation, fournisseur de solutions cloud (CSP), Microsoft Azure Enterprise ou Contrat client Microsoft.
- Vous devez être propriétaire ou acheteur de réservation sur l’abonnement.

Lorsque vous n’avez pas d’abonnement conforme aux exigences, vous obtenez l’erreur `No eligible subscriptions`.

### <a name="cause-1"></a>Cause 1

L’abonnement doit être un type d’offre pris en charge. Les types d’offre pris en charge sont : paiement à l’utilisation, CSP, Microsoft Azure Enterprise ou contrat client Microsoft. Votre type d’abonnement n’est pas pris en charge. Lorsque vous sélectionnez un abonnement correspondant à un type d’offre qui ne prend pas en charge les réservations, l’erreur suivante s’affiche.

```
Subscription not eligible for purchase

This subscription is not eligible for reservation benefit an cannot be used to purchase a reservation.
```

:::image type="content" source="./media/troubleshoot-no-eligible-subscriptions/subscription-not-eligible.png" alt-text="Exemple illustrant le message d’erreur L’abonnement ne peut pas faire l’objet d’un achat" :::

### <a name="cause-2"></a>Cause 2

Vous devez être propriétaire ou acheteur de réservation sur l’abonnement. Si vous ne disposez pas des autorisations suffisantes, l’erreur suivante s’affiche.

```
You do not have owner or reservation purchaser access on the subscription

You can only purchase reservations using subscriptions on which you have owner or reservation purchaser access.
```

## <a name="solution"></a>Solution

- Si votre offre actuelle ne prend pas en charge les réservations, vous devez créer un nouvel abonnement Azure.
- Si vous n’avez pas accès à une réservation existante, vous pouvez en obtenir l’accès à partir du propriétaire actuel.

### <a name="solution-1"></a>Solution 1

Pour acheter une réservation, vous devez créer un nouvel abonnement Azure qui prend en charge les réservations.

- Si vous disposez d’un essai gratuit Azure, vous pouvez [mettre à niveau votre abonnement](../manage/upgrade-azure-subscription.md).
- Vous pouvez créer un nouvel abonnement Azure avec les [tarifs de paiement à l’utilisation](https://azure.microsoft.com/pricing/purchase-options/pay-as-you-go/).
- Inscrivez-vous à un [Contrat client Microsoft](https://azure.microsoft.com/pricing/purchase-options/microsoft-customer-agreement/) et créez un nouvel abonnement.
- Achetez un nouvel abonnement avec un [CSP](https://www.microsoft.com/solution-providers/home) et créez un nouvel abonnement.

### <a name="solution-2"></a>Solution 2

Pour obtenir un accès de propriétaire à une réservation, vous devez obtenir l’accès à l’un ou l’autre des éléments suivants :

- L’ordre de réservation avec lequel la réservation a été achetée
- La réservation elle-même

Le propriétaire de l’ordre de réservation ou le propriétaire de la réservation en cours peut vous déléguer l’accès à l’aide des étapes suivantes.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sélectionnez **Tous les services** > **Réservation** pour afficher la liste des réservations auxquelles vous avez accès.
1. Sélectionnez la réservation pour laquelle vous souhaitez déléguer l’accès à d’autres utilisateurs.
1. Sélectionnez **Contrôle d’accès (IAM)** .
1. Sélectionnez **Ajouter une attribution de rôle** > **Rôle** > **Propriétaire**. Ou si vous souhaitez accorder un accès limité, sélectionnez un autre rôle.
1. Tapez l’adresse e-mail de l’utilisateur à ajouter comme propriétaire.
1. Sélectionnez l’utilisateur, puis **Enregistrer**.

Pour plus d’informations, consultez la rubrique [Ajouter ou modifier les utilisateurs qui peuvent gérer une réservation](manage-reserved-vm-instance.md#who-can-manage-a-reservation-by-default).

## <a name="next-steps"></a>Étapes suivantes

- Passez en revue [Ajouter ou modifier les utilisateurs qui peuvent gérer une réservation](manage-reserved-vm-instance.md#who-can-manage-a-reservation-by-default) si vous devez vous voir accorder l’accès par un propriétaire de la réservation.
