---
title: API REST Azure App Configuration – Autorisation
description: Pages de référence pour l’autorisation à l’aide de l’API REST Azure App Configuration
author: AlexandraKemperMS
ms.author: alkemper
ms.service: azure-app-configuration
ms.topic: reference
ms.date: 08/17/2020
ms.openlocfilehash: 70f05799ce2856ad490937a17b456e78789088f1
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96932640"
---
# <a name="authorization"></a>Autorisation

L’autorisation fait référence à la procédure utilisée pour déterminer les autorisations dont dispose un appelant lors de l’établissement d’une demande. Il existe plusieurs modèles d’autorisation. Le modèle d’autorisation utilisé pour une demande dépend de la méthode d’[authentification](./rest-api-authentication-index.md) utilisée. Les modèles d’autorisation sont répertoriés ci-dessous.

## <a name="hmac"></a>HMAC

Le [modèle d’autorisation](./rest-api-authorization-hmac.md) associé à l’authentification HMAC fractionne les autorisations en lecture seule ou en lecture-écriture. Pour plus d’informations, consultez la page [Autorisation HMAC](./rest-api-authorization-hmac.md) .

## <a name="azure-active-directory"></a>Azure Active Directory

Le [modèle d’autorisation](./rest-api-authorization-azure-ad.md) associé à l’authentification Azure Active Directory (Azure AD) utilise un RBAC Azure pour contrôler les autorisations. Pour plus d’informations, consultez la page [Autorisation Azure AD](./rest-api-authorization-azure-ad.md).
