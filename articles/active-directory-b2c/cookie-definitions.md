---
title: Définitions de cookie
titleSuffix: Azure AD B2C
description: Cet article fournit les définitions de certains cookies utilisés dans Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 01/23/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 5e44e2e1eb37e808e60134a6fba5051552e84029
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85389341"
---
# <a name="cookies-definitions-for-azure-ad-b2c"></a>Définitions de cookies pour Azure AD B2C

Les sections suivantes fournissent des informations sur les cookies utilisés dans Azure Active Directory B2C (Azure AD B2C).

## <a name="samesite"></a>SameSite

Le service Microsoft Azure AD B2C est compatible avec les configurations de navigateur SameSite, notamment la prise en charge de `SameSite=None` avec l’attribut `Secure`.

Pour protéger l’accès aux sites, les navigateurs web introduisent un nouveau modèle sécurisé par défaut reposant sur le principe que, sauf spécification contraire, tous les cookies doivent être protégés contre un accès externe. Le navigateur Chrome est le premier à implémenter ce changement, à commencer par [Chrome 80 en février 2020](https://www.chromium.org/updates/same-site). Pour plus d’informations sur la préparation au changement dans Chrome, voir [Developers: Get Ready for New SameSite=None; Secure Cookie Settings](https://blog.chromium.org/2019/10/developers-get-ready-for-new.html) sur le blog Chromium.

Les développeurs doivent utiliser le nouveau paramètre de cookie, `SameSite=None`, pour désigner les cookies pour l’accès intersite. Quand l’attribut `SameSite=None` est présent, un attribut `Secure` supplémentaire doit être utilisé pour que les cookies intersites ne soient accessibles que via des connexions HTTPs. Validez et testez toutes vos applications, y compris celles qui utilisent Azure AD B2C.

Pour plus d'informations, consultez les pages suivantes :

* [Gérer les changements de cookie SameSite dans le navigateur Chrome](../active-directory/develop/howto-handle-samesite-cookie-changes-chrome-browser.md)
* [Effet sur des sites web client et des services et produits Microsoft dans Chrome, versions 80 ou ultérieures](https://support.microsoft.com/help/4522904/potential-disruption-to-customer-websites-in-latest-chrome)

## <a name="cookies"></a>Cookies

Le tableau suivant répertorie les cookies utilisés dans Azure Active Directory B2C.

| Nom | Domain | Expiration | Objectif |
| ----------- | ------ | -------------------------- | --------- |
| `x-ms-cpim-admin` | main.b2cadmin.ext.azure.com | Fin de la [session de navigateur](session-behavior.md) | Contient les données d’appartenance de l’utilisateur sur tous les locataires. Les locataires dont un utilisateur est membre et son niveau d’appartenance (Admin ou Utilisateur). |
| `x-ms-cpim-slice` | b2clogin.com, login.microsoftonline.com, domaine personnalisé | Fin de la [session de navigateur](session-behavior.md) | Utilisé pour acheminer les requêtes vers l’instance de production appropriée. |
| `x-ms-cpim-trans` | b2clogin.com, login.microsoftonline.com, domaine personnalisé | Fin de la [session de navigateur](session-behavior.md) | Utilisé pour le suivi des transactions (nombre de requêtes d’authentification Azure AD B2C) et la transaction en cours. |
| `x-ms-cpim-sso:{Id}` | b2clogin.com, login.microsoftonline.com, domaine personnalisé | Fin de la [session de navigateur](session-behavior.md) | Permet de gérer la session d’authentification unique. |
| `x-ms-cpim-cache:{id}_n` | b2clogin.com, login.microsoftonline.com, domaine personnalisé | Fin de la [session de navigateur](session-behavior.md), authentification réussie | Permet de gérer l’état de la requête. |
| `x-ms-cpim-csrf` | b2clogin.com, login.microsoftonline.com, domaine personnalisé | Fin de la [session de navigateur](session-behavior.md) | Jeton de falsification de requête intersites utilisé pour la protection CRSF. |
| `x-ms-cpim-dc` | b2clogin.com, login.microsoftonline.com, domaine personnalisé | Fin de la [session de navigateur](session-behavior.md) | Utilisé pour le routage du réseau Azure AD B2C. |
| `x-ms-cpim-ctx` | b2clogin.com, login.microsoftonline.com, domaine personnalisé | Fin de la [session de navigateur](session-behavior.md) | Context |
| `x-ms-cpim-rp` | b2clogin.com, login.microsoftonline.com, domaine personnalisé | Fin de la [session de navigateur](session-behavior.md) | Utilisé pour le stockage des données d’appartenance pour le locataire du fournisseur de ressources. |
| `x-ms-cpim-rc` | b2clogin.com, login.microsoftonline.com, domaine personnalisé | Fin de la [session de navigateur](session-behavior.md) | Utilisé pour le stockage du cookie de relais. |
