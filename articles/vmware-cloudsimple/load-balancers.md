---
title: Azure VMware Solution by CloudSimple - Choisir une solution d'équilibrage de charge pour les clouds privés CloudSimple
description: Décrit les options d'équilibrage de charge déployant une application dans un cloud privé
author: sharaths-cs
ms.author: b-shsury
ms.date: 08/20/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 2fec54c19b9040e7cb44f1a5d2b7909510ab7313
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "82734601"
---
# <a name="choose-a-load-balancing-solution-for-cloudsimple-private-clouds"></a>Choisir une solution d'équilibrage de charge pour les clouds privés CloudSimple

Lorsque vous déployez une application dans un cloud privé CloudSimple, vous pouvez choisir l'une des nombreuses options d'équilibrage de charge.

Vous pouvez choisir un équilibreur de charge virtuel ou logiciel dans votre cloud privé CloudSimple, ou même utiliser l'équilibreur de charge Azure L7 dans votre abonnement Azure pour l’analyse frontale de vos machines virtuelles de niveau web exécutées dans le cloud privé CloudSimple. Voici quelques options :

## <a name="virtual-load-balancers"></a>Équilibreurs de charge virtuels

Vous pouvez déployer des appliances d'équilibrage de charge virtuelles dans votre environnement VMware via l'interface vCenter et les configurer pour l’analyse frontale du trafic des applications.

Voici certains fournisseurs populaires : NginX : http://nginx.org/en/docs/http/load_balancing.html F5- BigIP - Traffic Manager : https://www.f5.com/products/big-ip-services/virtual-editions Citrix ADC : https://www.citrix.com/products/citrix-adc/

## <a name="azure-l7-load-balancer"></a>Équilibreur de charge Azure L7

Lorsque vous utilisez Azure Application Gateway en tant qu'équilibreur de charge L7 pour votre application exécutée dans un cloud privé, vous n'avez pas besoin de gérer le logiciel d’équilibrage de charge. Le logiciel d'équilibrage de charge est géré par Azure. Toutes les machines virtuelles de niveau Web dans le cloud privé utilisent des adresses IP privées et n'ont pas besoin de règles NAT ou d'adresses IP publiques supplémentaires pour résoudre les noms. Les machines virtuelles Web communiquent avec la passerelle d'applications Azure via une connexion privée à bande passante élevée et à faible latence.

Pour en savoir plus sur la configuration de cette solution, reportez-vous au guide de la solution sur l'utilisation d'Azure Application Gateway comme équilibreur de charge L7.

## <a name="azure-internal-load-balancer"></a>Équilibreur de charge interne Azure

Si vous choisissez d'exécuter votre application dans un déploiement hybride où l’analyse frontale Web est exécutée dans un vNet Azure de votre abonnement Azure et que le niveau de base de données de l'application est exécuté dans des machines virtuelles VMware d’un cloud privé CloudSimple, vous pouvez utiliser l’équilibreur de charge interne Azure (équilibreur de charge L4) en amont de vos machines virtuelles de niveau base de données afin de gérer le trafic.

Pour en savoir plus, voir la documentation Azure [Équilibreur de charge interne](../load-balancer/components.md#frontend-ip-configurations).

## <a name="global-server-load-balancer"></a>Équilibreur de charge du serveur global

Si vous recherchez un équilibreur de charge DNS, vous pouvez utiliser des solutions tierces disponibles sur Place de marché Microsoft Azure ou opter pour la solution native Azure.

Azure Traffic Manager est un équilibreur de charge du trafic DNS qui vous permet de distribuer le trafic de manière optimale aux services dans toutes les régions Azure globales et en local, tout en offrant réactivité et haute disponibilité. Pour en savoir plus, consultez la documentation Azure [Traffic Manager](../traffic-manager/traffic-manager-configure-geographic-routing-method.md).
