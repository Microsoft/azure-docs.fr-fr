---
title: Bonnes pratiques de base Azure DDoS Protection
description: Découvrez les bonnes pratiques de sécurité à l’aide de la protection DDoS.
services: ddos-protection
documentationcenter: na
author: yitoh
ms.service: ddos-protection
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/08/2020
ms.author: yitoh
ms.openlocfilehash: 66c1ab1cb5ed478aa34825fb6903e4d06f834097
ms.sourcegitcommit: 9889a3983b88222c30275fd0cfe60807976fd65b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94989471"
---
# <a name="fundamental-best-practices"></a>Bonnes pratiques de base

La section suivante fournit des conseils normatifs pour générer des services résistants aux attaques DDoS sur Azure.

## <a name="design-for-security"></a>Conception dans l’optique de la sécurité

Assurez-vous que la sécurité est une priorité tout au long du cycle de vie d’une application, de la conception et de l’implémentation au déploiement et aux opérations. Les applications peuvent contenir des bogues qui laissent passer un volume relativement faible de requêtes conçues pour utiliser une quantité anormale de ressources, provoquant une panne de service. 

Pour permettre de protéger un service fonctionnant sur Microsoft Azure, vous devez bien comprendre l’architecture de votre application et respecter les [cinq piliers de la qualité logicielle](/azure/architecture/guide/pillars).
Vous devez avoir connaissance des volumes de trafic habituels, du modèle de connectivité entre l’application et d’autres applications, et des points de terminaison de service exposés à l’Internet public.

Il est extrêmement important que vous conceviez une application suffisamment résiliente pour surmonter une attaque ciblée par déni de service. La plateforme Azure intègre des fonctionnalités de sécurité et de confidentialité, à commencer par [SDL (Security Development Lifecycle)](https://www.microsoft.com/sdl/default.aspx). SDL tient compte de la sécurité à chaque phase de développement et vérifie qu’Azure est continuellement mis à jour pour le rendre encore plus sécurisé.

## <a name="design-for-scalability"></a>Conception dans l’optique de la scalabilité

L’extensibilité correspond à la capacité d’un système à traiter une charge accrue. Concevez vos applications de sorte qu’elles puissent être [mises à l’échelle horizontalement](/azure/architecture/guide/design-principles/scale-out) pour répondre à la demande d’une charge amplifiée, en particulier dans le cadre d’une attaque DDoS. Si votre application repose sur une seule instance unique d’un service, cela crée un point de défaillance unique. L’approvisionnement de plusieurs instances rend votre système plus résilient et plus évolutif.

Pour [Azure App Service](../app-service/overview.md), sélectionnez un [plan App Service](../app-service/overview-hosting-plans.md) qui offre plusieurs instances. Pour les Azure Cloud Services, configurez chacun de vos rôles de manière à utiliser [plusieurs instances](../cloud-services/cloud-services-choose-me.md). Pour [Machines virtuelles Azure](../virtual-machines/index.yml), vérifiez que votre architecture de machines virtuelles inclut plusieurs machines virtuelles et que chacune d’elles figure dans un [groupe à haute disponibilité](../virtual-machines/windows/tutorial-availability-sets.md). Nous vous recommandons d’utiliser des [groupes de machines virtuelles identiques](../virtual-machine-scale-sets/overview.md) pour les fonctionnalités de mise à l’échelle automatique.

## <a name="defense-in-depth"></a>Défense en profondeur

L’idée derrière la défense en profondeur est de gérer les risques en utilisant des stratégies de protection variées. Le fait de superposer des défenses dans une application réduit les chances de réussite d’une attaque. Nous vous recommandons d’implémenter des conceptions sécurisées pour vos applications par le biais des fonctionnalités intégrées à la plateforme Azure.

Par exemple, plus la taille (*surface d’exposition*) de l’application est importante, plus le risque d’attaque est élevé. Vous pouvez réduire la surface d’exposition en créant une liste d’approbation permettant de limiter l’espace d’adressage IP exposé et les ports d’écoute qui ne sont pas nécessaires sur les équilibreurs de charge ([Azure Load Balancer](../load-balancer/quickstart-load-balancer-standard-public-portal.md) et [Azure Application Gateway](../application-gateway/application-gateway-create-probe-portal.md)). Les [groupes de sécurité réseau](../virtual-network/network-security-groups-overview.md) permettent également de réduire la surface d’attaque.
Vous pouvez utiliser des [balises de service](../virtual-network/network-security-groups-overview.md#service-tags) et des [groupes de sécurité d’application](../virtual-network/network-security-groups-overview.md#application-security-groups) pour simplifier la création de règles de sécurité et configurer la sécurité réseau comme prolongement naturel de la structure d’une application.

Vous devez déployer les services Azure dans un [réseau virtuel](../virtual-network/virtual-networks-overview.md) dans la mesure du possible. Les ressources de service peuvent ainsi communiquer par le biais d’adresses IP privées. Le trafic du service Azure à partir d’un réseau virtuel utilise des adresses IP publiques comme adresses IP source par défaut. Le fait d’utiliser des [points de terminaison de service](../virtual-network/virtual-network-service-endpoints-overview.md) force le trafic de service à utiliser des adresses privées de réseau virtuel comme adresses IP source lors de l’accès au service Azure à partir d’un réseau virtuel.

Il arrive fréquemment que les ressources locales d’un client soient attaquées en même temps que ses ressources dans Azure. Si vous connectez un environnement local à Azure, nous vous recommandons de réduire l’exposition des ressources locales à l’Internet public. Pour bénéficier des fonctionnalités de mise à l’échelle et de protection DDoS avancée d’Azure, déployez vos entités publiques connues dans Azure. Ces entités accessibles publiquement étant souvent la cible d’attaques DDoS, le fait de les placer dans Azure réduit l’impact sur vos ressources locales.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [créer un plan de protection DDoS](manage-ddos-protection.md).