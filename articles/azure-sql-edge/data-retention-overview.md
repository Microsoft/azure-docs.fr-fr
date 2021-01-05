---
title: Vue d’ensemble de la stratégie de conservation des données - Azure SQL Edge
description: Découvrez la stratégie de conservation des données dans Azure SQL Edge
keywords: SQL Edge, conservation des données
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 09/04/2020
ms.openlocfilehash: bb059a946c03f41e5b65944eec67070f84ee6b08
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90976333"
---
# <a name="data-retention-overview"></a>Vue d’ensemble de la conservation des données

La collecte et le stockage de données à partir d’appareils IoT connectés sont des tâches importantes pour obtenir et exploiter des insights opérationnels et commerciaux. Toutefois, en raison du gros volume de données provenant de ces appareils, il devient essentiel pour les organisations de déterminer soigneusement quelle quantité de données conserver, et à quel niveau de précision. L’idéal serait de conserver toutes les données à tous les niveaux de précision, mais cela ne serait pas toujours réalisable. Par ailleurs, le volume de données pouvant être conservées est restreint par la quantité de stockage disponible sur les appareils IoT ou Edge. 

Dans Azure SQL Edge, les administrateurs de base de données peuvent définir la stratégie de conservation des données à appliquer à une base de données SQL Edge et à ses tables sous-jacentes. Une fois la stratégie de conservation des données définie, une tâche système en arrière-plan est exécutée pour vider les données obsolètes (anciennes) des tables utilisateur. 

> [!Note]
> Les données vidées de la table ne sont pas récupérables. La seule façon possible de récupérer les données vidées est de restaurer la base de données à partir d’une sauvegarde antérieure.

Guides de démarrage rapide :

- [Activer et désactiver les stratégies de conservation des données](data-retention-enable-disable.md)
- [Gérer les données historiques avec la stratégie de conservation](data-retention-cleanup.md)

## <a name="how-data-retention-works"></a>Fonctionnement de la conservation des données

Pour configurer la conservation des données, vous pouvez utiliser des instructions DDL. Pour plus d’informations, consultez [Activer et désactiver les stratégies de conservation des données](data-retention-enable-disable.md). Pour activer la suppression automatique des enregistrements obsolètes, la conservation des données doit préalablement être activée pour la base de données, mais aussi pour les tables à vider de cette base de données. 

Une fois la conservation des données configurée pour une table, une tâche en arrière-plan est exécutée pour identifier et supprimer les enregistrements obsolètes dans une table. Si, pour une raison quelconque, le nettoyage automatique des tâches ne s’effectue pas ou n’est pas en mesure de gérer les suppressions, une opération de nettoyage manuel est possible sur ces tables. Pour plus d’informations sur les nettoyages automatiques et manuels, consultez [Nettoyage automatique et manuel](data-retention-cleanup.md).

## <a name="limitations-and-restrictions"></a>Limitations et restrictions

- La conservation des données, si elle est activée, est automatiquement désactivée quand la base de données est restaurée à partir d’une sauvegarde complète, ou qu’elle est rattachée. 
- La conservation des données ne peut pas être activée pour une table d’historique temporelle.
- La colonne de filtre de la conservation des données ne peut pas être modifiée. Pour modifier la colonne, désactivez la conservation des données sur la table.  

## <a name="next-steps"></a>Étapes suivantes

- [Machine Learning et intelligence artificielle avec ONNX dans SQL Edge](onnx-overview.md).
- [Création d’une solution IoT de bout en bout avec SQL Edge à l’aide d’IoT Edge](tutorial-deploy-azure-resources.md).
- [Streaming de données dans Azure SQL Edge](stream-data.md)
