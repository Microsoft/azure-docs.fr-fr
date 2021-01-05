---
title: API REST Azure App Configuration – Authentification
description: Pages de référence pour l’authentification à l’aide de l’API REST Azure App Configuration
author: AlexandraKemperMS
ms.author: alkemper
ms.service: azure-app-configuration
ms.topic: reference
ms.date: 08/17/2020
ms.openlocfilehash: 58fba309f4cf7e4fc4364d075500c02e0ed45259
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96932691"
---
# <a name="authentication"></a>Authentification

Toutes les requêtes HTTP doivent être authentifiées. Les schémas d’authentification suivants sont pris en charge.

## <a name="hmac"></a>HMAC

[L’authentification HMAC](./rest-api-authentication-hmac.md) utilise un secret généré de manière aléatoire pour signer les charges utiles de la requête. Vous trouverez des détails sur le mode d’autorisation des requêtes utilisant cette méthode d’authentification dans la section [Autorisation HMAC](./rest-api-authorization-hmac.md).

## <a name="azure-active-directory"></a>Azure Active Directory

L’[authentification Azure Active Directory (Azure AD)](../active-directory/authentication/overview-authentication.md) utilise un porteur provenant d’Azure Active Directory pour authentifier les requêtes. Vous trouverez des détails sur le mode d’autorisation des requêtes utilisant cette méthode d’authentification dans la section [Autorisation Azure AD](./rest-api-authorization-azure-ad.md).