---
title: Résoudre les problèmes de création de ressources dans Azure HDInsight
description: Cet article aborde les erreurs courantes liées à la capacité, ainsi que les techniques d’atténuation adaptées.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: troubleshooting
ms.custom: seoapr2020
ms.date: 04/22/2020
ms.openlocfilehash: 2e9f6898dba559f557f49cc0ee9ff39644da243f
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96000689"
---
# <a name="troubleshoot-resource-creation-failures-in-azure-hdinsight"></a>Résoudre les problèmes de création de ressources dans Azure HDInsight

Cet article décrit les éventuelles solutions à appliquer pour résoudre les problèmes rencontrés lors d’interactions avec Azure HDInsight.

## <a name="error-the-deployment-would-exceed-the-quota-of-800"></a>Erreur : Le déploiement va entraîner le dépassement du quota de « 800 »

Azure a une limite de quota de 800 déploiements par groupe de ressources. Des quotas sont appliqués pour chaque groupe de ressources, abonnement, compte ou autre étendue. Par exemple, votre abonnement peut être configuré pour limiter le nombre de cœurs dans une région. Si vous tentez de déployer une machine virtuelle avec plus de cœurs que le nombre autorisé, le message d’erreur indique que le quota a été dépassé.

Pour résoudre ce problème, supprimez les déploiements qui ne sont plus nécessaires à l’aide du portail Azure, de CLI ou de PowerShell.

Pour plus d’informations, consultez [Résoudre les erreurs liées aux quotas de ressources](../azure-resource-manager/templates/error-resource-quota.md).

## <a name="error-the-maximum-node-exceeded-the-available-cores-in-this-region"></a>Erreur : Le nombre maximal de nœuds dépasse le nombre de cœurs disponibles dans cette région

Votre abonnement peut être configuré pour limiter le nombre de cœurs dans une région. Si vous tentez de déployer une ressource avec plus de cœurs que le nombre autorisé, le message d’erreur indique que le quota a été dépassé.

Pour demander une augmentation de quota, procédez comme suit :

1. Accédez au [portail Azure](https://portal.azure.com), puis sélectionnez **Aide + Support**.

1. Sélectionnez **Nouvelle demande de support**.

1. Sous l’onglet **Informations de base** dans la page **Nouvelle demande de support**, fournissez les informations suivantes :

   * **Type de problème :** Sélectionnez **Limites du service et de l’abonnement (quotas)** .
   * **Abonnement :** Sélectionnez l’abonnement que vous souhaitez modifier.
   * **Type de quota** : Sélectionnez **HDInsight**.

Pour plus d’informations, consultez [Créer un ticket de support pour augmenter le nombre de cœurs](hdinsight-capacity-planning.md#quotas).

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [troubleshooting next steps](../../includes/hdinsight-troubleshooting-next-steps.md)]