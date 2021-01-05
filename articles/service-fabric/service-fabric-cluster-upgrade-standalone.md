---
title: Mettre à niveau un cluster Azure Service Fabric autonome
description: Découvrez comment mettre à niveau la version ou la configuration d’un cluster Service Fabric autonome.  T
ms.topic: conceptual
ms.date: 11/12/2018
ms.openlocfilehash: 519a9163d16fda2dd9fcf49cf22fe4ad4a272e09
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86260989"
---
# <a name="upgrading-and-updating-a-service-fabric-standalone-cluster"></a>Mise à niveau et mise à jour d’un cluster Service Fabric autonome

Pour les systèmes modernes, la réussite à long terme d’un produit dépend de sa capacité à être mis à niveau. Un cluster Azure Service Fabric autonome est une ressource que vous possédez. Cet article décrit ce qui peut être mis à niveau ou mis à jour.

## <a name="controlling-the-fabric-version-that-runs-on-your-cluster"></a>Contrôle de la version de la structure qui s’exécute sur votre cluster
Veillez à ce que votre cluster exécute toujours une [version de Service Fabric prise en charge](service-fabric-versions.md). Lorsque Microsoft annonce le lancement d’une nouvelle version de Service Fabric, la fin de la prise en charge de la version précédente est signalée 60 jours minimum après la date de l’annonce. Les nouvelles versions sont annoncées [sur le blog de l’équipe Service Fabric](https://techcommunity.microsoft.com/t5/azure-service-fabric/bg-p/Service-Fabric). La nouvelle version peut alors être sélectionnée.

Vous pouvez définir votre cluster de façon à recevoir les nouvelles mises à niveau de structure automatiques publiées par Microsoft, ou opter pour la sélection manuelle d’une version de structure prise en charge que vous voulez associer à votre cluster. Pour plus d’informations, consultez [Mettre à niveau la version de Service Fabric qui s’exécute sur votre cluster](service-fabric-cluster-upgrade-windows-server.md).

## <a name="customize-configuration-settings"></a>Personnaliser les paramètres de configuration

De nombreux [paramètres de configuration](service-fabric-cluster-manifest.md) différents peuvent être définis dans le fichier *ClusterConfig.json*, tel que le niveau de fiabilité des propriétés du cluster et des nœuds.  Pour plus d’informations, consultez [Mettre à niveau la configuration d’un cluster autonome](service-fabric-cluster-config-upgrade-windows-server.md).  De nombreux autres paramètres plus avancés peuvent également être personnalisés.  Pour plus d’informations, consultez [Paramètres de structure du cluster Service Fabric](service-fabric-cluster-fabric-settings.md).

## <a name="define-node-properties"></a>Définir les propriétés des nœuds
Parfois, vous voulez garantir que certaines charges de travail s’exécutent uniquement sur certains types de nœud du cluster. Par exemple, certaines charges de travail peuvent nécessiter des GPU ou des SSD, tandis que d’autres n’en ont pas besoin. Pour chacun de ces types de nœud d’un cluster, vous pouvez ajouter des propriétés de nœud personnalisées aux nœuds du cluster. Les contraintes de placement sont les instructions associées aux différents services que vous sélectionnez pour une ou plusieurs propriétés de nœud. Les contraintes de placement définissent là où les services doivent s’exécuter.

Pour plus de détails sur l’utilisation des contraintes de placement, les propriétés de nœud et la manière de les définir, consultez [Propriétés de nœud et contraintes de placement](service-fabric-cluster-resource-manager-cluster-description.md#node-properties-and-placement-constraints).
 

## <a name="add-capacity-metrics"></a>Ajouter des métriques de capacité
Pour chaque type de nœud, vous pouvez ajouter des mesures de capacité personnalisées que vous souhaitez utiliser dans vos applications pour créer un rapport sur la charge. Pour plus d’informations sur l’utilisation de métriques de capacité pour créer un rapport sur la charge, consultez les documents Gestionnaire de ressources de cluster Service Fabric sur la [Description de votre cluster](service-fabric-cluster-resource-manager-cluster-description.md) et les [Métriques et charge](service-fabric-cluster-resource-manager-metrics.md).

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
