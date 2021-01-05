---
title: Informations de référence sur les paramètres d’application d’Azure Functions
description: Documentation de référence pour les paramètres d’application ou les variables d’environnement d’Azure Functions.
ms.topic: conceptual
ms.date: 09/22/2018
ms.openlocfilehash: 2b71bee620ab7d5b1ef98b60013d1978f49d127f
ms.sourcegitcommit: 4bee52a3601b226cfc4e6eac71c1cb3b4b0eafe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94505884"
---
# <a name="app-settings-reference-for-azure-functions"></a>Informations de référence sur les paramètres d’application d’Azure Functions

Les paramètres d’une application de fonction contiennent les options de configuration globale qui affectent l’ensemble des fonctions de cette application de fonction. Lors d’une exécution locale, ces paramètres sont accessibles en tant que [variables d’environnement](functions-run-local.md#local-settings-file) locales. Cet article répertorie les paramètres d’application qui sont disponibles dans les applications de fonction.

[!INCLUDE [Function app settings](../../includes/functions-app-settings.md)]

Les fichiers [host.json](functions-host-json.md) et [local.settings.json](functions-run-local.md#local-settings-file) contiennent d’autres options de configuration globale.

> [!NOTE]  
> Vous pouvez utiliser des paramètres d’application pour remplacer les valeurs de paramètres host.json sans avoir à modifier le fichier host.json proprement dit. C’est utile dans des scénarios où vous devez configurer ou modifier des paramètres host.json spécifiques pour un environnement spécifique. Cela vous permet également de modifier les paramètres host.json sans avoir à republier votre projet. Pour plus d’informations, consultez l’[article de référence host.json](functions-host-json.md#override-hostjson-values).  

## <a name="appinsights_instrumentationkey"></a>APPINSIGHTS_INSTRUMENTATIONKEY

Clé d’instrumentation pour Application Insights. Utilisez `APPINSIGHTS_INSTRUMENTATIONKEY` ou `APPLICATIONINSIGHTS_CONNECTION_STRING`. Lorsque Application Insights s’exécute dans un cloud souverain, utilisez `APPLICATIONINSIGHTS_CONNECTION_STRING`. Pour plus d’informations, consultez [Comment configurer la surveillance de Azure Functions](configure-monitoring.md). 

|Clé|Exemple de valeur|
|---|------------|
|APPINSIGHTS_INSTRUMENTATIONKEY|55555555-af77-484b-9032-64f83bb83bb|

## <a name="applicationinsights_connection_string"></a>APPLICATIONINSIGHTS_CONNECTION_STRING

Chaîne de connexion pour Application Insights. Utilisez `APPLICATIONINSIGHTS_CONNECTION_STRING` au lieu de `APPINSIGHTS_INSTRUMENTATIONKEY` dans les cas suivants :

+ lorsque votre application de fonction requiert les personnalisations ajoutées prises en charge à l’aide de la chaîne de connexion. 
+ Lorsque votre instance d’Application Insights s’exécute dans un cloud souverain, qui requiert un point de terminaison personnalisé.

Pour plus d’informations, consultez [Chaînes de connexion](../azure-monitor/app/sdk-connection-string.md). 

|Clé|Exemple de valeur|
|---|------------|
|APPLICATIONINSIGHTS_CONNECTION_STRING|InstrumentationKey=[key];IngestionEndpoint=[url];LiveEndpoint=[url];ProfilerEndpoint=[url];SnapshotEndpoint=[url];|

## <a name="azure_function_proxy_disable_local_call"></a>AZURE_FUNCTION_PROXY_DISABLE_LOCAL_CALL

Par défaut, les [proxies Functions](functions-proxies.md) utilisent un raccourci pour envoyer des appels d’API à partir de proxies directement vers les fonctions de la même application de fonction. Ce raccourci est utilisé au lieu de créer une nouvelle requête HTTP. Ce paramètre vous permet de désactiver le comportement de ce raccourci.

|Clé|Valeur|Description|
|-|-|-|
|AZURE_FUNCTION_PROXY_DISABLE_LOCAL_CALL|true|Les appels dont l’URL de back-end pointe vers une fonction de l’application de fonction locale ne seront pas envoyés directement vers la fonction. Au lieu de cela, les requêtes sont retournées au serveur frontal HTTP pour l’application de fonction.|
|AZURE_FUNCTION_PROXY_DISABLE_LOCAL_CALL|false|Les appels dont l’URL de back-end pointe vers une fonction de l’application de fonction locale sont transférés directement vers la fonction. Il s’agit de la valeur par défaut. |

## <a name="azure_function_proxy_backend_url_decode_slashes"></a>AZURE_FUNCTION_PROXY_BACKEND_URL_DECODE_SLASHES

Ce paramètre vérifie si les caractères `%2F` sont décodés en tant que barres obliques dans les paramètres d’itinéraire lorsqu’ils sont réinsérés dans l’URL principale. 

|Clé|Valeur|Description|
|-|-|-|
|AZURE_FUNCTION_PROXY_BACKEND_URL_DECODE_SLASHES|true|Les paramètres d’itinéraire avec des barres obliques encodées sont décodés. |
|AZURE_FUNCTION_PROXY_BACKEND_URL_DECODE_SLASHES|false|Tous les paramètres d’itinéraire sont transmis tels quels, ce qui correspond au comportement par défaut. |

Par exemple, considérez le fichier proxies.json pour une application de fonction au niveau du domaine `myfunction.com`.

```JSON
{
    "$schema": "http://json.schemastore.org/proxies",
    "proxies": {
        "root": {
            "matchCondition": {
                "route": "/{*all}"
            },
            "backendUri": "example.com/{all}"
        }
    }
}
```

Lorsque `AZURE_FUNCTION_PROXY_BACKEND_URL_DECODE_SLASHES` a la valeur `true`, l’URL `example.com/api%2ftest` correspond à `example.com/api/test`. Par défaut, l’URL reste inchangée en tant que `example.com/test%2fapi`. Pour plus d’informations, consultez [Proxys de fonctions](functions-proxies.md).

## <a name="azure_functions_environment"></a>AZURE_FUNCTIONS_ENVIRONMENT

Dans la version 2.x et les versions ultérieures du runtime Functions, configure le comportement de l’application en fonction de l’environnement d’exécution. Cette valeur est [lue lors de l’initialisation](https://github.com/Azure/azure-functions-host/blob/dev/src/WebJobs.Script.WebHost/Program.cs#L43). Vous pouvez définir `AZURE_FUNCTIONS_ENVIRONMENT` sur n’importe quelle valeur, mais [trois valeurs](/dotnet/api/microsoft.aspnetcore.hosting.environmentname) sont prises en charge : [Développement](/dotnet/api/microsoft.aspnetcore.hosting.environmentname.development), [Préproduction](/dotnet/api/microsoft.aspnetcore.hosting.environmentname.staging) et [Production](/dotnet/api/microsoft.aspnetcore.hosting.environmentname.production). Lorsque `AZURE_FUNCTIONS_ENVIRONMENT` n’est pas défini, la valeur `Development` par défaut est sur un environnement local et `Production` sur Azure. Ce paramètre doit être utilisé à la place de `ASPNETCORE_ENVIRONMENT` pour définir l’environnement d’exécution. 

## <a name="azurefunctionsjobhost__"></a>AzureFunctionsJobHost__\*

Dans la version 2.x et les versions ultérieures du runtime Functions, les paramètres d’application peuvent substituer les paramètres de [host.json](functions-host-json.md) dans l’environnement actuel. Ces remplacements sont exprimés sous la forme de paramètres d’application nommés `AzureFunctionsJobHost__path__to__setting`. Pour plus d’informations, consultez [Substituer les valeurs de host.json](functions-host-json.md#override-hostjson-values).

## <a name="azurewebjobsdashboard"></a>AzureWebJobsDashboard

Chaîne de connexion du compte de stockage facultatif pour stocker des journaux d’activité et les afficher dans l’onglet **Surveiller** du portail. Ce paramètre n’est valide que pour les applications qui ciblent la version 1.x du runtime Azure Functions. Le compte de stockage doit être à usage général. Il prend en charge les objets blob, les files d’attente et les tables. Pour plus d’informations, consultez [Exigences pour le compte de stockage](storage-considerations.md#storage-account-requirements).

|Clé|Exemple de valeur|
|---|------------|
|AzureWebJobsDashboard|DefaultEndpointsProtocol=https;AccountName=<name>;AccountKey=<key>|

> [!NOTE]
> Pour de meilleures performances et une meilleure expérience, le runtime version 2.x et les versions ultérieures utilisent APPINSIGHTS_INSTRUMENTATIONKEY et App Insights pour la supervision au lieu de `AzureWebJobsDashboard`.

## <a name="azurewebjobsdisablehomepage"></a>AzureWebJobsDisableHomepage

La valeur `true` désigne la désactivation de la page d’arrivée par défaut qui s’affiche pour l’URL racine d’une application de fonction. La valeur par défaut est `false`.

|Clé|Exemple de valeur|
|---|------------|
|AzureWebJobsDisableHomepage|true|

Lorsque ce paramètre d’application est omis ou défini sur `false`, une page semblable à celle indiquée dans l’exemple suivant s’affiche en réponse à l’URL `<functionappname>.azurewebsites.net`.

![Page d’arrivée d’une application de fonction](media/functions-app-settings/function-app-landing-page.png)

## <a name="azurewebjobsdotnetreleasecompilation"></a>AzureWebJobsDotNetReleaseCompilation

La valeur `true` désigne l’utilisation du mode Mise en production lors de la compilation de code .NET ; la valeur `false` désigne l’utilisation du mode Débogage. La valeur par défaut est `true`.

|Clé|Exemple de valeur|
|---|------------|
|AzureWebJobsDotNetReleaseCompilation|true|

## <a name="azurewebjobsfeatureflags"></a>AzureWebJobsFeatureFlags

Liste délimitée par des virgules des fonctionnalités bêta à activer. Les fonctionnalités bêta activées par ces indicateurs ne sont pas prêtes pour la production, mais peuvent être activées pour une utilisation expérimentale avant leur mise en service.

|Clé|Exemple de valeur|
|---|------------|
|AzureWebJobsFeatureFlags|feature1,feature2|

## <a name="azurewebjobssecretstoragetype"></a>AzureWebJobsSecretStorageType

Spécifie le référentiel ou le fournisseur à utiliser pour le stockage de clés. Actuellement, les référentiels pris en charge sont le stockage d’objets blob (« Blob ») et le système de fichiers local (« Fichiers »). La valeur par défaut est blob dans la version 2 et le système de fichiers dans la version 1.

|Clé|Exemple de valeur|
|---|------------|
|AzureWebJobsSecretStorageType|Fichiers|

## <a name="azurewebjobsstorage"></a>AzureWebJobsStorage

Le runtime Azure Functions utilise cette chaîne de connexion de compte de stockage pour fonctionner normalement. Ce compte de stockage est notamment utilisé pour la gestion des clés, la gestion des déclencheurs de minuterie et les points de contrôle Event Hubs. Le compte de stockage doit être à usage général. Il prend en charge les objets blob, les files d’attente et les tables. Consultez les sections [Compte de stockage](functions-infrastructure-as-code.md#storage-account) et [Conditions requises pour le compte de stockage](storage-considerations.md#storage-account-requirements).

|Clé|Exemple de valeur|
|---|------------|
|AzureWebJobsStorage|DefaultEndpointsProtocol=https;AccountName=[name];AccountKey=[key]|

## <a name="azurewebjobs_typescriptpath"></a>AzureWebJobs_TypeScriptPath

Chemin d’accès au compilateur utilisé pour TypeScript. Vous permet d’écraser la valeur par défaut au besoin.

|Clé|Exemple de valeur|
|---|------------|
|AzureWebJobs_TypeScriptPath|%HOME%\typescript|

## <a name="function_app_edit_mode"></a>FUNCTION\_APP\_EDIT\_MODE

Détermine si la modification est activée dans le portail Azure. Les valeurs valides sont « readwrite » et « readonly ».

|Clé|Exemple de valeur|
|---|------------|
|FUNCTION\_APP\_EDIT\_MODE|readonly|

## <a name="functions_extension_version"></a>FUNCTIONS\_EXTENSION\_VERSION

Version du runtime Azure Functions à utiliser dans cette application de fonction. Un tilde accompagné d’une version principale désigne l’utilisation de la version la plus récente de cette version principale (par exemple, « ~2 »). Lorsque de nouvelles versions sont disponibles pour une même version principale, elles sont automatiquement installées dans l’application de fonction. Pour épingler l’application à une version spécifique, utilisez le numéro de version complet (par exemple, « 2.0.12345 »). La valeur par défaut est « ~2 ». La valeur `~1` épingle votre application à la version 1.x du runtime.

|Clé|Exemple de valeur|
|---|------------|
|FUNCTIONS\_EXTENSION\_VERSION|~2|

## <a name="functions_v2_compatibility_mode"></a>FUNCTIONS\_V2\_COMPATIBILITY\_MODE

Ce paramètre permet à votre application de fonction de s’exécuter dans un mode compatible avec la version 2.x sur le runtime version 3.x. Utilisez ce paramètre uniquement si vous rencontrez des problèmes lors de la [mise à niveau de votre application de fonction de la version 2.x à la version 3.x du runtime](functions-versions.md#migrating-from-2x-to-3x). 

>[!IMPORTANT]
> Ce paramètre doit être considéré uniquement comme une solution de contournement à court terme pendant que vous mettez à jour votre application pour qu’elle s’exécute correctement sur la version 3.x. Ce paramètre est pris en charge tant que le [runtime 2.x est pris en charge](functions-versions.md). Si vous rencontrez des problèmes qui empêchent votre application de s’exécuter sur la version 3.x sans utiliser ce paramètre, veuillez [signaler votre problème](https://github.com/Azure/azure-functions-host/issues/new?template=Bug_report.md).

Nécessite que [FUNCTIONS\_EXTENSION\_VERSION](functions-app-settings.md#functions_extension_version) ait la valeur `~3`.

|Clé|Exemple de valeur|
|---|------------|
|FUNCTIONS\_V2\_COMPATIBILITY\_MODE|true|

## <a name="functions_worker_process_count"></a>FUNCTIONS\_WORKER\_PROCESS\_COUNT

Spécifie le nombre maximal de processus de traitement de langue, avec la valeur `1` par défaut. La valeur maximale autorisée est `10`. Les appels de fonction sont répartis uniformément entre les processus de travail de langage. Les processus de travail de langage sont générés toutes les 10 secondes jusqu’à ce que le nombre défini par \_WORKER\_PROCESS\_COUNT soit atteint. L’utilisation de plusieurs processus de traitement de langue n’a pas le même effet qu’une [mise à l’échelle](functions-scale.md). Envisagez d’utiliser ce paramètre lorsque votre charge de travail a une combinaison d’appels liés à l’UC et aux E/S. Ce paramètre s’applique à toutes les langues non-.NET.

|Clé|Exemple de valeur|
|---|------------|
|FUNCTIONS\_WORKER\_PROCESS\_COUNT|2|


## <a name="functions_worker_runtime"></a>FUNCTIONS\_WORKER\_RUNTIME

Runtime du rôle de travail de langage à charger dans l’application de fonction.  Correspond au langage utilisé dans votre application (par exemple, « dotnet »). Pour les fonctions dans plusieurs langages, vous devrez les publier dans plusieurs applications, chacune avec une valeur de runtime de travail correspondante.  Les valeurs valides sont `dotnet` (C#/F#), `node` (JavaScript/TypeScript), `java` (Java), `powershell` (PowerShell) et `python` (Python).

|Clé|Exemple de valeur|
|---|------------|
|FUNCTIONS\_WORKER\_RUNTIME|dotnet|

## <a name="pip_extra_index_url"></a>PIP\_EXTRA\_INDEX\_URL

La valeur de ce paramètre indique une URL d’index des packages personnalisée pour les applications Python. Utilisez ce paramètre lorsque vous devez exécuter une build distante à l’aide de dépendances personnalisées se trouvant dans un index de packages supplémentaire.   

|Clé|Exemple de valeur|
|---|------------|
|PIP\_EXTRA\_INDEX\_URL|http://my.custom.package.repo/simple |

Pour en savoir plus, consultez [Dépendances personnalisées](functions-reference-python.md#remote-build-with-extra-index-url) dans les informations de référence pour les développeurs Python.

## <a name="scale_controller_logging_enable"></a>SCALE\_CONTROLLER\_LOGGING\_ENABLE

_Ce paramètre est actuellement en préversion._  

Ce paramètre contrôle la journalisation à partir du contrôleur d’échelle Azure Functions. Pour plus d’informations, consultez [Journaux du contrôleur d’échelle](functions-monitoring.md#scale-controller-logs).

|Clé|Exemple de valeur|
|-|-|
|SCALE_CONTROLLER_LOGGING_ENABLE|AppInsights:Verbose|

La valeur de cette clé est fournie au format `<DESTINATION>:<VERBOSITY>`, qui est défini comme suit :

[!INCLUDE [functions-scale-controller-logging](../../includes/functions-scale-controller-logging.md)]

## <a name="website_contentazurefileconnectionstring"></a>WEBSITE\_CONTENTAZUREFILECONNECTIONSTRING

Pour les plans Consommation et Premium uniquement. Chaîne de connexion du compte de stockage dans lequel la configuration et le code de l’application de fonction sont stockés. Consultez la section [Créer une application de fonction](functions-infrastructure-as-code.md#create-a-function-app).

|Clé|Exemple de valeur|
|---|------------|
|WEBSITE_CONTENTAZUREFILECONNECTIONSTRING|DefaultEndpointsProtocol=https;AccountName=[name];AccountKey=[key]|

## <a name="website_contentovervnet"></a>WEBSITE\_CONTENTOVERVNET

Pour les plans Premium uniquement. La valeur `1` permet à votre application de fonction de se mettre à l’échelle lorsque votre compte de stockage est limité à un réseau virtuel. Vous devez activer ce paramètre lorsque vous limitez votre compte de stockage à un réseau virtuel. Pour en savoir plus, consultez [Restreindre votre compte de stockage à un réseau virtuel](functions-networking-options.md#restrict-your-storage-account-to-a-virtual-network-preview). 

|Clé|Exemple de valeur|
|---|------------|
|WEBSITE_CONTENTOVERVNET|1|

## <a name="website_contentshare"></a>WEBSITE\_CONTENTSHARE

Pour les plans Consommation et Premium uniquement. Chemin d’accès au fichier contenant la configuration et le code de l’application de fonction. Utilisé avec WEBSITE_CONTENTAZUREFILECONNECTIONSTRING. La valeur par défaut est une chaîne unique qui commence par le nom de l’application de fonction. Consultez la section [Créer une application de fonction](functions-infrastructure-as-code.md#create-a-function-app).

|Clé|Exemple de valeur|
|---|------------|
|WEBSITE_CONTENTSHARE|functionapp091999e2|

## <a name="website_max_dynamic_application_scale_out"></a>WEBSITE\_MAX\_DYNAMIC\_APPLICATION\_SCALE\_OUT

Nombre maximal d’instances possibles vers lequel l’application de fonction peut effectuer un scale-out. Par défaut, il n’y pas de limite.

> [!IMPORTANT]
> Ce paramètre est en préversion.  Une [propriété d’application pour un scale-out maximal](./functions-scale.md#limit-scale-out) a été ajoutée et constitue la méthode recommandée pour limiter le scale-out.

|Clé|Exemple de valeur|
|---|------------|
|WEBSITE\_MAX\_DYNAMIC\_APPLICATION\_SCALE\_OUT|5|

## <a name="website_node_default_version"></a>WEBSITE\_NODE\_DEFAULT_VERSION

_Windows uniquement._  
Définit la version de Node.js à utiliser lors de l’exécution de votre application de fonction sur Windows. Vous devez utiliser un tilde (~) pour que le runtime utilise la dernière version disponible de la version principale ciblée. Par exemple, lorsqu’elle est définie sur `~10`, la dernière version de Node.js 10 est utilisée. Quand une version majeure est ciblée avec un tilde, vous n’avez pas besoin de mettre à jour manuellement la version mineure. 

|Clé|Exemple de valeur|
|---|------------|
|WEBSITE\_NODE\_DEFAULT_VERSION|~10|

## <a name="website_run_from_package"></a>WEBSITE\_RUN\_FROM\_PACKAGE

Permet à votre application de fonction de s’exécuter à partir d’un fichier de package monté.

|Clé|Exemple de valeur|
|---|------------|
|WEBSITE\_RUN\_FROM\_PACKAGE|1|

Les valeurs valides sont soit une URL qui correspond à l’emplacement d’un fichier de package de déploiement, soit `1`. Lorsque la valeur `1` est définie, le package doit se trouver dans le dossier `d:\home\data\SitePackages`. Lorsque vous utilisez le déploiement zip avec ce paramètre, le package est automatiquement chargé vers cet emplacement. Dans la préversion, ce paramètre s’appelait `WEBSITE_RUN_FROM_ZIP`. Pour plus d’informations, consultez [Exécuter des fonctions Azure à partir d’un fichier de package](run-functions-from-deployment-package.md).

## <a name="website_time_zone"></a>WEBSITE\_TIME\_ZONE

Vous permet de définir le fuseau horaire de votre application de fonction. 

|Clé|Système d''exploitation|Exemple de valeur|
|---|--|------------|
|WEBSITE\_TIME\_ZONE|Windows|Heure standard de l'Est|
|WEBSITE\_TIME\_ZONE|Linux|America/New_York|

[!INCLUDE [functions-timezone](../../includes/functions-timezone.md)]

## <a name="next-steps"></a>Étapes suivantes

[Découvrez comment mettre à jour les paramètres d’application](functions-how-to-use-azure-function-app-settings.md#settings)

[Consultez les paramètres globaux dans le fichier host.json](functions-host-json.md)

[Consultez les autres paramètres des applications App Service](https://github.com/projectkudu/kudu/wiki/Configurable-settings)
