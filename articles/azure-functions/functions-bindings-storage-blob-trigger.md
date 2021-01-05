---
title: Déclencheur Stockage Blob Azure pour Azure Functions
description: Découvrez comment exécuter une fonction Azure lorsque des données Stockage Blob Azure changent.
author: craigshoemaker
ms.topic: reference
ms.date: 02/13/2020
ms.author: cshoe
ms.custom: devx-track-csharp, devx-track-python
ms.openlocfilehash: 45393f116149f6cf16763d2d7033f8425df235bf
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95998831"
---
# <a name="azure-blob-storage-trigger-for-azure-functions"></a>Déclencheur Stockage Blob Azure pour Azure Functions

Le déclencheur de stockage Blob démarre une fonction lors de la détection d’un objet blob nouveau ou mis à jour. Le contenu du blob est fourni comme [entrée de la fonction](./functions-bindings-storage-blob-input.md).

Le déclencheur Stockage Blob Azure nécessite un compte de stockage universel. Les comptes Stockage v2 avec [espaces de noms hiérarchiques](../storage/blobs/data-lake-storage-namespace.md) sont également pris en charge. Pour utiliser un compte dédié aux blobs ou si votre application a des besoins spécifiques, passez en revue les alternatives à l’utilisation de ce déclencheur.

Pour plus d’informations sur les détails d’installation et de configuration, consultez la [vue d’ensemble](./functions-bindings-storage-blob.md).

## <a name="polling"></a>Interrogation

L’interrogation fonctionne de façon hybride entre l’inspection des journaux et l’exécution d’analyses régulières des conteneurs. Les objets blob sont analysés dans des groupes de 10 000 à la fois avec un jeton de continuation utilisé entre les intervalles.

> [!WARNING]
> En outre, les [journaux d’activité de stockage sont créés selon le principe du meilleur effort](/rest/api/storageservices/About-Storage-Analytics-Logging). Il n’existe aucune garantie que tous les événements sont capturés. Dans certaines conditions, des journaux d’activité peuvent être omis.
> 
> Si vous avez besoin de traitement d’objets blob plus rapide ou plus fiable, envisagez de créer un [message de file d’attente](../storage/queues/storage-dotnet-how-to-use-queues.md) quand vous créez l’objet blob. Ensuite, utilisez un [déclencheur de file d’attente](functions-bindings-storage-queue.md) plutôt qu’un déclencheur d’objet blob pour traiter l’objet blob. Une autre option consiste à utiliser Event Grid ; consultez le didacticiel [Automatiser le redimensionnement des images téléchargées à l’aide d’Event Grid](../event-grid/resize-images-on-storage-blob-upload-event.md).
>

## <a name="alternatives"></a>Autres solutions

### <a name="event-grid-trigger"></a>Déclencheur Event Grid

Le [déclencheur Event Grid](functions-bindings-event-grid.md) possède également une prise en charge intégrée des [événements blobs](../storage/blobs/storage-blob-event-overview.md). Utilisez Event Grid au lieu du déclencheur de stockage Blob pour les scénarios suivants :

- **Comptes de stockage Blob uniquement** : Les [comptes de stockage Blob uniquement](../storage/common/storage-account-overview.md#types-of-storage-accounts) sont pris en charge pour les liaisons d’entrée et de sortie, mais pas pour les déclencheurs blob.

- **Scalabilité élevée** : La scalabilité élevée peut être définie comme des conteneurs qui contiennent plus de 100 000 objets blob ou des comptes de stockage avec plus de 100 mises à jour d’objets blob par seconde.

- **Minimisation de la latence** : Si votre application de fonction est dans le plan Consommation, il peut y avoir jusqu’à 10 minutes de délai dans le traitement des nouveaux objets blob si une application de fonction est devenue inactive. Pour éviter toute latence, vous pouvez passer à un plan App Service dans lequel est activé AlwaysOn. Vous pouvez également utiliser un [déclencheur Event Grid](functions-bindings-event-grid.md) avec votre compte de stockage Blob. Pour obtenir un exemple, consultez le [tutoriel Event Grid](../event-grid/resize-images-on-storage-blob-upload-event.md?toc=%2Fazure%2Fazure-functions%2Ftoc.json).

Pour obtenir un exemple Azure Event Grid, consultez le tutoriel [Redimensionnement d’image avec Event Grid](../event-grid/resize-images-on-storage-blob-upload-event.md).

### <a name="queue-storage-trigger"></a>Déclencheur de stockage de file d’attente

Une autre approche du traitement des blobs consiste à écrire des messages de file d’attente qui correspondent à des blobs créés ou modifiés, puis à utiliser un [déclencheur Stockage File d’attente](./functions-bindings-storage-queue.md) pour commencer le traitement.

## <a name="example"></a>Exemple

# <a name="c"></a>[C#](#tab/csharp)

L’exemple suivant montre une [fonction C#](functions-dotnet-class-library.md) qui écrit une entrée de journal lorsqu’un objet blob est ajouté ou mis à jour dans le conteneur `samples-workitems`.

```csharp
[FunctionName("BlobTriggerCSharp")]        
public static void Run([BlobTrigger("samples-workitems/{name}")] Stream myBlob, string name, ILogger log)
{
    log.LogInformation($"C# Blob trigger function Processed blob\n Name:{name} \n Size: {myBlob.Length} Bytes");
}
```

La chaîne `{name}` dans le chemin d’accès du déclencheur d’objet blob `samples-workitems/{name}` crée une [expression de liaison](./functions-bindings-expressions-patterns.md) que vous pouvez utiliser dans le code de fonction pour accéder au nom de fichier de l’objet blob déclencheur. Pour plus d’informations, consultez la section [Modèles de nom d’objet blob](#blob-name-patterns) dans la suite de cet article.

Pour plus d’informations sur l’attribut `BlobTrigger`, consultez [les attributs et les annotations](#attributes-and-annotations).

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

L’exemple suivant montre une liaison de déclencheur d’objet blob dans un fichier *function.json* et un code qui utilise la liaison. La fonction enregistre un journal lorsqu'un objet blob est ajouté ou mis à jour dans le `samples-workitems` [conteneur](../storage/blobs/storage-blobs-introduction.md#blob-storage-resources).

Voici les données de liaison dans le fichier *function.json* :

```json
{
    "disabled": false,
    "bindings": [
        {
            "name": "myBlob",
            "type": "blobTrigger",
            "direction": "in",
            "path": "samples-workitems/{name}",
            "connection":"MyStorageAccountAppSetting"
        }
    ]
}
```

La chaîne `{name}` dans le chemin d’accès du déclencheur d’objet blob `samples-workitems/{name}` crée une [expression de liaison](./functions-bindings-expressions-patterns.md) que vous pouvez utiliser dans le code de fonction pour accéder au nom de fichier de l’objet blob déclencheur. Pour plus d’informations, consultez la section [Modèles de nom d’objet blob](#blob-name-patterns) dans la suite de cet article.

Pour plus d’informations sur les propriétés du fichier *function.json*, consultez la section [Configuration](#configuration) qui décrit ces propriétés.

Voici le code Script C# qui lie à `Stream` :

```cs
public static void Run(Stream myBlob, string name, ILogger log)
{
   log.LogInformation($"C# Blob trigger function Processed blob\n Name:{name} \n Size: {myBlob.Length} Bytes");
}
```

Voici le code Script C# qui lie à `CloudBlockBlob` :

```cs
#r "Microsoft.WindowsAzure.Storage"

using Microsoft.WindowsAzure.Storage.Blob;

public static void Run(CloudBlockBlob myBlob, string name, ILogger log)
{
    log.LogInformation($"C# Blob trigger function Processed blob\n Name:{name}\nURI:{myBlob.StorageUri}");
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

L’exemple suivant montre une liaison de déclencheur d’objet blob dans un fichier *function.json* et un [code JavaScript](functions-reference-node.md) qui utilise la liaison. La fonction écrit un journal lorsqu’un objet blob est ajouté ou mis à jour dans le conteneur `samples-workitems`.

Voici le fichier *function.json* :

```json
{
    "disabled": false,
    "bindings": [
        {
            "name": "myBlob",
            "type": "blobTrigger",
            "direction": "in",
            "path": "samples-workitems/{name}",
            "connection":"MyStorageAccountAppSetting"
        }
    ]
}
```

La chaîne `{name}` dans le chemin d’accès du déclencheur d’objet blob `samples-workitems/{name}` crée une [expression de liaison](./functions-bindings-expressions-patterns.md) que vous pouvez utiliser dans le code de fonction pour accéder au nom de fichier de l’objet blob déclencheur. Pour plus d’informations, consultez la section [Modèles de nom d’objet blob](#blob-name-patterns) dans la suite de cet article.

Pour plus d’informations sur les propriétés du fichier *function.json*, consultez la section [Configuration](#configuration) qui décrit ces propriétés.

Voici le code JavaScript :

```javascript
module.exports = function(context) {
    context.log('Node.js Blob trigger function processed', context.bindings.myBlob);
    context.done();
};
```

# <a name="python"></a>[Python](#tab/python)

L’exemple suivant montre une liaison de déclencheur de blob dans un fichier *function.json* et un [code Python](functions-reference-python.md) qui utilise la liaison. La fonction enregistre un journal lorsqu'un objet blob est ajouté ou mis à jour dans le `samples-workitems` [conteneur](../storage/blobs/storage-blobs-introduction.md#blob-storage-resources).

Voici le fichier *function.json* :

```json
{
    "scriptFile": "__init__.py",
    "disabled": false,
    "bindings": [
        {
            "name": "myblob",
            "type": "blobTrigger",
            "direction": "in",
            "path": "samples-workitems/{name}",
            "connection":"MyStorageAccountAppSetting"
        }
    ]
}
```

La chaîne `{name}` dans le chemin d’accès du déclencheur d’objet blob `samples-workitems/{name}` crée une [expression de liaison](./functions-bindings-expressions-patterns.md) que vous pouvez utiliser dans le code de fonction pour accéder au nom de fichier de l’objet blob déclencheur. Pour plus d’informations, consultez la section [Modèles de nom d’objet blob](#blob-name-patterns) dans la suite de cet article.

Pour plus d’informations sur les propriétés du fichier *function.json*, consultez la section [Configuration](#configuration) qui décrit ces propriétés.

Voici le code Python :

```python
import logging
import azure.functions as func


def main(myblob: func.InputStream):
    logging.info('Python Blob trigger function processed %s', myblob.name)
```

# <a name="java"></a>[Java](#tab/java)

Cette fonction enregistre un journal lorsqu’un objet blob est ajouté ou mis à jour dans le conteneur `myblob`.

```java
@FunctionName("blobprocessor")
public void run(
  @BlobTrigger(name = "file",
               dataType = "binary",
               path = "myblob/{name}",
               connection = "MyStorageAccountAppSetting") byte[] content,
  @BindingName("name") String filename,
  final ExecutionContext context
) {
  context.getLogger().info("Name: " + filename + " Size: " + content.length + " bytes");
}
```

---

## <a name="attributes-and-annotations"></a>Attributs et annotations

# <a name="c"></a>[C#](#tab/csharp)

Dans les [bibliothèques de classes C#](functions-dotnet-class-library.md), utilisez les attributs suivants pour configurer un déclencheur d’objet blob :

* [BlobTriggerAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs.Extensions.Storage/Blobs/BlobTriggerAttribute.cs)

  Le constructeur de l’attribut prend une chaîne de chemin qui indique le conteneur à surveiller et éventuellement un [modèle de nom d’objet blob](#blob-name-patterns). Voici un exemple :

  ```csharp
  [FunctionName("ResizeImage")]
  public static void Run(
      [BlobTrigger("sample-images/{name}")] Stream image,
      [Blob("sample-images-md/{name}", FileAccess.Write)] Stream imageSmall)
  {
      ....
  }
  ```

  Vous pouvez définir la propriété `Connection` pour spécifier le compte de stockage à utiliser, comme indiqué dans l’exemple suivant :

   ```csharp
  [FunctionName("ResizeImage")]
  public static void Run(
      [BlobTrigger("sample-images/{name}", Connection = "StorageConnectionAppSetting")] Stream image,
      [Blob("sample-images-md/{name}", FileAccess.Write)] Stream imageSmall)
  {
      ....
  }
   ```

  Pour obtenir un exemple complet, consultez [Exemple de déclencheur](#example).

* [StorageAccountAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/StorageAccountAttribute.cs)

  Fournit une autre manière de spécifier le compte de stockage à utiliser. Le constructeur prend le nom d’un paramètre d’application comportant une chaîne de connexion de stockage. L’attribut peut être appliqué au niveau du paramètre, de la méthode ou de la classe. L’exemple suivant montre le niveau de la classe et celui de la méthode :

  ```csharp
  [StorageAccount("ClassLevelStorageAppSetting")]
  public static class AzureFunctions
  {
      [FunctionName("BlobTrigger")]
      [StorageAccount("FunctionLevelStorageAppSetting")]
      public static void Run( //...
  {
      ....
  }
  ```

Le compte de stockage à utiliser est déterminé dans l’ordre suivant :

* La propriété `Connection` de l’attribut `BlobTrigger`.
* L’attribut `StorageAccount` appliqué au même paramètre que l’attribut `BlobTrigger`.
* L’attribut `StorageAccount` appliqué à la fonction.
* L’attribut `StorageAccount` appliqué à la classe.
* Le compte de stockage par défaut pour l’application de fonction (paramètre d’application « AzureWebJobsStorage »).

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

Les attributs ne sont pas pris en charge par le script C#.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Les attributs ne sont pas pris en charge par JavaScript.

# <a name="python"></a>[Python](#tab/python)

Les attributs ne sont pas pris en charge par Python.

# <a name="java"></a>[Java](#tab/java)

L'attribut `@BlobTrigger` est utilisé pour vous permettre d’accéder à l’objet blob qui a déclenché la fonction. Reportez-vous à l’[exemple de déclencheur](#example) pour plus d'informations.

---

## <a name="configuration"></a>Configuration

Le tableau suivant décrit les propriétés de configuration de liaison que vous définissez dans le fichier *function.json* et l’attribut `BlobTrigger`.

|Propriété function.json | Propriété d’attribut |Description|
|---------|---------|----------------------|
|**type** | n/a | Cette propriété doit être définie sur `blobTrigger`. Cette propriété est définie automatiquement lorsque vous créez le déclencheur dans le portail Azure.|
|**direction** | n/a | Cette propriété doit être définie sur `in`. Cette propriété est définie automatiquement lorsque vous créez le déclencheur dans le portail Azure. Les exceptions sont notées à la section [utilisation](#usage). |
|**name** | n/a | Nom de la variable qui représente l’objet blob dans le code de la fonction. |
|**path** | **BlobPath** |[Conteneur](../storage/blobs/storage-blobs-introduction.md#blob-storage-resources) à superviser.  Peut être un [modèle de nom d’objet blob](#blob-name-patterns). |
|**connection** | **Connection** | Nom d’un paramètre d’application comportant la chaîne de connexion de stockage à utiliser pour cette liaison. Si le nom du paramètre d’application commence par « AzureWebJobs », vous ne pouvez spécifier que le reste du nom ici. Par exemple, si vous définissez `connection` sur « MyStorage », le runtime Functions recherche un paramètre d’application qui est nommé « AzureWebJobsMyStorage ». Si vous laissez `connection` vide, le runtime Functions utilise la chaîne de connexion de stockage par défaut dans le paramètre d’application nommé `AzureWebJobsStorage`.<br><br>La chaîne de connexion doit être destinée à un compte de stockage à usage général, et non pas à un [compte Stockage Blob](../storage/common/storage-account-overview.md#types-of-storage-accounts).|

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="usage"></a>Usage

# <a name="c"></a>[C#](#tab/csharp)

[!INCLUDE [functions-bindings-blob-storage-trigger](../../includes/functions-bindings-blob-storage-trigger.md)]

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

[!INCLUDE [functions-bindings-blob-storage-trigger](../../includes/functions-bindings-blob-storage-trigger.md)]

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Accédez aux données des objets blob à l'aide de `context.bindings.<NAME>`, sachant que `<NAME>` correspond à la valeur définie dans *function.json*.

# <a name="python"></a>[Python](#tab/python)

Accédez aux données blob via le paramètre de type [InputStream](/python/api/azure-functions/azure.functions.inputstream?view=azure-python). Reportez-vous à l’[exemple de déclencheur](#example) pour plus d'informations.

# <a name="java"></a>[Java](#tab/java)

L'attribut `@BlobTrigger` est utilisé pour vous permettre d’accéder à l’objet blob qui a déclenché la fonction. Reportez-vous à l’[exemple de déclencheur](#example) pour plus d'informations.

---

## <a name="blob-name-patterns"></a>Modèles de nom de blob

Vous pouvez spécifier un modèle de nom d’objet blob dans la propriété `path` du fichier *function.json* ou dans le constructeur d’attribut `BlobTrigger`. Le modèle de nom peut être une [expression de filtre ou de liaison](./functions-bindings-expressions-patterns.md). Les sections suivantes fournissent des exemples.

### <a name="get-file-name-and-extension"></a>Obtenir l’extension et le nom de fichier

L’exemple suivant montre comment se lier séparément à l’extension et au nom de fichier de l’objet blob :

```json
"path": "input/{blobname}.{blobextension}",
```

Si l’objet blob est nommé *original-Blob1.txt*, les valeurs des variables `blobname` et `blobextension` dans le code de la fonction sont *original-Blob1* et *txt*.

### <a name="filter-on-blob-name"></a>Filtrer sur le nom de l’objet blob

L’exemple suivant déclenche uniquement sur les objets blob du conteneur `input` qui commencent par la chaîne « original- » :

```json
"path": "input/original-{name}",
```

Si le nom de l’objet blob est *original-Blob1.txt*, la valeur de la variable `name` dans le code de la fonction est `Blob1.txt`.

### <a name="filter-on-file-type"></a>Filtrer sur le type de fichier

L’exemple suivant ne déclenche que sur des fichiers *.png* :

```json
"path": "samples/{name}.png",
```

### <a name="filter-on-curly-braces-in-file-names"></a>Filtrer sur les accolades dans les noms de fichiers

Pour rechercher les accolades dans les noms de fichiers, utilisez une séquence d’échappement sous la forme de deux accolades. L’exemple suivant filtre en recherchant les objets blob qui contiennent des accolades dans le nom :

```json
"path": "images/{{20140101}}-{name}",
```

Si l’objet blob est nommé *{20140101}-soundfile.mp3*, la valeur de la variable `name` dans le code de la fonction est *soundfile.mp3*.

## <a name="metadata"></a>Métadonnées

# <a name="c"></a>[C#](#tab/csharp)

[!INCLUDE [functions-bindings-blob-storage-trigger](../../includes/functions-bindings-blob-storage-metadata.md)]

# <a name="c-script"></a>[Script C#](#tab/csharp-script)

[!INCLUDE [functions-bindings-blob-storage-trigger](../../includes/functions-bindings-blob-storage-metadata.md)]

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
module.exports = function (context, myBlob) {
    context.log("Full blob path:", context.bindingData.blobTrigger);
    context.done();
};
```

# <a name="python"></a>[Python](#tab/python)

Les métadonnées ne sont pas disponibles dans Python.

# <a name="java"></a>[Java](#tab/java)

Les métadonnées ne sont pas disponibles dans Java.

---

## <a name="blob-receipts"></a>Reçus d’objets blob

Le runtime Azure Functions vérifie qu’aucune fonction de déclencheur d’objet blob n’est appelée plusieurs fois pour un même objet blob, nouveau ou mis à jour. Pour déterminer si la version d’un objet blob donné a été traitée, il gère des *reçus d’objet blob*.

Azure Functions stocke les reçus d’objet blob dans un conteneur appelé *azure-webjobs-hosts* dans le compte de stockage Azure de votre application de fonction (définie par le paramètre d’application `AzureWebJobsStorage`). Un reçu d’objet blob contient les informations suivantes :

* Fonction déclenchée (« *&lt;nom de l’application de fonction>* .Functions. *&lt;nom de la fonction>*  », par exemple : « MyFunctionApp.Functions.CopyBlob »)
* Nom du conteneur
* Type d’objet blob (« BlockBlob » ou « PageBlob »)
* Nom de l’objet blob
* Étiquette d’entité (identificateur de version de l’objet blob, par exemple : « 0x8D1DC6E70A277EF »)

Pour forcer le retraitement d’un objet blob, supprimez manuellement le reçu de l’objet blob du conteneur *azure-webjobs-hosts*. Si le retraitement n’a pas lieu immédiatement, il se produira ultérieurement. Pour effectuer un retraitement immédiatement, le blob *ScanInfo* dans *azure-webjobs-hosts/blobscaninfo* peut être mis à jour. Les blobs ayant un timestamp de dernière modification après la propriété `LatestScan` sont à nouveau analysés.

## <a name="poison-blobs"></a>Blobs incohérents

En cas d’échec d’une fonction de déclencheur d’objet blob, Azure Functions réessaie cette fonction jusqu’à 5 fois par défaut.

Si les 5 tentatives échouent, Azure Functions ajoute un message à une file d’attente de stockage nommée *webjobs-blobtrigger-poison*. Vous pouvez configurer le nombre maximal de tentatives. Le paramètre MaxDequeueCount est utilisé à la fois pour la gestion des objets blob incohérents et pour l’administration des messages de la file d’attente de messages incohérents. Le message en file d’attente associé aux objets blob incohérents correspond à un objet JSON, qui contient les propriétés suivantes :

* FunctionId (au format *&lt;nom de l’application de fonctions>* .Functions. *&lt;nom de la fonction>* )
* BlobType (« BlockBlob » ou « PageBlob »)
* ContainerName
* BlobName
* Étiquette d’entité (identificateur de version de l’objet blob, par exemple : « 0x8D1DC6E70A277EF »)

## <a name="concurrency-and-memory-usage"></a>Concurrence et utilisation de la mémoire

Le déclencheur de blob utilise une file d’attente en interne. Le nombre maximal d’appels de fonction concurrents est par conséquent contrôlé par la [configuration des files d’attente dans host.json](functions-host-json.md#queues). Les paramètres par défaut limitent la concurrence à 24 appels. Cette limite s’applique séparément à chaque fonction qui utilise un déclencheur de blob.

[Le plan Consommation](functions-scale.md#how-the-consumption-and-premium-plans-work) limite une application de fonction sur une machine virtuelle à 1,5 Go de mémoire. La mémoire est utilisée par chaque instance de la fonction qui s’exécutent simultanément et par le runtime de fonctions lui-même. Si une fonction déclenchée par blob charge le blob entier en mémoire, la mémoire maximale utilisée par cette fonction uniquement pour les blobs est 24 * la taille maximale du blob. Par exemple, une application de fonction avec trois fonctions déclenchées par blob et les paramètres par défaut aurait une concurrence par machine virtuelle maximale de 3 * 24 = 72 appels de fonction.

Les fonctions JavaScript et Java chargent l’objet blob entier en mémoire et les fonctions C# le font si vous faites la liaison avec `string` ou `Byte[]`.

## <a name="next-steps"></a>Étapes suivantes

- [Lire des données de stockage de blobs lors de l’exécution d’une fonction](./functions-bindings-storage-blob-input.md)
- [Écrire des données de stockage d’objets blob à partir d’une fonction](./functions-bindings-storage-blob-output.md)
