---
title: Exemple de stratégie de gestion des API - Filtrer sur l’adresse IP lors de l’utilisation d’Application Gateway
titleSuffix: Azure API Management
description: Exemple de stratégie de gestion des API Azure - Montre comment filtrer sur l’adresse IP de demande lors de l’utilisation d’une passerelle Application Gateway.
services: api-management
documentationcenter: ''
author: jftl6y
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 01/13/2020
ms.author: joscot
ms.custom: fasttrack-new
ms.openlocfilehash: 8249cc543c6334841c8e5152d5d1ceb84d4097dc
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92076111"
---
# <a name="filter-on-request-ip-address-when-using-an-application-gateway"></a>Filtrer sur l’adresse IP de demande lors de l’utilisation d’une passerelle Application Gateway

Cet article présente un exemple de stratégie de gestion des API Azure qui montre comment filtrer sur l’adresse IP de demande quand l’instance Gestion des API est accessible par le biais d’une passerelle Application Gateway ou d’un autre intermédiaire. Pour définir ou modifier un code de stratégie, suivez les étapes décrites dans [Définir ou modifier une stratégie](../set-edit-policies.md). Pour voir d’autres exemples, consultez [Exemples de stratégie](../policy-reference.md).

## <a name="policy"></a>Stratégie

Collez le code dans le bloc **inbound**.

[!code-xml[Main](../../../api-management-policy-samples/examples/Filter on IP Address when using Application Gateway.policy.xml)]

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les stratégies APIM :

+ [Politiques de restrictions d’accès](../api-management-access-restriction-policies.md)
+ [Exemples de stratégie](../policy-reference.md)