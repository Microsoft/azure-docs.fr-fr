---
title: Guide pratique pour utiliser les files d’attente Azure Service Bus avec Ruby
description: Dans ce tutoriel, vous allez apprendre à créer des applications Ruby afin d’envoyer des messages à une file d’attente Service Bus.
documentationcenter: ruby
ms.devlang: ruby
ms.topic: quickstart
ms.date: 06/23/2020
ms.openlocfilehash: 16dda6fc4637f052514a0e78a0804bf4702ed20b
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "85336656"
---
# <a name="quickstart-how-to-use-service-bus-queues-with-ruby"></a>Démarrage rapide : Utilisation des files d’attente Service Bus avec Ruby

[!INCLUDE [service-bus-selector-queues](../../includes/service-bus-selector-queues.md)]

Dans ce tutoriel, vous allez apprendre à créer des applications Ruby afin d’envoyer des messages à une file d’attente Service Bus. Les exemples sont écrits en Ruby et utilisent le module Azure gem.

## <a name="prerequisites"></a>Prérequis
1. Un abonnement Azure. Pour suivre ce tutoriel, vous avez besoin d’un compte Azure. Vous pouvez [activer les avantages de votre abonnement MSDN](https://azure.microsoft.com/pricing/member-offers/credit-for-visual-studio-subscribers/?WT.mc_id=A85619ABF) ou [vous inscrire pour un compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A85619ABF).
2. Suivez les étapes de l’article [Utiliser le portail Azure pour créer une file d’attente Service Bus](service-bus-quickstart-portal.md).
    1. Consultez la **vue d’ensemble** rapide des **files d’attente** Service Bus. 
    2. Créez un **espace de noms** Service Bus. 
    3. Obtenez la **chaîne de connexion**. 

        > [!NOTE]
        > Dans ce tutoriel, vous allez créer un **file d’attente** dans l’espace de noms Service Bus à l’aide de Ruby. 

[!INCLUDE [service-bus-ruby-setup](../../includes/service-bus-ruby-setup.md)]

## <a name="how-to-create-a-queue"></a>Création d’une file d’attente
L’objet **Azure::ServiceBusService** permet d’utiliser des files d’attente. Pour créer une file d’attente, utilisez la méthode `create_queue()`. L'exemple suivant crée une file d'attente ou imprime l'erreur le cas échéant.

```ruby
azure_service_bus_service = Azure::ServiceBus::ServiceBusService.new(sb_host, { signer: signer})
begin
  queue = azure_service_bus_service.create_queue("test-queue")
rescue
  puts $!
end
```

Vous pouvez également passer un objet **Azure::ServiceBus::Queue** avec des options complémentaires, ce qui vous permet de remplacer les paramètres de file d’attente par défaut comme la durée de vie de message ou la taille de file maximale. L'exemple suivant montre comment définir la taille maximale de la file d'attente sur 5 Go et la durée de vie de message sur 1 minute :

```ruby
queue = Azure::ServiceBus::Queue.new("test-queue")
queue.max_size_in_megabytes = 5120
queue.default_message_time_to_live = "PT1M"

queue = azure_service_bus_service.create_queue(queue)
```

## <a name="how-to-send-messages-to-a-queue"></a>Envoi de messages à une file d’attente
Pour envoyer un message à une file d’attente Service Bus, votre application appelle la méthode `send_queue_message()` sur l’objet **Azure::ServiceBusService**. Les messages envoyés aux files d’attente Service Bus (et reçus de celles-ci) sont des objets **Azure::ServiceBus::BrokeredMessage**. Ils possèdent un ensemble de propriétés standard (telles que `label` et `time_to_live`), un dictionnaire servant à conserver les propriétés personnalisées propres à une application, ainsi qu’un corps de données d’application arbitraires. Une application peut définir le corps du message en transmettant une valeur de chaîne en tant que message pour remplir toutes les propriétés standard requises avec les valeurs par défaut.

L’exemple suivant indique comment envoyer un message test à la file d’attente nommée `test-queue` au moyen de la méthode `send_queue_message()` :

```ruby
message = Azure::ServiceBus::BrokeredMessage.new("test queue message")
message.correlation_id = "test-correlation-id"
azure_service_bus_service.send_queue_message("test-queue", message)
```

Les files d’attente Service Bus prennent en charge une taille de message maximale de 256 Ko dans le [niveau Standard](service-bus-premium-messaging.md) et de 1 Mo dans le [niveau Premium](service-bus-premium-messaging.md). L’en-tête, qui comprend les propriétés d’application standard et personnalisées, peut avoir une taille maximale de 64 Ko. Si une file d'attente n'est pas limitée par le nombre de messages qu'elle peut contenir, elle l'est en revanche par la taille totale des messages qu'elle contient. Cette taille de file d'attente est définie au moment de la création. La limite maximale est de 5 Go.

## <a name="how-to-receive-messages-from-a-queue"></a>Réception des messages d’une file d’attente
La méthode `receive_queue_message()` de l’objet **Azure::ServiceBusService** permet de recevoir les messages d’une file d’attente. Par défaut, les messages sont lus et verrouillés sans être supprimés de la file d'attente. Il est toutefois possible de supprimer ces messages de la file d’attente lors de leur lecture en paramétrant l’option `:peek_lock` sur **false**.

Avec le comportement par défaut, la lecture et la suppression sont une opération en deux étapes, ce qui permet également de prendre en charge des applications ne pouvant pas fonctionner avec des messages manquants. Lorsque Service Bus reçoit une demande, il recherche le prochain message à consommer, le verrouille pour empêcher d'autres consommateurs de le recevoir, puis le renvoie à l'application. Dès lors que l’application a terminé le traitement du message (ou qu’elle l’a stocké de manière fiable pour un traitement ultérieur), elle accomplit la deuxième étape du processus de réception en appelant la méthode `delete_queue_message()` et en fournissant le message à supprimer sous la forme d’un paramètre. La méthode `delete_queue_message()` marque le message comme étant consommé et le supprime de la file d’attente.

Si le paramètre `:peek_lock` est défini sur **false**, le modèle le plus simple correspond à la lecture et à la suppression du message. Ce modèle fonctionne mieux pour les scénarios dans lesquels une application peut accepter de ne pas traiter un message en cas d’échec. Pour mieux comprendre, imaginez un scénario dans lequel le consommateur émet la demande de réception et subit un incident avant de la traiter. Étant donné que Service Bus a marqué le message comme étant consommé, lorsque l’application redémarre et recommence à consommer des messages, elle manque le message consommé avant l’incident.

L’exemple suivant montre comment recevoir et traiter des messages à l’aide de la méthode `receive_queue_message()`. Dans l’exemple, un message est d’abord reçu puis supprimé par le biais de `:peek_lock` défini sur **false**. Un autre message est ensuite reçu, puis supprimé via `delete_queue_message()` :

```ruby
message = azure_service_bus_service.receive_queue_message("test-queue",
  { :peek_lock => false })
message = azure_service_bus_service.receive_queue_message("test-queue")
azure_service_bus_service.delete_queue_message(message)
```

## <a name="how-to-handle-application-crashes-and-unreadable-messages"></a>Gestion des blocages d’application et des messages illisibles
Service Bus intègre des fonctionnalités destinées à faciliter la récupération à la suite d’erreurs survenues dans votre application ou de difficultés à traiter un message. Si une application réceptrice ne parvient pas à traiter le message pour une raison quelconque, elle appelle la méthode `unlock_queue_message()` pour l’objet **Azure::ServiceBusService**. Suite à cet appel, Service Bus déverrouille le message dans la file d'attente et le rend à nouveau disponible en réception, pour la même application consommatrice ou pour une autre.

De même, il faut savoir qu'un message verrouillé dans une file d'attente est assorti d'un délai d'expiration et que si l'application ne parvient pas à traiter le message dans le temps imparti (par exemple, si l'application subit un incident), Service Bus déverrouille le message automatiquement et le rend à nouveau disponible en réception.

Si l’application subit un incident après le traitement du message, mais avant l’appel de la méthode `delete_queue_message()`, le message est à nouveau remis à l’application lorsqu’elle redémarre. Dans ce type de traitement, souvent appelé *Au moins une fois*, chaque message est traité au moins une fois. Toutefois, dans certaines circonstances, un même message peut être remis une nouvelle fois. Ceci, grâce à la propriété `message_id` du message, qui reste constante entre les tentatives de transmission.

> [!NOTE]
> Vous pouvez gérer les ressources Service Bus à l'aide de [Service Bus Explorer](https://github.com/paolosalvatori/ServiceBusExplorer/). Service Bus Explorer permet aux utilisateurs de se connecter à un espace de noms Service Bus et de gérer les entités de messagerie en toute simplicité. L’outil fournit des fonctionnalités avancées telles que la fonction d’importation/exportation ou la possibilité de tester une rubrique, des files d’attente, des abonnements, des services de relais, des hubs de notification et des hubs d’événements. 

## <a name="next-steps"></a>Étapes suivantes
Maintenant que vous avez appris les principes de base des files d'attente Service Bus, consultez ces liens pour en savoir plus :

* Vue d’ensemble des [files d’attente, rubriques et abonnements](service-bus-queues-topics-subscriptions.md).
* Accédez au référentiel du [Kit de développement logiciel (SDK) Azure pour Ruby](https://github.com/Azure/azure-sdk-for-ruby) sur GitHub.

Pour consulter un comparatif entre les files d’attente Azure Service Bus évoquées dans cet article et les files d’attente Azure présentées dans l’article [Utilisation de Queue Storage à partir de Ruby](../storage/queues/storage-ruby-how-to-use-queue-storage.md), consultez [Files d’attente Azure et files d’attente Azure Service Bus - comparaison et différences](service-bus-azure-and-service-bus-queues-compared-contrasted.md)

