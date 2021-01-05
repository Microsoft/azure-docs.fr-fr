---
title: Optimiser les coûts des déploiements multirégions dans Azure Cosmos DB
description: Cet article explique comment gérer les coûts des déploiements multirégions dans Azure Cosmos DB.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 10/23/2020
ms.openlocfilehash: a559a51feafa310a4645282dc6368f520fc6b972
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96459611"
---
# <a name="optimize-multi-region-cost-in-azure-cosmos-db"></a>Optimiser le coût multirégion dans Azure Cosmos DB
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Vous pouvez ajouter et supprimer des régions dans votre compte Azure Cosmos à tout moment. Le débit que vous configurez pour différentes bases de données et conteneurs Azure Cosmos est réservé dans chaque région associée à votre compte. Si le débit approvisionné par heure, qui est la somme des RU/s configurées dans l’ensemble des bases de données et des conteneurs de votre compte Azure Cosmos, est égal à `T` et si le nombre de régions Azure associées à votre compte de base de données est de `N`, alors le débit total approvisionné pour votre compte Cosmos pour une heure donnée est égal à `T x N RU/s`.

Le débit approvisionné avec une seule région d’écriture coûte 0,008 $/heure pour 100 RU/seconde et le débit approvisionné avec plusieurs régions accessibles en écriture coûte 0,016 $/heure pour 100 RU/seconde. Pour en savoir plus, voir la [page de tarification](https://azure.microsoft.com/pricing/details/cosmos-db/) d’Azure Cosmos DB.

## <a name="costs-for-multiple-write-regions"></a>Coûts pour plusieurs régions d’écriture

Dans un système d’écritures multirégions, les unités de requête nettes disponibles pour les opérations d’écriture sont multipliées par `N`, où `N` correspond au nombre de régions d’écriture. Contrairement aux écritures dans une seule région, toutes les régions sont désormais accessibles en écriture et prennent en charge la résolution des conflits. Du point de vue du coût de planification, pour effectuer `M` RU/seconde d’écritures dans le monde, vous devez approvisionner M `RUs` au niveau de la base de données ou du conteneur. Vous pouvez ensuite ajouter autant de régions que vous le souhaitez et les utiliser pour les écritures pour effectuer `M` RU d’écritures dans le monde entier.

### <a name="example"></a>Exemple

Prenons un conteneur situé dans la région USA Ouest, configuré pour les écritures dans une seule région et provisionné avec un débit de 10 000 RU/s, qui stocke 0,5 To de données ce mois-ci. Vous ajoutez une région, USA Est, avec le même stockage et le même débit. Vous voulez pouvoir écrire dans les conteneurs des deux régions à partir de votre application. En supposant un mois de 730 heures, votre nouvelle facture mensuelle totale se présente ainsi :

|**Item**|**Utilisation (mensuelle)**|**Tarif**|**Coût mensuel**|
|----|----|----|----|
|Facture de débit pour le conteneur de la région USA Ouest (une seule région d’écriture) |10 000 RU/s × 730 heures |0,008 $ pour 100 RU/s par heure |584 $ |
|Facture de débit pour un conteneur dans deux régions : USA Ouest et USA Est (plusieurs régions d’écriture) |2 × 10 000 RU/s × 730 heures |0,016 $ pour 100 RU/s par heure |2 336 $ |
|Facture de stockage pour le conteneur de la région USA Ouest |0,5 To (ou 512 Go) |0,25 $/Go |128 $ |
|Facture de stockage pour le conteneur dans les deux régions (USA Ouest et USA Est) |2 × 0,5 To (ou 1 024 Go) |0,25 $/Go |256 $ |

## <a name="improve-throughput-utilization-on-a-per-region-basis"></a>Améliorer l’utilisation du débit par région

Si votre utilisation est inefficace, par exemple si une ou plusieurs régions de lecture sont sous-utilisées, vous pouvez prendre des mesures pour utiliser au maximum les RU dans les régions de lecture en utilisant le flux de modification de la région de lecture, ou déplacez-la vers une autre région secondaire en cas de surutilisation. Vous devez d’abord vous assurer d’optimiser le débit approvisionné (RU) dans la région d’écriture. Le coût des écritures est plus élevé que celui des lectures, sauf si les requêtes sont très volumineuses, de sorte qu’il peut être difficile de maintenir une utilisation uniforme. Dans l’ensemble, surveillez le débit consommé dans vos régions et ajoutez ou supprimez des régions à la demande pour mettre à l’échelle votre débit en lecture et en écriture, en vous assurant de comprendre l’impact sur la latence de toute application déployée dans la même région.

## <a name="next-steps"></a>Étapes suivantes

Pour continuer à développer vos connaissances sur l’optimisation des coûts dans Azure Cosmos DB, consultez les articles suivants :

* En savoir plus sur l’[optimisation pour le développement et le test](optimize-dev-test.md)
* En savoir plus sur les [factures Azure Cosmos DB](understand-your-bill.md)
* En savoir plus sur l’[optimisation du coût du débit](optimize-cost-throughput.md)
* En savoir plus sur l’[optimisation du coût de stockage](optimize-cost-storage.md)
* En savoir plus sur l’[optimisation du coût des lectures et écritures](optimize-cost-reads-writes.md)
* En savoir plus sur l’[optimisation du coût des requêtes](./optimize-cost-reads-writes.md)
