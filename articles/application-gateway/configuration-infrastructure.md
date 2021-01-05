---
title: Configuration de l’infrastructure Azure Application Gateway
description: Cet article explique comment configurer l’infrastructure Azure Application Gateway.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: conceptual
ms.date: 09/09/2020
ms.author: surmb
ms.openlocfilehash: f214b0b0751f44ea1357f569fd814a7621af61ab
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93397618"
---
# <a name="application-gateway-infrastructure-configuration"></a>Configuration de l’infrastructure Application Gateway

L’infrastructure Application Gateway comprend le réseau virtuel, les sous-réseaux, les groupes de sécurité réseau et les itinéraires définis par l’utilisateur.

## <a name="virtual-network-and-dedicated-subnet"></a>Réseau virtuel et sous-réseau dédié

Une passerelle d’application est un déploiement dédié dans votre réseau virtuel. Au sein de votre réseau virtuel, un sous-réseau dédié est nécessaire pour la passerelle d’application. Vous pouvez avoir plusieurs instances d’un déploiement de passerelle d’application donné dans un sous-réseau. Vous pouvez aussi déployer d’autres passerelles d’application dans le sous-réseau. Mais vous ne pouvez pas déployer une autre ressource dans le sous-réseau de la passerelle d’application. Vous ne pouvez pas mélanger Azure Application Gateway Standard_v2 et Standard sur le même sous-réseau.

> [!NOTE]
> Les [stratégies de points de terminaison de service de réseau virtuel](../virtual-network/virtual-network-service-endpoint-policies-overview.md) ne sont pas prises en charge dans un sous-réseau Application Gateway.

### <a name="size-of-the-subnet"></a>Taille du sous-réseau

Application Gateway utilise une adresse IP privée par instance, ainsi qu’une autre adresse IP privée si une adresse IP front-end privée est configurée.

Azure réserve également cinq adresses IP dans chaque sous-réseau pour un usage interne : les quatre premières et la dernière. Prenons l’exemple de 15 instances de passerelle d’application sans aucune adresse IP front-end privée. Vous avez besoin d’au moins 20 adresses IP pour ce sous-réseau : cinq pour une utilisation interne et 15 pour les instances de passerelle d’application.

Prenons l’exemple d’un sous-réseau disposant de 27 instances de passerelle d’application et d’une adresse front-end IP privée. Dans ce cas, vous avez besoin de 33 adresses IP : 27 pour les instances de passerelle d’application, une pour l’adresse front-end privée et cinq pour un usage interne.

Application Gateway (référence SKU Standard ou WAF) peut prendre en charge jusqu’à 32 instances (32 adresses IP d’instance + 1 IP frontale privée + 5 réservées pour Azure). Par conséquent, une taille de sous-réseau minimale de /26 est recommandée.

Application Gateway (référence SKU Standard_v2 ou WAF_v2) peut prendre en charge jusqu’à 125 instances (125 adresses IP d’instance + 1 IP frontale privée + 5 réservées pour Azure). Par conséquent, une taille de sous-réseau minimale de /24 est recommandée.

## <a name="network-security-groups"></a>Groupes de sécurité réseau

Les Groupes de sécurité réseau (NSG) sont pris en charge sur Application Gateway. Mais quelques restrictions s’appliquent :

- Vous devez autoriser le trafic Internet entrant sur les ports TCP 65503-65534 pour la référence (SKU) Application Gateway v1, et sur les ports TCP 65200-65535 pour la référence (SKU) v2 avec le sous-réseau de destination en tant que **Any** et la source en tant que balise de service **GatewayManager**. Cette plage de ports est nécessaire pour la communication avec l’infrastructure Azure. Ces ports sont protégés (verrouillés) par des certificats Azure. Les entités externes, y compris les clients de ces passerelles, ne peuvent pas communiquer avec ces points de terminaison.

- La connectivité Internet sortante ne peut pas être bloquée. Les règles de trafic sortant par défaut dans le groupe de sécurité réseau permettent une connectivité Internet. Nous vous recommandons :

  - De ne pas supprimer les règles de trafic sortant par défaut.
  - De ne pas créer d’autres règles de trafic sortant qui refusent toute connectivité sortante.

- Le trafic à partir de la balise **AzureLoadBalancer** avec le sous-réseau de destination comme **Tout** doit être autorisé.

### <a name="allow-access-to-a-few-source-ips"></a>Autoriser l’accès à quelques adresses IP sources

Pour ce scénario, utilisez des groupes de sécurité réseau sur le sous-réseau Application Gateway. Placez les restrictions suivantes sur le sous-réseau dans cet ordre de priorité :

1. Autorisez le trafic entrant à partir d’une adresse IP source ou d’une plage d’adresses IP avec la destination comme plage d’adresses de sous-réseau Application Gateway et le port de destination comme votre port d’accès entrant, par exemple, le port 80 pour l’accès HTTP.
2. Autorisez les demandes entrantes à partir de la source comme balise de service **GatewayManager** et la destination **Any** et les ports de destination 65503-65534 pour la référence SKU Application Gateway v1, et les ports 65200-65535 pour la référence SKU v2 pour les [communications de l’état d’intégrité du back-end](./application-gateway-diagnostics.md). Cette plage de ports est nécessaire pour la communication avec l’infrastructure Azure. Ces ports sont protégés (verrouillés) par des certificats Azure. Sans les certificats appropriés en place, les entités externes ne peuvent pas lancer des modifications sur ces points de terminaison.
3. Autorisez les sondes Azure Load Balancer entrantes (balise *AzureLoadBalancer* ) et le trafic de réseau virtuel entrant (balise *VirtualNetwork* ) sur le [Groupe de sécurité réseau](../virtual-network/network-security-groups-overview.md).
4. Bloquez tout autre trafic entrant avec une règle Tout refuser.
5. Autoriser le trafic sortant vers internet pour toutes les destinations.

## <a name="supported-user-defined-routes"></a>Itinéraires définis par l’utilisateur pris en charge 

> [!IMPORTANT]
> L’utilisation de routes définies par l’utilisateur sur le sous-réseau Application Gateway est susceptible d’entraîner l’indication de l’état d’intégrité **Inconnu** dans l’ [affichage de l’intégrité du back-end](./application-gateway-diagnostics.md#back-end-health). Elle peut également entraîner l’échec de la génération des journaux et métriques Application Gateway. Nous vous recommandons de ne pas utiliser de routes définies par l’utilisateur sur le sous-réseau Application Gateway afin de pouvoir voir l’état d’intégrité, les journaux et les métriques du back-end.

- **v1**

   Pour la référence SKU v1, les routes définies par l’utilisateur sont prises en charge sur le sous-réseau Application Gateway, tant qu’elles n’altèrent pas la communication de demande/réponse de bout en bout. Par exemple, vous pouvez configurer une route définie par l’utilisateur dans le sous-réseau Application Gateway pour pointer vers une appliance de pare-feu afin d’inspecter un paquet. Mais vous devez vérifier que le paquet peut atteindre sa destination prévue après l’inspection. S’il n’y parvient pas, cela peut entraîner un comportement incorrect de la sonde d’intégrité ou du routage du trafic. Sont incluses les routes apprises ou 0.0.0.0/0 par défaut propagées par Azure ExpressRoute ou des passerelles VPN dans le réseau virtuel. Tout scénario dans lequel 0.0.0.0/0 doit être redirigé localement (tunneling forcé) n’est pas pris en charge pour v1.

- **v2**

   Pour la référence SKU v2, il existe des scénarios pris en charge et non pris en charge :

   **Scénarios pris en charge par la v2**
   > [!WARNING]
   > Une configuration incorrecte de la table de routage peut entraîner un routage asymétrique dans Application Gateway v2. Assurez-vous que tout le trafic de gestion/plan de contrôle est envoyé directement à Internet et non par le biais d’une appliance virtuelle. La journalisation et les métriques peuvent également être affectées.


  **Scenario 1**  : UDR pour désactiver la propagation d’itinéraires du protocole de passerelle frontière (BGP) vers le sous-réseau d’Application Gateway

   Parfois, l’itinéraire de la passerelle par défaut (0.0.0.0/0) est publié via les passerelles VPN ou ExpressRoute associées au réseau virtuel Application Gateway. Cela interrompt le trafic du plan de gestion, qui nécessite un chemin d’accès direct à Internet. Dans de tels scénarios, un itinéraire défini par l’utilisateur (UDR) peut être utilisé pour désactiver la propagation d’itinéraires BGP. 

   Pour désactiver la propagation d’itinéraires BGP, procédez comme suit :

   1. Créez une ressource de table de routage dans Azure.
   2. Désactivez le paramètre **Propagation de la route de la passerelle de réseau virtuel**. 
   3. Associez la table de routage au sous-réseau approprié. 

   L’activation de l’UDR pour ce scénario ne doit pas perturber les configurations existantes.

  **Scénario 2**  : UDR pour diriger 0.0.0.0/0 vers Internet

   Vous pouvez créer un UDR pour envoyer le trafic de 0.0.0.0/0 directement vers Internet. 

  **Scénario 3** : UDR pour Azure Kubernetes Service avec kubenet

  Si vous utilisez kubenet avec Azure Kubernetes Service (AKS) et Application Gateway Ingress Controller (AGIC), vous avez besoin d’une table de route pour permettre au trafic envoyé aux pods d’Application Gateway d’être acheminé vers le bon nœud. Cela n’est pas nécessaire si vous utilisez Azure CNI. 

  Pour configurer la table de route afin de permettre à kubenet de fonctionner, procédez comme suit :

  1. Accédez au groupe de ressources créé par AKS (le nom du groupe de ressources doit commencer par « MC_ »)
  2. Recherchez la table de route créée par AKS dans ce groupe de ressources. La table de route doit être remplie avec les informations suivantes :
     - Le préfixe d’adresse doit être la plage d’adresses IP des pods que vous souhaitez atteindre dans AKS. 
     - Le type de tronçon suivant doit être Appliance virtuelle. 
     - L’adresse du tronçon suivant doit être l’adresse IP du nœud qui héberge les pods.
  3. Associez cette table de route au sous-réseau Application Gateway. 
    
  **Scénarios non pris en charge par la v2**

  **Scenario 1**  : UDR pour les appliances virtuelles

  Aucun scénario dans lequel 0.0.0.0/0 doit être redirigé via une appliance virtuelle, un réseau virtuel hub/spoke ou localement (tunneling forcé) n’est pas pris en charge pour V2.

## <a name="next-steps"></a>Étapes suivantes

- [En savoir plus sur la configuration des adresses IP frontales](configuration-front-end-ip.md).