---
title: Peering Internet - FAQ
titleSuffix: Azure
description: Peering Internet - FAQ
services: internet-peering
author: prmitiki
ms.service: internet-peering
ms.topic: reference
ms.date: 11/27/2019
ms.author: prmitiki
ms.openlocfilehash: 9b0b2b08e01c99fc918c4bc5649197c9caa4978a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75774032"
---
# <a name="internet-peering---faqs"></a>Peering Internet - FAQ

Vous pouvez consulter les informations ci-dessous si vous avez des questions d’ordre général.

**Quelle est la différence entre le Peering Internet et Peering Service ?**

Peering Service est un service qui a pour but de fournir une connectivité IP publique de qualité professionnelle à Microsoft pour ses clients d’entreprise. L’Internet de qualité professionnelle comprend une connectivité par le biais de fournisseurs de services Internet disposant d’une connectivité à haut débit à Microsoft et une redondance pour une connectivité haute disponibilité. De plus, le trafic utilisateur est optimisé pour la latence à la périphérie Microsoft la plus proche. Peering Service s’appuie sur la connectivité de Peering avec l’opérateur partenaire. La connectivité de Peering avec le partenaire doit être un Peering direct plutôt qu’un Peering Exchange. Le Peering direct doit avoir une redondance locale et une géoredondance.

**Qu’est-ce que le Peering hérité ?**

La connexion de Peering configurée à l’aide d’Azure PowerShell est gérée en tant que ressource Azure. Les connexions de Peering configurées dans le passé sont stockées dans notre système sous la forme de Peering hérité que vous pouvez choisir de convertir et gérer en tant que ressource Azure.

**Quand New-AzPeeringDirectConnectionObject est appelée, quelles sont les adresses IP attribuées aux appareils Microsoft et aux homologues ?**

Lors de l’appel de l’applet de commande New-AzPeeringDirectConnectionObject, une adresse /31 (a.b.c.d/31) ou une adresse /30 (a.b.c.d/30) est entrée. La première adresse IP (a.b.c.d+0) est donnée à l’appareil de l’homologue, tandis que la deuxième adresse IP (a.b.c.d+1) est donnée à l’appareil Microsoft.

**À quoi correspondent les paramètres MaxPrefixesAdvertisedIPv4 et MaxPrefixesAdvertisedIPv6 dans l’applet de commande New-AzPeeringDirectConnectionObject ?**

Les paramètres MaxPrefixesAdvertisedIPv4 et MaxPrefixesAdvertisedIPv6 représentent le nombre maximal de préfixes IPv4 et IPv6 qu’un homologue souhaite que Microsoft accepte. Vous pouvez modifier ces paramètres à tout moment.