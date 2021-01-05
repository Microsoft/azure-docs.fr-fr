---
title: Effectuer une mise à niveau vers la version 5 du SDK .NET Recherche Azure
titleSuffix: Azure Cognitive Search
description: Migrez le code vers le SDK .NET Recherche Azure version 5 à partir de versions antérieures. Découvrez les nouveautés et les modifications de code nécessaires.
manager: nitinme
author: brjohnstmsft
ms.author: brjohnst
ms.service: cognitive-search
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 11/04/2019
ms.openlocfilehash: 7f0e994cd41fc55824cf347a51ccfed25589a310
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91539535"
---
# <a name="upgrade-to-azure-search-net-sdk-version-5"></a>Effectuer une mise à niveau vers la version 5 du SDK .NET Recherche Azure

Si vous utilisez la version 4.0-preview ou une version antérieure du [SDK .NET](/dotnet/api/overview/azure/search), cet article vous aidera à mettre à niveau votre application pour utiliser la version 5.

Pour avoir un aperçu général du kit de développement logiciel et avoir des exemples, voir [Comment utiliser Azure Search à partir d’une application .NET](search-howto-dotnet-sdk.md).

La version 5 du SDK .NET Recherche Azure contient des modifications par rapport aux versions antérieures. Elles sont pour la plupart mineures, et donc, la modification de votre code devrait ne nécessiter que peu d’effort. Consultez la page [Procédure de mise à niveau](#UpgradeSteps) pour obtenir des instructions sur la façon de modifier le code à utiliser avec la nouvelle version du kit de développement logiciel.

> [!NOTE]
> Si vous utilisez la version 2.0-preview ou une version antérieure, vous devez tout d’abord effectuer une mise à niveau vers la version 3, puis vers la version 5. Consultez [Mise à niveau vers la version du Kit de développement logiciel (SDK) .NET version 3 d’Azure Search](search-dotnet-sdk-migration.md) pour obtenir des conseils sur la migration.
>
> Votre instance de service Recherche Azure prend en charge plusieurs versions de l’API REST, y compris la plus récente. Vous pouvez continuer à utiliser une version lorsqu’elle n’est pas la plus récente, mais nous vous recommandons de migrer votre code pour utiliser la dernière version. Lorsque vous utilisez l’API REST, vous devez spécifier la version de l’API dans chaque requête via le paramètre « api-version » (Version de l’API). Lorsque vous utilisez le Kit de développement logiciel (SDK) .NET, la version du Kit de développement logiciel (SDK) que vous utilisez détermine la version correspondante de l’API REST. Si vous utilisez un Kit de développement logiciel (SDK) plus ancien, vous pouvez continuer à exécuter ce code sans changement, même si le service est mis à niveau pour prendre en charge une version plus récente de l’API.

<a name="WhatsNew"></a>

## <a name="whats-new-in-version-5"></a>Nouveautés de la version 5
La version 5 du SDK .NET Recherche Azure cible la dernière version en disponibilité générale de l’API REST Recherche Azure, spécifiquement 2017-11-11. Cela permet d’utiliser de nouvelles fonctionnalités de Recherche Azure à partir d’une application .NET, notamment les suivantes :

* [Synonymes](search-synonyms.md).
* Vous pouvez à présent accéder par programmation aux avertissements dans l’historique de l’exécution d’un indexeur (voir la propriété `Warning` d’`IndexerExecutionResult` dans les [informations de référence sur .NET](/dotnet/api/microsoft.azure.search.models.indexerexecutionresult) pour plus d’informations).
* Prise en charge de .NET Core 2.
* La nouvelle structure des packages vous permet de n’utiliser que les parties du SDK dont vous avez besoin (voir [Modifications notables dans la version 5](#ListOfChanges) pour plus d’informations).

<a name="UpgradeSteps"></a>

## <a name="steps-to-upgrade"></a>Procédure de mise à niveau
Tout d’abord, mettez à jour vos références NuGet `Microsoft.Azure.Search` , soit en utilisant la console NuGet Package, soit en effectuant un clic droit sur les références de votre projet et en sélectionnant « Gérer les packages NuGet » dans Visual Studio.

Une fois que NuGet a téléchargé les nouveaux packages et leurs dépendances, recompilez votre projet. En fonction de la structure de votre code, la recompilation peut réussir. Si c’est le cas, vous êtes prêt !

Si la compilation échoue, vous devriez voir une erreur de ce type :

```output
The name 'SuggesterSearchMode' does not exist in the current context
```

L’étape suivante consiste à corriger cette erreur de build. Pour plus d’informations sur la cause de l’erreur et les possibilités de correction, consultez [Modifications notables dans la version 5](#ListOfChanges).

Notez qu’en raison des modifications dans l’empaquetage du SDK .NET Recherche Azure, vous devez regénérer votre application pour pouvoir utiliser la version 5. Ces modifications sont détaillées dans [Modifications notables dans la version 5](#ListOfChanges).

D’autres avertissements de compilation sont susceptibles de s’afficher ; ils sont liés à des propriétés ou des méthodes obsolètes. Ces avertissements incluent des instructions sur les fonctionnalités à utiliser à la place de la fonctionnalité déconseillée. Par exemple, si votre application utilise la méthode `IndexingParametersExtensions.DoNotFailOnUnsupportedContentType`, vous devez obtenir un avertissement indiquant que le comportement concerné est désormais activé par défaut et qu’il n’est plus nécessaire d’appeler cette méthode.

Une fois que vous avez résolu les erreurs de build ou les avertissements sur la génération, vous pouvez apporter des changements à votre application pour exploiter les nouvelles fonctionnalités, si vous le souhaitez. Les nouvelles fonctionnalités du Kit SDK sont détaillées dans la section [Nouveautés de la version 5](#WhatsNew).

<a name="ListOfChanges"></a>

## <a name="breaking-changes-in-version-5"></a>Modifications notables dans la version 5

### <a name="new-package-structure"></a>Nouvelle structure des packages

La modification notable la plus importante dans la version 5 est que l’assembly `Microsoft.Azure.Search` et son contenu ont été divisés en quatre assemblys distincts qui sont désormais distribués sous la forme de quatre packages NuGet distincts :

 - `Microsoft.Azure.Search`: Il s’agit d’un méta-package qui inclut tous les autres packages Recherche Azure en tant que dépendances. Si vous effectuez une mise à niveau à partir d’une version antérieure du SDK, la mise à niveau de ce package et sa regénération devraient suffire pour commencer à utiliser la nouvelle version.
 - `Microsoft.Azure.Search.Data`: Utilisez ce package si vous développez une application .NET à l’aide de Recherche Azure et que vous devez uniquement interroger ou mettre à jour des documents dans vos index. Si vous devez également créer ou mettre à jour des index, des cartes de synonymes ou d’autres ressources de niveau service, utilisez le package `Microsoft.Azure.Search` à la place.
 - `Microsoft.Azure.Search.Service`: Utilisez ce package si vous développez un processus d’automatisation en .NET pour gérer les index Recherche Azure, cartes de synonymes, indexeurs, sources de données ou autres ressources de niveau service. Si vous devez uniquement interroger ou mettre à jour des documents dans vos index, utilisez le package `Microsoft.Azure.Search.Data` à la place. Si vous avez besoin de toutes les fonctionnalités de Recherche Azure, utilisez le package `Microsoft.Azure.Search` à la place.
 - `Microsoft.Azure.Search.Common`: Types courants requis par les bibliothèques .NET de Recherche Azure. Vous ne devriez pas avoir besoin d’utiliser ce package directement dans votre application ; il est uniquement destiné à être utilisé en tant que dépendance.
 
Cette modification marque techniquement une rupture, car plusieurs types ont été déplacés entre les assemblys. C’est pourquoi il est indispensable de regénérer votre application pour effectuer une mise à niveau vers la version 5 du SDK.

Il n’y a qu’un petit nombre d’autres modifications dans la version 5 qui soient susceptibles de nécessiter des modifications de code en plus de la regénération de votre application.

### <a name="change-to-suggesters"></a>Changement apporté à Suggesters 

Le constructeur `Suggester` n’a plus de paramètre `enum` pour `SuggesterSearchMode`. Cette énumération avait une seule valeur et était donc redondante. Si vous voyez des erreurs de build suite à ce changement, supprimez simplement les références au paramètre `SuggesterSearchMode`.

### <a name="removed-obsolete-members"></a>Membres obsolètes supprimés

Vous pouvez voir des erreurs de build liées à des méthodes ou propriétés qui ont été marquées comme obsolètes dans les versions antérieures, puis supprimées de la version 5. Si vous rencontrez ces erreurs, voici comment les résoudre :

- Si vous utilisiez la méthode `IndexingParametersExtensions.IndexStorageMetadataOnly`, utilisez `SetBlobExtractionMode(BlobExtractionMode.StorageMetadata)` à la place.
- Si vous utilisiez la méthode `IndexingParametersExtensions.SkipContent`, utilisez `SetBlobExtractionMode(BlobExtractionMode.AllMetadata)` à la place.

### <a name="removed-preview-features"></a>Fonctionnalités préliminaires supprimées

Si vous effectuez une mise à niveau de la version 4.0-preview vers la version 5, n’oubliez pas que la prise en charge de l’analyse CSV et des tableaux JSON des indexeurs d’objets blob a été supprimée, car ces fonctionnalités sont toujours en préversion. En particulier, les méthodes suivantes de la classe `IndexingParametersExtensions` ont été supprimées :

- `ParseJsonArrays`
- `ParseDelimitedTextFiles`

Si votre application dépend fortement de ces fonctionnalités, vous ne pourrez pas effectuer une mise à niveau vers la version 5 du SDK .NET Recherche Azure. Vous pouvez continuer à utiliser la version 4.0-preview. Mais n’oubliez pas que **nous déconseillons l’utilisation de Kits de développement logiciel en version préliminaire dans les applications de production**. Les fonctionnalités préliminaires servent uniquement à des fins d’évaluation et peuvent changer.

## <a name="conclusion"></a>Conclusion
Si vous souhaitez plus d’informations sur l’utilisation du Kit de développement logiciel (SDK) Azure Search .NET, consultez les articles [Procédures .NET](search-howto-dotnet-sdk.md).

N’hésitez pas à nous faire part de vos commentaires sur le kit de développement logiciel. Si vous rencontrez des problèmes, n’hésitez pas à nous demander de l’aide sur [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-search). Si vous trouvez un bogue, vous pouvez signaler un problème dans le [Référentiel GitHub du Kit de développement logiciel (SDK) Azure .NET](https://github.com/Azure/azure-sdk-for-net/issues). N’oubliez pas de faire précéder le titre de votre problème du préfixe « [Recherche Azure] ».

Merci d’utiliser Azure Search !