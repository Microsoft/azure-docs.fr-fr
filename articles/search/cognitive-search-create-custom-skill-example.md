---
title: Exemple de compétence personnalisée utilisant l’API Recherche d’entités Bing
titleSuffix: Azure Cognitive Search
description: Présente l’utilisation du service Recherche d’entités Bing dans une compétence personnalisée mappée à un pipeline d’indexation IA dans Recherche cognitive Azure.
manager: nitinme
author: luiscabrer
ms.author: luisca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/04/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 5755e14e53d359fd8b322939bf1325d21536d593
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89020182"
---
# <a name="example-create-a-custom-skill-using-the-bing-entity-search-api"></a>Exemple : Créer une qualification personnalisée à l’aide de l’API Recherche d’entités Bing

Dans cet exemple, découvrez comment créer une compétence personnalisée d’API web. Cette qualification accepte des emplacements, des chiffres publics et des organisations, et renvoie leurs descriptions. L’exemple utilise une [fonction Azure](https://azure.microsoft.com/services/functions/) pour encapsuler l’[API Recherche d’entités Bing](https://azure.microsoft.com/services/cognitive-services/bing-entity-search-api/) de façon à ce qu’elle implémente l’interface de qualification personnalisée.

## <a name="prerequisites"></a>Prérequis

+ Si vous ne connaissez pas l’interface d’entrée/sortie qu’une compétence personnalisée doit implémenter, lisez l’article sur l’[interface de compétence personnalisée](cognitive-search-custom-skill-interface.md).

+ [!INCLUDE [cognitive-services-bing-entity-search-signup-requirements](../../includes/cognitive-services-bing-entity-search-signup-requirements.md)]

+ Installez [Visual Studio 2019](https://www.visualstudio.com/vs/) ou une version ultérieure, incluant la charge de travail de développement Azure.

## <a name="create-an-azure-function"></a>Création d’une fonction Azure

Bien que cet exemple utilise une Fonction Azure pour héberger une API web, elle n’est pas indispensable.  Pour autant que vous respectiez les [exigences d’interface pour une compétence cognitive](cognitive-search-custom-skill-interface.md), l’approche que vous adoptez est sans importance. Toutefois, Azure Functions facilite la création d’une compétence personnalisée.

### <a name="create-a-function-app"></a>Créer une application de fonction

1. Dans Visual Studio, dans le menu Fichier, sélectionnez **Nouveau** > **Projet**.

1. Dans la boîte de dialogue Nouveau projet, sélectionnez **Installé**, développez **Visual C#**  > **Cloud**, sélectionnez **Azure Functions**, tapez un Nom pour votre projet, puis cliquez sur **OK**. Le nom d’application de la fonction doit être valide en tant qu’espace de noms C#, afin de ne pas utiliser des traits d’union, des traits de soulignement ou d’autres caractères non alphanumériques.

1. Sélectionnez **Azure Functions v2 (.NET Core)** . Vous pouvez également utiliser la version 1, mais le code écrit ci-dessous est basé sur le modèle v2.

1. Sélectionnez le type **Déclencheur HTTP**

1. Pour Compte de stockage, vous pouvez sélectionner **Aucun**, car vous n’aurez besoin d’aucun stockage pour cette fonction.

1. Sélectionnez **OK** pour créer le projet de fonction et la fonction HTTP déclenchée.

### <a name="modify-the-code-to-call-the-bing-entity-search-service"></a>Modifier le code pour appeler le service Recherche d’entités Bing

Visual Studio crée un projet qui contient une classe qui présente un code réutilisable pour le type de fonction sélectionné. L’attribut *FunctionName* de la méthode définit le nom de la fonction. L’attribut *HttpTrigger* spécifie que la fonction est déclenchée par une requête HTTP.

À présent, remplacez tout le contenu du fichier *Function1.cs* par le code suivant :

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Net.Http;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Logging;
using Newtonsoft.Json;

namespace SampleSkills
{
    /// <summary>
    /// Sample custom skill that wraps the Bing entity search API to connect it with a 
    /// AI enrichment pipeline.
    /// </summary>
    public static class BingEntitySearch
    {
        #region Credentials
        // IMPORTANT: Make sure to enter your credential and to verify the API endpoint matches yours.
        static readonly string bingApiEndpoint = "https://api.cognitive.microsoft.com/bing/v7.0/entities/";
        static readonly string key = "<enter your api key here>";  
        #endregion

        #region Class used to deserialize the request
        private class InputRecord
        {
            public class InputRecordData
            {
                public string Name { get; set; }
            }

            public string RecordId { get; set; }
            public InputRecordData Data { get; set; }
        }

        private class WebApiRequest
        {
            public List<InputRecord> Values { get; set; }
        }
        #endregion

        #region Classes used to serialize the response

        private class OutputRecord
        {
            public class OutputRecordData
            {
                public string Name { get; set; } = "";
                public string Description { get; set; } = "";
                public string Source { get; set; } = "";
                public string SourceUrl { get; set; } = "";
                public string LicenseAttribution { get; set; } = "";
                public string LicenseUrl { get; set; } = "";
            }

            public class OutputRecordMessage
            {
                public string Message { get; set; }
            }

            public string RecordId { get; set; }
            public OutputRecordData Data { get; set; }
            public List<OutputRecordMessage> Errors { get; set; }
            public List<OutputRecordMessage> Warnings { get; set; }
        }

        private class WebApiResponse
        {
            public List<OutputRecord> Values { get; set; }
        }
        #endregion

        #region Classes used to interact with the Bing API
        private class BingResponse
        {
            public BingEntities Entities { get; set; }
        }
        private class BingEntities
        {
            public BingEntity[] Value { get; set; }
        }

        private class BingEntity
        {
            public class EntityPresentationinfo
            {
                public string[] EntityTypeHints { get; set; }
            }

            public class License
            {
                public string Url { get; set; }
            }

            public class ContractualRule
            {
                public string _type { get; set; }
                public License License { get; set; }
                public string LicenseNotice { get; set; }
                public string Text { get; set; }
                public string Url { get; set; }
            }

            public ContractualRule[] ContractualRules { get; set; }
            public string Description { get; set; }
            public string Name { get; set; }
            public EntityPresentationinfo EntityPresentationInfo { get; set; }
        }
        #endregion

        #region The Azure Function definition

        [FunctionName("EntitySearch")]
        public static async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Function, "post", Route = null)] HttpRequest req,
            ILogger log)
        {
            log.LogInformation("Entity Search function: C# HTTP trigger function processed a request.");

            var response = new WebApiResponse
            {
                Values = new List<OutputRecord>()
            };

            string requestBody = new StreamReader(req.Body).ReadToEnd();
            var data = JsonConvert.DeserializeObject<WebApiRequest>(requestBody);

            // Do some schema validation
            if (data == null)
            {
                return new BadRequestObjectResult("The request schema does not match expected schema.");
            }
            if (data.Values == null)
            {
                return new BadRequestObjectResult("The request schema does not match expected schema. Could not find values array.");
            }

            // Calculate the response for each value.
            foreach (var record in data.Values)
            {
                if (record == null || record.RecordId == null) continue;

                OutputRecord responseRecord = new OutputRecord
                {
                    RecordId = record.RecordId
                };

                try
                {
                    responseRecord.Data = GetEntityMetadata(record.Data.Name).Result;
                }
                catch (Exception e)
                {
                    // Something bad happened, log the issue.
                    var error = new OutputRecord.OutputRecordMessage
                    {
                        Message = e.Message
                    };

                    responseRecord.Errors = new List<OutputRecord.OutputRecordMessage>
                    {
                        error
                    };
                }
                finally
                {
                    response.Values.Add(responseRecord);
                }
            }

            return (ActionResult)new OkObjectResult(response);
        }

        #endregion

        #region Methods to call the Bing API
        /// <summary>
        /// Gets metadata for a particular entity based on its name using Bing Entity Search
        /// </summary>
        /// <param name="entityName">The name of the entity to extract data for.</param>
        /// <returns>Asynchronous task that returns entity data. </returns>
        private async static Task<OutputRecord.OutputRecordData> GetEntityMetadata(string entityName)
        {
            var uri = bingApiEndpoint + "?q=" + entityName + "&mkt=en-us&count=10&offset=0&safesearch=Moderate";
            var result = new OutputRecord.OutputRecordData();

            using (var client = new HttpClient())
            using (var request = new HttpRequestMessage {
                Method = HttpMethod.Get,
                RequestUri = new Uri(uri)
            })
            {
                request.Headers.Add("Ocp-Apim-Subscription-Key", key);

                HttpResponseMessage response = await client.SendAsync(request);
                string responseBody = await response?.Content?.ReadAsStringAsync();

                BingResponse bingResult = JsonConvert.DeserializeObject<BingResponse>(responseBody);
                if (bingResult != null)
                {
                    // In addition to the list of entities that could match the name, for simplicity let's return information
                    // for the top match as additional metadata at the root object.
                    return AddTopEntityMetadata(bingResult.Entities?.Value);
                }
            }

            return result;
        }

        private static OutputRecord.OutputRecordData AddTopEntityMetadata(BingEntity[] entities)
        {
            if (entities != null)
            {
                foreach (BingEntity entity in entities.Where(
                    entity => entity?.EntityPresentationInfo?.EntityTypeHints != null
                        && (entity.EntityPresentationInfo.EntityTypeHints[0] == "Person"
                            || entity.EntityPresentationInfo.EntityTypeHints[0] == "Organization"
                            || entity.EntityPresentationInfo.EntityTypeHints[0] == "Location")
                        && !String.IsNullOrEmpty(entity.Description)))
                {
                    var rootObject = new OutputRecord.OutputRecordData
                    {
                        Description = entity.Description,
                        Name = entity.Name
                    };

                    if (entity.ContractualRules != null)
                    {
                        foreach (var rule in entity.ContractualRules)
                        {
                            switch (rule._type)
                            {
                                case "ContractualRules/LicenseAttribution":
                                    rootObject.LicenseAttribution = rule.LicenseNotice;
                                    rootObject.LicenseUrl = rule.License.Url;
                                    break;
                                case "ContractualRules/LinkAttribution":
                                    rootObject.Source = rule.Text;
                                    rootObject.SourceUrl = rule.Url;
                                    break;
                            }
                        }
                    }

                    return rootObject;
                }
            }

            return new OutputRecord.OutputRecordData();
        }
        #endregion
    }
}
```

Veillez à entrer votre propre valeur de *clé* dans la constante `key`en fonction de la clé que vous avez obtenue lors de l’inscription à l’API Recherche d’entités Bing.

Cet exemple inclut tous le code nécessaire dans un fichier unique pour plus de commodité. Vous pouvez trouver une version légèrement plus structurée de cette qualification dans le [référentiel des qualifications puissantes](https://github.com/Azure-Samples/azure-search-power-skills/tree/master/Text/BingEntitySearch).

Vous pouvez bien sûr renommer le fichier de `Function1.cs` en `BingEntitySearch.cs`.

## <a name="test-the-function-from-visual-studio"></a>Tester la fonction à partir de Visual Studio

Appuyez sur **F5** pour exécuter le programme et tester le comportement de la fonction. En l’occurrence, nous allons utiliser la fonction ci-dessous pour examiner deux entités. Utilisez Postman ou Fiddler pour émettre un appel tel que celui ci-dessous :

```http
POST https://localhost:7071/api/EntitySearch
```

### <a name="request-body"></a>Corps de la demande
```json
{
    "values": [
        {
            "recordId": "e1",
            "data":
            {
                "name":  "Pablo Picasso"
            }
        },
        {
            "recordId": "e2",
            "data":
            {
                "name":  "Microsoft"
            }
        }
    ]
}
```

### <a name="response"></a>response
Vous devriez voir une réponse similaire à l’exemple suivant :

```json
{
    "values": [
        {
            "recordId": "e1",
            "data": {
                "name": "Pablo Picasso",
                "description": "Pablo Ruiz Picasso was a Spanish painter [...]",
                "source": "Wikipedia",
                "sourceUrl": "http://en.wikipedia.org/wiki/Pablo_Picasso",
                "licenseAttribution": "Text under CC-BY-SA license",
                "licenseUrl": "http://creativecommons.org/licenses/by-sa/3.0/"
            },
            "errors": null,
            "warnings": null
        },
        "..."
    ]
}
```

## <a name="publish-the-function-to-azure"></a>Publier la fonction sur Azure

Lorsque vous êtes satisfait du comportement de la fonction, vous pouvez la publier.

1. Dans **l’Explorateur de solutions**, cliquez avec le bouton droit sur le projet, puis sélectionnez **Publier**. Choisissez **Créer** > **Publier**.

1. Si vous n’avez pas encore connecté Visual Studio à votre compte Azure, sélectionnez **Ajouter un compte...** .

1. Suivez les invitations qui s’affichent à l’écran. Vous êtes invité à spécifier un nom unique pour votre service d’application, l’abonnement Azure, le groupe de ressources, le plan d’hébergement et le compte de stockage que vous souhaitez utiliser. Si vous n’en avez pas de groupe de ressources, de plan d’hébergement et de compte de stockage, vous pouvez les créer. Quand vous avez terminé, sélectionnez **Créer**.

1. Une fois le déploiement terminé, notez l’URL du site. Il s’agit de l’adresse de votre application de fonction dans Azure. 

1. Dans le [portail Azure](https://portal.azure.com), accédez au groupe de ressources, puis recherchez la fonction `EntitySearch` que vous avez publiée. Dans la section **Gérer**, vous devriez voir Host Keys. Sélectionnez l’icône **Copier** pour la clé d’hôte *par défaut*.  

## <a name="test-the-function-in-azure"></a>Tester la fonction dans Azure

À présent que vous disposez de la clé d’hôte par défaut, testez votre fonction comme suit :

```http
POST https://[your-entity-search-app-name].azurewebsites.net/api/EntitySearch?code=[enter default host key here]
```

### <a name="request-body"></a>Corps de la requête
```json
{
    "values": [
        {
            "recordId": "e1",
            "data":
            {
                "name":  "Pablo Picasso"
            }
        },
        {
            "recordId": "e2",
            "data":
            {
                "name":  "Microsoft"
            }
        }
    ]
}
```

Cet exemple devrait produire le même résultat que celui que vous avez obtenu précédemment lors de l’exécution de la fonction dans l’environnement local.

## <a name="connect-to-your-pipeline"></a>Connexion à votre pipeline
À présent que vous avez une nouvelle compétence personnalisée, vous pouvez l’ajouter à vos jeu de compétences. L’exemple ci-dessous montre comment appeler la qualification pour ajouter des descriptions aux organisations dans le document (il pourrait être étendu à des emplacements ou à des personnes). Remplacez `[your-entity-search-app-name]` par le nom de votre application.

```json
{
    "skills": [
      "[... your existing skills remain here]",  
      {
        "@odata.type": "#Microsoft.Skills.Custom.WebApiSkill",
        "description": "Our new Bing entity search custom skill",
        "uri": "https://[your-entity-search-app-name].azurewebsites.net/api/EntitySearch?code=[enter default host key here]",
          "context": "/document/merged_content/organizations/*",
          "inputs": [
            {
              "name": "name",
              "source": "/document/merged_content/organizations/*"
            }
          ],
          "outputs": [
            {
              "name": "description",
              "targetName": "description"
            }
          ]
      }
  ]
}
```

Ici, nous comptons sur le fait que la [compétence de reconnaissance d’entité](cognitive-search-skill-entity-recognition.md) intégrée soit présente dans l’ensemble de qualifications et ait enrichi le document avec la liste d’organisations. Pour référence, voici une configuration de qualification d’extraction d’entité qui suffirait pour générer les données dont nous avons besoin :

```json
{
    "@odata.type": "#Microsoft.Skills.Text.EntityRecognitionSkill",
    "name": "#1",
    "description": "Organization name extraction",
    "context": "/document/merged_content",
    "categories": [ "Organization" ],
    "defaultLanguageCode": "en",
    "inputs": [
        {
            "name": "text",
            "source": "/document/merged_content"
        },
        {
            "name": "languageCode",
            "source": "/document/language"
        }
    ],
    "outputs": [
        {
            "name": "organizations",
            "targetName": "organizations"
        }
    ]
},
```

## <a name="next-steps"></a>Étapes suivantes
Félicitations ! Vous avez créé votre première qualification personnalisée. Vous pouvez maintenant suivre le même schéma pour ajouter vos propres fonctionnalités personnalisées. Pour en savoir plus, cliquez sur les liens suivants.

+ [Super compétences : référentiel de compétences personnalisées](https://github.com/Azure-Samples/azure-search-power-skills)
+ [Ajouter une qualification personnalisée à un pipeline d’enrichissement par IA](cognitive-search-custom-skill-interface.md)
+ [Guide pratique pour définir un ensemble de compétences](cognitive-search-defining-skillset.md)
+ [Créer un jeu de compétences (REST)](/rest/api/searchservice/create-skillset)
+ [Guide pratique pour mapper des champs enrichis](cognitive-search-output-field-mapping.md)