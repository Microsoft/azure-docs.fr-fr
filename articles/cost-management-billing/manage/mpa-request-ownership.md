---
title: Obtenir la propriété de facturation d’abonnements Azure dans le cadre d’un Contrat Partenaire Microsoft (MPA)
description: Découvrez comment demander la propriété de facturation d’abonnements Azure d’autres utilisateurs pour un Contrat Partenaire Microsoft (MPA).
author: amberbhargava
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: how-to
ms.date: 10/19/2020
ms.author: banders
ms.openlocfilehash: aaa94f66f1b0441e025013369b3b674d799d4847
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92203260"
---
# <a name="get-billing-ownership-of-azure-subscriptions-to-your-mpa-account"></a>Obtenir la propriété de facturation des abonnements Azure associés à votre compte MPA

Afin de fournir une seule facture combinée pour les services managés et la consommation Azure, un fournisseur de solutions cloud peut prendre la propriété de la facturation des abonnements Azure à ses clients qui ont des Contrats Entreprise directs.

Cette fonctionnalité est disponible uniquement pour les partenaires CSP à facturation directe qui sont certifiés [Azure Expert MSP](https://partner.microsoft.com/membership/azure-expert-msp). Elle est soumise à la gouvernance et aux stratégies Microsoft, et peut nécessiter un examen et une approbation supplémentaires pour certains clients.

Pour demander la propriété de facturation, vous devez avoir le rôle **Administrateur général** ou **Agents d’administration** . Pour plus d’informations, consultez [Espace partenaires - Affecter des rôles et des autorisations aux utilisateurs](/partner-center/permissions-overview).

Cet article concerne les comptes de facturation associés aux Contrats Partenaire Microsoft. Ces comptes sont créés pour permettre aux fournisseurs de solutions Cloud (ou CSP) de gérer la facturation de leurs clients dans la nouvelle expérience de commerce. La nouvelle expérience est disponible uniquement pour les partenaires ayant au moins un client qui a accepté un Contrat client Microsoft (MCA) et qui possède un plan Azure. [Vérifiez si vous avez accès à un Contrat Partenaire Microsoft](#check-access-to-a-microsoft-partner-agreement).

## <a name="prerequisites"></a>Prérequis

1. Établissez une [relation de revendeur](/partner-center/request-a-relationship-with-a-customer) avec le client. Vérifiez dans la [vue d’ensemble des autorisations régionales CSP](/partner-center/regional-authorization-overview) que le locataire client et le locataire partenaire se trouvent dans les mêmes régions autorisées.
1. [Assurez-vous que le client a accepté le Contrat Client Microsoft ](/partner-center/confirm-customer-agreement).
1. Configurez un [plan Azure](/partner-center/purchase-azure-plan) pour le client. Si le client effectue des achats par l’entremise de plusieurs revendeurs, vous devez configurer un plan Azure pour chaque relation client-revendeur.

## <a name="request-billing-ownership"></a>Demander la propriété de facturation

1. Connectez-vous au [portail Azure](https://portal.azure.com) à l’aide des informations d’identification de l’agent d’administration CSP dans le locataire CSP.
1. Recherchez **Gestion des coûts + facturation** .  
    ![Capture d’écran montrant la recherche, sur le portail Azure, de « cost management + billing » pour demander la propriété de facturation.](./media/mpa-request-ownership/search-cmb.png)
1. Sélectionnez **Clients** dans la partie gauche, puis sélectionnez un client dans la liste.  
    [![Capture d’écran montrant la sélection de clients](./media/mpa-request-ownership/mpa-select-customers.png)](./media/mpa-request-ownership/mpa-select-customers.png#lightbox)
1. Sélectionnez **Demandes de transfert** dans la partie inférieure gauche, puis sélectionnez **Ajouter une nouvelle demande** .  
    [![Capture d’écran montrant la sélection des demandes de transfert](./media/mpa-request-ownership/mpa-select-transfer-requests.png)](./media/mpa-request-ownership/mpa-select-transfer-requests.png#lightbox)
1. Entrez l’adresse e-mail de l’utilisateur dans l’organisation du client qui doit accepter la demande de transfert. L’utilisateur doit être propriétaire d’un compte dans un Contrat Entreprise. Sélectionnez **Envoyer la demande de transfert** .  
    [![Capture d’écran montrant l’envoi d’une demande de transfert](./media/mpa-request-ownership/mpa-send-transfer-requests.png)](./media/mpa-request-ownership/mpa-send-transfer-requests.png#lightbox)
1. L’utilisateur reçoit un e-mail contenant des instructions pour l’examen de votre demande de transfert.  
    ![Capture d’écran de l’e-mail invitant à examiner une demande de transfert](./media/mpa-request-ownership/mpa-review-transfer-request-email.png)
1. Pour approuver la demande de transfert, l’utilisateur sélectionne le lien figurant dans l’e-mail, puis suit les instructions.  
    [![Capture d’écran montrant l’examen d’une demande de transfert](./media/mpa-request-ownership/review-transfer-requests.png)](./media/mpa-request-ownership/review-transfer-requests.png#lightbox) L’utilisateur peut sélectionner le compte de facturation à partir duquel il souhaite transférer les produits Azure. Une fois sélectionnés, les produits éligibles qui peuvent être transférés sont affichés. **Remarque :** Les abonnements désactivés ne peuvent pas être transférés et s’affichent dans la liste « Produits Azure non transférables », le cas échéant. Une fois que les produits Azure à transférer sont sélectionnés, sélectionnez **Valider** .
1. La zone **Résultat de la validation du transfert** indique l’impact des produits Azure qui vont être transférés. Voici les états possibles :
    * **Réussite** - La validation pour ce produit Azure a réussi et il peut être transféré.
    * **Avertissement** - Un avertissement s’affiche pour le produit Azure sélectionné. Même si le produit peut encore être transféré, cette opération aura un impact dont l’utilisateur doit être conscient s’il souhaite prendre des mesures d’atténuation. Par exemple, l’abonnement Azure transféré bénéficie d’une instance réservée. Après le transfert, l’abonnement ne bénéficie plus de cet avantage. Pour optimiser les économies, vérifiez que l’instance réservée est associée à un autre abonnement qui peut tirer parti de ses avantages. Au lieu de cela, l’utilisateur peut également choisir de revenir à la page de sélection et de désélectionner cet abonnement Azure.
    * **Échec** - Le produit Azure sélectionné ne peut pas être transféré en raison d’une erreur. L’utilisateur doit revenir à la page de sélection et désélectionner ce produit pour transférer les autres produits Azure sélectionnés.  
    ![Capture d’écran montrant l’expérience de validation](./media/mpa-request-ownership/validate-transfer-request.png)

## <a name="check-the-transfer-request-status"></a>Vérifier l’état d’une demande de transfert

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Recherchez **Gestion des coûts + facturation** .  
    ![Capture d’écran montrant la recherche, sur le portail Azure, de « cost management + billing » pour demander le statut de transfert.](./media/mpa-request-ownership/billing-search-cost-management-billing.png)
1. Sélectionnez **Clients** dans la partie gauche.  
    [![Capture d’écran montrant la sélection de clients](./media/mpa-request-ownership/mpa-select-customers.png)](./media/mpa-request-ownership/mpa-select-customers.png#lightbox)
1. Dans la liste, sélectionnez le client pour lequel vous avez envoyé la demande de transfert.
1. Sélectionnez **Demandes de transfert** dans la partie inférieure gauche. La page Demandes de transfert affiche les informations suivantes : [![Capture d’écran montrant la liste des demandes de transfert](./media/mpa-request-ownership/mpa-select-transfer-requests-for-status.png)](./media/mpa-request-ownership/mpa-select-transfer-requests-for-status.png#lightbox)

   |Colonne|Définition|
   |---------|---------|
   |Date de la demande|Date à laquelle la demande de transfert a été envoyée|
   |Recipient|Adresse e-mail de l’utilisateur auquel vous avez envoyé la demande de transfert de la propriété de facturation|
   |Date d'expiration|Date d’expiration de la demande|
   |Statut|Statut de la demande de transfert|

    La demande de transfert peut avoir l’un des statuts suivants :

   |Statut|Définition|
   |---------|---------|
   |En cours|L’utilisateur n’a pas accepté la demande de transfert.|
   |Traitement en cours|L’utilisateur a approuvé la demande de transfert. La facturation des abonnements que l’utilisateur a sélectionnés est en cours de transfert vers votre compte.|
   |Completed| La facturation des abonnements que l’utilisateur a sélectionnés a été transférée vers votre compte.|
   |Transfert terminé avec des erreurs|Le traitement de la demande est terminé, mais la facturation de certains abonnements que l’utilisateur a sélectionnés n’a pas pu être transférée.|
   |Expiré|L’utilisateur n’ayant pas accepté la demande à temps, celle-ci a expiré.|
   |Opération annulée|Une personne ayant accès à la demande de transfert a annulé celle-ci.|
   |Refusée|L’utilisateur a refusé la demande de transfert.|

1. Sélectionnez une demande de transfert pour en afficher les détails. La page des détails du transfert affiche les informations suivantes : [![Capture d’écran montrant la liste des abonnements transférés](./media/mpa-request-ownership/mpa-transfer-completed.png)](./media/mpa-request-ownership/mpa-transfer-completed.png#lightbox)

   |Colonne  |Définition|
   |---------|---------|
   |ID de la demande de transfert|ID unique de votre demande de transfert. Si vous envoyez une demande de support, partagez l’ID avec le support Azure pour accélérer le traitement de votre demande|
   |Transfert demandé le|Date à laquelle la demande de transfert a été envoyée|
   |Transfert demandé par|Adresse e-mail de l’utilisateur qui a envoyé la demande de transfert|
   |La requête de transfert expire le| Date d’expiration de la demande de transfert|
   |Adresse e-mail du destinataire|Adresse e-mail de l’utilisateur auquel vous avez envoyé la demande de transfert de la propriété de facturation|
   |Lien du transfert envoyé au destinataire|URL envoyée à l’utilisateur pour l’examen de la demande de transfert|

## <a name="supported-subscription-types"></a>Types d’abonnements pris en charge

Vous pouvez demander à obtenir la propriété de facturation des types d’abonnement répertoriés ci-dessous.

* [Enterprise Dev/Test](https://azure.microsoft.com/offers/ms-azr-0148p/)\*
* [Contrat Entreprise Microsoft](https://azure.microsoft.com/pricing/enterprise-agreement/)

\* Vous devez convertir un abonnement Dev/Test en offre d’entreprise Contrat Entreprise via un ticket de support. Un abonnement Enterprise Dev/Test sera facturé au tarif du paiement à l’utilisation après avoir été transféré. Toute remise offerte via l’offre d’entreprise Dev/Test via le Contrat Entreprise du client ne sera pas disponible pour le partenaire CSP.

## <a name="additional-information"></a>Informations supplémentaires

La section suivante fournit des informations supplémentaires sur le transfert d’abonnements.

### <a name="no-service-downtime"></a>Aucune interruption de service.

Les services Azure compris dans l’abonnement ne subissent aucune interruption. Nous effectuons la transition de la relation de facturation uniquement pour les abonnements Azure que l’utilisateur sélectionne pour le transfert.

### <a name="disabled-subscriptions"></a>Abonnements désactivés

Les abonnements désactivés ne peuvent pas être transférés. Le transfert de propriété de facturation ne peut être effectué que pour des abonnements en état inactif.

### <a name="azure-resources-transfer"></a>Transfert de ressources Azure

Toutes les ressources associées aux abonnements, telles que les machines virtuelles, les disques et les sites web sont transférées. Lors de leur transfert, les ID d’abonnement et les ID de ressource sont conservés. 

### <a name="azure-marketplace-products-transfer"></a>Transfert de produits de la Place de marché Azure

Les produits de la Place de marché Azure qui sont disponibles pour les abonnements gérés par les fournisseurs de solutions cloud sont transférés en même temps que leurs abonnements respectifs. Les abonnements contenant des produits de la Place de marché Azure qui ne sont pas activés pour les fournisseurs de solutions cloud ne peuvent pas être transférés.

### <a name="azure-reservations-transfer"></a>Transfert de réservations Azure

Les réservations Azure ne suivent pas automatiquement les abonnements. Vous pouvez conserver la réservation dans le cadre d’un Contrat Entreprise pour d’autres abonnements ou [annuler la réservation](../reservations/exchange-and-refund-azure-reservations.md) et la racheter dans le cadre du programme Fournisseur de solutions cloud pour les partenaires.

### <a name="access-to-azure-services"></a>Accès aux services Azure

La transition n’affecte pas l’accès d’utilisateurs, de groupes ou de principaux de service existants qui a été attribué à l’aide du [contrôle d’accès en fonction du rôle Azure (RBAC Azure)](../../role-based-access-control/overview.md). Le partenaire n’obtient aucun nouvel accès RBAC Azure aux abonnements.

Les partenaires doivent demander au client un accès aux abonnements. Les partenaires doivent obtenir un accès [Administrateur pour le compte de](https://channel9.msdn.com/Series/cspdev/Module-11-Admin-On-Behalf-Of-AOBO) ou [Azure Lighthouse](../../lighthouse/concepts/cloud-solution-provider.md) pour ouvrir des tickets de support.

### <a name="azure-support-plan"></a>Plan de support Azure

Le support Azure n’est pas transféré avec les abonnements. Si l’utilisateur transfère tous les abonnements Azure, demandez-leur d’annuler leur plan de support. Après le transfert, le partenaire CSP est responsable du support. Le client doit se rapprocher du partenaire CSP pour toute demande de support.  

### <a name="charges-for-transferred-subscription"></a>Frais de transfert d’abonnement

Le propriétaire de facturation d’origine des abonnements est responsable de tous les frais déclarés jusqu’à ce que le transfert soit terminé. Vous êtes responsable des frais déclarés à compter de la réalisation effective du transfert. Certains frais peuvent avoir été exposés avant le transfert mais déclarés ultérieurement. Ces frais apparaissent dans votre facture.

### <a name="cancel-a-transfer-request"></a>Annuler une demande de transfert

Vous pouvez annuler une demande de transfert jusqu’à ce qu’elle soit approuvée ou refusée. Pour annuler une demande de transfert, accédez à la [page des détails du transfert](#check-the-transfer-request-status) et sélectionnez Annuler en bas de celle-ci.

### <a name="software-as-a-service-saas-transfer"></a>Transfert SaaS (Software as a Service)

Les produits SaaS ne sont pas transférés avec les abonnements. Demandez à l’utilisateur de [contacter le support Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) pour transférer la propriété de facturation de produits SaaS. En même temps que la propriété de facturation, l’utilisateur peut transférer la propriété de la ressource. La propriété des ressources vous permet d’effectuer des opérations de gestion telles que la suppression et l’affichage des détails du produit. Pour transférer la propriété d’une ressource, l’utilisateur doit en être le propriétaire sur le produit SaaS.

### <a name="additional-approval-for-certain-customers"></a>Approbation supplémentaire pour certains clients

Certaines des demandes de transfert du client peuvent nécessiter un processus d’examen supplémentaire de la part de Microsoft en raison de la nature de la structure actuelle des inscriptions d’entreprise du client. Le partenaire est averti de ces exigences quand il essaie d’envoyer une invitation à des clients. Les partenaires sont invités à collaborer avec leur responsable de développement partenaire et l’équipe de compte du client pour effectuer ce processus d’examen.

### <a name="azure-subscription-directory"></a>Répertoire des abonnements Azure

Le répertoire des abonnements Azure qui sont transférés doit correspondre au répertoire du client qui a été sélectionné lors de l’établissement de la relation CSP.

Si ces deux répertoires ne correspondent pas, les abonnements ne peuvent pas être transférés. Vous devez alors soit établir une nouvelle relation de revendeur CSP avec le client en sélectionnant le répertoire approprié des abonnements Azure, soit changer le répertoire des abonnements Azure pour qu’il corresponde au répertoire de la relation CSP-client. Pour plus d’informations, consultez [Associer un abonnement existant à un annuaire Azure AD](../../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md#to-associate-an-existing-subscription-to-your-azure-ad-directory).

### <a name="ea-subscription-in-the-non-organization-directory"></a>Abonnement Contrat Entreprise dans l’annuaire hors organisation

Les abonnements Contrat Entreprise provenant d’annuaires hors organisation peuvent être transférés tant que l’annuaire a une relation de revendeur avec le fournisseur de solutions cloud. Si l’annuaire n’a pas de relation de revendeur, veillez à ce que l’utilisateur de l’organisation soit titulaire du rôle *Administrateur général* dans l’annuaire pour qu’il puisse accepter la relation de partenaire. La partie nom de domaine du nom d’utilisateur doit être le nom de domaine par défaut initial « [nom de domaine]. onmicrosoft.com » ou un nom de domaine personnalisé vérifié et non fédéré tel que « contoso.com ».  

Pour ajouter un utilisateur à l’annuaire, consultez [Démarrage rapide : Ajouter de nouveaux utilisateurs à Azure Active Directory](../../active-directory/fundamentals/add-users-azure-active-directory.md).

## <a name="check-access-to-a-microsoft-partner-agreement"></a>Vérifier l’accès à un Contrat Partenaire Microsoft

[!INCLUDE [billing-check-mpa](../../../includes/billing-check-mpa.md)]

## <a name="need-help-contact-support"></a>Vous avez besoin d’aide ? Contacter le support technique

Si vous avez toujours besoin d’aide, [contactez le support technique](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) pour obtenir une prise en charge rapide de votre problème.

## <a name="next-steps"></a>Étapes suivantes

* La propriété de facturation des abonnements Azure vous a été transférée. Suivez les frais liés à ces abonnements dans le [Portail Microsoft Azure](https://portal.azure.com).
* Demandez au client un accès aux abonnements Azure transférés. [Ajouter ou supprimer des attributions de rôles Azure à l’aide du portail Azure](../../role-based-access-control/role-assignments-portal.md).