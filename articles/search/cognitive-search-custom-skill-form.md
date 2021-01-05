---
title: Compétence personnalisée Form Recognizer (C#)
titleSuffix: Azure Cognitive Search
description: Découvrez comment créer une compétence personnalisée Form Recognizer à l’aide de C# et de Visual Studio.
manager: nitinme
author: PatrickFarley
ms.author: pafarley
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 01/21/2020
ms.openlocfilehash: 58f1c2621165a7074c04752832c6560b2fd3e423
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96011963"
---
# <a name="example-create-a-form-recognizer-custom-skill"></a>Exemple : Créer une compétence personnalisée Form Recognizer

Dans cet exemple d’ensemble de compétences de la Recherche cognitive Azure, vous allez apprendre à créer une compétence personnalisée Form Recognizer à l’aide de C# et de Visual Studio. Form Recognizer analyse des documents et extrait les paires clé/valeur et les données de table. En wrappant Form Recognizer dans l’[interface de compétence personnalisée](cognitive-search-custom-skill-interface.md), vous pouvez ajouter cette fonctionnalité en tant qu’étape d’un pipeline d’enrichissement de bout en bout. Le pipeline peut ensuite charger les documents et effectuer d’autres transformations.

## <a name="prerequisites"></a>Prérequis

- [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/) (toute édition).
- Au moins cinq formulaires du même type. Vous pouvez utiliser les exemples de données fournis avec ce guide.

## <a name="create-a-form-recognizer-resource"></a>Créer une ressource Form Recognizer

[!INCLUDE [create resource](../cognitive-services/form-recognizer/includes/create-resource.md)]

## <a name="train-your-model"></a>Entraîner votre modèle

Vous devez entraîner un modèle Form Recognizer avec vos formulaires d’entrée avant d’utiliser cette compétence. Suivez le [guide de démarrage rapide cURL](../cognitive-services/form-recognizer/quickstarts/curl-train-extract.md) pour savoir comment entraîner un modèle. Vous pouvez utiliser les exemples de formulaire fournis dans ce guide de démarrage rapide ou utiliser vos propres données. Une fois le modèle entraîné, copiez sa valeur d’ID dans un emplacement sécurisé.

## <a name="set-up-the-custom-skill"></a>Configurer la compétence personnalisée

Ce tutoriel utilise le projet [AnalyzeForm](https://github.com/Azure-Samples/azure-search-power-skills/tree/master/Vision/AnalyzeForm) dans le dépôt GitHub [Super compétences de la Recherche Azure](https://github.com/Azure-Samples/azure-search-power-skills). Clonez ce dépôt sur votre ordinateur local, puis naviguez vers **Vision/AnalyzeForm/** pour accéder au projet. Ouvrez ensuite _AnalyzeForm.csproj_ dans Visual Studio. Ce projet crée une ressource Azure Function qui satisfait l’[interface de compétence personnalisée](cognitive-search-custom-skill-interface.md) et qui peut être utilisée pour l’enrichissement de la Recherche cognitive Azure. Il prend des documents de formulaire comme entrées et génère (sous forme de texte) les paires clé/valeur que vous spécifiez.

Tout d’abord, ajoutez des variables d’environnement au niveau du projet. Localisez le projet **AnalyzeForm** dans le volet gauche, cliquez dessus avec le bouton droit, puis sélectionnez **Propriétés**. Dans la fenêtre **Propriétés**, cliquez sur l’onglet **Débogage**, puis recherchez le champ **Variables d’environnement**. Cliquez sur **Ajouter** pour ajouter les variables suivantes :
* `FORMS_RECOGNIZER_ENDPOINT_URL` avec la valeur définie sur l’URL de votre point de terminaison.
* `FORMS_RECOGNIZER_API_KEY` avec la valeur définie sur votre clé d’abonnement.
* `FORMS_RECOGNIZER_MODEL_ID` avec la valeur définie sur l’ID du modèle que vous avez entraîné.
* `FORMS_RECOGNIZER_RETRY_DELAY` avec la valeur définie sur 1000. Cette valeur est le temps, en millisecondes, pendant lequel le programme attend avant de retenter la requête.
* `FORMS_RECOGNIZER_MAX_ATTEMPTS` avec la valeur définie sur 100. Cette valeur est le nombre de fois où le programme interroge le service en tentant d’obtenir une réponse correcte.

Ensuite, ouvrez _AnalyzeForm.cs_, puis recherchez la variable `fieldMappings`, qui fait référence au fichier *field-mappings.json*. Ce fichier (et la variable qui y fait référence) définissent la liste des clés que vous voulez extraire de vos formulaires et une étiquette personnalisée pour chaque clé. Par exemple, la valeur `{ "Address:", "address" }, { "Invoice For:", "recipient" }` signifie que le script enregistre uniquement les valeurs définies pour les champs `Address:` et `Invoice For:` détectés, et qu’il étiquette ces valeurs avec `"address"` et `"recipient"`, respectivement.

Enfin, notez la variable `contentType`. Ce script exécute le modèle Form Recognizer donné sur des documents distants qui sont référencés par URL, de sorte que le type de contenu est `application/json`. Si vous voulez analyser des fichiers locaux en incluant leurs flux d’octets dans les requêtes HTTP, vous devez remplacer le `contentType` par le [type MIME](https://developer.mozilla.org/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Complete_list_of_MIME_types) approprié pour votre fichier.

## <a name="test-the-function-from-visual-studio"></a>Tester la fonction à partir de Visual Studio

Une fois que vous avez modifié votre projet, enregistrez-le et définissez le projet **AnalyzeForm** comme projet de démarrage dans Visual Studio (s’il n’est pas déjà défini). Appuyez ensuite sur **F5** pour exécuter la fonction dans votre environnement local. Utilisez un service REST comme [Postman](https://www.postman.com/) pour appeler la fonction.

### <a name="http-request"></a>Demande HTTP

Vous allez effectuer la requête suivante pour appeler la fonction.

```HTTP
POST https://localhost:7071/api/analyze-form
```

### <a name="request-body"></a>Corps de la demande

Commencez par le modèle de corps de la demande ci-dessous.

```json
{
    "values": [
        {
            "recordId": "record1",
            "data": { 
                "formUrl": "<your-form-url>",
                "formSasToken": "<your-sas-token>"
            }
        }
    ]
}
```

Ici, vous devez fournir l’URL d’un formulaire qui a le même type que les formulaires avec lesquels vous avez effectué l’entraînement. À des fins de test, vous pouvez utiliser l’un de vos formulaires d’entraînement. Si vous avez suivi le guide de démarrage rapide cURL, vos formulaires se trouvent dans un compte de stockage d’objets blob Azure. Ouvrez l’Explorateur Stockage Azure, localisez un fichier de formulaire, cliquez dessus avec le bouton droit, puis sélectionnez **Obtenir une signature d’accès partagé**. La fenêtre de boîte de dialogue suivante fournit une URL et un jeton SAS. Entrez ces chaînes dans les champs `"formUrl"` et `"formSasToken"` du corps de votre demande, respectivement.

> [!div class="mx-imgBorder"]
> ![Explorateur Stockage Azure ; un document PDF est sélectionné](media/cognitive-search-skill-form/form-sas.png)

Si vous voulez analyser un document distant qui ne se trouve pas dans le stockage d’objets blob Azure, collez son URL dans le champ `"formUrl"` et laissez le champ `"formSasToken"` vide.

> [!NOTE]
> Quand la compétence est intégrée dans un ensemble de compétences, l’URL et le jeton sont fournis par la Recherche cognitive.

### <a name="response"></a>response

Vous devriez voir une réponse similaire à l’exemple suivant :

```json
{
    "values": [
        {
            "recordId": "record1",
            "data": {
                "address": "1111 8th st. Bellevue, WA 99501 ",
                "recipient": "Southridge Video 1060 Main St. Atlanta, GA 65024 "
            },
            "errors": null,
            "warnings": null
        }
    ]
}
```

## <a name="publish-the-function-to-azure"></a>Publier la fonction sur Azure

Lorsque vous êtes satisfait du comportement de la fonction, vous pouvez la publier.

1. Dans l’**Explorateur de solutions** dans Visual Studio, cliquez avec le bouton droit sur le projet, puis sélectionnez **Publier**. Choisissez **Créer** > **Publier**.
1. Si vous n’avez pas encore connecté Visual Studio à votre compte Azure, sélectionnez **Ajouter un compte...** .
1. Suivez les invitations qui s’affichent à l’écran. Spécifiez un nom unique pour votre service d’application, l’abonnement Azure, le groupe de ressources, le plan d’hébergement et le compte de stockage que vous voulez utiliser. Si vous n’avez pas encore de groupe de ressources, de plan d’hébergement et de compte de stockage, vous pouvez les créer. Quand vous avez terminé, sélectionnez **Créer**.
1. Une fois le déploiement terminé, notez l’URL du site. Cette URL est l’adresse de votre application de fonction dans Azure. Enregistrez-la dans un emplacement temporaire.
1. Dans le [portail Azure](https://portal.azure.com), accédez au groupe de ressources, puis recherchez la fonction `AnalyzeForm` que vous avez publiée. Dans la section **Gérer**, vous devriez voir Host Keys. Copiez la clé d’hôte *par défaut*, puis enregistrez-la dans un emplacement temporaire.

## <a name="connect-to-your-pipeline"></a>Connexion à votre pipeline

Pour utiliser cette compétence dans un pipeline de Recherche cognitive, vous devez ajouter une définition de compétence à votre ensemble de compétences. Le bloc JSON suivant est un exemple de définition de compétence (vous devez mettre à jour les entrées et les sorties pour refléter l’environnement de votre scénario et votre ensemble de compétences spécifiques). Remplacez `AzureFunctionEndpointUrl` par votre URL de fonction, puis remplacez `AzureFunctionDefaultHostKey` par votre clé d’hôte.

```json
{ 
  "description":"Skillset that invokes the Form Recognizer custom skill",
  "skills":[ 
    "[... your existing skills go here]",
    { 
      "@odata.type":"#Microsoft.Skills.Custom.WebApiSkill",
      "name":"formrecognizer",
      "description":"Extracts fields from a form using a pre-trained form recognition model",
      "uri":"[AzureFunctionEndpointUrl]/api/analyze-form?code=[AzureFunctionDefaultHostKey]",
      "httpMethod":"POST",
      "timeout":"PT30S",
      "context":"/document",
      "batchSize":1,
      "inputs":[ 
        { 
          "name":"formUrl",
          "source":"/document/metadata_storage_path"
        },
        { 
          "name":"formSasToken",
          "source":"/document/metadata_storage_sas_token"
        }
      ],
      "outputs":[ 
        { 
          "name":"address",
          "targetName":"address"
        },
        { 
          "name":"recipient",
          "targetName":"recipient"
        }
      ]
    }
  ]
}
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide, vous avez créé une compétence personnalisée à partir du service Azure Form Recognizer. Pour en savoir plus sur les compétences personnalisées, consultez les ressources suivantes. 

* [Super compétences de la Recherche Azure : un dépôt de compétences personnalisées](https://github.com/Azure-Samples/azure-search-power-skills)
* [Ajouter une qualification personnalisée à un pipeline d’enrichissement par IA](cognitive-search-custom-skill-interface.md)
* [How to create a skillset in an enrichment pipeline](cognitive-search-defining-skillset.md) (Créer un ensemble de compétences dans un pipeline d’enrichissement)
* [Créer un ensemble de compétences (REST)](/rest/api/searchservice/create-skillset)
* [Mapper des champs enrichis](cognitive-search-output-field-mapping.md)