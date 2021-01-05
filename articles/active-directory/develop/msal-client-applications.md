---
title: Applications clientes publiques et confidentielles (MSAL) | Azure
titleSuffix: Microsoft identity platform
description: En savoir plus sur les applications clientes publiques et confidentielles dans la bibliothèque d’authentification Microsoft (MSAL).
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 04/25/2019
ms.author: marsma
ms.reviewer: saeeda
ms.custom: aaddev
ms.openlocfilehash: 9c3292a31e5f750c16933acf94509e0ad226080a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "81534310"
---
# <a name="public-client-and-confidential-client-applications"></a>Applications clientes publiques et confidentielles
La bibliothèque d’authentification Microsoft (MSAL) définit deux types de clients : les clients publics et les clients confidentiels. Les deux types de clients se distinguent par leur capacité à s’authentifier en toute sécurité avec le serveur d’autorisation et à préserver la confidentialité de leurs identifiants client. En revanche, Azure AD Authentication Library (ADAL) utilise ce qu’on appelle le *contexte d’authentification* (qui est une connexion à Azure AD).

- Les **applications clientes confidentielles** sont des applications qui s’exécutent sur des serveurs (applications web, applications d’API web, voire applications de service ou de démon). Elles sont considérées comme difficiles d’accès et, pour cette raison, capables de rester secrètes. Les clients confidentiels peuvent détenir des clés secrètes définies au moment de la configuration. Chaque instance du client possède une configuration distincte (comprenant l’ID client et la clé secrète du client). Ces valeurs sont difficiles à extraire pour les utilisateurs finaux. Une application web est le client confidentiel le plus courant. L’ID client est exposé via le navigateur web, mais la clé secrète n’est transmise que dans le canal arrière et jamais directement exposée.

    Applications clientes confidentielles : <BR>
    ![Application web](media/msal-client-applications/web-app.png) ![ API web](media/msal-client-applications/web-api.png) ![Démon/Service](media/msal-client-applications/daemon-service.png)

- **Les applications clientes publiques** sont des applications qui s’exécutent sur des appareils, des ordinateurs de bureau ou dans un navigateur web. N'étant pas approuvées pour conserver les clés secrètes d’application, elles n’accèdent aux API web que pour le compte de l’utilisateur. (Elles ne prennent en charge que les flux de clients publics.) Les clients publics ne disposent pas de clés secrètes de client comme elles ne peuvent pas contenir les clés secrètes définies lors de la configuration.

    Applications clientes publiques : <BR>
    ![Application de bureau](media/msal-client-applications/desktop-app.png) ![API sans navigateur](media/msal-client-applications/browserless-app.png) ![Application mobile](media/msal-client-applications/mobile-app.png)

> [!NOTE]
> Dans MSAL.js, il n’y a pas de séparation entre les applications clientes publiques et les applications clientes confidentielles.  MSAL.js représente les applications clientes sous la forme d’applications clientes basées sur l’agent utilisateur, applications clientes publiques dans lesquelles le code client est exécuté dans un agent utilisateur, tel qu’un navigateur web. Ces clients ne stockent pas de clés secrètes car le contexte du navigateur est directement accessible.

## <a name="comparing-the-client-types"></a>Comparaison des types de client
Voici quelques ressemblances et différences entre les applications clientes publiques et les applications clientes confidentielles :

- Les deux types d’applications gèrent un cache de jetons utilisateur et peuvent acquérir un jeton automatiquement (lorsque celui-ci réside déjà dans le cache de jetons). Les applications clientes confidentielles disposent également d’un cache de jetons d’application pour les jetons qui sont destinés à l’application elle-même.
- Les deux types d’application gèrent les comptes d’utilisateur et peuvent obtenir un compte à partir du cache des jetons utilisateur, obtenir un compte à partir de son identifiant ou supprimer un compte.
- Les applications clientes publiques ont quatre façons d’acquérir un jeton (via quatre flux d’authentification). Les applications clientes confidentielles ont trois façons d’acquérir un jeton (et une façon de calculer l’URL du point de terminaison d’autorisation du fournisseur d’identité). Pour plus d’informations, consultez [Acquisition des jetons](msal-acquire-cache-tokens.md).

Si vous avez utilisé la bibliothèque d’authentification Active Directory (ADAL), vous remarquerez peut-être que, contrairement au contexte d’authentification d’ADAL, dans MSAL, l’ID client (également appelé *ID d’application* ou *ID d’app*) est passé une seule fois lors de la construction de l’application. Il n’est pas nécessaire de le transmettre à nouveau lorsque l’application acquiert un jeton. Cela s’applique à la fois aux applications clientes publiques et aux applications clientes confidentielles. Les constructeurs d’applications clientes confidentielles se voient également recevoir des informations d’identification client : la clé secrète qu’ils partagent avec le fournisseur d’identité.

## <a name="next-steps"></a>Étapes suivantes
Vous en saurez plus sur :
- [Options de configuration d’application cliente](msal-client-application-configuration.md)
- [Instanciation des applications clientes avec MSAL.NET](msal-net-initializing-client-applications.md)
- [Instanciation des applications clientes avec MSAL.js](msal-js-initializing-client-applications.md)
