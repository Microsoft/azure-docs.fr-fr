---
title: Récupération d’urgence pour Azure Data Share
description: Récupération d’urgence pour Azure Data Share
author: jifems
ms.author: jife
ms.service: data-share
ms.topic: how-to
ms.date: 07/30/2020
ms.openlocfilehash: 1d7c9935d7e0d6bb2d457aa4c08f9b2b3e5fa910
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92218695"
---
# <a name="disaster-recovery-for-azure-data-share"></a>Récupération d’urgence pour Azure Data Share

Cet article explique comment configurer un environnement de récupération d’urgence pour Azure Data Share. Les pannes de centres de données Azure sont rares, mais elles peuvent durer de quelques minutes à plusieurs heures. Les pannes de centre de données peuvent entraîner des perturbations dans les environnements qui reposent sur le partage des données par le fournisseur de données. En suivant les étapes décrites dans cet article, les fournisseurs de données peuvent continuer à partager des données avec les consommateurs de données en cas de panne d’un centre de données pour la région primaire hébergeant le partage de données. 

## <a name="achieving-business-continuity-for-azure-data-share"></a>Atteindre la continuité d’activité pour Azure Data Share

Pour se préparer en cas de panne d’un centre de données, le fournisseur de données peut disposer d’un environnement de partage de données provisionné dans une région secondaire. Des mesures peuvent être prises pour garantir un basculement sans heurts en cas de panne de centre de données. 

Les fournisseurs de données ont la possibilité de provisionner des ressources Azure Data Share secondaires dans une région supplémentaire. Ces ressources de partage de données peuvent être configurées pour inclure des partages et des jeux de données qui existent dans la ressource Azure Data Share principale. Elles peuvent inviter des consommateurs de données à des partages secondaires lors de la configuration de l’environnement de récupération d’urgence ou ultérieurement (par exemple, lors d’une procédure de basculement manuel).

Si les consommateurs de données disposent d’abonnements de partage actif dans un environnement secondaire approvisionné à des fins de récupération d’urgence, ils peuvent activer la planification de capture instantanée dans le cadre d’un basculement. S’ils ne veulent pas s’abonner à une région secondaire à des fins de récupération d’urgence, ils peuvent être invités ultérieurement dans le partage secondaire. 

Les consommateurs de données peuvent avoir un abonnement de partage actif inactif à des fins de récupération d’urgence. Ou bien des fournisseurs de données peuvent les inviter ultérieurement dans le cadre de procédures de basculement manuel. 

## <a name="related-information"></a>Informations connexes

- [Continuité d’activité et reprise d’activité](../best-practices-availability-paired-regions.md)
- [Créer une haute disponibilité dans une stratégie BCDR](/azure/architecture/solution-ideas/articles/build-high-availability-into-your-bcdr-strategy)

## <a name="next-steps"></a>Étapes suivantes

Pour découvrir comment commencer à partager des données, passez au tutoriel [Partager vos données](share-your-data.md).