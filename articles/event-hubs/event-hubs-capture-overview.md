---
title: Capturer des événements de streaming - Azure Event Hubs | Microsoft Docs
description: Cet article fournit une vue d’ensemble de la fonctionnalité Capture qui vous permet de capturer un streaming d’événements avec Azure Event Hubs.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: 32741fdd98ec79c38568ff5a6c4fa476f27d794b
ms.sourcegitcommit: d76108b476259fe3f5f20a91ed2c237c1577df14
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92912614"
---
# <a name="capture-events-through-azure-event-hubs-in-azure-blob-storage-or-azure-data-lake-storage"></a>Capturer des événements avec Azure Event Hubs dans le stockage Blob Azure ou Azure Data Lake Storage
Azure Event Hubs vous permet de capturer automatiquement les données de streaming dans Event Hubs dans un compte [Stockage Blob Azure](https://azure.microsoft.com/services/storage/blobs/) ou [Azure Data Lake Store Gen 1 ou Gen 2](https://azure.microsoft.com/services/data-lake-store/) de votre choix, tout en vous permettant de spécifier un intervalle de temps ou de taille. La configuration de l’outil Capture est rapide : il n’existe aucun coût d’administration pour son exécution et il s’adapte automatiquement à vos [unités de débit](event-hubs-scalability.md#throughput-units) Event Hubs. Event Hubs Capture représente le moyen le plus simple de charger les données pour la diffusion en continu dans Azure et vous permet de vous concentrer sur le traitement des données plutôt que sur la capture de données.

> [!NOTE]
> La configuration d’Event Hubs Capture pour utiliser Azure Data Lake Storage **Gen 2** est identique à celle pour utiliser un stockage de blobs Azure. Pour plus d’informations, consultez [Configurer Event Hubs Capture](event-hubs-capture-enable-through-portal.md). 

Event Hubs Capture vous permet de traiter des pipelines basés sur des lots et en temps réel sur le même flux. Cela vous permet de créer des solutions capables d’évoluer avec vos besoins au fil du temps. Que vous créiez des systèmes basés sur des lots dès aujourd’hui en pensant au traitement en temps réel à l’avenir, ou que vous souhaitiez ajouter un chemin à froid efficace vers une solution existante en temps réel, Event Hubs Capture facilite la tâche avec les données diffusées en continu.

> [!IMPORTANT]
> Le compte de stockage de destination (Stockage Azure ou Azure Data Lake Storage) doit faire partie du même abonnement que le hub d’événements. 

## <a name="how-event-hubs-capture-works"></a>Fonctionnement d’Azure Event Hubs Capture

Event Hubs est une mémoire tampon durable de rétention temporelle pour l’entrée de télémétrie, similaire à un journal distribué. La clé de la mise à l’échelle dans Event Hubs est le [modèle de consommateur partitionné](event-hubs-scalability.md#partitions). Chaque partition est un segment de données indépendant, et est utilisée de manière indépendante. Au fil du temps ces données vieillissent, en fonction de la période de rétention configurable. Par conséquent, un hub d’événements donné n’est jamais « saturé ».

Event Hubs Capture vous permet de spécifier vos propres compte et conteneur Stockage Blob Azure, ou compte Azure Data Lake Storage, qui sont utilisés pour stocker les données capturées. Ces comptes peuvent se trouver dans la même région que votre hub d’événements ou dans une autre région, ce qui ajoute à la flexibilité de la fonctionnalité Event Hubs Capture.

Les données capturées sont écrites au format [Apache Avro][Apache Avro] : un format compact, rapide et binaire qui fournit des structures de données riches avec un schéma inclus. Ce format est largement utilisé dans l’écosystème Hadoop, Stream Analytics et Azure Data Factory. Vous trouverez plus d’informations sur l’utilisation d’Avro plus loin dans cet article.

### <a name="capture-windowing"></a>Fenêtrage de Capture

Event Hubs Capture vous permet de configurer une fenêtre de temps pour le contrôle de la capture. Cette fenêtre présente une taille et une configuration temporelle minimales avec une « stratégie de premier gagnant », ce qui signifie que le premier déclencheur rencontré entraîne une opération de capture. Si vous avez une fenêtre de capture de quinze minutes et de 100 Mo, et si vous envoyez 1 Mo par seconde, la fenêtre de taille se déclenche avant la fenêtre temporelle. Chaque partition capture indépendamment et écrit un objet blob de bloc complet au moment de la capture, nommé d’après l’heure à laquelle l’intervalle de capture a été rencontré. La convention d’affectation de noms de stockage est la suivante :

```
{Namespace}/{EventHub}/{PartitionId}/{Year}/{Month}/{Day}/{Hour}/{Minute}/{Second}
```

Les valeurs de date contiennent des zéros. Par exemple :

```
https://mystorageaccount.blob.core.windows.net/mycontainer/mynamespace/myeventhub/0/2017/12/08/03/03/17.avro
```

Si votre objet blob de stockage Azure est temporairement indisponible, Event Hubs Capture conservera vos données pendant la période de rétention de données configurée sur votre Event Hub et remplira les données une fois que votre compte de stockage sera à nouveau disponible.

### <a name="scaling-to-throughput-units"></a>Mise à l’échelle des unités de débit

Le trafic Event Hubs est contrôlé par les [unités de débit](event-hubs-scalability.md#throughput-units). Une unité de débit autorise 1 Mo/s ou 1 000 événements par seconde d’entrée et 2 000 événements par seconde de sortie. Les concentrateurs d’événements Standard peuvent être configurés avec 1 à 20 unités de débit et vous pouvez en acheter d’autres en soumettant une [demande de support][support request] d’augmentation de quota. L’utilisation au-delà des unités de débit que vous avez achetées est limitée. Event Hubs Capture copie les données directement depuis le stockage Event Hubs interne, en contournant les quotas de sortie des unités de débit et en enregistrant votre sortie pour d’autres lecteurs de traitement tels que Stream Analytics ou Spark.

Une fois configuré, Event Hubs Capture s’exécute automatiquement lorsque vous envoyez votre premier événement et continue de s’exécuter. Pour que votre traitement en aval sache plus facilement que le processus fonctionne, les Event Hubs écrivent des fichiers vides en l’absence de données. Ce processus fournit une cadence prévisible et un marqueur qui peuvent alimenter vos processeurs de traitement par lots.

## <a name="setting-up-event-hubs-capture"></a>Configuration de l’outil Event Hubs Capture

Vous pouvez configurer la fonctionnalité Capture lors de la création du concentrateur d’événements, à l’aide du [portail Azure](https://portal.azure.com) ou des modèles Azure Resource Manager. Pour plus d’informations, consultez les articles suivants :

- [Activer Event Hubs Capture à l’aide du portail Azure](event-hubs-capture-enable-through-portal.md)
- [Créer un espace de noms Event Hubs avec un concentrateur d’événements et activer Capture à l’aide d’un modèle Azure Resource Manager](event-hubs-resource-manager-namespace-event-hub-enable-capture.md)

> [!NOTE]
> Si elle est activée pour un hub d’événements existant, la fonctionnalité Capture capture les événements qui arrivent au hub **après** qu’elle a été activée. Ceux qui se trouvaient dans le hub avant l’activation de la fonctionnalité ne sont pas capturés. 

## <a name="exploring-the-captured-files-and-working-with-avro"></a>Exploration des fichiers capturés et utilisation d’Avro

Event Hubs Capture crée des fichiers au format Avro, tel que spécifié dans la fenêtre de temps configurée. Vous pouvez afficher ces fichiers à l’aide de n’importe quel outil tel que [l’Explorateur de stockage Azure][Azure Storage Explorer]. Vous pouvez télécharger les fichiers localement pour les utiliser.

Les fichiers générés par Event Hubs Capture présentent le schéma Avro suivant :

![Schéma Avro][3]

Un moyen facile d’explorer les fichiers Avro consiste à utiliser la boîte à [outils Avro][Avro Tools] d’Apache. Vous pouvez également utiliser [Apache Drill][Apache Drill] pour bénéficier d’une expérience légère pilotée par SQL ou [Apache Spark][Apache Spark] pour effectuer un traitement distribué complexe sur les données ingérées. 

### <a name="use-apache-drill"></a>Utiliser Apache Drill

[Apache Drill][Apache Drill] est un « moteur de requête SQL open source pour l’exploration de Big Data » capable d’interroger des données structurées et semi-structurées quel que soit leur emplacement. Le moteur peut s’exécuter en tant que nœud autonome ou en tant que cluster énorme pour bénéficier d’excellentes performances.

Une prise en charge native de Stockage Blob Azure est disponible, ce qui facilite l’interrogation des données dans un fichier Avro, comme décrit dans la documentation :

[Apache Drill : plugin Stockage Blob Azure][Apache Drill: Azure Blob Storage Plugin]

Pour interroger facilement des fichiers capturés, vous pouvez créer et exécuter une machine virtuelle avec Apache Drill activé via un conteneur pour accéder au Stockage Blob Azure :

https://github.com/yorek/apache-drill-azure-blob

Un exemple complet de bout en bout est disponible dans le dépôt Streaming at Scale :

[Streaming at Scale : Event Hubs Capture]

### <a name="use-apache-spark"></a>Utiliser Apache Spark

[Apache Spark][Apache Spark] est un « moteur d’analytique unifié pour le traitement des données à grande échelle ». Il prend en charge différents langages, notamment SQL, et vous pouvez accéder facilement au Stockage Blob Azure. Il existe quelques options pour exécuter Apache Spark dans Azure, et chacune fournit un accès facile au Stockage Blob Azure :

- [HDInsight : adressage des fichiers dans le stockage Azure][HDInsight: Address files in Azure storage]
- [Azure Databricks : Stockage Blob Azure][Azure Databricks: Azure Blob Storage]
- [Azure Kubernetes Service](../aks/spark-job.md) 

### <a name="use-avro-tools"></a>Utiliser les outils Avro

Les [outils Avro][Avro Tools] sont disponibles sous la forme d’un package jar. Après avoir téléchargé le fichier jar, vous pouvez voir le schéma d’un fichier Avro spécifique en exécutant la commande suivante :

```shell
java -jar avro-tools-1.9.1.jar getschema <name of capture file>
```

Cette commande renvoie

```json
{

    "type":"record",
    "name":"EventData",
    "namespace":"Microsoft.ServiceBus.Messaging",
    "fields":[
                 {"name":"SequenceNumber","type":"long"},
                 {"name":"Offset","type":"string"},
                 {"name":"EnqueuedTimeUtc","type":"string"},
                 {"name":"SystemProperties","type":{"type":"map","values":["long","double","string","bytes"]}},
                 {"name":"Properties","type":{"type":"map","values":["long","double","string","bytes"]}},
                 {"name":"Body","type":["null","bytes"]}
             ]
}
```

Vous pouvez également utiliser les outils Avro pour convertir le fichier au format JSON et effectuer d’autres traitements.

Pour effectuer un traitement plus avancé, téléchargez et installez Avro pour la plateforme de votre choix. Au moment de la rédaction de cet article, les implémentations sont disponibles pour C, C++, C\#, Java, NodeJS, Perl, PHP, Python et Ruby.

Apache Avro propose des guides de mise en route complets pour [Java][Java]et [Python][Python]. Vous pouvez également lire l’article [Prise en main d’Event Hubs Capture](event-hubs-capture-python.md).

## <a name="how-event-hubs-capture-is-charged"></a>Chargement d’Azure Event Hubs Capture

Event Hubs Capture est mesuré de la même façon que les unités de débit, au tarif horaire. La facturation est directement proportionnelle au nombre d’unités de débit achetées pour l’espace de noms. En même temps que les unités de débit augmentent et diminuent, Event Hubs Capture augmente et diminue pour fournir des performances adaptées. Les compteurs se produisent en même temps. Pour plus d’informations sur les prix appliqués, consultez [Tarification d’Event Hubs](https://azure.microsoft.com/pricing/details/event-hubs/). 

Capture n’utilise pas de quota de sortie puisque la fonctionnalité est facturée séparément. 

## <a name="integration-with-event-grid"></a>Intégration à Event Grid 

Vous pouvez créer un abonnement Azure Event Grid en utilisant un espace de noms Event Hubs comme source. Le tutoriel suivant vous montre comment créer un abonnement Event Grid avec un hub d’événements comme source et une application Azure Functions comme récepteur : [Traiter et migrer les données Event Hubs capturées vers Azure Synapse Analytics à l’aide d’Event Grid et d’Azure Functions](store-captured-data-data-warehouse.md).

## <a name="next-steps"></a>Étapes suivantes
Event Hubs Capture est la solution la plus simple pour charger des données dans Azure. À l’aide d’Azure Data Lake, d’Azure Data Factory et d’Azure HDInsight, vous pouvez effectuer un traitement par lots, ainsi que d’autres analyses en utilisant des outils et des plateformes de votre choix, à l’échelle requise.

Découvrez comment activer cette fonctionnalité avec le portail Azure et un modèle Azure Resource Manager :

- [Utiliser le portail Azure pour activer la capture Event Hubs](event-hubs-capture-enable-through-portal.md)
- [Utiliser un modèle Resource Manager pour activer Event Hubs Capture](event-hubs-resource-manager-namespace-event-hub-enable-capture.md)


[Apache Avro]: https://avro.apache.org/
[Apache Drill]: https://drill.apache.org/
[Apache Spark]: https://spark.apache.org/
[support request]: https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade
[Azure Storage Explorer]: https://github.com/microsoft/AzureStorageExplorer/releases
[3]: ./media/event-hubs-capture-overview/event-hubs-capture3.png
[Avro Tools]: https://downloads.apache.org/avro/stable/java/
[Java]: https://avro.apache.org/docs/current/gettingstartedjava.html
[Python]: https://avro.apache.org/docs/current/gettingstartedpython.html
[Event Hubs overview]: ./event-hubs-about.md
[HDInsight: Address files in Azure storage]:https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-use-blob-storage
[Azure Databricks: Azure Blob Storage]:https://docs.databricks.com/spark/latest/data-sources/azure/azure-storage.html
[Apache Drill: Azure Blob Storage Plugin]:https://drill.apache.org/docs/azure-blob-storage-plugin/
[Streaming at Scale : Event Hubs Capture]:https://github.com/yorek/streaming-at-scale/tree/master/event-hubs-capture
