---
title: Suivi des requêtes entrantes dans Azure Application Insights avec OpenCensus Python | Microsoft Docs
description: Supervisez les appels de requêtes pour vos applications Python via OpenCensus Python.
ms.topic: conceptual
author: lzchen
ms.author: lechen
ms.date: 10/15/2019
ms.custom: devx-track-python
ms.openlocfilehash: 4b88550ad489607bb66eb737067190d45a466a43
ms.sourcegitcommit: 4c89d9ea4b834d1963c4818a965eaaaa288194eb
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96607073"
---
# <a name="track-incoming-requests-with-opencensus-python"></a>Suivre les requêtes entrantes avec OpenCensus Python

Les données de requêtes entrantes sont collectées à l’aide d’OpenCensus Python et de ses diverses intégrations. Suivez les données des requêtes entrantes envoyées à vos applications Web basées sur les infrastructures Web populaires `django`, `flask` et `pyramid`. Ces données sont ensuite envoyées à Application Insights dans Azure Monitor, en tant que télémétrie de `requests`.

Commencez par instrumenter votre application Python avec le dernier [kit SDK OpenCensus Python](./opencensus-python.md).

> [!NOTE]
> Cet article contient des références au terme *liste noire*, un terme que Microsoft n’utilise plus. Lorsque le terme sera supprimé du logiciel, nous le supprimerons de cet article.

## <a name="tracking-django-applications"></a>Suivi des applications Django

1. Téléchargez et installez `opencensus-ext-django` à partir de [PyPI](https://pypi.org/project/opencensus-ext-django/) et instrumentez votre application grâce à l’intergiciel `django`. Les requêtes entrantes envoyées à votre application `django` feront l’objet d’un suivi.

2. Incluez `opencensus.ext.django.middleware.OpencensusMiddleware` dans votre fichier `settings.py` sous `MIDDLEWARE`.

    ```python
    MIDDLEWARE = (
        ...
        'opencensus.ext.django.middleware.OpencensusMiddleware',
        ...
    )
    ```

3. Vérifiez qu’AzureExporter est correctement configuré dans votre fichier `settings.py` sous `OPENCENSUS`. Pour les requêtes provenant d’URL que vous ne souhaitez pas suivre, ajoutez-les à `BLACKLIST_PATHS`.

    ```python
    OPENCENSUS = {
        'TRACE': {
            'SAMPLER': 'opencensus.trace.samplers.ProbabilitySampler(rate=1)',
            'EXPORTER': '''opencensus.ext.azure.trace_exporter.AzureExporter(
                connection_string="InstrumentationKey=<your-ikey-here>"
            )''',
            'BLACKLIST_PATHS': ['https://example.com'],  <--- These sites will not be traced if a request is sent to it.
        }
    }
    ```

## <a name="tracking-flask-applications"></a>Suivi des applications Flask

1. Téléchargez et installez `opencensus-ext-flask` à partir de [PyPI](https://pypi.org/project/opencensus-ext-flask/) et instrumentez votre application grâce à l’intergiciel `flask`. Les requêtes entrantes envoyées à votre application `flask` feront l’objet d’un suivi.

    ```python
    
    from flask import Flask
    from opencensus.ext.azure.trace_exporter import AzureExporter
    from opencensus.ext.flask.flask_middleware import FlaskMiddleware
    from opencensus.trace.samplers import ProbabilitySampler
    
    app = Flask(__name__)
    middleware = FlaskMiddleware(
        app,
        exporter=AzureExporter(connection_string="InstrumentationKey=<your-ikey-here>"),
        sampler=ProbabilitySampler(rate=1.0),
    )
    
    @app.route('/')
    def hello():
        return 'Hello World!'
    
    if __name__ == '__main__':
        app.run(host='localhost', port=8080, threaded=True)
    
    ```

2. Vous pouvez aussi configurer votre application `flask` via `app.config`. Pour les requêtes provenant d’URL que vous ne souhaitez pas suivre, ajoutez-les à `BLACKLIST_PATHS`.

    ```python
    app.config['OPENCENSUS'] = {
        'TRACE': {
            'SAMPLER': 'opencensus.trace.samplers.ProbabilitySampler(rate=1.0)',
            'EXPORTER': '''opencensus.ext.azure.trace_exporter.AzureExporter(
                connection_string="InstrumentationKey=<your-ikey-here>",
            )''',
            'BLACKLIST_PATHS': ['https://example.com'],  <--- These sites will not be traced if a request is sent to it.
        }
    }
    ```

## <a name="tracking-pyramid-applications"></a>Suivi des applications Pyramid

1. Téléchargez et installez `opencensus-ext-django` à partir de [PyPI](https://pypi.org/project/opencensus-ext-pyramid/) et instrumentez votre application grâce à l’interpolation `pyramid`. Les requêtes entrantes envoyées à votre application `pyramid` feront l’objet d’un suivi.

    ```python
    def main(global_config, **settings):
        config = Configurator(settings=settings)
    
        config.add_tween('opencensus.ext.pyramid'
                         '.pyramid_middleware.OpenCensusTweenFactory')
    ```

2. Vous pouvez configurer votre interpolation `pyramid` directement dans le code. Pour les requêtes provenant d’URL que vous ne souhaitez pas suivre, ajoutez-les à `BLACKLIST_PATHS`.

    ```python
    settings = {
        'OPENCENSUS': {
            'TRACE': {
                'SAMPLER': 'opencensus.trace.samplers.ProbabilitySampler(rate=1.0)',
                'EXPORTER': '''opencensus.ext.azure.trace_exporter.AzureExporter(
                    connection_string="InstrumentationKey=<your-ikey-here>",
                )''',
                'BLACKLIST_PATHS': ['https://example.com'],  <--- These sites will not be traced if a request is sent to it.
            }
        }
    }
    config = Configurator(settings=settings)
    ```

## <a name="tracking-fastapi-applications"></a>Suivi des applications FastAPI

OpenCensus n’a pas d’extension pour FastAPI. Pour écrire votre propre intergiciel FastAPI, procédez comme suit :

1. Les dépendances suivantes sont requises : 
    - [fastapi](https://pypi.org/project/fastapi/)
    - [uvicorn](https://pypi.org/project/uvicorn/)

2. Ajoutez l’[intergiciel FastAPI](https://fastapi.tiangolo.com/tutorial/middleware/). Veillez à définir le serveur de type d’étendue : `span.span_kind = SpanKind.SERVER`.

3. Exécutez votre application. Les appels adressés à votre application FastAPI doivent être automatiquement suivis et la télémétrie doit être enregistrée directement dans Azure Monitor.

    ```python 
    # Opencensus imports
    from opencensus.ext.azure.trace_exporter import AzureExporter
    from opencensus.trace.samplers import ProbabilitySampler
    from opencensus.trace.tracer import Tracer
    from opencensus.trace.span import SpanKind
    from opencensus.trace.attributes_helper import COMMON_ATTRIBUTES
    # FastAPI imports
    from fastapi import FastAPI, Request
    # uvicorn
    import uvicorn

    app = FastAPI()

    HTTP_URL = COMMON_ATTRIBUTES['HTTP_URL']
    HTTP_STATUS_CODE = COMMON_ATTRIBUTES['HTTP_STATUS_CODE']

    # fastapi middleware for opencensus
    @app.middleware("http")
    async def middlewareOpencensus(request: Request, call_next):
        tracer = Tracer(exporter=AzureExporter(connection_string=f'InstrumentationKey={APPINSIGHTS_INSTRUMENTATIONKEY}'),sampler=ProbabilitySampler(1.0))
        with tracer.span("main") as span:
            span.span_kind = SpanKind.SERVER

            response = await call_next(request)

            tracer.add_attribute_to_current_span(
                attribute_key=HTTP_STATUS_CODE,
                attribute_value=response.status_code)
            tracer.add_attribute_to_current_span(
                attribute_key=HTTP_URL,
                attribute_value=str(request.url))

        return response

    @app.get("/")
    async def root():
        return "Hello World!"

    if __name__ == '__main__':
        uvicorn.run("example:app", host="127.0.0.1", port=5000, log_level="info")
    ```

## <a name="next-steps"></a>Étapes suivantes

* [Plan de l’application](./app-map.md)
* [Disponibilité](./monitor-web-app-availability.md)
* [action](./diagnostic-search.md)
* [Requête de journal (Analytics)](../log-query/log-query-overview.md)
* [Diagnostics de transaction](./transaction-diagnostics.md)

