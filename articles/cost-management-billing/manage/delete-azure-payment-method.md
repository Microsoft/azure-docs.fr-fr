---
title: Supprimer un mode de paiement pour la facturation Azure
description: Décrit la procédure à suivre pour supprimer un mode de paiement utilisé par un abonnement Azure.
author: bandersmsft
ms.reviewer: judupont
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: how-to
ms.date: 10/07/2020
ms.author: banders
ms.openlocfilehash: 15d6c7731b541de638ceaf7828a7ce962cbf154a
ms.sourcegitcommit: d2222681e14700bdd65baef97de223fa91c22c55
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/07/2020
ms.locfileid: "91827552"
---
# <a name="delete-an-azure-billing-payment-method"></a>Supprimer un mode de paiement pour la facturation Azure

Ce document fournit des instructions pour supprimer un mode de paiement, comme une carte de crédit, de différents types d’abonnements Azure. Vous pouvez supprimer un mode de paiement pour l’un des abonnements suivants :

- Contrat client Microsoft (MCA)
- Microsoft Online Services Program (MOSP), également connu sous le nom de paiement à l’utilisation

Quel que soit votre type d’abonnement Azure, vous devez l’annuler pour pouvoir supprimer le mode de paiement qui lui est associé.

La suppression d’un mode de paiement pour d’autres types d’abonnements Azure comme le Contrat Partenaire Microsoft et l’Accord Entreprise n’est pas prise en charge.

## <a name="delete-an-mca-payment-method"></a>Supprimer un mode de paiement MCA

Seul l’utilisateur qui a créé le compte Contrat client Microsoft peut supprimer un mode de paiement.

Pour supprimer un mode de paiement pour un Contrat client Microsoft, effectuez les étapes suivantes.

1. Connectez-vous au portail Azure sur https://portal.azure.com/.
1. Accédez à **Cost Management + facturation**.
1. Si nécessaire, sélectionnez une étendue de facturation.
1. Dans la liste du menu de gauche, sous **Facturation**, sélectionnez **Profils de facturation**.  
    :::image type="content" source="./media/delete-azure-payment-method/billing-profiles.png" alt-text="Exemple de capture d’écran montrant Profils de facturation sur le portail Azure" lightbox="./media/delete-azure-payment-method/billing-profiles.png" :::
1. Dans la liste des profils de facturation, sélectionnez celui où le mode de paiement est utilisé.  
    :::image type="content" source="./media/delete-azure-payment-method/select-billing-profile.png" alt-text="Exemple d’image montrant la liste de profils de facturation" :::
1. Dans la liste du menu de gauche, sous **Paramètres**, sélectionnez **Modes de paiement**.
1. Dans la page Modes de paiement de votre profil de facturation, un tableau des modes de paiement est affiché sous la section **Vos cartes de crédit**. Recherchez la carte de crédit à supprimer, sélectionnez les points de suspension ( **...** ), puis **Supprimer**.  
    :::image type="content" source="./media/delete-azure-payment-method/delete-credit-card.png" alt-text="Exemple montrant où supprimer une carte de crédit" :::
1. La page Supprimer un mode de paiement s’affiche. Azure vérifie si le mode de paiement est en cours d’utilisation.
    - Si le mode de paiement n’est pas utilisé, l’option **Supprimer** est activée. Sélectionnez-la pour supprimer les informations de carte de crédit.
    - Si le mode de paiement est utilisé, il doit être remplacé ou détaché. Poursuivez la lecture des sections suivantes. Elles expliquent comment **détacher** le mode de paiement utilisé par votre abonnement.

### <a name="detach-payment-method-used-by-an-mca-billing-profile"></a>Détacher le mode de paiement utilisé par un profil de facturation MCA

Si votre mode de paiement est utilisé par un profil de facturation MCA, un message semblable à l’exemple suivant s’affiche.

:::image type="content" source="./media/delete-azure-payment-method/payment-method-in-use-microsoft-customer-agreement.png" alt-text="Exemple d’image montrant qu’un mode de paiement est utilisé par un Contrat client Microsoft" :::

Pour détacher un mode de paiement, une liste de conditions doit être remplie. Si une des conditions n’est pas remplie, des instructions apparaissent pour vous expliquer comment la satisfaire. De même, un lien apparaît pour vous pour diriger vers l’emplacement où vous pouvez résoudre la condition.

Dès lors que toutes les conditions sont satisfaites, vous pouvez détacher le mode de paiement du profil de facturation.

> [!NOTE]
> Une fois le mode de paiement par défaut détaché, le profil de facturation passe à l’état _inactif_. Tout ce qui est supprimé au cours de ce processus ne peut pas être récupéré. Une fois qu’un profil de facturation est défini comme étant inactif, vous devez souscrire un nouvel abonnement Azure pour créer de nouvelles ressources.

#### <a name="to-detach-a-payment-method"></a>Pour détacher un mode de paiement

1. Dans la zone Supprimer un mode de paiement, sélectionnez le lien **Détacher le mode de paiement actif**.
1. Si toutes les conditions sont remplies, sélectionnez **Détacher**. Sinon, passez à l’étape suivante.
1. Si l’option Détacher n’est pas disponible, une liste de conditions s’affiche. Effectuez les actions listées. Sélectionnez le lien figurant dans la zone Détacher le mode de paiement par défaut. Voici un exemple d’action corrective qui explique les actions que vous devez entreprendre.  
    :::image type="content" source="./media/delete-azure-payment-method/azure-subscriptions.png" alt-text="Exemple montrant l’action corrective nécessaire au détachement d’un mode de paiement pour MCA" :::
1. Quand vous sélectionnez le lien d’une action corrective, vous êtes redirigé vers la page Azure où vous pouvez effectuer l’action. Prenez toutes les mesures de correction nécessaires.
1. Si besoin, effectuez toutes les autres actions correctives.
1. Revenez à **Cost Management + Billing** > **Profils de facturation** > **Modes de paiement**. Sélectionnez **Détacher**. Au bas de la page Détacher le mode de paiement par défaut, sélectionnez **Détacher**.

> [!NOTE]
> - La suppression d’un abonnement annulé peut prendre jusqu’à 90 jours. Si vous souhaitez écourter l’attente, ouvrez une demande de support Azure et demandez à ce que l’abonnement soit supprimé immédiatement.
> - Vous ne pouvez pas supprimer un mode de paiement avant d’avoir réglé tous les frais précédents d’un profil de facturation. Si vous êtes dans une période de facturation active, vous devez attendre qu’elle soit terminée avant de pouvoir supprimer votre mode de paiement. **Vérifiez que toutes les autres conditions de détachement sont remplies en attendant que votre période de facturation soit terminée**.

## <a name="delete-a-mosp-payment-method"></a>Supprimer un mode de paiement MOSP

Vous devez être administrateur de compte pour pouvoir supprimer un mode de paiement.

Si votre mode de paiement est utilisé par un abonnement MOSP, effectuez les étapes suivantes.

1. Connectez-vous au portail Azure sur https://portal.azure.com/.
1. Accédez à **Cost Management + facturation**.
1. Si nécessaire, sélectionnez une étendue de facturation.
1. Dans la liste du menu de gauche, sous **Paramètres**, sélectionnez **Modes de paiement**.
1. Dans la zone Modes de paiement, sélectionnez la _ligne_ où figure votre mode de paiement. Ne sélectionnez pas le lien du mode de paiement. Vous n’êtes pas nécessairement informé visuellement que vous avez sélectionné le mode de paiement.
1. Sélectionnez **Supprimer**.  
    :::image type="content" source="./media/delete-azure-payment-method/delete-mosp-payment-method.png" alt-text="Exemple montrant l’action corrective nécessaire au détachement d’un mode de paiement pour MOSP" :::
1. Dans la zone Supprimer un mode de paiement, sélectionnez **Supprimer** si toutes les conditions sont remplies. Si l’option Supprimer n’est pas disponible, passez à l’étape suivante.
1. Une liste de conditions s’affiche. Effectuez les actions listées. Sélectionnez le lien figurant dans la zone Détacher un mode de paiement.  
    :::image type="content" source="./media/delete-azure-payment-method/payment-method-in-use-mosp.png" alt-text="Exemple d’image montrant qu’un mode de paiement est utilisé par un abonnement MOSP" :::
1. Quand vous sélectionnez le lien d’une action corrective, vous êtes redirigé vers la page Azure où vous pouvez effectuer l’action. Prenez toutes les mesures de correction nécessaires.
1. Si besoin, effectuez toutes les autres actions correctives.
1. Revenez à **Cost Management + Billing** > **Profils de facturation** > **Modes de paiement** et supprimez le mode de paiement.

> [!NOTE]
> La suppression d’un abonnement annulé peut prendre jusqu’à 90 jours. Si vous souhaitez écourter l’attente, ouvrez une demande de support Azure et demandez à ce que l’abonnement soit supprimé immédiatement.

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur l’annulation de votre abonnement Azure, consultez [Annuler votre abonnement Azure](cancel-azure-subscription.md).
- Pour plus d’informations sur l’ajout ou la mise à jour d’une carte de crédit, consultez [Ajouter ou mettre à jour une carte de crédit pour Azure](change-credit-card.md).