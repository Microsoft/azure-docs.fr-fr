---
title: Redémarrer des machines virtuelles pour des clusters Azure HDInsight
description: Découvrez comment redémarrer des machines virtuelles qui ne répondent pas pour des clusters Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.custom: hdinsightactive
ms.service: hdinsight
ms.topic: how-to
ms.date: 06/22/2020
ms.openlocfilehash: 725b616fec9c2bc4a0540a7941098377e01732e2
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92546464"
---
# <a name="reboot-vms-for-hdinsight-clusters"></a>Redémarrer des machines virtuelles pour des clusters HDInsight

Les clusters Azure HDInsight contiennent des groupes de machines virtuelles en tant que nœuds de cluster. Pour les clusters durables, ces nœuds peuvent cesser de répondre pour différentes raisons. Cet article explique comment redémarrer des machines virtuelles qui ne répondent pas dans un cluster HDInsight.

## <a name="when-to-reboot"></a>Quand redémarrer

> [!WARNING]
> Lorsque vous redémarrez des machines virtuelles dans un cluster, le nœud est indisponible et les services du nœud doivent redémarrer.

Pendant le redémarrage d’un nœud, le cluster peut devenir non sain et les travaux peuvent ralentir ou échouer. Si vous essayez de redémarrer le nœud principal actif, tous les travaux en cours d’exécution seront arrêtés. Vous ne serez pas en mesure de soumettre des travaux au cluster tant que les services ne seront pas à nouveau opérationnels. Pour ces raisons, vous devez redémarrer les machines virtuelles uniquement lorsque cela est nécessaire. Envisagez de redémarrer les machines virtuelles dans les cas suivants :

- Vous ne pouvez pas utiliser le protocole SSH pour accéder au nœud, mais ce dernier répond aux tests Ping.
- Le nœud Worker est en panne sans pulsation dans l’interface utilisateur Ambari.
- Le disque temporaire est plein sur le nœud.
- Le tableau de processus sur la machine virtuelle a de nombreuses entrées pour lesquelles le processus est terminé, mais qui sont répertoriées sous « État arrêté ».

> [!NOTE]
> Le redémarrage des machines virtuelles n’est pas pris en charge pour les clusters **HBase** et **Kafka** , car il peut entraîner la perte de données.

## <a name="use-powershell-to-reboot-vms"></a>Utiliser PowerShell pour redémarrer des machines virtuelles

Deux étapes sont nécessaires pour utiliser l’opération de redémarrage de nœud : répertorier les nœuds et les redémarrer.

1. Répertorier les nœuds. Vous pouvez récupérer la liste des nœuds de cluster à l’aide de la commande [Get-AzHDInsightHost](/powershell/module/az.hdinsight/get-azhdinsighthost).

      ```
      Get-AzHDInsightHost -ClusterName myclustername
      ```

1. Redémarrez les ordinateurs hôtes. Après avoir obtenu le nom des nœuds que vous souhaitez redémarrer, redémarrez-les à l’aide de la commande [Restart-AzHDInsightHost](/powershell/module/az.hdinsight/restart-azhdinsighthost).

      ```
      Restart-AzHDInsightHost -ClusterName myclustername -Name wn0-myclus, wn1-myclus
      ```

## <a name="use-a-rest-api-to-reboot-vms"></a>Utiliser une API REST pour redémarrer des machines virtuelles

Vous pouvez utiliser la fonctionnalité **Try it** dans la documentation de l’API pour envoyer des requêtes à HDInsight. Deux étapes sont nécessaires pour utiliser l’opération de redémarrage de nœud : répertorier les nœuds et les redémarrer.

1. Répertorier les nœuds. Vous pouvez récupérer la liste des nœuds de cluster à partir de l’API REST ou de Ambari. Pour plus d’informations, consultez [Opération de l’API REST Répertorier les hôtes HDInsight](/rest/api/hdinsight/virtualmachines/listhosts).

    ```
    POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.HDInsight/clusters/{clusterName}/listHosts?api-version=2018-06-01-preview
    ```

1. Redémarrez les ordinateurs hôtes. Après avoir obtenu le nom des nœuds que vous souhaitez redémarrer, utilisez l’API REST pour redémarrer les nœuds. Le nom du nœud suit le modèle *NodeType (wn/hn/zk/gw)*  + *x* + *les six premiers caractères du nom du cluster* . Pour plus d’informations, consultez [Opération de l’API REST Redémarrer des hôtes HDInsight](/rest/api/hdinsight/virtualmachines/restarthosts).

    ```
    POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.HDInsight/clusters/{clusterName}/restartHosts?api-version=2018-06-01-preview
    ```

Les noms réels des nœuds que vous souhaitez redémarrer sont spécifiés dans un tableau JSON dans le corps de la demande.

```json
[
  "wn0-abcdef",
  "zk1-abcdef"
]
```

## <a name="next-steps"></a>Étapes suivantes

* [Restart-AzHDInsightHost](/powershell/module/az.hdinsight/restart-azhdinsighthost)
* [API REST des machines virtuelles HDInsight](/rest/api/hdinsight/virtualmachines)
* [API REST de HDInsight](/rest/api/hdinsight/)