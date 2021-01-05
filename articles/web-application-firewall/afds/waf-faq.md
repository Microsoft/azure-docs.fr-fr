---
title: Pare-feu d’application web Azure sur Azure Front Door Service - Questions fréquentes (FAQ)
description: Cet article fournit des réponses aux questions les plus souvent posées sur le pare-feu d’applications web sur Azure Front Door.
services: web-application-firewall
author: vhorne
ms.service: web-application-firewall
ms.devlang: na
ms.topic: article
ms.date: 05/05/2020
ms.author: victorh
ms.openlocfilehash: 5c2763112b1aa2d58f5dc57cea72a3d0bdea961e
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95545667"
---
# <a name="frequently-asked-questions-for-azure-web-application-firewall-on-azure-front-door-service"></a>Forum aux questions sur le pare-feu d’applications web Azure sur Azure Front Door Service

Cet article répond aux questions courantes sur les fonctions et fonctionnalités du pare-feu d’application web (WAF) Azure sur Azure Front Door Service. 

## <a name="what-is-azure-waf"></a>Qu’est-ce qu’Azure WAF ?

Azure WAF est un pare-feu d’applications web qui aide à protéger vos applications web des menaces courantes telles que les injections de code SQL, le script de site à site et autres codes malveillants exploitant des failles de sécurité. Vous pouvez définir une stratégie de pare-feu d’applications web consistant en une combinaison de règles personnalisées et gérées pour contrôler l’accès à vos applications web.

Une stratégie de pare-feu d’applications web Azure peut être appliquée aux applications web hébergées sur Application Gateway ou Azure Front Door.

## <a name="what-is-waf-on-azure-front-door"></a>Qu’est-ce qu’un pare-feu d’applications web sur Azure Front Door ? 

Azure Front Door est un réseau de distribution de contenu et d’application hautement évolutif et distribué mondialement. Un pare-feu d’applications web Azure, lorsqu’intégré à Front Door, bloque les attaques DDoS et ciblant une application à la périphérie du réseau Azure, près de la source avant qu’elles ne pénètrent dans votre réseau virtuel, et offre une protection sans sacrifice des performances.

## <a name="does-azure-waf-support-https"></a>Le pare-feu d’applications web prend-il en charge HTTPS ?

Front Door offre le déplacement du traitement TLS. Le pare-feu d’applications web est intégré de façon native à Front Door et peut inspecter une demande après qu’elle ait été déchiffrée.

## <a name="does-azure-waf-support-ipv6"></a>Le pare-feu d’applications web prend-il en charge IPv6 ?

Oui. Vous pouvez configurer la restriction IP pour IPv4 et IPv6.

## <a name="how-up-to-date-are-the-managed-rule-sets"></a>Les ensembles de règles gérées sont-ils à jour ?

Nous faisons de notre mieux pour suivre le rythme d’un monde de menaces en constante évolution. Lorsqu’une nouvelle règle est mise à jour, elle est ajoutée à l’ensemble de règles par défaut avec un nouveau numéro de version.

## <a name="what-is-the-propagation-time-if-i-make-a-change-to-my-waf-policy"></a>Quelle est la durée de propagation si j’effectue une modification de ma stratégie de pare-feu d’applications web ?

Le déploiement global d’une stratégie de pare-feu d’applications web prend généralement 5 minutes, voire moins.

## <a name="can-waf-policies-be-different-for-different-regions"></a>Les stratégies de pare-feu d’applications web peuvent-elles être différentes d’une région à une autre ?

Une fois intégré à Front Door, le pare-feu d’applications web constitue une ressource globale. La même configuration s’applique à tous les emplacements Front Door.
 
## <a name="how-do-i-limit-access-to-my-back-end-to-be-from-front-door-only"></a>Comment limiter l’accès à mon serveur back-end au service Front Door uniquement ?

Vous pouvez configurer une liste de contrôle d’accès dans votre back-end pour n’autoriser que les plages d’adresses IP sortantes de Front Door et refuser tout accès direct depuis Internet. Les balises de service sont prises en charge pour une utilisation sur votre réseau virtuel. De plus, vous pouvez vérifier que le champ de l’en-tête HTTP X-Forwarded-Host est valide pour votre application web.

## <a name="which-azure-waf-options-should-i-choose"></a>Quelles options de pare-feu d’applications web Azure dois-je choisir ?

Il y a deux options lors de l’application de stratégies de pare-feu d’applications web dans Azure. Le pare-feu d’applications web avec Azure Front Door est une solution distribuée mondialement de sécurité en périphérie. Le pare-feu d’applications web avec Application Gateway est une solution dédiée et régionale. Nous vous recommandons de choisir une solution basée sur vos exigences générales en matière de performances et de sécurité. Pour en savoir plus, consultez [Équilibrage de charge avec la suite de livraison d’application Azure](../../frontdoor/front-door-lb-with-azure-app-delivery-suite.md).

## <a name="whats-the-recommended-approach-to-enabling-waf-on-front-door"></a>Quelle est l’approche recommandée pour activer le WAF sur Front Door ?

Lorsque vous activez le WAF sur une application existante, il est courant d’obtenir des détections de faux positifs, dans lesquels les règles WAF identifient le trafic légitime comme une menace. Pour réduire le risque d’impact sur vos utilisateurs, nous vous recommandons le processus suivant :

* Activez le WAF en mode [**Détection**](./waf-front-door-create-portal.md#change-mode) pour être sûr qu’il ne bloque pas les requêtes tandis que vous mettez en œuvre ce processus.
  > [!IMPORTANT]
  > Ce processus explique comment activer le WAF sur une solution nouvelle ou existante lorsque votre priorité est de réduire les perturbations pour les utilisateurs de votre application. En cas d’attaque ou de menace imminente, vous pouvez préférer déployer immédiatement le WAF en mode **Prévention**, et utiliser le processus de paramétrage pour superviser et ajuster le WAF au fil du temps. Le blocage d’une partie de votre trafic légitime en découlera probablement, c’est pourquoi nous vous recommandons d’agir ainsi uniquement lorsque vous êtes confronté à une menace.
* Suivez nos [conseils en matière de paramétrage du WAF](./waf-front-door-tuning.md). Ce processus nécessite l’activation de la journalisation des diagnostics, la consultation régulière des journaux et l’ajout d’exclusions de règles ainsi que d’autres solutions de contournement.
* Répétez ce processus dans son intégralité, en vérifiant les journaux régulièrement, jusqu’à être certain qu’aucun trafic légitime n’est bloqué. Le déroulement du processus entier peut prendre plusieurs semaines. Dans l’idéal, vous devriez constater moins de détections de faux positifs après chaque modification de paramétrage que vous effectuez.
* Au final, activez le WAF en **mode Prévention**.
* Même lorsque vous exécutez le WAF en production, il est impératif de continuer à superviser les journaux, afin d’identifier d’autres détections de faux positifs. En examinant régulièrement les journaux, vous pourrez également identifier les tentatives d’attaque réelle qui ont été bloquées.

## <a name="do-you-support-same-waf-features-in-all-integrated-platforms"></a>Prenez-vous en charge les mêmes fonctionnalités de pare-feu d’applications web dans toutes les plateformes intégrées ?

Actuellement, les règles ModSec CRS 2.2.9, CRS 3.0, et CRS 3.1 sont uniquement prises en charge avec le pare-feu d’applications web sur Application Gateway. Les règles de l’ensemble de règles par défaut Azure, la limitation du débit et le filtrage géographique sont pris en charge uniquement avec le pare-feu d’applications web sur Azure Front Door.

## <a name="is-ddos-protection-integrated-with-front-door"></a>La protection DDoS est-elle intégrée à Front Door ? 

Distribué mondialement aux périphéries réseau d’Azure, Azure Front Door peut absorber et isoler géographiquement d’importants volumes d’attaques. Vous pouvez créer une stratégie de pare-feu d’applications web personnalisée pour bloquer automatiquement et limiter le débit des attaques http(s) aux signatures connues. De plus, vous pouvez activer la protection DDos standard sur le réseau virtuel où vos back-ends sont déployés. Les clients de la protection DDos standard d’Azure profitent d’avantages supplémentaires dont la protection du coût, la garantie SLA et l’accès à des experts de l’équipe spécialisée en attaques DDoS, pour une aide immédiate en cas d’attaque. Pour plus d’informations, consultez [Protection DDoS sur Front Door](../../frontdoor/front-door-ddos.md).

## <a name="why-do-additional-requests-above-the-threshold-configured-for-my-rate-limit-rule-get-passed-to-my-backend-server"></a>Pourquoi les demandes au-delà du seuil configuré pour la règle de limite de débit sont-elles transmises à mon serveur principal ?

Une règle de limite de débit peut limiter le trafic anormalement élevé à partir de n’importe quelle adresse IP du client. Vous pouvez configurer un seuil sur le nombre de requêtes web autorisées à partir d’une adresse IP cliente pendant une durée d’une ou de cinq minutes. Pour contrôler le débit encore plus précisément, vous pouvez combiner des règles de limitation du débit avec des conditions de correspondance supplémentaires, telles que la mise en correspondance des paramètres HTTP/HTTPS. 

Les demandes provenant du même client arrivent souvent sur le même serveur Front Door. Dans ce cas, vous verrez que les demandes supplémentaires au-delà du seuil sont bloquées immédiatement. 

Toutefois, il est possible que les demandes émanant du même client arrivent sur un autre serveur Front Door qui n’a pas encore actualisé le compteur de limite de débit. Par exemple, un client peut ouvrir une nouvelle connexion pour chaque demande alors que le seuil est faible. Dans ce cas, la première demande au nouveau serveur Front Door réussit la vérification de limite de débit. Un seuil de limite de débit est généralement défini sur un niveau élevé pour se défendre contre les attaques par déni de service à partir de n’importe quelle adresse IP cliente. Pour un seuil très faible, vous pourriez voir des demandes au-delà du seuil.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur le [pare-feu d’application web Azure](../overview.md).
- En savoir plus sur [Azure Front Door](../../frontdoor/front-door-overview.md).