---
title: S’inscrire à Azure avec un compte Microsoft 365
description: Apprenez à créer un abonnement Azure avec un compte Microsoft 365. Vous pouvez également associer des comptes Azure et Microsoft 365 existants entre eux.
author: JiangChen79
ms.reviewer: adwise
tags: billing,top-support-issue
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: conceptual
ms.date: 08/20/2020
ms.author: banders
ms.openlocfilehash: 255fbe4036c83618dd0131b262d13709ab7f9c2f
ms.sourcegitcommit: 33368ca1684106cb0e215e3280b828b54f7e73e8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92131885"
---
# <a name="sign-up-for-an-azure-subscription-with-your-microsoft-365-account"></a>Souscrire un abonnement Azure avec un compte Microsoft 365

Si vous avez un abonnement Microsoft 365, vous pouvez utiliser votre compte Microsoft 365 pour créer un abonnement Azure. Connectez-vous au [portail Azure](https://portal.azure.com/) avec votre nom d’utilisateur et votre mot de passe Microsoft 365. Si vous souhaitez configurer des machines virtuelles ou utiliser d’autres services Azure, vous devez souscrire un abonnement Azure. Vous pouvez partager votre abonnement Azure avec d’autres personnes et [utiliser le contrôle d’accès en fonction du rôle Azure (RBAC Azure) pour gérer l’accès à vos ressources et abonnement Azure](../../role-based-access-control/role-assignments-portal.md).

Si vous avez déjà à la fois un compte Microsoft 365 et un abonnement Azure, consultez [Associer un locataire Microsoft 365 à un abonnement Azure](../../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md).

## <a name="get-an-azure-subscription-using-your-microsoft-365-account"></a>Obtenir un abonnement Azure avec un compte Microsoft 365

Gagnez du temps et évitez la prolifération de comptes en vous inscrivant à Azure avec vos nom d’utilisateur et mot de passe Microsoft 365.

1. Inscrivez-vous sur [Azure.com](https://account.azure.com/signup?offer=MS-AZR-0044p&appId=docs).
2. Connectez-vous avec votre nom d’utilisateur et votre mot de passe Microsoft 365. Le compte que vous utilisez n’a pas besoin de disposer d’autorisations d’administrateur. Si vous avez plusieurs comptes Microsoft 365, veillez à utiliser les informations d’identification du compte Microsoft 365 que vous souhaitez associer à votre abonnement Azure.

   ![Capture d’écran montrant la page de connexion.](./media/microsoft-365-account-for-azure-subscription/billing-sign-in-with-office-365-account.png)

3. Entrez les informations requises et terminez le processus d’inscription. Certaines informations ne sont peut-être pas nécessaires si vous avez déjà un compte Microsoft 365.

    ![Capture d’écran montrant le formulaire d’inscription.](./media/microsoft-365-account-for-azure-subscription/billing-azure-sign-up-fill-information.png)

- Si vous devez ajouter d’autres personnes de votre organisation à l’abonnement Azure, consultez la page [Bien démarrer avec la gestion des accès sur le Portail Azure](../../role-based-access-control/overview.md).

## <a name=""></a><a id="more-about-subs">En savoir plus sur les abonnements Azure et Microsoft 365</a>

Microsoft 365 et Azure utilisent le service Azure AD pour gérer les utilisateurs et les abonnements. Le répertoire Azure est comme un conteneur qui vous sert à regrouper les abonnements et les utilisateurs. Pour utiliser les mêmes comptes d’utilisateur pour vos abonnements Azure et Microsoft 365, vous devez vous assurer que les abonnements Azure sont créés dans le même répertoire que les abonnements Microsoft 365. Gardez à l’esprit les points suivants :

* Un abonnement est créé dans un répertoire.
* Les utilisateurs appartiennent à des répertoires.
* Un abonnement est créé dans le répertoire de l’utilisateur qui crée l’abonnement. Par conséquent, votre abonnement Microsoft 365 est lié au même compte que votre abonnement Azure.
* Les abonnements Azure sont détenus par des utilisateurs individuels dans le répertoire.
* Les abonnements Microsoft 365 sont détenus par le répertoire proprement dit. Les utilisateurs possédant les autorisations correspondantes dans le répertoire peuvent gérer ces abonnements.

![Capture d’écran montrant la relation entre le répertoire, les utilisateurs et les abonnements.](./media/microsoft-365-account-for-azure-subscription/19-background-information.png)

Pour plus d’informations, consultez la page [Association des abonnements Azure avec Azure Active Directory](../../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md).

## <a name="need-help-contact-us"></a>Vous avez besoin d’aide ? Contactez-nous.

Si vous avez des questions ou besoin d’aide, [créez une demande de support](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Étapes suivantes

- Partagez votre abonnement Azure avec d’autres personnes et [utilisez le contrôle d’accès en fonction du rôle Azure (RBAC Azure) pour gérer l’accès à vos ressources et abonnement Azure](../../role-based-access-control/role-assignments-portal.md).