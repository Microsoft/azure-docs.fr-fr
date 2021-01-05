---
title: Exemple C# pour les synonymes
titleSuffix: Azure Cognitive Search
description: Avec cet exemple C#, découvrez comment ajouter la fonctionnalité des synonymes à un index dans la Recherche cognitive Azure. Une carte de synonymes est une liste de termes équivalents. Les champs avec prise en charge des synonymes étendent les requêtes afin d'inclure le terme fourni par l’utilisateur et tous les synonymes associés.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/05/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 4b97b223ac180df7f8eb07ad8eaab66847f50776
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93422992"
---
# <a name="example-add-synonyms-for-azure-cognitive-search-in-c"></a>Exemple : Ajouter des synonymes pour le service Recherche cognitive Azure en C#

Les synonymes développent une requête en faisant correspondre les termes considérés comme sémantiquement équivalents à l’expression entrée. Par exemple, vous souhaiterez peut-être que le terme « voiture » vous permette de trouver des documents contenant les mots « automobile » ou « véhicule ». 

Dans la Recherche cognitive Azure, les synonymes sont définis dans une *carte de synonymes*, par le biais des *règles de mappage* qui associent des termes équivalents. Cet exemple décrit les étapes essentielles pour l’ajout et l’utilisation de synonymes avec un index existant.

Dans cet exemple, vous allez apprendre à effectuer les opérations suivantes :

> [!div class="checklist"]
> * Créer une carte de synonymes en utilisant la [classe SynonymMap](/dotnet/api/azure.search.documents.indexes.models.synonymmap). 
> * Définir la propriété [SynonymMaps](/dotnet/api/azure.search.documents.indexes.models.searchfield.synonymmapnames) sur des champs qui doivent prendre en charge l’extension de requête par le biais de synonymes.

Vous pouvez interroger un champ acceptant les synonymes comme vous le faites habituellement. Aucune syntaxe de requête supplémentaire n’est requise pour accéder aux synonymes.

Vous pouvez créer plusieurs cartes de synonymes, les valider en tant que ressources du service disponible pour tout index, et ensuite référencer ceux que vous souhaitez utiliser au niveau du champ. Au moment de la requête, en plus de la recherche dans un index, la Recherche cognitive Azure effectue une recherche dans une carte de synonymes, si une telle carte est spécifiée dans les champs utilisés dans la requête.

> [!NOTE]
> Les synonymes peuvent être créés par programmation, mais pas dans le portail.

## <a name="prerequisites"></a>Prérequis

La configuration requise du didacticiel est la suivante :

* [Visual Studio](https://www.visualstudio.com/downloads/)
* [Service Recherche cognitive Azure](search-create-service-portal.md)
* [Package Azure.Search.Documents](https://www.nuget.org/packages/Azure.Search.Documents/)

Si vous n’êtes pas familiarisé avec la bibliothèque cliente .NET, consultez [Utilisation de la Recherche cognitive Azure dans .NET](search-howto-dotnet-sdk.md).

## <a name="sample-code"></a>Exemple de code

Vous trouverez le code source complet de l’exemple d’application utilisé dans cet exemple sur [GitHub](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowToSynonyms).

## <a name="overview"></a>Vue d’ensemble

Des requêtes avant et après sont utilisées pour montrer la valeur des synonymes. Dans cet exemple, un exemple d’application exécute des requêtes et retourne des résultats sur un exemple d’index « hotels » rempli avec deux documents. Tout d’abord, l’application exécute des requêtes de recherche à l’aide de termes et d’expressions qui n’apparaissent pas dans l’index. Ensuite, le code active la fonctionnalité de synonymes, puis réémet les mêmes requêtes, cette fois en retournant les résultats en fonction des correspondances dans la carte de synonymes. 

Le code ci-dessous montre le flux global.

```csharp
static void Main(string[] args)
{
   SearchIndexClient indexClient = CreateSearchIndexClient();

   Console.WriteLine("Cleaning up resources...\n");
   CleanupResources(indexClient);

   Console.WriteLine("Creating index...\n");
   CreateHotelsIndex(indexClient);

   SearchClient searchClient = indexClient.GetSearchClient("hotels");

   Console.WriteLine("Uploading documents...\n");
   UploadDocuments(searchClient);

   SearchClient searchClientForQueries = CreateSearchClientForQueries();

   RunQueriesWithNonExistentTermsInIndex(searchClientForQueries);

   Console.WriteLine("Adding synonyms...\n");
   UploadSynonyms(indexClient);

   Console.WriteLine("Enabling synonyms in the test index...\n");
   EnableSynonymsInHotelsIndexSafely(indexClient);
   Thread.Sleep(10000); // Wait for the changes to propagate

   RunQueriesWithNonExistentTermsInIndex(searchClientForQueries);

   Console.WriteLine("Complete.  Press any key to end application...\n");

   Console.ReadKey();
}
```

## <a name="before-queries"></a>Requêtes « avant »

Dans `RunQueriesWithNonExistentTermsInIndex`, émettez des requêtes de recherche avec « five star », « internet » et « economy AND hotel ».

```csharp
Console.WriteLine("Search the entire index for the phrase \"five star\":\n");
results = searchClient.Search<Hotel>("\"five star\"", searchOptions);
WriteDocuments(results);

Console.WriteLine("Search the entire index for the term 'internet':\n");
results = searchClient.Search<Hotel>("internet", searchOptions);
WriteDocuments(results);

Console.WriteLine("Search the entire index for the terms 'economy' AND 'hotel':\n");
results = searchClient.Search<Hotel>("economy AND hotel", searchOptions);
WriteDocuments(results);
```

Aucun des deux documents indexés ne contenant les termes, nous avons la sortie suivante du premier `RunQueriesWithNonExistentTermsInIndex` : **aucun document correspondant**.

## <a name="enable-synonyms"></a>Activation des synonymes

Après exécution des requêtes « before », l’exemple de code active les synonymes. L’activation des synonymes est un processus en deux étapes. Tout d’abord, définir et charger les règles de synonymes. Ensuite, configurer les champs pour les utiliser. Le processus est décrit dans `UploadSynonyms` et `EnableSynonymsInHotelsIndex`.

1. Ajoutez une carte de synonymes à votre service de recherche. Dans `UploadSynonyms`, nous définissons quatre règles de notre carte de synonymes « desc-synonymmap » et effectuons le téléchargement vers le service.

   ```csharp
   private static void UploadSynonyms(SearchIndexClient indexClient)
   {
      var synonymMap = new SynonymMap("desc-synonymmap", "hotel, motel\ninternet,wifi\nfive star=>luxury\neconomy,inexpensive=>budget");

      indexClient.CreateOrUpdateSynonymMap(synonymMap);
   }
   ```

1. Configurez les champs pouvant faire l’objet d’une recherche pour utiliser la carte de synonymes dans la définition d’index. Dans `AddSynonymMapsToFields`, nous activons les synonymes sur deux champs `category` et `tags` en affectant à la propriété `SynonymMapNames` le nom de la carte de synonymes qui vient d’être téléchargée.

   ```csharp
   private static SearchIndex AddSynonymMapsToFields(SearchIndex index)
   {
      index.Fields.First(f => f.Name == "category").SynonymMapNames.Add("desc-synonymmap");
      index.Fields.First(f => f.Name == "tags").SynonymMapNames.Add("desc-synonymmap");
      return index;
   }
   ```

   Lorsque vous ajoutez une carte de synonymes, les reconstructions d’index ne sont pas requises. Vous pouvez ajouter une carte de synonymes à votre service, puis modifier les définitions de champ existantes dans n’importe quel index pour utiliser la nouvelle carte de synonymes. L’ajout de nouveaux attributs n’a aucun impact sur la disponibilité de l’index. Il en va de même pour la désactivation de synonymes pour un champ. Vous pouvez simplement affecter à la propriété `SynonymMapNames` une liste vide.

   ```csharp
   index.Fields.First(f => f.Name == "category").SynonymMapNames.Add("desc-synonymmap");
   ```

## <a name="after-queries"></a>Requêtes « après »

Une fois que la carte de synonymes est téléchargée et l’index mis à jour pour utiliser la carte de synonymes, le deuxième appel `RunQueriesWithNonExistentTermsInIndex` affiche les éléments suivants :

```
Search the entire index for the phrase "five star":

Name: Fancy Stay        Category: Luxury        Tags: [pool, view, wifi, concierge]

Search the entire index for the term 'internet':

Name: Fancy Stay        Category: Luxury        Tags: [pool, view, wifi, concierge]

Search the entire index for the terms 'economy' AND 'hotel':

Name: Roach Motel       Category: Budget        Tags: [motel, budget]
```

La première requête trouve le document à partir de la règle `five star=>luxury`. La deuxième requête étend la recherche à l’aide de `internet,wifi` et la troisième avec à la fois `hotel, motel` et `economy,inexpensive=>budget` pour trouver les documents en correspondance.

L’ajout de synonymes modifie complètement l’expérience de recherche. Dans cet exemple, les requêtes d’origine n’ont pas pu retourner de résultats significatifs même si les documents dans notre index étaient pertinents. En activant les synonymes, nous pouvons développer un index pour inclure les termes communément utilisés, sans modification de données sous-jacentes dans l’index.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Le moyen le plus rapide de procéder à un nettoyage après un exemple consiste à supprimer le groupe de ressources contenant le service Recherche cognitive Azure. Vous pouvez maintenant supprimer le groupe de ressources pour supprimer définitivement tout ce qu’il contient. Dans le portail, le nom du groupe de ressources figure dans la page Vue d’ensemble du service Recherche cognitive Azure.

## <a name="next-steps"></a>Étapes suivantes

Cet exemple a présenté la fonctionnalité de synonymes en code C# pour créer et publier des règles de mappage, puis appeler la carte de synonymes pour une requête. Vous trouverez des informations supplémentaires dans la documentation de référence du [Kit de développement logiciel (SDK) .NET](/dotnet/api/overview/azure/search.documents-readme) et de [l’API REST](/rest/api/searchservice/).

> [!div class="nextstepaction"]
> [Guide pratique pour utiliser des synonymes dans la Recherche cognitive Azure](search-synonyms.md)