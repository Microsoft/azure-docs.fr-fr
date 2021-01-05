---
title: Utiliser Apache Hive avec PowerShell dans HDInsight - Azure
description: Utiliser PowerShell pour exécuter des requêtes Apache Hive dans Apache Hadoop sur Azure HDInsight
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 12/24/2019
ms.openlocfilehash: 013eb8215064173e002aea23043bc2e9ba92b525
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92545240"
---
# <a name="run-apache-hive-queries-using-powershell"></a>Exécuter des requêtes Apache Hive avec PowerShell

[!INCLUDE [hive-selector](../../../includes/hdinsight-selector-use-hive.md)]

Ce document fournit un exemple d’utilisation de Azure PowerShell pour exécuter des requêtes Apache Hive dans un Apache Hadoop sur un cluster HDInsight.

> [!NOTE]  
> Ce document ne fournit pas de description détaillée de ce que font les instructions HiveQL utilisées dans les exemples. Pour plus d’informations sur le langage HiveQL utilisé dans cet exemple, consultez [Utiliser Apache Hive avec Apache Hadoop sur HDInsight](hdinsight-use-hive.md).

## <a name="prerequisites"></a>Conditions préalables requises

* Un cluster Apache Hadoop sur HDInsight. Consultez [Bien démarrer avec HDInsight sur Linux](./apache-hadoop-linux-tutorial-get-started.md).

* Le [module Az](/powershell/azure/) PowerShell installé.

## <a name="run-a-hive-query"></a>Exécution d'une tâche Hive

Azure PowerShell fournit des *cmdlets* qui vous permettent d'exécuter à distance des requêtes Hive sur HDInsight. En interne, les applets de commande effectuent des appels REST à [WebHCat](https://cwiki.apache.org/confluence/display/Hive/WebHCat) sur le cluster HDInsight.

Les applets de commande suivants sont utilisés lors de l'exécution de requêtes Hive sur un cluster à distance HDInsight :

* `Connect-AzAccount` : authentifie Azure PowerShell dans votre abonnement Azure.
* `New-AzHDInsightHiveJobDefinition` : crée une *définition de tâche* à l’aide des instructions HiveQL spécifiées.
* `Start-AzHDInsightJob` : envoie la définition de la tâche à HDInsight et démarre la tâche. Un objet *job* est retourné.
* `Wait-AzHDInsightJob` : utilise l’objet de la tâche pour vérifier l’état de la tâche. Il attend que la tâche soit terminée ou que le délai d’attente soit dépassé.
* `Get-AzHDInsightJobOutput` : utilisé pour récupérer la sortie de la tâche.
* `Invoke-AzHDInsightHiveJob` : utilisé pour exécuter les instructions HiveQL. Cette applet de commande bloque la fin de la requête, puis retourne les résultats.
* `Use-AzHDInsightCluster` : définit le cluster actuel à utiliser pour la commande `Invoke-AzHDInsightHiveJob`.

Les étapes suivantes montrent comment utiliser ces cmdlets pour exécuter une tâche sur votre cluster HDInsight :

1. À l’aide d’un éditeur, enregistrez le code suivant sous `hivejob.ps1`.

    [!code-powershell[main](../../../powershell_scripts/hdinsight/use-hive/use-hive.ps1?range=5-42)]

2. Ouvrez une invite de commandes **Azure PowerShell** . Accédez au répertoire du fichier `hivejob.ps1`, puis utilisez la commande suivante pour exécuter le script :

    ```azurepowershell
    .\hivejob.ps1
    ```

    Quand le script s’exécute, vous devez entrer le nom du cluster et les informations d’identification du compte HTTPS/Admin. Vous pouvez également être invité à vous connecter à votre abonnement Azure.

3. Lorsque la tâche est terminée, des informations similaires au texte suivant s’affichent :

    ```output
    Display the standard output...
    2012-02-03      18:35:34        SampleClass0    [ERROR] incorrect       id
    2012-02-03      18:55:54        SampleClass1    [ERROR] incorrect       id
    2012-02-03      19:25:27        SampleClass4    [ERROR] incorrect       id
    ```

4. Comme mentionné précédemment, `Invoke-Hive` peut être utilisé pour exécuter une requête et attendre la réponse. Pour voir comment fonctionne Invoke-Hive, utilisez le script suivant :

    [!code-powershell[main](../../../powershell_scripts/hdinsight/use-hive/use-hive.ps1?range=50-71)]

    La sortie ressemble au texte suivant :

    ```output
    2012-02-03    18:35:34    SampleClass0    [ERROR]    incorrect    id
    2012-02-03    18:55:54    SampleClass1    [ERROR]    incorrect    id
    2012-02-03    19:25:27    SampleClass4    [ERROR]    incorrect    id
    ```

   > [!NOTE]  
   > Pour les requêtes HiveQL plus longues, vous pouvez utiliser les fichiers de script HiveQL de PowerShell ou la cmdlet **Here-Strings** Azure PowerShell. L’extrait suivant montre comment utiliser la cmdlet `Invoke-Hive` pour exécuter un fichier de script HiveQL. Ce dernier doit être téléchargé vers wasbs://.
   >
   > `Invoke-AzHDInsightHiveJob -File "wasbs://<ContainerName>@<StorageAccountName>/<Path>/query.hql"`
   >
   > Pour plus d’informations sur les chaînes **Here-Strings** , consultez [HERE-STRINGS](/powershell/module/microsoft.powershell.core/about/about_quoting_rules#here-strings).

## <a name="troubleshooting"></a>Dépannage

Si aucune information n’est retournée lorsque la tâche est terminée, affichez les journaux d’activité d’erreurs. Pour afficher les informations d’erreur relatives à cette tâche, ajoutez les commandes suivantes à la fin du fichier `hivejob.ps1`, enregistrez-le, puis réexécutez-le.

```powershell
# Print the output of the Hive job.
Get-AzHDInsightJobOutput `
        -Clustername $clusterName `
        -JobId $job.JobId `
        -HttpCredential $creds `
        -DisplayOutputType StandardError
```

Cette applet de commande retourne les informations écrites sur STDERR lors de l’exécution de la tâche.

## <a name="summary"></a>Résumé

Comme vous pouvez le constater, Azure PowerShell permet d'exécuter facilement des requêtes Hive sur un cluster HDInsight, de surveiller l'état de la tâche et de récupérer le résultat.

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir des informations générales sur Hive dans HDInsight :

* [Utiliser Apache Hive avec Apache Hadoop sur HDInsight](hdinsight-use-hive.md)

Pour plus d’informations sur d’autres méthodes de travail avec Hadoop sur HDInsight :

* [Utiliser MapReduce avec Apache Hadoop sur HDInsight](hdinsight-use-mapreduce.md)
