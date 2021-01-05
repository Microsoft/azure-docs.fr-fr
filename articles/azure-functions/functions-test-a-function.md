---
title: Test des fonctions Azure
description: Créer des tests automatisés pour une fonction C# dans Visual Studio et une fonction JavaScript dans VS Code
author: craigshoemaker
ms.topic: conceptual
ms.custom: devx-track-csharp, devx-track-js
ms.date: 03/25/2019
ms.author: cshoe
ms.openlocfilehash: 2488a476fe40c2bf1f3e290b462babceff30a9b0
ms.sourcegitcommit: c4246c2b986c6f53b20b94d4e75ccc49ec768a9a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96601388"
---
# <a name="strategies-for-testing-your-code-in-azure-functions"></a>Stratégies permettant de tester votre code dans Azure Functions

Cet article explique comment créer des tests automatisés pour Azure Functions.

Tester l'intégralité du code est recommandé. Cela étant, vous pouvez obtenir de meilleurs résultats en compilant une logique de fonction et en créant des tests en dehors de cette fonction. Faire abstraction d'une logique limite les lignes de code de la fonction et permet uniquement à cette dernière d'appeler d'autres classes ou modules. Cet article, cependant, explique comment créer des tests automatisés pour des fonctions HTTP et déclenchées par un minuteur.

Le contenu suivant est divisé en deux sections distinctes destinées à cibler différents environnements et langages. Découvrez comment créer des tests dans :

- [C# dans Visual Studio avec xUnit](#c-in-visual-studio)
- [JavaScript dans VS Code avec Jest](#javascript-in-vs-code)

L’exemple de dépôt est disponible sur [GitHub](https://github.com/Azure-Samples/azure-functions-tests).

## <a name="c-in-visual-studio"></a>C# dans Visual Studio

L’exemple suivant explique comment créer une application de fonction C# dans Visual Studio et exécuter des tests avec [xUnit](https://github.com/xunit/xunit).

![Tester Azure Functions avec C# dans Visual Studio](./media/functions-test-a-function/azure-functions-test-visual-studio-xunit.png)

### <a name="setup"></a>Programme d’installation

Pour configurer votre environnement, créez une fonction et testez l’application. Les étapes suivantes vous permettent de créer les applications et les fonctions requises pour prendre en charge les tests :

1. [Créez une nouvelle application Functions](./functions-create-first-azure-function.md) et nommez-la **Functions**.
2. [Créez une fonction HTTP à partir du modèle](./functions-create-first-azure-function.md) et nommez-la **MyHttpTrigger**.
3. [Créez une fonction de minuteur à partir du modèle](./functions-create-scheduled-function.md) et nommez-la **MyTimerTrigger**.
4. [Créez une application de test xUnit](https://xunit.net/docs/getting-started/netcore/cmdline) dans la solution et nommez-la **Functions.Tests**.
5. Utilisez NuGet pour ajouter des références à partir de l’application de test à [Microsoft.AspNetCore.Mvc](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc/)
6. [Faites référence à l’application *Functions*](/visualstudio/ide/managing-references-in-a-project?view=vs-2017) à partir de l’application *Functions.Tests*.

### <a name="create-test-classes"></a>Créer des classes de test

Une fois les projets créés, vous pouvez créer les classes servant à exécuter les tests automatisés.

Chaque fonction utilise une instance [ILogger](/dotnet/api/microsoft.extensions.logging.ilogger) pour gérer la journalisation des messages. Certains tests ne consignent pas les messages ou ne se soucient pas de la manière dont la journalisation est implémentée. D'autres tests doivent évaluer les messages consignés afin de déterminer si un test a abouti.

Vous allez créer une nouvelle classe nommée `ListLogger` qui contient une liste interne de messages à évaluer pendant un test. Pour implémenter l’interface `ILogger` requise, la classe a besoin d’une portée. La classe suivante imite une portée pour les cas de test à passer à la classe `ListLogger`.

Créez une classe nommée **NullScope.cs** dans le projet *Functions.Tests* et entrez le code suivant :

```csharp
using System;

namespace Functions.Tests
{
    public class NullScope : IDisposable
    {
        public static NullScope Instance { get; } = new NullScope();

        private NullScope() { }

        public void Dispose() { }
    }
}
```

Maintenant, créez une classe nommée **ListLogger.cs** dans le projet *Functions.Tests* et entrez le code suivant :

```csharp
using Microsoft.Extensions.Logging;
using System;
using System.Collections.Generic;
using System.Text;

namespace Functions.Tests
{
    public class ListLogger : ILogger
    {
        public IList<string> Logs;

        public IDisposable BeginScope<TState>(TState state) => NullScope.Instance;

        public bool IsEnabled(LogLevel logLevel) => false;

        public ListLogger()
        {
            this.Logs = new List<string>();
        }

        public void Log<TState>(LogLevel logLevel,
                                EventId eventId,
                                TState state,
                                Exception exception,
                                Func<TState, Exception, string> formatter)
        {
            string message = formatter(state, exception);
            this.Logs.Add(message);
        }
    }
}
```

La classe `ListLogger` implémente les membres suivants comme contractés par l'interface `ILogger` :

- **BeginScope** : Les étendues ajoutent du contexte à votre journalisation. Dans ce cas, le test pointe vers l’instance statique de la classe `NullScope` pour permettre le bon fonctionnement du test.

- **IsEnabled** : La valeur par défaut `false` est indiquée.

- **Log** : Cette méthode utilise la fonction `formatter` fournie pour mettre en forme le message, puis ajoute le texte qui en résulte à la collection `Logs`.

La collection `Logs` est une instance `List<string>` initialisée dans le constructeur.

Maintenant, créez un fichier nommé **LoggerTypes.cs** dans le projet *Functions.Tests* et entrez le code suivant :

```csharp
namespace Functions.Tests
{
    public enum LoggerTypes
    {
        Null,
        List
    }
}
```

Cette énumération spécifie le type d'enregistreur d'événements utilisé par les tests.

Maintenant, créez une classe nommée **TestFactory.cs** dans le projet *Functions.Tests* et entrez le code suivant :

```csharp
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Http.Internal;
using Microsoft.Extensions.Logging;
using Microsoft.Extensions.Logging.Abstractions;
using Microsoft.Extensions.Primitives;
using System.Collections.Generic;

namespace Functions.Tests
{
    public class TestFactory
    {
        public static IEnumerable<object[]> Data()
        {
            return new List<object[]>
            {
                new object[] { "name", "Bill" },
                new object[] { "name", "Paul" },
                new object[] { "name", "Steve" }

            };
        }

        private static Dictionary<string, StringValues> CreateDictionary(string key, string value)
        {
            var qs = new Dictionary<string, StringValues>
            {
                { key, value }
            };
            return qs;
        }

        public static HttpRequest CreateHttpRequest(string queryStringKey, string queryStringValue)
        {
            var context = new DefaultHttpContext();
            var request = context.Request;
            request.Query = new QueryCollection(CreateDictionary(queryStringKey, queryStringValue));
            return request;
        }

        public static ILogger CreateLogger(LoggerTypes type = LoggerTypes.Null)
        {
            ILogger logger;

            if (type == LoggerTypes.List)
            {
                logger = new ListLogger();
            }
            else
            {
                logger = NullLoggerFactory.Instance.CreateLogger("Null Logger");
            }

            return logger;
        }
    }
}
```

La classe `TestFactory` implémente les membres suivants :

- **Données** : Cette propriété renvoie une collection [IEnumerable](/dotnet/api/system.collections.ienumerable) d’exemples de données. Les paires clé/valeur représentent les valeurs transmises dans une chaîne de requête.

- **CreateDictionary** : Cette méthode accepte une paire clé/valeur en tant qu’arguments et renvoie une nouvelle `Dictionary` utilisée pour créer `QueryCollection` afin de représenter les valeurs de chaîne de requête.

- **CreateHttpRequest** : Cette méthode crée une requête HTTP initialisée avec les paramètres d'une chaîne de requête donnée.

- **CreateLogger** : En fonction du type d'enregistreur d’événements, cette méthode renvoie une classe d’enregistreur d’événements utilisée à des fins de test. Le `ListLogger` effectue le suivi des messages consignés disponibles à des fins d'évaluation lors des tests.

Enfin, créez une classe nommée **FunctionsTests.cs** dans le projet *Functions.Tests* et entrez le code suivant :

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Logging;
using Xunit;

namespace Functions.Tests
{
    public class FunctionsTests
    {
        private readonly ILogger logger = TestFactory.CreateLogger();

        [Fact]
        public async void Http_trigger_should_return_known_string()
        {
            var request = TestFactory.CreateHttpRequest("name", "Bill");
            var response = (OkObjectResult)await MyHttpTrigger.Run(request, logger);
            Assert.Equal("Hello, Bill. This HTTP triggered function executed successfully.", response.Value);
        }

        [Theory]
        [MemberData(nameof(TestFactory.Data), MemberType = typeof(TestFactory))]
        public async void Http_trigger_should_return_known_string_from_member_data(string queryStringKey, string queryStringValue)
        {
            var request = TestFactory.CreateHttpRequest(queryStringKey, queryStringValue);
            var response = (OkObjectResult)await MyHttpTrigger.Run(request, logger);
            Assert.Equal($"Hello, {queryStringValue}. This HTTP triggered function executed successfully.", response.Value);
        }

        [Fact]
        public void Timer_should_log_message()
        {
            var logger = (ListLogger)TestFactory.CreateLogger(LoggerTypes.List);
            MyTimerTrigger.Run(null, logger);
            var msg = logger.Logs[0];
            Assert.Contains("C# Timer trigger function executed at", msg);
        }
    }
}
```

Les membres implémentés dans cette classe sont :

- **Http_trigger_should_return_known_string** : Ce test crée une requête avec les valeurs de chaîne de requête de `name=Bill` vers une fonction HTTP et vérifie que la réponse attendue est renvoyée.

- **Http_trigger_should_return_string_from_member_data** : Ce test utilise les attributs xUnit pour fournir des exemples de données à la fonction HTTP.

- **Timer_should_log_message** : Ce test crée une instance de `ListLogger` et la transmet à des fonctions de minuteur. Une fois la fonction exécutée, le journal est vérifié pour veiller à ce que le message attendu soit présent.

Si vous souhaitez accéder aux paramètres de l’application dans vos tests, vous pouvez [injecter](./functions-dotnet-dependency-injection.md) une instance de `IConfiguration` avec des valeurs de variables d’environnement factices dans votre fonction.

### <a name="run-tests"></a>Exécuter les tests

Pour exécuter les tests, accédez à l'**Explorateur de tests** et cliquez sur **Exécuter tout**.

![Tester Azure Functions avec C# dans Visual Studio](./media/functions-test-a-function/azure-functions-test-visual-studio-xunit.png)

### <a name="debug-tests"></a>Déboguer les tests

Pour déboguer les tests, définissez un point d’arrêt sur un test, accédez à l'**Explorateur de tests** et cliquez sur **Exécuter > Déboguer la dernière exécution**.

## <a name="javascript-in-vs-code"></a>JavaScript dans VS Code

L’exemple suivant explique comment créer une application de fonction JavaScript dans VS Code, puis exécuter les tests avec [Jest](https://jestjs.io). Cette procédure utilise l'[extension Fonctions VS Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) pour créer Azure Functions.

![Tester Azure Functions avec JavaScript dans VS Code](./media/functions-test-a-function/azure-functions-test-vs-code-jest.png)

### <a name="setup"></a>Programme d’installation

Pour configurer votre environnement, initialisez une application Node.js dans un dossier vide en exécutant `npm init`.

```bash
npm init -y
```

Installez ensuite Jest en exécutant la commande suivante :

```bash
npm i jest
```

À présent, mettez à jour _package.json_ pour remplacer la commande de test existante par la commande suivante :

```bash
"scripts": {
    "test": "jest"
}
```

### <a name="create-test-modules"></a>Créer des modules de test

Une fois le projet initialisé, vous pouvez créer les modules utilisés pour exécuter les tests automatisés. Commencez par créer un nouveau dossier nommé *testing* pour contenir les modules de prise en charge.

Dans le dossier *testing*, ajoutez un nouveau fichier, nommez-le **defaultContext.js**, puis ajoutez le code suivant :

```javascript
module.exports = {
    log: jest.fn()
};
```

Ce module simule la fonction *log* pour représenter le contexte d’exécution par défaut.

Ensuite, ajoutez un nouveau fichier, nommez-le **defaultTimer.js** et ajoutez le code suivant :

```javascript
module.exports = {
    IsPastDue: false
};
```

Ce module implémente la propriété `IsPastDue` en tant qu'instance fictive du minuteur. Des configurations de minuteur telles que des expressions NCRONTAB ne sont pas requises ici, car l’atelier de test appelle simplement la fonction directement pour tester le résultat.

Ensuite, utilisez l’extension VS Code Functions pour [créer une fonction HTTP JavaScript](/azure/developer/javascript/tutorial-vscode-serverless-node-01) et nommez-la *HttpTrigger*. Une fois la fonction créée, ajoutez un nouveau fichier au même dossier nommé **index.test.js** et ajoutez le code suivant :

```javascript
const httpFunction = require('./index');
const context = require('../testing/defaultContext')

test('Http trigger should return known text', async () => {

    const request = {
        query: { name: 'Bill' }
    };

    await httpFunction(context, request);

    expect(context.log.mock.calls.length).toBe(1);
    expect(context.res.body).toEqual('Hello Bill');
});
```

La fonction HTTP du modèle renvoie une chaîne « Hello » concaténée avec le nom fourni dans la chaîne de requête. Ce test crée une instance fictive de requête et la transmet à la fonction HTTP. Le test vérifie que la méthode *log* est appelée et que le texte renvoyé indique « Hello Bill ».

Ensuite, utilisez l’extension VS Code Functions pour créer une fonction de minuteur JavaScript et nommez-la *TimerTrigger*. Une fois la fonction créée, ajoutez un nouveau fichier au même dossier nommé **index.test.js** et ajoutez le code suivant :

```javascript
const timerFunction = require('./index');
const context = require('../testing/defaultContext');
const timer = require('../testing/defaultTimer');

test('Timer trigger should log message', () => {
    timerFunction(context, timer);
    expect(context.log.mock.calls.length).toBe(1);
});
```

La fonction de minuteur du modèle consigne un message à la fin du corps de la fonction. Ce test permet de s'assurer que la fonction *log* est appelée.

### <a name="run-tests"></a>Exécuter les tests

Pour exécuter les tests, appuyez sur **CTRL + ~** afin d'ouvrir la fenêtre de commande, puis exécutez `npm test` :

```bash
npm test
```

![Tester Azure Functions avec JavaScript dans VS Code](./media/functions-test-a-function/azure-functions-test-vs-code-jest.png)

### <a name="debug-tests"></a>Déboguer les tests

Pour déboguer vos tests, ajoutez la configuration suivante à votre fichier *launch.json* :

```json
{
  "type": "node",
  "request": "launch",
  "name": "Jest Tests",
  "disableOptimisticBPs": true,
  "program": "${workspaceRoot}/node_modules/jest/bin/jest.js",
  "args": [
      "-i"
  ],
  "internalConsoleOptions": "openOnSessionStart"
}
```

Ensuite, définissez un point d’arrêt dans votre test et appuyez sur **F5**.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous savez écrire des tests automatisés pour vos fonctions, penchez-vous sur les ressources suivantes :

- [Exécuter manuellement une fonction non déclenchée via HTTP](./functions-manually-run-non-http.md)
- [Gestion des erreurs Azure Functions](./functions-bindings-error-pages.md)
- [Débogage local lors du déclenchement de fonctions Azure Event Grid](./functions-debug-event-grid-trigger-local.md)
