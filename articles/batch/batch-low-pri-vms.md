---
title: Exécuter des charges de travail sur des machines virtuelles rentables de faible priorité
description: Apprenez à configurer des machines virtuelles de faible priorité pour réduire le coût des charges de travail Azure Batch.
author: mscurrell
ms.topic: how-to
ms.date: 09/08/2020
ms.custom: seodec18
ms.openlocfilehash: bd5b73cf55110985a2e7eecbc161c77ca6d645cb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89568453"
---
# <a name="use-low-priority-vms-with-batch"></a>Utiliser des machines virtuelles de faible priorité avec Batch

Azure Batch offre des machines virtuelles de faible priorité afin de réduire le coût des charges de travail Batch. Les machines virtuelles de faible priorité rendent possibles de nouveaux types de charges de travail Batch en permettant d’utiliser une grande quantité de puissance de calcul, moyennant un coût très faible.

Les machines virtuelles de faible priorité tirent parti de la capacité excédentaire dans Azure. Lorsque vous spécifiez des machines virtuelles de faible priorité dans vos pools, Azure Batch peut utiliser ce surplus lorsqu’il est disponible.

La contrepartie à l’utilisation de machines virtuelles de faible priorité est que ces machines virtuelles peuvent ne pas être toujours disponibles pour être réaffectées ou qu’elles peuvent être préemptées à tout moment, en fonction de la capacité disponible. Pour cette raison, les machines virtuelles de faible priorité sont plus adaptées à certains types de charges de travail. Utilisez des machines virtuelles de faible priorité pour les charges de travail par lots et de traitement asynchrone lorsque l’heure d’achèvement de la tâche est flexible et que le travail est réparti entre plusieurs machines virtuelles.

Les machines virtuelles de faible priorité sont proposées à un prix considérablement inférieur à celui des machines virtuelles dédiées. Pour plus d’informations sur la tarification, consultez [Tarification Batch](https://azure.microsoft.com/pricing/details/batch/).

> [!NOTE]
> Les [machines virtuelles Spot](https://azure.microsoft.com/pricing/spot/) sont désormais disponibles pour les [machines virtuelles mono-instance](../virtual-machines/spot-vms.md) et les [groupes de machines virtuelles identiques](../virtual-machine-scale-sets/use-spot.md). Les machines virtuelles Spot sont une évolution des machines virtuelles de faible priorité, mais diffèrent par leurs tarifs variables et la possibilité de définir un prix maximal lors de leur allocation.
>
> Les pools Azure Batch commenceront à prendre en charge les machines virtuelles Spot dans les mois qui suivront leur mise à disposition générale, avec de nouvelles versions des [API et outils Batch](./batch-apis-tools.md). Une fois que la prise en charge des machines virtuelles Spot sera disponible, les machines virtuelles de faible priorité seront déconseillées : elles continueront d’être prises en charge, à l’aide des versions actuelles des API et des outils, pendant au moins 12 mois, afin de laisser suffisamment de temps à une migration vers des machines virtuelles Spot. 
>
> Les machines virtuelles Spot ne sont pas prises en charge pour les pools de [configuration du service cloud](/rest/api/batchservice/pool/add#cloudserviceconfiguration). Pour utiliser des machines virtuelles Spot, les pools de service cloud doivent être migrés vers des pools de [configuration de machine virtuelle](/rest/api/batchservice/pool/add#virtualmachineconfiguration).

## <a name="use-cases-for-low-priority-vms"></a>Cas d’utilisation des machines virtuelles de faible priorité

Étant donné les caractéristiques des machines virtuelles de faible priorité, quelles sont les charges de travail qui peuvent et ne peuvent pas les utiliser ? En règle générale, les charges de traitement par lots sont adaptées, dans la mesure où les travaux sont répartis entre plusieurs tâches parallèles, ou s’il existe de nombreux travaux montés et distribués entre plusieurs machines virtuelles.

-   Pour optimiser l’utilisation de la capacité excédentaire dans Azure, les travaux appropriés peuvent effectuer un scale-out.

-   Parfois, les machines virtuelles ne sont pas disponibles ou sont préemptées, ce qui entraîne une réduction des capacités pour les tâches, et peut conduire à l’interruption d’une tâche et à sa réexécution. La durée d’exécution des travaux doit donc être flexible.

-   Les travaux avec des tâches plus longues peuvent être davantage impactés s’ils se trouvent interrompus. Si des tâches longues implémentent des points de contrôle pour enregistrer la progression au fil de leur exécution, l’impact de l’interruption est réduit. Les tâches plus courtes ont tendance à mieux fonctionner avec des machines virtuelles de faible priorité, car l’impact de l’interruption est moindre.

-   Les travaux MPI dont l’exécution est longue et qui utilisent plusieurs machines virtuelles ne sont pas adaptés à l’utilisation de machines virtuelles de faible priorité, car la préemption d’une machine virtuelle peut nécessiter la réexécution de l’ensemble du travail.

Voici des exemples de cas d’utilisation du traitement par lots bien adaptés à l’utilisation des machines virtuelles de faible priorité :

-   **Développement et test** : d’importantes économies peuvent être réalisées, en particulier en cas de développement de solutions à grande échelle. Tous les types de tests peuvent tirer parti de l’utilisation de machines virtuelles de faible priorité, mais les tests de charge à grande échelle et les tests de régression sont particulièrement concernés.

-   **Ajout de capacité à la demande** : les machines virtuelles de faible priorité peuvent être utilisées pour venir en complément de machines virtuelles standard dédiées. Quand elles sont disponibles, les travaux peuvent être mis à l’échelle et par conséquent se terminer plus rapidement, pour un coût moindre. Si elles ne sont pas disponibles, les machines virtuelles dédiées de base sont disponibles.

-   **Durée d’exécution des travaux flexible** : s’il existe une certaine flexibilité dans la durée d’exécution des travaux, des pertes potentielles de capacité peuvent être tolérées ; toutefois, avec l’ajout de machines virtuelles de faible priorité, les travaux sont souvent exécutés plus rapidement et pour un coût moindre.

Les pools Batch peuvent être configurés de façon à utiliser les machines virtuelles de faible priorité de plusieurs manières, en fonction de la flexibilité de la durée d’exécution des travaux :

-   Les machines virtuelles de faible priorité peuvent être utilisées seulement dans un pool. Dans ce cas, Batch récupère les capacités préemptées quand elles sont disponibles. Cette configuration est le moyen le plus économique d’exécuter des travaux, dans la mesure où seules des machines virtuelles de faible priorité sont utilisées.

-   Les machines virtuelles de faible priorité peuvent être utilisées conjointement avec une ligne de base fixe de machines virtuelles dédiées. Le nombre fixe de machines virtuelles dédiées garantit qu’il existe toujours une capacité permettant de poursuivre le traitement d’un travail.

-   Il est possible d’utiliser un mélange dynamique de machines virtuelles dédiées et de machines virtuelles de faible priorité, afin que les machines virtuelles de faible priorité, plus économiques, soient utilisées seulement quand elles sont disponibles, mais que les machines virtuelles dédiées (plus chères) soient montées en puissance quand c’est nécessaire. Cette configuration permet de conserver une quantité minimale de capacité disponible pour que les travaux continuent de progresser.

## <a name="batch-support-for-low-priority-vms"></a>Prise en charge des machines virtuelles de faible priorité par Batch

Azure Batch fournit plusieurs fonctionnalités permettant de facilement utiliser et tirer parti des machines virtuelles de faible priorité :

-   Les pools Batch peuvent contenir à la fois des machines virtuelles dédiées et des machines virtuelles de faible priorité. Le nombre de machines virtuelles de chaque type peut être indiqué lors de la création d’un pool et modifié à tout moment pour un pool existant, grâce à l’opération de redimensionnement explicite ou à la mise à l’échelle automatique. La soumission de travaux et de tâches peut rester inchangée, quels que soient les types de machines virtuelles figurant dans le pool. Vous pouvez aussi configurer un pool pour utiliser uniquement des machines virtuelles de faible priorité pour exécuter les travaux à un coût le plus faible possible, tout en faisant appel à des machines virtuelles dédiées si la capacité descend sous un seuil minimal, afin d’assurer la continuité de l’exécution des travaux.

-   Les pools Batch cherchent automatiquement à atteindre le nombre cible de machines virtuelles de faible priorité. Si des machines virtuelles sont préemptées, Batch tente de remplacer la capacité perdue et de revenir au nombre cible.

-   Quand des tâches sont interrompues, Batch détecte et replace automatiquement en file d’attente les tâches pour les réexécuter.

-   Les machines virtuelles de faible priorité ont un quota de processeurs virtuels distinct de celui des machines virtuelles dédiées. 
    Le quota des machines virtuelles de faible priorité est supérieur à celui des machines virtuelles dédiées, car les machines virtuelles de faible priorité sont moins onéreuses. Pour plus d’informations, consultez [Quotas et limites du service Batch](batch-quota-limit.md#resource-quotas).    

> [!NOTE]
> Les machines virtuelles de faible priorité ne sont actuellement pas prises en charge pour les comptes Batch créés en mode [Abonnement utilisateur](accounts.md).

## <a name="create-and-update-pools"></a>Créer et mettre à jour des pools

Un pool Batch peut contenir à la fois des machines virtuelles dédiées et des machines virtuelles de faible priorité (également appelées nœuds de calcul). Vous pouvez définir le nombre cible de nœuds de calcul, tant pour les machines virtuelles dédiées que pour les machines virtuelles de faible priorité. Le nombre cible de nœuds correspond au nombre de machines virtuelles que vous souhaitez avoir dans le pool.

Par exemple, pour créer un pool à l’aide de machines virtuelles du service cloud Azure avec une cible de 5 machines virtuelles dédiées et 20 machines virtuelles de faible priorité :

```csharp
CloudPool pool = batchClient.PoolOperations.CreatePool(
    poolId: "cspool",
    targetDedicatedComputeNodes: 5,
    targetLowPriorityComputeNodes: 20,
    virtualMachineSize: "Standard_D2_v2",
    cloudServiceConfiguration: new CloudServiceConfiguration(osFamily: "5") // WS 2016
);
```

Pour créer un pool à l’aide de machines virtuelles Azure (dans ce cas, des machines virtuelles Linux) avec une cible de 5 machines virtuelles dédiées et 20 machines virtuelles de faible priorité :

```csharp
ImageReference imageRef = new ImageReference(
    publisher: "Canonical",
    offer: "UbuntuServer",
    sku: "16.04-LTS",
    version: "latest");

// Create the pool
VirtualMachineConfiguration virtualMachineConfiguration =
    new VirtualMachineConfiguration("batch.node.ubuntu 16.04", imageRef);

pool = batchClient.PoolOperations.CreatePool(
    poolId: "vmpool",
    targetDedicatedComputeNodes: 5,
    targetLowPriorityComputeNodes: 20,
    virtualMachineSize: "Standard_D2_v2",
    virtualMachineConfiguration: virtualMachineConfiguration);
```

Vous pouvez obtenir le nombre actuel de nœuds pour les machines virtuelles dédiées et pour les machines virtuelles de faible priorité :

```csharp
int? numDedicated = pool1.CurrentDedicatedComputeNodes;
int? numLowPri = pool1.CurrentLowPriorityComputeNodes;
```

Une propriété des nœuds du pool permet d’indiquer si le nœud correspond à une machine virtuelle dédiée ou à une machine virtuelle de faible priorité :

```csharp
bool? isNodeDedicated = poolNode.IsDedicated;
```

Pour les pools de configurations de machines virtuelles, quand un ou plusieurs nœuds sont préemptés, une opération pour obtenir la liste des nœuds sur le pool retourne toujours ces nœuds. Le nombre actuel de nœuds de faible priorité reste inchangé, mais l’état de ces nœuds est défini sur **Préempté**. Batch tente de trouver des machines virtuelles de remplacement et, s’il y parvient, les nœuds passent à l’état **Création**, puis **Démarrage**, avant de devenir disponibles pour l’exécution des tâches, tout comme de nouveaux nœuds.

## <a name="scale-a-pool-containing-low-priority-vms"></a>Mettre à l’échelle un pool contenant des machines virtuelles de faible priorité

Comme avec pools composés uniquement de machines virtuelles dédiées, il est possible de mettre à l’échelle un pool contenant des machines virtuelles de faible priorité en appelant la méthode de redimensionnement ou à l’aide de la mise à l’échelle automatique.

L’opération de redimensionnement du pool prend en compte un deuxième paramètre facultatif qui met à jour la valeur de **targetLowPriorityNodes** :

```csharp
pool.Resize(targetDedicatedComputeNodes: 0, targetLowPriorityComputeNodes: 25);
```

La formule de mise à l’échelle automatique du pool prend en charge les machines virtuelles de faible priorité comme suit :

-   Vous pouvez obtenir ou définir la valeur de la variable **$TargetLowPriorityNodes** définie par le service.

-   Vous pouvez obtenir la valeur de la variable **$CurrentLowPriorityNodes** définie par le service.

-   Vous pouvez obtenir la valeur de la variable **$PreemptedNodeCount** définie par le service. 
    Cette variable renvoie le nombre de nœuds à l’état Reporté et vous permet d’augmenter ou de réduire le nombre de nœuds dédiés, en fonction du nombre de nœuds reportés qui ne sont pas disponibles.

## <a name="jobs-and-tasks"></a>Travaux et tâches

Les travaux et les tâches nécessitent peu de configuration supplémentaire pour les nœuds de faible priorité. La seule prise en charge est la suivante :

-   La propriété JobManagerTask d’une tâche a une nouvelle propriété, **AllowLowPriorityNode**. 
    Lorsque cette propriété est définie sur true, la tâche du gestionnaire de travaux peut être planifiée sur un nœud dédié ou sur un nœud de faible priorité. Si cette propriété a la valeur false, la tâche du gestionnaire de travaux est planifiée uniquement sur un nœud dédié.

-   Une [variable d’environnement](batch-compute-node-environment-variables.md) est disponible pour une application de tâche, afin qu’elle puisse déterminer si l’exécution se fait sur un nœud dédié ou sur un nœud de faible priorité. La variable d’environnement est AZ_BATCH_NODE_IS_DEDICATED.

## <a name="handling-preemption"></a>Gestion de préemption

Les machines virtuelles peuvent parfois être préemptées. Quand cela se produit, les tâches qui étaient en cours d’exécution sur les machines virtuelles préemptées sont replacées en file d’attente et réexécutées.

Pour les pools de configurations de machines virtuelles, Batch effectue également les opérations suivantes :

-   L’état des machines virtuelles reportées est mis à jour vers **Reporté**.
-   La machine virtuelle est effectivement supprimée, ce qui entraine la perte des données stockées localement sur la machine virtuelle.
-   Le pool tente continuellement d’atteindre le nombre cible de nœuds de faible priorité disponibles. Une fois la capacité de remplacement trouvée, les nœuds conservent leur ID, mais ils sont réinitialisés. Ils passent par les états **Création** et **Démarrage** avant d’être disponibles pour la planification des tâches.
-   Le nombre de préemptions est disponible sous la forme d’une mesure dans le portail Azure.

## <a name="metrics"></a>Mesures

De nouvelles mesures sont disponibles dans le [portail Azure](https://portal.azure.com) pour les nœuds de basse priorité. Ces mesures sont :

- Nombre de nœuds à priorité basse
- Nombre de cœurs à priorité basse
- Nombre de nœuds reportés

Pour afficher les mesures dans le portail Azure :

1. Accédez à votre compte de traitement par lots dans le portail et afficher les paramètres de votre compte Batch.
2. Sélectionnez **Mesures** à partir de la section **Analyse**.
3. Sélectionnez les mesures que vous souhaitez à partir de la liste des **Mesures disponibles**.

![Capture d’écran montrant la sélection des métriques pour les nœuds de faible priorité.](media/batch-low-pri-vms/low-pri-metrics.png)

## <a name="next-steps"></a>Étapes suivantes

- Apprenez-en davantage sur le [flux de travail et les ressources principales du service Batch](batch-service-workflow-features.md), telles que les pools, les nœuds, les travaux et les tâches.
- Découvrez les [outils et API Batch](batch-apis-tools.md) disponibles pour créer des solutions Batch.
- Commencez par planifier le passage de machines virtuelles de faible priorité à des machines virtuelles Spot. Si vous utilisez des machines virtuelles de faible priorité avec des pools de **configuration de service cloud**, envisagez de passer à des pools de **configuration de machine virtuelle**.
