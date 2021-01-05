---
title: Exemple de compétence personnalisée (Python)
titleSuffix: Azure Cognitive Search
description: En tant que développeur Python, découvrez les outils et les techniques qui vous permettront de créer une compétence personnalisée avec Azure Functions et Visual Studio. Les compétences personnalisées contiennent une logique ou des modèles définis par l’utilisateur que vous pouvez ajouter à un pipeline d’indexation enrichi par l’IA dans la Recherche cognitive Azure.
manager: nitinme
author: luiscabrer
ms.author: luisca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 01/15/2020
ms.custom: devx-track-python
ms.openlocfilehash: 40e20ad4bab0275b44cd868521c7dc70dec52567
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88936127"
---
# <a name="example-create-a-custom-skill-using-python"></a>Exemple : Créer une compétence personnalisée avec Python

Dans cet exemple d’ensemble de compétences concernant la Recherche cognitive Azure, vous allez apprendre à créer une compétence personnalisée d’API web à l’aide de Python et de Visual Studio Code. L’exemple utilise une [fonction Azure](https://azure.microsoft.com/services/functions/) qui implémente l’[interface de la compétence personnalisée](cognitive-search-custom-skill-interface.md).

La conception de la compétence personnalisée est simple (elle concatène deux chaînes), ce qui vous permet de vous concentrer sur les outils et les technologies utilisés pour le développement de compétences personnalisées en Python. Une fois que vous avez réussi avec une compétence simple, vous pouvez vous baser sur celle-ci pour créer des scénarios plus complexes.

## <a name="prerequisites"></a>Prérequis

+ Pour connaître l’interface d’entrée et de sortie qu’une compétence personnalisée doit implémenter, examinez l’[interface de compétence personnalisée](cognitive-search-custom-skill-interface.md).

+ Configurez votre environnement. Nous avons suivi [ce tutoriel de bout en bout](/azure/python/tutorial-vs-code-serverless-python-01) pour configurer une fonction Azure serverless à l’aide d’extensions Visual Studio Code et Python. Ce tutoriel va vous guider tout au long de l’installation des outils et des composants suivants : 

  + [Python 3.75](https://www.python.org/downloads/release/python-375/)
  + [Visual Studio Code](https://code.visualstudio.com/)
  + [Extension Python pour Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
  + [Azure Functions Core Tools](../azure-functions/functions-run-local.md#v2)
  + [Extension Azure Functions pour Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions)

## <a name="create-an-azure-function"></a>Création d’une fonction Azure

Cet exemple utilise une fonction Azure pour illustrer le concept d’hébergement d’une API web. Toutefois, d’autres approches sont possibles. Pour autant que vous respectiez les [exigences d’interface pour une compétence cognitive](cognitive-search-custom-skill-interface.md), l’approche que vous adoptez est sans importance. Toutefois, Azure Functions facilite la création d’une compétence personnalisée.

### <a name="create-a-function-app"></a>Créer une application de fonction

Le modèle de projet Azure Functions dans Visual Studio Code crée un projet qui peut être publié dans une application de fonction sur Azure. Une application de fonctions vous permet de regrouper les fonctions dans une unité logique pour la gestion, le déploiement et le partage des ressources.

1. Dans Visual Studio Code, appuyez sur F1 pour ouvrir la palette de commandes. Dans la palette de commandes, recherchez et sélectionnez `Azure Functions: Create new project...`.

1. Choisissez un emplacement de répertoire pour votre espace de travail de projet et optez pour **Sélectionner**.

    > [!NOTE]
    > Ces étapes ont été conçues pour être terminées en dehors d’un espace de travail. Pour cette raison, vous ne devez pas sélectionner un dossier de projet qui fait partie d’un espace de travail.

1. Sélectionnez un langage pour votre projet d’application de fonction. Pour ce tutoriel, sélectionnez **Python**.
1. Sélectionnez la version de Python (la version 3.7.5 est prise en charge par Azure Functions).
1. Sélectionnez un modèle pour la première fonction de votre projet. Sélectionnez un **déclencheur HTTP** pour créer une fonction déclenchée via HTTP dans la nouvelle application de fonction.
1. Attribuez un nom à la fonction. Dans ce cas, nous allons utiliser **Concatenator**. 
1. Sélectionnez **Fonction** comme niveau d’autorisation. Cela signifie que nous allons fournir une [clé de fonction](../azure-functions/functions-bindings-http-webhook-trigger.md#authorization-keys) pour appeler le point de terminaison HTTP de la fonction. 
1. Sélectionnez la façon dont vous souhaitez ouvrir votre projet. Pour cette étape, sélectionnez **Ajouter à l’espace de travail** afin de créer l’application de fonction dans l’espace de travail actuel.

Visual Studio Code crée le projet d’application de fonction dans un nouvel espace de travail. Ce projet contient les fichiers de configuration [host.json](../azure-functions/functions-host-json.md) et [local.settings.json](../azure-functions/functions-run-local.md#local-settings-file), ainsi que des fichiers de projet spécifiques au langage. 

Une nouvelle fonction déclenchée via HTTP est également créée dans le dossier **Concatenator** du projet d’application de fonction. Vous trouverez dans celui-ci un fichier nommé « \_\_init_.py » dont le contenu est le suivant :

```py
import logging

import azure.functions as func


def main(req: func.HttpRequest) -> func.HttpResponse:
    logging.info('Python HTTP trigger function processed a request.')

    name = req.params.get('name')
    if not name:
        try:
            req_body = req.get_json()
        except ValueError:
            pass
        else:
            name = req_body.get('name')

    if name:
        return func.HttpResponse(f"Hello {name}!")
    else:
        return func.HttpResponse(
             "Please pass a name on the query string or in the request body",
             status_code=400
        )

```

À présent, modifions ce code pour suivre l’[interface de compétence personnalisée](cognitive-search-custom-skill-interface.md). Modifiez le code avec le contenu suivant :

```py
import logging
import azure.functions as func
import json

def main(req: func.HttpRequest) -> func.HttpResponse:
    logging.info('Python HTTP trigger function processed a request.')

    try:
        body = json.dumps(req.get_json())
    except ValueError:
        return func.HttpResponse(
             "Invalid body",
             status_code=400
        )
    
    if body:
        result = compose_response(body)
        return func.HttpResponse(result, mimetype="application/json")
    else:
        return func.HttpResponse(
             "Invalid body",
             status_code=400
        )


def compose_response(json_data):
    values = json.loads(json_data)['values']
    
    # Prepare the Output before the loop
    results = {}
    results["values"] = []
    
    for value in values:
        output_record = transform_value(value)
        if output_record != None:
            results["values"].append(output_record)
    return json.dumps(results, ensure_ascii=False)

## Perform an operation on a record
def transform_value(value):
    try:
        recordId = value['recordId']
    except AssertionError  as error:
        return None

    # Validate the inputs
    try:         
        assert ('data' in value), "'data' field is required."
        data = value['data']        
        assert ('text1' in data), "'text1' field is required in 'data' object."
        assert ('text2' in data), "'text2' field is required in 'data' object."
    except AssertionError  as error:
        return (
            {
            "recordId": recordId,
            "errors": [ { "message": "Error:" + error.args[0] }   ]       
            })

    try:                
        concatenated_string = value['data']['text1'] + " " + value['data']['text2']  
        # Here you could do something more interesting with the inputs

    except:
        return (
            {
            "recordId": recordId,
            "errors": [ { "message": "Could not complete operation for record." }   ]       
            })

    return ({
            "recordId": recordId,
            "data": {
                "text": concatenated_string
                    }
            })
```

La méthode **transform_value** effectue une opération sur un seul enregistrement. Vous pouvez modifier la méthode en fonction de vos besoins. N’oubliez pas d’effectuer les validations d’entrée nécessaires, et de retourner une erreur ou un avertissement en cas d’échec de l’opération.

### <a name="debug-your-code-locally"></a>Déboguer votre code localement

Visual Studio Code facilite le débogage de votre code. Appuyez sur F5 ou accédez au menu **Déboguer** pour sélectionner **Démarrer le débogage**.

Vous pouvez définir des points d’arrêt dans le code en appuyant sur F9 sur la ligne qui vous intéresse.

Une fois que vous avez démarré le débogage, votre fonction s’exécute localement. Vous pouvez utiliser un outil tel que Postman ou Fiddler pour envoyer la requête à localhost. Notez l’emplacement de votre point de terminaison local dans la fenêtre de terminal. 

## <a name="publish-your-function"></a>Publier votre fonction

Lorsque vous êtes satisfait du comportement de la fonction, vous pouvez la publier.

1. Dans Visual Studio Code, appuyez sur F1 pour ouvrir la palette de commandes. Dans la palette de commandes, recherchez et sélectionnez **Déployer dans l’application de fonction...** . 

1. Sélectionnez l’abonnement Azure dans lequel vous souhaitez déployer votre application.

1. Sélectionnez **+ Créer une application de fonction dans Azure**.

1. Entrez un nom global unique pour votre application de fonction.

1. Sélectionnez la version de Python (Python 3.7.x est compatible avec cette fonction).

1. Sélectionnez un emplacement pour la nouvelle ressource (par exemple, USA Ouest 2).

À ce stade, les ressources nécessaires seront créées dans votre abonnement Azure pour héberger la nouvelle fonction Azure dans Azure. Attendez la fin du déploiement. La fenêtre de sortie affiche l’état du processus de déploiement.

1. Dans le [portail Azure](https://portal.azure.com), accédez à **Toutes les ressources**, puis recherchez la fonction que vous avez publiée à l’aide de son nom. Si vous l’avez nommée **Concatenator**, sélectionnez la ressource.

1. Cliquez sur le bouton **</> Obtenir l’URL de fonction**. Cela vous permettra de copier l’URL pour appeler la fonction.

## <a name="test-the-function-in-azure"></a>Tester la fonction dans Azure

À présent que vous disposez de la clé d’hôte par défaut, testez votre fonction comme suit :

```http
POST [Function URL you copied above]
```

### <a name="request-body"></a>Corps de la requête
```json
{
    "values": [
        {
            "recordId": "e1",
            "data":
            {
                "text1":  "Hello",
                "text2":  "World"
            }
        },
        {
            "recordId": "e2",
            "data": "This is an invalid input"
        }
    ]
}
```

Cet exemple devrait produire le même résultat que celui que vous avez obtenu précédemment lors de l’exécution de la fonction dans l’environnement local.

## <a name="connect-to-your-pipeline"></a>Connexion à votre pipeline

À présent que vous avez une nouvelle compétence personnalisée, vous pouvez l’ajouter à vos jeu de compétences. L’exemple ci-dessous montre comment appeler la compétence en vue de concaténer le titre et l’auteur du document dans un même champ unique que nous appellerons merged_title_author. Remplacez `[your-function-url-here]` par l’URL de votre nouvelle fonction Azure.

```json
{
    "skills": [
      "[... your existing skills remain here]",  
      {
        "@odata.type": "#Microsoft.Skills.Custom.WebApiSkill",
        "description": "Our new search custom skill",
        "uri": "https://[your-function-url-here]",        
          "context": "/document/merged_content/organizations/*",
          "inputs": [
            {
              "name": "text1",
              "source": "/document/metadata_title"
            },
            {
              "name": "text2",
              "source": "/document/metadata_author"
            },
          ],
          "outputs": [
            {
              "name": "text",
              "targetName": "merged_title_author"
            }
          ]
      }
  ]
}
```

## <a name="next-steps"></a>Étapes suivantes
Félicitations ! Vous avez créé votre première qualification personnalisée. Vous pouvez maintenant suivre le même schéma pour ajouter vos propres fonctionnalités personnalisées. Pour en savoir plus, cliquez sur les liens suivants.

+ [Super compétences : référentiel de compétences personnalisées](https://github.com/Azure-Samples/azure-search-power-skills)
+ [Ajouter une qualification personnalisée à un pipeline d’enrichissement par IA](cognitive-search-custom-skill-interface.md)
+ [Guide pratique pour définir un ensemble de compétences](cognitive-search-defining-skillset.md)
+ [Créer un jeu de compétences (REST)](/rest/api/searchservice/create-skillset)
+ [Guide pratique pour mapper des champs enrichis](cognitive-search-output-field-mapping.md)