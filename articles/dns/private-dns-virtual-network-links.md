---
title: Qu’est-ce qu’une sous-ressource de liaison de réseau virtuel dans les zones Azure DNS privées ?
description: Vue d’ensemble d’une sous-ressource de liaison de réseau virtuel dans une zone Azure DNS privée
services: dns
author: rohinkoul
ms.service: dns
ms.topic: article
ms.date: 9/24/2019
ms.author: rohink
ms.openlocfilehash: 5e43ad35cdaad89d5b9f43007bad8782f358c1a6
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94954339"
---
# <a name="what-is-a-virtual-network-link"></a>Qu’est-ce qu’une liaison de réseau virtuel ?

Quand vous créez une zone DNS privée dans Azure, cette zone n’est pas immédiatement accessible à partir d’un réseau virtuel. Vous devez lier la zone DNS privée au réseau virtuel pour qu’une machine virtuelle hébergée dans ce réseau puisse y accéder.
Pour lier une zone DNS privée à un réseau virtuel, vous devez créer une liaison de réseau virtuel sous la zone DNS privée. Chaque zone DNS privée a une collection de ressources enfants de liaisons de réseau virtuel. Chacune de ces ressources représente une connexion à un réseau virtuel.

Vous pouvez lier un réseau virtuel à une zone DNS privée en tant que réseau virtuel d’inscription ou réseau virtuel de résolution.

## <a name="registration-virtual-network"></a>Réseau virtuel d’inscription

Quand vous [créez une liaison](./private-dns-getstarted-portal.md#link-the-virtual-network) entre une zone DNS privée et un réseau virtuel, vous avez la possibilité d’activer l’[inscription automatique](./private-dns-autoregistration.md) des enregistrements DNS pour les machines virtuelles. Si vous choisissez cette option, le réseau virtuel devient un réseau virtuel d’inscription pour la zone DNS privée. Un enregistrement DNS est automatiquement créé pour les machines virtuelles que vous déployez dans le réseau. Des enregistrements DNS sont créés pour les machines virtuelles que vous avez déjà déployées dans le réseau virtuel. Du côté du réseau virtuel, la zone DNS privée devient la zone d’inscription pour ce réseau virtuel.
Une zone DNS privée peut avoir plusieurs réseaux virtuels d’inscription, mais chaque réseau virtuel peut être associé à une seule zone d’inscription.

## <a name="resolution-virtual-network"></a>Réseau virtuel de résolution

Quand vous créez une liaison de réseau virtuel sous une zone DNS privée et que vous choisissez de ne pas activer l’inscription automatique des enregistrements DNS, le réseau virtuel est considéré comme un réseau virtuel de résolution uniquement. Les enregistrements DNS pour les machines virtuelles déployées dans ce type de réseaux ne sont pas créés automatiquement dans la zone DNS privée liée. Toutefois, ces machines virtuelles peuvent interroger correctement les enregistrements DNS à partir de la zone DNS privée. Ces enregistrements peuvent être créés manuellement, ou être remplis à partir d’autres réseaux virtuels qui ont été liés à la zone DNS privée en tant que réseaux d’inscription.
Une zone DNS privée peut avoir plusieurs réseaux virtuels de résolution et un réseau virtuel peut être associé à plusieurs zones de résolution.

## <a name="limits"></a>limites

Pour savoir combien de réseaux d’inscription et de résolution vous pouvez lier à des zones DNS privées, consultez [Limites d’Azure DNS](../azure-resource-manager/management/azure-subscription-service-limits.md#azure-dns-limits).

## <a name="other-considerations"></a>Autres considérations

* Les réseaux virtuels déployés selon le modèle de déploiement Classic ne sont pas pris en charge.

* Vous ne pouvez créer qu’une seule liaison entre une zone DNS privée et un réseau virtuel.

* Chaque liaison de réseau virtuel sous une zone DNS privée doit avoir un nom unique dans le contexte de la zone DNS privée. Il peut y avoir des liaisons du même nom dans des zones DNS privées différentes.

* Après la création d’une liaison de réseau virtuel, vérifiez le champ « État du lien » de la ressource de liaison de réseau virtuel. Selon la taille du réseau virtuel, un délai de quelques minutes peut s’écouler avant que la liaison soit opérationnelle et que l’état de la liaison passe à *Terminé*.

* Quand vous supprimez un réseau virtuel, toutes les liaisons de réseau virtuel et les enregistrements DNS inscrits automatiquement qui y sont associés dans les différentes zones DNS privées sont automatiquement supprimés.

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment lier un réseau virtuel à une zone DNS privée à l’aide du [portail Azure](./private-dns-getstarted-portal.md#link-the-virtual-network).

* Découvrez comment créer une zone privée dans Azure DNS à l’aide [d’Azure PowerShell](./private-dns-getstarted-powershell.md) ou [d’Azure CLI](./private-dns-getstarted-cli.md).

* Passez en revue certains [scénarios de zones privées](./private-dns-scenarios.md) courants qui peuvent être réalisés avec des zones privées dans Azure DNS.

* Pour trouver des réponses aux questions fréquemment posées concernant les zones privées dans Azure DNS, notamment le comportement spécifique auquel vous attendre pour certains types d’opérations, voir le [FAQ sur les DNS privés](./dns-faq-private.md).