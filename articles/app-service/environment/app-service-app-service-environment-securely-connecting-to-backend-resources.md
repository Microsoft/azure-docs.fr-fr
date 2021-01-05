---
title: Se connecter à un back-end v1
description: Découvrez comment connecter de façon sécurisée des ressources de backend à partir d'un environnement App Service. Ce document s’adresse uniquement aux clients qui utilisent l’environnement ASE v1 hérité.
author: stefsch
ms.assetid: f82eb283-a6e7-4923-a00b-4b4ccf7c4b5b
ms.topic: article
ms.date: 10/04/2016
ms.author: stefsch
ms.custom: seodec18
ms.openlocfilehash: 9f8e288f771b9d584a0fd3430115f5fa60f68e47
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88961803"
---
# <a name="connect-securely-to-back-end-resources-from-an-app-service-environment"></a>Se connecter en toute sécurité aux ressources principales à partir d'un environnement App Service
Étant donné qu’un environnement App Service est toujours créé **soit** dans un réseau virtuel Azure Resource Manager **ou** un [réseau virtuel][virtualnetwork] de modèle de déploiement classique, les connexions sortantes d’un environnement App Service à destination d’autres ressources de back-end peuvent passer exclusivement sur le réseau virtuel. Depuis juin 2016, les environnements ASE peuvent également être déployés dans les réseaux virtuels qui utilisent soit des plages d’adresses publiques, soit des espaces d’adressage RFC1918 (adresses privées).  

Par exemple, un serveur SQL Server peut être en cours d'exécution sur un cluster de machines virtuelles dont le port 1433 est verrouillé.  Le point de terminaison peut être placé dans une liste de contrôle d'accès pour autoriser uniquement l'accès à partir d'autres ressources se trouvant sur le même réseau virtuel.  

De même, les points de terminaison sensibles peuvent s’exécuter localement et être connectés à Azure via des connexions de [site à site][SiteToSite] ou [Azure ExpressRoute][ExpressRoute].  Par conséquent, seules les ressources des réseaux virtuels connectés aux tunnels de site à site ou ExpressRoute peuvent accéder aux points de terminaison locaux.

Dans tous ces scénarios, les applications s’exécutant dans un environnement ASE peuvent se connecter de façon sécurisée aux différents serveurs et aux différentes ressources. Si le trafic sortant des applications s’effectue dans un environnement ASE vers des points de terminaison privés se trouvant sur le même réseau virtuel ou connectés au même réseau virtuel, il ne circule que sur le réseau virtuel.  Le trafic sortant vers des points de terminaison privés ne passe pas par l’Internet public.

Le trafic sortant d’un environnement ASE vers des points de terminaison se trouvant sur un réseau virtuel présente un problème. En effet, les environnements ASE ne peuvent pas atteindre les points de terminaison de machines virtuelles situés dans le **même** sous-réseau qu’eux. Cette limitation ne devrait normalement pas poser de problème tant que les environnements ASE sont déployés dans un sous-réseau réservé à leur usage exclusif.

[!INCLUDE [app-service-web-to-api-and-mobile](../../../includes/app-service-web-to-api-and-mobile.md)]

## <a name="outbound-connectivity-and-dns-requirements"></a>Connectivité sortante et configuration DNS requise
Pour qu’un environnement App Service fonctionne correctement, il requiert un accès sortant à différents points de terminaison. Une liste complète des points de terminaison externes utilisés par un ASE est disponible dans la section « Connectivité réseau requise » de l’article [Configuration réseau pour ExpressRoute](app-service-app-service-environment-network-configuration-expressroute.md#required-network-connectivity) .

Les environnements App Service nécessitent une infrastructure DNS valide configurée pour le réseau virtuel.  Si la configuration DNS est modifiée après la création d’un environnement ASE, les développeurs peuvent forcer ce dernier à utiliser la nouvelle configuration DNS. En haut du panneau de gestion de l’environnement ASE du portail, sélectionnez l’icône **Redémarrer** pour déclencher le redémarrage d’un environnement propagé, à la suite de quoi celui-ci récupère la nouvelle configuration DNS.

Il est également recommandé de configurer les serveurs DNS personnalisés sur le réseau virtuel à l’avance, avant de créer un environnement ASE.  Si la configuration DNS d’un réseau virtuel est modifiée pendant la création d’un environnement ASE, ce processus de création échoue. À l’autre extrémité d’une passerelle VPN, si un serveur DNS personnalisé n’est pas accessible ou disponible, le processus de création de l’environnement ASE échoue également.

## <a name="connecting-to-a-sql-server"></a>Connexion à un serveur SQL Server
Une configuration courante de SQL Server comprend un point de terminaison qui écoute sur le port 1433 :

![Point de terminaison de SQL Server][SqlServerEndpoint]

Pour limiter le trafic sur ce point de terminaison, vous avez le choix entre deux approches :

* [Listes de contrôle d’accès réseau][NetworkAccessControlLists] (ACL réseau)
* [Groupes de sécurité réseau][NetworkSecurityGroups]

## <a name="restricting-access-with-a-network-acl"></a>Restriction de l'accès à l'aide d'une ACL réseau
Le port 1433 peut être sécurisé à l'aide d'une liste de contrôle d'accès réseau.  L’exemple ci-dessous ajoute aux autorisations d’affectation les adresses de clients provenant d’un réseau virtuel, et bloque l’accès à tous les autres clients.

![Exemple de liste de contrôle d'accès réseau][NetworkAccessControlListExample]

Toutes les applications qui s’exécutent dans un environnement ASE, sur le même réseau virtuel que SQL Server, peuvent se connecter à l’instance SQL Server. Utilisez l’adresse IP **interne du réseau virtuel** pour la machine virtuelle SQL Server.  

L'exemple de chaîne de connexion ci-dessous fait référence au serveur SQL Server en utilisant son adresse IP privée.

`Server=tcp:10.0.1.6;Database=MyDatabase;User ID=MyUser;Password=PasswordHere;provider=System.Data.SqlClient`

Même si la machine virtuelle possède également un point de terminaison public, les tentatives de connexion visant à utiliser l’adresse IP publique sont rejetées en raison de la liste ACL réseau. 

## <a name="restricting-access-with-a-network-security-group"></a>Restriction de l'accès à l'aide d'un groupe de sécurité réseau
Un groupe de sécurité réseau constitue une autre approche de sécurisation de l'accès.  Les groupes de sécurité réseau peuvent être appliqués à des machines virtuelles individuelles ou à un sous-réseau comprenant des machines virtuelles.

Tout d’abord, vous devez créer un groupe de sécurité réseau :

```azurepowershell-interactive
New-AzureNetworkSecurityGroup -Name "testNSGexample" -Location "South Central US" -Label "Example network security group for an app service environment"
```

Il est facile de restreindre l’accès au seul trafic interne du réseau virtuel à l’aide d’un groupe de sécurité réseau.  Les règles par défaut d'un groupe de sécurité réseau autorisent l'accès uniquement à partir d'autres clients réseau du même réseau virtuel.

Par conséquent, le verrouillage de l’accès à SQL Server est simple. Il suffit d’appliquer un groupe de sécurité réseau avec ses règles par défaut aux machines virtuelles exécutant SQL Server ou au sous-réseau dans lequel elles se trouvent.

L'exemple ci-dessous applique un groupe de sécurité réseau au sous-réseau conteneur :

```azurepowershell-interactive
Get-AzureNetworkSecurityGroup -Name "testNSGExample" | Set-AzureNetworkSecurityGroupToSubnet -VirtualNetworkName 'testVNet' -SubnetName 'Subnet-1'
```

Le résultat final est un ensemble de règles de sécurité qui bloquent l’accès externe, tout en autorisant l’accès interne au réseau virtuel :

![Règles de sécurité réseau par défaut][DefaultNetworkSecurityRules]

## <a name="getting-started"></a>Prise en main
Pour prendre en main les environnements App Service, consultez [Présentation de l'environnement App Service][IntroToAppServiceEnvironment]

Pour plus d’informations sur le contrôle du trafic entrant vers votre environnement App Service, consultez [Contrôle du trafic entrant vers un environnement App Service][ControlInboundASE]

[!INCLUDE [app-service-web-try-app-service](../../../includes/app-service-web-try-app-service.md)]

<!-- LINKS -->
[virtualnetwork]: ../../virtual-network/virtual-networks-faq.md
[ControlInboundTraffic]:  app-service-app-service-environment-control-inbound-traffic.md
[SiteToSite]: ../../vpn-gateway/vpn-gateway-multi-site.md
[ExpressRoute]: https://azure.microsoft.com/services/expressroute/
[NetworkAccessControlLists]: /previous-versions/azure/virtual-network/virtual-networks-acl
[NetworkSecurityGroups]: ../../virtual-network/virtual-network-vnet-plan-design-arm.md
[IntroToAppServiceEnvironment]:  app-service-app-service-environment-intro.md
[ControlInboundASE]:  app-service-app-service-environment-control-inbound-traffic.md

<!-- IMAGES -->
[SqlServerEndpoint]: ./media/app-service-app-service-environment-securely-connecting-to-backend-resources/SqlServerEndpoint01.png
[NetworkAccessControlListExample]: ./media/app-service-app-service-environment-securely-connecting-to-backend-resources/NetworkAcl01.png
[DefaultNetworkSecurityRules]: ./media/app-service-app-service-environment-securely-connecting-to-backend-resources/DefaultNetworkSecurityRules01.png