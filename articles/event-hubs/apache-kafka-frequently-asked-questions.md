---
title: Forum aux questions (FAQ) - Azure Event Hubs pour Apache Kafka
description: Cet article répond aux questions fréquentes posées sur la prise en charge d’Azure Event Hubs pour les clients Apache Kafka non couvertes ailleurs.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: dc6a12b2098a1fdf33adda92b4347f91ab4e5489
ms.sourcegitcommit: d2222681e14700bdd65baef97de223fa91c22c55
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/07/2020
ms.locfileid: "91828103"
---
# <a name="frequently-asked-questions---event-hubs-for-apache-kafka"></a>Forum aux questions (FAQ) - Event Hubs pour Apache Kafka 
Cet article fournit des réponses à certaines des questions fréquemment posées concernant la migration vers Event Hubs pour Apache Kafka.

## <a name="does-azure-event-hubs-run-on-apache-kafka"></a>Azure Event Hubs s’exécute-t-il sur Apache Kafka ?

Non. Azure Event Hubs est un répartiteur multiniveau cloud natif qui prend en charge plusieurs protocoles développés et gérés par Microsoft et qui n’utilise pas de code Apache Kafka. L’un des protocoles pris en charge est le protocole RPC Kafka pour les API de consommateur et de producteur du client Kafka. Event Hubs fonctionne avec un grand nombre de vos applications Kafka existantes. Pour plus d’informations, consultez [Event Hubs pour Apache Kafka](event-hubs-for-kafka-ecosystem-overview.md). Dans la mesure où les concepts d’Apache Kafka et d’Azure Event Hubs sont très similaires (mais pas identiques), nous pouvons offrir la fiabilité inégalée d’Azure Event Hubs aux clients avec des investissements Apache Kafka existants. 

## <a name="event-hubs-consumer-group-vs-kafka-consumer-group"></a>Groupe de consommateurs Event Hubs et groupe de consommateurs Kafka
Quelle est la différence entre un groupe de consommateurs Event Hub et un groupe de consommateurs Kafka sur Event Hubs ? Les groupes de consommateurs Kafka sur Event Hubs sont entièrement distincts des groupes de consommateurs Event Hubs standard.

**Groupes de consommateurs Event Hubs**

- Ils sont gérés avec des opérations de création, de récupération, de mise à jour et de suppression (CRUD) via le portail, le Kit de développement logiciel (SDK) ou des modèles Azure Resource Manager. Les groupes de consommateurs Event Hubs ne peuvent pas être autocréés.
- Il s’agit d’entités enfants d’un hub d’événements. Cela signifie que le même nom de groupe de consommateurs peut être réutilisé entre les hubs d’événements d’un même espace de noms car ce sont des entités distinctes.
- Elles ne sont pas utilisées pour stocker des décalages. La consommation AMQP orchestrée s’effectue à l’aide d’un stockage de décalage externe, par exemple, Stockage Azure.

**Groupes de consommateurs Kafka**

- Ils sont autocréés.  Les groupes Kafka peuvent être gérés via les API du groupe de consommateurs Kafka.
- Ils peuvent stocker des décalages dans le service Event Hubs.
- Ils sont utilisés en tant que clés dans ce qui est effectivement un stockage clé-valeur de décalage. Pour une paire unique regroupant `group.id` et `topic-partition`, nous stockons un décalage dans Stockage Azure (triple réplication). Les utilisateurs Event Hubs n’entraînent pas de frais supplémentaires pour le stockage des décalages Kafka. Les décalages peuvent être gérés via les API du groupe de consommateurs Kafka, mais les *comptes* de stockage de décalage ne sont pas directement visibles ou gérables pour les utilisateurs du hub d’événements.  
- Ils s’étendent sur un espace de noms. L’utilisation du même nom de groupe Kafka pour plusieurs applications sur plusieurs rubriques signifie que la totalité des applications et leurs clients Kafka sera rééquilibrée chaque fois qu’une seule application nécessite un rééquilibrage.  Choisissez avec soin vos noms de groupes.
- Ils sont entièrement distincts des groupes de consommateurs Event Hubs. Vous **n’avez pas** besoin d’utiliser « $Default », pas plus que vous n’avez à vous soucier des clients Kafka interférant avec les charges de travail AMQP.
- Ils n’apparaissent pas dans le portail Azure. Les informations sur le groupe de consommateurs sont accessibles via les API Kafka.

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur Event Hubs et sur Event Hubs pour Kafka, consultez les articles suivants :  

- [Guide du développeur Apache Kafka pour Event Hubs](apache-kafka-developer-guide.md)
- [Guide de migration Apache Kafka pour Event Hubs](apache-kafka-migration-guide.md)
- [Guide de dépannage Apache Kafka pour Event Hubs](apache-kafka-troubleshooting-guide.md)
- [Configurations recommandées](apache-kafka-configurations.md)

