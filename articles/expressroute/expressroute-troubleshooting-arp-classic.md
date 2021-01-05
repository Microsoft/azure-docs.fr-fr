---
title: 'Azure ExpressRoute : Tables ARP - Dépannage : classique'
description: Cette page fournit des instructions sur l'obtention des tables ARP pour un circuit ExpressRoute - modèle de déploiement classique.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: troubleshooting
ms.date: 01/30/2017
ms.author: duau
ms.openlocfilehash: 7c83b6d25936e184c70d4bf0a51636a3986bf4b2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89394837"
---
# <a name="getting-arp-tables-in-the-classic-deployment-model"></a>Obtention de tables ARP dans le modèle de déploiement classique
> [!div class="op_single_selector"]
> * [PowerShell - Resource Manager](expressroute-troubleshooting-arp-resource-manager.md)
> * [PowerShell - Classique](expressroute-troubleshooting-arp-classic.md)
> 
> 

Cet article vous guide dans la procédure d’obtention des tables ARP (Address Resolution Protocol) pour votre circuit Azure ExpressRoute.

> [!IMPORTANT]
> Ce document a pour objet de vous aider à diagnostiquer et résoudre les problèmes simples. Il n’a pas pour objet de remplacer le support de Microsoft. Si vous ne pouvez pas résoudre le problème en suivant les conseils suivants, ouvrez une demande de support auprès de [Microsoft Azure - Aide + support](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).
> 
> 

## <a name="address-resolution-protocol-arp-and-arp-tables"></a>Protocole ARP (Address Resolution Protocol) et tables ARP
ARP est un protocole de couche 2 défini dans [RFC 826](https://tools.ietf.org/html/rfc826). Le protocole ARP permet de mapper une adresse Ethernet (adresse MAC) avec une adresse IP.

Une table ARP fournit un mappage de l’adresse IPv4 et de l’adresse MAC pour un peering particulier. La table ARP d’un peering de circuit ExpressRoute fournit les informations suivantes pour chaque interface (principale et secondaire) :

1. Mappage de l’adresse IP d’une interface de routeur local sur une adresse MAC
2. Mappage de l’adresse IP d’une interface de routeur ExpressRoute sur une adresse MAC
3. Âge du mappage

Les tables ARP permettent de valider la configuration de la couche 2 et de résoudre les problèmes de connectivité de base de la couche 2.

Vous trouverez ci-dessous un exemple de table ARP :

```output
Age InterfaceProperty IpAddress  MacAddress    
--- ----------------- ---------  ----------    
 10 On-Prem           10.0.0.1   ffff.eeee.dddd
  0 Microsoft         10.0.0.2   aaaa.bbbb.cccc
```


La section suivante fournit des informations sur l’affichage des tables ARP vues par les routeurs périphériques ExpressRoute.

## <a name="prerequisites-for-using-arp-tables"></a>Conditions préalables à l’utilisation des tables ARP
Assurez-vous que vous disposez des éléments suivants avant de poursuivre :

* Un circuit ExpressRoute valide configuré avec au moins un peering. Le circuit doit être entièrement configuré par le fournisseur de connectivité. Vous (ou votre fournisseur de connectivité) devez configurer au moins un des peerings (Azure privé, Azure public ou Microsoft) sur ce circuit.
* Plages d’adresses IP utilisées pour configurer les peerings (Azure privé, Azure public et Microsoft). Passez en revue les exemples d’affectation d’adresses IP dans la [page de configuration requise pour le routage ExpressRoute](expressroute-routing.md) pour comprendre de quelle manière les adresses IP sont mappées aux interfaces de votre côté et du côté d’ExpressRoute. Vous pouvez obtenir plus d’informations sur la configuration du peering en examinant la [page de configuration du peering ExpressRoute](expressroute-howto-routing-classic.md).
* Informations de votre équipe réseau ou de votre fournisseur de connectivité sur les adresses MAC des interfaces utilisées avec ces adresses IP.
* Le dernier module Windows PowerShell pour Azure (version 1.50 ou plus récente).

## <a name="arp-tables-for-your-expressroute-circuit"></a>Tables ARP pour votre circuit ExpressRoute
Cette section fournit des instructions sur l’affichage des tables ARP pour chaque type de peering à l’aide de PowerShell. Avant de continuer, vous ou votre fournisseur de connectivité devez configurer le peering. Chaque circuit a deux chemins d’accès (principal et secondaire). Vous pouvez contrôler indépendamment la table ARP de chaque chemin d’accès.

### <a name="arp-tables-for-azure-private-peering"></a>Tables ARP pour le peering privé Azure
L’applet de commande suivante fournit les tables ARP pour le peering privé Azure :

```azurepowershell
# Required variables
$ckt = "<your Service Key here>

# ARP table for Azure private peering--primary path
Get-AzureDedicatedCircuitPeeringArpInfo -ServiceKey $ckt -AccessType Private -Path Primary

# ARP table for Azure private peering--secondary path
Get-AzureDedicatedCircuitPeeringArpInfo -ServiceKey $ckt -AccessType Private -Path Secondary
```

Vous trouverez ci-dessous un exemple de sortie pour l’un des chemins d’accès :

```output
Age InterfaceProperty IpAddress  MacAddress    
--- ----------------- ---------  ----------    
 10 On-Prem           10.0.0.1   ffff.eeee.dddd
  0 Microsoft         10.0.0.2   aaaa.bbbb.cccc
```


### <a name="arp-tables-for-azure-public-peering"></a>Tables ARP pour le peering public Azure :
L’applet de commande suivante fournit les tables ARP pour le peering public Azure :

```azurepowershell
# Required variables
$ckt = "<your Service Key here>

# ARP table for Azure public peering--primary path
Get-AzureDedicatedCircuitPeeringArpInfo -ServiceKey $ckt -AccessType Public -Path Primary

# ARP table for Azure public peering--secondary path
Get-AzureDedicatedCircuitPeeringArpInfo -ServiceKey $ckt -AccessType Public -Path Secondary
```

Vous trouverez ci-dessous un exemple de sortie pour l’un des chemins d’accès :

```output
Age InterfaceProperty IpAddress  MacAddress    
--- ----------------- ---------  ----------    
 10 On-Prem           10.0.0.1   ffff.eeee.dddd
  0 Microsoft         10.0.0.2   aaaa.bbbb.cccc
```


Vous trouverez ci-dessous un exemple de sortie pour l’un des chemins d’accès :

```output
Age InterfaceProperty IpAddress  MacAddress    
--- ----------------- ---------  ----------    
 10 On-Prem           64.0.0.1   ffff.eeee.dddd
  0 Microsoft         64.0.0.2   aaaa.bbbb.cccc
```


### <a name="arp-tables-for-microsoft-peering"></a>Tables ARP pour le peering Microsoft
L’applet de commande suivante fournit les tables ARP du peering Microsoft :

```azurepowershell
# ARP table for Microsoft peering--primary path
Get-AzureDedicatedCircuitPeeringArpInfo -ServiceKey $ckt -AccessType Microsoft -Path Primary

# ARP table for Microsoft peering--secondary path
Get-AzureDedicatedCircuitPeeringArpInfo -ServiceKey $ckt -AccessType Microsoft -Path Secondary
```


Un exemple de sortie est affiché ci-dessous pour l’un des chemins d’accès :

```output
Age InterfaceProperty IpAddress  MacAddress    
--- ----------------- ---------  ----------    
 10 On-Prem           65.0.0.1   ffff.eeee.dddd
  0 Microsoft         65.0.0.2   aaaa.bbbb.cccc
```


## <a name="how-to-use-this-information"></a>Utilisation de ces informations
La table ARP d’un peering peut servir à valider la connectivité et la configuration de la couche 2. Cette section fournit une vue d’ensemble de l’aspect des tables ARP dans différents scénarios.

### <a name="arp-table-when-a-circuit-is-in-an-operational-expected-state"></a>Table ARP lorsqu’un circuit est à l’état opérationnel (attendu)
* La table ARP dispose d’une entrée pour le côté local avec une adresse IP valide et une adresse MAC, ainsi que d’une entrée similaire pour le côté Microsoft.
* Le dernier octet de l’adresse IP locale est toujours un nombre impair.
* Le dernier octet de l’adresse IP Microsoft est toujours un nombre pair.
* La même adresse MAC s’affiche côté Microsoft pour les trois peerings (principaux/secondaires).

```output
Age InterfaceProperty IpAddress  MacAddress    
--- ----------------- ---------  ----------    
 10 On-Prem           65.0.0.1   ffff.eeee.dddd
  0 Microsoft         65.0.0.2   aaaa.bbbb.cccc
```

### <a name="arp-table-when-its-on-premises-or-when-the-connectivity-provider-side-has-problems"></a>Table ARP en cas de problèmes en local ou côté fournisseur de connectivité
 Une seule entrée apparaît dans la table ARP. Elle affiche le mappage entre l’adresse MAC et l’adresse IP utilisée côté Microsoft.

```output
Age InterfaceProperty IpAddress  MacAddress    
--- ----------------- ---------  ----------    
   0 Microsoft         65.0.0.2  aaaa.bbbb.cccc
```

> [!NOTE]
> Si vous rencontrez un problème de ce type, ouvrez une demande de support auprès de votre fournisseur de connectivité pour le résoudre.
> 
> 

### <a name="arp-table-when-the-microsoft-side-has-problems"></a>Table ARP en cas de problèmes côté Microsoft
* Aucune table ARP ne s’affiche pour un peering en cas de problèmes côté Microsoft.
* Ouvrez une demande de support auprès de [Microsoft Azure - Aide + support](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade). Précisez que vous avez un problème au niveau de la connectivité de couche 2.

## <a name="next-steps"></a>Étapes suivantes
* Valider les configurations de couche 3 pour votre circuit ExpressRoute :
  * Obtenir un récapitulatif d’itinéraires pour déterminer l’état des sessions BGP.
  * Obtenir une table d’itinéraires pour déterminer quels préfixes sont publiés sur ExpressRoute.
* Valider le transfert des données en examinant les octets en entrée et en sortie.
* Ouvrir une demande de support auprès de [Microsoft Azure - Aide + support](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) si vous rencontrez encore des problèmes.

