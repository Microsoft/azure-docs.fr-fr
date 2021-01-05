---
title: Options avancées de signature de certificat de jeton SAML pour les applications Azure AD
description: Découvrez comment utiliser les options avancées de signature de certificats dans le jeton SAML pour les applications pré-intégrées dans Azure Active Directory
services: active-directory
documentationcenter: ''
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 03/25/2019
ms.author: kenwith
ms.reviewer: jeedes
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 79bc43bb2fa99f95a462dcc4c68d27840199b45c
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94658534"
---
# <a name="advanced-certificate-signing-options-in-the-saml-token-for-gallery-apps-in-azure-active-directory"></a>Options avancées de signature de certificats dans le jeton SAML pour les applications de galerie dans Azure Active Directory

Aujourd’hui Azure Active Directory (Azure AD) prend en charge des milliers d’applications pré-intégrées dans la galerie d’applications Azure Active Directory. Plus de 500 applications prennent en charge l’authentification unique à l’aide du protocole [Security Assertion Markup Language](https://wikipedia.org/wiki/Security_Assertion_Markup_Language) (SAML) 2.0, comme l’application [NetSuite](https://azuremarketplace.microsoft.com/marketplace/apps/aad.netsuite). Lorsqu’un client s’authentifie auprès d’une application par l’intermédiaire d’Azure AD en utilisant SAML, Azure AD envoie un jeton à l’application (via HTTP POST). L’application valide et utilise ce jeton pour connecter le client, au lieu de lui demander un nom d’utilisateur et un mot de passe. Ces jetons SAML sont signés avec le certificat unique qui est généré dans Azure AD et par des algorithmes standard spécifiques.

Azure AD utilise certains des paramètres par défaut pour les applications de la galerie. Les valeurs par défaut sont configurées selon les exigences de l’application.

Dans Azure AD, vous pouvez configurer les options de signature de certificat et l’algorithme de signature de certificat.

## <a name="certificate-signing-options"></a>Options de signature de certificat

Azure AD prend en charge trois options de signature de certificat :

* **Signer l’assertion SAML**. Cette option par défaut est définie pour la plupart des applications de la galerie. Si cette option est sélectionnée, Azure AD joue le rôle de fournisseur d’identité et signe l’assertion et le certificat SAML avec le certificat [X509](https://wikipedia.org/wiki/X.509) de l’application.

* **Signer la réponse SAML**. Si cette option est sélectionnée, Azure AD joue le rôle de fournisseur d’identité et signe la réponse SAML avec le certificat X509 de l’application.

* **Signer la réponse et l’assertion SAML**. Si cette option est sélectionnée, Azure AD joue le rôle de fournisseur d’identité et signe l’intégralité du jeton SAML avec le certificat X509 de l’application.

## <a name="certificate-signing-algorithms"></a>Algorithmes de signature de certificat

Azure AD prend en charge deux algorithmes de signature ou algorithmes SHA pour signer la réponse SAML :

* **SHA-256**. Azure AD utilise cet algorithme par défaut pour signer la réponse SAML. Il s’agit de l’algorithme le plus récent, plus sécurisé que SHA-1. La plupart des applications prennent en charge l’algorithme SHA-256. Si une application prend en charge uniquement SHA-1 comme algorithme de signature, vous pouvez la modifier. Dans le cas contraire, nous vous recommandons d’utiliser l’algorithme SHA-256 pour la signature de la réponse SAML.

* **SHA-1**. Il s’agit de l’ancien algorithme, considéré comme moins sécurisé que SH-256. Si une application prend uniquement en charge cet algorithme de signature, vous pouvez sélectionner cette option dans la liste déroulante **Algorithme de signature**. Azure AD signe alors la réponse SAML avec l’algorithme SHA-1.

## <a name="change-certificate-signing-options-and-signing-algorithm"></a>Modifier les options et l’algorithme de signature de certificat

Pour modifier les options de signature de certificat SAML d’une application, ainsi que l’algorithme de signature de certificat, choisissez l’application concernée :

1. Dans le [Portail Microsoft Azure Active Directory](https://aad.portal.azure.com/), connectez-vous à votre compte. La page du **Centre d’administration Azure Active Directory** s’affiche.
1. Dans le volet gauche, sélectionnez **Applications d’entreprise**. Une liste des applications d’entreprise de votre compte s’affiche.
1. Sélectionnez une application. Une page de vue d’ensemble de l’application s’affiche.

   ![Exemple : Page de vue d’ensemble de l’application](./media/certificate-signing-options/application-overview-page.png)

Ensuite, modifiez les options de signature de certificat dans le jeton SAML pour l’application :

1. Dans le volet gauche de la page de vue d’ensemble de l’application, sélectionnez **Authentification unique**.
1. Si la page **Configurer l’authentification unique avec SAML - Préversion** s’affiche, passez à l’étape 5.
1. Si la page **Sélectionner une méthode d’authentification unique** ne s’affiche pas, choisissez **Modifier les modes d’authentification unique** pour l’afficher.
1. Dans la page **Sélectionner une méthode d’authentification unique**, sélectionnez **SAML**, le cas échéant. (Si la valeur **SAML** n’est pas disponible, cela signifie que l’application ne prend en charge SAML, et vous pouvez ignorer le reste de cette procédure et de cet article.)
1. Dans la page **Configurer l’authentification unique avec SAML - Préversion**, recherchez l’en-tête **Certificat de signature SAML** et sélectionnez l’icône **Modifier** (associée à un crayon). La page **Certificat de signature SAML** s’affiche.

   ![Exemple : Page du certificat de signature SAML](./media/certificate-signing-options/saml-signing-page.png)

1. Dans la liste déroulante **Option de signature**, sélectionnez **Signer la réponse SAML**, **Signer l’assertion SAML** ou **Signer la réponse et l’assertion SAML**. Vous trouverez une description de ces options dans la section [Options de signature de certificat](#certificate-signing-options) du présent article.
1. Dans la liste déroulante **Algorithme de signature**, choisissez **SHA-1** ou **SHA-256**. Vous trouverez une description de ces options dans la section [Algorithmes de signature de certificat](#certificate-signing-algorithms) du présent article.
1. Si vous êtes satisfait de votre choix, sélectionnez **Enregistrer** pour appliquer les nouveaux paramètres de certificat de signature SAML. Sinon, sélectionnez **X** pour ignorer les modifications.

## <a name="next-steps"></a>Étapes suivantes

* [Configurer l’authentification unique pour les applications ne faisant pas partie de la galerie d’applications Azure Active Directory](./configure-saml-single-sign-on.md)
* [Résoudre les erreurs d’authentification unique basée sur SAML](./debug-saml-sso-issues.md)