---
title: Contrôler le trafic entrant v1
description: Découvrez comment contrôler le trafic entrant à destination d’un environnement ASE (App Service Environment). Ce document s’adresse uniquement aux clients qui utilisent l’environnement ASE v1 hérité.
author: ccompy
ms.assetid: 4cc82439-8791-48a4-9485-de6d8e1d1a08
ms.topic: article
ms.date: 01/11/2017
ms.author: stefsch
ms.custom: seodec18
ms.openlocfilehash: fe9326ea9ebd5afe981b7ba6c34b1a5d51e084b0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88962058"
---
# <a name="how-to-control-inbound-traffic-to-an-app-service-environment"></a>Contrôle du trafic entrant vers un environnement App Service
## <a name="overview"></a>Vue d’ensemble
Un environnement App Service peut être créé **soit** dans un réseau virtuel Azure Resource Manager, **soit** dans un [réseau virtuel][virtualnetwork] de modèle de déploiement classique.  Il est possible de définir un nouveau réseau virtuel et un nouveau sous-réseau au moment de la création d'un environnement App Service. Vous pouvez autrement créer un environnement ASE dans un réseau virtuel préexistant et un sous-réseau préexistant.  Depuis juin 2016, les environnements ASE peuvent également être déployés dans les réseaux virtuels qui utilisent soit des plages d’adresses publiques, soit des espaces d’adressage RFC1918 (adresses privées).  Pour plus d’informations, consultez [Comment créer un environnement ASE (App Service Environment)][HowToCreateAnAppServiceEnvironment].

Créez toujours un environnement App Service dans un sous-réseau. Un sous-réseau procure une limite réseau qui peut être utilisée pour bloquer le trafic entrant derrière les appareils et services en amont. Avec cette configuration, seules des adresses IP en amont spécifiques sont autorisées à accepter le trafic HTTP et HTTPS.

Le trafic réseau entrant et sortant sur un sous-réseau est contrôlé à l'aide d’un [groupe de sécurité réseau][NetworkSecurityGroups]. Pour contrôler le trafic entrant, créez des règles de sécurité réseau dans un groupe de sécurité réseau. Affectez ensuite le groupe de sécurité réseau au sous-réseau où se trouve App Service Environment.

Une fois que vous avez affecté un groupe de sécurité réseau à un sous-réseau, le trafic entrant à destination des applications dans App Service Environment est autorisé ou bloqué en fonction des règles d’autorisation et de refus qui sont définies dans le groupe de sécurité réseau.

[!INCLUDE [app-service-web-to-api-and-mobile](../../../includes/app-service-web-to-api-and-mobile.md)]

## <a name="inbound-network-ports-used-in-an-app-service-environment"></a>Ports réseau entrants dans un environnement App Service
Avant de bloquer le trafic réseau entrant au moyen d’un groupe de sécurité réseau, renseignez-vous sur les ports réseau obligatoires et facultatifs utilisés par un environnement ASE.  La fermeture accidentelle du trafic vers certains ports peut entraîner une perte de fonctionnalités dans un environnement App Service.

La liste suivante indique les ports utilisés par un environnement ASE. Tous les ports sont **TCP**, sauf indication contraire clairement spécifiée :

* 454 :  **Port obligatoire** utilisé par l’infrastructure Azure pour la gestion et la maintenance des environnements App Service par le biais de TLS.  Ne bloquez pas le trafic vers ce port.  Ce port est toujours lié à l’adresse IP virtuelle publique d’un ASE.
* 455 :  **Port obligatoire** utilisé par l’infrastructure Azure pour la gestion et la maintenance des environnements App Service par le biais de TLS.  Ne bloquez pas le trafic vers ce port.  Ce port est toujours lié à l’adresse IP virtuelle publique d’un ASE.
* 80 :  Port par défaut pour le trafic HTTP entrant vers des applications s’exécutant dans plans App Service d’un environnement App Service.  Sur un ASE avec équilibrage de charge interne, ce port est lié à l’adresse d’équilibrage de charge interne de l’ASE.
* 443 : port par défaut pour le trafic TLS entrant vers des applications qui s’exécutent dans des plans App Service d’un environnement ASE.  Sur un ASE avec équilibrage de charge interne, ce port est lié à l’adresse d’équilibrage de charge interne de l’ASE.
* 21 :  Canal de contrôle pour FTP.  Ce port peut être bloqué en toute sécurité si FTP n’est pas utilisé.  Sur un ASE avec équilibrage de charge interne, ce port peut être lié à l’adresse d’équilibrage de charge interne d’un ASE.
* 990 :  Canal de contrôle pour FTPS.  Ce port peut être bloqué en toute sécurité si FTPS n’est pas utilisé.  Sur un ASE avec équilibrage de charge interne, ce port peut être lié à l’adresse d’équilibrage de charge interne d’un ASE.
* 10001-10020 : Canaux de données pour FTP.  Comme avec le canal de contrôle, ces ports peuvent être bloqués en toute sécurité si FTP n’est pas utilisé.  Sur un ASE avec équilibrage de charge interne, ce port peut être lié à l’adresse d’équilibrage de charge interne de l’ASE.
* 4016 : Utilisé pour le débogage à distance avec Visual Studio 2012.  Ce port peut être bloqué en toute sécurité si la fonctionnalité n’est pas utilisée.  Sur un ASE avec équilibrage de charge interne, ce port est lié à l’adresse d’équilibrage de charge interne de l’ASE.
* 4018 : Utilisé pour le débogage à distance avec Visual Studio 2013.  Ce port peut être bloqué en toute sécurité si la fonctionnalité n’est pas utilisée.  Sur un ASE avec équilibrage de charge interne, ce port est lié à l’adresse d’équilibrage de charge interne de l’ASE.
* 4020 : Utilisé pour le débogage à distance avec Visual Studio 2015.  Ce port peut être bloqué en toute sécurité si la fonctionnalité n’est pas utilisée.  Sur un ASE avec équilibrage de charge interne, ce port est lié à l’adresse d’équilibrage de charge interne de l’ASE.

## <a name="outbound-connectivity-and-dns-requirements"></a>Connectivité sortante et configuration DNS requise
Pour qu’un environnement App Service fonctionne correctement, il requiert également un accès sortant à différents points de terminaison. Une liste complète des points de terminaison externes utilisés par un ASE est disponible dans la section « Connectivité réseau requise » de l’article [Configuration réseau pour ExpressRoute](app-service-app-service-environment-network-configuration-expressroute.md#required-network-connectivity) .

Les environnements App Service nécessitent une infrastructure DNS valide configurée pour le réseau virtuel.  Si la configuration DNS est modifiée après la création d’un environnement ASE, les développeurs peuvent forcer cet environnement à utiliser la nouvelle configuration DNS.  Si vous déclenchez un redémarrage progressif de l’environnement à l’aide de l’icône **Redémarrer**, l’environnement utilise la nouvelle configuration DNS. (L’icône **Redémarrer** se trouve en haut du panneau de gestion d’App Service Environment, dans le [portail Azure][NewPortal].)

Il est également recommandé de configurer les serveurs DNS personnalisés sur le réseau virtuel à l’avance, avant de créer un environnement ASE.  Si la configuration DNS d’un réseau virtuel est modifiée durant la création d’un environnement ASE, le processus de création de cet environnement échoue.  De la même manière, si un serveur DNS personnalisé n’est pas accessible ou disponible à l’autre extrémité d’une passerelle VPN, le processus de création de l’environnement ASE échoue aussi.

## <a name="creating-a-network-security-group"></a>Création d'un groupe de sécurité réseau
Pour plus d'informations sur le fonctionnement des groupes de sécurité réseau, consultez les [informations][NetworkSecurityGroups] suivantes.  L’exemple Azure Service Management ci-dessous aborde certains points sur les groupes de sécurité réseau. L’exemple configure un groupe de sécurité réseau et l’applique à un sous-réseau qui comporte un environnement ASE.

**Remarque :** Les groupes de sécurité réseau peuvent être configurés sous forme graphique à l’aide du [portail Azure](https://portal.azure.com) ou d’Azure PowerShell.

Les groupes de sécurité réseau sont tout d’abord créés en tant qu’entités autonomes associées à un abonnement. Dans la mesure où les groupes de sécurité réseau sont créés dans une région Azure, créez le groupe de sécurité réseau dans la même région que l'environnement ASE.

La commande suivante illustre la création d’un groupe de sécurité réseau :

```azurepowershell-interactive
New-AzureNetworkSecurityGroup -Name "testNSGexample" -Location "South Central US" -Label "Example network security group for an app service environment"
```

Une fois un groupe de sécurité réseau créé, une ou plusieurs règles de sécurité réseau lui sont ajoutées.  Étant donné que l’ensemble de règles peut changer au fil du temps, vous devez espacer le schéma de numérotation utilisé pour les priorités de règle. Cette pratique facilite l’insertion ultérieure de règles supplémentaires.

Dans l’exemple ci-dessous, une règle accorde explicitement l’accès aux ports de gestion utilisés par l’infrastructure Azure pour gérer et tenir à jour un environnement ASE.  Tout le trafic de gestion transite par TLS et est sécurisé par des certificats clients. Même si les ports sont ouverts, ils sont inaccessibles à toutes les entités autres que l’infrastructure de gestion Azure.

```azurepowershell-interactive
Get-AzureNetworkSecurityGroup -Name "testNSGexample" | Set-AzureNetworkSecurityRule -Name "ALLOW AzureMngmt" -Type Inbound -Priority 100 -Action Allow -SourceAddressPrefix 'INTERNET'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '454-455' -Protocol TCP
```

Quand vous bloquez l’accès aux ports 80 et 443 pour « masquer » un environnement ASE derrière des appareils ou services en amont, souvenez-vous de l’adresse IP en amont.  Par exemple, si vous utilisez un pare-feu d’applications web (WAF), WAF aura sa ou ses propres adresses IP. Le WAF les utilise pour router le trafic via un proxy vers un environnement ASE en aval.  Vous devez spécifier cette adresse IP dans le paramètre *SourceAddressPrefix* d’une règle de sécurité réseau.

Dans l'exemple ci-dessous, le trafic entrant à partir d'une adresse IP en amont spécifique est explicitement autorisé.  L'adresse *1.2.3.4* est utilisée comme un espace réservé pour l'adresse IP d'un pare-feu d'applications web (WAF) en amont.  Modifiez la valeur pour la faire correspondre à l'adresse utilisée par votre service ou appareil en amont.

```azurepowershell-interactive
Get-AzureNetworkSecurityGroup -Name "testNSGexample" | Set-AzureNetworkSecurityRule -Name "RESTRICT HTTP" -Type Inbound -Priority 200 -Action Allow -SourceAddressPrefix '1.2.3.4/32'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '80' -Protocol TCP
Get-AzureNetworkSecurityGroup -Name "testNSGexample" | Set-AzureNetworkSecurityRule -Name "RESTRICT HTTPS" -Type Inbound -Priority 300 -Action Allow -SourceAddressPrefix '1.2.3.4/32'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '443' -Protocol TCP
```

Si la prise en charge FTP est souhaitée, utilisez les règles suivantes comme modèle pour accorder l’accès au port de contrôle FTP et aux ports des canaux de donnés.  Étant donné que FTP est un protocole avec état, il se peut que vous ne puissiez pas router le trafic FTP via un pare-feu HTTP/HTTPS ou un appareil proxy traditionnel.  Dans ce cas, vous devez définir le paramètre *SourceAddressPrefix* sur une autre valeur, telle que la plage d’adresses IP de machines de développement ou de déploiement sur lesquelles s’exécutent les clients FTP. 

```azurepowershell-interactive
Get-AzureNetworkSecurityGroup -Name "testNSGexample" | Set-AzureNetworkSecurityRule -Name "RESTRICT FTPCtrl" -Type Inbound -Priority 400 -Action Allow -SourceAddressPrefix '1.2.3.4/32'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '21' -Protocol TCP
Get-AzureNetworkSecurityGroup -Name "testNSGexample" | Set-AzureNetworkSecurityRule -Name "RESTRICT FTPDataRange" -Type Inbound -Priority 500 -Action Allow -SourceAddressPrefix '1.2.3.4/32'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '10001-10020' -Protocol TCP
```

(**Remarque :** la plage de ports du canal de données peut changer pendant la durée de la version préliminaire.)

Si le débogage à distance avec Visual Studio est utilisé, les règles suivantes montrent comment accorder l'accès.  Il existe une règle distincte pour chaque version de Visual Studio prise en charge, car chaque version utilise un port différent pour le débogage à distance.  Comme avec l'accès FTP, il est possible que le trafic du débogage à distance ne transite pas correctement via un pare-feu d'applications web (WAF) ou un appareil proxy traditionnel.  À la place, le paramètre *SourceAddressPrefix* peut être défini sur la plage d'adresses IP des machines de développement exécutant Visual Studio.

```azurepowershell-interactive
Get-AzureNetworkSecurityGroup -Name "testNSGexample" | Set-AzureNetworkSecurityRule -Name "RESTRICT RemoteDebuggingVS2012" -Type Inbound -Priority 600 -Action Allow -SourceAddressPrefix '1.2.3.4/32'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '4016' -Protocol TCP
Get-AzureNetworkSecurityGroup -Name "testNSGexample" | Set-AzureNetworkSecurityRule -Name "RESTRICT RemoteDebuggingVS2013" -Type Inbound -Priority 700 -Action Allow -SourceAddressPrefix '1.2.3.4/32'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '4018' -Protocol TCP
Get-AzureNetworkSecurityGroup -Name "testNSGexample" | Set-AzureNetworkSecurityRule -Name "RESTRICT RemoteDebuggingVS2015" -Type Inbound -Priority 800 -Action Allow -SourceAddressPrefix '1.2.3.4/32'  -SourcePortRange '*' -DestinationAddressPrefix '*' -DestinationPortRange '4020' -Protocol TCP
```

## <a name="assigning-a-network-security-group-to-a-subnet"></a>Affectation d'un groupe de sécurité réseau à un sous-réseau
Un groupe de sécurité réseau a une règle de sécurité par défaut qui refuse l’accès à tout le trafic externe. Quand vous combinez cette règle avec les règles de sécurité réseau ci-dessus, seul le trafic provenant de plages d’adresses sources qui sont associées à une action *Autoriser* pourra être envoyé vers des applications qui s’exécutent dans un environnement ASE.

Une fois qu’un groupe de sécurité réseau est rempli avec des règles de sécurité, affectez-le au sous-réseau contenant l’environnement ASE.  La commande d’affectation référence deux noms : le nom du réseau virtuel où se trouve l’environnement ASE et le nom du sous-réseau où l’environnement ASE a été créé.  

L'exemple ci-dessous présente un groupe de sécurité réseau attribué à un sous-réseau et un réseau virtuel :

```azurepowershell-interactive
Get-AzureNetworkSecurityGroup -Name "testNSGexample" | Set-AzureNetworkSecurityGroupToSubnet -VirtualNetworkName 'testVNet' -SubnetName 'Subnet-test'
```

L’affectation est une opération longue qui peut prendre plusieurs minutes. Une fois l’affectation du groupe de sécurité réseau réussie, seul le trafic entrant qui répond aux règles *Autoriser* pourra atteindre les applications dans l’environnement ASE.

Par souci d’exhaustivité, l’exemple suivant montre comment supprimer et dissocier le groupe de sécurité réseau du sous-réseau :

```azurepowershell-interactive
Get-AzureNetworkSecurityGroup -Name "testNSGexample" | Remove-AzureNetworkSecurityGroupFromSubnet -VirtualNetworkName 'testVNet' -SubnetName 'Subnet-test'
```

## <a name="special-considerations-for-explicit-ip-ssl"></a>Considérations spécifiques concernant les adresses IP SSL explicites
Si une application est configurée avec une adresse IP SSL explicite (applicable *uniquement* aux environnements ASE qui ont une adresse IP virtuelle publique), au lieu de l’adresse IP par défaut de l’environnement ASE, le trafic HTTP et HTTPS transite dans le sous-réseau via d’autres ports que les ports 80 et 443.

Pour trouver la paire de ports utilisée par chaque adresse IP SSL, accédez au portail et affichez le panneau UX des détails de l’environnement ASE.  Sélectionnez **Tous les paramètres** > **Adresses IP**.  Le panneau **Adresses IP** présente un tableau de toutes les adresses IP SSL qui sont configurées explicitement pour l’environnement ASE. Le panneau affiche également la paire de ports spéciale qui est utilisée pour router le trafic HTTP et HTTPS associé à chaque adresse IP SSL.  Utilisez cette paire de ports pour les paramètres DestinationPortRange quand vous configurez des règles dans un groupe de sécurité réseau.

Quand une application sur un environnement ASE est configurée pour utiliser une adresse IP SSL, les clients externes ne voient pas et n’ont pas à se préoccuper du mappage de la paire de ports spéciale.  Le trafic vers les applications se dirigera normalement vers l’adresse IP-SSL configurée.  La traduction automatique de la paire de ports spéciale se produit en interne pendant la dernière phase du routage du trafic dans le sous-réseau où se trouve l’environnement ASE. 

## <a name="getting-started"></a>Prise en main
Pour bien démarrer avec les environnements ASE, consultez [Présentation d’App Service Environment][IntroToAppServiceEnvironment].

Pour plus d’informations, consultez [Connexion sécurisée à des ressources de back-end à partir d'un environnement ASE][SecurelyConnecttoBackend].

[!INCLUDE [app-service-web-try-app-service](../../../includes/app-service-web-try-app-service.md)]

<!-- LINKS -->
[virtualnetwork]: ../../virtual-network/virtual-networks-faq.md
[HowToCreateAnAppServiceEnvironment]: app-service-web-how-to-create-an-app-service-environment.md
[NetworkSecurityGroups]: ../../virtual-network/virtual-network-vnet-plan-design-arm.md
[IntroToAppServiceEnvironment]:  app-service-app-service-environment-intro.md
[SecurelyConnecttoBackend]:  app-service-app-service-environment-securely-connecting-to-backend-resources.md
[NewPortal]:  https://portal.azure.com  

<!-- IMAGES -->