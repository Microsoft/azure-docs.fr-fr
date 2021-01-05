---
title: Limites de service pour les niveaux tarifaires et les références SKU
titleSuffix: Azure Cognitive Search
description: Limites de service permettant de planifier la capacité et limites maximales des requêtes et réponses de la Recherche cognitive Azure.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 10/14/2020
ms.openlocfilehash: f3763857af1df8f34f38b36835a667c6610e1909
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92107825"
---
# <a name="service-limits-in-azure-cognitive-search"></a>Limites de service de la Recherche cognitive Azure

Les limites maximales du stockage, des charges de travail et des quantités d’index et autres objets varient selon que vous [approvisionnez le service Recherche cognitive Azure](search-create-service-portal.md) avec les niveaux tarifaires **Gratuit** , **De base** , **Standard** ou **Stockage optimisé** .

+ **Gratuit** est un service partagé multi-locataire qui est fourni avec votre abonnement Azure. 

+ **De base**  : fournit des ressources de calcul dédiées pour des charges de travail de production à plus petite échelle, mais partage une infrastructure réseau avec d’autres locataires.

+ Le niveau **Standard** est exécuté sur des ordinateurs dédiés, avec une capacité de stockage et de traitement beaucoup plus grande, et ce, à chaque niveau. Le niveau Standard se décompose en quatre catégories : S1, S2, S3 et S3 HD. La catégorie S3 HD (S3 High Density) est conçue pour des [utilisateurs multiples](search-modeling-multitenant-saas-applications.md) et de grandes quantités de petits index (trois mille index par service). S3 HD ne fournit pas la [fonctionnalité d’indexeur](search-indexer-overview.md) et l’ingestion des données doit tirer parti des API qui envoient (push) les données de la source vers l’index. 

+ **Stockage optimisé** s’exécute sur des ordinateurs dédiés avec plus de stockage total, de bande passante de stockage et de mémoire que **Standard** . Ce niveau cible les index volumineux et à variation lente. Stockage optimisé est disponible en deux niveaux : L1 et L2.

## <a name="subscription-limits"></a>Limites d’abonnement
[!INCLUDE [azure-search-limits-per-subscription](../../includes/azure-search-limits-per-subscription.md)]

## <a name="storage-limits"></a>Limites de stockage
[!INCLUDE [azure-search-limits-per-service](../../includes/azure-search-limits-per-service.md)]

<a name="index-limits"></a>

## <a name="index-limits"></a>Limites d’index

| Ressource | Gratuit | De base&nbsp;<sup>1</sup>  | S1 | S2 | S3 | S3&nbsp;HD | L1 | L2 |
| -------- | ---- | ------------------- | --- | --- | --- | --- | --- | --- |
| Nombre maximal d’index |3 |5 ou 15 |50 |200 |200 |1 000 par partition ou 3 000 par service |10 |10 |
| Nombre maximal de champs simples par index |1 000 |100 |1 000 |1 000 |1 000 |1 000 |1 000 |1 000 |
| Nombre maximal de champs de collection complexe par index |40 |40 |40 |40 |40 |40 |40 |40 |
| Nombre maximal d’éléments dans toutes les collections complexes par document&nbsp;<sup>2</sup> |3000 |3000 |3000 |3000 |3000 |3000 |3000 |3000 |
| Profondeur maximale des champs complexes |10 |10 |10 |10 |10 |10 |10 |10 |
| Nombre maximal de [générateurs de suggestions](/rest/api/searchservice/suggesters) par index |1 |1 |1 |1 |1 |1 |1 |1 |
| Nombre maximal de [profils de score](/rest/api/searchservice/add-scoring-profiles-to-a-search-index) par index |100 |100 |100 |100 |100 |100 |100 |100 |
| Nombre maximal de fonctions par profil |8 |8 |8 |8 |8 |8 |8 |8 |

<sup>1</sup> Les services de base créés avant décembre 2017 présentent des limites inférieures (5 au lieu de 15) sur les index. Le niveau De base est la seule référence soumise à une limite inférieure de 100 champs par index.

<sup>2</sup> Il existe une limite supérieure pour les éléments, car un grand nombre d’entre eux augmente considérablement la capacité de stockage nécessaire à votre index. Un élément d’une collection complexe est défini en tant que membre de cette collection. Supposons, par exemple, qu’un [document Hôtel avec une collection complexe de Chambres](search-howto-complex-data-types.md#indexing-complex-types), chaque espace de la collection Chambres est considéré comme un élément. Pendant l’indexation, le moteur d’indexation peut traiter en toute sécurité un maximum de 3 000 éléments dans l’ensemble du document. [Cette limite](search-api-migration.md#upgrade-to-2019-05-06) a été introduite dans `api-version=2019-05-06` et s’applique uniquement aux collections complexes, et non aux collections de chaînes ou aux champs complexes.

<a name="document-limits"></a>

## <a name="document-limits"></a>Limites du document 

À compter d’octobre 2018, il n’y a plus de limites du nombre de documents pour tout nouveau service créé, quel que soit le niveau facturable (De base, S1, S2, S3, S3 HD) et la région. Les services plus anciens créés avant octobre 2018 peuvent toujours être soumis à des limites de nombre de documents.

Pour déterminer si des limites du document sont définies pour votre service, utilisez l’[API REST GET Service Statistics](/rest/api/searchservice/get-service-statistics). Les limites du document figurent dans la réponse, `null` indiquant aucune limite.

> [!NOTE]
> Bien qu’il n’y ait aucune limite de document imposée par le service, il existe une limite de partition d’environ 24 milliards de documents par index sur les services de recherche de base, S1, S2 et S3. Pour S3 HD, la limite de partition est de 2 milliards de documents par index. Chaque élément d’une collection complexe compte comme un document distinct en termes de limites de partition.

### <a name="document-size-limits-per-api-call"></a>Limites de taille de document par appel d’API

La taille maximale d’un document lors de l’appel d’une API d’index est d’environ 16 mégaoctets.

La taille du document est en fait une limite de taille du corps de requête de l’API d’index. Étant donné que vous pouvez transmettre en une seule fois un lot de plusieurs documents à l’API d’index, la limite de taille dépend en réalité du nombre de documents présents dans le lot. Pour un lot comprenant un seul document, la taille maximale du document est de 16 Mo de JSON.

Lorsque vous estimez la taille du document, n’oubliez pas de prendre en compte uniquement les champs qui peuvent être utilisés par un service de recherche. Toutes les données binaires ou d’image des documents sources doivent être omises de vos calculs.  

## <a name="indexer-limits"></a>Limites de l’indexeur

Les durées d’exécution maximales existent pour fournir équilibre et stabilité au service dans son ensemble, mais l’indexation des jeux de données volumineux peut prendre plus de temps que la valeur maximale ne le permet. Si un travail d’indexation ne peut pas être terminé dans le délai maximal autorisé, essayez de l’exécuter selon une planification. Le planificateur effectue le suivi de l’état de l’indexation. Si une tâche d’indexation planifiée est interrompue pour une raison quelconque, à la prochaine exécution planifiée, l’indexeur peut repartir de là où il s’était arrêté.


| Ressource | Gratuit&nbsp;<sup>1</sup> | De base&nbsp;<sup>2</sup>| S1 | S2 | S3 | S3&nbsp;HD&nbsp;<sup>3</sup>|L1 |L2 |
| -------- | ----------------- | ----------------- | --- | --- | --- | --- | --- | --- |
| Nombre maximal d’indexeurs |3 |5 ou 15|50 |200 |200 |N/A |10 |10 |
| Nombre maximal de sources de données |3 |5 ou 15 |50 |200 |200 |N/A |10 |10 |
| Compétences maximales <sup>4</sup> |3 |5 ou 15 |50 |200 |200 |N/A |10 |10 |
| Charge d’indexation maximale par appel |10 000 documents |Limité uniquement par le nombre maximal de documents |Limité uniquement par le nombre maximal de documents |Limité uniquement par le nombre maximal de documents |Limité uniquement par le nombre maximal de documents |N/A |Aucune limite |Aucune limite |
| Planification minimale | 5 minutes |5 minutes |5 minutes |5 minutes |5 minutes |5 minutes |5 minutes | 5 minutes |
| Durée maximale d’exécution| 1-3 minutes |24 heures |24 heures |24 heures |24 heures |N/A  |24 heures |24 heures |
| Durée d’exécution maximale pour les indexeurs avec un ensemble de compétences <sup>5</sup> | 3-10 minutes |2 heures |2 heures |2 heures |2 heures |N/A  |2 heures |2 heures |
| Indexeur d’objets blob : taille maximale des objets blob, en Mo |16 |16 |128 |256 |256 |N/A  |256 |256 |
| Indexeur d’objets blob : nombre maximal de caractères du contenu extrait d’un objet blob |32 000 |64 000 |4&nbsp;millions |8&nbsp;millions |16&nbsp;millions |N/A |4&nbsp;millions |4&nbsp;millions |

<sup>1</sup> Les services du niveau Gratuit bénéficient d’une durée d’exécution maximale de l’indexeur de 3 minutes pour les sources d’objets blob, et de 1 minute pour toutes les autres sources de données. Pour l’indexation de l’intelligence artificielle qui appelle la méthode dans Cognitive Services, les services gratuits sont limités à 20 transactions gratuites par jour, une transaction étant définie comme un document qui traverse le pipeline d’enrichissement.

<sup>2</sup> Les services de base créés avant décembre 2017 présentent des limites inférieures (5 au lieu de 15) sur les index, les sources de données et les ensembles de compétences.

<sup>3</sup> Les services S3 HD ne comprennent pas de prise en charge de l’indexeur.

<sup>4</sup> Nombre maximal de 30 compétences par group de compétences.

<sup>5</sup> L’enrichissement par IA et l’analyse d’images sont gourmands en ressources et consomment une quantité disproportionnée de la puissance de traitement disponible. Le temps d’exécution de ces charges de travail a été réduit pour permettre l’exécution d’autres tâches dans la file d’attente.

> [!NOTE]
> Comme indiqué dans les [limites des index](#index-limits), les indexeurs appliquent également la limite supérieure de 3 000 éléments à toutes les collections complexes par document, en commençant par la dernière version de l’API en disponibilité générale qui prend en charge les types complexes (`2019-05-06`). Cela signifie que si vous avez créé votre indexeur avec une version antérieure de l’API, vous ne serez pas soumis à cette limite. Pour préserver une compatibilité maximale, un indexeur qui a été créé avec une version antérieure de l’API, puis mis à jour avec une version de l’API `2019-05-06` ou ultérieure, sera toujours **exclu** des limites. Les clients doivent être conscients de l’impact négatif dans le cas de collections complexes très grandes (comme indiqué précédemment) et nous recommandons vivement de créer les indexeurs avec la dernière version de l’API en disponibilité générale.

## <a name="shared-private-link-resource-limits"></a>Limites de ressource de liaison privée partagée

Les indexeurs peuvent accéder aux autres ressources Azure [via des points de terminaison privés](search-indexer-howto-access-private.md) gérés via [l’API de ressource de liaison privée partagée](/rest/api/searchmanagement/sharedprivatelinkresources). Cette section décrit les limites associées à cette fonctionnalité.

| Ressource | Gratuit | De base | S1 | S2 | S3 | S3 HD | L1 | L2
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Prise en charge de l’indexeur de point de terminaison privé | Non | Oui | Oui | Oui | Oui | Non | Oui | Oui |
| Prise en charge du point de terminaison privé pour les indexeurs avec un ensemble de compétences<sup>1</sup> | Non | Non | Non | Oui | Oui | Non | Oui | Oui |
| Nombre maximal de points de terminaison privés | N/A | 10 ou 30 | 100 | 400 | 400 | N/A | 20 | 20 |
| Nombre maximal de types de ressources distincts<sup>2</sup> | N/A | 4 | 7 | 15 | 15 | N/A | 4 | 4 |

<sup>1</sup> L’enrichissement par IA et l’analyse d’images sont gourmands en ressources et consomment une quantité disproportionnée de la puissance de traitement disponible. Pour cette raison, les connexions privées sont désactivées sur les niveaux inférieurs afin d’éviter un impact négatif sur les performances et la stabilité du service de recherche lui-même.

<sup>2</sup> Le nombre de types de ressources distincts est calculé comme le nombre de valeurs de `groupId` uniques utilisées dans toutes les ressources de liaison privée partagée pour un service de recherche donné, quel que soit l’état de la ressource.

## <a name="synonym-limits"></a>Limites des synonymes

Le nombre maximal de cartes de synonymes varie en fonction du niveau. Chaque règle peut avoir jusqu’à 20 expansions, où une expansion est un terme equivalvent. Par exemple, pour le mot « chat », l’association avec « minou », « félin » et « felis » (le genre des chats) est comptée comme 3 expansions.

| Ressource | Gratuit | De base | S1 | S2 | S3 | S3-HD |L1 | L2 |
| -------- | -----|------ |----|----|----|-------|---|----|
| Mappages de synonymes maximum |3 |3|5 |10 |20 |20 | 10 | 10 |
| Nombre maximal de règles par mappage |5 000 |20000|20000 |20000 |20000 |20000 | 20000 | 20000  |

## <a name="queries-per-second-qps"></a>Requêtes par seconde

Les estimations du nombre de requêtes par seconde doivent être développées indépendamment par chaque client. La taille et la complexité des index et des requêtes ainsi que la quantité de trafic sont les principaux facteurs qui déterminent le nombre de requêtes par seconde. Si ces facteurs sont inconnus, il est impossible d’établir des estimations significatives.

Les estimations sont plus prévisibles si elles sont calculées sur des services qui s’exécutent sur des ressources dédiées (niveaux De base et Standard). Vous pouvez mieux estimer les requêtes par seconde, car vous contrôlez davantage de paramètres. Pour obtenir de l’aide sur la manière d’aborder les estimations, consultez [Performances et optimisation de Recherche cognitive Azure](search-performance-optimization.md).

Pour les niveaux à stockage optimisé (L1 et L2), attendez-vous à un plus faible débit des requêtes et à une latence plus élevée que les niveaux Standard.

## <a name="data-limits-ai-enrichment"></a>Limites de données (enrichissement de l’IA)

Un [pipeline d’enrichissement par IA](cognitive-search-concept-intro.md) faisant appel à une ressource Analyse de texte pour la [reconnaissance d’entités](cognitive-search-skill-entity-recognition.md), l’[extraction de phrases clés](cognitive-search-skill-keyphrases.md), l’[analyse des sentiments](cognitive-search-skill-sentiment.md), la [détection de la langue](cognitive-search-skill-language-detection.md) et la [détection d’informations personnelles](cognitive-search-skill-pii-detection.md) est soumis à des limites de données. La taille maximale d’un enregistrement doit être de 50 000 caractères telle que mesurée par [`String.Length`](/dotnet/api/system.string.length). Si vous avez besoin de découper vos données avant de les envoyer à l’Analyseur des sentiments, utilisez la [compétence Fractionnement du texte](cognitive-search-skill-textsplit.md).

## <a name="throttling-limits"></a>Limites de la limitation

Les requêtes de recherche et les demandes d’indexation sont limitées dès que le système s’approche de la capacité maximale. Le comportement de la limitation varie en fonction des API. Les API de requête (recherche/suggestion/saisie semi-automatique) et les API d’indexation se limitent dynamiquement en fonction de la charge du service. Les API d’indexation sont soumises à des limites de taux de requêtes statiques. 

Limites de taux de requêtes statiques pour les opérations liées à un index :

+ Lister les index (GET /indexes) : 5 par seconde par unité de recherche
+ Obtenir un index (GET /indexes/myindex) : 10 par seconde par unité de recherche
+ Créer un index (POST /indexes) : 12 par minute par unité de recherche
+ Créer ou mettre à jour un index (PUT /indexes/myindex) : 6 par seconde par unité de recherche
+ Supprimer un index (DELETE /indexes/myindex) : 12 par minute par unité de recherche 

## <a name="api-request-limits"></a>Limites de requête d’API
* 16 Mo maximum par requête <sup>1</sup>
* La longueur maximale d’une URL est de 8 Ko
* 1 000 documents maximum par lot de charges, de fusions ou de suppressions d’index
* 32 champs maximum dans la clause $orderby
* La taille maximale des termes de recherche du texte encodé en UTF-8 est de 32 766 octets (32 Ko moins 2 octets)

<sup>1</sup> Dans la Recherche cognitive Azure, le corps d’une requête est soumis à une limite supérieure de 16 Mo. Cela signifie qu’une limite pratique est imposée au contenu des champs individuels ou des collections qui ne font pas l’objet de limites théoriques (pour plus d’informations sur la composition et les restrictions des champs, consultez [Types de données pris en charge](/rest/api/searchservice/supported-data-types)).

## <a name="api-response-limits"></a>Limites de réponse d’API
* 1 000 documents maximum retournés par page de résultats de recherche
* 100 suggestions maximum retournées par requête d’API de suggestion

## <a name="api-key-limits"></a>Limites de clés API
Les clés API sont utilisées pour l’authentification de service. Il existe deux types de clé API. Les clés d’administration sont spécifiées dans l’en-tête de la demande et accordent un accès complet en lecture et en écriture au service. Les clés de requête sont en lecture seule, spécifiées dans l’URL et généralement distribuées aux applications clientes.

* 2 clés administrateur maximum par service
* 50 clés de requête maximum par service