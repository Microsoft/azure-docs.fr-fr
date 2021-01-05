---
title: Créer une offre de machine virtuelle sur la Place de marché Azure.
description: Découvrez comment créer une offre de machine virtuelle sur la Place de marché commerciale Microsoft.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: mingshen-ms
ms.author: mingshen
ms.date: 10/20/2020
ms.openlocfilehash: 2ef80d26336ddbe3c015dfcde0c5ed29b762f39b
ms.sourcegitcommit: 9826fb9575dcc1d49f16dd8c7794c7b471bd3109
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2020
ms.locfileid: "94629714"
---
# <a name="how-to-create-a-virtual-machine-offer-on-azure-marketplace"></a>Comment créer une offre de machine virtuelle sur la Place de marché Azure

Cet article explique comment créer une offre de machine virtuelle Azure sur[la Place de marché Azure](https://azuremarketplace.microsoft.com/). Il s’applique aux machines virtuelles Windows et Linux qui contiennent un système d’exploitation, un disque dur virtuel (VHD) et jusqu’à 16 disques de données.

Avant de commencer, [créez un compte Place de marché commerciale dans l’Espace partenaires](partner-center-portal/create-account.md). Vérifiez que votre compte est inscrit dans le programme de Place de marché commerciale.

## <a name="before-you-begin"></a>Avant de commencer

Si vous ne l’avez pas encore fait, passez en revue [Planifier une offre de machine virtuelle](marketplace-virtual-machines.md). Elle explique les exigences techniques de votre machine virtuelle et répertorie les informations et les ressources dont vous aurez besoin lors de la création de votre offre.

## <a name="create-a-new-offer"></a>Créer une offre

1. Connectez-vous à l’[Espace partenaires](https://partner.microsoft.com/dashboard/home).
2. Dans le volet gauche, sélectionnez **Place de marché commerciale** > **Vue d’ensemble**.
3. Dans la page **Vue d’ensemble**, sélectionnez **Nouvelle offre** > **Machine virtuelle Azure**.

    ![Capture d’écran montrant les options du menu du volet gauche et le bouton « Nouvelle offre ».](./media/create-vm/new-offer-azure-virtual-machine.png)

> [!NOTE]
> Une fois votre offre publiée, les modifications que vous y apportez dans l’Espace partenaires apparaissent sur la Place de marché Azure uniquement après la republication de l’offre. Veillez à toujours republier une offre après y avoir apporté des modifications.

Entrez un **ID d’offre**. Il s’agit d’un identificateur unique par offre dans votre compte.

- Cet ID est visible par les clients dans l’adresse web de l’offre de la Place de marché Azure ainsi que dans Azure PowerShell et Azure CLI, le cas échéant.
- Utilisez uniquement des lettres minuscules et des chiffres. L’ID peut inclure des traits d’union et des traits de soulignement, mais pas d’espaces, et est limité à 50 caractères. Par exemple, si vous entrez **test-offer-1**, l’adresse web de l’offre sera `https://azuremarketplace.microsoft.com/marketplace/../test-offer-1`.
- L’ID d’offre ne peut pas être changé une fois que vous avez sélectionné **Créer**.

Entrez un **Alias d’offre**. L’alias d’offre est le nom attribué à l’offre dans l’Espace partenaires.

- Ce nom n’est pas utilisé sur la Place de marché Azure. Il est différent du nom de l’offre et des autres valeurs présentées aux clients.

Sélectionnez **Créer** pour générer l’offre et continuer. Espace partenaires ouvre la page **Configuration de l’offre**.

## <a name="enable-a-test-drive-optional"></a>Activer une version d’évaluation (facultatif)

Une version d’évaluation constitue un excellent moyen de présenter votre offre à des clients potentiels en leur donnant accès à un environnement préconfiguré pendant un nombre fixe d’heures. L’offre d’une version d’évaluation entraîne un taux de conversion accru et génère des prospects hautement qualifiés. Pour en savoir plus sur les versions d’évaluation, consultez [Qu’est-ce qu’une version d’évaluation ?](./what-is-test-drive.md).

> [!TIP]
> Une version d’évaluation n’est pas un essai gratuit. Vous pouvez proposer une version d’évaluation, un essai gratuit ou les deux. Dans les deux cas, vos clients disposent de votre solution pendant une période fixe. Cependant, une version d’évaluation propose en outre une visite autoguidée concrète des principales fonctionnalités de votre produit, ainsi qu’une démonstration des avantages de celui-ci au travers d’un scénario d’implémentation réel.

Pour activer une version d'évaluation, cochez la case **Activer une version d'évaluation**. Vous configurerez ultérieurement la version d'évaluation. La version d'évaluation nécessite la configuration d’un système CRM (voir la section suivante).

## <a name="configure-customer-leads-management"></a>Configurer la gestion des prospects

Quand vous publiez une offre sur la Place de marché commerciale avec l’Espace partenaires, connectez-la à votre système de gestion des relations avec la clientèle (CRM). Cela vous permet de recevoir les coordonnées des clients dès qu’une personne exprime son intérêt ou utilise votre produit. La connexion à un CRM est obligatoire si vous souhaitez activer une version d’évaluation (voir la section précédente). Dans le cas contraire, la connexion à un CRM est facultative.

1. Sous **Prospects**, sélectionnez le lien **Se connecter**.
1. Dans la boîte de dialogue **Détails de la connexion**, sélectionnez une destination de prospect.
1. Renseignez les champs qui s’affichent. Pour des instructions détaillées, consultez les articles suivants :

   - [Configurer votre offre pour envoyer des prospects à la table Azure](./partner-center-portal/commercial-marketplace-lead-management-instructions-azure-table.md#configure-your-offer-to-send-leads-to-the-azure-table)
   - [Configurer votre offre pour envoyer des prospects à Dynamics 365 Customer Engagement](./partner-center-portal/commercial-marketplace-lead-management-instructions-dynamics.md#configure-your-offer-to-send-leads-to-dynamics-365-customer-engagement) (anciennement Dynamics CRM en ligne)
   - [Configurer votre offre pour l’envoi des prospects au point de terminaison HTTPS](./partner-center-portal/commercial-marketplace-lead-management-instructions-https.md#configure-your-offer-to-send-leads-to-the-https-endpoint)
   - [Configurer votre offre pour envoyer des prospects à Marketo](./partner-center-portal/commercial-marketplace-lead-management-instructions-marketo.md#configure-your-offer-to-send-leads-to-marketo)
   - [Configurer votre offre pour envoyer des prospects à Salesforce](./partner-center-portal/commercial-marketplace-lead-management-instructions-salesforce.md#configure-your-offer-to-send-leads-to-salesforce)

1. Pour valider la configuration que vous avez fournie, sélectionnez le lien **Valider**.
1. Sélectionnez **Connecter**.

Si vous effectuez des modifications, sélectionnez **Enregistrer le brouillon** avant de poursuivre vers le prochaine tabulation du menu de navigation gauche **Propriétés**.

## <a name="next-steps"></a>Étapes suivantes

- [Comment configurer les propriétés d’une offre de machine virtuelle](azure-vm-create-properties.md)
- [Bonnes pratiques pour le référencement des offres](gtm-offer-listing-best-practices.md)