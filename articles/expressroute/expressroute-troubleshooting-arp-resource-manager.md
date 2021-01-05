---
title: 'Azure ExpressRoute : Tables ARP - Dépannage'
description: Cette page fournit des instructions sur l’obtention des tables ARP (Address Resolution Protocol) pour un circuit ExpressRoute.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: troubleshooting
ms.date: 12/15/2020
ms.author: duau
ms.custom: seodec18
ms.openlocfilehash: 7d8ae2c58979c66ebbbab366d172179bdeee4253
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97561577"
---
# <a name="getting-arp-tables-in-the-resource-manager-deployment-model"></a>Obtention de tables ARP dans le modèle de déploiement Resource Manager
> [!div class="op_single_selector"]
> * [PowerShell - Resource Manager](expressroute-troubleshooting-arp-resource-manager.md)
> * [PowerShell - Classique](expressroute-troubleshooting-arp-classic.md)
> 
> 

Cet article vous guide tout au long des étapes d’apprentissage des tables ARP pour votre circuit ExpressRoute.

> [!IMPORTANT]
> Ce document a pour objet de vous aider à diagnostiquer et résoudre les problèmes simples. Il n’a pas pour objet de remplacer le support de Microsoft. Si vous ne parvenez pas à résoudre le problème en suivant les conseils ci-dessous, ouvrez un ticket de support auprès du [support Microsoft](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) .
> 
> 

[!INCLUDE [updated-for-az](../../includes/hybrid-az-ps.md)]

## <a name="address-resolution-protocol-arp-and-arp-tables"></a>Protocole ARP (Address Resolution Protocol) et tables ARP
Le protocole ARP (Address Resolution Protocol) est un protocole de couche 2 défini dans [RFC 826](https://tools.ietf.org/html/rfc826). ARP est utilisé pour mapper l’adresse Ethernet (adresse MAC) avec une adresse IP.

La table ARP fournit les informations suivantes pour les interfaces principale et secondaire pour chaque type de Peering :

1. Mappage de l’adresse IP de l’interface du routeur local sur l’adresse MAC
2. Mappage de l’adresse IP de l’interface du routeur ExpressRoute sur l’adresse MAC
3. Âge du mappage

Les tables ARP permettent de valider la configuration de la couche 2 et de résoudre les problèmes de connectivité de base de la couche 2. 

Exemple de table ARP : 

```output
Age InterfaceProperty IpAddress  MacAddress    
--- ----------------- ---------  ----------    
 10 On-Prem           10.0.0.1   ffff.eeee.dddd
  0 Microsoft         10.0.0.2   aaaa.bbbb.cccc
```


La section suivante fournit des informations sur l’affichage des tables ARP vues par les routeurs de bordure ExpressRoute. 

## <a name="prerequisites-for-learning-arp-tables"></a>Conditions préalables à l’apprentissage des tables ARP
Assurez-vous que les informations ci-dessous sont exactes avant de continuer :

* Un circuit ExpressRoute valide configuré avec au moins un Peering. Le circuit doit être entièrement configuré par le fournisseur de connectivité. Vous ou votre fournisseur de connectivité devez avoir configuré au moins un Peering privé Azure, un Peering public Azure ou un Peering Microsoft sur ce circuit.
* Des plages d’adresses IP utilisées pour configurer les Peerings. Passez en revue les exemples d’affectation d’adresses IP dans la [page de configuration requise pour le routage ExpressRoute](expressroute-routing.md) pour comprendre de quelle manière les adresses IP sont mappées aux interfaces. Vous pouvez obtenir plus d’informations sur la configuration du peering en examinant la [page de configuration du peering ExpressRoute](expressroute-howto-routing-arm.md).
* Informations de votre équipe réseau/fournisseur de connectivité sur les adresses MAC des interfaces utilisées avec ces adresses IP.
* Vous devez disposer du dernier module PowerShell pour Azure (version 1.50 ou plus récente).

> [!NOTE]
> Si la couche 3 est fournie par le fournisseur de services et que les tables ARP sont vides dans le portail/la sortie ci-dessous, actualisez la configuration du circuit à l’aide du bouton Actualiser du portail. Cette opération appliquera la configuration de routage appropriée à votre circuit. 
>
>

## <a name="getting-the-arp-tables-for-your-expressroute-circuit"></a>Obtention des tables ARP pour votre circuit ExpressRoute
Cette section fournit des instructions sur la manière d’afficher les tables ARP par peering à l’aide de PowerShell. Vous ou votre fournisseur de connectivité devez avoir configuré le peering avant de poursuivre. Chaque circuit a deux chemins d’accès (principal et secondaire). Vous pouvez contrôler indépendamment la table ARP de chaque chemin d’accès.

### <a name="arp-tables-for-azure-private-peering"></a>Tables ARP pour le peering privé Azure
L’applet de commande suivante fournit les tables ARP pour le peering privé Azure

```azurepowershell
# Required Variables
$RG = "<Your Resource Group Name Here>"
$Name = "<Your ExpressRoute Circuit Name Here>"

# ARP table for Azure private peering - Primary path
Get-AzExpressRouteCircuitARPTable -ResourceGroupName $RG -ExpressRouteCircuitName $Name -PeeringType AzurePrivatePeering -DevicePath Primary

# ARP table for Azure private peering - Secondary path
Get-AzExpressRouteCircuitARPTable -ResourceGroupName $RG -ExpressRouteCircuitName $Name -PeeringType AzurePrivatePeering -DevicePath Secondary 
```

Un exemple de sortie est affiché ci-dessous pour l’un des chemins d’accès

```output
Age InterfaceProperty IpAddress  MacAddress    
--- ----------------- ---------  ----------    
 10 On-Prem           10.0.0.1   ffff.eeee.dddd
  0 Microsoft         10.0.0.2   aaaa.bbbb.cccc
```


### <a name="arp-tables-for-azure-public-peering"></a>Tables ARP pour le peering public Azure
L’applet de commande suivante fournit les tables ARP pour le peering public Azure

```azurepowershell
# Required Variables
$RG = "<Your Resource Group Name Here>"
$Name = "<Your ExpressRoute Circuit Name Here>"

# ARP table for Azure public peering - Primary path
Get-AzExpressRouteCircuitARPTable -ResourceGroupName $RG -ExpressRouteCircuitName $Name -PeeringType AzurePublicPeering -DevicePath Primary

# ARP table for Azure public peering - Secondary path
Get-AzExpressRouteCircuitARPTable -ResourceGroupName $RG -ExpressRouteCircuitName $Name -PeeringType AzurePublicPeering -DevicePath Secondary 
```


Un exemple de sortie est affiché ci-dessous pour l’un des chemins d’accès

```output
Age InterfaceProperty IpAddress  MacAddress    
--- ----------------- ---------  ----------    
 10 On-Prem           64.0.0.1   ffff.eeee.dddd
  0 Microsoft         64.0.0.2   aaaa.bbbb.cccc
```


### <a name="arp-tables-for-microsoft-peering"></a>Tables ARP pour le peering Microsoft
L’applet de commande suivante fournit les tables ARP du peering Microsoft

```azurepowershell
# Required Variables
$RG = "<Your Resource Group Name Here>"
$Name = "<Your ExpressRoute Circuit Name Here>"

# ARP table for Microsoft peering - Primary path
Get-AzExpressRouteCircuitARPTable -ResourceGroupName $RG -ExpressRouteCircuitName $Name -PeeringType MicrosoftPeering -DevicePath Primary

# ARP table for Microsoft peering - Secondary path
Get-AzExpressRouteCircuitARPTable -ResourceGroupName $RG -ExpressRouteCircuitName $Name -PeeringType MicrosoftPeering -DevicePath Secondary 
```


Un exemple de sortie est affiché ci-dessous pour l’un des chemins d’accès

```output
Age InterfaceProperty IpAddress  MacAddress    
--- ----------------- ---------  ----------    
 10 On-Prem           65.0.0.1   ffff.eeee.dddd
  0 Microsoft         65.0.0.2   aaaa.bbbb.cccc
```


## <a name="how-to-use-this-information"></a>Utilisation de ces informations
La table ARP d’un peering peut servir à valider la connectivité et la configuration de la couche 2. Cette section fournit une vue d’ensemble de l’aspect des tables ARP dans différents scénarios.

### <a name="arp-table-when-a-circuit-is-in-operational-state-expected-state"></a>Table ARP lorsqu’un circuit est dans un état opérationnel (état attendu)
* La table ARP aura une entrée pour le côté local avec une adresse IP valide et une adresse MAC. Il en est de même pour le côté Microsoft. 
* Le dernier octet de l’adresse IP locale sera toujours un nombre impair.
* Le dernier octet de l’adresse IP Microsoft sera toujours un nombre pair.
* La même adresse MAC s’affichera côté Microsoft pour les trois Peerings (principal/secondaires). 

```output
Age InterfaceProperty IpAddress  MacAddress    
--- ----------------- ---------  ----------    
 10 On-Prem           65.0.0.1   ffff.eeee.dddd
  0 Microsoft         65.0.0.2   aaaa.bbbb.cccc
```

### <a name="arp-table-when-on-premises--connectivity-provider-side-has-problems"></a>Table ARP en cas de problèmes côté fournisseur de connectivité/local
Si un problème survient avec le fournisseur local ou le fournisseur de connectivité, la table ARP affiche l’un des deux éléments suivants : Vous constaterez que l’adresse MAC locale est incomplète ou que seule l’entrée Microsoft figure dans la table ARP.
  
```output
Age InterfaceProperty IpAddress  MacAddress    
--- ----------------- ---------  ----------   
  0 On-Prem           65.0.0.1   Incomplete
  0 Microsoft         65.0.0.2   aaaa.bbbb.cccc
```
or
   
```output
Age InterfaceProperty IpAddress  MacAddress    
--- ----------------- ---------  ----------    
  0 Microsoft         65.0.0.2   aaaa.bbbb.cccc
```  

> [!NOTE]
> Ouvrez une demande de support avec votre fournisseur de connectivité pour déboguer ces problèmes. Si la table ARP ne comprend pas les adresses IP des interfaces mappées sur des adresses MAC, passez en revue les informations suivantes :
> 
> 1. Si la première adresse IP du sous-réseau /30 affecté pour la liaison entre les MSEE-PR et MSEE est utilisée dans l’interface de MSEE-PR. Azure utilise toujours la deuxième adresse IP pour les MSEE.
> 2. Vérifiez si les balises VLAN du client (C-Tag) et du service (S-Tag) correspondent à la paire MSEE-PR et MSEE.
> 

### <a name="arp-table-when-microsoft-side-has-problems"></a>Table ARP en cas de problèmes côté Microsoft
* Aucune table ARP ne s’affiche pour un Peering en cas de problèmes côté Microsoft. 
* Ouvrez un incident auprès du [support technique Microsoft](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade). Spécifiez que vous avez un problème au niveau de la connectivité de couche 2. 

## <a name="next-steps"></a>Étapes suivantes
* Valider les configurations de couche 3 pour votre circuit ExpressRoute.
  * Obtenir un récapitulatif d’itinéraires pour déterminer l’état des sessions BGP.
  * Obtenir une table de routage pour déterminer quels préfixes sont publiés sur ExpressRoute.
* Valider le transfert de données en examinant les octets entrants/sortants.
* Ouvrir un ticket de support auprès de [Support Microsoft](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) si vous rencontrez encore des problèmes.

