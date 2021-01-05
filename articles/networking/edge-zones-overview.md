---
title: À propos d’Azure Edge Zone (préversion)
description: 'Découvrez les offres Edge Computing de Microsoft : Azure Edge Zone.'
services: vnf-manager
author: ganesr
ms.service: vnf-manager
ms.topic: article
ms.date: 07/07/2020
ms.author: ganesr
ms.openlocfilehash: 68aa3342ab09be73a82f4f896ffdff99d15a5350
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91329568"
---
# <a name="about-azure-edge-zone-preview"></a>À propos d’Azure Edge Zone (préversion)

Azure Edge Zone est une famille d’offres de Microsoft Azure qui permet de traiter les données à proximité de l’utilisateur. Vous pouvez déployer des machines virtuelles, des conteneurs et certains services Azure dans Edge Zone pour répondre aux exigences de faible latence et de débit élevé des applications.

Les scénarios d’utilisation classiques pour les zones périphériques sont les suivants :

- Commande et contrôle en temps réel en robotique
- Analytique et inférence en temps réel au moyen de l’intelligence artificielle et du Machine Learning
- Vision industrielle
- Rendu à distance pour les scénarios VDI et de réalité mixte
- Gaming multijoueur immersif
- Streaming multimédia et distribution de contenu
- Supervision et sécurité

Il existe trois types de zones périphériques Azure :

- Azure Edge Zones
- Azure Edge Zones avec opérateur
- Azure Private Edge Zones

## <a name="azure-edge-zones"></a><a name="edge-zones"></a>Azure Edge Zones

![Zones périphériques Azure](./media/edge-zones-overview/edge-zones.png "Azure Edge Zones")

Les zones périphériques Azure sont des extensions Azure à faible empreinte mémoire qui sont placées dans des centres de population éloignés des régions Azure. Elles prennent en charge les machines virtuelles, les conteneurs et une sélection de services Azure qui permettent d’exécuter des applications sensibles à la latence et gourmandes en débit au plus près des utilisateurs finaux. Elles font partie du réseau Microsoft mondial. Elles fournissent une connectivité sécurisée, fiable et à large bande passante entre les applications qui s’exécutent à la zone périphérique à proximité de l’utilisateur. De plus, elles offrent l’ensemble complet des services Azure s’exécutant dans les régions Azure. Elles sont détenues et gérées par Microsoft. Vous pouvez utiliser le même ensemble d’outils Azure et le même portail pour gérer et déployer des services dans des zones périphériques.

Les cas d’utilisation classiques sont les suivants :

- Gaming et streaming de jeux
- Streaming multimédia et distribution de contenu
- Analytique et inférence en temps réel au moyen de l’intelligence artificielle et du Machine Learning
- Rendu pour la réalité mixte

Les zones périphériques Azure seront disponibles dans les zones métropolitaines suivantes :

- New York
- Los Angeles, CA
- Miami, FL

Pour plus d’informations, [contactez l’équipe Edge Zone](https://aka.ms/EdgeZones).

## <a name="azure-edge-zones-with-carrier"></a><a name="carrier"></a>Azure Edge Zones avec opérateur

![Edge Zones with Carrier](./media/edge-zones-overview/edge-carrier.png "Edge Zones avec opérateur")

Les zones périphériques Azure avec opérateur sont des extensions Azure à faible empreinte mémoire qui sont placées dans les centres de donnés des opérateurs de téléphonie mobile situés dans des centres de population. L’infrastructure des zones périphériques Azure avec opérateur est placée à une distance d’un tronçon par rapport au réseau 5G de l’opérateur mobile. Ce placement offre une latence inférieure à 10 millisecondes aux applications des appareils mobiles.

Les zones périphériques Azure avec opérateur sont déployées dans les centres de données des opérateurs de téléphonie mobile et connectées au réseau Microsoft mondial. Elles fournissent une connectivité sécurisée, fiable et à large bande passante entre les applications qui s’exécutent à proximité de l’utilisateur. De plus, elles offrent l’ensemble complet des services Azure s’exécutant dans les régions Azure. Les développeurs peuvent utiliser le même ensemble d’outils habituels pour créer et déployer des services dans les zones périphériques.

Les cas d’utilisation classiques sont les suivants :

- Gaming et streaming de jeux
- Streaming multimédia et distribution de contenu
- Analytique et inférence en temps réel au moyen de l’intelligence artificielle et du Machine Learning
- Rendu pour la réalité mixte
- Véhicules connectés
- Télémédecine

Les zones périphériques seront proposées en partenariat avec les opérateurs suivants :

- AT&T (Atlanta, Dallas et Los Angeles)

Les éditeurs de logiciels indépendants qui travaillent sur des applications optimisées et évolutives connectées à des réseaux 5G peuvent désormais utiliser le nouvel emplacement d’aperçu Los Angeles d’Azure Edge Zones avec AT&T lors de la génération et de l’expérimentation de scénarios mobiles et connectés avec des plateformes à latence ultra-faible. Inscrivez-vous au programme d’utilisateur précoce pour tirer parti d’une connectivité sécurisée à bande passante élevée.

Pour plus d’informations, [contactez l’équipe Edge Zone](https://aka.ms/EdgeZones).

## <a name="azure-private-edge-zones"></a><a name="private-edge-zones"></a>Azure Private Edge Zones

![Private Edge Zones](./media/edge-zones-overview/private-edge.png "Private Edge Zones")

Les zones périphériques privées Azure sont des extensions Azure à faible empreinte mémoire qui sont placées localement. Une zone périphérique privée Azure est basée sur la plateforme [Azure Stack Edge](https://azure.microsoft.com/products/azure-stack/edge/). Elle permet un accès à faible latence aux services de calcul et de stockage déployés localement. Une zone périphérique privée vous permet également de déployer des applications d’éditeurs de logiciels indépendants (ISV) et des fonctions réseau virtualisées (VNF), comme les [applications managées Azure](https://azure.microsoft.com/services/managed-applications/) avec des machines virtuelles et des conteneurs locaux. Ces fonctions VNF peuvent inclure des cœurs de paquets mobiles, des routeurs, des pare-feu et des appliances SD-WAN. Une zone périphérique privée Azure est proposée avec une solution d’orchestration native cloud qui vous permet de gérer les cycles de vie des fonctions VNF et des applications à partir du portail Azure.

Elle vous permet de développer et de déployer des applications localement à l’aide des mêmes outils habituels que vous utilisez pour créer et déployer des applications dans Azure.

Elle vous permet également d’effectuer les opérations suivantes :

- Exécuter des réseaux mobiles privés (LTE privée, 5G privée)
- Implémenter des fonctions de sécurité telles que les pare-feu
- Étendre vos réseaux locaux sur plusieurs succursales et Azure en utilisant des appliances SD-WAN sur les mêmes appliances de zone périphérique privée et les gérer à partir d’Azure

Les cas d’utilisation classiques sont les suivants :

- Commande et contrôle en temps réel en robotique
- Analytique et inférence en temps réel avec l’intelligence artificielle et le Machine Learning
- Vision industrielle
- Rendu à distance pour les scénarios VDI et de réalité mixte
- Supervision et sécurité

Nous disposons d’un écosystème complet de fournisseurs VNF, d’ISV et de partenaires MSP pour activer des solutions de bout en bout qui utilisent des zones périphériques privées. Pour plus d’informations, [contactez l’équipe des zones périphériques privées](https://aka.ms/EdgeZonesPartner).

### <a name="private-edge-zone-partners"></a><a name="private-edge-partners"></a>Partenaires pour les zones périphériques privées

![Partenaires pour les zones périphériques privées](./media/edge-zones-overview/partners.png "Partenaires pour les zones périphériques privées")

#### <a name="virtualized-network-functions-vnfs"></a><a name="vnf"></a>Fonctions réseau virtualisées (VNF)

##### <a name="virtualized-evolved-packet-core-vepc-for-mobile-networks"></a><a name="vEPC"></a>Evolved Packet Core virtualisé (vEPC) pour réseaux mobiles

- [Affirmed Networks](https://www.affirmednetworks.com/)
- [Celona](https://www.celona.io/azure-edge)
- [Druid Software](https://www.druidsoftware.com/)
- [Expeto](https://www.expeto.io/)
- [Mavenir](https://mavenir.com/)
- [Metaswitch](https://www.metaswitch.com/)
- [Nokia Digital Automation Cloud](https://www.dac.nokia.com/)

##### <a name="mobile-radio-partners"></a><a name="mobile-radio"></a>Partenaires de radio mobile

- [Celona](https://www.celona.io/azure-edge)
- [Commscope Ruckus](https://support.ruckuswireless.com/)

##### <a name="sd-wan-vendors"></a><a name="sdwan-vendors"></a>Fournisseurs SD-WAN

- [128 Technology](https://www.128technology.com/)
- [NetFoundry](https://netfoundry.io/)
- [Nuage Networks de Nokia](https://www.nuagenetworks.net/)
- [Versa Networks](https://www.versa-networks.com/)
- [SD-WAN VMware de Velocloud](https://www.velocloud.com/)

##### <a name="router-vendors"></a><a name="router-vendors"></a>Fournisseurs de routeur

- [Arista](https://www.arista.com/)

##### <a name="firewall-vendors"></a><a name="firewall-vendors"></a>Fournisseurs de pare-feu

- [Palo Alto Networks](https://www.paloaltonetworks.com/)

##### <a name="managed-solutions-providers-mobile-operators-and-global-system-integrators-gsis"></a><a name="msp-mobile"></a>Fournisseurs de solutions managées : opérateurs de téléphonie mobile et intégrateurs de systèmes mondiaux

| Intégrateurs de systèmes mondiaux et opérateurs | Opérateurs de téléphonie mobile |
| --- | --- |
| Amdocs                       | Etisalat             |
| American Tower               | NTT Communications   |
| CenturyLink                  | Proximus             |
| Expeto                       | Rogers               |
| Federated Wireless           | SK Telecom           |
| Infosys                      | Telefonica           |
| Tech Mahindra                | Telstra              |
|                              | Vodafone             |

Pour plus d’informations sur la façon de devenir partenaire, [contactez l’équipe des zones périphériques privées](https://aka.ms/EdgeZonesPartner).

### <a name="private-edge-zone-solutions"></a><a name="solutions-private-edge"></a>Solutions pour les zones périphériques privées

#### <a name="private-mobile-network-on-private-edge-zones"></a><a name="private-mobile-private-edge"></a>Réseau mobile privé sur les zones périphériques privées

![Réseau mobile privé sur les zones périphériques privées](./media/edge-zones-overview/mobile-networks.png "Réseau mobile privé sur les zones périphériques privées")

Vous pouvez maintenant déployer un réseau mobile privé sur les zones périphériques privées. Les réseaux mobiles privés permettent d’avoir une latence très faible, une capacité élevée et le réseau sans fil fiable et sécurisé nécessaire pour les applications stratégiques.

Les réseaux mobiles privés peuvent permettre des scénarios tels que les suivants :
- Commande et contrôle des véhicules guidés automatisés (AGV) dans les entrepôts
- Communication en temps réel entre les robots dans les usines intelligentes
- Réalité augmentée et applications de réalité virtuelle de pointe

La fonction réseau vEPC (Evolved Packet Core virtualisé) est le cerveau d’un réseau mobile privé. Vous pouvez maintenant déployer un vEPC sur Private Edge Zones. Pour obtenir la liste des partenaires vEPC disponibles sur les zones périphériques privées, consultez [ISV vEPC](#vEPC).

Le déploiement d’une solution de réseau mobile privé sur les zones périphériques privées nécessite d’autres composants, tels que des points d’accès mobiles, des cartes SIM et d’autres VNF (par exemple, des routeurs). L’accès à un spectre sous licence ou sans licence est essentiel à la configuration d’un réseau mobile privé. De plus, vous pouvez avoir besoin d’aide pour la planification RF, la disposition physique, l’installation et le support. Pour obtenir la liste des partenaires, consultez [Partenaires de radio mobile](#mobile-radio).

Microsoft fournit un écosystème de partenaires qui peut s’avérer utile pour tous les aspects de ce processus. Les partenaires peuvent vous aider à planifier le réseau, à acheter les appareils nécessaires, à configurer le matériel et à gérer la configuration à partir d’Azure. Un ensemble de partenaires agréés et étroitement intégrés à Microsoft garantit la fiabilité et la facilité d’utilisation de votre solution. Vous pouvez vous concentrer sur vos scénarios clés, Microsoft et ses partenaires seront là pour vous aider avec le reste.

#### <a name="sd-wan-on-private-edge-zones"></a><a name="sdwan-private-edge">SD-WAN sur Private Edge Zones</a>

![SD-WAN on Private Edge Zones](./media/edge-zones-overview/sd-wan.png "SD-WAN sur Private Edge Zones")

SD-WAN vous permet de créer des réseaux étendus (WAN) de niveau entreprise qui présentent les avantages suivants :

- Bande passante accrue
- Accès haute performance au cloud
- Insertion de services
- Fiabilité
- Gestion des stratégies
- Visibilité totale du réseau

SD-WAN fournit une connectivité transparente pour les succursales qui est orchestrée à partir de contrôleurs centraux redondants à un coût de possession réduit.
SD-WAN sur les zones périphériques privées vous permet de passer d’un modèle centré sur les investissements à un modèle SaaS (software-as-a-service) pour réduire les budgets informatiques. Vous pouvez utiliser les partenaires SD-WAN de votre choix (orchestrateur ou contrôleur) pour activer de nouveaux services et les propager immédiatement sur l’ensemble de votre réseau.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, contactez les équipes suivantes :

* [Équipe Edge Zone](https://aka.ms/EdgeZones)
* [Équipe des zones périphériques privées, pour devenir partenaire](https://aka.ms/EdgeZonesPartner)
