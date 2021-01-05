---
title: Nouveautés d’Azure Load Balancer
description: Découvrez les nouveautés d’Azure Load Balancer, notamment les dernières notes de publication, les problèmes connus, les corrections de bogues, les fonctionnalités dépréciées et les changements à venir.
services: load-balancer
author: anavinahar
ms.service: load-balancer
ms.topic: conceptual
ms.date: 07/07/2020
ms.author: anavin
ms.openlocfilehash: a30a42e8a8c4049b53274da512089dd29965e775
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96573151"
---
# <a name="whats-new-in-azure-load-balancer"></a>Quelles sont les nouveautés d’Azure Load Balancer ?

Azure Load Balancer est mis à jour régulièrement. Restez à jour avec les dernières annonces. Cet article vous fournit des informations sur :

- Versions les plus récentes
- Problèmes connus
- Résolution des bogues
- Fonctionnalités dépréciées (le cas échéant)

Vous pouvez également rechercher les dernières mises à jour d’Azure Load Balancer et vous abonner au flux RSS [ici](https://azure.microsoft.com/updates/?category=networking&query=load%20balancer).

## <a name="recent-releases"></a>Dernières mises en production

| Type |Nom |Description  |Date de l’ajout  |
| ------ |---------|---------|---------|
| Fonctionnalité | Support pour les déplacements de groupes de ressources | Prise en charge de Standard Load Balancer et des adresses IP publiques standard pour les [déplacements de groupes de ressources](https://azure.microsoft.com/updates/standard-resource-group-move/). | Octobre 2020 |
| Fonctionnalité | Prise en charge de la gestion des pools de back-ends basés sur IP (préversion) | Azure Load Balancer prend en charge l’ajout et la suppression de ressources d’un pool de back-ends par le biais des adresses IPv4 ou IPv6. Cela facilite la gestion des conteneurs, des machines virtuelles et des groupes de machines virtuelles identiques associés à Load Balancer. Cela permet également de réserver des adresses IP dans le cadre d’un pool de back-ends avant la création des ressources associées. Apprenez-en davantage [ici](backend-pool-management.md).|Juillet 2020 |
| Fonctionnalité| Insights Azure Load Balancer à l’aide d’Azure Monitor | Créées dans le cadre d’Azure Monitor pour réseaux, les clients disposent désormais de cartes topologiques pour toutes leurs configurations Load Balancer et de tableaux de bord d’intégrité pour leurs équilibreurs Standard Load Balancers préconfigurés avec des métriques dans le portail Azure. [Bien démarrer et en savoir plus](https://azure.microsoft.com/blog/introducing-azure-load-balancer-insights-using-azure-monitor-for-networks/) | Juin 2020 |
| Validation | Ajout de la validation pour les ports haute disponibilité | Une validation a été ajoutée pour s’assurer que les règles de ports haute disponibilité et les règles de ports sans haute disponibilité ne sont configurables que lorsque l’adresse IP flottante est activée. Auparavant, la configuration pouvait s’effectuer, mais ne fonctionnait pas comme prévu. Aucune modification de fonctionnalité n’a été apportée. Vous pouvez en savoir plus [ici](load-balancer-ha-ports-overview.md#limitations).| Juin 2020 |
| Fonctionnalité| Prise en charge d’ipv6 pour Azure Load Balancer (en disponibilité générale) | Vous pouvez avoir des adresses IPv6 en tant que front-end pour vos Azure Load Balancers. Découvrez comment [créer une application à double pile ici](../virtual-network/virtual-network-ipv4-ipv6-dual-stack-standard-load-balancer-powershell.md). |Avril 2020|
| Fonctionnalité| Réinitialisations TCP lors de l’expiration du délai d’inactivité (en disponibilité générale)| Utilisez les réinitialisations TCP pour créer un comportement d’application plus prévisible. [En savoir plus](load-balancer-tcp-reset.md)| Février 2020 |

## <a name="known-issues"></a>Problèmes connus

Le groupe du produit travaille activement sur la résolution des problèmes connus suivants :

|Problème |Description  |Limitation des risques  |
| ---------- |---------|---------|
| Événement d’alerte de l’équilibreur de charge et journaux d’état de la sonde d’intégrité | La journalisation ne fonctionne pas pour les événements d’alerte dans le cas de l’équilibreur de charge De base ou Standard ni pour les journaux d’état de la sonde d’intégrité pour l’équilibreur de charge De base  | [Utilisez Azure Monitor pour les métriques multidimensionnelles de votre équilibreur de charge standard](load-balancer-standard-diagnostics.md). Azure Monitor propose une visualisation pour un ensemble étendu de métriques multidimensionnelles qui peuvent aussi être exportées sous forme de journaux. Vous pouvez tirer parti du tableau de bord des métriques préconfigurées via le sous-panneau Insights de votre équilibreur de charge. Si vous utilisez un équilibreur de charge de base, [effectuez une mise à niveau vers un équilibreur de charge standard](upgrade-basic-standard.md) pour une supervision des métriques au niveau production.

  

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur Azure Load Balancer, consultez [Qu’est-ce qu’Azure Load Balancer ?](load-balancer-overview.md) et les [Questions fréquentes (FAQ)](load-balancer-faqs.md).
