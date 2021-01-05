---
title: Impossible d’ajouter des nœuds au cluster Azure HDInsight
description: Résoudre les problèmes liés à l’impossibilité d’ajouter des nœuds à un cluster Apache Hadoop dans Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.date: 07/31/2019
ms.openlocfilehash: 089eab563c83384bdb7e1681044fa89f5eb7f6c0
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96014667"
---
# <a name="scenario-unable-to-add-nodes-to-azure-hdinsight-cluster"></a>Scénario : Impossible d’ajouter des nœuds au cluster Azure HDInsight

Cet article décrit les éventuelles solutions à appliquer pour résoudre les problèmes rencontrés lors d’interactions avec des clusters Azure HDInsight.

## <a name="issue"></a>Problème

Impossible d’ajouter des nœuds au cluster Azure HDInsight.

## <a name="cause"></a>Cause

Les raisons peuvent varier.

## <a name="resolution"></a>Résolution

À l’aide de la fonctionnalité [Taille du cluster](../hdinsight-scaling-best-practices.md), calculez le nombre de cœurs supplémentaires nécessaires au cluster. Ce nombre dépend du nombre total de cœurs dans les nouveaux nœuds Worker. Essayez ensuite une ou plusieurs des étapes suivantes :

* Vérifiez si des cœurs sont disponibles à l’emplacement du cluster.

* Examinez le nombre de cœurs disponibles à d’autres emplacements. Envisagez éventuellement de recréer votre cluster à un autre emplacement et de lui attribuer suffisamment de cœurs.

* Si vous souhaitez augmenter le quota principal d’un emplacement précis, créez un ticket de support en demandant une augmentation du quota principal HDInsight.

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]