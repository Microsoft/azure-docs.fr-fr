---
title: Mettre automatiquement à l’échelle les nœuds de calcul dans un pool Azure Batch
description: Activer la mise à l’échelle automatique sur un pool de cloud pour ajuster dynamiquement le nombre de nœuds de calcul dans le pool.
ms.topic: how-to
ms.date: 11/23/2020
ms.custom: H1Hack27Feb2017, fasttrack-edit, devx-track-csharp
ms.openlocfilehash: 033272f22b98b27c67e9a551bce952368d35a043
ms.sourcegitcommit: 1bf144dc5d7c496c4abeb95fc2f473cfa0bbed43
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95737290"
---
# <a name="create-an-automatic-formula-for-scaling-compute-nodes-in-a-batch-pool"></a>Créer une formule automatique pour la mise à l’échelle des nœuds de calcul dans un pool Batch

Azure Batch peut automatiquement mettre à l’échelle des pools en fonction de paramètres que vous définissez, vous faisant ainsi gagner du temps et de l’argent. Grâce à la mise à l’échelle automatique, de façon dynamique, Batch ajoute des nœuds lorsque les demandes de tâches augmentent, et supprime des nœuds de calcul quand elles diminuent.

Pour activer la mise à l’échelle automatique sur un pool de nœuds de calcul, vous lui associez une *formule de mise à l’échelle automatique* définie par vos soins. Le service Batch utilise la formule de mise à l’échelle automatique pour déterminer le nombre de nœuds nécessaires à l’exécution de votre charge de travail. Il peut s’agir de nœuds dédiés ou de [nœuds de faible priorité](batch-low-pri-vms.md). Batch révise périodiquement les données métriques du service et les utilise pour ajuster le nombre de nœuds dans le pool en fonction de votre formule et à un intervalle que vous définissez.

Vous pouvez activer la mise à l’échelle automatique lorsque vous créez un pool, ou l’appliquer à un pool existant. Batch permet d’évaluer vos formules avant de les assigner à des pools et de surveiller l’état de l’exécution des mises à l’échelle automatiques. Une fois que vous avez configuré un pool avec mise à l’échelle automatique, vous pouvez apporter des modifications à la formule ultérieurement.

> [!IMPORTANT]
> Lorsque vous créez un compte Batch, vous pouvez spécifier le [mode allocation de pool](accounts.md), qui détermine si les pools sont alloués dans un abonnement au service Batch (par défaut), ou dans votre abonnement utilisateur. Si vous avez créé votre compte Batch avec la configuration de service Batch par défaut, votre compte est limité à un nombre maximal de cœurs utilisables pour le traitement. Le service Batch met à l’échelle des nœuds uniquement jusqu’à cette limite de cœurs. C’est pourquoi le service Batch peut ne pas atteindre le nombre cible de nœuds de calcul spécifié par une formule de mise à l’échelle automatique. Consultez [Quotas et limites du service Azure Batch](batch-quota-limit.md) pour obtenir des instructions sur l’affichage et l’augmentation des quotas de votre compte.
>
>Si vous avez créé votre compte en mode abonnement utilisateur, votre compte partage le quota de cœurs associé à l’abonnement. Pour en savoir plus, consultez le paragraphe [Limites de machines virtuelles](../azure-resource-manager/management/azure-subscription-service-limits.md#virtual-machines-limits) de la section [Abonnement Azure et limites, quotas et contraintes de service](../azure-resource-manager/management/azure-subscription-service-limits.md).

## <a name="autoscale-formulas"></a>Formules de mise à l’échelle automatique

Une formule de mise à l’échelle automatique est une valeur de chaîne définie par vos soins, qui contient une ou plusieurs instructions. La formule de mise à l’échelle automatique est affectée à l’élément [autoScaleFormula](/rest/api/batchservice/enable-automatic-scaling-on-a-pool) d’un pool (Batch REST) ou à la propriété [CloudPool.AutoScaleFormula](/dotnet/api/microsoft.azure.batch.cloudpool.autoscaleformula) (Batch .NET). Le service Batch utilise votre formule pour déterminer le nombre de nœuds de calcul cibles d’un pool pour le prochain intervalle de traitement. La chaîne de formule ne peut pas dépasser 8 Ko. Elle peut inclure jusqu’à 100 instructions séparées par des points-virgules, et comprendre des sauts de ligne et des commentaires.

Les formules de mise à l’échelle automatique reviennent en quelque sorte à utiliser un « langage » de mise à l’échelle Batch. Les instructions de formules sont des expressions de forme libre qui peuvent inclure des variables définies par le service Batch et des variables définies par l’utilisateur. Le formules peuvent effectuer différentes opérations sur ces valeurs à l’aide de types, d’opérateurs et de fonctions intégrés. Par exemple, une instruction peut prendre la forme suivante :

```
$myNewVariable = function($ServiceDefinedVariable, $myCustomVariable);
```

Les formules contiennent généralement plusieurs instructions qui effectuent des opérations sur les valeurs qui sont obtenues dans les instructions précédentes : Par exemple, nous obtenons tout d’abord une valeur pour `variable1`, puis la transmettons à une fonction pour renseigner `variable2` :

```
$variable1 = function1($ServiceDefinedVariable);
$variable2 = function2($OtherServiceDefinedVariable, $variable1);
```

Incluez ces instructions dans votre formule de mise à l’échelle pour arriver à un nombre cible de nœuds de calcul. Les nœuds dédiés et les nœuds basse priorité ont leurs propres paramètres cibles. Une formule de mise à l’échelle automatique peut inclure une valeur cible pour les nœuds dédiés, une valeur cible pour les nœuds de faible priorité, ou les deux.

Le nombre cible de nœuds peut être supérieur, inférieur ou égal au nombre actuel de nœuds de ce type dans le pool. Le service Batch évalue la formule de mise à l’échelle automatique d’un pool à [intervalles de mise à l’échelle automatique](#automatic-scaling-interval) spécifiques. Batch ajuste ensuite le nombre cible de chaque type de nœud dans le pool en fonction du nombre spécifié par votre formule de mise à l’échelle automatique au moment de l’évaluation.

### <a name="sample-autoscale-formulas"></a>Exemples de formules de mise à l’échelle automatique

Voici des exemples de formules de mise à l’échelle automatique qui peuvent être adaptés pour convenir à la plupart des scénarios. Les variables `startingNumberOfVMs` et `maxNumberofVMs` dans les exemples de formules peuvent être ajustées en fonction de vos besoins.

#### <a name="pending-tasks"></a>Tâches en attente

Avec cette formule de mise à l’échelle automatique, le pool est à l’origine créé avec une seule machine virtuelle. La mesure `$PendingTasks` définit le nombre de tâches en cours d’exécution ou en file d’attente. Cette formule recherche le nombre moyen de tâches en attente au cours des 180 dernières secondes et définit la variable `$TargetDedicatedNodes` en conséquence. La formule garantit que le nombre cible de nœuds dédiés ne dépasse jamais 25 machines virtuelles. À mesure que de nouvelles tâches sont envoyées, le pool s’accroît automatiquement. Lorsque des tâches se terminent, des machines virtuelles se libèrent, et la formule de mise à l’échelle automatique réduit le pool.

Cette formule met à l’échelle des nœuds dédiés, mais elle peut être modifiée pour s’appliquer également aux nœuds de faible priorité.

```
startingNumberOfVMs = 1;
maxNumberofVMs = 25;
pendingTaskSamplePercent = $PendingTasks.GetSamplePercent(180 * TimeInterval_Second);
pendingTaskSamples = pendingTaskSamplePercent < 70 ? startingNumberOfVMs : avg($PendingTasks.GetSample(180 * TimeInterval_Second));
$TargetDedicatedNodes=min(maxNumberofVMs, pendingTaskSamples);
$NodeDeallocationOption = taskcompletion;
```

#### <a name="preempted-nodes"></a>Nœuds reportés

Cet exemple crée un pool qui commence par 25 nœuds basse priorité. Chaque fois qu’un nœud basse priorité est reporté, il est remplacé par un nœud dédié. Comme pour le premier exemple, la variable `maxNumberofVMs` empêche le pool de dépasser 25 machines virtuelles. Cet exemple est utile pour tirer parti des machines virtuelles basse priorité, tout en garantissant que seul un nombre fixe de reports se produit pendant la durée de vie du pool.

```
maxNumberofVMs = 25;
$TargetDedicatedNodes = min(maxNumberofVMs, $PreemptedNodeCount.GetSample(180 * TimeInterval_Second));
$TargetLowPriorityNodes = min(maxNumberofVMs , maxNumberofVMs - $TargetDedicatedNodes);
$NodeDeallocationOption = taskcompletion;
```

Plus loin, cette rubrique explique [comment créer des formules de mise à l’échelle automatique](#write-an-autoscale-formula) et présente d’autres [exemples de formules de mise à l’échelle automatique](#example-autoscale-formulas).

## <a name="variables"></a>Variables

Vous pouvez utiliser aussi bien des variables **définies par le service** que des variables **définies par l’utilisateur** dans les formules de mise à l’échelle automatique.

Les variables définies par le service sont intégrées dans le service Batch. Certaines variables définies par le service sont en lecture-écriture, tandis que d’autres sont en lecture seule.

Les variables définies par l’utilisateur sont des variables que vous définissez. Dans l’exemple de formule ci-dessus, `$TargetDedicatedNodes` et `$PendingTasks` sont des variables définies par le service, tandis que `startingNumberOfVMs` et `maxNumberofVMs` sont des variables définies par l’utilisateur.

> [!NOTE]
> Les variables définies par le service sont toujours précédées d’un signe dollar ($). Pour les variables définies par l’utilisateur, le signe dollar est facultatif.

Les tableaux suivants montrent les variables en lecture-écriture et en lecture seule définies par le service Batch.

### <a name="read-write-service-defined-variables"></a>Variables définies par le service en lecture-écriture

Vous pouvez obtenir et définir les valeurs de ces variables définies par le service pour gérer le nombre de nœuds de calcul dans un pool.

| Variable | Description |
| --- | --- |
| $TargetDedicatedNodes |Nombre cible de nœuds de calcul dédiés pour le pool. Ce nombre est spécifié en tant que cible, car un pool peut ne pas toujours atteindre le nombre de nœuds souhaité. Par exemple, si le nombre cible de nœuds dédiés est modifié par une évaluation de la mise à l’échelle automatique avant que le pool ait atteint la cible initiale, le pool peut ne pas atteindre la cible. <br /><br /> Un pool dans un compte créé en mode service Batch peut ne pas atteindre sa cible si celle-ci dépasse le quota de nœuds ou de cœurs du compte Batch. Un pool dans un compte créé en mode abonnement utilisateur peut ne pas atteindre sa cible si celle-ci dépasse le quota de cœurs partagés pour l’abonnement.|
| $TargetLowPriorityNodes |Nombre cible de nœuds de calcul de faible priorité pour le pool. Le nombre de nœuds dédiés est spécifié en tant que cible, car un pool peut ne pas toujours atteindre le nombre de nœuds souhaité. Par exemple, si le nombre cible de nœuds de faible priorité est modifié par une évaluation de la mise à l’échelle automatique avant que le pool ait atteint la cible initiale, le pool peut ne pas atteindre la cible. Un pool peut également ne pas atteindre sa cible si la cible dépasse un quota de nœuds ou de cœurs défini pour le compte Batch. <br /><br /> Pour plus d’informations sur les nœuds de calcul basse priorité, voir [Utiliser des machines virtuelles basse priorité avec Batch](batch-low-pri-vms.md). |
| $NodeDeallocationOption |Action exécutée lorsque des nœuds de calcul sont supprimés d’un pool. Les valeurs possibles sont les suivantes :<ul><li>**requeue** : valeur par défaut. Arrête immédiatement les tâches et les replace dans la file d’attente des travaux pour qu’elles soient replanifiées. Cette action garantit que le nombre cible de nœuds est atteint aussi rapidement que possible. Toutefois, elle peut être moins efficace, car toutes les tâches en cours d’exécution sont interrompues et doivent ensuite être redémarrées complètement. <li>**terminate** : arrête immédiatement des tâches et les supprime de la file d’attente des travaux.<li>**taskcompletion** : attend la fin des tâches en cours d’exécution, puis supprime le nœud du pool. Utilisez cette option pour éviter l’interruption et la remise en file d’attente des tâches, ce qui gaspille le travail effectué par la tâche.<li>**retaineddata** : attend que toutes les données retenues par la tâche locale sur le nœud aient été nettoyées avant de supprimer le nœud du pool.</ul> |

> [!NOTE]
> La variable `$TargetDedicatedNodes` peut également être spécifiée à l’aide de l’alias `$TargetDedicated`. De même, la variable `$TargetLowPriorityNodes` peut être spécifiée à l’aide de l’alias `$TargetLowPriority`. Si la variable entièrement nommée et son alias sont définis par la formule, la valeur assignée à la variable entièrement nommée est prioritaire.

### <a name="read-only-service-defined-variables"></a>Variables définies par le service en lecture seule

Vous pouvez obtenir la valeur des variables définies par le service ci-après pour effectuer des ajustements basés sur les mesures à partir du service Batch.

> [!IMPORTANT]
> Les tâches de validation du travail ne font pas partie des variables qui indiquent le nombre de tâches, par exemple $ActiveTasks et $PendingTasks. En fonction de votre formule de mise à l’échelle automatique, cela peut entraîner la suppression de nœuds et l’absence de nœuds disponibles pour exécuter les tâches de validation du travail.

| Variable | Description |
| --- | --- |
| $CPUPercent |Pourcentage moyen d’utilisation du processeur. |
| $WallClockSeconds |Nombre de secondes consommées. |
| $MemoryBytes |Nombre moyen de mégaoctets utilisés. |
| $DiskBytes |Nombre moyen de gigaoctets utilisés sur les disques locaux. |
| $DiskReadBytes |Nombre d'octets lus. |
| $DiskWriteBytes |Nombre d’octets écrits. |
| $DiskReadOps |Nombre d’opérations de lecture sur disque effectuées. |
| $DiskWriteOps |Nombre d’opérations d’écriture sur disque effectuées. |
| $NetworkInBytes |Nombre d’octets entrants. |
| $NetworkOutBytes |Nombre d’octets sortants. |
| $SampleNodeCount |Nombre de nœuds de calcul. |
| $ActiveTasks |Nombre de tâches prêtes à être exécutées, mais pas encore en cours d’exécution. Cette valeur inclut toutes les tâches qui se trouvent dans un état actif et dont les dépendances ont été satisfaites. Toutes les tâches qui se trouvent dans un état actif mais dont les dépendances n’ont pas été satisfaites sont exclues de la valeur $ActiveTasks. Dans le cas d’une tâche multi-instance, $ActiveTasks inclura le nombre d’instances définies dans la tâche.|
| $RunningTasks |Nombre de tâches en cours d’exécution. |
| $PendingTasks |Somme de $ActiveTasks et de $RunningTasks. |
| $SucceededTasks |Nombre de tâches ayant abouti. |
| $FailedTasks |Nombre de tâches ayant échoué. |
| $CurrentDedicatedNodes |Nombre actuel de nœuds de calcul dédiés. |
| $CurrentLowPriorityNodes |Nombre actuel de nœuds de calcul de faible priorité, y compris tous les nœuds réalloués. |
| $PreemptedNodeCount | Nombre de nœuds dans le pool qui sont à l’état Reporté. |

> [!TIP]
> Ces variables en lecture seule définies par le service sont des *objets* qui fournissent diverses méthodes pour accéder aux données qui leur sont associées. Pour plus d’informations, consultez la section [Obtenir des échantillons de données](#obtain-sample-data) dans la suite de cet article.

> [!NOTE]
> Utilisez `$RunningTasks` lors d’une mise à l’échelle en fonction du nombre de tâches exécutées à un moment donné, et `$ActiveTasks` lors d’une mise à l’échelle en fonction du nombre de tâches mises en file d’attente pour s’exécuter.

## <a name="types"></a>Types

Les formules de mise à l’échelle automatique prennent en charge les types suivants :

- double
- doubleVec
- doubleVecList
- string
- timestamp, structure composée comprenant les éléments suivants :
  - year
  - mois (1-12)
  - jour (1-31)
  - jour de la semaine (sous forme de chiffre, par exemple 1 pour lundi)
  - heure (au format 24 heures, par exemple, 13 signifie 1 PM)
  - minute (00-59)
  - seconde (00-59)
- timeinterval
  - TimeInterval_Zero
  - TimeInterval_100ns
  - TimeInterval_Microsecond
  - TimeInterval_Millisecond
  - TimeInterval_Second
  - TimeInterval_Minute
  - TimeInterval_Hour
  - TimeInterval_Day
  - TimeInterval_Week
  - TimeInterval_Year

## <a name="operations"></a>Opérations

Les opérations autorisées sur les types répertoriés dans la section précédente sont les suivantes :

| Opération | Opérateurs pris en charge | Type de résultat |
| --- | --- | --- |
| double *opérateur* double |+, -, *, / |double |
| double *opérateur* timeinterval |* |timeinterval |
| doubleVec *opérateur* double |+, -, *, / |doubleVec |
| doubleVec *opérateur* doubleVec |+, -, *, / |doubleVec |
| timeinterval *opérateur* double |*, / |timeinterval |
| timeinterval *opérateur* timeinterval |+, - |timeinterval |
| timeinterval *opérateur* timestamp |+ |timestamp |
| timestamp *opérateur* timeinterval |+ |timestamp |
| timestamp *opérateur* timestamp |- |timeinterval |
| *operateur* double |-, ! |double |
| *operateur* timeinterval |- |timeinterval |
| double *opérateur* double |<, <=, ==, >=, >, != |double |
| string *opérateur* string |<, <=, ==, >=, >, != |double |
| timestamp *opérateur* timestamp |<, <=, ==, >=, >, != |double |
| timeinterval *opérateur* timeinterval |<, <=, ==, >=, >, != |double |
| double *opérateur* double |&&, &#124;&#124; |double |

Quand vous testez un double avec un opérateur ternaire (`double ? statement1 : statement2`), la valeur différente de zéro est **true** et zéro est **false**.

## <a name="functions"></a>Fonctions

Vous pouvez utiliser ces **fonctions** prédéfinies lors de la définition d’une formule de mise à l’échelle automatique.

| Fonction | Type de retour | Description |
| --- | --- | --- |
| avg(doubleVecList) |double |Retourne la valeur moyenne de toutes les valeurs de l’élément doubleVecList. |
| len(doubleVecList) |double |Retourne la longueur du vecteur créé à partir de l’élément doubleVecList. |
| lg(double) |double |Retourne la base logarithmique 2 de l’élément double. |
| lg(doubleVecList) |doubleVec |Retourne la base logarithmique 2 au niveau composant de l’élément doubleVecList. Un élément vec(double) doit être explicitement transmis pour le paramètre. Dans le cas contraire, la version double lg(double) est prise en compte. |
| ln(double) |double |Retourne le logarithme naturel de l’élément double. |
| ln(doubleVecList) |doubleVec |Retourne le logarithme naturel de l’élément double. |
| log(double) |double |Retourne la base logarithmique 10 de l’élément double. |
| log(doubleVecList) |doubleVec |Retourne la base logarithmique 10 au niveau composant de l’élément doubleVecList. Un élément vec(double) doit être explicitement transmis pour le paramètre de type double unique. Dans le cas contraire, la version double log(double) est prise en compte. |
| max(doubleVecList) |double |Retourne la valeur maximale de l’élément doubleVecList. |
| min(doubleVecList) |double |Retourne la valeur minimale de l’élément doubleVecList. |
| norm(doubleVecList) |double |Retourne la double norme du vecteur créé à partir de l’élément doubleVecList. |
| percentile(doubleVec v, double p) |double |Retourne l’élément de percentile du vecteur v. |
| rand() |double |Retourne une valeur aléatoire comprise entre 0,0 et 1,0. |
| range(doubleVecList) |double |Retourne la différence entre les valeurs minimale et maximale de l’élément doubleVecList. |
| std(doubleVecList) |double |Retourne l’écart type de l’échantillon des valeurs dans l’élément doubleVecList. |
| stop() | |Arrête l’évaluation de l’expression de mise à l’échelle automatique. |
| sum(doubleVecList) |double |Retourne la somme de tous les composants de l’élément doubleVecList. |
| time(string dateTime="") |timestamp |Retourne l’horodatage de l’heure actuelle si aucun paramètre n’est transmis, ou l’horodatage de la chaîne dateTime dans le cas contraire. Les formats dateTime pris en charge sont W3C-DTF et RFC 1123. |
| val(doubleVec v, double i) |double |Retourne la valeur de l’élément qui est à l’emplacement i du vecteur v avec un index de départ de zéro. |

Certaines des fonctions décrites dans le tableau précédent peuvent accepter une liste en tant qu’argument. La liste séparée par des virgules se compose de n’importe quelle combinaison d’éléments *double* et *doubleVec*. Par exemple :

`doubleVecList := ( (double | doubleVec)+(, (double | doubleVec) )* )?`

La valeur *doubleVecList* est convertie en un seul paramètre *doubleVec* avant l’évaluation. Par exemple, si `v = [1,2,3]`, alors appeler `avg(v)` revient à appeler `avg(1,2,3)`. Appeler `avg(v, 7)` équivaut à appeler `avg(1,2,3,7)`.

## <a name="metrics"></a>Mesures

Vous pouvez utiliser à la fois les mesures de ressources et de tâches quand vous définissez une formule. Vous ajustez le nombre cible de nœuds dédiés dans le pool en fonction des données métriques que vous obtenez et évaluez. Pour plus d’informations sur chaque métrique, consultez la section [Variables](#variables).

<table>
  <tr>
    <th>Métrique</th>
    <th>Description</th>
  </tr>
  <tr>
    <td><b>Ressource</b></td>
    <td><p>Les mesures de ressources sont basées sur l’utilisation du processeur, de la bande passante et de la mémoire par les nœuds de calcul, ainsi que sur le nombre de nœuds.</p>
        <p> Ces variables définies par le service sont utiles pour effectuer des ajustements en fonction du nombre de nœuds :</p>
    <p><ul>
            <li>$TargetDedicatedNodes</li>
            <li>$TargetLowPriorityNodes</li>
            <li>$CurrentDedicatedNodes</li>
            <li>$CurrentLowPriorityNodes</li>
            <li>$PreemptedNodeCount</li>
            <li>$SampleNodeCount</li>
    </ul></p>
    <p>Ces variables définies par le service sont utilisées pour effectuer des ajustements en fonction de l’utilisation des ressources du nœud :</p>
    <p><ul>
      <li>$CPUPercent</li>
      <li>$WallClockSeconds</li>
      <li>$MemoryBytes</li>
      <li>$DiskBytes</li>
      <li>$DiskReadBytes</li>
      <li>$DiskWriteBytes</li>
      <li>$DiskReadOps</li>
      <li>$DiskWriteOps</li>
      <li>$NetworkInBytes</li>
      <li>$NetworkOutBytes</li></ul></p>
  </tr>
  <tr>
    <td><b>Tâche</b></td>
    <td><p>Les mesures de tâches sont basées sur l’état des tâches (Active, En attente et Terminée). Les variables suivantes définies par le service sont utiles pour ajuster la taille du pool en fonction des métriques de tâche :</p>
    <p><ul>
      <li>$ActiveTasks</li>
      <li>$RunningTasks</li>
      <li>$PendingTasks</li>
      <li>$SucceededTasks</li>
            <li>$FailedTasks</li></ul></p>
        </td>
  </tr>
</table>

## <a name="obtain-sample-data"></a>Obtenir des échantillons de données

La principale opération d’une formule de mise à l’échelle automatique vise à obtenir des données métriques (échantillons) des tâches et des ressources, puis à ajuster la taille du pool en fonction de ces données. Par conséquent, il est important de comprendre clairement comment les formules de mise à l’échelle automatique interagissent avec les échantillons.

### <a name="methods"></a>Méthodes

Les formules de mise à l’échelle automatique agissent sur des échantillons de données métriques fournies par le service Batch. Une formule augmente ou rédui la taille du pool en fonction des valeurs obtenues. Les variables définies par le service sont des objets qui fournissent des méthodes permettant d’accéder aux données associées à cet objet. Par exemple, l’expression ci-après présente une requête visant à obtenir les cinq dernières minutes de l’utilisation du processeur :

```
$CPUPercent.GetSample(TimeInterval_Minute * 5)
```

Les méthodes suivantes permettent d’obtenir des échantillons de données sur les variables définies par le service.

| Méthode | Description |
| --- | --- |
| GetSample() |La méthode `GetSample()` retourne un vecteur d’échantillons de données.<br/><br/>Un échantillon correspond à 30 secondes de données de métrique. En d’autres termes, des échantillons sont obtenus toutes les 30 secondes, mais comme noté ci-dessus, il existe un délai entre le moment où un échantillon est collecté et le moment où il est disponible pour une formule. Par conséquent, tous les échantillons pour une période donnée ne sont pas forcément disponibles pour évaluation par une formule.<ul><li>`doubleVec GetSample(double count)`: Spécifie le nombre d’échantillons à obtenir à partir des échantillons collectés les plus récents. `GetSample(1)` retourne le dernier échantillon disponible. Toutefois, pour des métriques telles que `$CPUPercent`, la méthode `GetSample(1)` ne doit pas être utilisée, car il est impossible de savoir *quand* l’échantillon a été collecté. Il peut s’agir d’un événement récent, ou plus ancien en raison de problèmes système. Dans ce cas, il est préférable d’utiliser un intervalle de temps comme indiqué ci-dessous.<li>`doubleVec GetSample((timestamp or timeinterval) startTime [, double samplePercent])`: Spécifie le délai d’exécution de la collecte des exemples de données. Elle spécifie éventuellement le pourcentage d’échantillons qui doivent être disponibles dans le délai d’exécution demandé. Par exemple, l’intervalle `$CPUPercent.GetSample(TimeInterval_Minute * 10)` doit retourner 20 échantillons si tous les échantillons des 10 dernières minutes sont présents dans l’historique `CPUPercent`. Si la dernière minute de l’historique n’est pas disponible, seuls 18 échantillons sont renvoyés. Dans ce cas, l’intervalle `$CPUPercent.GetSample(TimeInterval_Minute * 10, 95)` échoue car seuls 90 % des exemples sont disponibles, mais l’intervalle `$CPUPercent.GetSample(TimeInterval_Minute * 10, 80)` réussit.<li>`doubleVec GetSample((timestamp or timeinterval) startTime, (timestamp or timeinterval) endTime [, double samplePercent])`: Spécifie le délai d’exécution de la collecte des données avec une heure de début et une heure de fin. Comme indiqué ci-dessus, il existe un délai entre le moment où un échantillon est collecté et le moment où il devient disponible pour une formule. Envisagez ce délai lorsque vous utilisez la méthode `GetSample`. Consultez `GetSamplePercent` ci-dessous. |
| GetSamplePeriod() |Retourne la période des échantillons considérés dans un jeu de données d’échantillon historiques. |
| Count() |Renvoie le nombre total d’échantillons dans l’historique des métriques. |
| HistoryBeginTime() |Retourne l’horodateur du plus ancien échantillon de données disponible pour la métrique. |
| GetSamplePercent() |Retourne le pourcentage d’échantillons disponibles pour un intervalle de temps donné. Par exemple : `doubleVec GetSamplePercent( (timestamp or timeinterval) startTime [, (timestamp or timeinterval) endTime] )`. Comme la méthode `GetSample` échoue si le pourcentage d’échantillons retourné est inférieur au `samplePercent` spécifié, vous pouvez utiliser la méthode `GetSamplePercent` pour procéder d’abord à une vérification. Vous pouvez ensuite effectuer une autre action si des échantillons insuffisants sont présents, sans arrêter l’évaluation de la mise à l’échelle automatique. |

### <a name="samples"></a>Exemples

Le service Batch prélève régulièrement des échantillons de mesures de tâches et de ressources pour les mettre à la disposition de vos formules de mise à l’échelle automatique. Ces échantillons sont enregistrés toutes les 30 secondes par le service Batch. Cependant, un retard est généralement constaté entre l’enregistrement de ces échantillons et leur mise à disposition (en lecture) pour vos formules de mise à l’échelle automatique. De plus, en raison de différents facteurs tels que des problèmes de réseau ou d’autres problèmes d’infrastructure, il arrive que des échantillons ne soient pas enregistrés pour un intervalle donné.

### <a name="sample-percentage"></a>Pourcentage d’échantillonnage

Quand `samplePercent` est transmis à la méthode `GetSample()` ou que la méthode `GetSamplePercent()` est appelée, le terme _pourcentage_ désigne une comparaison entre le nombre total possible d’échantillons enregistrés par le service Batch et le nombre d’échantillons effectivement disponibles pour la formule de mise à l’échelle automatique.

Prenons l’exemple d’un intervalle de 10 minutes. Comme les échantillons sont enregistrés toutes les 30 secondes, au cours de cet intervalle de 10 minutes, le nombre total maximal d’échantillons enregistrés par Batch devrait être de 20 échantillons (soit 2 par minute). Toutefois, en raison de la latence inhérente au mécanisme de création de rapports, ou d’un autre problème au sein d’Azure, il se peut que 15 échantillons seulement soient disponibles en lecture pour votre formule de mise à l’échelle automatique. Ainsi, pour cette période de 10 minutes, seuls 75 % du nombre total d’échantillons enregistrés seraient réellement disponibles pour votre formule.

### <a name="getsample-and-sample-ranges"></a>Méthode GetSample() et plages d’échantillonnage

Vos formules de mise à l’échelle automatique vont augmenter ou réduire vos pools en ajoutant ou en supprimant des nœuds. Comme les nœuds vous coûtent de l’argent, assurez-vous que vos formules utilisent une méthode d’analyse intelligente basée sur des données suffisantes. Nous vous recommandons d’utiliser une analyse de type tendance dans vos formules. Ce type augmente ou réduit vos pools en fonction d’une plage d’échantillons collectés.

Pour ce faire, utilisez `GetSample(interval look-back start, interval look-back end)` pour retourner un vecteur d’échantillons :

```
$runningTasksSample = $RunningTasks.GetSample(1 * TimeInterval_Minute, 6 * TimeInterval_Minute);
```

Lorsque Batch évalue la ligne ci-dessus, il retourne une plage d’exemples sous la forme d’un vecteur de valeurs. Par exemple :

```
$runningTasksSample=[1,1,1,1,1,1,1,1,1,1];
```

Une fois que vous avez collecté le vecteur d’échantillons, vous pouvez utiliser des fonctions telles que `min()`, `max()` et `avg()` pour dériver des valeurs significatives à partir de la plage collectée.

Pour plus de sécurité, vous pouvez forcer l’échec d’une évaluation de formule si le pourcentage d’échantillons disponible pendant une période donnée est inférieur à un certain seuil. Lorsque vous forcez l’échec d’une évaluation de formule, Batch reçoit l’instruction de cesser toute nouvelle évaluation de la formule si le pourcentage d’exemples spécifié n’est pas disponible. Dans ce cas, aucune modification n’est apportée à la taille du pool. Pour spécifier un pourcentage d’échantillons à respecter pour que l’évaluation aboutisse, spécifiez ce pourcentage en tant que troisième paramètre de `GetSample()`. Dans l’exemple ci-dessous, une exigence de 75 pour cent d’échantillons est spécifiée :

```
$runningTasksSample = $RunningTasks.GetSample(60 * TimeInterval_Second, 120 * TimeInterval_Second, 75);
```

Étant donné qu’il peut y avoir un délai de disponibilité des échantillons, vous devriez toujours spécifier un intervalle de temps avec une heure de début différé remontant à plus d’une minute. Il faut environ une minute aux exemples pour se propager dans le système, ce qui signifie que les exemples situés dans la plage `(0 * TimeInterval_Second, 60 * TimeInterval_Second)` ne sont pas toujours disponibles. Là encore, vous pouvez utiliser le paramètre pourcentage de `GetSample()` pour forcer une exigence de pourcentage d’échantillon particulière.

> [!IMPORTANT]
> Nous vous conseillons fortement d’**éviter de vous appuyer *uniquement* sur `GetSample(1)` dans vos formules de mise à l’échelle automatique**, car la méthode `GetSample(1)` dit globalement au service Batch : « Donne-moi le dernier exemple disponible, quelle que soit son ancienneté ». Dans la mesure où il s’agit uniquement d’un simple échantillon (potentiellement ancien), il risque de ne pas être représentatif de l’état récent de la tâche ou de la ressource. Si vous utilisez tout de même `GetSample(1)`, veillez à l’intégrer dans une instruction plus générale pour éviter de l’utiliser comme unique point de données sur lequel reposera votre formule.

## <a name="write-an-autoscale-formula"></a>Écrire une formule de mise à l’échelle automatique

Vous générez une formule de mise à l’échelle automatique en formant des instructions qui utilisent les composants ci-dessus, puis vous combinez ces instructions dans une formule complète. Dans cette section, nous créons un exemple de formule de mise à l’échelle automatique qui peut prendre des décisions de mise à l’échelle concrètes et apporter des ajustements.

Commençons par définir les exigences de notre nouvelle formule de mise à l’échelle automatique. La formule doit :

- Augmenter le nombre cible de nœuds de calcul dédiés dans un pool si l’utilisation du processeur est intensive.
- Réduire le nombre cible de nœuds de calcul dédiés dans un pool si l’utilisation du processeur est faible.
- Toujours limiter le nombre maximal de nœuds dédiés à 400.
- Lorsque vous réduisez le nombre de nœuds, ne supprimez pas de nœuds qui exécutent des tâches. Si nécessaire, attendez que les tâches soient terminées avant de supprimer des nœuds.

La première instruction de notre formule augmente le nombre de nœuds lors d’une utilisation intensive de l’UC. Nous allons définir une instruction qui renseigne une variable définie par l’utilisateur (`$totalDedicatedNodes`), qui utilise une valeur équivalente à 110 % du nombre cible actuel de nœuds dédiés, mais uniquement si l’utilisation moyenne minimale du processeur de l’UC au cours des 10 dernières minutes était supérieure à 70 %. Autrement, la variable utilise la valeur du nombre actuel de nœuds dédiés.

```
$totalDedicatedNodes =
    (min($CPUPercent.GetSample(TimeInterval_Minute * 10)) > 0.7) ?
    ($CurrentDedicatedNodes * 1.1) : $CurrentDedicatedNodes;
```

Pour réduire le nombre de nœuds dédiés en cas de faible utilisation du processeur, la prochaine instruction dans notre formule définit la même variable `$totalDedicatedNodes` sur 90 % du nombre cible actuel de nœuds dédiés si l’utilisation moyenne du processeur au cours des 60 dernières minutes était inférieure à 20 %. Autrement, elle utilise la valeur actuelle de `$totalDedicatedNodes` que nous avons renseignée dans l’instruction ci-dessus.

```
$totalDedicatedNodes =
    (avg($CPUPercent.GetSample(TimeInterval_Minute * 60)) < 0.2) ?
    ($CurrentDedicatedNodes * 0.9) : $totalDedicatedNodes;
```

À présent, nous allons limiter le nombre cible de nœuds de calcul dédiés à maximum 400.

```
$TargetDedicatedNodes = min(400, $totalDedicatedNodes)
```

Enfin, nous allons veiller à ce que les nœuds ne soient pas supprimés tant que leurs tâches ne sont pas terminées.

```
$NodeDeallocationOption = taskcompletion;
```

Voici la formule complète :

```
$totalDedicatedNodes =
    (min($CPUPercent.GetSample(TimeInterval_Minute * 10)) > 0.7) ?
    ($CurrentDedicatedNodes * 1.1) : $CurrentDedicatedNodes;
$totalDedicatedNodes =
    (avg($CPUPercent.GetSample(TimeInterval_Minute * 60)) < 0.2) ?
    ($CurrentDedicatedNodes * 0.9) : $totalDedicatedNodes;
$TargetDedicatedNodes = min(400, $totalDedicatedNodes)
$NodeDeallocationOption = taskcompletion;
```

> [!NOTE]
> Si vous le souhaitez, vous pouvez inclure des commentaires et des sauts de ligne dans des chaînes de formule. Sachez également que des points-virgules manquants peuvent entraîner des erreurs d’évaluation.

## <a name="automatic-scaling-interval"></a>Intervalle de mise à l’échelle automatique

Par défaut, le service Batch ajuste la taille d’un pool en fonction de sa formule de mise à l’échelle toutes les 15 minutes. Cet intervalle peut être configuré à l’aide des propriétés de pool suivantes :

- [CloudPool.AutoScaleEvaluationInterval](/dotnet/api/microsoft.azure.batch.cloudpool.autoscaleevaluationinterval) (Batch .NET)
- [autoScaleEvaluationInterval](/rest/api/batchservice/enable-automatic-scaling-on-a-pool) (API REST)

L’intervalle doit être compris entre cinq minutes et 168 heures. Si un intervalle en dehors de cette plage est spécifié, le service Batch renvoie une erreur de demande incorrecte (400).

> [!NOTE]
> La mise à l’échelle automatique ne peut pas actuellement répondre aux modifications en moins d’une minute, mais vise plutôt à ajuster progressivement la taille de votre pool pendant l’exécution d’une charge de travail.

## <a name="create-an-autoscale-enabled-pool-with-batch-sdks"></a>Créer un pool avec mise à l’échelle automatique avec des Kits de développement logiciel (SDK) Batch

La mise à l’échelle automatique du pool peut être configurée à l’ aide d’un des [Kits de développement logiciel (SDK) Batch](batch-apis-tools.md#azure-accounts-for-batch-development), des [cmdlets PowerShell Batch](/rest/api/batchservice/) de l’[API Rest Batch](batch-powershell-cmdlets-get-started.md) et de l’interface [CLI Batch](batch-cli-get-started.md). Cette section vous propose des exemples pour .NET et Python.

### <a name="net"></a>.NET

Pour créer un pool avec mise à l’échelle automatique dans .NET, procédez comme suit :

1. Créez le pool avec [BatchClient.PoolOperations.CreatePool](/dotnet/api/microsoft.azure.batch.pooloperations.createpool).
1. Affectez à la propriété [CloudPool.AutoScaleEnabled](/dotnet/api/microsoft.azure.batch.cloudpool.autoscaleenabled) la valeur `true`.
1. Affectez à la propriété [CloudPool.AutoScaleFormula](/dotnet/api/microsoft.azure.batch.cloudpool.autoscaleformula) votre formule de mise à l’échelle automatique.
1. (Facultatif) Définissez la propriété [CloudPool.AutoScaleEvaluationInterval](/dotnet/api/microsoft.azure.batch.cloudpool.autoscaleevaluationinterval) (la valeur par défaut est de 15 minutes).
1. Validez le pool avec [CloudPool.Commit](/dotnet/api/microsoft.azure.batch.cloudpool.commit) ou [CommitAsync](/dotnet/api/microsoft.azure.batch.cloudpool.commitasync).

L’exemple suivant crée un pool avec mise à l’échelle automatique dans .NET. La formule de mise à l’échelle automatique du pool définit le nombre cible de nœuds dédiés sur 5 les lundis et 1 les autres jours de la semaine. L’[intervalle de mise à l’échelle automatique](#automatic-scaling-interval) est de 30 minutes. Dans cet extrait de code C# et les autres extraits de cet article, `myBatchClient` est une instance entièrement initialisée de la classe [BatchClient](/dotnet/api/microsoft.azure.batch.batchclient).

```csharp
CloudPool pool = myBatchClient.PoolOperations.CreatePool(
                    poolId: "mypool",
                    virtualMachineSize: "standard_d1_v2",
                    cloudServiceConfiguration: new CloudServiceConfiguration(osFamily: "5"));
pool.AutoScaleEnabled = true;
pool.AutoScaleFormula = "$TargetDedicatedNodes = (time().weekday == 1 ? 5:1);";
pool.AutoScaleEvaluationInterval = TimeSpan.FromMinutes(30);
await pool.CommitAsync();
```

> [!IMPORTANT]
> Lorsque vous créez un pool avec mise à l’échelle automatique, ne spécifiez pas le paramètre _targetDedicatedNodes_ ou le paramètre _targetLowPriorityNodes_ sur l’appel à **CreatePool**. Au lieu de cela, spécifiez les propriétés **AutoScaleEnabled** et **AutoScaleFormula** sur le pool. Les valeurs de ces propriétés déterminent le nombre cible de chaque type de nœud.
>
> Pour redimensionner manuellement un pool avec mise à l’échelle automatique (par exemple, avec [BatchClient.PoolOperations.ResizePoolAsync](/dotnet/api/microsoft.azure.batch.pooloperations.resizepoolasync)), vous devez commencer par désactiver la mise à l’échelle automatique dans le pool avant de le redimensionner.

### <a name="python"></a>Python

Pour créer un pool avec mise à l’échelle automatique avec le kit de développement logiciel (SDK) Python :

1. Créez un pool et spécifiez sa configuration.
1. Ajoutez le pool au client du service.
1. Activez la mise à l’échelle automatique sur le pool avec une formule que vous écrivez.

L’exemple suivant illustre ces étapes.

```python
# Create a pool; specify configuration
new_pool = batch.models.PoolAddParameter(
    id="autoscale-enabled-pool",
    virtual_machine_configuration=batchmodels.VirtualMachineConfiguration(
        image_reference=batchmodels.ImageReference(
          publisher="Canonical",
          offer="UbuntuServer",
          sku="18.04-LTS",
          version="latest"
            ),
        node_agent_sku_id="batch.node.ubuntu 18.04"),
    vm_size="STANDARD_D1_v2",
    target_dedicated_nodes=0,
    target_low_priority_nodes=0
)
batch_service_client.pool.add(new_pool) # Add the pool to the service client

formula = """$curTime = time();
             $workHours = $curTime.hour >= 8 && $curTime.hour < 18;
             $isWeekday = $curTime.weekday >= 1 && $curTime.weekday <= 5;
             $isWorkingWeekdayHour = $workHours && $isWeekday;
             $TargetDedicated = $isWorkingWeekdayHour ? 20:10;""";

# Enable autoscale; specify the formula
response = batch_service_client.pool.enable_auto_scale(pool_id, auto_scale_formula=formula,
                                            auto_scale_evaluation_interval=datetime.timedelta(minutes=10),
                                            pool_enable_auto_scale_options=None,
                                            custom_headers=None, raw=False)
```

> [!TIP]
> Vous trouverez d’autres exemples d’utilisation du Kit de développement logiciel (SDK) Python dans le [référentiel Démarrage rapide Python Batch](https://github.com/Azure-Samples/batch-python-quickstart) sur GitHub.

## <a name="enable-autoscaling-on-an-existing-pool"></a>Activer la mise à l’échelle automatique sur un pool existant

Chaque Kit de développement logiciel (SDK) Batch fournit un moyen d’activer la mise à l’échelle automatique. Par exemple :

- [BatchClient.PoolOperations.EnableAutoScaleAsync](/dotnet/api/microsoft.azure.batch.pooloperations.enableautoscaleasync) (Batch .NET)
- [Activer la mise à l’échelle automatique sur un pool](/rest/api/batchservice/enable-automatic-scaling-on-a-pool) (API REST)

Quand vous activez la mise à l’échelle automatique sur un pool existant, n’oubliez pas les points suivants :

- Si la mise à l’échelle automatique est actuellement désactivée sur le pool, vous devez spécifier une formule de mise à l’échelle automatique valide quand vous émettez la demande. Vous pouvez éventuellement spécifier un intervalle de mise à l’échelle automatique. Si vous ne spécifiez aucun intervalle, la valeur par défaut de 15 minutes est utilisée.
- Si la mise à l’échelle automatique est actuellement activée sur le pool, vous pouvez spécifier une nouvelle formule, un nouvel intervalle, ou les deux. Vous devez spécifier au moins l’une de ces propriétés.
  - Si vous spécifiez un nouvel intervalle de mise à l’échelle automatique, la planification existante est arrêtée et une nouvelle planification est démarrée. L’heure de début de la nouvelle planification est l’heure à laquelle la requête d’activation de mise à l’échelle automatique a été émise.
  - Si vous omettez la formule de mise à l’échelle automatique ou l’intervalle, le service Batch continue d’utiliser la valeur actuelle de ce paramètre.

> [!NOTE]
> Si vous avez spécifié des valeurs pour les paramètres *targetDedicatedNodes* ou *targetLowPriorityNodes* de la méthode **CreatePool** lorsque vous avez créé le pool dans .NET, ou pour les paramètres comparables dans un autre langage, ces valeurs sont ignorées lors de l’évaluation de la formule de mise à l’échelle automatique.

Cet exemple en C# utilise la bibliothèque [Batch .NET](/dotnet/api/microsoft.azure.batch) pour activer la mise à l’échelle automatique sur un pool existant.

```csharp
// Define the autoscaling formula. This formula sets the target number of nodes
// to 5 on Mondays, and 1 on every other day of the week
string myAutoScaleFormula = "$TargetDedicatedNodes = (time().weekday == 1 ? 5:1);";

// Set the autoscale formula on the existing pool
await myBatchClient.PoolOperations.EnableAutoScaleAsync(
    "myexistingpool",
    autoscaleFormula: myAutoScaleFormula);
```

### <a name="update-an-autoscale-formula"></a>Mettre à jour une formule de mise à l’échelle automatique

Pour mettre à jour la formule sur un pool existant avec mise à l’échelle automatique, appelez l’opération pour réactiver la mise à l’échelle automatique à l’aide de la nouvelle formule. Par exemple, si la mise à l’échelle automatique est déjà activée sur `myexistingpool` quand le code .NET suivant est exécuté, la formule de mise à l’échelle automatique est remplacée par le contenu de `myNewFormula`.

```csharp
await myBatchClient.PoolOperations.EnableAutoScaleAsync(
    "myexistingpool",
    autoscaleFormula: myNewFormula);
```

### <a name="update-the-autoscale-interval"></a>Mettre à jour l’intervalle de mise à l’échelle automatique

Pour mettre à jour l’intervalle d’évaluation de mise à l’échelle d’un pool existant avec mise à l’échelle automatique, appelez l’opération afin de réactiver la mise à l’échelle automatique à l’aide du nouvel intervalle. Par exemple, si vous souhaitez définir un intervalle d’évaluation de 60 minutes pour un pool avec mise à l’échelle automatique dans .NET :

```csharp
await myBatchClient.PoolOperations.EnableAutoScaleAsync(
    "myexistingpool",
    autoscaleEvaluationInterval: TimeSpan.FromMinutes(60));
```

## <a name="evaluate-an-autoscale-formula"></a>Évaluer une formule de mise à l’échelle automatique

Vous pouvez évaluer une formule avant de l’appliquer à un pool. Cela vous permet de tester les résultats de la formule avant de la mettre en production.

Avant d’évaluer une formule de mise à l’échelle automatique, vous devez activer la mise à l’échelle automatique sur le pool à l’aide d’une formule valide, telle que la formule d’une seule ligne `$TargetDedicatedNodes = 0`. Ensuite, utilisez l’une des méthodes suivantes pour évaluer la formule à tester :

- [BatchClient.PoolOperations.EvaluateAutoScale](/dotnet/api/microsoft.azure.batch.pooloperations.evaluateautoscale) ou [EvaluateAutoScaleAsync](/dotnet/api/microsoft.azure.batch.pooloperations.evaluateautoscaleasync)

    Ces méthodes Batch .NET nécessitent l’ID d’un pool existant et une chaîne contenant la formule de mise à l’échelle automatique à évaluer.

- [Évaluer une formule de mise à l’échelle automatique](/rest/api/batchservice/evaluate-an-automatic-scaling-formula)

    Dans cette requête API REST, spécifiez l’ID du pool dans l’URI et la formule de mise à l’échelle automatique dans l’élément *autoScaleFormula* du corps de la requête. La réponse de l’opération contient les éventuelles informations d’erreur associées à la formule.

Cet exemple [Batch .NET](/dotnet/api/microsoft.azure.batch) évalue une formule de mise à l’échelle automatique. Si le pool n’utilise pas encore la mise à l’échelle automatique, nous commençons par l’activer.

```csharp
// First obtain a reference to an existing pool
CloudPool pool = await batchClient.PoolOperations.GetPoolAsync("myExistingPool");

// If autoscaling isn't already enabled on the pool, enable it.
// You can't evaluate an autoscale formula on a non-autoscale-enabled pool.
if (pool.AutoScaleEnabled == false)
{
    // We need a valid autoscale formula to enable autoscaling on the
    // pool. This formula is valid, but won't resize the pool:
    await pool.EnableAutoScaleAsync(
        autoscaleFormula: "$TargetDedicatedNodes = $CurrentDedicatedNodes;",
        autoscaleEvaluationInterval: TimeSpan.FromMinutes(5));

    // Batch limits EnableAutoScaleAsync calls to once every 30 seconds.
    // Because we want to apply our new autoscale formula below if it
    // evaluates successfully, and we *just* enabled autoscaling on
    // this pool, we pause here to ensure we pass that threshold.
    Thread.Sleep(TimeSpan.FromSeconds(31));

    // Refresh the properties of the pool so that we've got the
    // latest value for AutoScaleEnabled
    await pool.RefreshAsync();
}

// We must ensure that autoscaling is enabled on the pool prior to
// evaluating a formula
if (pool.AutoScaleEnabled == true)
{
    // The formula to evaluate - adjusts target number of nodes based on
    // day of week and time of day
    string myFormula = @"
        $curTime = time();
        $workHours = $curTime.hour >= 8 && $curTime.hour < 18;
        $isWeekday = $curTime.weekday >= 1 && $curTime.weekday <= 5;
        $isWorkingWeekdayHour = $workHours && $isWeekday;
        $TargetDedicatedNodes = $isWorkingWeekdayHour ? 20:10;
    ";

    // Perform the autoscale formula evaluation. Note that this code does not
    // actually apply the formula to the pool.
    AutoScaleRun eval =
        await batchClient.PoolOperations.EvaluateAutoScaleAsync(pool.Id, myFormula);

    if (eval.Error == null)
    {
        // Evaluation success - print the results of the AutoScaleRun.
        // This will display the values of each variable as evaluated by the
        // autoscale formula.
        Console.WriteLine("AutoScaleRun.Results: " +
            eval.Results.Replace("$", "\n    $"));

        // Apply the formula to the pool since it evaluated successfully
        await batchClient.PoolOperations.EnableAutoScaleAsync(pool.Id, myFormula);
    }
    else
    {
        // Evaluation failed, output the message associated with the error
        Console.WriteLine("AutoScaleRun.Error.Message: " +
            eval.Error.Message);
    }
}
```

Une évaluation réussie de la formule indiquée dans cet extrait de code produit des résultats du type suivant :

```
AutoScaleRun.Results:
    $TargetDedicatedNodes=10;
    $NodeDeallocationOption=requeue;
    $curTime=2016-10-13T19:18:47.805Z;
    $isWeekday=1;
    $isWorkingWeekdayHour=0;
    $workHours=0
```

## <a name="get-information-about-autoscale-runs"></a>Obtenir des informations sur les exécutions de mise à l’échelle automatique

Pour vérifier que votre formule fonctionne comme prévu, nous vous recommandons d’examiner régulièrement les résultats des exécutions de mise à l’échelle automatique effectuées par Batch sur votre pool. Pour ce faire, obtenez (ou actualisez) une référence au pool, puis examinez les propriétés de la dernière exécution de mise à l’échelle automatique.

Dans Batch .NET, la propriété [CloudPool.AutoScaleRun](/dotnet/api/microsoft.azure.batch.cloudpool.autoscalerun) comprend plusieurs propriétés qui fournissent des informations sur la dernière exécution de mise à l’échelle automatique effectuée sur le pool :

- [AutoScaleRun.Timestamp](/dotnet/api/microsoft.azure.batch.autoscalerun.timestamp)
- [AutoScaleRun.Results](/dotnet/api/microsoft.azure.batch.autoscalerun.results)
- [AutoScaleRun.Error](/dotnet/api/microsoft.azure.batch.autoscalerun.error)

Dans l’API REST, la requête [Obtenir des informations sur un pool](/rest/api/batchservice/get-information-about-a-pool) retourne des informations sur le pool, notamment des détails sur la dernière exécution de mise à l’échelle automatique dans la propriété [autoScaleRun](/rest/api/batchservice/get-information-about-a-pool).

L’exemple C# suivant utilise la bibliothèque Batch .NET pour imprimer des informations sur la dernière exécution de mise à l’échelle automatique sur le pool _myPool_.

```csharp
await Cloud pool = myBatchClient.PoolOperations.GetPoolAsync("myPool");
Console.WriteLine("Last execution: " + pool.AutoScaleRun.Timestamp);
Console.WriteLine("Result:" + pool.AutoScaleRun.Results.Replace("$", "\n  $"));
Console.WriteLine("Error: " + pool.AutoScaleRun.Error);
```

Échantillon de sortie de l’exemple précédent :

```
Last execution: 10/14/2016 18:36:43
Result:
  $TargetDedicatedNodes=10;
  $NodeDeallocationOption=requeue;
  $curTime=2016-10-14T18:36:43.282Z;
  $isWeekday=1;
  $isWorkingWeekdayHour=0;
  $workHours=0
Error:
```

## <a name="get-autoscale-run-history-using-pool-autoscale-events"></a>Obtenir l’historique des exécutions de mise à l’échelle automatique à l’aide des événements de mise à l’échelle automatique de pool
Vous pouvez également vérifier l’historique de mise à l’échelle automatique en interrogeant [PoolAutoScaleEvent](batch-pool-autoscale-event.md). Cet événement est émis par le service Batch pour enregistrer chaque occurrence d’évaluation et d’exécution de la formule de mise à l’échelle automatique, ce qui peut être utile pour résoudre des problèmes potentiels.

Exemple d’événement pour PoolAutoScaleEvent :
```json
{
    "id": "poolId",
    "timestamp": "2020-09-21T23:41:36.750Z",
    "formula": "...",
    "results": "$TargetDedicatedNodes=10;$NodeDeallocationOption=requeue;$curTime=2016-10-14T18:36:43.282Z;$isWeekday=1;$isWorkingWeekdayHour=0;$workHours=0",
    "error": {
        "code": "",
        "message": "",
        "values": []
    }
}
```

## <a name="example-autoscale-formulas"></a>Exemples de formules de mise à l’échelle automatique

Passons en revue quelques formules illustrant les différentes façons d’ajuster la quantité de ressources de calcul dans un pool.

### <a name="example-1-time-based-adjustment"></a>Exemple 1 : ajustement en fonction du temps

Supposons que vous souhaitiez ajuster la taille du pool selon le jour et l’heure. Cet exemple montre comment augmenter ou diminuer le nombre de nœuds dans le pool en conséquence.

Cette formule obtient dans un premier temps l’heure actuelle. S’il s’agit d’un jour de la semaine (1 à 5) et des heures de travail (8 heures à 18 heures), la taille du pool cible est définie sur 20 nœuds. Sinon, elle est définie sur 10 nœuds.

```
$curTime = time();
$workHours = $curTime.hour >= 8 && $curTime.hour < 18;
$isWeekday = $curTime.weekday >= 1 && $curTime.weekday <= 5;
$isWorkingWeekdayHour = $workHours && $isWeekday;
$TargetDedicatedNodes = $isWorkingWeekdayHour ? 20:10;
$NodeDeallocationOption = taskcompletion;
```

Le paramètre `$curTime` peut être ajusté pour refléter votre fuseau horaire local en ajoutant `time()` au produit de `TimeZoneInterval_Hour` et votre décalage UTC. Par exemple, utilisez `$curTime = time() + (-6 * TimeInterval_Hour);` pour Heure des Rocheuses (heure d'été). Gardez à l’esprit que le décalage doit être ajusté au début et à la fin de l’heure d’été (le cas échéant).

### <a name="example-2-task-based-adjustment"></a>Exemple 2 : ajustement en fonction de la tâche

Dans cet exemple C#, la taille du pool est ajustée en fonction du nombre de tâches présentes dans la file d’attente. Nous avons inclus des commentaires et des sauts de ligne dans les chaînes de formule.

```csharp
// Get pending tasks for the past 15 minutes.
$samples = $PendingTasks.GetSamplePercent(TimeInterval_Minute * 15);
// If we have fewer than 70 percent data points, we use the last sample point,
// otherwise we use the maximum of last sample point and the history average.
$tasks = $samples < 70 ? max(0,$PendingTasks.GetSample(1)) : max( $PendingTasks.GetSample(1), avg($PendingTasks.GetSample(TimeInterval_Minute * 15)));
// If number of pending tasks is not 0, set targetVM to pending tasks, otherwise
// half of current dedicated.
$targetVMs = $tasks > 0? $tasks:max(0, $TargetDedicatedNodes/2);
// The pool size is capped at 20, if target VM value is more than that, set it
// to 20. This value should be adjusted according to your use case.
$TargetDedicatedNodes = max(0, min($targetVMs, 20));
// Set node deallocation mode - let running tasks finish before removing a node
$NodeDeallocationOption = taskcompletion;
```

### <a name="example-3-accounting-for-parallel-tasks"></a>Exemple 3 : comptabilisation des tâches parallèles

Cet exemple C# montre l’ajustement de la taille du pool en fonction du nombre de tâches. Cette formule prend également en compte la valeur [TaskSlotsPerNode](/dotnet/api/microsoft.azure.batch.cloudpool.taskslotspernode) qui a été définie pour le pool. Cette approche est particulièrement utile dans les situations où [l’exécution parallèle de tâches](batch-parallel-node-tasks.md) a été activée sur votre pool.

```csharp
// Determine whether 70 percent of the samples have been recorded in the past
// 15 minutes; if not, use last sample
$samples = $ActiveTasks.GetSamplePercent(TimeInterval_Minute * 15);
$tasks = $samples < 70 ? max(0,$ActiveTasks.GetSample(1)) : max( $ActiveTasks.GetSample(1),avg($ActiveTasks.GetSample(TimeInterval_Minute * 15)));
// Set the number of nodes to add to one-fourth the number of active tasks
// (the TaskSlotsPerNode property on this pool is set to 4, adjust
// this number for your use case)
$cores = $TargetDedicatedNodes * 4;
$extraVMs = (($tasks - $cores) + 3) / 4;
$targetVMs = ($TargetDedicatedNodes + $extraVMs);
// Attempt to grow the number of compute nodes to match the number of active
// tasks, with a maximum of 3
$TargetDedicatedNodes = max(0,min($targetVMs,3));
// Keep the nodes active until the tasks finish
$NodeDeallocationOption = taskcompletion;
```

### <a name="example-4-setting-an-initial-pool-size"></a>Exemple 4 : définition d’une taille de pool initiale

Cet exemple C# avec une formule de mise à l’échelle automatique définit la taille du pool sur un nombre spécifié de nœuds pour une période initiale. Après cela, il ajuste la taille du pool en fonction du nombre de tâches en cours d’exécution et actives.

Plus précisément, cette formule effectue les opérations suivantes :

- Définit la taille initiale du pool sur quatre nœuds.
- N’ajuste pas la taille du pool dans les 10 premières minutes de cycle de vie du pool.
- Après 10 minutes, obtient la valeur maximale du nombre de tâches en cours d’exécution et actives au cours des 60 dernières minutes.
  - Si les deux valeurs sont égales à 0 (ce qui indique qu’aucune tâche n’était en cours d’exécution ni active au cours des 60 dernières minutes), la taille du pool est définie sur 0.
  - Si l’une des valeurs est supérieure à zéro, aucune modification n’est apportée.

```csharp
string now = DateTime.UtcNow.ToString("r");
string formula = string.Format(@"
    $TargetDedicatedNodes = {1};
    lifespan         = time() - time(""{0}"");
    span             = TimeInterval_Minute * 60;
    startup          = TimeInterval_Minute * 10;
    ratio            = 50;

    $TargetDedicatedNodes = (lifespan > startup ? (max($RunningTasks.GetSample(span, ratio), $ActiveTasks.GetSample(span, ratio)) == 0 ? 0 : $TargetDedicatedNodes) : {1});
    ", now, 4);
```

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [exécuter plusieurs tâches simultanément sur les nœuds de calcul de votre pool](batch-parallel-node-tasks.md). Parallèlement à la mise à l’échelle automatique, cela peut aider à réduire la durée du travail pour certaines charges de travail, et donc à économiser de l’argent.
- Découvrez comment [interroger efficacement le service Azure Batch](batch-efficient-list-queries.md) pour plus d’efficacité.
