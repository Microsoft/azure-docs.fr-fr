---
title: Mise à l’échelle des clusters Azure Service Fabric
description: En savoir plus sur la mise à l’échelle de clusters Azure Service Fabric, qu’il s’agisse d’une diminution ou d’une réduction de taille des instances, ou d’une montée ou descente en puissance. À mesure que les demandes des applications évoluent, il est possible que les clusters Service Fabric changent également.
ms.topic: conceptual
ms.date: 11/13/2018
ms.author: atsenthi
ms.openlocfilehash: 126be55c63c625995ad52b84a51a8983e220652d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85610198"
---
# <a name="scaling-azure-service-fabric-clusters"></a>Mise à l’échelle des clusters Azure Service Fabric
Un cluster Service Fabric est un groupe de machines virtuelles ou physiques connectées au réseau, sur lequel vos microservices sont déployés et gérés. Une machine ou une machine virtuelle faisant partie d’un cluster est appelée un nœud. Les clusters peuvent potentiellement comporter des milliers de nœuds. Une fois que vous avez créé un cluster Service Fabric, vous pouvez le mettre à l’échelle horizontalement (modifier le nombre de nœuds) ou verticalement (modifier les ressources des nœuds).  Une mise à l’échelle peut s’effectuer à tout moment, même lorsque des charges de travail sont en cours d’exécution sur le cluster.  Lorsque vous mettez vos nœuds à l’échelle, vos applications sont automatiquement mises à l’échelle.

Pourquoi procéder à la mise à l’échelle du cluster ? La demande des applications change au fil du temps.  Il vous faut éventuellement augmenter les ressources de cluster pour prendre en charge l’augmentation de la charge applicative ou du trafic réseau ou diminuer les ressources du cluster en cas de baisse de la demande.

## <a name="scaling-in-and-out-or-horizontal-scaling"></a>Augmentation et diminution de la taille des instances, ou mise à l’échelle horizontale
Modifie le nombre de nœuds dans le cluster.  Une fois que les nouveaux nœuds ont rejoint le cluster, [Cluster Resource Manager](service-fabric-cluster-resource-manager-introduction.md) déplace les services vers ces instances afin de réduire la charge pesant sur les nœuds existants.  Si les ressources du cluster ne sont pas utilisées efficacement, vous pouvez également réduire le nombre de nœuds.  À mesure que les nœuds quittent le cluster, les services se retirent également et la charge augmentent sur les nœuds restants.  La réduction du nombre de nœuds dans un cluster exécuté dans Azure peut vous permettre de réaliser des économies, dans la mesure où vous payez le nombre de machines virtuelles utilisées, non la charge de travail à laquelle elles sont soumises.  

- Avantages : une mise à l’échelle infinie, en théorie.  Si votre application est conçue pour l’évolutivité, vous pouvez prendre en charge une croissance illimitée en ajoutant davantage de nœuds.  Les outils de vos environnements cloud facilitant l’ajout et le retrait des nœuds, il est simple d’ajuster la capacité selon les besoins, en payant uniquement les ressources utilisées.  
- Inconvénients : les applications doivent être [conçues pour la scalabilité](service-fabric-concepts-scalability.md).  Les bases de données et la persistance de l’application peuvent nécessiter des activités architecturales supplémentaires dédiées là aussi à la mise à l’échelle.  [Les collections fiables](service-fabric-reliable-services-reliable-collections.md) dans les services avec état Service Fabric, cependant, simplifient grandement la mise à l’échelle de vos données applicatives.

Les jeux de mise à l’échelle de machine virtuelle sont des ressources de calcul Azure que vous pouvez utiliser pour déployer et gérer une collection de machines virtuelles en tant que jeu. Chaque type de nœud défini dans un cluster Azure est [ configuré comme un groupe identique distinct](service-fabric-cluster-nodetypes.md). Chaque type de nœud peut ensuite faire l’objet d’une augmentation ou d’une diminution de la taille des instances de manière indépendante, avoir différents jeux de ports ouverts et présenter différentes métriques de capacité. 

Lors de la mise à l’échelle d’un cluster Azure, gardez les instructions suivantes à l’esprit :
- Les types de nœud principal exécutant des charges de travail de production doivent toujours comporter au moins cinq nœuds.
- Les types de nœud non principal exécutant des charges de travail de production avec état doivent toujours comporter au moins cinq nœuds.
- Les types de nœud non principal exécutant des charges de travail de production sans état doivent toujours comporter au moins deux nœuds.
- Tout type de nœud dont le [niveau de durabilité](service-fabric-cluster-capacity.md#durability-characteristics-of-the-cluster) est Gold ou Silver doit toujours comporter au moins cinq nœuds.
- Ne supprimez pas d’instances de machine virtuelle/de nœuds aléatoires d’un type de nœud. Utilisez toujours la fonctionnalité de scale-in du groupe de machines virtuelles identiques. La suppression d’instances de machine virtuelle aléatoires peut réduire la capacité des systèmes à équilibrer la charge de manière appropriée.
- Si vous utilisez des règles de mise à l’échelle automatique, définissez-les de manière à ce que la diminution de la taille des instances s’effectue nœud par nœud. La descente en puissance de plusieurs instances en même temps présente des risques.

Étant donné que les types de nœuds Service Fabric de votre cluster sont constitués de groupes de machines virtuelles identiques sur le serveur principal, vous pouvez [définir des règles de mise à l’échelle automatique ou mettre à l’échelle manuellement](service-fabric-cluster-scale-in-out.md) chaque type de nœud/chaque groupe de machines virtuelles identiques.

### <a name="programmatic-scaling"></a>Mise à l’échelle par programmation
Dans de nombreux scénarios, [les mises à l’échelle d’un cluster manuellement ou avec des règles de mise à l’échelle automatiques](service-fabric-cluster-scale-in-out.md) s’avèrent être de bonnes solutions. Pour les scénarios plus avancés, cependant, ces deux possibilités ne sont pas indiquées. Les inconvénients potentiels de ces approches sont les suivants :

- La mise à l’échelle manuelle vous oblige à ouvrir une session et à demander explicitement des opérations de mise à l’échelle. Si les opérations de mise à l’échelle sont requises fréquemment ou à des moments imprévisibles, cette approche n’est probablement pas la bonne.
- Lorsque les règles de mise à l’échelle automatique suppriment une instance d’un groupe de machines virtuelles identiques, elles ne suppriment pas automatiquement les connaissances de ce nœud du cluster Service Fabric associé, sauf si le nœud a un niveau de durabilité agent ou or. Comme les règles de mise à l’échelle automatique fonctionnent au niveau du groupe de machines virtuelles (et non au niveau de Service Fabric), elles peuvent supprimer des nœuds Service Fabric sans les arrêter en douceur. Cette suppression brutale laisse l’état de nœud Service Fabric « ghost » derrière elle, après les opérations de mise à l’échelle. Une personne (ou un service) doit nettoyer régulièrement l’état du nœud supprimé dans le cluster Service Fabric.
- Un type de nœud ayant un niveau de durabilité argent ou or nettoie automatiquement les nœuds supprimés. Par conséquent, aucun nettoyage supplémentaire n’est requis.
- Bien que les règles de mise à l’échelle automatique prennent en charge [plusieurs mesures](../azure-monitor/platform/autoscale-common-metrics.md), ce groupe reste limité. Si votre scénario requiert une mise à l’échelle en fonction d’une mesure non incluse dans ce groupe, les règles de mise à l’échelle automatique ne sont peut-être pas la bonne solution.

Le mode de mise à l’échelle de Service Fabric dépend de votre cas de figure. Si la mise à l’échelle est rare, la possibilité d’ajouter ou de supprimer des nœuds manuellement est probablement suffisante. Dans les scénarios plus complexes, les règles de mise à l’échelle automatique et les kits de développement logiciel (SDK) qui assurent la mise à l’échelle par programmation offrent des alternatives puissantes.

Il existe des API Azure qui permettent aux applications de travailler par programmation avec les groupes de machines virtuelles identiques et des clusters Service Fabric. Si les options de mise à l’échelle automatique existantes ne conviennent pas à votre scénario, ces API permettent de mettre en œuvre une logique de mise à l’échelle personnalisée. 

Cette fonctionnalité de mise à l’échelle automatique personnalisée permet d’ajouter un service sans état à l’application Service Fabric pour gérer les opérations de mise à l’échelle. La création de votre propre service de mise à l’échelle optimise le niveau de contrôle et de personnalisation du comportement de mise à l’échelle de votre application. Cela peut être utile dans les cas qui requièrent un contrôle précis sur le moment ou les modalités de la mise à l’échelle ou la montée en charge d’une application. Toutefois, ce contrôle complique le code. Cette approche signifie que vous devez posséder votre propre code de mise à l’échelle, ce qui n’est pas négligeable. Dans la méthode `RunAsync` du service, un ensemble de déclencheurs peut déterminer si la mise à l’échelle est requise (avec vérification de paramètres tels que la taille maximale des clusters et de mise à l’échelle de recharges).   

L’API utilisée pour les interactions des groupes de machines virtuelles identiques (afin de vérifier et de modifier le nombre actuel d’instances de machines virtuelles) est la [bibliothèque Fluent Azure Management Compute](https://www.nuget.org/packages/Microsoft.Azure.Management.Compute.Fluent/). Cette bibliothèque de calcul fluide fournit une API conviviale pour interagir avec les groupes de machines virtuelles identiques.  Pour interagir avec le cluster Service Fabric, utilisez [System.Fabric.FabricClient](/dotnet/api/system.fabric.fabricclient).

Malgré tout, le code de mise à l’échelle n’a pas besoin de s’exécuter comme un service dans le cluster à mettre à l’échelle. `IAzure` et `FabricClient` peuvent se connecter à leurs ressources Azure associées à distance. Donc, le service de mise à l’échelle peut être une application console ou un service Windows exécuté hors de l’application Service Fabric.

Face à ces limitations, vous pouvez [mettre en œuvre des modèles davantage personnalisés de mise à l’échelle automatique](service-fabric-cluster-programmatic-scaling.md).

## <a name="scaling-up-and-down-or-vertical-scaling"></a>Montée et descente en puissance, ou mise à l’échelle verticale 
Modifie les ressources (processeur, mémoire ou stockage) des nœuds du cluster.
- Avantages : l’architecture de l’application et du logiciel demeure identique.
- Inconvénients : mise à l’échelle limitée, dans la mesure où l’augmentation des ressources sur les nœuds individuels est limitée. Des temps d’arrêt, car il vous faudra mettre hors ligne les machines physiques ou virtuelles afin d’ajouter ou de supprimer des ressources.

Les jeux de mise à l’échelle de machine virtuelle sont des ressources de calcul Azure que vous pouvez utiliser pour déployer et gérer une collection de machines virtuelles en tant que jeu. Chaque type de nœud défini dans un cluster Azure est [ configuré comme un groupe identique distinct](service-fabric-cluster-nodetypes.md). Chaque type de nœud peut alors faire l’objet d’une gestion séparée.  La mise à l’échelle d’un type de nœud vers le haut ou vers le bas implique l’ajout d’un nouveau type de nœud (avec la référence SKU de machine virtuelle mise à jour) et la suppression de l’ancien type de nœud.

Lors de la mise à l’échelle d’un cluster Azure, gardez les instructions suivantes à l’esprit :
- Pour la descente en puissance d’un type de nœud principal, vous ne devez jamais aller en dessous du [niveau de fiabilité](service-fabric-cluster-capacity.md#reliability-characteristics-of-the-cluster).

Le processus de montée ou descente en puissance d’un type de nœud diffère selon qu’il s’agisse d’un type de nœud principal ou non principal.

### <a name="scaling-non-primary-node-types"></a>Mise à l’échelle de types de nœuds non principaux
Créez un nouveau type de nœud avec les ressources dont vous avez besoin.  Mettez à jour les contraintes de placement des services en cours d’exécution afin d’inclure le nouveau type de nœud.  Progressivement (une à la fois), réduisez le nombre d’instances de l’ancien type de nœud à une valeur nulle, de manière à ne pas affecter la fiabilité du cluster.  Les services migrent progressivement vers le nouveau type de nœud, à mesure de la désactivation de l’ancien type de nœud.

### <a name="scaling-the-primary-node-type"></a>Mise à l’échelle du type de nœud principal
Déployez un nouveau type de nœud principal avec la référence SKU de machine virtuelle mise à jour, puis désactivez les instances de type de nœud principal d’origine une à la fois afin que les services système migrent vers le nouveau groupe identique. Vérifiez que le cluster et les nouveaux nœuds sont intègres, puis supprimez le groupe identique d’origine et l’état du nœud pour les nœuds supprimés.

Si cela n’est pas possible, créez un cluster et [restaurez l’état des applications](service-fabric-reliable-services-backup-restore.md) (le cas échéant) de votre ancien cluster. Vous n’avez pas besoin de restaurer l’état des services système ; ceux-ci sont recréés pendant le déploiement de vos applications sur le nouveau cluster. Si les applications que vous exécutiez sur votre cluster étaient sans état, tout ce que vous avez à faire, c’est déployer vos applications sur le nouveau cluster, sans rien avoir à restaurer.

## <a name="next-steps"></a>Étapes suivantes
* Découvrez-en plus sur l’[extensibilité des applications](service-fabric-concepts-scalability.md).
* [Procédez à une montée et une descente en puissance d’un cluster Azure](service-fabric-tutorial-scale-cluster.md).
* [Mettez à l’échelle un cluster Azure par programmation](service-fabric-cluster-programmatic-scaling.md), à l’aide du kit de développement logiciel de calcul Azure.
* [Augmentez ou diminuez la taille des instances d’un cluster autonome](service-fabric-cluster-windows-server-add-remove-nodes.md).

