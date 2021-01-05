---
title: Tutoriel – Apache Kafka et Sécurité Entreprise – Azure HDInsight
description: Didacticiel - Découvrez comment configurer des stratégies Apache Ranger pour Kafka dans Azure HDInsight avec le Pack Sécurité Entreprise (ESP, Enterprise Security Package).
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: tutorial
ms.date: 05/19/2020
ms.openlocfilehash: d2780b3456a802904800b894f6849544cfee4e61
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96010127"
---
# <a name="tutorial-configure-apache-kafka-policies-in-hdinsight-with-enterprise-security-package-preview"></a>Tutoriel : Configurer des stratégies Apache Kafka dans HDInsight avec le Pack Sécurité Entreprise (préversion)

Découvrez comment configurer des stratégies Apache Ranger pour des clusters Apache Kafka avec le Pack Sécurité Entreprise. Les clusters ESP sont connectés à un domaine permettant aux utilisateurs de s’authentifier avec les informations d’identification du domaine. Dans ce tutoriel, vous créez deux stratégies Ranger pour restreindre l’accès aux rubriques `sales` et `marketingspend`.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer des utilisateurs du domaine
> * Création de stratégies Ranger
> * Créer des rubriques dans un cluster Kafka
> * Tester des stratégies Ranger

## <a name="prerequisite"></a>Configuration requise

Un [cluster Kafka HDInsight avec le Pack Sécurité Entreprise](./apache-domain-joined-configure-using-azure-adds.md).

## <a name="connect-to-apache-ranger-admin-ui"></a>Connexion à l’interface utilisateur Apache Ranger

1. À partir d’un navigateur, connectez-vous à l’interface d’utilisateur administrateur de Ranger avec l’URL `https://ClusterName.azurehdinsight.net/Ranger/`. N’oubliez pas de remplacer `ClusterName` par le nom de votre cluster Kafka. Les informations d’identification de Ranger ne sont pas les mêmes que celles du cluster Hadoop. Pour empêcher les navigateurs d’utiliser les informations d’identification Hadoop mises en cache, utilisez une nouvelle fenêtre de navigation InPrivate pour vous connecter à l’interface utilisateur administrateur de Ranger.

2. Connectez-vous avec vos informations d’identification d’administrateur Azure Active Directory. Les informations d’identification administrateur Azure AD ne sont pas les mêmes que celles du cluster HDInsight ou que celles du protocole SSH du nœud HDInsight Linux.

   ![Interface utilisateur d’administration Apache Ranger HDInsight](./media/apache-domain-joined-run-kafka/apache-ranger-admin-login.png)

## <a name="create-domain-users"></a>Créer des utilisateurs du domaine

Consultez [Créer un cluster HDInsight avec le Pack Sécurité Entreprise](./apache-domain-joined-configure-using-azure-adds.md) pour découvrir comment créer les utilisateurs du domaine **sales_user** et **marketing_user**. Dans un scénario de production, les utilisateurs du domaine proviennent de votre locataire Active Directory.

## <a name="create-ranger-policy"></a>Créer une stratégie Ranger

Créez une stratégie Ranger pour **sales_user** et **marketing_user**.

1. Ouvrez **l’interface utilisateur de l’administrateur Ranger**.

2. Sélectionnez **\<ClusterName>_kafka** sous **Kafka**. Il est possible qu’une stratégie préconfigurée figure dans la liste.

3. Sélectionnez **Ajouter une nouvelle stratégie**, puis entrez les valeurs suivantes :

   |Paramètre  |Valeur suggérée  |
   |---------|---------|
   |Nom de la stratégie  |  hdi sales* policy   |
   |Rubrique   |  sales* |
   |Sélectionner un utilisateur  |  sales_user1 |
   |Autorisations  | publier, consommer, créer |

   Les caractères génériques suivants peuvent être inclus dans le nom de la rubrique :

   * « * » indique zéro, une ou plusieurs occurrences des caractères.
   * « ? » indique n’importe quel caractère individuel.

   ![Interface utilisateur de l’administrateur Apache Ranger - Créer une stratégie1](./media/apache-domain-joined-run-kafka/apache-ranger-admin-create-policy.png)

   Attendez quelques instants pour que Ranger se synchronise avec Azure AD si un utilisateur du domaine n’est pas renseigné automatiquement dans **Sélectionner un utilisateur**.

4. Sélectionnez **Ajouter** pour enregistrer la stratégie.

5. Sélectionnez **Ajouter une nouvelle stratégie**, puis entrez les valeurs suivantes :

   |Paramètre  |Valeur suggérée  |
   |---------|---------|
   |Nom de la stratégie  |  hdi marketing policy   |
   |Rubrique   |  marketingspend |
   |Sélectionner un utilisateur  |  marketing_user1 |
   |Autorisations  | publier, consommer, créer |

   ![Interface utilisateur de l’administrateur Apache Ranger - Créer une stratégie2](./media/apache-domain-joined-run-kafka/apache-ranger-admin-create-policy-2.png)  

6. Sélectionnez **Ajouter** pour enregistrer la stratégie.

## <a name="create-topics-in-a-kafka-cluster-with-esp"></a>Créer des rubriques dans un cluster Kafka avec ESP

Pour créer les deux rubriques, `salesevents` et `marketingspend` :

1. Utilisez la commande suivante pour ouvrir une connexion SSH avec le cluster :

   ```cmd
   ssh DOMAINADMIN@CLUSTERNAME-ssh.azurehdinsight.net
   ```

   Remplacez `DOMAINADMIN` par l’utilisateur administrateur de votre cluster qui a été configuré pendant la [création du cluster](./apache-domain-joined-configure-using-azure-adds.md#create-an-hdinsight-cluster-with-esp), puis remplacez `CLUSTERNAME` par le nom de votre cluster. Si vous y êtes invité, entrez le mot de passe du compte de l’utilisateur administrateur. Pour en savoir plus sur l’utilisation de `SSH` avec HDInsight, voir [Utiliser SSH avec Hadoop - Azure HDInsight](../../hdinsight/hdinsight-hadoop-linux-use-ssh-unix.md).

2. Utilisez les commandes suivantes pour enregistrer le nom du cluster dans une variable et pour installer un utilitaire d’analyse JSON `jq`. Quand vous y êtes invité, entrez le nom du cluster Kafka.

   ```bash
   sudo apt -y install jq
   read -p 'Enter your Kafka cluster name:' CLUSTERNAME
   ```

3. Utilisez les commandes suivantes pour obtenir les hôtes du répartiteur Kafka. Quand vous y êtes invité, entrez le mot de passe du compte de l’administrateur du cluster.

   ```bash
   export KAFKABROKERS=`curl -sS -u admin -G https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/services/KAFKA/components/KAFKA_BROKER | jq -r '["\(.host_components[].HostRoles.host_name):9092"] | join(",")' | cut -d',' -f1,2`; \
   ```

   Avant de continuer, il vous faut peut-être configurer votre environnement de développement, si vous ne l’avez pas déjà fait. Vous aurez besoin de composants comme JDK Java, Apache Maven et d’un client SSH avec scp. Pour plus d’informations, consultez la section relative aux [instructions de configuration](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started/tree/master/DomainJoined-Producer-Consumer).

1. Téléchargez les [exemples de producteur et consommateur joints au domaine Apache Kafka](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started/tree/master/DomainJoined-Producer-Consumer).

1. Suivez les étapes 2 et 3 de la section **Générer et déployer l’exemple** dans [Didacticiel : Utiliser les API de producteur et de consommateur Apache Kafka](../kafka/apache-kafka-producer-consumer-api.md#build-and-deploy-the-example)

1. Exécutez les commandes suivantes :

   ```bash
   java -jar -Djava.security.auth.login.config=/usr/hdp/current/kafka-broker/conf/kafka_client_jaas.conf kafka-producer-consumer.jar create salesevents $KAFKABROKERS
   java -jar -Djava.security.auth.login.config=/usr/hdp/current/kafka-broker/conf/kafka_client_jaas.conf kafka-producer-consumer.jar create marketingspend $KAFKABROKERS
   ```

## <a name="test-the-ranger-policies"></a>Tester les stratégies Ranger

En fonction des stratégies Ranger configurées, **sales_user** peut produire/consommer la rubrique `salesevents`, mais pas la rubrique `marketingspend`. À l’inverse, **marketing_user** peut produire/consommer la rubrique `marketingspend`, mais pas la rubrique `salesevents`.

1. Ouvrez une connexion SSH avec le cluster. Utilisez la commande suivante pour vous connecter en tant que **sales_user1** :

   ```bash
   ssh sales_user1@CLUSTERNAME-ssh.azurehdinsight.net
   ```

2. Utiliser les noms du répartiteur de la section précédente pour définir les variables d’environnement suivantes :

   ```bash
   export KAFKABROKERS=<brokerlist>:9092
   ```

   Exemple : `export KAFKABROKERS=wn0-khdicl.contoso.com:9092,wn1-khdicl.contoso.com:9092`

3. Suivez l’étape 3 de la section **Générer et déployer l’exemple** dans [Tutoriel : Utiliser les API de producteur et de consommateur Apache Kafka](../kafka/apache-kafka-producer-consumer-api.md#build-and-deploy-the-example) pour vérifier que `kafka-producer-consumer.jar` est également accessible à **sales_user**.

   > [!NOTE]  
   > Pour ce tutoriel, utilisez le fichier kafka-producer-consumer.jar qui se trouve sous le projet « DomainJoined-Producer-Consumer » (à ne pas confondre avec celui du projet Producer-Consumer, qui est destiné aux scénarios non joints à un domaine).

4. Vérifiez que **sales_user1** peut produire à destination de la rubrique `salesevents` en exécutant la commande suivante :

   ```bash
   java -jar -Djava.security.auth.login.config=/usr/hdp/current/kafka-broker/conf/kafka_client_jaas.conf kafka-producer-consumer.jar producer salesevents $KAFKABROKERS
   ```

5. Exécutez la commande suivante pour consommer ce qui provient de la rubrique `salesevents` :

   ```bash
   java -jar -Djava.security.auth.login.config=/usr/hdp/current/kafka-broker/conf/kafka_client_jaas.conf kafka-producer-consumer.jar consumer salesevents $KAFKABROKERS
   ```

   Vérifiez que vous pouvez lire les messages.

6. Vérifiez que **sales_user1** ne peut pas produire à destination de la rubrique `marketingspend` en exécutant ce qui suit dans la même fenêtre ssh :

   ```bash
   java -jar -Djava.security.auth.login.config=/usr/hdp/current/kafka-broker/conf/kafka_client_jaas.conf kafka-producer-consumer.jar producer marketingspend $KAFKABROKERS
   ```

   Une erreur d’autorisation se produit et elle peut être ignorée.

7. Notez que **marketing_user1** ne peut pas consommer ce qui provient de la rubrique `salesevents`.

   Répétez les étapes 1 à 3 ci-dessus, mais cette fois en tant que **marketing_user1**.

   Exécutez la commande suivante pour consommer ce qui provient de la rubrique `salesevents` :

   ```bash
   java -jar -Djava.security.auth.login.config=/usr/hdp/current/kafka-broker/conf/kafka_client_jaas.conf kafka-producer-consumer.jar consumer salesevents $KAFKABROKERS
   ```

   Les messages précédents ne sont pas visibles.

8. Consultez les événements d’accès de l’audit à partir de l’interface utilisateur de Ranger.

   ![Événements d’accès d’audit de stratégie de l’interface utilisateur Ranger ](./media/apache-domain-joined-run-kafka/apache-ranger-admin-audit.png)
   
## <a name="produce-and-consume-topics-in-esp-kafka-by-using-the-console"></a>Produire et consommer des rubriques dans ESP Kafka en utilisant la console

> [!NOTE]
> Vous ne pouvez pas utiliser des commandes de console pour créer des rubriques. Vous devez plutôt utiliser le code Java présenté dans la section précédente. Pour plus d’informations, consultez [Créer des rubriques dans un cluster Kafka avec ESP](#create-topics-in-a-kafka-cluster-with-esp).

Pour produire et consommer des rubriques dans ESP Kafka en utilisant la console :

1. Utilisez `kinit` avec le nom d’utilisateur de l’utilisateur. Entrez le mot de passe quand vous y êtes invité.

   ```bash
   kinit sales_user1
   ```

2. Définissez des variables d’environnement :

   ```bash
   export KAFKA_OPTS="-Djava.security.auth.login.config=/usr/hdp/current/kafka-broker/conf/kafka_client_jaas.conf"
   export KAFKABROKERS=<brokerlist>:9092
   ```

3. Produisez des messages dans la rubrique `salesevents` :

   ```bash
   /usr/hdp/current/kafka-broker/bin/kafka-console-producer.sh --topic salesevents --broker-list $KAFKABROKERS --security-protocol SASL_PLAINTEXT
   ```

4. Consommez des messages provenant de la rubrique `salesevents` :

   ```bash
   /usr/hdp/current/kafka-broker/bin/kafka-console-consumer.sh --topic salesevents --from-beginning --bootstrap-server $KAFKABROKERS --security-protocol SASL_PLAINTEXT
   ```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous ne comptez pas continuer à utiliser cette application, effectuez les étapes suivantes pour supprimer le cluster Kafka que vous avez créé :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
1. Dans la zone **Recherche** située en haut, tapez **HDInsight**.
1. Sous **Services**, sélectionnez **Clusters HDInsight**.
1. Dans la liste des clusters HDInsight qui s’affiche, cliquez sur les points de suspension **...** à côté du cluster que vous avez créé pour ce tutoriel. 
1. Cliquez sur **Supprimer**. Cliquez sur **Oui**.

## <a name="troubleshooting"></a>Dépannage
Si le fichier kafka-producer-consumer.jar ne fonctionne pas sur un cluster joint à un domaine, vérifiez que vous utilisez bien le fichier kafka-producer-consumer.jar qui se trouve sous le projet « DomainJoined-Producer-Consumer » (à ne pas confondre avec celui du projet Producer-Consumer, qui est destiné aux scénarios non joints à un domaine).

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Chiffrement de disque de clés gérées par le client](../disk-encryption.md)
