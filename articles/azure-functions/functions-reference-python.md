---
title: Informations de référence pour Azure Functions à destination des développeurs Python
description: Développer des fonctions avec Python
ms.topic: article
ms.date: 11/4/2020
ms.custom: devx-track-python
ms.openlocfilehash: 8254abda68949e6884143316d4b29b07ade129dc
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96167843"
---
# <a name="azure-functions-python-developer-guide"></a>Guide des développeurs Python sur Azure Functions

Cet article est une introduction au développement d’Azure Functions avec Python. Le contenu ci-dessous suppose d’avoir lu le [Guide de développement Azure Functions](functions-reference.md).

En tant que développeur Python, vous pouvez également être intéressé par l’un des articles suivants :

| Prise en main | Concepts| Scénarios/exemples |
| -- | -- | -- | 
| <ul><li>[Fonction Python avec Visual Studio Code](./create-first-function-vs-code-csharp.md?pivots=programming-language-python)</li><li>[Fonction Python avec le terminal/l’invite de commandes](./create-first-function-cli-csharp.md?pivots=programming-language-python)</li></ul> | <ul><li>[Guide du développeur](functions-reference.md)</li><li>[Options d’hébergement](functions-scale.md)</li><li>[Considérations relatives aux&nbsp;performances](functions-best-practices.md)</li></ul> | <ul><li>[Classification d’images avec PyTorch](machine-learning-pytorch.md)</li><li>[Exemple Azure Automation](/samples/azure-samples/azure-functions-python-list-resource-groups/azure-functions-python-sample-list-resource-groups/)</li><li>[Machine learning avec TensorFlow](functions-machine-learning-tensorflow.md)</li><li>[Parcourir les exemples Python](/samples/browse/?products=azure-functions&languages=python)</li></ul> |

## <a name="programming-model"></a>Modèle de programmation

Azure Functions s’attend à ce qu’une fonction soit une méthode sans état qui traite une entrée et produit une sortie dans un script Python. Par défaut, le runtime s’attend à ce que la méthode soit implémentée en tant que méthode globale nommée `main()` dans le fichier `__init__.py`. Vous pouvez également [spécifier un autre point d’entrée](#alternate-entry-point).

Les données issues des déclencheurs et des liaisons sont liées à la fonction par des attributs de méthode avec la propriété `name` qui est définie dans le fichier *function.json*. L’exemple de fichier _function.json_ ci-dessous décrit une fonction simple déclenchée par une requête HTTP nommée `req` :

:::code language="json" source="~/functions-quickstart-templates/Functions.Templates/Templates/HttpTrigger-Python/function.json":::

Sur la base de cette définition, le fichier `__init__.py` qui contient le code de fonction peut se présenter comme dans l’exemple suivant :

```python
def main(req):
    user = req.params.get('user')
    return f'Hello, {user}!'
```

Vous pouvez aussi déclarer explicitement les types d’attributs et le type de retour dans la fonction à l’aide d’annotations de type Python. L’utilisation des fonctionnalités IntelliSense et de saisie semi-automatique fournies par de nombreux éditeurs de code Python s’en trouve facilitée.

```python
import azure.functions


def main(req: azure.functions.HttpRequest) -> str:
    user = req.params.get('user')
    return f'Hello, {user}!'
```

Utilisez les annotations Python incluses dans le package [azure.functions.*](/python/api/azure-functions/azure.functions?view=azure-python&preserve-view=true) pour lier des entrées et des sorties à vos méthodes.

## <a name="alternate-entry-point"></a>Autre point d’entrée

Vous pouvez changer le comportement par défaut d’une fonction en spécifiant éventuellement les propriétés `scriptFile` et `entryPoint` dans le fichier *function.json*. Par exemple, le fichier _function.json_ (voir ci-dessous) indique au runtime d’utiliser la méthode `customentry()` dans le fichier _main.py_ comme point d’entrée de la fonction Azure.

```json
{
  "scriptFile": "main.py",
  "entryPoint": "customentry",
  "bindings": [
      ...
  ]
}
```

## <a name="folder-structure"></a>Structure de dossiers

La structure de dossiers recommandée d’un projet Python Functions se présente comme l’exemple suivant :

```
 <project_root>/
 | - .venv/
 | - .vscode/
 | - my_first_function/
 | | - __init__.py
 | | - function.json
 | | - example.py
 | - my_second_function/
 | | - __init__.py
 | | - function.json
 | - shared_code/
 | | - __init__.py
 | | - my_first_helper_function.py
 | | - my_second_helper_function.py
 | - tests/
 | | - test_my_second_function.py
 | - .funcignore
 | - host.json
 | - local.settings.json
 | - requirements.txt
 | - Dockerfile
```
Le dossier principal du projet (<project_root>) peut contenir les fichiers suivants :

* *local.settings.json* : Utilisé pour stocker les paramètres d’application et les chaînes de connexion lors d’une exécution locale. Ce fichier n’est pas publié sur Azure. Pour en savoir plus, consultez la section [local.settings.file](functions-run-local.md#local-settings-file).
* *requirements.txt* : Contient la liste des packages Python que le système installe lors de la publication sur Azure.
* *host.json* : Contient les options de configuration globale qui affectent toutes les fonctions d’une application de fonction. Ce fichier est publié sur Azure. Toutes les options ne sont pas prises en charge lors de l’exécution locale. Pour en savoir plus, consultez la section [host.json](functions-host-json.md).
* *.vscode/*  : (Facultatif) Contient la configuration VSCode du magasin. Pour plus d’informations, consultez [Paramètre VSCode](https://code.visualstudio.com/docs/getstarted/settings).
* *.venv/*  : (Facultatif) Contient un environnement virtuel Python utilisé par le développement local.
* *Dockerfile* : (Facultatif) Utilisé lors de la publication de votre projet dans un [conteneur personnalisé](functions-create-function-linux-custom-image.md).
* *tests/*  : (Facultatif) Contient les cas de test de votre application de fonction.
* *.funcignore* : (Facultatif) Déclare des fichiers qui ne devraient pas être publiés sur Azure. En règle générale, ce fichier contient `.vscode/` pour ignorer le paramètre de votre éditeur, `.venv/` pour ignorer l’environnement virtuel Python local, `tests/` pour ignorer les cas de test et `local.settings.json` pour empêcher la publication des paramètres de l’application locale.

Chaque fonction a son propre fichier de code et son propre fichier de configuration de liaison (function.json).

Quand vous déployez votre projet dans une application de fonction sur Azure, tout le contenu du dossier principal du projet ( *<project_root>* ) doit être inclus dans le package, mais pas le dossier lui-même, ce qui signifie que `host.json` doit se trouver dans la racine du package. Nous vous recommandons de conserver vos tests dans un dossier avec d’autres fonctions, `tests/` dans cet exemple. Pour plus d’informations, consultez [Test unitaire](#unit-testing).

## <a name="import-behavior"></a>Comportement d’importation

Vous pouvez importer des modules dans votre code de fonction en utilisant des références absolues et relatives. Sur la base de la structure de dossiers présentée ci-dessus, les importations suivantes fonctionnent à partir du fichier de fonction *<project_root>\my\_first\_function\\_\_init\_\_.py* :

```python
from shared_code import my_first_helper_function #(absolute)
```

```python
import shared_code.my_second_helper_function #(absolute)
```

```python
from . import example #(relative)
```

> [!NOTE]
>  Le dossier *shared_code/* doit contenir un fichier \_\_init\_\_.py pour le marquer comme package Python lors de l’utilisation de la syntaxe d’importation absolue.

L’importation \_\_app\_\_ suivante et l’importation relative au-delà du niveau supérieur sont déconseillées, car elles ne sont pas prises en charge par le vérificateur de type statique ni par les infrastructures de test Python :

```python
from __app__.shared_code import my_first_helper_function #(deprecated __app__ import)
```

```python
from ..shared_code import my_first_helper_function #(deprecated beyond top-level relative import)
```

## <a name="triggers-and-inputs"></a>Déclencheurs et entrées

Les entrées sont réparties en deux catégories dans Azure Functions : l’entrée du déclencheur et l’entrée supplémentaire. Ces entrées sont différentes dans le fichier `function.json`, mais leur utilisation est identique dans le code Python.  Les chaînes de connexion ou les secrets pour les sources de déclencheur et d’entrée sont mappés aux valeurs dans le fichier `local.settings.json` lors d’une exécution locale, et aux paramètres d’application lors d’une exécution dans Azure.

L’exemple de code suivant illustre la différence entre les deux :

```json
// function.json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "name": "req",
      "direction": "in",
      "type": "httpTrigger",
      "authLevel": "anonymous",
      "route": "items/{id}"
    },
    {
      "name": "obj",
      "direction": "in",
      "type": "blob",
      "path": "samples/{id}",
      "connection": "AzureWebJobsStorage"
    }
  ]
}
```

```json
// local.settings.json
{
  "IsEncrypted": false,
  "Values": {
    "FUNCTIONS_WORKER_RUNTIME": "python",
    "AzureWebJobsStorage": "<azure-storage-connection-string>"
  }
}
```

```python
# __init__.py
import azure.functions as func
import logging


def main(req: func.HttpRequest,
         obj: func.InputStream):

    logging.info(f'Python HTTP triggered function processed: {obj.read()}')
```

Quand la fonction est appelée, la requête HTTP est passée à la fonction dans `req`. Une entrée est récupérée du Stockage Blob Azure sur la base de l’_ID_ présent dans l’URL de la route, puis elle est mise à disposition comme `obj` dans le corps de la fonction.  Ici, le compte de stockage spécifié est la chaîne de connexion trouvée dans le paramètre d’application AzureWebJobsStorage, qui est le même compte de stockage utilisé par l’application de fonction.


## <a name="outputs"></a>Sorties

Les sorties peuvent être exprimées aussi bien dans une valeur de retour que dans des paramètres de sortie. S’il n’y a qu’une seule sortie, nous recommandons d’utiliser la valeur de retour. Lorsqu’il y en a plusieurs, il est nécessaire d’utiliser les paramètres de sortie.

Pour utiliser la valeur de retour d’une fonction comme valeur d’une liaison de sortie, la propriété `name` de la liaison doit être définie sur `$return` dans `function.json`.

Pour produire plusieurs sorties, utilisez la méthode `set()` fournie par l’interface [`azure.functions.Out`](/python/api/azure-functions/azure.functions.out?view=azure-python&preserve-view=true) afin d’affecter une valeur à la liaison. Par exemple, la fonction suivante peut placer un message dans une file d’attente tout en renvoyant une réponse HTTP.

```json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "name": "req",
      "direction": "in",
      "type": "httpTrigger",
      "authLevel": "anonymous"
    },
    {
      "name": "msg",
      "direction": "out",
      "type": "queue",
      "queueName": "outqueue",
      "connection": "AzureWebJobsStorage"
    },
    {
      "name": "$return",
      "direction": "out",
      "type": "http"
    }
  ]
}
```

```python
import azure.functions as func


def main(req: func.HttpRequest,
         msg: func.Out[func.QueueMessage]) -> str:

    message = req.params.get('body')
    msg.set(message)
    return message
```

## <a name="logging"></a>Journalisation

L’accès à l’enregistreur d’événements du runtime d’Azure Functions se fait par l’intermédiaire du gestionnaire [`logging`](https://docs.python.org/3/library/logging.html#module-logging) racine dans l’application de fonction. Cet enregistreur, lié à Application Insights, permet de signaler les avertissements et les erreurs générés lors de l’exécution de la fonction.

L’exemple suivant enregistre un message d’informations lorsque la fonction est appelée avec un déclencheur HTTP.

```python
import logging


def main(req):
    logging.info('Python HTTP trigger function processed a request.')
```

Des méthodes d’enregistrement supplémentaires permettent d’écrire dans la console à différents niveaux de trace :

| Méthode                 | Description                                |
| ---------------------- | ------------------------------------------ |
| **`critical(_message_)`**   | Écrit un message de niveau CRITIQUE sur l’enregistreur racine.  |
| **`error(_message_)`**   | Écrit un message de niveau ERREUR sur l’enregistreur racine.    |
| **`warning(_message_)`**    | Écrit un message de niveau AVERTISSEMENT sur l’enregistreur racine.  |
| **`info(_message_)`**    | Écrit un message de niveau INFO sur l’enregistreur racine.  |
| **`debug(_message_)`** | Écrit un message de niveau DÉBOGAGE sur l’enregistreur racine.  |

Pour en savoir plus sur la journalisation, consultez [Surveiller l’exécution des fonctions Azure](functions-monitoring.md).

## <a name="http-trigger-and-bindings"></a>Déclencheur et liaisons HTTP

Le déclencheur HTTP est défini dans le fichier function.json. Le `name` de la liaison doit correspondre au paramètre nommé dans la fonction.
Dans les exemples précédents, un nom de liaison `req` est utilisé. Ce paramètre est un objet [HttpRequest], et un objet [HttpResponse] est retourné.

À partir de l’objet [HttpRequest], vous pouvez obtenir des en-têtes de demande, des paramètres de requête, des paramètres de route et le corps du message.

L’exemple suivant provient du [modèle de déclencheur HTTP pour Python](https://github.com/Azure/azure-functions-templates/tree/dev/Functions.Templates/Templates/HttpTrigger-Python).

```python
def main(req: func.HttpRequest) -> func.HttpResponse:
    headers = {"my-http-header": "some-value"}

    name = req.params.get('name')
    if not name:
        try:
            req_body = req.get_json()
        except ValueError:
            pass
        else:
            name = req_body.get('name')

    if name:
        return func.HttpResponse(f"Hello {name}!", headers=headers)
    else:
        return func.HttpResponse(
             "Please pass a name on the query string or in the request body",
             headers=headers, status_code=400
        )
```

Dans cette fonction, la valeur du paramètre de requête `name` est obtenue à partir du paramètre `params` de l’objet [HttpRequest]. Le corps du message encodé JSON est lu à l’aide de la méthode `get_json`.

De même, vous pouvez définir les `status_code` et `headers` pour le message de réponse dans l’objet [HttpResponse] retourné.

## <a name="scaling-and-performance"></a>Mise à l'échelle et niveau de performance

Il est important de bien comprendre le fonctionnement des fonctions et la façon dont les performances affectent la mise à l'échelle de l'application de fonction, en particulier lors de la conception d'applications hautement performantes. Voici quelques facteurs à prendre en compte lors de la conception, de l'écriture et de la configuration de vos applications de fonction.

### <a name="horizontal-scaling"></a>Mise à l’échelle horizontale
Par défaut, Azure Functions surveille automatiquement la charge sur votre application et crée des instances d’hôte supplémentaires pour Python, si nécessaire. Functions utilise des seuils intégrés pour différents types de déclencheurs afin de décider quand ajouter des instances, comme l'ancienneté des messages et la taille de la file d'attente pour QueueTrigger. Ces seuils ne sont pas configurables par l'utilisateur. Pour plus d’informations, consultez [Fonctionnement des plans Consommation et Premium](functions-scale.md#how-the-consumption-and-premium-plans-work).

### <a name="improving-throughput-performance"></a>Améliorer les performances de débit

Pour améliorer les performances, vous devez comprendre comment votre application utilise les ressources et être en mesure de configurer votre application de fonction en conséquence.

#### <a name="understanding-your-workload"></a>Comprendre votre charge de travail

Les configurations par défaut conviennent à la plupart des applications Azure Functions. Cependant, vous pouvez améliorer les performances de débit de vos applications en utilisant des configurations basées sur votre profil de charge de travail. La première étape consiste à identifier le type de charge de travail que vous exécutez.

| | Charge de travail liée aux E/S | Charge de travail liée au processeur |
|--| -- | -- |
|**Caractéristiques de l'application de fonction**| <ul><li>L'application doit gérer un grand nombre d'appels simultanés.</li> <li> L'application traite un grand nombre d'événements d'E/S, tels que les appels réseau et les lectures/écritures sur le disque.</li> </ul>| <ul><li>L'application effectue des calculs de longue durée, tels que le redimensionnement d'images.</li> <li>L'application procède à une transformation des données.</li> </ul> |
|**Exemples**| <ul><li>API Web</li><ul> | <ul><li>Traitement des données</li><li> Inférence par Machine Learning</li><ul>|


> [!NOTE]
>  Comme la charge de travail des fonctions réelles est souvent une combinaison de charges de travail liées aux E/S et au processeur, nous recommandons de profiler la charge de travail sur la base de charges de production réalistes.


#### <a name="performance-specific-configurations"></a>Configurations spécifiques aux performances

Après avoir identifié le profil de charge de travail de votre application de fonction, voici les configurations que vous pouvez utiliser pour améliorer les performances de débit de vos fonctions.

##### <a name="async"></a>Async

Étant donné que [Python est un runtime à thread unique](https://wiki.python.org/moin/GlobalInterpreterLock), une instance de l'hôte pour Python ne peut traiter qu'un seul appel de fonction à la fois. Pour les applications qui traitent un grand nombre d'événements d'E/S et/ou qui sont liées aux E/S, vous pouvez considérablement améliorer les performances en exécutant des fonctions de manière asynchrone.

Pour exécuter une fonction de manière asynchrone, utilisez l’instruction `async def`, qui exécute la fonction avec [asyncio](https://docs.python.org/3/library/asyncio.html) directement :

```python
async def main():
    await some_nonblocking_socket_io_op()
```
Voici un exemple de fonction avec déclencheur HTTP qui utilise le client http [aiohttp](https://pypi.org/project/aiohttp/) :

```python
import aiohttp

import azure.functions as func

async def main(req: func.HttpRequest) -> func.HttpResponse:
    async with aiohttp.ClientSession() as client:
        async with client.get("PUT_YOUR_URL_HERE") as response:
            return func.HttpResponse(await response.text())

    return func.HttpResponse(body='NotFound', status_code=404)
```


Une fonction sans le mot clé `async` est exécutée automatiquement dans un pool de threads asyncio :

```python
# Runs in an asyncio thread-pool

def main():
    some_blocking_socket_io()
```

Afin de tirer pleinement parti de l'exécution asynchrone des fonctions, l'opération/la bibliothèque d'E/S utilisée dans votre code doit également être implémentée de manière asynchrone. L'utilisation d'opérations d'E/S synchrones dans des fonctions définies comme asynchrones **peut nuire** aux performances globales.

Voici quelques exemples de bibliothèques clientes qui ont implémenté un modèle asynchrone :
- [aiohttp](https://pypi.org/project/aiohttp/) : client/serveur HTTP pour asyncio 
- [Streams API](https://docs.python.org/3/library/asyncio-stream.html) : primitives de haut niveau, asynchrones et prêtes à l'emploi destinées à fonctionner avec une connexion réseau
- [Janus Queue](https://pypi.org/project/janus/) : file d'attente thread-safe et compatible asyncio pour Python
- [pyzmq](https://pypi.org/project/pyzmq/) : liaisons Python pour ZeroMQ
 

##### <a name="use-multiple-language-worker-processes"></a>Utiliser plusieurs processus Worker de langage

Par défaut, chaque instance d’hôte Functions a un seul processus Worker de langage. Vous pouvez augmenter le nombre de processus Worker par hôte (jusqu’à 10) à l’aide du paramètre d’application [FUNCTIONS_WORKER_PROCESS_COUNT](functions-app-settings.md#functions_worker_process_count). Azure Functions essaie ensuite de distribuer uniformément les appels de fonction simultanés à ces différents Workers.

Pour les applications liées au processeur, vous devez définir le nombre de Workers de langage de manière à ce qu'il soit supérieur ou égal au nombre de cœurs disponibles par application de fonction. Pour en savoir plus, consultez [Références SKU des instances disponibles](functions-premium-plan.md#available-instance-skus). 

Les applications liées aux E/S peuvent également bénéficier de l'augmentation du nombre de processus Worker au-delà du nombre de cœurs disponibles. Gardez à l'esprit que la définition d'un nombre de Workers trop élevé peut avoir un impact sur les performances globales en raison du nombre accru de changements de contexte requis. 

FUNCTIONS_WORKER_PROCESS_COUNT s’applique à chaque hôte créé par Functions lors du scale-out de votre application pour répondre à la demande.


## <a name="context"></a>Context

Pour obtenir le contexte d’appel d’une fonction pendant l’exécution, ajoutez l’argument [`context`](/python/api/azure-functions/azure.functions.context?view=azure-python&preserve-view=true) à sa signature.

Par exemple :

```python
import azure.functions


def main(req: azure.functions.HttpRequest,
         context: azure.functions.Context) -> str:
    return f'{context.invocation_id}'
```

La classe [**Contexte**](/python/api/azure-functions/azure.functions.context?view=azure-python&preserve-view=true) contient les attributs de chaîne suivants :

`function_directory` : répertoire dans lequel s’exécute la fonction.

`function_name` : nom de la fonction.

`invocation_id` : ID de l’appel de fonction en cours.

## <a name="global-variables"></a>Variables globales

L’état de votre application n’est pas systématiquement conservé en vue des exécutions ultérieures. Toutefois, le runtime Azure Functions réutilise souvent le même processus pour plusieurs exécutions de la même application. Pour mettre en cache les résultats d’un calcul coûteux, vous devez le déclarer en tant que variable globale.

```python
CACHED_DATA = None


def main(req):
    global CACHED_DATA
    if CACHED_DATA is None:
        CACHED_DATA = load_json()

    # ... use CACHED_DATA in code
```

## <a name="environment-variables"></a>Variables d'environnement

Dans Functions, les [paramètres de l’application](functions-app-settings.md), par exemple, les chaînes de connexion de service, sont exposés en tant que variables d’environnement pendant l’exécution. Vous pouvez accéder à ces paramètres en déclarant `import os` puis en utilisant `setting = os.environ["setting-name"]`.

L’exemple suivant obtient le [paramètre d’application](functions-how-to-use-azure-function-app-settings.md#settings), avec la clé nommée `myAppSetting` :

```python
import logging
import os
import azure.functions as func

def main(req: func.HttpRequest) -> func.HttpResponse:

    # Get the setting named 'myAppSetting'
    my_app_setting_value = os.environ["myAppSetting"]
    logging.info(f'My app setting value:{my_app_setting_value}')
```

Pour le développement local, les paramètres d’application sont [conservés dans le fichier local.settings.json](functions-run-local.md#local-settings-file).

## <a name="python-version"></a>Version Python

Azure Functions prend en charge les versions de Python suivantes :

| Version de Functions | Versions<sup>*</sup> de Python |
| ----- | ----- |
| 3.x | 3.8<br/>3.7<br/>3.6 |
| 2.x | 3.7<br/>3.6 |

<sup>*</sup>Distributions CPython officielles

Pour demander une version de Python particulière lorsque vous créez votre application de fonction dans Azure, utilisez l’option `--runtime-version` de la commande [`az functionapp create`](/cli/azure/functionapp#az-functionapp-create). La version du runtime Functions est définie par l’option `--functions-version`. La version Python est définie lors de la création de l’application de fonction et ne peut pas être modifiée.

Lors d’une exécution locale, le runtime utilise la version de Python disponible.

## <a name="package-management"></a>Gestion des packages

Si vous développez en local avec Azure Functions Core Tools ou Visual Studio Code, ajoutez les noms et les versions des packages requis au fichier `requirements.txt` et installez-les avec `pip`.

Par exemple, le fichier de configuration requise suivant et la commande pip permettent d’installer le package `requests` à partir de PyPI.

```txt
requests==2.19.1
```

```bash
pip install -r requirements.txt
```

## <a name="publishing-to-azure"></a>Publier sur Azure

Avant de procéder à la publication, vérifiez que toutes les dépendances disponibles publiquement sont listées dans le fichier requirements.txt, situé à la racine de votre répertoire de projet.

Les dossiers et fichiers projet qui sont exclus de la publication, y compris le dossier d’environnement virtuel, sont listés dans le fichier .funcignore.

Les actions de build prises en charge pour la publication de votre projet Python sur Azure sont au nombre de trois : build distante, build locale et builds utilisant des dépendances personnalisées.

Vous pouvez utiliser Azure Pipelines pour générer vos dépendances et publier en livraison continue (CD). Pour plus d’informations, consultez [Livraison continue à l’aide d’Azure DevOps](functions-how-to-azure-devops.md).

### <a name="remote-build"></a>Build distante

Quand la build distante est utilisée, les dépendances restaurées sur le serveur et les dépendances natives correspondent à l’environnement de production. La taille du package de déploiement à charger s’en trouve réduite. Utilisez la build distante pour développer des applications Python sur Windows. Si votre projet a des dépendances personnalisées, vous pouvez [utiliser la build distante avec une URL d’index supplémentaire](#remote-build-with-extra-index-url).

les dépendances sont obtenues à distance en fonction du contenu du fichier requirements.txt. L’option [Build distante](functions-deployment-technologies.md#remote-build) est la méthode de génération recommandée. Par défaut, Azure Functions Core Tools demande une build distante lorsque vous utilisez la commande [func azure functionapp publish](functions-run-local.md#publish) suivante pour publier votre projet Python dans Azure.

```bash
func azure functionapp publish <APP_NAME>
```

Veillez à remplacer `<APP_NAME>` par le nom de votre application de fonction dans Azure.

L’[extension Azure Functions pour Visual Studio Code](./create-first-function-vs-code-csharp.md#publish-the-project-to-azure) demande également une build distante par défaut.

### <a name="local-build"></a>Build locale

les dépendances sont obtenues localement en fonction du contenu du fichier requirements.txt. Vous pouvez éviter d’effectuer une build distante en utilisant la commande [func azure functionapp publish](functions-run-local.md#publish) suivante pour publier avec une build locale.

```command
func azure functionapp publish <APP_NAME> --build local
```

Veillez à remplacer `<APP_NAME>` par le nom de votre application de fonction dans Azure.

Avec l’option `--build local`, les dépendances de projet sont lues à partir du fichier requirements.txt et les packages dépendants sont téléchargés et installés localement. Les fichiers de projet et les dépendances sont déployés de votre ordinateur local vers Azure, ce qui entraîne le chargement d’un package de déploiement plus important sur Azure. Si, pour une raison quelconque, les dépendances de votre fichier requirements.txt ne peuvent pas être acquises par Core Tools, vous devez utiliser l’option de dépendances personnalisées pour la publication.

Nous vous déconseillons d’utiliser des builds locales pour un développement local sur Windows.

### <a name="custom-dependencies"></a>Dépendances personnalisées

Si les dépendances de votre projet sont introuvables dans l’[index du package Python](https://pypi.org/), il existe deux façons de générer le projet. La méthode de build dépend de la façon dont vous générez le projet.

#### <a name="remote-build-with-extra-index-url"></a>Build distante avec l’URL d’index supplémentaire

Si vos packages sont disponibles à partir d’un index de package personnalisé accessible, utilisez une build distante. Avant la publication, veillez à [créer un paramètre d’application](functions-how-to-use-azure-function-app-settings.md#settings) nommé `PIP_EXTRA_INDEX_URL`. La valeur de ce paramètre est l’URL de votre index de package personnalisé. L’utilisation de ce paramètre indique à la build distante d’exécuter `pip install` en utilisant l’option `--extra-index-url`. Pour plus d’informations, consultez la [documentation Python pip install](https://pip.pypa.io/en/stable/reference/pip_install/#requirements-file-format).

Vous pouvez aussi utiliser des informations d’authentification de base avec vos URL d’index de package supplémentaires. Pour plus d’informations, consultez [Basic authentication credentials](https://pip.pypa.io/en/stable/user_guide/#basic-authentication-credentials) dans la documentation Python.

#### <a name="install-local-packages"></a>Installer des packages locaux

Si votre projet utilise des packages non disponibles publiquement pour nos outils, vous pouvez les rendre disponibles pour votre application en les plaçant dans le répertoire \_\_app\_\_/.python_packages directory. Avant d’effectuer la publication, exécutez la commande suivante pour installer les dépendances localement :

```command
pip install  --target="<PROJECT_DIR>/.python_packages/lib/site-packages"  -r requirements.txt
```

Si vous utilisez des dépendances personnalisées, vous devez utiliser l’option de publication `--no-build`, car vous avez déjà installé les dépendances dans le dossier du projet.

```command
func azure functionapp publish <APP_NAME> --no-build
```

Veillez à remplacer `<APP_NAME>` par le nom de votre application de fonction dans Azure.

## <a name="unit-testing"></a>Tests unitaires

Les fonctions écrites en Python peuvent être testées comme tout autre code Python à l’aide des frameworks de test standard. Pour la plupart des liaisons, il est possible de créer un objet d’entrée factice en créant une instance d’une classe appropriée à partir du package `azure.functions`. Comme le package [`azure.functions`](https://pypi.org/project/azure-functions/) n’est pas immédiatement disponible, assurez-vous de l’installer via votre fichier `requirements.txt`, comme décrit dans la section [gestion des packages](#package-management) ci-dessus.

Prenez *my_second_function* comme exemple. Voici un test fictif d’une fonction déclenchée par HTTP :

Tout d’abord, nous devons créer le fichier *<project_root>/my_second_function/function.json* et définir cette fonction en tant que déclencheur http.

```json
{
  "scriptFile": "__init__.py",
  "entryPoint": "main",
  "bindings": [
    {
      "authLevel": "function",
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "$return"
    }
  ]
}
```

À présent, nous pouvons implémenter *my_second_function* et *shared_code.my_second_helper_function*.

```python
# <project_root>/my_second_function/__init__.py
import azure.functions as func
import logging

# Use absolute import to resolve shared_code modules
from shared_code import my_second_helper_function

# Define an http trigger which accepts ?value=<int> query parameter
# Double the value and return the result in HttpResponse
def main(req: func.HttpRequest) -> func.HttpResponse:
    logging.info('Executing my_second_function.')

    initial_value: int = int(req.params.get('value'))
    doubled_value: int = my_second_helper_function.double(initial_value)

    return func.HttpResponse(
      body=f"{initial_value} * 2 = {doubled_value}",
      status_code=200
    )
```

```python
# <project_root>/shared_code/__init__.py
# Empty __init__.py file marks shared_code folder as a Python package
```

```python
# <project_root>/shared_code/my_second_helper_function.py

def double(value: int) -> int:
  return value * 2
```

Nous pouvons commencer à écrire des cas de test pour notre déclencheur http.

```python
# <project_root>/tests/test_my_second_function.py
import unittest

import azure.functions as func
from my_second_function import main

class TestFunction(unittest.TestCase):
    def test_my_second_function(self):
        # Construct a mock HTTP request.
        req = func.HttpRequest(
            method='GET',
            body=None,
            url='/api/my_second_function',
            params={'value': '21'})

        # Call the function.
        resp = main(req)

        # Check the output.
        self.assertEqual(
            resp.get_body(),
            b'21 * 2 = 42',
        )
```

Dans votre environnement virtuel Python `.venv`, installez votre infrastructure de test Python favorite (par exemple, `pip install pytest`). Exécutez simplement `pytest tests` pour vérifier le résultat du test.

## <a name="temporary-files"></a>Fichiers temporaires

La méthode `tempfile.gettempdir()` retourne un dossier temporaire, `/tmp` sous Linux. Votre application peut utiliser ce répertoire pour stocker les fichiers temporaires générés et utilisés par vos fonctions lors de l’exécution.

> [!IMPORTANT]
> Il n’est pas garanti que les fichiers écrits dans le répertoire temporaire persistent d’un appel à l’autre. Lors du scale-out, les fichiers temporaires ne sont pas partagés entre les instances.

L’exemple suivant crée un fichier temporaire nommé dans le répertoire temporaire (`/tmp`) :

```python
import logging
import azure.functions as func
import tempfile
from os import listdir

#---
   tempFilePath = tempfile.gettempdir()
   fp = tempfile.NamedTemporaryFile()
   fp.write(b'Hello world!')
   filesDirListInTemp = listdir(tempFilePath)
```

Nous vous recommandons de conserver vos tests dans un dossier distinct du dossier de projet. Cela vous évite de déployer du code de test avec votre application.

## <a name="preinstalled-libraries"></a>Bibliothèques préinstallées

Le runtime Functions Python s’accompagne de quelques bibliothèques.

### <a name="python-standard-library"></a>Bibliothèque standard Python

La bibliothèque standard Python contient une liste de modules Python intégrés qui sont fournis avec chaque distribution Python. La plupart de ces bibliothèques vous permettent d’accéder à des fonctionnalités système, comme l’E/S de fichiers. Sur les systèmes Windows, ces bibliothèques sont installées avec Python. Sur les systèmes basés sur Unix, ils sont fournis par des collections de packages.

Pour consulter la liste de ces bibliothèques dans les détails, accédez aux liens ci-dessous :

* [Bibliothèque standard Python 3.6](https://docs.python.org/3.6/library/)
* [Bibliothèque standard Python 3.7](https://docs.python.org/3.7/library/)
* [Bibliothèque standard Python 3.8](https://docs.python.org/3.8/library/)

### <a name="azure-functions-python-worker-dependencies"></a>Dépendances du Worker Python Azure Functions

Le Worker Python Functions a besoin d’un ensemble spécifique de bibliothèques. Vous pouvez aussi utiliser ces bibliothèques dans vos fonctions, mais elles ne font pas partie intégrante du standard Python. Si vos fonctions reposent sur une de ces bibliothèques, il se peut que votre code n’y ait pas accès quand il s’exécute en dehors d’Azure Functions. Vous trouverez une liste détaillée des dépendances dans la section **install\_requires** du fichier [setup.py](https://github.com/Azure/azure-functions-python-worker/blob/dev/setup.py#L282).

> [!NOTE]
> Si le fichier requirements.txt de votre application de fonction contient une entrée `azure-functions-worker`, supprimez-la. Le worker functions est géré automatiquement par la Plateforme Azure Functions et nous le mettons régulièrement à jour avec de nouvelles fonctionnalités et de correctifs de bogues. L’installation manuelle d’une ancienne version du worker dans requirements.txt peut entraîner des problèmes inattendus.

### <a name="azure-functions-python-library"></a>Bibliothèque Python Azure Functions

Chaque mise à jour du Worker Python comprend une nouvelle version de la [bibliothèque Python Azure Functions (azure.functions)](https://github.com/Azure/azure-functions-python-library). Cette approche facilite la mise à jour continue de vos applications de fonction Python, car chaque mise à jour offre une compatibilité descendante. La liste des versions de cette bibliothèque se trouve dans [azure-functions PyPi](https://pypi.org/project/azure-functions/#history).

La version de la bibliothèque runtime est corrigée par Azure et ne peut pas être remplacée par requirements.txt. L’entrée `azure-functions` dans requirements.txt est réservée au linting et à la sensibilisation des clients.

Utilisez le code suivant pour effectuer le suivi de la version effective de la bibliothèque Functions Python dans votre runtime :

```python
getattr(azure.functions, '__version__', '< 1.2.1')
```

### <a name="runtime-system-libraries"></a>Bibliothèques système runtime

Pour obtenir la liste des bibliothèques système préinstallées dans les images Docker du Worker Python, suivez les liens ci-dessous :

|  Runtime Functions  | Version de Debian | Versions de Python |
|------------|------------|------------|
| Version 2.x | Stretch  | [Python 3.6](https://github.com/Azure/azure-functions-docker/blob/master/host/2.0/stretch/amd64/python/python36/python36.Dockerfile)<br/>[Python 3.7](https://github.com/Azure/azure-functions-docker/blob/master/host/2.0/stretch/amd64/python/python37/python37.Dockerfile) |
| Version 3.x | Buster | [Python 3.6](https://github.com/Azure/azure-functions-docker/blob/master/host/3.0/buster/amd64/python/python36/python36.Dockerfile)<br/>[Python 3.7](https://github.com/Azure/azure-functions-docker/blob/master/host/3.0/buster/amd64/python/python37/python37.Dockerfile)<br />[Python 3.8](https://github.com/Azure/azure-functions-docker/blob/master/host/3.0/buster/amd64/python/python38/python38.Dockerfile) |

## <a name="cross-origin-resource-sharing"></a>Partage de ressources cross-origin

[!INCLUDE [functions-cors](../../includes/functions-cors.md)]

CORS est entièrement pris en charge pour les applications de fonction Python.

## <a name="known-issues-and-faq"></a>Problèmes connus et FAQ

Voici la liste des guides de résolution des problèmes courants :

* [ModuleNotFoundError et ImportError](recover-python-functions.md#troubleshoot-modulenotfounderror)
* [Impossible d’importer « cygrpc »](recover-python-functions.md#troubleshoot-cannot-import-cygrpc)

Tous les problèmes connus et les demandes de fonctionnalités sont listés dans [Problèmes GitHub](https://github.com/Azure/azure-functions-python-worker/issues). Si vous rencontrez un problème qui n’apparaît pas dans GitHub, soumettez un nouveau problème en incluant une description détaillée.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez les ressources suivantes :

* [Documentation sur l’API du package Azure Functions](/python/api/azure-functions/azure.functions?view=azure-python&preserve-view=true)
* [Meilleures pratiques pour Azure Functions](functions-best-practices.md)
* [Azure Functions triggers and bindings (Déclencheurs et liaisons Azure Functions)](functions-triggers-bindings.md)
* [Liaisons de Stockage Blob](functions-bindings-storage-blob.md)
* [Liaisons HTTP et Webhook](functions-bindings-http-webhook.md)
* [Liaisons de Stockage File d’attente](functions-bindings-storage-queue.md)
* [Déclencheur de minuteur](functions-bindings-timer.md)

[Vous rencontrez des problèmes ? Faites-le nous savoir.](https://aka.ms/python-functions-ref-survey)


[HttpRequest]: /python/api/azure-functions/azure.functions.httprequest?view=azure-python&preserve-view=true
[HttpResponse]: /python/api/azure-functions/azure.functions.httpresponse?view=azure-python&preserve-view=true