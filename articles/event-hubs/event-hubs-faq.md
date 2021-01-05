---
title: Forum Aux Questions (FAQ) - Azure Event Hubs | Microsoft Docs
description: Cet article contient une liste des questions fréquemment posées (FAQ) sur Azure Event Hubs, ainsi que leurs réponses.
ms.topic: article
ms.date: 10/27/2020
ms.openlocfilehash: c756d0bccd9b2ad303bd97d3bfb7aed8b0b82b09
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "96002786"
---
# <a name="event-hubs-frequently-asked-questions"></a>Forum Aux Questions (FAQ) sur Event Hubs

## <a name="general"></a>Général

### <a name="what-is-an-event-hubs-namespace"></a>Qu’est-ce qu’un espace de noms Event Hubs ?
Un espace de noms est un conteneur d’étendue pour les rubriques Event Hub/Kafka. Il vous donne une valeur unique de [nom de domaine complet](https://en.wikipedia.org/wiki/Fully_qualified_domain_name). Un espace de noms constitue un conteneur d’application qui peut héberger plusieurs rubriques Event Hub/Kafka. 

### <a name="when-do-i-create-a-new-namespace-vs-use-an-existing-namespace"></a>À quel moment puis-je créer un espace de noms plutôt que d’en utiliser un ?
Les allocations de capacité ([unités de débit (TU)](#throughput-units)) sont facturées au niveau de l’espace de noms. Un espace de noms est également associé à une région.

Peut-être préférerez-vous créer un espace de noms plutôt que d’en utiliser un dans l’un des scénarios suivants : 

- Vous avez besoin d’un Event Hub associé à une nouvelle région.
- Vous avez besoin d’un Event Hub associé à un autre abonnement.
- Vous avez besoin d’un Event Hub avec une allocation de capacité distincte (autrement dit, la capacité nécessaire pour l’espace de noms avec l’Event Hub ajouté dépasse le seuil de 40 TU et vous ne souhaitez pas utiliser le cluster dédié).  

### <a name="what-is-the-difference-between-event-hubs-basic-and-standard-tiers"></a>Quelle est la différence entre les niveaux De Base et Standard pour les hubs d’événements ?

Le niveau Standard d’Azure Event Hubs fournit des fonctionnalités supérieures à celles disponibles dans le niveau de base. Les fonctionnalités suivantes sont incluses dans le niveau Standard :

* Rétention plus longue des événements
* Connexions réparties supplémentaires, avec frais supplémentaires pour le dépassement du nombre inclus
* Plusieurs [groupes de consommateurs](event-hubs-features.md#consumer-groups)
* [Capture](event-hubs-capture-overview.md)
* [Intégration de Kafka](event-hubs-for-kafka-ecosystem-overview.md)

Pour plus d’informations sur les niveaux tarifaires, notamment pour Event Hubs Dedicated, consultez la rubrique [Informations tarifaires pour Event Hubs](https://azure.microsoft.com/pricing/details/event-hubs/).

### <a name="where-is-azure-event-hubs-available"></a>Où Azure Event Hubs est-il disponible ?

Azure Event Hubs est disponible dans toutes les régions Azure prises en charge. Pour obtenir une liste, visitez la page [Régions Azure](https://azure.microsoft.com/regions/).  

### <a name="can-i-use-a-single-advanced-message-queuing-protocol-amqp-connection-to-send-and-receive-from-multiple-event-hubs"></a>Puis-je utiliser une connexion AMQP unique pour l’envoi et la réception à partir de plusieurs Event Hubs ?

Oui, à condition que la totalité des concentrateurs d’événements se trouve dans le même espace de noms.

### <a name="what-is-the-maximum-retention-period-for-events"></a>Quelle est la période de rétention maximale pour les événements ?

Le niveau Standard des hubs d’événements prend actuellement en charge une période de rétention maximale de sept jours. Les hubs d’événements ne sont pas destinés à servir de magasins de données permanents. Les périodes de conservation de plus de 24 heures conviennent aux scénarios dans lesquels il est utile de pouvoir reproduire un flux d’événements sur les mêmes systèmes. Par exemple, pour entraîner ou tester un nouveau modèle Machine Learning sur des données existantes. Si vous avez besoin que la rétention des messages dure plus de sept jours, activez [Event Hubs Capture](event-hubs-capture-overview.md) dans votre hub d’événements pour en tirer (pull) les données et les importer dans le compte de stockage ou le compte Azure Data Lake Service de votre choix. L’activation de Capture est facturée en fonction des unités de débit achetées.

Vous pouvez configurer la période de conservation pour les données capturées sur votre compte de stockage. La fonctionnalité de **gestion du cycle de vie** du Stockage Azure fournit une stratégie complète basée sur des règles pour les comptes de stockage Blob et v2 universel. Utilisez la stratégie pour effectuer la transition de vos données vers les niveaux d’accès appropriés ou pour faire en sorte qu’elles expirent à la fin de leur cycle de vie. Pour plus d’informations, consultez [Gérer le cycle de vie du Stockage Blob Azure](../storage/blobs/storage-lifecycle-management-concepts.md). 

### <a name="how-do-i-monitor-my-event-hubs"></a>Comment puis-je surveiller mes Event Hubs ?
Event Hubs émet des métriques exhaustives qui fournissent l’état de vos ressources à [Azure Monitor](../azure-monitor/overview.md). Elles vous permettent également d’évaluer l’intégrité globale du service Event Hubs non seulement au niveau de l’espace de noms mais également au niveau de l’entité. En savoir plus sur la supervision proposée pour les [Azure Event Hubs](event-hubs-metrics-azure-monitor.md).

### <a name="where-does-azure-event-hubs-store-customer-data"></a><a name="in-region-data-residency"></a>Où le service Azure Event Hubs stocke-t-il des données client ?
Azure Event Hubs stocke les données client. Event Hubs stockant automatiquement ces données dans une seule région, ce service répond automatiquement aux exigences en matière de résidence des données de la région, y compris celles spécifiées dans le [Centre de gestion de la confidentialité](https://azuredatacentermap.azurewebsites.net/).

[!INCLUDE [event-hubs-connectivity](../../includes/event-hubs-connectivity.md)]

## <a name="apache-kafka-integration"></a>Intégration d’Apache Kafka

### <a name="how-do-i-integrate-my-existing-kafka-application-with-event-hubs"></a>Comment intégrer mon application Kafka existante dans Event Hubs ?
Event Hubs fournit un point de terminaison Kafka qui peut être utilisé par vos applications Apache Kafka. Pour bénéficier de l’expérience PaaS Kafka, il est nécessaire de modifier la configuration. Il fournit une alternative à l’exécution de votre cluster Kafka. Event Hubs prend en charge Apache Kafka 1.0 et les versions clientes plus récentes, et fonctionne avec vos applications, outils et frameworks Kafka. Pour plus d’informations, consultez [Dépôt Event Hubs pour Kafka](https://github.com/Azure/azure-event-hubs-for-kafka).

### <a name="what-configuration-changes-need-to-be-done-for-my-existing-application-to-talk-to-event-hubs"></a>Quels changements de configuration doivent être effectués pour que mon application existante puisse communiquer avec les Event Hubs ?
Pour vous connecter à un Event Hub, vous devez mettre à jour les configurations du client Kafka. Cela s’effectue en créant un espace de noms Event Hubs et en obtenant la [chaîne de connexion](event-hubs-get-connection-string.md). Modifiez le fichier bootstrap.servers pour qu’il pointe vers le FQDN Event Hubs et sur le port 9093. Mettez à jour le fichier sasl.jaas.config pour diriger le client Kafka vers votre point de terminaison Event Hubs (ce point de terminaison est la chaîne de connexion que vous avez obtenue), avec une authentification correcte comme indiqué ci-dessous :

```properties
bootstrap.servers={YOUR.EVENTHUBS.FQDN}:9093
request.timeout.ms=60000
security.protocol=SASL_SSL
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="{YOUR.EVENTHUBS.CONNECTION.STRING}";
```

Exemple :

```properties
bootstrap.servers=dummynamespace.servicebus.windows.net:9093
request.timeout.ms=60000
security.protocol=SASL_SSL
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="Endpoint=sb://dummynamespace.servicebus.windows.net/;SharedAccessKeyName=DummyAccessKeyName;SharedAccessKey=XXXXXXXXXXXXXXXXXXXXX";
```
Remarque : Si sasl.jaas.config n’est pas une configuration prise en charge dans votre framework, recherchez les configurations utilisées pour définir le nom d’utilisateur et le mot de passe SASL, et utilisez-les. Définissez le nom d’utilisateur sur $ConnectionString et le mot de passe sur votre chaîne de connexion Event Hubs.

### <a name="what-is-the-messageevent-size-for-event-hubs"></a>Quelle est la taille du message ou de l’événement pour les Event Hubs ?
La taille de message maximale autorisée pour les Event Hubs est de 1 Mo.

## <a name="throughput-units"></a>Unités de débit

### <a name="what-are-event-hubs-throughput-units"></a>Que sont les unités de débit des hubs d'événements ?
Le débit dans les Event Hubs définit la quantité de données en méga-octets ou le nombre (en milliers) d’événements de 1 Ko qui entrent et sortent par les Event Hubs. Ce débit est mesuré en unités de débit (TU). Achetez des unités de débit pour pouvoir utiliser le service Event Hubs. Vous pouvez sélectionner explicitement les unités de débit Event Hubs en utilisant soit le portail, soit les modèles Resource Manager Event Hubs. 


### <a name="do-throughput-units-apply-to-all-event-hubs-in-a-namespace"></a>Les unités de débit s’appliquent-elles à tous les hubs d’événements dans un espace de noms ?
Oui, les unités de débit s’appliquent à tous les hubs d’événements dans un espace de noms Event Hubs. Cela signifie que vous achetez des unités de débit au niveau de l’espace de noms et que vous les partagez entre les Event Hubs sous cet espace de noms. Chaque TU donne accès à l’espace de noms pour les fonctionnalités suivantes :

- Jusqu’à 1 Mo par seconde d’événements d’entrée (événements envoyés à un concentrateur d’événements), mais pas plus de 1 000 événements d’entrée, opérations de gestion ou appels d’API de contrôle par seconde.
- Jusqu’à 2 Mo par seconde d’événements de sortie (événements consommés à partir d’un hub d’événements), mais pas plus de 4 096 événements de sortie.
- Jusqu’à 84 Go de stockage d’événements (suffisant pour la période de rétention de 24 heures par défaut).

### <a name="how-are-throughput-units-billed"></a>Comment les unités de débit sont-elles facturées ?
Les unités de débit des hubs d’événements sont facturées par heure. La facturation est basée sur le nombre maximal d’unités sélectionné pendant l’heure donnée. 

### <a name="how-can-i-optimize-the-usage-on-my-throughput-units"></a>Comment puis-je optimiser l’utilisation des unités de débit ?
Vous pouvez commencer avec une seule unité de débit (TU) et activer [la majoration automatique](event-hubs-auto-inflate.md). La fonctionnalité de majoration automatique fonctionnalité vous permet d’augmenter le nombre d’unités de débit à mesure que votre trafic/charge utile augmente. Vous pouvez également définir une limite supérieure pour le nombre d’unités de débit.

### <a name="how-does-auto-inflate-feature-of-event-hubs-work"></a>Comment la fonctionnalité Majoration automatique Event Hubs opère-t-elle ?
La fonctionnalité de majoration automatique vous permet de mettre à l’échelle les unités de débit. Cela signifie que vous pouvez commencer par acheter des unités de débit basses et que la majoration automatique fait monter en puissance à mesure que votre entrée augmente. Il s’agit d’une option rentable qui vous donne le contrôle complet du nombre de TU à gérer. Cette fonctionnalité est une fonctionnalité de **montée en puissance uniquement**, mais vous pouvez contrôler la descente en puissance en modifiant le nombre de TU. 

Vous pouvez commencer avec un faible nombre d’unités de débit, par exemple, 2 TU. Si vous prévoyez que votre trafic peut atteindre 15 unités de débit, activez la fonctionnalité de majoration automatique sur votre espace de noms et fixez la limite maximale à 15 unités de débit. Vous pouvez maintenant augmenter le nombre de TU automatiquement dès que votre trafic augmente.

### <a name="is-there-a-cost-associated-when-i-turn-on-the-auto-inflate-feature"></a>L’activation de la fonctionnalité de majoration automatique est-elle payante ?
L’activation de cette fonctionnalité n’engendre **acun coût**. 

### <a name="how-are-throughput-limits-enforced"></a>Comment les limites d’unités de débit sont-elles appliquées ?
Si le débit d’**entrée** total ou le taux d’événements d’entrée total de tous les hubs d’événements d’un espace de noms dépassent les allocations d’unité de débit agrégées, les expéditeurs sont limités et reçoivent des erreurs indiquant que le quota d’entrée a été dépassé.

Si le débit de **sortie** total ou le taux d’événements de sortie total de tous les hubs d’événements d’un espace de noms dépassent les allocations d’unité de débit agrégées, les récepteurs sont limités, mais aucune erreur de limitation n’est générée. 

Les quotas d’entrée et de sortie sont appliqués séparément afin qu’aucun expéditeur n’entraîne un ralentissement de la consommation d’événements ni qu’aucun récepteur n’empêche l’envoi d’événements à un concentrateur d’événements.

### <a name="is-there-a-limit-on-the-number-of-throughput-units-that-can-be-reservedselected"></a>Y a-t-il une limite au nombre d’unités de débit qui peuvent être réservées/sélectionnées ?

Lorsque vous créez un espace de noms de niveau de base ou standard dans le portail Azure, vous pouvez sélectionner jusqu’à 20 unités pour l’espace de noms. Pour qu’il atteigne **exactement** 40 unités, envoyez une [demande de support](../azure-portal/supportability/how-to-create-azure-support-request.md).  

1. Sur la page **Espace de noms du bus d’événements**, sélectionnez **Nouvelle demande de support** dans le menu gauche. 
1. Sur la page **Nouvelle demande de support**, procédez comme suit :
    1. Pour **Résumé**, décrivez le problème en quelques mots. 
    1. Pour **Type de problème**, sélectionnez **Quota**. 
    1. Pour **Sous-type de problème**, sélectionnez **Demande d'augmentation ou de diminution du nombre d'unités de débit**. 
    
        :::image type="content" source="./media/event-hubs-faq/support-request-throughput-units.png" alt-text="Page de demande de support":::

Au-delà de 40 TU, Event Hubs propose le modèle basé sur les ressources/capacités appelé Clusters dédiés Event Hubs. Les clusters dédiés sont vendus en unités de capacité (CU). Pour plus d’informations, consultez [Event Hubs Dedicated - Vue d’ensemble](event-hubs-dedicated-overview.md).

## <a name="dedicated-clusters"></a>Clusters dédiés

### <a name="what-are-event-hubs-dedicated-clusters"></a>Que sont les clusters Event Hubs Dedicated ?
Les clusters Event Hubs Dedicated offrent des déploiements à locataire unique pour les clients les plus exigeants. Cette offre constitue un cluster basé sur la capacité qui n’est pas liée par des unités de débit. Cela signifie que vous pouvez utiliser le cluster pour ingérer et diffuser vos données en streaming en fonction de l’utilisation du processeur et de le mémoire du cluster. Pour plus d’informations, consultez [Clusters Event Hubs Dedicated](event-hubs-dedicated-overview.md).

### <a name="how-do-i-create-an-event-hubs-dedicated-cluster"></a>Comment créer un cluster Event Hubs Dedicated ?
Pour obtenir des instructions pas à pas et des informations supplémentaires sur la configuration d’un cluster dédié Event Hubs, consultez le [Démarrage rapide : Créer un cluster dédié Event Hubs à l’aide du portail Azure](event-hubs-dedicated-cluster-create-portal.md). 


[!INCLUDE [event-hubs-dedicated-clusters-faq](../../includes/event-hubs-dedicated-clusters-faq.md)]


## <a name="partitions"></a>Partitions

### <a name="how-many-partitions-do-i-need"></a>De combien de partitions ai-je besoin ?
Le nombre de partitions est spécifié lors de la création du concentrateur d’événements. Il doit être compris entre 1 et 32. Le nombre de partitions n’est pas modifiable. Lorsque vous le définissez, tenez compte de la mise à l’échelle sur le long terme. Les partitions constituent un mécanisme d’organisation des données. Elles sont liées au degré de parallélisme en aval requis lors de la consommation des applications. Le choix du nombre de partitions dans un concentrateur d’événements est directement lié au nombre de lecteurs simultanés que vous prévoyez d’avoir. Pour plus d’informations sur les partitions, consultez [Partitions](event-hubs-features.md#partitions).

Vous pouvez choisir la valeur la plus élevée possible, à savoir 32, au moment de la création. N’oubliez pas que si vous avez plusieurs partitions, les événements sont envoyés à plusieurs partitions sans conserver l’ordre, sauf si vous configurez les expéditeurs pour qu’ils envoient uniquement à une partition unique à partir de la partition 32 en laissant les 31 partitions restantes. Dans le premier cas, vous devrez lire les événements sur les 32 partitions. Dans le dernier cas, il n’y a pas de frais supplémentaires significatifs par rapport à la configuration supplémentaire que vous devez effectuer sur l’hôte du processeur d’événements.

Event Hubs est conçu pour autoriser un lecteur de partition unique par groupe de consommateurs. Dans la plupart des cas, le paramètre par défaut de quatre partitions est suffisant. Si vous comptez mettre à l’échelle le traitement des événements, vous souhaiterez peut-être ajouter des partitions supplémentaires. Il n’y a aucune limite de débit spécifique sur une partition, mais le débit cumulé dans votre espace de noms est limité par le nombre d’unités de débit. Lorsque vous augmentez le nombre d’unités de débit de votre espace de noms, vous souhaiterez peut-être avoir des partitions supplémentaires pour permettre aux lecteurs fonctionnant simultanément d’atteindre leur débit maximal.

Toutefois, si vous disposez d’un modèle dans lequel votre application a une affinité avec une partition particulière, l’augmentation du nombre de partitions pourrait n’offrir aucun avantage. Pour plus d’informations, consultez [Disponibilité et cohérence](event-hubs-availability-and-consistency.md).

### <a name="increase-partitions"></a>Augmenter les partitions
Vous pouvez demander à ce que le nombre de partitions soit augmenté à 40 (exact) en soumettant une demande de support. 

1. Sur la page **Espace de noms du bus d’événements**, sélectionnez **Nouvelle demande de support** dans le menu gauche. 
1. Sur la page **Nouvelle demande de support**, procédez comme suit :
    1. Pour **Résumé**, décrivez le problème en quelques mots. 
    1. Pour **Type de problème**, sélectionnez **Quota**. 
    1. Pour **Sous-type de problème**, sélectionnez **Demande de modification de partition**. 
    
        :::image type="content" source="./media/event-hubs-faq/support-request-increase-partitions.png" alt-text="Augmenter le nombre de partitions":::

Le nombre de partitions peut être augmenté à 40 exactement. Dans ce cas, le nombre d’unités doit également être augmenté à 40. Si vous décidez ultérieurement de réduire le nombre limite d’unités à <= 20, la limite maximale du nombre de partitions est également réduite à 32. 

La diminution des partitions n’affecte pas les hubs d’événements existants, car les partitions sont appliquées au niveau de l’Event Hub et elles sont immuables après la création du hub. 

## <a name="pricing"></a>Tarifs

### <a name="where-can-i-find-more-pricing-information"></a>Où puis-je obtenir des informations complémentaires sur la tarification ?

Pour des informations complètes sur la tarification des hubs d’événements, consultez la rubrique [Tarification des hubs d’événements](https://azure.microsoft.com/pricing/details/event-hubs/).

### <a name="is-there-a-charge-for-retaining-event-hubs-events-for-more-than-24-hours"></a>Existe-t-il des frais pour la rétention de plus de 24 heures des événements de hubs d'événements ?

Le niveau Standard Event Hubs permet de conserver les messages au-delà de 24 heures, avec un maximum de sept jours. Si la taille du nombre total d’événements stockés dépasse l’allocation de stockage pour le nombre d’unités de débit sélectionnées (84 Go par unité de débit), la taille excédentaire est facturée au tarif de stockage Blob Azure publié. L'allocation de stockage dans chaque unité de débit couvre tous les frais de stockage pour les périodes de rétention de 24 heures (par défaut), même si l'unité de débit est utilisée pour l'allocation d'entrée maximale.

### <a name="how-is-the-event-hubs-storage-size-calculated-and-charged"></a>Comment la taille de stockage des hubs d'événements est-elle calculée et facturée ?

La taille totale de tous les événements stockés, notamment de toute surcharge interne pour les en-têtes d’événements ou les structures de stockage sur disque dans tous les concentrateurs d’événements, est mesurée tout au long de la journée. À la fin de la journée, la taille maximale de stockage est calculée. L'allocation de stockage quotidienne est calculée en fonction du nombre minimal d'unités de débit qui ont été sélectionnées pendant la journée (chaque unité de débit fournit une allocation de 84 Go). Si la taille totale dépasse l'allocation de stockage quotidienne calculée, l'excédent de stockage est facturé selon les taux de stockage Azure Blob (au taux **Stockage localement redondant** ).

### <a name="how-are-event-hubs-ingress-events-calculated"></a>Comment les événements d'entrée des hubs d'événements sont-ils calculés ?

Chaque événement envoyé à un concentrateur d’événements est considéré comme un message facturable. Un *événement d’entrée* est défini comme une unité de données inférieure ou égale à 64 Ko. Tout événement dont la taille est inférieure ou égale à 64 Ko est considéré comme un événement facturable. Si l’événement est supérieur à 64 Ko, le nombre d’événements facturables est calculé en fonction de la taille de l’événement, par multiples de 64 Ko. Par exemple, un événement de 8 Ko envoyé au concentrateur d’événements est facturé comme un événement, mais un message de 96 Ko envoyé au concentrateur d’événements est facturé comme deux événements.

Les événements utilisés à partir d’un concentrateur d’événements, ainsi que les opérations de gestion et les appels de contrôle comme les points de contrôle, ne sont pas comptabilisés comme des événements d’entrée facturables, mais s’ajoutent à l’allocation d’unité de débit.

### <a name="do-brokered-connection-charges-apply-to-event-hubs"></a>Des frais de connexion répartie s'appliquent-ils aux hubs d'événements ?

Des frais de connexion s'appliquent uniquement lorsque le protocole AMQP est utilisé. Il n'y a aucun frais de connexion pour l'envoi d'événements à l'aide de HTTP, quel que soit le nombre de systèmes ou de périphériques d’envoi. Si vous prévoyez d’utiliser AMQP (par exemple, pour améliorer le flux d’événements ou activer la communication bidirectionnelle sur des scénarios de commande et de contrôle IoT), reportez-vous à la page [Informations tarifaires pour Event Hubs](https://azure.microsoft.com/pricing/details/event-hubs/) pour plus de détails sur le nombre de connexions incluses dans chaque niveau de service.

### <a name="how-is-event-hubs-capture-billed"></a>Comment est facturé Event Hubs Capture ?

Capture est activé lorsque la fonctionnalité de capture est activée pour chaque concentrateur d’événements de l’espace de noms. Event Hubs Capture est facturé à l’heure, par unité de débit achetée. La facturation d’Event Hubs Capture reflète les variations du nombre d’unités de débit sous forme d’incréments d’heure pleine. Pour plus d’informations sur la facturation d’Event Hubs Capture, consultez [Informations tarifaires pour Event Hubs](https://azure.microsoft.com/pricing/details/event-hubs/).

### <a name="do-i-get-billed-for-the-storage-account-i-select-for-event-hubs-capture"></a>Suis-je facturé pour le compte de stockage que je sélectionne pour Event Hubs Capture ?

Capture utilise un compte de stockage que vous fournissez lorsqu’il est activé sur un concentrateur d’événements. Ce compte de stockage étant le vôtre, les éventuels changements de configurations seront facturés sur votre abonnement Azure.

## <a name="quotas"></a>Quotas

### <a name="are-there-any-quotas-associated-with-event-hubs"></a>Y a-t-il des quotas associés aux Event Hubs ?

Pour obtenir la liste de tous les quotas d’Event Hubs, consultez la page [quotas](event-hubs-quotas.md).

## <a name="troubleshooting"></a>Dépannage

### <a name="why-am-i-not-able-to-create-a-namespace-after-deleting-it-from-another-subscription"></a>Pourquoi ne suis-je pas en mesure de créer un espace de noms après l'avoir supprimé d'un autre abonnement ? 
Lorsque vous supprimez un espace de noms d’un abonnement, patientez pendant 4 heures avant de le recréer avec le même nom dans un autre abonnement. Sinon, le message d’erreur suivant peut s’afficher : `Namespace already exists`. 

### <a name="what-are-some-of-the-exceptions-generated-by-event-hubs-and-their-suggested-actions"></a>Quelles sont les exceptions générées par Event Hubs et les actions recommandées ?

Pour obtenir la liste des exceptions Event Hubs potentielles, consultez la page [Vue d’ensemble des exceptions](event-hubs-messaging-exceptions.md).

### <a name="diagnostic-logs"></a>Journaux de diagnostic

Event Hubs prend en charge deux types de [journaux de diagnostics](event-hubs-diagnostic-logs.md) (journaux d’activité d’erreurs Capture et journaux d’activité des opérations), qui sont tous deux représentés en json et peuvent être activés via le portail Azure.

### <a name="support-and-sla"></a>Prise en charge et contrats SLA

Un support technique pour Event Hubs est disponible via la [page de questions Microsoft Q&A sur Azure Service Bus](/answers/topics/azure-service-bus.html). La gestion de la facturation et des abonnements est fournie gratuitement.

Pour en savoir plus sur notre contrat SLA, consultez la section [Contrats de niveau de Service](https://azure.microsoft.com/support/legal/sla/) .

## <a name="azure-stack-hub"></a>Azure Stack Hub

### <a name="how-can-i-target-a-specific-version-of-azure-storage-sdk-when-using-azure-blob-storage-as-a-checkpoint-store"></a>Comment cibler une version spécifique du Kit de développement logiciel (SDK) Stockage Azure lors de l’utilisation de Stockage Blob Azure comme magasin de points de contrôle ?
Si vous exécutez ce code sur Azure Stack Hub, vous rencontrerez des erreurs d’exécution, sauf si vous ciblez une version spécifique de l’API Stockage. Ceci est dû au fait que le SDK Event Hubs utilise la dernière API Stockage Azure disponible dans Azure, qui peut ne pas être pas disponible sur votre plateforme Azure Stack Hub. Azure Stack Hub peut prendre en charge une autre version du Kit de développement logiciel (SDK) Stockage Blob que celles généralement disponibles sur Azure. Si vous utilisez Stockage Blob Azure comme magasin de points de contrôle, vérifiez la [version de l’API Stockage Azure prise en charge pour votre build Azure Stack Hub](/azure-stack/user/azure-stack-acs-differences?#api-version) et ciblez cette version dans votre code. 

Par exemple, si vous exécutez sur Azure Stack Hub version 2005, la version la plus élevée disponible pour le service Stockage est la version 2019-02-02. Par défaut, la bibliothèque de client du SDK Event Hubs utilise la version la plus récente disponible sur Azure (2019-07-07 au moment de la publication du SDK). Dans ce cas, en plus des étapes suivantes de cette section, vous devez également ajouter du code pour cibler la version 2019-02-02 de l’API du service Stockage. Pour obtenir un exemple sur la façon de cibler une version spécifique de l’API de stockage, consultez les exemples suivants pour les langages C#, Java, Python et JavaScript/TypeScript.  

Pour obtenir un exemple sur la façon de cibler une version spécifique de l’API de stockage à partir de votre code, consultez les exemples suivants sur GitHub : 

- [.NET](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/eventhub/Azure.Messaging.EventHubs.Processor/samples/)
- [Java](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/eventhubs/azure-messaging-eventhubs-checkpointstore-blob/src/samples/java/com/azure/messaging/eventhubs/checkpointstore/blob/EventProcessorWithCustomStorageVersion.java)
- Python : [synchrone](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/eventhub/azure-eventhub-checkpointstoreblob/samples/receive_events_using_checkpoint_store_storage_api_version.py), [asynchrone](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/eventhub/azure-eventhub-checkpointstoreblob-aio/samples/receive_events_using_checkpoint_store_storage_api_version_async.py)
- [JavaScript](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/eventhub/eventhubs-checkpointstore-blob/samples/javascript/receiveEventsWithApiSpecificStorage.js) et [TypeScript](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/eventhub/eventhubs-checkpointstore-blob/samples/typescript/src/receiveEventsWithApiSpecificStorage.ts)

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez en apprendre plus sur Event Hubs en consultant les liens suivants :

* [Vue d’ensemble d’Event Hubs](./event-hubs-about.md)
* [Créer un hub d’événements](event-hubs-create.md)
* [Majoration automatique Event Hubs](event-hubs-auto-inflate.md)
