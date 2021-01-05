---
title: Résolution des problèmes liés aux délais d’expiration et aux erreurs des réponses client à l’aide de Gestion des API
description: Résoudre les erreurs de connexion intermittente et les problèmes de latence connexes dans Gestion des API
author: vladvino
ms.topic: troubleshooting
ms.date: 12/04/2020
ms.author: apimpm
ms.service: api-management
ms.openlocfilehash: 770a8191b1b07a7ebc779b84f443ae96d66d1c97
ms.sourcegitcommit: 8b4b4e060c109a97d58e8f8df6f5d759f1ef12cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96839665"
---
# <a name="troubleshooting-client-response-timeouts-and-errors-with-api-management"></a>Résolution des problèmes liés aux délais d’expiration et aux erreurs des réponses client à l’aide de Gestion des API

Cet article vous aide à résoudre les erreurs de connexion intermittente et les problèmes de latence connexes dans [Gestion des API Azure](./api-management-key-concepts.md). Plus précisément, cet article fournit des informations supplémentaires et des conseils de résolution des problèmes relatifs à l’épuisement des ports de traduction d’adresses réseau sources (SNAT). Si vous avez besoin d’une aide supplémentaire, contactez les experts Azure sur la page [Support de la communauté Azure](https://azure.microsoft.com/support/community/) ou ouvrez une demande de support auprès de [Support Azure](https://azure.microsoft.com/support/options/).

## <a name="symptoms"></a>Symptômes

Les applications clientes appelant des API par le biais de votre service Gestion des API (APIM) peuvent présenter un ou plusieurs des symptômes suivants :

* Erreurs HTTP 500 intermittentes
* Messages d'erreur de dépassement de délai d'expiration

Ces symptômes se manifestent sous la forme d’instances `BackendConnectionFailure` dans vos [journaux de ressources Azure Monitor](../azure-monitor/platform/resource-logs.md).

## <a name="cause"></a>Cause

Ce modèle de symptômes se produit souvent en raison des limites de port SNAT avec votre service APIM.

Chaque fois qu’un client appelle l’une de vos API APIM, le service Gestion des API Azure ouvre un port SNAT pour accéder à votre API back-end. Comme évoqué dans [Connexions sortantes dans Azure](../load-balancer/load-balancer-outbound-connections.md), Azure utilise la traduction d’adresses réseau sources (SNAT) et un équilibreur de charge (non exposé aux clients) pour communiquer avec les points de terminaison en dehors d’Azure dans l’espace d’adressage IP public, ainsi que les points de terminaison internes d’Azure qui n’utilisent pas de [points de terminaison de service de réseau virtuel](../virtual-network/virtual-network-service-endpoints-overview.md). Cette situation s’applique uniquement aux API back-end exposées sur des IP publiques.

Chaque instance du service Gestion des API reçoit initialement un nombre pré-alloué de ports SNAT. Cette limite affecte l’ouverture des connexions à la même combinaison hôte/port. Les ports SNAT sont utilisés lorsque vous répétez des appels à la même combinaison adresse/port. Une fois qu’un port SNAT a été libéré, le port peut être réutilisé en fonction des besoins. L’équilibreur de charge réseau Azure récupère les ports SNAT des connexions fermées seulement après avoir attendu quatre minutes.

Une succession rapide de requêtes client à vos API peut épuiser le quota pré-alloué des ports SNAT si ces ports ne sont pas fermés et recyclés suffisamment rapidement, empêchant ainsi le service APIM de traiter les requêtes client en temps opportun.

## <a name="mitigations-and-solutions"></a>Atténuations et solutions

Pour résoudre le problème de l’épuisement des ports SNAT, vous devez d’abord diagnostiquer et optimiser les performances de vos services principaux.

Les stratégies générales d’atténuation de l’épuisement des ports SNAT sont décrites dans la section [Résolution des problèmes liés aux défaillances des connexions sortantes](../load-balancer/troubleshoot-outbound-connection.md) de la documentation *Azure Load Balancer*. Parmi ces stratégies, les suivantes sont applicables à Gestion des API.

### <a name="scale-your-apim-instance"></a>Mettre à l’échelle votre instance APIM

Chaque instance Gestion des API se voit allouer un certain nombre de ports SNAT, basé sur les unités APIM. Vous pouvez allouer des ports SNAT supplémentaires en mettant à l’échelle votre instance Gestion des API avec des unités supplémentaires. Pour plus d’informations, consultez [Mettre à l’échelle le service Gestion des API](upgrade-and-scale.md#scale-your-api-management-service).

> [!NOTE]
> L’utilisation de ports SNAT n’est actuellement pas disponible comme métrique de mise à l’échelle automatique des unités Gestion des API.

### <a name="use-multiple-ips-for-your-backend-urls"></a>Utiliser plusieurs adresses IP pour vos URL back-end

Chaque connexion de votre instance APIM à la même adresse IP de destination et au même port de destination de votre service principal utilisera un port SNAT afin de conserver un flux de trafic distinct. Sans des ports SNAT distincts pour le trafic de retour provenant de votre service principal, APIM n’aurait aucun moyen de distinguer une réponse d’une autre.

Étant donné que les ports SNAT peuvent être réutilisés si l’adresse IP ou le port de destination sont différents, une autre façon d’éviter l’épuisement des ports SNAT consiste à utiliser plusieurs adresses IP pour les URL de votre service principal.

Pour plus d’informations, consultez [Azure Load Balancer en tant que proxy sortant](../load-balancer/load-balancer-outbound-connections.md).

### <a name="place-your-apim-and-backend-service-in-the-same-vnet"></a>Placer APIM et votre service principal dans le même réseau virtuel

Si votre API back-end est hébergée sur un service Azure qui prend en charge les *points de terminaison de service* tels qu’App Service, vous pouvez éviter les problèmes d’épuisement des ports SNAT en plaçant votre instance APIM et votre service principal dans le même réseau virtuel et en exposant ce dernier via des [points de terminaison de service](../virtual-network/virtual-network-service-endpoints-overview.md) ou des [points de terminaison privés](../private-link/private-endpoint-overview.md). Lorsque vous utilisez un réseau virtuel commun et placez des points de terminaison de service sur le sous-réseau d’intégration, le trafic sortant de votre instance APIM vers ces services ignore Internet, évitant ainsi les restrictions de port SNAT. De même, si vous utilisez un réseau virtuel régional et des points de terminaison privés, aucun problème de ports SNAT sortants ne sera lié à cette destination.

Pour plus d’informations, consultez [Utilisation de Gestion des API Azure avec des réseaux virtuels](api-management-using-with-vnet.md) et [Intégrer App Service à un réseau virtuel Azure](../app-service/web-sites-integrate-with-vnet.md).

### <a name="place-your-apim-in-a-virtual-network-and-route-outbound-calls-to-azure-firewall"></a>Placer APIM dans un réseau virtuel et acheminer les appels sortants vers Pare-feu Azure

À l’instar du placement d’APIM et de vos services principaux dans un réseau virtuel, vous pouvez utiliser Pare-feu Azure dans un réseau virtuel avec votre service APIM, puis acheminer les appels APIM sortants vers Pare-feu Azure. Entre APIM et Pare-feu Azure (qui se trouvent dans le même réseau virtuel), aucun port SNAT n’est requis. Pour les connexions SNAT à vos services principaux, Pare-feu Azure dispose de 64 000 ports disponibles, un nombre bien plus élevé que celui alloué aux instances APIM.

Pour plus d’informations, consultez la documentation de [Pare-feu Azure](../firewall/overview.md).

### <a name="consider-response-caching-and-other-backend-performance-tuning"></a>Envisager la mise en cache des réponses et d’autres réglages des performances du back-end

Une autre atténuation possible à envisager est l’amélioration du temps de traitement de vos API back-end. Pour ce faire, vous pouvez configurer certaines API avec la mise en cache des réponses afin de réduire la latence entre les applications clientes qui appellent votre API et la charge de votre back-end APIM.

Pour plus d’informations, consultez [Ajouter une mise en cache pour améliorer les performances dans Gestion des API Azure](api-management-howto-cache.md).

### <a name="consider-implementing-access-restriction-policies"></a>Envisager d’implémenter des stratégies de restriction d’accès

Si cela est pertinent pour votre scénario d’entreprise, vous pouvez implémenter des stratégies de restriction d’accès pour votre produit Gestion des API. Par exemple, la stratégie `rate-limit-by-key` peut être utilisée pour éviter les pics d’utilisation des API par clé en limitant le débit d’appels pour une période donnée.

Pour plus d’informations, consultez [Stratégies de restriction des accès de Gestion des API](api-management-access-restriction-policies.md).

## <a name="see-also"></a>Voir aussi

* [Azure Load Balancer : Résolution des problèmes liés aux défaillances des connexions sortantes](../load-balancer/troubleshoot-outbound-connection.md)
* [Azure App Service : Résolution des erreurs de connexion sortante intermittente](../app-service/troubleshoot-intermittent-outbound-connection-errors.md)
