---
title: Présentation de la vérification de l'éditeur - Plateforme d'identités Microsoft | Azure
description: Fournit une vue d'ensemble du programme de vérification de l'éditeur pour la Plateforme d'identités Microsoft. Répertorie les avantages et conditions d'utilisation du programme, ainsi que les questions fréquemment posées. Lorsqu'une application est accompagnée de la mention « éditeur vérifié », cela signifie que l'éditeur a validé son identité à l'aide d'un compte Microsoft Partner Network auquel le processus de vérification a été appliqué jusqu'à son terme et qu'il a associé ce compte MPN à l'inscription de l'application.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/19/2020
ms.author: ryanwi
ms.custom: aaddev
ms.reviewer: jesakowi
ms.openlocfilehash: 1e913e3a5356ad7f49d8b3066f5bd3da7eddd2c2
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93308783"
---
# <a name="publisher-verification"></a>Vérification de l'éditeur

La vérification de l’éditeur permet aux administrateurs et aux utilisateurs finaux de s'assurer de l'authenticité des développeurs d'applications qui s'intègrent à la plateforme d'identités Microsoft. 

> [!VIDEO https://www.youtube.com/embed/IYRN2jDl5dc]

Lorsqu'une application est accompagnée de la mention « éditeur vérifié », cela signifie que l'éditeur a validé son identité à l'aide d'un compte [Microsoft Partner Network](https://partner.microsoft.com/membership) auquel le processus de [vérification](/partner-center/verification-responses) a été appliqué jusqu'à son terme et qu'il a associé ce compte MPN à son inscription d'application. 

Un badge bleu « vérifié » apparaît sur l'invite de consentement d'Azure AD et sur d'autres écrans : ![Invite de consentement](./media/publisher-verification-overview/consent-prompt.png)

Cette fonctionnalité est principalement destinée aux développeurs qui créent des applications multilocataires tirant parti d'[OAuth 2.0 et OpenID Connect](active-directory-v2-protocols.md) avec la [Plateforme d'identités Microsoft](v2-overview.md). Ces applications peuvent connecter les utilisateurs via OpenID Connect, ou utiliser OAuth 2.0 pour demander l'accès aux données à l'aide d'API telles que [Microsoft Graph](https://developer.microsoft.com/graph/).

## <a name="benefits"></a>Avantages
La vérification de l'éditeur offre les avantages suivants :
- **Transparence et réduction des risques accrues pour les clients**  : cette fonctionnalité permet aux clients d'identifier les applications de leur organisation publiées par des développeurs en qui ils ont confiance. 

- **Amélioration de l'image de marque**  : un badge « vérifié » apparaît sur l' [invite de consentement](application-consent-experience.md) Azure AD, la page Applications d'entreprise et les autres pages auxquelles ont accès les utilisateurs finaux et les administrateurs. 

- **Adoption plus fluide par les entreprises**  : les administrateurs peuvent configurer des [stratégies de consentement des utilisateurs](../manage-apps/configure-user-consent.md), l'état de vérification de l'éditeur étant l'un des principaux critères de ces stratégies.

> [!NOTE]
> À compter du mois de novembre 2020, les utilisateurs finaux ne seront plus en mesure d’accorder leur consentement à la plupart des applications multilocataires nouvellement inscrites sans éditeurs vérifiés. Cela s’appliquera aux applications inscrites après le 8 novembre 2020, qui utilisent OAuth 2.0 pour demander des autorisations au-delà de la connexion de base et lire le profil utilisateur, et qui demandent le consentement d’utilisateurs situés dans d’autres locataires que celui où l’application est inscrite. Un avertissement s’affichera sur l’écran de consentement pour informer les utilisateurs que ces applications sont risquées et proviennent d’éditeurs non vérifiés.    

## <a name="requirements"></a>Spécifications
Il existe quelques conditions préalables à la vérification de l'éditeur, dont certaines sont déjà remplies par de nombreux partenaires Microsoft. Il s'agit de : 

-  Un ID MPN pour un compte [Microsoft Partner Network](https://partner.microsoft.com/membership) valide auquel le processus de [vérification](/partner-center/verification-responses) a été appliqué jusqu'à son terme. Ce compte MPN doit être le [compte global partenaire (PGA)](/partner-center/account-structure#the-top-level-is-the-partner-global-account-pga) de votre organisation. 

-  Une application inscrite auprès d’un locataire Azure AD, avec un [domaine d’éditeur](howto-configure-publisher-domain.md) configuré.

-  Le domaine de l’adresse e-mail utilisée lors de la vérification du compte MPN doit correspondre au domaine de l’éditeur configuré sur l’application ou à un [domaine personnalisé](../fundamentals/add-custom-domain.md) vérifié par DNS, ajouté au locataire Azure AD. 

-  L'utilisateur qui procède à la vérification doit être autorisé à apporter des modifications à l'inscription de l'application dans Azure AD et au compte MPN dans l'Espace partenaires. 

    -  Dans Azure AD, cet utilisateur doit être membre d’un des [rôles](../roles/permissions-reference.md) suivants : Administrateur d’application, Administrateur d’application cloud ou Administrateur général. 

    -  Dans l'Espace partenaires, cet utilisateur doit disposer des [rôles](/partner-center/permissions-overview) suivants : Administrateur MPN, Administrateur de comptes ou Administrateur général (rôle partagé maîtrisé dans Azure AD).
    
-  L'utilisateur qui procède à la vérification doit se connecter en utilisant l'[authentification multifacteur](../authentication/howto-mfa-getstarted.md).

-  L'éditeur accepte les [conditions d'utilisation de la Plateforme d'identités Microsoft pour développeurs](/legal/microsoft-identity-platform/terms-of-use).

Les développeurs qui ont déjà rempli ces conditions préalables peuvent être vérifiés en quelques minutes. Si les conditions ne sont pas remplies, la configuration est gratuite. 

## <a name="frequently-asked-questions"></a>Forum aux questions 
Vous trouverez ci-dessous quelques questions fréquemment posées au sujet du programme de vérification de l'éditeur. Pour consulter le FAQ relatif aux exigences et au processus, consultez [Marquer une application avec la mention « éditeur vérifié »](mark-app-as-publisher-verified.md).

- **Quelles informations la vérification de l'éditeur ne fournit-elle __pas__  ?**  Lorsqu'une application est accompagnée de la mention « éditeur vérifié », cela n'indique pas si l'application ou son éditeur a obtenu des certifications spécifiques, est conforme aux normes du secteur, respecte les meilleures pratiques, etc. D'autres programmes Microsoft fournissent ces informations, notamment la [Certification d'application Microsoft 365](/microsoft-365-app-certification/overview).

- **Combien cela coûte-t-il ? Une licence est-elle requise ?** Microsoft ne facture pas la vérification de l'éditeur aux développeurs et n'exige pas de licence spécifique. 

- **Quel est le lien avec l'attestation d'éditeur Microsoft 365 ? Qu'en est-il de la certification d'application Microsoft 365 ?** Il s'agit de programmes complémentaires que les développeurs peuvent utiliser pour créer des applications fiables qui peuvent être adoptées en toute confiance par les clients. La vérification de l'éditeur est la première étape de ce processus et doit être effectuée par tous les développeurs qui créent des applications conformes aux critères ci-dessus. 

  Les développeurs qui s'intègrent également à Microsoft 365 peuvent bénéficier d'avantages supplémentaires grâce à ces programmes. Pour plus d'informations, reportez-vous à [Attestation d'éditeur Microsoft 365](/microsoft-365-app-certification/docs/attestation) et [Certification d'application Microsoft 365](/microsoft-365-app-certification/docs/certification). 

- **Est-ce la même chose que la Galerie d'applications Azure AD ?** Non. La vérification de l'éditeur est un programme complémentaire mais distinct de la [Galerie d'applications Azure Active Directory](v2-howto-app-gallery-listing.md). Les développeurs qui remplissent les critères ci-dessus doivent finaliser le processus de vérification de l'éditeur indépendamment de leur participation à ce programme. 

## <a name="next-steps"></a>Étapes suivantes
* Apprenez à [marquer une application avec la mention « éditeur vérifié »](mark-app-as-publisher-verified.md).
* [Résoudre les problèmes](troubleshoot-publisher-verification.md) de vérification de l'éditeur.
