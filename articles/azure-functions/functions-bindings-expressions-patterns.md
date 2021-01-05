---
title: Expressions et modèles de liaisons Azure Functions
description: Découvrez comment créer différentes expressions de liaison Azure Functions d’après des modèles courants.
author: craigshoemaker
ms.topic: reference
ms.custom: devx-track-csharp
ms.date: 02/18/2019
ms.author: cshoe
ms.openlocfilehash: 161e3e7fbc5b343ee73142f0e968367c3cbfaa6b
ms.sourcegitcommit: dd45ae4fc54f8267cda2ddf4a92ccd123464d411
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92927411"
---
# <a name="azure-functions-binding-expression-patterns"></a>Modèles d’expressions de liaison Azure Functions

Les *expressions de liaison* sont l’une des fonctionnalités les plus puissantes des [déclencheurs et liaisons](./functions-triggers-bindings.md). Dans le fichier *function.json* ainsi que dans les paramètres et le code de la fonction, vous pouvez utiliser des expressions qui sont remplacées par des valeurs provenant de diverses sources.

La plupart des expressions sont identifiées par les accolades qui les entourent. Par exemple, dans une fonction de déclencheur de file d’attente, `{queueTrigger}` correspond au texte du message de file d’attente. Si la propriété `path` pour une liaison de sortie d’objet blob est `container/{queueTrigger}` et que la fonction est déclenchée par un message de file d’attente `HelloWorld`, alors un objet blob nommé `HelloWorld` est créé.

Types d’expressions de liaison

* [Paramètres de l’application](#binding-expressions---app-settings)
* [Nom du fichier de déclencheur](#trigger-file-name)
* [Métadonnées de déclencheur](#trigger-metadata)
* [Charges utiles JSON](#json-payloads)
* [Nouveau GUID](#create-guids)
* [Date et heure actuelles](#current-time)

## <a name="binding-expressions---app-settings"></a>Expression de liaison - Paramètres d’application

En tant que meilleure pratique, les chaînes de connexion et les secrets doivent être gérés avec des paramètres de l’application, plutôt qu’avec des fichiers de configuration. Cela limite l’accès à ces secrets et permet de stocker des fichiers tels que *function.json* en toute sécurité dans des référentiels de contrôle de code source public.

Les paramètres de l’application sont également utiles lorsque vous souhaitez modifier la configuration en fonction de l’environnement. Par exemple, dans un environnement de test, vous pourriez vouloir surveiller une autre file d’attente ou un autre conteneur Stockage Blob.

Les expressions de liaison de paramètres d’application ne sont pas identifiées comme les autres expressions de liaison : elles sont entourées de signes de pourcentage et non d’accolades. Par exemple, si le chemin de liaison de sortie d’objet blob est `%Environment%/newblob.txt` et que la valeur du paramètre d’application `Environment` est `Development`, un objet blob sera créé dans le conteneur `Development`.

Lorsqu’une fonction s’exécute localement, les valeurs des paramètres de l’application proviennent du fichier *local.settings.json*.

> [!NOTE]
> La propriété `connection` des déclencheurs et des liaisons est un cas spécial et résout automatiquement les valeurs en tant que paramètres de l’application, sans les signes de pourcentage. 

L’exemple suivant est un déclencheur de stockage de file d’attente Azure qui utilise un paramètre d’application `%input_queue_name%` pour définir la file d’attente sur laquelle effectuer le déclenchement.

```json
{
  "bindings": [
    {
      "name": "order",
      "type": "queueTrigger",
      "direction": "in",
      "queueName": "%input_queue_name%",
      "connection": "MY_STORAGE_ACCT_APP_SETTING"
    }
  ]
}
```

Vous pouvez utiliser la même approche dans les bibliothèques de classes :

```csharp
[FunctionName("QueueTrigger")]
public static void Run(
    [QueueTrigger("%input_queue_name%")]string myQueueItem, 
    ILogger log)
{
    log.LogInformation($"C# Queue trigger function processed: {myQueueItem}");
}
```

## <a name="trigger-file-name"></a>Nom du fichier de déclencheur

Le `path` pour un objet blob de déclencheur peut être un modèle qui vous permet de faire référence au nom de l’objet blob de déclenchement dans d’autres liaisons et codes de fonction. Le modèle peut également inclure des critères de filtre qui indiquent les objets blob qui peuvent déclencher un appel de fonction.

Par exemple, dans la liaison de déclencheur d’objet blob suivante, le modèle `path` est `sample-images/{filename}`, ce qui crée une expression de liaison nommée `filename` :

```json
{
  "bindings": [
    {
      "name": "image",
      "type": "blobTrigger",
      "path": "sample-images/{filename}",
      "direction": "in",
      "connection": "MyStorageConnection"
    },
    ...
```

L’expression `filename` peut ensuite être utilisée dans une liaison de sortie pour spécifier le nom de l’objet blob en cours de création :

```json
    ...
    {
      "name": "imageSmall",
      "type": "blob",
      "path": "sample-images-sm/{filename}",
      "direction": "out",
      "connection": "MyStorageConnection"
    }
  ],
}
```

Le code de fonction peut accéder à cette même valeur en utilisant `filename` comme nom de paramètre :

```csharp
// C# example of binding to {filename}
public static void Run(Stream image, string filename, Stream imageSmall, ILogger log)  
{
    log.LogInformation($"Blob trigger processing: {filename}");
    // ...
} 
```

<!--TODO: add JavaScript example -->
<!-- Blocked by bug https://github.com/Azure/Azure-Functions/issues/248 -->

La même possibilité d’utiliser des modèles et des expressions de liaison s’applique aux attributs dans les bibliothèques de classes. Dans l’exemple suivant, les paramètres de constructeur d’attribut sont les mêmes valeurs `path` que les exemples de *function.json* précédents : 

```csharp
[FunctionName("ResizeImage")]
public static void Run(
    [BlobTrigger("sample-images/{filename}")] Stream image,
    [Blob("sample-images-sm/{filename}", FileAccess.Write)] Stream imageSmall,
    string filename,
    ILogger log)
{
    log.LogInformation($"Blob trigger processing: {filename}");
    // ...
}

```

Vous pouvez également créer des expressions pour certaines parties du nom de fichier. Dans l’exemple suivant, la fonction est déclenchée uniquement sur les noms de fichiers qui correspondent à un modèle : `anyname-anyfile.csv`

```json
{
    "name": "myBlob",
    "type": "blobTrigger",
    "direction": "in",
    "path": "testContainerName/{date}-{filetype}.csv",
    "connection": "OrderStorageConnection"
}
```

Pour plus d’informations sur l’utilisation des expressions et des modèles dans la chaîne du chemin d’accès à l’objet blob, consultez l’article sur la [référence de liaison de stockage blob](functions-bindings-storage-blob.md).

## <a name="trigger-metadata"></a>Métadonnées d’un déclencheur

En plus de la charge utile de données fournie par un déclencheur (par exemple, le contenu du message de file d’attente qui a déclenché une fonction), plusieurs déclencheurs fournissent des valeurs de métadonnées supplémentaires. Ces valeurs peuvent être utilisées comme paramètres d’entrée dans C# et F# ou comme propriétés sur l’objet `context.bindings` dans JavaScript. 

Par exemple, un déclencheur Stockage File d’attente Azure prend en charge les propriétés suivantes :

* QueueTrigger - déclenchant le contenu du message si une chaîne valide
* DequeueCount
* ExpirationTime
* Id
* InsertionTime
* NextVisibleTime
* PopReceipt

Ces valeurs de métadonnées sont accessibles dans les propriétés de fichier *function.json*. Par exemple, supposons que vous utilisez un déclencheur de file d’attente et que le message de la file d’attente contient le nom d’un objet blob que vous souhaitez lire. Dans le fichier *function.json* , vous pouvez utiliser la propriété de métadonnées `queueTrigger` dans la propriété de l’objet blob `path`, comme indiqué dans l’exemple suivant :

```json
  "bindings": [
    {
      "name": "myQueueItem",
      "type": "queueTrigger",
      "queueName": "myqueue-items",
      "connection": "MyStorageConnection",
    },
    {
      "name": "myInputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}",
      "direction": "in",
      "connection": "MyStorageConnection"
    }
  ]
```

Les détails des propriétés de métadonnées pour chaque déclencheur sont décrits dans l’article de référence correspondante. Pour obtenir un exemple, consultez [Métadonnées de déclencheur de file d’attente](functions-bindings-storage-queue-trigger.md#message-metadata). La documentation est également disponible dans l’onglet **Intégrer** du portail, dans la section **Documentation** située sous la zone de configuration de liaison.  

## <a name="json-payloads"></a>Charges utiles JSON

Lorsque la charge utile d’un déclencheur est JSON, vous pouvez faire référence à ses propriétés dans la configuration pour d’autres liaisons dans la même fonction et dans le code de la fonction.

L’exemple suivant illustre le fichier *function.json* pour une fonction de Webhook qui reçoit un nom d’objet blob dans JSON : `{"BlobName":"HelloWorld.txt"}`. Une liaison d’entrée d’objet blob lit l’objet blob et la liaison de sortie HTTP retourne le contenu de l’objet blob dans la réponse HTTP. Notez que la liaison d’entrée d’objet blob obtient le nom de l’objet blob en faisant directement référence à la propriété `BlobName` (`"path": "strings/{BlobName}"`)

```json
{
  "bindings": [
    {
      "name": "info",
      "type": "httpTrigger",
      "direction": "in",
      "webHookType": "genericJson"
    },
    {
      "name": "blobContents",
      "type": "blob",
      "direction": "in",
      "path": "strings/{BlobName}",
      "connection": "AzureWebJobsStorage"
    },
    {
      "name": "res",
      "type": "http",
      "direction": "out"
    }
  ]
}
```

Pour que cela fonctionne en C# et F#, vous avez besoin d’une classe qui définit les champs à désérialiser, comme dans l’exemple suivant :

```csharp
using System.Net;
using Microsoft.Extensions.Logging;

public class BlobInfo
{
    public string BlobName { get; set; }
}
  
public static HttpResponseMessage Run(HttpRequestMessage req, BlobInfo info, string blobContents, ILogger log)
{
    if (blobContents == null) {
        return req.CreateResponse(HttpStatusCode.NotFound);
    } 

    log.LogInformation($"Processing: {info.BlobName}");

    return req.CreateResponse(HttpStatusCode.OK, new {
        data = $"{blobContents}"
    });
}
```

Dans JavaScript, la désérialisation JSON est effectuée automatiquement.

```javascript
module.exports = function (context, info) {
    if ('BlobName' in info) {
        context.res = {
            body: { 'data': context.bindings.blobContents }
        }
    }
    else {
        context.res = {
            status: 404
        };
    }
    context.done();
}
```

### <a name="dot-notation"></a>Notation par points

Si certaines des propriétés de votre charge utile JSON sont des objets dotés de propriétés, vous pouvez y faire référence directement à l’aide de la notation par points. Par exemple, supposons que votre JSON ressemble à l’exemple qui suit :

```json
{
  "BlobName": {
    "FileName":"HelloWorld",
    "Extension":"txt"
  }
}
```

Vous pouvez faire référence directement à `FileName` par l’expression `BlobName.FileName`. Avec ce format JSON, voici ce à quoi la propriété `path` de l’exemple précédent ressemblerait :

```json
"path": "strings/{BlobName.FileName}.{BlobName.Extension}",
```

En C#, vous avez besoin de deux classes :

```csharp
public class BlobInfo
{
    public BlobName BlobName { get; set; }
}
public class BlobName
{
    public string FileName { get; set; }
    public string Extension { get; set; }
}
```

## <a name="create-guids"></a>Créer des GUID

L’expression de liaison `{rand-guid}` crée un GUID. Le chemin d’accès à l’objet blob suivant dans un fichier `function.json` crée un objet blob avec un nom similaire à ce qui suit : *50710cb5-84b9-4d87-9d83-a03d6976a682.txt*.

```json
{
  "type": "blob",
  "name": "blobOutput",
  "direction": "out",
  "path": "my-output-container/{rand-guid}.txt"
}
```

## <a name="current-time"></a>Heure actuelle

L’expression de liaison `DateTime` se résout en `DateTime.UtcNow`. Le chemin d’accès à l’objet blob suivant dans un fichier `function.json` crée un objet blob avec un nom similaire à ce qui suit : *2018-02-16T17-59-55Z.txt*.

```json
{
  "type": "blob",
  "name": "blobOutput",
  "direction": "out",
  "path": "my-output-container/{DateTime}.txt"
}
```
## <a name="binding-at-runtime"></a>Liaison au runtime

Avec C# et d’autres langages .NET, vous pouvez utiliser un schéma de liaison impératif, par opposition aux liaisons déclaratives dans *function.json* , et des attributs. La liaison impérative est utile lorsque les paramètres de liaison doivent être calculés au moment du runtime plutôt que lors de la conception. Pour plus d’informations, consultez les [informations de référence pour les développeurs C#](functions-dotnet-class-library.md#binding-at-runtime) ou les [informations de référence pour les développeurs de scripts C#](functions-reference-csharp.md#binding-at-runtime).

## <a name="next-steps"></a>Étapes suivantes
> [!div class="nextstepaction"]
> [Utilisation de la valeur de retour Azure Functions](./functions-bindings-return-value.md)
