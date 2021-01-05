---
title: Personnalisation de routes définies par l’utilisateur dans Azure Kubernetes Service (AKS)
description: Découvrez comment définir un route de sortie personnalisée dans Azure Kubernetes Service (AKS)
services: container-service
ms.topic: article
ms.date: 06/29/2020
ms.openlocfilehash: 103d7dc76dee56a336f08f2cc0c7c8489c0bc565
ms.sourcegitcommit: 99955130348f9d2db7d4fb5032fad89dad3185e7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93348132"
---
# <a name="customize-cluster-egress-with-a-user-defined-route"></a>Personnaliser la sortie du cluster avec une route définie par l’utilisateur

La sortie d’un cluster AKS peut être personnalisée pour l’adapter à des scénarios spécifiques. Par défaut, AKS provisionne un équilibreur de charge de référence SKU standard à configurer et utiliser pour la sortie. Cependant, la configuration par défaut peut ne pas répondre aux exigences de tous les scénarios si les adresses IP publiques ne sont pas autorisées ou si des tronçons supplémentaires sont nécessaires pour la sortie.

Cet article explique comment personnaliser une route de sortie d’un cluster pour prendre en charge des scénarios réseau personnalisés, comme ceux qui n’autorisent pas les adresses IP publiques et qui nécessitent que le cluster se trouve derrière une appliance virtuelle réseau ().

## <a name="prerequisites"></a>Prérequis
* Azure CLI version 2.0.81 ou ultérieure
* Version de l’API `2020-01-01` ou ultérieure


## <a name="limitations"></a>Limites
* OutboundType peut être défini seulement lors de la création du cluster et ne peut pas être mis à jour après cela.
* La définition de `outboundType` nécessite des clusters AKS avec un `vm-set-type` `VirtualMachineScaleSets` et une `load-balancer-sku` `Standard`.
* La définition de `outboundType` sur une valeur `UDR` nécessite une route définie par l’utilisateur avec une connectivité de sortie valide pour le cluster.
* La définition de `outboundType` sur la valeur `UDR` implique que l’adresse IP source en entrée routée vers l’équilibreur de charge puisse **ne pas correspondre** à l’adresse de destination en sortie de la sortie du cluster.

## <a name="overview-of-outbound-types-in-aks"></a>Vue d’ensemble des types de sortie dans AKS

Un cluster AKS peut être personnalisé à l’aide d’un `outboundType` unique de type `loadBalancer` ou `userDefinedRouting`.

> [!IMPORTANT]
> Le type de sortie impacte seulement le trafic sortant de votre cluster. Pour plus d’informations, consultez [Configuration des contrôleurs d’entrée](ingress-basic.md).

> [!NOTE]
> Vous pouvez utiliser votre propre [table de routage][byo-route-table] avec la mise en réseau UDR et kubenet. Assurez-vous que l’identité du cluster (principal du service ou identité managée) a des autorisations de contributeur sur la table de route personnalisée.

### <a name="outbound-type-of-loadbalancer"></a>Type de sortie loadBalancer

Si `loadBalancer` est défini, AKS effectue automatiquement la configuration suivante. L’équilibreur de charge est utilisé pour la sortie via une adresse IP publique affectée à AKS. Un type de sortie `loadBalancer` prend en charge les services Kubernetes de type `loadBalancer`, qui attendent une sortie de l’équilibreur de charge créé par le fournisseur de ressources AKS.

La configuration suivante est effectuée par AKS.
   * Une adresse IP publique est provisionnée pour la sortie du cluster.
   * L’adresse IP publique est affectée à la ressource d’équilibreur de charge.
   * Les pools de back-end pour l’équilibreur de charge sont configurés pour les nœuds d’agent dans le cluster.

Vous trouverez ci-dessous une topologie de réseau déployée dans des clusters AKS par défaut, qui utilise un `outboundType` `loadBalancer`.

![Le diagramme illustre les IP entrante et sortante, où l’IP entrante dirige le trafic vers un équilibreur de charge, qui dirige le trafic vers et depuis un cluster interne et un autre trafic vers l’IP sortante, qui dirige le trafic vers Internet, MCR, les services Azure requis et le plan de contrôle AKS.](media/egress-outboundtype/outboundtype-lb.png)

### <a name="outbound-type-of-userdefinedrouting"></a>Type de sortie userDefinedRouting

> [!NOTE]
> L’utilisation d’un type de sortie est un scénario réseau avancé et nécessite une configuration réseau appropriée.

Si `userDefinedRouting` est défini, AKS ne configure pas automatiquement les chemins de sortie. Vous devez configurer la sortie vous-même.

Le cluster AKS doit être déployé dans un réseau virtuel existant qui contient un sous-réseau préalablement configuré. En effet, quand vous n’utilisez pas l’architecture SLB (équilibreur de charge standard), vous devez configurer une sortie explicite. Cette architecture requiert l’envoi explicite du trafic sortant vers une appliance comme un pare-feu, une passerelle ou un proxy, ou pour permettre à la traduction d’adresses réseau (NAT) d’être effectuée par une adresse IP publique attribuée à l’équilibreur de charge standard ou à l’appliance.

#### <a name="load-balancer-creation-with-userdefinedrouting"></a>Création d’un équilibreur de charge avec userDefinedRouting

Les clusters AKS avec un type sortant de UDR reçoivent un équilibreur de charge standard (SLB) uniquement lorsque le premier service Kubernetes de type « loadBalancer » est déployé. L’équilibrage de charge est configuré avec une adresse IP publique pour les demandes *entrantes* et un pool principal pour les demandes *sortantes*. Les règles de trafic entrant sont configurées par le fournisseur cloud Azure, mais **aucune adresse IP publique sortante ou des règles de trafic sortant** sont configurées en raison d’un type d’UDR sortant. Votre UDR sera toujours la seule source de trafic sortant.

Les équilibreurs de charge Azure [n’entraînent pas de frais tant qu’une règle n’a pas été mise en place](https://azure.microsoft.com/pricing/details/load-balancer/).

## <a name="deploy-a-cluster-with-outbound-type-of-udr-and-azure-firewall"></a>Déployer un cluster avec le type de sortie UDR et le pare-feu Azure

Pour illustrer l’application d’un cluster avec un type de sortie utilisant une route définie par l’utilisateur, un cluster peut être configuré sur un réseau virtuel dont le sous-réseau comporte un pare-feu Azure. Consultez cet exemple de [limitation du trafic sortant avec le pare-feu Azure](limit-egress-traffic.md#restrict-egress-traffic-using-azure-firewall).

> [!IMPORTANT]
> Le type sortant d’UDR requiert une route pour 0.0.0.0/0 et la destination de tronçon suivant de l’appliance virtuelle réseau (NVA) dans la table de routage.
> La table de route a déjà une valeur par défaut 0.0.0.0/0 vers Internet, sans adresse IP publique vers SNAT. Le simple ajout de cette route ne vous fournira pas de sortie. AKS vérifiera que la route 0.0.0.0/0 que vous créez ne pointe pas vers Internet, mais vers une appliance réseau virtuelle, une passerelle, etc. Quand vous utilisez un type sortant d’UDR, aucune adresse IP publique d’équilibreur de charge pour les **demandes entrantes** n’est créée, sauf si un service de type *loadbalancer* est configuré. Une adresse IP publique pour les **demandes sortantes** n’est jamais créée par AKS si un type sortant d’UDR est défini.

## <a name="next-steps"></a>Étapes suivantes

Consultez [Vue d’ensemble des routes définies par l’utilisateur des réseaux Azure](../virtual-network/virtual-networks-udr-overview.md).

Consultez [Guide pratique pour créer, modifier ou supprimer une table de routage](../virtual-network/manage-route-table.md).

<!-- LINKS - internal -->
[az-aks-get-credentials]: /cli/azure/aks?view=azure-cli-latest#az-aks-get-credentials
[byo-route-table]: configure-kubenet.md#bring-your-own-subnet-and-route-table-with-kubenet
