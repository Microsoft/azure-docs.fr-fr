---
title: Sécurité en couche v1
description: Découvrez comment implémenter une architecture de sécurité en couche dans votre environnement App Service. Ce document s’adresse uniquement aux clients qui utilisent l’environnement ASE v1 hérité.
author: stefsch
ms.assetid: 73ce0213-bd3e-4876-b1ed-5ecad4ad5601
ms.topic: article
ms.date: 08/30/2016
ms.author: stefsch
ms.custom: seodec18
ms.openlocfilehash: ab26ad08947a4a9929ae0d41b669160b3d159224
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92150236"
---
# <a name="implementing-a-layered-security-architecture-with-app-service-environments"></a>Implémentation d’une architecture de sécurité en couche avec les environnements App Service
Dans la mesure où les environnements App Service fournissent un environnement d’exécution isolé déployé dans un réseau virtuel, les développeurs peuvent créer une architecture de sécurité en couche offrant différents niveaux d’accès réseau pour chaque couche application physique.

Un souhait commun est de masquer les API principales de l’accès Internet général, et d’autoriser uniquement les API à être appelées par les applications web en amont.  Les [groupes de sécurité réseau (NSG)][NetworkSecurityGroups] peuvent être utilisés sur des sous-réseaux contenant des environnements App Service pour restreindre l’accès public aux applications API.

Le schéma ci-dessous présente un exemple d’architecture avec une application WebAPI déployée dans un environnement App Service.  Trois instances d’application web distinctes, déployées sur trois environnements App Service distincts, effectuent des appels principaux à la même application WebAPI.

![Architecture conceptuelle][ConceptualArchitecture] 

Les symboles « plus » verts indiquent que le groupe de sécurité réseau sur le sous-réseau contenant « apiase » autorise les appels entrants des applications web en amont, ainsi que les appels internes.  Toutefois, le même groupe de sécurité réseau refuse explicitement l’accès au trafic entrant général à partir d’Internet. 

Le reste de cet article décrit les étapes nécessaires pour configurer le groupe de sécurité réseau sur le sous-réseau contenant « apiase ».

## <a name="determining-the-network-behavior"></a>Détermination du comportement réseau
Pour savoir quelles règles de sécurité réseau sont nécessaires, vous devez déterminer les clients réseau qui seront autorisés à atteindre l’environnement App Service contenant l’application API et les clients qui seront bloqués.

Étant donné que les [groupes de sécurité réseau (NSG)][NetworkSecurityGroups] sont appliqués aux sous-réseaux et que les environnements App Service sont déployés dans des sous-réseaux, les règles contenues dans un NSG s’appliquent à **toutes** les applications s’exécutant dans un environnement App Service.  À l’aide de l’exemple d’architecture de cet article, une fois qu’un groupe de sécurité réseau est appliqué au sous-réseau contenant « apiase », toutes les applications s’exécutant dans l’environnement App Service « apiase » seront protégées par le même ensemble de règles de sécurité. 

* **Déterminer l’adresse IP sortante des appelants en amont :**  Quelles sont les adresses IP des appelants en amont ?  L’accès de ces adresses devra être explicitement autorisé dans le NSG.  Les appels entre les environnements App Service étant considérés comme des appels « Internet », l’accès de l’adresse IP sortante assignée à chacun des trois environnements App Service en amont doit être autorisé dans le NSG pour le sous-réseau « apiase ».   Pour plus d’informations sur la détermination de l’adresse IP sortante pour les applications s’exécutant dans un environnement App Service, consultez l’article Présentation de l’[architecture réseau][NetworkArchitecture].
* **L’application API principale devra-t-elle s’appeler elle-même ?**  Un point subtil et parfois négligé est le scénario dans lequel l’application principale doit s’appeler elle-même.  Si une application API principale dans un environnement App Service doit s’appeler elle-même, elle est également traitée comme un appel « Internet ».  Dans l’exemple d’architecture, cette opération nécessite également d’autoriser l’accès à partir de l’adresse IP sortante de l’environnement App Service « apiase ».

## <a name="setting-up-the-network-security-group"></a>Configuration du groupe de sécurité réseau
Une fois que l’ensemble d’adresses IP sortantes est connu, l’étape suivante consiste à créer un groupe de sécurité réseau.  Les groupes de sécurité réseau peuvent être créés pour les réseaux virtuels reposant sur Resource Manager, ainsi que les réseaux virtuels classiques.  Les exemples ci-dessous illustrent la création et la configuration d’un groupe de sécurité réseau sur un réseau virtuel classique avec PowerShell.

Pour l’exemple d’architecture, les environnements sont situés dans la région USA Centre Sud, donc un NSG vide est créé dans cette région :

```azurepowershell-interactive
New-AzureNetworkSecurityGroup -Name "RestrictBackendApi" -Location "South Central US" 
-Label "Only allow web frontend and loopback traffic"
```

Tout d’abord, une règle d’autorisation explicite est ajoutée pour l’infrastructure de gestion Azure, comme indiqué dans l’article sur le [trafic entrant][InboundTraffic] pour les environnements App Service.

```azurepowershell-interactive
#Open ports for access by Azure management infrastructure
Get-AzureNetworkSecurityGroup -Name "RestrictBackendApi" | Set-AzureNetworkSecurityRule -Name "ALLOW AzureMngmt" 
-Type Inbound -Priority 100 -Action Allow -SourceAddressPrefix 'INTERNET' -SourcePortRange '*' 
-DestinationAddressPrefix '*' -DestinationPortRange '454-455' -Protocol TCP
```

Ensuite, deux règles sont ajoutées pour autoriser les appels HTTP et HTTPS à partir du premier environnement App Service en amont (« fe1ase »).

```azurepowershell-interactive
#Grant access to requests from the first upstream web front-end
Get-AzureNetworkSecurityGroup -Name "RestrictBackendApi" | Set-AzureNetworkSecurityRule -Name "ALLOW HTTP fe1ase" 
-Type Inbound -Priority 200 -Action Allow -SourceAddressPrefix '65.52.xx.xyz'  -SourcePortRange '*' 
-DestinationAddressPrefix '*' -DestinationPortRange '80' -Protocol TCP
Get-AzureNetworkSecurityGroup -Name "RestrictBackendApi" | Set-AzureNetworkSecurityRule -Name "ALLOW HTTPS fe1ase" 
-Type Inbound -Priority 300 -Action Allow -SourceAddressPrefix '65.52.xx.xyz'  -SourcePortRange '*' 
-DestinationAddressPrefix '*' -DestinationPortRange '443' -Protocol TCP
```

Effacez et recommencez pour les deuxième et troisième environnements App Service en amont (« fe2ase » et « fe3ase »).

```azurepowershell-interactive
#Grant access to requests from the second upstream web front-end
Get-AzureNetworkSecurityGroup -Name "RestrictBackendApi" | Set-AzureNetworkSecurityRule -Name "ALLOW HTTP fe2ase" 
-Type Inbound -Priority 400 -Action Allow -SourceAddressPrefix '191.238.xyz.abc'  -SourcePortRange '*' 
-DestinationAddressPrefix '*' -DestinationPortRange '80' -Protocol TCP
Get-AzureNetworkSecurityGroup -Name "RestrictBackendApi" | Set-AzureNetworkSecurityRule -Name "ALLOW HTTPS fe2ase" 
-Type Inbound -Priority 500 -Action Allow -SourceAddressPrefix '191.238.xyz.abc'  -SourcePortRange '*' 
-DestinationAddressPrefix '*' -DestinationPortRange '443' -Protocol TCP

#Grant access to requests from the third upstream web front-end
Get-AzureNetworkSecurityGroup -Name "RestrictBackendApi" | Set-AzureNetworkSecurityRule -Name "ALLOW HTTP fe3ase" 
-Type Inbound -Priority 600 -Action Allow -SourceAddressPrefix '23.98.abc.xyz'  -SourcePortRange '*' 
-DestinationAddressPrefix '*' -DestinationPortRange '80' -Protocol TCP
Get-AzureNetworkSecurityGroup -Name "RestrictBackendApi" | Set-AzureNetworkSecurityRule -Name "ALLOW HTTPS fe3ase" 
-Type Inbound -Priority 700 -Action Allow -SourceAddressPrefix '23.98.abc.xyz'  -SourcePortRange '*' 
-DestinationAddressPrefix '*' -DestinationPortRange '443' -Protocol TCP
```

Enfin, accordez l’accès à l’adresse IP sortante de l’environnement App Service de l’API principale afin qu’elle puisse s’appeler elle-même.

```azurepowershell-interactive
#Allow apps on the apiase environment to call back into itself
Get-AzureNetworkSecurityGroup -Name "RestrictBackendApi" | Set-AzureNetworkSecurityRule -Name "ALLOW HTTP apiase" 
-Type Inbound -Priority 800 -Action Allow -SourceAddressPrefix '70.37.xyz.abc'  -SourcePortRange '*' 
-DestinationAddressPrefix '*' -DestinationPortRange '80' -Protocol TCP
Get-AzureNetworkSecurityGroup -Name "RestrictBackendApi" | Set-AzureNetworkSecurityRule -Name "ALLOW HTTPS apiase" 
-Type Inbound -Priority 900 -Action Allow -SourceAddressPrefix '70.37.xyz.abc'  -SourcePortRange '*' 
-DestinationAddressPrefix '*' -DestinationPortRange '443' -Protocol TCP
```

Aucune autre règle de sécurité réseau n’est requise, car chaque NSG possède un ensemble de règles par défaut qui bloquent l’accès entrant à partir d’Internet par défaut.

La liste complète des règles du groupe de sécurité réseau est affichée ci-dessous.  Notez la façon dont la dernière règle, mise en surbrillance, bloque l’accès entrant de tous les appelants, à l’exception des appelants auxquels l’accès a été explicitement accordé.

![Configuration de NSG][NSGConfiguration] 

L’étape finale consiste à appliquer le NSG au sous-réseau qui contient l’environnement App Service « apiase ».

```azurepowershell-interactive
#Apply the NSG to the backend API subnet
Get-AzureNetworkSecurityGroup -Name "RestrictBackendApi" | Set-AzureNetworkSecurityGroupToSubnet 
-VirtualNetworkName 'yourvnetnamehere' -SubnetName 'API-ASE-Subnet'
```

Avec le NSG appliqué au sous-réseau, seuls les trois environnements App Service en amont et l’environnement App Service contenant l’API principale sont autorisés à appeler dans l’environnement « apiase ».

## <a name="additional-links-and-information"></a>Informations et liens supplémentaires
Informations sur les [groupes de sécurité réseau](../../virtual-network/network-security-groups-overview.md).

Présentation des [adresses IP sortantes][NetworkArchitecture] et des environnements App Service.

[Ports réseau][InboundTraffic] utilisés par les environnements App Service.

[!INCLUDE [app-service-web-try-app-service](../../../includes/app-service-web-try-app-service.md)]

<!-- LINKS -->
[NetworkSecurityGroups]: ../../virtual-network/virtual-network-vnet-plan-design-arm.md
[NetworkArchitecture]:  app-service-app-service-environment-network-architecture-overview.md
[InboundTraffic]:  app-service-app-service-environment-control-inbound-traffic.md

<!-- IMAGES -->
[ConceptualArchitecture]: ./media/app-service-app-service-environment-layered-security/ConceptualArchitecture-1.png
[NSGConfiguration]:  ./media/app-service-app-service-environment-layered-security/NSGConfiguration-1.png