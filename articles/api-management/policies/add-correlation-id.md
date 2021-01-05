---
title: Exemple de la stratégie de Gestion des API - Ajouter un en-tête contenant un ID de corrélation
titleSuffix: Azure API Management
description: Exemple de la stratégie de Gestion des API Azure - Montre comment ajouter un en-tête contenant un ID de corrélation à la requête entrante.
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
ms.openlocfilehash: 922565d26274aee12c9397c08c19330b4fce00e7
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92076298"
---
# <a name="add-a-header-containing-a-correlation-id"></a>Ajouter un en-tête contenant un ID de corrélation

Cet article montre un exemple de la stratégie de Gestion des API Azure qui illustre comment ajouter un en-tête contenant un ID de corrélation à la requête entrante. Pour définir ou modifier un code de stratégie, suivez les étapes décrites dans [Définir ou modifier une stratégie](../set-edit-policies.md). Pour voir d’autres exemples, consultez [Exemples de stratégie](../policy-reference.md).

## <a name="policy"></a>Stratégie

Collez le code dans le bloc **inbound**.

[!code-xml[Main](../../../api-management-policy-samples/examples/Add correlation id to inbound request.policy.xml)]

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les stratégies APIM :

+ [Stratégies de transformation](../api-management-transformation-policies.md)
+ [Exemples de stratégie](../policy-reference.md)