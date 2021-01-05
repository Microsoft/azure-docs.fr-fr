---
title: Informations de référence pour les développeurs sur Azure Functions
description: Découvrez comment développer des fonctions à l’aide de PowerShell.
author: eamonoreilly
ms.topic: conceptual
ms.custom: devx-track-dotnet, devx-track-azurepowershell
ms.date: 04/22/2019
ms.openlocfilehash: af9490433c344c712da55e9b29bf9df364380736
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93422533"
---
# <a name="azure-functions-powershell-developer-guide"></a>Guide des développeurs PowerShell sur Azure Functions

Cet article fournit des détails sur la méthode d’écriture d’Azure Functions à l’aide de PowerShell.

Une Azure Function PowerShell (fonction) est représentée comme un script PowerShell qui s’exécute lorsqu’elle est déclenchée. Chaque script de fonction inclut un fichier `function.json` qui définit le comportement de la fonction, comme sa méthode de déclenchement et ses paramètres d’entrée et de sortie. Pour en savoir plus, consultez l’[article sur les déclencheurs et les liaisons](functions-triggers-bindings.md). 

Comme d’autres types de fonctions, les fonctions de script PowerShell utilisent des paramètres correspondant aux noms de toutes les liaisons d’entrée définies dans le fichier `function.json`. Un paramètre `TriggerMetadata` contenant des informations supplémentaires sur le déclencheur qui a démarré la fonction est également transmis.

Cet article suppose que vous ayez déjà lu l’article [Informations de référence pour les développeurs sur Azure Functions](functions-reference.md). Vous devez également avoir effectué le [démarrage rapide de Functions pour PowerShell](./create-first-function-vs-code-powershell.md) pour créer votre première fonction PowerShell.

## <a name="folder-structure"></a>Structure de dossiers

La structure de dossiers requise pour un projet PowerShell est similaire à la structure ci-après. Cette valeur par défaut peut être modifiée. Pour plus d’informations, consultez la section [scriptFile](#configure-function-scriptfile)ci-dessous.

```
PSFunctionApp
 | - MyFirstFunction
 | | - run.ps1
 | | - function.json
 | - MySecondFunction
 | | - run.ps1
 | | - function.json
 | - Modules
 | | - myFirstHelperModule
 | | | - myFirstHelperModule.psd1
 | | | - myFirstHelperModule.psm1
 | | - mySecondHelperModule
 | | | - mySecondHelperModule.psd1
 | | | - mySecondHelperModule.psm1
 | - local.settings.json
 | - host.json
 | - requirements.psd1
 | - profile.ps1
 | - extensions.csproj
 | - bin
```

À la racine du projet se trouve un fichier [`host.json`](functions-host-json.md) partagé que vous pouvez utiliser pour configurer l’application de fonction. Chaque fonction a un dossier avec ses propres fichier de code (.ps1) et fichier de configuration de liaison (`function.json`). Le nom du répertoire parent du fichier function.json est toujours le nom de votre fonction.

Certaines liaisons nécessitent la présence d’un fichier `extensions.csproj`. Les extensions de liaison, nécessaires dans la [version 2.x et les versions ultérieures](functions-versions.md) du runtime Functions, sont définies dans le fichier `extensions.csproj`, les fichiers bibliothèque proprement dits se trouvant dans le dossier `bin`. Quand vous développez localement, vous devez [inscrire les extensions de liaison](functions-bindings-register.md#extension-bundles). Quand vous développez des fonctions dans le portail Azure, cet enregistrement est effectué pour vous.

Dans une Function App PowerShell, vous pouvez éventuellement disposer d’un `profile.ps1` qui s’exécute au démarrage d’une application de fonction (sinon, c’est ce qu’on appelle un *[démarrage à froid](#cold-start)* ). Pour plus d’informations, consultez [Profil PowerShell](#powershell-profile).

## <a name="defining-a-powershell-script-as-a-function"></a>Définition d’un script PowerShell en tant que fonction

Par défaut, le runtime Functions recherche votre fonction dans `run.ps1`, où `run.ps1` partage le même répertoire parent en tant que `function.json` correspondant.

Votre script doit valider un certain nombre d’arguments à l’exécution. Pour gérer ces paramètres, ajoutez un bloc `param` en haut de votre script, comme dans l’exemple suivant :

```powershell
# $TriggerMetadata is optional here. If you don't need it, you can safely remove it from the param block
param($MyFirstInputBinding, $MySecondInputBinding, $TriggerMetadata)
```

### <a name="triggermetadata-parameter"></a>Paramètre TriggerMetadata

Le paramètre `TriggerMetadata` est utilisé pour fournir des informations supplémentaires sur le déclencheur. Les métadonnées supplémentaires varient d’une liaison à une autre, mais elles contiennent toutes une propriété `sys` incluant les données suivantes :

```powershell
$TriggerMetadata.sys
```

| Propriété   | Description                                     | Type     |
|------------|-------------------------------------------------|----------|
| UtcNow     | Moment, au format UTC, où fonction a été déclenchée        | DateTime |
| MethodName | Nom de la fonction qui a été déclenchée     | string   |
| RandGuid   | GUID unique pour cette exécution de la fonction | string   |

Chaque type de déclencheur a un ensemble de métadonnées différent. Par exemple, `$TriggerMetadata` pour `QueueTrigger` contient, entre autres, `InsertionTime`, `Id`, `DequeueCount`. Pour plus d’informations sur les métadonnées du déclencheur de file d’attente, accédez à la [documentation officielle des déclencheurs de file d’attente](functions-bindings-storage-queue-trigger.md#message-metadata). Consultez la documentation sur le [déclencheur](functions-triggers-bindings.md) que vous utilisez pour savoir ce que les métadonnées de déclencheur incluent.

## <a name="bindings"></a>Liaisons

Dans PowerShell, les [liaisons](functions-triggers-bindings.md) sont configurées et définies dans le fichier function.json d’une fonction. Les fonctions interagissent avec des liaisons de plusieurs façons.

### <a name="reading-trigger-and-input-data"></a>Lecture du déclencheur et données d’entrée

Le déclencheur et les liaisons d’entrée sont lus sous forme de paramètres transmis à votre fonction. `direction` est défini sur `in` dans le fichier function.json des liaisons d’entrée. La propriété `name` définie dans `function.json` correspond au nom du paramètre, dans le bloc `param`. Étant donné que PowerShell utilise des paramètres nommés pour la liaison, l’ordre des paramètres importe peu. Toutefois, il est recommandé de suivre l’ordre des liaisons défini dans `function.json`.

```powershell
param($MyFirstInputBinding, $MySecondInputBinding)
```

### <a name="writing-output-data"></a>Écriture des données de sortie

Dans Functions, `direction` est défini sur `out` dans le fichier function.json d’une liaison de sortie. Vous pouvez écrire dans une liaison de sortie à l’aide de la cmdlet `Push-OutputBinding`, qui est disponible pour le runtime Functions. Dans tous les cas, la propriété `name` de la liaison, telle que définie dans `function.json`, correspond au paramètre `Name` de la cmdlet `Push-OutputBinding`.

L’exemple suivant montre comment appeler `Push-OutputBinding` dans votre script de fonction :

```powershell
param($MyFirstInputBinding, $MySecondInputBinding)

Push-OutputBinding -Name myQueue -Value $myValue
```

Vous pouvez également transmettre une valeur pour une liaison spécifique via le pipeline.

```powershell
param($MyFirstInputBinding, $MySecondInputBinding)

Produce-MyOutputValue | Push-OutputBinding -Name myQueue
```

`Push-OutputBinding` se comporte différemment selon la valeur spécifiée pour `-Name` :

* Lorsque le nom spécifié ne peut pas être résolu en liaison de sortie valide, une erreur est levée.

* Lorsque la liaison de sortie accepte une collection de valeurs, vous pouvez appeler `Push-OutputBinding` à plusieurs reprises pour envoyer (push) plusieurs valeurs.

* Lorsque la liaison de sortie n’accepte qu’une valeur singleton, un second appel de `Push-OutputBinding` renvoie une erreur.

#### <a name="push-outputbinding-syntax"></a>Syntaxe de `Push-OutputBinding`

Les éléments suivants sont des paramètres valides pour appeler `Push-OutputBinding` :

| Nom | Type | Position | Description |
| ---- | ---- |  -------- | ----------- |
| **`-Name`** | String | 1 | Nom de la liaison de sortie que vous souhaitez définir. |
| **`-Value`** | Object | 2 | Valeur de la liaison de sortie vous souhaitez définir, qui est acceptée par la valeur ByValue du pipeline. |
| **`-Clobber`** | SwitchParameter | named | (Facultatif) Si spécifié, force la définition de la valeur pour une liaison de sortie spécifiée. | 

Les paramètres communs suivants sont également pris en charge : 
* `Verbose`
* `Debug`
* `ErrorAction`
* `ErrorVariable`
* `WarningAction`
* `WarningVariable`
* `OutBuffer`
* `PipelineVariable`
* `OutVariable` 

Pour plus d’informations, consultez [À propos des paramètres communs](/powershell/module/microsoft.powershell.core/about/about_commonparameters).

#### <a name="push-outputbinding-example-http-responses"></a>Exemple de Push-OutputBinding : Réponses HTTP

Un déclencheur HTTP retourne une réponse à l’aide d’une liaison de sortie nommée `response`. Dans l’exemple suivant, la liaison de sortie `response` a la valeur « output #1 » :

```powershell
PS >Push-OutputBinding -Name response -Value ([HttpResponseContext]@{
    StatusCode = [System.Net.HttpStatusCode]::OK
    Body = "output #1"
})
```

Étant donné que la sortie utilise le protocole HTTP, qui n’accepte qu’une valeur singleton, une erreur est levée lorsque `Push-OutputBinding` est appelé une seconde fois.

```powershell
PS >Push-OutputBinding -Name response -Value ([HttpResponseContext]@{
    StatusCode = [System.Net.HttpStatusCode]::OK
    Body = "output #2"
})
```

Pour les sorties qui n’acceptent que les valeurs singleton, vous pouvez utiliser le paramètre `-Clobber` pour remplacer l’ancienne valeur au lieu de tenter de l’ajouter à une collection. L’exemple suivant suppose que vous avez déjà ajouté une valeur. Grâce à `-Clobber`, la réponse de l’exemple suivant remplace la valeur existante pour retourner la valeur « output #3 » :

```powershell
PS >Push-OutputBinding -Name response -Value ([HttpResponseContext]@{
    StatusCode = [System.Net.HttpStatusCode]::OK
    Body = "output #3"
}) -Clobber
```

#### <a name="push-outputbinding-example-queue-output-binding"></a>Exemple de Push-OutputBinding : Liaison de sortie de file d’attente

`Push-OutputBinding` est utilisé pour envoyer des données aux liaisons de sortie, comme une [liaison de sortie de Stockage File d’attente Azure](functions-bindings-storage-queue-output.md). Dans l’exemple suivant, le message écrit dans la file d’attente a la valeur de « output #1 » :

```powershell
PS >Push-OutputBinding -Name outQueue -Value "output #1"
```

La liaison de sortie d’une file d’attente Stockage accepte plusieurs valeurs de sortie. Dans ce cas, l’appel de l’exemple suivant après la première écrit dans la file d’attente une liste de deux éléments : « output #1 » et « output #2 ».

```powershell
PS >Push-OutputBinding -Name outQueue -Value "output #2"
```

L’exemple suivant, lors d’un appel après les deux précédents, ajoute deux valeurs supplémentaires à la collection de sortie :

```powershell
PS >Push-OutputBinding -Name outQueue -Value @("output #3", "output #4")
```

Lors de l’écriture dans la file d’attente, le message contient ces quatre valeurs : « output #1 », « output #2 », « output #3 » et « output #4 ».

#### <a name="get-outputbinding-cmdlet"></a>Cmdlet `Get-OutputBinding`

Vous pouvez utiliser la cmdlet `Get-OutputBinding` pour récupérer les valeurs actuellement définies pour vos liaisons de sortie. Cette cmdlet récupère une table de hachage qui contient les noms des liaisons de sortie avec leurs valeurs respectives. 

Voici un exemple d’utilisation de `Get-OutputBinding` pour retourner des valeurs de liaisons actuelles :

```powershell
Get-OutputBinding
```

```Output
Name                           Value
----                           -----
MyQueue                        myData
MyOtherQueue                   myData
```

`Get-OutputBinding` contient également un paramètre nommé `-Name`, qui peut être utilisé pour filtrer la liaison retournée, comme dans l’exemple suivant :

```powershell
Get-OutputBinding -Name MyQ*
```

```Output
Name                           Value
----                           -----
MyQueue                        myData
```

Les caractères génériques (*) sont pris en charge dans `Get-OutputBinding`.

## <a name="logging"></a>Journalisation

La journalisation dans des fonctions PowerShell fonctionne comme la journalisation PowerShell standard. Vous pouvez utiliser les cmdlets de journalisation pour écrire dans chaque flux de sortie. Chaque cmdlet est mappée à un niveau de journal utilisé par Functions.

| Niveau de journalisation de Functions | Cmdlet de journalisation |
| ------------- | -------------- |
| Error | **`Write-Error`** |
| Avertissement | **`Write-Warning`**  | 
| Information | **`Write-Information`** <br/> **`Write-Host`** <br /> **`Write-Output`**      | Information | Écrit dans la journalisation du niveau d’ _information_. |
| Débogage | **`Write-Debug`** |
| Trace | **`Write-Progress`** <br /> **`Write-Verbose`** |

En plus de ces cmdlets, tout de qui est écrit dans le pipeline est redirigé vers le niveau de journal `Information` et affiché avec la mise en forme PowerShell par défaut.

> [!IMPORTANT]
> L’utilisation des cmdlets `Write-Verbose` ou `Write-Debug` ne suffit pas pour voir la journalisation commentée et de niveau de débogage. Vous devez également configurer le seuil du niveau de journal, qui déclare le niveau de journaux que vous souhaitez. Pour en savoir plus, consultez [Configurer le niveau de journal de Function App](#configure-the-function-app-log-level).

### <a name="configure-the-function-app-log-level"></a>Configurer le niveau de journal de Function App

Azure Functions vous permet de définir le niveau de seuil pour contrôler plus facilement comment Azure Functions écrit dans les journaux. Pour définir le seuil de toutes les traces écrites dans la console, utilisez la propriété `logging.logLevel.default` dans le [`host.json` fichier][informations de référence sur host.json]. Ce paramètre s’applique à toutes les fonctions dans votre Function App.

L’exemple suivant définit le seuil d’activation de la journalisation commentée pour toutes les fonctions, mais définit le seuil d’activation la journalisation du débogage pour une fonction nommée `MyFunction` :

```json
{
    "logging": {
        "logLevel": {
            "Function.MyFunction": "Debug",
            "default": "Trace"
        }
    }
}  
```

Pour plus d’informations, consultez l’article de référence sur [Informations de référence sur host.json].

### <a name="viewing-the-logs"></a>Affichage des journaux

Si votre Function App s’exécute dans Azure, vous pouvez utiliser Application Insights pour la surveiller. Consultez [Supervision des fonctions Azure](functions-monitoring.md) pour en savoir plus sur l’affichage et l’interrogation des journaux d’activité de fonction.

Si vous exécutez votre Function App localement pour le développement, elle consigne par défaut dans le système de fichiers. Pour afficher les journaux dans la console, définissez la variable d’environnement `AZURE_FUNCTIONS_ENVIRONMENT` sur `Development` avant de démarrer Function App.

## <a name="triggers-and-bindings-types"></a>Types de déclencheurs et liaisons

Vous pouvez utiliser différents déclencheurs et liaisons avec votre Function App. La liste complète des déclencheurs et liaisons est disponible [ici](functions-triggers-bindings.md#supported-bindings).

Tous les déclencheurs et liaisons sont représentés dans le code par quelques types de données réels :

* Hashtable
* string
* byte[]
* int
* double
* HttpRequestContext
* HttpResponseContext

Les cinq premiers types de cette liste sont des types .NET standard. Les deux derniers sont utilisés par le [déclencheur HttpTrigger](#http-triggers-and-bindings) uniquement.

Chaque paramètre de liaison dans vos fonctions doit être de l’un de ces types.

### <a name="http-triggers-and-bindings"></a>Déclencheurs et liaisons HTTP

Les déclencheurs HTTP et webhook ainsi que les liaisons de sortie HTTP utilisent les objets de requête et de réponse pour représenter la messagerie HTTP.

#### <a name="request-object"></a>Objet Requête

L’objet de requête qui est transmis dans le script est de type `HttpRequestContext`, qui inclut les propriétés suivantes :

| Propriété  | Description                                                    | Type                      |
|-----------|----------------------------------------------------------------|---------------------------|
| **`Body`**    | Objet qui contient le corps de la demande. `Body` est sérialisé dans le meilleur type en fonction des données. Par exemple, si les données sont au format JSON, il est transmis sous forme de table de hachage. Si les données sont au format chaîne, il est transmis sous forme de chaîne. | object |
| **`Headers`** | Dictionnaire qui contient les en-têtes de requête.                | Dictionnaire<chaîne,chaîne><sup>*</sup> |
| **`Method`** | Méthode HTTP de la demande.                                | string                    |
| **`Params`**  | Objet qui contient les paramètres de routage de la demande. | Dictionnaire<chaîne,chaîne><sup>*</sup> |
| **`Query`** | Objet qui contient les paramètres de la requête.                  | Dictionnaire<chaîne,chaîne><sup>*</sup> |
| **`Url`** | URL de la demande.                                        | string                    |

<sup>*</sup> Toutes les clés `Dictionary<string,string>` sont insensibles à la casse.

#### <a name="response-object"></a>Objet Réponse

L’objet de réponse qui doit être retourné est de type `HttpResponseContext`, qui inclut les propriétés suivantes :

| Propriété      | Description                                                 | Type                      |
|---------------|-------------------------------------------------------------|---------------------------|
| **`Body`**  | Objet qui contient le corps de la réponse.           | object                    |
| **`ContentType`** | Abréviation définissant le type de contenu pour la réponse. | string                    |
| **`Headers`** | Objet qui contient les en-têtes de la réponse.               | Dictionnaire ou table de hachage   |
| **`StatusCode`**  | Code d’état HTTP de la réponse.                       | chaîne ou entier             |

#### <a name="accessing-the-request-and-response"></a>Accès à la demande et à la réponse

Lorsque vous utilisez des déclencheurs HTTP, vous pouvez accéder à la requête HTTP comme vous le feriez avec toute autre liaison d’entrée. Elle se trouve dans le bloc `param`.

Utilisez un objet `HttpResponseContext` pour retourner une réponse, comme indiqué dans l’exemple suivant :

`function.json`

```json
{
  "bindings": [
    {
      "type": "httpTrigger",
      "direction": "in",
      "authLevel": "anonymous"
    },
    {
      "type": "http",
      "direction": "out"
    }
  ]
}
```

`run.ps1`

```powershell
param($req, $TriggerMetadata)

$name = $req.Query.Name

Push-OutputBinding -Name res -Value ([HttpResponseContext]@{
    StatusCode = [System.Net.HttpStatusCode]::OK
    Body = "Hello $name!"
})
```

Le résultat de l’appel de cette fonction est alors :

```
PS > irm http://localhost:5001?Name=Functions
Hello Functions!
```

### <a name="type-casting-for-triggers-and-bindings"></a>Forçage de type pour les déclencheurs et liaisons

Pour certaines liaisons, comme la liaison d’objet blob, vous pouvez spécifier le type du paramètre.

Par exemple, pour que les données du stockage d’objets blob soient fournies sous forme de chaîne, ajoutez le forçage de type suivant au bloc `param` :

```powershell
param([string] $myBlob)
```

## <a name="powershell-profile"></a>Profil PowerShell

PowerShell inclut le concept de profil PowerShell. Si vous ne connaissez pas les profils PowerShell, consultez [À propos des profils](/powershell/module/microsoft.powershell.core/about/about_profiles).

Dans les fonctions PowerShell, le script de profil est exécuté une fois par instance worker PowerShell dans l’application, lors du premier déploiement et après une inactivité ([démarrage à froid](#cold-start)). Lorsque la concurrence est activée en définissant la valeur [PSWorkerInProcConcurrencyUpperBound](#concurrency), le script de profil est exécuté pour chaque instance d’exécution créée.

Lorsque vous créez une Function App à l’aide d’outils, tels que Visual Studio Code et Azure Functions Core Tools, une valeur `profile.ps1` par défaut est créée pour vous. Le profil par défaut est conservé [dans le référentiel Core Tools GitHub](https://github.com/Azure/azure-functions-core-tools/blob/dev/src/Azure.Functions.Cli/StaticResources/profile.ps1) et contient :

* Authentification MSI automatique sur Azure.
* La possibilité d’activer des alias Azure PowerShell `AzureRM` PowerShell si vous le souhaitez.

## <a name="powershell-versions"></a>Versions de PowerShell

Le tableau suivant répertorie les versions de PowerShell disponibles pour chaque version majeure du runtime Functions, ainsi que la version .NET requise :

| Version de Functions | Version de PowerShell                               | Version de .NET  | 
|-------------------|--------------------------------------------------|---------------|
| 3.x (version recommandée) | PowerShell 7 (version recommandée)<br/>PowerShell Core 6 | .NET Core 3.1<br/>.NET Core 2.1 |
| 2.x               | PowerShell Core 6                                | .NET Core 2.2 |

Vous pouvez afficher la version en imprimant `$PSVersionTable` à partir de n’importe quelle fonction.

### <a name="running-local-on-a-specific-version"></a>Exécution locale sur une version spécifique

Lors d’une exécution locale, le runtime Azure Functions utilise par défaut PowerShell Core 6. Pour utiliser PowerShell 7, vous devez ajouter le paramètre `"FUNCTIONS_WORKER_RUNTIME_VERSION" : "~7"` au tableau `Values` du fichier local.setting.json, situé à la racine du projet. Lors d’une exécution locale sur PowerShell 7, votre fichier local.settings.json ressemble à l’exemple suivant : 

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "",
    "FUNCTIONS_WORKER_RUNTIME": "powershell",
    "FUNCTIONS_WORKER_RUNTIME_VERSION" : "~7"
  }
}
```

### <a name="changing-the-powershell-version"></a>Changement de version de PowerShell

Votre application de fonction doit être exécutée sur la version 3.x pour permettre la mise à niveau de PowerShell Core 6 vers PowerShell 7. Pour savoir comment procéder, consultez [Afficher et mettre à jour la version actuelle du runtime](set-runtime-version.md#view-and-update-the-current-runtime-version).

Procédez comme suit pour modifier la version de PowerShell utilisée par votre application de fonction. Pour ce faire, vous pouvez utiliser le portail Azure ou PowerShell.

# <a name="portal"></a>[Portail](#tab/portal)

1. Dans le [portail Azure](https://portal.azure.com), accédez à une application de fonction.

1. Sous **Paramètres** , sélectionnez **Configuration**. Dans l’onglet **Paramètres généraux** , recherchez la **version de PowerShell**. 

    :::image type="content" source="media/functions-reference-powershell/change-powershell-version-portal.png" alt-text="Choisir la version de PowerShell utilisée par l’application de fonction"::: 

1. Choisissez la version de **PowerShell Core** souhaitée, puis sélectionnez **Enregistrer**. Lorsque vous êtes averti du redémarrage en attente, sélectionnez **Continuer**. L’application de fonction redémarre sur la version de PowerShell choisie. 

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Exécutez le script suivant pour modifier la version de PowerShell : 

```powershell
Set-AzResource -ResourceId "/subscriptions/<SUBSCRIPTION_ID>/resourceGroups/<RESOURCE_GROUP>/providers/Microsoft.Web/sites/<FUNCTION_APP>/config/web" -Properties @{  powerShellVersion  = '<VERSION>' } -Force -UsePatchSemantics

```

Remplacez `<SUBSCRIPTION_ID>`, `<RESOURCE_GROUP>`et `<FUNCTION_APP>` par l’ID de votre abonnement Azure, le nom de votre groupe de ressources et de votre application de fonction, respectivement.  Remplacez également `<VERSION>` par `~6` ou `~7`. Vous pouvez vérifier la valeur mise à jour du paramètre `powerShellVersion` dans `Properties` de la table de hachage retournée. 

---

L’application de fonction redémarre une fois la modification apportée à la configuration.

## <a name="dependency-management"></a>Gestion des dépendances

Azure Functions vous permet de tirer parti de [PowerShell Gallery](https://www.powershellgallery.com) pour la gestion des dépendances. Une fois la gestion des dépendances activée, le fichier requirements.psd1 est utilisé pour télécharger automatiquement les modules requis. Pour activer ce comportement, affectez à la propriété `managedDependency` la valeur `true` à la racine du fichier [host.json](functions-host-json.md), comme dans l’exemple suivant :

```json
{
  "managedDependency": {
          "enabled": true
       }
}
```

Lorsque vous créez un projet PowerShell Functions, la gestion des dépendances est activée par défaut, avec le module Azure [`Az`](/powershell/azure/new-azureps-module-az) inclus. Le nombre maximal de modules actuellement pris en charge est de 10. La syntaxe prise en charge est _`MajorNumber`_ `.*` ou une version exacte du module, comme indiqué dans le fichier d’exemple requirements.psd1 :

```powershell
@{
    Az = '1.*'
    SqlServer = '21.1.18147'
}
```

Lorsque vous mettez à jour le fichier requirements.psd1, les modules mis à jour sont installés après un redémarrage.

> [!NOTE]
> Les dépendances managées nécessitent un accès à www.powershellgallery.com pour télécharger les modules. Lors de l’exécution locale, assurez-vous que le runtime peut accéder à cette URL en ajoutant les règles de pare-feu requises.

> [!NOTE]
> Actuellement, les dépendances managées ne prennent pas en charge les modules qui nécessitent l’acceptation d’une licence par l’utilisateur, soit en acceptant la licence de manière interactive, soit en fournissant un `-AcceptLicense` commutateur lors de l’appel de `Install-Module`.

Vous pouvez utiliser les paramètres d’application suivants pour changer la façon de télécharger et d’installer les dépendances managées. La mise à niveau de votre application démarre dans `MDMaxBackgroundUpgradePeriod`, puis le processus de mise à niveau se termine globalement dans `MDNewSnapshotCheckPeriod`.

| Paramètre d’application de fonction              | Valeur par défaut             | Description                                         |
|   -----------------------------   |   -------------------     |  -----------------------------------------------    |
| **`MDMaxBackgroundUpgradePeriod`**      | `7.00:00:00` (7 jours)     | Chaque rôle de travail PowerShell lance la vérification des mises à niveau de module dans PowerShell Gallery au démarrage du processus et à chaque `MDMaxBackgroundUpgradePeriod` par la suite. Quand une nouvelle version de module est disponible dans PowerShell Gallery, elle est installée dans le système de fichiers et mise à la disposition des rôles de travail PowerShell. Diminuer cette valeur permet à votre application de fonction d’obtenir plus rapidement les versions de module les plus récentes, mais cela augmente aussi l’utilisation des ressources d’application (E/S réseau, processeur, stockage). Augmenter cette valeur permet de diminuer l’utilisation des ressources d’application, mais retarde aussi la remise des nouvelles versions de module à votre application. | 
| **`MDNewSnapshotCheckPeriod`**         | `01:00:00` (1 heure)       | Lorsque les nouvelles versions de module sont installées dans le système de fichiers, chaque rôle de travail PowerShell doit être redémarré. Le redémarrage des rôles de travail PowerShell affecte la disponibilité de votre application, car il peut interrompre l’exécution de la fonction actuelle. Tant que tous les rôles de travail PowerShell n’ont pas redémarré, les appels de fonction peuvent utiliser les anciennes ou nouvelles versions du module. Le redémarrage de tous les rôles de travail PowerShell se termine dans `MDNewSnapshotCheckPeriod`. L’augmentation de cette valeur diminue la fréquence des interruptions, mais peut également augmenter la durée pendant laquelle les appels de fonction utilisent les anciennes ou nouvelles version de module de manière non déterministe. |
| **`MDMinBackgroundUpgradePeriod`**      | `1.00:00:00` (1 jour)     | Pour éviter des mises à niveau de module excessives lors des fréquents redémarrages des rôles de travail, la vérification des mises à niveau de module n’est pas effectuée si un rôle de travail l’a déjà lancée pendant la dernière période `MDMinBackgroundUpgradePeriod`. |

L’utilisation de vos propres modules personnalisés est légèrement différente de l’utilisation normale.

Le module est installé sur votre ordinateur local, dans un des dossiers globalement disponibles dans votre `$env:PSModulePath`. Lors de l’exécution dans Azure, vous n’avez pas accès aux modules installés sur votre machine. Cela signifie que le `$env:PSModulePath` pour une application de fonction PowerShell doit être différent de `$env:PSModulePath` dans un script PowerShell classique.

Dans Functions, `PSModulePath` contient deux chemins d’accès :

* Un dossier `Modules` à la racine de votre application de fonction.
* Un chemin d’accès à un dossier `Modules` contrôlé par le rôle de travail du langage PowerShell.


### <a name="function-app-level-modules-folder"></a>Dossier `Modules` de niveau Function App

Pour utiliser des modules personnalisés vous, pouvez placer les modules dont dépendent vos fonctions dans un dossier `Modules`. Dans ce dossier, les modules sont automatiquement disponibles au runtime de Functions. Toute fonction dans Function App peut utiliser ces modules. 

> [!NOTE]
> Les modules spécifiés dans le fichier requirements.psd1 sont automatiquement téléchargés et inclus dans le chemin d’accès, de sorte que vous n’avez pas besoin de les inclure dans le dossier des modules. Ceux-ci sont stockés en local dans le dossier `$env:LOCALAPPDATA/AzureFunctions` et dans le dossier `/data/ManagedDependencies` lorsqu’ils sont exécutés dans le cloud.

Pour tirer parti de cette fonctionnalité de module personnalisé, vous devez créer un dossier `Modules` à la racine de votre Function App. Copiez les modules que vous souhaitez utiliser dans vos fonctions à cet emplacement.

```powershell
mkdir ./Modules
Copy-Item -Path /mymodules/mycustommodule -Destination ./Modules -Recurse
```

Pour un dossier `Modules`, votre application de fonction doit avoir la structure de dossiers suivante :

```
PSFunctionApp
 | - MyFunction
 | | - run.ps1
 | | - function.json
 | - Modules
 | | - MyCustomModule
 | | - MyOtherCustomModule
 | | - MySpecialModule.psm1
 | - local.settings.json
 | - host.json
 | - requirements.psd1
```

Lorsque vous démarrez votre Function App, le rôle de travail du langage PowerShell ajoute ce dossier `Modules` à `$env:PSModulePath` afin que vous puissiez vous appuyer sur le chargement automatique de module comme vous le feriez dans un script PowerShell classique.

### <a name="language-worker-level-modules-folder"></a>Dossier `Modules` de niveau rôle de travail du langage

Plusieurs modules sont couramment utilisés par le rôle de travail du langage PowerShell. Ces modules sont définis en dernière position de `PSModulePath`. 

La liste actuelle des modules est la suivante :

* [Microsoft.PowerShell.Archive](https://www.powershellgallery.com/packages/Microsoft.PowerShell.Archive) : module utilisé pour utiliser des archives, par exemple `.zip`, `.nupkg`, et d’autres encore.
* **ThreadJob** : implémentation basée sur un thread des API de travail PowerShell.

Functions utilise par défaut la version la plus récente de ces modules. Pour utiliser une version de module spécifique, veuillez la placer dans le dossier `Modules` de votre application de fonction.

## <a name="environment-variables"></a>Variables d'environnement

Dans Functions, les [paramètres de l’application](functions-app-settings.md), par exemple, les chaînes de connexion de service, sont exposées en tant que variables d’environnement pendant l’exécution. Vous pouvez accéder à ces paramètres à l’aide de `$env:NAME_OF_ENV_VAR`, comme illustré ici dans l’exemple suivant :

```powershell
param($myTimer)

Write-Host "PowerShell timer trigger function ran! $(Get-Date)"
Write-Host $env:AzureWebJobsStorage
Write-Host $env:WEBSITE_SITE_NAME
```

[!INCLUDE [Function app settings](../../includes/functions-app-settings.md)]

Lors de l’exécution en local, les paramètres de l’application sont lus à partir du fichier projet [local.settings.json](functions-run-local.md#local-settings-file).

## <a name="concurrency"></a>Accès concurrentiel

Par défaut, le runtime Functions PowerShell ne peut traiter qu’un appel de fonction à la fois. Toutefois, ce niveau de concurrence peut ne pas suffire dans les situations suivantes :

* Lorsque vous essayez de gérer un grand nombre d’appels en même temps.
* Lorsque vous disposez de fonctions qui appellent d’autres fonctions dans la même Function App.

Il existe plusieurs modèles de concurrence que vous pouvez explorer en fonction du type de charge de travail :

* Augmenter ```FUNCTIONS_WORKER_PROCESS_COUNT```. Cela permet de gérer les appels de fonction dans plusieurs processus au sein de la même instance, ce qui entraîne une certaine surcharge du processeur et de la mémoire. En général, les fonctions liées aux E/S ne subissent pas cette surcharge. Pour les fonctions liées à l’UC, l’impact peut être significatif.

* Augmenter la valeur du paramètre d’application ```PSWorkerInProcConcurrencyUpperBound```. Cela permet de créer plusieurs instances d’exécution dans le même processus, ce qui réduit considérablement la surcharge du processeur et de la mémoire.

Vous définissez ces variables d’environnement dans les [paramètres d’application](functions-app-settings.md) de votre application de fonction.

Selon votre cas d’usage, Durable Functions peut considérablement améliorer la scalabilité. Pour plus d’informations, consultez les [modèles d’application Durable Functions](./durable/durable-functions-overview.md?tabs=powershell#application-patterns).

>[!NOTE]
> Il est possible que vous receviez des avertissements de type « les requêtes sont mises en file d’attente en l’absence d’instances d’exécution disponibles ». Notez qu’il ne s’agit pas d’une erreur. Ce message vous informe que les requêtes sont en file d’attente et qu’elles seront gérées au terme des requêtes précédentes.

### <a name="considerations-for-using-concurrency"></a>Considérations relatives à l’utilisation de la concurrence

PowerShell est un langage de scripts _à un thread_ par défaut. Toutefois, la concurrence peut être ajoutée en utilisant plusieurs instances d’exécution de PowerShell dans le même processus. La quantité d’instances d’exécution créée correspond au paramètre d’application ```PSWorkerInProcConcurrencyUpperBound```. Le débit sera affecté par la quantité d’UC et de mémoire disponible dans le plan sélectionné.

Azure PowerShell utilise des contextes _au niveau du processus_ et un état pour vous libérer d’une saisie excessive. Toutefois, si vous activez la concurrence dans votre Function App et appelez des actions qui changent l’état, vous pouvez être confronté à des conditions de concurrence. Ces conditions de concurrence sont difficiles à déboguer car un appel s’appuie sur un état donné et que l’autre appel a changé l’état.

La concurrence est très importante dans Azure PowerShell car certaines opérations peuvent prendre beaucoup de temps. Faites toutefois preuve de vigilance. Si vous pensez que vous êtes confronté à une condition de concurrence, définissez le paramètre d’application PSWorkerInProcConcurrencyUpperBound sur `1` et utilisez plutôt l’[isolation du niveau processus Worker du langage](functions-app-settings.md#functions_worker_process_count) pour la concurrence.

## <a name="configure-function-scriptfile"></a>Configurer une fonction `scriptFile`

Par défaut, une fonction PowerShell est exécutée à partir de `run.ps1`, fichier qui partage le même répertoire parent que son `function.json` correspondant.

La propriété `scriptFile` dans `function.json` peut être utilisée pour obtenir une structure de dossiers semblable à l’exemple suivant :

```
FunctionApp
 | - host.json
 | - myFunction
 | | - function.json
 | - lib
 | | - PSFunction.ps1
```

Dans ce cas, le `function.json` pour `myFunction` inclut une propriété `scriptFile` faisant référence au fichier contenant la fonction exportée à exécuter.

```json
{
  "scriptFile": "../lib/PSFunction.ps1",
  "bindings": [
    // ...
  ]
}
```

## <a name="use-powershell-modules-by-configuring-an-entrypoint"></a>Utiliser des modules PowerShell en configurant un entryPoint

Cet article vous a montré les fonctions PowerShell dans le fichier de script `run.ps1` par défaut généré par les modèles.
Vous pouvez toutefois également inclure vos fonctions dans des modules PowerShell. Vous pouvez faire référence à votre code de fonction spécifique dans le module à l’aide des champs `scriptFile` et `entryPoint` dans le fichier de configuration « function.json ».

Dans ce cas, `entryPoint` est le nom d’une fonction ou d’une cmdlet dans le module PowerShell référencé dans `scriptFile`.

Prenez en compte la structure de dossiers suivante :

```
FunctionApp
 | - host.json
 | - myFunction
 | | - function.json
 | - lib
 | | - PSFunction.psm1
```

Où `PSFunction.psm1` contient :

```powershell
function Invoke-PSTestFunc {
    param($InputBinding, $TriggerMetadata)

    Push-OutputBinding -Name OutputBinding -Value "output"
}

Export-ModuleMember -Function "Invoke-PSTestFunc"
```

Dans cet exemple, la configuration de `myFunction` inclut une propriété `scriptFile` qui fait référence à `PSFunction.psm1`, un module PowerShell se trouvant dans un autre dossier.  La propriété `entryPoint` fait référence à la fonction `Invoke-PSTestFunc`, le point d’entrée dans le module.

```json
{
  "scriptFile": "../lib/PSFunction.psm1",
  "entryPoint": "Invoke-PSTestFunc",
  "bindings": [
    // ...
  ]
}
```

Avec cette configuration, le `Invoke-PSTestFunc` est exécuté exactement comme le serait un `run.ps1`.

## <a name="considerations-for-powershell-functions"></a>Considérations relatives aux fonctions PowerShell

Lorsque vous utilisez des fonctions PowerShell, tenez compte des considérations décrites dans les sections suivantes.

### <a name="cold-start"></a>Démarrage à froid

Lorsque vous développez Azure Functions dans le [modèle d’hébergement serverless](functions-scale.md#consumption-plan), les démarrages à froid sont une réalité. Le *démarrage à froid* fait référence à la période nécessaire à votre Function App pour commencer à traiter une requête. Le démarrage à froid se produit plus fréquemment dans le plan de consommation, car votre Function App s’arrête pendant les périodes d’inactivité.

### <a name="bundle-modules-instead-of-using-install-module"></a>Regrouper des modules au lieu d’utiliser `Install-Module`

Votre script est exécuté à chaque appel. Évitez d’utiliser `Install-Module` dans votre script. Utilisez plutôt `Save-Module` avant la publication pour que votre fonction ne perde pas de temps à télécharger le module. Si les démarrages à froid affectent vos fonctions, envisagez de déployer votre Function App sur un [plan App Service](functions-scale.md#app-service-plan) défini sur *always on* ou sur un [plan Premium](functions-scale.md#premium-plan).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez les ressources suivantes :

* [Meilleures pratiques pour Azure Functions](functions-best-practices.md)
* [Informations de référence pour les développeurs sur Azure Functions](functions-reference.md)
* [Azure Functions triggers and bindings (Déclencheurs et liaisons Azure Functions)](functions-triggers-bindings.md)

[référence de host.json]: functions-host-json.md