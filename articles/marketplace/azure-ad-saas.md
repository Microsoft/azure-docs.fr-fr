---
title: Azure Active Directory et offres SaaS pouvant faire l’objet d’une transaction dans la Place de marché commerciale
description: Découvrez comment Azure Active Directory fonctionne avec les offres SaaS pouvant faire l’objet d’une transaction dans la Place de marché commerciale Microsoft.
author: mingshen-ms
ms.author: mingshen
ms.reviewer: dannyevers
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 09/04/2020
ms.openlocfilehash: 674f267d3d99dd22c1ae06b6d32587761d5983ce
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93124915"
---
# <a name="azure-ad-and-transactable-saas-offers-in-the-commercial-marketplace"></a>Azure AD et offres SaaS pouvant faire l’objet d’une transaction dans la Place de marché commerciale

Le service de gestion de l’accès et des identités basé sur le cloud de Microsoft, [Azure Active Directory](../active-directory/fundamentals/active-directory-whatis.md) (Azure AD), permet à vos employés de se connecter et d’accéder aux ressources internes et externes. Dans la Place de marché commerciale Microsoft, Azure AD permet à tous les utilisateurs, y compris les éditeurs, les acheteurs et les utilisateurs, de gérer plus facilement et en toute sécurité les offres SaaS pouvant faire l’objet d’une transaction. Avec Azure AD, les éditeurs peuvent automatiser l’approvisionnement des utilisateurs sur leurs applications SaaS (Software as a Service), et les acheteurs eux-mêmes peuvent gérer ces utilisateurs approvisionnés. 

De plus, l’[authentification unique Azure AD](../active-directory/manage-apps/what-is-single-sign-on.md) (SSO) constitue la méthode la plus sécurisée et la plus pratique lorsque les utilisateurs se connectent aux applications dans Azure AD. Un engagement plus rapide et des expériences optimisées donnent également à l’acheteur et l’utilisateur suffisamment de confiance pour gérer la toute première interaction avec l’application SaaS d’un éditeur. Cela donne une impression positive qui crée une visibilité et encourage les entreprises à effectuer de nouvelles transactions.

Les instructions de cet article vous aideront à certifier votre offre SaaS dans la Place de marché commerciale. Pour plus d’informations sur la certification, consultez les [stratégies de certification de la Place de marché commerciale](/legal/marketplace/certification-policies#100-general) détaillées, y compris celles [spécifiques à SaaS](/legal/marketplace/certification-policies#1000-software-as-a-service-saas).

## <a name="before-you-begin"></a>Avant de commencer

Lorsque vous [créez votre offre SaaS](./create-new-saas-offer.md) dans l’Espace partenaires, vous pouvez choisir parmi un ensemble d’options de référencement spécifiques qui seront affichées dans le référencement de l’offre. Votre choix détermine la manière dont votre offre est traitée dans la Place de marché commerciale. Les offres vendues par le biais de Microsoft sont appelées offres avec transaction. Nous facturons le client en votre nom pour toutes les offres avec transaction. Si vous choisissez de vendre par le biais de Microsoft et que vous avez des transactions hôtes en votre nom (l’option **Oui** ), alors vous avez choisi de créer une offre avec transaction et cet article vous concerne. Nous vous recommandons de le lire dans son intégralité.

Si vous choisissez de répertorier uniquement votre offre par le biais de la place de marché commerciale et de traiter les transactions indépendamment (option **Non** ), les clients potentiels pourront accéder à votre offre de trois façons : Obtenir maintenant (gratuit), Essai gratuit et Me contacter. Si vous sélectionnez **Obtenir maintenant (gratuit)** ou **Essai gratuit** , cet article ne vous concerne pas. Consultez plutôt l’article [Créer la page d’accueil de votre offre SaaS gratuite ou d’évaluation dans la Place de marché commerciale](./azure-ad-free-or-trial-landing-page.md) pour plus d’informations. Si vous sélectionnez **Me contacter** , il n’y a aucune responsabilité directe pour l’éditeur. Continuez à créer votre offre dans l’Espace partenaires.

## <a name="how-azure-ad-works-with-the-commercial-marketplace-for-saas-offers"></a>Fonctionnement d’Azure AD dans la Place de marché commerciale pour les offres SaaS

Azure AD permet d’acheter, de traiter et de gérer facilement des solutions dans la Place de marché commerciale. La figure 1 montre comment l’éditeur, l’acheteur et l’utilisateur interagissent pour acheter et activer un abonnement. Elle montre également comment les clients utilisent et gèrent les applications SaaS qu’ils obtiennent à partir de la Place de marché commerciale. Dans le cadre de cette illustration, l’acheteur est l’utilisateur de l’application SaaS qui initie un achat à partir de la Place de marché commerciale.

Comme le montre la figure 1, lorsqu’un acheteur sélectionne votre offre, il lance une chaîne de workflows incluant l’achat, l’abonnement et la gestion des utilisateurs. Dans cette chaîne, vous devez, en tant qu’éditeur, respecter certaines exigences, et Microsoft vous fournit une assistance à des étapes clés.

**_Figure 1 : Utilisation d’Azure AD pour des offres SaaS dans la Place de marché commerciale_* _

:::image type="content" source="./media/azure-ad-saas/azure-ad-saas-flow.png" alt-text="Illustre la gestion des achats, la gestion des abonnements et les étapes facultatives du processus de gestion des utilisateurs.":::

Les sections suivantes fournissent des détails sur la configuration requise pour chaque étape du processus.

## <a name="process-steps-for-purchase-management"></a>Étapes de processus pour la gestion des achats

Cette illustration montre les quatre étapes du processus de gestion des achats.

:::image type="content" source="./media/azure-ad-saas/azure-ad-saas-flow-1-4.png" alt-text="Illustre les quatre étapes du processus de gestion des achats.":::

Ce tableau fournit des détails sur les étapes du processus de gestion des achats.

| Étape du processus | Action de l’éditeur | Recommandé ou requis pour les éditeurs |
| ------------ | ------------- | ------------- |
| 1. L’acheteur se connecte à la place de marché commerciale avec son identité Azure ID et sélectionne une offre SaaS. | Aucune action n’est requise de la part de l’éditeur. | Non applicable |
| 2. Après l’achat, l’acheteur sélectionne _ *Configurer un compte* * dans la Place de marché Azure ou **Configurer maintenant** dans AppSource, qui dirige l’acheteur vers la page d’accueil de l’éditeur pour cette offre. L’acheteur doit pouvoir se connecter à l’application SaaS de l’éditeur avec Azure AD SSO et uniquement donner un consentement minimal qui ne nécessite pas d’approbation de l’administrateur Azure AD. | Concevez une [page d’accueil](azure-ad-transactable-saas-landing-page.md) pour l’offre permettant d’accueillir un utilisateur avec son identité Azure AD ou compte Microsoft (MSA) et qui facilite tout approvisionnement ou configuration supplémentaire nécessaire. | Obligatoire |
| 3. L’éditeur demande les détails de l’achat à l’API de traitement SaaS. | À l’aide d’un [jeton d’accès](./partner-center-portal/pc-saas-registration.md) généré à partir de l’ID d’application de la page d’accueil, [appelez le point de terminaison de résolution](./partner-center-portal/pc-saas-fulfillment-api-v2.md#resolve-a-purchased-subscription) pour obtenir des détails sur l’achat. | Obligatoire |
| 4. Via Azure AD et l’API Microsoft Graph, l’éditeur recueille les détails de l’entreprise et de l’utilisateur requis pour approvisionner l’acheteur dans l’application SaaS de l’éditeur.  | Décomposez le jeton d’utilisateur Azure AD pour rechercher le nom et l’adresse e-mail, ou [appelez l’API Microsoft Graph](/graph/use-the-api) et utilisez des autorisations déléguées pour [récupérer des informations](/graph/api/user-get) sur l’utilisateur connecté. | Obligatoire |
||||

## <a name="process-steps-for-subscription-management"></a>Étapes du processus pour la gestion des abonnements

Cette illustration montre les deux étapes du processus de gestion des abonnements.

:::image type="content" source="./media/azure-ad-saas/azure-ad-saas-flow-5-6.png" alt-text="Illustre les deux étapes du processus de gestion des abonnements.":::

Ce tableau décrit les étapes du processus de gestion des abonnements.

| Étape du processus | Action de l’éditeur | Recommandé ou requis pour les éditeurs |
| ------------ | ------------- | ------------- |
| 5. L’éditeur gère l’abonnement à l’application SaaS par le biais de l’API de traitement SaaS. | Gérez les modifications d’abonnement et les autres tâches de gestion via les [API de traitement SaaS](./partner-center-portal/pc-saas-fulfillment-api-v2.md).<br><br>Cette étape nécessite un jeton d’accès, comme décrit dans l’étape 3 du processus. | Obligatoire |
| 6. Si vous utilisez la facturation à l’usage, l’éditeur envoie des événements d’utilisation à l’API du service de mesure. | Si votre application SaaS intègre une facturation à l’utilisation, effectuez des notifications d’utilisation par le biais des [API du service de mesure de la Place de marché](./partner-center-portal/marketplace-metering-service-apis.md).<br><br>Cette étape nécessite un jeton d’accès, comme décrit dans l’étape 3. | Requis pour la mesure |
||||

## <a name="process-steps-for-user-management"></a>Étapes de processus pour la gestion des utilisateurs

Cette illustration montre les trois étapes du processus de gestion des utilisateurs.

:::image type="content" source="./media/azure-ad-saas/azure-ad-saas-flow-7-9.png" alt-text="Illustre les trois étapes de processus facultatives pour la gestion des utilisateurs.":::

Les étapes de processus 7 à 9 sont des étapes facultatives du processus de gestion des utilisateurs. Elles offrent des avantages supplémentaires pour les éditeurs qui prennent en charge l’authentification unique (SSO) Azure AD. Ce tableau décrit les étapes du processus de gestion des utilisateurs.

| Étape du processus | Action de l’éditeur | Recommandé ou requis pour les éditeurs |
| ------------ | ------------- | ------------- |
| 7. Les administrateurs Azure AD de la société de l’acheteur peuvent éventuellement gérer l’accès des utilisateurs et des groupes via Azure AD. | Aucune action n’est requise de la part de l’éditeur pour activer cet accès si l’authentification unique Azure AD est configurée pour les utilisateurs (étape 9). | Non applicable |
| 8. Le service d’approvisionnement Azure AD échange les modifications entre Azure AD et l’application SaaS de l’éditeur. | [Implémentez un de point de terminaison SCIM](../active-directory/app-provisioning/use-scim-to-provision-users-and-groups.md) pour recevoir les mises à jour d’Azure AD à mesure que des utilisateurs sont ajoutés et supprimés. | Recommandé |
| 9. Une fois l’application autorisée et approvisionnée, les utilisateurs de la société de l’acheteur peuvent utiliser Azure AD SSO pour se connecter à l’application SaaS de l’éditeur. | [Utilisez Azure AD SSO](../active-directory/manage-apps/what-is-single-sign-on.md) pour permettre aux utilisateurs de se connecter une seule fois avec un compte à l’application SaaS de l’éditeur. | Recommandé |
||||

## <a name="next-steps"></a>Étapes suivantes

- [Créer la page d'accueil de votre offre SaaS avec transaction dans la Place de marché commerciale](azure-ad-transactable-saas-landing-page.md)
- [Créer la page d'accueil de votre offre SaaS gratuite ou d’évaluation dans la Place de marché commerciale](azure-ad-free-or-trial-landing-page.md)
- [Créer une offre SaaS dans la place de marché commerciale](create-new-saas-offer.md)