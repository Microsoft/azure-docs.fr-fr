---
title: Groupes de sécurité réseau avec Azure Site Recovery | Microsoft Docs
description: Décrit comment utiliser les groupes de sécurité réseau avec Azure Site Recovery pour la reprise après sinistre et la migration
author: Harsha-CS
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 04/08/2019
ms.author: harshacs
ms.openlocfilehash: 367aba09f84da1e227c08721077aa1b2132a62bf
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92367971"
---
# <a name="network-security-groups-with-azure-site-recovery"></a>Groupes de sécurité réseau avec Azure Site Recovery

Vous pouvez utiliser les groupes de sécurité réseau pour limiter le trafic réseau vers les ressources d’un réseau virtuel. Un [groupe de sécurité réseau (NSG)](../virtual-network/network-security-groups-overview.md#network-security-groups) contient une liste de règles de sécurité qui autorisent ou refusent le trafic réseau entrant ou sortant en fonction de l’adresse IP source ou de destination, du port et du protocole.

Sous le modèle de déploiement Resource Manager, les groupes de sécurité réseau peuvent être associés à des sous-réseaux ou à des interfaces réseau individuelles. Lorsqu’un NSG est associé à un sous-réseau, les règles s’appliquent à toutes les ressources connectées au sous-réseau. Le trafic peut être limité davantage en associant aussi un groupe NSG à des interfaces réseau individuelles au sein d’un sous-réseau qui a déjà un groupe NSG associé.

Cet article décrit comment utiliser des groupes de sécurité réseau avec Azure Site Recovery.

## <a name="using-network-security-groups"></a>Utilisation de groupes de sécurité réseau

Un sous-réseau individuel peut avoir zéro ou un groupe NSG associé. Une interface réseau individuelle peut avoir zéro ou un groupe NSG associé. Vous pouvez donc en fait obtenir une double restriction du trafic pour une machine virtuelle en associant d’abord un groupe NSG à un sous-réseau, puis un autre groupe NSG à l’interface réseau de la machine virtuelle. L’application des règles NSG dépend dans ce cas du sens du trafic et de la priorité des règles de sécurité appliquées.

Prenons un exemple simple avec une machine virtuelle comme suit :
-    La machine virtuelle est placée au sein du **sous-réseau Contoso** .
-    Le **sous-réseau Contoso** est associé au **groupe NSG du sous-réseau** .
-    L’interface réseau de la machine virtuelle est en plus associée au **groupe NSG de la machine virtuelle** .

![Groupe NSG avec Site Recovery](./media/concepts-network-security-group-with-site-recovery/site-recovery-with-network-security-group.png)

Dans cet exemple, pour le trafic entrant, le groupe NSG du sous-réseau est évalué en premier. Tout trafic autorisé via le groupe NSG du sous-réseau est ensuite évalué par le groupe NSG de la machine virtuelle. L’inverse s’applique pour le trafic sortant, à savoir que le groupe NSG de la machine virtuelle est évalué en premier. Tout trafic autorisé via le groupe NSG de la machine virtuelle est ensuite évalué par le groupe NSG du sous-réseau.

Vous pouvez ainsi appliquer des règles de sécurité avec précision. Par exemple, vous souhaiterez peut-être autoriser l’accès Internet entrant à quelques machines virtuelles d’application (par exemple, des machines virtuelles de frontend) dans un sous-réseau, mais restreindre l’accès Internet entrant aux autres machines virtuelles (par exemple, des machines virtuelles de base de données et autre backend). Dans ce cas, vous pouvez avoir une règle plus modérée sur le groupe NSG du sous-réseau, règle qui autorisera le trafic Internet et limitera l’accès à des machines virtuelles spécifiques en refusant l’accès sur le groupe NSG de la machine virtuelle. Vous pouvez faire de même pour le trafic sortant.

Lorsque vous définissez de telles configurations NSG, vérifiez que les priorités appliquées aux [règles de sécurité](../virtual-network/network-security-groups-overview.md#security-rules) sont les bonnes. Les règles sont traitées dans l’ordre croissant, car les nombres les plus faibles sont prioritaires. Une fois que le trafic correspond à une règle, le traitement s’arrête. Par conséquent, les règles avec des priorités plus faibles (des nombres plus élevés) et ayant les mêmes attributs que les règles de priorité supérieure ne sont pas traitées.

Vous ne pouvez pas toujours savoir si les groupes de sécurité réseau sont appliquées à la fois à une interface réseau et à un sous-réseau. Vous pouvez vérifier les règles d’agrégation appliquées à une interface réseau en consultant les [règles de sécurité en vigueur](../virtual-network/virtual-network-network-interface.md#view-effective-security-rules) de l’interface réseau. Vous pouvez aussi utiliser la fonctionnalité de [vérification du flux IP](../network-watcher/diagnose-vm-network-traffic-filtering-problem.md) dans [Azure Network Watcher](../network-watcher/network-watcher-monitoring-overview.md) pour déterminer si la communication est autorisée vers ou depuis une interface réseau. L’outil vous indique si la communication est autorisée, ainsi que la règle de sécurité réseau qui autorise ou refuse le trafic.

## <a name="on-premises-to-azure-replication-with-nsg"></a>Réplication d’éléments locaux sur Azure avec un groupe NSG

Azure Site Recovery permet une récupération d’urgence et une migration sur Azure de [machines virtuelles Hyper-V](hyper-v-azure-architecture.md), de [machines virtuelles VMware](vmware-azure-architecture.md) et de [serveurs physiques](physical-azure-architecture.md) locaux. Pour tous les scénarios de réplication de service local sur Azure, les données sont envoyées à un compte de stockage Azure et stockées sur celui-ci. Lors d’une réplication, vous ne payez aucuns frais de machine virtuelle. Quand vous exécutez un basculement vers Azure, Site Recovery crée automatiquement des machines virtuelles Azure IaaS.

Une fois que vous avez créé les machines virtuelles après le basculement vers Azure, vous pouvez utiliser les groupes NSG pour limiter le trafic réseau vers le réseau virtuel et les machines virtuelles. Site Recovery ne crée aucun groupe NSG pendant l’opération de basculement. Nous vous recommandons de créer les groupes NSG Azure nécessaires avant de lancer le basculement. Vous pouvez ensuite associer les groupes NSG aux machines virtuelles basculées automatiquement pendant le basculement en utilisant des scripts d’automatisation avec les [plans de récupération](site-recovery-create-recovery-plans.md) performants de Site Recovery.

Par exemple, si la configuration de machines virtuelles post-basculement est similaire à l’[exemple de scénario](concepts-network-security-group-with-site-recovery.md#using-network-security-groups) détaillé ci-dessus :
-    Vous pouvez créer un **réseau virtuel Contoso** et un **sous-réseau Contoso** lors de la planification d’une reprise après sinistre sur la région Azure cible.
-    You pouvez aussi créer et configurer un **groupe NSG de sous-réseau** et un **groupe NSG de machine virtuelle** lors de la même planification de reprise après sinistre.
-    Le **groupe NSG de sous-réseau** peut alors être immédiatement associé au **sous-réseau Contoso** , puisque le groupe NSG et le sous-réseau sont déjà disponibles.
-    Le **groupe NSG de machine virtuelle** peut être associé aux machines virtuelles pendant le basculement à l’aide de plans de récupération.

Une fois que les groupes NSG sont créés et configurés, nous vous recommandons d’exécuter un [test de basculement](site-recovery-test-failover-to-azure.md) pour vérifier les associations de groupe NSG scriptées et la connectivité des machines virtuelles post-basculement.

## <a name="azure-to-azure-replication-with-nsg"></a>Réplication d’Azure sur Azure avec les groupes NSG

Azure Site Recovery permet une récupération d’urgence de [machines virtuelles Azure](azure-to-azure-architecture.md). Quand la réplication est activée pour les machines virtuelles Azure, Site Recovery peut créer des réplicas de réseaux virtuels (notamment des sous-réseaux et des sous-réseaux de passerelle) sur la région cible et créer les mappages nécessaires entre les réseaux virtuels sources et cibles. Vous pouvez également précréer les sous-réseaux et réseaux cibles pour les utiliser quand la réplication est activée. Site Recovery ne crée aucune machine virtuelle dans la région Azure cible avant le [basculement](azure-to-azure-tutorial-failover-failback.md).

Pour la réplication des machines virtuelles Azure, vérifiez que les règles NSG sur la région Azure source autorisent une [connectivité sortante](azure-to-azure-about-networking.md#outbound-connectivity-using-service-tags)pour le trafic de réplication. Vous pouvez également tester et vérifier ces règles requises via cet [exemple de configuration NSG](azure-to-azure-about-networking.md#example-nsg-configuration).

Site Recovery ne crée ni ne réplique de groupe NSG pendant l’opération de basculement. Nous vous recommandons de créer les groupes NSG nécessaires dans la région Azure cible avant de lancer le basculement. Vous pouvez ensuite associer les groupes NSG aux machines virtuelles basculées automatiquement pendant le basculement en utilisant des scripts d’automatisation avec les [plans de récupération](site-recovery-create-recovery-plans.md) performants de Site Recovery.

Si nous prenons en compte l’[exemple de scénario](concepts-network-security-group-with-site-recovery.md#using-network-security-groups) décrit précédemment :
-    Site Recovery peut créer des réplicas du **réseau virtuel Contoso** et du **sous-réseau Contoso**  dans la région Azure cible quand la réplication est activée pour la machine virtuelle.
-    Vous pouvez créer les réplicas souhaités du **groupe NSG de sous-réseau** et du **groupe NSG de machine virtuelle** (nommés, par exemple, **Groupe NSG de sous-réseau cible** et **Groupe NSG de machine virtuelle cible** , respectivement) dans la région Azure cible, en autorisant toutes les règles supplémentaires nécessaires dans la région cible.
-    Le **groupe NSG de sous-réseau cible** peut alors être immédiatement associé au sous-réseau de la région cible, puisque le groupe NSG et le sous-réseau sont déjà disponibles.
-    Le **groupe NSG de machine virtuelle cible** peut être associé aux machines virtuelles pendant le basculement à l’aide de plans de récupération.

Une fois que les groupes NSG sont créés et configurés, nous vous recommandons d’exécuter un [test de basculement](azure-to-azure-tutorial-dr-drill.md) pour vérifier les associations de groupe NSG scriptées et la connectivité des machines virtuelles post-basculement.

## <a name="next-steps"></a>Étapes suivantes
-    En savoir plus sur les [groupes de sécurité réseau](../virtual-network/network-security-groups-overview.md#network-security-groups).
-    Découvrez plus en détail les [règles de sécurité](../virtual-network/network-security-groups-overview.md#security-rules) des groupes NSG.
-    Découvrez plus en détail les [règles de sécurité effectives](../virtual-network/diagnose-network-traffic-filter-problem.md) d’un groupe NSG.
-    En savoir plus sur les [plans de récupération](site-recovery-create-recovery-plans.md) pour automatiser le basculement d’application