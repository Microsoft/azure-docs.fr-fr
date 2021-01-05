---
title: Didacticiel – Planifier une tâche ACR
description: Dans ce didacticiel, découvrez comment exécuter une tâche Azure Container Registry Task selon une planification définie en définissant un ou plusieurs déclencheurs de minuteur.
ms.topic: article
ms.date: 11/24/2020
ms.openlocfilehash: 13a4ccac4ea97538583c1c063a6dc61e4d25686a
ms.sourcegitcommit: 2e9643d74eb9e1357bc7c6b2bca14dbdd9faa436
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96030609"
---
# <a name="tutorial-run-an-acr-task-on-a-defined-schedule"></a>Tutoriel : Exécuter une tâche ACR selon une planification définie

Ce didacticiel montre comment exécuter une [tâche ACR](container-registry-tasks-overview.md) selon une planification. Planifiez une tâche en définissant un ou plusieurs *déclencheurs de minuteur*. Les déclencheurs de minuteur peuvent être utilisés seuls ou en combinaison avec d’autres déclencheurs de tâche.

Dans ce didacticiel, découvrez comment planifier des tâches et :

> [!div class="checklist"]
> * Créer une tâche avec un déclencheur de minuteur
> * Gérer les déclencheurs de minuteur

Planifier une tâche est utile pour les scénarios tels que les suivants :

* Exécuter une charge de travail de conteneur pour les opérations de maintenance planifiée. Par exemple, exécuter une application conteneurisée pour supprimer les images inutiles de votre registre.
* Exécuter un ensemble de tests sur une image de production pendant la journée dans le cadre de votre analyse de site actif.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

## <a name="about-scheduling-a-task"></a>À propos de la planification d’une tâche

* **Déclencheur avec expression cron** - Le déclencheur de minuteur d'une tâche utilise une *expression cron*. L’expression est une chaîne à cinq champs spécifiant la minute, l'heure, le jour, le mois et le jour de la semaine où déclencher la tâche. Les fréquences jusqu'à une fois par minute sont prises en charge.

  Par exemple, l’expression `"0 12 * * Mon-Fri"` déclenche une tâche à midi UTC chaque jour de la semaine. Consultez les [détails](#cron-expressions) plus loin dans cet article.
* **Déclencheurs de minuteurs multiples** - L'ajout de plusieurs minuteurs à une tâche est autorisée, à condition que les planifications diffèrent.
    * Spécifiez plusieurs déclencheurs de minuteurs lorsque vous créez la tâche ou ajoutez-les ultérieurement.
    * Nommez les déclencheurs pour faciliter leur gestion, à défaut de quoi, ACR Tasks leur attribue des noms par défaut.
    * Si des planifications de minuteur se chevauchent, ACR Tasks déclenche la tâche à l’heure planifiée pour chaque minuteur.
* **Autres déclencheurs de tâche** : dans le cadre d'une tâche déclenchée par minuteur, vous pouvez également activer des déclencheurs basés sur la [validation du code source](container-registry-tutorial-build-task.md) ou les [mises à jour d’images de base](container-registry-tutorial-base-image-update.md). Comme d’autres fonctionnalités ACR Tasks, vous pouvez également [exécuter manuellement][az-acr-task-run] une tâche planifiée.

## <a name="create-a-task-with-a-timer-trigger"></a>Créer une tâche avec un déclencheur de minuteur

### <a name="task-command"></a>Commande de la tâche

Tout d’abord, renseignez la variable d’environnement d’interpréteur de commandes suivante à l’aide d’une valeur appropriée pour votre environnement. Cette étape n’est pas strictement obligatoire, mais facilite un peu l’exécution des commandes multilignes de l’interface de ligne de commande Azure de ce didacticiel. Si vous ne renseignez pas la variable d’environnement, vous devez remplacer manuellement chaque valeur chaque fois qu’elle apparaît dans les exemples de commandes.

[![Lancement de l’incorporation](https://shell.azure.com/images/launchcloudshell.png "Lancement d’Azure Cloud Shell")](https://shell.azure.com)

```console
ACR_NAME=<registry-name>        # The name of your Azure container registry
```

Lorsque vous créez une tâche avec la commande [az acr task create][az-acr-task-create], vous pouvez ajouter un déclencheur de minuteur. Ajoutez le paramètre `--schedule` et transmettez une expression cron pour le minuteur.

À titre d’exemple simple, la tâche suivante déclenche l’exécution de l’image `hello-world` à partir de Microsoft Container Registry tous les jours à 21 h 00 UTC. La tâche s’exécute sans contexte de code source.

```azurecli
az acr task create \
  --name timertask \
  --registry $ACR_NAME \
  --cmd mcr.microsoft.com/hello-world \
  --schedule "0 21 * * *" \
  --context /dev/null
```

Exécutez la commande [az acr task show][az-acr-task-show] pour vérifier que le déclencheur de minuteur est configuré. Par défaut, le déclencheur de mise à jour d’image de base est également activé.

```azurecli
az acr task show --name timertask --registry $ACR_NAME --output table
```

```output
NAME      PLATFORM    STATUS    SOURCE REPOSITORY       TRIGGERS
--------  ----------  --------  -------------------     -----------------
timertask linux       Enabled                           BASE_IMAGE, TIMER
```

## <a name="trigger-the-task"></a>Déclencher la tâche

Déclenchez la tâche manuellement avec [az acr task run][az-acr-task-run] pour vous assurer qu’elle est correctement configurée :

```azurecli
az acr task run --name timertask --registry $ACR_NAME
```

Si le conteneur s’exécute correctement, la sortie ressemble à ce qui suit. La sortie est condensée pour afficher les principales étapes.

```output
Queued a run with ID: cf2a
Waiting for an agent...
2020/11/20 21:03:36 Using acb_vol_2ca23c46-a9ac-4224-b0c6-9fde44eb42d2 as the home volume
2020/11/20 21:03:36 Creating Docker network: acb_default_network, driver: 'bridge'
[...]
2020/11/20 21:03:38 Launching container with name: acb_step_0

Hello from Docker!
This message shows that your installation appears to be working correctly.
[...]
```

Après l’heure planifiée, exécutez la commande [az acr task list-runs][az-acr-task-list-runs] pour vérifier que le minuteur a déclenché la tâche comme prévu :

```azurecli
az acr task list-runs --name timertask --registry $ACR_NAME --output table
```

Si tel est le cas, la sortie ressemble à ce qui suit :

```output
RUN ID    TASK       PLATFORM    STATUS     TRIGGER    STARTED               DURATION
--------  ---------  ----------  ---------  ---------  --------------------  ----------
ca15      timertask  linux       Succeeded  Timer      2020-11-20T21:00:23Z  00:00:06
ca14      timertask  linux       Succeeded  Manual     2020-11-20T20:53:35Z  00:00:06
```

## <a name="manage-timer-triggers"></a>Gérer les déclencheurs de minuteur

Utilisez la commande [az acr task timer][az-acr-task-timer] pour gérer les déclencheurs de minuteur pour une tâche d’ACR.

### <a name="add-or-update-a-timer-trigger"></a>Ajouter ou mettre à jour un déclencheur de minuteur

Après avoir créé une tâche, vous pouvez lui ajouter un déclencheur de minuteur en utilisant la commande [az acr task timer add][az-acr-task-timer-add]. L’exemple suivant ajoute un nom de déclencheur de minuteur *timer2* à la tâche *timertask* créée. Ce minuteur déclenche la tâche tous les jours à 10:30 UTC.

```azurecli
az acr task timer add \
  --name timertask \
  --registry $ACR_NAME \
  --timer-name timer2 \
  --schedule "30 10 * * *"
```

Mettez à jour la planification d’un déclencheur existant ou modifier son état, à l’aide de la commande [az acr task timer update][az-acr-task-timer-update]. Par exemple, mettez à jour le déclencheur nommé *timer2* pour déclencher la tâche à 11:30 UTC :

```azurecli
az acr task timer update \
  --name timertask \
  --registry $ACR_NAME \
  --timer-name timer2 \
  --schedule "30 11 * * *"
```

### <a name="list-timer-triggers"></a>Répertorier les déclencheurs de minuteur

La commande [az acr task timer list][az-acr-task-timer-list] affiche les déclencheurs de minuteur définis pour une tâche :

```azurecli
az acr task timer list --name timertask --registry $ACR_NAME
```

Exemple de sortie :

```JSON
[
  {
    "name": "timer2",
    "schedule": "30 11 * * *",
    "status": "Enabled"
  },
  {
    "name": "t1",
    "schedule": "0 21 * * *",
    "status": "Enabled"
  }
]
```

### <a name="remove-a-timer-trigger"></a>Supprimer un déclencheur de minuteur

Utilisez la commande [az acr task timer remove][az-acr-task-timer-remove] commande pour supprimer un déclencheur de minuteur associé à une tâche. L’exemple suivant supprime le déclencheur *timer2* de *timertask* :

```azurecli
az acr task timer remove \
  --name timertask \
  --registry $ACR_NAME \
  --timer-name timer2
```

## <a name="cron-expressions"></a>Expressions cron

ACR Tasks utilise la bibliothèque [NCronTab](https://github.com/atifaziz/NCrontab) pour interpréter les expressions cron. Les expressions prises en charge dans ACR Tasks présentent cinq champs requis, séparés par un espace blanc :

`{minute} {hour} {day} {month} {day-of-week}`

Le fuseau horaire utilisé avec les expressions cron est le temps universel coordonné (UTC). Les heures sont au format 24 heures.

> [!NOTE]
> ACR Tasks ne prend pas en charge le champ `{second}` ou `{year}` dans les expressions cron. Si vous copiez une expression cron utilisée dans un autre système, supprimez ces champs, le cas échéant.

Chaque champ peut être associé aux types de valeurs suivants :

|Type  |Exemple  |En cas de déclenchement  |
|---------|---------|---------|
|Une valeur spécifique |<nobr>`"5 * * * *"`</nobr>|toutes les heures, 5 minutes après l’heure|
|Toutes les valeurs (`*`)|<nobr>`"* 5 * * *"`</nobr>|toutes les minutes, à partir de 5:00 UTC (60 fois par jour)|
|Une plage (opérateur `-`)|<nobr>`"0 1-3 * * *"`</nobr>|3 fois par jour à 1:00, 2:00 et 3:00 UTC|
|Un ensemble de valeurs (opérateur `,`)|<nobr>`"20,30,40 * * * *"`</nobr>|3 fois par heure, 20 minutes, 30 minutes et 40 minutes après l’heure|
|Une valeur d’intervalle (opérateur `/`)|<nobr>`"*/10 * * * *"`</nobr>|6 fois par heure, 10 minutes, 20 minutes et ainsi de suite, après l’heure

[!INCLUDE [functions-cron-expressions-months-days](../../includes/functions-cron-expressions-months-days.md)]

### <a name="cron-examples"></a>Exemples cron

|Exemple|En cas de déclenchement  |
|---------|---------|
|`"*/5 * * * *"`|une fois toutes les cinq minutes|
|`"0 * * * *"`|une fois toutes les heures|
|`"0 */2 * * *"`|une fois toutes les deux heures|
|`"0 9-17 * * *"`|une fois toutes les heures entre 9:00 et 17:00 UTC|
|`"30 9 * * *"`|à 9:30 UTC tous les jours|
|`"30 9 * * 1-5"`|à 9:30 UTC tous les jours de la semaine|
|`"30 9 * Jan Mon"`|à 9:30 UTC tous les lundis en janvier|

## <a name="clean-up-resources"></a>Nettoyer les ressources

Pour supprimer toutes les ressources créées dans cette série de tutoriels, y compris le ou les registres de conteneurs, l’instance de conteneur, le coffre de clés et le principal du service, exécutez les commandes suivantes :

```azurecli
az group delete --resource-group $RES_GROUP
az ad sp delete --id http://$ACR_NAME-pull
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à créer des tâches Azure Container Registry qui sont déclenchées automatiquement par un minuteur. 

Pour obtenir un exemple d’utilisation d’une tâche planifiée afin de nettoyer des dépôts dans un registre , consultez [Purger automatiquement les images d’un registre de conteneurs Azure](container-registry-auto-purge.md).

Pour obtenir des exemples de tâches déclenchées par les validations de code source ou les mises à jour d'images de base, consultez d’autres articles dans les [didacticiels ACR Tasks](container-registry-tutorial-quick-task.md).



<!-- LINKS - External -->
[task-examples]: https://github.com/Azure-Samples/acr-tasks


<!-- LINKS - Internal -->
[az-acr-task-create]: /cli/azure/acr/task#az-acr-task-create
[az-acr-task-show]: /cli/azure/acr/task#az-acr-task-show
[az-acr-task-list-runs]: /cli/azure/acr/task#az-acr-task-list-runs
[az-acr-task-timer]: /cli/azure/acr/task/timer
[az-acr-task-timer-add]: /cli/azure/acr/task/timer#az-acr-task-timer-add
[az-acr-task-timer-remove]: /cli/azure/acr/task/timer#az-acr-task-timer-remove
[az-acr-task-timer-list]: /cli/azure/acr/task/timer#az-acr-task-timer-list
[az-acr-task-timer-update]: /cli/azure/acr/task/timer#az-acr-task-timer-update
[az-acr-task-run]: /cli/azure/acr/task#az-acr-task-run
[az-acr-task]: /cli/azure/acr/task
[azure-cli-install]: /cli/azure/install-azure-cli
