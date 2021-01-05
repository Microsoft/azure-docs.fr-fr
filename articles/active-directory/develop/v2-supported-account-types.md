---
title: Types de comptes pris en charge – Plateforme d’identités Microsoft | Azure
description: Documentation conceptuelle sur les publics et les types de compte pris en charge dans les applications
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 07/14/2020
ms.author: jmprieur
ms.reviewer: saeeda
ms.custom: aaddev, identityplatformtop40
ms.openlocfilehash: f0895cfc14b0237db8f2f762602fee949354e6cb
ms.sourcegitcommit: fa807e40d729bf066b9b81c76a0e8c5b1c03b536
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97347453"
---
# <a name="supported-account-types"></a>Types de comptes pris en charge

Cet article explique quels types de comptes (parfois appelés *audiences*) sont pris en charge dans les applications de Plateforme d’identités Microsoft.

<!-- This section can be in an include for many of the scenarios (SPA, web app signing-in users, protecting a web API, Desktop (depending on the flows), Mobile -->

## <a name="account-types-in-the-public-cloud"></a>Types de comptes dans le cloud public

Dans le cloud public Microsoft Azure, la plupart des types d’applications peuvent connecter des utilisateurs avec toute audience :

- Si vous écrivez une application métier, vous pouvez connecter des utilisateurs à votre propre organisation. Une telle application est parfois dite *à client unique*.
- Si vous êtes un éditeur de logiciels, vous pouvez écrire une application qui connecte des utilisateurs :

  - À n’importe quelle organisation. Une telle application est appelée application web *mutualisée*. Vous noterez parfois qu’elle connecte des utilisateurs à leurs comptes professionnels ou scolaires.
  - Avec leurs comptes Microsoft professionnels, scolaires ou personnels.
  - Uniquement avec des comptes Microsoft personnels.
    
- Si vous écrivez une application B2C, vous pouvez également connecter des utilisateurs avec leurs identités sociales en utilisant Active Directory B2C (Azure AD B2C).

## <a name="account-type-support-in-authentication-flows"></a>Prise en charge du type de compte dans les flux d’authentification

Certains types de compte ne peuvent pas être utilisés avec certains flux d’authentification. Par exemple, dans les applications de bureau, UWP ou de démon :

- Des applications de démon ne peuvent être utilisées qu’avec des organisations Azure AD. Il n’est pas judicieux d’essayer d’utiliser des applications de démon pour manipuler des comptes personnels Microsoft. Le consentement de l’administrateur ne sera jamais accordé.
- Vous pouvez utiliser le flux de l’authentification Windows intégrée uniquement avec des comptes professionnels ou scolaires (dans votre organisation ou une autre). L’authentification Windows intégrée fonctionne avec des comptes de domaine et nécessite que les ordinateurs soient joints à un domaine ou joints à Azure AD. Ce flux est inutile pour des comptes Microsoft personnels.
- L’[octroi d’informations d’identification liées au mot de passe du propriétaire de ressource](./v2-oauth-ropc.md) (nom d’utilisateur/mot de passe) ne peut pas être utilisé avec des comptes Microsoft personnels. Les comptes Microsoft personnels nécessitent que l’utilisateur consente à l’accès aux ressources personnelles à chaque session de connexion. C’est pourquoi ce comportement n’est pas compatible avec des flux non interactifs.

## <a name="account-types-in-national-clouds"></a>Types de compte dans les clouds nationaux

Les applications peuvent également connecter les utilisateurs à des [clouds nationaux](authentication-national-cloud.md). Toutefois, les comptes personnels Microsoft ne sont pas pris en charge dans ces clouds. C’est pourquoi les types de comptes pris en charge sont réduits, pour ces clouds, à votre organisation (un seul locataire) ou à toute organisation (applications mutualisées).

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur la [location dans Azure Active Directory](./single-and-multi-tenant-apps.md).
- En savoir plus sur les [clouds nationaux](./authentication-national-cloud.md).
