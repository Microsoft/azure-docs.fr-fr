---
title: Préfixe d’adresses IPv6 publiques dans Réseau virtuel Azure
titlesuffix: Azure Virtual Network
description: Apprenez-en davantage sur le préfixe d’adresses IPv6 publiques dans Réseau virtuel Azure.
services: virtual-network
documentationcenter: na
author: KumudD
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/31/2020
ms.author: kumud
ms.openlocfilehash: ff33bf771065e7af209934a5c54b6f8f2e34835e
ms.sourcegitcommit: 67e8e1caa8427c1d78f6426c70bf8339a8b4e01d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/02/2020
ms.locfileid: "91666781"
---
# <a name="reserved-public-ipv6-address-prefix"></a>Préfixe d’adresses IPv6 publiques réservées

Dans Azure, les machines virtuelles et les réseaux virtuels (VNet) à double pile (IPv4+IPv6) sont sécurisés par défaut, puisqu’ils n’ont pas de connectivité Internet. Vous pouvez facilement ajouter une connectivité Internet IPv6 à vos équilibreurs de charge Azure et à vos machines virtuelles avec des adresses IPv6 publiques que vous obtenez à partir d’Azure.

Toutes les adresses IP publiques que vous réservez sont associées à une région Azure de votre choix et à votre abonnement Azure. Vous pouvez déplacer une adresse IP publique IPv6 réservée (statique) entre les équilibreurs de charge Azure ou les machines virtuelles de votre abonnement. Vous pouvez dissocier entièrement l'adresse IP publique IPv6, et celle-ci sera conservée pour votre usage lorsque vous serez prêt.

> [!WARNING]
> Faites attention à ne pas supprimer accidentellement vos adresses IP publiques. La suppression d’une adresse IP publique entraîne sa suppression de votre abonnement, et vous ne pourrez pas la récupérer (même avec l’aide du support Azure).

En plus de réserver des adresses IPv6 individuelles, vous pouvez réserver des plages contiguës d’adresses IPv6 Azure (appelées préfixes IP) pour votre utilisation.  À l’instar des adresses IP individuelles, les préfixes réservés sont associés à une région Azure de votre choix et à votre abonnement Azure. La réservation d’une plage d’adresses contiguë et prévisible a de nombreuses utilisations. Par exemple, vous pouvez grandement simplifier le *filtrage*, par votre entreprise et vos clients, des adresses IP de vos applications hébergées par Azure, car vos plages d’adresses IP statiques peuvent être facilement programmées dans des pare-feu locaux.  Vous pouvez créer des adresses IP publiques à partir de votre préfixe IP en fonction des besoins et, quand vous supprimez ces adresses IP publiques, elles sont *retournées* à votre plage réservée afin que vous puissiez les réutiliser ultérieurement. Toutes les adresses IP de votre préfixe IP sont réservées à votre usage exclusif jusqu’à ce que vous supprimiez votre préfixe.



## <a name="ipv6-prefix-sizes"></a>Tailles de préfixe IPv6
Les tailles de préfixe IP public suivantes sont disponibles :

-  Taille minimale de préfixe IPv6 :  /127 = 2 adresses
-  Taille maximale de préfixe IPv6 :  /124 = 16 adresses

La taille de préfixe est spécifiée en tant que taille de masque CIDR (Classless Inter-Domain Routing). Par exemple, un masque de /128 représente une adresse IPv6 individuelle, car les adresses IPv6 sont composées de 128 bits.

## <a name="pricing"></a>Tarifs
 
Pour connaître les coûts associés à l’utilisation des adresses IP publiques Azure (adresses IP individuelles et plages d’adresses IP), consultez [Tarification des adresses IP](https://azure.microsoft.com/pricing/details/ip-addresses/).

## <a name="limitations"></a>Limites
IPv6 est pris en charge sur les adresses IP publiques de base uniquement avec l'allocation « dynamique », ce qui signifie que l'adresse IPv6 changera si vous supprimez et redéployez votre application (machines virtuelles ou équilibreurs de charge) dans Azure. Les adresses IP publiques standard IPv6 prennent uniquement en charge l'allocation statique (réservée) tandis que les équilibreurs de charge INTERNES standard peuvent également prendre en charge l'allocation dynamique à partir du sous-réseau auquel ils sont attribués.  

Nous vous recommandons d’utiliser des adresses IP publiques standard et des équilibreurs de charge standard pour vos applications IPv6.

## <a name="next-steps"></a>Étapes suivantes
- Réserver un [préfixe d’adresses IPv6](ipv6-reserve-public-ip-address-prefix.md) publiques.
- En savoir plus sur les [adresses IPv6](ipv6-overview.md).
- En savoir plus sur la façon de [créer et utiliser des adresses IP publiques](virtual-network-public-ip-address.md) (IPv4 et IPv6) dans Azure.
