---
title: Introduction
description: Découvrez comment Azure App Service Environment vous permet de mettre à l’échelle, sécuriser et optimiser vos applications dans un environnement totalement isolé et dédié.
author: ccompy
ms.assetid: 3c7eaefa-1850-4643-8540-428e8982b7cb
ms.topic: overview
ms.date: 04/19/2018
ms.author: ccompy
ms.custom: mvc, seodec18
ms.openlocfilehash: 6fff19498e9ca70991d3190165df70a48136f502
ms.sourcegitcommit: fb3c846de147cc2e3515cd8219d8c84790e3a442
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92631136"
---
# <a name="introduction-to-the-app-service-environments"></a>Présentation des environnements App Service #
## <a name="overview"></a>Vue d’ensemble ##

L’environnement Azure App Service est une fonctionnalité d’Azure App Service qui fournit un environnement totalement isolé et dédié pour l’exécution sécurisée de vos applications App Service à grande échelle. Cette fonctionnalité peut héberger vos :

* Applications web Windows
* Applications web Linux 
* Conteneurs Docker
* Applications mobiles
* Fonctions

Les environnements App Service (ASE) conviennent aux charges de travail d’application qui nécessitent :

* Une très grande échelle.
* Une isolation et un accès réseau sécurisé.
* Une utilisation élevée de la mémoire.

Les clients peuvent créer plusieurs environnements App Service au sein d’une même région Azure ou dans plusieurs régions Azure. Grâce à cette souplesse, les environnements ASE sont parfaits pour la mise à l’échelle horizontale des niveaux d’application sans état pour la prise en charge des charges de travail présentant un niveau élevé de requêtes par seconde.

Les environnements ASE hébergent des applications d’un seul client, et le font sur l’un de leurs réseaux virtuels. Les clients peuvent contrôler précisément le trafic réseau entrant et sortant des applications. Les applications peuvent établir des connexions sécurisées à haute vitesse aux ressources d’entreprise locales par le biais de réseaux privés virtuels.

* Le service ASE est fourni avec son propre niveau tarifaire. Découvrez comment les [offres isolées](https://channel9.msdn.com/Shows/Azure-Friday/Security-and-Horsepower-with-App-Service-The-New-Isolated-Offering?term=app%20service%20environment) favorisent les déploiements hyperscale et la sécurité.
* Les [environnements App Service v2](https://channel9.msdn.com/Blogs/Azure/Azure-Application-Service-Environments-v2-Private-PaaS-Environments-in-the-Cloud?term=app%20service%20environment) fournissent un cadre qui protège vos applications dans un sous-réseau de votre réseau, et assurent votre propre déploiement privé d’Azure App Service.
* Vous pouvez utiliser plusieurs environnements App Service pour une mise à l’échelle horizontale. Pour plus d’informations, consultez [Mise à l’échelle géolocalisée avec les environnements App Service](app-service-app-service-environment-geo-distributed-scale.md).
* Vous pouvez utiliser des environnements App Service pour configurer l’architecture de sécurité, comme indiqué dans l’Immersion AzureCon. Pour découvrir comment a été configurée l’architecture de sécurité illustrée dans l’Immersion AzureCon, consultez l’[article sur l’implémentation d’une architecture de sécurité en couches](app-service-app-service-environment-layered-security.md) avec les environnements App Service.
* L’accès aux applications qui s’exécutent sur des environnements App Service peut être contrôlé par des appareils en amont tels que les pare-feu d’applications web (WAF). Pour plus d’informations, consultez l’article [Pare-feu d’applications web (WAF)][AppGW].
* Les environnements ASE (App Service Environment) peuvent être déployés dans des zones de disponibilité (AZ) à l’aide de l’épinglage de zone.  Pour plus d’informations, consultez [Prise en charge d’App Service Environment pour les zones de disponibilité][ASEAZ].

## <a name="dedicated-environment"></a>Environnement dédié ##

Un environnement App Service est dédié exclusivement à un abonnement et peut héberger 100 instances de plan App Service. Il peut s’agir de 100 instances d’un même plan App Service ou de 100 plans App Service à instance unique, ou de toutes les combinaisons possibles des deux.

Un environnement App Service est composé de frontends et de workers. Les frontends sont responsables de la terminaison du protocole HTTP/HTTPS et de l’équilibrage de charge automatique des demandes d’application dans un environnement App Service. Ils sont ajoutés automatiquement à mesure que les plans App Service dans l’environnement App Service augmentent leur échelle.

Les workers sont des rôles qui hébergent des applications clientes. Ils sont disponibles dans trois tailles fixes :

* Un processeur virtuel/3,5 Go de RAM
* Deux processeurs virtuels/7 Go de RAM
* Quatre processeurs virtuels/14 Go de RAM

Les clients n’ont pas besoin de gérer les frontends et les workers. Toute infrastructure est automatiquement ajoutée à mesure que les clients effectuent un scale-out de leurs plans App Service. À mesure que des plans App Service sont créés ou mis à l’échelle dans un environnement App Service, l’infrastructure requise est ajoutée ou supprimée en conséquence.

Un tarif fixe mensuel couvre l’infrastructure d’un environnement App Service et ne change pas avec la taille de l’environnement. Chaque processeur virtuel de plan App Service présente aussi un coût. Toutes les applications hébergées dans un environnement App Service sont dans la référence (SKU) de prix Isolée. Pour plus d’informations sur les tarifs d’un environnement App Service, consultez la page de [tarification App Service][Pricing] et passez en revue les options disponibles pour les environnements App Service.

## <a name="virtual-network-support"></a>Prise en charge des réseaux virtuels ##

La fonctionnalité ASE est un déploiement d’Azure App Service effectué directement dans le réseau virtuel Azure Resource Manager d’un client. Pour en savoir plus sur les réseaux virtuels Azure, consultez le [FAQ sur les réseaux virtuels Azure](../../virtual-network/virtual-networks-faq.md). Un environnement App Service existe toujours dans un réseau virtuel, et plus précisément un sous-réseau d’un réseau virtuel. Vous pouvez utiliser les fonctionnalités de sécurité des réseaux virtuels pour contrôler les communications réseau entrantes et sortantes de vos applications.

Un environnement App Service peut être soit accessible sur Internet avec une adresse IP publique, soit accessible en interne avec uniquement une adresse d’équilibreur de charge interne (ILB) Azure.

Les [groupes de sécurité réseau][NSGs] limitent les communications réseau entrantes vers le sous-réseau sur lequel réside l’environnement App Service. Vous pouvez utiliser des groupes de sécurité réseau pour exécuter des applications derrière des appareils et services en amont tels que des pare-feu d’applications web et des fournisseurs SaaS réseau.

Les applications doivent souvent accéder à des ressources d'entreprise telles que des bases de données internes et des services web. Si vous déployez l’environnement App Service sur un réseau virtuel qui a une connexion VPN au réseau local, les applications de l’environnement App Service peuvent accéder aux ressources locales. Cela est vrai, que le réseau privé virtuel soit un VPN [de site à site](../../vpn-gateway/vpn-gateway-multi-site.md) ou un VPN [Azure ExpressRoute](https://azure.microsoft.com/services/expressroute/).

Pour plus d’informations sur le fonctionnement des environnements App Service avec des réseaux virtuels et des réseaux locaux, consultez [Considérations sur les réseaux d’environnement App Service][ASENetwork].

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/Azure-Application-Service-Environments-v2-Private-PaaS-Environments-in-the-Cloud/player]

## <a name="app-service-environment-v1"></a>Environnement App Service v1 ##

App Service Environment est disponible en deux versions : ASEv1 et ASEv2. Les informations précédentes sont basées sur ASEv2. Cette section montre les différences entre ASEv1 et ASEv2. 

Dans ASEv1, vous devez gérer toutes les ressources manuellement. Celles-ci incluent les frontends, les workers et les adresses IP utilisées pour le protocole SSL basé sur IP. Pour pouvoir effectuer un scale-out de votre plan App Service, vous devez d’abord effectuer un scale-out du pool de workers où vous voulez héberger le plan.

Les versions ASEv1 et ASEv2 utilisent un modèle tarifaire différent. Dans ASEv1, vous payez pour chaque processeur virtuel alloué. Cela inclut les processeurs virtuels utilisés pour les frontends ou les workers qui n’hébergent pas de charges de travail. Dans la version ASEv1, la taille d’échelle maximale par défaut d’un environnement App Service correspond à un total de 55 hôtes, dont les workers et les frontends. L’un des avantages d’un ASEv1 est qu’il peut être déployé sur un réseau virtuel classique et sur un réseau virtuel Resource Manager. Pour plus d’informations sur ASEv1, consultez [Présentation de l’environnement App Service v1][ASEv1Intro].

<!--Links-->
[App Service Environments v2]: https://channel9.msdn.com/Blogs/Azure/Azure-Application-Service-Environments-v2-Private-PaaS-Environments-in-the-Cloud?term=app%20service%20environment
[Isolated offering]: https://channel9.msdn.com/Shows/Azure-Friday/Security-and-Horsepower-with-App-Service-The-New-Isolated-Offering?term=app%20service%20environment
[Intro]: ./intro.md
[MakeExternalASE]: ./create-external-ase.md
[MakeASEfromTemplate]: ./create-from-template.md
[MakeILBASE]: ./create-ilb-ase.md
[ASENetwork]: ./network-info.md
[UsingASE]: ./using-an-ase.md
[UDRs]: ../../virtual-network/virtual-networks-udr-overview.md
[NSGs]: ../../virtual-network/network-security-groups-overview.md
[ConfigureASEv1]: app-service-web-configure-an-app-service-environment.md
[ASEv1Intro]: app-service-app-service-environment-intro.md
[webapps]: ../overview.md
[mobileapps]: /previous-versions/azure/app-service-mobile/app-service-mobile-value-prop
[Functions]: ../../azure-functions/index.yml
[Pricing]: https://azure.microsoft.com/pricing/details/app-service/
[ARMOverview]: ../../azure-resource-manager/management/overview.md
[ConfigureSSL]: ../configure-ssl-certificate.md
[Kudu]: https://azure.microsoft.com/resources/videos/super-secret-kudu-debug-console-for-azure-web-sites/
[ASEWAF]: app-service-app-service-environment-web-application-firewall.md
[AppGW]: ../../web-application-firewall/ag/ag-overview.md
[ASEAZ]: https://azure.github.io/AppService/2019/12/12/App-Service-Environment-Support-for-Availability-Zones.html