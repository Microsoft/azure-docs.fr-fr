---
title: Mettre à niveau vers la version 10 du Kit de développement logiciel (SDK) .NET
titleSuffix: Azure Cognitive Search
description: Migrez le code vers le SDK .NET Recherche cognitive Azure version 10 à partir de versions antérieures. Découvrez les nouveautés et les modifications de code nécessaires.
manager: nitinme
author: arv100kri
ms.author: arjagann
ms.service: cognitive-search
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 11/04/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: bfe24ff38446fa0d0ccea96799e6f42b561713bf
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89002808"
---
# <a name="upgrade-to-azure-cognitive-search-net-sdk-version-10"></a>Effectuer une mise à niveau vers la version 10 du SDK .NET Recherche cognitive Azure

Si vous utilisez la version 9.0-ou une version antérieure du [SDK .NET](/dotnet/api/overview/azure/search), cet article vous aidera à mettre à niveau votre application pour utiliser la version 10.

La Recherche Azure est renommée Recherche cognitive Azure dans la version 10, mais les espace de noms et les noms de package restent inchangés. Les versions antérieures du SDK (9.0 et précédentes) continuent d’utiliser l’ancien nom. Pour plus d’informations sur l’utilisation du SDK et pour découvrir des exemples, consultez [Guide pratique pour utiliser la Recherche cognitive Azure à partir d’une application .NET](search-howto-dotnet-sdk.md).

La version 10 ajoute plusieurs fonctionnalités et correctifs de bogues, ce qui la place au même niveau fonctionnel que la version de l’API REST `2019-05-06`. Dans les cas où une modification rompt le code existant, nous vous guiderons à travers les [étapes requises pour résoudre le problème](#UpgradeSteps).

> [!NOTE]
> Si vous utilisez la version 8.0-preview ou une version antérieure, vous devez tout d’abord effectuer une mise à niveau vers la version 9, puis vers la version 10. Consultez [Mise à niveau vers la version 9 du SDK .NET Recherche Azure](search-dotnet-sdk-migration-version-9.md) pour obtenir des conseils sur la migration.
>
> Votre instance de service de recherche prend en charge plusieurs versions de l’API REST, y compris la plus récente. Vous pouvez continuer à utiliser une version lorsqu’elle n’est pas la plus récente, mais nous vous recommandons de migrer votre code pour utiliser la dernière version. Lorsque vous utilisez l’API REST, vous devez spécifier la version de l’API dans chaque requête via le paramètre « api-version » (Version de l’API). Lorsque vous utilisez le Kit de développement logiciel (SDK) .NET, la version du Kit de développement logiciel (SDK) que vous utilisez détermine la version correspondante de l’API REST. Si vous utilisez un Kit de développement logiciel (SDK) plus ancien, vous pouvez continuer à exécuter ce code sans changement, même si le service est mis à niveau pour prendre en charge une version plus récente de l’API.

<a name="WhatsNew"></a>

## <a name="whats-new-in-version-10"></a>Nouveautés de la version 10
Version 10 du SDK .NET Recherche cognitive Azure cible l’API REST (`2019-05-06`) avec les mises à jour suivantes :

* Présentation de deux nouvelles compétences : [compétence Conditionnelle](cognitive-search-skill-conditional.md) et [compétence Traduction de texte](cognitive-search-skill-text-translation.md).
* Les entrées de [compétence du modélisateur](cognitive-search-skill-shaper.md) ont été restructurées pour s’adapter à la consolidation des contextes imbriqués. Pour plus d’informations, consultez cet [exemple de définition JSON](./cognitive-search-skill-shaper.md#scenario-3-input-consolidation-from-nested-contexts).
* Ajout de deux nouvelles [fonctions de mappage de champs](search-indexer-field-mappings.md) :
    - [urlEncode](./search-indexer-field-mappings.md#urlencode-function)
    - [urlDecode](./search-indexer-field-mappings.md#urldecode-function)
* À certaines occasions, les erreurs et avertissements qui s’affichent dans [l’état d’exécution de l’indexeur](/rest/api/searchservice/get-indexer-status) peuvent contenir des détails supplémentaires qui facilitent le débogage. `IndexerExecutionResult` a été mis à jour pour refléter ce comportement.
* Les compétences individuelles définies dans un [ensemble de compétences](cognitive-search-defining-skillset.md) peuvent éventuellement être identifiées en spécifiant une propriété `name`.
* `ServiceLimits` affiche les limites pour les [types complexes](./search-howto-complex-data-types.md) et `IndexerExecutionInfo` affiche les limites/quotas pertinent(e)s de l’indexeur.

<a name="UpgradeSteps"></a>

## <a name="steps-to-upgrade"></a>Procédure de mise à niveau

1. Mettez à jour vos références NuGet `Microsoft.Azure.Search`, soit en utilisant la console NuGet Package, soit en effectuant un clic droit sur les références de votre projet et en sélectionnant « Gérer les packages NuGet » dans Visual Studio.

2. Une fois que NuGet a téléchargé les nouveaux packages et leurs dépendances, recompilez votre projet. 

3. Si votre build échoue, vous devrez résoudre chaque erreur de build. Consultez [Changements cassants dans la version 10](#ListOfChanges) pour en savoir plus sur la façon de résoudre chaque erreur de build potentielle.

4. Une fois que vous avez résolu les erreurs de build ou les avertissements sur la génération, vous pouvez apporter des changements à votre application pour exploiter les nouvelles fonctionnalités, si vous le souhaitez. Les nouvelles fonctionnalités du Kit SDK sont détaillées dans la section [Nouveautés de la version 10](#WhatsNew).

<a name="ListOfChanges"></a>

## <a name="breaking-changes-in-version-10"></a>Changements cassants dans la version 10

Il y a plusieurs changements cassants dans la version 10 qui sont susceptibles de nécessiter des modifications de code en plus de la régénération de votre application.

> [!NOTE]
> La liste des modifications ci-dessous n’est pas exhaustive. Certaines modifications ne généreront probablement pas d’erreur de build mais un arrêt technique, car elles rompent la compatibilité binaire avec des assemblys qui dépendent de versions antérieures d’assemblys du SDK .NET Azure Cognitive Search. Les changements significatifs qui appartiennent à cette catégorie sont également répertoriés avec des recommandations. Régénérez votre application lors de la mise à niveau vers la version 10 pour éviter tout problème de compatibilité binaire.

### <a name="custom-web-api-skill-definition"></a>Définition de compétence API web personnalisée

La définition de la [compétence API web personnalisée](cognitive-search-custom-skill-web-api.md) n’a pas été correctement spécifiée dans la version 9 et les versions antérieures. 

Le modèle de `WebApiSkill` spécifiait `HttpHeaders` en tant que propriété d’objet qui _contient_ un dictionnaire. La création d’un ensemble de compétences avec un `WebApiSkill` construit de cette manière entraînait une exception, car l’API REST considérait que la demande était mal formée. Ce problème a été corrigé, en faisant de `HttpHeaders` une **propriété de dictionnaire de niveau supérieur** sur le modèle `WebApiSkill` proprement dit, ce qui est considéré comme une demande valide de l’API REST.

Par exemple, si vous avez précédemment tenté d’instancier un `WebApiSkill` comme suit :

```csharp

var webApiSkill = new WebApiSkill(
            inputs, 
            outputs,
            uri: "https://contoso.example.org")
{
    HttpHeaders = new WebApiHttpHeaders()
    {
        Headers = new Dictionary<string, string>()
        {
            ["header"] = "value"
        }
    }
};

```

Remplacez-le par ce qui suit, afin d’éviter l’erreur de validation de l’API REST :

```csharp

var webApiSkill = new WebApiSkill(
            inputs, 
            outputs,
            uri: "https://contoso.example.org")
{
    HttpHeaders = new Dictionary<string, string>()
    {
        ["header"] = "value"
    }
};

```

## <a name="shaper-skill-allows-nested-context-consolidation"></a>La compétence du modélisateur permet la consolidation des contextes imbriqués

La compétence du modélisateur peut désormais autoriser la consolidation des entrées à partir de contextes imbriqués. Pour activer ce changement, nous avons modifié `InputFieldMappingEntry` de sorte qu’il puisse être instancié en spécifiant simplement une propriété `Source`, ou les propriétés `SourceContext` et `Inputs`.

Vous n’aurez probablement pas besoin d’apporter des modifications au code. Toutefois, notez qu’une seule de ces deux combinaisons est autorisée. La procédure est la suivante :

- La création d’un `InputFieldMappingEntry` où seul `Source` est initialisé est valide.
- La création d’un `InputFieldMappingEntry` où seuls `SourceContext` et `Inputs` sont initialisés est valide.
- Toutes les autres combinaisons qui impliquent ces trois propriétés ne sont pas valides.

Si vous décidez de commencer à utiliser cette nouvelle fonctionnalité, assurez-vous d’abord que tous vos clients sont mis à jour pour utiliser la version 10, avant de déployer ce changement. Dans le cas contraire, il est possible qu’une mise à jour par un client (à l’aide d’une version antérieure du SDK) vers la compétence du modélisateur puisse entraîner des erreurs de validation.

> [!NOTE]
> Même si le modèle `InputFieldMappingEntry` sous-jacent a été modifié pour permettre la consolidation à partir de contextes imbriqués, son utilisation n’est valide que dans la définition d’une compétence de modélisateur. L’utilisation de cette fonctionnalité dans d’autres compétences, bien que valide au moment de la compilation, génère une erreur de validation au moment de l’exécution.

## <a name="skills-can-be-identified-by-a-name"></a>Les compétences peuvent être identifiées par un nom

Chaque compétence au sein d’un jeu de compétences a maintenant une nouvelle propriété `Name`, qui peut être initialisée dans votre code pour aider à identifier la compétence. Cette option est facultative. Quand elle n’est pas spécifiée (valeur par défaut, si aucun changement de code explicite n’a été apporté), un nom par défaut lui est attribué à l’aide de l’index de base 1 de la compétence dans le jeu de compétences, avec en préfixe le caractère « # ». Par exemple, dans la définition du jeu de compétences suivante (la plupart des initialisations ont été ignorées par souci de concision) :

```csharp
var skillset = new Skillset()
{
    Skills = new List<Skill>()
    {
        new SentimentSkill(),
        new WebApiSkill(),
        new ShaperSkill(),
        ...
    }
}
```

`SentimentSkill` reçoit le nom `#1`, `WebApiSkill` reçoit le nom `#2`, `ShaperSkill` reçoit le nom `#3`, etc.

Si vous choisissez d’identifier les compétences avec un nom personnalisé, veillez d’abord à mettre à jour toutes les instances de vos clients vers la version 10 du SDK. Dans le cas contraire, il est possible qu’un client qui utilise une version antérieure du SDK puisse `null` la propriété `Name` d’une compétence, ce qui amène le client à revenir au schéma de nommage par défaut.

## <a name="details-about-errors-and-warnings"></a>Détails sur les erreurs et les avertissements

Les modèles `ItemError` et `ItemWarning` qui encapsulent les détails des erreurs et des avertissements (respectivement) qui se produisent pendant l’exécution d’un indexeur ont été modifiés de façon à inclure trois nouvelles propriétés avec l’objectif de faciliter le débogage de l’indexeur. Ces propriétés sont :

- `Name`: Nom de la source à l’origine de l’erreur. Par exemple, cela peut faire référence à une compétence particulière dans le jeu de compétences joint.
- `Details`: Détails supplémentaires sur l’erreur ou l’avertissement.
- `DocumentationLink`: Lien vers un guide de dépannage pour l’erreur ou l’avertissement spécifique.

> [!NOTE]
> Nous avons commencé à structurer nos erreurs et avertissements pour inclure ces détails utiles chaque fois que cela est possible. Nous nous efforçons de faire en sorte que, pour l’ensemble des erreurs et des avertissements, ces détails soient présents, mais il s’agit d’un travail en cours et ces détails supplémentaires ne sont pas toujours remplis.

## <a name="next-steps"></a>Étapes suivantes

- Les modifications apportées à la compétence du modélisateur ont l’impact potentiel le plus important sur le code nouveau ou existant. Pour continuer, veillez à consulter à nouveau cet exemple illustrant la structure d’entrée : [Exemple de définition JSON de compétence du modélisateur](cognitive-search-skill-shaper.md)
- Consultez la [vue d’ensemble de l’enrichissement par IA](cognitive-search-concept-intro.md).
- N’hésitez pas à nous faire part de vos commentaires sur le kit de développement logiciel. Si vous rencontrez des problèmes, n’hésitez pas à nous demander de l’aide sur [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-search). Si vous trouvez un bogue, vous pouvez signaler un problème dans le [Référentiel GitHub du Kit de développement logiciel (SDK) Azure .NET](https://github.com/Azure/azure-sdk-for-net/issues). Veillez à faire précéder le titre de votre problème du préfixe « [Recherche cognitive Azure] ».