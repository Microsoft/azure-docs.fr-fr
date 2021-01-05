---
title: Mise à l’échelle automatique dans Microsoft Azure
description: Mise à l’échelle automatique dans Microsoft Azure
ms.subservice: autoscale
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: bd7c1582cdb4b2b1b72d3f969ad08879d208785f
ms.sourcegitcommit: 4bee52a3601b226cfc4e6eac71c1cb3b4b0eafe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94505835"
---
# <a name="overview-of-autoscale-in-microsoft-azure"></a>Vue d’ensemble de la mise à l’échelle automatique dans Microsoft Azure
Cet article décrit la mise à l’échelle automatique Microsoft Azure ainsi que ses avantages, et comment commencer à l’utiliser.  

La mise à l’échelle automatique Azure Monitor s’applique uniquement à [Virtual Machine Scale Sets](https://azure.microsoft.com/services/virtual-machine-scale-sets/), [Azure Cloud Services](https://azure.microsoft.com/services/cloud-services/), [App Service - Web Apps](https://azure.microsoft.com/services/app-service/web/), [services Gestion des API](../../api-management/api-management-key-concepts.md) et [Cluster Azure Data Explorer](/azure/data-explorer/).

> [!NOTE]
> Azure dispose de deux méthodes de mise à l’échelle automatique. L’ancienne version de la mise à l’échelle automatique s’applique aux machines virtuelles (groupes à haute disponibilité). Cette fonctionnalité assure une prise en charge limitée et nous vous recommandons de migrer vers les jeux de mise à l’échelle de machine virtuelle pour une prise en charge plus rapide et plus fiable de la mise à l’échelle automatique. Un lien sur la façon d’utiliser l’ancienne technologie est inclus dans cet article.  
>

## <a name="what-is-autoscale"></a>Qu’est-ce que la mise à l’échelle automatique ?
La mise à l’échelle automatique vous permet de disposer de la bonne quantité de ressources en cours d’exécution pour gérer la charge sur votre application. Elle vous permet d’ajouter des ressources pour gérer les augmentations de charge et d’économiser de l’argent en supprimant les ressources qui sont inactives. Vous spécifiez un nombre minimal et maximal d’instances à exécuter et ajoutez ou supprimez des machines virtuelles automatiquement en fonction d’un ensemble de règles. Avoir un nombre minimal d’instances permet de vous assurer que votre application est toujours en cours d’exécution même en l’absence de charge. Avoir un nombre maximal d’instances limite votre coût total possible par heure. Vous effectuez une mise à l’échelle automatique entre ces deux extrêmes, à l’aide de règles que vous créez.

 ![Description de la mise à l’échelle automatique. Ajouter et supprimer des machines virtuelles](./media/autoscale-overview/AutoscaleConcept.png)

Lorsque les conditions relatives aux règles sont remplies, une ou plusieurs actions de mise à l’échelle automatique sont déclenchées. Vous pouvez ajouter et supprimer des machines virtuelles ou effectuer d’autres actions. Le schéma conceptuel suivant illustre ce processus.  

 ![Diagramme de flux de la mise à l’échelle automatique](./media/autoscale-overview/Autoscale_Overview_v4.png)

La description suivante s’applique aux éléments du schéma précédent.   

## <a name="resource-metrics"></a>Métriques de ressources
Les ressources émettent des métriques, qui sont ensuite traitées par des règles. Les mesures sont fournies via différentes méthodes.
Les jeux de mise à l’échelle de machine virtuelle utilisent les données de télémétrie des agents de diagnostics Azure, tandis que les données de télémétrie pour les applications web et les services cloud proviennent directement de l’infrastructure Azure. Parmi les statistiques couramment utilisées figurent l’utilisation du processeur, l’utilisation de la mémoire, le nombre de threads, la longueur de la file d’attente et l’utilisation du disque. Pour obtenir une liste des données de télémétrie que vous pouvez utiliser, voir [Mesures courantes pour la mise à l’échelle automatique](autoscale-common-metrics.md).

## <a name="custom-metrics"></a>Métriques personnalisées
Vous pouvez également utiliser vos propres métriques personnalisées que peuvent émettre vos applications. Si vous avez configuré vos applications pour envoyer des métriques à Application Insights, vous pouvez en tirer parti pour prendre des décisions au sujet de la mise à l’échelle.

## <a name="time"></a>Temps
Les règles basées sur la planification s’appuient sur l’heure UTC. Vous devez définir votre fuseau horaire correctement lorsque vous configurez vos règles.  

## <a name="rules"></a>Règles
Le diagramme montre une seule règle de mise à l’échelle automatique, mais vous pouvez en avoir plusieurs. Vous pouvez créer des règles complexes qui se chevauchent en fonction de vos besoins.  Les types de règles incluent  

* **Basées sur des mesures** : par exemple, effectuer cette action lorsque l’utilisation du processeur est supérieure à 50 %.
* **Basées sur l’heure** : par exemple, déclencher un webhook chaque samedi à 8h dans un fuseau horaire donné.

Les règles basées sur les mesures mesurent la charge de l’application et ajoutent ou suppriment des machines virtuelles en fonction de cette charge. Les règles basées sur la planification vous permettent d’effectuer une mise à l’échelle lorsque vous voyez des modèles horaires dans votre charge et que vous souhaitez effectuer la mise à l’échelle avant qu’une augmentation ou diminution de charge possible ne se produise.  

## <a name="actions-and-automation"></a>Actions et automatisation
Les règles peuvent déclencher un ou plusieurs types d’actions.

* **Mise à l’échelle** : mettre les machines virtuelles à l’échelle
* **E-mail** : envoyer un e-mail aux administrateurs et coadministrateurs d’un abonnement, et/ou à toute adresse e-mail supplémentaire que vous spécifiez
* **Automatisation via webhooks** : appeler des webhooks, qui peuvent déclencher plusieurs actions complexes à l’intérieur ou en dehors d’Azure. Dans Azure, vous pouvez démarrer un runbook Azure Automation, une fonction Azure ou une application logique Azure. Parmi les URL tierces en dehors d’Azure figurent des services comme Slack et Twilio.

## <a name="autoscale-settings"></a>Paramètres de mise à l’échelle automatique
La mise à l’échelle automatique utilise la terminologie et la structure suivantes.

- Un **paramètre de mise à l’échelle automatique** est lu par le moteur de mise à l’échelle automatique afin de déterminer si une montée ou une descente en puissance est nécessaire. Il contient un ou plusieurs profils, des informations sur la ressource cible et des paramètres de notification.

  - Un **profil de mise à l’échelle automatique** est la combinaison des éléments suivants :

    - Un **paramètre de capacité** indiquant les valeurs minimum, maximum et par défaut pour le nombre d’instances.
    - Un **ensemble de règles** , comprenant chacun un déclencheur (heure ou métrique) et une action de mise à l’échelle (augmentation ou réduction).
    - Une **périodicité** indiquant le moment auquel une mise à l’échelle automatique doit appliquer ce profil.

      Vous pouvez avoir plusieurs profils, ce qui vous permet de prendre en charge des exigences différentes qui se chevauchent. Vous pouvez avoir des profils de mise à l’échelle automatique différents pour différentes heures de la journée ou jours de la semaine, par exemple.

  - Un **paramètre de notification** définit quelles notifications doivent se produire lorsqu’un événement de mise à l’échelle automatique a lieu en fonction de la satisfaction des critères de l’un des profils de paramètre de mise à l’échelle automatique. La mise à l’échelle automatique peut notifier une ou plusieurs adresses e-mail ou appeler un ou plusieurs webhooks.


![Structure de règle, de profil et de paramètre de mise à l’échelle automatique Azure](./media/autoscale-overview/AzureResourceManagerRuleStructure3.png)

La liste complète des descriptions et champs configurables est disponible dans [l’API REST de mise à l’échelle automatique](/rest/api/monitor/autoscalesettings).

Pour plus d’exemples de code, consultez

* [Configuration avancée de la mise à l’échelle automatique à l’aide des modèles Resource Manager pour les groupes de machines virtuelles identiques](autoscale-virtual-machine-scale-sets.md)  
* [Paramètres de mise à l’échelle automatique](/rest/api/monitor/autoscalesettings)

## <a name="horizontal-vs-vertical-scaling"></a>Mise à l’échelle horizontale/verticale
La mise à l’échelle automatique s’effectue uniquement horizontalement, ce qui correspond à une augmentation (« out ») ou à une diminution (« in ») du nombre d’instances de machine virtuelle.  La mise à l’échelle horizontale est plus flexible dans un environnement cloud, car elle vous permet d’exécuter des milliers de machines virtuelles pour gérer la charge.

La mise à l’échelle verticale est différente. Elle conserve le même nombre de machines virtuelles, mais rend la machine virtuelle plus (« up ») ou moins (« down ») puissante. La puissance se mesure en termes de mémoire, de vitesse du processeur, d’espace disque, etc.  La mise à l’échelle verticale a davantage de limites. Elle dépend de la disponibilité d’un matériel plus puissant, qui peut atteindre rapidement sa limite et varier d’une région à l’autre. La mise à l’échelle verticale nécessite généralement un arrêt et un redémarrage de la machine virtuelle.

## <a name="methods-of-access"></a>Méthodes d’accès
Vous pouvez configurer la mise à l’échelle automatique via

* [Azure portal](autoscale-get-started.md)
* [PowerShell](../samples/powershell-samples.md#create-and-manage-autoscale-settings)
* [Interface de ligne de commande interplateforme (CLI)](../samples/cli-samples.md#autoscale)
* [API REST Azure Monitor](/rest/api/monitor/autoscalesettings)

## <a name="supported-services-for-autoscale"></a>Services pris en charge pour la mise à l’échelle automatique
| Service | Schéma et documentation |
| --- | --- |
| Web Apps |[Mise à l’échelle des applications web](autoscale-get-started.md) |
| Cloud Services |[Mise à l’échelle automatique d’un service cloud](../../cloud-services/cloud-services-how-to-scale-portal.md) |
| Machines virtuelles : Classique |[Mise à l’échelle de groupes à haute disponibilité de machines virtuelles classiques](/archive/blogs/kaevans/autoscaling-azurevirtual-machines) |
| Machines virtuelles : Groupes identiques Windows |[Mise à l’échelle des jeux de mise à l’échelle de machine virtuelle dans Windows](../../virtual-machine-scale-sets/tutorial-autoscale-powershell.md) |
| Machines virtuelles : Groupes identiques Linux |[Mise à l’échelle des jeux de mise à l’échelle de machine virtuelle dans Linux](../../virtual-machine-scale-sets/tutorial-autoscale-cli.md) |
| Machines virtuelles : Exemple Windows |[Configuration avancée de la mise à l’échelle automatique à l’aide des modèles Resource Manager pour les groupes de machines virtuelles identiques](autoscale-virtual-machine-scale-sets.md) |
| Azure App Service |[Montez une application en puissance dans Azure App Service](../../app-service/manage-scale-up.md)|
| Service Gestion des API|[Mettre à l’échelle automatiquement une instance du service Gestion des API Azure](../../api-management/api-management-howto-autoscale.md)
| Clusters Azure Data Explorer|[Gérer la mise à l’échelle des clusters Azure Data Explorer pour prendre en compte les fluctuations de la demande](/azure/data-explorer/manage-cluster-horizontal-scaling)|
| Logic Apps |[Ajout de la fonctionnalité d’environnement de service d’intégration (ISE)](../../logic-apps/ise-manage-integration-service-environment.md#add-ise-capacity)|
| Spring Cloud |[Configurer la mise à l’échelle automatique pour les applications de microservice](../../spring-cloud/spring-cloud-tutorial-setup-autoscale.md)|
| Service Bus |[Mettre à jour automatiquement les unités de messagerie d’un espace de noms Azure Service Bus](../../service-bus-messaging/automate-update-messaging-units.md)|

## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur la mise à l’échelle automatique, utilisez les guides sur la mise à l’échelle automatique répertoriés précédemment ou consultez les ressources suivantes :

* [Mesures courantes pour la mise à l’échelle automatique dans Azure Monitor](autoscale-common-metrics.md)
* [Meilleures pratiques pour la mise à l’échelle automatique d’Azure Insights](autoscale-best-practices.md)
* [Utilisation d’actions de mise à l’échelle automatique pour envoyer des notifications d’alerte webhook et par courrier électronique](autoscale-webhook-email.md)
* [Paramètres de mise à l’échelle automatique](/rest/api/monitor/autoscalesettings)
* [Dépannage de la mise à l’échelle automatique avec des jeux de mise à l’échelle de machine virtuelle](../../virtual-machine-scale-sets/virtual-machine-scale-sets-troubleshoot.md)
