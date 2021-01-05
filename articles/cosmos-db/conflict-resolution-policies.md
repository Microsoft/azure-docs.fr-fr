---
title: Types de résolution des conflits et stratégies de résolution dans Azure Cosmos DB
description: Cet article décrit les catégories de conflits et les stratégies de résolution des conflits dans Azure Cosmos DB.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 04/20/2020
ms.author: mjbrown
ms.reviewer: sngun
ms.openlocfilehash: ba55d88de3a5a4087db30613b22a7d2441de9be1
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93334376"
---
# <a name="conflict-types-and-resolution-policies-when-using-multiple-write-regions"></a>Types de conflits et stratégies de résolution lors de l’utilisation de plusieurs régions d’écriture
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Les conflits et les stratégies de résolution de conflits sont applicables si votre compte Azure Cosmos DB est configuré avec plusieurs régions d’écriture.

Pour les comptes Azure Cosmos configurés avec plusieurs régions d’écriture, des conflits de mise à jour peuvent se produire lorsque des rédacteurs mettent à jour simultanément le même élément dans plusieurs régions. Les conflits de mise à jour peuvent être des trois types suivants :

* **Conflits d’insertion** : ces conflits peuvent se produire quand une application insère simultanément deux éléments, ou plus, avec le même index unique dans de deux régions, ou plus. Par exemple, ce conflit peut apparaître avec une propriété ID.

* **Conflits de remplacement** : ces conflits peuvent se produire quand une application met à jour un même élément dans deux régions, ou plus, simultanément.

* **Conflits de suppression** : ces conflits peuvent se produire quand une application supprime un élément dans une région et le met à jour dans une autre région, simultanément.

## <a name="conflict-resolution-policies"></a>Stratégies de résolution des conflits

Azure Cosmos DB offre un mécanisme souple, piloté par les stratégies, pour résoudre les conflits d’écriture. Vous pouvez choisir entre deux stratégies de résolution de conflits sur un conteneur Azure Cosmos :

* **Dernière écriture prioritaire (LWW)** : cette stratégie de résolution, par défaut, utilise une propriété d’horodatage définie par le système. Elle est basée sur le protocole d’horloge de synchronisation de l’heure. Si vous utilisez l’API SQL, vous pouvez spécifier toute autre propriété numérique personnalisée (par exemple, votre propre notion de timestamp) à utiliser pour la résolution de conflits. Une propriété numérique personnalisée est également appelée *chemin de résolution du conflit*. 

  Si deux ou plusieurs éléments sont en conflit lors d’opérations d’insertion ou de remplacement, l’élément ayant la valeur la plus élevée pour le chemin de résolution du conflit l’emporte. Le système détermine le gagnant si plusieurs éléments présentent la même valeur numérique pour le chemin de résolution du conflit. Toutes les régions convergeront vers un seul gagnant et obtiennent une version identique de l’élément validé. Lorsque des conflits de suppression sont impliqués, la version supprimée l’emporte toujours sur les conflits d’insertion ou de remplacement. Ce résultat se produit quelle que soit la valeur du chemin de résolution du conflit.

  > [!NOTE]
  > La dernière écriture prévaut est la stratégie de résolution de conflit par défaut et utilise l'horodatage `_ts` pour les API suivantes : SQL, MongoDB, Cassandra, Gremlin et Table. La propriété numérique personnalisée est disponible uniquement pour l’API SQL.

  Pour plus d’informations, consultez les [exemples qui utilisent des stratégies de résolution de conflit LWW](how-to-manage-conflicts.md).

* **Personnalisé**  : cette stratégie de résolution est destinée à la sémantique définie au niveau de l’application pour le rapprochement des conflits. Lorsque vous définissez cette stratégie sur votre conteneur Azure Cosmos, vous devez également inscrire une *procédure stockée de fusion*. Cette procédure est automatiquement appelée lorsque des conflits sont détectés sous une transaction de base de données côté serveur. Le système fournit exactement une garantie pour l’exécution d’une procédure de fusion dans le cadre du protocole d’engagement.  

  Si vous configurez votre conteneur avec l’option de résolution personnalisée mais ne pouvez pas inscrire une procédure de fusion sur le conteneur, ou si la procédure de fusion lève une exception lors de l’exécution, les conflits sont écrits dans le *flux de conflits*. Les conflits de l’application doivent ensuite être résolus manuellement dans le flux de conflits. Pour plus d’informations, consultez les [exemples illustrant la façon d’utiliser la stratégie de résolution personnalisée, et comment utiliser le flux de conflits](how-to-manage-conflicts.md).

  > [!NOTE]
  > La stratégie personnalisée de résolution de conflits est disponible uniquement pour les comptes d’API SQL.

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment configurer des stratégies de résolution de conflits :

* [Guide pratique de configuration de plusieurs régions d’écriture pour vos applications](how-to-multi-master.md)
* [How to manage conflict resolution policies (Comment gérer des stratégies de résolution de conflits)](how-to-manage-conflicts.md)
* [How to read from the conflicts feed (Comment lire à partir du flux de conflits)](how-to-manage-conflicts.md#read-from-conflict-feed)
