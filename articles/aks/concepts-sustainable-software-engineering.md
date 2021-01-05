---
title: Concepts – Génie logiciel durable dans Azure Kubernetes Service (AKS)
description: En savoir plus sur le génie logiciel durable dans Azure Kubernetes Service (AKS).
services: container-service
ms.topic: conceptual
ms.date: 08/26/2020
ms.openlocfilehash: 700723041855fdae4f994480d180292fdfd12e15
ms.sourcegitcommit: fa807e40d729bf066b9b81c76a0e8c5b1c03b536
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97346892"
---
# <a name="sustainable-software-engineering-principles-in-azure-kubernetes-service-aks"></a>Principes du génie logiciel durable dans Azure Kubernetes service (AKS)

Les principes du génie logiciel durable sont un ensemble de compétences visant à vous aider à définir, créer et exécuter des applications durables. L’objectif global est de réduire l’empreinte carbone de tous les aspects de votre application. La page [Principes du génie logiciel durable][principles-sse] présente une vue d’ensemble de ces principes.

Il est important de comprendre qu’il s’agit d’une évolution des priorités et de l’orientation. Dans de nombreux cas, le logiciel est conçu et exécuté de manière à privilégier des performances rapides et une faible latence. Le génie logiciel durable vise à réduire autant que possible les émissions de carbone. Dans certains cas, l’application de principes de génie logiciel durable peut offrir des performances plus rapides ou une latence plus faible, par exemple en réduisant le nombre total de déplacements sur le réseau. Dans d’autres cas, la réduction des émissions de carbone peut entraîner un ralentissement des performances ou une latence accrue, par exemple en retardant des charges de travail de faible priorité. Avant d’envisager d’appliquer des principes de génie logiciel durable à votre application, passez en revue les priorités, les besoins et les compromis de votre application.

## <a name="measure-and-optimize"></a>Mesurer et optimiser

Pour réduire l’empreinte carbone de vos clusters AKS, vous devez comprendre de quelle façon les ressources de votre cluster sont utilisées. [Azure Monitor][azure-monitor] fournit des détails sur l’utilisation des ressources de votre cluster, telles que l’utilisation de la mémoire et du processeur. Ces données peuvent vous aider à prendre des décisions éclairées visant à réduire l’empreinte carbone de votre cluster et à observer l’effet de vos modifications. Vous pouvez également installer [Microsoft Sustainability Calculator][sustainability-calculator] pour connaître l’empreinte carbone de toutes vos ressources Azure.

## <a name="increase-resource-utilization"></a>Augmenter l’utilisation des ressources

L’une des méthodes permettant de réduire votre empreinte carbone consiste à réduire la durée d’inactivité de vos ressources de calcul. La réduction de la durée d’inactivité implique une augmentation de l’utilisation de vos ressources de calcul. Par exemple, si vous aviez quatre nœuds dans votre cluster, chacun s’exécutant à 50 % de sa capacité, les quatre nœuds ont une capacité inutilisée de 50 % qui reste inactive. Si vous réduisiez votre cluster à trois nœuds, la même charge de travail ferait que vos trois nœuds fonctionneraient à 67 % de leur capacité, réduisant la capacité inutilisée à 33 % sur chaque nœud et augmentant votre utilisation.

> [!IMPORTANT]
> Lorsque vous envisagez d’apporter des modifications aux ressources de votre cluster, vérifiez que les [pools de votre système][system-pools] disposent de suffisamment de ressources pour maintenir la stabilité des composants du système central de votre cluster. Ne réduisez jamais les ressources de votre cluster au point qu’il devienne instable.

Après avoir examiné l’utilisation de votre cluster, envisagez d’utiliser les fonctionnalités offertes par [les pools multinœuds][multiple-node-pools]. Vous pouvez utiliser le [dimensionnement des nœuds][node-sizing] pour définir des pools de nœuds avec des profils de processeur et de mémoire spécifiques, ce qui vous permet d’adapter vos nœuds aux besoins de votre charge de travail. Le dimensionnement de vos nœuds selon les besoins de votre charge de travail peut vous aider à faire fonctionner quelques nœuds à un taux d’utilisation plus élevé. Vous pouvez également configurer le mode de [mise à l’échelle][scale] de votre cluster et utiliser l’[autoscaler de pods horizontaux][scale-horizontal] et l’[autoscaler de cluster][scale-auto] pour mettre votre cluster à l’échelle automatiquement en fonction de votre configuration. Le contrôle de l’échelle de votre cluster peut vous aider à maintenir tous vos nœuds à un niveau d’utilisation élevé tout en suivant l’évolution de la charge de travail de votre cluster. Dans les cas où une charge de travail tolère des interruptions ou des arrêts soudains, vous pouvez utiliser [pools spots][spot-pools] pour tirer parti de la capacité inactive dans Azure. Par exemple, les pools spots peuvent fonctionner correctement pour les programmes de traitement par lots ou les environnements de développement.

L’augmentation de l’utilisation peut également réduire le nombre de nœuds excédentaires, ce qui réduit l’énergie consommée par les [réservations de ressources sur chaque nœud][resource-reservations].

Examinez également les *requêtes* et les *limites* de processeur et de mémoire dans les manifestes Kubernetes de vos applications. À mesure que vous réduisez ces valeurs pour la mémoire et le processeur, le cluster dispose de davantage de mémoire et de processeur pour exécuter d’autres charges de travail. À mesure que vous exécutez plus de charges de travail avec moins de processeur et de mémoire, l’allocation de votre cluster devient plus dense, ce qui augmente votre utilisation. Lorsque vous réduisez le processeur et la mémoire de vos applications, le comportement de vos applications peut se dégrader ou devenir instable si vous définissez ces valeurs sur une valeur trop faible. Avant de modifier les *requêtes* et les *limites* du processeur et de la mémoire, pensez à effectuer des tests de référence pour savoir si ces valeurs sont définies de manière appropriée. En outre, ne réduisez jamais ces valeurs au point que votre application devienne instable.

## <a name="reduce-network-travel"></a>Réduire les déplacements sur le réseau

La réduction de la distance que doivent parcourir les requêtes et les réponses depuis et vers votre cluster permet généralement de réduire la consommation d’électricité grâce à la mise en réseau des appareils et de réduire les émissions de carbone. Après avoir examiné votre trafic, envisagez de créer des clusters [dans des régions][regions] plus proches de la source de votre trafic. Vous pouvez également utiliser [Azure Traffic Manager][azure-traffic-manager] pour vous aider à acheminer le trafic vers le cluster le plus proche et des [groupes de placement de proximité][proiximity-placement-groups] pour réduire la distance entre les ressources Azure.

> [!IMPORTANT]
> Lorsque vous envisagez d’apporter des modifications à la mise en réseau de votre cluster, ne réduisez jamais les déplacements sur le réseau au détriment de la charge de travail. Par exemple, l’utilisation de [zones de disponibilité][availability-zones] entraîne davantage de déplacements réseau sur votre cluster, mais l’utilisation de cette fonctionnalité peut être nécessaire pour faire face aux exigences de la charge de travail.

## <a name="demand-shaping"></a>Façonner la requête

Dans la mesure du possible, envisagez de déplacer la demande de ressources de votre cluster vers des heures ou des régions où vous pouvez utiliser une capacité excédentaire. Par exemple, envisagez de changer l’heure ou la région d’un programme de traitement par lots pour qu’il s’exécute ou utilise des [pools spots][spot-pools]. Envisagez également de refactoriser votre application afin d’utiliser une file d’attente pour différer l’exécution des charges de travail qui n’ont pas besoin d’un traitement immédiat.

## <a name="next-steps"></a>Étapes suivantes

Découvrez plus en détail les fonctionnalités ’AKS mentionnées dans cet article :

* [Pools multinœuds][multiple-node-pools]
* [Dimensionnement de nœuds][node-sizing]
* [Mise à l’échelle d’un cluster][scale]
* [Autoscaler de pods horizontaux][scale-horizontal]
* [Autoscaler de cluster][scale-auto]
* [Pools spots][spot-pools]
* [Pools de système][system-pools]
* [Réservations de ressources][resource-reservations]
* [Groupes de placement de proximité][proiximity-placement-groups]
* [Zones de disponibilité][availability-zones]

[availability-zones]: availability-zones.md
[azure-monitor]: ../azure-monitor/insights/container-insights-overview.md
[azure-traffic-manager]: ../traffic-manager/traffic-manager-overview.md
[proiximity-placement-groups]: reduce-latency-ppg.md
[regions]: faq.md#which-azure-regions-currently-provide-aks
[resource-reservations]: concepts-clusters-workloads.md#resource-reservations
[scale]: concepts-scale.md
[scale-auto]: concepts-scale.md#cluster-autoscaler
[scale-horizontal]: concepts-scale.md#horizontal-pod-autoscaler
[spot-pools]: spot-node-pool.md
[multiple-node-pools]: use-multiple-node-pools.md
[node-sizing]: use-multiple-node-pools.md#specify-a-vm-size-for-a-node-pool
[sustainability-calculator]: https://azure.microsoft.com/blog/microsoft-sustainability-calculator-helps-enterprises-analyze-the-carbon-emissions-of-their-it-infrastructure/
[system-pools]: use-system-pools.md
[principles-sse]: https://docs.microsoft.com/learn/modules/sustainable-software-engineering-overview/