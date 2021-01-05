---
title: Effectuer une mise à niveau vers la version 9 du SDK .NET Recherche Azure
titleSuffix: Azure Cognitive Search
description: Migrez le code vers le SDK .NET Recherche Azure version 9 à partir de versions antérieures. Découvrez les nouveautés et les modifications de code nécessaires.
manager: nitinme
author: brjohnstmsft
ms.author: brjohnst
ms.service: cognitive-search
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 11/04/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 7726877efcb72caf5bb4c92e2e1c65e5df68f270
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89002706"
---
# <a name="upgrade-to-azure-search-net-sdk-version-9"></a>Effectuer une mise à niveau vers la version 9 du SDK .NET Recherche Azure

Si vous utilisez la version 7.0-preview ou une version antérieure du [SDK .NET Recherche Azure](/dotnet/api/overview/azure/search), cet article vous aidera à mettre à niveau votre application pour utiliser la version 9.

> [!NOTE]
> Si vous souhaitez utiliser la version 8.0-preview pour tester les fonctionnalités qui ne sont pas encore à la disposition générale, vous pouvez également suivre les instructions de cet article pour passer à la version 8.0-preview à partir de versions antérieures.

Pour avoir un aperçu général du kit de développement logiciel et avoir des exemples, voir [Comment utiliser Azure Search à partir d’une application .NET](search-howto-dotnet-sdk.md).

La version 9 du SDK .NET Recherche Azure contient de nombreuses modifications par rapport aux versions antérieures. Certaines de ces modifications sont des changements cassants, mais elles ne devraient que des changement relativement mineurs sur votre code. Consultez la page [Procédure de mise à niveau](#UpgradeSteps) pour obtenir des instructions sur la façon de modifier le code à utiliser avec la nouvelle version du kit de développement logiciel.

> [!NOTE]
> Si vous utilisez la version 4.0-preview ou une version antérieure, vous devez tout d’abord effectuer une mise à niveau vers la version 5, puis vers la version 9. Consultez [Mise à niveau vers la version 5 du SDK .NET Recherche Azure](search-dotnet-sdk-migration-version-5.md) pour obtenir des conseils sur la migration.
>
> Votre instance de service Recherche Azure prend en charge plusieurs versions de l’API REST, y compris la plus récente. Vous pouvez continuer à utiliser une version lorsqu’elle n’est pas la plus récente, mais nous vous recommandons de migrer votre code pour utiliser la dernière version. Lorsque vous utilisez l’API REST, vous devez spécifier la version de l’API dans chaque requête via le paramètre « api-version » (Version de l’API). Lorsque vous utilisez le Kit de développement logiciel (SDK) .NET, la version du Kit de développement logiciel (SDK) que vous utilisez détermine la version correspondante de l’API REST. Si vous utilisez un Kit de développement logiciel (SDK) plus ancien, vous pouvez continuer à exécuter ce code sans changement, même si le service est mis à niveau pour prendre en charge une version plus récente de l’API.

<a name="WhatsNew"></a>

## <a name="whats-new-in-version-9"></a>Nouveautés de la version 9
La version 9 d’Azure Search .NET SDK est basée sur la version du 06/05/2019 de l’API REST Azure Search, avec les fonctions suivantes :

* L’[enrichissement par IA](cognitive-search-concept-intro.md) est la capacité à extraire du texte à partir d’images, d’objets blob et d’autres sources de données non structurées pour enrichir le contenu et ainsi faciliter les recherches dans un index de Recherche Azure.
* La prise en charge de [types complexes](search-howto-complex-data-types.md) vous permet de modéliser presque n’importe quelle structure JSON imbriquée dans un index Recherche Azure.
* La fonctionnalité [Autocomplétion](search-autocomplete-tutorial.md) fournit une alternative à l’API **Suggestion** pour implémenter le comportement de recherche au cours de la frappe. L’autocomplétion « termine » le mot ou l’expression que l’utilisateur est en train de taper.
* Le [mode d’analyse JsonLines](search-howto-index-json-blobs.md), inclus dans l’indexation des objets Blob Azure, crée un document de recherche par entité JSON, séparé par un saut de ligne.

### <a name="new-preview-features-in-version-80-preview"></a>Nouvelles fonctionnalités d’évaluation dans la version 8.0-preview
La version 8.0-preview du SDK .NET Recherche Azure cible l’API version 2017-11-11-Preview. Cette version inclut toutes les fonctionnalités de la version 9, plus :

* [Clés de chiffrement gérées par le client](search-security-manage-encryption-keys.md) pour le chiffrement au repos côté service est une nouvelle fonctionnalité d’évaluation. Outre le chiffrement au repos intégré géré par Microsoft, vous pouvez appliquer une couche supplémentaire de chiffrement lorsque vous êtes l’unique propriétaire des clés.

<a name="UpgradeSteps"></a>

## <a name="steps-to-upgrade"></a>Procédure de mise à niveau
Tout d’abord, mettez à jour vos références NuGet `Microsoft.Azure.Search` , soit en utilisant la console NuGet Package, soit en effectuant un clic droit sur les références de votre projet et en sélectionnant « Gérer les packages NuGet » dans Visual Studio.

Une fois que NuGet a téléchargé les nouveaux packages et leurs dépendances, recompilez votre projet. En fonction de la structure de votre code, la recompilation peut réussir. Si c’est le cas, vous êtes prêt !

Si votre build échoue, vous devrez résoudre chaque erreur de build. Voir [Changements cassants dans la version 9](#ListOfChanges) pour en savoir plus sur la façon de résoudre chaque erreur de build potentielle.

D’autres avertissements de compilation sont susceptibles de s’afficher ; ils sont liés à des propriétés ou des méthodes obsolètes. Ces avertissements incluent des instructions sur les fonctionnalités à utiliser à la place de la fonctionnalité déconseillée. Par exemple, si votre application utilise la propriété `DataSourceType.DocumentDb`, vous devriez obtenir un avertissement indiquant « Ce membre est déconseillé. Utilisez CosmosDb à la place ».

Une fois que vous avez résolu les erreurs de build ou les avertissements sur la génération, vous pouvez apporter des changements à votre application pour exploiter les nouvelles fonctionnalités, si vous le souhaitez. Les nouvelles fonctionnalités du Kit SDK sont détaillées dans la section [Nouveautés de la version 9](#WhatsNew).

<a name="ListOfChanges"></a>

## <a name="breaking-changes-in-version-9"></a>Changements cassants dans la version 9

Il y a plusieurs changements cassants dans la version 9 qui sont susceptibles de nécessiter des modifications de code en plus de la régénération de votre application.

> [!NOTE]
> La liste des modifications ci-dessous n’est pas exhaustive. Certains changements ne provoqueront probablement pas d’erreurs de build, mais sont techniquement cassants dans la mesure où ils rompent la compatibilité binaire avec des assemblys qui dépendent de versions antérieures d’assemblys SDK .NET Recherche Azure. Ces modifications ne sont pas répertoriées ci-dessous. Régénérez votre application lors de la mise à niveau vers la version 9 pour éviter tout problème de compatibilité binaire.

### <a name="immutable-properties"></a>Propriétés immuables

Les propriétés publiques de plusieurs classes de modèle sont désormais immuables. Si vous avez besoin créer des instances personnalisées de ces classes à des fins de test, vous pouvez utiliser les nouveaux constructeurs paramétrés :

  - `AutocompleteItem`
  - `DocumentSearchResult`
  - `DocumentSuggestResult`
  - `FacetResult`
  - `SearchResult`
  - `SuggestResult`

### <a name="changes-to-field"></a>Changements dans les champs

La classe `Field` a été modifiée maintenant qu’elle peut également représenter des champs complexes.

Les propriétés `bool` suivantes sont désormais Nullable :

  - `IsFilterable`
  - `IsFacetable`
  - `IsSearchable`
  - `IsSortable`
  - `IsRetrievable`
  - `IsKey`

Cela vient du fait que ces propriétés doivent maintenant être `null` dans le cas des champs complexes. Si vous avez un code qui lit ces propriétés, il doit être prêt à gérer `null`. Notez que toutes les autres propriétés de `Field` ont toujours été et continuent à être Nullable, et certaines d’entre elles seront également `null` dans le cas de champs complexes, notamment dans les cas suivants :

  - `Analyzer`
  - `SearchAnalyzer`
  - `IndexAnalyzer`
  - `SynonymMaps`

Le constructeur sans paramètre de `Field` est devenu `internal`. Dès lors, chaque `Field` requiert un nom explicite et un type de données au moment de la construction.

### <a name="simplified-batch-and-results-types"></a>Types de lot et résultats simplifiés

Dans la version 7.0-preview et versions antérieures, les différentes classes qui encapsulent des groupes de documents ont été structurées dans des hiérarchies de classes parallèles :

  -  `DocumentSearchResult` et `DocumentSearchResult<T>` héritées de `DocumentSearchResultBase`
  -  `DocumentSuggestResult` et `DocumentSuggestResult<T>` héritées de `DocumentSuggestResultBase`
  -  `IndexAction` et `IndexAction<T>` héritées de `IndexActionBase`
  -  `IndexBatch` et `IndexBatch<T>` héritées de `IndexBatchBase`
  -  `SearchResult` et `SearchResult<T>` héritées de `SearchResultBase`
  -  `SuggestResult` et `SuggestResult<T>` héritées de `SuggestResultBase`

Les types dérivés sans paramètre de type générique étaient destinés à être utilisés dans des scénarios de « type dynamique » et supposaient l’utilisation du type `Document`.

À compter de la version 8.0-preview, les classes de base et les classes dérivées non génériques ont toutes été supprimées. Pour les scénarios de type dynamique, vous pouvez utiliser `IndexBatch<Document>`, `DocumentSearchResult<Document>`, et ainsi de suite.
 
### <a name="removed-extensibleenum"></a>ExtensibleEnum supprimé

La classe de base `ExtensibleEnum` a été supprimée. Toutes les classes qui en sont dérivées sont désormais de type struct, comme `AnalyzerName`, `DataType` et `DataSourceType`, par exemple. Leurs méthodes `Create` ont également été supprimées. Vous pouvez simplement supprimer les appels à `Create` étant donné que ces types sont convertibles implicitement à partir de chaînes. Si cela entraîne des erreurs du compilateur, vous pouvez appeler explicitement l’opérateur de conversion par le biais de cast pour lever l’ambiguïté des types. Par exemple, vous pouvez modifier le code comme suit :

```csharp
var index = new Index()
{
    Fields = new[]
    {
        new Field("id", DataType.String) { IsKey = true },
        new Field("message", AnalyzerName.Create("my_email_analyzer")) { IsSearchable = true }
    },
    ...
}
```

par ceci :

```csharp
var index = new Index()
{
    Fields = new[]
    {
        new Field("id", DataType.String) { IsKey = true },
        new Field("message", (AnalyzerName)"my_email_analyzer") { IsSearchable = true }
    },
    ...
}
```

Les propriétés qui détenaient des valeurs facultatives de ces types sont maintenant explicitement de type Nullable afin qu’elles continuent à être facultatives.

### <a name="removed-facetresults-and-hithighlights"></a>FacetResults et HitHighlights supprimés

Les classes `FacetResults` et `HitHighlights` ont été supprimées. Les résultats de facette ont désormais le type `IDictionary<string, IList<FacetResult>>` et les mises en surbrillance des correspondance le type `IDictionary<string, IList<string>>`. Un moyen rapide pour résoudre les erreurs de build introduits par cette modification consiste à ajouter les alias `using` en haut de chaque fichier qui utilise les types supprimés. Par exemple :

```csharp
using FacetResults = System.Collections.Generic.IDictionary<string, System.Collections.Generic.IList<Models.FacetResult>>;
using HitHighlights = System.Collections.Generic.IDictionary<string, System.Collections.Generic.IList<string>>;
```

### <a name="change-to-synonymmap"></a>Changer pour SynonymMap 

Le constructeur `SynonymMap` n’a plus de paramètre `enum` pour `SynonymMapFormat`. Cette énumération avait une seule valeur et était donc redondante. Si vous voyez des erreurs de build suite à ce changement, supprimez simplement les références au paramètre `SynonymMapFormat`.

### <a name="miscellaneous-model-class-changes"></a>Diverses modifications de modèles de classe

La propriété `AutocompleteMode` de `AutocompleteParameters` n’est plus Nullable. Si vous avez du code qui attribue cette propriété à `null`, vous pouvez simplement le supprimer et la propriété sera automatiquement initialisée à la valeur par défaut.

L’ordre des paramètres pour le constructeur `IndexAction`, maintenant que ce constructeur est généré automatiquement. Au lieu d’utiliser le constructeur, nous vous recommandons d’utiliser les méthodes de fabrique `IndexAction.Upload`, `IndexAction.Merge` et ainsi de suite.

### <a name="removed-preview-features"></a>Fonctionnalités préliminaires supprimées

Si vous mettez à niveau à partir de la version 8.0-preview vers la version 9, n’oubliez pas que le chiffrement avec des clés gérées par le client a été supprimé dans la mesure où cette fonctionnalité est toujours en préversion. Plus précisément, les propriétés `EncryptionKey` de `Index` et `SynonymMap` ont été supprimées.

Si votre application dépend fortement de cette fonctionnalité, vous ne pourrez pas effectuer une mise à niveau vers la version 9 du SDK .NET Recherche Azure. Vous pouvez continuer à utiliser la version 8.0-preview. Mais n’oubliez pas que **nous déconseillons l’utilisation de Kits de développement logiciel en version préliminaire dans les applications de production**. Les fonctionnalités préliminaires servent uniquement à des fins d’évaluation et peuvent changer.

> [!NOTE]
> Si vous avez créé des index chiffrés ou des cartes de synonymes à l’aide de la version 8.0-preview du SDK, vous pourrez toujours les utiliser et modifier leurs définitions à l’aide de la version 9 du SDK sans nuire à leur état de chiffrement. La version 9 du kit SDK n’envoie pas la propriété `encryptionKey` de l’API REST et par conséquent l’API REST ne changera pas l’état de chiffrement de la ressource. 

### <a name="behavioral-change-in-data-retrieval"></a>Changement de comportement dans l’extraction de données

Si vous utilisez les API de « type dynamique » `Search`, `Suggest` ou `Get` qui retournent des instances de type `Document`, n’oubliez pas qu’elles désérialisent maintenant les tableaux JSON vides sur `object[]` au lieu de `string[]`.

## <a name="conclusion"></a>Conclusion
Si vous souhaitez plus d’informations sur l’utilisation du Kit de développement logiciel (SDK) Azure Search .NET, consultez les articles [Procédures .NET](search-howto-dotnet-sdk.md).

N’hésitez pas à nous faire part de vos commentaires sur le kit de développement logiciel. Si vous rencontrez des problèmes, n’hésitez pas à nous demander de l’aide sur [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-search). Si vous trouvez un bogue, vous pouvez signaler un problème dans le [Référentiel GitHub du Kit de développement logiciel (SDK) Azure .NET](https://github.com/Azure/azure-sdk-for-net/issues). N’oubliez pas de faire précéder le titre de votre problème du préfixe « [Recherche Azure] ».

Merci d’utiliser Azure Search !