---
title: 'Azure ExpressRoute : Prérequis'
description: Cette page fournit une liste des exigences à respecter avant de commander un circuit Azure ExpressRoute. Il inclut une liste de vérification.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: conceptual
ms.date: 09/18/2019
ms.author: duau
ms.openlocfilehash: 1fad1bca18d16ac3b6a654a3c289d0a14e3cd2e2
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92204790"
---
# <a name="expressroute-prerequisites--checklist"></a>Configuration requise pour ExpressRoute et liste de contrôle
Pour vous connecter aux services cloud de Microsoft en utilisant ExpressRoute, vous devez vérifier que les conditions requises des sections suivantes sont remplies.

[!INCLUDE [expressroute-office365-include](../../includes/expressroute-office365-include.md)]

## <a name="azure-account"></a>Compte Azure
* Un compte Microsoft Azure actif et valide. Ce compte est nécessaire pour configurer le circuit ExpressRoute. Les circuits ExpressRoute sont des ressources au sein des abonnements Azure. Un abonnement Azure est nécessaire, même si la connectivité est limitée aux services cloud Microsoft autres qu’Azure, comme Microsoft 365.
* Un abonnement Microsoft 365 actif (si vous utilisez des services Microsoft 365). Pour plus d’informations, consultez la section de la configuration requise de Microsoft 365 dans cet article.

## <a name="connectivity-provider"></a>Fournisseur de connectivité

* Vous pouvez travailler avec un [partenaire de connectivité ExpressRoute](expressroute-locations.md#partners) pour vous connecter à Microsoft Cloud. Vous pouvez établir une connexion entre votre réseau local et Microsoft de [trois façons différentes](expressroute-introduction.md).
* Si votre fournisseur n’est pas un partenaire de connectivité ExpressRoute, vous pouvez toujours vous connecter à Microsoft Cloud par le biais d’un [fournisseur de solutions d’interconnexion de clouds](expressroute-locations.md#connectivity-through-exchange-providers).

## <a name="network-requirements"></a>Configuration requise pour le réseau
* **Redondance à chaque emplacement de peering** : Microsoft exige la configuration de sessions BGP redondantes entre ses routeurs et les routeurs de peering sur chaque circuit ExpressRoute, même quand vous ne disposez que d’[une seule connexion physique à une solution d’interconnexion de clouds](expressroute-faqs.md#onep2plink).
* **Redondance pour la reprise d’activité après sinistre** : Microsoft vous recommande vivement de configurer au moins deux circuits ExpressRoute à des emplacements de peering différents pour éviter d’avoir un point de défaillance unique.
* **Routage** : selon la façon dont vous vous connectez à Microsoft Cloud, vous ou votre fournisseur pouvez avoir besoin de configurer et gérer les sessions BGP pour les [domaines de routage](expressroute-circuit-peerings.md). Certains fournisseurs de connectivité Ethernet ou fournisseurs d’échange cloud peuvent proposer la gestion BGP en tant que service à valeur ajoutée.
* **Traduction d’adresses réseau (NAT)** : Microsoft accepte uniquement les adresses IP publiques par le biais du peering Microsoft. Si votre réseau local comporte des adresses IP privées, vous ou votre fournisseur devez traduire les adresses IP privées en adresses IP publiques [à l’aide de la traduction d’adresses réseau](expressroute-nat.md).
* **Qualité de service (QoS)** : Skype Entreprise comprend différents services (par exemple, voix, vidéo, texte) nécessitant un traitement QoS différencié. Vous et votre fournisseur devez suivre la [configuration requise pour ExpressRoute QoS](expressroute-qos.md).
* **Sécurité réseau** : pensez à la [sécurité réseau](/azure/cloud-adoption-framework/reference/networking-vdc) lorsque vous vous connectez à Microsoft Cloud par le biais d’ExpressRoute.

## <a name="microsoft-365"></a>Microsoft 365
Si vous prévoyez d’activer Microsoft 365 sur ExpressRoute, consultez les documents suivants pour plus d’informations sur la configuration requise pour Microsoft 365.

* [Azure ExpressRoute pour Microsoft 365](/microsoft-365/enterprise/azure-expressroute)
* [Routage avec ExpressRoute pour Microsoft 365](/microsoft-365/enterprise/routing-with-expressroute)
* [Haute disponibilité et basculement avec ExpressRoute](/microsoft-365/enterprise/network-planning-with-expressroute)
* [URL et plages d’adresses IP Microsoft 365](/microsoft-365/enterprise/urls-and-ip-address-ranges)
* [Planification réseau et optimisation des performances pour Microsoft 365](/microsoft-365/enterprise/network-planning-and-performance)
* [Planification réseau et planification de la migration pour Microsoft 365](/microsoft-365/enterprise/network-and-migration-planning)
* [Intégration de Microsoft 365 à des environnements locaux](/microsoft-365/enterprise/microsoft-365-integration)
* [Vidéos de formation avancée à ExpressRoute sur Office 365](https://channel9.msdn.com/series/aer/)

## <a name="next-steps"></a>Étapes suivantes
* Pour plus d'informations sur ExpressRoute, consultez la [FAQ sur ExpressRoute](expressroute-faqs.md).
* Recherchez un fournisseur de connectivité ExpressRoute. Consultez [Partenaires ExpressRoute et emplacements de peering](expressroute-locations.md).
* Reportez-vous aux conditions requises pour le [routage](expressroute-routing.md), la [traduction d’adresses réseau](expressroute-nat.md) et la [qualité de service](expressroute-qos.md).
* Configurez votre connexion ExpressRoute.
  * [Création d’un circuit ExpressRoute](expressroute-howto-circuit-arm.md)
  * [Configuration du routage](expressroute-howto-routing-arm.md)
  * [Liaison d’un réseau virtuel à un circuit ExpressRoute](expressroute-howto-linkvnet-arm.md)