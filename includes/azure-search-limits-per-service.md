---
title: Fichier include
description: Fichier include
services: search
author: HeidiSteen
ms.service: cognitive-search
ms.topic: include
ms.date: 05/11/2020
ms.author: heidist
ms.custom: include file
ms.openlocfilehash: fe5c8129434ddb4eec2dd25a3f123f28b4db221b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83682611"
---
Un service de recherche est limité par l’espace disque ou le nombre maximum d’index ou d’indexeurs, selon la limite atteinte en premier. Le tableau suivant décrit les limites de stockage. Pour connaître le nombre maximal d’objets, examinez les [limites par ressource](../articles/search/search-limits-quotas-capacity.md#index-limits).

| Ressource | Gratuit | De base<sup>1</sup> | S1 | S2 | S3 | S3&nbsp;HD | L1 | L2 |
| -------- | --- | --- | --- | --- | --- | --- | --- | --- |
| Contrat de Niveau de Service (SLA)<sup>2</sup>  |Non |Oui |Oui |Oui |Oui |Oui |Oui |Oui |
| Stockage par partition |50 Mo |2 Go |25 Go |100 Go |200 Go |200 Go |1 To |2 To |
| Partitions par service |N/A |1 |12 |12 |12 |3 |12 |12 |
| Taille de partition |N/A |2 Go |25 Go |100 Go |200 Go |200 Go |1 To |2 To |
| Réplicas |N/A |3 |12 |12 |12 |12 |12 |12 |

<sup>1</sup> Le niveau De base comporte une partition fixe. Vous pouvez utiliser des unités de recherche supplémentaires pour ajouter des réplicas en vue d’exécuter des requêtes plus volumineuses.

<sup>2</sup> Des contrats de niveau de service (SLA) sont en vigueur pour les services facturables sur les ressources dédiées. Les services gratuits et les fonctionnalités en préversion n’en ont pas. En ce qui concerne les services facturables, les contrats SLA entrent en vigueur dès lors que vous provisionnez une redondance suffisante pour votre service. Un SLA de requête (lecture) nécessite au moins deux réplicas. Un SLA de requête et d’indexation (lecture-écriture) nécessite au moins trois réplicas. Le nombre de partitions n’est pas pris en compte dans les SLA. 