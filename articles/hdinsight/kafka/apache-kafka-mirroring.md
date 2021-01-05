---
title: Mettre en miroir les rubriques Apache Kafka - Azure HDInsight
description: Découvrez comment utiliser la fonctionnalité de mise en miroir d’Apache Kafka afin de conserver un réplica Kafka sur un cluster HDInsight par la mise en miroir de rubriques sur un cluster secondaire.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 11/29/2019
ms.openlocfilehash: d4a2be6719fdaaa9dc859df21cc030478e474210
ms.sourcegitcommit: 6906980890a8321dec78dd174e6a7eb5f5fcc029
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92428251"
---
# <a name="use-mirrormaker-to-replicate-apache-kafka-topics-with-kafka-on-hdinsight"></a>MirrorMaker permet de répliquer des rubriques Apache Kafka avec Kafka sur HDInsight

Découvrez comment utiliser la fonctionnalité de mise en miroir d’Apache Kafka pour répliquer des rubriques sur un cluster secondaire. La mise en miroir peut être exécutée sous la forme d’un processus continu ou être utilisée par intermittence comme une méthode de migration des données d’un cluster à l’autre.

> [!NOTE]
> Cet article contient des références au terme *liste verte* , un terme que Microsoft n’utilise plus. Lorsque le terme sera supprimé du logiciel, nous le supprimerons de cet article.

Dans cet exemple, la mise en miroir est utilisée pour répliquer des rubriques entre deux clusters HDInsight. Les deux clusters se trouvent dans différents réseaux virtuels dans différents centres de données.

> [!WARNING]  
> La mise en miroir ne doit pas être considérée comme un moyen de bénéficier d’une tolérance de pannes. Le décalage aux éléments d’un sujet diffère entre le cluster principal et le cluster secondaire : les clients ne peuvent donc pas utiliser indifféremment les deux.
>
> Si la tolérance de pannes vous préoccupe, définissez la réplication pour les sujets au sein de votre cluster. Pour plus d’informations, consultez [Bien démarrer avec Apache Kafka sur HDInsight](apache-kafka-get-started.md).

## <a name="how-apache-kafka-mirroring-works"></a>Fonctionnement de la mise en miroir Apache Kafka

La mise en miroir se fait à l’aide de l’outil [MirrorMaker](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=27846330) (qui fait partie d’Apache Kafka). Cet outil permet d’utiliser les enregistrements de rubriques du cluster principal et de créer une copie locale sur le cluster secondaire. MirrorMaker utilise un (ou plusieurs) *consommateurs* qui lisent le cluster principal, ainsi qu’un *producteur* qui écrit dans le cluster local (secondaire).

Le programme de mise en miroir le plus utile en cas de récupération d'urgence utilise des clusters Kafka dans différentes régions Azure. Pour ce faire, les réseaux virtuels abritant les clusters sont appairés.

Le diagramme suivant illustre le processus de mise en miroir et le flux de communication entre les clusters :

![Diagramme du processus de mise en miroir](./media/apache-kafka-mirroring/kafka-mirroring-vnets2.png)

Le cluster principal et le cluster secondaire peuvent comporter un nombre différent de nœuds et de partitions, et les décalages dans les sujets ne sont pas forcément les mêmes. La mise en miroir conserve la valeur clé utilisée pour le partitionnement afin de préserver l’ordre d’enregistrement en fonction de la clé.

### <a name="mirroring-across-network-boundaries"></a>Mise en miroir au-delà des limites réseau

Si vous avez besoin d’effectuer une mise en miroir entre des clusters Kafka sur des réseaux différents, vous devez tenir compte des points suivants :

* **Passerelles** : les réseaux doivent être capables de communiquer au niveau TCP/IP.

* **Adressage du serveur**  : Vous pouvez choisir d'adresser vos nœuds de cluster à l’aide de leurs adresses IP ou noms de domaine complets.

    * **Adresses IP**  : Si vous configurez vos clusters Kafka pour la publication d'adresses IP, vous pouvez configurer la mise en miroir à l'aide des adresses IP des nœuds Broker et Zookeeper.
    
    * **Noms de domaine**  : Si vous ne configurez pas vos clusters Kafka pour la publication d’adresses IP, vos clusters doivent pouvoir se connecter entre eux à l’aide de noms de domaine complets (FQDN). Cela implique la configuration d’un serveur DNS (Domain Name System) dans chaque réseau pour l’acheminement des demandes vers les autres réseaux. Lorsque vous créez un réseau virtuel Azure, au lieu d’utiliser le DNS automatique fourni avec le réseau, vous devez spécifier un serveur DNS personnalisé et l’adresse IP du serveur. Après avoir créé le réseau virtuel, vous devez ensuite créer une machine virtuelle Azure qui utilise cette adresse IP, puis y installer et configurer le logiciel DNS.

    > [!WARNING]  
    > Créez et configurez le serveur DNS personnalisé avant d’installer HDInsight sur le réseau virtuel. Aucune configuration supplémentaire n’est requise pour HDInsight dans le cadre de l’utilisation du serveur DNS configuré pour le réseau virtuel.

Pour plus d’informations sur la connexion de deux réseaux virtuels Azure, consultez [Configurer une connexion de réseau virtuel à réseau virtuel](../../vpn-gateway/vpn-gateway-vnet-vnet-rm-ps.md).

## <a name="mirroring-architecture"></a>Architecture de mise en miroir

Cette architecture comporte deux clusters dans différents groupes de ressources et réseaux virtuels : un **principal** et un **secondaire**.

### <a name="creation-steps"></a>Étapes de création

1. Créez deux groupes de ressources :

    |Groupe de ressources | Emplacement |
    |---|---|
    | kafka-primary-rg | USA Centre |
    | kafka-secondary-rg | Centre-Nord des États-Unis |

1. Créez un réseau virtuel **kafka-principal-vnet** dans **kafka-principal-rg**. Conservez les paramètres par défaut.
1. Créez un réseau virtuel **kafka-secondary-vnet** dans **kafka-secondary-rg** , également avec les paramètres par défaut.

1. Créez deux nouveaux clusters Kafka :

    | Nom du cluster | Groupe de ressources | Réseau virtuel | Compte de stockage |
    |---|---|---|---|
    | kafka-primary-cluster | kafka-primary-rg | kafka-primary-vnet | kafkaprimarystorage |
    | kafka-secondary-cluster | kafka-secondary-rg | kafka-secondary-vnet | kafkasecondarystorage |

1. Créez des peerings de réseaux virtuels. Cette étape permet de créer deux peerings : un entre **kafka-primary-vnet** et **kafka-secondary-vnet** et un autre entre **kafka-secondary-vnet** et **kafka-primary-vnet**.
    1. Sélectionnez le réseau virtuel **kafka-primary-vnet**.
    1. Sélectionnez **Peerings** sous **Paramètres**.
    1. Sélectionnez **Ajouter**.
    1. Dans l'écran **Ajouter le peering** , entrez les détails comme indiqué dans la capture d’écran ci-dessous.

        ![HDInsight Kafka - Ajout de l’appairage de réseau virtuel](./media/apache-kafka-mirroring/hdi-add-vnet-peering.png)

### <a name="configure-ip-advertising"></a>Configurer la publication d’adresses IP

Configurez la publication d’adresses IP pour permettre à un client de se connecter à l’aide des adresses IP du répartiteur plutôt que des noms de domaine.

1. Accédez au tableau de bord Ambari pour le cluster principal : `https://PRIMARYCLUSTERNAME.azurehdinsight.net`.
1. Sélectionnez **Services** > **Kafka**. Sélectionnez l’onglet **Configurations**.
1. Ajoutez les lignes de configuration suivantes en bas de la section **kafka-env template**. Sélectionnez **Enregistrer**.

    ```
    # Configure Kafka to advertise IP addresses instead of FQDN
    IP_ADDRESS=$(hostname -i)
    echo advertised.listeners=$IP_ADDRESS
    sed -i.bak -e '/advertised/{/advertised@/!d;}' /usr/hdp/current/kafka-broker/conf/server.properties
    echo "advertised.listeners=PLAINTEXT://$IP_ADDRESS:9092" >> /usr/hdp/current/kafka-broker/conf/server.properties
    ```

1. Entrez une note dans l'écran **Enregistrer la configuration** , puis cliquez sur **Enregistrer**.
1. Si un avertissement de configuration s’affiche, cliquez sur **Continuer**.
1. Sélectionnez **OK** dans **Enregistrer les modifications de configuration**.
1. Sélectionnez **Redémarrer** > **Redémarrer tous les éléments affectés** dans la notification **Redémarrage requis**. Sélectionnez **Confirmer le redémarrage**.

    ![Apache Ambari - Redémarrer tous les éléments affectés](./media/apache-kafka-mirroring/ambari-restart-notification.png)

### <a name="configure-kafka-to-listen-on-all-network-interfaces"></a>Configurez Kafka de manière à écouter sur toutes les interfaces réseau.
    
1. Restez sur l'onglet **Configurations** sous **Services** > **Kafka**. Dans la section **Kafka Broker** , définissez la propriété **listeners** sur `PLAINTEXT://0.0.0.0:9092`.
1. Sélectionnez **Enregistrer**.
1. Sélectionnez **Redémarrer** , puis **Confirmer le redémarrage**.

### <a name="record-broker-ip-addresses-and-zookeeper-addresses-for-primary-cluster"></a>Enregistrez les adresses IP Broker et Zookeeper pour le cluster principal.

1. Sélectionnez **Hôtes** dans le tableau de bord Ambari.
1. Notez les adresses IP Broker et Zookeeper. Les nœuds Broker portent les deux premières lettres du nom d'hôte, **wn** , et les nœuds Zookeeper les deux premières lettres du nom d'hôte **zk**.

    ![Apache Ambari - Afficher les adresses IP du nœud](./media/apache-kafka-mirroring/view-node-ip-addresses2.png)

1. Répétez les trois étapes précédentes pour le deuxième cluster **kafka-secondary-cluster**  : configurer la publication d’adresses IP, définissez listeners et notez les adresses IP Broker et Zookeeper.

## <a name="create-topics"></a>Création de sujets

1. Connectez-vous au cluster **principal** à l'aide de SSH :

    ```bash
    ssh sshuser@PRIMARYCLUSTER-ssh.azurehdinsight.net
    ```

    Remplacez **sshuser** par le nom d’utilisateur SSH que vous avez utilisé lors de la création du cluster. Remplacez **PRIMARYCLUSTER** par le nom de base utilisé pour créer le cluster.

    Pour plus d’informations, consultez [Utiliser SSH avec HDInsight](../hdinsight-hadoop-linux-use-ssh-unix.md).

1. Utilisez la commande suivante pour créer une variable avec les hôtes Apache Zookeeper pour le cluster principal. Les chaînes telles que `ZOOKEEPER_IP_ADDRESS1` doivent être remplacées par les adresses IP réelles enregistrées précédemment, telles que `10.23.0.11` et `10.23.0.7`. Si vous utilisez la résolution de nom de domaine complet avec un serveur DNS personnalisé, suivez [ces étapes](apache-kafka-get-started.md#getkafkainfo) pour obtenir les noms Broker et Zookeeper :

    ```bash
    # get the zookeeper hosts for the primary cluster
    export PRIMARY_ZKHOSTS='ZOOKEEPER_IP_ADDRESS1:2181, ZOOKEEPER_IP_ADDRESS2:2181, ZOOKEEPER_IP_ADDRESS3:2181'
    ```

1. Pour créer une rubrique nommée `testtopic`, utilisez la commande suivante :

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 2 --partitions 8 --topic testtopic --zookeeper $PRIMARY_ZKHOSTS
    ```

1. Exécutez la commande suivante pour vérifier la création du sujet :

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --list --zookeeper $PRIMARY_ZKHOSTS
    ```

    La réponse contient `testtopic`.

1. Utilisez la commande suivante afin d’afficher les informations sur l’hôte Zookeeper pour ce cluster ( **principal** ) :

    ```bash
    echo $PRIMARY_ZKHOSTS
    ```

    Cette commande renvoie des informations semblables au texte suivant :

    `10.23.0.11:2181,10.23.0.7:2181,10.23.0.9:2181`

    Enregistrez ces informations. Elles seront utilisées dans la prochaine section.

## <a name="configure-mirroring"></a>Configuration de la mise en miroir

1. Connectez-vous au cluster **secondaire** à l’aide d’une autre session SSH :

    ```bash
    ssh sshuser@SECONDARYCLUSTER-ssh.azurehdinsight.net
    ```

    Remplacez **sshuser** par le nom d’utilisateur SSH que vous avez utilisé lors de la création du cluster. Remplacez **SECONDARYCLUSTER** par le nom utilisé lors de la création du cluster.

    Pour plus d’informations, consultez [Utiliser SSH avec HDInsight](../hdinsight-hadoop-linux-use-ssh-unix.md).

1. Un fichier `consumer.properties` est utilisé pour configurer la communication avec le cluster **principal**. Pour créer le fichier, utilisez la commande suivante :

    ```bash
    nano consumer.properties
    ```

    Utilisez les données suivantes comme contenu du fichier `consumer.properties` :

    ```yaml
    zookeeper.connect=PRIMARY_ZKHOSTS
    group.id=mirrorgroup
    ```

    Remplacez **PRIMARY_ZKHOSTS** par les adresses IP Zookeeper du cluster **principal**.

    Ce fichier détaille les informations de consommateur à utiliser lors de la lecture du cluster principal Kafka. Pour plus d’informations sur la configuration du consommateur, consultez [Consumer Configs](https://kafka.apache.org/documentation#consumerconfigs) (Configurations de consommateur) sur kafka.apache.org.

    Pour enregistrer le fichier, appuyez sur **Ctrl+X** , sur **O** , puis sur **Entrée**.

1. Avant de configurer le producteur qui communique avec le cluster secondaire, définissez une variable pour les adresses IP Broker du cluster **secondaire**. Utilisez les commandes suivantes pour créer cette variable :

    ```bash
    export SECONDARY_BROKERHOSTS='BROKER_IP_ADDRESS1:9092,BROKER_IP_ADDRESS2:9092,BROKER_IP_ADDRESS2:9092'
    ```

    La commande `echo $SECONDARY_BROKERHOSTS` doit renvoyer des informations semblables aux informations suivantes :

    `10.23.0.14:9092,10.23.0.4:9092,10.23.0.12:9092`

1. Un fichier `producer.properties` est utilisé pour communiquer avec le cluster **secondaire**. Pour créer le fichier, utilisez la commande suivante :

    ```bash
    nano producer.properties
    ```

    Utilisez les données suivantes comme contenu du fichier `producer.properties` :

    ```yaml
    bootstrap.servers=SECONDARY_BROKERHOSTS
    compression.type=none
    ```

    Remplacez **SECONDARY_BROKERHOSTS** par les adresses IP Broker utilisées à l’étape précédente.

    Pour plus d’informations sur la configuration du producteur, consultez [Producer Configs](https://kafka.apache.org/documentation#producerconfigs) (Configurations de producteur) sur kafka.apache.org.

1. Utilisez les commandes suivantes pour créer une variable d’environnement avec les adresses IP des hôtes Zookeeper pour le cluster secondaire :

    ```bash
    # get the zookeeper hosts for the secondary cluster
    export SECONDARY_ZKHOSTS='ZOOKEEPER_IP_ADDRESS1:2181,ZOOKEEPER_IP_ADDRESS2:2181,ZOOKEEPER_IP_ADDRESS3:2181'
    ```

1. La configuration par défaut pour Kafka sur HDInsight n’autorise pas la création automatique de rubriques. Vous devez utiliser une des options suivantes avant de commencer le processus de mise en miroir :

    * **Créer les rubriques sur le cluster secondaire**  : cette option vous permet également de définir le nombre de partitions et le facteur de réplication.

        Vous pouvez créer à l’avance les rubriques avec la commande suivante :

        ```bash
        /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 2 --partitions 8 --topic testtopic --zookeeper $SECONDARY_ZKHOSTS
        ```

        Remplacez `testtopic` par le nom de la rubrique à créer.

    * **Configure the cluster for automatic topic creation** (Configurer le cluster pour la création automatique des rubriques) : cette option permet à MirrorMaker de créer automatiquement des rubriques, mais en créant éventuellement un nombre différent de partitions ou un autre facteur de réplication que la rubrique principale.

        Pour configurer le cluster secondaire afin de créer automatiquement des rubriques, procédez comme suit :

        1. Accédez au tableau de bord Ambari pour le cluster secondaire : `https://SECONDARYCLUSTERNAME.azurehdinsight.net`.
        1. Cliquez sur **Services** > **Kafka**. Sélectionnez l’onglet **Configurations** .
        1. Dans le champ __Filter__ (Filtrer), entrez la valeur `auto.create`. Cette option filtre la liste des propriétés et affiche le paramètre `auto.create.topics.enable`.
        1. Définissez la valeur `auto.create.topics.enable` sur true, puis sélectionnez __Save__ (Enregistrer). Ajoutez une note, puis sélectionnez à nouveau __Save__ (Enregistrer).
        1. Sélectionnez le service __Kafka__ , choisissez __Restart__ (Redémarrer), puis __Restart all affected__ (Redémarrer tous les éléments affectés). Lorsque vous y êtes invité, sélectionnez __Confirm Restart All__ (Confirmer le redémarrage).

        ![Kafka - Activer la création automatique des rubriques](./media/apache-kafka-mirroring/kafka-enable-auto-create-topics.png)

## <a name="start-mirrormaker"></a>Lancement de MirrorMaker

1. À partir de la connexion SSH du cluster **secondaire** , utilisez la commande suivante pour lancer le processus MirrorMaker :

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-run-class.sh kafka.tools.MirrorMaker --consumer.config consumer.properties --producer.config producer.properties --whitelist testtopic --num.streams 4
    ```

    Les paramètres utilisés dans cet exemple sont :

    |Paramètre |Description |
    |---|---|
    |--consumer.config|spécifie le fichier qui contient les propriétés du consommateur. Ces propriétés permettent de créer un consommateur qui lit le cluster *principal* Kafka.|
    |--producer.config|spécifie le fichier qui contient les propriétés du producteur. Ces propriétés permettent de créer un producteur qui écrit dans le cluster *secondaire* Kafka.|
    |--whitelist|liste de sujets que MirrorMaker réplique, du cluster principal vers le cluster secondaire.|
    |--num.streams|nombre de threads de consommateur à créer.|

    Le consommateur du nœud secondaire attend maintenant de recevoir des messages.

2. À partir de la connexion SSH au cluster **principal** , utilisez la commande suivante pour démarrer un producteur et envoyer des messages à la rubrique :

    ```bash
    export PRIMARY_BROKERHOSTS=BROKER_IP_ADDRESS1:9092,BROKER_IP_ADDRESS2:9092,BROKER_IP_ADDRESS2:9092
    /usr/hdp/current/kafka-broker/bin/kafka-console-producer.sh --broker-list $SOURCE_BROKERHOSTS --topic testtopic
    ```

     Lorsque vous arrivez sur une ligne vide avec un curseur, tapez quelques messages texte. Les messages sont envoyés à la rubrique sur le cluster **principal**. Lorsque vous avez terminé, utilisez **Ctrl + C** pour terminer le processus de production.

3. À partir de la connexion SSH au cluster **secondaire** , utilisez **Ctrl + C** pour mettre fin au processus MirrorMaker. Plusieurs secondes peuvent être nécessaires pour terminer le processus. Pour vérifier que les messages ont été répliqués vers le cluster secondaire, utilisez la commande suivante :

    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-console-consumer.sh --bootstrap-server $SECONDARY_ZKHOSTS --topic testtopic --from-beginning
    ```

    La liste des rubriques inclut désormais `testtopic`, qui est créé lorsque MirrorMaster reflète la rubrique du cluster principal vers le cluster secondaire. Les messages récupérés de la rubrique sont les mêmes que ceux entrés dans le cluster principal.

## <a name="delete-the-cluster"></a>Supprimer le cluster

[!INCLUDE [delete-cluster-warning](../../../includes/hdinsight-delete-cluster-warning.md)]

Les étapes décrites dans ce document ont permis de créer des clusters dans différents groupes de ressources Azure. Pour supprimer toutes les ressources créées, vous pouvez supprimer les deux groupes ressources créés : **kafka-principal-rg** et **kafka-secondary_rg**. La suppression de ces groupes de ressources supprime toutes les ressources créées par la suite dans ce document, y compris les clusters, réseaux virtuels et comptes de stockage.

## <a name="next-steps"></a>Étapes suivantes

Dans ce document, vous avez appris à utiliser [MirrorMaker](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=27846330) pour créer un réplica d’un cluster [Apache Kafka](https://kafka.apache.org/). Utilisez les liens suivants pour découvrir d’autres façons de travailler avec Kafka :

* [Documentation d’Apache Kafka MirrorMaker](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=27846330) sur cwiki.apache.org.
* [Meilleures pratiques Kafka Mirror Maker](https://community.cloudera.com/t5/Community-Articles/Kafka-Mirror-Maker-Best-Practices/ta-p/249269)
* [Prise en main d’Apache Kafka sur HDInsight](apache-kafka-get-started.md)
* [Utiliser Apache Spark avec Apache Kafka sur HDInsight](../hdinsight-apache-spark-with-kafka.md)
* [Se connecter à Apache Kafka via un réseau virtuel Azure](apache-kafka-connect-vpn-gateway.md)
