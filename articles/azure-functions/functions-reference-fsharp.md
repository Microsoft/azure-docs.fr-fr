---
title: Informations de référence sur Azure Functions pour les développeurs F#
description: Découvrez comment développer sur Azure Functions à l’aide de scripts F#.
author: sylvanc
ms.assetid: e60226e5-2630-41d7-9e5b-9f9e5acc8e50
ms.topic: conceptual
ms.date: 10/09/2018
ms.author: syclebsc
ms.openlocfilehash: f9b7b92fd21e12f1d86c5d5878e48c6ec6b0e748
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87088017"
---
# <a name="azure-functions-f-developer-reference"></a>Informations de référence pour les développeurs F# sur Azure Functions

F# for Azure Functions est une solution conçue pour exécuter facilement de petits morceaux de code, ou « fonctions », dans le cloud. Les données circulent dans votre fonction F# par le biais d’arguments de fonction. Les noms d’argument sont spécifiés dans `function.json`, et il existe des noms prédéfinis pour accéder à des éléments tels que l’enregistreur de fonctions et les jetons d’annulation. 

>[!IMPORTANT]
>Le script F# (.fsx) est pris en charge uniquement par la [version 1.x](functions-versions.md#creating-1x-apps) du runtime Azure Functions. Si vous voulez utiliser F# avec la version 2.x et ultérieure du runtime, vous devez utiliser un projet de bibliothèque de classes F# précompilé (.fs). Vous créez, gérez et publiez un projet de bibliothèque de classes F# en utilisant Visual Studio comme vous le feriez pour un [projet de bibliothèque de classes C# ](functions-dotnet-class-library.md). Pour plus d’informations sur les versions de Functions, consultez [Vue d’ensemble des versions du runtime Azure Functions](functions-versions.md).

Cet article suppose que vous ayez déjà lu l’article [Informations de référence pour les développeurs sur Azure Functions](functions-reference.md).

## <a name="how-fsx-works"></a>Fonctionnement de .fsx
Un fichier `.fsx` est un script F#. Il peut être considéré comme un projet F# contenu dans un seul fichier. Ce fichier contient à la fois le code de votre programme (dans ce cas précis, votre fonction Azure) et des directives concernant la gestion des dépendances.

Lorsque vous utilisez un fichier `.fsx` pour une fonction Azure, les assemblys généralement requis sont automatiquement inclus à votre intention, ce qui vous permet de vous concentrer sur la fonction plutôt que sur le code « réutilisable ».

## <a name="folder-structure"></a>Structure de dossiers

La structure de dossiers pour un projet de script F# est similaire à la structure ci-après :

```
FunctionsProject
 | - MyFirstFunction
 | | - run.fsx
 | | - function.json
 | | - function.proj
 | - MySecondFunction
 | | - run.fsx
 | | - function.json
 | | - function.proj
 | - host.json
 | - extensions.csproj
 | - bin
```

Il existe un fichier [host.json](functions-host-json.md) partagé que vous pouvez utiliser pour configurer l’application de fonction. Chaque fonction a son propre fichier de code (.fsx) et un fichier de configuration de liaison (function.json).

Les extensions de liaison nécessaires dans la [version 2.x et ultérieure](functions-versions.md) du runtime Functions sont définies dans le fichier `extensions.csproj`, les fichiers de bibliothèque proprement dits se trouvant dans le dossier `bin`. Quand vous développez localement, vous devez [inscrire les extensions de liaison](./functions-bindings-register.md#extension-bundles). Quand vous développez des fonctions dans le portail Azure, cet enregistrement est effectué pour vous.

## <a name="binding-to-arguments"></a>Liaison aux arguments
Chaque liaison prend en charge un ensemble spécifique d’arguments, comme décrit en détail dans [Informations de référence pour les développeurs sur les déclencheurs et liaisons Azure Functions](functions-triggers-bindings.md). Par exemple, l’une des liaisons d’argument prises en charge par un déclencheur d’objet blob est un objet CLR traditionnel (POCO), qui peut être exprimé à l’aide d’un enregistrement F#. Par exemple :

```fsharp
type Item = { Id: string }

let Run(blob: string, output: byref<Item>) =
    let item = { Id = "Some ID" }
    output <- item
```

Votre fonction Azure F# utilisera un ou plusieurs arguments. Quand nous parlons d’arguments Azure Functions, nous faisons référence à des arguments *d’entrée* et à des arguments de *sortie*. Un argument d’entrée est exactement ce que son nom laisse entendre : une entrée pour votre fonction Azure F#. Un argument de *sortie* correspond à des données mutables ou à un argument `byref<>` permettant de retransmettre des données *hors* de votre fonction.

Dans l’exemple ci-dessus, `blob` est un argument d’entrée, tandis que `output` est un argument de sortie. Notez que nous avons utilisé le type `byref<>` pour l’argument `output` (il n’est pas nécessaire d’ajouter l’annotation `[<Out>]`). L’utilisation d’un type `byref<>` permet à votre fonction de modifier l’enregistrement ou l’objet auxquels l’argument fait référence.

Lorsqu’un enregistrement F# est utilisé comme type d’entrée, la définition d’enregistrement doit être indiquée par `[<CLIMutable>]` pour permettre à l’infrastructure Azure Functions de définir les champs de manière appropriée avant de transmettre l’enregistrement à votre fonction. En arrière-plan, `[<CLIMutable>]` génère des méthodes setter pour les propriétés d’enregistrement. Par exemple :

```fsharp
[<CLIMutable>]
type TestObject =
    { SenderName : string
      Greeting : string }

let Run(req: TestObject, log: ILogger) =
    { req with Greeting = sprintf "Hello, %s" req.SenderName }
```

Une classe F# est également utilisable pour les arguments d’entrée et de sortie. Pour une classe, les propriétés nécessitent généralement des méthodes getter et setter. Par exemple :

```fsharp
type Item() =
    member val Id = "" with get,set
    member val Text = "" with get,set

let Run(input: string, item: byref<Item>) =
    let result = Item(Id = input, Text = "Hello from F#!")
    item <- result
```

## <a name="logging"></a>Journalisation
Pour consigner la sortie dans vos [journaux d’activité de streaming](../app-service/troubleshoot-diagnostic-logs.md) en F#, votre fonction doit utiliser un argument de type [ILogger](/dotnet/api/microsoft.extensions.logging.ilogger). Par souci de cohérence, nous vous recommandons de nommer cet argument `log`. Par exemple :

```fsharp
let Run(blob: string, output: byref<string>, log: ILogger) =
    log.LogInformation(sprintf "F# Azure Function processed a blob: %s" blob)
    output <- input
```

## <a name="async"></a>Async
Il est possible d’utiliser le workflow `async`, mais le résultat doit renvoyer un élément `Task`. Pour ce faire, utilisez `Async.StartAsTask`. Par exemple :

```fsharp
let Run(req: HttpRequestMessage) =
    async {
        return new HttpResponseMessage(HttpStatusCode.OK)
    } |> Async.StartAsTask
```

## <a name="cancellation-token"></a>Jeton d’annulation
Si votre fonction doit gérer les arrêts de manière appropriée, vous pouvez lui fournir un argument [`CancellationToken`](/dotnet/api/system.threading.cancellationtoken) . conjointement utilisable avec `async`. Par exemple :

```fsharp
let Run(req: HttpRequestMessage, token: CancellationToken)
    let f = async {
        do! Async.Sleep(10)
        return new HttpResponseMessage(HttpStatusCode.OK)
    }
    Async.StartAsTask(f, token)
```

## <a name="importing-namespaces"></a>Importation des espaces de noms
Les espaces de noms peuvent être ouverts de la manière habituelle :

```fsharp
open System.Net
open System.Threading.Tasks
open Microsoft.Extensions.Logging

let Run(req: HttpRequestMessage, log: ILogger) =
    ...
```

Les espaces de noms ci-après sont ouverts automatiquement :

* `System`
* `System.Collections.Generic`
* `System.IO`
* `System.Linq`
* `System.Net.Http`
* `System.Threading.Tasks`
* `Microsoft.Azure.WebJobs`
* `Microsoft.Azure.WebJobs.Host`.

## <a name="referencing-external-assemblies"></a>Référencement des assemblys externes
De même, les références d’assembly d’infrastructure peuvent être ajoutées avec la directive `#r "AssemblyName"`.

```fsharp
#r "System.Web.Http"

open System.Net
open System.Net.Http
open System.Threading.Tasks
open Microsoft.Extensions.Logging

let Run(req: HttpRequestMessage, log: ILogger) =
    ...
```

Les assemblys suivants sont ajoutés automatiquement par l’environnement hébergeant Azure Functions :

* `mscorlib`
* `System`
* `System.Core`
* `System.Xml`
* `System.Net.Http`
* `Microsoft.Azure.WebJobs`
* `Microsoft.Azure.WebJobs.Host`
* `Microsoft.Azure.WebJobs.Extensions`
* `System.Web.Http`
* `System.Net.Http.Formatting`.

En outre, les assemblys ci-après ont une casse spécifique et peuvent être référencés par leur nom simple (par exemple, `#r "AssemblyName"`) :

* `Newtonsoft.Json`
* `Microsoft.WindowsAzure.Storage`
* `Microsoft.ServiceBus`
* `Microsoft.AspNet.WebHooks.Receivers`
* `Microsoft.AspNEt.WebHooks.Common`.

Si vous avez besoin de référencer un assembly privé, vous pouvez charger le fichier d’assembly dans un dossier `bin` relatif à votre fonction et le référencer à l’aide du nom de fichier (par exemple, `#r "MyAssembly.dll"`). Pour plus d’informations sur le téléchargement de fichiers vers votre conteneur de fonctions, consultez la section suivante sur la gestion des packages.

## <a name="editor-prelude"></a>Préambule destiné à l’éditeur
Un éditeur prenant en charge F# Compiler Services ne reconnaît pas les espaces de noms et les assemblys automatiquement inclus par Azure Functions. Il peut donc être utile d’insérer un préambule conçu pour permettre à l’éditeur de localiser les assemblys que vous utilisez et d’ouvrir explicitement les espaces de noms. Par exemple :

```fsharp
#if !COMPILED
#I "../../bin/Binaries/WebJobs.Script.Host"
#r "Microsoft.Azure.WebJobs.Host.dll"
#endif

open System
open Microsoft.Azure.WebJobs.Host
open Microsoft.Extensions.Logging

let Run(blob: string, output: byref<string>, log: ILogger) =
    ...
```

Lorsque la solution Azure Functions exécute votre code, elle traite la source avec l’élément `COMPILED` défini, de sorte que le préambule destiné à l’éditeur sera ignoré.

<a name="package"></a>

## <a name="package-management"></a>Gestion des packages
Pour utiliser des packages NuGet dans une fonction F#, ajoutez un fichier `project.json` au dossier de la fonction dans le système de fichiers de l’application de fonctions. Voici un exemple de fichier `project.json` ajoutant une référence de package NuGet à `Microsoft.ProjectOxford.Face` version 1.1.0 :

```json
{
  "frameworks": {
    "net46":{
      "dependencies": {
        "Microsoft.ProjectOxford.Face": "1.1.0"
      }
    }
   }
}
```

Seul .NET Framework 4.6 est pris en charge. Par conséquent, assurez-vous que votre fichier `project.json` spécifie `net46` comme indiqué ici.

Lorsque vous chargez un fichier `project.json` , le runtime obtient les packages et ajoute automatiquement des références aux assemblys de packages. Vous n’avez pas besoin d’ajouter de directives `#r "AssemblyName"` . Il vous suffit d’ajouter les instructions `open` requises à votre fichier `.fsx`.

Si vous le souhaitez, vous pouvez placer automatiquement les assemblys de référence dans votre préambule destiné à l’éditeur, afin d’améliorer l’interaction de votre éditeur avec F# Compile Services.

### <a name="how-to-add-a-projectjson-file-to-your-azure-function"></a>Ajout d’un fichier `project.json` à votre fonction Azure
1. Commencez par vous assurer que votre conteneur de fonctions est en cours d’exécution. Ce que vous pouvez faire en ouvrant votre fonction dans le portail Azure. Il donne également accès aux journaux d’activité de diffusion en continu où le résultat de l’installation du package s’affiche.
2. Pour charger un fichier `project.json` , utilisez l’une des méthodes décrites dans l’article [Comment mettre à jour les fichiers du conteneur de fonctions](functions-reference.md#fileupdate). Si vous utilisez l’article [Déploiement continu pour Azure Functions](functions-continuous-deployment.md), vous pouvez ajouter un fichier `project.json` à votre branche intermédiaire afin de le tester avant de l’ajouter à votre branche de déploiement.
3. Une fois le fichier `project.json` ajouté, une sortie semblable à l’exemple ci-après apparaîtra dans le journal de diffusion en continu de votre fonction :

```
2016-04-04T19:02:48.745 Restoring packages.
2016-04-04T19:02:48.745 Starting NuGet restore
2016-04-04T19:02:50.183 MSBuild auto-detection: using msbuild version '14.0' from 'D:\Program Files (x86)\MSBuild\14.0\bin'.
2016-04-04T19:02:50.261 Feeds used:
2016-04-04T19:02:50.261 C:\DWASFiles\Sites\facavalfunctest\LocalAppData\NuGet\Cache
2016-04-04T19:02:50.261 https://api.nuget.org/v3/index.json
2016-04-04T19:02:50.261
2016-04-04T19:02:50.511 Restoring packages for D:\home\site\wwwroot\HttpTriggerCSharp1\Project.json...
2016-04-04T19:02:52.800 Installing Newtonsoft.Json 6.0.8.
2016-04-04T19:02:52.800 Installing Microsoft.ProjectOxford.Face 1.1.0.
2016-04-04T19:02:57.095 All packages are compatible with .NETFramework,Version=v4.6.
2016-04-04T19:02:57.189
2016-04-04T19:02:57.189
2016-04-04T19:02:57.455 Packages restored.
```

## <a name="environment-variables"></a>Variables d'environnement
Pour obtenir une variable d’environnement ou une valeur de paramètre d’application, utilisez `System.Environment.GetEnvironmentVariable`. Par exemple :

```fsharp
open System.Environment
open Microsoft.Extensions.Logging

let Run(timer: TimerInfo, log: ILogger) =
    log.LogInformation("Storage = " + GetEnvironmentVariable("AzureWebJobsStorage"))
    log.LogInformation("Site = " + GetEnvironmentVariable("WEBSITE_SITE_NAME"))
```

## <a name="reusing-fsx-code"></a>Réutilisation du code .fsx
Vous pouvez utiliser le code d’autres fichiers `.fsx` au moyen d’une directive `#load`. Par exemple :

`run.fsx`

```fsharp
#load "logger.fsx"

let Run(timer: TimerInfo, log: ILogger) =
    mylog log (sprintf "Timer: %s" DateTime.Now.ToString())
```

`logger.fsx`

```fsharp
let mylog(log: ILogger, text: string) =
    log.LogInformation(text);
```

Les chemins d’accès fournis à la directive `#load` sont relatifs à l’emplacement de votre fichier `.fsx`.

* `#load "logger.fsx"` charge un fichier situé dans le dossier de la fonction.
* `#load "package\logger.fsx"` charge un fichier situé dans le dossier `package` du dossier de la fonction.
* `#load "..\shared\mylogger.fsx"` charge un fichier situé dans le dossier `shared` au même niveau que le dossier de la fonction, c’est-à-dire, directement sous `wwwroot`.

La directive `#load` ne fonctionne qu’avec des fichiers `.fsx` (script F#), et non avec des fichiers `.fs`.

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations, consultez les ressources suivantes :

* [F# Guide](/dotnet/articles/fsharp/index)
* [Meilleures pratiques pour Azure Functions](functions-best-practices.md)
* [Informations de référence pour les développeurs sur Azure Functions](functions-reference.md)
* [Azure Functions triggers and bindings (Déclencheurs et liaisons Azure Functions)](functions-triggers-bindings.md)
* [Test des fonctions Azure](functions-test-a-function.md)
* [Mise à l’échelle d’Azure Functions](functions-scale.md)
