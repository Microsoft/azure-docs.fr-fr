---
title: Pare-feu d’applications web Azure sur Application Gateway – Forum aux questions
description: Cet article fournit des réponses aux questions les plus souvent posées concernant le pare-feu d’applications web sur Application Gateway.
services: web-application-firewall
author: vhorne
ms.service: web-application-firewall
ms.topic: article
ms.date: 05/05/2020
ms.author: victorh
ms.openlocfilehash: 1fbe930780ff026be748bc42104f821ee9e5c443
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92785100"
---
# <a name="frequently-asked-questions-for-azure-web-application-firewall-on-application-gateway"></a>Forum aux questions concernant le pare-feu d’applications web Azure sur Application Gateway

Cet article répond à des questions courantes concernant les fonctions et fonctionnalités du pare-feu d’applications web (WAF) sur Application Gateway. 

## <a name="what-is-azure-waf"></a>Qu’est-ce qu’Azure WAF ?

Azure WAF est un pare-feu d’applications web qui aide à protéger vos applications web des menaces courantes telles que les injections de code SQL, le script de site à site et autres codes malveillants exploitant des failles de sécurité. Vous pouvez définir une stratégie de pare-feu d’applications web consistant en une combinaison de règles personnalisées et gérées pour contrôler l’accès à vos applications web.

Une stratégie de pare-feu d’applications web Azure peut être appliquée aux applications web hébergées sur Application Gateway ou Azure Front Door.

## <a name="what-features-does-the-waf-sku-support"></a>Quelles fonctionnalités la référence (SKU) WAF prend-elle en charge ?

La référence (SKU) WAF prend-elle en charge toutes les fonctionnalités disponibles dans la référence (SKU) Standard ?

## <a name="how-do-i-monitor-waf"></a>Comment puis-je surveiller le pare-feu d’application web ?

Surveillez le pare-feu d'applications web via la journalisation des diagnostics. Pour plus d'informations, consultez [Journalisation des diagnostics et métriques pour Application Gateway](../../application-gateway/application-gateway-diagnostics.md).

## <a name="does-detection-mode-block-traffic"></a>Est-ce que le mode de détection bloque le trafic ?

Non. Le mode de détection journalise uniquement le trafic qui déclenche une règle de pare-feu d’applications web.

## <a name="can-i-customize-waf-rules"></a>Puis-je personnaliser les règles de pare-feu d’application web ?

Oui. Pour plus d’informations, consultez [Personnaliser les règles et groupes de règles WAF](application-gateway-customize-waf-rules-portal.md).

## <a name="what-rules-are-currently-available-for-waf"></a>Quelles sont les règles actuellement disponibles pour WAF ?

WAF prend actuellement en charge CRS [2.2.9](application-gateway-crs-rulegroups-rules.md#owasp229), [3.0](application-gateway-crs-rulegroups-rules.md#owasp30) et [3.1](application-gateway-crs-rulegroups-rules.md#owasp31). Ces règles constituent un rempart contre la plupart des 10 principales vulnérabilités identifiées par l'OWASP (Open Web Application Security Project) : 

* Protection contre les injections de code SQL
* Protection contre les scripts intersites
* Protection contre les attaques web courantes comme l’injection de commande, les dissimulations de requêtes HTTP, la séparation de réponse HTTP et les attaques RFI (Remote File Inclusion)
* Protection contre les violations de protocole HTTP
* Protection contre les anomalies de protocole HTTP comme un agent-utilisateur hôte manquant et les en-têtes Accept
* Protection contre les robots, les crawlers et les scanneurs
* Détection des erreurs de configuration d’application courantes (par exemple, Apache, IIS, etc.)

Pour plus d’informations, consultez [Les 10 principales vulnérabilités identifiées par l'OWASP](https://owasp.org/www-project-top-ten/).

## <a name="does-waf-support-ddos-protection"></a>WAF prend-il en charge la protection DDoS ?

Oui. Vous pouvez activer le service de protection DDoS sur le réseau virtuel sur lequel la passerelle d’application est déployée. Ainsi, le service Azure DDoS Protection protège aussi l’adresse IP virtuelle de la passerelle d’application.

### <a name="does-waf-store-customer-data"></a>Le pare-feu d'applications web stocke-t-il les données client ?

Non, le pare-feu d'applications web ne stocke pas les données client.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur le [pare-feu d’application web Azure](../overview.md).
- En savoir plus sur [Azure Front Door](../../frontdoor/front-door-overview.md).
