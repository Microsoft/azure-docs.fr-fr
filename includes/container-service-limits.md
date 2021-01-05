---
title: Fichier include
description: Fichier include
services: container-service
author: mlearned
ms.service: container-service
ms.topic: include
ms.date: 11/22/2019
ms.author: mlearned
ms.custom: include file
ms.openlocfilehash: a42bba1b6524825aa571e4c18319b61b97829792
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96584543"
---
| Ressource | Limite |
| --- | :--- |
| Clusters maximum par abonnement | 1 000 |
| Nombre maximal de nœuds par cluster avec les groupes à haute disponibilité de machines virtuelles et la référence SKU Standard Load Balancer  | 100 |
| Nombre maximal de nœuds par cluster avec Virtual Machine Scale Sets et la référence [SKU Standard Load Balancer][standard-load-balancer] | 1 000 (100 nœuds par [pool de nœuds][node-pool]) |
| Nombre maximal de pods par nœud : [réseau de base][basic-networking] avec Kubenet | 110 |
| Nombre maximal de pods par nœud : [mise en réseau avancée][advanced-networking] avec Azure Container Networking Interface | Déploiement de l’interface Azure CLI : 30<sup>1</sup><br />Modèle Azure Resource Manager : 30<sup>1</sup><br />Déploiement du portail : 30 |

<sup>1</sup> Quand vous déployez un cluster AKS avec l’interface de ligne de commande Azure ou un modèle Resource Manager, cette valeur est configurable jusqu’à 250 pods par nœud. Vous ne pouvez pas configurer le nombre maximum de pods par nœud après avoir déployé un cluster AKS ou si vous avez déployé un cluster à l’aide du Portail Azure.<br />

<!-- LINKS - Internal -->
[basic-networking]: ../articles/aks/concepts-network.md#kubenet-basic-networking
[advanced-networking]: ../articles/aks/concepts-network.md#azure-cni-advanced-networking
[standard-load-balancer]: ../articles/load-balancer/load-balancer-overview.md
[node-pool]: ../articles/aks/use-multiple-node-pools.md

<!-- LINKS - External -->
[azure-support]: https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest