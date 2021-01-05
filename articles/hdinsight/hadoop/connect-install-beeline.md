---
title: Se connecter à Apache Beeline ou l’installer – Azure HDInsight
description: Découvrez comment vous connecter au client Apache Beeline pour exécuter des requêtes Hive avec Hadoop sur HDInsight. Beeline est un utilitaire qui permet d’utiliser HiveServer2 sur JDBC.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: contperf-fy21q1
ms.date: 05/27/2020
ms.openlocfilehash: a0a693b2dc2c236cbac79ec4789b2a480ce65427
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97028935"
---
# <a name="connect-to-apache-beeline-on-hdinsight-or-install-it-locally"></a>Se connecter à Apache Beeline sur HDInsight ou l’installer localement

[Apache Beeline](https://cwiki.apache.org/confluence/display/Hive/HiveServer2+Clients#HiveServer2Clients-Beeline–NewCommandLineShell) est un client Hive inclus dans les nœuds principaux de votre cluster HDInsight. Cet article explique comment se connecter au client Beeline installé sur votre cluster HDInsight via différents types de connexions. Il explique également comment [installer le client Beeline localement](#install-beeline-client). 

## <a name="types-of-connections"></a>Types de connexions

### <a name="from-an-ssh-session"></a>À partir d’une session SSH

Lors de la connexion à partir d’une session SSH à un nœud principal de cluster, vous pouvez ensuite vous connecter à l’adresse `headnodehost` sur le port `10001` :

```bash
beeline -u 'jdbc:hive2://headnodehost:10001/;transportMode=http'
```

### <a name="over-an-azure-virtual-network"></a>Sur un réseau virtuel Azure

Lors de la connexion d’un client à HDInsight sur un réseau virtuel Azure, vous devez fournir le nom de domaine complet d’un nœud principal de cluster. Puisque cette connexion est établie directement aux nœuds de cluster, la connexion utilise le port `10001` :

```bash
beeline -u 'jdbc:hive2://<headnode-FQDN>:10001/;transportMode=http'
```

Remplacez `<headnode-FQDN>` par le nom de domaine complet d’un nœud principal de cluster. Pour rechercher le nom de domaine complet d’un nœud principal, utilisez les informations contenues dans le document [Gérer des clusters HDInsight à l’aide de l’API REST d’Apache Ambari](../hdinsight-hadoop-manage-ambari-rest-api.md#get-the-fqdn-of-cluster-nodes).

### <a name="to-hdinsight-enterprise-security-package-esp-cluster-using-kerberos"></a>Vers un cluster du Pack Sécurité Entreprise (ESP) HDInsight à l'aide de Kerberos

Lors de la connexion d’un client à un cluster Pack Sécurité Entreprise (ESP) joint à Azure Active Directory (AAD) DS sur une machine dans le même domaine du cluster, vous devez également spécifier le nom de domaine `<AAD-Domain>` et le nom d’un compte d’utilisateur de domaine autorisé à accéder au cluster `<username>` :

```bash
kinit <username>
beeline -u 'jdbc:hive2://<headnode-FQDN>:10001/default;principal=hive/_HOST@<AAD-Domain>;auth-kerberos;transportMode=http' -n <username>
```

Remplacez `<username>` par le nom d’un compte de domaine autorisé à accéder au cluster. Remplacez `<AAD-DOMAIN>` par le nom d’Azure Active Directory (AAD) auquel le cluster est joint. Utilisez une chaîne en majuscules pour la valeur `<AAD-DOMAIN>`, sinon les informations d’identification ne seront pas trouvées. Vérifiez `/etc/krb5.conf` pour les noms de domaine si nécessaire.

Pour rechercher l’URL JDBC à partir d’Ambari :

1. Dans un navigateur web, accédez à `https://CLUSTERNAME.azurehdinsight.net/#/main/services/HIVE/summary`, où `CLUSTERNAME` est le nom de votre cluster. Vérifiez que HiveServer2 est en cours d’exécution.

1. Utilisez le Presse-papiers pour copier l’URL JDBC HiveServer2.

### <a name="over-public-or-private-endpoints"></a>Via des points de terminaison publics ou privés

En cas de connexion à un cluster utilisant des points de terminaison publics ou privés, vous devez fournir le nom de compte de connexion de cluster (par défaut `admin`) et le mot de passe. Par exemple, utilisation de Beeline à partir d’un système client pour se connecter à l’adresse `clustername.azurehdinsight.net`. Cette connexion est établie via le port `443` et est chiffrée à l'aide du protocole TLS/SSL.

Remplacez `clustername` par le nom de votre cluster HDInsight : Remplacez `admin` par le compte de connexion de votre cluster. Pour des clusters ESP, utilisez l’UPN complet (par exemple, user@domain.com). Remplacez `password` par le mot de passe du compte de connexion du cluster.

```bash
beeline -u 'jdbc:hive2://clustername.azurehdinsight.net:443/;ssl=true;transportMode=http;httpPath=/hive2' -n admin -p 'password'
```

ou pour le point de terminaison privé :

```bash
beeline -u 'jdbc:hive2://clustername-int.azurehdinsight.net:443/;ssl=true;transportMode=http;httpPath=/hive2' -n admin -p 'password'
```

Les points de terminaison privés pointent vers un équilibreur de charge de base, qui est uniquement accessible à partir des réseaux virtuels associés dans la même région. Voir [constraints on global VNet peering and load balancers](../../virtual-network/virtual-networks-faq.md#what-are-the-constraints-related-to-global-vnet-peering-and-load-balancers) (Contraintes liées à l’homologation Global VNet Peering et aux équilibreurs de charge) pour plus d’informations. Vous pouvez utiliser la commande `curl` avec l’option `-v` pour résoudre les problèmes de connectivité avec les points de terminaison publics ou privés avant d’utiliser Beeline.

### <a name="use-beeline-with-apache-spark"></a>Utiliser Beeline avec Apache Spark

Apache Spark fournit sa propre implémentation de HiveServer2, qui est quelquefois appelée « serveur Spark Thrift ». Ce service utilise Spark SQL et non Hive pour résoudre les requêtes, ce qui selon les requêtes peut offrir de meilleures performances.

#### <a name="through-public-or-private-endpoints"></a>Via des points de terminaison publics ou privés

La chaîne de connexion utilisée est légèrement différente. Au lieu de contenir `httpPath=/hive2`, elle utilise `httpPath/sparkhive2`. Remplacez `clustername` par le nom de votre cluster HDInsight : Remplacez `admin` par le compte de connexion de votre cluster. Pour des clusters ESP, utilisez l’UPN complet (par exemple, user@domain.com). Remplacez `password` par le mot de passe du compte de connexion du cluster.

```bash
beeline -u 'jdbc:hive2://clustername.azurehdinsight.net:443/;ssl=true;transportMode=http;httpPath=/sparkhive2' -n admin -p 'password'
```

ou pour le point de terminaison privé :

```bash
beeline -u 'jdbc:hive2://clustername-int.azurehdinsight.net:443/;ssl=true;transportMode=http;httpPath=/sparkhive2' -n admin -p 'password'
```

Les points de terminaison privés pointent vers un équilibreur de charge de base, qui est uniquement accessible à partir des réseaux virtuels associés dans la même région. Voir [constraints on global VNet peering and load balancers](../../virtual-network/virtual-networks-faq.md#what-are-the-constraints-related-to-global-vnet-peering-and-load-balancers) (Contraintes liées à l’homologation Global VNet Peering et aux équilibreurs de charge) pour plus d’informations. Vous pouvez utiliser la commande `curl` avec l’option `-v` pour résoudre les problèmes de connectivité avec les points de terminaison publics ou privés avant d’utiliser Beeline.

#### <a name="from-cluster-head-or-inside-azure-virtual-network-with-apache-spark"></a>À partir du cluster principal ou à l’intérieur du réseau virtuel Azure avec Apache Spark

Lorsque vous vous connectez directement depuis le nœud principal du cluster, ou à partir d’une ressource à l’intérieur du même réseau virtuel Azure que le cluster HDInsight, le port `10002` doit être utilisé pour le serveur Spark Thrift à la place de `10001`. L’exemple suivant montre comment se connecter directement au nœud principal :

```bash
/usr/hdp/current/spark2-client/bin/beeline -u 'jdbc:hive2://headnodehost:10002/;transportMode=http'
```

## <a name="install-beeline-client"></a>Installer le client Beeline

Bien que Beeline soit inclus sur les nœuds principaux, il est utile dans certains cas de l’installer localement.  La procédure d’installation sur un ordinateur local concerne un [Sous-système Windows pour Linux](/windows/wsl/install-win10).

1. Mettez à jour vos listes de packages. Entrez la commande suivante dans votre interpréteur de commandes Bash :

    ```bash
    sudo apt-get update
    ```

1. Installez Java s’il n’est pas installé. Vous pouvez vérifier à l’aide de la commande `which java`.

    1. Si aucun package Java n’est installé, entrez la commande suivante :

        ```bash
        sudo apt install openjdk-11-jre-headless
        ```

    1. Ouvrez le fichier bashrc (souvent situé dans ~/.bashrc) : `nano ~/.bashrc`.

    1. Modifiez le fichier bashrc. Ajoutez la ligne suivante à la fin du fichier :

        ```bash
        export JAVA_HOME=/usr/lib/jvm/java-1.11.0-openjdk-amd64
        ```

        Appuyez ensuite sur **Ctrl+X**, sur **Y**, puis sur Entrée.

1. Téléchargez les archives Hadoop et Beeline, puis entrez les commandes suivantes :

    ```bash
    wget https://archive.apache.org/dist/hadoop/core/hadoop-2.7.3/hadoop-2.7.3.tar.gz
    wget https://archive.apache.org/dist/hive/hive-1.2.1/apache-hive-1.2.1-bin.tar.gz
    ```

1. Décompressez les archives, puis entrez les commandes suivantes :

    ```bash
    tar -xvzf hadoop-2.7.3.tar.gz
    tar -xvzf apache-hive-1.2.1-bin.tar.gz
    ```

1. Modifiez le fichier bashrc. Vous devez identifier le chemin d’accès de l’emplacement où les archives ont été décompressées. Si vous utilisez le [sous-système Windows pour Linux](/windows/wsl/install-win10) et avez scrupuleusement suivi les étapes, votre chemin doit être `/mnt/c/Users/user/`, où `user` est votre nom d’utilisateur.

    1. Ouvrez le fichier : `nano ~/.bashrc`

    1. Modifiez les commandes ci-dessous avec le chemin d’accès approprié, puis entrez-les à la fin du fichier bashrc :

        ```bash
        export HADOOP_HOME=/path_where_the_archives_were_unpacked/hadoop-2.7.3
        export HIVE_HOME=/path_where_the_archives_were_unpacked/apache-hive-1.2.1-bin
        PATH=$PATH:$HIVE_HOME/bin
        ```

    1. Appuyez ensuite sur **Ctrl+X**, sur **Y**, puis sur Entrée.

1. Fermez, puis rouvrez votre session bash.

1. Testez votre connexion. Utilisez le format de connexion à partir de [Via des points de terminaison publics ou privés](#over-public-or-private-endpoints) ci-dessus.

## <a name="next-steps"></a>Étapes suivantes

* Pour obtenir des exemples d’utilisation du client Beeline avec Apache Hive, consultez [Utiliser Apache Beeline avec Apache Hive](apache-hadoop-use-hive-beeline.md)
* Pour plus d’informations sur Hive dans HDInsight, consultez [Utiliser Apache Hive avec Apache Hadoop sur HDInsight](hdinsight-use-hive.md)