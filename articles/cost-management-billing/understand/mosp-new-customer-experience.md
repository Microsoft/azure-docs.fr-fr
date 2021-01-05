---
title: Bien démarrer avec votre compte de facturation Azure mis à jour
description: Bien démarrer avec votre compte de facturation Azure mis à jour pour comprendre les modifications apportées à la nouvelle expérience de facturation et de gestion des coûts
author: bandersmsft
ms.reviewer: amberbhargava
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: conceptual
ms.date: 08/03/2020
ms.author: banders
ms.openlocfilehash: 7bec455b804d1f4b13ab7e13677092077214a121
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94965848"
---
# <a name="get-started-with-your-updated-azure-billing-account"></a>Bien démarrer avec votre compte de facturation Azure mis à jour

La gestion des coûts et des factures est l’un des principaux composants de votre expérience cloud. Elle vous aide à contrôler et à comprendre vos dépenses sur le cloud. Pour faciliter la gestion de vos coûts et factures, Microsoft met à jour votre compte de facturation Azure afin d’inclure des fonctionnalités de facturation et de gestion des coûts améliorées. Cet article décrit les mises à jour de votre compte de facturation et vous présente les nouvelles fonctionnalités.

> [!IMPORTANT]
> Votre compte sera mis à jour lorsque vous recevrez un e-mail de Microsoft vous informant des mises à jour de votre compte. La notification est envoyée 60 jours avant la mise à jour de votre compte.

## <a name="more-flexibility-with-your-new-billing-account"></a>Plus de flexibilité avec votre nouveau compte de facturation

Le diagramme suivant compare l’ancien compte de facturation avec le nouveau :

![Diagramme montrant la comparaison entre la hiérarchie de facturation dans l’ancien compte et le nouveau compte](./media/mosp-new-customer-experience/comparison-old-new-account.png)

Votre nouveau compte de facturation contient un ou plusieurs profils de facturation qui vous permettent de gérer vos factures et modes de paiement. Chaque profil de facturation contient une ou plusieurs sections de facture qui vous permettent d'organiser les coûts sur la facture du profil de facturation.

![Diagramme montrant la nouvelle hiérarchie de facturation](./media/mosp-new-customer-experience/new-billing-account-hierarchy.png)

Les rôles du compte de facturation disposent du plus haut niveau d’autorisations. Ces rôles sont à attribuer aux utilisateurs qui doivent consulter des factures et suivre les coûts pour tout votre compte, comme les responsables des services financiers et informatiques d’une organisation ou la personne qui a souscrit un compte. Pour plus d’informations, consultez [Rôles et tâches liés au compte de facturation](../manage/understand-mca-roles.md#billing-account-roles-and-tasks). Une fois votre compte mis à jour, l’utilisateur doté d’un rôle d’administrateur de compte dans l’ancien compte de facturation reçoit un rôle de propriétaire sur le nouveau compte.

## <a name="billing-profiles"></a>Profils de facturation

Un profil de facturation vous permet de gérer vos factures et modes de paiement. Une facture mensuelle est générée au début du mois pour chaque profil de facturation associé à votre compte. La facture contient les frais respectifs du mois précédent pour tous les abonnements associés au profil de facturation.

Lorsque votre compte est mis à jour, un profil de facturation est automatiquement créé pour chaque abonnement. Les frais liés à l’abonnement sont facturés dans le profil de facturation qui convient et apparaissent sur la facture.

Les rôles associés aux profils de facturation sont autorisés à afficher et à gérer les factures et les modes de paiement. Ces rôles sont à attribuer aux utilisateurs qui paient les factures tels que les membres du service de comptabilité d’une organisation. Pour plus d’informations, consultez [Rôles et tâches liés au profil de facturation](../manage/understand-mca-roles.md#billing-profile-roles-and-tasks). 

Lorsque votre compte est mis à jour, pour chaque abonnement pour lequel vous avez accordé à d’autres utilisateurs l’autorisation d’[afficher des factures](download-azure-invoice.md#allow-others-to-download-the-your-subscription-invoice), les utilisateurs titulaires d’un rôle Azure propriétaire, contributeur, lecteur ou lecteur de facturation se voient attribuer le rôle lecteur sur le profil de facturation correspondant.

## <a name="invoice-sections"></a>Sections de facture

Une section de facture vous permet d'organiser les coûts sur votre facture. Par exemple, une seule facture peut suffire, mais vous pouvez souhaitez organiser les coûts par service, équipe ou projet. Pour ce scénario, vous disposez d'un seul profil de facturation dans lequel vous créez une section de facture pour chaque service, équipe ou projet.

Une fois votre compte mis à jour, une section de facture est créée pour chaque profil de facturation et l’abonnement associé est attribué à la section de facture. Lorsque vous ajoutez d’autres abonnements, vous pouvez créer des sections supplémentaires et affecter les abonnements aux sections de facture. Ces sections apparaissent sur la facture et reflètent l’utilisation de chaque abonnement que vous leur avez attribué.

Les rôles figurant dans la section de facture sont autorisés à contrôler qui crée des abonnements Azure. Ces rôles sont à attribuer aux utilisateurs configurant l’environnement Azure pour les équipes de votre organisation, tels que les responsables d'ingénierie et les architectes techniques. Pour plus d’informations, consultez [Rôles et tâches liés aux sections de facture](../manage/understand-mca-roles.md#invoice-section-roles-and-tasks).

## <a name="enhanced-features-in-your-new-experience"></a>Fonctionnalités améliorées dans votre nouvelle expérience

Votre nouvelle expérience comprend les fonctionnalités de facturation et de gestion des coûts suivantes qui vous permettent de gérer facilement vos coûts et factures :

#### <a name="invoice-management"></a>Gestion des factures

**Période de facturation mensuelle plus prévisible** : dans votre nouveau compte, la période de facturation commence le premier jour du mois et se termine le dernier jour du mois, quelle que soit la date à laquelle vous vous inscrivez pour utiliser Azure. Une facture est générée au début de chaque mois et contient tous les frais du mois précédent.

**Obtenez une seule facture mensuelle pour plusieurs abonnements** : vous avez la possibilité d’obtenir une facture mensuelle pour chaque abonnement ou une facture unique pour plusieurs abonnements.

**Recevez une seule facture mensuelle pour les abonnements Azure, les plans de support et les produits de la Place de marché Azure** : vous obtenez une facture mensuelle pour tous les frais, y compris les frais d’utilisation des abonnements Azure, les plans de support et les achats effectués sur la Place de marché Azure.

**Regroupez les coûts sur la facture selon vos besoins** : vous pouvez regrouper les coûts sur votre facture selon vos besoins, par services, projets ou équipes.

**Définissez un numéro de bon de commande facultatif sur la facture** : pour associer votre facture à vos systèmes financiers internes, définissez un numéro de bon de commande. Gérez-le et mettez-le à jour à tout moment dans le portail Azure.

#### <a name="payment-management"></a>Gestion des paiements

**Payez les factures immédiatement à l’aide d’une carte de crédit** :vous n’êtes pas tenu d’attendre que le paiement automatique intervienne sur votre carte de crédit. Vous pouvez utiliser n’importe quelle carte de crédit pour payer une facture due ou en retard dans le portail Azure.

**Effectuez le suivi de tous les paiements appliqués au compte** : affichez tous les paiements appliqués à votre compte, notamment le mode de paiement utilisé, le montant payé et la date de paiement dans le portail Azure.

#### <a name="cost-management"></a>la gestion des coûts ;

**Planifiez des exportations mensuelles d’utilisation des données dans un compte de stockage** : publiez automatiquement vos données sur les coûts et l’utilisation dans un compte de stockage, sur une base quotidienne, hebdomadaire ou mensuelle.

#### <a name="account-and-subscription-management"></a>Gestion des comptes et des abonnements

**Affectez plusieurs administrateurs aux opérations de facturation** : attribuez des autorisations de facturation à plusieurs utilisateurs pour gérer la facturation de votre compte. Bénéficiez d’une certaine flexibilité en accordant des autorisations de lecture, d’écriture ou les deux à d’autres utilisateurs.

**Créez des abonnements supplémentaires directement dans le portail Azure** : créez tous vos abonnements en un seul clic dans le portail Azure.

#### <a name="api-support"></a>prise en charge des API

**Effectuez des opérations de facturation et de gestion des coûts via les API, kits de développement logiciel (SDK) et PowerShell** : utilisez les API de gestion des coûts, de facturation et de consommation pour extraire les données de facturation et de coût dans vos outils d’analyse de données préférés.

**Effectuez toutes les opérations d’abonnement via les API, kits de développement logiciel (SDK) et PowerShell** : utilisez les API d’abonnement Azure pour automatiser la gestion de vos abonnements Azure, y compris la création, le changement de nom et l’annulation d’un abonnement.

## <a name="get-prepared-for-your-new-experience"></a>Préparez-vous à votre nouvelle expérience

Pour vous préparer à votre nouvelle expérience, nous vous recommandons d’utiliser les éléments suivants :

**Période de facturation mensuelle et date de facture différente**

Dans la nouvelle expérience, votre facture est générée le neuvième jour de chaque mois et contient tous les frais du mois précédent. Cette date peut différer de la date à laquelle votre facture est générée dans l’ancien compte. Si vous partagez vos factures avec d’autres personnes, informez-les de ce changement de date.

**Nouvelles API de facturation et de gestion des coûts**

Si vous utilisez les API de facturation ou de gestion des coûts pour interroger et mettre à jour vos données de facturation ou de coût, vous devez utiliser les nouvelles API. Le tableau ci-dessous répertorie les API qui ne fonctionnent pas avec le nouveau compte de facturation, ainsi que les modifications que vous devez apporter à votre nouveau compte de facturation.

|API | Modifications  |
|---------|---------|
|[Comptes de facturation - Liste](/rest/api/billing/2019-10-01-preview/billingaccounts/list) | Dans l’API Comptes de facturation - Liste, votre ancien compte de facturation indique pour agreementType **MicrosoftOnlineServiceProgram**, votre nouveau compte de facturation indiquera pour agreementType **MicrosoftCustomerAgreement**. En cas de dépendance pour agreementType, mettez-la à jour. |
|[Factures - Liste par abonnement de facturation](/rest/api/billing/2019-10-01-preview/invoices/listbybillingsubscription)     | Cette API renvoie uniquement les factures générées avant la mise à jour de votre compte. Utilisez l’API [Factures - Liste par compte de facturation](/rest/api/billing/2019-10-01-preview/invoices/listbybillingaccount) pour obtenir les factures générées dans votre nouveau compte de facturation. |

## <a name="additional-information"></a>Informations supplémentaires

Les sections suivantes fournissent des informations supplémentaires sur votre nouvelle expérience.

**Aucune interruption de service.** Les services Azure compris dans votre abonnement continuent de s’exécuter sans aucune interruption. La seule mise à jour porte sur votre expérience de facturation. Les ressources, groupes de ressources ou groupes d’administration existants ne sont pas affectés.

**Aucune modification apportée aux ressources Azure** L’accès aux ressources Azure défini à l’aide du contrôle d’accès en fonction du rôle Azure (Azure RBAC) n’est pas affecté par la mise à jour.

**Les factures antérieures sont disponibles dans la nouvelle expérience** Les factures générées avant la mise à jour de votre compte restent disponibles dans le portail Azure.

**Factures pour le compte mis à jour au milieu du mois** Si votre compte est mis à jour au milieu du mois, vous recevrez une facture pour les frais accumulés jusqu’au jour de la mise à jour de votre compte. Vous recevrez une autre facture pour le reste du mois. Par exemple, votre compte dispose d’un abonnement et il est mis à jour le 15 septembre. Vous recevrez une facture pour les frais accumulés jusqu’au 15 septembre. Vous recevrez une autre facture pour la période comprise entre le 15 septembre et le 30 septembre. Au-delà du mois de septembre, vous recevrez une facture par mois.

## <a name="need-help-contact-support"></a>Vous avez besoin d’aide ? Contactez le support technique.

Si vous avez toujours besoin d’aide, [contactez le support technique](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) pour obtenir une prise en charge rapide de votre problème.

## <a name="next-steps"></a>Étapes suivantes

Consultez les articles suivants pour en savoir plus sur votre compte de facturation.

- [Comprendre les rôles d’administration associés à votre nouveau compte de facturation](../manage/understand-mca-roles.md)
- [Créer un abonnement Azure supplémentaire associé à votre compte de facturation](../manage/create-subscription.md)
- [Créer des sections sur votre facture pour organiser vos coûts](../manage/mca-section-invoice.md)