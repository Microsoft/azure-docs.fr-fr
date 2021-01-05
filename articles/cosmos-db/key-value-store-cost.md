---
title: Frais associés aux unités de demande pour Azure Cosmos DB en tant que magasin de valeurs de clés
description: Découvrez les frais des unités de demande pour Azure Cosmos DB pour de simples opérations de lecture et d’écriture lorsqu’il est utilisé comme magasin de valeurs de clés.
author: SnehaGunda
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 08/23/2019
ms.author: sngun
ms.custom: seodec18
ms.openlocfilehash: 9354ae0a22ef2e8ab4ee6a57563d3f3c4c8e4547
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93339298"
---
# <a name="azure-cosmos-db-as-a-key-value-store--cost-overview"></a>Azure Cosmos DB comme magasin de valeurs de clés – Synthèse des coûts
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Azure Cosmos DB est un service de base de données multimodèle, distribué dans le monde entier, qui permet de créer facilement des applications à grande échelle et à haute disponibilité. Par défaut, Azure Cosmos DB indexe automatiquement et efficacement toutes les données qu’il reçoit. Vous pouvez ainsi créer des requêtes [SQL](./sql-query-getting-started.md) (et [JavaScript](stored-procedures-triggers-udfs.md)) rapides et cohérentes sur les données. 

Cet article décrit le coût qu’implique l’utilisation d’Azure Cosmos DB pour de simples opérations de lecture et d’écriture lorsqu’il est utilisé comme magasin de valeurs de clés. Les opérations d’écriture incluent des insertions, des remplacements, des suppressions et des upserts d’éléments de données. En plus d’un contrat SLA de disponibilité de 99,999 % pour tous les comptes multirégions, Azure Cosmos DB garantit une latence inférieure à 10 ms pour les opérations de lecture et d’écriture (indexées), au 99e centile. 

## <a name="why-we-use-request-units-rus"></a>Pourquoi utiliser des unités de requête (RU) ?

Les performances d’Azure Cosmos DB sont basées sur le volume de débit provisionné exprimé en [unités de requête](request-units.md) (RU/s). Le provisionnement se fait avec une précision d’une seconde et s’achète en unité de requête par seconde ([à ne pas confondre avec la facturation horaire](https://azure.microsoft.com/pricing/details/cosmos-db/)). Les unités de requête doivent être considérées comme une abstraction logique (une devise) qui simplifie le provisionnement du débit dont a besoin l’application. Les utilisateurs n’ont donc plus à différencier le débit de lecture et d’écriture. Le modèle de devise unique des unités de requête permet de partager efficacement la capacité approvisionnée entre les opérations de lecture et d’écriture. Ce modèle de capacité provisionnée permet au service de fournir un **débit prévisible et constant, une faible latence garantie et une haute disponibilité**. Enfin, alors que le modèle des unités de requête permet de décrire le débit, chaque unité de requête provisionnée a également une quantité définie de ressources (mémoire, cœurs/processeur et E/S par seconde).

En tant que système de base de données distribué dans le monde entier, Cosmos DB est le seul service Azure à fournir des contrats SLA complets sur la latence, le débit, la cohérence et la haute disponibilité. Le débit que vous provisionnez est appliqué à chacune des régions associées à votre compte Cosmos. Pour les opérations de lecture, Azure Cosmos DB propose plusieurs [niveaux de cohérence](consistency-levels.md) bien définis. 

Le tableau suivant indique le nombre d’unités de requête (RU) nécessaires pour effectuer des opérations de lecture et d’écriture en fonction d’un élément de données d’une taille de 1 Ko et de 100 Ko avec l’indexation automatique par défaut désactivée. 

|Taille de l’élément|1 lecture|1 écriture|
|-------------|------|-------|
|1 Ko|1 unité de requête|5 unités de requête|
|100 Ko|10 unités de requête|50 unités de requête|

## <a name="cost-of-reads-and-writes"></a>Coût des lectures et des écritures

Si vous provisionnez 1 000 unités de requête par seconde, vous obtenez un total de 3,6 millions d’unités de requête par heure, à un coût de 0,08 dollar l’heure (aux États-Unis et en Europe). Pour un élément de données d’une taille de 1 Ko, vous pouvez donc consommer 3,6 millions de lectures ou 0,72 million d’écritures (3,6 millions d’unités de requête/5) en utilisant le débit provisionné. Normalisé au million de lectures et d’écritures, le coût serait de 0,022 $/million de lectures (0,08 $/3,6) et de 0,111 $/million d’écritures (0,08 $/0,72). Le coût par million devient minime, comme le montre le tableau ci-dessous.

|Taille de l’élément|Coût de 1 million de lectures|Coût de 1 million d’écritures|
|-------------|-------|--------|
|1 Ko|0,022 $|0,111 $|
|100 Ko|0,222 $|1,111 $|


La plupart des magasins d’objets et d’objets Blob de base facturent 0,40 $ le million de transactions de lecture et 5 $ le million de transactions d’écriture. S’il est utilisé de manière optimale, Cosmos DB peut être jusqu’à 98 % moins cher que les autres solutions (pour les transactions de 1 Ko).

## <a name="next-steps"></a>Étapes suivantes

* Utilisez la [calculatrice d’unités de requête](https://cosmos.azure.com/capacitycalculator/) pour estimer le débit de vos charges de travail.