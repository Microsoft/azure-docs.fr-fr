---
title: E-mail de solde dû de la part d’Azure
description: Décrit comment procéder au paiement si votre abonnement Azure présente un solde impayé.
author: genlin
ms.reviewer: dcscontentpm
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: how-to
ms.date: 08/20/2020
ms.author: banders
ms.openlocfilehash: 3c42db58017dc3a692c227f6eb454cf7b597e77b
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91297582"
---
# <a name="resolve-past-due-balance-for-your-azure-subscription"></a>Résoudre un solde impayé pour votre abonnement Azure

Cet article s’applique aux clients qui se sont inscrits à Azure Online avec une carte de crédit, et qui disposent d’un compte de facturation Programme Microsoft Online Services. Découvrez comment [vérifier votre type de compte de facturation](#check-the-type-of-your-account). Si vous avez un compte de facturation Contrat client Microsoft, consultez [Payer votre facture pour Microsoft Azure](../understand/pay-bill.md) à la place.

Si votre paiement n’a pas été reçu ou si nous ne pouvons pas traiter votre paiement, vous recevrez un e-mail et verrez une alerte sur le portail Azure vous informant que l’échéance du paiement de cet abonnement est dépassée. Si votre mode de paiement par défaut est la carte de crédit, l’[administrateur du compte](../understand/subscription-transfer.md#whoisaa) peut régler ces frais en souffrance sur le portail Azure. Si vous payez par facture (chèque/virement bancaire), envoyez votre paiement à l’adresse indiquée au bas de la facture.

> [!IMPORTANT]
> * Si vous avez plusieurs abonnements associés à la même carte de crédit et qu’ils sont tous en retard de paiement, vous devez payer l’intégralité du solde restant dû en une seule fois.
> * La carte de crédit utilisée pour régler les frais en souffrance deviendra le nouveau mode de paiement par défaut pour tous les abonnements qui utilisaient le mode de paiement ayant échoué.

## <a name="resolve-past-due-balance-in-the-azure-portal"></a>Régler le solde restant dû dans le portail Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com) en tant qu’administrateur de compte.
1. Recherchez **Gestion des coûts + facturation**.
1. Sélectionnez l’abonnement dont le paiement est dû dans la page **Vue d’ensemble**.
1. Dans la page **Vue d’ensemble de l’abonnement**, cliquez sur la bannière rouge notifiant le retard pour régler le solde.
    > [!NOTE]
    > Si vous n’êtes pas l’administrateur du compte, vous ne pourrez pas régler le solde.
1. Dans la nouvelle page **Régler le solde**, cliquez sur **Sélectionner le mode de paiement**.
    ![Capture d’écran montrant le lien de sélection du mode de paiement](./media/resolve-past-due-balance/settle-balance-screen.png)

1. Dans le nouveau panneau à droite, sélectionnez une carte de crédit dans la liste déroulante ou ajoutez-en une en cliquant sur le lien bleu **Ajouter un nouveau mode de paiement**. Cette carte de crédit deviendra le mode de paiement actif pour tous les abonnements utilisant actuellement le mode de paiement ayant échoué.
     > [!NOTE]
     > * Le solde restant dû total reflète les frais en attente de paiement sur tous les services Microsoft utilisant le mode de paiement ayant échoué.
     > * Si le mode de paiement sélectionné a également des frais en souffrance pour les services Microsoft, cela apparaîtra dans le solde restant dû total. Vous devez également payer ces frais en souffrance.
1. Cliquez sur **Payer**.

## <a name="troubleshoot-declined-credit-card"></a>Résoudre les problèmes de cartes de crédit refusées

Si un paiement effectué par carte de crédit est refusé par votre institution financière, contactez cette dernière pour résoudre le problème. Contactez votre banque pour vérifier que :
- Les transactions internationales sont activées sur la carte.
- La carte a une limite de crédit ou des fonds suffisants pour régler le solde.
- Les paiements récurrents sont activés sur la carte.

## <a name="not-getting-billing-email-notifications"></a>Vous ne recevez pas les notifications de facturation par e-mail ?

Si vous êtes administrateur de compte, [vérifiez quelle adresse e-mail est utilisée pour les notifications](change-azure-account-profile.md). Nous vous recommandons d’utiliser une adresse e-mail que vous consultez régulièrement. Si l’adresse e-mail est correcte, consultez votre dossier de courrier indésirable.

## <a name="if-i-forget-to-pay-what-happens"></a>Que se passe-t-il si j’oublie de payer ?

Le service est annulé et vos ressources ne sont plus disponibles. Vos données Azure sont supprimées 90 jours après l’arrêt du service. Pour en savoir plus, consultez [Microsoft Trust Center - How we manage your data](https://go.microsoft.com/fwLink/p/?LinkID=822930&clcid=0x409) (Centre de gestion de la confidentialité de Microsoft - Comment nous gérons vos données).

Si vous savez que votre paiement a été traité, alors que votre abonnement est toujours désactivé, contactez le [support Azure](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade).

## <a name="check-the-type-of-your-account"></a>Vérifier le type de votre compte
[!INCLUDE [billing-check-mca](../../../includes/billing-check-account-type.md)]

## <a name="need-help-contact-us"></a>Vous avez besoin d’aide ? Contactez-nous.

Si vous avez des questions ou besoin d’aide, [créez une demande de support](https://go.microsoft.com/fwlink/?linkid=2083458).
