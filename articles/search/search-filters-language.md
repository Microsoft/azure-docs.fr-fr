---
title: Filtrer par langue dans un index de recherche
titleSuffix: Azure Cognitive Search
description: Critères de filtre pour prendre en charge la recherche multilingue et l’exécution de requêtes selon des champs spécifiques à une langue.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 04/22/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 120c30803d1423dc8b1255c65703e84d5938f235
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89002519"
---
# <a name="how-to-filter-by-language-in-azure-cognitive-search"></a>Comment filtrer par langue dans la Recherche cognitive Azure 

Une condition requise dans une application de recherche multilingue est la possibilité d’effectuer une recherche et de récupérer les résultats dans la langue de l’utilisateur. Dans la Recherche cognitive Azure, une manière de remplir les conditions de langue d’une application multilingue consiste à créer une série de champs dédiés au stockage de chaînes dans une langue spécifique, puis à limiter la recherche en texte intégral à ces seuls champs au moment de la requête.

Des paramètres de requête appliqués à la demande sont utilisés pour déterminer l’étendue de l’opération de recherche, puis écarter les résultats de tous les champs qui ne fournissent pas de contenu compatible avec l’expérience de recherche que vous souhaitez offrir.

| Paramètres | Objectif |
|-----------|--------------|
| **searchFields** | Limite la recherche en texte intégral à la liste des champs nommés. |
| **$select** | Réduit la réponse pour inclure uniquement les champs que vous spécifiez. Par défaut, tous les champs récupérables sont retournés. Le paramètre **$select** vous permet de choisir les champs à retourner. |

Le succès de cette technique dépend de l’intégrité du contenu des champs. La Recherche cognitive Azure ne convertit pas les chaînes et n’effectue pas de détection de la langue. Il vous appartient de vous assurer que les champs contiennent les chaînes que vous attendez.

## <a name="define-fields-for-content-in-different-languages"></a>Définir des champs de contenu dans différentes langues

Dans la Recherche cognitive Azure, les requêtes ciblent un index unique. Les développeurs qui souhaitent fournir des chaînes spécifiques d’une langue dans une expérience de recherche unique définissent généralement des champs dédiés pour stocker les valeurs : un champ pour les chaînes en français, un autre pour les chaînes en anglais, et ainsi de suite. 

L'exemple suivant est extrait de l'[exemple d’immobilier](search-get-started-portal.md), qui comporte plusieurs champs de chaîne avec du contenu dans différentes langues. Notez les assignations de l’analyseur de langue pour les champs de cet index. Les champs qui contiennent des chaînes produisent de meilleurs résultats en lien avec une recherche en texte intégral quand ils sont associés à un analyseur conçu pour traiter les règles linguistiques de la langue cible.

  ![Capture d’écran montrant l’écran Champs de l’exemple immobilier. Un groupe de champs est mis en surbrillance pour indiquer de quelle façon les attributions de l’analyseur de langue correspondent aux langues des champs en surbrillance.](./media/search-filters-language/lang-fields.png)

> [!Note]
> Pour des exemples de code montrant des définitions de champ avec des analyseurs de langue, voir [Définir un index (.NET)](./search-get-started-dotnet.md) et [Définir un index (REST)](./search-get-started-powershell.md).

## <a name="build-and-load-an-index"></a>Créer et charger un index

La [création et le remplissage de l’index](./search-get-started-dotnet.md) sont une étape intermédiaire (et peut-être évidente) avant la formulation d’une requête. Nous signalons cette étape ici par souci d’exhaustivité. Une manière de déterminer si l’index est disponible consiste à vérifier la liste des index dans le [portail](https://portal.azure.com).

## <a name="constrain-the-query-and-trim-results"></a>Limiter la requête et réduire les résultats

Les paramètres de la requête sont utilisés pour limiter la recherche à des champs spécifiques, puis écarter les résultats des champs dépourvus d’utilité pour votre scénario. Si votre objectif est de limiter la recherche aux champs contenant des chaînes en français, utilisez **searchFields** pour cibler la requête sur les champs contenant des chaînes dans cette langue. 

Par défaut, une recherche retourne tous les champs marqués comme récupérables. Par conséquent, il peut être utile d’exclure les champs non conformes à l’expérience de recherche spécifique de la langue que vous souhaitez offrir. Plus précisément, si vous limitez la recherche à un champ contenant des chaînes en français, vous voulez probablement exclure des résultats les champs contenant des chaînes en anglais. Le paramètre de requête **$select** vous permet de contrôler les champs qui sont retournés à l’application appelante.

```csharp
parameters =
    new SearchParameters()
    {
        searchFields = "description_fr" 
        Select = new[] { "description_fr"  }
    };
```
> [!Note]
> Bien que la requête ne contienne pas d’argument $filter, ce cas d’utilisation est étroitement apparenté aux concepts de filtre. C’est pourquoi il est présenté comme un scénario de filtrage.

## <a name="see-also"></a>Voir aussi

+ [Filtres dans la Recherche cognitive Azure](search-filters.md)
+ [Analyseurs de langage](/rest/api/searchservice/language-support)
+ [Fonctionnement de la recherche en texte intégral dans la Recherche cognitive Azure](search-lucene-query-architecture.md)
+ [API REST de recherche de documents](/rest/api/searchservice/search-documents)