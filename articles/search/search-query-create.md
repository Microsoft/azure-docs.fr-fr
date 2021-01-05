---
title: Créer une requête de base
titleSuffix: Azure Cognitive Search
description: Découvrez la procédure de construction d’une demande de requête dans Recherche cognitive, les outils et API à utiliser pour les tests et le code, ainsi que les décisions de requête, qui commencent par la conception d’index.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 12/14/2020
ms.openlocfilehash: db36a77d93735b151ad893b7e25ba86f104e7b90
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97510462"
---
# <a name="create-a-query-in-azure-cognitive-search"></a>Créer une requête dans la Recherche cognitive Azure

Si vous générez une requête pour la première fois, cet article décrit les outils et API dont vous aurez besoin, les méthodes utilisées pour créer une requête et la manière dont la structure et le contenu de l’index peuvent avoir un impact sur les résultats de la requête. Pour découvrir à quoi ressemble une demande de requête, commencez par consulter [Types et composition de requête](search-query-overview.md).

## <a name="choose-tools-and-apis"></a>Choisir les outils et les API

Vous aurez besoin d’un outil ou d’une API pour créer une requête. Les suggestions suivantes sont utiles pour les charges de travail de test et de production.

| Méthodologie | Description |
|-------------|-------------|
| Portail| Le [Navigateur de recherche (portail)](search-explorer.md) est une interface de requête dans le portail Azure qui exécute des requêtes dans des index sur le service de recherche sous-jacent. Le portail effectue les appels de l’API REST en arrière-plan jusqu’à l’opération [Recherche dans des documents](/rest/api/searchservice/search-documents), mais ne peut pas appeler la saisie semi-automatique, les suggestions ni la recherche de documents.<br/><br/> Vous pouvez sélectionner n’importe quel index et version de l’API REST, y compris la version préliminaire. Une chaîne de requête peut utiliser une syntaxe simple ou complète, avec prise en charge de tous les paramètres de requête (Filter, Select, searchFields, etc.). Dans le portail, lorsque vous ouvrez un index, vous pouvez utiliser le Navigateur de recherche en même temps que la définition JSON de l’index dans les onglets côte à côte pour faciliter l’accès aux attributs du champ. Vérifiez les champs pouvant faire l’objet d’une recherche, d’un tri, de filtrage et de facettes lors du test des requêtes. <br/>Recommandé pour les investigations précoces, le test et la validation. [En savoir plus.](search-explorer.md) |
| Outils de test web| [Postman ou Visual Studio Code](search-get-started-rest.md) sont des choix forts pour la formulation d’une demande [Recherche dans des documents](/rest/api/searchservice/search-documents) et de toute autre demande dans REST. Les API REST prennent en charge toutes les opérations de programmation possibles dans Recherche cognitive Azure et, lorsque vous utilisez un outil, tel que Postman ou Visual Studio Code, vous pouvez émettre des demandes de manière interactive pour comprendre le fonctionnement de la fonctionnalité avant d’investir dans du code. Un outil de test web constitue un bon choix si vous n’avez pas de droits d’administration ou de contributeur dans le portail Azure. Tant que vous avez une URL de recherche et une clé API de requête, vous pouvez utiliser les outils pour exécuter des requêtes sur un index existant. |
| Azure SDK | Lorsque vous êtes prêt à écrire du code, vous pouvez utiliser les bibliothèques clientes Azure.Search.Document des Kits de développement logiciel (SDK) Azure pour .NET, Python, JavaScript ou Java. Chaque Kit de développement logiciel (SDK) dispose de son propre calendrier de publication, mais vous pouvez créer et interroger des index dans chacun d’eux. <br/><br/>[SearchClient (.NET)](/dotnet/api/azure.search.documents.searchclient) peut être utilisé pour interroger un index de recherche dans C#.  [En savoir plus.](search-howto-dotnet-sdk.md)<br/><br/>[SearchClient (Python)](/dotnet/api/azure.search.documents.searchclient) peut être utilisé pour interroger un index de recherche dans Python. [En savoir plus.](search-get-started-python.md)<br/><br/>[SearchClient (JavaScript)](/dotnet/api/azure.search.documents.searchclient) peut être utilisé pour interroger un index de recherche dans JavaScript. [En savoir plus.](search-get-started-javascript.md) |

## <a name="set-up-a-search-client"></a>Configurer un client de recherche

Un client de recherche s’authentifie auprès du service de recherche, envoie des demandes et traite les réponses. Quel que soit l’outil ou l’API que vous utilisez, un client de recherche doit disposer des éléments suivants :

| Propriétés | Description |
|------------|-------------|
| Point de terminaison | Un service de recherche est adressable par URL au format suivant : `https://[service-name].search.windows.net`. |
| Clé d’accès d’API (administrateur ou requête) | Authentifie la demande auprès du service de recherche. |
| Nom d’index | Les requêtes sont toujours dirigées vers la collection de documents d’un index unique. Vous ne pouvez pas joindre des index ou créer des structures de données personnalisées ou temporaires en tant que requête cible. |
| Version de l'API | Les appels REST requièrent explicitement le `api-version` sur la demande. En revanche, les bibliothèques clientes dans le Kit de développement logiciel (SDK) Azure sont gérées par rapport à une version spécifique de l’API REST. Pour les Kits de développement logiciel (SDK), `api-version` est implicite. |

### <a name="in-the-portal"></a>Sur le portail

Le Navigateur de recherche et les autres outils du portail ont une connexion client intégrée au service, avec des index d’accès direct et d’autres objets à partir des pages du portail. L’accès aux outils, aux assistants et aux objets requiert l’appartenance au rôle Contributeur ou supérieur sur le service. 

### <a name="using-rest"></a>Utilisation de REST

Pour les appels REST, vous pouvez utiliser [Postman ou des outils similaires](search-get-started-rest.md) en tant que client pour spécifier une demande [Recherche dans des documents](/rest/api/searchservice/search-documents). Chaque requête est autonome. vous devez donc fournir le point de terminaison, le nom de l’index et la version de l’API à chaque demande. D’autres propriétés, Content-Type et clé API, sont transmises à l’en-tête de demande. 

Vous pouvez utiliser POST ou GET pour interroger un index. POST, avec les paramètres spécifiés dans le corps de la demande, est plus facile à utiliser. Si vous utilisez POST, veillez à inclure `docs/search` dans l’URL :

```http
POST https://myservice.search.windows.net/indexes/hotels-sample-index/docs/search?api-version=2020-06-30
{
    "count": true,
    "queryType": "simple",
    "search": "*"
}
```

### <a name="using-azure-sdks"></a>Utilisation de Kits de développement logiciel (SDK) Azure

Si vous utilisez un Kit de développement logiciel (SDK) Azure, vous allez créer le client dans le code. Tous les Kits de développement logiciel (SDK) fournissent des clients de recherche qui peuvent conserver l’état, ce qui permet la réutilisation des connexions. Pour les opérations de requête, vous allez instancier un **`SearchClient`** et attribuer des valeurs aux propriétés suivantes : Endpoint, Key, Index. Vous pouvez ensuite appeler le **`Search method`** pour transmettre la chaîne de requête. 

| Langage | Client | Exemple |
|----------|--------|---------|
| C# et .NET | [SearchClient](/dotnet/api/azure.search.documents.searchclient) | [Envoyer votre première requête de recherche dans C#](/dotnet/api/overview/azure/search.documents-readme#send-your-first-search-query) |
| Python      | [SearchClient](/python/api/azure-search-documents/azure.search.documents.searchclient) | [Envoyer votre première requête de recherche dans Python](/python/api/overview/azure/search-documents-readme#send-your-first-search-request) |
| Java        | [SearchClient](/java/api/com.azure.search.documents.searchclient) | [Envoyer votre première requête de recherche dans Java](/java/api/overview/azure/search-documents-readme#send-your-first-search-query)  |
| JavaScript  | [SearchClient](/javascript/api/@azure/search-documents/searchclient) | [Envoyer votre première requête de recherche dans JavaScript](/javascript/api/overview/azure/search-documents-readme#send-your-first-search-query)  |

## <a name="choose-a-parser-simple--full"></a>Choisir un analyseur : simple | full

Si votre requête est une recherche en texte intégral, un analyseur est utilisé pour traiter le contenu du paramètre de recherche. Recherche cognitive Azure offre deux analyseurs de requêtes. L’analyseur simple comprend la [syntaxe de requête simple](query-simple-syntax.md). Cet analyseur a été sélectionné comme valeur par défaut pour sa vitesse et son efficacité dans les requêtes de texte de forme libre. La syntaxe prend en charge les opérateurs de recherche courants (AND, OR, NOT) pour les recherches de termes et d’expressions, ainsi que la recherche de préfixe (`*`) (comme dans « sea* » pour Seattle et Seaside). Une recommandation générale consiste à essayer d’abord l’analyseur simple, puis à passer à l’analyseur complet si les exigences de l’application nécessitent des requêtes plus puissantes.

La [syntaxe de requête Lucene complète](query-Lucene-syntax.md#bkmk_syntax), activée lorsque vous ajoutez `queryType=full` à la demande, est basée sur l’[analyseur Apache Lucene](https://lucene.apache.org/core/6_6_1/queryparser/org/apache/lucene/queryparser/classic/package-summary.html).

Une syntaxe complète et une syntaxe simple se chevauchent dans la mesure où toutes deux prennent en charge le même préfixe et les mêmes opérations booléennes, mais la syntaxe complète fournit davantage d’opérateurs. Dans la syntaxe complète, il existe d’autres opérateurs pour les expressions booléennes, et plus d’opérateurs pour les requêtes avancées, telles que la recherche approximative, la recherche de caractères génériques, la recherche de proximité et les expressions régulières.

## <a name="choose-query-methods"></a>Choisir des méthodes d’interrogation

La recherche est essentiellement un exercice piloté par l’utilisateur, où les termes ou expressions sont collectés à partir d’une zone de recherche ou à partir d’événements de clic sur une page. Le tableau suivant récapitule les mécanismes par lesquels vous pouvez collecter les entrées utilisateur, ainsi que l’expérience de recherche attendue.

| Entrée | Expérience |
|-------|---------|
| [Méthode de recherche](/rest/api/searchservice/search-documents) | Un utilisateur tape des termes ou des expressions dans une zone de recherche, avec ou sans opérateurs, et clique sur Rechercher pour envoyer la demande. La recherche peut être utilisée avec des filtres sur la même demande, mais pas avec la saisie semi-automatique ni les suggestions. |
| [Méthode de saisie semi-automatique](/rest/api/searchservice/autocomplete) | Un utilisateur tape quelques caractères et des requêtes sont lancées après la saisie de chaque nouveau caractère. La réponse est une chaîne complétée à partir de l’index. Si la chaîne fournie est valide, l’utilisateur clique sur Rechercher pour envoyer cette requête au service. |
| [Méthode de suggestions](/rest/api/searchservice/suggestions) | Comme avec la saisie semi-automatique, un utilisateur tape quelques caractères et des requêtes incrémentielles sont générées. La réponse est une liste déroulante de documents correspondants, généralement représentée par quelques champs uniques ou descriptifs. Si l’une des sélections est valide, l’utilisateur clique dessus et le document correspondant est retourné. |
| [Navigation à facettes](/rest/api/searchservice/search-documents#query-parameters) | Une page affiche des liens de navigation cliquables ou des barres de navigation qui réduisent la portée de la recherche. Une structure de navigation à facettes est composée dynamiquement en fonction d’une requête initiale. Par exemple, `search=*` pour remplir une arborescence de navigation à facettes composée de chaque catégorie possible. Une structure de navigation à facettes est créée à partir d’une réponse à une requête, mais il s’agit également d’un mécanisme permettant d’exprimer la requête suivante. Dans la référence d’API REST, `facets` est documenté comme un paramètre de requête d’une opération Recherche dans des documents, mais peut être utilisé sans le paramètre `search`.|
| [Méthode de filtre](/rest/api/searchservice/search-documents#query-parameters) | Les filtres sont utilisés avec des facettes pour affiner les résultats. Vous pouvez également implémenter un filtre derrière la page, par exemple pour initialiser la page avec des champs spécifiques à une langue. Dans la référence d’API REST, `$filter` est documenté comme un paramètre de requête d’une opération Recherche dans des documents, mais peut être utilisé sans le paramètre `search`.|

## <a name="know-your-field-attributes"></a>Découvrir vos attributs de champ

Si vous avez déjà passé en revue les [notions de base d’une demande de requête](search-query-overview.md), vous vous souvenez peut-être que les paramètres de la demande de requête dépendent de la façon dont les champs sont attribués dans un index. Par exemple, pour être utilisé dans une requête, un filtre ou un ordre de tri, un champ doit pouvoir faire l’objet d’une *recherche*, d’un *filtrage* et d’un *tri*. De même, seuls les champs marqués comme *récupérables* peuvent apparaître dans les résultats. Lorsque vous commencez à spécifier les paramètres `search`, `filter` et `orderby` dans votre demande, veillez à vérifier les attributs à mesure que vous progressez pour éviter des résultats inattendus.

Dans la capture d’écran du portail ci-dessous de l’[exemple d’index Hotels](search-get-started-portal.md), seuls les deux derniers champs « LastRenovationDate » et « Rating » peuvent être utilisés dans une clause `"$orderby"` uniquement.

![Définition d’index pour l’exemple des hôtels](./media/search-query-overview/hotel-sample-index-definition.png "Définition d’index pour l’exemple Hôtel")

Pour obtenir une description des attributs de champ, consultez [Créer un index (API REST)](/rest/api/searchservice/create-index).

## <a name="know-your-tokens"></a>Découvrir vos jetons

Pendant l’indexation, le moteur de requête utilise un analyseur pour effectuer une analyse de texte sur les chaînes, ce qui optimise le potentiel de correspondance au moment de la requête. Au minimum, les chaînes sont en minuscules, mais elles peuvent également être soumises à la lemmatisation et à l’arrêt de la suppression de mots. Les chaînes plus grandes ou les mots composés sont généralement fractionnés en fonction des espaces blancs, des traits d’union ou des tirets, et indexés comme des jetons distincts. 

Ce qu’il importe de retenir ici, c’est qu’il peut exister des différences entre ce que vous pensez que votre index contient et ce qu’il contient en réalité. Si les requêtes ne retournent pas les résultats attendus, vous pouvez inspecter les jetons créés par l’analyseur à l’aide de [Analyser le texte (API REST)](/rest/api/searchservice/test-analyzer). Pour plus d’informations sur la création de jetons et l’impact sur les requêtes, consultez [Recherche de termes partiels et modèles avec des caractères spéciaux](search-query-partial-matching.md).

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez une meilleure compréhension du mode de construction d’une demande de requête, essayez les démarrages rapides suivants pour une expérience pratique.

+ [Navigateur de recherche](search-explorer.md)
+ [Guide pratique pour interroger dans REST](search-get-started-rest.md)
+ [Guide pratique pour interroger dans .NET](search-get-started-dotnet.md)
+ [Guide pratique pour interroger dans Python](search-get-started-python.md)
+ [Guide pratique pour interroger dans JavaScript](search-get-started-javascript.md)