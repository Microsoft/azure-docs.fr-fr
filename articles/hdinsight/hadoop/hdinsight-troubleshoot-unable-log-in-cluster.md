---
title: Impossible de se connecter au cluster Azure HDInsight
description: Résoudre les problèmes liés à l’impossibilité de se connecter à un cluster Apache Hadoop dans Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.date: 07/31/2019
ms.openlocfilehash: 222c48ab2b9b718bc373e710f37f906e68bd31d4
ms.sourcegitcommit: 7863fcea618b0342b7c91ae345aa099114205b03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93289017"
---
# <a name="scenario-unable-to-log-into-azure-hdinsight-cluster"></a>Scénario : Impossible de se connecter au cluster Azure HDInsight

Cet article décrit les éventuelles solutions à appliquer pour résoudre les problèmes rencontrés lors d’interactions avec des clusters Azure HDInsight.

## <a name="issue"></a>Problème

Impossible de se connecter au cluster Azure HDInsight.

## <a name="cause"></a>Cause

Les raisons peuvent varier. N’oubliez pas d’utiliser vos informations d’identification « connexion cluster » ou « HTTP » lorsque vous vous connectez aux tableaux de bord des clusters ou des applications. Lorsque vous vous connectez à distance, utilisez vos informations d’identification SSH (Secure Shell) ou du Bureau à distance.

## <a name="resolution"></a>Résolution

Pour résoudre les problèmes courants, essayez une ou plusieurs des étapes suivantes.

* Essayez d’ouvrir le tableau de bord du cluster dans un nouvel onglet du navigateur, en mode de confidentialité.

* Si vous avez oublié vos informations d’identification SSH, [réinitialisez-les dans l’interface utilisateur d’Ambari](../hdinsight-administer-use-portal-linux.md#change-passwords).

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]