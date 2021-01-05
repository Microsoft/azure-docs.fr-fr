---
title: Faire expirer des données dans Cosmos DB avec la durée de vie
description: Avec la TTL, Microsoft Azure Cosmos DB offre la possibilité de vider automatiquement les documents du système après une période déterminée.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 11/04/2020
ms.reviewer: sngun
ms.openlocfilehash: cf9d0aea9ab9e79a5f184a42e1bb785b6fb870a7
ms.sourcegitcommit: 6a902230296a78da21fbc68c365698709c579093
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93360086"
---
# <a name="time-to-live-ttl-in-azure-cosmos-db"></a>Durée de vie (TTL) dans Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Avec la **Durée de vie** (TTL, Time to Live), Azure Cosmos DB permet de supprimer automatiquement des éléments d’un conteneur après une période déterminée. La durée de vie par défaut peut être définie au niveau du conteneur et être substituée par élément. Une fois la durée de vie définie au niveau d'un conteneur ou d'un élément, Azure Cosmos DB supprime automatiquement les éléments correspondants au terme de la période écoulée depuis la dernière modification. La valeur de durée de vie est définie en secondes. Lorsque vous définissez la durée de vie, le système supprime automatiquement les éléments arrivés à expiration en fonction de la valeur de durée de vie, sans avoir besoin d’une opération de suppression explicitement émise par l’application cliente. La valeur maximale pour la durée de vie est 2147483647.

La suppression des éléments expirés est une tâche en arrière-plan qui utilise des [unités de requête](request-units.md) restantes, qui sont des unités de requête qui n’ont pas été utilisées par les demandes de l’utilisateur. Même après l’expiration de la durée de vie, si le conteneur est surchargé avec les demandes et si le nombre d’unités réservées est insuffisant, la suppression des données est retardée. Les données sont supprimées une fois que le nombre d’unités de requête disponibles est suffisant pour effectuer l’opération de suppression. Bien que la suppression des données soit différée, les données ne sont renvoyées par aucune requête (quelle que soit l’API) après l’expiration de la durée de vie.

> Ce contenu concerne la TTL des magasins transactionnels d’Azure Cosmos DB. Si vous recherchez la TTL des magasins analytiques, qui permet des scénarios de HTAP NoETL via [Azure Synapse Link](./synapse-link.md), cliquez [ici](./analytical-store-introduction.md#analytical-ttl).

## <a name="time-to-live-for-containers-and-items"></a>Durée de vie pour les conteneurs et éléments

La valeur de durée de vie est définie en secondes et interprétée en tant qu’écart par rapport à la dernière modification de l’élément. Vous pouvez définir la durée de vie sur un conteneur ou sur un élément présent dans le conteneur :

1. **Durée de vie sur un conteneur** (définie via `DefaultTimeToLive`) :

   - Si ce paramètre est manquant (ou s'il est défini sur null), les éléments n'expirent pas automatiquement.

   - Si ce paramètre est présent et que sa valeur est définie sur « -1 », il est égal à l’infini et, par défaut, les éléments n’expirent pas.

   - Si ce paramètre est présent et que sa valeur est définie sur un nombre *différent de zéro* *« n »* , les éléments expirent *« n »*  secondes après leur dernière modification.

2. **Durée de vie sur un élément** (définie via `ttl`) :

   - Cette propriété s'applique uniquement si la propriété `DefaultTimeToLive` est présente et qu'elle n'est pas définie sur null pour le conteneur parent.

   - Si elle est présente, elle remplace la valeur `DefaultTimeToLive` du conteneur parent.

## <a name="time-to-live-configurations"></a>Configurations de durée de vie

- Si une durée de vie de *« n »* est définie sur un conteneur, les éléments présents dans ce conteneur expireront après *n* secondes.  Si le même conteneur contient des éléments qui possèdent leur propre durée de vie, définie sur -1 (ce qui indique qu’ils n’expirent pas), ou si certains éléments ont remplacé le paramètre de durée de vie par un autre nombre, ces éléments expirent en fonction de la valeur de durée de vie définie.

- Si aucune durée de vie n'est définie sur un conteneur, la durée de vie définie sur un élément présent dans ce conteneur n'a aucun effet.

- Si une durée de vie de -1 est définie sur un conteneur, un élément présent dans ce conteneur et dont la durée de vie est définie sur n expirera après n secondes, et les autres éléments n'expireront pas.

## <a name="examples"></a>Exemples

Cette section présente quelques exemples avec différentes valeurs de durée de vie affectées aux conteneurs et aux éléments :

### <a name="example-1"></a>Exemple 1

La durée de vie sur le conteneur est définie par null (DefaultTimeToLive = null)

|Durée de vie sur un élément| Résultats|
|---|---|
|ttl = null|La durée de vie est désactivée. L’élément n’expire jamais (par défaut).|
|ttl = -1|La durée de vie est désactivée. L’élément n’expire jamais.|
|ttl = 2000|La durée de vie est désactivée. L’élément n’expire jamais.|

### <a name="example-2"></a>Exemple 2

La durée de vie sur le conteneur est définie par -1 (DefaultTimeToLive = -1)

|Durée de vie sur un élément| Résultats|
|---|---|
|ttl = null|La durée de vie est activée. L’élément n’expire jamais (par défaut).|
|ttl = -1|La durée de vie est activée. L’élément n’expire jamais.|
|ttl = 2000|La durée de vie est activée. L’élément expire après 2 000 secondes.|

### <a name="example-3"></a>Exemple 3

La durée de vie sur le conteneur est définie par 1000 (DefaultTimeToLive = 1000)

|Durée de vie sur un élément| Résultats|
|---|---|
|ttl = null|La durée de vie est activée. L’élément expire après 1000 secondes (par défaut).|
|ttl = -1|La durée de vie est activée. L’élément n’expire jamais.|
|ttl = 2000|La durée de vie est activée. L’élément expire après 2 000 secondes.|

## <a name="next-steps"></a>Étapes suivantes

Apprenez à configurer la durée de vie dans les articles suivants :

- [Configurer la durée de vie](how-to-time-to-live.md)