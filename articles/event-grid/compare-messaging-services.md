---
title: Comparer les services de messagerie Azure
description: Décrit les trois services de messagerie Azure - Azure Event Grid, Event Hubs et Service Bus. Recommande le service à utiliser pour différents scénarios.
ms.topic: overview
ms.date: 07/07/2020
ms.openlocfilehash: 7a3a0cd7f63a67206053ae55f33bd71aee2c19c6
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "87460403"
---
# <a name="choose-between-azure-messaging-services---event-grid-event-hubs-and-service-bus"></a>Choisissez entre les trois services de messagerie Azure - Event Grid, Event Hubs et Service Bus

Azure propose trois services qui facilitent la remise des messages d’événement dans une solution. Ces services sont :

* [Event Grid](./index.yml)
* [Hubs d'événements](../event-hubs/index.yml)
* [Service Bus](../service-bus-messaging/index.yml)

Même si ces services ont des similitudes, chacun d’eux est conçu pour certains scénarios particuliers. Cet article décrit les différences entre ces services et vous aide à déterminer celui qui convient à votre application. Dans de nombreux cas, les services de messagerie sont complémentaires et peuvent être utilisés ensemble.

## <a name="event-vs-message-services"></a>Services d’événement et services de message

Il existe une distinction importante entre les services qui remettent un événement et les services qui remettent un message.

### <a name="event"></a>Événement

Un événement est une notification légère d’une condition ou d’un changement d’état. Le serveur de publication de l’événement n’a aucune attente vis-à-vis de la façon dont est géré l’événement. Le consommateur de l’événement décide ce qu’il fait de la notification. Les événements peuvent être des unités discrètes ou appartenir à une série.

Les événements discrets signalent un changement d’état et sont actionnables. Pour passer à l’étape suivante, le consommateur a uniquement besoin de savoir que quelque chose s’est produit. Les données d’événement contiennent des informations sur ce qui est arrivé, mais ne comportent pas les données qui ont déclenché l’événement. Par exemple, un événement avertit les consommateurs qu’un fichier a été créé. Il peut contenir des informations générales sur le fichier, mais pas le fichier lui-même. Les événements discrets conviennent parfaitement aux solutions [sans serveur](https://azure.com/serverless) qui ont besoin d’être mises à l’échelle.

Les événements en série signalent une condition et sont analysables. Ces événements sont chronologiques et liés entre eux. Le consommateur a besoin de la série séquencée des événements pour analyser ce qui s’est passé.

### <a name="message"></a>Message

Un message est constitué de données brutes générées par un service et destinées à être consommées ou stockées ailleurs. Le message contient les données qui ont déclenché le pipeline de message. Le serveur de publication du message a des attentes vis-à-vis du consommateur quant à sa manière de gérer le message. Il existe un contrat entre les deux côtés. Par exemple, le serveur de publication envoie un message avec les données brutes et s’attend à ce que le consommateur crée un fichier à partir de ces données et envoie une réponse quand il a terminé.

## <a name="comparison-of-services"></a>Comparaison des services

| Service | Objectif | Type | Quand l’utiliser |
| ------- | ------- | ---- | ----------- |
| Event Grid | Programmation réactive | Distribution d’événements (discrets) | Réagir aux changements d’état |
| Event Hubs | Pipeline de Big Data | Streaming d’événements (série) | Données de télémétrie et streaming de données distribuées |
| Service Bus | Messages importants de l’entreprise | Message | Traitement des commandes et transactions financières |

### <a name="event-grid"></a>Event Grid

Event Grid est un fond de panier de gestion des événements qui permet la programmation réactive, pilotée par événements. Il utilise un modèle publication-abonnement. Les éditeurs émettent des événements, mais ne savent pas quels événements sont traités. Les abonnés décident quels événements ils souhaitent traiter.

Event Grid est pleinement intégré aux services Azure et peut être intégré à des services tiers. Il simplifie la consommation d’événements et réduit les coûts en éliminant la nécessité d’une interrogation constante. Event Grid achemine les événements à partir de ressources Azure et non-Azure de façon efficace et fiable. Il distribue les événements à des points de terminaison d’abonné inscrits. Le message d’événement contient les informations dont vous avez besoin pour réagir aux modifications apportées aux services et applications. Event Grid n’est pas un pipeline de données et ne fournit pas l’objet réel qui a été mis à jour.

Event Grid prend en charge les lettres mortes pour les événements qui ne sont pas remis à un point de terminaison.

Il présente les caractéristiques suivantes :

* Dynamiquement scalable
* Économique
* Serverless
* Au moins une remise

### <a name="event-hubs"></a>Event Hubs

Azure Event Hubs est un pipeline de Big Data. Il facilite la capture, la rétention et la relecture des données de télémétrie et des données de flux d’événements. Les données peuvent provenir de nombreuses sources simultanées. Event Hubs permet de mettre les données de télémétrie et d’événements à la disposition d’un grand nombre de services d’analytique et d’infrastructures de traitement de flux. Les données sont disponibles sous forme de flux de données ou de lots d’événements regroupés. Ce service fournit une solution unique pour la récupération rapide de données à traiter en temps réel et pour la relecture répétée de données brutes stockées. Il peut capturer les données de diffusion en continu dans un fichier à des fins de traitement et d’analyse.

Il présente les caractéristiques suivantes :

* Latence faible
* Capable de recevoir et de traiter des millions d’événements par seconde
* Au moins une remise

### <a name="service-bus"></a>Service Bus

Service Bus est destiné aux applications d’entreprise traditionnelles. Ces applications d’entreprise traditionnelles nécessitent des transactions, des classements, la détection des doublons et une cohérence instantanée. Service Bus permet aux applications [cloud natives](https://azure.microsoft.com/overview/cloudnative/) de fournir une gestion fiable des transitions d’état pour les processus d’entreprise. Pour le traitement de messages importants qui ne peuvent pas être perdus ou dupliqués, utilisez Azure Service Bus. De plus, Service Bus facilite la communication hautement sécurisée entre les solutions cloud hybrides et peut connecter des systèmes locaux existants aux solutions cloud.

Service Bus est un système de messagerie réparti. Il stocke les messages dans un « broker » (par exemple, une file d’attente) jusqu’à ce que le récepteur soit prêt à les recevoir.

Il présente les caractéristiques suivantes :

* Remise de messages asynchrone fiable (messagerie d’entreprise en tant que service) qui nécessite une interrogation
* Fonctionnalités de messagerie avancées telles que FIFO, traitement par lot/sessions, transactions, lettres mortes, contrôle temporel, routage, filtrage et détection des doublons
* Au moins une remise
* Livraison chronologique en option

## <a name="use-the-services-together"></a>Utiliser les services ensemble

Dans certains cas, vous utilisez les services côte à côte afin qu’ils remplissent des rôles distincts. Par exemple, un site marchand peut utiliser Service Bus pour traiter les commandes, Event Hubs pour capturer la télémétrie du site et Event Grid pour répondre à des événements comme l’expédition d’un article.

Dans d’autres cas, vous les liez pour former un pipeline d’événements et de données. Event Grid vous permet de répondre aux événements dans les autres services. Pour obtenir un exemple d’utilisation d’Event Grid avec Event Hubs afin de migrer des données vers un entrepôt de données, consultez [Flux de Big Data dans un entrepôt de données](event-grid-event-hubs-integration.md). L’image suivante montre le flux de travail du flux des données.

![Vue d’ensemble du flux des données](./media/compare-messaging-services/overview.png)

## <a name="next-steps"></a>Étapes suivantes
Voir les articles suivants : 
- [Options de messagerie asynchrone dans Azure](/azure/architecture/guide/technology-choices/messaging)
- [Événements, points de données et messages - Sélection du service de messagerie Azure adapté à vos données](https://azure.microsoft.com/blog/events-data-points-and-messages-choosing-the-right-azure-messaging-service-for-your-data/).
- [Files d’attente de stockage et files d’attente Service Bus : comparaison et différences](../service-bus-messaging/service-bus-azure-and-service-bus-queues-compared-contrasted.md)
- Pour bien démarrer avec Event Grid, consultez [Créer et acheminer des événements personnalisés avec Azure Event Grid](custom-event-quickstart.md).
- Pour bien démarrer avec Event Hubs, consultez [Créer un espace de noms Event Hubs et un hub d’événements à l’aide du portail Azure](../event-hubs/event-hubs-create.md).
- Pour bien démarrer avec Service Bus, consultez [Créer un espace de noms Service Bus à l’aide du Portail Azure](../service-bus-messaging/service-bus-create-namespace-portal.md).
