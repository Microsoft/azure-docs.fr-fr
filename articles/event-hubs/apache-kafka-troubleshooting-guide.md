---
title: Résoudre les problèmes liés à Azure Event Hubs pour Apache Kafka
description: Cet article explique comment résoudre les problèmes liés à Azure Event Hubs pour Apache Kafka
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: e32e02947b9f004755381d562fd3f3c897b70674
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90061425"
---
# <a name="apache-kafka-troubleshooting-guide-for-event-hubs"></a>Guide de dépannage Apache Kafka pour Event Hubs
Cet article fournit des conseils de dépannage pour les problèmes que vous pouvez rencontrer lors de l’utilisation d’Event Hubs pour Apache Kafka. 

## <a name="server-busy-exception"></a>Exception de serveur occupé
Vous pouvez recevoir une exception de serveur occupé en raison de la limitation Kafka. Avec les clients AMQP, Event Hubs retourne immédiatement une exception de **serveur occupé** lors de la limitation du service. Cela équivaut à un message « réessayer plus tard ». Dans Kafka, les messages sont retardés avant d’être terminés. La longueur du délai est retournée en millisecondes au format `throttle_time_ms` dans la réponse de production/récupération. Dans la plupart des cas, ces requêtes retardées ne sont pas consignées en tant qu’exceptions de serveur occupé dans les tableaux de bord Event Hubs. Au lieu de cela, la valeur `throttle_time_ms` de la réponse doit être utilisée comme un indicateur que le débit a dépassé le quota provisionné.

Si le trafic est excessif, le service affiche le comportement suivant :

- Si le délai de la demande de production dépasse le délai d’expiration de la demande, Event Hubs renvoie un code d’erreur **Violation de stratégie**.
- Si le délai de la demande de récupération dépasse le délai d’expiration de la demande, Event Hubs consigne la demande comme étant limitée et répond avec un jeu d’enregistrements vide et sans code d’erreur.

Les [clusters dédiés](event-hubs-dedicated-overview.md) ne comportent pas de mécanismes de limitation. Vous êtes libre de consommer toutes vos ressources de cluster.

## <a name="no-records-received"></a>Aucun enregistrement reçu
Vous pouvez constater que les consommateurs ne reçoivent aucun enregistrement et se rééquilibrent en permanence. Dans ce scénario, les consommateurs n’obtiennent aucun enregistrement et se rééquilibrent en permanence. Il n’y a aucune exception ni erreur lorsque cela se produit, mais les journaux Kafka indiquent que les consommateurs ont été bloqués en tentant de rejoindre le groupe et d’affecter des partitions. Il existe quelques causes possibles :

- Assurez-vous que votre paramètre `request.timeout.ms` correspond au moins à la valeur recommandée de 60000 et que votre paramètre `session.timeout.ms` correspond au moins à la valeur recommandée de 30000. Le fait que ces paramètres soient trop faibles peut entraîner des délais d’attente pour les consommateurs, ce qui provoque alors des rééquilibrages (et donc des délais d’expiration supplémentaires entraînant davantage de rééquilibrage, etc.) 
- Si votre configuration correspond à ces valeurs recommandées et que vous constatez toujours un rééquilibrage constant, n’hésitez pas à signaler le problème (veillez à inclure l’intégralité de votre configuration dans le problème pour faciliter le débogage) !

## <a name="compressionmessage-format-version-issue"></a>Problème de version de format de compression/message
Kafka prend en charge la compression, et Event Hubs pour Kafka n’est pas disponible actuellement. Les erreurs qui mentionnent une version de format de message (par exemple, `The message format version on the broker does not support the request.`) sont provoquées par le fait qu’un client tente d’envoyer des messages Kafka compressés à nos répartiteurs.

Si des données compressées sont nécessaires, compressez vos données avant de les envoyer aux répartiteurs et de les décompresser une fois que vous avez reçu une solution de contournement valide. Le corps du message n’est qu’un tableau d’octets vers le service, de sorte que la compression/décompression côté client ne pose aucun problème.

## <a name="unknownserverexception"></a>UnknownServerException
Vous pouvez recevoir une erreur UnknownServerException provenant des bibliothèques clientes Kafka semblables à l’exemple suivant : 

```
org.apache.kafka.common.errors.UnknownServerException: The server experienced an unexpected error when processing the request
```

Ouvrez un ticket auprès du Support Microsoft.  La journalisation au niveau du débogage et les horodatages d’exception au format UTC sont utiles pour déboguer le problème. 

## <a name="other-issues"></a>Autres problèmes
Vérifiez les éléments suivants si vous rencontrez des problèmes lors de l’utilisation de Kafka avec Event Hubs.

- **Pare-feu bloquant le trafic** : assurez-vous que le port **9093** n’est pas bloqué par votre pare-feu.
- **TopicAuthorizationException** : les causes les plus courantes de cette exception sont les suivantes :
    - Une faute de frappe dans la chaîne de connexion de votre fichier de configuration, ou
    - Une tentative d’utilisation d’Event Hubs pour Kafka sur un espace de noms de niveau de base. La fonctionnalité Event Hubs pour Kafka est [prise en charge uniquement pour les espaces de noms de niveau Standard et Dedicated](https://azure.microsoft.com/pricing/details/event-hubs/).
- **Non-concordance des versions Kafka** : Event Hubs pour les écosystèmes Kafka prend en charge Kafka versions 1.0 et ultérieures. Certaines applications utilisant Kafka version 0.10 et ultérieures peuvent parfois fonctionner en raison de la compatibilité descendante du protocole Kafka, mais nous vous recommandons fortement d’utiliser les anciennes versions de l’API. Les versions Kafka 0.9 et antérieures ne prennent pas en charge les protocoles SASL requis et ne peuvent pas se connecter à Event Hubs.
- **Encodages étranges sur les en-têtes AMQP lors de l’utilisation de Kafka** : lors de l’envoi d’événements à un hub d’événements via AMQP, tous les en-têtes de charge utile AMQP sont sérialisés avec un encodage AMQP. Les consommateurs Kafka ne désérialisent pas les en-têtes à partir de AMQP. Pour lire les valeurs d’en-tête, décodez manuellement les en-têtes AMQP. Vous pouvez également éviter d’utiliser des en-têtes AMQP si vous savez que vous allez consommer via le protocole Kafka. Pour plus d’informations, consultez [ce problème GitHub](https://github.com/Azure/azure-event-hubs-for-kafka/issues/56).
- **Authentification SASL** : faire coopérer votre infrastructure avec le protocole d’authentification SASL requis par Event Hubs peut être plus difficile que prévu. Vérifiez si vous pouvez résoudre les problèmes de configuration à l’aide des ressources de votre infrastructure sur l’authentification SASL. 

## <a name="limits"></a>limites
Apache Kafka et Event Hubs Kafka. Dans la plupart des cas, l’interface d’Azure Event Hubs Kafka affiche les mêmes valeurs par défaut, les mêmes propriétés, les mêmes codes d’erreur et le même comportement général qu’Apache Kafka. Les instances où ces deux écosystèmes diffèrent de façon explicite (là où Event Hubs impose une limite, contrairement à Kafka) sont répertoriées ci-dessous :

- La longueur maximale de la propriété `group.id` est de 256 caractères
- La taille maximale de `offset.metadata.max.bytes` est de 1024 octets
- Les validations de décalage sont limitées à 4 appels par seconde et par partition, avec une taille de journal interne maximale de 1 Mo


## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur Event Hubs et sur Event Hubs pour Kafka, consultez les articles suivants :  

- [Guide du développeur Apache Kafka pour Event Hubs](apache-kafka-developer-guide.md)
- [Guide de migration Apache Kafka pour Event Hubs](apache-kafka-migration-guide.md)
- [Forum aux questions (FAQ) - Event Hubs pour Apache Kafka](apache-kafka-frequently-asked-questions.md)
- [Configurations recommandées](apache-kafka-configurations.md)
