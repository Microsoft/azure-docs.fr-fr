---
title: Indexation de recherche pour requêtes de recherche effectuées dans une langue autre que l’anglais
titleSuffix: Azure Cognitive Search
description: Recherche cognitive Azure prend en charge 56 langues, tirant parti des analyseurs de langue de la technologie Lucene et Natural Language Processing de Microsoft.
manager: nitinme
author: yahnoosh
ms.author: jlembicz
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 07/12/2020
ms.openlocfilehash: 588de9c9cae114b5f5396db17f7ecb19bcde25c6
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93423077"
---
# <a name="how-to-create-an-index-for-multiple-languages-in-azure-cognitive-search"></a>Comment créer un index dans plusieurs langues dans Recherche cognitive Azure

Les index peuvent comprendre des champs avec du contenu provenant de plusieurs langues, par exemple pour créer des champs pour des chaînes spécifiques à une langue. Pour obtenir des résultats optimaux lors de l’indexation et de l’interrogation, affectez un analyseur de langue qui fournit les règles linguistiques appropriées. 

Recherche cognitive Azure offre une large sélection d’analyseurs de langue Lucene et Microsoft, qui peuvent être affectés à des champs individuels à l’aide de la propriété Analyzer. Vous pouvez également spécifier un analyseur de langue dans le portail, comme décrit dans cet article.

## <a name="add-analyzers-to-fields"></a>Ajouter des analyseurs aux champs

L’analyseur de langue est spécifié lors de la création d’un champ. L’ajout d’un analyseur à une définition de champ existante nécessite le remplacement (et le rechargement) de l’index ou la création d’un nouveau champ identique à l’original, mais avec une affectation d’analyseur. Vous pouvez ensuite supprimer le champ inutilisé à votre convenance.

1. Connectez-vous au [portail Azure](https://portal.azure.com), puis trouvez votre service de recherche.
1. Cliquez sur **Ajouter un index** dans la barre de commandes en haut du tableau de bord de service pour démarrer un nouvel index ou ouvrez un index existant pour définir un analyseur sur des nouveaux champs que vous ajoutez à un index existant.
1. Démarrez une définition de champ en fournissant un nom.
1. Choisissez le type de données Edm.String. Seuls les champs de type chaîne peuvent faire l’objet d’une recherche en texte intégral.
1. Définissez l’attribut **Searchable** pour activer la propriété de l’analyseur. Le champ doit être basé sur du texte pour pouvoir utiliser un analyseur de langue.
1. Choisissez l’un des analyseurs disponibles. 

![Affecter des analyseurs de langage lors de la définition du champ](media/search-language-support/select-analyzer.png "Affecter des analyseurs de langage lors de la définition du champ")

Par défaut, tous les champs pouvant faire l’objet d’une recherche utilisent l’[analyseur Lucene Standard](https://lucene.apache.org/core/6_6_1/core/org/apache/lucene/analysis/standard/StandardAnalyzer.html) qui est indépendant de la langue. Pour afficher la liste complète des analyseurs pris en charge, consultez [Ajouter des analyseurs linguistiques à un index de Recherche cognitive Azure](index-add-language-analyzers.md).

Dans le portail, les analyseurs sont conçus pour être utilisés tels quels. Si vous avez besoin d’une personnalisation ou d’une configuration spécifique des filtres et des générateurs de jetons, vous devez [créer un analyseur personnalisé](index-add-custom-analyzers.md) dans le code. Le portail ne prend pas en charge la sélection ou la configuration d’analyseurs personnalisés.

## <a name="query-language-specific-fields"></a>Champs spécifiques à la langue de la requête

Une fois que l'analyseur de langage est sélectionné pour un champ, il sera utilisé à chaque demande de recherche et d'indexation pour ce champ. Lorsqu’une requête est émise sur plusieurs champs à l’aide de différents analyseurs, elle sera traitée indépendamment par les analyseurs affectés pour chaque champ.

Si la langue de l'agent d'émission d'une requête est connue, une demande de recherche peut être étendue à un champ spécifique à l'aide du paramètre de requête **searchFields** . La requête suivante sera émise uniquement sur la description en polonais :

`https://[service name].search.windows.net/indexes/[index name]/docs?search=darmowy&searchFields=PolishContent&api-version=2020-06-30`

Vous pouvez interroger votre index à partir du portail avec l’[**Explorateur de recherche**](search-explorer.md) pour coller une requête similaire à celle présentée ci-dessus.

## <a name="boost-language-specific-fields"></a>Améliorer les champs spécifiques à la langue

Parfois, la langue de l'agent d'émission d'une requête n'est pas connue, auquel cas la requête peut être exécutée simultanément sur tous les champs. Si nécessaire, la préférence de résultats dans une langue donnée peut être définie à l'aide des [profils de score](index-add-scoring-profiles.md). Dans l'exemple ci-dessous, les correspondances trouvées dans la description en anglais auront un score supérieur par rapport aux correspondances en polonais et en français :

```http
    "scoringProfiles": [
      {
        "name": "englishFirst",
        "text": {
          "weights": { "description_en": 2 }
        }
      }
    ]
```

`https://[service name].search.windows.net/indexes/[index name]/docs?search=Microsoft&scoringProfile=englishFirst&api-version=2020-06-30`

## <a name="next-steps"></a>Étapes suivantes

Si vous êtes développeur .NET, notez que vous pouvez configurer les analyseurs de langue à l’aide du [Kit de développement logiciel (SDK) Recherche cognitive Azure .NET](https://www.nuget.org/packages/Microsoft.Azure.Search) et de la propriété [LexicalAnalyzer](/dotnet/api/azure.search.documents.indexes.models.lexicalanalyzer).