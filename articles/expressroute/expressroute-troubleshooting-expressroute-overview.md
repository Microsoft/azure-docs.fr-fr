---
title: 'Azure ExpressRoute : Vérifier la connectivité – Guide de résolution des problèmes'
description: Cette page fournit des instructions sur le dépannage et la validation de la connectivité de bout en bout d’un circuit ExpressRoute.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: troubleshooting
ms.date: 10/31/2019
ms.author: duau
ms.custom: seodec18
ms.openlocfilehash: af4ef156cccded6afe2db09628446a6ffe1ad53a
ms.sourcegitcommit: 9826fb9575dcc1d49f16dd8c7794c7b471bd3109
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2020
ms.locfileid: "92204637"
---
# <a name="verifying-expressroute-connectivity"></a>Vérification de la connectivité ExpressRoute
Cet article vous permet de vérifier et de résoudre les problèmes de connectivité ExpressRoute. ExpressRoute étend un réseau local au cloud de Microsoft via une connexion privée qui est généralement facilitée par un fournisseur de connectivité. La connectivité ExpressRoute implique traditionnellement trois zones de réseau distinctes, comme suit :

-   Réseau de client
-   Réseau de fournisseur
-   Centre de données Microsoft

> [!NOTE]
> Dans le modèle de connectivité directe ExpressRoute (proposé à une bande passante de 10/100 Gbits/s), les clients peuvent se connecter directement au port des routeurs Microsoft Enterprise Edge (MSEE). Par conséquent, dans le modèle de connectivité directe, il existe uniquement des zones de réseau client et Microsoft.
>


L’objectif de ce document est d’aider l’utilisateur à déterminer si un problème de connectivité existe et où celui-ci se situe le cas échéant. Et par la même, de faciliter la demande de support auprès de l’équipe appropriée afin de résoudre un problème. Si le support technique de Microsoft est nécessaire pour résoudre un problème, ouvrez un ticket de support auprès du [support Microsoft][Support].

> [!IMPORTANT]
> Ce document a pour but de vous aider à diagnostiquer et à résoudre des problèmes simples. Il n’a pas pour objet de remplacer le support de Microsoft. Si vous ne parvenez pas à résoudre le problème en suivant les conseils donnés, ouvrez un ticket de support auprès du [support Microsoft][Support].
>
>

## <a name="overview"></a>Vue d’ensemble
Le diagramme suivant illustre la connectivité logique d’un réseau de client vers un réseau Microsoft à l’aide d’ExpressRoute.
[![1]][1]

Dans le diagramme précédent, les nombres indiquent les points clés de réseau. Ces points de réseau sont parfois référencés dans cet article par le numéro qui leur est associé. Selon le modèle de connectivité ExpressRoute (colocalisation Cloud Exchange, connexion Ethernet point à point ou universelle [IPVPN]), les points de réseau 3 et 4 peuvent être des commutateurs (appareils de couche 2) ou des routeurs (appareils de couche 3). Dans le modèle de connectivité directe, il n’y a pas de points réseau 3 et 4 ; au lieu de cela, les CE (2) sont directement connectés aux MSEE via une fibre optique noire. Les points clés de réseau illustrés sont les suivants :

1.  Appareil de calcul du client (par exemple, un serveur ou un PC)
2.  CE : routeurs de périphérie du client 
3.  PE (collaborant avec des CE) : routeurs/commutateurs de périphérie du fournisseur qui collaborent avec des routeurs de périphérie du client. Appelé PE-CE dans ce document.
4.  PE (collaborant avec des MSEE) : routeurs/commutateurs de périphérie du fournisseur qui collaborent avec des MSEE. Appelé PE-MSEE dans ce document.
5.  MSEE : routeurs ExpressRoute Microsoft Enterprise Edge (MSEE)
6.  Passerelle de réseau virtuel (VNet)
7.  Appareil de calcul sur le réseau virtuel Azure

Si les modèles de connectivité Colocalisation Cloud Exchange, Ethernet point à point ou de connectivité directe sont utilisés, les CE (2) établissent le Peering BGP avec des MSEE (5). 

Si le modèle de connectivité universelle (IPVPN) est utilisé, les PE-MSEE (4) établissent un Peering BGP avec des MSEE (5). Les PE-MSEE propagent les itinéraires reçus de Microsoft vers le réseau du client par le biais du réseau de fournisseur de services IPVPN.

> [!NOTE]
>Pour une haute disponibilité, Microsoft établit une connectivité parallèle entièrement redondante entre les paires MSEE (5) et PE-MSEE (4). Un chemin d’accès réseau parallèle entièrement redondant est également recommandée entre le réseau du client et la paire PE-CE. Pour plus d’informations sur la haute disponibilité, consultez l’article [Conception pour la haute disponibilité avec ExpressRoute][HA]
>
>

Voici les étapes logiques à suivre pour détecter un problème lié au circuit ExpressRoute :

* [Vérification de l’approvisionnement et de l’état du circuit](#verify-circuit-provisioning-and-state)
  
* [Validation de la configuration du Peering](#validate-peering-configuration)
  
* [Validation d’ARP](#validate-arp)
  
* [Validation de BGP et des itinéraires sur les MSEE](#validate-bgp-and-routes-on-the-msee)
  
* [Confirmation du flux de trafic](#confirm-the-traffic-flow)


## <a name="verify-circuit-provisioning-and-state"></a>Vérification de l’approvisionnement et de l’état du circuit
L’approvisionnement d’un circuit ExpressRoute établit une connexion de couche 2 redondante entre les CE/PE-MSEE (2)/(4) et les MSEE (5). Pour plus d’informations sur la manière de créer, de modifier, de provisionner et de vérifier un circuit ExpressRoute, consultez l’article [Création et modification d’un circuit ExpressRoute][CreateCircuit].

>[!TIP]
>Une clé de service identifie un circuit ExpressRoute de façon unique. Si vous avez besoin d’assistance de la part de Microsoft ou d’un partenaire ExpressRoute pour résoudre un problème lié à ExpressRoute, entrez la clé de service afin d’identifier facilement le circuit.
>
>

### <a name="verification-via-the-azure-portal"></a>Vérification par le biais du portail Azure
Dans le Portail Azure, ouvrez le panneau du circuit ExpressRoute. Dans la section ![3][3] du panneau, les informations essentielles d’ExpressRoute sont indiquées comme illustré dans la capture d’écran suivante :

![4][4]    

Dans ExpressRoute Essentials, *l'état du circuit* indique l’état du circuit côté Microsoft. *L'état du fournisseur* indique si le circuit a été *approvisionné/non approvisionné* côté fournisseur de service. 

Pour qu'un circuit ExpressRoute soit opérationnel, *l'état du Circuit* doit être *activé* et *l'état du fournisseur* doit être *approvisionné*.

> [!NOTE]
> Après la configuration d’un circuit ExpressRoute, si l’*état du circuit* est désactivé, contactez le [Support Microsoft][Support]. En revanche, si l’*état du fournisseur* est non approvisionné, contactez votre fournisseur de services.
>
>

### <a name="verification-via-powershell"></a>Vérification par le biais de PowerShell
Pour répertorier tous les circuits ExpressRoute dans un groupe de ressources, utilisez la commande suivante :

```azurepowershell
Get-AzExpressRouteCircuit -ResourceGroupName "Test-ER-RG"
```

>[!TIP]
>Si vous recherchez le nom d’un groupe de ressources, vous pouvez l’obtenir en répertoriant tous les groupes de ressources dans votre abonnement à l’aide de la commande *Get-AzResourceGroup*.
>


Pour sélectionner un circuit ExpressRoute spécifique dans un groupe de ressources, utilisez la commande suivante :

```azurepowershell
Get-AzExpressRouteCircuit -ResourceGroupName "Test-ER-RG" -Name "Test-ER-Ckt"
```

Voici un exemple de réponse :

```output
Name                             : Test-ER-Ckt
ResourceGroupName                : Test-ER-RG
Location                         : westus2
Id                               : /subscriptions/***************************/resourceGroups/Test-ER-RG/providers/***********/expressRouteCircuits/Test-ER-Ckt
Etag                             : W/"################################"
ProvisioningState                : Succeeded
Sku                              : {
                                    "Name": "Standard_UnlimitedData",
                                    "Tier": "Standard",
                                    "Family": "UnlimitedData"
                                    }
CircuitProvisioningState         : Enabled
ServiceProviderProvisioningState : Provisioned
ServiceProviderNotes             : 
ServiceProviderProperties        : {
                                    "ServiceProviderName": "****",
                                    "PeeringLocation": "******",
                                    "BandwidthInMbps": 100
                                    }
ServiceKey                       : **************************************
Peerings                         : []
Authorizations                   : []
```

Pour vérifier si un circuit ExpressRoute est opérationnel, portez une attention particulière aux champs suivants :

```output
CircuitProvisioningState         : Enabled
ServiceProviderProvisioningState : Provisioned
```

> [!NOTE]
> Après la configuration d’un circuit ExpressRoute, si l’*état du circuit* est désactivé, contactez le [Support Microsoft][Support]. En revanche, si l’*état du fournisseur* est non approvisionné, contactez votre fournisseur de services.
>
>

## <a name="validate-peering-configuration"></a>Validation de la configuration du peering
Une fois que le fournisseur de services a terminé l’approvisionnement du circuit ExpressRoute, plusieurs configuration de routage basées sur eBGP peuvent être créées sur le circuit ExpressRoute entre des CE/MSEE-PR (2)/(4) et des MSEE (5). Chaque circuit ExpressRoute peut avoir : le Peering privé Azure (trafic vers des réseaux virtuels privés dans Azure) et/ou le Peering Microsoft (trafic vers des points de terminaison PaaS et SaaS). Pour plus d’informations sur la création et la modification de la configuration de routage, consultez l’article [Créer et modifier le routage pour un circuit ExpressRoute][CreatePeering].

### <a name="verification-via-the-azure-portal"></a>Vérification par le biais du portail Azure

> [!NOTE]
> Dans le modèle de connectivité IPVPN, les fournisseurs de services gèrent la responsabilité de la configuration des Peerings (services de couche 3). Dans ce modèle, une fois que le fournisseur de services a configuré un Peering et si le Peering est vide dans le portail, essayez d’actualiser la configuration du circuit à l’aide du bouton Actualiser sur le portail. Cette opération extraira la configuration de routage actuelle de votre circuit. 
>

Dans le Portail Azure, l’état d’un Peering de circuit ExpressRoute peut être vérifié sous le panneau du circuit ExpressRoute. Dans la section ![3][3] du panneau, les Peerings d’ExpressRoute sont indiqués comme illustré dans la capture d’écran suivante :

![5][5]

Dans l’exemple précédent, nous avons noté que le Peering privé Azure est approvisionné, tandis que les Peerings public Azure et Microsoft ne sont pas approvisionnés. Un contexte de Peering correctement approvisionné doit également répertorier les sous-réseaux point à point principaux et secondaires. Les sous-réseaux /30 sont utilisés pour l’adresse IP d’interface des MSEE et CE/PE-MSEE. Pour les Peerings approvisionnées, la liste indique également qui a modifié la configuration pour la dernière fois. 

> [!NOTE]
> Si l’activation d’un Peering échoue, vérifiez si les sous-réseaux principaux et secondaires attribués correspondent à la configuration sur le CE/PE-MSEE lié. Vérifiez également si les bons *VlanId*, *AzureASN* et *PeerASN* sont utilisés sur des MSEE et si ces valeurs correspondent à celles utilisées sur le CE/PE-MSEE lié. Si le code de hachage MD5 est choisi, la clé partagée doit être la même sur la paire MSEE et CE/PE-MSEE. La clé partagée précédemment configurée n’est pas affichée pour des raisons de sécurité. Si vous devez modifier l’une de ces configurations sur un routeur MSEE, reportez-vous à la rubrique [Créer et modifier le routage pour un circuit ExpressRoute][CreatePeering].  
>

> [!NOTE]
> Sur un sous-réseau /30 attribué à l’interface, Microsoft choisit la deuxième adresse IP utilisable du sous-réseau pour l’interface MSEE. Par conséquent, assurez-vous que la première adresse IP utilisable du sous-réseau a été attribuée sur le CE/PE-MSEE appairé.
>


### <a name="verification-via-powershell"></a>Vérification par le biais de PowerShell
Pour obtenir les détails sur la configuration du peering privé Azure, utilisez les commandes suivantes :

```azurepowershell
$ckt = Get-AzExpressRouteCircuit -ResourceGroupName "Test-ER-RG" -Name "Test-ER-Ckt"
Get-AzExpressRouteCircuitPeeringConfig -Name "AzurePrivatePeering" -ExpressRouteCircuit $ckt
```

Voici un exemple de réponse pour un peering privé correctement configuré :

```output
Name                       : AzurePrivatePeering
Id                         : /subscriptions/***************************/resourceGroups/Test-ER-RG/providers/***********/expressRouteCircuits/Test-ER-Ckt/peerings/AzurePrivatePeering
Etag                       : W/"################################"
PeeringType                : AzurePrivatePeering
AzureASN                   : 12076
PeerASN                    : 123##
PrimaryPeerAddressPrefix   : 172.16.0.0/30
SecondaryPeerAddressPrefix : 172.16.0.4/30
PrimaryAzurePort           : 
SecondaryAzurePort         : 
SharedKey                  : 
VlanId                     : 200
MicrosoftPeeringConfig     : null
ProvisioningState          : Succeeded
```

 Un contexte de peering correctement activé répertorie les préfixes d’adresses principales et secondaires. Les sous-réseaux /30 sont utilisés pour l’adresse IP d’interface des MSEE et CE/PE-MSEE.

Pour obtenir les détails sur la configuration du peering public Azure, utilisez les commandes suivantes :

```azurepowershell
$ckt = Get-AzExpressRouteCircuit -ResourceGroupName "Test-ER-RG" -Name "Test-ER-Ckt"
Get-AzExpressRouteCircuitPeeringConfig -Name "AzurePublicPeering" -ExpressRouteCircuit $ckt
```

Pour obtenir les détails sur la configuration du peering Microsoft, utilisez les commandes suivantes :

```azurepowershell
$ckt = Get-AzExpressRouteCircuit -ResourceGroupName "Test-ER-RG" -Name "Test-ER-Ckt"
Get-AzExpressRouteCircuitPeeringConfig -Name "MicrosoftPeering" -ExpressRouteCircuit $ckt
```

Si aucun peering n’est configuré, un message d’erreur est généré. Voici un exemple de réponse si les peerings mentionnés (le peering public Azure dans cet exemple) ne sont pas configurés dans le circuit :

```azurepowershell
Get-AzExpressRouteCircuitPeeringConfig : Sequence contains no matching element
At line:1 char:1
    + Get-AzExpressRouteCircuitPeeringConfig -Name "AzurePublicPeering ...
    + ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
        + CategoryInfo          : CloseError: (:) [Get-AzExpr...itPeeringConfig], InvalidOperationException
        + FullyQualifiedErrorId : Microsoft.Azure.Commands.Network.GetAzureExpressRouteCircuitPeeringConfigCommand
```

> [!NOTE]
> Si l’activation d’un Peering échoue, vérifiez si les sous-réseaux principaux et secondaires attribués correspondent à la configuration sur le CE/PE-MSEE lié. Vérifiez également si les bons *VlanId*, *AzureASN* et *PeerASN* sont utilisés sur des MSEE et si ces valeurs correspondent à celles utilisées sur le CE/PE-MSEE lié. Si le code de hachage MD5 est choisi, la clé partagée doit être la même sur la paire MSEE et CE/PE-MSEE. La clé partagée précédemment configurée n’est pas affichée pour des raisons de sécurité. Si vous devez modifier l’une de ces configurations sur un routeur MSEE, reportez-vous à la rubrique [Créer et modifier le routage pour un circuit ExpressRoute][CreatePeering].  
>
>

> [!NOTE]
> Sur un sous-réseau /30 attribué à l’interface, Microsoft choisit la deuxième adresse IP utilisable du sous-réseau pour l’interface MSEE. Par conséquent, assurez-vous que la première adresse IP utilisable du sous-réseau a été attribuée sur le CE/PE-MSEE appairé.
>

## <a name="validate-arp"></a>Validation d’ARP

La table ARP fournit un mappage de l’adresse IP et de l’adresse MAC pour un Peering particulier. La table ARP d’un peering de circuit ExpressRoute fournit les informations suivantes pour chaque interface (principale et secondaire) :
* Mappage de l’adresse IP de l’interface du routeur local sur l’adresse MAC
* Mappage de l’adresse IP de l’interface du routeur ExpressRoute sur l’adresse MAC
* L’ancienneté des tables ARP de mappage permettent de valider la configuration de la couche 2 et de résoudre les problèmes de connectivité de base de la couche 2.


Pour plus d’informations sur l’affichage de la table ARP d’un Peering ExpressRoute et sur l’utilisation des informations pour résoudre les problèmes de connectivité de la couche 2, consultez [Obtention de tables ARP dans le modèle de déploiement Resource Manager][ARP].


## <a name="validate-bgp-and-routes-on-the-msee"></a>Validation de BGP et des itinéraires sur les MSEE

Pour obtenir la table de route à partir du MSEE sur le chemin d’accès *principal* pour le contexte de routage *privé*, utilisez la commande suivante :

```azurepowershell
Get-AzExpressRouteCircuitRouteTable -DevicePath Primary -ExpressRouteCircuitName ******* -PeeringType AzurePrivatePeering -ResourceGroupName ****
```

Voici un exemple de réponse :

```output
Network : 10.1.0.0/16
NextHop : 10.17.17.141
LocPrf  : 
Weight  : 0
Path    : 65515

Network : 10.1.0.0/16
NextHop : 10.17.17.140*
LocPrf  : 
Weight  : 0
Path    : 65515

Network : 10.2.20.0/25
NextHop : 172.16.0.1
LocPrf  : 
Weight  : 0
Path    : 123##
```

> [!NOTE]
> Si l’état d’un Peering eBGP entre un MSEE et un CE/PE-MSEE est Activé ou Inactif, vérifiez si les sous-réseaux de Peering principaux et secondaires attribués correspondent à la configuration sur le CE/PE-MSEE lié. Vérifiez également si les *VlanId*, *AzureAsn* et *PeerAsn* sont utilisés sur des MSEE et si ces valeurs correspondent à celles utilisées sur le CE/PE-MSEE lié. Si le code de hachage MD5 est choisi, la clé partagée doit être la même sur la paire MSEE et CE/PE-MSEE. Si vous devez modifier l’une de ces configurations sur un routeur MSEE, reportez-vous à la rubrique [Créer et modifier le routage pour un circuit ExpressRoute][CreatePeering].
>


> [!NOTE]
> Si certaines destinations ne sont pas accessibles par le biais d’un Peering, vérifiez la table de route des MSEE pour le contexte de Peering correspondant. Si un préfixe correspondant (éventuellement NATed IP) est présent dans la table de route, vérifiez s’il existe des pare-feu/NSG/ACL sur le chemin d’accès qui bloquent le trafic.
>


L’exemple suivant montre la réponse de la commande pour un Peering inexistant :

```azurepowershell
Get-AzExpressRouteCircuitRouteTable : The BGP Peering AzurePublicPeering with Service Key ********************* is not found.
StatusCode: 400
```

## <a name="confirm-the-traffic-flow"></a>Confirmation du flux de trafic
Pour obtenir les statistiques combinées du trafic de chemin d’accès primaire et secondaire (octets en entrée et en sortie) d’un contexte de peering, utilisez la commande suivante :

```azurepowershell
Get-AzExpressRouteCircuitStats -ResourceGroupName $RG -ExpressRouteCircuitName $CircuitName -PeeringType 'AzurePrivatePeering'
```

Voici un exemple de sortie de la commande :

```output
PrimaryBytesIn PrimaryBytesOut SecondaryBytesIn SecondaryBytesOut
-------------- --------------- ---------------- -----------------
     240780020       239863857        240565035         239628474
```

Voici un exemple de sortie de la commande pour un peering inexistant :

```azurepowershell
Get-AzExpressRouteCircuitRouteTable : The BGP Peering AzurePublicPeering with Service Key ********************* is not found.
StatusCode: 400
```

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations ou d'aide, consultez les liens suivants :

- [Support Microsoft][Support]
- [Création et modification d’un circuit ExpressRoute][CreateCircuit]
- [Créer et modifier le routage pour un circuit ExpressRoute][CreatePeering]

<!--Image References-->
[1]: ./media/expressroute-troubleshooting-expressroute-overview/expressroute-logical-diagram.png "Connectivité logique ExpressRoute"
[2]: ./media/expressroute-troubleshooting-expressroute-overview/portal-all-resources.png "Icône Toutes les ressources"
[3]: ./media/expressroute-troubleshooting-expressroute-overview/portal-overview.png "Icône Vue d’ensemble"
[4]: ./media/expressroute-troubleshooting-expressroute-overview/portal-circuit-status.png "Capture d’écran : exemple pour ExpressRoute Essentials"
[5]: ./media/expressroute-troubleshooting-expressroute-overview/portal-private-peering.png "Capture d’écran : exemple pour ExpressRoute Essentials"

<!--Link References-->
[Support]: https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade
[CreateCircuit]: ./expressroute-howto-circuit-portal-resource-manager.md
[CreatePeering]: ./expressroute-howto-routing-portal-resource-manager.md
[ARP]: ./expressroute-troubleshooting-arp-resource-manager.md
[HA]: ./designing-for-high-availability-with-expressroute.md
[DR-Pvt]: ./designing-for-disaster-recovery-with-expressroute-privatepeering.md