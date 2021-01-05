---
title: Superviser les applications Python avec Azure Monitor | Microsoft Docs
description: Fournit des instructions permettant de connecter OpenCensus Python à Azure Monitor
ms.topic: conceptual
ms.date: 09/24/2020
ms.reviewer: mbullwin
ms.custom: devx-track-python
ms.openlocfilehash: 1e6376cd8389a4f1f0defebce0a2c7b6d0f9deed
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91323263"
---
# <a name="set-up-azure-monitor-for-your-python-application"></a>Configurer Azure Monitor pour votre application Python

Azure Monitor prend en charge le traçage distribué, la collecte de métriques et la journalisation des applications Python via l’intégration à [OpenCensus](https://opencensus.io). Cet article vous guide pas à pas pour configurer OpenCensus pour Python et envoyer vos données de supervision à Azure Monitor.

## <a name="prerequisites"></a>Prérequis

- Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.
- Installation de Python. Si cet article s’appuie sur [Python 3.7.0](https://www.python.org/downloads/release/python-370/), il est toutefois possible d’utiliser d’autres versions moyennant quelques modifications mineures. Le kit SDK prend uniquement en charge Python v2.7 et v3.4-v3.7.
- Créez une [ressource](./create-new-resource.md) Application Insights. Vous recevrez votre propre clé d’instrumentation (ikey) pour votre ressource.

## <a name="instrument-with-opencensus-python-sdk-for-azure-monitor"></a>Instrumentez à l’aide du Kit de développement logiciel (SDK) OpenCensus Python pour Azure Monitor

Installez les exportateurs OpenCensus Azure Monitor comme suit :

```console
python -m pip install opencensus-ext-azure
```

> [!NOTE]
> La commande `python -m pip install opencensus-ext-azure` suppose qu’une variable d’environnement `PATH` est définie pour votre installation Python. Dans le cas contraire, vous devez fournir le chemin d’accès complet du répertoire contenant votre fichier exécutable Python. Voici un exemple de commande qui en résulte : `C:\Users\Administrator\AppData\Local\Programs\Python\Python37-32\python.exe -m pip install opencensus-ext-azure`.

Le kit SDK utilise trois exportateurs Azure Monitor pour envoyer différents types de données de télémétrie à Azure Monitor. Il s’agit des traces, des métriques et des journaux. Pour plus d’informations sur ces types de télémétrie, consultez [la vue d’ensemble de la plateforme de données](../platform/data-platform.md). Utilisez les instructions suivantes pour envoyer ces types de télémétrie par le biais des trois exportateurs.

## <a name="telemetry-type-mappings"></a>Mappages des types de données de télémétrie

Voici les exportateurs fournis par OpenCensus, mappés aux types de données de télémétrie que vous voyez dans Azure Monitor.

| Pilier de l’observabilité | Type de données de télémétrie dans Azure Monitor    | Explication                                         |
|-------------------------|------------------------------------|-----------------------------------------------------|
| Journaux d’activité                    | Traces, exceptions, customEvents   | Données de télémétrie de journaux, d’exceptions, d’événements |
| Métriques                 | customMetrics, performanceCounters | Métriques personnalisées, conteurs de performances                |
| Suivi                 | Requêtes, dépendances             | Requêtes entrantes, requêtes sortantes                |

### <a name="logs"></a>Journaux d’activité

1. Commençons par générer des données de journal locales.

    ```python
    import logging

    logger = logging.getLogger(__name__)

    def valuePrompt():
        line = input("Enter a value: ")
        logger.warning(line)

    def main():
        while True:
            valuePrompt()

    if __name__ == "__main__":
        main()
    ```

1. Le code vous demande continuellement d’entrer une valeur. Une entrée de journal est émise pour chaque valeur entrée.

    ```output
    Enter a value: 24
    24
    Enter a value: 55
    55
    Enter a value: 123
    123
    Enter a value: 90
    90
    ```

1. Bien qu’entrer des valeurs soit utile à des fins de démonstration, nous voulons au final émettre les données du journal pour Azure Monitor. Passez votre chaîne de connexion directement à l’exportateur. Vous pouvez aussi la spécifier dans une variable d’environnement, `APPLICATIONINSIGHTS_CONNECTION_STRING`. Modifiez le code de l’étape précédente sur la base de l’exemple de code suivant :

    ```python
    import logging
    from opencensus.ext.azure.log_exporter import AzureLogHandler

    logger = logging.getLogger(__name__)

    # TODO: replace the all-zero GUID with your instrumentation key.
    logger.addHandler(AzureLogHandler(
        connection_string='InstrumentationKey=00000000-0000-0000-0000-000000000000')
    )

    def valuePrompt():
        line = input("Enter a value: ")
        logger.warning(line)

    def main():
        while True:
            valuePrompt()

    if __name__ == "__main__":
        main()
    ```

1. L’exportateur envoie les données de journal à Azure Monitor. Vous pouvez trouver les données sous `traces`. 

    > [!NOTE]
    > Dans ce contexte, `traces` n’est pas identique à `tracing`. Ici, `traces` fait référence au type de données de télémétrie que vous voyez dans Azure Monitor quand vous utilisez `AzureLogHandler`. Or, `tracing` fait référence à un concept dans OpenCensus et se rapporte au [suivi distribué](./distributed-tracing.md).

    > [!NOTE]
    > L’enregistreur d’événements (« logger ») racine est configuré avec le niveau d’avertissement (« WARNING »). Cela signifie que tous les journaux que vous envoyez dont le niveau de gravité est inférieur sont ignorés et, à leur tour, ne sont pas envoyés à Azure Monitor. Pour plus d’informations, consultez la [documentation](https://docs.python.org/3/library/logging.html#logging.Logger.setLevel).

1. Vous pouvez aussi ajouter des propriétés personnalisées à vos messages de journal dans l’argument de mot clé *extra* en utilisant le champ custom_dimensions. Ces propriétés se présentent sous forme de paires clé-valeur dans `customDimensions`, dans Azure Monitor.
    > [!NOTE]
    > Pour que cette fonctionnalité fonctionne, vous devez passer un dictionnaire au champ custom_dimensions. Si vous passez des arguments de tout autre type, l’enregistreur d’événements les ignore.

    ```python
    import logging

    from opencensus.ext.azure.log_exporter import AzureLogHandler

    logger = logging.getLogger(__name__)
    # TODO: replace the all-zero GUID with your instrumentation key.
    logger.addHandler(AzureLogHandler(
        connection_string='InstrumentationKey=00000000-0000-0000-0000-000000000000')
    )

    properties = {'custom_dimensions': {'key_1': 'value_1', 'key_2': 'value_2'}}

    # Use properties in logging statements
    logger.warning('action', extra=properties)
    ```

#### <a name="configure-logging-for-django-applications"></a>Configurer la journalisation pour les applications Django

Vous pouvez configurer explicitement la journalisation dans votre code d’application comme ci-dessus pour vos applications Django, ou vous pouvez la spécifier dans la configuration de la journalisation Django. Ce code peut être placé dans n’importe quel fichier que vous utilisez pour la configuration des paramètres Django. Pour savoir comment configurer les paramètres Django, consultez [Django settings](https://docs.djangoproject.com/en/3.0/topics/settings/). Pour plus d’informations sur la configuration de la journalisation, consultez [Django logging](https://docs.djangoproject.com/en/3.0/topics/logging/).

```json
LOGGING = {
    "handlers": {
        "azure": {
            "level": "DEBUG",
        "class": "opencensus.ext.azure.log_exporter.AzureLogHandler",
            "instrumentation_key": "<your-ikey-here>",
         },
        "console": {
            "level": "DEBUG",
            "class": "logging.StreamHandler",
            "stream": sys.stdout,
         },
      },
    "loggers": {
        "logger_name": {"handlers": ["azure", "console"]},
    },
}
```

Veillez à utiliser l’enregistreur d’événements dont le nom est le même que celui spécifié dans votre configuration.

```python
import logging

logger = logging.getLogger("logger_name")
logger.warning("this will be tracked")
```

#### <a name="send-exceptions"></a>Envoyer les exceptions

OpenCensus Python ne suit pas et n’envoie pas automatiquement les données de télémétrie `exception`. Elles sont envoyées via `AzureLogHandler` en utilisant les exceptions de la bibliothèque de journalisation Python. Vous pouvez ajouter des propriétés personnalisées comme pour la journalisation normale.

```python
import logging

from opencensus.ext.azure.log_exporter import AzureLogHandler

logger = logging.getLogger(__name__)
# TODO: replace the all-zero GUID with your instrumentation key.
logger.addHandler(AzureLogHandler(
    connection_string='InstrumentationKey=00000000-0000-0000-0000-000000000000')
)

properties = {'custom_dimensions': {'key_1': 'value_1', 'key_2': 'value_2'}}

# Use properties in exception logs
try:
    result = 1 / 0  # generate a ZeroDivisionError
except Exception:
    logger.exception('Captured an exception.', extra=properties)
```
Sachant que vous devez journaliser les exceptions de manière explicite, il revient à l’utilisateur de décider comment il souhaite journaliser les exceptions non gérées. OpenCensus n’impose pas de restrictions aux utilisateurs quant à la méthode à employer, du moment qu’ils journalisent de manière explicite les données de télémétrie des exceptions.

#### <a name="send-events"></a>Envoyer des événements

Vous pouvez envoyer les données de télémétrie `customEvent` exactement de la même façon que vous envoyez les données de télémétrie `trace`, à ceci près que vous devez utiliser à la place `AzureEventHandler`.

```python
import logging

from opencensus.ext.azure.log_exporter import AzureEventHandler

logger = logging.getLogger(__name__)
logger.addHandler(AzureEventHandler(connection_string='InstrumentationKey=<your-instrumentation_key-here>'))
logger.setLevel(logging.INFO)
logger.info('Hello, World!')
```

#### <a name="sampling"></a>échantillonnage

Pour plus d’informations sur l’échantillonnage dans OpenCensus, consultez [Échantillonnage dans OpenCensus](sampling.md#configuring-fixed-rate-sampling-for-opencensus-python-applications).

#### <a name="log-correlation"></a>Corrélation des journaux

Pour plus d’informations sur l’enrichissement de vos journaux avec les données de contexte de trace, consultez [Intégration des journaux](./correlation.md#log-correlation) OpenCensus Python.

#### <a name="modify-telemetry"></a>Modifier la télémétrie

Pour savoir comment modifier les données de télémétrie suivies avant leur envoi à Azure Monitor, consultez [Processeurs de télémétrie OpenCensus Python](./api-filtering-sampling.md#opencensus-python-telemetry-processors).


### <a name="metrics"></a>Mesures

1. Commençons par générer des données de métrique locales. Nous allons créer une métrique simple pour suivre le nombre de fois que l’utilisateur appuie sur la touche **Entrée**.

    ```python
    from datetime import datetime
    from opencensus.stats import aggregation as aggregation_module
    from opencensus.stats import measure as measure_module
    from opencensus.stats import stats as stats_module
    from opencensus.stats import view as view_module
    from opencensus.tags import tag_map as tag_map_module

    stats = stats_module.stats
    view_manager = stats.view_manager
    stats_recorder = stats.stats_recorder

    prompt_measure = measure_module.MeasureInt("prompts",
                                               "number of prompts",
                                               "prompts")
    prompt_view = view_module.View("prompt view",
                                   "number of prompts",
                                   [],
                                   prompt_measure,
                                   aggregation_module.CountAggregation())
    view_manager.register_view(prompt_view)
    mmap = stats_recorder.new_measurement_map()
    tmap = tag_map_module.TagMap()

    def prompt():
        input("Press enter.")
        mmap.measure_int_put(prompt_measure, 1)
        mmap.record(tmap)
        metrics = list(mmap.measure_to_view_map.get_metrics(datetime.utcnow()))
        print(metrics[0].time_series[0].points[0])

    def main():
        while True:
            prompt()

    if __name__ == "__main__":
        main()
    ```
1. L’exécution répétée du code vous invite à sélectionner **Entrer**. Une métrique est créée pour suivre le nombre de fois que vous appuyez sur **Entrée**. À chaque entrée, la valeur est incrémentée et les informations de la métrique s’affichent dans la console. Les informations incluent la valeur actuelle et l’horodatage en cours au moment de la mise à jour de la métrique.

    ```output
    Press enter.
    Point(value=ValueLong(5), timestamp=2019-10-09 20:58:04.930426)
    Press enter.
    Point(value=ValueLong(6), timestamp=2019-10-09 20:58:06.570167)
    Press enter.
    Point(value=ValueLong(7), timestamp=2019-10-09 20:58:07.138614)
    ```

1. Bien qu’entrer des valeurs soit utile à des fins de démonstration, nous souhaitons surtout émettre les données de métrique vers Azure Monitor. Passez votre chaîne de connexion directement à l’exportateur. Vous pouvez aussi la spécifier dans une variable d’environnement, `APPLICATIONINSIGHTS_CONNECTION_STRING`. Modifiez le code de l’étape précédente sur la base de l’exemple de code suivant :

    ```python
    from datetime import datetime
    from opencensus.ext.azure import metrics_exporter
    from opencensus.stats import aggregation as aggregation_module
    from opencensus.stats import measure as measure_module
    from opencensus.stats import stats as stats_module
    from opencensus.stats import view as view_module
    from opencensus.tags import tag_map as tag_map_module

    stats = stats_module.stats
    view_manager = stats.view_manager
    stats_recorder = stats.stats_recorder

    prompt_measure = measure_module.MeasureInt("prompts",
                                               "number of prompts",
                                               "prompts")
    prompt_view = view_module.View("prompt view",
                                   "number of prompts",
                                   [],
                                   prompt_measure,
                                   aggregation_module.CountAggregation())
    view_manager.register_view(prompt_view)
    mmap = stats_recorder.new_measurement_map()
    tmap = tag_map_module.TagMap()

    # TODO: replace the all-zero GUID with your instrumentation key.
    exporter = metrics_exporter.new_metrics_exporter(
        connection_string='InstrumentationKey=00000000-0000-0000-0000-000000000000')

    view_manager.register_exporter(exporter)

    def prompt():
        input("Press enter.")
        mmap.measure_int_put(prompt_measure, 1)
        mmap.record(tmap)
        metrics = list(mmap.measure_to_view_map.get_metrics(datetime.utcnow()))
        print(metrics[0].time_series[0].points[0])

    def main():
        while True:
            prompt()

    if __name__ == "__main__":
        main()
    ```

1. L’exportateur envoie les données de métrique à Azure Monitor à intervalle fixe. La valeur par défaut est toutes les 15 secondes. Nous effectuons le suivi d’une seule métrique, de sorte que les données de cette métrique, quels que soient la valeur et l’horodatage qu’elles contiennent, sont envoyées à chaque intervalle. Vous pouvez trouver les données sous `customMetrics`.

#### <a name="performance-counters"></a>Compteurs de performance

Par défaut, l’exportateur de métriques envoie un ensemble de compteurs de performances à Azure Monitor. Vous pouvez désactiver ce paramètre en définissant l’indicateur `enable_standard_metrics` sur `False` dans le constructeur de l’exportateur de métriques.

```python
...
exporter = metrics_exporter.new_metrics_exporter(
  enable_standard_metrics=False,
  connection_string='InstrumentationKey=<your-instrumentation-key-here>')
...
```

Les compteurs de performances actuellement envoyés sont les suivants :

- Mémoire disponible (en octets)
- Temps processeur UC (en pourcentage)
- Taux de requêtes entrantes (par seconde)
- Durée moyenne d’exécution de la requête entrante (en millisecondes)
- Utilisation du processeur UC (en pourcentage)
- Octets privés de processus (en octets)

Vous devriez être en mesure de voir ces métriques dans `performanceCounters`. Pour plus d’informations, consultez [compteurs de performances](./performance-counters.md).

#### <a name="modify-telemetry"></a>Modifier la télémétrie

Pour savoir comment modifier les données de télémétrie suivies avant leur envoi à Azure Monitor, consultez [Processeurs de télémétrie OpenCensus Python](./api-filtering-sampling.md#opencensus-python-telemetry-processors).

### <a name="tracing"></a>Traçage

> [!NOTE]
> Dans OpenCensus, `tracing` fait référence au [suivi distribué](./distributed-tracing.md). `AzureExporter` envoie de la télémétrie pour `requests` et `dependency` à Azure Monitor.

1. Commençons par générer des données de trace en local. Dans IDLE de Python ou dans l’éditeur de votre choix, saisissez le code suivant :

    ```python
    from opencensus.trace.samplers import ProbabilitySampler
    from opencensus.trace.tracer import Tracer

    tracer = Tracer(sampler=ProbabilitySampler(1.0))

    def valuePrompt():
        with tracer.span(name="test") as span:
            line = input("Enter a value: ")
            print(line)

    def main():
        while True:
            valuePrompt()

    if __name__ == "__main__":
        main()
    ```

1. L’exécution répétée du code vous invite à entrer une valeur. À chaque entrée, la valeur est imprimée dans l’interpréteur de commandes. Le module OpenCensus Python génère un élément correspondant de `SpanData`. Le projet OpenCensus définit une [trace comme une arborescence d’étendues](https://opencensus.io/core-concepts/tracing/).
    
    ```output
    Enter a value: 4
    4
    [SpanData(name='test', context=SpanContext(trace_id=8aa41bc469f1a705aed1bdb20c342603, span_id=None, trace_options=TraceOptions(enabled=True), tracestate=None), span_id='15ac5123ac1f6847', parent_span_id=None, attributes=BoundedDict({}, maxlen=32), start_time='2019-06-27T18:21:22.805429Z', end_time='2019-06-27T18:21:44.933405Z', child_span_count=0, stack_trace=None, annotations=BoundedList([], maxlen=32), message_events=BoundedList([], maxlen=128), links=BoundedList([], maxlen=32), status=None, same_process_as_parent_span=None, span_kind=0)]
    Enter a value: 25
    25
    [SpanData(name='test', context=SpanContext(trace_id=8aa41bc469f1a705aed1bdb20c342603, span_id=None, trace_options=TraceOptions(enabled=True), tracestate=None), span_id='2e512f846ba342de', parent_span_id=None, attributes=BoundedDict({}, maxlen=32), start_time='2019-06-27T18:21:44.933405Z', end_time='2019-06-27T18:21:46.156787Z', child_span_count=0, stack_trace=None, annotations=BoundedList([], maxlen=32), message_events=BoundedList([], maxlen=128), links=BoundedList([], maxlen=32), status=None, same_process_as_parent_span=None, span_kind=0)]
    Enter a value: 100
    100
    [SpanData(name='test', context=SpanContext(trace_id=8aa41bc469f1a705aed1bdb20c342603, span_id=None, trace_options=TraceOptions(enabled=True), tracestate=None), span_id='f3f9f9ee6db4740a', parent_span_id=None, attributes=BoundedDict({}, maxlen=32), start_time='2019-06-27T18:21:46.157732Z', end_time='2019-06-27T18:21:47.269583Z', child_span_count=0, stack_trace=None, annotations=BoundedList([], maxlen=32), message_events=BoundedList([], maxlen=128), links=BoundedList([], maxlen=32), status=None, same_process_as_parent_span=None, span_kind=0)]
    ```

1. Bien qu’il soit utile d’entrer des valeurs à titre de démonstration, le but ultime est d’émettre `SpanData` vers Azure Monitor. Passez votre chaîne de connexion directement à l’exportateur. Vous pouvez aussi la spécifier dans une variable d’environnement, `APPLICATIONINSIGHTS_CONNECTION_STRING`. Modifiez le code de l’étape précédente sur la base de l’exemple de code suivant :

    ```python
    from opencensus.ext.azure.trace_exporter import AzureExporter
    from opencensus.trace.samplers import ProbabilitySampler
    from opencensus.trace.tracer import Tracer

    # TODO: replace the all-zero GUID with your instrumentation key.
    tracer = Tracer(
        exporter=AzureExporter(
            connection_string='InstrumentationKey=00000000-0000-0000-0000-000000000000'),
        sampler=ProbabilitySampler(1.0),
    )

    def valuePrompt():
        with tracer.span(name="test") as span:
            line = input("Enter a value: ")
            print(line)

    def main():
        while True:
            valuePrompt()

    if __name__ == "__main__":
        main()
    ```

1. Maintenant, lorsque vous exécutez le script Python, vous êtes à nouveau invité à entrer une valeur, mais cette valeur est la seule à être imprimée dans l’interpréteur de commandes. Le `SpanData` créé est envoyé à Azure Monitor. Vous pouvez trouver les données d’unités de travail émises sous `dependencies`. Pour plus d’informations sur les requêtes sortantes, consultez les [dépendances](./opencensus-python-dependency.md) OpenCensus Python.
Pour plus d’informations sur les requêtes entrantes, consultez les [requêtes](./opencensus-python-request.md) OpenCensus Python.

#### <a name="sampling"></a>échantillonnage

Pour plus d’informations sur l’échantillonnage dans OpenCensus, consultez [Échantillonnage dans OpenCensus](sampling.md#configuring-fixed-rate-sampling-for-opencensus-python-applications).

#### <a name="trace-correlation"></a>Corrélation de trace

Pour plus d’informations sur la corrélation de télémétrie dans vos données de trace, consultez la [corrélation de télémétrie](./correlation.md#telemetry-correlation-in-opencensus-python) OpenCensus Python.

#### <a name="modify-telemetry"></a>Modifier la télémétrie

Pour plus d’informations sur la modification des données de télémétrie suivies avant leur envoi à Azure Monitor, consultez [Processeurs de télémétrie OpenCensus Python](./api-filtering-sampling.md#opencensus-python-telemetry-processors).

## <a name="configure-azure-monitor-exporters"></a>Configurer les exportateurs Azure Monitor

Comme indiqué, les exportateurs Azure Monitor qui prennent en charge OpenCensus sont au nombre de trois. Chacun envoie différents types de données de télémétrie à Azure Monitor. Pour connaître les types des données de télémétrie envoyées par chaque exportateur, consultez la liste suivante.

Chaque exportateur accepte les mêmes arguments de configuration passés par les constructeurs. Chacun d’eux est détaillé ici :

- `connection_string`: chaîne de connexion utilisée pour se connecter à votre ressource Azure Monitor. Est prioritaire sur `instrumentation_key`.
- `enable_standard_metrics`: utilisé pour `AzureMetricsExporter`. Indique à l’exportateur d’envoyer automatiquement les métriques du [compteur de performances](../platform/app-insights-metrics.md#performance-counters) à Azure Monitor. La valeur par défaut est `True`.
- `export_interval`: sert à spécifier la fréquence d’exportation en secondes.
- `instrumentation_key`: clé d’instrumentation utilisée pour se connecter à votre ressource Azure Monitor.
- `logging_sampling_rate`: utilisé pour `AzureLogHandler`. Fournit un taux d’échantillonnage [0,1.0] pour l’exportation des journaux. La valeur par défaut est 1.0.
- `max_batch_size`: spécifie la taille maximale des données de télémétrie exportées simultanément.
- `proxies`: spécifie la séquence de proxys à utiliser pour l’envoi des données à Azure Monitor. Pour plus d'informations, consultez [proxies](https://requests.readthedocs.io/en/master/user/advanced/#proxies).
- `storage_path`: chemin d’accès au dossier de stockage local (données de télémétrie non envoyées). À partir de `opencensus-ext-azure` v1.0.3, le chemin par défaut est le répertoire temp du système d’exploitation + `opencensus-python` + `your-ikey`. Avant v1.0.3, le chemin par défaut est $USER + `.opencensus` + `.azure` + `python-file-name`.

## <a name="view-your-data-with-queries"></a>Voir vos données avec des requêtes

Vous pouvez voir les données de télémétrie envoyées à partir de votre application via l’onglet **Journaux (Analytics)** .

![Capture d’écran du volet de vue d’ensemble, où « Journaux (Analytics) » est sélectionné dans un encadré rouge](./media/opencensus-python/0010-logs-query.png)

Dans la liste sous **Activé** :

- Pour les données de télémétrie envoyées avec l’exportateur de traces Azure Monitor, les requêtes entrantes s’affichent sous `requests`. Les requêtes sortantes ou In-process s’affichent sous `dependencies`.
- Pour la télémétrie envoyée avec l’exportateur de métriques Azure Monitor, les métriques envoyées s’affichent sous `customMetrics`.
- Pour la télémétrie envoyée avec l’exportateur de journaux Azure Monitor, les journaux envoyés s’affichent sous `traces`. Les exceptions s’affichent sous `exceptions`.

Pour obtenir des informations plus détaillées sur l’utilisation des requêtes et des journaux, consultez [Journaux dans Azure Monitor](../platform/data-platform-logs.md).

## <a name="learn-more-about-opencensus-for-python"></a>En savoir plus sur OpenCensus pour Python

* [OpenCensus Python sur GitHub](https://github.com/census-instrumentation/opencensus-python)
* [Personnalisation](https://github.com/census-instrumentation/opencensus-python/blob/master/README.rst#customization)
* [Exportateurs Azure Monitor sur GitHub](https://github.com/census-instrumentation/opencensus-python/tree/master/contrib/opencensus-ext-azure)
* [Intégrations OpenCensus ](https://github.com/census-instrumentation/opencensus-python#extensions)
* [Exemples d'applications Azure Monitor](https://github.com/Azure-Samples/azure-monitor-opencensus-python)

## <a name="next-steps"></a>Étapes suivantes

* [Suivi des requêtes entrantes](./opencensus-python-dependency.md)
* [Suivi des requêtes sortantes](./opencensus-python-request.md)
* [Plan de l’application](./app-map.md)
* [Analyse des performances de bout en bout](../learn/tutorial-performance.md)

### <a name="alerts"></a>Alertes

* [Tests de disponibilité](./monitor-web-app-availability.md) : créez des tests pour vous assurer que votre site est visible sur le web.
* [Diagnostics intelligents](./proactive-diagnostics.md) : ces tests s’exécutent automatiquement, sans que vous n’ayez rien à faire pour les configurer. Ils vous indiquent si votre application affiche un taux inhabituel de demandes ayant échoué.
* [Alertes de métriques](../platform/alerts-log.md) : Définissez des alertes qui vous avertissent si un indicateur de performance dépasse un seuil. Vous pouvez définir des mesures personnalisées que vous codez dans votre application.

