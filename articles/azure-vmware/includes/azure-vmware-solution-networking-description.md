---
title: Connectivité et réseau Azure VMware Solution
description: Description de la connectivité et du réseau Azure VMware Solution.
ms.topic: include
ms.date: 09/28/2020
ms.openlocfilehash: 47b5fab8430f4e462f1e4a0e8cb6ddb69ec38c02
ms.sourcegitcommit: d76108b476259fe3f5f20a91ed2c237c1577df14
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92925031"
---
<!-- Used in introduction.md and concepts-networking.md -->

Azure VMware Solution offre un environnement de cloud privé, qui est accessible à partir de ressources ou d’environnements locaux et Azure. Des services tels qu’Azure ExpressRoute et les connexions VPN fournissent la connectivité. Ces services requièrent des plages d’adresses réseau et des ports de pare-feu spécifiques pour leur activation.

Lors du déploiement d’un cloud privé, des réseaux privés pour la gestion, l’approvisionnement et vMotion sont créés. Utilisez ces réseaux privés pour accéder à vCenter et NSX-T Manager et à vMotion des machines virtuelles, ou pour le déploiement.  ExpressRoute Global Reach est utilisé pour connecter les clouds privés à votre environnement local. La connexion requiert un réseau virtuel avec un circuit ExpressRoute dans votre abonnement.

Les ressources, telles que les serveurs web et les machines virtuelles, sont accessibles via Internet par le biais de la fonctionnalité d’IP publique d’Azure Virtual WAN.  Par défaut, l’accès à Internet est désactivé pour les nouveaux clouds privés. Pour plus d’informations, consultez [Comment utiliser la fonctionnalité d’adresse IP publique dans Azure VMware Solution](../public-ip-usage.md).