---
title: Afficher les réservations pour les ressources Azure
description: Découvrez comment afficher les réservations Azure dans le portail Azure. Affichez les réservations et leur utilisation à l’aide d’API, de PowerShell, de l’interface CLI et de Power BI.
author: yashesvi
ms.reviewer: yashar
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: how-to
ms.date: 12/15/2020
ms.author: banders
ms.openlocfilehash: 8c69f477f363654b8bd707949f0a5b4c46a4e8df
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97561102"
---
# <a name="view-azure-reservations"></a>Voir les réservations Azure

Cet article explique comment afficher les réservations Azure dans le portail Azure. Vous pouvez voir et gérer une réservation achetée dans le portail Azure.

## <a name="who-can-manage-a-reservation-by-default"></a>Qui peut gérer une réservation par défaut

Par défaut, les utilisateurs suivants peuvent voir et gérer des réservations :

- La personne qui achète une réservation et l’administrateur de compte de l’abonnement de facturation utilisé pour acheter la réservation sont ajoutés à l’ordre de réservation.
- Les administrateurs de facturation de l’Accord Entreprise et du Contrat client Microsoft.

Pour permettre à d’autres personnes de gérer des réservations, vous avez le choix entre deux options :

- Déléguer la gestion de l’accès d’un ordre de réservation individuelle :
    1. Connectez-vous au [portail Azure](https://portal.azure.com).
    1. Sélectionnez **Tous les services** > **Réservation** pour afficher la liste des réservations auxquelles vous avez accès.
    1. Sélectionnez la réservation pour laquelle vous souhaitez déléguer l’accès à d’autres utilisateurs.
    1. Dans Détails de la réservation, sélectionnez l’ordre de réservation.
    1. Sélectionnez **Contrôle d’accès (IAM)** .
    1. Sélectionnez **Ajouter une attribution de rôle** > **Rôle** > **Propriétaire**. Si vous souhaitez accorder un accès limité, sélectionnez un autre rôle.
    1. Tapez l’adresse e-mail de l’utilisateur à ajouter comme propriétaire.
    1. Sélectionnez l’utilisateur, puis **Enregistrer**.

- Ajouter un utilisateur en tant qu’administrateur de facturation à un Accord Entreprise ou à un Contrat client Microsoft :
    - Pour un Accord Entreprise, ajoutez des utilisateurs avec le rôle d’_Administrateur d’entreprise_ qui permet d’afficher et de gérer tous les ordres de réservation qui s’appliquent à l’Accord Entreprise. Les utilisateurs détenant le rôle d’_Administrateur d’entreprise (lecture seule)_ peuvent uniquement afficher la réservation. Les administrateurs de service et les propriétaires de compte ne peuvent pas afficher les réservations _à moins_ d’être explicitement ajoutés à celles-ci à l’aide du contrôle d’accès (IAM). Pour plus d’informations, consultez [Gestion des rôles Azure Enterprise](../manage/understand-ea-roles.md).

        _Les administrateurs d’entreprise peuvent prendre possession d’un ordre de réservation et ils peuvent ajouter d’autres utilisateurs à une réservation à l’aide du contrôle d’accès (IAM)._
    - Pour un Contrat client Microsoft, les utilisateurs détenant le rôle de propriétaire du profil de facturation ou le rôle de contributeur du profil de facturation peuvent gérer l’ensemble des achats de réservation effectués à l’aide du profil de facturation. Les lecteurs de profil de facturation et les gestionnaires de facture peuvent voir toutes les réservations qui sont réglées avec le profil de facturation. Toutefois, ils ne peuvent apporter aucune modification aux réservations.
    Pour plus d’informations, consultez [Rôles et tâches liés au profil de facturation](../manage/understand-mca-roles.md#billing-profile-roles-and-tasks).

### <a name="how-billing-administrators-view-or-manage-reservations"></a>Comment les administrateurs de facturation affichent ou gèrent les réservations

1. Accédez à **Cost Management + Billing** puis, sur le côté gauche de la page, sélectionnez **Transactions de réservation**.
2. Si vous disposez des autorisations de facturation nécessaires, vous pouvez afficher et gérer les réservations. Si vous ne voyez aucune réservation, assurez-vous que vous êtes connecté à l’aide du locataire Azure AD dans lequel les réservations ont été créées.

## <a name="view-reservation-and-utilization-in-the-azure-portal"></a>Afficher la réservation et l’utilisation dans le portail Azure

Pour afficher une réservation en tant que propriétaire ou que lecteur

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Accédez à [Réservations](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/ReservationsBrowseBlade).
3. La liste répertorie toutes les réservations sur lesquelles vous avez le rôle propriétaire ou lecteur. Chaque réservation affiche le dernier pourcentage d’utilisation connu.
4. Sélectionnez le pourcentage d’utilisation pour voir l’historique et les détails de l’utilisation. Consultez les détails dans la vidéo ci-dessous.
   > [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4sYwk] 

## <a name="get-reservations-and-utilization-using-apis-powershell-and-cli"></a>Obtenir des réservations et leur utilisation à l’aide d’API, de PowerShell et de l’interface CLI

Obtenez la liste de toutes les réservations à l’aide des ressources suivantes :

- [API : Ordre de réservation - Liste](/rest/api/reserved-vm-instances/reservationorder/list)
- [PowerShell : Ordre de réservation - Liste](/powershell/module/azurerm.reservations/get-azurermreservationorder)
- [CLI : Ordre de réservation - Liste](/cli/azure/reservations/reservation-order#az-reservations-reservation-order-list)

Vous pouvez également obtenir l’[utilisation des réservations](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-usage) à l’aide de l’API d’utilisation d’instances réservées. 

## <a name="see-reservations-and-utilization-in-power-bi"></a>Voir les réservations et leur utilisation dans Power BI

Les utilisateurs de Power BI ont deux options.
- Pack de contenu : La date d’achat des réservations et les données d’utilisation sont disponibles dans le [pack de contenu Power BI Consumption Insights](/power-bi/desktop-connect-azure-cost-management). Créez les rapports souhaités à l’aide du pack de contenu. 
- Application Cost Management : Utilisez l’[application Cost Management](https://appsource.microsoft.com/product/power-bi/costmanagement.azurecostmanagementapp) pour accéder à des rapports précréés que vous pouvez personnaliser.

## <a name="need-help-contact-us"></a>Vous avez besoin d’aide ? Nous contacter

Si vous avez des questions ou besoin d’aide, [créez une demande de support](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Étapes suivantes

- [Gérer les réservations Azure](manage-reserved-vm-instance.md)
- [Comprendre l’utilisation d’une réservation pour votre abonnement avec paiement à l’utilisation](understand-reserved-instance-usage.md).
- [Comprendre l’utilisation d’une réservation pour votre inscription Entreprise](understand-reserved-instance-usage-ea.md).
- [Comprendre l’utilisation d’une réservation pour les abonnements CSP](/partner-center/azure-reservations).

