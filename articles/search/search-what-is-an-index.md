---
title: Créer un index de recherche
titleSuffix: Azure Cognitive Search
description: Présente les concepts et les outils d’indexation dans Recherche cognitive Azure, notamment les définitions de schéma et la structure physique des données.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 07/15/2020
ms.openlocfilehash: aa7c06c3bad59bad11fa288631042cca86109706
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94701131"
---
# <a name="create-a-basic-search-index-in-azure-cognitive-search"></a>Créer un index de recherche de base dans Recherche cognitive Azure

Dans Recherche cognitive Azure, un *index de recherche* stocke le contenu pouvant faire l’objet d’une recherche utilisé pour le texte intégral et les requêtes filtrées. Un index est défini par un schéma et enregistré dans le service, l’importation des données se faisant dans un deuxième temps. 

Les index contiennent des *documents*. Conceptuellement, un document correspond à une unité de données pouvant faire l’objet d’une recherche dans un index. Un détaillant peut posséder un document pour chaque produit, un organisme de presse peut posséder un document par article, et ainsi de suite. Pour comparer avec des éléments de base de données plus familiers, un *index de recherche* correspond à une *table*, et les *documents* équivalent plus ou moins aux *lignes* d’une table.

La structure physique d’un index est déterminée par le schéma, les champs marqués « Possibilité de recherche » donnant lieu à la création d’un index inversé pour ce champ. 

Vous pouvez créer un index à l’aide des outils et API suivants :

* Dans le portail Azure, utilisez l’Assistant **Ajouter un index** ou **Importer des données**
* À l’aide de [Create Index (REST API)](/rest/api/searchservice/create-index)
* À l’aide du [SDK .NET](./search-get-started-dotnet.md)

Il est plus facile d’apprendre avec un outil du portail. Le portail applique des exigences et des règles de schéma pour des types de données spécifiques, telles que l’interdiction de la recherche en texte intégral sur les champs numériques. Une fois que vous disposez d’un index exploitable, vous pouvez effectuer la transition vers le code en extrayant la définition JSON du service à l’aide de [Get Index (REST API)](/rest/api/searchservice/get-index) et en l’ajoutant à votre solution.

## <a name="recommended-workflow"></a>Workflow recommandé

L’élaboration d’un index final est un processus itératif. Il est courant de commencer par le portail pour créer l’index initial, puis de basculer vers le code pour placer l’index sous le contrôle de code source.

1. Déterminez si vous pouvez utiliser [**Importer des données**](search-import-data-portal.md). L’Assistant effectue une indexation tout-en-un basée sur un indexeur si les données sources proviennent d’un [type de source de données pris en charge dans Azure](search-indexer-overview.md#supported-data-sources).

1. Si vous ne pouvez pas utiliser **Importer des données**, commencez par **Ajouter un index** pour définir le schéma.

   ![Add index command](media/search-what-is-an-index/add-index.png "Ajouter une commande d’index")

1. Fournissez un nom et une clé utilisés pour identifier de manière unique chaque document de recherche dans l’index. La clé est obligatoire et doit être de type Edm.String. Lors de l’importation, vous devez prévoir de faire correspondre à ce champ un champ unique dans les données sources. 

   Le portail vous donne un champ `id` pour la clé. Pour remplacer le `id` par défaut, créez un nouveau champ (par exemple, une nouvelle définition de champ appelée `HotelId`), puis sélectionnez-le dans **Clé**.

   ![Fill in required properties](media/search-what-is-an-index//field-attributes.png "Renseigner les propriétés requises")

1. Ajoutez d’autres champs. Le portail vous indique les [attributs de champ](#index-attributes) disponibles en fonction des types de données. Si vous débutez dans la conception d’index, cela vous sera utile.

   Si les données entrantes sont de nature hiérarchique, attribuez le type de données [Type complexe](search-howto-complex-data-types.md) pour représenter les structures imbriquées. L’exemple de jeu de données intégré, Hotels, illustre des types complexes utilisant une adresse (contenant plusieurs sous-champs) qui entretient une relation un-à-un avec chaque hôtel, et une collection complexe Rooms, où plusieurs chambres sont associées à chaque hôtel. 

1. Attribuez des [analyseurs](#analyzers) aux champs de type chaîne avant la création de l’index. Procédez de même pour les [suggesteurs](#suggesters) si vous souhaitez activer l’Autocomplétion sur des champs spécifiques.

1. Cliquez sur **Créer** pour générer les structures physiques dans votre service de recherche.

1. Après la création d’un index, utilisez des commandes supplémentaires pour vérifier les définitions ou ajouter d’autres éléments.

   ![Ajouter une page d’index avec les attributs par type de données](media/search-what-is-an-index//field-definitions.png "Ajouter une page d’index avec les attributs par type de données")

1. Téléchargez le schéma d’index à l’aide de [Get Index (REST API)](/rest/api/searchservice/get-index) et d’un outil de test web comme [Postman](search-get-started-rest.md). Vous disposez maintenant d’une représentation JSON de l’index que vous pouvez adapter pour le code.

1. [Chargez votre index avec des données](search-what-is-data-import.md). La Recherche cognitive Azure accepte les documents JSON. Pour charger vos données par programmation, vous pouvez utiliser Postman avec des documents JSON dans la charge utile de demande. S’il n’est pas facilement d’exprimer vos données au format JSON, cette étape sera la plus fastidieuse. 

    Une fois qu’un index est chargé avec des données, la plupart des modifications apportées aux champs existants nécessitent la suppression et la régénération d’un index.

1. Interrogez votre index, examinez les résultats et continuez d’itérer sur le schéma d’index jusqu’à ce que vous commenciez à obtenir les résultats attendus. Pour interroger votre index, vous pouvez utiliser l’[**Explorateur de recherche**](search-explorer.md) ou Postman.

Pendant le développement, prévoyez des régénérations fréquentes. Comme les structures physiques sont créées dans le service, il est nécessaire de [supprimer et de recréer les index](search-howto-reindex.md) pour la plupart des modifications apportées à une définition de champ existante. Vous pouvez envisager de travailler sur une partie de vos données pour regénérer plus rapidement. 

> [!Tip]
> Il est préférable d’utiliser du code plutôt que le portail pour travailler simultanément sur la conception de l’index et l’importation des données. Sinon, les outils tels que [Postman et Visual Studio Code](search-get-started-rest.md) s’avèrent utiles pour tester la preuve de concept aux phases initiales d’un projet de développement. Vous pouvez apporter des modifications incrémentielles à une définition d’index dans un corps de demande, puis envoyer la demande à votre service pour recréer un index en utilisant un schéma mis à jour.

## <a name="index-schema"></a>Schéma d’index

Un index doit avoir un nom et un champ clé désigné (Edm.string) dans la collection de champs. La [*collection de champs*](#fields-collection) correspond généralement à la majeure partie de l’index, dans laquelle chaque champ est nommé, tapé et pourvu de comportements autorisés qui déterminent son utilisation. 

Parmi les autres éléments, citons les [suggesteurs](#suggesters), les [profils de scoring](#scoringprofiles), les [analyseurs utilisés](#analyzers) pour traiter les chaînes en jetons selon des règles linguistiques ou d’autres caractéristiques prises en charge par l’analyseur et des paramètres [CORS (Cross-Origin Remote Scripting)](#corsoptions).

```json
{
  "name": (optional on PUT; required on POST) "name_of_index",
  "fields": [
    {
      "name": "name_of_field",
      "type": "Edm.String | Collection(Edm.String) | Edm.Int32 | Edm.Int64 | Edm.Double | Edm.Boolean | Edm.DateTimeOffset | Edm.GeographyPoint",
      "searchable": true (default where applicable) | false (only Edm.String and Collection(Edm.String) fields can be searchable),
      "filterable": true (default) | false,
      "sortable": true (default where applicable) | false (Collection(Edm.String) fields cannot be sortable),
      "facetable": true (default where applicable) | false (Edm.GeographyPoint fields cannot be facetable),
      "key": true | false (default, only Edm.String fields can be keys),
      "retrievable": true (default) | false,
      "analyzer": "name_of_analyzer_for_search_and_indexing", (only if 'searchAnalyzer' and 'indexAnalyzer' are not set)
      "searchAnalyzer": "name_of_search_analyzer", (only if 'indexAnalyzer' is set and 'analyzer' is not set)
      "indexAnalyzer": "name_of_indexing_analyzer", (only if 'searchAnalyzer' is set and 'analyzer' is not set)
      "synonymMaps": [ "name_of_synonym_map" ] (optional, only one synonym map per field is currently supported)
    }
  ],
  "suggesters": [
    {
      "name": "name of suggester",
      "searchMode": "analyzingInfixMatching",
      "sourceFields": ["field1", "field2", ...]
    }
  ],
  "scoringProfiles": [
    {
      "name": "name of scoring profile",
      "text": (optional, only applies to searchable fields) {
        "weights": {
          "searchable_field_name": relative_weight_value (positive #'s),
          ...
        }
      },
      "functions": (optional) [
        {
          "type": "magnitude | freshness | distance | tag",
          "boost": # (positive number used as multiplier for raw score != 1),
          "fieldName": "...",
          "interpolation": "constant | linear (default) | quadratic | logarithmic",
          "magnitude": {
            "boostingRangeStart": #,
            "boostingRangeEnd": #,
            "constantBoostBeyondRange": true | false (default)
          },
          "freshness": {
            "boostingDuration": "..." (value representing timespan leading to now over which boosting occurs)
          },
          "distance": {
            "referencePointParameter": "...", (parameter to be passed in queries to use as reference location)
            "boostingDistance": # (the distance in kilometers from the reference location where the boosting range ends)
          },
          "tag": {
            "tagsParameter": "..." (parameter to be passed in queries to specify a list of tags to compare against target fields)
          }
        }
      ],
      "functionAggregation": (optional, applies only when functions are specified) 
        "sum (default) | average | minimum | maximum | firstMatching"
    }
  ],
  "analyzers":(optional)[ ... ],
  "charFilters":(optional)[ ... ],
  "tokenizers":(optional)[ ... ],
  "tokenFilters":(optional)[ ... ],
  "defaultScoringProfile": (optional) "...",
  "corsOptions": (optional) {
    "allowedOrigins": ["*"] | ["origin_1", "origin_2", ...],
    "maxAgeInSeconds": (optional) max_age_in_seconds (non-negative integer)
  },
  "encryptionKey":(optional){
    "keyVaultUri": "azure_key_vault_uri",
    "keyVaultKeyName": "name_of_azure_key_vault_key",
    "keyVaultKeyVersion": "version_of_azure_key_vault_key",
    "accessCredentials":(optional){
      "applicationId": "azure_active_directory_application_id",
      "applicationSecret": "azure_active_directory_application_authentication_key"
    }
  }
}
```

<a name="fields-collection"></a>

## <a name="fields-collection-and-field-attributes"></a>Collection et attributs de champs

Les champs ont un nom, un type qui classe les données stockées et des attributs qui spécifient la façon dont le champ est utilisé.

### <a name="data-types"></a>Types de données

| Type | Description |
|------|-------------|
| Edm.String |Texte pour lequel un jeton peut éventuellement être généré pour la recherche en texte intégral (césure de mots, recherche de radical, etc). |
| Collection(Edm.String) |Liste de chaînes pouvant être éventuellement tokenisées pour la recherche en texte intégral. En théorie, il n’existe pas de limite supérieure quant au nombre d’éléments d’une collection, mais la limite supérieure de 16 Mo sur la taille de charge utile s’applique aux collections. |
| Edm.Boolean |Contient des valeurs true/false. |
| Edm.Int32 |Valeurs entières 32 bits. |
| Edm.Int64 |Valeurs entières 64 bits. |
| Edm.Double |Données numériques à double précision. |
| Edm.DateTimeOffset |Valeurs de date et heure représentées au format OData V4 (par exemple, `yyyy-MM-ddTHH:mm:ss.fffZ` ou `yyyy-MM-ddTHH:mm:ss.fff[+/-]HH:mm`). |
| Edm.GeographyPoint |Point représentant un emplacement géographique de la planète. |

Pour en savoir plus, consultez les [types de données pris en charge](/rest/api/searchservice/Supported-data-types).

<a name="index-attributes"></a>

### <a name="attributes"></a>Attributs

Les attributs d’un champ déterminent son utilisation, par exemple s’il est utilisé dans la recherche en texte intégral, la navigation par facettes, les opérations de tri et ainsi de suite. 

Les champs de type chaîne sont souvent marqués avec les attributs « Possibilité de recherche » et « Récupérable ». Vous pouvez utiliser les champs « Triable », « Filtrable » et « À choix multiple » pour affiner les résultats de la recherche.

|Attribut|Description|  
|---------------|-----------------|  
|« Possibilité de recherche » |Recherche en texte intégral, avec analyse lexicale (césure de mots) lors de l’indexation. Si vous définissez un champ avec possibilité de recherche sur une valeur comme « journée ensoleillée », cette valeur est fractionnée au niveau interne en jetons individuels « journée » et « ensoleillée ». Pour en savoir plus, consultez la rubrique [Fonctionnement de la recherche en texte intégral](search-lucene-query-architecture.md).|  
|« Filtrable » |Référencé dans les requêtes $filter. Les champs filtrables de type `Edm.String` ou `Collection(Edm.String)` ne font pas l’objet d’une analyse lexicale, les comparaisons ne concernent donc que les correspondances exactes. Par exemple, si vous définissez un champ avec la valeur « journée ensoleillée », la requête `$filter=f eq 'sunny'` ne renverra aucune correspondance, contrairement à `$filter=f eq 'sunny day'`. |  
|« Triable » |Le système trie les résultats par score par défaut, mais vous pouvez configurer le tri en fonction des champs des documents. Les champs de type `Collection(Edm.String)` ne sont pas « triables ». |  
|« À choix multiple » |Généralement utilisé dans une présentation des résultats de recherche qui inclut un nombre de correspondances par catégorie (par exemple, les hôtels dans une ville spécifique). Cette option ne peut pas être utilisée avec des champs de type `Edm.GeographyPoint`. Les champs de type `Edm.String` qui sont « filtrables », « triables » ou « à choix multiple » ne peuvent pas dépasser 32 Ko de longueur. Pour plus d’informations, consultez l’article [Créer un index (API REST)](/rest/api/searchservice/create-index).|  
|« Clé » |Identificateur unique des documents dans l’index. Un seul champ doit être choisi comme champ clé et il doit être de type `Edm.String`.|  
|« Récupérable » |Définit si le champ peut être retourné dans un résultat de recherche. Cet attribut est utile quand vous voulez utiliser un champ (comme *profit margin*) comme mécanisme de filtre, de tri ou de score, mais que vous ne voulez pas qu’il soit visible par l’utilisateur final. Il doit être `true` for `key` .|  

Même si vous pouvez ajouter de nouveaux champs à tout moment, les définitions de champ existantes sont verrouillées pour toute la durée de vie de l’index. C’est pourquoi les développeurs utilisent généralement le portail pour créer des index simples, tester des idées ou rechercher une définition de paramètre. Il est plus efficace d’effectuer des itérations fréquentes sur la conception d’un index si vous suivez une approche basée sur du code pour reconstruire l’index facilement.

> [!NOTE]
> Les API que vous utilisez pour créer un index ont différents comportements par défaut. Pour les [API REST](/rest/api/searchservice/Create-Index), la plupart des attributs sont activés par défaut (par exemple, « Possibilité de recherche » et « Récupérable » sont actifs pour les champs de type chaîne) et vous n’avez généralement besoin de les définir que si vous voulez les désactiver. Pour le Kit de développement logiciel (SDK) .NET, l’inverse est vrai. Pour les propriétés que vous ne définissez pas explicitement, l’option par défaut désactive le comportement de recherche correspondante, sauf si vous l’activez de façon spécifique.

## `analyzers`

L’élément analyseurs définit le nom de l’analyseur linguistique à utiliser pour le champ. Pour plus d’informations sur les différents analyseurs disponibles, consultez [Ajout d’analyseurs à un index Recherche cognitive Azure](search-analyzers.md). Les analyseurs peuvent être utilisés uniquement avec les champs pouvant faire l’objet d’une recherche. Une fois que l’analyseur est affecté à un champ, il ne peut plus être modifié, à moins de regénérer l’index.

## `suggesters`

Un suggesteur est une section du schéma qui définit quels champs d’un index sont utilisés pour prendre en charge l’autocomplétion et les requêtes prédictives dans les recherches. En général, les chaînes de recherche partielle sont envoyées à l’[API REST Suggestions](/rest/api/searchservice/suggestions) pendant que l’utilisateur tape une requête de recherche, et l’API retourne un ensemble de documents ou d’expressions suggérés. 

Les champs ajoutés à un suggesteur sont utilisés pour générer des termes de recherche à saisie semi-automatique (« type-ahead »). Tous les termes de recherche sont créés pendant l’indexation et stockés séparément. Pour plus d’informations sur la création d’une structure de suggesteur, consultez [Ajouter des suggesteurs](index-add-suggesters.md).

## `corsOptions`

Le code JavaScript côté client ne peut pas appeler d’API par défaut, car le navigateur empêche toutes les requêtes cross-origin. Pour autoriser les requêtes cross-origin dans l'index, activez CORS (partage des ressources cross-origin) en définissant l'attribut **corsOptions**. Pour des raisons de sécurité, seules les API de requête prennent en charge CORS. 

Les options suivantes peuvent être définies pour CORS :

+ **allowedOrigins** (obligatoire) : il s'agit de la liste des origines pouvant accéder à l’index. Cela signifie que le code JavaScript distribué à partir de ces origines est autorisé à interroger l’index (s'il fournit la bonne clé API). Chaque origine se présente généralement sous la forme `protocol://<fully-qualified-domain-name>:<port>`, bien que `<port>` soit souvent omis. Pour plus d'informations, voir [Partage des ressources cross-origin (Wikipédia)](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing).

  Si vous voulez autoriser l'accès à toutes les origines, incluez uniquement l’élément `*` dans le tableau **allowedOrigins**. Si *cette pratique est déconseillée pour les services de recherche de production*, elle est souvent utile pour le développement et le débogage.

+ **maxAgeInSeconds** (facultatif) : les navigateurs utilisent cette valeur pour déterminer la durée (en secondes) de mise en cache des réponses CORS préliminaires. Il doit s'agir d'un entier non négatif. Plus cette valeur est importante, meilleures sont les performances, mais plus il faut de temps pour que les modifications apportées à la stratégie CORS prennent effet. Si la valeur n'est pas définie, une durée par défaut de 5 minutes est utilisée.

## `scoringProfiles`

Un [profil de score](index-add-scoring-profiles.md) est une section du schéma qui définit des comportements de score personnalisés permettant de faire apparaître certains éléments plus haut dans les résultats de la recherche. Les profils de calcul de score sont constitués de pondérations et de fonctions de champ. Pour les utiliser, vous spécifiez un profil par nom dans la chaîne de requête.

Un profil de score par défaut fonctionne en arrière-plan pour calculer un score de recherche pour chaque élément d’un jeu de résultats. Vous pouvez utiliser le profil de score interne et sans nom. Vous pouvez aussi définir **defaultScoringProfile** pour utiliser par défaut un profil personnalisé, appelé chaque fois qu’aucun profil personnalisé n’est spécifié dans la chaîne de requête.

<a name="index-size"></a>

## <a name="attributes-and-index-size-storage-implications"></a>Attributs et taille de l’index (implications en matière de stockage)

La taille d’un index est déterminée par la taille des documents que vous chargez, plus la configuration de l’index, par exemple si vous incluez des suggesteurs, et la façon dont vous définissez des attributs sur des champs individuels. 

La capture d’écran suivante illustre les caractéristiques du stockage d’index résultant des différentes combinaisons d’attributs. L’index est basé sur l’**exemple d’index de biens immobiliers**, que vous pouvez créer facilement à l’aide de l’Assistant Importer des données. Bien que les schémas de l’index ne soient pas montrés, vous pouvez en déduire les attributs d’après le nom de l’index. Par exemple, pour l’index *realestate-searchable*, seul l’attribut « Possibilité de recherche » est sélectionné ; pour l’index *realestate-retrievable*, seul l’index « Récupérable » est sélectionné, et ainsi de suite.

![Taille de l’index en fonction de la sélection de l’attribut](./media/search-what-is-an-index/realestate-index-size.png "Taille de l’index en fonction de la sélection de l’attribut")

Bien que ces variantes d’index soient artificielles, nous pouvons nous y reporter pour nous faire une idée de la façon dont les attributs affectent le stockage. Le paramètre « Récupérable » fait-il croître l’index ? Non. L’ajout de champs à un **suggesteur** fait-il croître l’index ? Oui.

Les index qui prennent en charge le filtrage et le tri sont proportionnellement plus volumineux que ceux qui prennent en charge uniquement la recherche en texte intégral. Cela est dû au fait que les opérations de filtrage et de tri recherchent des correspondances exactes, ce qui nécessite la présence de chaînes textuelles verbatim. En revanche, les champs pouvant faire l’objet d’une recherche prenant en charge les requêtes en texte intégral utilisent des index inversés, qui sont remplis avec des termes assortis de jetons qui occupent moins d’espace que des documents entiers. 

> [!Note]
> L’architecture de stockage est considérée comme un détail d’implémentation de Recherche cognitive Azure et est susceptible d’évoluer sans préavis. Il n’est pas garanti que le comportement actuel persistera dans l’avenir.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous comprenez la composition de l’index, vous pouvez créer votre premier index sur le portail. Nous vous recommandons de commencer par l’Assistant **Importer des données**, en choisissant les sources de données hébergées *realestate-us-sample* ou *hotels-sample*.

> [!div class="nextstepaction"]
> [Assistant Importer des données (portail)](search-get-started-portal.md)

Pour les deux jeux de données, l’Assistant peut déduire un schéma d’index, importer les données et générer un index pouvant faire l’objet d’une recherche que vous pouvez interroger à l’aide de l’explorateur de recherche. Recherchez ces sources de données dans la page **Connexion à vos données** de l’Assistant **Importer des données**.

   ![Create a sample index](media/search-what-is-an-index//import-wizard-sample-data.png "Créer un exemple d’index")