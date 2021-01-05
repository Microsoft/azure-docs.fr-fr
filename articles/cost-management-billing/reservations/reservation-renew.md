---
title: Renouvellement automatique de réservations Azure
description: Découvrez comment vous pouvez renouveler automatiquement des réservations Azure pour continuer à obtenir des remises de réservation.
author: bandersmsft
ms.reviewer: yashar
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: how-to
ms.date: 07/24/2020
ms.author: banders
ms.openlocfilehash: bf7945c56fc05c33ae3cfed1c67085b3b28f847f
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/20/2020
ms.locfileid: "88690508"
---
# <a name="automatically-renew-reservations"></a>Renouvellement automatique de réservations

Vous pouvez renouveler des réservations pour acheter automatiquement une réservation de remplacement en cas d’arrivée à expiration de la précédente. Le renouvellement automatique constitue un moyen simple de continuer à obtenir des remises de réservation. Cela vous évite également d’avoir à surveiller l’arrivée à expiration d’une réservation. Grâce au renouvellement automatique, vous réalisez des économies en évitant d’avoir à la renouveler manuellement. Le paramètre de renouvellement est par défaut désactivé. Activez ou désactivez à tout moment le paramètre de renouvellement, jusqu’à l’arrivée à expiration de la réservation existante.

Le renouvellement crée une autre réservation lors de l’arrivée à expiration de la réservation existante. Elle n’étend pas la durée de validité de la réservation existante.

Abonnez-vous pour choisir à tout moment le renouvellement automatique des réservations. Le prix de renouvellement est valable 30 jours avant la date d’expiration de la réservation existante. Lorsque vous activez le renouvellement plus de 30 jours avant l’arrivée à expiration de la réservation, vous recevez un e-mail détaillant les frais de renouvellement 30 jours avant la date d’expiration. Le prix de la réservation peut varier entre le moment où vous verrouillez le prix de renouvellement et la date du renouvellement. Si tel est le cas, le coût de renouvellement est le coût le plus bas des deux. Vous pouvez modifier la quantité de réservation. Dans ce cas, le renouvellement est mis à jour de manière à utiliser le tarif appliqué sur le marché qui est défini au moment de la modification de la quantité.

Vous n’êtes pas obligé de procéder à un renouvellement. Vous pouvez à tout moment refuser le renouvellement avant l’arrivée à expiration de la réservation existante.

## <a name="set-up-renewal"></a>Configurer le renouvellement

Sélectionnez le Portail Microsoft Azure, puis cliquez sur **Réservations**.

1. Sélectionnez la réservation.
2. Cliquez sur **Renouvellement**.
3. Sélectionnez **Acheter automatiquement une nouvelle réservation à l’expiration**.  
  ![Exemple montrant le renouvellement d’une réservation](./media/reservation-renew/reservation-renewal.png)

## <a name="if-you-dont-renew"></a>Si vous n’optez pas pour le renouvellement

Vos services continuent de s’exécuter normalement. Vous êtes facturé au tarif du paiement à l’utilisation après l’arrivée à expiration de la réservation.

## <a name="required-renewal-permissions"></a>Autorisations de renouvellement requises

Les conditions suivantes sont requises pour renouveler une réservation :

- Vous devez être propriétaire de la réservation existante.
- Vous devez être propriétaire de l’abonnement si la réservation est limitée à un seul abonnement ou groupe de ressources.
- Vous devez être propriétaire de l’abonnement s’il possède une étendue partagée.

## <a name="default-renewal-settings"></a>Paramètres de renouvellement par défaut

Par défaut, le renouvellement hérite de toutes les propriétés de la réservation arrivant à expiration. L’achat d’un renouvellement de réservation repose sur les mêmes référence SKU, région, étendue, abonnement de facturation, terme et quantité.

Toutefois, vous pouvez mettre à jour la quantité d’achat de la réservation de renouvellement, afin de réaliser davantage d’économies.

## <a name="when-the-new-reservation-is-purchased"></a>Lors de l’achat de la nouvelle réservation

Une nouvelle réservation est achetée au moment où la réservation existante arrive à expiration. Nous essayons d’éviter tout délai entre deux réservations. Grâce à cette continuité, les coûts restent prévisibles et vous continuez à bénéficier de remises.

## <a name="changing-parent-reservation-after-setting-renewal"></a>Modification de la réservation parente après la configuration du renouvellement

Si vous apportez l’une des modifications suivantes à la réservation arrivant à expiration, le renouvellement de la réservation est annulé :

- Split
- Fusionner
- Transfert de la réservation d’un compte à un autre
- Transfert de la réservation d’un abonnement WebDirect vers un abonnement de Contrat Entreprise (EA) ou toute autre méthode d’achat
- Renouveler une inscription

La nouvelle réservation hérite du paramètre de flexibilité d’étendue et de taille d’instance de la réservation arrivant à expiration lors du renouvellement.

## <a name="new-reservation-permissions"></a>Nouvelles autorisations de réservation

Azure copie les autorisations de la réservation arrivant à expiration vers la nouvelle réservation. En outre, l’administrateur du compte d’abonnement associé à l’achat de réservation a accès à la nouvelle réservation.

## <a name="potential-renewal-problems"></a>Problèmes éventuels liés au renouvellement

Azure peut ne pas traiter le renouvellement si :

- Le paiement ne peut pas être collecté
- Une erreur système se produit pendant le renouvellement
- La référence SKU arrivant à expiration n’est pas active durant le renouvellement
- Le Contrat Entreprise fait l’objet d’un renouvellement vers un type de contrat différent

Si l’une des conditions précédentes se produit et que le renouvellement est désactivé, vous recevez une notification par e-mail.

## <a name="renewal-notification"></a>Notification de renouvellement

Les e-mails sont envoyés à différentes personnes en fonction de votre méthode d’achat :

- Clients de Contrat Entreprise : les e-mails sont envoyés aux contacts devant recevoir les notifications qui sont définis sur le portail des Contrats Entreprise.
- Clients d’abonnement individuels avec paiement à l’utilisation : les e-mails sont envoyés aux utilisateurs configurés en tant qu’administrateurs de compte.
- Clients du fournisseur de solutions Cloud : les e-mails sont envoyés au contact devant recevoir les notifications du partenaire.

## <a name="next-steps"></a>Étapes suivantes
- Pour plus d’informations sur les réservations Azure, voir [Qu’est-ce qu’une réservation Azure ?](save-compute-costs-reservations.md)
