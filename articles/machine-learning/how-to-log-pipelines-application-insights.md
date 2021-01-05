---
title: Analyser & collecter les fichiers journaux de pipeline
titleSuffix: Azure Machine Learning
description: Ajoutez la journalisation à vos pipelines de formation et de scoring par lots et affichez les résultats enregistrés dans Application Insights.
services: machine-learning
author: NilsPohlmann
ms.author: nilsp
ms.service: machine-learning
ms.subservice: core
ms.date: 08/11/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python
ms.openlocfilehash: 624409be4d7e2cfba37dbe16e083904766ae1389
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93309658"
---
# <a name="collect-machine-learning-pipeline-log-files-in-application-insights-for-alerts-and-debugging"></a>Collecter les fichiers journaux de pipeline de Machine Learning dans Application Insights pour les alertes et le débogage


La bibliothèque Python [OpenCensus](https://opencensus.io/quickstart/python/) peut être utilisée pour acheminer les journaux vers Application Insights à partir de vos scripts. L’agrégation des journaux issus des exécutions de pipeline dans un même emplacement vous permet de créer des requêtes et de diagnostiquer les problèmes. L’utilisation d’Application Insights vous permet de suivre les journaux dans le temps et de comparer les journaux de pipeline entre les exécutions.

Le fait de disposer de vos journaux dans un emplacement unique permet d’obtenir un historique des exceptions et des messages d’erreur. Étant donné qu’Application Insights s’intègre aux alertes Azure, vous pouvez également créer des alertes basées sur des requêtes Application Insights.

## <a name="prerequisites"></a>Prérequis

* Suivez les étapes pour créer un espace de travail [Azure Machine Learning](./how-to-manage-workspace.md) et [créer votre premier pipeline](./how-to-create-your-first-pipeline.md)
* [Configurez votre environnement de développement](./how-to-configure-environment.md) pour installer le SDK Azure Machine Learning.
* Installez le package [OpenCensus Azure Monitor Exporter](https://pypi.org/project/opencensus-ext-azure/) localement :
  ```python
  pip install opencensus-ext-azure
  ```
* Créer une [instance Application Insights](../azure-monitor/app/opencensus-python.md) (ce document contient également des informations sur l’obtention de la chaîne de connexion pour la ressource)

## <a name="getting-started"></a>Mise en route

Cette section est une introduction spécifique à l’utilisation d’OpenCensus à partir d’un pipeline Azure Machine Learning. Pour obtenir un didacticiel détaillé, consultez [Exportateurs OpenCensus Azure Monitor](https://github.com/census-instrumentation/opencensus-python/tree/master/contrib/opencensus-ext-azure)

Ajoutez un PythonScriptStep à votre pipeline Azure ML. Configurez votre [RunConfiguration](/python/api/azureml-core/azureml.core.runconfiguration?preserve-view=true&view=azure-ml-py) avec la dépendance définie sur opencensus-ext-azure. Configurez la variable d’environnement `APPLICATIONINSIGHTS_CONNECTION_STRING`.

```python
from azureml.core.conda_dependencies import CondaDependencies
from azureml.core.runconfig import RunConfiguration
from azureml.pipeline.core import Pipeline
from azureml.pipeline.steps import PythonScriptStep

# Connecting to the workspace and compute target not shown

# Add pip dependency on OpenCensus
dependencies = CondaDependencies()
dependencies.add_pip_package("opencensus-ext-azure>=1.0.1")
run_config = RunConfiguration(conda_dependencies=dependencies)

# Add environment variable with Application Insights Connection String
# Replace the value with your own connection string
run_config.environment.environment_variables = {
    "APPLICATIONINSIGHTS_CONNECTION_STRING": 'InstrumentationKey=00000000-0000-0000-0000-000000000000'
}

# Configure step with runconfig
sample_step = PythonScriptStep(
        script_name="sample_step.py",
        compute_target=compute_target,
        runconfig=run_config
)

# Submit new pipeline run
pipeline = Pipeline(workspace=ws, steps=[sample_step])
pipeline.submit(experiment_name="Logging_Experiment")
```

Création d’un fichier appelé `sample_step.py`. Importez la classe AzureLogHandler pour acheminer les journaux vers Application Insights. Vous devez également importer la bibliothèque de journalisation Python.

```python
from opencensus.ext.azure.log_exporter import AzureLogHandler
import logging
```

Ensuite, ajoutez AzureLogHandler à l’enregistreur d’événements Python.

```python
logger = logging.getLogger(__name__)
logger.setLevel(logging.DEBUG)
logger.addHandler(logging.StreamHandler())

# Assumes the environment variable APPLICATIONINSIGHTS_CONNECTION_STRING is already set
logger.addHandler(AzureLogHandler())
logger.warning("I will be sent to Application Insights")
```

## <a name="logging-with-custom-dimensions"></a>Journalisation avec des dimensions personnalisées
 
Par défaut, les journaux transférés à Application Insights ne disposent pas de suffisamment de contexte pour remonter jusqu’à l’exécution ou l’expérience. Pour que les journaux soient exploitables pour diagnostiquer des problèmes, des champs supplémentaires sont nécessaires. 

Pour ajouter ces champs, des dimensions personnalisées peuvent être ajoutées pour fournir du contexte à un message de journal. C’est le cas, par exemple, lorsqu’un utilisateur souhaite consulter les journaux de plusieurs étapes d’une même exécution de pipeline.

Les dimensions personnalisées constituent un dictionnaire de paires clé-valeur (stockées sous forme de chaîne, chaîne). Le dictionnaire est ensuite envoyé à Application Insights et affiché sous la forme d’une colonne dans les résultats de la requête. Ses dimensions individuelles peuvent être utilisées comme des [paramètres de requête](#additional-helpful-queries).

### <a name="helpful-context-to-include"></a>Contexte utile à inclure

| Champ                          | Raisonnement/exemple                                                                                                                                                                       |
|--------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| parent_run_id                  | Peut interroger les journaux ayant le même parent_run_id pour voir les journaux dans le temps pour toutes les étapes, au lieu de devoir explorer chaque étape individuelle                                        |
| step_id                        | Peut interroger les journaux ayant le même step_id pour voir où un problème s’est produit avec une étendue limitée à la seule étape                                                        |
| step_name                      | Peut interroger les journaux pour voir les performances de l’étape au fil du temps. Permet également de trouver un step_id pour les exécutions récentes sans explorer l’interface utilisateur du portail                                          |
| experiment_name                | Peut interroger des journaux pour voir les performances des expériences dans le temps. Permet également de trouver un parent_run_id ou step_id pour les exécutions récentes sans explorer l’interface utilisateur du portail                   |
| run_url                 | Peut fournir un lien directement à l’exécution pour investigation. |

**Autres champs utiles**

Ces champs peuvent nécessiter une instrumentation de code supplémentaire et ne sont pas fournis par le contexte d’exécution.

| Champ                   | Raisonnement/exemple                                                                                                                                                                                                           |
|-------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| build_url/build_version | Si vous utilisez CI/CD pour le déploiement, ce champ peut corréler les journaux à la version de code qui a fourni la logique d’étape de pipeline. Ce lien peut vous aider à diagnostiquer des problèmes ou à identifier des modèles dotés de caractéristiques spécifiques (valeurs de journal/métrique) |
| run_type                       | Peut faire la distinction entre les différents types de modèles ou entre les exécutions de formation et celles de scoring |

### <a name="creating-a-custom-dimensions-dictionary"></a>Création d’un dictionnaire de dimensions personnalisées

```python
from azureml.core import Run

run = Run.get_context(allow_offline=False)

custom_dimensions = {
    "parent_run_id": run.parent.id,
    "step_id": run.id,
    "step_name": run.name,
    "experiment_name": run.experiment.name,
    "run_url": run.parent.get_portal_url(),
    "run_type": "training"
}

# Assumes AzureLogHandler was already registered above
logger.info("I will be sent to Application Insights with Custom Dimensions", custom_dimensions)
```

## <a name="opencensus-python-logging-considerations"></a>Considérations relatives à la journalisation Python OpenCensus

L’AzureLogHandler OpenCensus est utilisé pour acheminer les journaux Python vers Application Insights. Par conséquent, les nuances de la journalisation Python doivent être prises en compte. Lorsqu’un enregistreur d’événements est créé, il dispose d’un niveau de journalisation par défaut et affiche les journaux supérieurs ou égaux à ce niveau. Une bonne référence pour l’utilisation des fonctionnalités de journalisation Python est le [Livre de recettes de la journalisation](https://docs.python.org/3/howto/logging-cookbook.html).

La variable d’environnement `APPLICATIONINSIGHTS_CONNECTION_STRING` est nécessaire pour la bibliothèque OpenCensus. Nous vous recommandons de définir cette variable d’environnement plutôt que de l’introduire comme paramètre de pipeline pour éviter de faire passer des chaînes de connexion en texte clair.

## <a name="querying-logs-in-application-insights"></a>Interrogation des journaux dans Application Insights

Les journaux acheminés vers Application Insights s’affichent sous « traces » ou « exceptions ». Veillez à ajuster votre fenêtre de temps pour inclure votre exécution de pipeline.

![Résultats de la requête Application Insights](./media/how-to-debug-pipelines-application-insights/traces-application-insights-query.png)

Le résultat dans Application Insights affiche le message du journal ainsi que le niveau, le chemin d’accès du fichier et le numéro de la ligne de code. Il affiche également toutes les dimensions personnalisées incluses. Dans cette image, le dictionnaire customDimensions affiche les paires clé-valeur de l’[exemple de code](#creating-a-custom-dimensions-dictionary) précédent.

### <a name="additional-helpful-queries"></a>Autres requêtes utiles

Certaines des requêtes ci-dessous utilisent « customDimensions.Level ». Ces niveaux de gravité correspondent au niveau avec lequel le journal Python a été envoyé à l’origine. Pour plus d’informations sur les requêtes, consultez [Requêtes de journal Azure Monitor](/azure/data-explorer/kusto/query/).

| Cas d’utilisation                                                               | Requête                                                                                              |
|------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------|
| Consigner les résultats pour une dimension personnalisée spécifique, par exemple « parent_run_id » | <pre>traces \| <br>where customDimensions.parent_run_id == '931024c2-3720-11ea-b247-c49deda841c1</pre> |
| Consigner les résultats pour toutes les exécutions de formation au cours des sept derniers jours                     | <pre>traces \| <br>where timestamp > ago(7d) <br>and customDimensions.run_type == 'training'</pre>           |
| Consigner les résultats accompagnés de severityLevel Error au cours des sept derniers jours              | <pre>traces \| <br>where timestamp > ago(7d) <br>and customDimensions.Level == 'ERROR'                     |
| Nombre de résultats accompagnés de severityLevel Error au cours des sept derniers jours     | <pre>traces \| <br>where timestamp > ago(7d) <br>and customDimensions.Level == 'ERROR' \| <br>summarize count()</pre> |

## <a name="next-steps"></a>Étapes suivantes

Une fois que vous avez des journaux dans votre instance Application Insights, vous pouvez les utiliser pour définir des [alertes Azure Monitor](../azure-monitor/platform/alerts-overview.md#what-you-can-alert-on) en fonction des résultats de la requête.

Vous pouvez également ajouter des résultats de requêtes à un [tableau de bord Azure](../azure-monitor/learn/tutorial-app-dashboards.md#add-logs-query) pour obtenir des informations supplémentaires.