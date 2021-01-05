---
title: Gestionnaires personnalisés Azure Functions
description: Apprenez à utiliser Azure Functions avec n’importe quel langage ou version de runtime.
author: anthonychu
ms.author: antchu
ms.date: 12/1/2020
ms.topic: article
ms.openlocfilehash: 099f90ba8c5d9dabb6c4c505e50d8c077e3eaf0f
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96746027"
---
# <a name="azure-functions-custom-handlers"></a>Gestionnaires personnalisés Azure Functions

Chaque application Functions est exécutée par un gestionnaire propre au langage. Même si Azure Functions prend en charge un grand nombre de [gestionnaires de langage](./supported-languages.md) par défaut, dans certains cas, vous souhaiterez peut-être utiliser d’autres langages ou runtimes.

Les gestionnaires personnalisés sont des serveurs web légers qui reçoivent des événements de l’hôte Functions. Tout langage qui prend en charge les primitives HTTP peut implémenter un gestionnaire personnalisé.

Les gestionnaires personnalisés sont particulièrement adaptés dans les cas où vous voulez :

- Implémenter une application de fonction dans un langage qui n’est pas actuellement pris en charge, comme Go ou Rust.
- Implémenter une application de fonction dans un langage qui n’est pas actuellement pris en charge, comme Deno.

Avec les gestionnaires personnalisés, vous pouvez utiliser des [déclencheurs et des liaisons d’entrée et de sortie](./functions-triggers-bindings.md) via des [bundles d’extension](./functions-bindings-register.md).

Démarrez avec les gestionnaires personnalisés Azure Functions avec [les démarrages rapides en Go et Rust](create-first-function-vs-code-other.md).

## <a name="overview"></a>Vue d’ensemble

Le diagramme suivant illustre la relation entre l’hôte Functions et un serveur web implémenté en tant que gestionnaire personnalisé.

![Vue d’ensemble d’un gestionnaire personnalisé Azure Functions](./media/functions-custom-handlers/azure-functions-custom-handlers-overview.png)

1. Chaque événement déclenche une demande qui est envoyée à l’hôte Functions. Un événement est un déclencheur pris en charge par Azure Functions.
1. L’hôte Functions émet ensuite une [charge utile de demande](#request-payload) à destination du serveur web. La charge utile contient des données de déclencheur et de liaison d’entrée ainsi que d’autres métadonnées pour la fonction.
1. Le serveur web exécute la fonction individuelle, puis retourne une [charge utile de réponse](#response-payload) à l’hôte Functions.
1. L’hôte Functions transmet les données de la réponse aux liaisons de sortie de la fonction pour traitement.

Une application Azure Functions implémentée en tant que gestionnaire personnalisé doit configurer les fichiers *host.json*, *local.settings.json* et *function.json* en respectant quelques conventions.

## <a name="application-structure"></a>Structure d'application

Pour implémenter un gestionnaire personnalisé, votre application a besoin des éléments suivants :

- Un fichier *host.json* à la racine de l’application
- Un fichier *local.settings.json* à la racine de votre application
- Un fichier *function.json* pour chaque fonction (à l’intérieur d’un dossier dont le nom correspond à celui de la fonction)
- Une commande, un script ou un exécutable qui exécute un serveur web

Le schéma suivant montre comment se présentent ces fichiers dans le système de fichiers pour une fonction nommée « MyQueueFunction » et un exécutable de gestionnaire personnalisé nommé *handler. exe*.

```bash
| /MyQueueFunction
|   function.json
|
| host.json
| local.settings.json
| handler.exe
```

### <a name="configuration"></a>Configuration

L’application est configurée via les fichiers *host.json* et *local.settings.json*.

#### <a name="hostjson"></a>host.json

Le fichier *host.json* indique à l’hôte Functions où envoyer les demandes en pointant vers un serveur web capable de traiter les événements HTTP.

Un gestionnaire personnalisé est configuré dans le fichier *host.json* avec des détails sur la façon d’exécuter le serveur web dans la section `customHandler`.

```json
{
  "version": "2.0",
  "customHandler": {
    "description": {
      "defaultExecutablePath": "handler.exe"
    }
  }
}
```

La section `customHandler` pointe vers une cible définie par `defaultExecutablePath`. La cible d’exécution peut être une commande, un exécutable ou un fichier où le serveur web est implémenté.

Utilisez le tableau `arguments` pour transmettre des arguments à l’exécutable. Les arguments prennent en charge l’extension des variables d’environnement (paramètres d’application) avec la notation `%%`.

Vous pouvez aussi remplacer le répertoire de travail utilisé par le fichier exécutable avec `workingDirectory`.

```json
{
  "version": "2.0",
  "customHandler": {
    "description": {
      "defaultExecutablePath": "app/handler.exe",
      "arguments": [
        "--database-connection-string",
        "%DATABASE_CONNECTION_STRING%"
      ],
      "workingDirectory": "app"
    }
  }
}
```

##### <a name="bindings-support"></a>Prise en charge des liaisons

Vous pouvez rendre disponibles des déclencheurs standard, de même que des liaisons d’entrée et de sortie, en référençant des [bundles d’extension](./functions-bindings-register.md) dans le fichier *host.json*.

#### <a name="localsettingsjson"></a>local.settings.json

*local.settings.json* définit les paramètres d’application utilisés pendant l’exécution locale de l’application de fonction. Comme le fichier *local.settings.json* peut contenir des secrets, il doit être exclu du contrôle de code source. Dans Azure, utilisez plutôt les paramètres d’application.

Pour les gestionnaires personnalisés, définissez `FUNCTIONS_WORKER_RUNTIME` sur `Custom` dans *local.settings.jso*.

```json
{
  "IsEncrypted": false,
  "Values": {
    "FUNCTIONS_WORKER_RUNTIME": "Custom"
  }
}
```

> [!NOTE]
> `Custom` peut ne pas être reconnu comme un runtime valide dans les plans App Service ou Linux Premium. S’il s’agit de votre cible de déploiement, définissez `FUNCTIONS_WORKER_RUNTIME` sur une chaîne vide.

### <a name="function-metadata"></a>Métadonnées de fonction

Quand il est utilisé avec un gestionnaire personnalisé, le contenu de *function.json* ne se définit pas différemment qu’une fonction dans un autre contexte. La seule exigence est que les fichiers *function.json* doivent se trouver dans un dossier dont le nom correspond à celui de la fonction.

Le fichier *function.json* suivant configure une fonction qui possède un déclencheur de file d’attente et une liaison de sortie de file d’attente. Comme il se trouve dans un dossier nommé *MyQueueFunction*, il définit une fonction nommée *MyQueueFunction*.

**MyQueueFunction/function.json**

```json
{
  "bindings": [
    {
      "name": "myQueueItem",
      "type": "queueTrigger",
      "direction": "in",
      "queueName": "messages-incoming",
      "connection": "AzureWebJobsStorage"
    },
    {
      "name": "$return",
      "type": "queue",
      "direction": "out",
      "queueName": "messages-outgoing",
      "connection": "AzureWebJobsStorage"
    }
  ]
}
```

### <a name="request-payload"></a>Charge utile de demande

À réception d’un message de file d’attente, l’hôte Functions envoie une requête HTTP post au gestionnaire personnalisé avec une charge utile dans le corps.

Le code suivant représente un exemple de charge utile de demande. La charge utile comprend une structure JSON à deux membres : `Data` et `Metadata`.

Le membre `Data` comprend des clés dont le nom correspond aux noms d’entrée et de déclencheur définis dans le tableau de liaisons du fichier *function.json*.

Le membre `Metadata` comprend les [métadonnées générées à partir de la source d’événement](./functions-bindings-expressions-patterns.md#trigger-metadata).

```json
{
  "Data": {
    "myQueueItem": "{ message: \"Message sent\" }"
  },
  "Metadata": {
    "DequeueCount": 1,
    "ExpirationTime": "2019-10-16T17:58:31+00:00",
    "Id": "800ae4b3-bdd2-4c08-badd-f08e5a34b865",
    "InsertionTime": "2019-10-09T17:58:31+00:00",
    "NextVisibleTime": "2019-10-09T18:08:32+00:00",
    "PopReceipt": "AgAAAAMAAAAAAAAAAgtnj8x+1QE=",
    "sys": {
      "MethodName": "QueueTrigger",
      "UtcNow": "2019-10-09T17:58:32.2205399Z",
      "RandGuid": "24ad4c06-24ad-4e5b-8294-3da9714877e9"
    }
  }
}
```

### <a name="response-payload"></a>Charge utile de réponse

Par convention, les réponses de fonction se présentent sous la forme de paires clé/valeur. Les clés prises en charge sont les suivantes :

| <nobr>Clé de charge utile</nobr>   | Type de données | Notes                                                      |
| ------------- | --------- | ------------------------------------------------------------ |
| `Outputs`     | object    | Contient les valeurs de réponse définies par le tableau `bindings` dans le fichier *function.json*.<br /><br />Par exemple, si une fonction est configurée avec une liaison de sortie de file d’attente nommée « myQueueOutput », `Outputs` contient une clé nommée `myQueueOutput`, qui est définie par le gestionnaire personnalisé sur les messages envoyés à la file d’attente. |
| `Logs`        | tableau     | Les messages figurent dans les journaux d’invocations Functions.<br /><br />Quand l’exécution se produit dans Azure, les messages s’affichent dans Application Insights. |
| `ReturnValue` | string    | Utilisé pour fournir une réponse quand une sortie est configurée avec la valeur `$return` dans le fichier *function.json*. |

Voici un exemple de charge utile de réponse.

```json
{
  "Outputs": {
    "res": {
      "body": "Message enqueued"
    },
    "myQueueOutput": [
      "queue message 1",
      "queue message 2"
    ]
  },
  "Logs": [
    "Log message 1",
    "Log message 2"
  ],
  "ReturnValue": "{\"hello\":\"world\"}"
}
```

## <a name="examples"></a>Exemples

Les gestionnaires personnalisés peuvent être implémentés dans n’importe quel langage prenant en charge la réception d’événements HTTP. Les exemples suivants montrent comment implémenter un gestionnaire personnalisé avec le langage de programmation Go.

### <a name="function-with-bindings"></a>Fonction avec liaisons

Le scénario implémenté dans cet exemple présente une fonction nommée `order` qui accepte un `POST` avec une charge utile représentant une commande de produit. Quand une commande est transmise à la fonction, un message du service Stockage File d’attente est créé et une réponse HTTP est retournée.

<a id="bindings-implementation" name="bindings-implementation"></a>

#### <a name="implementation"></a>Implémentation

Dans un dossier nommé *order*, le fichier *function.json* configure la fonction déclenchée par HTTP.

**order/function.json**

```json
{
  "bindings": [
    {
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": ["post"]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "res"
    },
    {
      "type": "queue",
      "name": "message",
      "direction": "out",
      "queueName": "orders",
      "connection": "AzureWebJobsStorage"
    }
  ]
}
```

Cette fonction est définie comme étant une [fonction déclenchée par HTTP](./functions-bindings-http-webhook-trigger.md) qui retourne une [réponse HTTP](./functions-bindings-http-webhook-output.md) et génère un message de [Stockage File d’attente](./functions-bindings-storage-queue-output.md).

À la racine de l’application, le fichier *host.json* est configuré pour exécuter un fichier exécutable nommé `handler.exe` (`handler` dans Linux ou macOS).

```json
{
  "version": "2.0",
  "customHandler": {
    "description": {
      "defaultExecutablePath": "handler.exe"
    }
  },
  "extensionBundle": {
    "id": "Microsoft.Azure.Functions.ExtensionBundle",
    "version": "[1.*, 2.0.0)"
  }
}
```

Il s’agit de la requête HTTP envoyée au runtime Functions.

```http
POST http://127.0.0.1:7071/api/order HTTP/1.1
Content-Type: application/json

{
  "id": 1005,
  "quantity": 2,
  "color": "black"
}
```

Le runtime Functions envoie alors la requête HTTP suivante au gestionnaire personnalisé :

```http
POST http://127.0.0.1:<FUNCTIONS_CUSTOMHANDLER_PORT>/order HTTP/1.1
Content-Type: application/json

{
  "Data": {
    "req": {
      "Url": "http://localhost:7071/api/order",
      "Method": "POST",
      "Query": "{}",
      "Headers": {
        "Content-Type": [
          "application/json"
        ]
      },
      "Params": {},
      "Body": "{\"id\":1005,\"quantity\":2,\"color\":\"black\"}"
    }
  },
  "Metadata": {
  }
}
```

> [!NOTE]
> Certaines parties de la charge utile ont été supprimées par souci de concision.

*handler. exe* est le programme de gestionnaire personnalisé Go compilé qui exécute un serveur web et répond aux demandes d’appel de fonction de l’hôte Functions.

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "os"
)

type InvokeRequest struct {
    Data     map[string]json.RawMessage
    Metadata map[string]interface{}
}

type InvokeResponse struct {
    Outputs     map[string]interface{}
    Logs        []string
    ReturnValue interface{}
}

func orderHandler(w http.ResponseWriter, r *http.Request) {
    var invokeRequest InvokeRequest

    d := json.NewDecoder(r.Body)
    d.Decode(&invokeRequest)

    var reqData map[string]interface{}
    json.Unmarshal(invokeRequest.Data["req"], &reqData)

    outputs := make(map[string]interface{})
    outputs["message"] = reqData["Body"]

    resData := make(map[string]interface{})
    resData["body"] = "Order enqueued"
    outputs["res"] = resData
    invokeResponse := InvokeResponse{outputs, nil, nil}

    responseJson, _ := json.Marshal(invokeResponse)

    w.Header().Set("Content-Type", "application/json")
    w.Write(responseJson)
}

func main() {
    customHandlerPort, exists := os.LookupEnv("FUNCTIONS_CUSTOMHANDLER_PORT")
    if !exists {
        customHandlerPort = "8080"
    }
    mux := http.NewServeMux()
    mux.HandleFunc("/order", orderHandler)
    fmt.Println("Go server Listening on: ", customHandlerPort)
    log.Fatal(http.ListenAndServe(":"+customHandlerPort, mux))
}
```

Dans cet exemple, le gestionnaire personnalisé exécute un serveur web pour gérer les événements HTTP et est défini pour écouter les demandes via `FUNCTIONS_CUSTOMHANDLER_PORT`.

Même si l’hôte Functions a reçu la requête HTTP d’origine dans `/api/order`, il appelle le gestionnaire personnalisé en utilisant le nom de la fonction (son nom de dossier). Dans cet exemple, la fonction est définie sur le chemin `/order`. L’hôte envoie au gestionnaire personnalisé une requête HTTP sur le chemin `/order`.

Dès que des requêtes `POST` sont envoyées à cette fonction, les données de déclencheur et les métadonnées de fonction sont disponibles via le corps de la requête HTTP. Le corps de la requête HTTP d’origine est accessible dans le `Data.req.Body` de la charge utile.

La réponse de la fonction se présente sous la forme de paires clé/valeur où le membre `Outputs` contient une valeur JSON et où les clés correspondent aux sorties définies dans le fichier *function.json*.

Voici un exemple de charge utile que ce gestionnaire retourne à l’hôte Functions.

```json
{
  "Outputs": {
    "message": "{\"id\":1005,\"quantity\":2,\"color\":\"black\"}",
    "res": {
      "body": "Order enqueued"
    }
  },
  "Logs": null,
  "ReturnValue": null
}
```

En définissant la sortie `message` comme étant égale aux données de commande résultant de la demande, la fonction sort ces données de commande vers la file d’attente configurée. L’hôte Functions retourne aussi la réponse HTTP configurée dans `res` pour l’appelant.

### <a name="http-only-function"></a>Fonction HTTP uniquement

Pour les fonctions déclenchées par HTTP sans aucune liaison ou sortie supplémentaire, vous souhaiterez peut-être que votre gestionnaire utilise directement la requête et la réponse HTTP au lieu des charges utiles de [demande](#request-payload) et de [réponse](#response-payload) du gestionnaire personnalisé. Ce comportement peut être configuré dans *host.json* à l’aide du paramètre `enableForwardingHttpRequest`.

> [!IMPORTANT]
> La fonctionnalité de gestionnaire personnalisé vise essentiellement à activer les langages et les runtimes qui ne disposent pas actuellement d’une prise en charge de première classe dans Azure Functions. Bien qu’il soit possible d’exécuter les applications web en utilisant des gestionnaires personnalisés, Azure Functions n’est pas un proxy inverse standard. Certaines fonctionnalités, comme le streaming des réponses, HTTP/2 et les WebSockets, ne sont pas disponibles. Certains composants de la requête HTTP, notamment certains en-têtes et certaines routes, peuvent être restreints. Votre application peut aussi faire l’objet d’un [démarrage à froid](functions-scale.md#cold-start) excessif.
>
> Pour faire face à ces situations, envisagez d’exécuter vos applications web dans [Azure App Service](../app-service/overview.md).

L’exemple suivant montre comment configurer une fonction déclenchée par HTTP sans aucune autre liaison ni sortie. Le scénario implémenté dans cet exemple présente une fonction nommée `hello` qui accepte un `GET` ou un `POST`.

<a id="hello-implementation" name="hello-implementation"></a>

#### <a name="implementation"></a>Implémentation

Dans un dossier nommé *hello*, le fichier *function.json* configure la fonction déclenchée par HTTP.

**hello/function.json**

```json
{
  "bindings": [
    {
      "type": "httpTrigger",
      "authLevel": "anonymous",
      "direction": "in",
      "name": "req",
      "methods": ["get", "post"]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "res"
    }
  ]
}
```

La fonction est configurée pour accepter à la fois les demandes `GET` et `POST`, et la valeur de résultat est fournie via un argument nommé `res`.

À la racine de l’application, le fichier *host.json* est configuré pour exécuter `handler.exe` et `enableForwardingHttpRequest` est défini sur `true`.

```json
{
  "version": "2.0",
  "customHandler": {
    "description": {
      "defaultExecutablePath": "handler.exe"
    },
    "enableForwardingHttpRequest": true
  }
}
```

Quand `enableForwardingHttpRequest` a la valeur `true`, le comportement des fonctions HTTP uniquement diffère du comportement par défaut des gestionnaires personnalisés par défaut. Voici comment :

* La requête HTTP ne contient pas la charge utile de [demande](#request-payload) du gestionnaire personnalisé. Au lieu de cela, l’hôte Functions appelle le gestionnaire à partir d’une copie de la requête HTTP d’origine.
* L’hôte Functions appelle le gestionnaire en suivant le même chemin que la demande d’origine, avec les éventuels paramètres de chaîne de requête.
* L’hôte Functions retourne une copie de la réponse HTTP du gestionnaire comme réponse à la demande d’origine.

Voici une requête POST adressée à l’hôte Functions. L’hôte Functions envoie ensuite une copie de la demande au gestionnaire personnalisé sur le même chemin.

```http
POST http://127.0.0.1:7071/api/hello HTTP/1.1
Content-Type: application/json

{
  "message": "Hello World!"
}
```

Le fichier *handler.js* implémente un serveur web et une fonction HTTP.

```go
package main

import (
    "fmt"
    "io/ioutil"
    "log"
    "net/http"
    "os"
)

func helloHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    if r.Method == "GET" {
        w.Write([]byte("hello world"))
    } else {
        body, _ := ioutil.ReadAll(r.Body)
        w.Write(body)
    }
}

func main() {
    customHandlerPort, exists := os.LookupEnv("FUNCTIONS_CUSTOMHANDLER_PORT")
    if !exists {
        customHandlerPort = "8080"
    }
    mux := http.NewServeMux()
    mux.HandleFunc("/api/hello", helloHandler)
    fmt.Println("Go server Listening on: ", customHandlerPort)
    log.Fatal(http.ListenAndServe(":"+customHandlerPort, mux))
}
```

Dans cet exemple, le gestionnaire personnalisé crée un serveur web pour gérer les événements HTTP et est défini pour écouter les demandes via `FUNCTIONS_CUSTOMHANDLER_PORT`.

Les demandes `GET` sont gérées en retournant une chaîne, et les demandes `POST` ont accès au corps de la demande.

La route pour la fonction order est ici `/api/hello`, comme pour la demande d’origine.

>[!NOTE]
>`FUNCTIONS_CUSTOMHANDLER_PORT` n’est pas le port public utilisé pour appeler la fonction. Ce port est utilisé par l’hôte Functions pour appeler le gestionnaire personnalisé.

## <a name="deploying"></a>Déploiement en cours

Il est possible de déployer un gestionnaire personnalisé pour chaque option d’hébergement Azure Functions. Si votre gestionnaire nécessite des dépendances de système d’exploitation ou de plateforme (comme le runtime d’un langage), vous devrez peut-être utiliser un [conteneur personnalisé](./functions-create-function-linux-custom-image.md).

Si vous souhaitez créer une application de fonction dans Azure pour des gestionnaires personnalisés, nous vous recommandons de sélectionner .NET Core en guise de pile. Une pile « personnalisée » pour les gestionnaires personnalisés sera ajoutée dans l’avenir.

Pour déployer une application de gestionnaire personnalisée à l’aide d’Azure Functions Core Tools, exécutez la commande suivante.

```bash
func azure functionapp publish $functionAppName
```

> [!NOTE]
> Vérifiez que tous les fichiers nécessaires à l’exécution de votre gestionnaire personnalisé se trouvent dans le dossier et sont inclus dans le déploiement. Si votre gestionnaire personnalisé est un exécutable binaire ou possède des dépendances spécifiques de plateforme, vérifiez que ces fichiers correspondent à la plateforme de déploiement cible.

## <a name="restrictions"></a>Restrictions

- Le serveur web du gestionnaire personnalisé doit démarrer dans un délai de 60 secondes.

## <a name="samples"></a>Exemples

Vous trouverez sur [GitHub un dépôt d’exemples de gestionnaires personnalisés](https://github.com/Azure-Samples/functions-custom-handlers) qui montrent comment implémenter des fonctions dans différents langages.

## <a name="troubleshooting-and-support"></a>Résolution des problèmes et support

### <a name="trace-logging"></a>Journalisation du suivi

Si le processus de votre gestionnaire personnalisé ne parvient pas à démarrer ou s’il rencontre des problèmes de communication avec l’hôte Functions, vous pouvez monter le niveau de journalisation de l’application de fonction à `Trace` pour afficher davantage de messages de diagnostic de l’hôte.

Pour changer le niveau de journalisation par défaut de l’application de fonction, configurez le paramètre `logLevel` dans la section `logging` de *host.json*.

```json
{
  "version": "2.0",
  "customHandler": {
    "description": {
      "defaultExecutablePath": "handler.exe"
    }
  },
  "logging": {
    "logLevel": {
      "default": "Trace"
    }
  }
}
```

L’hôte Functions sort des messages de journal supplémentaires, notamment des informations relatives au processus du gestionnaire personnalisé. Utilisez les journaux pour étudier les problèmes de démarrage du processus de votre gestionnaire personnalisé ou les problèmes d’appel de fonctions dans votre gestionnaire personnalisé.

Localement, les journaux sont imprimés dans la console.

Dans Azure, [interrogez les traces Application Insights](analyze-telemetry-data.md#query-telemetry-data) pour afficher les messages du journal. Si votre application produit un grand nombre de journaux, seul un sous-ensemble des messages de journal est envoyé à Application Insights. [Désactivez l’échantillonnage](configure-monitoring.md#configure-sampling) pour faire en sorte que tous les messages soient journalisés.

### <a name="test-custom-handler-in-isolation"></a>Tester le gestionnaire personnalisé en isolation

Sachant qu’une application de gestionnaire personnalisé est un processus de serveur web, il peut être utile de la faire démarrer seule et de tester les appels de fonction en envoyant des [requêtes HTTP](#request-payload) fictives à l’aide d’un outil comme [cURL](https://curl.haxx.se/) ou [Postman](https://www.postman.com/).

Vous pouvez aussi utiliser cette stratégie dans vos pipelines CI/CD pour exécuter des tests automatisés sur votre gestionnaire personnalisé.

### <a name="execution-environment"></a>Environnement d’exécution

Les gestionnaires personnalisés s’exécutent dans le même environnement qu’une application Azure Functions classique. Testez votre gestionnaire pour vérifier que l’environnement contient toutes les dépendances dont il a besoin pour s’exécuter. Pour les applications qui nécessitent des dépendances supplémentaires, vous devrez peut-être les exécuter en utilisant une [image de conteneur personnalisée](functions-create-function-linux-custom-image.md) hébergée dans un [plan Premium](functions-premium-plan.md) Azure Functions.

### <a name="get-support"></a>Obtenir de l’aide

Si vous avez besoin d’aide sur une application de fonction assortie de gestionnaires personnalisés, vous pouvez envoyer une demande via les canaux de support standard. Cependant, compte tenu de la grande diversité des langages pouvant être utilisés pour créer des applications de gestionnaire personnalisé, le support n’est pas illimité.

Un support est fourni si l’hôte Functions rencontre des problèmes de démarrage ou de communication avec le processus du gestionnaire personnalisé. Pour les problèmes propres au fonctionnement interne du processus de votre gestionnaire personnalisé, comme ceux liés au langage ou à l’infrastructure choisis, notre équipe de support ne peut pas proposer d’assistance dans ce contexte.

## <a name="next-steps"></a>Étapes suivantes

Commencez à créer une application Azure Functions en Go ou Rust à l’aide du [démarrage rapide des gestionnaires personnalisés](create-first-function-vs-code-other.md).
