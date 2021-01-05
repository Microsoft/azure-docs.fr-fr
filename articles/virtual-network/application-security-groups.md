---
title: Présentation des groupes de sécurité d’application Azure
titlesuffix: Azure Virtual Network
description: En savoir plus sur l’utilisation des groupes de sécurité d’application.
services: virtual-network
documentationcenter: na
author: KumudD
ms.service: virtual-network
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/27/2020
ms.author: kumud
ms.reviewer: kumud
ms.openlocfilehash: 3542ae2e94c2fa3d3e9d6100738b2aabded94d15
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96005263"
---
# <a name="application-security-groups"></a>Groupes de sécurité d’application

Les groupes de sécurité d’application permettent de configurer la sécurité réseau comme un prolongement naturel de la structure de l’application, et donc de regrouper les machines virtuelles et définir des stratégies de sécurité réseau basés sur ces groupes. Vous pouvez réutiliser votre stratégie de sécurité à grande échelle sans maintenance manuelle d’adresses IP explicites. La plateforme gère la complexité des adresses IP explicites et plusieurs ensembles de règles, ce qui vous permet de vous concentrer sur la logique métier. Pour mieux comprendre les groupes de sécurité d’application, prenons l’exemple suivant :

![Groupes de sécurité d’application](./media/security-groups/application-security-groups.png)

Dans l’image précédente, *NIC1* et *NIC2* sont membres du groupe de sécurité d’application *AsgWeb*. *NIC3* est un membre du groupe de sécurité d’application *AsgLogic*. *NIC4* est un membre du groupe de sécurité d’application *AsgDb*. Bien que chaque interface réseau dans cet exemple soit membre d’un seul groupe de sécurité réseau, une interface réseau peut être membre de plusieurs groupes de sécurité d’application, jusqu’aux [limites Azure](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits). Aucune de ces interfaces réseau ne dispose d’un groupe de sécurité réseau associé. *NSG1* est associé aux deux sous-réseaux et contient les règles suivantes :

## <a name="allow-http-inbound-internet"></a>Allow-HTTP-Inbound-Internet

Cette règle est nécessaire pour autoriser le trafic internet vers les serveurs web. Étant donné que le trafic entrant à partir d’internet est refusé par la règle de sécurité par défaut **DenyAllInbound**, aucune règle supplémentaire n’est nécessaire pour les groupes de sécurité d’application *AsgLogic* ou *AsgDb*.

|Priority|Source|Ports source| Destination | Ports de destination | Protocol | Accès |
|---|---|---|---|---|---|---|
| 100 | Internet | * | AsgWeb | 80 | TCP | Allow |

## <a name="deny-database-all"></a>Deny-Database-All

Étant donné que la règle de sécurité par défaut **AllowVNetInBound** autorise toutes les communications entre les ressources dans le même réseau virtuel, cette règle est nécessaire pour refuser le trafic à partir de toutes les ressources.

|Priority|Source|Ports source| Destination | Ports de destination | Protocol | Accès |
|---|---|---|---|---|---|---|
| 120 | * | * | AsgDb | 1433 | Quelconque | Deny |

## <a name="allow-database-businesslogic"></a>Allow-Database-BusinessLogic

Cette règle autorise le trafic du groupe de sécurité d’application *AsgLogic* vers le groupe de sécurité d’application *AsgDb*. Cette règle est prioritaire par rapport à la règle *Deny-Database-All*. Par conséquent, cette règle est traitée avant la règle *Deny-Database-All*, c’est pourquoi le trafic en provenance du groupe de sécurité d’application *AsgLogic* est autorisé, tandis que tout autre trafic est bloqué.

|Priority|Source|Ports source| Destination | Ports de destination | Protocol | Accès |
|---|---|---|---|---|---|---|
| 110 | AsgLogic | * | AsgDb | 1433 | TCP | Allow |

Les règles spécifiant un groupe de sécurité d’application en tant que source ou destination sont appliquées uniquement aux interfaces réseau qui sont membres du groupe de sécurité d’application. Si l’interface réseau n’est pas membre d’un groupe de sécurité d’application, la règle n’est pas appliquée à l’interface réseau, même si le groupe de sécurité réseau est associé au sous-réseau.

Les groupes de sécurité d’application ont les contraintes suivantes :

-    Le nombre de groupes de sécurité d’application que vous pouvez avoir dans un abonnement, ainsi que d’autres paramètres relatifs aux groupes de sécurité d’application, sont limités. Pour plus d’informations, consultez [limites Azure](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits).
- Vous pouvez spécifier un groupe de sécurité d’application en tant que source et destination dans une règle de sécurité. Vous ne pouvez pas spécifier plusieurs groupes de sécurité d’application dans la source ou la destination.
- Toutes les interfaces réseau affectées à un groupe de sécurité d’application doivent exister dans le même réseau virtuel que celui où se trouve la première interface réseau affectée au groupe de sécurité d’application. Par exemple, si la première interface réseau assignée à un groupe de sécurité d’application nommé *AsgWeb* se trouve dans le réseau virtuel nommé *VNet1*, toutes les interfaces réseau suivantes affectées à *AsgWeb* doivent exister dans *VNet1*. Vous ne pouvez pas ajouter d’interfaces réseau à partir de différents réseaux virtuels au même groupe de sécurité d’application.
- Si vous spécifiez un groupe de sécurité d’application en tant que source et destination dans une règle de sécurité, les interfaces réseau dans les deux groupes de sécurité d’application doivent se trouver dans le même réseau virtuel. Par exemple, si *AsgLogic* contient des interfaces réseau de *VNet1*, et si *AsgDb* contient des interfaces réseau de *VNet2*, vous ne pouvez pas assigner *AsgLogic* en tant que source et *AsgDb* en tant que destination dans une règle. Toutes les interfaces réseau pour les groupes de sécurité d’application source et destination doivent exister dans le même réseau virtuel.

> [!TIP]
> Pour réduire le nombre de règles de sécurité dont vous avez besoin et la nécessité de modifier les règles, planifiez les groupes de sécurité d’application dont vous avez besoin et créez des règles à l’aide de balises de service ou des groupes de sécurité d’application, plutôt que des adresses IP individuelles ou des plages d’adresses IP, chaque fois que c’est possible.

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment [créer des groupes de sécurité réseau](tutorial-filter-network-traffic.md).
