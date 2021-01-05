---
title: Acheminer le trafic via des appliances virtuelles réseau à l’aide de paramètres personnalisés
titleSuffix: Azure Virtual WAN
description: Ce scénario vous aide à acheminer le trafic via des appliances virtuelles réseau en utilisant une appliance virtuelle réseau différente pour le trafic lié à Internet.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 09/22/2020
ms.author: cherylmc
ms.custom: fasttrack-edit
ms.openlocfilehash: 122e76e4bde96823ff18207bc24df4a8e91afb1c
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92517966"
---
# <a name="scenario-route-traffic-through-nvas-by-using-custom-settings"></a>Scénario : Acheminer le trafic via des appliances virtuelles réseau à l’aide de paramètres personnalisés

Lorsque vous utilisez le routage de hub virtuel Azure Virtual WAN, vous disposez d’un certain nombre d’options. Cet article vous aide quand vous souhaitez acheminer le trafic via une appliance virtuelle réseau (NVA) pour la communication entre les réseaux virtuels et les branches et d’utiliser une NVA distincte pour le trafic Internet. Pour plus d’informations, consultez [À propos du routage de hub virtuel](about-virtual-hub-routing.md).

## <a name="design"></a>Conception

* Les **spokes** pour les réseaux virtuels connectés au hub virtuel. (Par exemple, VNet 1, VNet 2 et VNet 3 dans le diagramme plus loin dans cet article.)
* **Réseau virtuel de service** pour les réseaux virtuels où les utilisateurs ont déployé une NVA pour inspecter le trafic non-Internet, et éventuellement avec des services communs accédés par des spokes. (Par exemple, VNet 4 dans le diagramme plus loin dans cet article.) 
* **Réseau virtuel de périmètre** pour les réseaux virtuels où les utilisateurs ont déployé une NVA pour inspecter le trafic Internet. (Par exemple, VNet 5 dans le diagramme plus loin dans cet article.)
* **Hubs** pour les hubs Virtual WAN gérés par Microsoft.

Le tableau suivant fournit un récapitulatif des connexions prises en charge dans ce scénario :

| À partir          | À|Spokes|Réseau virtuel de service|Branches|Internet|
|---|---|:---:|:---:|:---:|:---:|:---:|
| **Spokes**| ->| directement ; |directement ; | via le réseau virtuel de service |Via un réseau virtuel de périmètre |
| **Réseau virtuel de service**| ->| directement ; |n/a| directement ; | |
| **Branches** | ->| via le réseau virtuel de service |directement ;| directement ; |  |

Chacune des cellules de la matrice de connectivité indique si la connectivité circule directement via Virtual WAN ou sur l’un des réseaux virtuels avec une appliance virtuelle réseau. 

Notez les points suivants :
* Spokes :
  * Les spokes atteindront d’autres spokes directement via des hubs Virtual WAN.
  * Les spokes obtiendront la connectivité aux branches via un itinéraire statique qui pointe vers le réseau virtuel de service. Ils n’apprennent pas les préfixes spécifiques des branches, car ceux-ci sont plus spécifiques et remplaceraient le résumé.
  * Les spokes enverront le trafic Internet vers le réseau virtuel de périmètre via un appairage direct de réseaux virtuels.
* Les branches atteindront les spokes via un routage statique pointant vers le réseau virtuel de service. Ils n’apprennent pas les préfixes spécifiques des réseaux virtuels qui remplacent l’itinéraire statique résumé.
* Le réseau virtuel de service sera similaire à un réseau virtuel de services partagés qui doit être accessible à partir de chaque réseau virtuel et de chaque branche.
* Le réseau virtuel de périmètre n’a pas besoin d’une connectivité sur Virtual WAN, car le seul trafic qu’il prendra en charge passe par des peerings de réseaux virtuels directs. Toutefois, afin de simplifier la configuration, nous allons utiliser le même modèle de connectivité que pour le réseau virtuel de périmètre.

Nous avons trois modèles de connectivité distincts, qui se traduisent par trois tables de routage. Les associations aux différents réseaux virtuels sont les suivantes :

* Spokes :
  * Table de routage associée : **RT_V2B**
  * Propagation aux tables de routage : **RT_V2B** et **RT_SHARED**
* Réseaux virtuels d’appliance virtuelle réseau (réseau virtuel de service et réseau virtuel DMZ) :
  * Table de routage associée : **RT_SHARED**
  * Propagation aux tables de routage : **RT_SHARED**
* Branches :
  * Table de routage associée : **Par défaut**
  * Propagation aux tables de routage : **RT_SHARED** et **Par défaut**

Ces itinéraires statiques garantissent que le trafic à destination et en provenance du réseau virtuel et de la branche parcourt l’appliance virtuelle réseau dans le réseau virtuel de service (VNet 4) :

| Description | Table de routage | Itinéraire statique              |
| ----------- | ----------- | ------------------------- |
| Branches    | RT_V2B      | 10.2.0.0/16 -> vnet4conn  |
| Spokes NVA  | Default     | 10.1.0.0/16 -> vnet4conn  |

Vous pouvez maintenant utiliser Virtual WAN pour sélectionner la connexion appropriée à laquelle envoyer les paquets. Vous devez également utiliser Virtual WAN pour sélectionner l’action appropriée à effectuer lors de la réception de ces paquets. Pour ce faire, vous utilisez les tables de routage de connexion, comme suit :

| Description | Connexion | Itinéraire statique            |
| ----------- | ---------- | ----------------------- |
| VNet2Branch | vnet4conn  | 10.2.0.0/16 -> 10.4.0.5 |
| Branch2VNet | vnet4conn  | 10.1.0.0/16 -> 10.4.0.5 |

Pour plus d’informations, consultez [À propos du routage de hub virtuel](about-virtual-hub-routing.md).

## <a name="architecture"></a>Architecture

Voici un diagramme de l’architecture décrite précédemment dans cet article.

Il y a un hub, appelé **Hub 1** .

* **Hub1** est connecté directement aux réseaux virtuels NVA **VNet 4** et **VNet 5** .

* Le trafic entre les VNets 1, 2 et 3 et les branches devrait passer par la **NVA VNet 4**  10.4.0.5.

* Tout le trafic Internet en provenance des VNets 1, 2 et 3 devrait passer par la **NVA VNet 5**  10.5.0.5.

:::image type="content" source="./media/routing-scenarios/nva-custom/figure-1.png" alt-text="Diagramme de l’architecture d’un réseau.":::

## <a name="workflow"></a>Workflow

Pour configurer le routage via la NVA, voici les étapes à prendre en compte :

1. Pour que le trafic Internet transite via VNet 5, vous avez besoin que les VNets 1, 2 et 3 se connectent directement à VNet 5 via l’appairage de réseaux virtuels. Vous avez également besoin d’un itinéraire défini par l’utilisateur dans les réseaux virtuels pour 0.0.0.0/0 et le tronçon suivant 10.5.0.5. Actuellement, Virtual WAN n’autorise pas une appliance virtuelle réseau de tronçon suivant dans le hub virtuel pour 0.0.0.0/0.

1. Dans le portail Azure, accédez à votre hub virtuel et créez une table de routage personnalisée appelée **RT_Shared** . Ce tableau apprend les itinéraires via la propagation à partir de toutes connexions de réseau virtuel et de branche. Vous pouvez voir cette table vide dans le diagramme suivant.

   * **Itinéraires :** Vous n’avez pas besoin d’ajouter des itinéraires statiques.

   * **Association :** Sélectionnez les VNet 4 et 5, ce qui signifie que les connexions de ces réseaux virtuels sont associées à la table de routage **RT_Shared** .

   * **Propagation :** Étant donné que vous souhaitez que l’ensemble des branches et des connexions de réseau virtuel propagent leurs itinéraires de manière dynamique vers cette table de routage, sélectionnez les branches et tous les réseaux virtuels.

1. Créez une table de routage personnalisée appelée **RT_V2B** pour acheminer le trafic des VNets 1, 2 et 3 vers les branches.

   * **Itinéraires :** ajoutez une entrée d’itinéraire statique agrégé pour les branches, avec un tronçon suivant en guise de connexion de VNet 4. Configurez un itinéraire statique dans la connexion de VNet 4 pour les préfixes de branche et indiquez le tronçon suivant vers l’adresse IP spécifique de la NVA dans VNet 4.

   * **Association :** sélectionnez tous les VNets 1, 2 et 3. Cela implique que les connexions de réseau virtuel 1, 2 et 3 seront associées à cette table de routage et pourront apprendre des itinéraires statiques et dynamiques à partir de la propagation dans cette table de routage.

   * **Propagation :** les connexions propagent des itinéraires vers des tables de routage. La sélection des VNets 1, 2 et 3 permet de propager les itinéraires des VNets 1, 2 et 3 vers cette table de routage. Il n’est pas nécessaire de propager les itinéraires des connexions de branche vers **RT_V2B** , car le trafic de réseau virtuel de branche transite par la NVA dans VNet 4.
  
1. Modifiez la table de routage par défaut **DefaultRouteTable** .

   Toutes les connexions VPN, Azure ExpressRoute et VPN utilisateur sont associées à la table de routage par défaut. Toutes les connexions VPN, ExpressRoute et VPN utilisateur propagent des itinéraires vers le même ensemble de tables de routage.

   * **Itinéraires :** ajoutez une entrée d’itinéraire statique agrégé pour les VNets 1, 2 et 3, avec un tronçon en guise de connexion de VNets 4. Configurez un itinéraire statique dans la connexion de VNet 4 pour les préfixes agrégés des VNets 1, 2 et 3 et indiquez le tronçon suivant vers l’adresse IP spécifique de la NVA dans VNet 4.

   * **Association :** Assurez-vous que l’option des branches (VPN/ER/P2S) est sélectionnée, en veillant à ce que les connexions de branches locales soient associées à la table de routage par défaut.

   * **Propagation à partir de :** Assurez-vous que l’option des branches (VPN/ER/P2S) est sélectionnée, en veillant à ce que les connexions locales soient des routes propagées à la table de routage par défaut.

:::image type="content" source="./media/routing-scenarios/nva-custom/figure-2.png" alt-text="Diagramme de l’architecture d’un réseau." lightbox="./media/routing-scenarios/nva-custom/figure-2.png":::

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur Virtual WAN, consultez la [FAQ](virtual-wan-faq.md).
* Pour plus d’informations sur le routage de hub virtuel, consultez [À propos du routage de hub virtuel](about-virtual-hub-routing.md).
