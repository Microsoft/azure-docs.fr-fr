---
title: Qu’est-ce qu’Azure Load Balancer ?
titleSuffix: Azure Load Balancer
description: Présentation des fonctionnalités, de l'architecture et de l'implémentation de l'équilibrage de charge Azure. Découvrez comment fonctionne Load Balancer et comment l’utiliser dans le cloud.
services: load-balancer
documentationcenter: na
author: asudbring
ms.service: load-balancer
Customer intent: As an IT administrator, I want to learn more about the Azure Load Balancer service and what I can use it for.
ms.devlang: na
ms.topic: overview
ms.custom: seodec18
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 1/14/2020
ms.author: allensu
ms.openlocfilehash: 96c0242414c67d97fc324977b1259f63c27c3a26
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94696810"
---
# <a name="what-is-azure-load-balancer"></a>Qu’est-ce qu’Azure Load Balancer ?

L’*équilibrage de charge* correspond à une répartition équilibrée de la charge ou du trafic réseau entrant au sein d’un groupe de ressources ou serveurs back-end. 

Azure Load Balancer opère à la couche quatre du modèle OSI (Open Systems Interconnection). Il s’agit du point de contact unique pour les clients. Load Balancer distribue les flux entrants arrivant sur le serveur front-end de l’équilibreur de charge sur des instances du pool de back-ends. Ces flux dépendent des règles d’équilibrage de charge et des sondes d’intégrité configurées. Les instances de pool de back-ends peuvent être des machines virtuelles Azure ou des instances d’un groupe de machines virtuelles identiques.

Un **[équilibreur de charge public](./components.md#frontend-ip-configurations)** permet des connexions sortantes pour les machines virtuelles de votre réseau virtuel. Ces connexions s’effectuent par la traduction de leurs adresses IP privées en adresses IP publiques. Les équilibreurs de charge publics sont utilisés pour équilibrer la charge du trafic Internet sur vos machines virtuelles.

Un **[équilibreur de charge interne (ou privé)](./components.md#frontend-ip-configurations)** est utilisé lorsque des adresses IP privées sont nécessaires uniquement au niveau du serveur front-end. Les équilibreurs de charge internes sont utilisés pour équilibrer la charge du trafic au sein d’un réseau virtuel. Vous pouvez accéder au front-end d’un équilibreur de charge à partir d’un réseau local dans le cadre d’un scénario hybride.

<p align="center">
  <img src="./media/load-balancer-overview/load-balancer.svg" alt="Figure depicts both public and internal load balancers directing traffic to port 80 on multiple servers on a Web tier and port 443 on multiple servers on a business tier." width="512" title="Azure Load Balancer">
</p>

*Figure : Équilibrage d’applications multiniveaux à l’aide d’un Load Balancer public et d’un Load Balancer interne*

Pour plus d’informations sur les composants de l’équilibreur de charge, consultez [Composants Azure Load Balancer](./components.md).

## <a name="why-use-azure-load-balancer"></a>Pourquoi utiliser Azure Load Balancer ?
Standard Load Balancer vous permet de mettre à l’échelle vos applications et de créer des services à haute disponibilité. Load Balancer prend en charge à la fois les scénarios entrants et les scénarios sortants. Un équilibreur de charge offre une latence faible et un débit élevé, et peut augmenter l’échelle jusqu’à des millions de flux pour toutes les applications TCP et UDP.

Voici les principaux scénarios que vous pouvez gérer avec Standard Load Balancer :

- Équilibrer la charge du trafic **[interne](./quickstart-load-balancer-standard-internal-portal.md)** et **[externe](./tutorial-load-balancer-standard-manage-portal.md)** sur les machines virtuelles Azure

- Augmenter la disponibilité en répartissant les ressources **[au sein des zones](./tutorial-load-balancer-standard-public-zonal-portal.md)** et **[entre les zones](./tutorial-load-balancer-standard-public-zone-redundant-portal.md)**

- Configurer la **[connectivité sortante](./load-balancer-outbound-connections.md)** pour les machines virtuelles Azure

- Utiliser des **[sondes d’intégrité](./load-balancer-custom-probe-overview.md)** pour superviser les ressources à charge équilibrée

- Utiliser le **[réacheminement de port](./tutorial-load-balancer-port-forwarding-portal.md)** pour accéder aux machines virtuelles d’un réseau virtuel via une adresse IP publique et un port

- Activer la prise en charge de l’ **[équilibrage de charge](../virtual-network/virtual-network-ipv4-ipv6-dual-stack-standard-load-balancer-powershell.md)** de **[IPv6](../virtual-network/ipv6-overview.md)**

- Standard Load Balancer fournit des métriques multidimensionnelles via [Azure Monitor](../azure-monitor/overview.md).  Ces métriques peuvent être filtrées, regroupées et réparties pour une dimension donnée.  Elles fournissent des analyses en cours et historiques sur les performances et l’intégrité de votre service.  Resource Health est également pris en charge. Pour plus d’informations, consultez **[Diagnostics de Standard Load Balancer](load-balancer-standard-diagnostics.md)** .

- Équilibrer la charge des services sur **[plusieurs ports, plusieurs adresses IP ou les deux](./load-balancer-multivip-overview.md)**

- Déplacer des ressources d’équilibreur de charge **[internes](./move-across-regions-internal-load-balancer-portal.md)** et **[externes](./move-across-regions-external-load-balancer-portal.md)** entre les différentes régions Azure

- Équilibrer simultanément la charge du flux TCP et UDP sur tous les ports à l’aide des **[ports haute disponibilité](./load-balancer-ha-ports-overview.md)** .

### <a name="secure-by-default"></a><a name="securebydefault"></a>Sécuriser par défaut

Standard Load Balancer repose sur le modèle de sécurité réseau Confiance Zéro. Standard Load Balancer est sécurisé par défaut et fait partie de votre réseau virtuel. Le réseau virtuel est un réseau privé et isolé.  Cela signifie que les équilibreurs de charge Standard et les adresses IP publiques standard sont fermés aux flux entrants, sauf s’ils sont ouverts par des groupes de sécurité réseau. Les groupes de sécurité réseau sont utilisés pour autoriser explicitement le trafic autorisé.  Si vous n’avez pas de groupe de sécurité réseau sur un sous-réseau ou une carte réseau de votre ressource de machine virtuelle, le trafic n’est pas autorisé à atteindre cette ressource. Pour plus d’informations sur les Groupes de sécurité réseau et la façon de les appliquer à votre scénario, consultez [Filtrer le trafic réseau avec les groupes de sécurité réseau](../virtual-network/network-security-groups-overview.md).
Par défaut, Basic Load Balancer est ouvert sur Internet. En outre, Load Balancer ne stocke pas les données client.

## <a name="pricing-and-sla"></a>Tarifs et contrat SLA

Pour plus d’informations sur la tarification de Load Balancer Standard, consultez [Tarification de Load Balancer](https://azure.microsoft.com/pricing/details/load-balancer/).
La référence SKU De base de Load Balancer est proposée gratuitement.
Consultez [Contrat SLA pour Load Balancer](https://aka.ms/lbsla). Basic Load Balancer n’a pas de contrat SLA.

## <a name="whats-new"></a>Nouveautés

Abonnez-vous au flux RSS et découvrez les dernières mises à jour des fonctionnalités Azure Load Balancer dans la page [Mises à jour Azure](https://azure.microsoft.com/updates/?category=networking&query=load%20balancer).

## <a name="next-steps"></a>Étapes suivantes

Pour bien démarrer avec les équilibreurs de charge, consultez [Créer un équilibreur de charge standard public](quickstart-load-balancer-standard-public-portal.md).

Pour plus d’informations sur les limitations et les composants d’Azure Load Balancer, consultez [Composants Azure Load Balancer](./components.md) et [Concepts relatifs à Azure Load Balancer](./concepts.md).