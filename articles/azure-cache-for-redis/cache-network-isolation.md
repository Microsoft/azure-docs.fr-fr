---
title: Options d’isolement réseau d’Azure Cache pour Redis
description: Dans cet article, vous allez apprendre à déterminer la meilleure solution d’isolement réseau pour vos besoins. Nous allons passer en revue les concepts de base d’Azure Private Link, l’injection Azure Virtual Network et des règles de pare-feu Azure avec leurs avantages et leurs limites.
author: curib
ms.author: cauribeg
ms.service: cache
ms.topic: conceptual
ms.date: 10/15/2020
ms.openlocfilehash: ef284661d44f700cf0b5282efcd2e6f7b94fa3b6
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96621516"
---
# <a name="azure-cache-for-redis-network-isolation-options"></a>Options d’isolement réseau d’Azure Cache pour Redis 
Dans cet article, vous allez apprendre à déterminer la meilleure solution d’isolement réseau pour vos besoins. Nous allons passer en revue les concepts de base d’Azure Private Link, l’injection Azure Virtual Network et des règles de pare-feu Azure avec leurs avantages et leurs limites.  

## <a name="azure-private-link-public-preview"></a>Azure Private Link (préversion publique) 
Azure Private Link fournit une connectivité privée entre un réseau virtuel et les services Azure PaaS. Il simplifie l’architecture réseau et sécurise la connexion entre les points de terminaison dans Azure en éliminant l’exposition des données à l’internet public. 

### <a name="advantages"></a>Avantages
* Prise en charge des instances Azure Cache pour Redis De base, Standard et Premium. 
* En utilisant [Azure Private Link](../private-link/private-link-overview.md), vous pouvez vous connecter à une instance Azure Cache depuis votre réseau virtuel via un point de terminaison privé, auquel est attribuée une adresse IP privée dans un sous-réseau du réseau virtuel. Ainsi, les instances de cache sont disponibles à partir du réseau virtuel et publiquement.  
* Une fois qu’un point de terminaison privé est créé, l’accès au réseau public peut être limité à l’aide de l’indicateur `publicNetworkAccess`. Cet indicateur est défini sur `Disabled` par défaut, ce qui autorise uniquement l’accès aux liaisons privées. Vous pouvez définir la valeur sur `Enabled` ou `Disabled` avec une requête PATCH. Pour plus d’informations, consultez [Azure Cache pour Redis avec Azure Private Link (préversion)](cache-private-link.md). 
* Toutes les dépendances de cache externes seront sans effet sur les règles NSG du réseau virtuel.

### <a name="limitations"></a>Limites 
* Les groupes de sécurité réseau (NSG) sont désactivés pour les points de terminaison privés. Toutefois, s’il existe d’autres ressources sur le sous-réseau, l’application du NSG s’appliquera à ces ressources.
* La géoréplication, les règles de pare-feu, la prise en charge de la console du portail, la multiplicité des points de terminaison par cache en cluster, la persistance au pare-feu et les caches injectés sur le réseau virtuel ne sont pas pris en charge pour le moment. 
* Pour vous connecter à un cache en cluster, `publicNetworkAccess` doit être défini sur `Disabled`, et il ne peut y avoir qu’une seule connexion de point de terminaison privé.

> [!NOTE]
> Lors de l’ajout d’un point de terminaison privé à une instance de cache, tout le trafic de Redis est acheminé vers le point de terminaison privé en raison du DNS.
> Vérifiez que les règles de pare-feu précédentes sont ajustées avant.  
>
>

## <a name="azure-virtual-network-injection"></a>Injection sur le réseau virtuel Azure 
Le réseau virtuel est l’élément de base de votre réseau privé dans Azure. Il permet à de nombreuses ressources Azure de communiquer entre elles, avec Internet et avec les réseaux locaux en toute sécurité. Le réseau virtuel est comme un réseau traditionnel que vous utiliseriez dans votre propre centre de données, mais avec les avantages de l’infrastructure, de l’échelle, de la disponibilité et de l’isolement Azure. 

### <a name="advantages"></a>Avantages
* Lorsqu’une instance Azure Cache pour Redis est configurée avec un réseau virtuel, elle n’est pas adressable publiquement et est accessible uniquement à partir de machines virtuelles et d’applications sur le réseau virtuel.  
* Lorsque le réseau virtuel est combiné à des stratégies NSG limitées, il contribue à réduire le risque d’exfiltration de données. 
* Le déploiement de réseau virtuel fournit un isolement et une sécurité renforcés pour votre instance Azure Cache pour Redis, ainsi que des sous-réseaux, des stratégies de contrôle d’accès et d’autres fonctionnalités permettant de restreindre davantage l’accès. 
* La géoréplication est prise en charge. 

### <a name="limitations"></a>Limites
* Les caches injectés sur le réseau virtuel sont uniquement disponibles pour Azure Cache pour Redis Premium. 
* Lorsque vous utilisez un cache injecté sur le réseau virtuel, vous devez ouvrir votre réseau virtuel pour mettre en cache des dépendances telles que des listes de révocation de certificats, une infrastructure à clé publique (PKI), Azure Key Vault, Stockage Azure, Azure Monitor, et plus encore.  


## <a name="azure-firewall-rules"></a>Règles de Pare-feu Azure
[Pare-feu Azure](../firewall/overview.md) est un service de sécurité réseau informatique managé qui protège vos ressources de réseau virtuel Azure. Il s’agit d’un service de pare-feu avec état intégral, doté d’une haute disponibilité intégrée et d’une scalabilité illimitée dans le cloud. Vous pouvez créer, appliquer et consigner des stratégies de connectivité réseau et d’application de façon centralisée entre les abonnements et les réseaux virtuels.  

### <a name="advantages"></a>Avantages
* Lorsque des règles de pare-feu sont configurées, seules les connexions client à partir des plages d’adresses IP spécifiées peuvent se connecter au cache. Les connexions depuis les systèmes de surveillance de cache Azure pour Redis sont toujours autorisées, même si des règles de pare-feu sont configurées. Les règles NSG que vous définissez sont également autorisées.  

### <a name="limitations"></a>Limites
* Les règles de pare-feu peuvent être utilisées conjointement avec les caches injectés sur le réseau virtuel, mais pas avec les points de terminaison privés pour le moment. 


## <a name="next-steps"></a>Étapes suivantes
* Découvrez comment configurer un [cache injecté sur un réseau virtuel pour une instance Azure Cache pour Redis Premium](cache-how-to-premium-vnet.md).  
* Découvrez comment configurer des [règles de pare-feu pour tous les niveaux d’Azure Cache pour Redis](cache-configure.md#firewall). 
* Découvrez comment [configurer des points de terminaison privés pour tous les niveaux d’Azure Cache pour Redis](cache-private-link.md).