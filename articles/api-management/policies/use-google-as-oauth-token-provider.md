---
title: Exemple de stratégie de gestion des API - Autoriser l’accès à l’aide du jeton Google OAuth
titleSuffix: Azure API Management
description: Exemple de stratégie de gestion des API Azure – Comment autoriser l’accès à vos points de terminaison en utilisant Google comme fournisseur de jeton OAuth.
services: api-management
documentationcenter: ''
author: vladvino
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 10/13/2017
ms.author: apimpm
ms.openlocfilehash: 0f6c9fe2146414f78e90d6ade1a00045cdf3a04f
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92078015"
---
# <a name="authorize-access-using-google-oauth-token"></a>Autoriser l’accès à l’aide du jeton Google OAuth

Cet article montre un exemple de stratégie de Gestion des API Azure qui illustre comment autoriser l’accès à vos points de terminaison en utilisant Google comme fournisseur de jeton OAuth. Pour définir ou modifier un code de stratégie, suivez les étapes décrites dans [Définir ou modifier une stratégie](../set-edit-policies.md). Pour voir d’autres exemples, consultez [Exemples de stratégie](../policy-reference.md).

## <a name="policy"></a>Stratégie

Collez le code dans le bloc **inbound**.

[!code-xml[Main](../../../api-management-policy-samples/examples/Simple Google OAuth validate-jwt.policy.xml)]

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les stratégies APIM :

+ [Stratégies de transformation](../api-management-transformation-policies.md)
+ [Exemples de stratégie](../policy-reference.md)