---
title: Configurer le cache et l’enrichissement incrémentiel (préversion)
titleSuffix: Azure Cognitive Search
description: Activez la mise en cache et préservez l’état du contenu enrichi pour un traitement contrôlé dans un ensemble de compétences cognitives. Cette fonctionnalité est actuellement disponible en préversion publique.
author: vkurpad
manager: eladz
ms.author: vikurpad
ms.service: cognitive-search
ms.devlang: rest-api
ms.topic: conceptual
ms.date: 01/06/2020
ms.openlocfilehash: a1b317b651b0e17c07eb17dbdb8a7c6657d39564
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90971623"
---
# <a name="how-to-configure-caching-for-incremental-enrichment-in-azure-cognitive-search"></a>Guide de configuration de la mise en cache pour l’enrichissement incrémentiel dans Recherche cognitive Azure

> [!IMPORTANT] 
> L’enrichissement incrémentiel est actuellement en préversion publique. Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). 
> [Les préversions de l’API REST](search-api-preview.md) offrent cette fonctionnalité. Il n’y a pas de prise en charge de portail ou de SDK .NET pour l’instant.

Cet article explique comment ajouter la mise en cache à un pipeline d’enrichissement afin de pouvoir modifier les étapes de façon incrémentielle sans avoir à effectuer une régénération à chaque fois. Par défaut, un ensemble de compétences n'a pas d'état, et toute modification de sa composition nécessite une réexécution complète de l'indexeur. Grâce à l’enrichissement incrémentiel, l’indexeur peut déterminer les parties de l’arborescence de documents qui doivent être actualisées en fonction des modifications détectées dans l’ensemble de compétences ou les définitions de l’indexeur. La sortie traitée existante est conservée et réutilisée dans la mesure du possible. 

Le contenu mis en cache est placé dans le stockage Azure à l’aide des informations de compte que vous fournissez. Le conteneur, nommé `ms-az-search-indexercache-<alpha-numerc-string>`, est créé lors de l’exécution de l’indexeur. Il doit être considéré comme un composant interne géré par votre service de recherche et ne doit pas être modifié.

Si le processus de configuration des indexeurs ne vous est pas familier, commencez par la [Présentation de l'indexeur](search-indexer-overview.md), puis passez aux [Ensembles de compétences](cognitive-search-working-with-skillsets.md) pour en savoir plus sur les pipelines d'enrichissement. Pour plus d’informations sur les concepts clés, consultez [Enrichissement incrémentiel](cognitive-search-incremental-indexing-conceptual.md).

## <a name="enable-caching-on-an-existing-indexer"></a>Activer la mise en cache sur un indexeur existant

Si vous avez un indexeur existant qui est déjà doté d’un ensemble de compétences, suivez les étapes de cette section pour ajouter la mise en cache. Dans le cadre d’une opération ponctuelle, vous devez réinitialiser et réexécuter l’indexeur entièrement pour que le traitement incrémentiel puisse prendre effet.

> [!TIP]
> En guise de preuve de concept, vous pouvez utiliser ce [démarrage rapide du portail](cognitive-search-quickstart-blob.md) pour créer les objets nécessaires, puis utiliser Postman ou le portail pour effectuer vos mises à jour. Vous souhaiterez peut-être joindre une ressource Cognitive Services facturable. L’exécution de l’indexeur à plusieurs reprises entraîne l’épuisement de l’allocation quotidienne gratuite avant que vous ne puissiez effectuer toutes les étapes.

### <a name="step-1-get-the-indexer-definition"></a>Étape 1 : Obtenir la définition de l’indexeur

Démarrez par un indexeur valide existant qui contient les composants suivants : source de données, ensemble de compétences, index. Votre indexeur doit être exécutable. 

À l’aide d’un client d’API, créez une [requête GET Indexer](/rest/api/searchservice/get-indexer) pour obtenir la configuration actuelle de l’indexeur. Lorsque vous utilisez la version préliminaire de l’API pour faire une requête GET sur l’indexeur, une propriété `cache` définie sur Null est ajoutée aux définitions.

```http
GET https://[YOUR-SEARCH-SERVICE].search.windows.net/indexers/[YOUR-INDEXER-NAME]?api-version=2020-06-30-Preview
Content-Type: application/json
api-key: [YOUR-ADMIN-KEY]
```

Copiez la définition de l’indexeur à partir de la réponse.

### <a name="step-2-modify-the-cache-property-in-the-indexer-definition"></a>Étape 2 : Modifier la propriété du cache dans la définition de l’indexeur

Par défaut, la propriété `cache`est Null. Utilisez un client d’API pour définir la configuration du cache (le portail ne prend pas en charge cette mise à jour particulière). 

Modifiez l’objet cache pour inclure les propriétés obligatoires et facultatives suivantes : 

+ La propriété `storageConnectionString` est obligatoire et doit être définie sur une chaîne de connexion de stockage Azure. 
+ La propriété booléenne `enableReprocessing` est facultative (`true` par défaut) et indique que l’enrichissement incrémentiel est activé. Si nécessaire, vous pouvez la définir sur `false` pour interrompre le traitement incrémentiel pendant que d’autres opérations gourmandes en ressources, telles que l’indexation de nouveaux documents, sont en cours d’exécution, puis la rétablir sur `true` plus tard.

```json
{
    "name": "<YOUR-INDEXER-NAME>",
    "targetIndexName": "<YOUR-INDEX-NAME>",
    "dataSourceName": "<YOUR-DATASOURCE-NAME>",
    "skillsetName": "<YOUR-SKILLSET-NAME>",
    "cache" : {
        "storageConnectionString" : "<YOUR-STORAGE-ACCOUNT-CONNECTION-STRING>",
        "enableReprocessing": true
    },
    "fieldMappings" : [],
    "outputFieldMappings": [],
    "parameters": []
}
```

### <a name="step-3-reset-the-indexer"></a>Étape 3 : Réinitialiser l’indexeur

Une réinitialisation de l’indexeur est nécessaire lors de la configuration de l’enrichissement incrémentiel pour que les indexeurs existants puissent garantir que tous les documents sont dans un état cohérent. Vous pouvez utiliser le portail ou un client d’API, ainsi que l’[API REST Reset Indexer](/rest/api/searchservice/reset-indexer) (réinitialisation de l’indexeur) pour cette tâche.

```http
POST https://[YOUR-SEARCH-SERVICE].search.windows.net/indexers/[YOUR-INDEXER-NAME]/reset?api-version=2020-06-30-Preview
Content-Type: application/json
api-key: [YOUR-ADMIN-KEY]
```

### <a name="step-4-save-the-updated-definition"></a>Étape 4 : Enregistrer la définition mise à jour

[Mettez à jour l’indexeur](/rest/api/searchservice/preview-api/update-indexer) à l’aide d’une requête PUT. Le corps de la requête doit contenir la définition mise à jour de l’indexeur dotée de la propriété du cache. Si vous recevez une erreur 400, vérifiez la définition de l’indexeur pour vous assurer que toutes les exigences sont satisfaites (source de données, ensemble de compétences, index).

```http
PUT https://[YOUR-SEARCH-SERVICE].search.windows.net/indexers/[YOUR-INDEXER-NAME]?api-version=2020-06-30-Preview
Content-Type: application/json
api-key: [YOUR-ADMIN-KEY]
{
    "name" : "<YOUR-INDEXER-NAME>",
    ...
    "cache": {
        "storageConnectionString": "<YOUR-STORAGE-ACCOUNT-CONNECTION-STRING>",
        "enableReprocessing": true
    }
}
```

Si vous émettez à présent une autre requête GET sur l’indexeur, la réponse du service inclut une propriété `ID` dans l’objet cache. La chaîne alphanumérique est ajoutée au nom du conteneur contenant tous les résultats mis en cache et l’état intermédiaire de chaque document traité par cet indexeur. L’ID sera utilisé pour nommer le cache de manière unique dans le stockage d’objets blob.

```http
    "cache": {
        "ID": "<ALPHA-NUMERIC STRING>",
        "enableReprocessing": true,
        "storageConnectionString": "DefaultEndpointsProtocol=https;AccountName=<YOUR-STORAGE-ACCOUNT>;AccountKey=<YOUR-STORAGE-KEY>;EndpointSuffix=core.windows.net"
    }
```

### <a name="step-5-run-the-indexer"></a>Étape 5 : Exécuter l’indexeur

Pour exécuter l’indexeur, vous pouvez utiliser le portail ou l’API. Dans le portail, à partir de la liste des indexeurs, sélectionnez l’indexeur souhaité, puis cliquez sur **Exécuter**. L’un des avantages à utiliser le portail est que vous pouvez surveiller l’état de l’indexeur, noter la durée du travail et connaître le nombre de documents traités. Les pages du portail sont actualisées à des intervalles de quelques minutes.

Vous pouvez également utiliser REST pour [exécuter l’indexeur](/rest/api/searchservice/run-indexer) :

```http
POST https://[YOUR-SEARCH-SERVICE].search.windows.net/indexers/[YOUR-INDEXER-NAME]/run?api-version=2020-06-30-Preview
Content-Type: application/json
api-key: [YOUR-ADMIN-KEY]
```

Après l’exécution de l’indexeur, vous trouverez le cache dans Stockage Blob Azure. Le nom du conteneur respecte le format suivant : `ms-az-search-indexercache-<YOUR-CACHE-ID>`

> [!NOTE]
> Une réinitialisation et une réexécution de l’indexeur entraînent une régénération complète afin que le contenu puisse être mis en cache. Tous les enrichissements cognitifs sont réexécutés sur tous les documents.
>

### <a name="step-6-modify-a-skillset-and-confirm-incremental-enrichment"></a>Étape 6 : Modifier un ensemble de compétences et confirmer l’enrichissement incrémentiel

Pour modifier un ensemble de compétences, vous pouvez utiliser le portail ou l’API. Par exemple, si vous utilisez la traduction de texte, une simple modification inlined de `en` à `es` ou une autre langue suffit pour tester la preuve de concept de l’enrichissement incrémentiel.

Réexécutez l’indexeur. Seules les parties d’une arborescence enrichie de documents sont mises à jour. Si vous avez utilisé le [démarrage rapide du portail](cognitive-search-quickstart-blob.md) comme preuve de concept, en modifiant la compétence de traduction de texte en « es », vous remarquerez que seuls 8 documents sont mis à jour au lieu des 14 originaux. Les fichiers image non concernés par le processus de traduction sont réutilisés à partir du cache.

## <a name="enable-caching-on-new-indexers"></a>Activer la mise en cache sur les nouveaux indexeurs

Pour configurer l’enrichissement incrémentiel d’un nouvel indexeur, il vous suffit d’inclure la propriété `cache` dans la charge utile de la définition de l’indexeur lors de l’appel de [Create Indexer (2020-06-30-Preview)](/rest/api/searchservice/preview-api/create-indexer). 


```json
{
    "name": "<YOUR-INDEXER-NAME>",
    "targetIndexName": "<YOUR-INDEX-NAME>",
    "dataSourceName": "<YOUR-DATASOURCE-NAME>",
    "skillsetName": "<YOUR-SKILLSET-NAME>",
    "cache" : {
        "storageConnectionString" : "<YOUR-STORAGE-ACCOUNT-CONNECTION-STRING>",
        "enableReprocessing": true
    },
    "fieldMappings" : [],
    "outputFieldMappings": [],
    "parameters": []
    }
}
```

## <a name="checking-for-cached-output"></a>Vérification de la sortie mise en cache

Le cache est créé, utilisé et géré par l’indexeur, et son contenu n’est pas représenté dans un format lisible. Pour déterminer si le cache est utilisé, la meilleure méthode consiste à exécuter l’indexeur et à comparer les métriques avant et après du temps d’exécution et du nombre de documents. 

Par exemple, supposez qu’il s’agit d’un ensemble de compétences qui commence par l’analyse des images et la reconnaissance optique de caractères (OCR) de documents numérisés, suivies de l’analyse en aval du texte résultant. Si vous modifiez une compétence de texte en aval, l’indexeur peut récupérer tout le contenu des images et de l’OCR précédemment traité à partir du cache, en mettant à jour et en traitant uniquement les modifications relatives au texte indiquées par vos modifications. Vous pouvez vous attendre à voir moins de documents dans le nombre de documents (par exemple 8/8 au lieu de 14/14 lors de l’exécution d’origine), des durées d’exécution plus courtes et moins de frais sur votre facture.

## <a name="working-with-the-cache"></a>Utilisation du cache

Une fois le cache opérationnel, les indexeurs vérifient le cache chaque fois que [Run Indexer](/rest/api/searchservice/run-indexer) (Exécuter l’indexeur) est appelé, afin de déterminer quelles parties de la sortie existante peuvent être utilisées. 

Le tableau suivant récapitule la relation entre différentes API et le cache :

| API           | Impact du cache     |
|---------------|------------------|
| [Create Indexer (2020-06-30-Preview)](/rest/api/searchservice/preview-api/create-indexer) | Crée et exécute un indexeur lors de la première utilisation, y compris la création d’un cache si la définition de votre indexeur le spécifie. |
| [Run Indexer](/rest/api/searchservice/run-indexer) | Exécute un pipeline d’enrichissement à la demande. Cette API lit le cache, le cas échéant, ou crée un cache si vous avez ajouté la mise en cache à une définition d’indexeur mise à jour. Lorsque vous exécutez un indexeur pour lequel la mise en cache est activée, l’indexeur omet des étapes si la sortie mise en cache peut être utilisée. Vous pouvez utiliser la version généralement disponible ou la préversion de cette API.|
| [Reset Indexer](/rest/api/searchservice/reset-indexer)| Efface toutes les informations d’indexation incrémentielles présentes dans l’indexeur. L’exécution suivante de l’indexeur (à la demande ou planifiée) est un retraitement complet à partir de zéro, y compris la réexécution de tous les ensembles de compétences et la recréation du cache. Sur le plan fonctionnel, cela équivaut à supprimer l’indexeur et à le recréer. Vous pouvez utiliser la version généralement disponible ou la préversion de cette API.|
| [Reset Skills](/rest/api/searchservice/preview-api/reset-skills) | Spécifie les compétences à réexécuter lors de l’exécution suivante de l’indexeur, même si vous n’avez modifié aucune compétence. Le cache est mis à jour en conséquence. Les sorties, telles qu’une banque de connaissances ou un index de recherche, sont actualisées à l’aide de données réutilisables provenant du cache et du nouveau contenu selon la compétence mise à jour. |

Pour plus d’informations sur le contrôle de ce qui se passe dans le cache, consultez [Gestion du cache](cognitive-search-incremental-indexing-conceptual.md#cache-management).

## <a name="next-steps"></a>Étapes suivantes

L’enrichissement incrémentiel s’applique aux indexeurs qui contiennent des ensembles de compétences. Comme étape suivante, consultez la documentation relative aux ensembles de compétences pour comprendre les concepts et la composition. 

En outre, une fois que vous avez activé le cache, vous souhaiterez connaître les paramètres et les API qui sont pris en compte dans la mise en cache, y compris la façon de substituer ou forcer des comportements particuliers. Pour plus d’informations, consultez les liens suivants :

+ [Concepts et composition des ensembles de compétences](cognitive-search-working-with-skillsets.md)
+ [Guide pratique pour créer un ensemble de compétences](cognitive-search-defining-skillset.md)
+ [Présentation de l’enrichissement incrémentiel et de la mise en cache](cognitive-search-incremental-indexing-conceptual.md)