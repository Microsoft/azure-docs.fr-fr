---
title: Utiliser MapReduce et PowerShell avec Apache Hadoop - Azure HDInsight
description: Apprenez à utiliser PowerShell pour exécuter des tâches MapReduce à distance avec Apache Hadoop sur HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 01/08/2020
ms.openlocfilehash: 8b5f5a9fb8ddd25f750b19044bac7253bbff2f98
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92545206"
---
# <a name="run-mapreduce-jobs-with-apache-hadoop-on-hdinsight-using-powershell"></a>Exécuter des travaux MapReduce avec Apache Hadoop sur HDInsight à l’aide de PowerShell

[!INCLUDE [mapreduce-selector](../../../includes/hdinsight-selector-use-mapreduce.md)]

Ce document fournit un exemple d’utilisation d’Azure PowerShell pour exécuter une tâche MapReduce sur un Hadoop sur un cluster HDInsight.

## <a name="prerequisites"></a>Conditions préalables requises

* Un cluster Apache Hadoop sur HDInsight. Consultez [Créer des clusters Apache Hadoop à l’aide du Portail Azure](../hdinsight-hadoop-create-linux-clusters-portal.md).

* Le [module Az](/powershell/azure/) PowerShell installé.

## <a name="run-a-mapreduce-job"></a>Exécuter une tâche MapReduce

Azure PowerShell propose des *applets de commande* qui vous permettent d'exécuter à distance des tâches MapReduce sur HDInsight. En interne, PowerShell effectue des appels REST à [WebHCat](https://cwiki.apache.org/confluence/display/Hive/WebHCat) (anciennement nommé Templeton) exécuté sur le cluster HDInsight.

Les applets de commande suivantes sont utilisées lors de l’exécution des tâches MapReduce sur un cluster HDInsight à distance.

|Applet de commande | Description |
|---|---|
|Connect-AzAccount|Authentifie Azure PowerShell dans votre abonnement Azure.|
|New-AzHDInsightMapReduceJobDefinition|Crée une *définition de travail* avec les informations MapReduce spécifiées.|
|Start-AzHDInsightJob|Envoie la définition de travail à HDInsight et démarre le travail. Un objet *job* est retourné.|
|Wait-AzHDInsightJob|Utilise l’objet de travail pour vérifier l’état du travail. Il attend que la tâche soit terminée ou que le délai d’attente soit dépassé.|
|Get-AzHDInsightJobOutput|Utilisé pour récupérer la sortie du travail.|

Les étapes suivantes montrent comment utiliser ces applets de commande pour exécuter une tâche sur votre cluster HDInsight.

1. À l’aide d’un éditeur, enregistrez le code suivant sous **mapreducejob.ps1** .

    [!code-powershell[main](../../../powershell_scripts/hdinsight/use-mapreduce/use-mapreduce.ps1?range=5-69)]

2. Ouvrez une invite de commandes **Azure PowerShell** . Accédez à l’emplacement du fichier **mapreducejob.ps1** , puis utilisez les éléments suivants pour exécuter le script :

    ```azurepowershell
    .\mapreducejob.ps1
    ```

    Lorsque vous exécutez le script, vous êtes invité à entrer le nom du cluster HDInsight et le nom de connexion du cluster. Vous pouvez également être invité à vous authentifier sur votre abonnement Azure.

3. Une fois la tâche terminée, vous obtenez un résultat similaire au texte suivant :

    ```output
    Cluster         : CLUSTERNAME
    ExitCode        : 0
    Name            : wordcount
    PercentComplete : map 100% reduce 100%
    Query           :
    State           : Completed
    StatusDirectory : f1ed2028-afe8-402f-a24b-13cc17858097
    SubmissionTime  : 12/5/2014 8:34:09 PM
    JobId           : job_1415949758166_0071
    ```

    Cela indique que la tâche a été effectuée avec succès.

    > [!NOTE]  
    > Si **ExitCode** correspond à une valeur différente de 0, consultez [Dépannage](#troubleshooting).

    Cet exemple stocke également les fichiers téléchargés dans un dossier **output.txt** dans le répertoire à partir duquel vous avez exécuté le script.

### <a name="view-output"></a>Voir la sortie

Pour afficher les mots et les décomptes générés par la tâche, ouvrez le fichier **output.txt** dans un éditeur de texte.

> [!NOTE]  
> Les fichiers de résultat d’une tâche MapReduce sont immuables. Donc, si vous réexécutez cet exemple, vous devez modifier le nom du fichier de sortie.

## <a name="troubleshooting"></a>Dépannage

Si aucune information n’est retournée lorsque la tâche est terminée, affichez les erreurs liées à cette tâche. Pour afficher les informations d’erreur pour ce projet, ajoutez la commande suivante à la fin du fichier **mapreducejob.ps1** . Ensuite, enregistrez le fichier et réexécutez le script.

```powershell
# Print the output of the WordCount job.
Write-Host "Display the standard output ..." -ForegroundColor Green
Get-AzHDInsightJobOutput `
        -Clustername $clusterName `
        -JobId $wordCountJob.JobId `
        -HttpCredential $creds `
        -DisplayOutputType StandardError
```

Cette applet de commande retourne les informations écrites sur STDERR lors de l’exécution de la tâche.

## <a name="next-steps"></a>Étapes suivantes

Comme vous pouvez le voir, Azure PowerShell offre un moyen facile d’exécuter des tâches MapReduce sur un cluster HDInsight, de surveiller l’état de la tâche et de récupérer la sortie. Pour plus d’informations sur d’autres méthodes de travail avec Hadoop sur HDInsight :

* [Utilisation de MapReduce dans Hadoop HDInsight](hdinsight-use-mapreduce.md)
* [Utiliser Apache Hive avec Apache Hadoop sur HDInsight](hdinsight-use-hive.md)