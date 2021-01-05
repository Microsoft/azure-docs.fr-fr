---
title: Résoudre les problèmes d’affichage de votre compte de facturation dans le portail Azure
description: Cet article vous aide à résoudre les problèmes que vous pouvez rencontrer quand vous essayez d’afficher votre compte de facturation dans le portail Azure.
author: amberbhargava
ms.reviewer: banders
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: conceptual
ms.date: 08/20/2020
ms.author: banders
ms.openlocfilehash: 71561f6ecdde09f259e54260e7bab4dcc0e85c98
ms.sourcegitcommit: 33368ca1684106cb0e215e3280b828b54f7e73e8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92132633"
---
# <a name="troubleshoot-viewing-your-billing-account-in-the-azure-portal"></a>Résoudre les problèmes d’affichage de votre compte de facturation dans le portail Azure

Un compte de facturation est créé lorsque vous vous inscrivez pour utiliser Azure. Votre compte de facturation vous permet de gérer vos factures et paiements, et de suivre les coûts. Vous pouvez accéder à plusieurs comptes de facturation. Par exemple, vous pouvez vous être inscrit à Azure pour une utilisation personnelle. Vous pouvez aussi accéder à Azure via le Contrat Entreprise de votre organisation ou le Contrat client Microsoft. Pour chacun de ces scénarios, vous disposez d’un compte de facturation distinct. Cet article vous aide à résoudre les problèmes que vous pouvez rencontrer quand vous essayez d’afficher votre compte de facturation dans le portail Azure.

Vous pouvez afficher vos comptes de facturation dans la page [Azure Cost Management + Facturation](https://portal.azure.com/#blade/Microsoft_Azure_GTM/ModernBillingMenuBlade).

Pour plus d’informations sur les comptes de facturation et savoir comment identifier le type de votre compte de facturation, consultez [Afficher les comptes de facturation dans le portail Azure](view-all-accounts.md).

Si vous ne parvenez pas à voir votre compte de facturation dans le portail Azure, essayez les options suivantes :

## <a name="sign-in-to-a-different-tenant"></a>Connectez-vous à un autre locataire

Votre compte de facturation est associé à un seul locataire Azure Active Directory. Votre compte de facturation n’apparaît pas dans la page Cost Management + Facturation si vous êtes connecté à un locataire incorrect. Procédez comme suit pour basculer vers un autre locataire dans le portail Azure et afficher vos comptes de facturation dans ce locataire.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sélectionnez votre profil (adresse e-mail) en haut à droite de la page.
1. Sélectionnez **Changer de répertoire**.  
    ![Capture d’écran montrant la sélection du changement de répertoire dans le portail](./media/troubleshoot-account-not-found/select-switch-directory.png)
1. Sélectionnez un répertoire sous la section **Tous les répertoires**.  
    ![Capture d’écran montrant la sélection d’un répertoire dans le portail](./media/troubleshoot-account-not-found/select-directory.png)

## <a name="sign-in-with-a-different-email-address"></a>Connectez-vous avec une autre adresse e-mail

Certains utilisateurs disposent de plusieurs adresses e-mail pour se connecter au [portail Azure](https://portal.azure.com). Toutes les adresses e-mail n’ont pas accès à un compte de facturation. Si vous vous connectez avec une adresse e-mail qui dispose d’autorisations pour gérer les ressources, mais qui ne dispose pas d’autorisations pour afficher un compte de facturation, ce dernier n’apparaît pas dans la page [Cost Management + Facturation](https://portal.azure.com/#blade/Microsoft_Azure_GTM/ModernBillingMenuBlade) du portail Azure.

Connectez-vous au portail Azure avec une adresse e-mail disposant d’une autorisation sur le compte de facturation pour accéder à ce dernier.

## <a name="sign-in-with-a-different-identity"></a>Connectez-vous avec une autre identité

Certains utilisateurs ont deux identités avec la même adresse e-mail : un compte professionnel ou scolaire et un compte personnel. En règle générale, une seule de leurs identités dispose des autorisations d’affichage d’un compte de facturation. Vous pouvez avoir deux identités avec une seule adresse e-mail. Quand vous vous connectez avec une identité qui n’est pas autorisée à afficher un compte de facturation, vous ne voyez pas ce dernier dans la page [Cost Management + Facturation](https://portal.azure.com/#blade/Microsoft_Azure_GTM/ModernBillingMenuBlade). Pour changer votre identité, procédez comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com) dans une fenêtre InPrivate/Incognito.
1. Si votre adresse e-mail a deux identités, vous voyez une option permettant de sélectionner un compte personnel ou un compte professionnel ou scolaire. Sélectionnez l’un des comptes.
1. Si vous ne voyez pas le compte de facturation dans la page Cost Management + Facturation du portail Azure, répétez les étapes 1 et 2, puis sélectionnez l’autre identité.

## <a name="contact-us-for-help"></a>Contactez-nous pour obtenir de l'aide

Si vous avez des questions ou besoin d’aide, [créez une demande de support](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

## <a name="next-steps"></a>Étapes suivantes

Lisez les articles suivants sur la facturation et les abonnements pour vous aider à résoudre les problèmes.

- [Carte refusée](./troubleshoot-declined-card.md)
- [Problèmes de connexion à l’abonnement](./troubleshoot-sign-in-issue.md)
- [Aucun abonnement trouvé](./no-subscriptions-found.md)
- [Désactivation de l’affichage des coûts d’entreprise](./enterprise-mgmt-grp-troubleshoot-cost-view.md)