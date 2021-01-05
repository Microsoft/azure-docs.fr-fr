---
title: Scénarios pour Private Zones – Azure DNS
description: Dans cet article, vous découvrez des scénarios courants pour l’utilisation d’Azure DNS Private Zones.
services: dns
author: rohinkoul
ms.service: dns
ms.topic: article
ms.date: 10/05/2019
ms.author: rohink
ms.openlocfilehash: ab850adb2e9a25778d5f44ba711eb0762fe562c8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "76939338"
---
# <a name="azure-dns-private-zones-scenarios"></a>Scénarios Azure DNS Private Zones

Azure DNS Private Zones fournit la résolution de noms au sein d’un réseau virtuel, ainsi qu’entre des réseaux virtuels. Dans cet article, nous étudions certains des scénarios courants pouvant être mis en œuvre à l’aide de cette fonctionnalité.

## <a name="scenario-name-resolution-scoped-to-a-single-virtual-network"></a>Scénario : Résolution de noms pour un seul réseau virtuel
Dans ce scénario, vous disposez d’un réseau virtuel dans Azure qui possède un certain nombre de ressources Azure, y compris des machines virtuelles. Vous souhaitez résoudre les ressources à partir du réseau virtuel via un nom de domaine spécifique (zone DNS), et vous avez besoin que la résolution de noms soit privée et non accessible depuis Internet. En outre, pour les machines virtuelles dans le réseau virtuel, vous avez besoin d’Azure pour les enregistrer automatiquement dans la zone DNS. 

Ce scénario est illustré ci-dessous. Le réseau virtuel nommé « A » contient deux machines virtuelles (VNETA-VM1 et VNETA-VM2). Chacune d’entre elles a des adresses IP privées associées. Une fois que vous créez une zone privée nommée contoso.com et que vous associez ce réseau virtuel en tant que réseau virtuel d’inscription, Azure DNS crée automatiquement deux enregistrements A dans la zone, comme illustré. Désormais, les requêtes DNS de VNETA-VM1 pour résoudre VNETA-VM2.contoso.com reçoivent une réponse DNS qui contient l’adresse IP privée de VNETA-VM2. En outre, une requête DNS inverse (PTR) pour l’adresse IP privée de VNETA-VM1 (10.0.0.1) émise depuis VNETA-VM2 reçoit une réponse DNS qui contient le nom de VNETA-VM1, comme prévu. 

![Résolution pour un seul réseau virtuel](./media/private-dns-scenarios/single-vnet-resolution.png)

## <a name="scenario-name-resolution-across-virtual-networks"></a>Scénario : Résolution de noms sur plusieurs réseaux virtuels

Ce scénario est le cas le plus courant où vous devez associer une zone privée à plusieurs réseaux virtuels. Ce scénario peut accepter des architectures comme le modèle « Hub and Spoke » où il existe un réseau virtuel Hub central auquel plusieurs réseaux virtuels Spoke sont connectés. Le réseau virtuel Hub central peut être associé en tant que réseau virtuel d’inscription à une zone privée, et les réseaux virtuels Spoke peuvent être associés en tant que réseaux virtuels de résolution. 

Le schéma suivant présente une version simple de ce scénario, où il existe seulement deux réseaux virtuels : A et B. A est désigné en tant que réseau virtuel d’inscription et B en tant que réseau virtuel de résolution. L’objectif est que les deux réseaux virtuels partagent une même zone contoso.com. Lorsque la zone est créée et quand les réseaux virtuels d’inscription et de résolution sont associés à la zone, Azure enregistre automatiquement les enregistrements DNS des machines virtuelles (VNETA-VM1 et VNETA-VM2) à partir du réseau virtuel A. Vous pouvez également ajouter manuellement des enregistrements DNS dans la zone pour les machines virtuelles dans le réseau virtuel de résolution B. Avec cette configuration, vous pourrez observer le comportement suivant pour les requêtes DNS directes et inverses :
* Une requête DNS de VNETB-VM1 dans le réseau virtuel de résolution B, pour VNETA-VM1.contoso.com, reçoit une réponse DNS contenant l’adresse IP privée de VNETA-VM1.
* Une requête DNS inverse (PTR) de VNETB-VM2 dans le réseau virtuel de résolution B, pour 10.1.0.1, reçoit une réponse DNS contenant le nom de domaine complet VNETB-VM1.contoso.com.  
* Une requête DNS inverse (PTR) de VNETB-VM3 dans le réseau virtuel de résolution B, pour 10.0.0.1, reçoit NXDOMAIN. Cela s’explique par le fait que l’étendue des requêtes DNS inverses est uniquement le même réseau virtuel. 


![Résolution pour plusieurs réseaux virtuels](./media/private-dns-scenarios/multi-vnet-resolution.png)

## <a name="scenario-split-horizon-functionality"></a>Scénario : Fonctionnalité de découpage d’horizon

Dans ce scénario, vous avez un cas d’usage où vous souhaitez qu’un comportement de résolution DNS différent s’applique en fonction de l’endroit où se trouve le client (dans Azure ou sur Internet) pour la même zone DNS. Par exemple, vous avez peut-être une version privée et publique de votre application qui a des fonctionnalités ou un comportement différents, mais vous souhaitez utiliser le même nom de domaine pour les deux versions. Ce scénario peut être mis en œuvre avec Azure DNS en créant une zone DNS publique, ainsi qu’une zone privée, portant le même nom.

Le schéma suivant illustre ce scénario. Vous avez un réseau virtuel A qui a deux machines virtuelles (VNETA-VM1 et VNETA-VM2) pour lesquelles des adresses IP privées et publiques sont allouées. Vous créez une zone DNS publique nommée contoso.com et enregistrez les adresses IP publiques pour ces machines virtuelles en tant qu’enregistrements DNS dans la zone. En outre, vous créez une zone DNS privée également nommée contoso.com en spécifiant A en tant que réseau virtuel d’inscription. Azure enregistre automatiquement les machines virtuelles en tant qu’enregistrements A dans la zone privée, en pointant vers leurs adresses IP privées.

Maintenant, lorsqu’un client Internet émet une requête DNS pour rechercher VNETA-VM1.contoso.com, Azure renvoie l’enregistrement d’adresse IP publique de la zone publique. Si la même requête DNS est émise d’une autre machine virtuelle (par exemple, VNETA-VM2) dans le même réseau virtuel A, Azure renvoie l’enregistrement d’adresse IP privée de la zone privée. 

![Découpage - résolution Brian](./media/private-dns-scenarios/split-brain-resolution.png)

## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur les zones DNS privées, consultez la session relative à [l’utilisation d’Azure DNS pour les domaines privés](private-dns-overview.md).

Découvrez comment [créer une zone DNS privée](./private-dns-getstarted-powershell.md) dans Azure DNS.

Obteniez plus d’informations sur les zones et enregistrements DNS en consultant : [Vue d’ensemble des enregistrements et zones DNS](dns-zones-records.md).

Découvrez certaines des autres [fonctionnalités de réseau](../networking/networking-overview.md) clés d’Azure.

