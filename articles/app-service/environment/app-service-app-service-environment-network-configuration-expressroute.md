---
title: Configurer Azure ExpressRoute v1
description: Configuration réseau pour App Service Environment pour PowerApps avec Azure ExpressRoute. Ce document s’adresse uniquement aux clients qui utilisent l’environnement ASE v1 hérité.
author: stefsch
ms.assetid: 34b49178-2595-4d32-9b41-110c96dde6bf
ms.topic: article
ms.date: 10/14/2016
ms.author: stefsch
ms.custom: seodec18
ms.openlocfilehash: 3fa2677f7c71b118ec3f7da717e29d5879ee982b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88961820"
---
# <a name="network-configuration-details-for-app-service-environment-for-powerapps-with-azure-expressroute"></a>Détails de la configuration réseau pour l’environnement App Service pour PowerApps avec Azure ExpressRoute

Les clients peuvent connecter un circuit [Azure ExpressRoute][ExpressRoute] à leur infrastructure de réseau virtuel pour étendre leur réseau local à Azure. Vous pouvez créer un environnement App Service dans un sous-réseau de cette infrastructure de [réseau virtuel][virtualnetwork]. Les applications exécutées dans l’environnement App Service établissent des connexions sécurisées à des ressources principales accessibles uniquement par le biais de la connexion ExpressRoute.  

L’environnement App Service peut être créé dans ces scénarios :
- Réseaux virtuels Azure Resource Manager.
- Réseaux virtuels avec modèle de déploiement classique.
- Réseaux virtuels qui utilisent des plages d’adresses publiques ou des espaces d’adressage RFC1918 (autrement dit des adresses privées). 

[!INCLUDE [app-service-web-to-api-and-mobile](../../../includes/app-service-web-to-api-and-mobile.md)]

## <a name="required-network-connectivity"></a>Connectivité réseau requise

L’environnement App Service est associé à certaines exigences de connectivité réseau qui peuvent ne pas être initialement satisfaites dans un réseau virtuel connecté à ExpressRoute.

L’environnement App Service requiert les paramètres de connectivité réseau suivants pour fonctionner correctement :

* Connectivité réseau sortante à des points de terminaison de Stockage Azure dans le monde entier sur les ports 80 et 443. Ces points de terminaison se trouvent dans la même région que l’environnement App Service ainsi que d’autres régions Azure. Les points de terminaison du Stockage Azure se résolvent dans les domaines DNS suivants : table.core.windows.net, blob.core.windows.net, queue.core.windows.net et file.core.windows.net.  

* Connectivité réseau sortante au service de fichiers Azure sur le port 445.

* Connectivité réseau sortante à des points de terminaison Azure SQL Database situés dans la même région que l’environnement App Service. Les points de terminaison SQL Database se résolvent dans le domaine database.windows.net, qui nécessite un accès ouvert aux ports 1433, 11000-11999 et 14000-14999. Pour plus d’informations sur l’utilisation des ports de SQL Database V12, voir [Ports au-delà de 1433 pour ADO.NET 4.5](../../azure-sql/database/adonet-v12-develop-direct-route-ports.md).

* Connectivité réseau sortante aux points de terminaison du plan de gestion Azure (modèle de déploiement classique Azure et points de terminaison Azure Resource Manager). La connectivité à ces points de terminaison inclut les domaines management.core.windows.net et management.azure.com. 

* Connectivité réseau sortante aux domaines ocsp.msocsp.com, mscrl.microsoft.com et crl.microsoft.com. Une connectivité à ces domaines est nécessaire pour prendre en charge la fonctionnalité TLS.

* La configuration DNS pour le réseau virtuel doit être capable de résoudre tous les points de terminaison et les domaines mentionnés dans cet article. Si les points de terminaison ne peuvent pas être résolus, la création de l’environnement App Service échoue. Tout environnement App Service existant est marqué comme défectueux.

* L’accès sortant sur le port 53 est nécessaire pour la communication avec des serveurs DNS.

* S’il existe un serveur DNS personnalisé à l’autre extrémité d’une passerelle VPN, il doit être accessible depuis le sous-réseau contenant l’environnement App Service. 

* Le chemin d’accès réseau sortant ne peut pas traverser des proxys d’entreprise internes et il ne peut pas non plus être tunnelisé de force en local. Ces actions modifient l’adresse NAT réelle du trafic réseau sortant de l’environnement App Service. La modification de l’adresse NAT du trafic réseau sortant de l’environnement App Service entraîne des échecs de connectivité vers de nombreux points de terminaison. La création de l’environnement App Service échoue. Tout environnement App Service existant est marqué comme défectueux.

* L’accès réseau entrant vers les ports nécessaires à l’environnement App Service doit être autorisé. Pour plus de détails, voir [Contrôle du trafic entrant vers un environnement App Service][requiredports].

Pour satisfaire la configuration DNS requise, assurez-vous qu’une infrastructure DNS valide est configurée et gérée pour le réseau virtuel. Si la configuration DNS est modifiée après la création d’un environnement App Service, les développeurs peuvent forcer ce dernier à récupérer la nouvelle configuration DNS. Vous pouvez déclencher le redémarrage d’un environnement à l’aide de l’icône **Redémarrer** située sous les options de gestion de l’environnement App Service dans le [Portail Azure][NewPortal]. Suite à ce redémarrage , l’environnement récupère la nouvelle configuration DNS.

Pour satisfaire les exigences en matière d’accès réseau entrant, configurez un [groupe de sécurité réseau (NSG)][NetworkSecurityGroups] sur le sous-réseau App Service Environment. Le groupe de sécurité réseau autorise l’accès requis [pour contrôler le trafic entrant vers l’environnement App Service][requiredports].

## <a name="outbound-network-connectivity"></a>Connectivité réseau sortante

Par défaut, un circuit ExpressRoute nouvellement créé publie un itinéraire par défaut qui autorise la connectivité Internet sortante. L’environnement App Service peut utiliser cette configuration pour se connecter à d’autres points de terminaison Azure.

Une configuration cliente courante consiste à définir un itinéraire par défaut (0.0.0.0/0), ce qui force le trafic Internet sortant à évoluer sur site. Ce flux de trafic décompose invariablement l’environnement App Service. Le trafic sortant est soit bloqué sur site, soit fait l’objet d’une opération NAT sur un jeu d’adresses non reconnaissable qui ne fonctionne plus avec différents points de terminaison Azure.

La solution consiste à définir un (ou plusieurs) itinéraires définis par l’utilisateur (UDR) sur le sous-réseau qui contient l’environnement App Service. Un itinéraire défini par l’utilisateur définit des itinéraires spécifiques au sous-réseau qui sont respectés au lieu de l’itinéraire par défaut.

Si possible, utilisez la configuration suivante :

* La configuration d’ExpressRoute utilise 0.0.0.0/0. Par défaut, elle tunnelise de force tout le trafic sortant local.
* L’itinéraire défini par l’utilisateur appliqué au sous-réseau contenant l’environnement App Service définit 0.0.0.0/0 avec un type de tronçon suivant Internet. Un exemple de cette configuration est décrit plus loin dans cet article.

L’effet combiné de cette configuration est que l’UDR de niveau sous-réseau a la priorité sur le tunneling forcé ExpressRoute. Un accès internet sortant à partir de l’environnement App Service est garanti.

> [!IMPORTANT]
> Les itinéraires définis dans un UDR doivent être suffisamment spécifiques pour avoir la priorité sur les itinéraires annoncés par la configuration ExpressRoute. L’exemple décrit dans la section suivante utilise la plage d’adresses 0.0.0.0/0 étendue. Cette plage peut être accidentellement remplacée par des annonces de routage utilisant des plages d’adresses plus spécifiques.
> 
> L’environnement App Service n’est pas pris en charge avec les configurations ExpressRoute qui annoncent de façon croisée des itinéraires à partir du chemin d’accès de peering public vers le chemin d’accès de peering privé. Les configurations ExpressRoute ayant un peering public configuré reçoivent des annonces de routage en provenance de Microsoft pour un grand ensemble de plages d’adresses IP Microsoft Azure. Si ces plages d’adresses sont publiées de façon croisée sur le chemin d’accès de peering privé, il en résulte que tous les paquets réseau sortants du sous-réseau de l’environnement App Service sont tunnelisés de force vers l’infrastructure réseau local du client. Ce flux de réseau n’est actuellement pas pris en charge avec l’environnement App Service. L’une des solutions consiste à arrêter les itinéraires croisés depuis le chemin d’accès de peering public vers le chemin d’accès de peering privé.
> 
> 

Pour plus d’informations contextuelles sur les itinéraires définis par l’utilisateur, consultez [Routage du trafic de réseau virtuel][UDROverview].  

Pour savoir comment créer et configurer des itinéraires définis par l’utilisateur, voir [Acheminer le trafic réseau avec une table de routage à l’aide de PowerShell][UDRHowTo].

## <a name="udr-configuration"></a>Configuration des itinéraires définis par l’utilisateur

Cette section présente un exemple de configuration d’itinéraire défini par l’utilisateur dans un environnement App Service.

### <a name="prerequisites"></a>Prérequis

* Installez Azure PowerShell à partir de la [page Téléchargements Azure][AzureDownloads]. Choisissez un téléchargement datant au minimum de juin 2015. Sous **Outils en ligne de commande** > **Windows PowerShell**, sélectionnez **Installer** pour installer les dernières applets de commande PowerShell.

* Créez un sous-réseau unique pour une utilisation exclusive par l’environnement App Service. Ce sous-réseau unique garantit que les itinéraires définis par l’utilisateur qui y seront appliqués ouvriront le trafic sortant pour l’environnement App Service uniquement.

> [!IMPORTANT]
> Ne déployez un environnement App Service qu’après avoir terminé les étapes de configuration. Ces étapes permettent de garantir que la connectivité réseau sortante est disponible avant la tentative de déploiement d’un environnement App Service.

### <a name="step-1-create-a-route-table"></a>Étape 1 : Créer une table de routage

Créer une table de routage nommée **DirectInternetRouteTable** dans la région Azure USA Ouest, comme le montre l’extrait de code suivant :

`New-AzureRouteTable -Name 'DirectInternetRouteTable' -Location uswest`

### <a name="step-2-create-routes-in-the-table"></a>Étape 2 : Créer des itinéraires dans la table

Ajoutez des itinéraires à la table de routage pour autoriser l’accès internet sortant.  

Configurez un accès sortant à Internet. Définissez un itinéraire pour 0.0.0.0/0, comme indiqué dans cet extrait de code :

`Get-AzureRouteTable -Name 'DirectInternetRouteTable' | Set-AzureRoute -RouteName 'Direct Internet Range 0' -AddressPrefix 0.0.0.0/0 -NextHopType Internet`

0.0.0.0/0 est une plage d’adresses étendue. Cette plage est remplacée par les plages d’adresses plus spécifiques publiées par ExpressRoute. Un UDR avec un itinéraire 0.0.0.0/0 doit être utilisé conjointement avec une configuration ExpressRoute qui publie uniquement 0.0.0.0/0. 

Vous pouvez également télécharger la liste complète actuelle des plages CIDR utilisées par Azure. Le fichier XML de toutes les plages d’adresses IP Azure est disponible dans le [Centre de téléchargement Microsoft][DownloadCenterAddressRanges].  

> [!NOTE]
>
> Les plages d’adresses IP Azure changent au fil du temps. Les itinéraires définis par l’utilisateur doivent être mis à jour régulièrement pour rester synchronisés.
>
> Un seul UDR a une limite maximale par défaut de 100 itinéraires. Vous devez « ajuster » les plages d’adresses IP Azure pour respecter la limite de 100 itinéraires. Les itinéraires définis par UDR doivent être plus spécifiques que les itinéraires publiés par votre connexion ExpressRoute.
> 

### <a name="step-3-associate-the-table-to-the-subnet"></a>Étape 3 : Associer la table au sous-réseau

Associez la table de routage au sous-réseau sur lequel vous comptez déployer l’environnement App Service. Cette commande associe la table **DirectInternetRouteTable** au sous-réseau **ASESubnet** qui contiendra l’environnement App Service.

`Set-AzureSubnetRouteTable -VirtualNetworkName 'YourVirtualNetworkNameHere' -SubnetName 'ASESubnet' -RouteTableName 'DirectInternetRouteTable'`

### <a name="step-4-test-and-confirm-the-route"></a>Étape 4 : Tester et confirmer l’itinéraire

Une fois la table de routage liée au sous-réseau, testez et confirmez l’itinéraire.

Déployez une machine virtuelle sur le sous-réseau et vérifiez les conditions suivantes :

* Le trafic sortant vers les points de terminaison Azure et non Azure décrits dans cet article n’est **pas** transmis vers le circuit ExpressRoute. Si le trafic sortant du sous-réseau est tunnelisé de force en local, la création de l’environnement App Service échoue systématiquement.
* Les recherches DNS pour les points de terminaison décrits dans cet article se résolvent toutes correctement. 

Après avoir terminé les étapes de configuration et confirmé l’itinéraire, supprimez la machine virtuelle. Le sous-réseau doit être « vide » lors de la création de l’environnement App Service.

Vous êtes maintenant prêt à déployer l’environnement App Service.

## <a name="next-steps"></a>Étapes suivantes

Pour prendre en main les environnements App Service pour PowerApps, consultez [Présentation d’App Service Environment][IntroToAppServiceEnvironment].

<!-- LINKS -->
[virtualnetwork]: https://azure.microsoft.com/services/virtual-network/ 
[ExpressRoute]: https://azure.microsoft.com/services/expressroute/ 
[requiredports]: app-service-app-service-environment-control-inbound-traffic.md 
[NetworkSecurityGroups]: ../../virtual-network/virtual-network-vnet-plan-design-arm.md
[UDROverview]: ../../virtual-network/virtual-networks-udr-overview.md
<!-- Old link -- [UDRHowTo]: https://azure.microsoft.com/documentation/articles/virtual-networks-udr-how-to/ -->

[UDRHowTo]: ../../virtual-network/tutorial-create-route-table-powershell.md
[HowToCreateAnAppServiceEnvironment]: app-service-web-how-to-create-an-app-service-environment.md 
[AzureDownloads]: https://azure.microsoft.com/downloads/ 
[DownloadCenterAddressRanges]: https://www.microsoft.com/download/details.aspx?id=41653 
[NetworkSecurityGroups]: ../../virtual-network/virtual-network-vnet-plan-design-arm.md
[IntroToAppServiceEnvironment]:  app-service-app-service-environment-intro.md 
[NewPortal]:  https://portal.azure.com 


<!-- IMAGES -->