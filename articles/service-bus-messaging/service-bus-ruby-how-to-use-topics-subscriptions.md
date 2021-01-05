---
title: 'Démarrage rapide : Guide pratique pour utiliser les rubriques Service Bus (Ruby)'
description: 'Démarrage rapide : Découvrez comment utiliser les rubriques et abonnements Service Bus dans Azure. Les exemples de code sont écrits pour les applications Ruby.'
services: service-bus-messaging
documentationcenter: ruby
ms.devlang: ruby
ms.topic: quickstart
ms.date: 06/23/2020
ms.openlocfilehash: aba326a63558632bee3bf0c48d34e471bbe30886
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "88067560"
---
# <a name="quickstart-how-to-use-service-bus-topics-and-subscriptions-with-ruby"></a>Démarrage rapide : Utilisation des rubriques et abonnements Service Bus avec Ruby
 
[!INCLUDE [service-bus-selector-topics](../../includes/service-bus-selector-topics.md)]

Cet article décrit l’utilisation des rubriques et des abonnements Service Bus depuis les applications Ruby. Parmi les scénarios présentés :

- Création de rubriques et d’abonnements 
- Création de filtres d’abonnement 
- Envoi de messages à une rubrique 
- Réception de messages à partir d’un abonnement
- Suppression de rubriques et d’abonnements


## <a name="prerequisites"></a>Prérequis
1. Un abonnement Azure. Pour suivre ce tutoriel, vous avez besoin d’un compte Azure. Vous pouvez activer les [avantages de votre abonnement Visual Studio ou MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A85619ABF) ou vous inscrire pour créer un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A85619ABF).
2. Suivez les étapes dans [Démarrage rapide : utiliser le portail Azure pour créer une rubrique Service Bus et des abonnements à cette rubrique](service-bus-quickstart-topics-subscriptions-portal.md) pour créer un **espace de noms** Service Bus et obtenir la **chaîne de connexion**. 

    > [!NOTE]
    > Vous allez créer une **rubrique** et un **abonnement** à la rubrique à l’aide de **Ruby** dans ce démarrage rapide. 

[!INCLUDE [service-bus-ruby-setup](../../includes/service-bus-ruby-setup.md)]

## <a name="create-a-topic"></a>Création d'une rubrique
L’objet **Azure::ServiceBusService** permet d’utiliser des rubriques. Le code suivant crée un objet **Azure::ServiceBusService**. Pour créer une rubrique, utilisez la méthode `create_topic()`. L’exemple suivant crée une rubrique ou imprime les éventuelles erreurs.

```ruby
azure_service_bus_service = Azure::ServiceBus::ServiceBusService.new(sb_host, { signer: signer})
begin
  topic = azure_service_bus_service.create_topic("test-topic")
rescue
  puts $!
end
```

Vous pouvez également transmettre un objet **Azure::ServiceBus::Topic** avec des options complémentaires, ce qui vous permet de remplacer les paramètres de rubrique par défaut comme la durée de vie de message ou la taille de file d’attente maximale. L’exemple suivant montre comment définir la taille maximale de la file d’attente sur 5 Go et la durée de vie du message, sur 1 minute :

```ruby
topic = Azure::ServiceBus::Topic.new("test-topic")
topic.max_size_in_megabytes = 5120
topic.default_message_time_to_live = "PT1M"

topic = azure_service_bus_service.create_topic(topic)
```

## <a name="create-subscriptions"></a>Création d’abonnements
Les abonnements de rubrique sont également créés à l’aide de l’objet **Azure::ServiceBusService**. Les abonnements sont nommés et peuvent être assortis d'un filtre facultatif qui limite l'ensemble des messages transmis à la file d'attente virtuelle de l'abonnement.

Les abonnements sont persistants par défaut. Ils sont conservés jusqu’à leur suppression ou celle de la rubrique à laquelle ils sont associés. Si votre application contient une logique pour la création d’un abonnement, elle doit d’abord vérifier si l’abonnement existe déjà en utilisant la méthode getSubscription.

Vous pouvez supprimer automatiquement les abonnements en définissant la [propriété AutoDeleteOnIdle](/dotnet/api/microsoft.servicebus.messaging.subscriptiondescription.autodeleteonidle).

### <a name="create-a-subscription-with-the-default-matchall-filter"></a>Création d’un abonnement avec le filtre par défaut (MatchAll)
Si aucun filtre n’est spécifié lors de la création d’un abonnement, le filtre **MatchAll** (par défaut) est utilisé. Lorsque le filtre **MatchAll** est utilisé, tous les messages publiés dans la rubrique sont placés dans la file d’attente virtuelle de l’abonnement. Dans l’exemple suivant, l’abonnement « all-messages » qui est créé utilise le filtre par défaut **MatchAll**.

```ruby
subscription = azure_service_bus_service.create_subscription("test-topic", "all-messages")
```

### <a name="create-subscriptions-with-filters"></a>Création d’abonnements avec des filtres
Vous pouvez également définir des filtres pour spécifier quels sont les messages, parmi ceux envoyés à une rubrique, qui doivent apparaître dans un abonnement spécifique.

Parmi les types de filtres pris en charge par les abonnements, **Azure::ServiceBus::SqlFilter** est le plus flexible ; il implémente un sous-ensemble de SQL92. Les filtres SQL opèrent au niveau des propriétés des messages publiés dans la rubrique. Pour plus de détails sur les expressions utilisables avec un filtre SQL, examinez la syntaxe [SqlFilter](service-bus-messaging-sql-filter.md).

Vous pouvez ajouter des filtres à un abonnement en utilisant la méthode `create_rule()` de l’objet **Azure::ServiceBusService**. Cette méthode vous permet d’ajouter de nouveaux filtres à un abonnement existant.

Comme le filtre par défaut est appliqué automatiquement à tous les nouveaux abonnements, vous devez commencer par supprimer le filtre par défaut, sans quoi le filtre **MatchAll** remplace tous les autres filtres spécifiés. Vous pouvez supprimer la règle par défaut en utilisant la méthode `delete_rule()` sur l’objet **Azure::ServiceBusService**.

Dans l’exemple suivant, l’abonnement « high-messages » est créé avec un filtre **Azure::ServiceBus::SqlFilter** qui sélectionne uniquement les messages dont la propriété personnalisée `message_number` a une valeur supérieure à 3 :

```ruby
subscription = azure_service_bus_service.create_subscription("test-topic", "high-messages")
azure_service_bus_service.delete_rule("test-topic", "high-messages", "$Default")

rule = Azure::ServiceBus::Rule.new("high-messages-rule")
rule.topic = "test-topic"
rule.subscription = "high-messages"
rule.filter = Azure::ServiceBus::SqlFilter.new({
  :sql_expression => "message_number > 3" })
rule = azure_service_bus_service.create_rule(rule)
```

De même, l’exemple suivant crée l’abonnement `low-messages` avec un objet **Azure::ServiceBus::SqlFilter** qui ne sélectionne que les messages dont la propriété `message_number` a une valeur inférieure ou égale à 3 :

```ruby
subscription = azure_service_bus_service.create_subscription("test-topic", "low-messages")
azure_service_bus_service.delete_rule("test-topic", "low-messages", "$Default")

rule = Azure::ServiceBus::Rule.new("low-messages-rule")
rule.topic = "test-topic"
rule.subscription = "low-messages"
rule.filter = Azure::ServiceBus::SqlFilter.new({
  :sql_expression => "message_number <= 3" })
rule = azure_service_bus_service.create_rule(rule)
```

Lorsqu’un message est envoyé à `test-topic`, il est toujours remis aux destinataires abonnés à l’abonnement de rubrique `all-messages`, et est remis de manière sélective aux destinataires abonnés aux abonnements de rubrique `high-messages` et `low-messages` (en fonction du contenu du message).

## <a name="send-messages-to-a-topic"></a>Envoi de messages à une rubrique
Pour envoyer un message à une rubrique Service Bus, votre application doit utiliser la méthode `send_topic_message()` sur l’objet **Azure::ServiceBusService**. Les messages envoyés aux rubriques Service Bus sont des instances des objets **Azure::ServiceBus::BrokeredMessage**. Les objets **Azure::ServiceBus::BrokeredMessage** possèdent un ensemble de propriétés standard (telles que `label` et `time_to_live`), un dictionnaire servant à conserver les propriétés personnalisées propres à une application, ainsi qu’un corps de données de chaîne. Une application peut définir le corps du message en passant une valeur de chaîne à la méthode `send_topic_message()` pour définir toutes les propriétés standard requises avec les valeurs par défaut.

L’exemple suivant montre comment envoyer cinq messages de test à `test-topic`. La valeur de la propriété personnalisée `message_number` de chaque message varie au niveau de l’itération de la boucle (elle détermine l’abonnement qui le reçoit) :

```ruby
5.times do |i|
  message = Azure::ServiceBus::BrokeredMessage.new("test message " + i,
    { :message_number => i })
  azure_service_bus_service.send_topic_message("test-topic", message)
end
```

Les rubriques Service Bus prennent en charge une taille de message maximale de 256 Ko dans le [niveau Standard](service-bus-premium-messaging.md) et de 1 Mo dans le [niveau Premium](service-bus-premium-messaging.md). L’en-tête, qui comprend les propriétés d’application standard et personnalisées, peut avoir une taille maximale de 64 Ko. Si une rubrique n'est pas limitée par le nombre de messages qu'elle peut contenir, elle l'est en revanche par la taille totale des messages qu'elle contient. Cette taille de rubrique est définie au moment de la création. La limite maximale est de 5 Go.

## <a name="receive-messages-from-a-subscription"></a>Réception des messages d’un abonnement
La méthode `receive_subscription_message()` de l’objet **Azure::ServiceBusService** permet de recevoir les messages associés à un abonnement. Par défaut, les messages sont lus et verrouillés sans être supprimés de l’abonnement. Il est possible de lire et de supprimer le message de l’abonnement en définissant l’option `peek_lock` sur **false**.

Avec le comportement par défaut, la lecture et la suppression sont une opération en deux étapes, ce qui permet également de prendre en charge des applications ne pouvant pas fonctionner avec des messages manquants. Lorsque Service Bus reçoit une demande, il recherche le prochain message à consommer, le verrouille pour empêcher d'autres consommateurs de le recevoir, puis le renvoie à l'application. Dès lors que l’application a terminé le traitement du message (ou qu’elle l’a stocké de manière fiable pour un traitement ultérieur), elle accomplit la deuxième étape du processus de réception en appelant la méthode `delete_subscription_message()` et en fournissant le message à supprimer sous la forme d’un paramètre. La méthode `delete_subscription_message()` marque le message comme étant consommé et le supprime de l’abonnement.

Si le paramètre `:peek_lock` est défini sur **false**, le modèle le plus simple correspond à la lecture et à la suppression du message. Ce modèle fonctionne mieux pour les scénarios dans lesquels une application peut accepter de ne pas traiter un message après un échec. Imaginez un scénario dans lequel le consommateur émet la demande de réception et subit un incident avant de la traiter. Étant donné que Service Bus a marqué le message comme étant consommé, quand l’application redémarre et recommence à consommer des messages, elle a manqué le message consommé avant l’incident.

L’exemple suivant montre comment les messages peuvent être reçus et traités à l’aide de `receive_subscription_message()`. Dans l’exemple, un message est d’abord reçu, puis supprimé par le biais de l’abonnement `low-messages`, le paramètre `:peek_lock` étant défini sur **false**, puis `high-messages` envoie un autre message, qui est supprimé via `delete_subscription_message()` :

```ruby
message = azure_service_bus_service.receive_subscription_message(
  "test-topic", "low-messages", { :peek_lock => false })
message = azure_service_bus_service.receive_subscription_message(
  "test-topic", "high-messages")
azure_service_bus_service.delete_subscription_message(message)
```

## <a name="how-to-handle-application-crashes-and-unreadable-messages"></a>Gestion des blocages d’application et des messages illisibles
Service Bus intègre des fonctionnalités destinées à faciliter la récupération à la suite d’erreurs survenues dans votre application ou de difficultés à traiter un message. Si une application réceptrice ne parvient pas à traiter le message pour une raison quelconque, elle appelle la méthode `unlock_subscription_message()` pour l’objet **Azure::ServiceBusService**. Cela amène Service Bus à déverrouiller le message dans l’abonnement et à le rendre à nouveau disponible en réception, pour la même application consommatrice ou pour une autre.

De même, il faut savoir qu’un message verrouillé dans un abonnement est assorti d’un délai d’expiration et que si l’application ne parvient pas à traiter le message dans le temps imparti (par exemple, si l’application plante), Service Bus déverrouille le message automatiquement et le rend à nouveau disponible en réception.

Si l’application subit un incident après le traitement du message, mais avant l’appel de la méthode `delete_subscription_message()`, le message est à nouveau remis à l’application lorsqu’elle redémarre. Ce traitement est souvent appelé *au moins une fois*. Chaque message est traité au moins une fois, mais dans certaines circonstances, un même message peut être remis une nouvelle fois. Toutefois, dans certaines circonstances, un même message peut être remis une nouvelle fois. Cette logique est généralement obtenue grâce à la propriété `message_id` du message, qui reste constante entre chaque nouvelle tentative de remise.

## <a name="delete-topics-and-subscriptions"></a>Suppression de rubriques et d'abonnements
Les rubriques et abonnements sont persistants, sauf si la [propriété AutoDeleteOnIdle](/dotnet/api/microsoft.servicebus.messaging.subscriptiondescription.autodeleteonidle) est définie. Ils peuvent être supprimés via le [portail Microsoft Azure][Azure portal] ou par programmation. L’exemple suivant montre comment supprimer la rubrique `test-topic`.

```ruby
azure_service_bus_service.delete_topic("test-topic")
```

La suppression d’une rubrique a également pour effet de supprimer les abonnements inscrits dans la rubrique. Les abonnements peuvent aussi être supprimés de manière indépendante. Le code suivant montre comment supprimer l’abonnement `high-messages` de la rubrique `test-topic` :

```ruby
azure_service_bus_service.delete_subscription("test-topic", "high-messages")
```

> [!NOTE]
> Vous pouvez gérer les ressources Service Bus à l'aide de [Service Bus Explorer](https://github.com/paolosalvatori/ServiceBusExplorer/). Service Bus Explorer permet aux utilisateurs de se connecter à un espace de noms Service Bus et de gérer les entités de messagerie en toute simplicité. L’outil fournit des fonctionnalités avancées telles que la fonction d’importation/exportation ou la possibilité de tester une rubrique, des files d’attente, des abonnements, des services de relais, des hubs de notification et des hubs d’événements. 

## <a name="next-steps"></a>Étapes suivantes
Maintenant que vous avez appris les principes de base des rubriques Service Bus, consultez ces liens pour en savoir plus.

* Consultez [Files d’attente, rubriques et abonnements](service-bus-queues-topics-subscriptions.md).
* Référence d’API pour [SqlFilter](/dotnet/api/microsoft.servicebus.messaging.sqlfilter).
* Accédez au référentiel du [Kit de développement logiciel (SDK) Azure pour Ruby](https://github.com/Azure/azure-sdk-for-ruby) sur GitHub.

[Azure portal]: https://portal.azure.com