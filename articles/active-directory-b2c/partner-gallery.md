---
title: Galerie de partenaires ISV pour Azure AD B2C
titleSuffix: Azure AD B2C
description: 'Découvrez comment intégrer les solutions de nos partenaires ISV pour adapter l’expérience de vos utilisateurs finaux à vos besoins. Notre réseau de partenaires étend les fonctionnalités de nos solutions : activez MFA, activez l’authentification forte du client, appliquez le contrôle d’accès en fonction du rôle et luttez contre les fraudes grâce à la vérification et la confirmation d’identité.'
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 06/08/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 4592742b99d7c3e915c31ca5a202355d32e76d68
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97356121"
---
# <a name="azure-active-directory-b2c-isv-partners"></a>Partenaires ISV pour Azure Active Directory B2C

Notre réseau de partenaires ISV étend les fonctionnalités de nos solutions et vous aide ainsi à offrir des expériences fluides aux utilisateurs finaux. Azure AD B2C vous pouvez de vous intégrer avec des partenaires fournisseurs de logiciels indépendants pour activer des méthodes d’authentification multifacteur (MFA), effectuer un contrôle d’accès en fonction du rôle, activer la vérification et la confirmation d’identité, améliorer la sécurité grâce à la détection des bots et la protection contre les fraudes, et répondre aux exigences en matière d’authentification sécurisée des clients de la directive 2 sur les services de paiement (PSD2). Suivez nos exemples de procédures détaillées pour découvrir comment intégrer des applications avec les partenaires fournisseurs de logiciels indépendants.

>[!NOTE]
>Sur le [site de la communauté Azure Active Directory B2C sur GitHub](https://azure-ad-b2c.github.io/azureadb2ccommunity.io/), vous trouverez également des exemples de stratégies personnalisées fournis par la communauté.

## <a name="identity-verification-and-proofing"></a>Vérification des identités

Avec des partenaires Azure AD B2C, les clients peuvent activer la vérification la confirmation d’identité de leurs utilisateurs finaux avant d’autoriser l’inscription de comptes ou l’accès à ceux-ci. La vérification des identités permet de contrôler des documents, des informations basées sur des connaissances et l’activité.

Un diagramme d’architecture de haut niveau explique le flux.

![Diagramme montrant le flux de vérification et de confirmation d’identité](./media/partner-gallery/third-party-identity-proofing.png)

Microsoft travaille en partenariat avec les fournisseurs de logiciels indépendants (ISV) suivants pour la vérification et la confirmation d’identité.

| Partenaire fournisseur de logiciels indépendant | Description et procédures pas à pas d’intégration |
|:-------------------------|:--------------|
|![Capture d’écran d’un logo Experian.](./media/partner-gallery/experian-logo.png) | [Experian](./partner-experian.md) est un fournisseur de solutions de vérification et de confirmation d’identité qui effectue des évaluations de risques basées sur les attributs utilisateur pour empêcher les fraudes. |
|![Capture d’écran d’un logo IDology.](./media/partner-gallery/idology-logo.png) | [IDology](./partner-idology.md) est un fournisseur de solutions de vérification et de confirmation d’identité, comprenant entre autres des solutions de vérification d’identité, des solutions de prévention des fraudes et des solutions de conformité.|
|![Capture d’écran d’un logo Jumio.](./media/partner-gallery/jumio-logo.png) | [Jumio](./partner-jumio.md) est un service de vérification d’ID, qui permet de vérifier l’ID automatisé en temps réel et de protéger les données client. |
| ![Capture d’écran d’un logo LexisNexis.](./media/partner-gallery/lexisnexis-logo.png) | [LexisNexis](./partner-lexisnexis.md) est un fournisseur de solutions de profilage et de validation d’identité qui vérifie l’identification des utilisateurs et fournit des évaluations de risques complètes basées sur l’appareil de l’utilisateur. |
| ![Capture d’écran d’un logo Onfido](./media/partner-gallery/onfido-logo.png) | [Onfido](./partner-onfido.md) est un ID de document et une solution de vérification de biométrie faciale qui permet aux entreprises de répondre au exigences *de connaître la clientèle*  et d’identités en temps réel.  |

## <a name="mfa-and-passwordless-authentication"></a>Authentification multifacteur et authentification sans mot de passe

Microsoft travaille en partenariat avec les fournisseurs de logiciels indépendants (ISV) suivants pour l’authentification multifacteur et l’authentification sans mot de passe.

| Partenaire fournisseur de logiciels indépendant | Description et procédures pas à pas d’intégration |
|:-------------------------|:--------------|
| ![Capture d’écran d’un logo Hypr](./media/partner-gallery/hypr-logo.png) | [Hypr](./partner-hypr.md) est un fournisseur d’authentification sans mot de passe qui remplace les mots de passe par des chiffrements à clé publique, éliminant les fraudes, le hameçonnage et la réutilisation des informations d’identification. |
| ![Capture d’écran d’un logo Itsme](./media/partner-gallery/itsme-logo.png) | [itsme](./partner-itsme.md) est une solution d’identification numérique conforme aux normes eiDAS (Electronic Identification, Authentication and Trust Services) qui permet aux utilisateurs de se connecter en toute sécurité sans avoir à utiliser de lecteurs de cartes, de mots de passe, de méthode d’authentification à deux facteurs ni de codes PIN multiples. |
| ![Capture d’écran d’un logo Nevis](./media/partner-gallery/nevis-logo.png) | [Nevis](./partner-nevis.md) permet une authentification sans mot de passe et offre une expérience utilisateur final axée sur l’interface mobile, entièrement personnalisée avec l’application Nevis Access pour une authentification forte du client et pour garantir le respect des obligations relatives aux transactions de la directive 2 sur les services de paiement (PSD2). |
| ![Capture d’écran d’un logo Trusona](./media/partner-gallery/trusona-logo.png) | L’intégration avec [Trusona](./partner-trusona.md) vous aide à sécuriser vos connexions, et permet l’authentification sans mot de passe, l’authentification multifacteur et l’analyse de licence numérique. |
| ![Capture d’écran d’un logo Twilio](./media/partner-gallery/twilio-logo.png) | [Twilio Verify App](./partner-twilio.md) fournit plusieurs solutions qui permettent l’authentification multifacteur (MFA) par l’envoi par SMS d’un mot de passe à usage unique (OTP), d’un mot de passe à usage unique et durée définie (TOTP) ou de notifications Push, et aident à se conformer aux exigences en matière d’authentification sécurisée des clients de la directive 2 sur les services de paiement (PSD2). |
| ![Capture d’écran d’un logo typingDNA](./media/partner-gallery/typingdna-logo.png) | [TypingDNA](./partner-twilio.md) permet une authentification forte du client en analysant le modèle de saisie d’un utilisateur. Il permet aux entreprises d’activer une authentification multifacteur sans assistance et de se conformer aux exigences en matière d’authentification sécurisée des clients de la directive 2 sur les services de paiement (PSD2). |
| ![Capture d’écran d’un logo WhoIAM](./media/partner-gallery/whoiam-logo.png) | [WhoIAM](./partner-whoiam.md) est une application BRIMS (Branded Identity Management System) qui permet aux organisations de vérifier leur base d’utilisateurs par la voix, les SMS et les e-mails. |

## <a name="role-based-access-control"></a>Contrôle d’accès en fonction du rôle 
 
Microsoft travaille en partenariat avec les fournisseurs de logiciels indépendants (ISV) suivants pour le contrôle d’accès en fonction du rôle.

| Partenaire fournisseur de logiciels indépendant | Description et procédures pas à pas d’intégration |
|:-------------------------|:--------------|
| ![Capture d’écran d’un logo n8identity](./media/partner-gallery/n8identity-logo.png) | [N8Identity](./partner-n8identity.md) est une plateforme de gouvernance d’identité en tant que service qui fournit une solution pour répondre à la migration des comptes clients et à l’administration CSR (Customer Service Requests) s’exécutant sur Microsoft Azure. |
| ![Capture d’écran d’un logo Saviynt](./media/partner-gallery/saviynt-logo.png) | La plateforme [Saviynt](./partner-Saviynt.md) native Cloud améliore la sécurité, la conformité et la gouvernance grâce à une analytique intelligente et à une intégration entre applications pour rationaliser la modernisation informatique. |

## <a name="security"></a>Sécurité

Microsoft travaille en partenariat avec les fournisseurs de logiciels indépendants (ISV) suivants pour la sécurité.

| Partenaire fournisseur de logiciels indépendant | Description et procédures pas à pas d’intégration |
|:-------------------------|:--------------|
| ![Capture d’écran d’un logo Arkose Labs](./media/partner-gallery/arkose-logo.png) | [Arkose Labs](./partner-arkose-labs.md) est un fournisseur de solutions de prévention des fraudes qui aide les organisations à se protéger contre les attaques de bots, les attaques de prise de contrôle de comptes et les ouvertures de comptes frauduleuses. |
| ![Capture d’écran d’un logo Strata](./media/partner-gallery/strata-logo.png) | [Strata](./partner-strata.md) fournit un accès hybride sécurisé à des applications locales en appliquant des stratégies d’accès cohérentes, en assurant la synchronisation des identités et en simplifiant la transition des applications entre des systèmes d’identité hérités et le contrôle d’accès et l’authentification standard fournis par Azure AD B2C. |
| ![Capture d’écran d’un logo Zscaler](./media/partner-gallery/zscaler-logo.png) | [Zscaler](./partner-zscaler.md) fournit un accès sécurisé basé sur des stratégies aux applications privées et aux ressources, sans les risques liés aux coûts, à la charge ou à la sécurité d’un réseau privé virtuel (VPN). |

## <a name="additional-information"></a>Informations supplémentaires

- [Stratégies personnalisées dans Azure AD B2C](https://docs.microsoft.com/azure/active-directory-b2c/custom-policy-overview)

- [Bien démarrer avec les stratégies personnalisées dans Azure AD B2C](https://docs.microsoft.com/azure/active-directory-b2c/custom-policy-get-started?tabs=applications)

## <a name="next-steps"></a>Étapes suivantes

Sélectionnez un partenaire dans les tableaux mentionnés pour savoir comment intégrer leur solution avec Azure AD B2C.
