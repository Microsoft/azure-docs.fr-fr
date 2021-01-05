---
title: Qu’est-ce qu’un hub virtuel sécurisé ?
description: Découvrir des informations sur les hubs virtuels sécurisés
author: vhorne
ms.service: firewall-manager
services: firewall-manager
ms.topic: conceptual
ms.date: 10/12/2020
ms.author: victorh
ms.openlocfilehash: c840bf9e82b8dcdb1fbf9b380ea847b3d1b08dd9
ms.sourcegitcommit: a2d8acc1b0bf4fba90bfed9241b299dc35753ee6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/12/2020
ms.locfileid: "91948066"
---
# <a name="what-is-a-secured-virtual-hub"></a>Qu’est-ce qu’un hub virtuel sécurisé ?

Un hub virtuel est un réseau virtuel géré par Microsoft qui permet la connectivité à partir d’autres ressources. Lorsqu’un hub virtuel est créé à partir d’un Virtual WAN dans le Portail Azure, un réseau virtuel et des passerelles (facultatif) de hub virtuel sont créés en tant que composants.

Un hub virtuel *sécurisé* est un [hub Azure Virtual WAN](../virtual-wan/virtual-wan-about.md#resources) avec des stratégies de sécurité et de routage associées, configurées par Azure Firewall Manager. Utilisez des hubs virtuels sécurisés pour créer facilement des architectures transitives et de réseau en étoile avec des services de sécurité natifs pour la gouvernance et la protection du trafic. 

Vous pouvez utiliser un hub virtuel sécurisé pour filtrer le trafic entre des réseaux virtuels (V2V), des réseaux virtuels et succursales (B2V) et le trafic vers Internet (B2I/V2I). Un hub virtuel sécurisé fournit un routage automatisé. Il n’est pas nécessaire de configurer vos propres UDR (itinéraires définis par l’utilisateur) pour acheminer le trafic via votre pare-feu.

Vous pouvez choisir les fournisseurs de sécurité nécessaires pour protéger et gérer votre trafic réseau, notamment le Pare-feu Azure, des fournisseurs SECaaS (Security as a service) tiers, ou les deux. Actuellement, un hub sécurisé ne prend pas en charge le filtrage de branche à branche (B2B) ni le filtrage entre plusieurs hubs. Pour en savoir plus, consultez [Qu’est-ce qu’Azure Firewall Manager ?](overview.md#known-issues). 

## <a name="create-a-secured-virtual-hub"></a>Créer un hub virtuel sécurisé

À l’aide de Firewall Manager dans le Portail Azure, vous pouvez créer un nouveau hub virtuel sécurisé ou convertir un hub virtuel existant que vous avez créé précédemment à l’aide d’Azure Virtual WAN.

## <a name="next-steps"></a>Étapes suivantes

Pour créer un hub virtuel sécurisé et l’utiliser pour sécuriser et gérer un réseau en étoile, consultez le tutoriel [ : Sécuriser votre réseau cloud avec Azure Firewall Manager à l’aide du Portail Azure](secure-cloud-network.md).