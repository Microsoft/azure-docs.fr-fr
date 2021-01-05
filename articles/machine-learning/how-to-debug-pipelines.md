---
title: Déboguer et dépanner des pipelines ML
titleSuffix: Azure Machine Learning
description: Déboguez vos pipelines Azure Machine Learning en Python. Découvrez les écueils habituels et des conseils pour vous aider à déboguer les scripts avant et pendant l’exécution à distance.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
author: lobrien
ms.author: laobri
ms.date: 10/22/2020
ms.topic: conceptual
ms.custom: troubleshooting, devx-track-python, contperf-fy21q2
ms.openlocfilehash: a150a0745911a70fc71db6b9c05fe6610cd960bf
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97033253"
---
# <a name="debug-and-troubleshoot-machine-learning-pipelines"></a>Déboguer et résoudre les problèmes de pipelines de machine learning

Dans cet article, vous allez découvrir comment déboguer et résoudre les problèmes de [pipelines d’apprentissage automatique](concept-ml-pipelines.md) dans le [SDK Azure Machine Learning](/python/api/overview/azure/ml/intro?preserve-view=true&view=azure-ml-py) et le [concepteur Azure Machine Learning](./concept-designer.md). 

## <a name="troubleshooting-tips"></a>Conseils de dépannage

Le tableau suivant présente les problèmes courants qui se produisent pendant le développement de pipelines ainsi que les solutions possibles.

| Problème | Solution possible |
|--|--|
| Impossible de transmettre les données au répertoire `PipelineData` | Vérifiez que vous avez créé un répertoire dans le script qui correspond à l’emplacement où votre pipeline attend les données de sortie de l’étape. Dans la plupart des cas, un argument d’entrée définit le répertoire de sortie, puis crée le répertoire explicitement. Utilisez `os.makedirs(args.output_dir, exist_ok=True)` pour créer le répertoire de sortie. Pour obtenir un exemple de script de scoring qui illustre ce modèle de conception, consultez ce [tutoriel](tutorial-pipeline-batch-scoring-classification.md#write-a-scoring-script). |
| Bogues de dépendance | Si vous constatez dans votre pipeline à distance des erreurs de dépendance qui ne se sont pas produites lors des tests en local, vérifiez que vos dépendances d’environnement distant et les versions correspondent à celles de votre environnement de test. (Voir [Création, mise en cache et réutilisation d’environnement](./concept-environments.md#environment-building-caching-and-reuse))|
| Erreurs ambiguës liées aux cibles de calcul | Essayez de supprimer et recréer les cibles de calcul. La recréation de cibles de calcul est rapide et peut résoudre certains problèmes temporaires. |
| Le pipeline ne réutilise pas les étapes | La réutilisation d’étape est activée par défaut, mais vérifiez que vous ne l’avez pas désactivée dans une étape du pipeline. Si la réutilisation est désactivée, le paramètre `allow_reuse` de l’étape est défini sur `False`. |
| Le pipeline se réexécute inutilement | Pour faire en sorte que les étapes ne se réexécutent que lorsque leurs données ou scripts sous-jacents changent, découplez les répertoires de votre code source pour chaque étape. Si vous utilisez le même répertoire source pour plusieurs étapes, des réexécutions inutiles peuvent se produire. Utilisez le paramètre `source_directory` sur un objet d’étape de pipeline pour pointer vers votre répertoire isolé pour cette étape, et vérifiez que vous n’utilisez pas le même chemin `source_directory` pour plusieurs étapes. |
| Étape ralentissant les époques d’apprentissage ou tout autre comportement de bouclage | Essayez de faire basculer les écritures de fichier, notamment a journalisation, de `as_mount()` à `as_upload()`. Le mode **montage** utilise un système de fichiers virtualisé distant et charge l’intégralité du fichier chaque fois qu’il y est ajouté. |

## <a name="troubleshooting-parallelrunstep"></a>Résolution des problèmes `ParallelRunStep` 

Le script d'une étape `ParallelRunStep` *doit contenir* deux fonctions :
- `init()`: utilisez cette fonction pour toute préparation coûteuse ou courante à une prochaine inférence. Par exemple, utilisez-la pour charger le modèle dans un objet global. Cette fonction est appelée une seule fois au début du processus.
-  `run(mini_batch)`: cette fonction s’exécute pour chaque instance de `mini_batch`.
    -  `mini_batch` : `ParallelRunStep` va appeller la méthode d’exécution, et passer une liste ou un `DataFrame` Pandas en tant qu’argument à la méthode. Chaque entrée de mini_batch sera un chemin de fichier si l’entrée est un `FileDataset`, ou un `DataFrame` Pandas si l’entrée est un `TabularDataset`.
    -  `response` : la méthode run() doit retourner un `DataFrame` Pandas ou un tableau. Pour append_row output_action, les éléments retournés sont ajoutés au fichier de sortie commun. Pour summary_only, le contenu des éléments est ignoré. Pour toutes les actions de sortie, chaque élément de sortie retourné indique la réussite de l’exécution d’une entrée dans le mini-lot d’entrée. Vérifiez que suffisamment de données sont incluses dans le résultat de l’exécution pour mapper l’entrée au résultat de la sortie de l’exécution. La sortie de l’exécution sera écrite dans le fichier de sortie, mais pas nécessairement dans l’ordre. Vous devez utiliser une clé dans la sortie pour la mapper à l’entrée.

```python
%%writefile digit_identification.py
# Snippets from a sample script.
# Refer to the accompanying digit_identification.py
# (https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/machine-learning-pipelines/parallel-run)
# for the implementation script.

import os
import numpy as np
import tensorflow as tf
from PIL import Image
from azureml.core import Model


def init():
    global g_tf_sess

    # Pull down the model from the workspace
    model_path = Model.get_model_path("mnist")

    # Construct a graph to execute
    tf.reset_default_graph()
    saver = tf.train.import_meta_graph(os.path.join(model_path, 'mnist-tf.model.meta'))
    g_tf_sess = tf.Session()
    saver.restore(g_tf_sess, os.path.join(model_path, 'mnist-tf.model'))


def run(mini_batch):
    print(f'run method start: {__file__}, run({mini_batch})')
    resultList = []
    in_tensor = g_tf_sess.graph.get_tensor_by_name("network/X:0")
    output = g_tf_sess.graph.get_tensor_by_name("network/output/MatMul:0")

    for image in mini_batch:
        # Prepare each image
        data = Image.open(image)
        np_im = np.array(data).reshape((1, 784))
        # Perform inference
        inference_result = output.eval(feed_dict={in_tensor: np_im}, session=g_tf_sess)
        # Find the best probability, and add it to the result list
        best_result = np.argmax(inference_result)
        resultList.append("{}: {}".format(os.path.basename(image), best_result))

    return resultList
```

Si vous avez un autre fichier ou dossier dans le même répertoire que votre script d’inférence, vous pouvez le référencer en recherchant le répertoire de travail actuel.

```python
script_dir = os.path.realpath(os.path.join(__file__, '..',))
file_path = os.path.join(script_dir, "<file_name>")
```

### <a name="parameters-for-parallelrunconfig"></a>Paramètres de ParallelRunConfig

`ParallelRunConfig` est la configuration principale de l’instance `ParallelRunStep` dans le pipeline Azure Machine Learning. Elle permet de wrapper votre script et de configurer les paramètres nécessaires, dont toutes les entrées suivantes :
- `entry_script`: script utilisateur utilisé comme un chemin de fichier local qui sera exécuté en parallèle sur plusieurs nœuds. Si `source_directory` est présent, utilisez un chemin relatif. Dans le cas contraire, utilisez un chemin accessible sur la machine.
- `mini_batch_size`: taille du mini-lot passé à un appel `run()` unique (Facultatif ; la valeur par défaut est `10` fichiers pour `FileDataset` et `1MB` pour `TabularDataset`.)
    - Pour `FileDataset`, il s’agit du nombre de fichiers avec une valeur minimale de `1`. Vous pouvez combiner plusieurs fichiers dans un mini-lot.
    - Pour `TabularDataset`, il s’agit de la taille des données. Par exemple, il peut s’agir des valeurs `1024`, `1024KB`, `10MB` ou `1GB`. `1MB` est la valeur recommandée. Le mini-lot de `TabularDataset` ne franchira jamais les limites du fichier. Par exemple, si vous avez des fichiers .csv de différentes tailles, le plus petit fichier aura une taille de 100 Ko et le plus grand une taille de 10 Mo. Si vous définissez `mini_batch_size = 1MB`, les fichiers dont la taille est inférieure à 1 Mo seront traités ensemble comme un mini-lot. Les fichiers dont la taille est supérieure à 1 Mo seront répartis dans plusieurs mini-lots.
- `error_threshold`: nombre d’échecs d’enregistrement pour `TabularDataset` et d’échecs de fichiers pour `FileDataset` qui doivent être ignorés pendant le traitement. Si le nombre d’erreurs présentes dans la totalité de l’entrée dépasse cette valeur, le travail est annulé. Le seuil d’erreur concerne la totalité de l’entrée et non le mini-lot envoyé à la méthode `run()`. La plage est la suivante : `[-1, int.max]`. La partie `-1` indique qu’il faut ignorer tous les échecs au cours du traitement.
- `output_action`: l’une des valeurs suivantes indique comment la sortie sera organisée :
    - `summary_only`: le script utilisateur va stocker la sortie. `ParallelRunStep` utilisera la sortie uniquement pour le calcul du seuil d’erreurs.
    - `append_row`: pour toutes les entrées, un seul fichier sera créé dans le dossier de sortie. Vous y ajouterez toutes les sorties séparées par une ligne.
- `append_row_file_name`: permet de personnaliser le nom du fichier de sortie pour append_row output_action (facultatif ; la valeur par défaut est `parallel_run_step.txt`).
- `source_directory`: chemins des dossiers qui contiennent tous les fichiers à exécuter sur la cible de calcul (facultatif).
- `compute_target`: Seul `AmlCompute` est pris en charge.
- `node_count`: nombre de nœuds de calcul à utiliser pour l’exécution du script utilisateur.
- `process_count_per_node`: nombre de processus par nœud. La bonne pratique consiste à définir la valeur sur le nombre de GPU ou d’UC dont dispose un nœud (facultatif ; la valeur par défaut est `1`).
- `environment`: définition de l’environnement Python. Vous pouvez la configurer de manière à utiliser un environnement Python existant ou un environnement temporaire. La définition est également chargée de définir les dépendances d’application nécessaires (facultatif).
- `logging_level`: Verbosité du journal. Les valeurs permettant d’augmenter le niveau de verbosité sont les suivantes : `WARNING`, `INFO` et `DEBUG`. (Facultatif ; la valeur par défaut est `INFO`.)
- `run_invocation_timeout`: délai d’attente de l’appel de la méthode `run()`, en secondes. (Facultatif ; la valeur par défaut est `60`.)
- `run_max_try`: nombre maximal de tentatives de `run()` pour un mini-lot. `run()` a échoué si une exception est levée, ou si rien n’est retourné lorsque `run_invocation_timeout` est atteint (facultatif ; la valeur par défaut est `3`). 

Vous pouvez spécifier `mini_batch_size`, `node_count`, `process_count_per_node`, `logging_level`, `run_invocation_timeout` et `run_max_try` en tant que `PipelineParameter` ; ainsi, lorsque vous soumettez à nouveau une exécution de pipeline, vous pouvez ajuster les valeurs des paramètres. Dans cet exemple, vous utilisez `PipelineParameter` pour `mini_batch_size` et `Process_count_per_node`, et vous changez ces valeurs quand vous soumettez à nouveau une exécution ultérieurement. 

### <a name="parameters-for-creating-the-parallelrunstep"></a>Paramètres de création de l'étape ParallelRunStep

Créez ParallelRunStep à l’aide du script, de la configuration de l’environnement et des paramètres. Spécifiez la cible de calcul que vous avez déjà jointe à votre espace de travail en tant que cible d’exécution de votre script d’inférence. Utilisez `ParallelRunStep` pour créer l’étape de pipeline d’inférence par lots, qui accepte tous les paramètres suivants :
- `name`: nom de l’étape, avec les restrictions de nommage suivantes : unique, de 3 à 32 caractères et expression régulière ^\[a-z\]([-a-z0-9]*[a-z0-9])?$.
- `parallel_run_config`: objet `ParallelRunConfig`, comme défini précédemment.
- `inputs`: un ou plusieurs jeux de données Azure Machine Learning à un seul type, à partitionner pour un traitement parallèle.
- `side_inputs`: une ou plusieurs données, ou jeux de données de référence utilisés comme entrées supplémentaires sans avoir besoin d’être partitionnés.
- `output`: objet `PipelineData` qui correspond au répertoire de sortie.
- `arguments`: liste d’arguments passés au script utilisateur. Utilisez unknown_args pour les récupérer dans votre script d’entrée (facultatif).
- `allow_reuse`: indique si l’étape doit réutiliser les résultats précédents lorsqu’elle est exécutée avec les mêmes paramètres ou entrées. Si ce paramètre a la valeur `False`, une nouvelle exécution sera toujours générée pour cette étape pendant l’exécution du pipeline (Facultatif ; la valeur par défaut est `True`.)

```python
from azureml.pipeline.steps import ParallelRunStep

parallelrun_step = ParallelRunStep(
    name="predict-digits-mnist",
    parallel_run_config=parallel_run_config,
    inputs=[input_mnist_ds_consumption],
    output=output_dir,
    allow_reuse=True
)
```

## <a name="debugging-techniques"></a>Techniques de débogage

Il existe trois techniques principales pour déboguer les pipelines : 

* Déboguer des étapes de pipeline individuelles sur votre ordinateur local
* Utiliser une journalisation et Application Insights pour isoler et diagnostiquer la source du problème
* Attacher un débogueur distant à un pipeline en cours d’exécution dans Azure

### <a name="debug-scripts-locally"></a>Déboguer les scripts localement

L’une des erreurs les plus courantes dans un pipeline est que le script de domaine ne s’exécute pas comme prévu, ou contient des erreurs de runtime dans le contexte de calcul distant, qui sont difficiles à déboguer.

Les pipelines eux-mêmes ne peuvent pas être exécutés localement, mais l’exécution de scripts en isolation sur votre ordinateur local vous permet de déboguer plus rapidement dans la mesure où vous n’avez pas besoin d’attendre le processus de génération de calcul et d’environnement. Cela demande un peu de travail de développement :

* Si vos données se trouvent dans un magasin de données cloud, vous devez les télécharger et les rendre accessibles à votre script. Utiliser un petit échantillon de données est un bon moyen de réduire la durée d’exécution et d’être rapidement renseigné sur le comportement du script.
* Si vous tentez de simuler une étape de pipeline intermédiaire, vous devrez peut-être créer manuellement les types d’objet que le script attend de l’étape précédente.
* Vous devrez aussi définir votre propre environnement et répliquer les dépendances définies dans votre environnement de calcul distant.

Une fois que vous avez configuré un script pour qu’il s’exécute dans un environnement local, il est beaucoup plus facile d’effectuer certaines tâches de débogage, notamment :

* Attacher une configuration de débogage personnalisée
* Suspendre l’exécution et inspecter l’état des objets
* Intercepter les erreurs de type ou les erreurs logiques qui n’apparaîtront pas avant l’exécution

> [!TIP] 
> Une fois que vous pouvez vérifier que votre script s’exécute comme prévu, nous vous recommandons d’exécuter le script dans un pipeline à une seule étape avant de tenter de l’exécuter dans un pipeline à plusieurs étapes.

## <a name="configure-write-to-and-review-pipeline-logs"></a>Configurer, écrire et consulter des journaux de pipeline

Tester les scripts localement est un excellent moyen de déboguer des fragments de code importants et une logique complexe avant de commencer à créer un pipeline. Mais à un moment donné, vous devrez probablement déboguer des scripts pendant l’exécution effective du pipeline, surtout quand il s’agira de diagnostiquer le comportement que les étapes du pipeline interagiront. Nous vous recommandons d’utiliser à loisir les instructions `print()` dans les scripts d’étapes pour voir l’état des objets et les valeurs attendues pendant l’exécution distante, comme vous le feriez pour déboguer du code JavaScript.

### <a name="logging-options-and-behavior"></a>Options et comportement de journalisation

Le tableau ci-dessous fournit des informations sur les différentes options de débogage pour les pipelines. Cette liste n’est pas exhaustive, car il existe d’autres options que les solutions zure Machine Learning, Python ou OpenCensus présentées ici.

| Bibliothèque                    | Type   | Exemple                                                          | Destination                                  | Ressources                                                                                                                                                                                                                                                                                                                    |
|----------------------------|--------|------------------------------------------------------------------|----------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Kit de développement logiciel (SDK) Azure Machine Learning | Métrique | `run.log(name, val)`                                             | Interface utilisateur du portail Azure Machine Learning             | [Comment suivre les expériences](how-to-track-experiments.md)<br>[azureml.core.Run class](/python/api/azureml-core/azureml.core.run%28class%29?preserve-view=true&view=azure-ml-py)                                                                                                                                                 |
| Impression/Journalisation pour Python    | Journal    | `print(val)`<br>`logging.info(message)`                          | Journaux du pilote, concepteur Azure Machine Learning | [Comment suivre les expériences](how-to-track-experiments.md)<br><br>[Journalisation pour Python](https://docs.python.org/2/library/logging.html)                                                                                                                                                                       |
| OpenCensus pour Python          | Journal    | `logger.addHandler(AzureLogHandler())`<br>`logging.log(message)` | Application Insights – Traces                | [Déboguer des pipelines dans Application Insights](./how-to-log-pipelines-application-insights.md)<br><br>[Exportateurs OpenCensus Azure Monitor](https://github.com/census-instrumentation/opencensus-python/tree/master/contrib/opencensus-ext-azure)<br>[Fiches pratiques concernant la journalisation pour Python](https://docs.python.org/3/howto/logging-cookbook.html) |

#### <a name="logging-options-example"></a>Exemple d’options du journalisation

```python
import logging

from azureml.core.run import Run
from opencensus.ext.azure.log_exporter import AzureLogHandler

run = Run.get_context()

# Azure ML Scalar value logging
run.log("scalar_value", 0.95)

# Python print statement
print("I am a python print statement, I will be sent to the driver logs.")

# Initialize python logger
logger = logging.getLogger(__name__)
logger.setLevel(args.log_level)

# Plain python logging statements
logger.debug("I am a plain debug statement, I will be sent to the driver logs.")
logger.info("I am a plain info statement, I will be sent to the driver logs.")

handler = AzureLogHandler(connection_string='<connection string>')
logger.addHandler(handler)

# Python logging with OpenCensus AzureLogHandler
logger.warning("I am an OpenCensus warning statement, find me in Application Insights!")
logger.error("I am an OpenCensus error statement with custom dimensions", {'step_id': run.id})
``` 

## <a name="azure-machine-learning-designer"></a>Concepteur Azure Machine Learning

Pour les pipelines créés dans le concepteur, vous trouverez le fichier **70_driver_log** dans la page de création ou dans la page des détails d’exécutions de pipeline.

### <a name="enable-logging-for-real-time-endpoints"></a>Activer la journalisation pour les points de terminaison en temps réel

Pour dépanner et déboguer des points de terminaison en temps réel dans le concepteur, vous devez activer la journalisation Application Insights à l’aide du Kit de développement logiciel (SDK). La journalisation vous permet de déboguer et dépanner les problèmes de déploiement et d’utilisation du modèle. Pour plus d’informations, consultez [Journalisation pour les modèles déployés](./how-to-enable-app-insights.md). 

### <a name="get-logs-from-the-authoring-page"></a>Obtenir des journaux à partir de la page de création

Quand vous publiez une exécution de pipeline en restant dans la page de création, vous voyez les fichiers journaux générés pour chaque module à mesure que chaque module termine son exécution.

1. Sélectionnez un module dont l’exécution est terminée dans le canevas de création.
1. Dans le volet droit du module, accédez à l’onglet **Sorties + journaux**.
1. Développez le volet droit, puis sélectionnez le fichier **70_driver_log.txt** pour afficher le fichier dans le navigateur. Vous pouvez également télécharger les journaux localement.

    ![Volet de sortie développé dans le concepteur](./media/how-to-debug-pipelines/designer-logs.png)?view=azure-ml-py&preserve-view=true)?view=azure-ml-py&preserve-view=true)

### <a name="get-logs-from-pipeline-runs"></a>Obtenir des journaux à partir des exécutions de pipeline

Vous pouvez également trouver les fichiers journaux d’exécutions spécifiques dans la page des détails d’exécutions de pipeline, qui est disponible dans la section **Pipelines** ou la section **Expériences** du studio.

1. Sélectionnez une exécution de pipeline créée dans le concepteur.

    ![Page d’exécutions de pipeline](./media/how-to-debug-pipelines/designer-pipelines.png)

1. Sélectionnez un module dans le volet d’aperçu.
1. Dans le volet droit du module, accédez à l’onglet **Sorties + journaux**.
1. Développez le volet droit pour afficher le fichier **70_driver_log.txt** dans le navigateur, ou sélectionnez le fichier pour télécharger les journaux localement.

> [!IMPORTANT]
> Pour mettre à jour un pipeline à partir de la page des détails d’exécution du pipeline, vous devez **cloner** l’exécution du pipeline dans un nouveau brouillon de pipeline. Une exécution de pipeline est un instantané du pipeline. Elle est similaire à un fichier journal et ne peut pas être modifiée. 

## <a name="application-insights"></a>Application Insights
Pour en savoir plus sur l’utilisation de la bibliothèque Python OpenCensus de cette manière, consultez ce guide : [Déboguer et résoudre les problèmes de pipelines de Machine Learning dans Application Insights](./how-to-log-pipelines-application-insights.md)

## <a name="interactive-debugging-with-visual-studio-code"></a>Débogage interactif avec Visual Studio Code

Dans certains cas, vous devrez peut-être déboguer interactivement le code Python utilisé dans votre pipeline ML. À l’aide de Visual Studio Code (VS Code) et de debugpy, vous pouvez attacher le code au fur et à mesure de son exécution dans l’environnement d’apprentissage. Pour plus d’informations, consultez le [guide de débogage interactif dans VS Code](how-to-debug-visual-studio-code.md#debug-and-troubleshoot-machine-learning-pipelines).

## <a name="next-steps"></a>Étapes suivantes

* Pour accéder à un tutoriel complet sur l'utilisation de `ParallelRunStep`, consultez [Tutoriel : Créer un pipeline Azure Machine Learning pour le scoring par lots](tutorial-pipeline-batch-scoring-classification.md).

* Pour obtenir un exemple complet de Machine Learning automatisé dans des pipelines ML, consultez [Utiliser le ML automatisé dans un pipeline Azure Machine Learning en Python](how-to-use-automlstep-in-pipelines.md).

* Consultez les informations de référence sur le SDK pour obtenir de l’aide sur les packages [azureml-pipelines-core](/python/api/azureml-pipeline-core/?preserve-view=true&view=azure-ml-py) et [azureml-pipelines-steps](/python/api/azureml-pipeline-steps/?preserve-view=true&view=azure-ml-py).

* Consultez la liste des [exceptions et codes d’erreur du concepteur](algorithm-module-reference/designer-error-codes.md).