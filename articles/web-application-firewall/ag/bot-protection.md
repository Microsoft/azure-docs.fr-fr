---
title: Configurer la protection bot pour le pare-feu d’applications web (WAF)
description: Découvrez comment configurer la protection bot pour le pare-feu d’applications web (WAF) sur Azure Application Gateway.
services: web-application-firewall
ms.topic: article
author: vhorne
ms.service: web-application-firewall
ms.date: 10/31/2019
ms.author: victorh
ms.openlocfilehash: 89c863e85d9eab27a47bc1bf7b98cd1c8d89e900
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "73511522"
---
# <a name="configure-bot-protection-for-web-application-firewall-on-azure-application-gateway-preview"></a>Configurer la protection bot pour le pare-feu d’applications web (WAF) sur Azure Application Gateway (préversion)

Cet article explique comment configurer une règle de protection bot dans le pare-feu d’applications Web (WAF) Azure pour Application Gateway à l’aide du Portail Azure. 

Vous pouvez activer un ensemble de règles de protection bot managées pour votre WAF afin de bloquer ou de journaliser des requêtes provenant d’adresses IP malveillantes. Ces adresses IP proviennent du flux Microsoft Threat Intelligence. Intelligent Security Graph alimente l’intelligence des menaces Microsoft et est utilisé par de nombreux services, dont Azure Security Center.

> [!NOTE]
> Un ensemble de règles de protection bot, actuellement disponible en préversion publique, est fourni avec un contrat de niveau de service en préversion. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d'informations, consultez les  [Conditions d’utilisation supplémentaires des préversions de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) .

## <a name="prerequisites"></a>Conditions préalables requises

Créez une stratégie de WAF de base pour Application Gateway en suivant les instructions décrites dans  [Créer des stratégies de pare-feu d’applications web pour Application Gateway](create-waf-policy-ag.md).

## <a name="enable-bot-protection-rule-set"></a>Activer un ensemble de règles de protection bot

1. Dans la page de la stratégie **de base** que vous avez créée précédemment, sous **Paramètres**, sélectionnez **Règles**.  

2. Dans la page de détails, sous la section  **Gérer les règles** , dans le menu déroulant, cochez la case située devant la règle Protection bot, puis sélectionnez **Enregistrer**.

> [!div class="mx-imgBorder"]
> ![Protection bot](../media/bot-protection/bot-protection.png)

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les règles personnalisées, consultez [Règles personnalisées pour un pare-feu d’applications web v2 sur Azure Application Gateway](custom-waf-rules-overview.md).