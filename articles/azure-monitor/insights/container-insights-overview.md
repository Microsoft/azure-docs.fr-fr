---
title: Vue d’ensemble d’Azure Monitor pour les conteneurs | Microsoft Docs
description: Cet article décrit Azure Monitor pour les conteneurs qui surveille la solution AKS Container Service et la valeur qu’elle offre en surveillant l’intégrité de vos clusters AKS et Container Instances dans Azure.
ms.topic: conceptual
ms.date: 09/08/2020
ms.openlocfilehash: 215427adc38ebd554ca7ac650ec65a05f5a781e8
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91317155"
---
# <a name="azure-monitor-for-containers-overview"></a>Vue d’ensemble d’Azure Monitor pour les conteneurs

Azure Monitor pour les conteneurs est une fonctionnalité conçue pour superviser les performances des charges de travail de conteneur déployées sur :

- des clusters Kubernetes managés hébergés sur [Azure Kubernetes Service (AKS)](../../aks/intro-kubernetes.md)
- des clusters Kubernetes automanagés hébergés sur Azure à l’aide du [moteur AKS](https://github.com/Azure/aks-engine)
- [Azure Container Instances](../../container-instances/container-instances-overview.md)
- des clusters Kubernetes automanagés hébergés sur [Azure Stack](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview?view=azs-1910&preserve-view=true) ou localement
- [Azure Red Hat OpenShift](../../openshift/intro-openshift.md)
- [Kubernetes avec Azure Arc activé](../../azure-arc/kubernetes/overview.md) (préversion)

Azure Monitor pour les conteneurs prend en charge les clusters exécutant le système d’exploitation Linux et Windows Server 2019. Les runtimes de conteneur qu’il prend en charge sont Docker, Moby et tout runtime compatible CRI, tel que élément CRI-O et ContainerD.

La surveillance de vos conteneurs est cruciale, particulièrement lorsque vous exécutez un cluster de production à grande échelle, avec plusieurs applications.

Azure Monitor pour les conteneurs vous permet de surveiller les performances en collectant des métriques sur le processeur et la mémoire à partir des contrôleurs, des nœuds et des conteneurs qui sont disponibles dans Kubernetes via l’API Metrics. Les journaux d’activité de conteneur sont aussi collectés.  Une fois que vous avez activé la supervision des clusters Kubernetes, les métriques et les journaux d’activité sont automatiquement collectés à l’aide d’une version conteneurisée de l’agent Log Analytics pour Linux. Les métriques sont rédigées dans la banque de métriques et les données de journaux dans la banque de journaux associée à votre espace de travail [Log Analytics](../log-query/log-query-overview.md).

![Architecture d’Azure Monitor pour conteneurs](./media/container-insights-overview/azmon-containers-architecture-01.png)

## <a name="what-does-azure-monitor-for-containers-provide"></a>Qu’est-ce qu’Azure Monitor pour les conteneurs vous apporte ?

Grâce à ses différentes fonctionnalités, Azure Monitor pour conteneurs vous offre une expérience de supervision complète. Ces fonctionnalités vous permettent d’avoir une meilleure visibilité des performances et de l’intégrité de votre cluster Kubernetes Linux ou Windows Server 2019 ainsi que des charges de travail des conteneurs. Avec Azure Monitor pour conteneurs, vous pouvez :

* Identifier les conteneurs AKS en cours d’exécution sur le nœud, ainsi que leur utilisation moyenne du processeur et de la mémoire. Cette information peut vous aider à identifier les goulots d’étranglement des ressources.
* Identifier l’utilisation du processeur et de la mémoire des groupes de conteneurs et de leurs conteneurs hébergés dans Azure Container Instances.
* Identifier l’emplacement du conteneur dans un contrôleur ou un pod. Cette information peut vous permettre de voir les performances globales du contrôleur ou du pod.
* Voir la quantité de ressources utilisée par les charges de travail qui sont exécutées sur l’hôte et qui ne sont pas liées aux processus standard nécessaires à la prise en charge du pod
* Comprendre le comportement du cluster lorsqu’il subit des charges moyennes et très importantes. Cette information peut vous aider à identifier les besoins en capacité et à déterminer la charge maximale que le cluster peut supporter.
* Configurer des alertes pour vous avertir de manière proactive ou les enregistrer lorsque l’utilisation de l’UC et de la mémoire sur des nœuds ou des conteneurs dépasse vos seuils, ou encore lorsqu’un changement d’état d’intégrité se produit dans le cluster au niveau de l’infrastructure ou du cumul d’intégrité des nœuds.
* Bénéficier d’une intégration à [Prometheus](https://prometheus.io/docs/introduction/overview/) pour afficher les métriques d’application et de charge de travail qu’il collecte à partir des nœuds et Kubernetes à l’aide de [requêtes](container-insights-log-search.md) pour créer des alertes personnalisées, des tableaux de bord et une analyse détaillée des performances.
* Surveiller les charges de travail de conteneur [déployées sur le moteur AKS](https://github.com/Azure/aks-engine) local et le [moteur AKS sur Azure Stack](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview?view=azs-1908&preserve-view=true).
* Surveiller les charges de travail de conteneur [déployées sur Azure Red Hat OpenShift](../../openshift/intro-openshift.md).

    >[!NOTE]
    >La prise en charge d’Azure Red Hat OpenShift est actuellement une fonctionnalité en préversion publique.
    >

* Surveiller les charges de travail de conteneur [déployées sur Kubernetes avec Azure Arc activé (préversion)](../../azure-arc/kubernetes/overview.md).

Les principales différences entre la supervision d’un cluster Windows Server et celle d’un cluster Linux sont les suivantes :

- Windows ne dispose pas d’une métrique Mémoire RSS et celle-ci n’est donc pas disponible pour le nœud et les conteneurs Windows. La métrique [Plage de travail](/windows/win32/memory/working-set) est disponible.
- Les informations de capacité de stockage des disques ne sont pas disponibles pour les nœuds Windows.
- Seuls les environnements de pod sont surveillés, pas les environnements Docker.
- Avec la préversion, un maximum de 30 conteneurs Windows Server sont pris en charge. Cette limitation ne s’applique pas aux conteneurs Linux.

Regardez la vidéo suivante qui fournit une présentation approfondie de niveau intermédiaire pour vous apprendre à surveiller votre cluster AKS avec Azure Monitor pour les conteneurs.

> [!VIDEO https://www.youtube.com/embed/RjsNmapggPU]

## <a name="how-do-i-access-this-feature"></a>Comment accéder à cette fonctionnalité ?

Vous pouvez accéder à Azure Monitor pour les conteneurs de deux manières : à partir d’Azure Monitor ou à partir du cluster AKS sélectionné directement. À partir d’Azure Monitor, vous disposez d’une perspective globale de tous les conteneurs déployés, ceux qui sont surveillés et ceux qui ne sont pas, vous permettant ainsi de rechercher et de filtrer sur vos abonnements et groupes de ressources, puis d’explorer Azure Monitor pour les conteneurs à partir du conteneur sélectionné.  Sinon, vous pouvez accéder directement à la fonctionnalité à partir d’un conteneur AKS sélectionné dans la page AKS.

![Vue d’ensemble des méthodes d’accès à Azure Monitor pour les conteneurs](./media/container-insights-overview/azmon-containers-experience.png)

Si vous souhaitez surveiller et gérer vos hôtes de conteneur Docker et Windows exécutés en dehors d’AKS pour afficher la configuration, un audit et l’utilisation des ressources, consultez la [solution de supervision des conteneurs](./containers.md).

## <a name="next-steps"></a>Étapes suivantes

Pour commencer la surveillance de votre cluster Kubernetes, consultez [Comment activer Azure Monitor pour les conteneurs](container-insights-onboard.md) afin de comprendre les exigences et les méthodes disponibles pour activer la surveillance.

