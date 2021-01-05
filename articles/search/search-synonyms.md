---
title: Synonymes pour une extension de requête sur un index de recherche
titleSuffix: Azure Cognitive Search
description: Créer une carte de synonymes pour élargir l’étendue d’une requête de recherche sur un index de Recherche cognitive Azure. L’étendue est élargie pour inclure des termes équivalents que vous fournissez dans une liste.
manager: nitinme
author: brjohnstmsft
ms.author: brjohnst
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 08/26/2020
ms.openlocfilehash: a8f1fa07b94072d37cf83320b6c8956d3b412f12
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95993582"
---
# <a name="synonyms-in-azure-cognitive-search"></a>Synonymes dans Recherche cognitive Azure

Dans les moteurs de recherche, les synonymes associent des termes équivalents qui élargissent implicitement l’étendue d’une requête, sans que l’utilisateur ait à fournir le terme. Par exemple, si l’on considère le terme « chien » et les associations de synonymes « canin » et « chiot », tous les documents contenant « chien », « canin » ou « chiot » seront pris en compte dans la requête.

Dans Recherche cognitive Azure, l’expansion des synonymes est effectuée au moment de la requête. Vous pouvez ajouter des cartes de synonymes à un service sans interrompre les opérations existantes. Vous pouvez ajouter une propriété **synonymMaps** à une définition de champ sans avoir à reconstruire l’index.

## <a name="create-synonyms"></a>Créer des synonymes

Le portail ne prend pas en charge la création de synonymes, mais vous pouvez utiliser l’API REST ou le kit SDK .NET. Pour prendre en main REST, nous vous recommandons [Postman ou Visual Studio Code](search-get-started-rest.md) et la formulation de requêtes à l’aide de cette API : [Créer des cartes de synonymes](/rest/api/searchservice/create-synonym-map). Pour les développeurs C#, vous pouvez commencer par [Ajouter des synonymes dans Recherche cognitive Azure à l’aide de C#](search-synonyms-tutorial-sdk.md).

En outre, si vous utilisez des [clés gérée par le client](search-security-manage-encryption-keys.md) pour le chiffrement au repos côté service, vous pouvez appliquer cette protection au contenu de votre carte de synonymes.

## <a name="use-synonyms"></a>Utiliser des synonymes

Dans Recherche cognitive Azure, la prise en charge des synonymes repose sur des cartes de synonymes que vous définissez et chargez sur votre service. Ces cartes constituent des ressources indépendantes (telles que des index ou des sources de données) et peuvent être utilisées par n’importe quel champ de recherche dans un index de votre service de recherche.

Les cartes de synonymes et les index sont gérés de manière indépendante. Une fois que vous définissez une carte de synonymes et la téléchargez sur votre service, vous pouvez activer la fonctionnalité de synonyme sur un champ en ajoutant une nouvelle propriété nommée **synonymMaps** dans la définition du champ. La création, la mise à jour et la suppression d’une carte de synonymes constituent des opérations qui s’appliquent au document entier, ce qui signifie que vous ne peut pas créer, mettre à jour ou supprimer des parties de la carte de synonyme de façon incrémentielle. Même la mise à jour d’une seule entrée nécessite un rechargement.

L’incorporation de synonymes dans votre application de recherche est un processus en deux étapes :

1.  Ajoutez une carte de synonymes à votre service de recherche via les API ci-dessous.  

2.  Configurez un champ pouvant faire l’objet d’une recherche pour utiliser la carte de synonymes dans la définition d’index.

Vous pouvez créer plusieurs cartes de synonymes pour votre application de recherche (par exemple, par langue si votre application prend en charge une base de clients multilingue). Actuellement, un champ peut uniquement utiliser une de ces deux méthodes. Vous pouvez mettre à jour la propriété synonymMaps d’un champ à tout moment.

### <a name="synonymmaps-resource-apis"></a>API de ressources SynonymMaps

#### <a name="add-or-update-a-synonym-map-under-your-service-using-post-or-put"></a>Ajoutez ou mettez à jour une carte de synonymes dans votre service à l’aide d’une requête POST ou PUT.

Les cartes de synonymes sont téléchargées sur le service via une requête POST ou PUT. Chaque règle doit être délimitée par le caractère de nouvelle ligne ('\n'). Vous pouvez définir jusqu'à 5 000 règles par carte de synonymes dans un service gratuit et 20 000 règles par carte dans toutes les autres références SKU. Chaque règle peut avoir jusqu'à 20 extensions.

Les cartes de synonymes doivent être au format Apache Solr décrit ci-dessous. Si vous disposez d’un dictionnaire de synonymes existant dans un autre format et souhaitez l’utiliser directement, faites-le-nous savoir sur [UserVoice](https://feedback.azure.com/forums/263029-azure-search).

Vous pouvez créer une nouvelle carte de synonymes à l’aide d’une requête HTTP POST, comme dans l’exemple suivant :

```synonym-map
    POST https://[servicename].search.windows.net/synonymmaps?api-version=2020-06-30
    api-key: [admin key]

    {
       "name":"mysynonymmap",
       "format":"solr",
       "synonyms": "
          USA, United States, United States of America\n
          Washington, Wash., WA => WA\n"
    }
```

Vous pouvez également utiliser une requête PUT en spécifiant le nom de la carte de synonymes sur l'URI. Si la carte de synonymes n'existe pas, elle est créée.

```synonym-map
    PUT https://[servicename].search.windows.net/synonymmaps/mysynonymmap?api-version=2020-06-30
    api-key: [admin key]

    {
       "format":"solr",
       "synonyms": "
          USA, United States, United States of America\n
          Washington, Wash., WA => WA\n"
    }
```

##### <a name="apache-solr-synonym-format"></a>Format de synonymes Apache Solr

Le format Solr prend en charge les cartes de synonymes équivalentes et explicites. Les règles de mappage respectent la spécification de filtre de synonyme open source d’Apache Solr, décrite dans ce document : [SynonymFilter](https://cwiki.apache.org/confluence/display/solr/Filter+Descriptions#FilterDescriptions-SynonymFilter). Voici un exemple de règle pour des synonymes équivalents.

```
USA, United States, United States of America
```

Avec la règle ci-dessus, une requête de recherche « USA » s’étendra à « USA » OR « United States » OR « United States of America ».

Un mappage explicite est indiqué par une flèche « => ». Lorsqu’elle spécifiée, une séquence de termes d’une requête de recherche qui correspond à la partie gauche de « => » est remplacée par les alternatives sur la partie droite. Étant donné la règle ci-dessous, les requêtes de recherche « Washington », « Wash. » ou « WA » seront réécrites « WA ». Le mappage explicite s’applique dans le sens spécifié uniquement et ne réécrit pas la requête « WA » en « Washington » dans ce cas.

```
Washington, Wash., WA => WA
```

Si vous devez définir des synonymes qui contiennent des virgules, vous pouvez échapper celles-ci à l’aide d’une barre oblique inverse, comme dans l’exemple suivant :

```
WA\, USA, WA, Washington
```

Étant donné que la barre oblique inverse est elle-même un caractère spécial dans d’autres langages tels que JSON et C#, vous devrez probablement effectuer un double échappement. Par exemple, le code JSON envoyé à l’API REST pour la carte de synonymes ci-dessus ressemble à ceci :

```json
    {
       "format":"solr",
       "synonyms": "WA\\, USA, WA, Washington"
    }
```

#### <a name="list-synonym-maps-under-your-service"></a>Répertorier les cartes de synonymes de votre service.

```synonym-map
    GET https://[servicename].search.windows.net/synonymmaps?api-version=2020-06-30
    api-key: [admin key]
```

#### <a name="get-a-synonym-map-under-your-service"></a>Obtenir une carte de synonymes pour votre service.

```synonym-map
    GET https://[servicename].search.windows.net/synonymmaps/mysynonymmap?api-version=2020-06-30
    api-key: [admin key]
```

#### <a name="delete-a-synonyms-map-under-your-service"></a>Supprimer une carte de synonymes de votre service.

```synonym-map
    DELETE https://[servicename].search.windows.net/synonymmaps/mysynonymmap?api-version=2020-06-30
    api-key: [admin key]
```

### <a name="configure-a-searchable-field-to-use-the-synonym-map-in-the-index-definition"></a>Configurez un champ pouvant faire l’objet d’une recherche pour utiliser la carte de synonymes dans la définition d’index.

Une nouvelle propriété de champ **synonymMaps** permet de spécifier une carte de synonymes à utiliser pour un champ pouvant faire l’objet d’une recherche. Les cartes de synonymes sont des ressources de niveau de service et peuvent être référencées selon n’importe quel champ d’index dans le service.

```synonym-map
    POST https://[servicename].search.windows.net/indexes?api-version=2020-06-30
    api-key: [admin key]

    {
       "name":"myindex",
       "fields":[
          {
             "name":"id",
             "type":"Edm.String",
             "key":true
          },
          {
             "name":"name",
             "type":"Edm.String",
             "searchable":true,
             "analyzer":"en.lucene",
             "synonymMaps":[
                "mysynonymmap"
             ]
          },
          {
             "name":"name_jp",
             "type":"Edm.String",
             "searchable":true,
             "analyzer":"ja.microsoft",
             "synonymMaps":[
                "japanesesynonymmap"
             ]
          }
       ]
    }
```

**synonymMaps** peut être spécifié pour les champs pouvant faire l’objet d’une recherche de type « Edm.String » ou « Collection(Edm.String) ».

> [!NOTE]
> Vous ne pouvez utiliser qu’une carte de synonymes par champ. Si vous souhaitez utiliser plusieurs cartes de synonymes, faites-le-nous savoir sur [UserVoice](https://feedback.azure.com/forums/263029-azure-search).

## <a name="impact-of-synonyms-on-other-search-features"></a>Impact des synonymes sur les autres fonctionnalités de recherche

La fonctionnalité de synonymes réécrit la requête d’origine avec des synonymes utilisant l’opérateur OR. Pour cette raison, la mise en surbrillance des correspondances et les profils de score traitent les synonymes et le terme d’origine comme équivalents.

La fonctionnalité de synonymes s’applique aux requêtes de recherche et non aux filtres ou facettes. De même, les suggestions sont basées uniquement sur le terme d’origine : les correspondances de synonymes n’apparaissent pas dans la réponse.

Les extensions de synonymes ne s’appliquent pas aux termes de recherche génériques : les préfixes, correspondances partielles et les expressions régulières ne sont pas étendus.

Si vous devez faire une requête unique qui applique l’expansion de synonymes et des recherches avec des caractères génériques, des expressions régulières ou une correspondance approximative, vous pouvez combiner les requêtes avec la syntaxe d’OR. Par exemple, pour combiner des synonymes avec des caractères génériques pour une syntaxe de requête simple, le terme serait `<query> | <query>*`.

Si vous avez un index existant dans un environnement de déploiement (non production), faites des essais avec un petit dictionnaire pour voir comment l’ajout de synonymes modifie l’expérience de recherche, notamment son impact sur les profils de score, la mise en surbrillance des correspondances et les suggestions.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Créer une carte de synonymes](/rest/api/searchservice/create-synonym-map)