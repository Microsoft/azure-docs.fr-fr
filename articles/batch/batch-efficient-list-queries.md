---
title: Concevoir des requêtes de liste efficaces
description: Améliorez les performances en filtrant vos requêtes lorsque vous demandez des informations sur des ressources Batch comme les pools, les travaux, les tâches et les nœuds de calcul.
ms.topic: how-to
ms.date: 06/18/2020
ms.custom: seodec18, devx-track-csharp
ms.openlocfilehash: 3a767cc8ae3c8c48e1e40e0735c33fa807ba0015
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88933512"
---
# <a name="create-queries-to-list-batch-resources-efficiently"></a>Créer des requêtes pour répertorier les ressources Batch efficacement

Presque toutes les applications Batch doivent effectuer un certain type de surveillance ou une autre opération qui interroge le service Batch, souvent à intervalles réguliers. Par exemple, pour déterminer s’il existe des tâches restant en file d’attente dans un travail, vous devez obtenir des données sur chaque tâche du travail. Pour déterminer l’état des nœuds dans le pool, vous devez obtenir des données sur chacun d’eux. Cet article explique comment exécuter ces requêtes de la manière la plus efficace.

Vous pouvez améliorer les performances de votre application Azure Batch en réduisant la quantité de données retournées par le service quand vous interrogez des travaux, des tâches, des nœuds de calcul et d’autres ressources avec la bibliothèque [Batch .NET](/dotnet/api/microsoft.azure.batch).

> [!NOTE]
> Le service Batch fournit une prise en charge des API pour des scénarios courants, comme le comptage des tâches d’un travail et le comptage des nœuds dans un pool Batch. Au lieu d’utiliser pour cela une requête de liste, vous pouvez appeler les opérations [Obtenir le nombre de tâches](/rest/api/batchservice/job/gettaskcounts) et [Répertorier le nombre de nœuds des pools](/rest/api/batchservice/account/listpoolnodecounts). Ces opérations sont plus efficaces qu’une requête de liste, mais elles retournent des informations plus limitées qui ne sont pas forcément à jour. Pour plus d’informations, consultez [Compter les tâches et les nœuds de calcul par état](batch-get-resource-counts.md).

## <a name="specify-a-detail-level"></a>Spécifier un niveau de détail

Dans une application Batch de production, les entités telles que les travaux, les tâches et les nœuds de calcul peuvent se compter par milliers. Lorsque vous demandez des informations sur ces ressources, une grande quantité potentielle de données doivent être transférées du service Batch vers votre application à chaque requête. En limitant le nombre d’éléments et le type d’informations retournés par une requête, vous pouvez augmenter la vitesse des requêtes et donc, les performances de votre application.

Cet extrait de code de l’API [Batch.NET](/dotnet/api/microsoft.azure.batch) répertorie *chaque* tâche associée à un travail, ainsi que *toutes* les propriétés de chaque tâche :

```csharp
// Get a collection of all of the tasks and all of their properties for job-001
IPagedEnumerable<CloudTask> allTasks =
    batchClient.JobOperations.ListTasks("job-001");
```

Toutefois, vous pouvez effectuer une requête de liste beaucoup plus efficace en lui appliquant un « niveau de détail ». Pour ce faire, fournissez un objet [ODATADetailLevel](/dotnet/api/microsoft.azure.batch.odatadetaillevel) à la méthode [JobOperations.ListTasks](/dotnet/api/microsoft.azure.batch.joboperations). Cet extrait de code renvoie simplement les propriétés d’identifiant, de ligne de commande et d’informations du nœud de calcul pour les tâches terminées uniquement :

```csharp
// Configure an ODATADetailLevel specifying a subset of tasks and
// their properties to return
ODATADetailLevel detailLevel = new ODATADetailLevel();
detailLevel.FilterClause = "state eq 'completed'";
detailLevel.SelectClause = "id,commandLine,nodeInfo";

// Supply the ODATADetailLevel to the ListTasks method
IPagedEnumerable<CloudTask> completedTasks =
    batchClient.JobOperations.ListTasks("job-001", detailLevel);
```

Si, dans l’exemple de scénario, les tâches associées au travail se comptent par milliers, les résultats de la deuxième requête sont généralement renvoyés bien plus rapidement que ceux de la première requête. Vous trouverez d’autres informations sur l’utilisation de l’objet ODATADetailLevel pour répertorier les éléments avec l’API .NET Batch [ci-dessous](#efficient-querying-in-batch-net).

> [!IMPORTANT]
> Nous vous recommandons vivement de toujours fournir un objet ODATADetailLevel à vos appels de liste d’API .NET afin de garantir une efficacité et des performances maximales pour votre application. En spécifiant un niveau de détail, vous pouvez réduire les délais de réponse du service Batch, améliorer le taux d’utilisation du réseau et réduire l’utilisation de la mémoire par les applications clientes.

## <a name="filter-select-and-expand"></a>Filtrer, sélectionner et développer

Les API [Batch .NET](/dotnet/api/microsoft.azure.batch) et [REST Batch](/rest/api/batchservice/) permettent de réduire le nombre d’éléments retournés dans une liste, ainsi que la quantité d’informations retournées pour chacun d’eux. Pour ce faire, spécifiez des chaînes **filter**, **select** et **expand** lors de l’exécution des requêtes de liste.

### <a name="filter"></a>Filtrer

La chaîne filter est une expression permettant de réduire le nombre d’éléments renvoyés. Par exemple, vous pouvez lister uniquement les tâches en cours d’exécution pour un travail ou lister uniquement les nœuds de calcul prêts à exécuter des tâches.

Une chaîne filter se compose d’une ou de plusieurs expressions dont l’une se compose d’un nom de propriété, d’un opérateur et d’une valeur. Les propriétés spécifiables sont propres à chaque type d’entité que vous interrogez, de même que les opérateurs pris en charge pour chaque propriété.  Plusieurs expressions peuvent être combinées à l’aide des opérateurs logiques `and` et `or`.

Cet exemple de chaîne filter répertorie uniquement les tâches de « rendu » en cours d’exécution : `(state eq 'running') and startswith(id, 'renderTask')`.

### <a name="select"></a>Sélectionnez

La chaîne select limite les valeurs de la propriété retournée pour chaque élément. Vous pouvez spécifier une liste de noms de propriétés séparés par des virgules pour que seules ces valeurs de propriétés soient retournées pour les éléments des résultats de la requête. Vous pouvez spécifier n’importe quelle propriété pour le type d’entité que vous interrogez.

Cet exemple de chaîne select spécifie que seules trois valeurs de propriété doivent être retournées pour chaque tâche : `id, state, stateTransitionTime`.

### <a name="expand"></a>Développez

La chaîne expand réduit le nombre d’appels d’API nécessaires pour obtenir certaines informations. Lorsque vous utilisez une chaîne expand, vous pouvez obtenir davantage d'informations sur chaque élément avec un seul appel d'API. Au lieu d'obtenir la liste des entités, puis de demander des informations pour chaque élément de la liste, vous utilisez une chaîne expand pour obtenir les mêmes informations dans un seul appel d'API, ce qui permet d’améliorer les performances en réduisant le nombre d’appels d’API.

De la même manière que la chaîne select, la chaîne expand détermine si certaines données doivent être incluses dans les résultats de la requête de liste. Quand toutes les propriétés sont requises et qu’aucune chaîne select n’est spécifiée, la chaîne expand *doit* être utilisée pour obtenir les informations statistiques. Si une chaîne select est utilisée pour obtenir un sous-ensemble de propriétés, la propriété `stats` peut alors être spécifiée dans la chaîne select. Il est alors inutile de spécifier la chaîne expand.

La chaîne complète n’est prise en charge que lorsqu’elle est utilisée dans la liste des travaux, la planification de travaux, des tâches et des pools. Actuellement, elle ne prend en charge que les informations statistiques.

Cet exemple de chaîne expand spécifie que les informations statistiques doivent être retournées pour chaque élément de la liste : `stats`.

> [!NOTE]
> Lors de la construction de l’un des trois types de chaînes de requête (filter, select et expand), vous devez vous assurer que les noms de propriété et la casse correspondent à ceux de leurs homologues de l’API REST. Par exemple, lorsque vous utilisez la classe [CloudTask](/dotnet/api/microsoft.azure.batch.cloudtask) .NET, vous devez spécifier **état** et non **État** même si la propriété .NET est [CloudTask.State](/dotnet/api/microsoft.azure.batch.cloudtask.state#Microsoft_Azure_Batch_CloudTask_State). Consultez les tableaux ci-dessous pour connaître les mappages de propriété entre les API .NET et REST.

### <a name="rules-for-filter-select-and-expand-strings"></a>Règles des chaînes filter, select et expand

- Le nom des propriétés des chaînes filter, select et expand doit être affiché comme dans l’API [REST Batch](/rest/api/batchservice/) et ce, même si vous utilisez la bibliothèque [Batch .NET](/dotnet/api/microsoft.azure.batch) ou l’un des autres SDK Batch.
- Les noms de propriété respectent la casse contrairement aux valeurs de propriété.
- Les chaînes de date/heure peuvent être d’un format ou de l’autre et doivent être précédées de `DateTime`.
  
  - Exemple de format W3C-DTF : `creationTime gt DateTime'2011-05-08T08:49:37Z'`
  - Exemple de format RFC 1123 : `creationTime gt DateTime'Sun, 08 May 2011 08:49:37 GMT'`
- Les chaînes booléennes ont la valeur `true` ou `false`.
- Si une propriété ou un opérateur non valide est spécifié, une erreur `400 (Bad Request)` se produit.

## <a name="efficient-querying-in-batch-net"></a>Interrogation efficace dans Batch.NET

Dans l’API [Batch .NET](/dotnet/api/microsoft.azure.batch), la classe [ODATADetailLevel](/dotnet/api/microsoft.azure.batch.odatadetaillevel) sert à fournir les chaînes filter, select et expand aux opérations de liste. La classe ODataDetailLevel possède trois propriétés de chaîne publiques qui peuvent être spécifiées dans le constructeur ou définies directement sur l'objet. Vous transmettez ensuite l’objet ODataDetailLevel en tant que paramètre à plusieurs opérations de liste comme [ListPools](/dotnet/api/microsoft.azure.batch.pooloperations), [ListJobs](/dotnet/api/microsoft.azure.batch.joboperations) et [ListTasks](/dotnet/api/microsoft.azure.batch.joboperations).

- [ODATADetailLevel.FilterClause](/dotnet/api/microsoft.azure.batch.odatadetaillevel.filterclause) : limitez le nombre d’éléments qui sont retournés.
- [ODATADetailLevel.SelectClause](/dotnet/api/microsoft.azure.batch.odatadetaillevel.selectclause) : spécifiez les valeurs de propriété retournées avec chaque élément.
- [ODATADetailLevel.ExpandClause](/dotnet/api/microsoft.azure.batch.odatadetaillevel.expandclause) : récupèrez les données de tous les éléments en utilisant un seul appel d’API et non des appels distincts pour chaque élément.

L’extrait de code suivant utilise l’API .NET Batch pour interroger efficacement le service Batch pour les statistiques d’un ensemble spécifique de pools. Dans ce scénario, l’utilisateur de Batch comporte à la fois des pools de test et des pools de production. Les ID de pool de test sont précédés du préfixe « test » et les ID de production sont précédés de « prod ». Dans l’extrait de code, *myBatchClient* est une instance initialisée correctement de la classe [BatchClient](/dotnet/api/microsoft.azure.batch.batchclient) .

```csharp
// First we need an ODATADetailLevel instance on which to set the filter, select,
// and expand clause strings
ODATADetailLevel detailLevel = new ODATADetailLevel();

// We want to pull only the "test" pools, so we limit the number of items returned
// by using a FilterClause and specifying that the pool IDs must start with "test"
detailLevel.FilterClause = "startswith(id, 'test')";

// To further limit the data that crosses the wire, configure the SelectClause to
// limit the properties that are returned on each CloudPool object to only
// CloudPool.Id and CloudPool.Statistics
detailLevel.SelectClause = "id, stats";

// Specify the ExpandClause so that the .NET API pulls the statistics for the
// CloudPools in a single underlying REST API call. Note that we use the pool's
// REST API element name "stats" here as opposed to "Statistics" as it appears in
// the .NET API (CloudPool.Statistics)
detailLevel.ExpandClause = "stats";

// Now get our collection of pools, minimizing the amount of data that is returned
// by specifying the detail level that we configured above
List<CloudPool> testPools =
    await myBatchClient.PoolOperations.ListPools(detailLevel).ToListAsync();
```

> [!TIP]
> Une instance d’[ODATADetailLevel](/dotnet/api/microsoft.azure.batch.odatadetaillevel) configurée avec les clauses Select et Expand peut également être transmise aux méthodes Get appropriées telles que [PoolOperations.GetPool](/dotnet/api/microsoft.azure.batch.pooloperations.getpool#Microsoft_Azure_Batch_PoolOperations_GetPool_System_String_Microsoft_Azure_Batch_DetailLevel_System_Collections_Generic_IEnumerable_Microsoft_Azure_Batch_BatchClientBehavior__) pour limiter la quantité de données retournées.

## <a name="batch-rest-to-net-api-mappings"></a>Mappages entre les API Batch REST et .NET

Les noms de propriété dans les chaînes filter, select et expand doivent refléter leurs homologues dans l’API REST, aussi bien au niveau du nom que de la casse. Les tableaux ci-dessous fournissent les correspondances existant entre les homologues .NET et REST.

### <a name="mappings-for-filter-strings"></a>Mappages pour les chaînes filter

- **Méthodes de liste .NET** : chacune des méthodes de l’API .NET de cette colonne accepte un objet [ODATADetailLevel](/dotnet/api/microsoft.azure.batch.odatadetaillevel) en tant que paramètre.
- **Requêtes de liste REST** : chaque page de l’API REST de cette colonne contient une table spécifiant les propriétés et les opérations autorisées dans les chaînes *filter* . Vous devez utiliser ces noms de propriétés et ces opérations lors de la construction d’une chaîne [ODATADetailLevel.FilterClause](/dotnet/api/microsoft.azure.batch.odatadetaillevel.filterclause).

| Méthodes de liste .NET | Requêtes de liste REST |
| --- | --- |
| [CertificateOperations.ListCertificates](/dotnet/api/microsoft.azure.batch.certificateoperations) |[Création de la liste des certificats dans un compte](/rest/api/batchservice/certificate/list) |
| [CloudTask.ListNodeFiles](/dotnet/api/microsoft.azure.batch.cloudtask) |[Création de la liste des fichiers associés à une tâche](/rest/api/batchservice/file/listfromtask) |
| [JobOperations.ListJobPreparationAndReleaseTaskStatus](/dotnet/api/microsoft.azure.batch.joboperations) |[Création de la liste des états de préparation du travail et des tâches de lancement d’un travail](/rest/api/batchservice/job/listpreparationandreleasetaskstatus) |
| [JobOperations.ListJobs](/dotnet/api/microsoft.azure.batch.joboperations) |[Création de la liste des travaux dans un compte](/rest/api/batchservice/job/list) |
| [JobOperations.ListNodeFiles](/dotnet/api/microsoft.azure.batch.joboperations) |[Création de la liste des fichiers sur un nœud](/rest/api/batchservice/file/listfromcomputenode) |
| [JobOperations.ListTasks](/dotnet/api/microsoft.azure.batch.joboperations) |[Création de la liste des tâches associées à un travail](/rest/api/batchservice/task/list) |
| [JobScheduleOperations.ListJobSchedules](/dotnet/api/microsoft.azure.batch.jobscheduleoperations) |[Création de la liste des planifications de travaux dans un compte](/rest/api/batchservice/jobschedule/list) |
| [JobScheduleOperations.ListJobs](/dotnet/api/microsoft.azure.batch.jobscheduleoperations) |[Création de la liste des travaux associés à une planification de travail](/rest/api/batchservice/job/listfromjobschedule) |
| [PoolOperations.ListComputeNodes](/dotnet/api/microsoft.azure.batch.pooloperations) |[Création de la liste des nœuds de calcul dans un pool](/rest/api/batchservice/computenode/list) |
| [PoolOperations.ListPools](/dotnet/api/microsoft.azure.batch.pooloperations) |[Création de la liste des pools d’un compte](/rest/api/batchservice/pool/list) |

### <a name="mappings-for-select-strings"></a>Mappages pour les chaînes select

- **Types .NET Batch** : types d’API .NET Batch.
- **Entités de l’API REST** : chaque page de cette colonne contient une ou plusieurs tables qui listent les noms des propriétés de l’API REST correspondant au type. Ces noms de propriétés sont utilisés lorsque vous construisez des chaînes *select* . Vous devez utiliser ces mêmes noms de propriétés lors de la construction d’une chaîne [ODATADetailLevel.SelectClause](/dotnet/api/microsoft.azure.batch.odatadetaillevel.selectclause).

| Types .NET Batch | Entités de l’API REST |
| --- | --- |
| [Certificate](/dotnet/api/microsoft.azure.batch.certificate) |[Obtenir des informations sur un certificat](/rest/api/batchservice/certificate/get) |
| [CloudJob](/dotnet/api/microsoft.azure.batch.cloudjob) |[Obtenir des informations sur un travail](/rest/api/batchservice/job/get) |
| [CloudJobSchedule](/dotnet/api/microsoft.azure.batch.cloudjobschedule) |[Obtenir des informations sur la planification d’un travail](/rest/api/batchservice/jobschedule/get) |
| [ComputeNode](/dotnet/api/microsoft.azure.batch.computenode) |[Obtenir des informations sur un nœud](/rest/api/batchservice/computenode/get) |
| [CloudPool](/dotnet/api/microsoft.azure.batch.cloudpool) |[Obtenir des informations sur un pool](/rest/api/batchservice/pool/get) |
| [CloudTask](/dotnet/api/microsoft.azure.batch.cloudtask) |[Obtenir des informations sur une tâche](/rest/api/batchservice/task/get) |

## <a name="example-construct-a-filter-string"></a>Exemple : Construire une chaîne filter

Lors de la construction d’une chaîne filter pour [ODATADetailLevel.FilterClause](/dotnet/api/microsoft.azure.batch.odatadetaillevel.filterclause), consultez le tableau ci-dessous sous « Mappages pour les chaînes filter » pour rechercher la page de documentation de l’API REST correspondant à l’opération de liste que vous souhaitez effectuer. Vous trouverez les propriétés filtrables et leurs opérateurs pris en charge dans le premier tableau à plusieurs lignes de cette page. Si vous souhaitez récupérer toutes les tâches ayant un code de sortie non nul, par exemple, cette ligne de [Création de la liste des tâches associées à un travail](/rest/api/batchservice/task/list) spécifie la chaîne de propriétés applicable et les opérateurs autorisés :

| Propriété | Opérations autorisées | Type |
|:--- |:--- |:--- |
| `executionInfo/exitCode` |`eq, ge, gt, le , lt` |`Int` |

Par conséquent, la chaîne filter permettant de répertorier toutes les tâches dont le code de sortie serait autre que nul serait :

`(executionInfo/exitCode lt 0) or (executionInfo/exitCode gt 0)`

## <a name="example-construct-a-select-string"></a>Exemple : Construire une chaîne select

Pour construire une clause [ODATADetailLevel.SelectClause](/dotnet/api/microsoft.azure.batch.odatadetaillevel.selectclause), consultez le tableau qui précède sous « Mappages pour les chaînes select » et accédez à la page de l’API REST correspondant au type d’entité que vous listez. Vous trouverez les propriétés sélectionnables et leurs opérateurs pris en charge dans le premier tableau à plusieurs lignes de cette page. Si, par exemple, vous souhaitez récupérer uniquement l’ID et la ligne de commande de chaque tâche dans une liste, vous trouverez ces lignes dans le tableau correspondant sur [Obtenir des informations sur une tâche](/rest/api/batchservice/task/get) :

| Propriété | Type | Notes |
|:--- |:--- |:--- |
| `id` |`String` |`The ID of the task.` |
| `commandLine` |`String` |`The command line of the task.` |

La chaîne select permettant d’inclure uniquement l’ID et la ligne de commande avec chaque tâche de la liste serait alors :

`id, commandLine`

## <a name="code-samples"></a>Exemples de code

### <a name="efficient-list-queries-code-sample"></a>Exemple de code pour des requêtes de liste efficaces

L’exemple de projet [EfficientListQueries](https://github.com/Azure-Samples/azure-batch-samples/tree/master/CSharp/ArticleProjects/EfficientListQueries) sur GitHub montre comment une interrogation efficace de liste peut affecter les performances d’une application. Cette application de console C# crée et ajoute de nombreuses tâches à un travail. Ensuite, elle effectue plusieurs appels de la méthode [JobOperations.ListTasks](/dotnet/api/microsoft.azure.batch.joboperations) et transmet des objets [ODATADetailLevel](/dotnet/api/microsoft.azure.batch.odatadetaillevel) configurés avec différentes valeurs de propriété pour faire varier la quantité de données à retourner. Vous générez ainsi des informations qui ressemblent à ce qui suit :

```
Adding 5000 tasks to job jobEffQuery...
5000 tasks added in 00:00:47.3467587, hit ENTER to query tasks...

4943 tasks retrieved in 00:00:04.3408081 (ExpandClause:  | FilterClause: state eq 'active' | SelectClause: id,state)
0 tasks retrieved in 00:00:00.2662920 (ExpandClause:  | FilterClause: state eq 'running' | SelectClause: id,state)
59 tasks retrieved in 00:00:00.3337760 (ExpandClause:  | FilterClause: state eq 'completed' | SelectClause: id,state)
5000 tasks retrieved in 00:00:04.1429881 (ExpandClause:  | FilterClause:  | SelectClause: id,state)
5000 tasks retrieved in 00:00:15.1016127 (ExpandClause:  | FilterClause:  | SelectClause: id,state,environmentSettings)
5000 tasks retrieved in 00:00:17.0548145 (ExpandClause: stats | FilterClause:  | SelectClause: )

Sample complete, hit ENTER to continue...
```

Comme indiqué dans les temps écoulés, vous pouvez réduire de façon significative les temps de réponse de requête en limitant les propriétés et le nombre d’éléments retournés. Cet exemple et d’autres exemples de projet sont disponibles dans le dépôt [azure-batch-samples](https://github.com/Azure-Samples/azure-batch-samples) sur GitHub.

### <a name="batchmetrics-library-and-code-sample"></a>Bibliothèque BatchMetrics et exemple de code

En plus de l’exemple de code EfficientListQueries ci-dessus, l’exemple de projet [BatchMetrics](https://github.com/Azure-Samples/azure-batch-samples/tree/master/CSharp/BatchMetrics) montre comment superviser efficacement la progression du travail Azure Batch à l’aide de l’API Batch.

L’exemple [BatchMetrics](https://github.com/Azure-Samples/azure-batch-samples/tree/master/CSharp/BatchMetrics) inclut un projet de bibliothèque de classes .NET que vous pouvez incorporer dans vos propres projets ainsi qu’un programme de ligne de commande simple pour tester et démontrer l’utilisation de la bibliothèque.

L’exemple d’application au sein du projet illustre les opérations suivantes :

1. Sélection d’attributs spécifiques afin de ne télécharger que les propriétés nécessaires
2. Filtrage basé sur les heures de changement d’état afin de ne télécharger que les modifications apportées depuis la dernière requête

Par exemple, la méthode suivante apparaît dans la bibliothèque BatchMetrics. Elle retourne un objet ODATADetailLevel qui spécifie que seules les propriétés `id` et `state` doivent être obtenues pour les entités interrogées. Elle indique également que seules les entités dont l’état a changé depuis le paramètre `DateTime` spécifié doivent être retournées.

```csharp
internal static ODATADetailLevel OnlyChangedAfter(DateTime time)
{
    return new ODATADetailLevel(
        selectClause: "id, state",
        filterClause: string.Format("stateTransitionTime gt DateTime'{0:o}'", time)
    );
}
```

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [optimiser l’utilisation des ressources de calcul Azure Batch avec les tâches de nœud simultanées](batch-parallel-node-tasks.md). Certains types de charge de travail peuvent tirer parti de l’exécution de tâches parallèles sur des nœuds de calcul plus volumineux (mais moins nombreux). Découvrez [l’exemple de scénario](batch-parallel-node-tasks.md#example-scenario) de l’article pour en connaître les détails.
- Découvrez comment [superviser les solutions Batch en comptant les tâches et les nœuds par état](batch-get-resource-counts.md).


[api_net]: /dotnet/api/microsoft.azure.batch
[api_net_listjobs]: /dotnet/api/microsoft.azure.batch.joboperations
[api_rest]: /rest/api/batchservice/
[batch_metrics]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/BatchMetrics
[efficient_query_sample]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/EfficientListQueries
[github_samples]: https://github.com/Azure/azure-batch-samples
[odata]: /dotnet/api/microsoft.azure.batch.odatadetaillevel
[odata_ctor]: /dotnet/api/microsoft.azure.batch.odatadetaillevel
[odata_expand]: /dotnet/api/microsoft.azure.batch.odatadetaillevel
[odata_filter]: /dotnet/api/microsoft.azure.batch.odatadetaillevel
[odata_select]: /dotnet/api/microsoft.azure.batch.odatadetaillevel

[net_list_certs]: /dotnet/api/microsoft.azure.batch.certificateoperations
[net_list_compute_nodes]: /dotnet/api/microsoft.azure.batch.pooloperations
[net_list_job_schedules]: /dotnet/api/microsoft.azure.batch.jobscheduleoperations
[net_list_jobprep_status]: /dotnet/api/microsoft.azure.batch.joboperations
[net_list_jobs]: /dotnet/api/microsoft.azure.batch.joboperations
[net_list_nodefiles]: /dotnet/api/microsoft.azure.batch.joboperations
[net_list_pools]: /dotnet/api/microsoft.azure.batch.pooloperations
[net_list_schedule_jobs]: /dotnet/api/microsoft.azure.batch.jobscheduleoperations
[net_list_task_files]: /dotnet/api/microsoft.azure.batch.cloudtask
[net_list_tasks]: /dotnet/api/microsoft.azure.batch.joboperations

[rest_list_certs]: /rest/api/batchservice/certificate/list
[rest_list_compute_nodes]: /rest/api/batchservice/computenode/list
[rest_list_job_schedules]: /rest/api/batchservice/jobschedule/list
[rest_list_jobprep_status]: /rest/api/batchservice/job/listpreparationandreleasetaskstatus
[rest_list_jobs]: /rest/api/batchservice/job/list
[rest_list_nodefiles]: /rest/api/batchservice/file/listfromcomputenode
[rest_list_pools]: /rest/api/batchservice/pool/list
[rest_list_schedule_jobs]: /rest/api/batchservice/job/listfromjobschedule
[rest_list_task_files]: /rest/api/batchservice/file/listfromtask
[rest_list_tasks]: /rest/api/batchservice/task/list

[rest_get_cert]: /rest/api/batchservice/certificate/get
[rest_get_job]: /rest/api/batchservice/job/get
[rest_get_node]: /rest/api/batchservice/computenode/get
[rest_get_pool]: /rest/api/batchservice/pool/get
[rest_get_schedule]: /rest/api/batchservice/jobschedule/get
[rest_get_task]: /rest/api/batchservice/task/get

[net_cert]: /dotnet/api/microsoft.azure.batch.certificate
[net_job]: /dotnet/api/microsoft.azure.batch.cloudjob
[net_node]: /dotnet/api/microsoft.azure.batch.computenode
[net_pool]: /dotnet/api/microsoft.azure.batch.cloudpool
[net_schedule]: /dotnet/api/microsoft.azure.batch.cloudjobschedule
[net_task]: /dotnet/api/microsoft.azure.batch.cloudtask

[rest_get_task_counts]: /rest/api/batchservice/get-the-task-counts-for-a-job
[rest_get_node_counts]: /rest/api/batchservice/account/listpoolnodecounts
