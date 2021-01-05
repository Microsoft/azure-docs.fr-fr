---
title: Mettre automatiquement à l’échelle des instances Apache Spark
description: Utilisez la fonctionnalité de mise à l’échelle automatique d’Azure Synapse pour mettre automatiquement à l’échelle des instances Apache Spark
author: euangMS
ms.author: euang
ms.reviewer: euang
services: synapse-analytics
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: spark
ms.date: 03/31/2020
ms.openlocfilehash: f34bcfa8b743fbee6ee3b78fc1a042d1df0abfde
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93313631"
---
# <a name="automatically-scale-azure-synapse-analytics-apache-spark-pools"></a>Mettre automatiquement à l’échelle des pools Apache Spark d’Azure Synapse Analytics

Apache Spark pour la fonctionnalité de mise à l’échelle automatique du pool Azure Synapse Analytics met automatiquement à l’échelle le nombre de nœuds dans une instance de cluster. Lors de la création d’un Apache Spark pour un pool Azure Synapse Analytics, des nombres minimal et maximal de nœuds peuvent être définis lorsque la mise à l’échelle automatique est sélectionnée. La mise à l’échelle automatique surveille ensuite les besoins en ressources de la charge, et met à l’échelle le nombre de nœuds. L’utilisation de cette fonctionnalité n’entraîne aucun coût supplémentaire.

## <a name="metrics-monitoring"></a>Supervision des métriques

La mise à l’échelle automatique surveille en permanence l’instance Spark et collecte les métriques suivantes :

|Métrique|Description|
|---|---|
|Total Pending CPU|Nombre total de cœurs requis pour commencer l’exécution de tous les nœuds en attente.|
|Total Pending Memory|Mémoire totale (en Mo) nécessaire pour commencer l’exécution de tous les nœuds en attente.|
|Total Free CPU|Somme de tous les cœurs inutilisés sur les nœuds actifs.|
|Total Free Memory|Somme de la mémoire inutilisée (en Mo) sur les nœuds actifs.|
|Used Memory per Node|Charge sur un nœud. Un nœud sur lequel 10 Go de mémoire sont utilisés est considéré comme étant plus sollicité qu’un nœud avec 2 Go de mémoire utilisés.|

Les métriques ci-dessus sont contrôlées toutes les 30 secondes. La fonction de mise à l’échelle automatique prend des décisions de montée en puissance ou de descente en puissance en fonction de ces métriques.

## <a name="load-based-scale-conditions"></a>Conditions de mise à l’échelle basée sur la charge

Lorsque les conditions suivantes sont détectées, la mise à l’échelle automatique émet une requête de mise à l’échelle :

|Scale-up|Scale-down|
|---|---|
|« Total pending CPU » est supérieur à la valeur de « Total free CPU » pendant plus de 1 minute.|« Total pending CPU » est inférieur à la valeur de « Total free CPU » pendant plus de 2 minutes.|
|« Total pending memory » est supérieur à la valeur de « Total free memory » pendant plus de 1 minute.|« Total pending memory » est inférieur à la valeur de « Total free memory » pendant plus de 2 minutes.|

Pour un scale-up, le service Azure Synapse Analytics calcule combien de nouveaux nœuds sont utiles pour répondre aux besoins actuels en UC et en mémoire, puis il émet une requête de scale-up pour ajouter le nombre de nœuds nécessaires.

Pour un scale-down, selon le nombre d’exécuteurs, de maîtres d’application par nœud et les besoins actuels en UC et en mémoire, la fonction de mise à l’échelle automatique émet une requête de suppression d’un certain nombre de nœuds. Le service détecte également les nœuds à supprimer en fonction de l’exécution des travaux en cours. L’opération de descente en puissance désactive tout d’abord les nœuds, puis les supprime du cluster.

## <a name="get-started"></a>Bien démarrer

### <a name="create-a-serverless-apache-spark-pool-with-autoscaling"></a>Créer un pool Apache Spark serverless avec mise à l’échelle automatique

Pour activer la fonctionnalité de mise à l’échelle automatique, procédez comme suit dans le cadre du processus normal de création de pool :

1. Sous l’onglet **De base** , cochez la case **Activer la mise à l’échelle automatique**.
1. Entrez les valeurs souhaitées pour les propriétés suivantes :  

    * Nombre **Minimum** de nœuds.
    * Nombre **Maximum** de nœuds.

Le nombre initial de nœuds est le nombre minimal. Cette valeur définit la taille initiale de l’instance lors de sa création. Le nombre minimal de nœuds ne peut pas être inférieur à trois.

## <a name="best-practices"></a>Meilleures pratiques

### <a name="consider-the-latency-of-scale-up-or-scale-down-operations"></a>Prendre en compte la latence de la montée ou de la descente en puissance

Une opération de mise à l’échelle peut prendre entre 1 et 5 minutes.

### <a name="prepare-for-scaling-down"></a>Préparation pour la mise  à l’échelle vers le bas

Pendant la mise à l’échelle vers le bas des instances, la mise à l’échelle automatique place les nœuds dans un état de désaffectation afin qu’aucun nouvel exécuteur ne puisse être lancé dessus.

Les travaux en cours d’exécution continueront jusqu’à leur conclusion. Les travaux en attente attendrons d’être planifiée comme d’habitude avec moins de nœuds disponibles.

## <a name="next-steps"></a>Étapes suivantes

Démarrage rapide pour configurer un nouveau pool Spark [Créer un pool Spark](../quickstart-create-apache-spark-pool-portal.md)
