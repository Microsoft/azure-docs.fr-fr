---
title: Comment créer une offre Azure Application dans la Place de marché commerciale
description: Découvrez comment créer une offre Azure Application pour référencer ou vendre votre application sur la Place de marché Azure ou via le programme Fournisseur de solutions cloud (CSP), en utilisant le portail de la Place de marché commerciale dans l’Espace partenaires Microsoft.
author: aarathin
ms.author: aarathin
ms.reviewer: dannyevers
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
ms.date: 11/06/2020
ms.openlocfilehash: 3cc5e5114b435965eee4aa096e5898538b0a56e7
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94369735"
---
# <a name="how-to-create-an-azure-application-offer-in-the-commercial-marketplace"></a>Comment créer une offre Azure Application dans la Place de marché commerciale

En tant qu’éditeur publiant sur la Place de marché commerciale, vous pouvez créer une offre Azure Application de façon à ce que des clients potentiels puissent acheter votre solution. Cet article explique le processus de création d’une offre Azure Application pour la Place de marché commerciale Microsoft.

Si vous ne l’avez pas encore fait, lisez [Planifier une offre Azure Application pour la Place de marché commerciale](plan-azure-application-offer.md). Cet article fournit les ressources nécessaires et vous aide à recueillir les informations et ressources dont vous aurez besoin pour créer votre offre.

## <a name="create-a-new-offer"></a>Créer une offre

1. Connectez-vous à l’[Espace partenaires](https://partner.microsoft.com/dashboard/home).

1. Dans le menu de navigation de gauche, sélectionnez **Place de marché commerciale** > **Vue d’ensemble**.

1. Dans la page Vue d’ensemble, sélectionnez **+ Nouvelle offre** > **Application Azure**.

    ![Illustre le menu de navigation de gauche.](./media/create-new-azure-app-offer/new-offer-azure-app.png)

1. Dans la boîte de dialogue **Nouvelle offre**, entrez l’**ID de l’offre**. Il s’agit d’un identificateur unique par offre dans votre compte. Cet ID est visible dans l’URL du référencement de la place de marché commercial et des modèles Azure Resource Manager, le cas échéant. Par exemple, si vous entrez test-offer-1 dans cette zone, l’adresse web de l’offre sera `https://azuremarketplace.microsoft.com/marketplace/../test-offer-1`.

     * Chaque offre au sein de votre compte doit avoir un ID d’offre unique.
     * Utilisez uniquement des lettres minuscules et des chiffres. Il peut inclure des traits d’union et des traits de soulignement, mais pas d’espaces, et est limité à 50 caractères.
     * L’ID d’offre ne peut pas être changé une fois que vous avez sélectionné **Créer**.

1. Entrez un **Alias d’offre**. Il s’agit du nom attribué à l’offre dans l’Espace partenaires.

     * Ce nom n’est visible que dans l’Espace partenaires et diffère du nom de l’offre et d’autres valeurs présentées aux clients.
     * L’alias d’offre ne peut pas être modifié une fois que vous avez sélectionné **Créer**.

1. Pour générer l’offre et continuer, sélectionnez **Créer**.

## <a name="configure-your-azure-application-offer-setup-details"></a>Configurer les détails de configuration de votre offre Azure Application

Sous l’onglet **Configuration de l’offre**, sous **Détails de la configuration**, vous pouvez choisir de configurer un test drive. Vous allez également connecter votre système de gestion de la relation client (CRM) à votre offre de Place de marché commerciale.

### <a name="enable-a-test-drive-optional"></a>Activer une version d’évaluation (facultatif)

Une version d’évaluation constitue un excellent moyen de présenter votre offre à des clients potentiels en leur donnant accès à un environnement préconfiguré pendant un nombre fixe d’heures. L’offre d’une version d’évaluation entraîne un taux de conversion accru et génère des prospects hautement qualifiés. Pour en savoir plus sur les versions d’évaluation, consultez [Test drive](plan-azure-application-offer.md#test-drive).

#### <a name="to-enable-a-test-drive"></a>Pour activer une version d’évaluation

- Sous **Version d’évaluation**, cochez la case **Activer une version d’évaluation**.

### <a name="customer-lead-management"></a>Gestion des prospects

Connectez votre système de gestion des relations avec la clientèle (CRM) à votre offre de la place de marché commerciale pour recevoir les coordonnées d’un client qui exprime son intérêt votre produit ou déploie celui-ci.

#### <a name="to-configure-the-connection-details-in-partner-center"></a>Pour configurer les détails de connexion dans l’Espace partenaires

1. Sous **Prospects**, sélectionnez le lien **Se connecter**.
1. Dans la boîte de dialogue **Détails de la connexion**, sélectionnez une destination de prospect dans la liste.
1. Renseignez les champs qui s’affichent. Pour des instructions détaillées, consultez les articles suivants :

   - [Configurer votre offre pour envoyer des prospects à la table Azure](partner-center-portal/commercial-marketplace-lead-management-instructions-azure-table.md#configure-your-offer-to-send-leads-to-the-azure-table)
   - [Configurer votre offre pour envoyer des prospects à Dynamics 365 Customer Engagement](partner-center-portal/commercial-marketplace-lead-management-instructions-dynamics.md#configure-your-offer-to-send-leads-to-dynamics-365-customer-engagement) (anciennement Dynamics CRM en ligne)
   - [Configurer votre offre pour l’envoi des prospects au point de terminaison HTTPS](partner-center-portal/commercial-marketplace-lead-management-instructions-https.md#configure-your-offer-to-send-leads-to-the-https-endpoint)
   - [Configurer votre offre pour envoyer des prospects à Marketo](partner-center-portal/commercial-marketplace-lead-management-instructions-marketo.md#configure-your-offer-to-send-leads-to-marketo)
   - [Configurer votre offre pour envoyer des prospects à Salesforce](partner-center-portal/commercial-marketplace-lead-management-instructions-salesforce.md#configure-your-offer-to-send-leads-to-salesforce)

1. Pour valider la configuration que vous avez fournie, sélectionnez le lien **Valider**, le cas échéant.
1. Pour fermer la boîte de dialogue, sélectionnez **Connecter**.
1. Avant de passer à l’onglet suivant, sélectionnez **Enregistrer le brouillon** : Propriétés.

> [!NOTE]
> Assurez-vous que la connexion à la destination de prospect reste à jour afin de ne perdre aucun prospect. Veillez à mettre à jour ces connexions chaque fois qu’un changement a été apporté.

## <a name="next-steps"></a>Étapes suivantes

- [Configurer les propriétés de votre offre Azure Application](create-new-azure-apps-offer-properties.md)
