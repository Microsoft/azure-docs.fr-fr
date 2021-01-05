---
title: Rechercher les erreurs de travail et de tâche
description: Découvrez les erreurs à rechercher et comment résoudre les problèmes liés aux travaux et aux tâches.
author: mscurrell
ms.topic: how-to
ms.date: 11/23/2020
ms.author: markscu
ms.openlocfilehash: d8cf3b5e28d4455e00e0bdcbae2063771d3e8acd
ms.sourcegitcommit: 1bf144dc5d7c496c4abeb95fc2f473cfa0bbed43
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95736797"
---
# <a name="job-and-task-error-checking"></a>Vérification des erreurs de travail et de tâche

Plusieurs erreurs peuvent se produire lors de l’ajout de travaux et de tâches. La détection des échecs de ces opérations est simple, car les échecs sont retournés immédiatement par l’API, l’interface CLI ou l’interface utilisateur. Toutefois, des échecs peuvent également se produire ultérieurement lorsque des travaux et des tâches sont planifiés et exécutés.

Cet article traite des erreurs qui peuvent se produire après l’envoi de travaux et de tâches, ainsi que de la façon de les rechercher et de les gérer.

## <a name="jobs"></a>travaux

Un travail est un regroupement d’une ou plusieurs tâches, ces dernières spécifiant réellement les lignes de commande à exécuter.

Lorsque vous ajoutez un travail, les paramètres suivants peuvent être spécifiés, influençant éventuellement la façon dont le travail peut échouer :

- [Contraintes de travail](/rest/api/batchservice/job/add#jobconstraints)
  - La propriété `maxWallClockTime` peut être spécifiée de manière facultative pour définir la durée maximale pendant laquelle un travail peut être actif ou en cours d’exécution. En cas de dépassement, le travail sera terminé avec la propriété `terminateReason` définie dans le champ [executionInfo](/rest/api/batchservice/job/get#cloudjob) du travail.
- [Tâche de préparation du travail](/rest/api/batchservice/job/add#jobpreparationtask)
  - Si elle est spécifiée, une tâche de préparation du travail est exécutée lors de la première exécution d’une tâche pour un travail sur un nœud. La tâche de préparation du travail peut échouer, ce qui se traduira par le fait que la tâche ne sera pas exécutée et que le travail ne sera pas finalisé.
- [Tâche de validation du travail](/rest/api/batchservice/job/add#jobreleasetask)
  - Une tâche de validation du travail ne peut être spécifiée que si une tâche de préparation du travail est configurée. Lorsqu’un travail est terminé, la tâche de validation du travail est exécutée sur chacun des nœuds du pool où une tâche de préparation du travail a été exécutée. Une tâche de validation du travail peut échouer, mais le travail passera tout de même à l’état `completed`.

### <a name="job-properties"></a>Propriétés du travail

Les propriétés de travail suivantes doivent être vérifiées afin de détecter les erreurs :

- « [executionInfo](/rest/api/batchservice/job/get#jobexecutioninformation) » :
  - La propriété `terminateReason` peut avoir des valeurs permettant d’indiquer que le `maxWallClockTime`, spécifié dans les contraintes du travail, a été dépassé et, par conséquent, que la tâche a été terminée. Elle peut également être définie pour indiquer qu’une tâche a échoué si la propriété `onTaskFailure` du travail a été définie en conséquence.
  - La propriété [schedulingError](/rest/api/batchservice/job/get#jobschedulingerror) est définie si une erreur de planification s’est produite.

### <a name="job-preparation-tasks"></a>Tâches de préparation du travail

Si une tâche de préparation du travail est spécifiée pour un travail, une instance de cette tâche est alors exécutée la première fois qu’une tâche du travail est exécutée sur un nœud. La tâche de préparation du travail configurée sur le travail peut être considérée comme un modèle de tâche, doté de plusieurs instances de tâche de préparation du travail en cours d’exécution, pouvant aller jusqu’au nombre de nœuds dans un pool.

Les instances de tâche de préparation du travail doivent être vérifiées pour déterminer si des erreurs se sont produites :

- Quand une tâche de préparation du travail est exécutée, la tâche qui a déclenché la tâche de préparation du travail passe à un [état](/rest/api/batchservice/task/get#taskstate)`preparing`. Si la tâche de préparation du travail échoue, la tâche de déclenchement revient à l’état `active` et n’est pas exécutée.
- Toutes les instances de la tâche de préparation du travail qui ont été exécutées peuvent être obtenues à partir du travail à l’aide de l’API [Répertorier l’état des tâches de préparation et de validation](/rest/api/batchservice/job/listpreparationandreleasetaskstatus). Comme pour n’importe quelle tâche, des [informations d’exécution](/rest/api/batchservice/job/listpreparationandreleasetaskstatus#jobpreparationandreleasetaskexecutioninformation) sont disponibles avec des propriétés telles que `failureInfo`, `exitCode` et `result`.
- Si les tâches de préparation du travail échouent, les tâches de déclenchement de travail ne sont pas exécutées et le travail n’est pas finalisé et sera bloqué. Le pool peut ne pas être utilisé s’il n’y a pas d’autres travaux dont les tâches peuvent être planifiées.

### <a name="job-release-tasks"></a>Tâches de validation du travail

Si une tâche de validation de travail est spécifiée pour un travail, lorsqu’un travail est en cours d’achèvement, une instance de la tâche de validation du travail est exécutée sur chacun des nœuds du pool où une tâche de préparation du travail a été exécutée. Les instances de tâche de validation du travail doivent être vérifiées pour déterminer si des erreurs se sont produites :

- Toutes les instances de la tâche de validation du travail qui ont été exécutées peuvent être obtenues à partir du travail à l’aide de l’API [Répertorier l’état des tâches de préparation et de validation](/rest/api/batchservice/job/listpreparationandreleasetaskstatus). Comme pour n’importe quelle tâche, des [informations d’exécution](/rest/api/batchservice/job/listpreparationandreleasetaskstatus#jobpreparationandreleasetaskexecutioninformation) sont disponibles avec des propriétés telles que `failureInfo`, `exitCode` et `result`.
- Si une ou plusieurs tâches de validation du travail échouent, le travail sera tout de même terminé et passera à l’état `completed`.

## <a name="tasks"></a>Tâches

Les tâches du travail peuvent échouer pour plusieurs raisons :

- La ligne de commande de la tâche échoue, en retournant avec un code de sortie différent de zéro.
- Des `resourceFiles` sont spécifiés pour une tâche, mais un échec est survenu signifiant qu’un ou plusieurs fichiers n’ont pas été téléchargés.
- Des `outputFiles` sont spécifiés pour une tâche, mais un échec est survenu signifiant qu’un ou plusieurs fichiers n’ont pas été chargés.
- Le temps écoulé de la tâche, spécifié par la propriété `maxWallClockTime` dans la tâche [contraintes](/rest/api/batchservice/task/add#taskconstraints), a été dépassé.

Dans tous les cas, les erreurs et les informations relatives à ces dernières doivent être vérifiées dans les propriétés suivantes :

- La propriété [executionInfo](/rest/api/batchservice/task/get#taskexecutioninformation) des tâches contient plusieurs propriétés qui fournissent des informations sur une erreur. [result](/rest/api/batchservice/task/get#taskexecutionresult) indique si la tâche a échoué pour une raison quelconque, avec `exitCode` et `failureInfo` fournissant des informations supplémentaires sur l’échec.
- La tâche passe toujours à l’ [état](/rest/api/batchservice/task/get#taskstate)`completed`, indépendamment du fait qu’elle a réussi ou échoué.

L’impact des échecs de tâche sur le travail et les dépendances de tâches doivent être pris en compte. La propriété [exitConditions](/rest/api/batchservice/task/add#exitconditions) peut être spécifiée pour une tâche afin de configurer une action pour les dépendances et le travail.

- Pour les dépendances, [DependencyAction](/rest/api/batchservice/task/add#dependencyaction) contrôle si les tâches dépendantes de la tâche ayant échoué sont bloquées ou exécutées.
- Pour le travail, [JobAction](/rest/api/batchservice/task/add#jobaction) contrôle si la tâche ayant échoué a pour conséquence que le travail est désactivé, terminé ou laissé inchangé.

### <a name="task-command-line-failures"></a>Échecs de ligne de commande de la tâche

Lors de l’exécution de la ligne de commande de la tâche, la sortie est écrite dans `stderr.txt` et `stdout.txt`. En outre, l’application peut écrire dans des fichiers journaux qui lui sont spécifiques.

Si le nœud de pool sur lequel une tâche a été exécutée existe toujours, il est possible d’obtenir et d’afficher les fichiers journaux. Par exemple, le portail Azure répertorie et peut afficher les fichiers journaux d’une tâche ou d’un nœud de pool. Plusieurs API permettent également de répertorier et d’obtenir des fichiers de tâche. C’est notamment le cas de l’API [Obtenir à partir de la tâche](/rest/api/batchservice/file/getfromtask).

Les pools et les nœuds de pool étant souvent éphémères, et des nœuds étant continuellement ajoutés et supprimés, nous vous recommandons d’enregistrer les fichiers journaux. Les [fichiers de sortie de tâche](./batch-task-output-files.md) sont un moyen pratique d’enregistrer des fichiers journaux dans Stockage Azure.

Les lignes de commande exécutées par les tâches sur les nœuds de calcul ne s’exécutent pas sous un interpréteur de commandes. Elles ne peuvent donc pas tirer parti en mode natif des fonctionnalités de l’interpréteur de commandes telles que l’expansion des variables d’environnement. Pour tirer parti de ces fonctionnalités, vous devez [appeler l’interpréteur de commandes dans la ligne de commande](batch-compute-node-environment-variables.md#command-line-expansion-of-environment-variables).

### <a name="output-file-failures"></a>Échecs de sortie de fichiers

À chaque chargement de fichier, Batch écrit deux fichiers journaux dans le nœud de calcul, `fileuploadout.txt` et `fileuploaderr.txt`. Vous pouvez examiner ces fichiers journaux pour en savoir plus sur un échec spécifique. Si le chargement de fichier n’a jamais été tenté, par exemple, parce que la tâche proprement dite n’a pas pu être exécutée, ces fichiers journaux n’existent pas.  

## <a name="next-steps"></a>Étapes suivantes

- Vérifiez que votre application implémente une vérification complète des erreurs. Il peut être essentiel de détecter et de diagnostiquer rapidement les problèmes.
- Apprenez-en davantage sur [les travaux et les tâches](jobs-and-tasks.md).