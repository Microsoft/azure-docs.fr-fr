---
title: 'Azure AD Connect : authentification unique transparente - Fonctionnement | Microsoft Docs'
description: Cet article décrit le fonctionnement de la fonctionnalité d’authentification unique transparente d’Azure Active Directory.
services: active-directory
keywords: Qu’est-ce qu’Azure AD Connect, Installation d’Active Directory, Composants requis pour Azure AD, SSO, Authentification unique
documentationcenter: ''
author: billmath
manager: daveba
ms.assetid: 9f994aca-6088-40f5-b2cc-c753a4f41da7
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 04/16/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: bde937adba8d2469390a6cf404f6cce8c5008e87
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86144708"
---
# <a name="azure-active-directory-seamless-single-sign-on-technical-deep-dive"></a>Authentification unique transparente Azure Active Directory : Présentation technique approfondie

Cet article vous fournit des détails techniques sur le fonctionnement de la fonctionnalité d’authentification unique (SSO) transparente d’Azure Active Directory.

## <a name="how-does-seamless-sso-work"></a>Fonctionnement de l’authentification unique transparente (SSO)

Cette section est composée de trois parties :

1. Configuration de la fonctionnalité d’authentification unique transparente.
2. Fonctionnement d’une transaction de connexion mono-utilisateur dans un navigateur web avec l’authentification unique transparente
3. Fonctionnement d’une transaction de connexion mono-utilisateur sur un client natif avec l’authentification unique transparente

### <a name="how-does-set-up-work"></a>Comment la configuration s’opère-t-elle ?

L’authentification unique transparente s’active via Azure AD Connect comme indiqué [ici](how-to-connect-sso-quick-start.md). Voici ce qu’il se passe pendant l’activation de la fonctionnalité :

- Un compte d’ordinateur (`AZUREADSSOACC`) est créé dans votre Active Directory (AD) local dans chaque forêt AD que vous synchronisez sur Azure AD (à l’aide d’Azure AD Connect).
- Par ailleurs, des noms de principaux du service Kerberos (SPN) sont créés pour être utilisés pendant le processus de connexion à Azure AD.
- La clé de déchiffrement Kerberos du compte d’ordinateur est partagée en toute sécurité avec Azure AD. S’il existe plusieurs forêts Active Directory, chaque compte d’ordinateur a sa propre clé de déchiffrement Kerberos unique.

>[!IMPORTANT]
> Pour des raisons de sécurité, le compte d’ordinateur `AZUREADSSOACC` doit être fortement protégé. Seuls des administrateurs de domaine doivent être en mesure de gérer le compte d’ordinateur. Vérifiez que la délégation Kerberos sur le compte d’ordinateur est désactivée et qu’aucun autre compte dans Active Directory ne dispose d’autorisations de délégation sur le compte d’ordinateur `AZUREADSSOACC`. Stockez le compte d’ordinateur dans une unité d’organisation (UO) où il sera protégé contre les suppressions accidentelles et à laquelle seuls des administrateurs de domaine ont accès. La clé de déchiffrement Kerberos sur le compte d’ordinateur doit également être traitée comme sensible. Il est fortement recommandé que vous [substituiez la clé de déchiffrement Kerberos](how-to-connect-sso-faq.md) du `AZUREADSSOACC` compte d’ordinateur au moins tous les 30 jours.

>[!IMPORTANT]
> L’authentification unique transparente prend en charge les types de chiffrement AES256_HMAC_SHA1, AES128_HMAC_SHA1 et RC4_HMAC_MD5 pour Kerberos. Il est recommandé de définir le type de chiffrement du compte AzureADSSOAcc$ sur AES256_HMAC_SHA1 ou l’un des types AES plutôt que sur RC4 pour plus de sécurité. Le type de chiffrement est stocké dans l’attribut msDS-SupportedEncryptionTypes du compte de votre annuaire Active Directory.  Si le type de chiffrement du compte AzureADSSOAcc$ a la valeur RC4_HMAC_MD5 et que vous voulez le remplacer par l’un des types de chiffrement AES, veillez d’abord à remplacer la clé de déchiffrement Kerberos du compte AzureADSSOAcc$, comme expliqué dans les [questions fréquentes (FAQ)](how-to-connect-sso-faq.md), sous la question concernée, sans quoi l’authentification unique transparente ne se produira pas.

Une fois la configuration terminée, l’authentification unique transparente fonctionne de la même façon que n’importe quelle autre connexion utilisant l’authentification Windows intégrée (IWA).

### <a name="how-does-sign-in-on-a-web-browser-with-seamless-sso-work"></a>Fonctionnement des connexions dans un navigateur web avec l’authentification unique transparente

Le flux de connexion dans un navigateur web est le suivant :

1. Un utilisateur tente d’accéder à une application web (par exemple, Outlook Web App - https://outlook.office365.com/owa/) ) à partir d’un appareil d’entreprise joint à un domaine du réseau de l’entreprise.
2. Si l’utilisateur n’est pas déjà connecté, il est redirigé vers la page de connexion Azure AD.
3. L’utilisateur tape son nom d’utilisateur dans la page de connexion Azure AD.

   >[!NOTE]
   >Pour [certaines applications](./how-to-connect-sso-faq.md), les étapes 2 et 3 ne sont pas nécessaires.

4. En utilisant JavaScript en arrière-plan, Azure AD demande au client, via une réponse 401 Non autorisé, de fournir un ticket Kerberos.
5. À son tour, le navigateur demande un ticket à Active Directory pour le compte d’ordinateur `AZUREADSSOACC` (qui représente Azure AD).
6. Active Directory localise le compte d’ordinateur et retourne un ticket Kerberos au navigateur chiffré avec le secret du compte d’ordinateur.
7. Le navigateur transfère le ticket Kerberos reçu de la part d’Active Directory à Azure AD.
8. Azure AD déchiffre le ticket Kerberos, qui comprend l’identité de l’utilisateur connecté à l’appareil d’entreprise, en utilisant la clé partagée précédemment.
9. À l’issue de l’évaluation, Azure AD retourne un jeton à l’application ou bien demande à l’utilisateur de fournir des preuves supplémentaires, telles qu’une authentification multifacteur.
10. Si l’utilisateur parvient à se connecter, il peut accéder à l’application.

Le schéma suivant illustre tous les composants et les étapes impliquées dans ce processus.

![Authentification unique transparente : flux des applications web](./media/how-to-connect-sso-how-it-works/sso2.png)

L’authentification unique transparente est opportuniste, ce qui signifie que si elle échoue, l’expérience de connexion reprend son comportement normal (l’utilisateur doit entrer son mot de passe pour se connecter).

### <a name="how-does-sign-in-on-a-native-client-with-seamless-sso-work"></a>Fonctionnement des connexions sur un client natif avec l’authentification unique transparente

Le flux de connexion sur un client natif est le suivant :

1. Un utilisateur tente d’accéder à une application native (par exemple, le client Outlook) à partir d’un appareil d’entreprise joint à un domaine du réseau de l’entreprise.
2. Si l’utilisateur n’est pas déjà connecté, l’application native récupère le nom d’utilisateur de l’utilisateur à partir de la session Windows de l’appareil.
3. L’application envoie le nom d’utilisateur à Azure AD et récupère le point de terminaison WS-Trust MEX de votre locataire. Ce point de terminaison WS-Trust est exclusivement utilisé par la fonctionnalité d'authentification unique transparente et il ne s'agit pas d'une implémentation générale du protocole WS-Trust sur Azure AD.
4. L’application interroge ensuite le point de terminaison WS-Trust MEX pour savoir si le point de terminaison d’authentification intégrée est disponible. Le point de terminaison d'authentification intégré est exclusivement utilisé par la fonctionnalité d'authentification unique transparente.
5. Si l’étape 4 a réussi, une demande d’authentification Kerberos est émise.
6. Si l’application est en mesure de récupérer le ticket Kerberos, il le transfère au point de terminaison d’authentification intégrée d’Azure AD.
7. Azure AD déchiffre le ticket Kerberos, puis le valide.
8. Azure AD connecte l’utilisateur et émet un jeton SAML pour l’application.
9. L’application envoie ensuite le jeton SAML au point de terminaison du jeton OAuth2 d’Azure AD.
10. Azure AD valide le jeton SAML et envoie un jeton d’accès et un jeton d’actualisation à l’application pour la ressource spécifiée, ainsi qu’un jeton d’ID.
11. L’utilisateur peut alors accéder aux ressources de l’application.

Le schéma suivant illustre tous les composants et les étapes impliquées dans ce processus.

![Authentification unique transparente : flux des applications natives](./media/how-to-connect-sso-how-it-works/sso14.png)

## <a name="next-steps"></a>Étapes suivantes

- [**Démarrage rapide**](how-to-connect-sso-quick-start.md) : découvrez l’authentification unique transparente Azure AD.
- [**Questions fréquentes (FAQ)** ](how-to-connect-sso-faq.md) : réponses aux questions fréquentes.
- [**Résolution des problèmes**](tshoot-connect-sso.md) : découvrez comment résoudre les problèmes courants susceptibles de survenir avec cette fonctionnalité.
- [**UserVoice**](https://feedback.azure.com/forums/169401-azure-active-directory/category/160611-directory-synchronization-aad-connect) : pour le dépôt de nouvelles demandes de fonctionnalités.
