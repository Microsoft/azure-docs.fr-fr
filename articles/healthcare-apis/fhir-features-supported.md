---
title: Fonctionnalités de FHIR prises en charge dans Azure - API Azure pour FHIR
description: Cet article détaille les fonctionnalités de la spécification de FHIR qui sont implémentées dans l’API Azure pour FHIR
services: healthcare-apis
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: reference
ms.date: 02/07/2019
ms.author: cavoeg
ms.openlocfilehash: 9a4c331d82695aecb53990fd604ade82f3361959
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96452917"
---
# <a name="features"></a>Fonctionnalités

L’API Azure pour FHIR fournit un déploiement complètement managé du serveur FHIR Microsoft pour Azure. Le serveur est une implémentation de la [FHIR](https://hl7.org/fhir) standard. Ce document répertorie les principales fonctionnalités du serveur FHIR.

## <a name="fhir-version"></a>Version FHIR

Dernière version prise en charge : `4.0.1`

Versions antérieures également prises en charge : `3.0.2`

## <a name="rest-api"></a>API REST

| API                            | Prise en charge - PaaS | Prise en charge - OSS (SQL) | Prise en charge - OSS (Cosmos DB) | Commentaire                                             |
|--------------------------------|-----------|-----------|-----------|-----------------------------------------------------|
| lire                           | Oui       | Oui       | Oui       |                                                     |
| vread                          | Oui       | Oui       | Oui       |                                                     |
| update                         | Oui       | Oui       | Oui       |                                                     |
| update with optimistic locking | Oui       | Oui       | Oui       |                                                     |
| update (conditional)           | Oui       | Oui       | Oui       |                                                     |
| patch                          | Non        | Non        | Non        |                                                     |
| supprimer                         | Oui       | Oui       | Oui       |                                                     |
| delete (conditional)           | Non        | Non        | Non        |                                                     |
| history                        | Oui       | Oui       | Oui       |                                                     |
| create                         | Oui       | Oui       | Oui       | Prend en charge POST et PUT                               |
| create (conditional)           | Oui       | Oui       | Oui       | Problème [no 1382](https://github.com/microsoft/fhir-server/issues/1382) |
| recherche                         | Partiel   | Partiel   | Partiel   | Voir ci-dessous                                           |
| recherche chaînée                 | Non        | Oui       | Non        |                                           |
| recherche chaînée inversée         | Non        | Non        | Non        |                                            |
| capabilities                   | Oui       | Oui       | Oui       |                                                     |
| lot                          | Oui       | Oui       | Oui       |                                                     |
| transaction                    | Non        | Oui       | Non        |                                                     |
| paging                         | Partiel   | Partiel   | Partiel   | `self` et `next` sont pris en charge                     |
| intermediaries                 | Non        | Non        | Non        |                                                     |

## <a name="search"></a>Recherche

Tous les types de paramètre de recherche sont pris en charge. 

| Type de paramètre de recherche | Prise en charge - PaaS | Prise en charge - OSS (SQL) | Prise en charge - OSS (Cosmos DB) | Commentaire |
|-----------------------|-----------|-----------|-----------|---------|
| Number                | Oui       | Oui       | Oui       |         |
| Date/DateHeure         | Oui       | Oui       | Oui       |         |
| String                | Oui       | Oui       | Oui       |         |
| par jeton                 | Oui       | Oui       | Oui       |         |
| Informations de référence             | Oui       | Oui       | Oui       |         |
| Composite             | Oui       | Oui       | Oui       |         |
| Quantité              | Oui       | Oui       | Oui       |         |
| URI                   | Oui       | Oui       | Oui       |         |
| Spécial               | Non        | Non        | Non        |         |


| Modificateurs             | Prise en charge - PaaS | Prise en charge - OSS (SQL) | Prise en charge - OSS (Cosmos DB) | Commentaire |
|-----------------------|-----------|-----------|-----------|---------|
|`:missing`             | Oui       | Oui       | Oui       |         |
|`:exact`               | Oui       | Oui       | Oui       |         |
|`:contains`            | Oui       | Oui       | Oui       |         |
|`:text`                | Oui       | Oui       | Oui       |         |
|`:in` (jeton)          | Non        | Non        | Non        |         |
|`:below` (jeton)       | Non        | Non        | Non        |         |
|`:above` (jeton)       | Non        | Non        | Non        |         |
|`:not-in` (jeton)      | Non        | Non        | Non        |         |
|`:[type]` (référence)  | Non        | Non        | Non        |         |
|`:below` (URI)         | Oui       | Oui       | Oui       |         |
|`:not`                 | Non        | Non        | Non        |         |
|`:above` (URI)         | Non        | Non        | Non        | Problème [n° 158](https://github.com/Microsoft/fhir-server/issues/158) |

| Paramètre de recherche courant | Prise en charge - PaaS | Prise en charge - OSS (SQL) | Prise en charge - OSS (Cosmos DB) | Commentaire |
|-------------------------| ----------| ----------| ----------|---------|
| `_id`                   | Oui       | Oui       | Oui       |         |
| `_lastUpdated`          | Oui       | Oui       | Oui       |         |
| `_tag`                  | Oui       | Oui       | Oui       |         |
| `_profile`              | Oui       | Oui       | Oui       |         |
| `_security`             | Oui       | Oui       | Oui       |         |
| `_text`                 | Non        | Non        | Non        |         |
| `_content`              | Non        | Non        | Non        |         |
| `_list`                 | Oui       | Oui       | Oui       |         |
| `_has`                  | Non        | Non        | Non        |         |
| `_type`                 | Oui       | Oui       | Oui       |         |
| `_query`                | Non        | Non        | Non        |         |
| `_filter`               | Non        | Non        | Non        |         |

| Paramètres des résultats de la recherche | Prise en charge - PaaS | Prise en charge - OSS (SQL) | Prise en charge - OSS (Cosmos DB) | Commentaire |
|-------------------------|-----------|-----------|-----------|---------|
| `_sort`                 | Partiel        | Partiel   | Partiel        |   `_sort=_lastUpdated` est pris en charge       |
| `_count`                | Oui       | Oui       | Oui       | `_count` est limité à 100 caractères. Si la valeur est supérieure à 100, seuls 100 résultats sont retournés et un avertissement est renvoyé dans le lot. |
| `_include`              | Oui       | Oui       | Oui       |Les éléments inclus sont limités à 100. Include sur PaaS et OSS sur Cosmos DB n’inclut pas la prise en charge de l’élément :iterate.|
| `_revinclude`           | Oui       | Oui       | Oui       | Les éléments inclus sont limités à 100. Include sur PaaS et OSS sur Cosmos DB n’inclut pas la prise en charge de l’élément :iterate.|
| `_summary`              | Partiel   | Partiel   | Partiel   | `_summary=count` est pris en charge |
| `_total`                | Partiel   | Partiel   | Partiel   | _total=non et _total=accurate      |
| `_elements`             | Oui       | Oui       | Oui       |         |
| `_contained`            | Non        | Non        | Non        |         |
| `containedType`         | Non        | Non        | Non        |         |
| `_score`                | Non        | Non        | Non        |         |

## <a name="extended-operations"></a>Opérations étendues

Toutes les opérations prises en charge qui étendent l’API RESTful.

| Type de paramètre de recherche | Prise en charge - PaaS | Prise en charge - OSS (SQL) | Prise en charge - OSS (Cosmos DB) | Commentaire |
|------------------------|-----------|-----------|-----------|---------|
| $export (système entier) | Oui       | Oui       | Oui       |         |
| Patient/$export        | Oui       | Oui       | Oui       |         |
| Group/$export          | Oui       | Oui       | Oui       |         |

## <a name="persistence"></a>Persistance

Le serveur FHIR Microsoft dispose d’un module de persistance enfichable (voir [`Microsoft.Health.Fhir.Core.Features.Persistence`](https://github.com/Microsoft/fhir-server/tree/master/src/Microsoft.Health.Fhir.Core/Features/Persistence)).

Le code open source de FHIR Server inclut une implémentation pour [Azure Cosmos DB](../cosmos-db/index-overview.md) et [SQL Database](https://azure.microsoft.com/services/sql-database/).

Cosmos DB est une base de données multimodèle (SQL API, MongoDB API, etc.) distribuée à l’échelle mondiale. Il prend en charge différents [niveaux de cohérence](../cosmos-db/consistency-levels.md). Le modèle de déploiement par défaut configure le serveur FHIR avec une cohérence `Strong`, mais la stratégie de cohérence peut être modifiée (généralement assouplie) en envoyant une demande individuelle à l’aide de l’en-tête de demande `x-ms-consistency-level`.

## <a name="role-based-access-control"></a>Contrôle d’accès en fonction du rôle

Le serveur FHIR utilise [Azure Active Directory](https://azure.microsoft.com/services/active-directory/) pour contrôler l’accès. Plus précisément, le contrôle d’accès en fonction du rôle (RBAC) est appliqué, si le paramètre de configuration `FhirServer:Security:Enabled` a la valeur `true`. De plus, l’en-tête de requête `Authorization` de toutes les requêtes (à l’exception de `/metadata`) envoyées au serveur FHIR Server doit avoir la valeur `Bearer <TOKEN>`. Le jeton doit contenir au moins un rôle tel que défini dans la revendication `roles`. Une demande sera autorisée si le jeton contient un rôle qui permet l’application de l’action spécifiée sur la ressource spécifiée.

Actuellement, les actions autorisées pour un rôle donné sont appliquées *à l’ensemble* de l’API.

## <a name="service-limits"></a>Limites du service

* [**Unités de requête (RU)** ](../cosmos-db/concepts-limits.md) : vous pouvez configurer jusqu’à 10 000 unités de requête dans le portail pour l’API Azure pour FHIR. Vous aurez besoin au minimum de 400 unités de requête ou de 10 unités de requête/Go, selon la valeur la plus grande. Si vous avez besoin de plus de 10 000 unités de requête, vous pouvez envoyer un ticket de support pour augmenter ce quota. Le maximum disponible est 1 000 000.

* **Connexions simultanées** et **Instances** : vous avez, par défaut, cinq connexions simultanées sur deux instances du cluster (pour un total de 10 requêtes simultanées). Si vous pensez avoir besoin de plus de requêtes simultanées, ouvrez un ticket de support en détaillant vos besoins.

* **Taille de bundle** : chaque bundle est limité à 500 éléments.

* **Taille des données** : les éléments Données/Documents doivent chacun être un peu inférieurs à 2 Mo.

## <a name="performance-expectations"></a>Attentes en matière de niveau de performance

Les performances du système dépendent du nombre d’unités de requête, des connexions simultanées et du type d’opérations que vous exécutez (Put, Post, etc.). Vous trouverez ci-dessous quelques plages générales de ce que vous pouvez attendre en fonction des unités de requête configurées. En règle générale, les performances évoluent de façon linéaire avec l’augmentation des unités de requête :

| Nombre d’unités de requête | Ressources/s |
|----------|---------------|
| 400      | 5-10          |
| 1 000    | 100-150       |
| 10 000   | 225-400       |
| 100 000  | 2 500-4 000   |

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez obtenu des informations sur les fonctionnalités FHIR prises en charge dans l’API Azure pour FHIR. Déployez ensuite l’API Azure pour FHIR.
 
>[!div class="nextstepaction"]
>[Déployer l’API Azure pour FHIR](fhir-paas-portal-quickstart.md)