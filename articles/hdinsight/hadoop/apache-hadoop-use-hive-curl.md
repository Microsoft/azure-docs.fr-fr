---
title: Utiliser Apache Hadoop Hive avec Curl dans HDInsight - Azure
description: Découvrez comment transmettre à distance des tâches Apache Pig vers Azure HDInsight à l’aide de Curl.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 01/06/2020
ms.openlocfilehash: 956406ec5ac99be5973f1928bbb89db10e68b339
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96000491"
---
# <a name="run-apache-hive-queries-with-apache-hadoop-in-hdinsight-using-rest"></a>Exécuter des requêtes Apache Hive avec Apache Hadoop dans HDInsight à l’aide de REST

[!INCLUDE [hive-selector](../../../includes/hdinsight-selector-use-hive.md)]

Découvrez comment utiliser l’API REST WebHCat pour exécuter des requêtes Apache Hive avec Apache Hadoop sur un cluster Azure HDInsight.

## <a name="prerequisites"></a>Prérequis

* Un cluster Apache Hadoop sur HDInsight. Consultez [Bien démarrer avec HDInsight sur Linux](./apache-hadoop-linux-tutorial-get-started.md).

* Client REST. Ce document utilise [Invoke-WebRequest](/powershell/module/microsoft.powershell.utility/invoke-webrequest) dans Windows PowerShell et [Curl](https://curl.haxx.se/) dans [Bash](/windows/wsl/install-win10).

* Si vous utilisez Bash, vous devez également recourir à jq, un processeur JSON en ligne de commande.  Voir [https://stedolan.github.io/jq/](https://stedolan.github.io/jq/).

## <a name="base-uri-for-rest-api"></a>URI de base pour l’API Rest

L’URI (Uniform Resource Identifier) de base pour l’API REST sur HDInsight est `https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME`, où `CLUSTERNAME` est le nom de votre cluster.  Les noms de cluster dans les URI sont **sensibles à la casse**.  Le nom du cluster dans la partie du nom de domaine complet (FQDN) de l’URI (`CLUSTERNAME.azurehdinsight.net`) n’est pas sensible à la casse, au contraire des autres occurrences dans l’URI.

## <a name="authentication"></a>Authentification

Lorsque vous utilisez cURL ou toute autre communication REST avec WebHCat, vous devez authentifier les requêtes en fournissant le nom d’utilisateur et le mot de passe de l’administrateur du cluster HDInsight. L’API REST est sécurisée à l’aide de l’ [authentification de base](https://en.wikipedia.org/wiki/Basic_access_authentication). Pour aider à vous assurer que vos informations d’identification sont envoyées en toute sécurité sur le serveur, procédez toujours aux requêtes via le protocole HTTP sécurisé (HTTPS).

### <a name="setup-preserve-credentials"></a>Programme d’installation (conservez vos informations d’identification)

Conservez vos informations d’identification pour éviter de devoir les entrer pour chaque exemple.  Le nom du cluster est conservé lors d’une étape distincte.

**A. Bash**  
Modifiez le script ci-dessous en remplaçant `PASSWORD` par votre mot de passe réel.  Ensuite, entrez la commande.

```bash
export password='PASSWORD'
```  

**B. PowerShell** Exécutez le code ci-dessous et saisissez vos informations d’identification dans la fenêtre indépendante :

```powershell
$creds = Get-Credential -UserName "admin" -Message "Enter the HDInsight login"
```

### <a name="identify-correctly-cased-cluster-name"></a>Identifier le nom de cluster présentant la bonne casse

La casse réelle du nom du cluster peut être différente de la casse attendue, suivant la façon dont le cluster a été créé.  Les étapes suivantes indiquent la casse réelle, avant de la stocker dans une variable pour tous les exemples suivants.

Modifiez les scripts ci-dessous, en remplaçant `CLUSTERNAME` par le nom de votre cluster. Ensuite, entrez la commande. (Le nom du cluster pour le nom de domaine complet ne respecte pas la casse.)

```bash
export clusterName=$(curl -u admin:$password -sS -G "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters" | jq -r '.items[].Clusters.cluster_name')
echo $clusterName
```  

```powershell
# Identify properly cased cluster name
$resp = Invoke-WebRequest -Uri "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters" `
    -Credential $creds -UseBasicParsing
$clusterName = (ConvertFrom-Json $resp.Content).items.Clusters.cluster_name;

# Show cluster name
$clusterName
```

## <a name="run-a-hive-query"></a>Exécution d'une tâche Hive

1. Pour vérifier que vous pouvez vous connecter à votre cluster HDInsight, utilisez l’une des commandes suivantes :

    ```bash
    curl -u admin:$password -G https://$clusterName.azurehdinsight.net/templeton/v1/status
    ```

    ```powershell
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/templeton/v1/status" `
       -Credential $creds `
       -UseBasicParsing
    $resp.Content
    ```

    La réponse reçue est similaire au texte suivant :

    ```json
    {"status":"ok","version":"v1"}
    ```

    Les paramètres utilisés dans cette commande sont les suivants :

    * `-u` : nom d’utilisateur et mot de passe utilisés pour authentifier la requête.
    * `-G` : indique que la requête correspond à une opération GET.

1. Le début de l’URL (`https://$CLUSTERNAME.azurehdinsight.net/templeton/v1`) est le même pour toutes les requêtes. Le chemin (`/status`) indique que la requête doit retourner l’état de WebHCat (également appelé Templeton) au serveur. Vous pouvez également prendre connaissance de la version de Hive à l'aide de la commande suivante :

    ```bash
    curl -u admin:$password -G https://$clusterName.azurehdinsight.net/templeton/v1/version/hive
    ```

    ```powershell
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/templeton/v1/version/hive" `
       -Credential $creds `
       -UseBasicParsing
    $resp.Content
    ```

    Cette requête renvoie une réponse similaire au texte suivant :

    ```json
    {"module":"hive","version":"1.2.1000.2.6.5.3008-11"}
    ```

1. Utilisez la commande suivante pour créer une table nommée **log4jLogs** :

    ```bash
    jobid=$(curl -s -u admin:$password -d user.name=admin -d execute="DROP+TABLE+log4jLogs;CREATE+EXTERNAL+TABLE+log4jLogs(t1+string,t2+string,t3+string,t4+string,t5+string,t6+string,t7+string)+ROW+FORMAT+DELIMITED+FIELDS+TERMINATED+BY+' '+STORED+AS+TEXTFILE+LOCATION+'/example/data/';SELECT+t4+AS+sev,COUNT(*)+AS+count+FROM+log4jLogs+WHERE+t4+=+'[ERROR]'+AND+INPUT__FILE__NAME+LIKE+'%25.log'+GROUP+BY+t4;" -d statusdir="/example/rest" https://$clusterName.azurehdinsight.net/templeton/v1/hive | jq -r .id)
    echo $jobid
    ```

    ```powershell
    $reqParams = @{"user.name"="admin";"execute"="DROP TABLE log4jLogs;CREATE EXTERNAL TABLE log4jLogs(t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string) ROW FORMAT DELIMITED BY ' ' STORED AS TEXTFILE LOCATION '/example/data/;SELECT t4 AS sev,COUNT(*) AS count FROM log4jLogs WHERE t4 = '[ERROR]' GROUP BY t4;";"statusdir"="/example/rest"}
    $resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/templeton/v1/hive" `
       -Credential $creds `
       -Body $reqParams `
       -Method POST `
       -UseBasicParsing
    $jobID = (ConvertFrom-Json $resp.Content).id
    $jobID
    ```

    Cette requête utilise la méthode POST, qui envoie des données dans le cadre de la requête à l’API REST. Les valeurs de données suivantes sont envoyées avec la requête :

     * `user.name` : utilisateur qui exécute la commande.
     * `execute` : instructions HiveQL qui doivent être exécutées.
     * `statusdir`: répertoire dans lequel l’état de cette tâche est écrit.

   Ces instructions effectuent les opérations suivantes :

   * `DROP TABLE` : si la table existe déjà, elle est supprimée.
   * `CREATE EXTERNAL TABLE` : crée une nouvelle table externe dans Hive. Les tables externes stockent uniquement la définition de table dans Hive. Les données restent à l'emplacement d'origine.

     > [!NOTE]  
     > Les tables externes doivent être utilisées lorsque vous vous attendez à ce que les données sous-jacentes soient mises à jour par une source externe, par exemple, par un processus de téléchargement de données automatisé ou une autre opération MapReduce.
     >
     > La suppression d'une table externe ne supprime **pas** les données, mais seulement la définition de la table.

   * `ROW FORMAT` : formatage des données. Les champs de chaque journal sont séparés par un espace.
   * `STORED AS TEXTFILE LOCATION` : emplacement de stockage des données (répertoire example/data) stockées sous forme de texte.
   * `SELECT` : sélectionne toutes les lignes dont la colonne **t4** contient la valeur **[ERROR]** . Cette instruction renvoie la valeur **3**, car trois lignes contiennent cette valeur.

     > [!NOTE]  
     > Notez que les espaces entre les instructions HiveQL sont remplacés par le caractère `+` lorsqu'ils sont utilisés avec Curl. Les valeurs citées qui contiennent un espace, tel que le séparateur, ne doivent pas être remplacées par `+`.

      Cette commande retourne un ID de tâche qui peut être utilisé pour vérifier le statut de la tâche.

1. Pour vérifier le statut de la tâche, utilisez la commande suivante :

    ```bash
    curl -u admin:$password -d user.name=admin -G https://$clusterName.azurehdinsight.net/templeton/v1/jobs/$jobid | jq .status.state
    ```

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

    Si la tâche est terminée, l’état est **TERMINÉ**.

1. Une fois que le statut de la tâche est passé à **TERMINÉ**, vous pouvez récupérer les résultats depuis le stockage blob Azure. Le paramètre `statusdir` transmis avec la requête contient l’emplacement du fichier de sortie. Dans notre cas `/example/rest`. Cette adresse stocke le résultat dans le répertoire `example/curl` dans le stockage en cluster par défaut.

    Vous pouvez répertorier et télécharger ces fichiers à l’aide de l' [interface de ligne de commande Azure](/cli/azure/install-azure-cli). Pour plus d’informations sur l’utilisation d’Azure CLI avec Stockage Azure, consultez le document [Utiliser Azure CLI avec Stockage Azure](../../storage/blobs/storage-quickstart-blobs-cli.md).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur d’autres méthodes de travail avec Hadoop sur HDInsight :

* [Utiliser Apache Hive avec Apache Hadoop sur HDInsight](hdinsight-use-hive.md)
* [Utiliser MapReduce avec Apache Hadoop sur HDInsight](hdinsight-use-mapreduce.md)

Pour plus d’informations sur l’API REST utilisée dans ce document, consultez le document [Référence WebHCat](https://cwiki.apache.org/confluence/display/Hive/WebHCat+Reference).