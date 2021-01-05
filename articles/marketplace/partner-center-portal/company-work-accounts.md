---
title: Comptes professionnels d’entreprise et Espace partenaires
description: Comment vérifier si votre entreprise possède un compte professionnel configuré avec Microsoft, créer un nouveau compte professionnel ou configurer plusieurs comptes professionnels à utiliser avec l’Espace partenaires.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: parthpandyaMSFT
ms.author: parthp
ms.date: 05/30/2019
ms.openlocfilehash: 5b4aadc506b2f4a251ccef5e9488066b609ba5a5
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93130525"
---
# <a name="company-work-accounts-and-partner-center"></a>Comptes professionnels d’entreprise et Espace partenaires

L’Espace partenaires utilise des comptes professionnels d’entreprise, également connus sous le nom de locataires Azure Active Directory (AD), pour gérer l’accès au compte pour plusieurs utilisateurs, les autorisations de contrôle, les groupes hôtes et les applications, et gérer les données de profil. En liant le domaine de compte de messagerie professionnelle de votre entreprise à votre compte d’Espace partenaires, les employés de votre entreprise peuvent se connecter à l’Espace partenaires pour gérer des offres de place de marché à l’aide de leurs propres noms d’utilisateur et mots de passe de comptes professionnels.

## <a name="check-whether-your-company-already-has-a-work-account"></a>Vérifier si votre entreprise possède déjà un compte professionnel

Si votre entreprise s’est abonnée à un service cloud Microsoft comme Azure, Microsoft Intune ou Microsoft 365, vous disposez déjà d’un domaine de compte de messagerie (également appelé locataire Azure Active Directory) qui peut être utilisé avec l’Espace partenaires.

Pour vérifier, procédez comme suit :
1. Connectez-vous au portail d’administration Azure sur https://portal.azure.com.
2. Sélectionnez **Azure Active Directory** dans le menu de navigation de gauche, puis sélectionnez **Noms de domaine personnalisés**.
3. Si vous avez déjà un compte professionnel, votre nom de domaine s’affichera.

Si votre entreprise n’a pas encore de compte professionnel, il en sera créé un pendant le processus d’inscription à l’Espace partenaires.

## <a name="set-up-multiple-work-accounts"></a>Configurer plusieurs comptes professionnels

Avant de décider d’utiliser un compte professionnel existant, déterminez combien d’utilisateurs dans le compte professionnel auront besoin d’accéder à l’Espace partenaires. Si vous avez des utilisateurs dans le compte professionnel qui n’auront pas besoin d’accéder à l’Espace partenaires, vous souhaiterez peut-être envisager de créer plusieurs comptes professionnels, afin que seuls les utilisateurs qui ont besoin d’accéder à l’Espace partenaires soient représentés sur un compte particulier.

## <a name="create-a-new-work-account"></a>Créer un nouveau compte professionnel

Pour créer un nouveau compte professionnel pour votre entreprise, suivez les étapes ci-dessous. Vous devrez peut-être demander de l’aide à quelqu’un disposant d’autorisations d’administration sur le compte Microsoft Azure de votre entreprise.

1. Connectez-vous au [portail Microsoft Azure](https://portal.azure.com).
2. Dans la barre de navigation de gauche, sélectionnez **Azure Active Directory** -> **Utilisateurs**.
3. Sélectionnez **Nouvel utilisateur** et créez un nouveau compte professionnel Azure en entrant une nom et adresse e-mail. Vérifiez que le **Rôle d’annuaire** est défini sur **Utilisateur** et activez la case à cocher **Afficher le mot de passe** en bas pour afficher et prendre note du mot de passe généré automatiquement.
4. Sélectionnez **Créer** pour enregistrer le nouvel utilisateur.

L’adresse e-mail du compte d’utilisateur doit être un nom de domaine vérifié dans votre répertoire. Vous pouvez répertorier tous les domaines vérifiés dans votre répertoire en sélectionnant **Azure Active Directory** -> **Noms de domaine personnalisés** dans le menu de navigation de gauche.

Pour en savoir plus sur l’ajout de domaines personnalisés dans Azure Active Directory, consultez [Ajouter ou associer un domaine dans Azure AD](../../active-directory/fundamentals/add-custom-domain.md).

## <a name="troubleshoot-work-email-sign-in"></a>Résoudre les problèmes de connexion à la messagerie professionnelle

Si vous ne parvenez pas à vous connecter à votre compte professionnel (également appelé locataire Azure AD), identifiez sur le diagramme ci-dessous le scénario qui correspond le mieux à votre situation et suivez les étapes recommandées.

![Diagramme pour la résolution des problèmes de connexion à un compte professionnel](./media/onboarding-aad-flow.png)

## <a name="next-steps"></a>Étapes suivantes

- [Gérer votre compte Place de marché commerciale dans l’Espace partenaires](./manage-account.md)