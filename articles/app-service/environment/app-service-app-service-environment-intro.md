---
title: Présentation d’ASE v1
description: Découvrez les fonctionnalités d’App Service Environment v1. Ce document s’adresse uniquement aux clients qui utilisent l’environnement ASE v1 hérité.
author: stefsch
ms.assetid: 78e6d4f5-da46-4eb5-a632-b5fdc17d2394
ms.topic: article
ms.date: 07/11/2017
ms.author: ccompy
ms.custom: seodec18
ms.openlocfilehash: 878cef39ade0487c03dba83c9e3631fe3c72088f
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96008620"
---
# <a name="introduction-to-app-service-environment-v1"></a>Présentation d’App Service Environment v1

> [!NOTE]
> Cet article traite de l’environnement App Service Environment v1.  Il existe une version plus récente de l’environnement App Service Environment, plus facile à utiliser et qui s’exécute sur des infrastructures plus puissantes. Pour en savoir plus sur la nouvelle version, commencez par la section [Présentation de l’environnement App Service Environment](intro.md).

## <a name="overview"></a>Vue d’ensemble

Un environnement App Service est une option de plan de service [Premium][PremiumTier][Azure App Service](../overview.md) qui fournit un environnement totalement isolé et dédié pour exécuter en toute sécurité des applications Azure App Service à grande échelle, comme des applications web, mobiles et d’API.  

Les environnements App Service constituent le meilleur choix pour les charges de travail applicatives avec les exigences suivantes :

* Très grande échelle
* Isolation et accès réseau sécurisé

Les clients peuvent créer plusieurs environnements App Service au sein d’une même région Azure, ainsi que dans plusieurs régions Azure.  Les environnements App Service sont donc parfaits pour l’évolution horizontale des applications sans état pour la prise en charge de lourdes charges de travail RPS.

Les environnements App Service sont isolés de façon à exécuter les applications d’un seul client et ils sont toujours déployés dans un réseau virtuel.  Les clients ont un contrôle affiné sur le trafic réseau entrant et sortant des applications, et les applications peuvent établir des connexions sécurisées à haut débit sur les réseaux virtuels avec les ressources d’entreprise locales.

Pour avoir une idée générale de la façon dont les environnements App Service permettent l’accès réseau sécurisé à grande échelle, consultez la vidéo [AzureCon Deep Dive][AzureConDeepDive] sur les environnements App Service.

Pour une exploration approfondie de la mise à l’échelle horizontale à l’aide de plusieurs environnements App Service, voir l’article sur la configuration d’une [empreinte d’application géo-distribuée][GeodistributedAppFootprint].

Pour afficher la configuration de l’architecture de sécurité illustrée dans AzureCon Deep Dive, consultez l’article sur l’implémentation d’une [architecture de sécurité en couches](app-service-app-service-environment-layered-security.md) avec les environnements App Service.

L’accès aux applications qui s’exécutent sur des environnements App Service peut être contrôlé par des appareils en amont tels que les pare-feu d’applications web (WAF).  Ce scénario est traité dans l’article sur la [configuration d’un pare-feu d’application web pour les environnements App Service](app-service-app-service-environment-web-application-firewall.md) .

[!INCLUDE [app-service-web-to-api-and-mobile](../../../includes/app-service-web-to-api-and-mobile.md)]

## <a name="dedicated-compute-resources"></a>Ressources de calcul dédiées

Toutes les ressources de calcul d’un environnement App Service sont exclusivement dédiées à un seul abonnement, et il est possible de configurer un environnement App Service avec jusqu’à cinquante (50) ressources de calcul pour une seule application.

Un environnement App Service se compose d’un pool de ressources de calcul frontal, ainsi que de un à trois pools de ressources de calcul de travail.

Le pool frontal contient des ressources de calcul responsables de l'arrêt TLS ainsi que de l'équilibrage de charge automatique des demandes d'application dans un environnement App Service.

Chaque pool de travaux contient des ressources de calcul allouées à des [plans App Service][AppServicePlan], qui, eux-mêmes, contiennent une ou plusieurs applications Azure App Service.  Étant donné qu'un environnement App Service peut comprendre jusqu'à trois pools de travaux différents, vous pouvez choisir des ressources de calcul différentes pour chaque pool de travaux.  

Cela vous permet, par exemple, de créer un pool de travaux avec des ressources de calcul moins puissantes pour les plans App Service destinés aux applications de développement ou de test.  Un deuxième (ou même troisième) pool de travaux peut utiliser des ressources de calcul plus puissantes destinées aux plans App Service exécutant des applications de production.

Pour plus d'informations sur la quantité de ressources de calcul disponibles pour les pools frontaux et de travail, consultez [Comment configurer un environnement App Service][HowToConfigureanAppServiceEnvironment].  

Pour plus d'informations sur les tailles de ressources de calcul disponibles prises en charge dans un environnement App Service, consultez la page [Service d'application Tarification][AppServicePricing] et passez en revue les options disponibles pour les environnements App Service dans le niveau de tarification Premium.

## <a name="virtual-network-support"></a>Prise en charge des réseaux virtuels

Un environnement App Service peut être créé **soit** dans un réseau virtuel Azure Resource Manager, **ou** dans un réseau virtuel de modèle de déploiement classique ([plus d’informations sur les réseaux virtuels][MoreInfoOnVirtualNetworks]).  Étant donné qu'il existe toujours un environnement App Service sur un réseau virtuel, et plus précisément sur un sous-réseau d'un réseau virtuel, vous pouvez exploiter les fonctionnalités de sécurité des réseaux virtuels pour contrôler les communications réseau entrantes et sortantes.  

Un environnement App Service peut être soit accessible sur Internet avec une adresse IP publique, soit accessible en interne avec une adresse d’équilibrage de charge interne (ILB) Azure uniquement.

Vous pouvez utiliser des [groupes de sécurité réseau][NetworkSecurityGroups] pour restreindre les communications réseau entrantes vers le sous-réseau sur lequel réside un environnement App Service.  Cela vous permet d'exécuter des applications derrière des appareils et services en amont tels des pare-feu d'applications web, ainsi que des fournisseurs SaaS réseau.

Les applications doivent souvent accéder à des ressources d'entreprise telles que des bases de données internes et des services web.  Une approche courante consiste à rendre ces points de terminaison disponibles uniquement au trafic réseau interne circulant au sein d'un réseau virtuel Azure.  Une fois qu’un environnement App Service est joint au même réseau virtuel que les services internes, les applications s’exécutant dans l’environnement peuvent y accéder, notamment les points de terminaison accessibles via des connexions [site à site][SiteToSite] et [Azure ExpressRoute][ExpressRoute].

Pour plus d’informations sur le fonctionnement des environnements App Service avec les réseaux virtuels et les réseaux locaux, consultez les articles suivants sur l’[architecture réseau][NetworkArchitectureOverview], le [contrôle du trafic entrant][ControllingInboundTraffic] et la [connexion sécurisée aux serveurs principaux][SecurelyConnectingToBackends]. 

## <a name="getting-started"></a>Prise en main

Pour prendre en main les environnements App Service, consultez [Comment créer un environnement App Service][HowToCreateAnAppServiceEnvironment]

Pour obtenir une vue d’ensemble de l’architecture réseau de l’environnement App Service, consultez l’article [Présentation de l’architecture réseau][NetworkArchitectureOverview].

Pour plus d’informations sur l’utilisation d’un environnement App Service avec ExpressRoute, consultez l’article suivant sur [ExpressRoute et environnements App Service][NetworkConfigDetailsForExpressRoute].

[!INCLUDE [app-service-web-try-app-service](../../../includes/app-service-web-try-app-service.md)]

<!-- LINKS -->
[PremiumTier]: https://azure.microsoft.com/pricing/details/app-service/
[MoreInfoOnVirtualNetworks]: ../../virtual-network/virtual-networks-faq.md
[AppServicePlan]: ../overview-hosting-plans.md
[HowToCreateAnAppServiceEnvironment]: app-service-web-how-to-create-an-app-service-environment.md
[LogicApps]: ../../logic-apps/logic-apps-overview.md
[AzureConDeepDive]:  https://azure.microsoft.com/documentation/videos/azurecon-2015-deploying-highly-scalable-and-secure-web-and-mobile-apps/
[GeodistributedAppFootprint]:  app-service-app-service-environment-geo-distributed-scale.md
[NetworkSecurityGroups]: ../../virtual-network/virtual-network-vnet-plan-design-arm.md
[SiteToSite]: ../../vpn-gateway/vpn-gateway-multi-site.md
[ExpressRoute]: https://azure.microsoft.com/services/expressroute/
[HowToConfigureanAppServiceEnvironment]:  app-service-web-configure-an-app-service-environment.md
[ControllingInboundTraffic]:  app-service-app-service-environment-control-inbound-traffic.md
[SecurelyConnectingToBackends]:  app-service-app-service-environment-securely-connecting-to-backend-resources.md
[NetworkArchitectureOverview]:  app-service-app-service-environment-network-architecture-overview.md
[NetworkConfigDetailsForExpressRoute]:  app-service-app-service-environment-network-configuration-expressroute.md
[AppServicePricing]: https://azure.microsoft.com/pricing/details/app-service/ 

<!-- IMAGES -->