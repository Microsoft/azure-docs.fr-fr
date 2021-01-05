---
title: Utiliser MapReduce et Curl avec Apache Hadoop dans HDInsight - Azure
description: Découvrez comment exécuter à distance des tâches MapReduce avec Apache Hadoop sur HDInsight à l’aide de Curl.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 01/13/2020
ms.openlocfilehash: 407db727f521ea7731f0cbdbdd05c4338c9f452e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86207726"
---
# <a name="run-mapreduce-jobs-with-apache-hadoop-on-hdinsight-using-rest"></a>Exécuter des tâches MapReduce avec Apache Hadoop sur HDInsight avec REST

Découvrez comment utiliser l’API REST WebHCat Apache Hive pour exécuter des tâches MapReduce sur un cluster Apache Hadoop sur HDInsight. Curl est utilisé pour illustrer comment interagir avec HDInsight en utilisant des requêtes HTTP brutes pour exécuter des tâches MapReduce.

> [!NOTE]  
> Si vous vous êtes déjà familiarisé avec l’utilisation de serveurs Hadoop sous Linux, mais que vous découvrez HDInsight, consultez le document [Ce qu’il faut savoir sur Apache Hadoop dans HDInsight sous Linux](../hdinsight-hadoop-linux-information.md).

## <a name="prerequisites"></a>Prérequis

* Un cluster Apache Hadoop sur HDInsight. Consultez [Créer des clusters Apache Hadoop à l’aide du Portail Azure](../hdinsight-hadoop-create-linux-clusters-portal.md).

Au choix :
  * Windows PowerShell ou,
  * [Curl](https://curl.haxx.se/) avec [jq](https://stedolan.github.io/jq/)

## <a name="run-a-mapreduce-job"></a>Exécuter une tâche MapReduce

> [!NOTE]  
> Lorsque vous utilisez Curl ou toute autre communication REST avec WebHCat, vous devez authentifier les requêtes en fournissant le nom d’utilisateur et le mot de passe d’administrateur de cluster HDInsight. Vous devez utiliser le nom du cluster dans l’URI utilisé pour envoyer les requêtes au serveur.
>
> L’API REST est sécurisée à l’aide de l’ [authentification d’accès de base](https://en.wikipedia.org/wiki/Basic_access_authentication). Vous devez toujours effectuer les requêtes à l’aide de HTTPS pour vous assurer que vos informations d’identification sont envoyées en toute sécurité sur le serveur.

### <a name="curl"></a>Curl

1. Pour faciliter l’utilisation, définissez les variables ci-dessous. Cet exemple est fondé sur un environnement Windows ; vous devez le modifier si nécessaire pour votre environnement.

    ```cmd
    set CLUSTERNAME=
    set PASSWORD=
    ```

1. À partir d’une ligne de commande, exécutez la commande suivante pour vérifier que vous pouvez vous connecter à votre cluster HDInsight.

    ```bash
    curl -u admin:%PASSWORD% -G https://%CLUSTERNAME%.azurehdinsight.net/templeton/v1/status
    ```

    Les paramètres utilisés dans cette commande sont les suivants :

   * **-u** : Indique le nom d’utilisateur et le mot de passe utilisés pour authentifier la demande
   * **-G** : Indique que cette opération est une requête GET

   Le début de l’URI, `https://CLUSTERNAME.azurehdinsight.net/templeton/v1`, est le même pour toutes les demandes.

    La réponse reçue est similaire au JSON suivant :

    ```json
    {"version":"v1","status":"ok"}
    ```

1. Pour soumettre une tâche MapReduce, utilisez la commande suivante. Modifiez le chemin d’accès à **jq** si nécessaire.

    ```cmd
    curl -u admin:%PASSWORD% -d user.name=admin ^
    -d jar=/example/jars/hadoop-mapreduce-examples.jar ^
    -d class=wordcount -d arg=/example/data/gutenberg/davinci.txt -d arg=/example/data/output ^
    https://%CLUSTERNAME%.azurehdinsight.net/templeton/v1/mapreduce/jar | ^
    C:\HDI\jq-win64.exe .id
    ```

    La fin de l’URI (/mapreduce/jar) indique à WebHCat que cette demande lance une tâche MapReduce à partir d’une classe dans un fichier jar. Les paramètres utilisés dans cette commande sont les suivants :

   * **-d** : `-G` n’est pas utilisé, la requête fait donc défaut à la méthode POST. `-d` spécifie les valeurs de données envoyées avec la demande.
     * **user.name** : Utilisateur qui exécute la commande
     * **jar** : L’emplacement du fichier jar contenant la classe à exécuter
     * **class** : La classe contenant la logique MapReduce
     * **arg** : Les arguments à transmettre à la tâche MapReduce. Dans le cas présent, le fichier texte d’entrée et le répertoire utilisés pour la sortie

    Cette commande doit retourner un ID de tâche qui peut être utilisé pour vérifier le statut de la tâche : `job_1415651640909_0026`.

1. Pour vérifier le statut de la tâche, utilisez la commande suivante. Remplacez la valeur de `JOBID` par la **valeur de** réelle renvoyée à l’étape précédente. Revoyez l’emplacement des **jq** si nécessaire.

    ```cmd
    set JOBID=job_1415651640909_0026

    curl -G -u admin:%PASSWORD% -d user.name=admin https://%CLUSTERNAME%.azurehdinsight.net/templeton/v1/jobs/%JOBID% | ^
    C:\HDI\jq-win64.exe .status.state
    ```

### <a name="powershell"></a>PowerShell

1. Pour faciliter l’utilisation, définissez les variables ci-dessous. Remplacez `CLUSTERNAME` par le véritable nom de votre cluster. Exécutez la commande et saisissez le mot de passe de connexion du cluster lorsque vous y êtes invité.

    ```powershell
    $clusterName="CLUSTERNAME"
    $creds = Get-Credential -UserName admin -Message "Enter the cluster login password"
    ```

1. Utilisez la commande suivante pour vérifier que vous pouvez vous connecter à votre cluster Azure HDInsight :

    ```powershell
    $resp = Invoke-WebRequest -Uri "https://$clustername.azurehdinsight.net/templeton/v1/status" `
        -Credential $creds `
        -UseBasicParsing
    $resp.Content
    ```

    La réponse reçue est similaire au JSON suivant :

    ```json
    {"version":"v1","status":"ok"}
    ```

1. Pour envoyer une tâche MapReduce, utilisez la commande suivante :

    ```powershell
    $reqParams = @{}
    $reqParams."user.name" = "admin"
    $reqParams.jar = "/example/jars/hadoop-mapreduce-examples.jar"
    $reqParams.class = "wordcount"
    $reqParams.arg = @()
    $reqParams.arg += "/example/data/gutenberg/davinci.txt"
    $reqparams.arg += "/example/data/output"
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/templeton/v1/mapreduce/jar" `
       -Credential $creds `
       -Body $reqParams `
       -Method POST `
       -UseBasicParsing
    $jobID = (ConvertFrom-Json $resp.Content).id
    $jobID
    ```

    La fin de l’URI (/mapreduce/jar) indique à WebHCat que cette demande lance une tâche MapReduce à partir d’une classe dans un fichier jar. Les paramètres utilisés dans cette commande sont les suivants :

    * **user.name** : Utilisateur qui exécute la commande
    * **jar** : L’emplacement du fichier jar contenant la classe à exécuter
    * **class** : La classe contenant la logique MapReduce
    * **arg** : Les arguments à transmettre à la tâche MapReduce. Dans le cas présent, le fichier texte d’entrée et le répertoire utilisés pour la sortie

   Cette commande doit retourner un ID de tâche qui peut être utilisé pour vérifier le statut de la tâche : `job_1415651640909_0026`.

1. Pour vérifier le statut de la tâche, utilisez la commande suivante :

    ```powershell
    $reqParams=@{"user.name"="admin"}
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/templeton/v1/jobs/$jobID" `
       -Credential $creds `
       -Body $reqParams `
       -UseBasicParsing

    # ConvertFrom-JSON can't handle duplicate names with different case
    # So change one to prevent the error
    $fixDup=$resp.Content.Replace("jobID","job_ID")
    (ConvertFrom-Json $fixDup).status.state
    ```

### <a name="both-methods"></a>Les deux méthodes

1. Si la tâche est terminée, l’état est `SUCCEEDED`.

1. Lorsque l’état de la tâche passe à `SUCCEEDED`, vous pouvez récupérer les résultats depuis le Stockage Blob Azure. Le paramètre `statusdir` transmis avec la requête contient l’emplacement du fichier de sortie. Dans cet exemple, l’emplacement est `/example/curl`. Cette adresse stocke le résultat de la tâche dans le stockage par défaut du cluster : `/example/curl`.

Vous pouvez répertorier et télécharger ces fichiers à l’aide de l' [interface de ligne de commande Azure](/cli/azure/install-azure-cli). Pour plus d’informations sur l’utilisation d’Azure CLI avec le stockage Blob Azure, consultez [Démarrage rapide : Créer, télécharger et lister des objets blob avec Azure CLI](../../storage/blobs/storage-quickstart-blobs-cli.md).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur d’autres méthodes de travail avec Hadoop sur HDInsight :

* [Utiliser MapReduce avec Apache Hadoop sur HDInsight](hdinsight-use-mapreduce.md)
* [Utiliser Apache Hive avec Apache Hadoop sur HDInsight](hdinsight-use-hive.md)

Pour plus d’informations sur l’interface REST utilisée dans cet article, consultez la [Référence WebHCat](https://cwiki.apache.org/confluence/display/Hive/WebHCat+Reference).
