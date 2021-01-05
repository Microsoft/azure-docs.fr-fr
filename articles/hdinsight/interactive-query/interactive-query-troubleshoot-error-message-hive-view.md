---
title: Message d’erreur non affiché en mode Apache Hive – Azure HDInsight
description: La requête échoue dans la vue Apache Hive sans explications sur le cluster Azure HDInsight.
ms.service: hdinsight
ms.topic: troubleshooting
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.date: 07/30/2019
ms.openlocfilehash: 7aadef0d4c70f748b4f7a7c7f9fc16c38216fa7a
ms.sourcegitcommit: 7863fcea618b0342b7c91ae345aa099114205b03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93288943"
---
# <a name="scenario-query-error-message-not-displayed-in-apache-hive-view-in-azure-hdinsight"></a>Scénario : Message d’erreur de requête non affiché dans l’affichage Apache Hive dans Azure HDInsight

Cet article décrit la procédure à suivre pour résoudre les problèmes rencontrés lors de l’utilisation de composants Interactive Query dans des clusters Azure HDInsight.

## <a name="issue"></a>Problème

Le message d’erreur de la requête d’affichage de Apache Hive ressemble à ce qui suit, sans informations supplémentaires :

```
"Failed to execute query. <a href="#/messages/1">(details)</a>"
```

## <a name="cause"></a>Cause

Parfois, le message d’erreur d’un échec de requête peut être trop long pour être affiché sur la page principale de l’affichage Hive.

## <a name="resolution"></a>Résolution

Vérifiez l’onglet Notifications dans l’angle supérieur droit de Hive_view pour afficher le message d’erreur et le StackTrace complet.

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]