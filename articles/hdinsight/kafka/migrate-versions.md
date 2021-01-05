---
title: Migrer des charges de travail Apache Kafka vers Azure HDInsight 4.0
description: Découvrez comment migrer des charges de travail Apache Kafka de HDInsight 3.6 vers HDInsight 4.0.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.date: 12/18/2019
ms.openlocfilehash: 0a31c6cf32222277e033aacf7d04622c54aef9ea
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "80437012"
---
# <a name="migrate-apache-kafka-workloads-to-azure-hdinsight-40"></a>Migrer des charges de travail Apache Kafka vers Azure HDInsight 4.0

Azure HDInsight 4.0 offre les derniers composants open source avec des améliorations significatives en matière de performances, de connectivité et de sécurité. Ce document explique comment migrer des charges de travail Apache Kafka de HDInsight 3.6 vers HDInsight 4.0. Après avoir migré vos charges de travail vers HDInsight 4.0, vous pouvez utiliser un grand nombre des nouvelles fonctionnalités qui ne sont pas disponibles sur HDInsight 3.6.

## <a name="hdinsight-36-kafka-migration-paths"></a>Chemins de migration Kafka HDInsight 3.6

HDInsight 3.6 prend en charge deux versions de Kafka : 1.0.0 et 1.1.0. HDInsight 4.0 prend en charge les versions 1.1.0 et 2.1.0. En fonction de la version de Kafka et de la version de HDInsight que vous souhaitez exécuter, plusieurs chemins de migration sont pris en charge. Ces chemins sont expliqués ci-dessous et illustrés dans le diagramme suivant.

* **Exécuter les versions les plus récentes de Kafka et de HDInsight (recommandé)**  : Migrez une application HDInsight 3.6 et Kafka 1.0.0 ou 1.1.0 vers HDInsight 4.0 avec Kafka 2.1.0 (chemins D et E ci-dessous).
* **Exécuter la version la plus récente de HDInsight, mais uniquement une version plus récente de Kafka** : Migrez une application HDInsight 3.6 et Kafka 1.0.0 vers HDInsight 4.0 avec Kafka 1.1.0 (chemin B ci-dessous).
* **Exécuter la version la plus récente de HDInsight et conserver la version de Kafka** : Migrez une application HDInsight 3.6 et Kafka 1.1.0 vers HDInsight 4.0 avec Kafka 1.1.0 (chemin C ci-dessous).
* **Exécuter une version plus récente de Kafka et conserver la version de HDInsight** : Migrez une application Kafka 1.0.0 vers Kafka 1.1.0 et restez sur HDInsight 3.6 (chemin A ci-dessous). Notez que cette option nécessite encore le déploiement d’un nouveau cluster. La mise à niveau de la version de Kafka sur un cluster existant n’est pas prise en charge. Après avoir créé un cluster avec la version de votre choix, migrez vos clients Kafka pour utiliser le nouveau cluster.

![Chemins de mise à niveau pour Apache Kafka sur la version 3.6](./media/upgrade-threesix-to-four/apache-kafka-upgrade-path.png)

## <a name="apache-kafka-versions"></a>Versions d’Apache Kafka

### <a name="kafka-110"></a>Kafka 1.1.0
  
Si vous migrez de Kafka 1.0.0 vers 1.1.0, vous pouvez tirer parti des nouvelles fonctionnalités suivantes :

* Les améliorations apportées au contrôleur Kafka accélèrent l’arrêt contrôlé, ce qui vous permet de redémarrer les répartiteurs et de récupérer plus rapidement en cas de problème. 
* Les améliorations apportées à la [logique FetchRequests](https://issues.apache.org/jira/browse/KAFKA-6254) vous permettent d’avoir davantage de partitions (et donc plus de rubriques) sur le cluster. 
* Kafka Connect prend en charge les [en-têtes d’enregistrement](https://issues.apache.org/jira/browse/KAFKA-5142) et les [expressions régulières](https://issues.apache.org/jira/browse/KAFKA-3073) pour les rubriques. 

Pour obtenir la liste complète des mises à jour, consultez les [notes de publication d’Apache Kafka 1.1](https://archive.apache.org/dist/kafka/1.1.0/RELEASE_NOTES.html).

### <a name="apache-kafka-210"></a>Apache Kafka 2.1.0

Si vous migrez à partir de Kafka 2.1, vous pouvez tirer parti des nouvelles fonctionnalités suivantes :

* Meilleure résilience des répartiteurs en raison d’un protocole de réplication amélioré.
* Nouvelles fonctionnalités de l’API KafkaAdminClient.
* Gestion de quota configurable.
* Prise en charge de la compression Zstandard.

Pour obtenir la liste complète des mises à jour, consultez les [notes de publication d’Apache Kafka 2.0](https://archive.apache.org/dist/kafka/2.0.0/RELEASE_NOTES.html) et les [notes de publication d’Apache Kafka 2.1](https://archive.apache.org/dist/kafka/2.1.0/RELEASE_NOTES.html).

## <a name="kafka-client-compatibility"></a>Compatibilité des clients Kafka

Les nouveaux répartiteurs Kafka prennent en charge les clients plus anciens. L’article [KIP-35 - Retrieving protocol version](https://cwiki.apache.org/confluence/display/KAFKA/KIP-35+-+Retrieving+protocol+version) introduit un mécanisme permettant de déterminer de façon dynamique les fonctionnalités d’un répartiteur Kafka et l’article [KIP-97 : Improved Kafka Client RPC Compatibility Policy](https://cwiki.apache.org/confluence/display/KAFKA/KIP-97%3A+Improved+Kafka+Client+RPC+Compatibility+Policy) introduit une nouvelle stratégie de compatibilité et des garanties pour le client Java. Auparavant, un client Kafka devait interagir avec un répartiteur de la même version ou d’une version plus récente. Désormais, les versions les plus récentes des clients Java et des autres clients qui prennent en charge KIP-35, telles que `librdkafka`, peuvent revenir à des types de demandes plus anciens ou lever des erreurs appropriées si la fonctionnalité n’est pas disponible.

![Mettre à niveau la compatibilité des clients Kafka](./media/upgrade-threesix-to-four/apache-kafka-client-compatibility.png)

Notez que cela ne signifie pas que le client prend en charge des répartiteurs plus anciens.  Pour plus d’informations, consultez [Compatibility Matrix](https://cwiki.apache.org/confluence/display/KAFKA/Compatibility+Matrix).

## <a name="general-migration-process"></a>Processus de migration général

Les instructions de migration suivantes supposent qu’un cluster Apache Kafka 1.0.0 ou 1.1.0 est déployé sur HDInsight 3.6 dans un réseau virtuel unique. Le répartiteur existant comporte des rubriques et il est activement utilisé par les producteurs et les consommateurs.

![Environnement supposé Kafka actuel](./media/upgrade-threesix-to-four/apache-kafka-presumed-environment.png)

Pour effectuer la migration, procédez comme suit :

1. **Déployez un nouveau cluster HDInsight 4.0 et des clients pour le test.** Déployez un nouveau cluster Kafka HDInsight 4.0. Si plusieurs versions de cluster Kafka peuvent être sélectionnées, il est recommandé de sélectionner la version la plus récente. Après le déploiement, définissez des paramètres en fonction des besoins et créez une rubrique avec le même nom que votre environnement existant. De plus, définissez le chiffrement TLS et BYOK (Bring-Your-Own-Key) en fonction des besoins. Vérifiez ensuite s’il fonctionne correctement avec le nouveau cluster.

    ![Déployer de nouveaux clusters HDInsight 4.0](./media/upgrade-threesix-to-four/deploy-new-hdinsight-clusters.png)

1. **Changez le cluster utilisé pour l’application de producteur et attendez que toutes les données de file d’attente soient consommées par les consommateurs actuels.** Quand le nouveau cluster Kafka HDInsight 4.0 est prêt, basculez la destination de producteur existante vers le nouveau cluster. Laissez-le tel quel jusqu’à ce que l’application consommateur existante ait consommé toutes les données du cluster existant.

    ![Basculer le cluster pour l’application producteur](./media/upgrade-threesix-to-four/switch-cluster-producer-app.png)

1. **Basculez le cluster sur l’application consommateur.** Après avoir confirmé que l’application consommateur existante a fini de consommer toutes les données du cluster existant, basculez la connexion vers le nouveau cluster.

    ![Basculer le cluster sur l’application consommateur](./media/upgrade-threesix-to-four/switch-cluster-consumer-app.png)

1. **Supprimez l’ancien cluster et testez les applications en fonction des besoins.** Une fois que le basculement est terminé et qu’il fonctionne correctement, supprimez l’ancien cluster Kafka HDInsight 3.6, ainsi que les producteurs et les consommateurs utilisés dans le test en fonction des besoins.

## <a name="next-steps"></a>Étapes suivantes

* [Optimisation des performances pour les clusters Kafka HDInsight Apache](apache-kafka-performance-tuning.md)
* [Démarrage rapide : Créer un cluster Apache Kafka dans Azure HDInsight à l’aide du Portail Azure](apache-kafka-get-started.md)
