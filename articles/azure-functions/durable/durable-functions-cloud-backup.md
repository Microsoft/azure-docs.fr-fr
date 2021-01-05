---
title: Scénarios fan-out/fan-in dans Fonctions durables - Azure
description: Découvrez comment implémenter un scénario fan-out/fan-in dans l’extension Fonctions durables pour Azure Functions.
ms.topic: conceptual
ms.date: 11/02/2019
ms.author: azfuncdf
ms.openlocfilehash: d61600801286126ea6ffb9a97bc5655b6f233816
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77562188"
---
# <a name="fan-outfan-in-scenario-in-durable-functions---cloud-backup-example"></a>Scénario fan-out/fan-in dans Fonctions durables - exemple de sauvegarde cloud

*Fan-out/fan-in* fait référence à un modèle incluant une exécution simultanée de plusieurs fonctions puis une agrégation à partir des résultats. Cet article décrit un exemple utilisant [Fonctions durables](durable-functions-overview.md) pour implémenter un scénario fan-in/fan-out. L’exemple représente une fonction durable qui sauvegarde tout ou partie du contenu du site d’une application dans Stockage Azure.

[!INCLUDE [durable-functions-prerequisites](../../../includes/durable-functions-prerequisites.md)]

## <a name="scenario-overview"></a>Présentation du scénario

Dans cet exemple, les fonctions chargent tous les fichiers dans un répertoire spécifié de manière récursive dans le stockage d’objets blob. Elles comptent également le nombre total d’octets qui ont été chargés.

Il est possible d’écrire une fonction unique qui prend tout en charge. Le principal problème que vous risquez de rencontrer est **l’évolutivité**. Comme une fonction unique ne peut s’exécuter que sur une seule machine virtuelle, son débit est limité par celui de cette machine virtuelle. Un autre problème est la **fiabilité**. Si une défaillance survient en cours de route, ou que l’ensemble du processus prend plus de cinq minutes, la sauvegarde risque d’échouer dans un état partiellement terminé. Le processus devra alors être redémarré.

Une approche plus robuste consiste à écrire deux fonctions régulières : une pour énumérer les fichiers et ajouter les noms de fichiers à une file d’attente, et une autre pour lire à partir de la file d’attente et charger les fichiers vers le stockage d’objets blob. Cette approche est préférable du point de vue du débit et de la fiabilité, mais elle oblige à configurer et à gérer une file d’attente. Plus important encore, elle ajoute une complexité significative en termes de **gestion d’état** et de **coordination** si vous souhaitez effectuer une tâche supplémentaire, comme calculer le nombre total d’octets chargés.

Une approche Fonctions durables vous offre tous les avantages mentionnés, à peu de frais.

## <a name="the-functions"></a>Les fonctions

Cet article explique les fonctions suivantes dans l’exemple d’application :

* `E2_BackupSiteContent`: [orchestrateur de fonction](durable-functions-bindings.md#orchestration-trigger) qui appelle `E2_GetFileList` pour obtenir une liste de fichiers à sauvegarder, puis appelle `E2_CopyFileToBlob` pour sauvegarder chaque fichier.
* `E2_GetFileList`: [fonction d’activité](durable-functions-bindings.md#activity-trigger) qui retourne une liste de fichiers dans un répertoire.
* `E2_CopyFileToBlob`: fonction d’activité qui sauvegarde un fichier dans le stockage Blob Azure.

### <a name="e2_backupsitecontent-orchestrator-function"></a>Fonction d’orchestrateur E2_BackupSiteContent

Cette fonction d’orchestrateur effectue essentiellement les opérations suivantes :

1. Prend une valeur `rootDirectory` comme paramètre d’entrée.
2. Appelle une fonction pour obtenir une liste récursive des fichiers sous `rootDirectory`.
3. Effectue plusieurs appels parallèles de fonction pour charger chaque fichier dans Stockage Blob Azure.
4. Attend la fin de tous les chargements.
5. Retourne le nombre total d’octets qui ont été chargés dans Stockage Blob Azure.

# <a name="c"></a>[C#](#tab/csharp)

Voici le code qui implémente la fonction d’orchestrateur :

[!code-csharp[Main](~/samples-durable-functions/samples/precompiled/BackupSiteContent.cs?range=16-42)]

Notez la ligne `await Task.WhenAll(tasks);`. *Aucun* des différents appels à la fonction `E2_CopyFileToBlob` n’a été attendu, ce qui leur permet de s’exécuter en parallèle. Lorsque nous transmettons ce tableau de tâches à `Task.WhenAll`, nous obtenons une tâche qui ne se termine pas *tant que toutes les opérations de copie ne sont pas finies*. Si vous connaissez la bibliothèque parallèle de tâches (Task Parallel Library, TPL) dans .NET, cela n’est pas nouveau pour vous. La différence est que ces tâches peuvent s’exécuter simultanément sur plusieurs machines virtuelles, et que l’extension Durable Functions garantit que l’exécution de bout en bout n’est pas interrompue par un recyclage de processus.

Après avoir attendu `Task.WhenAll`, nous savons que tous les appels à la fonction sont terminés et nous ont retourné des valeurs. Chaque appel à `E2_CopyFileToBlob` renvoie le nombre d’octets chargés. Pour calculer le nombre total d’octets, il suffit donc d’additionner toutes ces valeurs retournées.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

La fonction utilise le fichier *function.json* standard pour les fonctions d’orchestrateur.

[!code-json[Main](~/samples-durable-functions/samples/javascript/E2_BackupSiteContent/function.json)]

Voici le code qui implémente la fonction d’orchestrateur :

[!code-javascript[Main](~/samples-durable-functions/samples/javascript/E2_BackupSiteContent/index.js)]

Notez la ligne `yield context.df.Task.all(tasks);`. *Aucun* des différents appels à la fonction `E2_CopyFileToBlob` n’a été interrompu, ce qui leur permet de s’exécuter en parallèle. Lorsque nous transmettons ce tableau de tâches à `context.df.Task.all`, nous obtenons une tâche qui ne se termine pas *tant que toutes les opérations de copie ne sont pas finies*. Si vous connaissez déjà [`Promise.all`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all) en JavaScript, ce n’est pas une nouveauté pour vous. La différence est que ces tâches peuvent s’exécuter simultanément sur plusieurs machines virtuelles, et que l’extension Durable Functions garantit que l’exécution de bout en bout n’est pas interrompue par un recyclage de processus.

> [!NOTE]
> Bien que les tâches soient conceptuellement similaires aux promesses JavaScript, les fonctions d’orchestrateur doivent utiliser `context.df.Task.all` et `context.df.Task.any` plutôt que `Promise.all` et `Promise.race` pour gérer la parallélisation de la tâche.

Maintenant que nous avons interrompu l’exécution à partir de `context.df.Task.all`, nous savons que tous les appels à la fonction sont terminés et nous ont retourné des valeurs. Chaque appel à `E2_CopyFileToBlob` renvoie le nombre d’octets chargés. Pour calculer le nombre total d’octets, il suffit donc d’additionner toutes ces valeurs retournées.

---

### <a name="helper-activity-functions"></a>Fonctions d’activité d’assistance

Les fonctions d’activité d’assistance, tout comme avec d’autres exemples, sont des fonctions standard qui utilisent la liaison de déclencheur `activityTrigger`.

#### <a name="e2_getfilelist-activity-function"></a>Fonction d’activité E2_GetFileList

# <a name="c"></a>[C#](#tab/csharp)

[!code-csharp[Main](~/samples-durable-functions/samples/precompiled/BackupSiteContent.cs?range=44-54)]

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Le fichier *function.json* pour `E2_GetFileList` ressemble à ce qui suit :

[!code-json[Main](~/samples-durable-functions/samples/javascript/E2_GetFileList/function.json)]

Et voici l’implémentation :

[!code-javascript[Main](~/samples-durable-functions/samples/javascript/E2_GetFileList/index.js)]

La fonction utilise le module `readdirp` (version 2.x) pour lire de manière récursive la structure de répertoires.

---

> [!NOTE]
> Vous vous demandez peut-être pourquoi ne pas simplement placer ce code directement dans la fonction d’orchestrateur ? C’est possible, mais cela compromettrait une des règles fondamentales des fonctions d’orchestrateur, à savoir qu’elles ne doivent effectuer d’opérations E/S, y compris avec accès au système de fichiers local. Pour plus d’informations, consultez [Contraintes du code des fonctions d’orchestrateur](durable-functions-code-constraints.md).

#### <a name="e2_copyfiletoblob-activity-function"></a>Fonction d’activité E2_CopyFileToBlob

# <a name="c"></a>[C#](#tab/csharp)

[!code-csharp[Main](~/samples-durable-functions/samples/precompiled/BackupSiteContent.cs?range=56-81)]

> [!NOTE]
> Vous devez installer le package NuGet `Microsoft.Azure.WebJobs.Extensions.Storage` pour exécuter l’exemple de code.

La fonction utilise certaines fonctionnalités avancées des liaisons d’Azure Functions (autrement dit, le [paramètre `Binder`](../functions-dotnet-class-library.md#binding-at-runtime)). Toutefois, vous n’avez pas à vous soucier de ces détails dans cette procédure pas à pas.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Le fichier *function.json* pour `E2_CopyFileToBlob` est tout aussi simple :

[!code-json[Main](~/samples-durable-functions/samples/javascript/E2_CopyFileToBlob/function.json)]

L’implémentation JavaScript utilise le [SDK du stockage Azure pour Node](https://github.com/Azure/azure-storage-node) afin de charger les fichiers dans le stockage Blob Azure.

[!code-javascript[Main](~/samples-durable-functions/samples/javascript/E2_CopyFileToBlob/index.js)]

---

L’implémentation charge le fichier à partir du disque et transmet de manière asynchrone le contenu vers un objet blob du même nom dans le conteneur « backups ». La valeur de retour correspond au nombre d’octets copiés vers le stockage, utilisée ensuite par la fonction d’orchestrateur pour calculer la somme d’agrégation.

> [!NOTE]
> Il s’agit d’un exemple parfait de déplacement d’opérations d’E/S vers une fonction `activityTrigger`. Non seulement le travail peut être réparti sur plusieurs machines différentes, mais vous avez également la possibilité de créer des points de contrôle tout au long de la progression. Si le processus hôte est interrompu pour une raison quelconque, vous savez que les chargements ont déjà été effectués.

## <a name="run-the-sample"></a>Exécution de l'exemple

Vous pouvez démarrer l’orchestration en envoyant la requête HTTP POST suivante.

```
POST http://{host}/orchestrators/E2_BackupSiteContent
Content-Type: application/json
Content-Length: 20

"D:\\home\\LogFiles"
```

> [!NOTE]
> La fonction `HttpStart` que vous appelez fonctionne uniquement avec un contenu au format JSON. Pour cette raison, l’en-tête `Content-Type: application/json` est requis, et le chemin d’accès au répertoire est encodé sous forme de chaîne JSON. De plus, l’extrait de code HTTP suppose qu’il existe une entrée dans le fichier `host.json`, qui supprime le préfixe `api/` par défaut de toutes les URL de fonctions de déclencheur HTTP. Le balisage pour cette configuration figure dans le fichier `host.json` dans les exemples.

Cette requête HTTP déclenche l’orchestrateur `E2_BackupSiteContent` et transmet la chaîne `D:\home\LogFiles` en tant que paramètre. La réponse fournit un lien pour obtenir l’état de l’opération de sauvegarde :

```
HTTP/1.1 202 Accepted
Content-Length: 719
Content-Type: application/json; charset=utf-8
Location: http://{host}/runtime/webhooks/durabletask/instances/b4e9bdcc435d460f8dc008115ff0a8a9?taskHub=DurableFunctionsHub&connection=Storage&code={systemKey}

(...trimmed...)
```

Selon le nombre de fichiers journaux que contient votre application de fonction, cette opération peut prendre plusieurs minutes. Vous pouvez obtenir l’état le plus récent en interrogeant l’URL dans l’en-tête `Location` de la réponse HTTP 202 précédente.

```
GET http://{host}/runtime/webhooks/durabletask/instances/b4e9bdcc435d460f8dc008115ff0a8a9?taskHub=DurableFunctionsHub&connection=Storage&code={systemKey}
```

```
HTTP/1.1 202 Accepted
Content-Length: 148
Content-Type: application/json; charset=utf-8
Location: http://{host}/runtime/webhooks/durabletask/instances/b4e9bdcc435d460f8dc008115ff0a8a9?taskHub=DurableFunctionsHub&connection=Storage&code={systemKey}

{"runtimeStatus":"Running","input":"D:\\home\\LogFiles","output":null,"createdTime":"2019-06-29T18:50:55Z","lastUpdatedTime":"2019-06-29T18:51:16Z"}
```

Dans ce cas, la fonction est toujours en cours d’exécution. Vous pouvez voir l’entrée enregistrée dans l’état de l’orchestrateur et l’heure de la dernière mise à jour. Vous pouvez continuer à utiliser les valeurs d’en-tête `Location` jusqu’à la fin de l’interrogation. Lorsque l’état indique « Completed » (Terminé), une valeur de réponse HTTP semblable à ce qui suit apparaît :

```
HTTP/1.1 200 OK
Content-Length: 152
Content-Type: application/json; charset=utf-8

{"runtimeStatus":"Completed","input":"D:\\home\\LogFiles","output":452071,"createdTime":"2019-06-29T18:50:55Z","lastUpdatedTime":"2019-06-29T18:51:26Z"}
```

Vous pouvez maintenant voir que l’orchestration est terminée et la durée approximative de cette opération. Vous voyez également une valeur pour le champ `output`, indiquant qu’environ 450 ko de journaux d’activité ont été chargés.

## <a name="next-steps"></a>Étapes suivantes

Cet exemple a montré comment implémenter le modèle fan-out/fan-in. L’exemple suivant montre comment implémenter le modèle de surveillance à l’aide de [minuteurs durables](durable-functions-timers.md).

> [!div class="nextstepaction"]
> [Exécuter l’exemple de surveillance](durable-functions-monitor.md)