---
title: Peering de réseaux virtuels Azure
titlesuffix: Azure Virtual Network
description: Découvrez l’appairage de réseaux virtuels dans Azure, notamment la façon dont il vous permet de connecter des réseaux dans Réseau virtuel Microsoft Azure.
services: virtual-network
documentationcenter: na
author: altambaw
ms.service: virtual-network
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 11/15/2019
ms.author: kumud
ms.openlocfilehash: 61617777efa8241c93b2b5ffe42fc1d914b6ebcf
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96004974"
---
# <a name="virtual-network-peering"></a>Peering de réseau virtuel

L'appairage de réseaux virtuels vous permet de connecter en toute transparence deux ou plusieurs [réseaux virtuels](virtual-networks-overview.md) dans Azure. Les réseaux virtuels apparaissent comme un seul réseau à des fins de connectivité. Le trafic entre les machines virtuelles des réseaux virtuels appairés utilise l'infrastructure principale de Microsoft. À l’instar du trafic entre les machines virtuelles du même réseau, le trafic est acheminé via le réseau *privé* de Microsoft uniquement.

Azure prend en charge les types de Peering suivants :

* Appairage de réseaux virtuels : connecte des réseaux virtuels au sein d’une même région Azure.
* Appairage de réseaux virtuels mondiaux : connecte des réseaux virtuels entre les différentes régions Azure.

Voici quelques-uns des avantages du peering de réseaux virtuels, qu’il soit local ou global :

* Connexion à latence faible et haut débit entre les ressources de différents réseaux virtuels.
* Possibilité pour les ressources d’un réseau virtuel de communiquer avec celles d’un autre réseau virtuel.
* Possibilité de transférer des données entre des réseaux virtuels dans des abonnements Azure, des locataires Azure Active Directory, des modèles de déploiement et des régions Azure.
* Possibilité d’appairer des réseaux virtuels créés via Azure Resource Manager.
* Possibilité d’appairer un réseau virtuel créé via Resource Manager à un réseau créé par le biais du modèle de déploiement classique. Pour en savoir plus sur les modèles de déploiement Azure, consultez l’article [Déploiement Azure Resource Manager et déploiement classique : comprendre les modèles de déploiement et l’état de vos ressources](../azure-resource-manager/management/deployment-models.md?toc=%2fazure%2fvirtual-network%2ftoc.json).
* Aucune interruption des ressources dans un réseau virtuel lors de la création du peering ou après que le peering est créé.

Le trafic réseau entre les réseaux virtuels homologués est privé. Le trafic entre les réseaux virtuels reste sur le réseau principal de Microsoft. Aucun chiffrement et aucune connexion Internet publique, ni passerelle ne sont nécessaires pour que les réseau virtuels communiquent.

## <a name="connectivity"></a>Connectivité

Pour les réseaux virtuels appairés, les ressources de l’un peuvent se connecter directement à celles du réseau virtuel appairé.

La latence du réseau entre des machines virtuelles de réseaux virtuels homologués dans la même région est la même que celle d’un seul réseau virtuel. Le débit du réseau repose sur la bande passante autorisée pour la machine virtuelle proportionnellement à sa taille. Aucune restriction de bande passante supplémentaire n’est appliquée au sein du peering.

Le trafic entre les machines virtuelles dans des réseaux virtuels homologués est acheminé directement via l’infrastructure principale de Microsoft et non via une passerelle ou une connexion Internet publique.

Vous pouvez appliquer des groupes de sécurité réseau dans l’un ou l’autre des réseaux virtuels pour bloquer l’accès à d’autres réseaux virtuels ou sous-réseaux.
Lors de la configuration de l’appairage de réseaux virtuels, vous pouvez ouvrir ou fermer les règles du groupe de sécurité réseau entre les réseaux virtuels. Si vous ouvrez totalement la connectivité entre les réseaux virtuels appairés, vous pouvez appliquer des groupes de sécurité réseau pour bloquer ou refuser certains accès. La connectivité entièrement ouverte est l’option par défaut. Pour en savoir plus sur les groupes de sécurité réseau, voir [Groupes de sécurité](security-overview.md).

## <a name="service-chaining"></a>Chaînage de services

Le chaînage de services vous permet de diriger le trafic d’un réseau virtuel vers l’appliance virtuelle ou la passerelle d’un réseau appairé via des itinéraires définis par l’utilisateur.

Pour activer le chaînage de services, configurez des itinéraires définis par l’utilisateur qui pointent vers des machines virtuelles de réseaux virtuels appairés en tant qu’adresse IP du *tronçon suivant*. Les itinéraires définis par l’utilisateur peuvent également pointer vers des passerelles de réseau virtuel pour activer le chaînage de services.

Vous pouvez également déployer des réseaux de type hub-and-spoke où le réseau virtuel hub héberge des composants d’infrastructure tels qu’une appliance virtuelle réseau ou une passerelle VPN. Tous les réseaux virtuels spoke peuvent ensuite être homologués avec le réseau virtuel hub. Le trafic transite par des appliances virtuelles réseau ou des passerelles VPN sur le réseau virtuel hub.

Le peering de réseaux virtuels permet de définir le tronçon suivant dans un itinéraire défini par l’utilisateur sur l’adresse IP d’une machine virtuelle du réseau virtuel appairé ou une passerelle VPN. Vous ne pouvez pas définir d’itinéraires entre plusieurs réseaux virtuels avec un itinéraire défini par l’utilisateur qui spécifie une passerelle Azure ExpressRoute comme type de tronçon suivant. Pour en savoir plus sur les routages définis par l’utilisateur, voir [Vue d’ensemble des routages définis par l’utilisateur](virtual-networks-udr-overview.md#user-defined). Pour découvrir comment créer une topologie de réseau hub-and-spoke, consultez [Implémenter une topologie de réseau hub-and-spoke dans Azure](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke?toc=%2fazure%2fvirtual-network%2ftoc.json).

## <a name="gateways-and-on-premises-connectivity"></a>Passerelles et connectivité locale

Chaque réseau virtuel, y compris un réseau virtuel appairé, peut avoir sa propre passerelle. Un réseau virtuel peut utiliser sa passerelle pour se connecter à un réseau local. Vous pouvez également configurer des [connexions de réseau virtuel à réseau virtuel](../vpn-gateway/vpn-gateway-vnet-vnet-rm-ps.md?toc=%2fazure%2fvirtual-network%2ftoc.json) à l’aide de passerelles, même pour les réseaux virtuels appairés.

Lorsque vous configurez les deux options d’interconnexion de réseaux virtuels, le trafic entre les réseaux virtuels transite via la configuration de Peering. Le trafic utilise le réseau principal Azure.

Vous pouvez également configurer la passerelle du réseau virtuel appairé en tant que point de transit vers un réseau local. Dans ce cas, le réseau virtuel qui utilise une passerelle distante ne peut pas posséder sa propre passerelle. Un réseau virtuel ne possède qu’une seule passerelle. Il peut s’agir d’une passerelle locale ou distante dans le réseau virtuel appairé, comme l’illustre le schéma suivant :

![Transit de peering de réseaux virtuels](./media/virtual-networks-peering-overview/local-or-remote-gateway-in-peered-virual-network.png)

L’appairage de réseaux virtuels et l’appairage de réseaux virtuels mondiaux prennent en charge le transit par passerelle.

Le transit par passerelle entre des réseaux virtuels créés via des modèles de déploiement différents est pris en charge. La passerelle doit donc figurer dans un réseau virtuel du modèle Resource Manager. Pour en savoir plus sur l’utilisation d’une passerelle pour le transit, consultez [Configure a VPN gateway for transit in a virtual network peering (Configurer une passerelle VPN pour le transit dans une homologation de réseaux virtuels)](../vpn-gateway/vpn-gateway-peering-gateway-transit.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

Lorsque vous appairez des réseaux virtuels qui partagent une connexion Azure ExpressRoute unique, le trafic entre eux transite par la relation de Peering. Ce trafic utilise le réseau principal Azure. Vous pouvez toujours utiliser des passerelles locales dans chaque réseau virtuel pour vous connecter au circuit local. Dans le cas contraire, vous pouvez utiliser une passerelle partagée et configurer le transit pour la connectivité locale.

## <a name="troubleshoot"></a>Dépanner

Pour confirmer que les réseaux virtuels sont appairés, vous pouvez vérifier les itinéraires effectifs. Vérifiez les itinéraires pour une interface réseau dans tout sous-réseau d’un réseau virtuel. Si une homologation de réseaux virtuels existe, tous les sous-réseaux au sein du réseau virtuel ont des itinéraires avec le type de tronçon suivant *VNet Peering* pour chaque espace d’adressage de chaque réseau virtuel homologué. Pour plus d’informations, consultez [Diagnostiquer un problème de routage sur une machine virtuelle](diagnose-network-routing-problem.md).

Vous pouvez aussi résoudre les problèmes de connectivité à la machine virtuelle d’un réseau virtuel appairé à l’aide d’Azure Network Watcher. Une vérification de la connectivité vous permet de voir comment le trafic est acheminé à partir de l’interface réseau d’une machine virtuelle source vers l’interface réseau d’une machine virtuelle de destination. Pour plus d’informations, consultez [Détecter un problème de connexion avec Azure Network Watcher à l’aide du Portail Azure](../network-watcher/network-watcher-connectivity-portal.md#check-connectivity-to-a-virtual-machine).

Vous pouvez également essayer [Détecter un problème d’appairage de réseaux virtuels](virtual-network-troubleshoot-peering-issues.md).

## <a name="constraints-for-peered-virtual-networks"></a>Contraintes pour les réseaux virtuels appairés<a name="requirements-and-constraints"></a>

Les contraintes ci-après s’appliquent uniquement quand des réseaux virtuels sont appairés à l’échelle mondiale :

* Les ressources situées dans un réseau virtuel ne peuvent pas communiquer avec l’adresse IP frontale d’un équilibreur de charge interne (ILB) De base dans un réseau virtuel appairé à l’échelle mondiale.
* Certains services qui utilisent un équilibreur de charge De base ne fonctionnent pas sur l’appairage de réseaux virtuels mondiaux. Pour plus d’informations, consultez [Quelles sont les contraintes liées aux équilibreurs de charge et à l’appairage de réseaux virtuels mondiaux ?](virtual-networks-faq.md#what-are-the-constraints-related-to-global-vnet-peering-and-load-balancers).

Pour plus d’informations, consultez [Configuration requise et contraintes](virtual-network-manage-peering.md#requirements-and-constraints). Pour en savoir plus sur le nombre de Peerings pris en charge, consultez [Limites de mise en réseau](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits).

## <a name="permissions"></a>Autorisations

Pour en savoir plus sur les autorisations requises pour créer un appairage de réseaux virtuels, consultez [Autorisations](virtual-network-manage-peering.md#permissions).

## <a name="pricing"></a>Tarifs

Un coût nominal s’applique pour le trafic entrant et sortant qui utilise une connexion d’appairage de réseaux virtuels. Pour plus d’informations, consultez [Tarification de Réseau virtuel Microsoft Azure](https://azure.microsoft.com/pricing/details/virtual-network).

Le transit par passerelle est une propriété de Peering qui permet à un réseau virtuel d’exploiter la passerelle VPN/ExpressRoute d’un réseau virtuel appairé. Le transit par passerelle fonctionne pour la connectivité intersite et de réseau à réseau. Le trafic vers la passerelle (entrant ou sortant) dans le réseau virtuel appairé entraîne des frais d’appairage de réseaux virtuels sur le réseau virtuel spoke (ou réseau virtuel non-passerelle). Pour plus d’informations, consultez [Tarification de la passerelle VPN](https://azure.microsoft.com/pricing/details/vpn-gateway/) pour connaître les frais de passerelle VPN et Tarification de la passerelle ExpressRoute pour ceux de la passerelle ExpressRoute.

>[!NOTE]
> Une version précédente de ce document indiquait que les frais d’appairage de réseaux virtuels ne s’appliquaient pas au réseau virtuel spoke (ou réseau virtuel non-passerelle) avec un transit par passerelle. Le document reflète désormais la tarification exacte, conformément à la page de tarification.

## <a name="next-steps"></a>Étapes suivantes

* Vous pouvez créer un Peering entre deux réseaux virtuels. Les réseaux peuvent appartenir au même abonnement, à des modèles de déploiement différents dans le même abonnement ou à des abonnements différents. Suivez un didacticiel pour l’un des scénarios suivants :

    |Modèle de déploiement Azure             | Abonnement  |
    |---------                          |---------|
    |Les deux modèles Resource Manager              |[Identique](tutorial-connect-virtual-networks-portal.md)|
    |                                   |[Différent](create-peering-different-subscriptions.md)|
    |Un modèle Resource Manager, un modèle classique  |[Identique](create-peering-different-deployment-models.md)|
    |                                   |[Différent](create-peering-different-deployment-models-subscriptions.md)|

* Pour découvrir comment créer une topologie de réseau hub-and-spoke, consultez [Implémenter une topologie de réseau hub-and-spoke dans Azure](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke?toc=%2fazure%2fvirtual-network%2ftoc.json).
* Pour en savoir plus sur tous les paramètres d’appairage de réseaux virtuels, consultez [Créer, modifier ou supprimer un appairage de réseaux virtuels](virtual-network-manage-peering.md).
* Pour obtenir des réponses aux questions fréquentes sur l’appairage de réseaux virtuels et l’appairage de réseaux virtuels mondiaux, consultez [Appairage de réseaux virtuels](virtual-networks-faq.md#vnet-peering).
