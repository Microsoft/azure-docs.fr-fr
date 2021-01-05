---
title: Créer une offre SaaS dans la place de marché commerciale de Microsoft
description: Découvrez comment créer une offre SaaS (Software as a service) à référencer ou vendre dans Microsoft AppSource, la Place de marché Azure, ou par le biais du programme de fournisseur de solutions cloud (CSP) en utilisant le portail de la place de marché commerciale dans l’Espace partenaires de Microsoft.
author: mingshen-ms
ms.author: mingshen
ms.reviewer: dannyevers
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
ms.date: 09/02/2020
ms.openlocfilehash: f689993ce56a1125a1d1de8f65ce05d01f776ea9
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93130066"
---
# <a name="how-to-create-a-saas-offer-in-the-commercial-marketplace"></a>Créer une offre SaaS dans la place de marché commerciale

En tant qu’éditeur de la place de marché commerciale, vous pouvez créer une offre SaaS afin que des clients potentiels puissent acheter votre solution technique basée sur SaaS. Cet article explique le processus de création d’une offre SaaS pour la place de marché commerciale de Microsoft.

## <a name="before-you-begin"></a>Avant de commencer

Si vous ne l’avez pas déjà fait, lisez [Planifier une offre SaaS pour la place de marché commerciale](plan-saas-offer.md). Cet article explique les exigences techniques liées à votre application SaaS, ainsi que les informations et ressources dont vous aurez besoin pour créer votre offre. À moins que vous n’envisagez de publier un référencement simple (option de référencement **Me contacter** ) dans la place de marché commerciale, votre application SaaS doit répondre à des exigences techniques relatives à l’authentification.

## <a name="create-a-new-saas-offer"></a>Créer une nouvelle offre SaaS

1. Connectez-vous à l’[Espace partenaires](https://partner.microsoft.com/dashboard/home).
1. Dans le menu de navigation de gauche, sélectionnez **Place de marché commerciale** > **Vue d’ensemble**.
1. Sous l’onglet **Vue d’ensemble** , sélectionnez **+ Nouvelle offre** > **SaaS**.

   :::image type="content" source="media/new-offer-saas.png" alt-text="Illustre le menu de navigation gauche et la liste Nouvelle offre.":::

1. Dans la boîte de dialogue **Nouvelle offre** , entrez l’ **ID de l’offre**. Cet ID est visible dans l’URL du référencement de la place de marché commercial et des modèles Azure Resource Manager, le cas échéant. Par exemple, si vous entrez **test-offer-1** dans cette zone, l’adresse web de l’offre sera `https://azuremarketplace.microsoft.com/marketplace/../test-offer-1`.
   + Chaque offre au sein de votre compte doit avoir un ID d’offre unique.
   + Utilisez uniquement des lettres minuscules et des chiffres. Il peut inclure des traits d’union et des traits de soulignement, mais pas d’espaces, et est limité à 50 caractères.
   + Vous ne pouvez pas modifier L’ID de l’offre une fois que vous avez sélectionné **Créer**.

1. Entrez un **Alias d’offre**. Il s’agit du nom attribué à l’offre dans l’Espace partenaires.

   + Ce nom n’est pas visible dans la place de marché commerciale, et diffère du nom de l’offre et d’autres valeurs présentées aux clients.
   + Vous ne pouvez pas modifier l’alias de l’offre une fois que vous avez sélectionné **Créer**.
1. Pour générer l’offre et continuer, sélectionnez **Créer**.

## <a name="configure-your-saas-offer-setup-details"></a>Définit les détails de configuration de votre offre SaaS

Sous l’onglet **Configuration de l’offre** , sous **Détails de la configuration** , indiquez si vous voulez vendre votre offre par le biais de Microsoft ou de gérer vos transactions individuellement. Les offres vendues par le biais de Microsoft sont appelées _offres pouvant faire l’objet d’une transaction_ , ce qui signifie que Microsoft facilite l’échange d’argent contre une licence logicielle pour le compte de l’éditeur. Pour plus d’informations sur ces options, consultez [Options de référencement](plan-saas-offer.md#listing-options) puis [Déterminer votre option de publication](determine-your-listing-type.md).

1. Pour vendre par le biais de Microsoft et faire en sorte que nous facilitions les transactions pour vous, sélectionnez **Oui**. Accédez à [Activer une version d’évaluation](#enable-a-test-drive-optional).

1. Pour référencer votre offre sur la place de marché commerciale et traiter les transactions de façon indépendante, sélectionnez **Non** , puis effectuez l’une des opérations suivantes :
   + Pour fournir un abonnement gratuit à votre offre, sélectionnez **Obtenir maintenant (gratuit)** . Ensuite, dans la zone **URL de l’offre** qui s’affiche, entrez l’URL (commençant par *http* ou *https* ), à laquelle les clients peuvent obtenir un essai via l’ [authentification en un clic à l’aide d’Azure Active Directory (Azure AD)](azure-ad-saas.md). Par exemple : `https://contoso.com/saas-app`.
   + Pour proposer un essai gratuit de 30 jours, sélectionnez **Essai gratuit** , puis, dans la zone **URL de la version d’essai** qui s’affiche, entrez l’URL (commençant par *http* ou *https* ) à laquelle les clients peuvent accéder à votre essai gratuit via l’ [authentification en un clic à l’aide d’Azure Active Directory (Azure AD)](azure-ad-saas.md). Par exemple : `https://contoso.com/trial/saas-app`.
   + Pour que les clients potentiels vous contactent pour acheter votre offre, sélectionnez **Me contacter**.

### <a name="enable-a-test-drive-optional"></a>Activer une version d’évaluation (facultatif)

Une version d’évaluation constitue un excellent moyen de présenter votre offre à des clients potentiels en leur donnant accès à un environnement préconfiguré pendant un nombre fixe d’heures. L’offre d’une version d’évaluation entraîne un taux de conversion accru et génère des prospects hautement qualifiés. Pour en savoir plus sur les versions d’évaluation, consultez [Qu’est-ce qu’une version d’évaluation ?](./what-is-test-drive.md).

> [!TIP]
> Une version d’évaluation n’est pas un essai gratuit. Vous pouvez proposer une version d’évaluation, un essai gratuit ou les deux. Dans les deux cas, vos clients disposent de votre solution pendant une période fixe. Cependant, une version d’évaluation propose en outre une visite autoguidée concrète des principales fonctionnalités de votre produit, ainsi qu’une démonstration des avantages de celui-ci au travers d’un scénario d’implémentation réel.

**Pour activer une version d’évaluation**
1.  Sous **Version d’évaluation** , cochez la case **Activer une version d’évaluation**.
1.  Sélectionnez le type de version d’évaluation dans la liste qui s’affiche.

### <a name="configure-lead-management"></a>Configurer la gestion des prospects

Connectez votre système de gestion des relations avec la clientèle (CRM) à votre offre de la place de marché commerciale pour recevoir les coordonnées d’un client qui exprime son intérêt votre produit ou déploie celui-ci. Vous pouvez modifier cette connexion à tout moment pendant ou après la création de l’offre.

> [!NOTE]
> Vous devez configurer la gestion des prospects si vous vendez votre offre par le biais de Microsoft ou si vous avez sélectionné l’option de référencement **Me contacter**. Pour obtenir des instructions détaillées, consultez [Prospects de votre offre de marketplace commercial](partner-center-portal/commercial-marketplace-get-customer-leads.md).

#### <a name="to-configure-the-connection-details-in-partner-center"></a>Pour configurer les détails de connexion dans l’Espace partenaires

1.  Sous **Prospects** , sélectionnez le lien **Se connecter**.
1. Dans la boîte de dialogue **Détails de la connexion** , sélectionnez une destination de prospect dans la liste.
1. Renseignez les champs qui s’affichent. Pour des instructions détaillées, consultez les articles suivants :

   - [Configurer votre offre pour envoyer des prospects à la table Azure](./partner-center-portal/commercial-marketplace-lead-management-instructions-azure-table.md#configure-your-offer-to-send-leads-to-the-azure-table)
   - [Configurer votre offre pour envoyer des prospects à Dynamics 365 Customer Engagement](./partner-center-portal/commercial-marketplace-lead-management-instructions-dynamics.md#configure-your-offer-to-send-leads-to-dynamics-365-customer-engagement) (anciennement Dynamics CRM en ligne)
   - [Configurer votre offre pour l’envoi des prospects au point de terminaison HTTPS](./partner-center-portal/commercial-marketplace-lead-management-instructions-https.md#configure-your-offer-to-send-leads-to-the-https-endpoint)
   - [Configurer votre offre pour envoyer des prospects à Marketo](./partner-center-portal/commercial-marketplace-lead-management-instructions-marketo.md#configure-your-offer-to-send-leads-to-marketo)
   - [Configurer votre offre pour envoyer des prospects à Salesforce](./partner-center-portal/commercial-marketplace-lead-management-instructions-salesforce.md#configure-your-offer-to-send-leads-to-salesforce)

1. Pour valider la configuration que vous avez fournie, sélectionnez le lien **Valider**.
1. Pour fermer la boîte de dialogue, sélectionnez **OK**.

## <a name="next-steps"></a>Étapes suivantes

- [Comment configurer les propriétés de votre offre SaaS](create-new-saas-offer-properties.md)