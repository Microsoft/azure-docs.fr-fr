---
title: Transferts Azure Enterprise
description: Décrit les transferts Azure EA
author: bandersmsft
ms.reviewer: baolcsva
ms.service: cost-management-billing
ms.subservice: enterprise
ms.topic: conceptual
ms.date: 11/11/2020
ms.author: banders
ms.custom: contperf-fy21q1
ms.openlocfilehash: f5555662c6cb5f44be3959ec3d22d1cf1953674d
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97030091"
---
# <a name="azure-enterprise-transfers"></a>Transferts Azure Enterprise

Cet article fournit une vue d’ensemble des transferts Enterprise.

## <a name="transfer-an-enterprise-account-to-a-new-enrollment"></a>Transférer un compte d’entreprise vers une nouvelle inscription

Un transfert de compte déplace un propriétaire de compte d’une inscription à une autre. Tous les abonnements associés sous le propriétaire du compte seront déplacés vers l’inscription cible. Utilisez un transfert de compte quand vous avez plusieurs inscriptions actives et que vous voulez uniquement déplacer les propriétaires de comptes sélectionnés.

Cette section est fournie à titre d’information uniquement, car l’action ne peut pas être effectuée par un administrateur d’entreprise. Pour transférer un compte d’entreprise vers une nouvelle inscription, une demande de support est nécessaire.

Gardez les points suivants à l’esprit quand vous transférez un compte d’entreprise vers une nouvelle inscription :

- Seuls les comptes spécifiés dans la demande sont transférés. Si tous les comptes sont choisis, ils sont tous transférés.
- L’inscription source conserve son état actif ou étendu. Vous pouvez continuer à utiliser l’inscription jusqu’à ce qu’elle expire.

### <a name="prerequisites"></a>Prérequis

Lorsque vous demandez un transfert de compte, fournissez les informations suivantes :

- Le numéro de l’inscription cible, le nom du compte et l’e-mail du propriétaire du compte à transférer
- Pour l’inscription source, le numéro d’inscription et le compte à transférer

Autres points à garder à l’esprit avant un transfert de compte :

- L’approbation d’un administrateur EA est requise pour les inscriptions source et cible.
- Si un transfert de compte ne répond pas à vos exigences, envisagez un transfert d’inscription.
- Le transfert de compte transfère tous les services et abonnements associés aux comptes spécifiques.
- Au terme du transfert, le compte transféré est indiqué comme inactif sous l’inscription source et comme actif sous l’inscription cible.
- Le compte affiche la date de fin correspondant à la date de transfert effectif sur l’inscription source, et comme date de début sur l’inscription cible.
- Toute utilisation du compte effectuée avant la date de transfert effectif reste sous l’inscription source.

## <a name="transfer-enterprise-enrollment-to-a-new-one"></a>Transférer l’inscription d’entreprise vers une nouvelle inscription

Un transfert d’inscription est envisagé dans les cas suivants :

- La durée de Paiement anticipé d’une inscription en cours se termine.
- Une inscription présente l’état expiré/étendu et un nouveau contrat est négocié.
- Vous avez plusieurs inscriptions et vous voulez combiner tous les comptes et la facturation sous une seule inscription.

Cette section est fournie à titre d’information uniquement, car l’action ne peut pas être effectuée par un administrateur d’entreprise. Pour transférer une inscription d’entreprise vers une nouvelle, une demande de support est nécessaire.

Lorsque vous demandez à transférer une inscription d’entreprise complète vers une autre inscription, les actions suivantes se produisent :

- L’ensemble des services, abonnements et comptes Azure, ainsi que la structure d’inscription toute entière, dont tous les administrateurs de service EA, effectuent un transfert vers une nouvelle inscription cible.
- L’état de l’inscription est défini sur _Transféré_. L’inscription transférée est disponible uniquement à des fins de création de rapports d’utilisation.
- Vous ne pouvez pas ajouter de rôles ni d’abonnements à une inscription transférée. L’état transféré empêche une utilisation supplémentaire en relation avec l’inscription.
- Tout solde restant de Paiement anticipé Azure dans le contrat est perdu, y compris les termes futurs.
-    Si l’inscription à partir de laquelle vous effectuez le transfert inclut des achats RI, les frais d’achat RI resteront dans l’inscription source. Toutefois, tous les avantages RI seront transférés pour être utilisés dans la nouvelle inscription.
-    Les frais d’achat à usage unique de la place de marché et les frais fixes mensuels déjà engagés sur l’ancienne inscription ne seront pas transférés vers la nouvelle inscription. Les frais de la place de marché basés sur la consommation seront transférés.

### <a name="effective-transfer-date"></a>Date de transfert effectif

Le jour du transfert effectif peut correspondre à la date de début de l’inscription cible ou à une date ultérieure.

L’utilisation de l’inscription source est facturée dans le cadre du Paiement anticipé Azure ou comme dépassement. L’utilisation postérieure à la date de transfert effectif est transférée vers la nouvelle inscription et facturée en conséquence.

### <a name="prerequisites"></a>Prérequis

Lorsque vous demandez un transfert d’inscription, fournissez les informations suivantes :

- Pour l’inscription source, le numéro d’inscription.
- Pour l’inscription cible, le numéro de l’inscription vers laquelle effectuer le transfert.
- Pour la date de transfert effectif de l’inscription, il peut s’agir d’une date identique ou postérieure à la date de début de l’inscription cible. La date choisie ne peut pas affecter l’utilisation pour une facture de dépassement déjà émise.

Autres points à garder à l’esprit avant un transfert d’inscription :

- L’approbation des administrateurs EA pour les inscriptions source et cible est obligatoire.
- Si un transfert d’inscription ne répond pas à vos exigences, envisagez un transfert de compte.
- L’état de l’inscription source est mis à jour sur Transféré et sera disponible uniquement à des fins de création de rapports d’utilisation historiques.

### <a name="azure-prepayment"></a>Paiement anticipé Azure

Le Paiement anticipé Azure n’est pas transférable entre les inscriptions. Les soldes de Paiement anticipé Azure sont liés contractuellement à l’inscription dans laquelle ils ont été commandés. Le Paiement anticipé Azure n’est pas transféré dans le cadre du processus de transfert d’un compte ou d’une inscription.

### <a name="no-services-affected-for-account-and-enrollment-transfers"></a>Aucun service n’est affecté pour les transferts de comptes et d’inscriptions

Il n’y a aucun temps d’arrêt lors du transfert d’un compte ou d’une inscription. Il peut être effectué le même jour que votre demande si toutes les informations requises sont fournies.

## <a name="transfer-an-enterprise-subscription-to-a-pay-as-you-go-subscription"></a>Transformer un abonnement Enterprise en abonnement avec paiement à l’utilisation

Pour transformer un abonnement Enterprise en abonnement individuel avec des tarifs de paiement à l’utilisation, vous devez créer une demande de support dans le portail Azure Enterprise. Pour créer une demande de support, sélectionnez **+ Nouvelle demande de support** dans la zone **Aide et support**.

## <a name="change-azure-subscription-or-account-ownership"></a>Modifier l’abonnement Azure ou la propriété du compte

Le portail Azure EA peut transférer des abonnements d’un propriétaire de compte à un autre. Pour plus d’informations, consultez [Modifier l’abonnement Azure ou la propriété du compte](ea-portal-administration.md#change-azure-subscription-or-account-ownership).

## <a name="subscription-transfer-effects"></a>Effets d’un transfert d'abonnement

Lorsqu’un abonnement Azure est transféré vers un compte figurant dans le même locataire Azure Active Directory, tous les utilisateurs, groupes et principaux de service qui disposaient d’un [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../../role-based-access-control/overview.md) pour gérer les ressources conservent leur accès.

Pour afficher les utilisateurs disposant d’un accès RBAC à l’abonnement :

1. Sur le portail Azure, ouvrez **Abonnements**.
2. Sélectionnez l’abonnement à visualiser, puis sélectionnez **Contrôle d’accès (IAM)** .
3. Sélectionnez **Attributions de rôles**. La page des attributions de rôles liste tous les utilisateurs qui disposent d’un accès RBAC à l’abonnement.

Si l’abonnement est transféré vers un compte figurant dans un autre locataire Azure AD, tous les utilisateurs, groupes et principaux de service qui disposaient d’un contrôle [RBAC](../../role-based-access-control/overview.md) pour gérer les ressources _perdent_ leur accès. Même si l’accès RBAC n’est pas présent, l’accès à l’abonnement peut être disponible par le biais de mécanismes de sécurité, notamment :

- Certificats de gestion accordant à l’utilisateur des droits d’administrateur sur les ressources d’abonnement. Pour plus d'informations, consultez la rubrique [Créer et télécharger un certificat de gestion pour Microsoft Azure](../../cloud-services/cloud-services-certs-create.md).
- Touches d’accès rapide pour les services tels que Storage. Pour plus d’informations, consultez [Vue d’ensemble des comptes de stockage Azure](../../storage/common/storage-account-overview.md).
- Informations d’identification d’accès à distance pour les services tels que les machines virtuelles Azure.

Le destinataire doit envisager la mise à jour des secrets associés au service s’il doit restreindre l’accès à ses ressources Azure. La plupart des ressources peuvent être mises à jour en procédant comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Dans le menu Hub, sélectionnez **Toutes les ressources**.
3. Sélectionnez la ressource.
4. Dans la page des ressources, sélectionnez **Paramètres** pour voir et mettre à jour les secrets existants.

## <a name="next-steps"></a>Étapes suivantes

- Si vous avez besoin d’aide pour résoudre des problèmes rencontrés avec le portail Azure EA, consultez [Résoudre les problèmes d’accès au portail Azure EA](ea-portal-troubleshoot.md).