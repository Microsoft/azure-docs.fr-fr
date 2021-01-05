---
title: Réduire les coûts d’Azure App Service avec une capacité réservée
description: Découvrez comment réduire les coûts liés aux frais de timbre Azure App Service Isolé avec une capacité réservée.
author: yashesvi
ms.reviewer: yashar
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: how-to
ms.date: 07/24/2020
ms.author: banders
ms.openlocfilehash: feb7be6ad5dbd0dfbd04b05836842d0476780d40
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92151539"
---
# <a name="save-costs-for-azure-app-service-isolated-stamp-fee-with-reserved-capacity"></a>Réduire les coûts liés aux frais de timbre Azure App Service Isolé avec une capacité réservée

Vous pouvez réaliser des économies sur les frais de timbre Azure App Service Isolé en optant pour une réservation de votre utilisation de timbre pour une durée de trois ans. Pour acheter une capacité réservée de frais de timbre Isolé, vous devez choisir la région Azure dans laquelle l’horodatage sera déployé et le nombre d’horodatages à acheter.

Lorsque vous achetez une réservation, l’utilisation des frais de timbre Isolé correspondant aux attributs de la réservation n’est plus facturée au tarif de paiement à l’utilisation. La réservation est appliquée automatiquement au nombre d’horodatages Isolé qui correspondent à la région et à l’étendue de la capacité réservée. Vous n’avez pas besoin d’affecter une réservation à un horodatage Isolé. La réservation ne s’applique pas aux workers, si bien que toutes les autres ressources associées à l’horodatage sont facturées séparément.

Lorsque la capacité réservée arrive à expiration, les instances Isolé continuent de s’exécuter, mais elles sont facturées au tarif de paiement à l’utilisation. Les réservations ne sont pas renouvelées automatiquement.

## <a name="determine-the-right-reservation-to-purchase"></a>Déterminer la réservation appropriée à acheter

En achetant une réservation, vous vous engagez à utiliser les quantités réservées au cours des trois prochaines années. Vérifiez vos données d’utilisation pour déterminer le nombre de timbres App Service Isolé que vous utilisez régulièrement et que vous êtes susceptible d’utiliser dans le futur.

En outre, assurez-vous que vous comprenez comment l’horodatage Isolé émet un compteur Linux ou Windows.

- Par défaut, un horodatage Isolé vide émet le compteur de timbres Windows. Par exemple, sans worker déployé. Il continue à émettre le compteur lorsque les workers Windows sont déployés sur l’horodatage.
- Si vous déployez un rôle de travail Linux, le compteur prend la valeur du compteur de tampon Linux.
- Lorsque les workers Linux et Windows sont déployés, l’horodatage émet le compteur Windows.

Le compteur d’horodatage peut donc changer entre Windows et Linux pendant toute la durée de vie de l’horodatage.

Achetez des réservations d’horodatage Windows si vous avez un ou plusieurs workers Windows sur l’horodatage. Le seul cas où vous devez acheter une réservation d’horodatage Linux est si vous envisagez d’avoir _uniquement_ des workers Linux dans l’horodatage.

## <a name="buy-isolated-stamp-reserved-capacity"></a>Achetez une capacité réservée d’horodatage Isolé

Vous pouvez acheter une capacité réservée d’horodatage Isolé dans le [portail Azure](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/CreateBlade/referrer/documentation/filters/%7B%22reservedResourceType%22%3A%22AppService%22%7D). Payez la réservation [à l’avance ou par paiements mensuels](./prepare-buy-reservation.md). Pour acheter une capacité réservée, vous devez disposer du rôle Propriétaire sur au moins un abonnement Entreprise ou d’un abonnement individuel avec des tarifs de paiement à l’utilisation.

- Pour les abonnements Entreprise, l’option **Ajouter des instances réservées** doit être activée sur le [portail EA](https://ea.azure.com/). Ou, si le paramètre est désactivé, vous devez être un administrateur EA.
- Pour le programme Fournisseur de solutions cloud (CSP), seuls des agents d’administration ou des agents commerciaux peuvent acheter une capacité de réserve Azure Synapse Analytics.

**Pour acheter :**

1. Accédez au [portail Azure](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/CreateBlade/referrer/documentation/filters/%7B%22reservedResourceType%22%3A%22AppService%22%7D).
1. Sélectionnez un abonnement. Utilisez la liste **Abonnement** pour choisir l’abonnement à utiliser pour payer la capacité réservée. Les coûts de la capacité réservée sont facturés conformément au mode de paiement défini pour l’abonnement. Le type d’abonnement doit être Accord Entreprise (numéros de l’offre : MS-AZR-0017P ou MS-AZR-0148P) ou Paiement à l’utilisation (numéros de l’offre : MS-AZR-0003P ou MS-AZR-0023P) ou un abonnement CSP.
    - Pour un abonnement Entreprise, les frais sont déduits du solde d’engagement monétaire de l’inscription ou facturés comme un dépassement.
    - Pour un abonnement Paiement à l’utilisation, les frais sont facturés sur le mode de paiement par carte de crédit ou par facture défini sur l’abonnement.
1. Sélectionnez une **Étendue** pour choisir une étendue d’abonnement.
    - **Étendue de groupe de ressources unique** : applique la remise de réservation aux ressources correspondantes incluses dans le groupe de ressources sélectionné uniquement.
    - **Étendue d’abonnement unique** : applique la remise de réservation aux ressources correspondantes incluses dans l’abonnement sélectionné.
    - **Étendue partagée** : applique la remise de réservation aux ressources correspondantes dans les abonnements éligibles inclus dans le contexte de facturation. Pour les clients Contrat Entreprise, le contexte de facturation correspond à l’inscription. Pour les abonnements individuels utilisant les tarifs du paiement à l’utilisation, l’étendue de facturation correspond à tous les abonnements éligibles créés par l’administrateur de compte.
1. Sélectionnez une **région** éligible pour la capacité réservée et ajoutez la réservation au panier.
1. Sélectionnez un type de plan Isolé, puis cliquez sur **Sélectionner**.  
    ![Exemple](./media/prepay-app-service-isolated-stamp/app-service-isolated-stamp-select.png)
1. Entrez le nombre d’horodatages App Service Isolé à réserver. Par exemple, une quantité de trois donne trois horodatages réservés dans une région. Cliquez sur **Suivant : Vérifier + acheter**.
1. Vérifiez, puis cliquez sur **Acheter maintenant**.

Après l’achat, accédez à [Réservations](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/ReservationsBrowseBlade) pour afficher l’état de l’achat et le surveiller à tout moment.

## <a name="cancel-exchange-or-refund-reservations"></a>Annuler, échanger ou rembourser des réservations

Vous pouvez annuler, échanger ou rembourser des réservations avec certaines limitations. Pour plus d’informations, consultez [Échanges et remboursements en libre-service pour les réservations Azure](exchange-and-refund-azure-reservations.md).

## <a name="discount-application-shown-in-usage-data"></a>Application de remise dans les données d’utilisation

Vos données d’utilisation ont un prix effectif de zéro pour l’utilisation qui bénéficie d’une remise de réservation. Les données d’utilisation affichent la remise de réservation pour chaque instance d’horodatage dans chaque réservation.

Pour plus d’informations sur l’affichage de la remise de réservation dans les données d’utilisation, consultez [Obtenir les données d’utilisation et de coûts de la réservation pour les Contrats Entreprise](understand-reserved-instance-usage-ea.md) si vous êtes un client ayant conclu un Accord Entreprise (EA). Sinon, consultez [Comprendre l’utilisation d’une réservation Azure pour votre abonnement individuel avec tarifs de paiement à l’utilisation](understand-reserved-instance-usage.md).

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur les réservations Azure, consultez les articles suivants :
  - [Qu’est-ce qu’une réservation Azure ?](save-compute-costs-reservations.md)
  - [Comprendre comment une remise de réservation Azure App Service Isolé est appliquée](reservation-discount-app-service-isolated-stamp.md)
  - [Comprendre l’utilisation d’une réservation pour votre Accord de Mise en Œuvre Entreprise](understand-reserved-instance-usage-ea.md)