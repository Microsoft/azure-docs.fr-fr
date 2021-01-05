---
title: Qu’est-ce qu’une zone Azure DNS privée ?
description: Vue d’ensemble d’une zone DNS privée
services: dns
author: rohinkoul
ms.service: dns
ms.topic: article
ms.date: 9/24/2019
ms.author: rohink
ms.openlocfilehash: 9eaa320e79f1d595303c6d9fe1399df12cb6c52b
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94954407"
---
# <a name="what-is-a-private-azure-dns-zone"></a>Qu’est-ce qu’une zone Azure DNS privée ?

Le DNS privé Azure fournit un service DNS fiable et sécurisé pour gérer et résoudre les noms de domaine dans un réseau virtuel sans nécessiter l’ajout de solution DNS personnalisée. Grâce aux zones DNS privées, vous pouvez utiliser vos propres noms de domaine personnalisés au lieu des noms fournis par Azure actuellement disponibles. 

Les enregistrements contenus dans une zone DNS privée ne peuvent pas être résolus à partir d’Internet. La résolution des enregistrements DNS sur une zone DNS privée fonctionne uniquement à partir de réseaux virtuels qui sont liés à cette zone.

Vous pouvez lier une zone DNS privée à un ou plusieurs réseaux virtuels en créant des [liaisons de réseau virtuel](./private-dns-virtual-network-links.md).
Vous pouvez également activer la fonctionnalité d’[inscription automatique](./private-dns-autoregistration.md) pour gérer automatiquement le cycle de vie des enregistrements DNS des machines virtuelles déployées dans un réseau virtuel.

## <a name="limits"></a>limites

Pour savoir combien de zones DNS privées vous pouvez créer dans un abonnement et combien de jeux d’enregistrements peuvent être pris en charge dans une zone DNS privée, consultez [Limites d’Azure DNS](../azure-resource-manager/management/azure-subscription-service-limits.md#azure-dns-limits).

## <a name="restrictions"></a>Restrictions

* Les zones DNS privées à étiquette unique ne sont pas prises en charge. Une zone DNS privée doit avoir au moins deux étiquettes. Par exemple, contoso.com a deux étiquettes séparées par un point. Une zone DNS privée peut avoir un maximum de 34 étiquettes.
* Vous ne pouvez pas créer de délégations de zone (enregistrements NS) dans une zone DNS privée. Si vous comptez utiliser un domaine enfant, vous pouvez directement créer le domaine en tant que zone DNS privée et le lier au réseau virtuel sans configurer une délégation de serveur de noms à partir de la zone parente.

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment créer une zone privée dans Azure DNS à l’aide [d’Azure PowerShell](./private-dns-getstarted-powershell.md) ou [d’Azure CLI](./private-dns-getstarted-cli.md).

* Passez en revue certains [scénarios de zones privées](./private-dns-scenarios.md) courants qui peuvent être réalisés avec des zones privées dans Azure DNS.

* Pour trouver des réponses aux questions fréquemment posées concernant les zones privées dans Azure DNS, notamment le comportement spécifique auquel vous attendre pour certains types d’opérations, voir le [FAQ sur les DNS privés](./dns-faq-private.md).