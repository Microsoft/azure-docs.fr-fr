---
title: Azure Front Door - Protection DDoS
description: Cette page fournit des informations sur la façon dont Azure Front Door vous aide à vous protéger contre les attaques DDoS.
services: frontdoor
documentationcenter: ''
author: johndowns
ms.service: frontdoor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/28/2020
ms.author: jodowns
ms.openlocfilehash: 24a8a559d72eae4dab0b6e740b34890f1d7ff114
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96350781"
---
# <a name="ddos-protection-on-front-door"></a>Protection DDoS sur Front Door

Azure Front Door offre plusieurs fonctionnalités et caractéristiques qui peuvent contribuer à empêcher les attaques par déni de service distribué (DDoS). Ces fonctionnalités peuvent empêcher les attaquants d’atteindre votre application et de nuire à la disponibilité et aux performances de celle-ci.

## <a name="integration-with-azure-ddos-protection-basic"></a>Intégration avec Azure DDoS Protection Basic

Front Door est protégé par Azure DDoS Protection Basic. Cette protection est intégrée à la plateforme Front Door par défaut sans frais additionnels. De par son échelle et sa capacité, le réseau déployé à l’échelle mondiale de Front Door assure une défense contre les attaques courantes de la couche réseau. Cette défense est par ailleurs renforcée par le monitoring continu du trafic et l’atténuation en temps réel. La protection DDoS de base protège également contre les avalanches de requêtes DNS de couche 7 et les attaques volumétriques de couches 3 et 4 les plus fréquentes qui ciblent les points de terminaison publics. Ce service a également fait ses preuves dans la protection des services professionnels et grand public de Microsoft contre les attaques de grande envergure. Pour plus d’informations, consultez [Azure DDoS Protection](../security/fundamentals/ddos-best-practices.md).

## <a name="protocol-blocking"></a>Blocage de protocole

Front Door accepte uniquement le trafic sur les protocoles HTTP et HTTPS, et traite uniquement les demandes valides avec un en-tête `Host` connu. Ce comportement permet de limiter certains types d’attaques par déni de service courantes, y compris les attaques volumétriques qui sont réparties entre toute une série de protocoles et de ports, les attaques d’amplifications DNS et les attaques par empoisonnement TCP.

## <a name="capacity-absorption"></a>Absorption de la capacité

Front Door est un service distribué à l’échelle mondiale et à très grande échelle. Nous avons de nombreux clients, y compris des produits cloud de grande ampleur de Microsoft, qui reçoivent des centaines de milliers de demandes chaque seconde. Front Door est situé à la périphérie du réseau d’Azure, pour absorber et isoler géographiquement les attaques à volume élevé. Cela peut empêcher le trafic malveillant d’aller plus loin que la périphérie du réseau Azure.

## <a name="caching"></a>Mise en cache

[Les fonctionnalités de mise en cache de Front Door](./front-door-caching.md) peuvent être utilisées pour protéger les back-ends des volumes de trafic importants générés par une attaque. Les ressources mises en cache sont retournées à partir des nœuds de périphérie Front Door, afin qu’elles ne soient pas transférées vers votre back-end. Même les temps d’expiration de cache courts (secondes ou minutes) sur des réponses dynamiques peuvent réduire de façon considérable la charge sur les services back-ends. Pour plus d’informations sur les concepts et les modèles de mise en cache, consultez [Considérations relatives à la mise en cache](/azure/architecture/best-practices/caching) et [modèle de réserve de caches](/azure/architecture/patterns/cache-aside).

## <a name="web-application-firewall-waf"></a>Pare-feu d’applications web (WAF)

[Le pare-feu d’applications Web (WAF) de Front Door](../web-application-firewall/afds/afds-overview.md) peut être utilisé pour limiter un certain nombre de types d’attaques :

* L’utilisation de l’ensemble de règles gérées offre une protection contre un certain nombre d’attaques courantes.
* Le trafic provenant de l’extérieur d’une région géographique définie ou à l’intérieur d’une région bien précise peut être bloqué ou redirigé vers une page web statique. Pour plus d’informations, consultez [Géo-filtrage](../web-application-firewall/afds/waf-front-door-geo-filtering.md).
* Les adresses et les plages d’adresses IP que vous identifiez comme malveillantes peuvent être bloquées.
* La limitation du débit peut être appliquée pour empêcher les adresses IP d’appeler votre service trop fréquemment.
* Vous pouvez créer des [règles WAF personnalisées](../web-application-firewall/afds/waf-front-door-custom-rules.md) pour bloquer automatiquement et limiter le débit des attaques HTTP ou HTTPS aux signatures connues.

## <a name="for-further-protection"></a>Pour une protection renforcée

Si vous avez besoin d’une protection renforcée, vous pouvez activer [la protection DDoS Azure standard](../security/fundamentals/ddos-best-practices.md#ddos-protection-standard) sur le réseau virtuel où vos back-ends sont déployés. Les clients de la protection DDoS standard profitent d’avantages supplémentaires dont la protection du coût, la garantie SLA et l’accès à des experts de l’équipe spécialisée en attaques DDoS, pour une aide immédiate en cas d’attaque.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment configurer un [profil WAF sur Front Door](front-door-waf.md). 
- Découvrez comment [créer une porte d’entrée](quickstart-create-front-door.md).
- Découvrez [comment fonctionne Front Door](front-door-routing-architecture.md).