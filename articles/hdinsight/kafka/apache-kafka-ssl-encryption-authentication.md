---
title: Chiffrement et authentification TLS Apache Kafka – Azure HDInsight
description: Configurez le chiffrement TLS pour la communication entre des clients Kafka et des répartiteurs Kafka, ainsi qu’entre des répartiteurs Kafka. Configurez l’authentification SSL des clients.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 05/01/2019
ms.openlocfilehash: 9a95970647a26ea80db9f63fb8523c6a65cc5e06
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96012303"
---
# <a name="set-up-tls-encryption-and-authentication-for-apache-kafka-in-azure-hdinsight"></a>Configurer le chiffrement et l’authentification TLS pour Apache Kafka dans Azure HDInsight

Cet article explique comment configurer le chiffrement TLS (Transport Layer Security), précédemment appelé chiffrement SSL (Secure Sockets Layer), entre les clients Apache Kafka et les répartiteurs Apache Kafka. Il fournit également les étapes nécessaires pour configurer l’authentification des clients (parfois appelée « TLS bidirectionnel »).

> [!Important]
> Vous pouvez utiliser deux clients pour les applications Kafka : un client Java et un client de console. Seul le client Java `ProducerConsumer.java` peut utiliser TLS à la fois pour la production et la consommation. Le client du producteur de console `console-producer.sh` ne fonctionne pas avec TLS.

> [!Note]
> Le producteur de console HDInsight Kafka avec la version 1.1 ne prend pas en charge le protocole SSL.

## <a name="apache-kafka-broker-setup"></a>Configuration du répartiteur Apache Kafka

Lors de sa configuration, le répartiteur TLS Kafka utilise quatre machines virtuelles de cluster HDInsight, de la manière suivante :

* Nœud principal 0 - Autorité de certification (CA)
* Nœuds Worker 0, 1, et 2 - Répartiteurs

> [!Note] 
> Ce guide utilise des certificats auto-signés. Toutefois, la solution la plus sécurisée consiste à utiliser des certificats émis par des autorités de certification.

Le récapitulatif du processus de configuration du répartiteur est le suivant :

1. Les étapes suivantes sont répétées sur chacun des trois nœuds Worker :

    1. Génération d’un certificat.
    1. Création d’une requête de signature de certificat.
    1. Envoi de cette requête à l’autorité de certification (CA).
    1. Connexion à l’autorité de certification et signature de la requête.
    1. Utilisez SCP pour renvoyer le certificat au nœud Worker.
    1. Utilisez SCP pour renvoyer le certificat de l’autorité de certification au nœud Worker.

1. Une fois que vous disposez de l’ensemble des certificats, vous pouvez les placer dans le magasin de certificats.
1. Accédez à Ambari et modifiez les configurations.

Utilisez les instructions détaillées ci-dessous pour terminer la configuration du répartiteur :

> [!Important]
> Dans les extraits de code suivants, « wnX » est une abréviation de l’un des trois nœuds Worker, qui doit être remplacée par `wn0`, `wn1` ou `wn2` selon le cas. `WorkerNode0_Name` et `HeadNode0_Name` doivent être remplacés par les noms des machines respectives.

1. Effectuez la configuration initiale sur le nœud principal 0, qui remplira le rôle d’autorité de certification pour HDInsight.

    ```bash
    # Create a new directory 'ssl' and change into it
    mkdir ssl
    cd ssl
    ```

1. Effectuez la même configuration initiale sur chacun des répartiteurs (nœuds Worker 0, 1 et 2).

    ```bash
    # Create a new directory 'ssl' and change into it
    mkdir ssl
    cd ssl
    ```

1. Sur chacun des nœuds Worker, exécutez les étapes suivantes à l’aide de l’extrait de code ci-dessous.
    1. Créez un magasin de clés et remplissez-le avec un nouveau certificat privé.
    1. Créez une requête de signature de certificat.
    1. Utilisez SCP pour envoyer la requête de signature de certificat à l’autorité de certification (le nœud principal 0).

    ```bash
    keytool -genkey -keystore kafka.server.keystore.jks -validity 365 -storepass "MyServerPassword123" -keypass "MyServerPassword123" -dname "CN=FQDN_WORKER_NODE" -storetype pkcs12
    keytool -keystore kafka.server.keystore.jks -certreq -file cert-file -storepass "MyServerPassword123" -keypass "MyServerPassword123"
    scp cert-file sshuser@HeadNode0_Name:~/ssl/wnX-cert-sign-request
    ```

1. Sur l’ordinateur de l’autorité de certification, exécutez la commande suivante pour créer des fichiers ca-cert et ca-key :

    ```bash
    openssl req -new -newkey rsa:4096 -days 365 -x509 -subj "/CN=Kafka-Security-CA" -keyout ca-key -out ca-cert -nodes
    ```

1. Passez à l’ordinateur associé à l’autorité de certification et signez toutes les requêtes de signature de certificat reçues :

    ```bash
    openssl x509 -req -CA ca-cert -CAkey ca-key -in wn0-cert-sign-request -out wn0-cert-signed -days 365 -CAcreateserial -passin pass:"MyServerPassword123"
    openssl x509 -req -CA ca-cert -CAkey ca-key -in wn1-cert-sign-request -out wn1-cert-signed -days 365 -CAcreateserial -passin pass:"MyServerPassword123"
    openssl x509 -req -CA ca-cert -CAkey ca-key -in wn2-cert-sign-request -out wn2-cert-signed -days 365 -CAcreateserial -passin pass:"MyServerPassword123"
    ```

1. Envoyez les certificats signés sur les nœuds Worker à partir de l’autorité de certification (headnode0).

    ```bash
    scp wn0-cert-signed sshuser@WorkerNode0_Name:~/ssl/cert-signed
    scp wn1-cert-signed sshuser@WorkerNode1_Name:~/ssl/cert-signed
    scp wn2-cert-signed sshuser@WorkerNode2_Name:~/ssl/cert-signed
    ```

1. Envoyez le certificat public à l’autorité de certification de chaque nœud Worker.

    ```bash
    scp ca-cert sshuser@WorkerNode0_Name:~/ssl/ca-cert
    scp ca-cert sshuser@WorkerNode1_Name:~/ssl/ca-cert
    scp ca-cert sshuser@WorkerNode2_Name:~/ssl/ca-cert
    ```

1. Sur chacun de ces nœuds, ajoutez le certificat public de l’autorité de certificat dans le magasin d’approbations et le magasin de clés. Ensuite, ajoutez le certificat signé du nœud Worker dans le magasin de clés.

    ```bash
    keytool -keystore kafka.server.truststore.jks -alias CARoot -import -file ca-cert -storepass "MyServerPassword123" -keypass "MyServerPassword123" -noprompt
    keytool -keystore kafka.server.keystore.jks -alias CARoot -import -file ca-cert -storepass "MyServerPassword123" -keypass "MyServerPassword123" -noprompt
    keytool -keystore kafka.server.keystore.jks -import -file cert-signed -storepass "MyServerPassword123" -keypass "MyServerPassword123" -noprompt

    ```

## <a name="update-kafka-configuration-to-use-tls-and-restart-brokers"></a>Mettre à jour la configuration Kafka pour utiliser le protocole TLS et redémarrer les répartiteurs

Vous avez désormais configuré chaque répartiteur Kafka avec un magasin de clés et un magasin d’approbations, et importé les certificats nécessaires. Nous allons maintenant modifier les propriétés de configuration Kafka associées à l’aide d’Ambari, puis redémarrer les répartiteurs Kafka.

Pour terminer la modification de configuration, procédez comme suit :

1. Connectez-vous au portail Azure, puis sélectionnez votre cluster Apache Kafka Azure HDInsight.
1. Accédez à l’interface utilisateur d’Ambari en cliquant sur **Accueil Ambari** sous **Tableaux de bord du cluster**.
1. Sous **Kafka Broker** (Répartiteur Kafka), définissez la propriété **listeners** sur `PLAINTEXT://localhost:9092,SSL://localhost:9093`.
1. Sous **Advanced kafka-broker** (Répartiteurs Kafka avancés), définissez la propriété **security.inter.broker.protocol** sur `SSL`

    ![Modification des propriétés de configuration SSL Kafka dans Ambari](./media/apache-kafka-ssl-encryption-authentication/editing-configuration-ambari.png)

1. Sous **Custom kafka-broker** (Répartiteur Kafka personnalisé), définissez la propriété **ssl.client.auth** sur `required`. Cette étape n’est nécessaire que si vous configurez l’authentification et le chiffrement.

    ![Modification des propriétés de configuration SSL Kafka dans Ambari](./media/apache-kafka-ssl-encryption-authentication/editing-configuration-ambari2.png)

1. Pour la version 3.6 de HDI, accédez à l’interface utilisateur d’Ambari et ajoutez les configurations suivantes sous **kafka-env avancé** et la propriété de **modèle kafka-env**.

    ```bash
    # Configure Kafka to advertise IP addresses instead of FQDN
    IP_ADDRESS=$(hostname -i)
    echo advertised.listeners=$IP_ADDRESS
    sed -i.bak -e '/advertised/{/advertised@/!d;}' /usr/hdp/current/kafka-broker/conf/server.properties
    echo "advertised.listeners=PLAINTEXT://$IP_ADDRESS:9092,SSL://$IP_ADDRESS:9093" >> /usr/hdp/current/kafka-broker/conf/server.properties
    echo "ssl.keystore.location=/home/sshuser/ssl/kafka.server.keystore.jks" >> /usr/hdp/current/kafka-broker/conf/server.properties
    echo "ssl.keystore.password=MyServerPassword123" >> /usr/hdp/current/kafka-broker/conf/server.properties
    echo "ssl.key.password=MyServerPassword123" >> /usr/hdp/current/kafka-broker/conf/server.properties
    echo "ssl.truststore.location=/home/sshuser/ssl/kafka.server.truststore.jks" >> /usr/hdp/current/kafka-broker/conf/server.properties
    echo "ssl.truststore.password=MyServerPassword123" >> /usr/hdp/current/kafka-broker/conf/server.properties
    ```

1. Voici la capture d’écran qui montre l’interface utilisateur de configuration d’Ambari avec ces modifications.

    Pour HDI version 3.6 :

    ![Modification de la propriété kafka-env template dans Ambari](./media/apache-kafka-ssl-encryption-authentication/editing-configuration-kafka-env.png)

    Pour HDI version 4.0 :

     ![Modification de la propriété kafka-env template dans Ambari 4](./media/apache-kafka-ssl-encryption-authentication/editing-configuration-kafka-env-four.png)

1. Redémarrez tous les répartiteurs Kafka.

## <a name="client-setup-without-authentication"></a>Configuration du client (sans authentification)

Si vous n’avez pas besoin d’authentification, les étapes de configuration du chiffrement TLS uniquement sont les suivantes :

1. Connectez-vous à l’autorité de certification (nœud principal actif).
1. Copiez le certificat d’autorité de certification de l’ordinateur de l’autorité de certification (wn0) vers la machine client.
1. Connectez-vous à la machine client (hn1) et accédez au dossier `~/ssl`.
1. Importez le certificat d’autorité de certification dans le magasin d’approbations.
1. Importez le certificat d’autorité de certification dans le magasin de clés.

Ces étapes sont détaillées dans les extraits de code suivants.

1. Connectez-vous au nœud de l’autorité de certification.

    ```bash
    ssh sshuser@HeadNode0_Name
    cd ssl
    ```

1. Copiez le certificat d’autorité de certification vers la machine client.

    ```bash
    scp ca-cert sshuser@HeadNode1_Name:~/ssl/ca-cert
    ```

1. Connectez-vous à la machine cliente (nœud principal en attente).

    ```bash
    ssh sshuser@HeadNode1_Name
    cd ssl
    ```

1. Importez le certificat d’autorité de certification dans le magasin d’approbations.

    ```bash
    keytool -keystore kafka.client.truststore.jks -alias CARoot -import -file ca-cert -storepass "MyClientPassword123" -keypass "MyClientPassword123" -noprompt
    ```

1. Importer le certificat d’autorité de certification dans le magasin de clés.
    
    ```bash
    keytool -keystore kafka.client.keystore.jks -alias CARoot -import -file ca-cert -storepass "MyClientPassword123" -keypass "MyClientPassword123" -noprompt
    ```

1. Créez le fichier `client-ssl-auth.properties` sur l’ordinateur client (hn1). Il doit contenir les lignes suivantes :

    ```config
    security.protocol=SSL
    ssl.truststore.location=/home/sshuser/ssl/kafka.client.truststore.jks
    ssl.truststore.password=MyClientPassword123
    ```

1. Démarrez le client d’administration avec les options de producteur et consommateur, afin de vérifier que les producteurs et consommateurs fonctionnent sur le port 9093. Reportez-vous à la section [Vérification](apache-kafka-ssl-encryption-authentication.md#verification) ci-dessous pour savoir comment vérifier l’installation à l’aide du producteur/du consommateur de console.

## <a name="client-setup-with-authentication"></a>Configuration du client (avec authentification)

> [!Note]
> Les étapes suivantes ne sont nécessaires que si vous installez à la fois le chiffrement **et** l’authentification TLS. Si vous configurez uniquement le chiffrement, voir [Configuration du client sans authentification](apache-kafka-ssl-encryption-authentication.md#client-setup-without-authentication).

Les quatre étapes suivantes résument les tâches nécessaires pour configurer le client :

1. Connectez-vous à la machine cliente (nœud principal en attente).
1. Créez un magasin de clés Java afin d’obtenir un certificat auto-signé pour le répartiteur. Ensuite, copiez le certificat sur la machine virtuelle où l’autorité de certification est exécutée.
1. Basculez vers la machine de l’autorité de certification (nœud principal actif) pour signer le certificat client.
1. Accédez à la machine cliente (nœud principal en attente), puis au dossier `~/ssl`. Copiez le certificat auto-signé sur la machine cliente.

Les détails de chaque étape sont fournis ci-dessous.

1. Connectez-vous à la machine cliente (nœud principal en attente).

    ```bash
    ssh sshuser@HeadNode1_Name
    ```

1. Supprimez un répertoire SSL existant.

    ```bash
    rm -R ~/ssl
    mkdir ssl
    cd ssl
    ```

1. Créez un magasin de clés Java et créez une demande de signature de certificat. 

    ```bash
    keytool -genkey -keystore kafka.client.keystore.jks -validity 365 -storepass "MyClientPassword123" -keypass "MyClientPassword123" -dname "CN=HEADNODE1_FQDN" -storetype pkcs12
    
    keytool -keystore kafka.client.keystore.jks -certreq -file client-cert-sign-request -storepass "MyClientPassword123" -keypass "MyClientPassword123"
    ```

1. Copiez la demande de signature de certificat vers l’autorité de certification.

    ```bash
    scp client-cert-sign-request sshuser@HeadNode0_Name:~/ssl/client-cert-sign-request
    ```

1. Basculez vers l’ordinateur de l’autorité de certification (nœud principal actif) et signez le certificat client.

    ```bash
    ssh sshuser@HeadNode0_Name
    cd ssl
    openssl x509 -req -CA ca-cert -CAkey ca-key -in ~/ssl/client-cert-sign-request -out ~/ssl/client-cert-signed -days 365 -CAcreateserial -passin pass:MyClientPassword123
    ```

1. Copiez le certificat client signé de l’autorité de certification (nœud principal actif) vers la machine client.

    ```bash
    scp client-cert-signed sshuser@HeadNode1_Name:~/ssl/client-signed-cert
    ```

1. Copiez le certificat d’autorité de certification vers la machine client.

    ```bash
    scp ca-cert sshuser@HeadNode1_Name:~/ssl/ca-cert
    ```

    1. Connectez-vous à l’ordinateur client (nœud principal en attente) et accédez au dossier ssl.

    ```bash
    ssh sshuser@HeadNode1_Name
    cd ssl
    ```

1. Créez un magasin client avec un certificat signé, puis importez le certificat d’autorité de certification dans le magasin de clés et le magasin d’approbations sur l’ordinateur client (hn1) :

    ```bash
    keytool -keystore kafka.client.truststore.jks -alias CARoot -import -file ca-cert -storepass "MyClientPassword123" -keypass "MyClientPassword123" -noprompt
    
    keytool -keystore kafka.client.keystore.jks -alias CARoot -import -file ca-cert -storepass "MyClientPassword123" -keypass "MyClientPassword123" -noprompt
    
    keytool -keystore kafka.client.keystore.jks -import -file client-cert-signed -storepass "MyClientPassword123" -keypass "MyClientPassword123" -noprompt
    ```

1. Créez un fichier `client-ssl-auth.properties` sur l’ordinateur client (hn1). Il doit contenir les lignes suivantes :

    ```bash
    security.protocol=SSL
    ssl.truststore.location=/home/sshuser/ssl/kafka.client.truststore.jks
    ssl.truststore.password=MyClientPassword123
    ssl.keystore.location=/home/sshuser/ssl/kafka.client.keystore.jks
    ssl.keystore.password=MyClientPassword123
    ssl.key.password=MyClientPassword123
    ```

## <a name="verification"></a>Vérification

Exécutez ces étapes sur l’ordinateur client.

> [!Note]
> Si HDInsight 4.0 et Kafka 2.1 sont installés, vous pouvez utiliser le producteur/consommateur de console pour vérifier votre configuration. Si ce n’est pas le cas, exécutez le producteur Kafka sur le port 9092 et envoyez des messages à la rubrique, puis utilisez le contrôle serveur consommateur Kafka sur le port 9093 qui utilise TLS.

### <a name="kafka-21-or-above"></a>Kafka 2.1 ou version ultérieure

1. Créez une rubrique s’il n’en existe pas.

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --zookeeper <ZOOKEEPER_NODE>:2181 --create --topic topic1 --partitions 2 --replication-factor 2
    ```

1. Démarrez le producteur de console et fournissez le chemin d’accès à `client-ssl-auth.properties` en tant que fichier de configuration pour le producteur.

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-console-producer.sh --broker-list <FQDN_WORKER_NODE>:9093 --topic topic1 --producer.config ~/ssl/client-ssl-auth.properties
    ```

1. Ouvrez une autre connexion ssh à la machine client et démarrez le consommateur de console, puis fournissez le chemin d’accès à `client-ssl-auth.properties` en tant que fichier de configuration pour le consommateur.

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-console-consumer.sh --bootstrap-server <FQDN_WORKER_NODE>:9093 --topic topic1 --consumer.config ~/ssl/client-ssl-auth.properties --from-beginning
    ```

### <a name="kafka-11"></a>Kafka 1.1

1. Créez une rubrique s’il n’en existe pas.

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --zookeeper <ZOOKEEPER_NODE_0>:2181 --create --topic topic1 --partitions 2 --replication-factor 2
    ```

1. Démarrez le producteur de console et fournissez le chemin d’accès à client-ssl-auth.properties en tant que fichier de configuration pour le producteur.

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-console-producer.sh --broker-list <FQDN_WORKER_NODE>:9092 --topic topic1 
    ```

1. Ouvrez une autre connexion ssh à la machine client et démarrez le consommateur de console, puis fournissez le chemin d’accès à `client-ssl-auth.properties` en tant que fichier de configuration pour le consommateur.

    ```bash
    $ /usr/hdp/current/kafka-broker/bin/kafka-console-consumer.sh --bootstrap-server <FQDN_WORKER_NODE>:9093 --topic topic1 --consumer.config ~/ssl/client-ssl-auth.properties --from-beginning
    ```

## <a name="next-steps"></a>Étapes suivantes

* [Présentation d’Apache Kafka sur HDInsight](apache-kafka-introduction.md)
