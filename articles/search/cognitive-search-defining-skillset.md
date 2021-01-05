---
title: Créer un ensemble de compétences
titleSuffix: Azure Cognitive Search
description: Définissez des étapes d’extraction des données, de traitement du langage naturel ou d’analyse de l’image pour enrichir et extraire des informations structurées à partir de vos données pour les utiliser dans la Recherche cognitive Azure.
manager: nitinme
author: luiscabrer
ms.author: luisca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/04/2019
ms.openlocfilehash: 39a7c92ca6c83684658cf767722698806ed994ec
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88935447"
---
# <a name="how-to-create-a-skillset-in-an-ai-enrichment-pipeline-in-azure-cognitive-search"></a>Comment créer un ensemble de compétences dans un pipeline d’enrichissement de l’IA dans la Recherche cognitive Azure 

![Étapes de l'indexeur](media/cognitive-search-defining-skillset/indexer-stages-skillset.png "étapes de l'indexeur")

Un ensemble de compétences définit les opérations qui extraient et enrichissent des données afin qu’elles puissent faire l’objet de recherches. Un ensemble de compétences est exécuté une fois que le contenu du texte et de l’image est extrait des documents sources et que les champs du document source sont (éventuellement) mappés aux champs de destination d’un index ou d’une base de connaissances.

Un ensemble de compétences contient une ou plusieurs *compétences cognitives* qui représentent une opération d’enrichissement spécifique, par exemple la traduction d’un texte, l’extraction d’expressions clés ou l’exécution d’une reconnaissance optique de caractères à partir d’un fichier image. Pour créer un ensemble de compétences, vous pouvez utiliser les [compétences intégrées](cognitive-search-predefined-skills.md) de Microsoft ou des compétences personnalisées contenant des modèles ou une logique de traitement que vous fournissez (consultez [Exemple : Création d’une compétence personnalisée dans un pipeline d’enrichissement de l’intelligence artificielle](cognitive-search-create-custom-skill-example.md) pour plus d’informations).

Dans cet article, vous allez découvrir comment créer un pipeline d’enrichissement pour les compétences que vous souhaitez utiliser. Un ensemble de compétences est attaché à un [indexeur](search-indexer-overview.md) de Recherche cognitive Azure. Une partie de la conception du pipeline, traitée dans cet article, constitue le jeu de compétences proprement dit. 

> [!NOTE]
> Une autre partie de la conception du pipeline spécifie un indexeur, décrit dans [l’étape suivante](#next-step). Une définition d’indexeur inclut une référence au jeu de compétences, ainsi que les mappages de champs utilisés pour la connexion des entrées aux sorties dans l’index cible.

Points importants à retenir :

+ Vous ne pouvez avoir qu’un jeu de compétences par indexeur.
+ Un ensemble de compétences doit avoir au moins une compétence.
+ Vous pouvez créer plusieurs compétences du même type (par exemple, les variantes d’une compétence d’analyse d’image).

## <a name="begin-with-the-end-in-mind"></a>Commencer en ayant la fin à l’esprit

Nous vous recommandons de choisir d’abord les données à extraire de vos données brutes et le mode d’utilisation souhaité pour ces données dans une solution de recherche. Il peut être utile de créer une illustration du pipeline d’enrichissement complet pour vous aider à identifier les étapes nécessaires.

Par exemple, vous souhaitez traiter un ensemble de commentaires d’analystes financiers. Pour chaque fichier, vous souhaitez extraire les noms de sociétés et la tendance générale des commentaires. Vous souhaiterez peut-être également écrire un enrichisseur personnalisé qui utilise le service Recherche d’entités Bing pour rechercher des informations supplémentaires sur la société, comme le type d’activité de l’entreprise. Sur le fond, vous souhaitez extraire des informations telles que celles qui suivent, qui sont indexées pour chaque document :

| dossier-texte | sociétés | tendance | description de sociétés |
|--------|-----|-----|-----|
|exemple de dossier| [« Microsoft », « LinkedIn »] | 0,99 | [« Microsoft Corporation est une multinationale informatique américaine... », « LinkedIn est un réseau social professionnel... »]

Le diagramme suivant illustre un pipeline d’enrichissement hypothétique :

![Un pipeline d’enrichissement hypothétique](media/cognitive-search-defining-skillset/sample-skillset.png "Un pipeline d’enrichissement hypothétique")


Une fois que vous avez une idée assez claire de ce que vous souhaitez inclure dans le pipeline, vous pouvez représenter le jeu de compétences qui correspond à ces étapes. Au niveau fonctionnel, le jeu de compétences est représenté lorsque vous chargez la définition de l’indexeur dans la Recherche cognitive Azure. Pour en savoir plus sur le chargement de l’indexeur, consultez la [documentation relative à l’indexeur](/rest/api/searchservice/create-indexer).


Dans le diagramme, l’étape de *décodage de document* a lieu automatiquement. La Recherche cognitive Azure sait comment ouvrir des fichiers connus, et crée un champ de *contenu* dont le texte est extrait à partir de chaque document. Les cases blanches sont des enrichisseurs intégrés, et la case « Recherche d’entités Bing » en pointillé représente un enrichisseur personnalisé que vous créez. Comme illustré sur le diagramme, le jeu de compétences contient trois compétences.

## <a name="skillset-definition-in-rest"></a>Définition du jeu de compétences dans REST

Un jeu de compétences est défini comme un tableau de compétences. Chaque compétence définit la source de ses entrées et le nom des sorties générées. À l’aide de [l’API REST Create Skillset](/rest/api/searchservice/create-skillset), vous pouvez définir un jeu de compétences qui correspond au diagramme précédent : 

```http
PUT https://[servicename].search.windows.net/skillsets/[skillset name]?api-version=2020-06-30
api-key: [admin key]
Content-Type: application/json
```

```json
{
  "description": 
  "Extract sentiment from financial records, extract company names, and then find additional information about each company mentioned.",
  "skills":
  [
    {
      "@odata.type": "#Microsoft.Skills.Text.EntityRecognitionSkill",
      "context": "/document",
      "categories": [ "Organization" ],
      "defaultLanguageCode": "en",
      "inputs": [
        {
          "name": "text",
          "source": "/document/content"
        }
      ],
      "outputs": [
        {
          "name": "organizations",
          "targetName": "organizations"
        }
      ]
    },
    {
      "@odata.type": "#Microsoft.Skills.Text.SentimentSkill",
      "inputs": [
        {
          "name": "text",
          "source": "/document/content"
        }
      ],
      "outputs": [
        {
          "name": "score",
          "targetName": "mySentiment"
        }
      ]
    },
    {
      "@odata.type": "#Microsoft.Skills.Custom.WebApiSkill",
     "description": "Calls an Azure function, which in turn calls Bing Entity Search",
      "uri": "https://indexer-e2e-webskill.azurewebsites.net/api/InvokeTextAnalyticsV3?code=foo",
      "httpHeaders": {
          "Ocp-Apim-Subscription-Key": "foobar"
      },
      "context": "/document/organizations/*",
      "inputs": [
        {
          "name": "query",
          "source": "/document/organizations/*"
        }
      ],
      "outputs": [
        {
          "name": "description",
          "targetName": "companyDescription"
        }
      ]
    }
  ]
}
```

## <a name="create-a-skillset"></a>Créer un ensemble de compétences

Lorsque vous créez un jeu de compétences, vous pouvez en indiquer une description pour qu’il se documente automatiquement. Une description d’un jeu de compétences est facultative, mais utile pour en conserver une trace. Comme un jeu de compétences est un document JSON, qui n’autorise pas de commentaires, vous devez utiliser un élément `description` à cet effet.

```json
{
  "description": 
  "This is our first skill set, it extracts sentiment from financial records, extract company names, and then finds additional information about each company mentioned.",
  ...
}
```

L’élément suivant du jeu de compétences est un tableau de compétences. Vous pouvez penser chaque compétence comme étant une primitive d’enrichissement. Chaque compétence effectue une petite tâche dans ce pipeline d’enrichissement. Chacune d’elles accepte une entrée (ou un ensemble d’entrées) et retourne des sorties. Les sections suivantes sont consacrées à la spécification des compétences prédéfinies et personnalisées, chaînant ainsi les compétences ensemble via des références d’entrée et de sortie. Les entrées peuvent provenir de données sources ou d’une autre compétence. Les sorties peuvent être mappées à un champ dans un index de recherche ou utilisées en tant qu’entrée pour une compétence en aval.

## <a name="add-built-in-skills"></a>Ajouter des compétences prédéfinies

Examinons la première compétence, qui est la [compétence de reconnaissance d’entité](cognitive-search-skill-entity-recognition.md) prédéfinie :

```json
    {
      "@odata.type": "#Microsoft.Skills.Text.EntityRecognitionSkill",
      "context": "/document",
      "categories": [ "Organization" ],
      "defaultLanguageCode": "en",
      "inputs": [
        {
          "name": "text",
          "source": "/document/content"
        }
      ],
      "outputs": [
        {
          "name": "organizations",
          "targetName": "organizations"
        }
      ]
    }
```

* Chaque compétence prédéfinie dispose des propriétés `odata.type`, `input` et `output`. Les propriétés propres à une compétence fournissent des informations supplémentaires applicables à cette compétence. Pour la reconnaissance d’entité, `categories` est une entité parmi un ensemble fixe de types d’entité que le modèle préformé peut reconnaître.

* Chaque compétence doit posséder un ```"context"```. Le contexte représente le niveau auquel les opérations ont lieu. Dans la compétence ci-dessus, le contexte représente l’ensemble du document, ce qui implique que la compétence de reconnaissance d’entité est appelée une fois par document. Les sorties sont également générées à ce niveau. Plus spécifiquement, les ```"organizations"``` sont générées en tant que membre de ```"/document"```. Dans les compétences en aval, vous pouvez faire référence à ces informations qui viennent d’être créées sous la forme ```"/document/organizations"```.  Si le champ ```"context"``` n’est pas défini explicitement, le contexte par défaut est le document.

* La compétence possède une entrée appelée « texte », avec une entrée source définie sur ```"/document/content"```. La compétence (reconnaissance d’entité) fonctionne sur le champ *contenu* de chaque document. Il s’agit d’un champ standard créé par l’indexeur des objets blob Azure. 

* La compétence possède une sortie appelée ```"organizations"```. Les sorties existent uniquement pendant le traitement. Pour chaîner cette sortie à l’entrée d’une compétence en aval, référencez la sortie en tant que ```"/document/organizations"```.

* Pour un document particulier, la valeur de ```"/document/organizations"``` est un tableau des organisations extraites du texte. Par exemple :

  ```json
  ["Microsoft", "LinkedIn"]
  ```

Certaines situations demandent de référencer chaque élément d’un tableau séparément. Par exemple, vous souhaitez transmettre chaque élément de ```"/document/organizations"``` séparément à une autre compétence (par exemple, l’enrichisseur personnalisé Recherche d’entités Bing). Vous pouvez faire référence à chaque élément du tableau en ajoutant un astérisque dans le chemin d’accès : ```"/document/organizations/*"``` 

La deuxième compétence correspondant à l’extraction de la tendance suit le même modèle que le premier enrichisseur. Elle dispose de l’entrée ```"/document/content"```, et retourne un score de tendance pour chaque instance de contenu. Comme vous n’avez pas défini le champ ```"context"``` explicitement, la sortie (mySentiment) est maintenant un enfant de ```"/document"```.

```json
    {
      "@odata.type": "#Microsoft.Skills.Text.SentimentSkill",
      "inputs": [
        {
          "name": "text",
          "source": "/document/content"
        }
      ],
      "outputs": [
        {
          "name": "score",
          "targetName": "mySentiment"
        }
      ]
    },
```

## <a name="add-a-custom-skill"></a>Ajouter une compétence personnalisée

Rappelez la structure de l’enrichisseur personnalisé Recherche d’entités Bing :

```json
    {
      "@odata.type": "#Microsoft.Skills.Custom.WebApiSkill",
     "description": "This skill calls an Azure function, which in turn calls Bing Entity Search",
      "uri": "https://indexer-e2e-webskill.azurewebsites.net/api/InvokeTextAnalyticsV3?code=foo",
      "httpHeaders": {
          "Ocp-Apim-Subscription-Key": "foobar"
      },
      "context": "/document/organizations/*",
      "inputs": [
        {
          "name": "query",
          "source": "/document/organizations/*"
        }
      ],
      "outputs": [
        {
          "name": "description",
          "targetName": "companyDescription"
        }
      ]
    }
```

Cette définition est une [compétence personnalisée](cognitive-search-custom-skill-web-api.md) qui appelle une API web dans le cadre du processus d'enrichissement. Pour chaque organisation identifiée par la reconnaissance d’entité, cette compétence appelle une API web pour rechercher la description de cette organisation. L’orchestration du moment auquel appeler l’API web et du traitement des informations reçues est gérée en interne par le moteur d’enrichissement. Toutefois, l’initialisation nécessaire pour appeler cette API personnalisée doit être indiquée dans le fichier JSON (par exemple, l’URI, les en-têtes HTTP et les entrées attendus). Pour obtenir des conseils sur la création d’une API web personnalisée pour le pipeline d’enrichissement, consultez [Guide pratique pour définir une interface personnalisée](cognitive-search-custom-skill-interface.md).

Notez que le champ « contexte » contient la valeur ```"/document/organizations/*"``` avec un astérisque, ce qui signifie que l’étape d’enrichissement est appelée *pour chaque* organisation sous ```"/document/organizations"```. 

La sortie, dans ce cas une description de société, est générée pour chaque organisation identifiée. Lorsque vous faites référence à la description d’une étape en aval (par exemple, dans l’extraction d’expressions clés), vous utilisez le chemin d’accès ```"/document/organizations/*/description"``` pour ce faire. 

## <a name="add-structure"></a>Ajouter une structure

Le jeu de compétences génère des informations structurées à partir de données non structurées. Prenons l’exemple suivant :

*« In its fourth quarter, Microsoft logged $1.1 billion in revenue from LinkedIn, the social networking company it bought last year. The acquisition enables Microsoft to combine LinkedIn capabilities with its CRM and Office capabilities. Stockholders are excited with the progress so far. »*

Un résultat probable serait une structure générée semblable à l’illustration suivante :

![Exemple de structure de sortie](media/cognitive-search-defining-skillset/enriched-doc.png "Exemple de structure de sortie")

Jusqu’à présent, cette structure est interne uniquement, en mémoire seulement et utilisée uniquement dans les index de Recherche cognitive Azure. L’ajout d’une base de connaissances vous permet d’enregistrer les enrichissements mis en forme pour une utilisation hors de la recherche.

## <a name="add-a-knowledge-store"></a>Ajouter une base de connaissances

La [Base de connaissances](knowledge-store-concept-intro.md) est une fonctionnalité de Recherche cognitive Azure destinée à l’enregistrement de vos documents enrichis. La base de connaissances que vous créez à l’aide d’un compte de stockage Azure est le référentiel dans lequel vos données enrichies résident. 

Une définition de la base de connaissances est ajoutée à un ensemble de compétences. Pour obtenir une procédure pas à pas de l’ensemble du processus, consultez la section [Créer une base de connaissances dans REST](knowledge-store-create-rest.md).

```json
"knowledgeStore": {
  "storageConnectionString": "<an Azure storage connection string>",
  "projections" : [
    {
      "tables": [ ]
    },
    {
      "objects": [
        {
          "storageContainer": "containername",
          "source": "/document/EnrichedShape/",
          "key": "/document/Id"
        }
      ]
    }
  ]
}
```

Vous pouvez choisir d’enregistrer les documents enrichis sous forme de tables avec des relations hiérarchiques conservées ou sous forme de documents JSON dans le stockage blob. Toute sortie à partir des compétences de l’ensemble de compétences peut servir de source d’entrée de la projection. Si vous cherchez à projeter les données dans une forme spécifique, la [compétence de modélisateur](cognitive-search-skill-shaper.md) mise à jour peut maintenant modéliser les types complexes que vous utilisez. 

<a name="next-step"></a>

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous connaissez le pipeline d’enrichissement et les jeux de compétences, poursuivez avec [How to reference annotations in a skillset](cognitive-search-concept-annotations-syntax.md) (Référencement des annotations dans un jeu de compétences) ou [How to map outputs to fields in an index](cognitive-search-output-field-mapping.md) (Mappage de sorties à des champs dans un index).