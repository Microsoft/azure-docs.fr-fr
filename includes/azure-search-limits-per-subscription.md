---
title: Fichier Include
description: Fichier Include
author: HeidiSteen
ms.service: cognitive-search
ms.topic: include
ms.date: 05/06/2019
ms.author: heidist
ms.custom: include file
ms.openlocfilehash: ee430241173a6c19e2a32e176f28411631d9cb19
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "80272664"
---
Vous pouvez créer plusieurs services au sein d’un abonnement. Chacun peut être approvisionné à un niveau spécifique. Vous êtes uniquement limité par le nombre de services autorisé à chaque niveau. Ainsi, vous pouvez créer jusqu’à 12 services au niveau de base et 12 autres au niveau S1 au sein du même abonnement. Pour en savoir plus sur ces niveaux, consultez [Choisir un niveau tarifaire pour Recherche cognitive Azure](../articles/search/search-sku-tier.md).

Les limites de service maximales peuvent être augmentées sur demande. S’il vous faut davantage de services dans le même abonnement, contactez le Support Azure.

| Ressource            | Gratuit<sup>1</sup> | De base | S1  | S2 | S3 | S3&nbsp;HD | L1 | L2 |
| ------------------- | ---- | ----- | --- | -- | -- | ----- | -- | -- |
| Nombre de services maximum    |1     | 16    | 16  | 8  | 6  | 6     | 6  | 6  |
| Nombre maximal d’unités de recherche scale-in (SU)<sup>2</sup> |N/A |3 unités de recherche |36 unités de recherche |36 unités de recherche |36 unités de recherche |36 unités de recherche |36 unités de recherche |36 unités de recherche |

<sup>1</sup> Le niveau Gratuit est basé sur des ressources partagées, non des ressources dédiées. Le « scale up » n’est pas pris en charge sur les ressources partagées.

<sup>2</sup> Les unités de recherche sont des unités de facturation, allouées en tant que *réplicas* ou *partitions*. Vous avez besoin des deux types de ressource pour les opérations de stockage, d’indexation et d’interrogation. Pour en savoir plus sur les calculs SU, consultez [Mettre à l’échelle les niveaux de ressources pour les charges de travail de requête et d’index](../articles/search/search-capacity-planning.md). 