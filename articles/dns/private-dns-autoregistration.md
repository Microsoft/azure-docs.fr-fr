---
title: Qu’est-ce que la fonctionnalité d’inscription automatique des zones privées Azure DNS ?
description: Vue d’ensemble de la fonctionnalité d’inscription automatique des zones privées Azure DNS
services: dns
author: rohinkoul
ms.service: dns
ms.topic: article
ms.date: 9/24/2019
ms.author: rohink
ms.openlocfilehash: 8e976199730ec74f9e76b9f4199e90e192916474
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94966715"
---
# <a name="what-is-the-autoregistration-feature-of-azure-dns-private-zones"></a>Qu’est-ce que la fonctionnalité d’inscription automatique des zones privées Azure DNS ?

La fonctionnalité d’inscription automatique des zones privées Azure DNS vous évite d’avoir à gérer vous-même les enregistrements DNS pour les machines virtuelles déployées dans un réseau virtuel. Quand vous [liez un réseau virtuel](./private-dns-virtual-network-links.md) avec une zone DNS privée et activez l’inscription automatique pour toutes les machines virtuelles, les enregistrements DNS des machines virtuelles déployées dans le réseau virtuel sont automatiquement créés dans la zone DNS privée. En plus des enregistrements de recherche directe (enregistrements A), les enregistrements de recherche inversée (enregistrements PTR) sont également créés automatiquement pour les machines virtuelles.
Si vous ajoutez d’autres machines virtuelles au réseau virtuel, leurs enregistrements DNS sont également créés automatiquement dans la zone DNS privée liée.

Quand vous supprimez une machine virtuelle, les enregistrements DNS de cette machine sont automatiquement supprimés dans la zone DNS privée.

Vous pouvez activer l’inscription automatique en sélectionnant l’option « Activer l’inscription automatique » au moment de la création d’une liaison de réseau virtuel.

![Activer l’inscription automatique](./media/privatedns-concepts/enable-autoregistration.png)

## <a name="restrictions"></a>Restrictions

* L’inscription automatique est possible uniquement pour les machines virtuelles. Pour toutes les autres ressources, comme les équilibreurs de charge internes, vous pouvez créer manuellement des enregistrements DNS dans la zone DNS privée qui est liée au réseau virtuel.
* Les enregistrements DNS sont créés automatiquement uniquement pour la carte réseau principale des machines virtuelles. Si vos machines virtuelles ont plusieurs cartes réseau, vous pouvez créer manuellement les enregistrements DNS pour les autres interfaces réseau.
* Les enregistrements DNS sont créés automatiquement uniquement si la carte réseau principale des machines virtuelles utilise le protocole DHCP. Si des adresses IP statiques sont configurées (par exemple, pour utiliser [plusieurs adresses IP dans Azure](../virtual-network/virtual-network-multiple-ip-addresses-portal.md#os-config)), l’inscription automatique ne crée pas d’enregistrements pour cette machine virtuelle.
* L’inscription automatique pour IPv6 (enregistrements AAAA) n’est pas prise en charge.

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment créer une zone privée dans Azure DNS à l’aide d’[Azure PowerShell](./private-dns-getstarted-powershell.md) ou d’[Azure CLI](./private-dns-getstarted-cli.md).

* Passez en revue certains [scénarios de zones privées](./private-dns-scenarios.md) courants qui peuvent être réalisés avec des zones privées dans Azure DNS.

* Pour trouver des réponses aux questions fréquemment posées concernant les zones privées dans Azure DNS, notamment le comportement spécifique auquel vous attendre pour certains types d’opérations, voir le [FAQ sur les DNS privés](./dns-faq-private.md).