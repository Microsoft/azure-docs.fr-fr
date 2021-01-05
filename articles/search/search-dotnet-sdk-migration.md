---
title: Effectuer une mise à niveau vers la version 3 du SDK .NET Recherche Azure
titleSuffix: Azure Cognitive Search
description: Migrez le code vers le SDK .NET de la Recherche Azure version 3 à partir de versions antérieures. Découvrez les nouveautés et les modifications de code nécessaires.
manager: nitinme
author: brjohnstmsft
ms.author: brjohnst
ms.service: cognitive-search
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 11/04/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 0e9f4602582a8c24f74d6603b732e7b91d97fa8d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89002689"
---
# <a name="upgrade-to-azure-search-net-sdk-version-3"></a>Effectuer une mise à niveau vers la version 3 du SDK .NET Recherche Azure

<!--- DETAILS in the word doc
cosmosdb
NER v1 skill 
Indexer execution result errors no longer have status
the data source API will no longer return in the response of any REST operation, the connection string specified by the user.
--->

Si vous utilisez la version préliminaire 2.0 ou une version antérieure du [Kit de développement logiciel (SDK) .NET Azure Search](/dotnet/api/overview/azure/search), cet article vous aidera à mettre à niveau votre application pour utiliser la version 3.

Pour avoir un aperçu général du kit de développement logiciel et avoir des exemples, voir [Comment utiliser Azure Search à partir d’une application .NET](search-howto-dotnet-sdk.md).

La version 3 du SDK .NET Azure Search contient des modifications par rapport aux versions antérieures. Elles sont pour la plupart mineures, et donc, la modification de votre code devrait ne nécessiter que peu d’effort. Consultez la page [Procédure de mise à niveau](#UpgradeSteps) pour obtenir des instructions sur la façon de modifier le code à utiliser avec la nouvelle version du kit de développement logiciel.

> [!NOTE]
> Si vous utilisez la version 1.0.2-preview ou une version antérieure, vous devez tout d’abord mettre à niveau vers la version 1.1, puis vers la version 3. Consultez [Mise à niveau vers la version du Kit de développement logiciel (SDK) .NET version 1.1 d’Azure Search](search-dotnet-sdk-migration-version-1.md) pour plus d’instructions.
>
> Votre instance de service Recherche Azure prend en charge plusieurs versions de l’API REST, y compris la plus récente. Vous pouvez continuer à utiliser une version lorsqu’elle n’est pas la plus récente, mais nous vous recommandons de migrer votre code pour utiliser la dernière version. Lorsque vous utilisez l’API REST, vous devez spécifier la version de l’API dans chaque requête via le paramètre « api-version » (Version de l’API). Lorsque vous utilisez le Kit de développement logiciel (SDK) .NET, la version du Kit de développement logiciel (SDK) que vous utilisez détermine la version correspondante de l’API REST. Si vous utilisez un Kit de développement logiciel (SDK) plus ancien, vous pouvez continuer à exécuter ce code sans changement, même si le service est mis à niveau pour prendre en charge une version plus récente de l’API.

<a name="WhatsNew"></a>

## <a name="whats-new-in-version-3"></a>Nouveautés de la version 3
La version 3 du SDK .NET Azure Search cible la dernière version disponible de l’API REST Azure Search, spécifiquement 2016-09-01. Cela permet d’utiliser de nombreuses nouvelles fonctionnalités d’Azure Search à partir d’une application .NET, notamment les suivantes :

* [Analyseurs personnalisés](index-add-custom-analyzers.md)
* Prise en charge des indexeurs [Stockage Blob Azure](search-howto-indexing-azure-blob-storage.md) et [Stockage Table Azure](search-howto-indexing-azure-tables.md)
* Personnalisation des indexeurs avec les [mappages de champs](search-indexer-field-mappings.md)
* Prise en charge des ETag pour permettre la mise à jour simultanée sécurisée des définitions d’index, des indexeurs et des sources de données
* Prise en charge de la création déclarative des définitions de champ index en décorant votre classe de modèle et en utilisant la nouvelle classe `FieldBuilder`.
* Prise en charge de .NET Core et .NET Portable Profile 111

<a name="UpgradeSteps"></a>

## <a name="steps-to-upgrade"></a>Procédure de mise à niveau
Tout d’abord, mettez à jour vos références NuGet `Microsoft.Azure.Search` , soit en utilisant la console NuGet Package, soit en effectuant un clic droit sur les références de votre projet et en sélectionnant « Gérer les packages NuGet » dans Visual Studio.

Une fois que NuGet a téléchargé les nouveaux packages et leurs dépendances, recompilez votre projet. En fonction de la structure de votre code, la recompilation peut réussir. Si c’est le cas, vous êtes prêt !

Si la compilation échoue, vous devriez voir une erreur de ce type :

```output
Program.cs(31,45,31,86): error CS0266: Cannot implicitly convert type 'Microsoft.Azure.Search.ISearchIndexClient' to 'Microsoft.Azure.Search.SearchIndexClient'. An explicit conversion exists (are you missing a cast?)
```

L’étape suivante consiste à corriger cette erreur de build. Consultez [Dernières modifications dans la version 3](#ListOfChanges) pour plus d’informations sur la cause de l’erreur et les possibilités de correction.

D’autres avertissements de compilation sont susceptibles de s’afficher ; ils sont liés à des propriétés ou des méthodes obsolètes. Ces avertissements incluent des instructions sur les fonctionnalités à utiliser à la place de la fonctionnalité déconseillée. Par exemple, si votre application utilise la propriété `IndexingParameters.Base64EncodeKeys`, vous devriez obtenir un avertissement indiquant `"This property is obsolete. Please create a field mapping using 'FieldMapping.Base64Encode' instead."`

Une fois que vous avez résolu les erreurs de build, vous pouvez apporter des modifications à votre application pour exploiter les nouvelles fonctionnalités si vous le souhaitez. Les nouvelles fonctionnalités du SDK sont détaillées dans [Nouveautés de la version 3](#WhatsNew).

<a name="ListOfChanges"></a>

## <a name="breaking-changes-in-version-3"></a>Dernières modifications dans la version 3
Il n’y a qu’un petit nombre de modifications dans la version 3 qui soient susceptibles de nécessiter des modifications de code en plus de la recompilation de votre application.

### <a name="indexesgetclient-return-type"></a>Type de retour Indexes.GetClient
La méthode `Indexes.GetClient` a un nouveau type de retour. Auparavant, elle renvoyait `SearchIndexClient`, mais elle a été remplacée par `ISearchIndexClient` dans la version 2.0-preview et cette modification a été répercutée dans la version 3. L’objectif est de prendre en charge les clients qui souhaitent simuler la méthode `GetClient` pour les tests unitaires en retournant une implémentation fictive de `ISearchIndexClient`.

#### <a name="example"></a>Exemple
Si votre code ressemble à ce qui suit :

```csharp
SearchIndexClient indexClient = serviceClient.Indexes.GetClient("hotels");
```

vous pouvez le modifier pour résoudre les éventuelles erreurs de build :

```csharp
ISearchIndexClient indexClient = serviceClient.Indexes.GetClient("hotels");
```

### <a name="analyzername-datatype-and-others-are-no-longer-implicitly-convertible-to-strings"></a>Les éléments AnalyzerName et DataType, entre autres, ne sont plus implicitement convertibles en chaînes
Il existe de nombreux types dans le SDK .NET Azure Search qui dérivent de `ExtensibleEnum`. Précédemment, ces types étaient tous implicitement convertibles en type `string`. Toutefois, un bogue a été découvert dans l'implémentation `Object.Equals` de ces classes et la résolution de ce bogue a nécessité la désactivation de cette conversion implicite. Une conversion explicite vers `string` reste autorisée.

#### <a name="example"></a>Exemple
Si votre code ressemble à ce qui suit :

```csharp
var customTokenizerName = TokenizerName.Create("my_tokenizer"); 
var customTokenFilterName = TokenFilterName.Create("my_tokenfilter"); 
var customCharFilterName = CharFilterName.Create("my_charfilter"); 
 
var index = new Index();
index.Analyzers = new Analyzer[] 
{ 
    new CustomAnalyzer( 
        "my_analyzer",  
        customTokenizerName,  
        new[] { customTokenFilterName },  
        new[] { customCharFilterName }), 
}; 
```

vous pouvez le modifier pour résoudre les éventuelles erreurs de build :

```csharp
const string CustomTokenizerName = "my_tokenizer"; 
const string CustomTokenFilterName = "my_tokenfilter"; 
const string CustomCharFilterName = "my_charfilter"; 
 
var index = new Index();
index.Analyzers = new Analyzer[] 
{ 
    new CustomAnalyzer( 
        "my_analyzer",  
        CustomTokenizerName,  
        new TokenFilterName[] { CustomTokenFilterName },  
        new CharFilterName[] { CustomCharFilterName })
}; 
```

### <a name="removed-obsolete-members"></a>Membres obsolètes supprimés

Vous pouvez voir des erreurs de build liées à des méthodes ou propriétés qui ont été marquées comme obsolètes dans la version 2.0-preview puis supprimées de la version 3. Si vous rencontrez ces erreurs, voici comment les résoudre :

- Si vous utilisiez ce constructeur : `ScoringParameter(string name, string value)`, remplacez-le par `ScoringParameter(string name, IEnumerable<string> values)`
- Si vous utilisiez la propriété `ScoringParameter.Value`, remplacez-la par la propriété `ScoringParameter.Values` ou la méthode `ToString`.
- Si vous utilisiez la propriété `SearchRequestOptions.RequestId`, remplacez-la par la propriété `ClientRequestId`.

### <a name="removed-preview-features"></a>Fonctionnalités préliminaires supprimées

Si vous effectuez une mise à niveau de la version 2.0-preview vers la version 3, n’oubliez pas que la prise en charge de l'analyse JSON et CSV des indexeurs d’objets blob a été supprimée car ces fonctionnalités sont toujours en version préliminaire. En particulier, les méthodes suivantes de la classe `IndexingParametersExtensions` ont été supprimées :

- `ParseJson`
- `ParseJsonArrays`
- `ParseDelimitedTextFiles`

Si votre application dépend fortement de ces fonctionnalités, vous ne pourrez pas mettre à niveau vers la version 3 du SDK .NET Azure Search. Vous pouvez continuer à utiliser la version 2.0-preview. Mais n’oubliez pas que **nous déconseillons l’utilisation de Kits de développement logiciel en version préliminaire dans les applications de production**. Les fonctionnalités préliminaires servent uniquement à des fins d’évaluation et peuvent changer.

## <a name="conclusion"></a>Conclusion
Si vous souhaitez plus d’informations sur l’utilisation du Kit de développement logiciel (SDK) Azure Search .NET, consultez les articles [Procédures .NET](search-howto-dotnet-sdk.md).

N’hésitez pas à nous faire part de vos commentaires sur le kit de développement logiciel. Si vous rencontrez des problèmes, n’hésitez pas à nous demander de l’aide sur [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-search). Si vous trouvez un bogue, vous pouvez signaler un problème dans le [Référentiel GitHub du Kit de développement logiciel (SDK) Azure .NET](https://github.com/Azure/azure-sdk-for-net/issues). N’oubliez pas de faire précéder le titre de votre problème du préfixe « [Recherche Azure] ».

Merci d’utiliser Azure Search !