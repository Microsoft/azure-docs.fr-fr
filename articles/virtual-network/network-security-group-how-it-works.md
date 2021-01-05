---
title: Groupe de sécurité réseau – Fonctionnement
titlesuffix: Azure Virtual Network
description: En savoir plus sur la façon dont les groupes de sécurité réseau vous permettent de filtrer le trafic entre les ressources Azure.
services: virtual-network
documentationcenter: na
author: KumudD
ms.service: virtual-network
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/24/2020
ms.author: kumud
ms.reviewer: kumud
ms.openlocfilehash: e60cdfb00d0dc9d446bd52a72e9fd15676acd285
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89458193"
---
# <a name="how-network-security-groups-filter-network-traffic"></a>Façon dont les groupes de sécurité réseau filtrent le trafic
<a name="network-security-groups"></a>

Vous pouvez utiliser un groupe de sécurité réseau Azure pour filtrer le trafic réseau à destination et en provenance des ressources Azure dans un réseau virtuel Azure. Un groupe de sécurité réseau contient des [règles de sécurité](https://docs.microsoft.com/azure/virtual-network/security-overview#security-rules) qui autorisent ou rejettent le trafic réseau entrant et sortant vers différents types de ressources Azure. Pour chaque règle, vous pouvez spécifier la source et la destination, le port et le protocole.

Vous pouvez déployer des ressources à partir de plusieurs services Azure dans un réseau virtuel Azure. Pour une liste complète, consultez [Services pouvant être déployés dans un réseau virtuel](virtual-network-for-azure-services.md#services-that-can-be-deployed-into-a-virtual-network). Vous pouvez associer zéro ou un groupe de sécurité réseau à chaque [sous-réseau](virtual-network-manage-subnet.md#change-subnet-settings) de réseau virtuel et [interface réseau](virtual-network-network-interface.md#associate-or-dissociate-a-network-security-group) dans une machine virtuelle. Vous pouvez associer le même groupe de sécurité réseau à autant d’interfaces réseau individuelles et autant de sous-réseaux que vous le souhaitez.

L’image suivante illustre les différents scénarios de déploiement des groupes de sécurité réseau afin d’autoriser le trafic réseau vers et à partir d’internet via le port TCP 80 :

![NSG-processing](./media/network-security-group-how-it-works/network-security-group-interaction.png)

Référez-vous à l’image précédente ainsi qu’au texte suivant pour comprendre comment Azure traite les règles entrantes et sortantes pour les groupes de sécurité réseau :

## <a name="inbound-traffic"></a>Trafic entrant

Pour le trafic entrant, Azure traite d’abord les règles dans un groupe de sécurité réseau associées à un sous-réseau, si elles existent, puis les règles dans un groupe de sécurité réseau associées à l’interface réseau, si elles existent.

- **VM1** : Les règles de la sécurité dans *NSG1* sont traitées dans la mesure où elles sont associées à *Subnet1* et *VM1* est dans *Subnet1*. Sauf si vous avez créé une règle autorisant le trafic entrant par le port 80, le trafic est refusé par la règle de sécurité par défaut [DenyAllInbound](https://docs.microsoft.com/azure/virtual-network/security-overview#denyallinbound) et n’est jamais évalué par *NSG2*, étant donné que *NSG2* est associé à l’interface réseau. Si *NSG1* dispose d’une règle de sécurité qui autorise le trafic entrant par le port 80, le trafic est ensuite traité par *NSG2*. Pour autoriser le port 80 vers la machine virtuelle, *NSG1* et *NSG2* doivent disposer tous deux d’une règle autorisant l’accès au port 80 à partir d’Internet.
- **VM2** : Les règles dans *NSG1* sont traitées étant donné que *VM2* se trouve également dans *Subnet1*. Dans la mesure où *VM2* ne dispose pas d’un groupe de sécurité réseau associé à son interface réseau, elle reçoit tout le trafic autorisé via *NSG1* ou se voit refuser tout le trafic refusé par *NSG1*. Le trafic est soit autorisé, soit refusé à toutes les ressources dans le même sous-réseau lorsqu’un groupe de sécurité réseau est associé à un sous-réseau.
- **VM3** : Dans la mesure où il n’existe aucun groupe de sécurité réseau associé à *Subnet2*, le trafic est autorisé dans le sous-réseau et traité par *NSG2*, car *NSG2* est associé à l’interface réseau attachée à *VM3*.
- **VM4** : Le trafic est autorisé vers *VM4*, car aucun groupe de sécurité réseau n’est associé à *Subnet3* ou à l’interface réseau dans la machine virtuelle. L’intégralité du trafic réseau est autorisée via une interface réseau et un sous-réseau si aucun groupe de sécurité réseau leur est associé.

## <a name="outbound-traffic"></a>Trafic sortant

Pour le trafic sortant, Azure traite d’abord les règles dans un groupe de sécurité réseau associées à une interface réseau, si elles existent, puis les règles dans un groupe de sécurité réseau associées au sous-réseau, si elles existent.

- **VM1** : Les règles de sécurité dans *NSG2* sont traitées. Sauf si vous créez une règle de sécurité qui refuse le trafic sortant vers internet au niveau du port 80, le trafic est autorisé par la règle de sécurité par défaut [AllowInternetOutbound](https://docs.microsoft.com/azure/virtual-network/security-overview#allowinternetoutbound) à la fois dans *NSG1* et *NSG2*. Si *NSG2* présente une règle de sécurité qui refuse le trafic par le port 80, le trafic est refusé et n’est jamais évalué par *NSG1*. Pour refuser le port 80 à partir de la machine virtuelle, l’un au moins des groupes de sécurité réseau doit disposer d’une règle qui refuse l’accès à Internet par le port 80.
- **VM2** : L’ensemble du trafic est envoyé au sous-réseau via l’interface réseau puisque l’interface réseau attachée à *VM2* ne dispose pas d’un groupe de sécurité réseau associé. Les règles dans *NSG1* sont traitées.
- **VM3** : Si *NSG2* présente une règle de sécurité qui refuse le trafic par le port 80, le trafic est refusé. Si *NSG2* a une règle de sécurité qui autorise le trafic par le port 80, le port 80 autorise le trafic sortant vers internet, étant donné qu’un groupe de sécurité réseau n’est pas associé à *Subnet2*.
- **VM4** : L’ensemble du trafic réseau est autorisé depuis *VM4*, car aucun groupe de sécurité réseau n’est associé à *Subnet3* ou à l’interface réseau associée à la machine virtuelle.


## <a name="intra-subnet-traffic"></a>Trafic intra-sous-réseau

Il est important de noter que les règles de sécurité dans un groupe de sécurité réseau (NSG) associé à un sous-réseau peuvent perturber la connectivité entre les machines virtuelles (VM) qui le composent. Par exemple, si une règle est ajoutée à *NSG1* qui refuse tout le trafic entrant et sortant, *VM1* et *VM2* ne seront plus en mesure de communiquer entre elles. Une autre règle doit être ajoutée spécifiquement pour l’autoriser. 

Vous pouvez facilement afficher des règles d’agrégation appliquées à une interface réseau en consultant les [règles de sécurité efficaces](virtual-network-network-interface.md#view-effective-security-rules) relatives à une interface réseau. Vous pouvez également utiliser la fonctionnalité de [vérification du flux IP](../network-watcher/diagnose-vm-network-traffic-filtering-problem.md?toc=%2fazure%2fvirtual-network%2ftoc.json) dans Azure Network Watcher pour déterminer si la communication est autorisée vers ou à partir d’une interface réseau. La vérification du flux IP vous indique si la communication est autorisée ou refusée, ainsi que la règle de sécurité réseau qui autorise ou refuse le trafic.

> [!NOTE]
> Les groupes de sécurité réseau sont associés à des sous-réseaux ou à des machines virtuelles et services cloud déployés dans le modèle de déploiement classique, et à des sous-réseaux ou des interfaces réseau dans le modèle de déploiement Resource Manager. Pour en savoir plus sur les modèles de déploiement Azure, consultez l’article [Déploiement Azure Resource Manager et déploiement classique : comprendre les modèles de déploiement et l’état de vos ressources](../azure-resource-manager/management/deployment-models.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

> [!TIP]
> Sauf si vous avez une raison particulière, nous vous recommandons d’associer un groupe de sécurité réseau à un sous-réseau ou à une interface réseau, mais pas aux deux. Dans la mesure où les règles d’un groupe de sécurité réseau associées à un sous-réseau peut entrer en conflit avec des règles d’un groupe de sécurité réseau associées à une interface réseau, vous pourrez subir des problèmes de communication inattendus qui nécessiteront une intervention.

## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus sur les ressources Azure pouvant être déployées dans un réseau virtuel et auxquelles des groupes de sécurité réseau peuvent être associées, consultez [Intégration d’un réseau virtuel pour les services Azure](virtual-network-for-azure-services.md).
* Si vous n’avez jamais créé un groupe de sécurité réseau, vous pouvez suivre un [didacticiel](tutorial-filter-network-traffic.md) rapide pour vous entraîner.
* Si vous êtes familier avec les groupes de sécurité réseau et que vous devez en gérer un, consultez [Gérer un groupe de sécurité réseau](manage-network-security-group.md). 
* Si vous rencontrez des problèmes de communication et que vous avez besoin résoudre les problèmes de groupes de sécurité réseau, consultez [Diagnostiquer un problème de filtre de trafic réseau sur une machine virtuelle](diagnose-network-traffic-filter-problem.md). 
* Découvrez comment activer les [journaux de flux de groupe de sécurité réseau](../network-watcher/network-watcher-nsg-flow-logging-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json) pour analyser le trafic réseau vers et à partir des ressources auxquelles un groupe de sécurité réseau est associé.
