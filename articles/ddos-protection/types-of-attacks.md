---
title: Types d’attaques atténuées par Azure DDoS Protection Standard
description: Découvrez les types d’attaques contre lesquels Azure DDoS Protection Standard agit.
services: ddos-protection
documentationcenter: na
author: yitoh
ms.service: ddos-protection
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/08/2020
ms.author: yitoh
ms.openlocfilehash: 8b213755aeed1590df3d1f5e91f44796c13c7711
ms.sourcegitcommit: 9889a3983b88222c30275fd0cfe60807976fd65b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94991772"
---
# <a name="types-of-ddos-attacks-overview"></a>Vue d’ensemble des types d’attaques DDoS

Le service Protection DDoS Standard peut atténuer les types d’attaques suivants :

- **Attaques volumétriques** : L’objectif de ces attaques est de submerger la couche réseau d’une quantité substantielle de trafic apparemment légitime. Elles incluent les saturations UDP, les saturations par amplification et autres saturations par falsification de paquets. Le service Protection DDoS Standard atténue ces attaques potentielles de plusieurs gigaoctets en les absorbant et en les purgeant, grâce à l’échelle du réseau global d’Azure, automatiquement.
- **Attaques de protocole** : ces attaques rendent une cible inaccessible, en exploitant une faille dans la pile de protocole des couches 3 et 4. Elles incluent les attaques par saturation SYN, les attaques par réflexion et d’autres attaques de protocole. Le service Protection DDoS Standard atténue ces attaques en faisant la distinction entre le trafic légitime et le trafic malveillant, qu’il bloque par l’intermédiaire du client. 
- **Attaques de la couche Ressource (Application)** : ces attaques ciblent les paquets d’application web pour interrompre la transmission des données entre des hôtes. Elles incluent les violations de protocole HTTP, l’injection SQL, les scripts de site à site et autres attaques de la couche 7. Utilisez un pare-feu d'applications web, tel que le [pare-feu d’applications web Application Gateway](../web-application-firewall/ag/ag-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) Azure, ainsi que le service Protection DDos Standard pour offrir une défense contre ces attaques. Il existe également des offres de pare-feu d’application web tiers sur la [Place de marché Microsoft Azure](https://azuremarketplace.microsoft.com/marketplace/apps?page=1&search=web%20application%20firewall).

## <a name="azure-ddos-protection-standard"></a>Service Protection DDos Standard Azure

Le service Protection DDos Standard protège les ressources d’un réseau virtuel, y compris les adresses IP publiques associées aux machines virtuelles, les équilibreurs de charge internes et les passerelles d’application. Associé au pare-feu d'applications web sur Application Gateway ou à un pare-feu d'applications web tiers déployé dans un réseau virtuel avec une adresse IP publique, le service Protection DDos Standard peut fournir une fonctionnalité d’atténuation complète pour les couches 3 à 7.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [créer un plan de protection DDoS](manage-ddos-protection.md).