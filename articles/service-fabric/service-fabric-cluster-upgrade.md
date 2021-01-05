---
title: Mettre à niveau un cluster Azure Service Fabric
description: 'Découvrez-en plus sur la mise à niveau de la version ou la configuration d’un cluster Azure Service Fabric : définition du mode de mise à jour du cluster, mise à niveau des certificats, ajout de ports d’application, application de correctifs du système d’exploitation et effets possibles des mises à niveau.'
ms.topic: conceptual
ms.date: 11/12/2018
ms.openlocfilehash: 028c91f85a6e318f7ea686c1bcd50262eb7c6bf1
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96571026"
---
# <a name="upgrading-and-updating-an-azure-service-fabric-cluster"></a>Mise à niveau et mise à jour d’un cluster Azure Service Fabric

Pour les systèmes modernes, la réussite à long terme d’un produit dépend de sa capacité à être mis à niveau. Un cluster Azure Service Fabric est une ressource que vous possédez mais qui est en partie gérée par Microsoft. Cet article décrit ce qui est géré automatiquement et ce que vous pouvez configurer vous-même.

## <a name="controlling-the-fabric-version-that-runs-on-your-cluster"></a>Contrôle de la version de la structure qui s’exécute sur votre cluster

Vérifiez que votre cluster exécute toujours une [version de structure fabric prise en charge](service-fabric-versions.md). Chaque fois que Microsoft annonce le lancement d’une nouvelle version de Service Fabric, la fin de la prise en charge de la version précédente est signalée 60 jours minimum après la date de lancement. Les nouvelles versions sont annoncées sur le [blog de l’équipe Service Fabric](https://techcommunity.microsoft.com/t5/azure-service-fabric/bg-p/Service-Fabric).

Un événement d’intégrité est généré 14 jours avant la date d’expiration de la version qu’exécute votre cluster. Il place le cluster dans un état d’avertissement. Ce dernier conserve cet état tant que vous n’avez pas effectué de mise à niveau vers une version prise en charge.

Vous pouvez définir votre cluster de façon à recevoir les nouvelles mises à niveau de structure automatiques publiées par Microsoft, ou opter pour la sélection de la version de structure prise en charge que vous voulez associer à votre cluster.  Pour en savoir plus, consultez [Mettre à niveau la version de Service Fabric de votre cluster](service-fabric-cluster-upgrade-version-azure.md).

## <a name="fabric-upgrade-behavior-during-automatic-upgrades"></a>Comportement de mise à niveau de la structure pendant les mises à niveau automatiques

Microsoft tient à jour le code de l’infrastructure et la configuration exécutés dans un cluster Azure. Nous effectuons des mises à niveau automatiques surveillées du logiciel en fonction des besoins. Ces mises à niveau peuvent concerner le code, la configuration ou les deux. Pour garantir que votre application n’est pas affectée par ces mises à niveau, ou l’est au minimum, celles-ci sont effectuées en plusieurs phases :

### <a name="phase-1-an-upgrade-is-performed-by-using-all-cluster-health-policies"></a>Phase 1 : Une mise à niveau est effectuée à l'aide de toutes les stratégies d'intégrité du cluster

Pendant cette phase, les mises à niveau traitent un domaine de mise à niveau à la fois et les applications qui étaient en cours d'exécution dans le cluster continuent à fonctionner sans interruption. Les stratégies d’intégrité du cluster (pour l’intégrité du nœud et l’intégrité de l’application) sont respectées pendant la mise à niveau.

Si les stratégies d’intégrité du cluster ne sont pas satisfaites, la mise à niveau est annulée et un e-mail est envoyé au propriétaire de l’abonnement. Le message électronique contient les informations suivantes :

* Une notification indiquant que nous avons dû annuler une mise à niveau du cluster.
* Des suggestions d'actions correctives, le cas échéant.
* Le nombre de jours (*n*) avant l’exécution de la Phase 2.

Nous essayons d'exécuter la même mise à niveau plusieurs fois dans le cas où les mises à niveau ont échoué pour des raisons d'infrastructure. Après le délai de *n* jours à partir de la date d’envoi de l’e-mail, nous passons à la Phase 2.

Si les stratégies d'intégrité du cluster sont respectées, la mise à niveau est considérée comme réussie et marquée comme terminée. Cela peut se produire pendant la première exécution ou l'une des exécutions de la mise à niveau suivantes dans cette phase. Aucun message électronique de confirmation d'une exécution réussie n'est envoyé. Ceci afin d’éviter de vous envoyer trop d’e-mails, la réception d’un message devant être considérée comme une exception. Nous pensons que la plupart des mises à niveau du cluster s'exécuteront sans affecter la disponibilité de votre application.

### <a name="phase-2-an-upgrade-is-performed-by-using-default-health-policies-only"></a>Phase 2 : Une mise à niveau est effectuée à l'aide des stratégies d'intégrité par défaut uniquement

Les stratégies d'intégrité de cette phase sont définies de telle sorte que le nombre d'applications intègres au début de la mise à niveau reste identique pendant la durée de la mise à niveau. Comme lors de la Phase 1, les mises à niveau de la Phase 2 traitent un domaine de mise à niveau à la fois et les applications qui étaient en cours d'exécution dans le cluster continuent à fonctionner sans interruption. Les stratégies d’intégrité du cluster (une combinaison de l’intégrité du nœud et l’intégrité de toutes les applications en cours d’exécution dans le cluster) sont respectées pendant la durée de la mise à niveau.

Si les stratégies d'intégrité de cluster en vigueur ne sont pas respectées, la mise à niveau est annulée. Un message électronique est envoyé au propriétaire de l’abonnement. Le message électronique contient les informations suivantes :

* Une notification indiquant que nous avons dû annuler une mise à niveau du cluster.
* Des suggestions d'actions correctives, le cas échéant.
* Le nombre de jours (n) avant l’exécution de la Phase 3.

Nous essayons d'exécuter la même mise à niveau plusieurs fois dans le cas où les mises à niveau ont échoué pour des raisons d'infrastructure. Un message électronique de rappel est envoyé deux jours avant que le délai de n jours ne soit écoulé. Après le délai de n jours à partir de la date d'envoi du message électronique, nous passons à la Phase 3. Les messages électroniques que nous vous envoyons lors de la Phase 2 doivent être pris au sérieux et des actions correctives doivent être prises.

Si les stratégies d'intégrité du cluster sont respectées, la mise à niveau est considérée comme réussie et marquée comme terminée. Cela peut se produire pendant la première exécution ou l'une des exécutions de la mise à niveau suivantes dans cette phase. Aucun message électronique de confirmation d'une exécution réussie n'est envoyé.

### <a name="phase-3-an-upgrade-is-performed-by-using-aggressive-health-policies"></a>Phase 3 : Une mise à niveau est effectuée à l'aide de stratégies d'intégrité agressives

Les stratégies d'intégrité de cette phase visent à compléter la mise à niveau plutôt qu’à protéger l'intégrité des applications. Peu de mises à niveau de cluster se terminent par cette phase. Si votre cluster atteint cette phase, il est très probable que votre application devienne défectueuse et/ou ne soit plus disponible.

Comme pour les deux autres phases, les mises à niveau de la Phase 3 traitent un domaine de mise à niveau à la fois.

Si les stratégies d'intégrité du cluster ne sont pas respectées, la mise à niveau est annulée. Nous essayons d'exécuter la même mise à niveau plusieurs fois dans le cas où les mises à niveau ont échoué pour des raisons d'infrastructure. Après cela, le cluster est épinglé et il ne bénéficiera plus des mises à niveau et/ou du support.

Un message électronique contenant ces informations est envoyé au propriétaire de l’abonnement, ainsi que les actions correctives. Nous pensons qu'aucun cluster ne sera confronté à un échec de la Phase 3.

Si les stratégies d'intégrité du cluster sont respectées, la mise à niveau est considérée comme réussie et marquée comme terminée. Cela peut se produire pendant la première exécution ou l'une des exécutions de la mise à niveau suivantes dans cette phase. Aucun message électronique de confirmation d'une exécution réussie n'est envoyé.

## <a name="manage-certificates"></a>Gérer des certificats

Service Fabric utilise les [certificats de serveur X.509](service-fabric-cluster-security.md) que vous spécifiez lorsque vous créez un cluster pour sécuriser les communications entre les nœuds du cluster et authentifier les clients. Vous pouvez ajouter, mettre à jour ou supprimer des certificats pour le cluster et le client dans le [portail Azure](https://portal.azure.com) ou à l’aide de l’interface de ligne de commande Azure ou PowerShell.  Pour en savoir plus, consultez [Ajouter ou supprimer des certificats](service-fabric-cluster-security-update-certs-azure.md).

## <a name="open-application-ports"></a>Ouvrir les ports d’application

Vous pouvez modifier les ports d'application en modifiant les propriétés de ressource d'équilibrage de charge associées au type de nœud. Vous pouvez utiliser le portail Azure ou l’interface de ligne de commande Azure ou PowerShell. Pour plus d’informations, consultez [Ouvrir des ports d’application pour un cluster](create-load-balancer-rule.md).

## <a name="define-node-properties"></a>Définir les propriétés des nœuds

Parfois, vous voulez garantir que certaines charges de travail s’exécutent uniquement sur certains types de nœud du cluster. Par exemple, certaines charges de travail peuvent nécessiter des GPU ou des SSD, tandis que d’autres n’en ont pas besoin. Pour chacun de ces types de nœud d’un cluster, vous pouvez ajouter des propriétés de nœud personnalisées aux nœuds du cluster. Les contraintes de placement sont les instructions associées aux différents services que vous sélectionnez pour une ou plusieurs propriétés de nœud. Les contraintes de placement définissent là où les services doivent s’exécuter.

Pour plus de détails sur l’utilisation des contraintes de placement, les propriétés de nœud et la manière de les définir, consultez [Propriétés de nœud et contraintes de placement](service-fabric-cluster-resource-manager-cluster-description.md#node-properties-and-placement-constraints).

## <a name="add-capacity-metrics"></a>Ajouter des métriques de capacité

Pour chaque type de nœud, vous pouvez ajouter des mesures de capacité personnalisées que vous souhaitez utiliser dans vos applications pour créer un rapport sur la charge. Pour plus d’informations sur l’utilisation de métriques de capacité pour créer un rapport sur la charge, consultez les documents Gestionnaire de ressources de cluster Service Fabric sur la [Description de votre cluster](service-fabric-cluster-resource-manager-cluster-description.md) et les [Métriques et charge](service-fabric-cluster-resource-manager-metrics.md).

## <a name="set-health-policies-for-automatic-upgrades"></a>Définir des stratégies d’intégrité pour les mises à niveau automatiques

Vous pouvez spécifier des stratégies d’intégrité personnalisées pour la mise à niveau de la structure. Si vous avez défini votre cluster sur une mise à niveau automatique de la structure, ces stratégies sont appliquées lors de la phase 1 de cette mise à niveau.
Si vous avez défini votre cluster sur une mise à niveau manuelle de la structure, ces stratégies sont appliquées chaque fois que vous sélectionnez une nouvelle version, ce qui amène le système à déclencher la mise à niveau de la structure dans votre cluster. Si vous ne remplacez pas les stratégies, les valeurs par défaut sont utilisées.

Vous pouvez spécifier des stratégies de contrôle d’intégrité personnalisées ou passer en revue les paramètres actuels dans le panneau Fabric Upgrade (Mise à niveau de la structure), en sélectionnant les paramètres de mise à niveau avancés. L’image suivante vous explique comment procéder.

![Gérer les stratégies de contrôle d’intégrité personnalisées][HealthPolices]

## <a name="customize-fabric-settings-for-your-cluster"></a>Personnaliser les paramètres de la structure pour votre cluster

De nombreux paramètres de configuration différents peuvent être personnalisés sur un cluster, tels que le niveau de fiabilité des propriétés du cluster et des nœuds. Pour plus d’informations, consultez [Paramètres de structure du cluster Service Fabric](service-fabric-cluster-fabric-settings.md).

## <a name="patch-the-os-in-the-cluster-nodes"></a>Appliquer des correctifs au système d’exploitation dans les nœuds du cluster

L’application d’orchestration des correctifs est une application Service Fabric qui automatise l’application de correctifs du système d’exploitation sur un cluster Service Fabric sans temps d’arrêt. L’[application d’orchestration des correctifs pour Windows](service-fabric-patch-orchestration-application.md) peut être déployée sur votre cluster pour installer des correctifs de manière orchestrée, tout en maintenant les services disponibles en permanence.

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment personnaliser certains [paramètres de la structure du cluster Service Fabric](service-fabric-cluster-fabric-settings.md)
* Découvrez comment [mettre votre cluster à l’échelle](service-fabric-cluster-scale-in-out.md)
* Découvrez les [mises à niveau de l’application](service-fabric-application-upgrade.md)

<!--Image references-->
[CertificateUpgrade]: ./media/service-fabric-cluster-upgrade/CertificateUpgrade2.png
[AddingProbes]: ./media/service-fabric-cluster-upgrade/addingProbes2.PNG
[AddingLBRules]: ./media/service-fabric-cluster-upgrade/addingLBRules.png
[HealthPolices]: ./media/service-fabric-cluster-upgrade/Manage_AutomodeWadvSettings.PNG
[ARMUpgradeMode]: ./media/service-fabric-cluster-upgrade/ARMUpgradeMode.PNG
[Create_Manualmode]: ./media/service-fabric-cluster-upgrade/Create_Manualmode.PNG
[Manage_Automaticmode]: ./media/service-fabric-cluster-upgrade/Manage_Automaticmode.PNG
