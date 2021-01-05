---
title: Traitement des événements d’Event Hubs avec Storm - Azure HDInsight
description: Découvrez comment traiter les données de concentrateurs d’événements Azure avec une topologie Storm C# créée dans Visual Studio à l’aide des outils HDInsight pour Visual Studio.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.date: 01/14/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: dac56059455a75f4d64a698c416dc22793432bc8
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92545597"
---
# <a name="process-events-from-azure-event-hubs-with-apache-storm-on-hdinsight-c"></a>Traiter des événements issus d’Azure Event Hubs avec Apache Storm sur HDInsight (C#)

Découvrez comment travailler avec Azure Event Hubs à partir [d’Apache Storm](https://storm.apache.org/) sur HDInsight. Ce document utilise une topologie C# Storm pour lire et écrire des données à partir d’Event Hubs.

> [!NOTE]  
> Pour la version Java de ce projet, voir [Traiter des événements issus d’Azure Event Hubs avec Apache Storm sur HDInsight (Java)](https://github.com/Azure-Samples/hdinsight-java-storm-eventhub).

## <a name="scpnet"></a>SCP.NET

Les étapes indiquées dans ce document utilisent SCP.NET, un package NuGet qui vous permet de créer des topologies et des composants C# pour les utiliser avec Storm sur HDInsight.

La version 3.4 de HDInsight (et les versions suivantes) utilisent Mono pour exécuter des topologies C#. L’exemple utilisé dans ce document fonctionne avec HDInsight 3.6. Si vous prévoyez de créer vos propres solutions .NET pour HDInsight, consultez le document sur la [mono-compatibilité](https://www.mono-project.com/docs/about-mono/compatibility/) pour identifier les éventuelles incompatibilités.

### <a name="cluster-versioning"></a>Contrôle de version de cluster

Le package NuGet Microsoft.SCP.Net.SDK que vous utilisez pour votre projet doit correspondre à la version principale de Storm installée sur HDInsight. Les versions 3.5 et 3.6 de HDInsight utilisent Storm 1.x. Vous devez donc utiliser la version 1.0.x.x de SCP.NET avec ces clusters.

Les topologies C# doivent également cibler la version 4.5 de .NET.

## <a name="how-to-work-with-event-hubs"></a>Utilisation d’Event Hubs

Microsoft fournit un ensemble de composants Java qui peut être utilisé pour communiquer avec les concentrateurs d’événements à partir d’une topologie Storm. Vous pouvez trouver le fichier d’archives Java (JAR) qui contient une version 3.6 compatible de HDInsight pour ces composants à l’adresse [https://github.com/hdinsight/mvn-repo/raw/master/org/apache/storm/storm-eventhubs/1.1.0.1/storm-eventhubs-1.1.0.1.jar](https://github.com/hdinsight/mvn-repo/raw/master/org/apache/storm/storm-eventhubs/1.1.0.1/storm-eventhubs-1.1.0.1.jar).

> [!IMPORTANT]  
> Même si les composants sont écrits en Java, vous pouvez les utiliser facilement à partir d’une topologie C#.

Les composants suivants sont utilisés dans cet exemple :

* __EventHubSpout__ : lit les données à partir d’Event Hubs.
* __EventHubBolt__ : écrit des données dans Event Hubs.
* __EventHubSpoutConfig__ : permet de configurer EventHubSpout.
* __EventHubBoltConfig__ : permet de configurer EventHubBolt.

### <a name="example-spout-usage"></a>Exemple d’utilisation du spout

SCP.NET fournit des méthodes pour l’ajout d’un EventHubSpout à votre topologie. Il est plus facile d’utiliser ces méthodes pour ajouter un Spout que d’utiliser les méthodes génériques pour ajouter un composant Java. L’exemple suivant illustre la création d’un spout en utilisant les méthodes __SetEventHubSpout__ et **EventHubSpoutConfig** de SCP.NET :

```csharp
 topologyBuilder.SetEventHubSpout(
    "EventHubSpout",
    new EventHubSpoutConfig(
        ConfigurationManager.AppSettings["EventHubSharedAccessKeyName"],
        ConfigurationManager.AppSettings["EventHubSharedAccessKey"],
        ConfigurationManager.AppSettings["EventHubNamespace"],
        ConfigurationManager.AppSettings["EventHubEntityPath"],
        eventHubPartitions),
    eventHubPartitions);
```

L’exemple précédent crée un composant spout nommé __EventHubSpout__ et le configure pour communiquer avec un concentrateur d’événements. L’indicateur de parallélisme du composant est défini sur le nombre de partitions dans le concentrateur d’événements. Ce paramètre permet à Storm de créer une instance du composant pour chaque partition.

### <a name="example-bolt-usage"></a>Exemple d’utilisation du bolt

Utilisez la méthode **JavaComponentConstructor** pour créer une instance du bolt. L’exemple suivant illustre la création et la configuration d’une instance de **l’EventHubBolt**  :

```csharp
// Java construcvtor for the Event Hub Bolt
JavaComponentConstructor constructor = JavaComponentConstructor.CreateFromClojureExpr(
    String.Format(@"(org.apache.storm.eventhubs.bolt.EventHubBolt. (org.apache.storm.eventhubs.bolt.EventHubBoltConfig. " +
        @"""{0}"" ""{1}"" ""{2}"" ""{3}"" ""{4}"" {5}))",
        ConfigurationManager.AppSettings["EventHubPolicyName"],
        ConfigurationManager.AppSettings["EventHubPolicyKey"],
        ConfigurationManager.AppSettings["EventHubNamespace"],
        "servicebus.windows.net",
        ConfigurationManager.AppSettings["EventHubName"],
        "true"));

// Set the bolt to subscribe to data from the spout
topologyBuilder.SetJavaBolt(
    "eventhubbolt",
    constructor,
    partitionCount)
        .shuffleGrouping("Spout");
```

> [!NOTE]  
> Cet exemple utilise une expression Clojure transmise sous forme de chaîne au lieu d’utiliser **JavaComponentConstructor** pour créer un **EventHubBoltConfig** , comme dans l’exemple de spout. Les deux méthodes fonctionnent. Utilisez celle qui vous convient.

## <a name="download-the-completed-project"></a>Télécharger le projet complet

Vous pouvez télécharger une version complète du projet créé dans cet article à partir de [GitHub](https://github.com/Azure-Samples/hdinsight-dotnet-java-storm-eventhub). Toutefois, vous devrez tout de même fournir des paramètres de configuration en suivant les étapes de cet article.

### <a name="prerequisites"></a>Conditions préalables requises

* Un cluster Apache Storm sur HDInsight. Consultez la section [Création de clusters Apache Hadoop à l’aide du portail Azure](../hdinsight-hadoop-create-linux-clusters-portal.md) et sélectionnez **Storm** dans le champ **Type de cluster** .

* Un [concentrateur d’événements Azure](../../event-hubs/event-hubs-create.md).

* Le [kit de développement logiciel (SDK) Azure .NET](https://azure.microsoft.com/downloads/).

* Les [outils HDInsight pour Visual Studio](../hadoop/apache-hadoop-visual-studio-tools-get-started.md).

* Java JDK 1.8 ou version ultérieure sur votre environnement de développement. Les téléchargements JDK sont disponibles dans [Oracle](/azure/developer/java/fundamentals/java-jdk-long-term-support).

  * La variable d’environnement **JAVA_HOME** doit pointer vers le répertoire qui contient Java.
  * Le répertoire **%JAVA_HOME%/bin** doit se trouver dans le chemin d’accès.

## <a name="download-the-event-hubs-components"></a>Télécharger les composants des concentrateurs d’événements

Télécharger les spouts et bolts Event Hubs à partir de [https://github.com/hdinsight/mvn-repo/raw/master/org/apache/storm/storm-eventhubs/1.1.0.1/storm-eventhubs-1.1.0.1.jar](https://github.com/hdinsight/mvn-repo/raw/master/org/apache/storm/storm-eventhubs/1.1.0.1/storm-eventhubs-1.1.0.1.jar).

Créez un répertoire nommé `eventhubspout` et enregistrez le fichier dans le répertoire.

## <a name="configure-event-hubs"></a>Configurer les hubs d’événement

Event Hubs est la source de données pour cet exemple. Utilisez les informations contenues dans la section « Création d’un espace de noms de concentrateurs d’événements et d’un concentrateur d’événements » du document [Bien démarrer avec l’envoi de messages vers des concentrateurs d’événements Azure dans .NET Standard](../../event-hubs/event-hubs-create.md).

1. Après avoir créé le hub d’événements, affichez les paramètres **EventHub** dans le portail Azure et sélectionnez **Stratégies d’accès partagé** . Sélectionnez **+ Ajouter** pour créer les stratégies suivantes :

   | Name | Autorisations |
   | --- | --- |
   | writer |Envoyer |
   | reader |Écouter |

    ![Capture d’écran de la fenêtre de stratégies d’accès partagé](./media/apache-storm-develop-csharp-event-hub-topology/share-access-policies.png)

2. Sélectionnez les stratégies **reader** et **writer** . Copiez et enregistrez la valeur de clé primaire des deux stratégies, car elles sont utilisées ultérieurement.

## <a name="configure-the-eventhubwriter"></a>Configuration de l’EventHubWriter

1. Si vous n’avez pas encore installé la dernière version des outils HDInsight pour Visual Studio, consultez [Bien démarrer avec les outils HDInsight pour Visual Studio](../hadoop/apache-hadoop-visual-studio-tools-get-started.md).

2. Téléchargez la solution depuis [eventhub-storm-hybrid](https://github.com/Azure-Samples/hdinsight-dotnet-java-storm-eventhub).

3. Ouvrez **EventHubExample.sln** . Dans le projet **EventHubWriter** , ouvrez le fichier **App.config** . Utilisez les informations à partir du concentrateur d’événements que vous avez configuré précédemment pour renseigner la valeur des clés suivantes :

   | Clé | Valeur |
   | --- | --- |
   | EventHubPolicyName |writer (si vous avez utilisé un nom différent pour la stratégie avec l’autorisation *Send* , utilisez-le à la place.) |
   | EventHubPolicyKey |La clé de la stratégie writer. |
   | eventHubNamespace |L’espace de noms qui contient votre concentrateur d’événements. |
   | eventHubName |Le nom de votre concentrateur d’événements. |
   | EventHubPartitionCount |Le nombre de partitions dans votre concentrateur d’événements. |

4. Enregistrez et fermez le fichier **App.config** .

## <a name="configure-the-eventhubreader"></a>Configuration de l’EventHubReader

1. Ouvrez le projet **EventHubReader** .

2. Ouvrez le fichier **App.config** pour **EventHubReader** . Utilisez les informations à partir du concentrateur d’événements que vous avez configuré précédemment pour renseigner la valeur des clés suivantes :

   | Clé | Valeur |
   | --- | --- |
   | EventHubPolicyName |reader (si vous avez utilisé un nom différent pour la stratégie avec l’autorisation *listen* , utilisez-le à la place.) |
   | EventHubPolicyKey |La clé de la stratégie reader. |
   | eventHubNamespace |L’espace de noms qui contient votre concentrateur d’événements. |
   | eventHubName |Le nom de votre concentrateur d’événements. |
   | EventHubPartitionCount |Le nombre de partitions dans votre concentrateur d’événements. |

3. Enregistrez et fermez le fichier **App.config** .

## <a name="deploy-the-topologies"></a>Déploiement des topologies

1. Dans **l’Explorateur de solutions** , cliquez avec le bouton droit de la souris sur le projet **EventHubReader** et sélectionnez **Envoyer à Storm sur HDInsight** .

    ![Capture d’écran de l’Explorateur de solutions, avec l’option Envoyer à Storm sur HDInsight mise en surbrillance](./media/apache-storm-develop-csharp-event-hub-topology/submit-to-apache-storm.png)

2. Dans la boîte de dialogue **Envoyer la topologie** , sélectionnez votre **cluster Storm** . Développez **Configurations supplémentaires** , sélectionnez **Chemins d’accès aux fichiers Java** , sélectionnez **...** , puis choisissez le répertoire qui contient le fichier JAR que vous avez téléchargé précédemment. Pour finir, cliquez sur **Envoyer** .

    ![Capture d’écran de la boîte de dialogue Envoyer la topologie](./media/apache-storm-develop-csharp-event-hub-topology/submit-storm-topology.png)

3. Une fois la topologie envoyée, la **Visionneuse de topologies Storm** apparaît. Sélectionnez la topologie **EventHubReader** dans le volet gauche pour afficher des informations sur la topologie.

    ![Capture d’écran de la visionneuse de topologies Storm](./media/apache-storm-develop-csharp-event-hub-topology/storm-topology-viewer.png)

4. Dans **l’Explorateur de solutions** , cliquez avec le bouton droit de la souris sur le projet **EventHubWriter** et sélectionnez **Envoyer à Storm sur HDInsight** .

5. Dans la boîte de dialogue **Envoyer la topologie** , sélectionnez votre **cluster Storm** . Développez **Configurations supplémentaires** , sélectionnez **Chemins d’accès aux fichiers Java** , sélectionnez **...** , puis choisissez le répertoire qui contient le fichier JAR que vous avez téléchargé précédemment. Pour finir, cliquez sur **Envoyer** .

6. Une fois la topologie envoyée, actualisez la liste des topologies dans la **Visionneuse de topologies Storm** pour vérifier que les deux topologies sont en cours d’exécution sur le cluster.

7. Dans la **Visionneuse de topologies Storm** , sélectionnez la topologie **EventHubReader** .

8. Pour ouvrir le résumé du composant associé au bolt, double-cliquez sur le composant **LogBolt** dans le diagramme.

9. Dans la section **Exécuteurs** , sélectionnez un des liens dans la colonne **Port** . Cela affiche les informations enregistrées par le composant. Les informations enregistrées sont similaires au texte suivant :

    ```output
    2017-03-02 14:51:29.255 m.s.p.TaskHost [INFO] Received C# STDOUT: 2017-03-02 14:51:29,255 [1] INFO  EventHubReader_LogBolt [(null)] - Received data: {"deviceValue":1830978598,"deviceId":"8566ccbc-034d-45db-883d-d8a31f34068e"}
    2017-03-02 14:51:29.283 m.s.p.TaskHost [INFO] Received C# STDOUT: 2017-03-02 14:51:29,283 [1] INFO  EventHubReader_LogBolt [(null)] - Received data: {"deviceValue":1756413275,"deviceId":"647a5eff-823d-482f-a8b4-b95b35ae570b"}
    2017-03-02 14:51:29.313 m.s.p.TaskHost [INFO] Received C# STDOUT: 2017-03-02 14:51:29,312 [1] INFO  EventHubReader_LogBolt [(null)] - Received data: {"deviceValue":1108478910,"deviceId":"206a68fa-8264-4d61-9100-bfdb68ee8f0a"}
    ```

## <a name="stop-the-topologies"></a>Arrêt des topologies

Pour arrêter les topologies, sélectionnez chaque topologie dans la **Visionneuse de topologies Storm** , puis cliquez sur **Supprimer** .

![Capture d’écran de la visionneuse de topologies Storm avec le bouton Supprimer mis en surbrillance](./media/apache-storm-develop-csharp-event-hub-topology/kill-storm-topology1.png)

## <a name="delete-your-cluster"></a>Supprimer votre cluster

[!INCLUDE [delete-cluster-warning](../../../includes/hdinsight-delete-cluster-warning.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce document, vous avez vu comment utiliser le spout et le bolt du hub d’événements Java à partir d’une topologie C# pour utiliser des données tirées d’Azure Event Hubs. Pour en savoir plus sur la création de topologies C#, consultez les documents suivants :

* [Développement de topologies C# pour Apache Storm dans HDInsight à l'aide de Visual Studio](apache-storm-develop-csharp-visual-studio-topology.md)
* [Guide de programmation SCP](apache-storm-scp-programming-guide.md)
* [Exemples de topologies pour Apache Storm dans HDInsight](apache-storm-example-topology.md)