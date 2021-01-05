---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 04/14/2019
ms.author: glenga
ms.openlocfilehash: d944d1d3e9c72471fab2435430a7d13e1770e807
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96010480"
---
## <a name="local-settings-file"></a>Fichier de paramètres locaux

Le fichier local.settings.json stocke des paramètres d’application, des chaînes de connexion et des paramètres utilisés par des outils de développement locaux. Les paramètres dans le fichier local.settings.json sont uniquement utilisés lorsque vous exécuter les projets localement. Le fichier de paramètres locaux possède la structure suivante :

```json
{
  "IsEncrypted": false,
  "Values": {
    "FUNCTIONS_WORKER_RUNTIME": "<language worker>",
    "AzureWebJobsStorage": "<connection-string>",
    "AzureWebJobsDashboard": "<connection-string>",
    "MyBindingConnection": "<binding-connection-string>",
    "AzureWebJobs.HttpExample.Disabled": "true"
  },
  "Host": {
    "LocalHttpPort": 7071,
    "CORS": "*",
    "CORSCredentials": false
  },
  "ConnectionStrings": {
    "SQLConnectionString": "<sqlclient-connection-string>"
  }
}
```

Ces paramètres sont pris en charge lorsque vous exécutez des projets localement :

| Paramètre      | Description                            |
| ------------ | -------------------------------------- |
| **`IsEncrypted`** | Lorsque ce paramètre est défini sur `true`, toutes les valeurs sont chiffrées à l’aide d’une clé d’ordinateur local. Utilisé avec les commandes `func settings`. La valeur par défaut est `false`. Vous voudrez peut-être chiffrer le fichier local.settings.json sur votre ordinateur local lorsqu’il contient des secrets, tels que des chaînes de connexion de service. L’hôte déchiffre automatiquement les paramètres quand il s’exécute. Utilisez la commande `func settings decrypt` avant d’essayer de lire les paramètres chiffrés localement. |
| **`Values`** | Tableau des paramètres d’application et des chaînes de connexion utilisés lors de l’exécution locale d’un projet. Ces paires clé-valeur (chaîne-chaîne) correspondent aux paramètres d’application dans votre Function App dans Azure, tels que [`AzureWebJobsStorage`]. Plusieurs déclencheurs et liaisons ont une propriété qui fait référence à un paramètre d’application de chaîne de connexion, par exemple `Connection` pour le [déclencheur de stockage blob](../articles/azure-functions/functions-bindings-storage-blob-trigger.md#configuration). Pour ces propriétés, vous avez besoin d’un paramètre d’application défini dans le tableau `Values`. Consultez le tableau suivant pour une liste des paramètres couramment utilisés. <br/>Les valeurs doivent être des chaînes et non des objets JSON ou des tableaux. Les noms de paramètres ne peuvent pas contenir le signe deux points (`:`) ou un trait de soulignement double (`__`). Les caractères de soulignement double sont réservés par le runtime, et le signe deux-points est réservé pour la prise en charge de l’[injection de dépendances](../articles/azure-functions/functions-dotnet-dependency-injection.md#working-with-options-and-settings). |
| **`Host`** | Les paramètres de cette section personnalisent le processus hôte Functions lorsque vous exécutez localement des projets. Ces paramètres sont séparés des paramètres host.json, qui s’appliquent également lorsque vous exécutez localement des projets dans Azure. |
| **`LocalHttpPort`** | Définit le port par défaut utilisé lors de l’exécution de l’hôte Functions local (`func host start` et `func run`). `--port`L’option de ligne de commande est prioritaire sur ce paramètre. |
| **`CORS`** | Définit les origines autorisées pour [cross-origin resource sharing (CORS)](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing). Les origines sont fournies sous la forme d’une liste séparée par des virgules, sans espaces. La valeur de caractère générique (\*) est prise en charge, ce qui autorise les demandes à partir de toute origine. |
| **`CORSCredentials`** |  Lorsque qu’il est défini sur `true`, autorise `withCredentials` les requêtes. |
| **`ConnectionStrings`** | Une collection. N’utilisez pas cette collection pour les chaînes de connexion utilisées par vos liaisons de fonction. Cette collection est utilisée seulement par les infrastructures qui obtiennent généralement les chaînes de connexion à partir de la `ConnectionStrings`section d’un fichier de configuration, comme [Entity Framework](/ef/ef6/). Les chaînes de connexion dans cet objet sont ajoutées à l’environnement avec le type de fournisseur [System.Data.SqlClient](/dotnet/api/system.data.sqlclient). Les éléments de cette collection ne sont pas publiés sur Azure avec d’autres paramètres d’application. Vous devez explicitement ajouter ces valeurs à la collection `Connection strings` des paramètres de Function App. Si vous créez un [`SqlConnection`](/dotnet/api/system.data.sqlclient.sqlconnection) dans votre code de fonction, vous devez stocker la valeur de la chaîne de connexion et vos autres connexions dans **Paramètres d’application** dans le portail. |

Les paramètres d’application suivants peuvent être inclus dans le tableau **`Values`** lors d’une exécution locale :

| Paramètre | Valeurs | Description |
|-----|-----|-----|
|**`AzureWebJobsStorage`**| Chaîne de connexion de compte de stockage ou<br/>`UseDevelopmentStorage=true`| Contient la chaîne de connexion pour un compte de stockage Azure. Obligatoire lors de l’utilisation de déclencheurs autres que HTTP. Pour plus d’informations, consultez les informations de référence sur [`AzureWebJobsStorage`].<br/>Lorsque vous installez l’[émulateur de stockage Azure](../articles/storage/common/storage-use-emulator.md) localement et définissez [`AzureWebJobsStorage`] sur `UseDevelopmentStorage=true`, Core Tools utilise l’émulateur. L’émulateur est utile lors du développement, mais vous devez le tester avec une connexion de stockage réelle avant le déploiement.| 
|**`AzureWebJobs.<FUNCTION_NAME>.Disabled`**| `true`\|`false` | Pour désactiver une fonction qui s’exécute localement, ajoutez `"AzureWebJobs.<FUNCTION_NAME>.Disabled": "true"` à la collection, où `<FUNCTION_NAME>` est le nom de la fonction. Pour en savoir plus, consultez [Guide pratique pour désactiver des fonctions dans Azure Functions](../articles/azure-functions/disable-function.md#localsettingsjson) |
|**`FUNCTIONS_WORKER_RUNTIME`** | `dotnet`<br/>`node`<br/>`java`<br/>`powershell`<br/>`python`| Indique le langage ciblé du runtime Functions. Obligatoire pour la version 2.x et ultérieure du runtime Functions. Ce paramètre est généré pour votre projet par Core Tools. Pour plus d’informations, consultez les informations de référence sur [`FUNCTIONS_WORKER_RUNTIME`](../articles/azure-functions/functions-app-settings.md#functions_worker_runtime).|
| **`FUNCTIONS_WORKER_RUNTIME_VERSION`** | `~7` |Indique que PowerShell 7 doit être utilisé lors d’une exécution locale. S’il n’est pas défini, PowerShell Core 6 est utilisé. Ce paramètre est utilisé seulement lors d’une exécution locale. Lors d’une exécution dans Azure, la version du runtime PowerShell est déterminée par le paramètre de configuration du site `powerShellVersion`, qui peut être [défini dans le portail](../articles/azure-functions/functions-reference-powershell.md#changing-the-powershell-version). | 

[AzureWebJobsStorage]: ../articles/azure-functions/functions-app-settings.md#azurewebjobsstorage