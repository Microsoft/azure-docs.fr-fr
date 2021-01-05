---
title: Authentification Windows Virtual Desktop – Azure
description: Méthodes d’authentification pour Windows Virtual Desktop.
services: virtual-desktop
author: Heidilohr
ms.service: virtual-desktop
ms.topic: conceptual
ms.date: 09/04/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 5681228e5da2708912d69f16a4b09a4a93d8bb04
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89500298"
---
# <a name="supported-authentication-methods"></a>Méthodes d’authentification prises en charge

Dans cet article, nous vous fournirons un aperçu des types d’authentification que vous pouvez utiliser dans Windows Virtual Desktop.

## <a name="session-host-authentication"></a>Authentification de l’hôte de session

Windows Virtual Desktop prend en charge NT LAN Manager (NTLM) et Kerberos pour l’authentification de l’hôte de session. Toutefois, pour utiliser Kerberos, le client doit obtenir des tickets de sécurité Kerberos auprès d’un service de centre de distribution de clés (KDC, Key Distribution Center) fonctionnant sur un contrôleur de domaine. Pour obtenir des tickets, le client a besoin d’une ligne de vue directe sur le contrôleur de domaine. Vous pouvez obtenir une ligne de vue directe en utilisant le réseau de votre entreprise. Vous pouvez également utiliser une connexion VPN qui vous relie au réseau de votre entreprise.

Voici les méthodes de connexion actuellement prises en charge :

- Client Windows Desktop
    - Nom d’utilisateur et mot de passe
    - Carte à puce
    - Windows Hello
- Client Windows Store
    - Nom d’utilisateur et mot de passe
- Client web
    - Nom d’utilisateur et mot de passe
- Android
    - Nom d’utilisateur et mot de passe
- iOS
    - Nom d’utilisateur et mot de passe
- macOS
    - Nom d’utilisateur et mot de passe

>[!NOTE]
>La carte à puce et Windows Hello peuvent uniquement utiliser Kerberos pour se connecter. La connexion avec Kerberos nécessite une ligne de vue sur le contrôleur de domaine.

## <a name="hybrid-identity"></a>Identité hybride

Windows Virtual Desktop prend en charge les [identités hybrides](../active-directory/hybrid/whatis-hybrid-identity.md) par le biais d’Azure Active Directory (AD), notamment celles fédérées à l’aide des services de fédération Active Directory (AD FS). Comme les utilisateurs doivent pouvoir être découverts via Azure AD, Windows Virtual Desktop ne prend pas en charge les déploiements autonomes d’Active Directory avec AD FS.

## <a name="single-sign-on-sso"></a>Authentification unique (SSO)

Windows Virtual Desktop ne prend actuellement pas en charge les services de fédération Active Directory (AD FS) pour l’authentification unique.

La seule façon de ne pas être invité à entrer vos informations d’identification pour l’hôte de la session consiste à les enregistrer dans le client. Nous vous recommandons de le faire uniquement avec des appareils sécurisés pour empêcher d’autres utilisateurs d’accéder à vos ressources.

## <a name="next-steps"></a>Étapes suivantes

Vous êtes curieux de connaître d’autres moyens de sécuriser votre déploiement ? Consultez les [meilleures pratiques relatives à la sécurité](security-guide.md).
