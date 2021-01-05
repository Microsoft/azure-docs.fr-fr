---
title: Analyseurs pour la linguistique et le traitement de texte
titleSuffix: Azure Cognitive Search
description: Attribuez des analyseurs aux champs de texte d’un index pouvant faire l’objet d’une recherche, afin de remplacer l’instance de Lucene standard par défaut par des solutions alternatives personnalisées, prédéfinies ou spécifiques à une langue.
author: HeidiSteen
manager: nitinme
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/20/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 544509a8c90c9273b748591509b1fa86510d71c3
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96013817"
---
# <a name="analyzers-for-text-processing-in-azure-cognitive-search"></a>Analyseurs pour le traitement de texte dans la Recherche cognitive Azure

Un *analyseur* est un composant du [moteur de recherche en texte intégral](search-lucene-query-architecture.md) chargé de traiter le texte dans les chaînes de requête et les documents indexés. Le traitement du texte (également appelé analyse lexicale) est à l’origine de transformations. Il modifie une chaîne via des actions telles que celles-ci :

+ Supprimer les mots (mots vides) et la ponctuation non essentiels
+ Segmenter des expressions et des mots avec tirets en différents composants
+ Mettre les mots en majuscules en minuscules
+ Réduire les mots dans des formes racines primitives pour une efficacité de stockage et de sorte que des correspondances puissent être trouvées, quels que soient les temps

L’analyse s’applique à des champs `Edm.String` marqués comme étant « interrogeables », ce qui indique une recherche en texte intégral. Pour les champs avec cette configuration, l’analyse a lieu pendant l’indexation lors de la création de jetons, puis à nouveau lors de l’exécution de requêtes lorsque ces dernières sont analysées et que le moteur recherche les jetons correspondants. Une correspondance est plus probable lorsque le même analyseur est utilisé à la fois pour l’indexation et pour les requêtes, mais vous pouvez définir l’analyseur de manière indépendante pour chaque charge de travail, en fonction de vos besoins.

Les types de requêtes qui ne sont pas une recherche en texte intégral, tels que l’expression régulière ou la recherche approximative, ne passent pas par la phase d’analyse du côté de la requête. Au lieu de cela, l’analyseur envoie ces chaînes directement au moteur de recherche, à l’aide du modèle que vous fournissez comme base pour la correspondance. Ces formulaires de requête nécessitent généralement des jetons de chaîne entière pour effectuer un travail de correspondance de modèle. Pour recevoir des jetons de termes complets pendant l’indexation, vous aurez peut-être besoin d’[analyseurs personnalisés](index-add-custom-analyzers.md). Pour plus d’informations sur le moment et la raison de l’analyse des termes de la requête, consultez [Recherche en texte intégral dans la Recherche cognitive Azure](search-lucene-query-architecture.md).

Pour plus d’informations sur l’analyse lexicale, regardez le clip vidéo suivant, qui donne une brève explication.

> [!VIDEO https://www.youtube.com/embed/Y_X6USgvB1g?version=3&start=132&end=189]

## <a name="default-analyzer"></a>Analyseur par défaut  

Dans les requêtes de la Recherche cognitive Azure, un analyseur est automatiquement appelé sur tous les champs de chaînes marqués comme pouvant faire l’objet d’une recherche. 

Par défaut, la Recherche cognitive Azure utilise l’[analyseur Apache Lucene Standard (lucene standard)](https://lucene.apache.org/core/6_6_1/core/org/apache/lucene/analysis/standard/StandardAnalyzer.html), qui décompose le texte en éléments en suivant les règles de la [« Segmentation du texte Unicode »](https://unicode.org/reports/tr29/). Par ailleurs, l'analyseur standard convertit tous les caractères en minuscules. Les documents indexés et les termes de recherche sont analysés pendant l'indexation et le traitement des requêtes.  

Vous pouvez substituer l’analyseur par défaut champ par champ. Ces analyseurs alternatifs peuvent être un [analyseur linguistique](index-add-language-analyzers.md) pour le traitement linguistique, un [analyseur personnalisé](index-add-custom-analyzers.md) ou un analyseur prédéfini figurant dans la [liste des analyseurs disponibles](index-add-custom-analyzers.md#AnalyzerTable).

## <a name="types-of-analyzers"></a>Types d’analyseurs

La liste suivante décrit les analyseurs disponibles dans la Recherche cognitive Azure.

| Category | Description |
|----------|-------------|
| [Analyseur Lucene standard](https://lucene.apache.org/core/6_6_1/core/org/apache/lucene/analysis/standard/StandardAnalyzer.html) | Par défaut. Aucune spécification ou configuration n’est nécessaire. Cet analyseur à usage général est efficace pour de nombreux scénarios et langues.|
| Analyseurs prédéfinis | Proposés comme produits finis destinés à être utilisés tels quels. <br/>Il en existe deux types : spécialisé et linguistique. Ils sont dits « prédéfinis », car vous les référencez par leur nom, sans aucune configuration ni personnalisation. <br/><br/>Les [analyseurs spécialisés (non dépendants de la langue)](index-add-custom-analyzers.md#AnalyzerTable) sont employés quand les entrées de texte nécessitent un traitement spécialisé ou minimal. Les analyseurs prédéfinis qui ne dépendent pas de la langue sont les suivants : **Asciifolding**, **Keyword**, **Pattern**, **Simple**, **Stop**, **Whitespace**.<br/><br/>Utilisez les [analyseurs linguistiques](index-add-language-analyzers.md) quand vous avez besoin d’une prise en charge linguistique avancée pour différentes langues. La Recherche cognitive Azure prend en charge 35 analyseurs linguistiques Lucene et 50 analyseurs de traitement en langage naturel Microsoft. |
|[Analyseurs personnalisés](/rest/api/searchservice/Custom-analyzers-in-Azure-Search) | Ils renvoient à une configuration définie par l’utilisateur constituée d’une combinaison d’éléments existants, dont un générateur de jetons (obligatoire) et des filtres facultatifs (caractères ou jetons).|

Certains analyseurs prédéfinis, comme **Pattern** ou **Stop**, prennent en charge un ensemble limité d’options de configuration. Pour définir ces options, vous créez en réalité un analyseur personnalisé constitué de l’analyseur prédéfini et d’une des autres options documentées dans les [informations de référence sur les analyseurs prédéfinis](index-add-custom-analyzers.md#AnalyzerTable). Comme pour toute configuration personnalisée, nommez votre nouvelle configuration (par exemple, *MonAnalyseurSéquencesOctets*) pour la distinguer de l’analyseur Lucene Pattern.

## <a name="how-to-specify-analyzers"></a>Comment spécifier des analyseurs

La définition d’un analyseur est facultative. En règle générale, essayez d’abord d’utiliser l’analyseur Lucene standard par défaut pour voir comment il fonctionne. Si les requêtes ne renvoient pas les résultats attendus, le passage à un autre analyseur est souvent la bonne solution.

1. Lors de la création d’une définition de champ dans l’[index](/rest/api/searchservice/create-index) , définissez la propriété **analyzer** sur l’un des éléments suivants : un [analyseur prédéfini](index-add-custom-analyzers.md#AnalyzerTable) tel que `keyword`, un [analyseur de langage](index-add-language-analyzers.md) tel que `en.microsoft` ou un analyseur personnalisé (défini dans le même schéma d’index).  
 
   ```json
     "fields": [
    {
      "name": "Description",
      "type": "Edm.String",
      "retrievable": true,
      "searchable": true,
      "analyzer": "en.microsoft",
      "indexAnalyzer": null,
      "searchAnalyzer": null
    },
   ```

   Si vous utilisez un [analyseur de langage](index-add-language-analyzers.md), vous devez utiliser la propriété **analyzer** pour le spécifier. Les propriétés **searchAnalyzer** et **indexAnalyzer** ne prennent pas en charge les analyseurs de langage.

1. Vous pouvez également définir **indexAnalyzer** et **searchAnalyzer** afin de changer d’analyseur pour chaque charge de travail. Ces propriétés sont définies ensemble et remplacent la propriété **analyzer**, qui doit avoir la valeur Null. Vous pouvez utiliser différents analyseurs pour la préparation et la récupération des données si l’une de ces activités nécessite une transformation spécifique et l’autre non.

   ```json
     "fields": [
    {
      "name": "Description",
      "type": "Edm.String",
      "retrievable": true,
      "searchable": true,
      "analyzer": null,
      "indexAnalyzer": "keyword",
      "searchAnalyzer": "whitespace"
    },
   ```

1. Pour les analyseurs personnalisés uniquement, créez une entrée dans la section **[analyseurs]** de l’index, puis affectez votre analyseur personnalisé à la définition de champ pour l’une ou l’autre des deux étapes précédentes. Pour plus d’informations, consultez [Créer un index](/rest/api/searchservice/create-index) et [Ajouter des analyseurs personnalisés](index-add-custom-analyzers.md).

## <a name="when-to-add-analyzers"></a>Quand ajouter des analyseurs

La phase de développement actif est le meilleur moment pour ajouter et affecter des analyseurs, là où la suppression et la recréation d’index sont des activités courantes.

Étant donné que les analyseurs sont utilisés pour créer des jetons pour les termes, vous devez affecter un analyseur lors de la création du champ. En fait, l’affectation d’**analyzer** ou d’ **indexAnalyzer** à un champ déjà créé physiquement n’est pas autorisée (bien que vous puissiez modifier la propriété **searchAnalyzer** à tout moment sans aucun impact sur l’index).

Pour modifier l’analyseur d’un champ existant, vous devez [recréer entièrement l’index](search-howto-reindex.md) (vous ne pouvez pas recréer des champs individuels). Pour les index en production, vous devez retarder une regénération en créant un champ avec la nouvelle affectation d’analyseur et commencer à l’utiliser à la place de l’ancien. Utilisez [Mettre à jour l’index](/rest/api/searchservice/update-index) pour incorporer le nouveau champ et [mergeOrUpload](/rest/api/searchservice/addupdate-or-delete-documents) pour le remplir. Par la suite, pendant l’opération de maintenance planifiée de l’index, vous pouvez le nettoyer de façon à supprimer les champs obsolètes.

Pour ajouter un nouveau champ à un index existant, appelez [Mettre à jour l’index](/rest/api/searchservice/update-index) et [mergeOrUpload](/rest/api/searchservice/addupdate-or-delete-documents) pour le remplir.

Pour ajouter un analyseur personnalisé à un index existant, transférez l’indicateur **allowIndexDowntime** dans [Mettre à jour l’index](/rest/api/searchservice/update-index) si vous souhaitez éviter cette erreur :

*« Mise à jour d’index non autorisée, car cette opération pourrait entraîner un temps d’arrêt. Pour ajouter de nouveaux analyseurs, générateurs de jetons, filtres de jetons ou filtres de caractères à un index existant, définissez le paramètre de requête « allowIndexDowntime » sur « true » dans la demande de mise à jour d’index. Notez que cette opération placera votre index hors connexion pendant au moins quelques secondes, ce qui entraînera l’échec de vos demandes d’indexation et de requête. Les performances et la disponibilité en écriture peuvent être altérées pendant plusieurs minutes après la mise à jour de l’index, voire plus longtemps pour les index de très grande taille ».*

## <a name="recommendations-for-working-with-analyzers"></a>Suggestions pour utiliser des analyseurs

Cette section offre des conseils pour utiliser les analyseurs.

### <a name="one-analyzer-for-read-write-unless-you-have-specific-requirements"></a>Un même analyseur pour les opérations de lecture-écriture, sauf besoins spécifiques

La Recherche cognitive Azure vous permet de spécifier différents analyseurs pour l’indexation et la recherche par le biais des propriétés de champ supplémentaires **indexAnalyzer** et **searchAnalyzer**. Par défaut, l’analyseur défini avec la propriété **analyzer** est utilisé pour l’indexation et la recherche. Si **analyzer** n’est pas spécifié, l’analyseur Lucene standard est utilisé par défaut.

En règle générale, il est préférable d’utiliser le même analyseur pour l’indexation et l’interrogation, sauf si des besoins spécifiques vous obligent à faire autrement. Veillez à effectuer des tests approfondis. Quand il existe une divergence de traitement de texte pendant la recherche et l’indexation, le risque est que les termes de la requête et les termes indexés ne correspondent pas si la configuration de l’analyseur de recherche et celle de l’analyseur d’indexation ne sont pas conformes.

### <a name="test-during-active-development"></a>Effectuer des tests pendant le développement actif

La substitution de l’analyseur standard nécessite une regénération de l’index. Si possible, choisissez les analyseurs à utiliser pendant le développement actif, avant de déployer l’index dans un environnement de production.

### <a name="inspect-tokenized-terms"></a>Examiner les termes sous forme de jetons

Si une recherche ne renvoie pas les résultats attendus, cela est très probablement dû aux différences de jetons entre les termes entrés dans la requête et les termes sous forme de jetons présents dans l’index. Si les jetons ne sont pas identiques, les correspondances ne sont pas détectées. Pour examiner les résultats du générateur de jetons, nous vous recommandons d’utiliser [l’API d’analyse](/rest/api/searchservice/test-analyzer) comme outil d’investigation. La réponse se compose de jetons qui sont générés par un analyseur spécifique.

<a name="examples"></a>

## <a name="rest-examples"></a>Exemples REST

Les exemples ci-dessous montrent des définitions d’analyseur pour quelques scénarios clés.

+ [Exemple d’analyseur personnalisé](#Custom-analyzer-example)
+ [Exemple d’affectation d’analyseurs à un champ](#Per-field-analyzer-assignment-example)
+ [Combinaison d’analyseurs pour l’indexation et la recherche](#Mixing-analyzers-for-indexing-and-search-operations)
+ [Exemple d’analyseur linguistique](#Language-analyzer-example)

<a name="Custom-analyzer-example"></a>

### <a name="custom-analyzer-example"></a>Exemple d’analyseur personnalisé

Cet exemple montre une définition d’analyseur avec des options personnalisées. Les options personnalisées pour les filtres de caractères, les générateurs de jetons et les filtres de jetons sont spécifiées séparément comme des constructions nommées, puis elles sont référencées dans la définition de l’analyseur. Les éléments prédéfinis sont utilisés tels quels et sont référencés par leur nom.

Si nous suivons cet exemple :

* Les analyseurs sont une propriété de la classe d’un champ pouvant faire l’objet d’une recherche.
* Un analyseur personnalisé fait partie d’une définition d’index. Il peut faire l’objet d’une légère personnalisation (par exemple, la personnalisation d’une option d’un filtre) ou d’une personnalisation à plusieurs endroits.
* Dans ce cas, l’analyseur personnalisé est « my_analyzer », qui utilise alors le générateur de jetons standard personnalisé « my_standard_tokenizer », et deux filtres de jetons : le filtre « lowercase » et le filtre asciifolding personnalisé « my_asciifolding ».
* Il définit également les deux filtres de caractères personnalisés « map_dash » et « remove_whitespace ». Le premier remplace tous les tirets par des traits de soulignement et le second supprime tous les espaces. Les espaces doivent être encodés en UTF-8 dans les règles de mappage. Les filtres de caractères sont appliqués avant la segmentation du texte en unités lexicales et affectent les jetons qui en résultent (les tirets et les espaces provoquent l'arrêt du générateur de jetons, mais pas les traits de soulignement).

```json
  {
     "name":"myindex",
     "fields":[
        {
           "name":"id",
           "type":"Edm.String",
           "key":true,
           "searchable":false
        },
        {
           "name":"text",
           "type":"Edm.String",
           "searchable":true,
           "analyzer":"my_analyzer"
        }
     ],
     "analyzers":[
        {
           "name":"my_analyzer",
           "@odata.type":"#Microsoft.Azure.Search.CustomAnalyzer",
           "charFilters":[
              "map_dash",
              "remove_whitespace"
           ],
           "tokenizer":"my_standard_tokenizer",
           "tokenFilters":[
              "my_asciifolding",
              "lowercase"
           ]
        }
     ],
     "charFilters":[
        {
           "name":"map_dash",
           "@odata.type":"#Microsoft.Azure.Search.MappingCharFilter",
           "mappings":["-=>_"]
        },
        {
           "name":"remove_whitespace",
           "@odata.type":"#Microsoft.Azure.Search.MappingCharFilter",
           "mappings":["\\u0020=>"]
        }
     ],
     "tokenizers":[
        {
           "name":"my_standard_tokenizer",
           "@odata.type":"#Microsoft.Azure.Search.StandardTokenizerV2",
           "maxTokenLength":20
        }
     ],
     "tokenFilters":[
        {
           "name":"my_asciifolding",
           "@odata.type":"#Microsoft.Azure.Search.AsciiFoldingTokenFilter",
           "preserveOriginal":true
        }
     ]
  }
```

<a name="Per-field-analyzer-assignment-example"></a>

### <a name="per-field-analyzer-assignment-example"></a>Exemple d’affectation d’analyseur par champ

L’analyseur standard est celui qui est utilisé par défaut. Supposons que vous souhaitiez remplacer l’analyseur par défaut par un autre analyseur prédéfini, tel que l’analyseur de pattern. Si vous ne définissez pas d’options personnalisées, vous devez uniquement le spécifier par son nom dans la définition du champ.

L’élément « analyzer » remplace l’analyseur standard champ après champ. Aucun remplacement global n’est possible. Dans cet exemple, `text1` utilise l’analyseur de pattern, et `text2`, qui ne spécifie pas d’analyseur, utilise celui par défaut.

```json
  {
     "name":"myindex",
     "fields":[
        {
           "name":"id",
           "type":"Edm.String",
           "key":true,
           "searchable":false
        },
        {
           "name":"text1",
           "type":"Edm.String",
           "searchable":true,
           "analyzer":"pattern"
        },
        {
           "name":"text2",
           "type":"Edm.String",
           "searchable":true
        }
     ]
  }
```

<a name="Mixing-analyzers-for-indexing-and-search-operations"></a>

### <a name="mixing-analyzers-for-indexing-and-search-operations"></a>Combinaison d’analyseurs pour les opérations d’indexation et de recherche

Les API comprennent des attributs d’index supplémentaires qui permettent de spécifier des analyseurs différents pour l’indexation et la recherche. Les attributs **searchAnalyzer** et **indexAnalyzer** doivent être spécifiés sous forme de paire, en remplacement de l’attribut **analyzer**.


```json
  {
     "name":"myindex",
     "fields":[
        {
           "name":"id",
           "type":"Edm.String",
           "key":true,
           "searchable":false
        },
        {
           "name":"text",
           "type":"Edm.String",
           "searchable":true,
           "indexAnalyzer":"whitespace",
           "searchAnalyzer":"simple"
        },
     ],
  }
```

<a name="Language-analyzer-example"></a>

### <a name="language-analyzer-example"></a>Exemple d’analyseur linguistique

Les champs qui contiennent des chaînes dans différentes langues peuvent utiliser un analyseur linguistique, tandis que les autres champs conservent l’analyseur par défaut (ou utilisent un autre analyseur prédéfini ou personnalisé). Si vous utilisez un analyseur linguistique, vous devez l’utiliser à la fois pour les opérations d’indexation et de recherche. Les champs qui utilisent un analyseur linguistique ne peuvent pas avoir des analyseurs différents pour l’indexation et la recherche.

```json
  {
     "name":"myindex",
     "fields":[
        {
           "name":"id",
           "type":"Edm.String",
           "key":true,
           "searchable":false
        },
        {
           "name":"text",
           "type":"Edm.String",
           "searchable":true,
           "indexAnalyzer":"whitespace",
           "searchAnalyzer":"simple"
        },
        {
           "name":"text_fr",
           "type":"Edm.String",
           "searchable":true,
           "analyzer":"fr.lucene"
        }
     ],
  }
```

## <a name="c-examples"></a>Exemples C#

Si vous utilisez les exemples de code du SDK .NET, vous pouvez ajouter ces exemples pour utiliser ou configurez des analyseurs.

+ [Attribuer un analyseur intégré](#Assign-a-language-analyzer)
+ [Configurer un analyseur](#Define-a-custom-analyzer)

<a name="Assign-a-language-analyzer"></a>

### <a name="assign-a-language-analyzer"></a>Attribuer un analyseur de langage

Tout analyseur qui est utilisé tel quel, sans configuration, est spécifié sur une définition de champ. La création d’une entrée dans la section **[analyzers]** de l’index n’est pas obligatoire. 

Cet exemple attribue les analyseurs Anglais et Français de Microsoft aux champs de description. C’est un extrait de code issu d’une définition plus grande de l’index des hôtels, créé à l’aide de la classe Hotel dans le fichier hotels.cs de l’exemple [DotNetHowTo](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowTo).

Appelez [LexicalAnalyzer](/dotnet/api/azure.search.documents.indexes.models.lexicalanalyzer) en spécifiant le type [LexicalAnalyzerName](/dotnet/api/azure.search.documents.indexes.models.lexicalanalyzername) pour fournir un analyseur de texte pris en charge dans Recherche cognitive Azure.

```csharp
    public partial class Hotel
    {
       . . . 

        [IsSearchable]
        [Analyzer(AnalyzerName.AsString.EnMicrosoft)]
        [JsonProperty("description")]
        public string Description { get; set; }

        [IsSearchable]
        [Analyzer(AnalyzerName.AsString.FrLucene)]
        [JsonProperty("description_fr")]
        public string DescriptionFr { get; set; }

      . . .
    }
```
<a name="Define-a-custom-analyzer"></a>

### <a name="define-a-custom-analyzer"></a>Définir un analyseur personnalisé

Lorsqu’une personnalisation ou configuration est requise, vous devez ajouter une construction de l’analyseur à un index. Une fois que vous la définissez, vous pouvez l’ajouter à la définition de champ comme illustré dans l’exemple précédent.

Créez un objet [CustomAnalyzer](/dotnet/api/azure.search.documents.indexes.models.customanalyzer). Pour plus d’exemples, consultez [CustomAnalyzerTests.cs](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/search/Microsoft.Azure.Search/tests/Tests/CustomAnalyzerTests.cs).

```csharp
{
   var definition = new Index()
   {
         Name = "hotels",
         Fields = FieldBuilder.BuildForType<Hotel>(),
         Analyzers = new[]
            {
               new CustomAnalyzer()
               {
                     Name = "url-analyze",
                     Tokenizer = TokenizerName.UaxUrlEmail,
                     TokenFilters = new[] { TokenFilterName.Lowercase }
               }
            },
   };

   serviceClient.Indexes.Create(definition);
```

## <a name="next-steps"></a>Étapes suivantes

+ Lisez notre explication complète du [fonctionnement de la recherche en texte intégral dans la Recherche cognitive Azure](search-lucene-query-architecture.md). Cet article utilise des exemples pour expliquer les comportements qui, au premier abord, peuvent sembler contraires à la logique.

+ Essayez une syntaxe de requête supplémentaire à partir de la section Exemple [Rechercher des documents](/rest/api/searchservice/search-documents#bkmk_examples) ou à partir de la [syntaxe de requête simple](query-simple-syntax.md) dans l’Explorateur de recherche du portail.

+ Découvrez comment appliquer des [analyseurs lexicaux propres au langage](index-add-language-analyzers.md).

+ [Configurez des analyseurs personnalisés](index-add-custom-analyzers.md) pour un traitement minimal ou pour un traitement spécialisé sur certains champs.

## <a name="see-also"></a>Voir aussi

 [API REST de recherche de documents](/rest/api/searchservice/search-documents) 

 [Syntaxe de requête simple](query-simple-syntax.md) 

 [Syntaxe de requête complète Lucene](query-lucene-syntax.md) 
 
 [Traiter les résultats de recherche](search-pagination-page-layout.md)

<!--Image references-->
[1]: ./media/search-lucene-query-architecture/architecture-diagram2.png