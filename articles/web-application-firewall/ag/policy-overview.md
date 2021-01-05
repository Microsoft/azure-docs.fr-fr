---
title: Vue d'ensemble de la stratégie du pare-feu d’applications web (WAF) Azure
description: Cet article offre une vue d’ensemble des stratégies globales, par site et par URI du pare-feu d’applications web (WAF).
services: web-application-firewall
ms.topic: article
author: winthrop28
ms.service: web-application-firewall
ms.date: 11/20/2020
ms.author: victorh
ms.openlocfilehash: 59ca0b85ba2aff29bdb2ad3379c1054041d2b4cb
ms.sourcegitcommit: 84e3db454ad2bccf529dabba518558bd28e2a4e6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96518734"
---
# <a name="azure-web-application-firewall-waf-policy-overview"></a>Vue d'ensemble de la stratégie du pare-feu d’applications web (WAF) Azure

Les stratégies de pare-feu d’applications web contiennent tous les paramètres et configurations WAF. Cela englobe les exclusions, les règles personnalisées, les règles managées, etc. Ces stratégies sont ensuite associées à une passerelle d’application (globale), à un écouteur (par site) ou à une règle basée sur le chemin d’accès (par URI) afin d'être appliquées.

Il n'existe aucune limite quant au nombre de stratégies que vous pouvez créer. Lorsque vous créez une stratégie, celle-ci doit être associée à une passerelle d’application pour être appliquée. Elle peut être associée à n’importe quelle combinaison de passerelles d’application, d’écouteurs et de règles basées sur le chemin d’accès.

## <a name="global-waf-policy"></a>Stratégie WAF globale

Lorsque vous associez une stratégie WAF globalement, chaque site situé derrière votre Application Gateway WAF est protégé par les mêmes règles managées, règles personnalisées, exclusions et tout autre paramètre configuré.

Si vous souhaitez qu’une seule stratégie s’applique à tous les sites, vous pouvez associer la stratégie à la passerelle d’application. Pour plus d’informations sur la création et l'application d'une stratégie WAF à l'aide du portail Azure, consultez [Créer des stratégies de pare-feu d’applications web pour Application Gateway](create-waf-policy-ag.md). 

## <a name="per-site-waf-policy"></a>Stratégie WAF par site

Les stratégies WAF par site vous permettent de protéger plusieurs sites présentant différents besoins de sécurité derrière un WAF unique à l’aide de stratégies par site. Par exemple, s’il existe cinq sites derrière votre WAF, vous pouvez avoir cinq stratégies WAF distinctes (une par écouteur) de façon à pouvoir personnaliser les exclusions, les règles personnalisées et les ensembles de règles managés, et tous les autres paramètres WAF pour chaque site.

Par exemple, une stratégie globale est appliquée à votre passerelle d’application. Vous appliquez ensuite une stratégie différente à un écouteur sur cette passerelle d’application. La stratégie de l’écouteur s'applique maintenant uniquement à cet écouteur. La stratégie globale d’Application Gateway s’applique encore à tous les autres écouteurs et règles basées sur le chemin auxquels aucune stratégie spécifique n’est attribuée.

## <a name="per-uri-policy"></a>Stratégie par URI

Pour plus de personnalisation au niveau de l’URI, vous pouvez associer une stratégie WAF à une règle basée sur le chemin d’accès. Si certaines pages d’un même site requièrent des stratégies différentes, vous pouvez apporter des modifications à la stratégie WAF qui affectent uniquement un URI donné. Cela peut s’appliquer à une page de paiement ou de connexion, ou à tout autre URI nécessitant une stratégie WAF plus spécifique que les autres sites situés derrière votre WAF.

À l'instar des stratégies WAF par site, les stratégies plus spécifiques remplacent les stratégies moins spécifiques. Ainsi, une stratégie par URI sur un mappage de chemin d’URL remplace toute stratégie de WAF par site ou globale de niveau supérieur.

### <a name="example"></a>Exemple

Imaginons que vous avez trois sites : contoso.com, fabrikam.com et adatum.com derrière la même passerelle d'application. Vous souhaitez qu’un WAF soit appliqué aux trois sites, mais vous avez besoin d’une sécurité renforcée avec adatum.com, car c’est là que les clients accèdent, parcourent et achètent des produits.

Vous pouvez appliquer une stratégie globale au WAF, avec des paramètres de base, des exclusions ou des règles personnalisées, si besoin, afin d'éviter que des faux positifs ne bloquent le trafic. Dans ce cas, vous n'êtes pas tenu d'exécuter les règles d’injection SQL globales car fabrikam.com et contoso.com correspondent à des pages statiques, sans serveur SQL principal. Vous pouvez donc désactiver ces règles dans la stratégie globale.

Cette stratégie globale est adaptée à contoso.com et fabrikam.com, mais il vous faut être plus vigilant avec adatum.com qui gère les informations de connexion et les paiements. Vous pouvez appliquer une stratégie par site à l’écouteur adatum et conserver les règles SQL en cours d’exécution. Supposons également qu'un cookie bloque partiellement le trafic, vous pouvez créer une exclusion pour ce cookie afin d’arrêter le faux positif. 

Il vous faut prêter une attention particulière à l'URI adatum.com/payments. Dès lors, appliquez une autre stratégie à cet URI, laissez toutes les règles activées et supprimez également toutes les exclusions.

Dans cet exemple, vous disposez d’une stratégie globale qui s’applique à deux sites. Vous disposez d'une stratégie par site qui s’applique à un site et d'une stratégie par URI qui s’applique à une règle basée sur le chemin d’accès spécifique. Consultez [Configurer des stratégies WAF par site à l’aide d’Azure PowerShell](per-site-policies.md) pour la commande PowerShell correspondante de cet exemple.

## <a name="existing-waf-configurations"></a>Configurations WAF existantes

Tous les nouveaux paramètres WAF du pare-feu d’applications web (règles personnalisées, configurations d’ensemble de règles managé, exclusions, etc.) sont présents dans une stratégie WAF. Si vous disposez d’un WAF, ces paramètres peuvent toujours exister dans la configuration de votre WAF. Pour plus d’informations sur l’adoption de la nouvelle stratégie WAF, voir [Migrer une configuration WAF vers une stratégie WAF](./migrate-policy.md). 


## <a name="next-steps"></a>Étapes suivantes

- [Créez des stratégies par site et par URI à l’aide d’Azure PowerShell](per-site-policies.md).
