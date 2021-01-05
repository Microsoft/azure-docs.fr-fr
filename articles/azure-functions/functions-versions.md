---
title: Vue d’ensemble des versions du runtime Azure Functions
description: Azure Functions prend en charge plusieurs versions du runtime. Découvrez les différences entre elles et comment choisir celle qui vous convient.
ms.topic: conceptual
ms.custom: devx-track-dotnet
ms.date: 12/09/2019
ms.openlocfilehash: 935291c461e275902cb6905c4440fe4d289f0c16
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97653348"
---
# <a name="azure-functions-runtime-versions-overview"></a>Vue d’ensemble des versions du runtime Azure Functions

Azure Functions prend actuellement en charge trois versions de l’hôte du runtime : 1.x, 2.x et 3.x. Les trois versions sont prises en charge pour les scénarios de production.  

> [!IMPORTANT]
> La version 1.x est en mode maintenance et ne prend en charge que le développement dans le Portail Azure, dans le portail Azure Stack Hub ou localement sur des ordinateurs Windows. Les améliorations sont fournies uniquement dans les versions ultérieures. 

Cet article explique en détail certaines différences existant entre les versions, comment vous pouvez créer chaque version et comment les modifier.

## <a name="languages"></a>Languages

À compter de la version 2.x, le runtime utilise un modèle d’extensibilité de langage, et toutes les fonctions d’une application de fonction doivent partager le même langage. Le langage des fonctions d’une application de fonction est choisi au moment de la création de l’application, et est conservé dans le paramètre [FUNCTIONS\_WORKER\_RUNTIME](functions-app-settings.md#functions_worker_runtime). 

Le tableau suivant montre les langages de programmation actuellement pris en charge dans chaque version du runtime.

[!INCLUDE [functions-supported-languages](../../includes/functions-supported-languages.md)]

## <a name="run-on-a-specific-version"></a><a name="creating-1x-apps"></a>Exécuter sur une version spécifique

Par défaut, les applications de fonction créées dans le portail Azure et par l’interface Azure CLI sont configurées pour la version 3.x. Vous pouvez modifier cette version en fonction de vos besoins. Vous pouvez changer la version du runtime en 1.x seulement après avoir créé votre application de fonction, mais avant d’ajouter des fonctions.  Le passage entre 2.x et 3.x est autorisé même avec les applications qui ont des fonctions, mais il est néanmoins recommandé de tester d’abord dans une nouvelle application.

## <a name="migrating-from-1x-to-later-versions"></a>Migration de la version 1.x vers les versions ultérieures

Vous pouvez choisir de migrer une application existante écrite pour utiliser la version 1.x du runtime pour qu’elle utilise à la place une version plus récente. La plupart des modifications que vous devez apporter sont liées au runtime du langage, par exemple des modifications de l’API C# entre .NET Framework 4.7 et .NET Core. Vous devez également vous assurer que le code et les bibliothèques sont compatibles avec le runtime de langage choisi. Enfin, veillez à noter toutes les modifications soulignées ci-après qui affectent les déclencheurs, liaisons et fonctionnalités. Pour une migration plus performante, vous devez créer une application de fonction dans une nouvelle version et porter le code de la fonction existante en version 1.x vers la nouvelle application.  

Bien qu’il soit possible de procéder à une mise à niveau « sur place » en mettant à jour manuellement la configuration de l’application, passer de la version 1.x à une version ultérieure implique certains changements cassants. Par exemple, en C#, l’objet de débogage passe de `TraceWriter` à `ILogger`. Quand vous créez un projet en version 3.x, vous commencez par des fonctions mises à jour basées sur les modèles de version 3.x les plus récents.

### <a name="changes-in-triggers-and-bindings-after-version-1x"></a>Changements dans les déclencheurs et les liaisons après la version 1.x

À compter de la version 2.x, vous devez installer dans votre application les extensions pour les déclencheurs et les liaisons spécifiques utilisés par les fonctions. La seule exception concerne les déclencheurs HTTP et de la minuterie, qui ne nécessitent aucune extension.  Pour plus d’informations, voir [Inscrire et installer des extensions de liaison](./functions-bindings-register.md).

Il convient également de noter quelques modifications dans *function.json* ou dans les attributs de la fonction entre les versions. Par exemple, la propriété `path` d’Event Hub est désormais `eventHubName`. Consultez le [tableau des liaisons existantes](#bindings). Il contient des liens vers de la documentation sur chaque liaison.

### <a name="changes-in-features-and-functionality-after-version-1x"></a>Changements apportés aux fonctionnalités après la version 1.x

Quelques fonctionnalités ont été supprimées, mises à jour ou remplacées après la version 1.x. Cette section détaille les changements que vous voyez dans les versions ultérieures après avoir utilisé la version 1.x.

Les modifications suivantes ont été apportées à la version 2.x :

* Les clés permettant d’appeler les points de terminaison HTTP sont toujours stockées et chiffrées dans le Stockage Blob Azure. Dans la version 1.x, les clés étaient stockées par défaut dans le Stockage Fichier Azure. Au moment de mettre à niveau une application de la version 1.x à la version 2.x, les secrets existants qui se trouvent dans le Stockage Fichier sont réinitialisés.

* La version 2.x du runtime n’intègre aucune prise en charge des fournisseurs de webhooks. Cette modification a été apportée pour améliorer les performances. Vous pouvez continuer à utiliser des déclencheurs HTTP comme points de terminaison des webhooks.

* Le fichier de configuration d’hôte (host.json) doit être vide ou contenir la chaîne `"version": "2.0"`.

* Pour améliorer la surveillance, le tableau de bord WebJobs se trouvant dans le portail, qui a utilisé le paramètre [`AzureWebJobsDashboard`](functions-app-settings.md#azurewebjobsdashboard), est remplacé par Azure Application Insights, qui utilise le paramètre [`APPINSIGHTS_INSTRUMENTATIONKEY`](functions-app-settings.md#appinsights_instrumentationkey). Pour plus d’informations, consultez [Surveiller l’exécution des fonctions Azure](functions-monitoring.md).

* Toutes les fonctions d’une application de fonction doivent partager le même langage. Lorsque vous créez une application de fonction, vous devez choisir une pile d’exécution pour l’application. La pile d’exécution est spécifiée par la valeur [`FUNCTIONS_WORKER_RUNTIME`](functions-app-settings.md#functions_worker_runtime) dans les paramètres de l’application. Cette exigence a été ajoutée pour améliorer l’empreinte mémoire et le temps de démarrage. Lorsque vous développez en local, vous devez également inclure ce paramètre dans le [fichier local.settings.json](functions-run-local.md#local-settings-file).

* Le délai d’expiration par défaut pour les fonctions dans un plan App Service est de 30 minutes. Vous pouvez modifier ce délai d’expiration manuellement et l’indiquer à nouveau comme étant illimité en utilisant le paramètre [functionTimeout](functions-host-json.md#functiontimeout) dans host.json.

* Les limitations d’accès concurrentiel HTTP sont implémentées par défaut pour les fonctions de plan Consommation, avec une valeur par défaut de 100 requêtes simultanées par instance. Vous pouvez configurer ces limitations dans le paramètre [`maxConcurrentRequests`](functions-host-json.md#http) du fichier host.json.

* En raison des [limitations de .NET Core](https://github.com/Azure/azure-functions-host/issues/3414), la prise en charge des fonctions de script F# (.fsx) a été supprimée. Les fonctions F# compilées (.fs) restent prises en charge.

* Le format d’URL des webhooks de déclencheur Event Grid a été remplacé par `https://{app}/runtime/webhooks/{triggerName}`.

## <a name="migrating-from-2x-to-3x"></a>Migration de 2.x vers 3.x

Azure Functions version 3.x offre une compatibilité descendante forte avec la version 2.x.  De nombreuses applications doivent normalement être mise à niveau sans problème vers 3.x, sans aucune modification du code.  Si le passage à 3. x est encouragé, veillez néanmoins à effectuer des tests intensifs avant de changer la version majeure dans les applications de production.

### <a name="breaking-changes-between-2x-and-3x"></a>Changements cassants entre 2.x et 3.x

Voici les changements à prendre en considération avant de mettre à niveau une application 2.x vers 3.x.

#### <a name="javascript"></a>JavaScript

* Les liaisons de sortie affectées via `context.done` ou des valeurs de retour se comportent désormais de la même façon qu’une définition dans `context.bindings`.

* L’objet de déclencheur de minuteur est en casse mixte au lieu de la casse Pascal

* Les fonctions déclenchées par Event Hub avec un `dataType` binaire reçoivent un tableau de `binary` au lieu d’un tableau de `string`.

* La charge utile des requêtes HTTP n’est plus accessible via `context.bindingData.req`.  Elle néanmoins toujours accessible en tant que paramètre d’entrée, `context.req`, et dans `context.bindings`.

* Node.js 8 n’est plus pris en charge et ne s’exécute pas dans les fonctions 3.x.

#### <a name="net"></a>.NET

* [Les opérations de serveur synchrones sont désactivées par défaut](/dotnet/core/compatibility/2.2-3.0#http-synchronous-io-disabled-in-all-servers).

### <a name="changing-version-of-apps-in-azure"></a>Changement de la version des applications dans Azure

La version du runtime Functions utilisée par les applications publiées dans Azure dépend du paramètre d’application [`FUNCTIONS_EXTENSION_VERSION`](functions-app-settings.md#functions_extension_version). Les valeurs des versions majeures du runtime suivantes sont prises en charge :

| Valeur | Cible du runtime |
| ------ | -------- |
| `~3` | 3.x |
| `~2` | 2.x |
| `~1` | 1.x |

>[!IMPORTANT]
> Ne modifiez pas arbitrairement ce paramètre, car d’autres modifications des paramètres de l’application et d’autres modifications du code dans vos fonctions peuvent être nécessaires.

### <a name="locally-developed-application-versions"></a>Versions d’une application développée en local

Vous pouvez effectuer les mises à jour suivantes sur les applications de fonction afin de changer localement les versions ciblées.

#### <a name="visual-studio-runtime-versions"></a>Versions du runtime Visual Studio

Dans Visual Studio, vous sélectionnez la version du runtime au moment de créer un projet. Azure Functions Tools pour Visual Studio prend en charge les trois versions majeures du runtime. La version appropriée est utilisée au moment du débogage et de la publication, en fonction des paramètres de projet. Les paramètres de version sont définis dans le fichier `.csproj`, dans les propriétés suivantes :

##### <a name="version-1x"></a>Version 1.x

```xml
<TargetFramework>net472</TargetFramework>
<AzureFunctionsVersion>v1</AzureFunctionsVersion>
```

##### <a name="version-2x"></a>Version 2.x

```xml
<TargetFramework>netcoreapp2.1</TargetFramework>
<AzureFunctionsVersion>v2</AzureFunctionsVersion>
```

##### <a name="version-3x"></a>Version 3.x

```xml
<TargetFramework>netcoreapp3.1</TargetFramework>
<AzureFunctionsVersion>v3</AzureFunctionsVersion>
```

> [!NOTE]
> Azure Functions 3.x et .NET nécessitent que l’extension `Microsoft.NET.Sdk.Functions` soit au moins `3.0.0`.

###### <a name="updating-2x-apps-to-3x-in-visual-studio"></a>Mise à jour des applications 2.x vers 3.x dans Visual Studio

Vous pouvez ouvrir une fonction existante ciblant 2.x et passer à 3.x en modifiant le fichier `.csproj` et en mettant à jour les valeurs ci-dessus.  Visual Studio gère automatiquement pour vous les versions du runtime en fonction des métadonnées du projet.  C’est néanmoins possible si vous n’ayez jamais créé d’application 3.x avant que Visual Studio ne dispose des modèles et du runtime pour 3.x sur votre machine.  Ceci peut se présenter soi-même avec une erreur comme « Aucun runtime Functions disponible ne correspond à la version spécifiée dans le projet ».  Pour récupérer les modèles et le runtime les plus récents, effectuez l’expérience de création d’un projet de fonction.  Quand vous accédez à l’écran de sélection de la version et du modèle, attendez que Visual Studio termine la récupération des modèles les plus récents.  Une fois les derniers modèles .NET Core 3 disponibles et affichés, vous devez être en mesure d’exécuter et de déboguer les projets configurés pour la version 3.x.

> [!IMPORTANT]
> Les fonctions en version 3.x peuvent être développées dans Visual Studio seulement si vous utilisez Visual Studio version 16.4 ou ultérieure.

#### <a name="vs-code-and-azure-functions-core-tools"></a>VS Code et Azure Functions Core Tools

[Azure Functions Core Tools](functions-run-local.md) est utilisé pour développer la ligne de commande et également par [l’extension Azure Functions](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) pour Visual Studio Code. Pour développer pour la version 3.x, installez la version 3.x de Core Tools. Le développement en version 2.x nécessite la version 2.x de Core Tools, etc. Pour plus d’informations, voir [Installer Azure Functions Core Tools](functions-run-local.md#install-the-azure-functions-core-tools).

Pour un développement avec Visual Studio Code, vous devrez peut-être également mettre à jour le paramètre utilisateur pour `azureFunctions.projectRuntime` afin qu’il corresponde à la version des outils installés.  Ce paramètre met également à jour les modèles et les langages utilisés lors de la création de l’application de fonction.  Pour créer des applications dans `~3`, vous devez mettre à jour le paramètre utilisateur `azureFunctions.projectRuntime` sur `~3`.

![Paramètre de runtime de l’extension Azure Functions](./media/functions-versions/vs-code-version-runtime.png)

#### <a name="maven-and-java-apps"></a>Applications Maven et Java

Vous pouvez migrer des applications Java de la version 2.x vers la version 3.x en [installant la version 3.x de Core Tools](functions-run-local.md#install-the-azure-functions-core-tools) nécessaire pour une exécution locale.  Après avoir vérifié que votre application fonctionne correctement en local sur la version 3.x, mettez à jour le fichier `POM.xml` de l’application en changeant le paramètre `FUNCTIONS_EXTENSION_VERSION` en `~3`, comme dans l’exemple suivant :

```xml
<configuration>
    <resourceGroup>${functionResourceGroup}</resourceGroup>
    <appName>${functionAppName}</appName>
    <region>${functionAppRegion}</region>
    <appSettings>
        <property>
            <name>WEBSITE_RUN_FROM_PACKAGE</name>
            <value>1</value>
        </property>
        <property>
            <name>FUNCTIONS_EXTENSION_VERSION</name>
            <value>~3</value>
        </property>
    </appSettings>
</configuration>
```

## <a name="bindings"></a>Liaisons

À compter de la version 2.x, le runtime utilise un nouveau [modèle d’extensibilité de liaison](https://github.com/Azure/azure-webjobs-sdk-extensions/wiki/Binding-Extensions-Overview) qui présente les avantages suivants :

* Prise en charge pour les extensions de liaison de tiers.

* Découplage du runtime et des liaisons. Cela permet de gérer les versions des extensions de liaison et leur publication de façon indépendante. Vous pouvez, par exemple, choisir de mettre à niveau vers une version d’une extension qui s’appuie sur une version plus récente d’un kit de développement logiciel (SDK) sous-jacent.

* Un environnement d’exécution plus léger, où seules les liaisons en cours d’utilisation sont connues et chargées par le runtime.

À l’exception des déclencheurs HTTP et de la minuterie, toutes les liaisons doivent être explicitement ajoutées au projet d’application de fonction, ou enregistrées dans le portail. Pour plus d’informations, voir [Inscrire des extensions de liaison](./functions-bindings-expressions-patterns.md).

Le tableau suivant indique les liaisons prises en charge dans chaque version du runtime.

[!INCLUDE [Full bindings table](../../includes/functions-bindings.md)]

[!INCLUDE [Timeout Duration section](../../includes/functions-timeout-duration.md)]

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez les ressources suivantes :

* [Coder et tester Azure Functions localement](functions-run-local.md)
* [Procédure pour cibler des versions du runtime Azure Functions](set-runtime-version.md)
* [Notes de publication](https://github.com/Azure/azure-functions-host/releases)
