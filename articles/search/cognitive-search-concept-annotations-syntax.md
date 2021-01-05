---
title: Référencer des entrées et des sorties dans les ensembles de compétences
titleSuffix: Azure Cognitive Search
description: Cet article explique la syntaxe d’annotation et la manière de référencer une annotation dans les entrées et sorties d’un jeu de compétences dans un pipeline d’enrichissement de l’IA dans Recherche cognitive Azure.
manager: nitinme
author: LuisCabrer
ms.author: luisca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/04/2019
ms.openlocfilehash: 03431d861ca6d469b894e45c36fe2a3d7904c3a2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88935532"
---
# <a name="how-to-reference-annotations-in-an-azure-cognitive-search-skillset"></a>Comment référencer des annotations dans un jeu de compétences Recherche cognitive Azure

Cet article vous permet de découvrir comment référencer des annotations dans des définitions de compétences, en utilisant des exemples pour illustrer différents scénarios. À mesure que le contenu d’un document traverse un jeu de compétences, il s’enrichit d’annotations. Ces annotations peuvent être utilisées comme entrées pour un enrichissement supplémentaire en aval, ou mappées à un champ de sortie dans un index. 
 
Les exemples présentés dans cet article sont basés sur le champ de *contenu* généré automatiquement par les [indexeurs d’objets blob Azure](search-howto-indexing-azure-blob-storage.md) durant la phase de décodage du document. Lorsque vous faites référence à des documents à partir d’un conteneur d’objets blob, utilisez un format tel que `"/document/content"`, où le champ de *contenu* fait partie du *document*. 

## <a name="background-concepts"></a>Concepts de base

Avant d’examiner la syntaxe, revenons sur quelques concepts importants pour mieux comprendre les exemples fournis plus loin.

| Terme | Description |
|------|-------------|
| Document enrichi | Un document enrichi est une structure interne créée et utilisée par le pipeline pour recueillir toutes les annotations liées à un document. Il faut imaginer un document enrichi comme une arborescence d’annotations. En règle générale, une annotation créée à partir d’une annotation précédente devient l’enfant de celle-ci.<p/>Des documents enrichis n’existent que pendant l’exécution d’un jeu de compétences. Une fois le contenu mappé à l’index de recherche, le document enrichi n’est plus nécessaire. Bien que vous n’interagissiez pas directement avec des documents enrichis, il est utile d’avoir un modèle mental des documents lors de la création d’un jeu de compétences. |
| Contexte d’enrichissement | Contexte dans lequel l’enrichissement a lieu, déterminant l’élément qui est enrichi. Par défaut, le contexte d’enrichissement est au niveau `"/document"`, limité à des documents individuels. Quand une compétence est appliquée, ses résultats deviennent des [propriétés du contexte défini](#example-2).|

<a name="example-1"></a>
## <a name="example-1-simple-annotation-reference"></a>Exemple 1 : référencer une annotation simple

Dans un stockage d’objets blob Azure, supposons que vous ayez une variété de fichiers contenant des références à des noms de personnes que vous souhaitez extraire à l’aide d’une reconnaissance d’entité. Dans la définition de compétence ci-dessous, `"/document/content"` est la représentation textuelle du document entier, et « personnes » une extraction de noms complets d’entités identifiées en tant que personnes.

Le contexte par défaut étant `"/document"`, la liste de personnes peut désormais être référencée comme `"/document/people"`. En l’occurrence, `"/document/people"` est une annotation qui peut maintenant être mappée à un champ dans un index, ou utilisée dans une autre compétence du même jeu de compétences.

```json
  {
    "@odata.type": "#Microsoft.Skills.Text.EntityRecognitionSkill",
    "categories": [ "Person"],
    "defaultLanguageCode": "en",
    "inputs": [
      {
        "name": "text",
        "source": "/document/content"
      }
    ],
    "outputs": [
      {
        "name": "persons",
        "targetName": "people"
      }
    ]
  }
```

<a name="example-2"></a>

## <a name="example-2-reference-an-array-within-a-document"></a>Exemple 2 : référencer un tableau à l’intérieur d’un document

Cet exemple s’appuie sur le précédent pour montrer comment appeler une étape d’enrichissement plusieurs fois sur le même document. Supposons que l’exemple précédent a généré un tableau de chaînes comprenant 10 noms de personnes à partir d’un seul document. L’étape suivante pourrait raisonnablement être un deuxième enrichissement extrayant le nom de famille d’un nom complet. Étant donné qu’il y a 10 noms, vous voulez que cette étape soit appelée 10 fois dans ce document, soit une fois par personne. 

Pour appeler le nombre approprié d’itérations, définissez le contexte comme `"/document/people/*"`, où l’astérisque (`"*"`) représente tous les nœuds figurant dans le document enrichi en tant que descendants de `"/document/people"`. Bien que cette compétence ne soit définie qu’une seule fois dans le tableau des compétences, elle est appelée pour chaque membre figurant dans le document jusqu’à ce que tous les membres soient traités.

```json
  {
    "@odata.type": "#Microsoft.Skills.Custom.WebApiSkill",
    "description": "Fictitious skill that gets the last name from a full name",
    "uri": "http://names.azurewebsites.net/api/GetLastName",
    "context" : "/document/people/*",
    "defaultLanguageCode": "en",
    "inputs": [
      {
        "name": "fullname",
        "source": "/document/people/*"
      }
    ],
    "outputs": [
      {
        "name": "lastname",
        "targetName": "last"
      }
    ]
  }
```

Lorsque les annotations sont des tableaux ou des collections de chaînes, vous pouvez cibler des membres spécifiques plutôt que le tableau dans son ensemble. L’exemple ci-dessus génère une annotation appelée `"last"` sous chaque nœud représenté par le contexte. Si vous souhaitez faire référence à cette famille d’annotations, vous pouvez utiliser la syntaxe `"/document/people/*/last"`. Si vous voulez faire référence à une annotation particulière, vous pouvez utiliser un index explicite `"/document/people/1/last` pour faire référence au nom de famille de la première personne identifiée dans le document. Notez que, dans cette syntaxe, les tableaux sont « indexés sur 0 ».

<a name="example-3"></a>

## <a name="example-3-reference-members-within-an-array"></a>Exemple 3 : référencer des membres à l’intérieur d’un tableau

Parfois, vous devez regrouper toutes les annotations d’un type particulier pour les transmettre à une compétence particulière. Imaginez une compétence personnalisée hypothétique qui identifie le nom de famille le plus courant parmi tous les noms de famille extraits dans l’Exemple 2. Pour fournir uniquement les noms de famille à la compétence personnalisée, spécifiez le contexte comme `"/document"` et l’entrée comme `"/document/people/*/lastname"`.

Notez que la cardinalité de `"/document/people/*/lastname"` est supérieure à celle du document. Il peut y avoir 10 nœuds de nom de famille alors qu’il n’y a qu’un seul nœud de document pour ce document. Dans ce cas, le système crée automatiquement un tableau de `"/document/people/*/lastname"` contenant tous les éléments du document.

```json
  {
    "@odata.type": "#Microsoft.Skills.Custom.WebApiSkill",
    "description": "Fictitious skill that gets the most common string from an array of strings",
    "uri": "http://names.azurewebsites.net/api/MostCommonString",
    "context" : "/document",
    "inputs": [
      {
        "name": "strings",
        "source": "/document/people/*/lastname"
      }
    ],
    "outputs": [
      {
        "name": "mostcommon",
        "targetName": "common-lastname"
      }
    ]
  }
```



## <a name="see-also"></a>Voir aussi
+ [Comment intégrer une compétence personnalisée dans un pipeline d’enrichissement](cognitive-search-custom-skill-interface.md)
+ [Guide pratique pour définir un ensemble de compétences](cognitive-search-defining-skillset.md)
+ [Créer un jeu de compétences (REST)](/rest/api/searchservice/create-skillset)
+ [Guide pratique pour mapper des champs enrichis à un index](cognitive-search-output-field-mapping.md)