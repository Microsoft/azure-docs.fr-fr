---
title: Concepts et flux de travail des ensembles de compétences
titleSuffix: Azure Cognitive Search
description: Les ensembles de compétences consistent à créer un pipeline d’enrichissement de l’IA dans Recherche cognitive Azure. Découvrez les concepts et détails importants relatifs à la composition d’ensembles de compétences.
manager: nitinme
author: vkurpad
ms.author: vikurpad
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/15/2020
ms.openlocfilehash: b5a893ee1923ba4b2bec53b20fb164337bd65902
ms.sourcegitcommit: 65a4f2a297639811426a4f27c918ac8b10750d81
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96558111"
---
# <a name="skillset-concepts-in-azure-cognitive-search"></a>Concepts des ensembles de compétences dans Recherche cognitive Azure

Cet article est destiné aux développeurs qui ont besoin d’une compréhension plus approfondie des concepts et de la composition des compétences. Il suppose une bonne connaissance du processus d’enrichissement par IA. Si vous ne connaissez pas ce concept, commencez par [Enrichissement par IA dans Azure Recherche cognitive](cognitive-search-concept-intro.md).

## <a name="introducing-skillsets"></a>Présentation des ensembles de compétences

Un ensemble de compétences est une ressource réutilisable dans Recherche cognitive Azure et qui est liée à un indexeur. Cette ressource spécifie une collection de compétences servant à analyser, transformer et enrichir du texte ou des images durant l’indexation. Les compétences ont des entrées et des sorties, et souvent la sortie d’une compétence devient l’entrée d’une autre dans une chaîne ou une séquence de processus.

Chaque ensemble de compétences possède trois propriétés principales :

+ `skills`, une collection non triée de compétences dont la séquence d’exécution est déterminée par la plateforme en fonction des entrées requises pour chaque compétence.
+ `cognitiveServices`, la clé d’une ressource Cognitive Services qui effectue le traitement des images et du texte pour les compétences qui incluent des compétences intégrées.
+ `knowledgeStore` (facultatif), le compte de stockage Azure dans lequel seront projetés vos documents. Les documents enrichis sont également utilisés par les index de recherche.

Les ensembles de compétences sont créés dans JSON. L’exemple suivant est une version légèrement simplifiée de l’[ensemble de compétences hotel-reviews](https://github.com/Azure-Samples/azure-search-sample-data/blob/master/hotelreviews/HotelReviews_skillset.json) utilisée pour illustrer les concepts de cet article. 

Les deux premières compétences sont présentées ci-dessous :

+ La compétence 1 est une [compétence de fractionnement de texte](cognitive-search-skill-textsplit.md) qui accepte le contenu du champ « reviews_text » comme entrée et divise ce contenu en « pages » de 5 000 caractères en sortie.
+ La compétence 2 est une [compétence de détection des sentiments](cognitive-search-skill-sentiment.md) qui accepte « pages » comme entrée et génère un nouveau champ appelé « Sentiment » qui contient les résultats de l’analyse des sentiments.


```json
{
    "skills": [
        {
            "@odata.type": "#Microsoft.Skills.Text.SplitSkill",
            "name": "#1",
            "description": null,
            "context": "/document/reviews_text",
            "defaultLanguageCode": "en",
            "textSplitMode": "pages",
            "maximumPageLength": 5000,
            "inputs": [
                {
                    "name": "text",
                    "source": "/document/reviews_text"
                }
            ],
            "outputs": [
                {
                    "name": "textItems",
                    "targetName": "pages"
                }
            ]
        },
        {
            "@odata.type": "#Microsoft.Skills.Text.SentimentSkill",
            "name": "#2",
            "description": null,
            "context": "/document/reviews_text/pages/*",
            "defaultLanguageCode": "en",
            "inputs": [
                {
                    "name": "text",
                    "source": "/document/reviews_text/pages/*",
                }
            ],
            "outputs": [
                {
                    "name": "score",
                    "targetName": "Sentiment"
                }
            ]
        },
  "cognitiveServices": null,
  "knowledgeStore": {  }
}
```
> [!NOTE]
> Vous pouvez créer des ensembles de compétences complexes, avec des boucles et des branches, à l’aide de la [compétence conditionnelle](cognitive-search-skill-conditional.md) pour créer des expressions. La syntaxe se base sur la notation du [pointeur JSON](https://tools.ietf.org/html/rfc6901) pour les chemins, légèrement modifiée afin d’identifier les nœuds dans l’arborescence d’enrichissements. Un `"/"` fait passer à un niveau inférieur dans l’arborescence et `"*"` est utilisé comme un opérateur for-each dans le contexte. De nombreux exemples de cet article illustrent la syntaxe. 

### <a name="enrichment-tree"></a>Arborescence d’enrichissements

Dans la progression des [étapes du pipeline d’enrichissement](cognitive-search-concept-intro.md#enrichment-steps), le traitement du contenu suit la phase de *craquage de document*, au cours de laquelle le texte et les images sont extraits de la source. Le contenu des images peut ensuite être acheminé vers des compétences qui spécifient le traitement des images, tandis que le contenu texte est mis en file d’attente pour le traitement du texte. Pour les documents sources qui contiennent de grandes quantités de texte, vous pouvez définir un *mode d’analyse* sur l’indexeur pour découper le texte en plus petits segments pour un traitement optimal. 

![Diagramme de base de connaissances au sein d’un pipeline](./media/knowledge-store-concept-intro/knowledge-store-concept-intro.svg "Diagramme de base de connaissances au sein d’un pipeline")

Une fois qu’un document se trouve dans le pipeline d’enrichissement, il est représenté sous la forme d’une arborescence du contenu et des enrichissements associés. Cette arborescence est instanciée en tant que sortie du craquage du document.  Le format de l’arborescence d’enrichissements permet au pipeline d’enrichissement d’attacher des métadonnées même à des types de données primitifs ; ce n’est pas un objet JSON valide, mais il peut être projeté dans un format JSON valide. Le tableau suivant indique l’état d’un document qui entre dans le pipeline d’enrichissement :

|Data Source\Parsing Mode|Default|JSON, JSON Lines & CSV|
|---|---|---|
|Stockage Blob|/document/content<br>/document/normalized_images/*<br>…|/document/{key1}<br>/document/{key2}<br>…|
|SQL|/document/{column1}<br>/document/{column2}<br>…|N/A |
|Cosmos DB|/document/{key1}<br>/document/{key2}<br>…|N/A|

 À mesure que les compétences s’exécutent, elles ajoutent de nouveaux nœuds à l’arborescence d’enrichissements. Ces nouveaux nœuds peuvent ensuite être utilisés comme entrées pour les compétences en aval, en les projetant dans la base de connaissances ou en les mappant aux champs d’index. Les enrichissements ne sont pas mutables : une fois créés, les nœuds ne peuvent pas être modifiés. Plus votre ensemble de compétences est complexe, plus votre arborescence d’enrichissements l’est aussi. Toutefois, vous n’avez pas besoin d’inclure systématiquement tous les nœuds de l’arborescence d’enrichissements dans l’index ou la base de connaissances. 

Vous pouvez choisir de conserver uniquement une partie des enrichissements dans l’index ou la base de connaissances.

### <a name="context"></a>Context

Chaque compétence demande un contexte. Un contexte détermine :

+ Le nombre de fois que la compétence est exécutée, en fonction des nœuds sélectionnés. Pour les valeurs de contexte d’une collection, l’ajout de `/*` à la fin spécifie que la compétence est appelée une fois pour chaque instance dans la collection. 

+ L’endroit dans l’arborescence d’enrichissements où les sorties de la compétence sont ajoutées. Les sorties sont toujours ajoutées à l’arborescence en tant qu’enfants du nœud de contexte. 

+ La forme des entrées. Pour les collections à plusieurs niveaux, la définition du contexte sur la collection parente détermine la forme de l’entrée de la compétence. Par exemple, dans une arborescence d’enrichissements avec une liste de pays/régions, chaque entrée est enrichie avec une liste d’États contenant elle-même une liste de codes postaux.

|Context|Entrée|Forme de l’entrée|Appel de compétence|
|-------|-----|--------------|----------------|
|`/document/countries/*` |`/document/countries/*/states/*/zipcodes/*` |Liste de tous les codes postaux du pays/de la région |Une fois par pays/région |
|`/document/countries/*/states/*` |`/document/countries/ */states/* /zipcodes/*`` |Liste de tous les codes postaux de l’état | Une fois par combinaison pays/région et État|

## <a name="generate-enriched-data"></a>Générer des données enrichies 

À l’aide de l’[ensemble de compétences hotel-reviews](https://github.com/Azure-Samples/azure-search-sample-data/blob/master/hotelreviews/HotelReviews_skillset.json) comme point de référence, nous allons examiner les éléments suivants :

+ L’arborescence d’enrichissements évolue au fur et à mesure de l’exécution de chaque compétence.
+ Le contexte et les entrées déterminent le nombre de fois qu’une compétence s’exécute.
+ Le contexte affecte la forme de l’entrée.

Un « document » au sein du processus d’enrichissement est une ligne (un avis sur un hôtel) dans le fichier source hotel_reviews.csv.

### <a name="skill-1-split-skill"></a>Compétence n° 1 : Division

Lorsque le contenu source est constitué de gros blocs de texte, il est utile de le diviser en plus petits composants pour plus de précision dans la détection de la langue, du sentiment et des expressions clés. Deux grains sont disponibles : les pages et les phrases. Une page se compose d’environ 5000 caractères.

La compétence de fractionnement de texte se trouve généralement d’abord dans un ensemble de compétences.

```json
      "@odata.type": "#Microsoft.Skills.Text.SplitSkill",
      "name": "#1",
      "description": null,
      "context": "/document/reviews_text",
      "defaultLanguageCode": "en",
      "textSplitMode": "pages",
      "maximumPageLength": 5000,
      "inputs": [
        {
          "name": "text",
          "source": "/document/reviews_text"
        }
      ],
      "outputs": [
        {
          "name": "textItems",
          "targetName": "pages"
        }
```

Avec le contexte de compétence `"/document/reviews_text"`, la compétence de fractionnement s’exécute une fois pour `reviews_text`. La sortie de la compétence est une liste où `reviews_text` est segmenté en 5 000 séquences de caractères. La sortie de la compétence de division est nommée `pages` et elle est ajoutée à l’arborescence d’enrichissements. Avec `targetName`, vous pouvez renommer une sortie de compétence avant de l’ajouter à l’arborescence d’enrichissements.

L’arborescence d’enrichissements comporte maintenant un nouveau nœud, situé sous le contexte de la compétence. Ce nœud peut être utilisé pour d’autres compétences, projections ou mappages de champs de sortie. Conceptuellement, l’arborescence ressemble à ce qui suit :

![arborescence d’enrichissement après le craquage de document](media/cognitive-search-working-with-skillsets/enrichment-tree-doc-cracking.png "Arborescence d’enrichissement après le craquage de document et avant l’exécution de la compétence")

Le nœud racine de tous les enrichissements est `"/document"`. Quand vous utilisez des indexeurs d’objets blob, le nœud `"/document"` contient les nœuds enfants `"/document/content"` et `"/document/normalized_images"`. Si vous utilisez des données CSV, comme dans cet exemple, les noms de colonne sont mappés aux nœuds figurant sous `"/document"`. 

Pour accéder à un enrichissement qui a été ajouté à un nœud par une compétence, vous devez indiquer le chemin complet de l’enrichissement. Par exemple, si vous souhaitez utiliser le texte du nœud ```pages``` comme entrée dans une autre compétence, vous devez spécifier le chemin de cette façon : ```"/document/reviews_text/pages/*"```.
 
 ![arborescence d’enrichissement après la compétence n° 1](media/cognitive-search-working-with-skillsets/enrichment-tree-skill1.png "L’arborescence d’enrichissement après la compétence n° 1 s’exécute")

### <a name="skill-2-language-detection"></a>Compétence n° 2 : Détection de la langue

Les documents d’avis sur les hôtels comprennent des commentaires clients exprimés en plusieurs langues. La compétence de détection de langue détermine la langue utilisée. Le résultat est ensuite transmis à l’extraction d’expressions clés et à la détection de sentiments, en prenant en considération la langue lors de la détection des sentiments et des expressions.

La compétence de détection de la langue est la troisième compétence (compétence n° 3) définie dans l’ensemble de compétences, mais c’est la compétence suivante à exécuter. Comme elle n’est pas bloquée dans l’attente d’entrées, elle s’exécute parallèlement à la compétence précédente. À l’instar de la compétence de division qui l’a précédée, la compétence de détection de la langue est également appelée une fois pour chaque document. L’arborescence d’enrichissements comporte désormais un nouveau nœud pour la langue.

 ![arborescence d’enrichissement après la compétence n° 2](media/cognitive-search-working-with-skillsets/enrichment-tree-skill2.png "Arborescence d’enrichissement après exécution de la compétence n°2")
 
 ### <a name="skill-3-key-phrases-skill"></a>Compétence n° 3 : Expressions clés 

Avec le contexte `/document/reviews_text/pages/*`, la compétence des expressions clés est appelée une fois pour chacun des éléments dans la collection `pages`. La sortie de la compétence est un nœud placé sous l’élément page associé. 

 Vous pouvez maintenant examiner le reste des compétences dans l’ensemble de compétences et regarder comment l’arborescence des enrichissements continue de croître à l’exécution de chaque compétence. Certaines compétences, telles que la compétence de fusion et la compétence de modélisation, créent également des nœuds, mais utilisent uniquement les données de nœuds existants et ne créent pas d’enrichissements supplémentaires.

![arborescence d’enrichissement après toutes les compétences](media/cognitive-search-working-with-skillsets/enrichment-tree-final.png "Arborescence d’enrichissement après toutes les compétences")

Les couleurs des connecteurs dans l’arborescence ci-dessus indiquent que les enrichissements ont été créés par différentes compétences, c’est-à-dire que les nœuds devront être traités individuellement et qu’ils ne feront pas partie de l’objet retourné lors de la sélection du nœud parent.

## <a name="save-enrichments"></a>Enregistrer les enrichissements

Dans Recherche cognitive Azure, l’indexeur enregistre la sortie qu’il crée. L’une des sorties est toujours un [index pouvant faire l’objet d’une recherche](search-what-is-an-index.md). La spécification d’un index est une exigence et, lorsque vous attachez un ensemble de compétences, les données ingérées par l’index incluent la substance des enrichissements. En règle générale, les sorties de compétences spécifiques, comme les expressions clés ou les scores de sentiment, sont ingérées dans l’index d’un champ créé à cet effet.

Si vous le souhaitez, l’indexeur peut également envoyer la sortie vers une [base de connaissances](knowledge-store-concept-intro.md) à des fins de consommation dans d’autres outils ou processus. La base de connaissances est définie dans le cadre de l’ensemble de compétences. Sa définition détermine si vos documents enrichis sont projetés sous forme de tables ou d’objets (fichiers ou objets blob). Les projections tabulaires sont bien adaptées à l’analyse interactive dans des outils tels que Power BI, tandis que les fichiers et les objets blob sont généralement utilisés en science des données ou dans des processus similaires. Dans cette section, vous allez apprendre comment la composition des ensembles de compétences peut mettre en forme les tables ou les objets que vous souhaitez projeter.

### <a name="projections"></a>Projections

Pour le contenu qui cible une base de connaissances, vous devez prendre en compte la façon dont le contenu est structuré. La *projection* est le processus qui consiste à sélectionner les nœuds de l’arborescence d’enrichissements et à en créer une expression physique dans la base de connaissances. Les projections sont des formes personnalisées du document (contenu et enrichissements) qui peuvent être générées en sortie sous forme de projections de tables ou d’objets. Pour en savoir plus sur l’utilisation des projections, consultez [Utilisation de projections](knowledge-store-projection-overview.md).

![Options de mappage de champs](./media/cognitive-search-working-with-skillsets/field-mapping-options.png "Options de mappage de champs pour le pipeline d’enrichissement")

### <a name="sourcecontext"></a>SourceContext

L’élément `sourceContext` s’utilise uniquement dans les entrées de compétence et les projections. Il permet de construire des objets imbriqués à plusieurs niveaux. Vous devrez peut-être créer un objet pour le transmettre en tant qu’entrée à une compétence ou à un projet dans la base de connaissances. Étant donné que des nœuds d’enrichissement peuvent ne pas être des objets JSON valides dans l’arborescence d’enrichissement et que le référencement d’un nœud dans l’arborescence retourne uniquement cet état du nœud lors de sa création, l’utilisation d’enrichissements en guise d’entrées ou projections de compétence vous oblige à créer un objet JSON bien formé. Avec `sourceContext`, vous pouvez construire un objet hiérarchique de type anonyme, ce qui nécessiterait plusieurs compétences si vous utilisiez uniquement le contexte. 

L’utilisation de `sourceContext` est illustré dans l’exemple suivant. Examinez la sortie de compétence qui a généré un enrichissement afin de déterminer s’il s’agit d’un objet JSON valide et non d’un type primitif.

### <a name="slicing-projections"></a>Découpage de projections

Quand vous définissez un groupe de projections de tables, un nœud de l’arborescence d’enrichissements peut être divisé pour être projeté dans plusieurs tables associées. Si vous ajoutez une table dont le chemin source est un nœud enfant d’une projection de table existante, le nœud enfant qui en résulte n’est pas un enfant de cette projection, mais il est projeté à la place dans la nouvelle table associée. Cette technique de découpage vous permet de définir un nœud unique dans une compétence de modélisation qui peut être la source de toutes vos projections de tables. 

### <a name="shaping-projections"></a>Mise en forme de projections

Il existe deux façons de définir une projection :

+ Utilisez la modélisation de texte pour créer un nœud qui est le nœud racine pour tous les enrichissements que vous projetez. Ensuite, dans vos projections, vous référencez uniquement la sortie de la compétence de modélisation.

+ Utilisez la mise en forme d’une projection dans la définition de la projection.

L’approche de la modélisation est plus détaillée que la mise en forme incluse, mais elle garantit que toutes les mutations de l’arborescence d’enrichissements sont présentes dans les compétences et que la sortie est un objet réutilisable. Par contre, la mise en forme incluse vous permet de créer la forme dont vous avez besoin, mais elle constitue un objet anonyme qui est utilisable uniquement dans la projection pour laquelle elle est définie. Les approches peuvent s’utiliser ensemble ou séparément. L’ensemble de compétences créé pour vous dans le workflow du portail contient les deux. Il utilise une compétence de modélisation pour les projections de tables, mais aussi une mise en forme incluse pour projeter la table d’expressions clés.

Pour compléter l’exemple, vous pouvez choisir de supprimer la mise en forme incluse et d’utiliser une compétence de modélisation afin de créer un nœud spécifique pour les expressions clés. Pour créer une forme projetée dans trois tables (`hotelReviewsDocument`, `hotelReviewsPages` et `hotelReviewsKeyPhrases`), vous utilisez les deux options décrites dans les sections suivantes.

#### <a name="shaper-skill-and-projection"></a>Compétence de modélisation et projection

> [!Note]
> Certaines colonnes de la table de documents ont été supprimées de cet exemple par souci de concision.
>
```json
{
    "@odata.type": "#Microsoft.Skills.Util.ShaperSkill",
    "name": "#5",
    "description": null,
    "context": "/document",
    "inputs": [        
        {
            "name": "reviews_text",
            "source": "/document/reviews_text",
            "sourceContext": null,
            "inputs": []
        },
        {
            "name": "reviews_title",
            "source": "/document/reviews_title",
            "sourceContext": null,
            "inputs": []
        },
        {
            "name": "AzureSearch_DocumentKey",
            "source": "/document/AzureSearch_DocumentKey",
            "sourceContext": null,
            "inputs": []
        },  
        {
            "name": "pages",
            "source": null,
            "sourceContext": "/document/reviews_text/pages/*",
            "inputs": [
                {
                    "name": "SentimentScore",
                    "source": "/document/reviews_text/pages/*/Sentiment",
                    "sourceContext": null,
                    "inputs": []
                },
                {
                    "name": "LanguageCode",
                    "source": "/document/Language",
                    "sourceContext": null,
                    "inputs": []
                },
                {
                    "name": "Page",
                    "source": "/document/reviews_text/pages/*",
                    "sourceContext": null,
                    "inputs": []
                },
                {
                    "name": "keyphrase",
                    "sourceContext": "/document/reviews_text/pages/*/Keyphrases/*",
                    "inputs": [
                        {
                            "source": "/document/reviews_text/pages/*/Keyphrases/*",
                            "name": "Keyphrases"
                        }
                    ]
                }
            ]
        }
    ],
    "outputs": [
        {
            "name": "output",
            "targetName": "tableprojection"
        }
    ]
}
```

Avec le nœud `tableprojection` défini dans la section `outputs` ci-dessus, nous pouvons maintenant utiliser la fonctionnalité de découpage pour projeter les éléments du nœud `tableprojection` dans des tables différentes :

> [!Note]
> Il s’agit uniquement d’un extrait de la projection configurée dans la base de connaissances.
>
```json
"projections": [
    {
        "tables": [
            {
                "tableName": "hotelReviewsDocument",
                "generatedKeyName": "Documentid",
                "source": "/document/tableprojection"
            },
            {
                "tableName": "hotelReviewsPages",
                "generatedKeyName": "Pagesid",
                "source": "/document/tableprojection/pages/*"
            },
            {
                "tableName": "hotelReviewsKeyPhrases",
                "generatedKeyName": "KeyPhrasesid",
                "source": "/document/tableprojection/pages/*/keyphrase/*"
            }
        ]
    }
]
```

#### <a name="inline-shaping-projections"></a>Mise en forme incluse pour les projections

L’approche de la mise en forme incluse ne nécessite pas de compétence de modélisation, car toutes les formes nécessaires aux projections sont créées au fur et à mesure des besoins. Pour projeter les mêmes données que l’exemple précédent, nous utilisons une option de projection incluse similaire à ceci :

```json
"projections": [
    {
        "tables": [
            {
                "tableName": "hotelReviewsInlineDocument",
                "generatedKeyName": "Documentid",
                "sourceContext": "/document",     
                "inputs": [
                    {
                        "name": "reviews_text",
                        "source": "/document/reviews_text"
                    },
                    {
                        "name": "reviews_title",
                        "source": "/document/reviews_title"
                    },
                    {
                        "name": "AzureSearch_DocumentKey",
                        "source": "/document/AzureSearch_DocumentKey"
                    }                             
                ]
            },
            {
                "tableName": "hotelReviewsInlinePages",
                "generatedKeyName": "Pagesid",
                "sourceContext": "/document/reviews_text/pages/*",
                "inputs": [
                        {
                    "name": "SentimentScore",
                    "source": "/document/reviews_text/pages/*/Sentiment"
                    },
                    {
                        "name": "LanguageCode",
                        "source": "/document/Language"
                    },
                    {
                        "name": "Page",
                        "source": "/document/reviews_text/pages/*"
                    }
                ]
            },
            {
                "tableName": "hotelReviewsInlineKeyPhrases",
                "generatedKeyName": "KeyPhraseId",
                "sourceContext": "/document/reviews_text/pages/*/Keyphrases/*",
                "inputs": [
                    {
                        "name": "Keyphrases",
                        "source": "/document/reviews_text/pages/*/Keyphrases/*"
                    }
                ]
            }
        ]
    }
]
```
  
Dans les deux approches, nous pouvons observer comment les valeurs de `"Keyphrases"` sont projetées à l’aide de `"sourceContext"`. Le nœud `"Keyphrases"`, qui contient une collection de chaînes, est lui-même un enfant du texte de la page. Toutefois, étant donné que les projections requièrent un objet JSON et que la page est de type primitif (chaîne), `"sourceContext"` est utilisé pour inclure l’expression clé dans un objet avec une propriété nommée. Cette technique permet aussi de projeter des primitives de manière indépendante.

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez maintenant créer votre premier ensemble de compétences avec des compétences cognitives.

> [!div class="nextstepaction"]
> [Créez votre premier ensemble de compétences](cognitive-search-defining-skillset.md).
