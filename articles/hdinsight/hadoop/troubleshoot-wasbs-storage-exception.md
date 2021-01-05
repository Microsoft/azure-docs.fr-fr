---
title: Le compte auquel vous accédez ne prend pas en charge les erreurs HTTP dans Azure HDInsight
description: Cet article décrit les éventuelles solutions à appliquer pour résoudre les problèmes rencontrés lors d’interactions avec des clusters Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 02/06/2020
ms.openlocfilehash: b7f3a3b76169b99389fe8222177ddcb713c27713
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92546583"
---
# <a name="the-account-being-accessed-does-not-support-http-error-in-azure-hdinsight"></a>Le compte auquel vous accédez ne prend pas en charge les erreurs HTTP dans Azure HDInsight

Cet article décrit les éventuelles solutions à appliquer pour résoudre les problèmes rencontrés lors d’interactions avec des clusters Azure HDInsight.

## <a name="issue"></a>Problème

Le message d’erreur suivant est reçu :

```
com.microsoft.azure.storage.StorageException: The account being accessed does not support http.
```

## <a name="cause"></a>Cause :

Le message d’erreur est reçu pour plusieurs raisons :

* Le [transfert sécurisé](../../storage/common/storage-require-secure-transfer.md) du compte de stockage est activé et un [schéma d’URI](../hdinsight-hadoop-linux-information.md#URI-and-scheme) incorrect est utilisé.

* Un cluster a été créé avec un compte de stockage dont le transfert sécurisé est *désactivé* . Par la suite, le transfert sécurisé a été activé sur le compte de stockage.

## <a name="resolution"></a>Résolution

Si l’option de transfert sécurisé était activée pour Stockage Azure ou Data Lake Storage Gen2, l’URI serait `wasbs://` ou `abfss://`, respectivement.  Voir aussi [transfert sécurisé](../../storage/common/storage-require-secure-transfer.md).

Pour les nouveaux clusters, utilisez un compte de stockage qui possède déjà le paramètre de transfert sécurisé souhaité. Ne modifiez pas le paramètre de transfert sécurisé d’un compte de stockage qui est utilisé par un cluster existant.

## <a name="next-steps"></a>Étapes suivantes

Si votre problème ne figure pas dans cet article ou si vous ne parvenez pas à le résoudre, utilisez un des canaux suivants pour obtenir de l’aide :

* Obtenez des réponses de la part d’experts Azure en faisant appel au [Support de la communauté Azure](https://azure.microsoft.com/support/community/).

* Connectez-vous à [@AzureSupport](https://twitter.com/azuresupport), le compte Microsoft Azure officiel pour améliorer l’expérience client. Connexion de la communauté Azure aux ressources appropriées : réponses, support technique et experts.

* Si vous avez besoin d’une aide supplémentaire, vous pouvez envoyer une requête de support à partir du [Portail Microsoft Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade/). Sélectionnez **Support** dans la barre de menus, ou ouvrez le hub **Aide + Support** . Pour plus d’informations, consultez [Création d’une demande de support Azure](../../azure-portal/supportability/how-to-create-azure-support-request.md). L’accès au support relatif à la gestion et à la facturation des abonnements est inclus avec votre abonnement Microsoft Azure. En outre, le support technique est fourni avec l’un des [plans de support Azure](https://azure.microsoft.com/support/plans/).