---
title: Rétrofacturation des coûts Azure Reservations
description: Découvrez comment afficher les coûts Azure Reservations pour la rétrofacturation.
author: yashesvi
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: how-to
ms.date: 07/24/2020
ms.author: banders
ms.openlocfilehash: aba6ea467788c51d179ef9377243efb6035b6f98
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92148359"
---
# <a name="charge-back-azure-reservation-costs"></a>Rétrofacturation des coûts Azure Reservations

Les lecteurs de la facturation Contrat Entreprise et Contrat client Microsoft peuvent afficher les données de coût amorti des réservations. Ils ont la possibilité d’utiliser ces données pour rétrofacturer la valeur monétaire d’un abonnement, d’un groupe de ressources, d’une ressource ou d’une balise à leurs partenaires. Dans les données amorties, le prix effectif est le coût de la réservation horaire au prorata. Le coût correspond au coût total de l’utilisation de la réservation par la ressource ce jour-là.

Les utilisateurs disposant d’un abonnement individuel ont accès aux données de coût amorti dans leur fichier d’utilisation. Lorsqu’une ressource obtient une remise de réservation, la section *Informations complémentaires* du fichier d’utilisation contient les détails de la réservation. Pour plus d’informations, consultez [Téléchargement de l’utilisation sur le Portail Azure](../understand/download-azure-daily-usage.md#download-usage-from-the-azure-portal-csv).

## <a name="get-reservation-charge-back-data-for-chargeback"></a>Récupération des données de rétrofacturation des réservations

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Accédez à **Gestion des coûts + Facturation**.
1. Sous **Coût réel**, sélectionnez la métrique **Coût amorti**.
1. Pour afficher les ressources utilisées par une réservation, appliquez un filtre pour **Réservation**, puis sélectionnez Réservations.
1. Sous **Granularité**, sélectionnez **Mensuelle** ou **Quotidienne**.
1. Sélectionnez **Tableau** comme type de graphe.
1. Définissez l’option **Regrouper par** sur **Ressource**.

[![Exemple indiquant les coûts des ressources de réservation utilisables pour la rétrofacturation](./media/charge-back-usage/amortized-reservation-costs.png)](./media/charge-back-usage/amortized-reservation-costs.png#lightbox)

Voici une vidéo montrant comment afficher les coûts d’utilisation des réservations sur le Portail Azure.

 > [!VIDEO https://www.microsoft.com/videoplayer/embed/RE4sQOw] 

## <a name="need-help-contact-us"></a>Vous avez besoin d’aide ? Contactez-nous.

Si vous avez des questions ou besoin d’aide, [créez une demande de support](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

## <a name="next-steps"></a>Étapes suivantes

- Pour découvrir comment gérer une réservation, consultez [Gérer les réservations Azure](manage-reserved-vm-instance.md).
- Pour plus d’informations sur les réservations Azure, consultez les articles suivants :
  - [Qu’est-ce qu’une réservation Azure ?](save-compute-costs-reservations.md)
  - [Gérer les réservations dans Azure](manage-reserved-vm-instance.md)