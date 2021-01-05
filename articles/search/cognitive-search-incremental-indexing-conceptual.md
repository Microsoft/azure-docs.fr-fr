---
title: Concepts d’enrichissement incrémentiel (préversion)
titleSuffix: Azure Cognitive Search
description: Mettez en cache le contenu intermédiaire et les modifications incrémentielles du pipeline d’enrichissement par IA dans le stockage Azure pour protéger les investissements dans les documents traités existants. Cette fonctionnalité est actuellement disponible en préversion publique.
manager: nitinme
author: Vkurpad
ms.author: vikurpad
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/18/2020
ms.openlocfilehash: 9fb76c5c96795b8092c86e22acbab4ea5963b42e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90971634"
---
# <a name="incremental-enrichment-and-caching-in-azure-cognitive-search"></a>Enrichissement incrémentiel et mise en cache dans Recherche cognitive Azure

> [!IMPORTANT] 
> L’enrichissement incrémentiel est actuellement en version préliminaire publique. Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). 
> [Les préversions de l'API REST](search-api-preview.md) offrent cette fonctionnalité. Il n’y a pas de prise en charge de portail ou de SDK .NET pour l’instant.

*L’enrichissement incrémentiel* est une fonctionnalité qui cible des [ensembles de compétences](cognitive-search-working-with-skillsets.md). Elle tire parti du Stockage Azure pour enregistrer la sortie de traitement émise par un pipeline d’enrichissement en vue d’une réutilisation dans de futures exécutions de l’indexeur. Dans la mesure du possible, l’indexeur réutilise toute sortie mise en cache qui est toujours valide. 

L’enrichissement incrémentiel permet non seulement de préserver votre investissement financier dans le traitement (en particulier la reconnaissance optique des caractères et le traitement des images), mais aussi d’améliorer l’efficacité d’un système. Quand des structures et du content sont mis en cache, un indexeur peut déterminer les compétences qui ont changé et exécuter uniquement celles qui ont été modifiées, ainsi que toutes les compétences dépendantes en aval. 

Un flux de travail qui utilise une mise en cache incrémentielle comprend les étapes suivantes :

1. [Créer ou identifier un compte de stockage Azure](../storage/common/storage-account-create.md) pour stocker le cache.
1. [Activer l’enrichissement incrémentiel](search-howto-incremental-index.md) dans l’indexeur.
1. [Créer un indexeur](/rest/api/searchservice/create-indexer) (et un [ensemble de compétences](/rest/api/searchservice/create-skillset)) pour appeler le pipeline. Lors du traitement, les étapes d’enrichissement sont enregistrées pour chaque document dans le stockage d’objets blob en vue d’une utilisation ultérieure.
1. Testez votre code, puis, après avoir apporté des modifications, utilisez la commande [Mettre à jour l’ensemble de compétences](/rest/api/searchservice/update-skillset) pour modifier une définition.
1. [Exécutez l’indexeur](/rest/api/searchservice/run-indexer) pour appeler le pipeline, en extrayant la sortie mise en cache pour un traitement plus rapide et plus économique.

Pour plus d’informations sur les étapes et les considérations à prendre en compte lors de l’utilisation d’un indexeur existant, consultez [Configurer l’enrichissement incrémentiel](search-howto-incremental-index.md).

## <a name="indexer-cache"></a>Cache d’indexeur

L’enrichissement incrémentiel ajoute un cache au pipeline d’enrichissement. Cet indexeur met en cache les résultats du craquage de document, ainsi que les résultats de chaque compétence pour chaque document. Quand un ensemble de compétences est mis à jour, seules les compétences ayant changé ou situées en aval sont réexécutées. Les résultats mis à jour sont écrits dans le cache. Le document est mis à jour dans l’index ou dans la base de connaissances.

Physiquement, le cache est stocké dans un conteneur d’objets blob de votre compte de stockage Azure. Le cache utilise également le stockage table pour un enregistrement interne des mises à jour de traitement. Tous les index d’un service de recherche peuvent partager le même compte de stockage pour le cache de l’indexeur. Chaque indexeur se voir affecté un identificateur de cache unique et non modifiable au conteneur qu’il utilise.

## <a name="cache-configuration"></a>Configuration du cache

Vous devez définir la propriété `cache` sur l’indexeur pour commencer à bénéficier de l’enrichissement incrémentiel. L’exemple suivant illustre un indexeur pour lequel la mise en cache est activée. Des parties spécifiques de cette configuration sont décrites dans les sections suivantes. Pour plus d’informations, consultez [Configurer l’enrichissement incrémentiel](search-howto-incremental-index.md).

```json
{
    "name": "myIndexerName",
    "targetIndexName": "myIndex",
    "dataSourceName": "myDatasource",
    "skillsetName": "mySkillset",
    "cache" : {
        "storageConnectionString" : "Your storage account connection string",
        "enableReprocessing": true
    },
    "fieldMappings" : [],
    "outputFieldMappings": [],
    "parameters": []
}
```

Si vous définissez cette propriété sur un indexeur, vous devez également le réinitialiser et le réexécuter, ce qui entraînera un nouveau traitement de tous les documents présents dans votre source de données. Cette étape est nécessaire pour éliminer les documents enrichis par les versions précédentes des ensembles de compétences. 

## <a name="cache-management"></a>Gestion du cache

Le cycle de vie du cache est géré par l’indexeur. Si la propriété `cache` définie sur l’indexeur a une valeur null ou si la chaîne de connexion est modifiée, le cache existant est supprimé lors de la prochaine exécution de l’indexeur. Le cycle de vie du cache est également lié au cycle de vie de l’indexeur. Si un indexeur est supprimé, le cache associé est également supprimé.

Alors que l’enrichissement incrémentiel est conçu pour détecter les modifications et y répondre sans aucune intervention de votre part, vous pouvez utiliser certains paramètres pour remplacer les comportements par défaut :

+ Classer les nouveaux documents par ordre de priorité
+ Ignorer les vérifications des ensembles de compétences
+ Ignorer les vérifications des sources de données
+ Forcer l’évaluation des ensembles de compétences

### <a name="prioritize-new-documents"></a>Classer les nouveaux documents par ordre de priorité

Définissez la propriété `enableReprocessing` pour contrôler le traitement des documents entrants déjà représentés dans le cache. Lorsque la valeur est `true` (valeur par défaut), les documents qui se trouvent déjà dans le cache sont traités à nouveau lorsque vous réexécutez l’indexeur, en supposant que la mise à jour de vos compétences affecte ce document. 

Lorsque la valeur est `false`, les documents existants ne sont pas retraités, ce qui permet de hiérarchiser efficacement le nouveau contenu entrant par rapport au contenu existant. Vous devez uniquement définir `enableReprocessing` sur la valeur `false` de manière temporaire. Pour garantir la cohérence dans le corpus, la valeur de `enableReprocessing` doit être `true` la plupart du temps, afin que tous les documents, nouveaux ou existants, soient valides conformément à la définition actuelle de l’ensemble de compétences.

### <a name="bypass-skillset-evaluation"></a>Ignorer l’évaluation des ensembles de compétences

La modification d’un ensemble de compétences et le retraitement de cet ensemble de compétences vont généralement de pair. Toutefois, certaines modifications des ensembles de compétences ne n’entraînent pas de retraitement (par exemple, le déploiement d’une compétence personnalisée vers un nouvel emplacement ou avec une nouvelle clé d’accès). La plupart du temps, il s’agit de modifications périphériques qui n’ont pas d’impact réel sur la substance des ensembles de compétences. 

Si vous savez que la modification apportée à l’ensemble de compétences est en effet superficielle, vous pouvez remplacer l’évaluation des compétences en définissant le paramètre `disableCacheReprocessingChangeDetection` sur `true` :

1. Appelez Mettre à jour les compétences et modifiez la définition de l’ensemble de compétences.
1. Ajoutez le paramètre `disableCacheReprocessingChangeDetection=true` à la requête.
1. Envoyer la modification.

Ce paramètre garantit que seules les mises à jour de la définition de l’ensemble de compétences sont validées et que le changement n’est pas évalué en ce qui concerne ses effets sur le corpus existant.

L’exemple suivant illustre une requête de mise à jour de l’ensemble de compétences avec le paramètre :

```http
PUT https://customerdemos.search.windows.net/skillsets/callcenter-text-skillset?api-version=2020-06-30-Preview&disableCacheReprocessingChangeDetection=true
```

### <a name="bypass-data-source-validation-checks"></a>Ignorer les vérifications de validation de la source de données

La plupart des modifications apportées à la définition de source de données invalident le cache. Toutefois, dans les scénarios où vous savez qu’une modification ne doit pas invalider le cache, par exemple, la modification d’une chaîne de connexion ou la rotation de la clé sur le compte de stockage, ajoutez le paramètre `ignoreResetRequirement` à la mise à jour de la source de données. La définition de ce paramètre sur `true` permet à la validation se poursuivre sans déclencher de réinitialisation qui entraînerait la reconstruction de tous les objets et leur remplissage complet.

```http
PUT https://customerdemos.search.windows.net/datasources/callcenter-ds?api-version=2020-06-30-Preview&ignoreResetRequirement=true
```

### <a name="force-skillset-evaluation"></a>Forcer l’évaluation des ensembles de compétences

L’objectif du cache est d’éviter les traitements inutiles, mais supposons que vous apportiez une modification à une compétence que l’indexeur ne détecte pas (par exemple, en modifiant un texte dans du code externe, par exemple une compétence personnalisée).

Dans ce cas, vous pouvez utiliser les [compétences de réinitialisation](/rest/api/searchservice/preview-api/reset-skills) pour forcer le retraitement d’une compétence en particulier, y compris les compétences en aval qui dépendent du résultat de cette compétence. Cette API accepte la requête POST avec une liste de compétences qui doit être invalidée et marquée pour retraitement. Après avoir réinitialisé les compétences, exécutez l’indexeur pour appeler le pipeline.

## <a name="change-detection"></a>Détection des changements

Une fois que vous avez activé un cache, l’indexeur évalue les modifications apportées à la composition de votre pipeline pour déterminer le contenu qui peut être réutilisé et celui qui nécessite un retraitement. Cette section énumère les modifications qui invalident directement le cache, suivies des modifications qui déclenchent un traitement incrémentiel. 

### <a name="changes-that-invalidate-the-cache"></a>Modifications qui invalident le cache

Un changement invalidant est un changement qui rend la totalité du cache non valide. La mise à jour de votre source de données est un exemple de changement invalidant. Voici la liste complète des changements qui invalident votre cache :

* Changement de votre type de source de données
* Changement du conteneur de source de données
* Informations d’identification de la source de données
* Stratégie de détection des changements apportés à la source de données
* Stratégie de détection de la suppression des sources de données
* Mappages de champs de l’indexeur
* Paramètres d’indexeur
    * Mode d’analyse
    * Extensions de noms de fichiers exclues
    * Extensions de noms de fichiers indexées
    * Indexer les métadonnées de stockage uniquement pour les documents volumineux
    * En-têtes de texte délimité
    * Délimiteur de texte délimité
    * Racine du document
    * Action d’image (changements apportés au mode d’extraction des images)

### <a name="changes-that-trigger-incremental-processing"></a>Modifications qui déclenchent un traitement incrémentiel

Le traitement incrémentiel évalue la définition de votre ensemble de compétences et détermine les compétences à réexécuter, en mettant à jour de manière sélective les parties affectées de l’arborescence du document. Voici la liste complète des modifications entraînant un enrichissement incrémentiel :

* Une compétence de l’ensemble de compétences est d’un type distinct. Mise à jour du type OData de la compétence
* Mise à jour des paramètres spécifiques à une compétence, par exemple l’URL, les paramètres par défaut ou tout autre paramètre
* Changement des sorties d’une compétence, la compétence retourne des sorties supplémentaires ou distinctes
* Mises à jour de compétences entraînant un changement d’ancêtres, par exemple un changement du chaînage des compétences entrées de compétences
* Toute invalidation de compétence amont, si une compétence qui fournit une entrée à cette compétence est mise à jour
* Mises à jour de l’emplacement de projection de la base de connaissances, qui entraînent une reprojection des documents
* Apport de changements aux projections de la base de connaissances, qui entraînent une reprojection des documents
* Changements des mappages de champs de sortie d’un indexeur, qui entraînent une reprojection des documents dans l’index

## <a name="api-reference"></a>Informations de référence sur l'API

La version `2020-06-30-Preview` de l’API REST fournit un enrichissement incrémentiel par le biais de propriétés supplémentaires sur des indexeurs. Des ensembles de compétences et des sources de données peuvent utiliser la version généralement disponible. En plus de la documentation de référence, consultez [Configurer la mise en cache pour l’enrichissement incrémentiel](search-howto-incremental-index.md) pour plus d’informations sur la façon d’appeler les API.

+ [Créer un indexeur (api-version=2020-06-30-Preview)](/rest/api/searchservice/create-indexer) 

+ [Mettre à jour un indexeur (api-version=2020-06-30-Preview)](/rest/api/searchservice/update-indexer) 

+ [Mettre à jour un ensemble de compétences (api-version=2020-06-30)](/rest/api/searchservice/update-skillset) (nouveau paramètre d’URI sur la requête)

+ [Réinitialiser les compétences (api-version=2020-06-30)](/rest/api/searchservice/preview-api/reset-skills)

+ Indexeurs de base de données (SQL Azure, Cosmos DB). Certains indexeurs récupèrent les données par le biais de requêtes. Pour les requêtes qui récupèrent des données, [Mettre à jour la source de données](/rest/api/searchservice/update-data-source) prend en charge un nouveau paramètre de requête **ignoreResetRequirement**, qui doit avoir la valeur `true` quand votre action de mise à jour ne doit pas invalider le cache. 

  Utilisez **ignoreResetRequirement** avec modération, car cela peut entraîner une incohérence involontaire de vos données, qui n’est pas facilement détectée.

## <a name="next-steps"></a>Étapes suivantes

L’enrichissement incrémentiel est une fonctionnalité puissante qui étend le suivi aux ensembles de compétences et à l’enrichissement par IA. L'enrichissement incrémentiel permet de réutiliser le contenu traité existant au fur et à mesure que vous itérez au sein de la conception de compétences.

En guise d’étape suivante, activez la mise en cache sur un indexeur existant ou ajoutez un cache lors de la définition d’un nouvel indexeur.

> [!div class="nextstepaction"]
> [Configurer la mise en cache pour l’enrichissement incrémentiel](search-howto-incremental-index.md)