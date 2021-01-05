---
title: Ressources du développeur - Language Understanding
description: Les kits de développement logiciel (SDK), les API REST et l'interface CLI vous aident à développer des applications LUIS (Language Understanding) dans votre langage de programmation. Gérez vos ressources Azure et vos prévisions LUIS.
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: reference
ms.date: 05/19/2020
ms.custom: devx-track-js, devx-track-csharp
ms.openlocfilehash: 8641235fb754080303a9a463d0964e5655234ff2
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95024549"
---
# <a name="sdk-rest-and-cli-developer-resources-for-language-understanding-luis"></a>Ressources de développement SDK, REST et CLI pour implémenter LUIS (Language Understanding)

Les kits de développement logiciel (SDK), les API REST et l'interface CLI vous aident à développer des applications LUIS (Language Understanding) dans votre langage de programmation. Gérez vos ressources Azure et vos prévisions LUIS.

## <a name="azure-resource-management"></a>Gestion des ressources Azure

Utilisez la couche de gestion d’Azure Cognitive Services pour créer, modifier, lister et supprimer la ressource Language Understanding ou Cognitive Services.

Recherchez de la documentation de référence basée sur l’outil :

* [Azure CLI](/cli/azure/cognitiveservices#az-cognitiveservices-list)

* [Azure RM PowerShell](/powershell/module/azurerm.cognitiveservices/?view=azurermps-4.4.1#cognitive_services)


## <a name="language-understanding-authoring-and-prediction-requests"></a>Demandes de création et de prédiction Language Understanding

Le service Language Understanding est accessible à partir d’une ressource Azure que vous devez créer. Il existe deux ressources :

* Utilisez la ressource de **création** pour vous entraîner à créer, modifier, former et publier.
* Utilisez la **prédiction** pour que le runtime envoie le texte de l'utilisateur et reçoive une prédiction.

Apprenez-en davantage sur le [point de terminaison de prédiction V3](luis-migration-api-v3.md).

Utilisez l'[exemple de code Cognitive Services](https://github.com/Azure-Samples/cognitive-services-quickstart-code) pour découvrir et utiliser les tâches les plus courantes.

### <a name="rest-specifications"></a>Spécifications REST

Les [spécifications REST LUIS](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/cognitiveservices/data-plane/LUIS), ainsi que toutes les [spécifications REST Azure](https://github.com/Azure/azure-rest-api-specs), sont publiquement accessibles sur GitHub.

### <a name="rest-apis"></a>API REST

Les API de création et de point de terminaison de prédiction sont disponibles à partir d’API REST :

|Type|Version|
|--|--|
|Création|[V2](https://go.microsoft.com/fwlink/?linkid=2092087)<br>[V3 en préversion](https://westeurope.dev.cognitive.microsoft.com/docs/services/luis-programmatic-apis-v3-0-preview)|
|Prédiction|[V2](https://go.microsoft.com/fwlink/?linkid=2092356)<br>[V3](https://westcentralus.dev.cognitive.microsoft.com/docs/services/luis-endpoint-api-v3-0/)|

### <a name="rest-endpoints"></a>Points de terminaison REST

LUIS dispose actuellement de 2 types de points de terminaison :

* **création** sur le point de terminaison de formation
* **prédiction** de requête sur le point de terminaison d’exécution

|Objectif|URL|
|--|--|
|Création V2 sur le point de terminaison de formation|`https://{your-resource-name}.api.cognitive.microsoft.com/luis/api/v2.0/apps/{appID}/`|
|Création V3 sur le point de terminaison de formation|`https://{your-resource-name}.api.cognitive.microsoft.com/luis/authoring/v3.0-preview/apps/{appID}/`|
|Prédiction V2 - toutes les prédictions sur le point de terminaison d’exécution|`https://{your-resource-name}.api.cognitive.microsoft.com/luis/v2.0/apps/{appId}?q={q}[&timezoneOffset][&verbose][&spellCheck][&staging][&bing-spell-check-subscription-key][&log]`|
|Prédiction V3 - prédiction des versions sur le point de terminaison d’exécution|`https://{your-resource-name}.api.cognitive.microsoft.com/luis/prediction/v3.0/apps/{appId}/versions/{versionId}/predict?query={query}[&verbose][&log][&show-all-intents]`|
|Prédiction V3 - prédiction d’emplacement sur le point de terminaison d’exécution|`https://{your-resource-name}.api.cognitive.microsoft.com/luis/prediction/v3.0/apps/{appId}/slots/{slotName}/predict?query={query}[&verbose][&log][&show-all-intents]`|

Le tableau suivant décrit les paramètres qui apparaissent entre accolades `{}` dans le tableau précédent.

|Paramètre|Objectif|
|--|--|
|`your-resource-name`|Nom de la ressource Azure|
|`q` ou `query`|Texte de l’énoncé envoyé à partir de l’application cliente en tant que bot de conversation|
|`version`|Nom à 10 caractères de la version|
|`slot`| `production` ou `staging`|

### <a name="rest-query-string-parameters"></a>Paramètres de chaîne de requête REST

[!INCLUDE [V3 query params](./includes/v3-prediction-query-params.md)]

## <a name="app-schema"></a>Schéma d’application

Le [schéma d’application](app-schema-definition.md) est importé et exporté au format `.json` ou `.lu`.

### <a name="language-based-sdks"></a>Kits SDK basés sur le langage

|Langage |Documentation de référence|Package|Démarrages rapides|
|--|--|--|--|
|C#|[Création](/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring?view=azure-dotnet)</br>[Prédiction](/dotnet/api/microsoft.azure.cognitiveservices.language.luis.runtime?view=azure-dotnet)|[NuGet - création](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring/)<br>[NuGet - prédiction](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime/)|[Création](azure-sdk-quickstart.md?pivots=programming-language-csharp)<br>[Prédiction de requête](azure-sdk-quickstart.md?pivots=programming-language-csharp)|
|Go|[Création et prédiction](https://godoc.org/github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v2.0/luis)|[Kit SDK](https://github.com/Azure/azure-sdk-for-go/tree/master/services/cognitiveservices/v2.0/luis)||
|Java|[Création et prédiction](/java/api/overview/azure/cognitiveservices/client/languageunderstanding?view=azure-java-stable)|[Maven - création](https://search.maven.org/artifact/com.microsoft.azure.cognitiveservices/azure-cognitiveservices-luis-authoring)<br>[Maven - prédiction](https://search.maven.org/artifact/com.microsoft.azure.cognitiveservices/azure-cognitiveservices-luis-runtime)|
|JavaScript|[Création](/javascript/api/@azure/cognitiveservices-luis-authoring/?view=azure-node-latest)<br>[Prédiction](/javascript/api/@azure/cognitiveservices-luis-runtime/?view=azure-node-latest)|[NPM - création](https://www.npmjs.com/package/@azure/cognitiveservices-luis-authoring)<br>[NPM - prédiction](https://www.npmjs.com/package/@azure/cognitiveservices-luis-runtime)|[Création](azure-sdk-quickstart.md?pivots=programming-language-javascript)<br>[Prédiction](azure-sdk-quickstart.md?pivots=programming-language-javascript)|
|Python|[Création et prédiction](azure-sdk-quickstart.md?pivots=programming-language-python)|[Pip](https://pypi.org/project/azure-cognitiveservices-language-luis/)|[Création](azure-sdk-quickstart.md?pivots=programming-language-python)<br>[Prédiction](azure-sdk-quickstart.md?pivots=programming-language-python)|


### <a name="containers"></a>Containers

LUIS (Language Understanding) propose un [conteneur](luis-container-howto.md) qui permet de fournir des versions locales et autonomes de votre application.

### <a name="export-and-import-formats"></a>Formats d'exportation et d'importation

Language Understanding vous permet de gérer votre application et ses modèles au format JSON, au format `.LU` ([LUDown](https://github.com/microsoft/botbuilder-tools/blob/master/packages/Ludown)), et sous forme de package compressé pour le conteneur Language Understanding.

L'importation et l'exportation de ces formats sont disponibles à partir des API et du portail LUIS. Le portail permet l'importation et l'exportation à partir de la liste Applications et de la liste Versions.

## <a name="workshops"></a>Ateliers

* GitHub : (Atelier) [IA conversationnelle : NLU avec LUIS](https://github.com/GlobalAICommunity/Workshop-Conversational-AI)

## <a name="continuous-integration-tools"></a>Outils d’intégration continue

* GitHub : (Préversion) [Développement d’une application LUIS à l’aide des pratiques DevOps](https://github.com/Azure-Samples/LUIS-DevOps-Template)
* GitHub : [NLU.DevOps](https://github.com/microsoft/NLU.DevOps) - Outils prenant en charge l’intégration et le déploiement continus pour les services NLU.

## <a name="bot-framework-tools"></a>Outils Bot Framework

Bot Framework est disponible en tant que [kit SDK](https://github.com/Microsoft/botframework) dans différents langages et en tant que service avec [Azure Bot Service](https://dev.botframework.com/).

Bot Framework fournit [plusieurs outils](https://github.com/microsoft/botbuilder-tools) pour vous aider à utiliser Language Understanding, notamment :

* [LUDown](https://github.com/microsoft/botbuilder-tools/blob/master/packages/Ludown) : créez des modèles de compréhension de la langue LUIS à l’aide de fichiers Markdown.
* [LUIS CLI](https://github.com/microsoft/botbuilder-tools/blob/master/packages/LUIS) : créez et gérez vos applications Luis.ai.
* [Dispatch](https://github.com/microsoft/botbuilder-tools/blob/master/packages/Dispatch) : gérez des applications parentes et enfants.
* [LUISGen](https://github.com/microsoft/botbuilder-tools/blob/master/packages/LUISGen) : générez automatiquement des classes C#/Typescript de support pour vos intentions et entités LUIS.
* [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases) : application de bureau permettant aux développeurs de bots de tester et de déboguer les bots créés à l'aide du kit SDK Bot Framework.
* [Bot Framework Composer](https://github.com/microsoft/BotFramework-Composer/blob/stable/README.md) : outil de développement intégré, destiné aux développeurs et aux équipes pluridisciplinaires pour créer des bots et des expériences de conversation avec Microsoft Bot Framework
* [Exemples Bot Framework](https://github.com/microsoft/botbuilder-samples) en C#, JavaScript, TypeScript et Python
## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les [codes d’erreur HTTP](luis-reference-response-codes.md) courants
* [Documentation de référence](../../index.yml) pour l'ensemble des API et des kits de développement logiciel (SDK)
* [Bot Framework](https://github.com/Microsoft/botbuilder-dotnet) et [Azure Bot Service](https://dev.botframework.com/)
* [LUDown](https://github.com/microsoft/botbuilder-tools/blob/master/packages/Ludown)
* [Conteneurs cognitifs](../cognitive-services-container-support.md)