---
title: Configurations recommandées pour les clients Apache Kafka – Azure Event Hubs
description: Cet article fournit des configurations Apache Kafka recommandées pour les clients qui interagissent avec Azure Event Hubs pour Apache Kafka.
ms.topic: reference
ms.date: 07/20/2020
ms.openlocfilehash: f9a03d1d3433461a575b32cd69893408a8b0ef97
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87094086"
---
# <a name="recommended-configurations-for-apache-kafka-clients"></a>Configurations recommandées pour les clients Apache Kafka
Voici les configurations recommandées pour l’utilisation d’Azure Event Hubs à partir d’applications clientes Apache Kafka. 

## <a name="java-client-configuration-properties"></a>Propriétés de configuration de client Java

### <a name="producer-and-consumer-configurations"></a>Configurations de producteur et de consommateur

Propriété | Valeurs recommandées | Plage autorisée | Notes
---|---:|-----:|---
`metadata.max.age.ms` | 180000 (approximation) | < 240000 | Peut être abaissé pour récupérer les modifications de métadonnées plus tôt.
`connections.max.idle.ms`   | 180000 | < 240000 | Azure ferme le TCP entrant inactif > 240 000 ms, ce qui peut entraîner un envoi sur des connexions mortes (affichées comme des lots arrivés à expiration en raison du délai d’expiration d’envoi).

### <a name="producer-configurations-only"></a>Configurations du producteur uniquement
Les configurations de producteur se trouvent [ici](https://kafka.apache.org/documentation/#producerconfigs).

Propriété | Valeurs recommandées | Plage autorisée | Notes
---|---:|---:|---
`max.request.size` | 1000000 | < 1046528 | Le service ferme les connexions si les demandes envoyées dépassent 1 046 528 octets.  *Cette valeur **doit** être modifiée et provoquera des problèmes dans les scénarios de production à haut débit.*
`retries` | > 0 | | Peut nécessiter une augmentation de la valeur delivery.timeout.ms, voir la documentation.
`request.timeout.ms` | 30000 60000 | > 20000| La valeur de EH en interne sera au minimum de 20 000 ms par défaut.  *Si des demandes avec des valeurs de délai d’expiration inférieures sont acceptées, le comportement du client n’est pas garanti.*
`metadata.max.idle.ms` | 180000 | > 5000 | Contrôle la durée pendant laquelle le producteur met en cache les métadonnées pour une rubrique inactive. Si le temps écoulé depuis la dernière production d’une rubrique dépasse la durée d’inactivité des métadonnées, les métadonnées de la rubrique sont oubliées et l’accès suivant à celles-ci force une demande d’extraction des métadonnées.
`linger.ms` | > 0 | | Pour les scénarios à haut débit, la valeur de linger doit être égale à la valeur tolérable la plus élevée pour tirer parti du traitement par lot.
`delivery.timeout.ms` | | | Définissez la valeur en fonction de la formule (`request.timeout.ms` + `linger.ms`) * `retries`.
`enable.idempotence` | false | | Idempotence actuellement non prise en charge.
`compression.type` | `none` | | Compression actuellement non prise en charge.

### <a name="consumer-configurations-only"></a>Configurations de consommateur uniquement.
Les configurations de consommateur se trouvent [ici](https://kafka.apache.org/documentation/#consumerconfigs).

Propriété | Valeurs recommandées | Plage autorisée | Notes
---|---:|-----:|---
`heartbeat.interval.ms` | 3000 | | 3000 est la valeur par défaut qui ne doit pas être modifiée.
`session.timeout.ms` | 30000 |6000 300000| Commencez par 30000, et augmentez si vous constatez des rééquilibrages fréquents en raison de pulsations manquées.


## <a name="librdkafka-configuration-properties"></a>Propriétés de configuration de librdkafka
Le fichier de configuration `librdkafka` principal ([link](https://github.com/edenhill/librdkafka/blob/master/CONFIGURATION.md)) contient des descriptions étendues des propriétés ci-dessous.

### <a name="producer-and-consumer-configurations"></a>Configurations de producteur et de consommateur

Propriété | Valeurs recommandées | Plage autorisée | Notes
---|---:|-----:|---
`socket.keepalive.enable` | true | | Nécessaire si la connexion est censée être inactive.  Azure va fermer le TCP inactif entrant > 240 000 ms.
`metadata.max.age.ms` | 180000| < 240000 | Peut être abaissé pour récupérer les modifications de métadonnées plus tôt.

### <a name="producer-configurations-only"></a>Configurations du producteur uniquement

Propriété | Valeurs recommandées | Plage autorisée | Notes
---|---:|-----:|---
`retries` | > 0 | | La valeur par défaut est 2. Nous vous conseillons de conserver cette valeur. 
`request.timeout.ms` | 30000 60000 | > 20000| La valeur de EH en interne sera au minimum de 20 000 ms par défaut.  La valeur par défaut de `librdkafka` est 5000, ce qui peut être problématique. *Si des demandes avec des valeurs de délai d’expiration inférieures sont acceptées, le comportement du client n’est pas garanti.*
`partitioner` | `consistent_random` | Consultez la documentation de librdkafka | `consistent_random` est la valeur par défaut et la meilleure.  Dans la plupart des cas, les clés vides et NULL sont gérées de façon idéale.
`enable.idempotence` | false | | Idempotence actuellement non prise en charge.
`compression.codec` | `none` || Compression actuellement non prise en charge.

### <a name="consumer-configurations-only"></a>Configurations de consommateur uniquement.

Propriété | Valeurs recommandées | Plage autorisée | Notes
---|---:|-----:|---
`heartbeat.interval.ms` | 3000 || 3000 est la valeur par défaut qui ne doit pas être modifiée.
`session.timeout.ms` | 30000 |6000 300000| Commencez par 30000, et augmentez si vous constatez des rééquilibrages fréquents en raison de pulsations manquées.


## <a name="further-notes"></a>Autres remarques

Consultez le tableau suivant des scénarios d’erreur courantes liées à la configuration.

Symptômes | Problème | Solution
----|---|-----
Échecs de validation de décalage en raison du rééquilibrage | Votre consommateur attend trop longtemps entre les appels à poll(), et le service éjecte le consommateur du groupe. | Vous disposez de plusieurs options : <ul><li>augmenter le délai d’expiration de session</li><li>réduire la taille du lot de messages pour accélérer le traitement</li><li>améliorez la parallélisation du traitement pour éviter le blocage de consumer.poll()</li></ul> L’application d’une combinaison des trois est probablement le choix le plus judicieux.
Exceptions réseau à un débit de production élevé | Utilisez-vous le client Java + valeur de max.request.size par défaut ?  Vos demandes sont peut-être trop volumineuses. | Consultez les configurations Java ci-dessus.

## <a name="next-steps"></a>Étapes suivantes
Pour connaître les quotas et limites de tous les services Azure, consultez [Abonnement Azure et limites, quotas et contraintes de service](..//azure-resource-manager/management/azure-subscription-service-limits.md). 
