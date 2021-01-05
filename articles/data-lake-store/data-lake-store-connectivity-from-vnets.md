---
title: Se connecter à Azure Data Lake Storage Gen1 à partir de réseaux virtuels | Microsoft Docs
description: Découvrez comment activer l’accès à Azure Data Lake Storage Gen1 à partir de machines virtuelles Azure ayant un accès limité aux ressources.
services: data-lake-store,data-catalog
documentationcenter: ''
author: esung22
manager: mtillman
editor: cgronlun
ms.assetid: 683fcfdc-cf93-46c3-b2d2-5cb79f5e9ea5
ms.service: data-lake-store
ms.devlang: na
ms.topic: how-to
ms.date: 01/31/2018
ms.author: elsung
ms.openlocfilehash: e319cf9dfc01546607e20572c5bf4930fd974c75
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92104034"
---
# <a name="access-azure-data-lake-storage-gen1-from-vms-within-an-azure-vnet"></a>Accéder à Azure Data Lake Storage Gen1 à partir de machines virtuelles au sein d’un réseau virtuel Azure
Azure Data Lake Storage Gen1 est un service PaaS qui s’exécute sur des adresses IP Internet publiques. En général, tout serveur qui peut se connecter à l’Internet public peut également se connecter aux points de terminaison Azure Data Lake Storage Gen1. Par défaut, toutes les machines virtuelles qui se trouvent dans des réseaux virtuels Azure peuvent accéder à Internet et, ainsi, accéder à Azure Data Lake Storage Gen1. Toutefois, il est possible de configurer des machines virtuelles dans un réseau virtuel afin qu’elles n’aient pas accès à Internet. Pour ces machines virtuelles, l’accès à Azure Data Lake Storage Gen1 est également restreint. Bloquer l’accès à l’Internet public pour des machines virtuelles dans des réseaux virtuels Azure peut être effectué à l’aide d’une des approches suivantes :

* En configurant des groupes de sécurité réseau (NSG)
* En configurant des itinéraires définis par l’utilisateur
* En échangeant des itinéraires par le biais de BGP (protocole de routage dynamique standard du secteur) quand ExpressRoute bloque l’accès à Internet

Dans cet article, vous allez apprendre à activer l’accès à Azure Data Lake Storage Gen1 à partir de machines virtuelles Azure soumises à des restrictions d’accès aux ressources, à l’aide d’une des trois méthodes mentionnées plus haut.

## <a name="enabling-connectivity-to-azure-data-lake-storage-gen1-from-vms-with-restricted-connectivity"></a>Activation de la connectivité à Azure Data Lake Storage Gen1 à partir de machines virtuelles bénéficiant d’une connectivité limitée
Pour accéder à Azure Data Lake Storage Gen1 à partir de ces machines virtuelles, vous devez les configurer pour qu’elles accèdent à l’adresse IP correspondant à la région où le compte Azure Data Lake Storage Gen1 est disponible. Vous pouvez identifier les adresses IP des régions de vos comptes Data Lake Storage Gen1 en résolvant les noms DNS de ces derniers (`<account>.azuredatalakestore.net`). Pour résoudre des noms DNS de vos comptes, vous pouvez utiliser des outils tels que **nslookup**. Ouvrez une invite de commandes sur votre ordinateur et exécutez la commande suivante :

```console
nslookup mydatastore.azuredatalakestore.net
```

La sortie se présente comme suit. La valeur en regard de la propriété **Address** est l’adresse IP associée à votre compte Data Lake Storage Gen1.

```output
Non-authoritative answer:
Name:    1434ceb1-3a4b-4bc0-9c69-a0823fd69bba-mydatastore.projectcabostore.net
Address:  104.44.88.112
Aliases:  mydatastore.azuredatalakestore.net
```


### <a name="enabling-connectivity-from-vms-restricted-by-using-nsg"></a>Activation de la connectivité à partir de machines virtuelles restreintes à l’aide d’un groupe de sécurité réseau
Quand une règle de groupe de sécurité réseau est utilisée pour bloquer l’accès à Internet, vous pouvez créer un autre groupe de sécurité réseau qui autorise l’accès à l’adresse IP de Data Lake Storage Gen1. Pour plus d’informations concernant les règles du groupe de sécurité réseau, consultez [Vue d’ensemble des groupes de sécurité réseau](../virtual-network/network-security-groups-overview.md). Pour savoir comment créer des groupes de sécurité réseau, consultez [Créer un groupe de sécurité réseau à l’aide du portail Azure](../virtual-network/tutorial-filter-network-traffic.md).

### <a name="enabling-connectivity-from-vms-restricted-by-using-udr-or-expressroute"></a>Activation de la connectivité à partir de machines virtuelles restreintes à l’aide d’un itinéraire défini par l’utilisateur ou d’ExpressRoute
Quand des itinéraires, définis par l’utilisateur ou échangés par le biais de BGP, sont utilisés pour bloquer l’accès à Internet, un itinéraire spécial doit être configuré afin que les machines virtuelles dans ces sous-réseaux puissent accéder aux points de terminaison Data Lake Storage Gen1. Pour plus d’informations, consultez [Vue d’ensemble des itinéraires définis par l’utilisateur](../virtual-network/virtual-networks-udr-overview.md). Pour obtenir des instructions sur la création d’itinéraires définis par l’utilisateur, consultez [Création d’itinéraires définis par l’utilisateur (UDR) dans Resource Manager](../virtual-network/tutorial-create-route-table-powershell.md).

### <a name="enabling-connectivity-from-vms-restricted-by-using-expressroute"></a>Activation de la connectivité à partir de machines virtuelles restreintes à l’aide d’ExpressRoute
Quand un circuit ExpressRoute est configuré, les serveurs locaux peuvent accéder à Data Lake Storage Gen1 par le biais de le peering public. Pour plus d’informations sur la configuration d’ExpressRoute pour le peering public, consultez [Forum Aux Questions ExpressRoute](../expressroute/expressroute-faqs.md).

## <a name="see-also"></a>Voir aussi
* [Vue d’ensemble d’Azure Data Lake Storage Gen1](data-lake-store-overview.md)
* [Sécurisation des données stockées dans Azure Data Lake Storage Gen1](data-lake-store-security-overview.md)