---
title: Guide pratique pour utiliser les rubriques Azure Service Bus avec PHP
description: Dans ce tutoriel, vous allez apprendre à utiliser les rubriques et abonnements Azure Service Bus à partir d’une application PHP.
ms.devlang: PHP
ms.topic: quickstart
ms.date: 06/23/2020
ms.openlocfilehash: 706f523fdfb3c710bb16b048cfc68ce98875adb1
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "88066200"
---
# <a name="quickstart-how-to-use-service-bus-topics-and-subscriptions-with-php"></a>Démarrage rapide : Utilisation des rubriques et abonnements Service Bus avec PHP

[!INCLUDE [service-bus-selector-topics](../../includes/service-bus-selector-topics.md)]

Cet article vous montre comment utiliser les rubriques et les abonnements Service Bus. Les exemples sont écrits en PHP et utilisent le [Kit de développement logiciel (SDK) Azure pour PHP](https://github.com/Azure/azure-sdk-for-php). Parmi les scénarios présentés :

- Création de rubriques et d’abonnements 
- Création de filtres d’abonnement 
- Envoi de messages à une rubrique 
- Réception de messages à partir d’un abonnement
- Suppression de rubriques et d’abonnements

## <a name="prerequisites"></a>Prérequis
1. Un abonnement Azure. Pour suivre ce tutoriel, vous avez besoin d’un compte Azure. Vous pouvez activer les [avantages de votre abonnement Visual Studio ou MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A85619ABF) ou vous inscrire pour un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A85619ABF).
2. Suivez les étapes dans [Démarrage rapide : utiliser le portail Azure pour créer une rubrique Service Bus et des abonnements à cette rubrique](service-bus-quickstart-topics-subscriptions-portal.md) pour créer un **espace de noms** Service Bus et obtenir la **chaîne de connexion**.

    > [!NOTE]
    > Vous allez créer une **rubrique** et un **abonnement** à la rubrique à l’aide de **PHP** dans ce démarrage rapide. 

## <a name="create-a-php-application"></a>Création d'une application PHP
Le référencement de classes issues du [Kit de développement logiciel (SDK) Azure pour PHP](https://github.com/Azure/azure-sdk-for-php) dans votre code constitue la seule exigence pour créer une application PHP qui accède au service blob Azure. Vous pouvez utiliser tous les outils de développement pour créer votre application, ou Bloc-notes.

> [!NOTE]
> [L’extension OpenSSL](https://php.net/openssl) doit également être installée et activée dans votre installation PHP.
> 
> 

Cet article décrit comment utiliser des fonctionnalités de service qui peuvent être appelées dans une application PHP localement ou dans le code d’un rôle web, d’un rôle de travail ou d’un site web Azure.

## <a name="get-the-azure-client-libraries"></a>Obtention des bibliothèques clientes Azure

### <a name="install-via-composer"></a>Installation via Composer
1. Créez un fichier nommé **composer.json** à la racine de votre projet et ajoutez-y le code suivant :
   
    ```json
    {
      "require": {
        "microsoft/windowsazure": "*"
      }
    }
    ```
2. Téléchargez **[composer.phar][composer-phar]** à la racine du projet.
3. Ouvrez une invite de commande et exécutez la commande suivante à la racine du projet
   
    ```
    php composer.phar install
    ```

## <a name="configure-your-application-to-use-service-bus"></a>Configuration de votre application pour l’utilisation de Service Bus
Utilisation des API Service Bus :

1. Référencez le fichier de chargeur automatique à l’aide de l’instruction [require_once][require-once].
2. référencer toute classe que vous êtes susceptible d'utiliser.

L’exemple suivant montre comment inclure le fichier du chargeur automatique et référencer la classe **ServiceBusService**.

> [!NOTE]
> Cet exemple et d'autres exemples de cet article partent du principe que vous avez installé les bibliothèques clientes PHP pour Azure via Composer. Si vous avez installé les bibliothèques manuellement ou en tant que package PEAR, vous devez référencer le fichier de chargeur automatique **WindowsAzure.php**.
> 
> 

```php
require_once 'vendor/autoload.php';
use WindowsAzure\Common\ServicesBuilder;
```

Dans les exemples suivants, l’instruction `require_once` s’affiche toujours, mais seules les classes nécessaires aux besoins de l’exemple à exécuter sont référencées.

## <a name="set-up-a-service-bus-connection"></a>Configuration d’une connexion Service Bus
Pour instancier un client Service Bus, vous devez disposer au préalable d'une chaîne de connexion valide au format suivant :

```
Endpoint=[yourEndpoint];SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=[Primary Key]
```

où `Endpoint` est généralement au format `https://[yourNamespace].servicebus.windows.net`.

Pour créer un client de service Azure, vous devez utiliser la classe `ServicesBuilder`. Vous pouvez :

* Lui passer directement la chaîne de connexion.
* utiliser **CloudConfigurationManager (CCM)** pour vérifier plusieurs sources externes pour la chaîne de connexion :
  * par défaut une source externe est prise en charge : variables d'environnement.
  * Vous pouvez ajouter de nouvelles sources en étendant la classe `ConnectionStringSource`.

Dans les exemples ci-dessous, la chaîne de connexion est passée directement.

```php
require_once 'vendor/autoload.php';

use WindowsAzure\Common\ServicesBuilder;

$connectionString = "Endpoint=[yourEndpoint];SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=[Primary Key]";

$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);
```

## <a name="create-a-topic"></a>Création d'une rubrique
Vous pouvez effectuer des opérations de gestion pour les rubriques Service Bus avec la classe `ServiceBusRestProxy`. Un objet `ServiceBusRestProxy` est construit via la méthode d’usine `ServicesBuilder::createServiceBusService` avec une chaîne de connexion appropriée qui encapsule les autorisations de jeton pour le gérer.

L’exemple suivant montre comment instancier un `ServiceBusRestProxy` et appeler `ServiceBusRestProxy->createTopic` pour créer une rubrique nommée `mytopic` dans un espace de noms `MySBNamespace` :

```php
require_once 'vendor/autoload.php';

use WindowsAzure\Common\ServicesBuilder;
use WindowsAzure\Common\ServiceException;
use WindowsAzure\ServiceBus\Models\TopicInfo;

// Create Service Bus REST proxy.
$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);

try {
    // Create topic.
    $topicInfo = new TopicInfo("mytopic");
    $serviceBusRestProxy->createTopic($topicInfo);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here: 
    // https://docs.microsoft.com/rest/api/storageservices/Common-REST-API-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

> [!NOTE]
> vous pouvez utiliser la méthode `listTopics` sur les objets `ServiceBusRestProxy` pour vérifier s’il existe déjà une rubrique d’un nom déterminé dans un espace de noms de service.
> 
> 

## <a name="create-a-subscription"></a>Création d’un abonnement
La méthode `ServiceBusRestProxy->createSubscription` permet également de créer des abonnements à une rubrique. Les abonnements sont nommés et peuvent être assortis d'un filtre facultatif qui limite l'ensemble des messages transmis à la file d'attente virtuelle de l'abonnement.

### <a name="create-a-subscription-with-the-default-matchall-filter"></a>Création d’un abonnement avec le filtre par défaut (MatchAll)
Si aucun filtre n’est spécifié lors de la création d’un abonnement, le filtre **MatchAll** (par défaut) est utilisé. Lorsque le filtre **MatchAll** est utilisé, tous les messages publiés dans la rubrique sont placés dans la file d’attente virtuelle de l’abonnement. L’exemple suivant crée un abonnement intitulé `mysubscription` et utilise le filtre par défaut **MatchAll**.

```php
require_once 'vendor/autoload.php';

use WindowsAzure\Common\ServicesBuilder;
use WindowsAzure\Common\ServiceException;
use WindowsAzure\ServiceBus\Models\SubscriptionInfo;

// Create Service Bus REST proxy.
$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);

try {
    // Create subscription.
    $subscriptionInfo = new SubscriptionInfo("mysubscription");
    $serviceBusRestProxy->createSubscription("mytopic", $subscriptionInfo);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here: 
    // https://docs.microsoft.com/rest/api/storageservices/Common-REST-API-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

### <a name="create-subscriptions-with-filters"></a>Création d’abonnements avec des filtres
Vous pouvez également configurer des filtres pour spécifier quels sont les messages, parmi ceux envoyés à une rubrique, qui doivent apparaître dans un abonnement de rubrique spécifique. Parmi les types de filtre pris en charge par les abonnements, [SqlFilter](/dotnet/api/microsoft.servicebus.messaging.sqlfilter) est le plus flexible. Il implémente un sous-ensemble de SQL92. Les filtres SQL opèrent au niveau des propriétés des messages publiés dans la rubrique. Pour plus d’informations sur les filtres SQL, consultez la page [Propriété SqlFilter.SqlExpression][sqlfilter].

> [!NOTE]
> Chaque règle d’un abonnement traite les messages entrants de façon indépendante, ajoutant leurs messages de résultat à l’abonnement. En outre, chaque nouvel abonnement dispose d’un objet **Rule** par défaut avec un filtre qui ajoute tous les messages de la rubrique à l’abonnement. Pour recevoir uniquement les messages correspondant à votre filtre, vous devez supprimer la règle par défaut. Vous pouvez supprimer la règle par défaut à l'aide de la méthode `ServiceBusRestProxy->deleteRule`.
> 
> 

L’exemple suivant crée l’abonnement `HighMessages` avec un objet **SqlFilter** qui ne sélectionne que les messages dont la propriété personnalisée `MessageNumber` a une valeur supérieure à 3. Pour plus d’informations sur l’ajout de propriétés personnalisées à des messages, consultez [Envoyer des messages à une rubrique](#send-messages-to-a-topic).

```php
$subscriptionInfo = new SubscriptionInfo("HighMessages");
$serviceBusRestProxy->createSubscription("mytopic", $subscriptionInfo);

$serviceBusRestProxy->deleteRule("mytopic", "HighMessages", '$Default');

$ruleInfo = new RuleInfo("HighMessagesRule");
$ruleInfo->withSqlFilter("MessageNumber > 3");
$ruleResult = $serviceBusRestProxy->createRule("mytopic", "HighMessages", $ruleInfo);
```

Ce code nécessite l’utilisation d’un espace de noms supplémentaire : `WindowsAzure\ServiceBus\Models\SubscriptionInfo`.

De même, l’exemple suivant crée l’abonnement `LowMessages` avec un objet `SqlFilter` qui ne sélectionne que les messages dont la propriété `MessageNumber` a une valeur inférieure ou égale à 3.

```php
$subscriptionInfo = new SubscriptionInfo("LowMessages");
$serviceBusRestProxy->createSubscription("mytopic", $subscriptionInfo);

$serviceBusRestProxy->deleteRule("mytopic", "LowMessages", '$Default');

$ruleInfo = new RuleInfo("LowMessagesRule");
$ruleInfo->withSqlFilter("MessageNumber <= 3");
$ruleResult = $serviceBusRestProxy->createRule("mytopic", "LowMessages", $ruleInfo);
```

À présent, dès lors qu’un message est envoyé vers la rubrique `mytopic`, il est toujours remis aux destinataires abonnés à l’abonnement `mysubscription` et est remis de manière sélective aux destinataires abonnés aux abonnements `HighMessages` et `LowMessages` (en fonction du contenu du message).

## <a name="send-messages-to-a-topic"></a>Envoi de messages à une rubrique
Pour envoyer un message à une rubrique Service Bus, votre application appelle la méthode `ServiceBusRestProxy->sendTopicMessage`. Le code suivant montre comment envoyer un message à la rubrique `mytopic` créée plus haut dans l’espace de noms de service `MySBNamespace`.

```php
require_once 'vendor/autoload.php';

use WindowsAzure\Common\ServicesBuilder;
use WindowsAzure\Common\ServiceException;
use WindowsAzure\ServiceBus\Models\BrokeredMessage;

// Create Service Bus REST proxy.
$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);

try {
    // Create message.
    $message = new BrokeredMessage();
    $message->setBody("my message");

    // Send message.
    $serviceBusRestProxy->sendTopicMessage("mytopic", $message);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here: 
    // https://docs.microsoft.com/rest/api/storageservices/Common-REST-API-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

Les messages envoyés aux rubriques Service Bus sont des instances de la classe [BrokeredMessage][BrokeredMessage]. Les objets [BrokeredMessage][BrokeredMessage] possèdent un ensemble de propriétés et de méthodes standard ainsi que des propriétés permettant de conserver les propriétés personnalisées propres à une application. L’exemple suivant montre comment envoyer cinq messages de test à la rubrique `mytopic` créée précédemment. La méthode `setProperty` sert à ajouter une propriété personnalisée (`MessageNumber`) à chaque message. La valeur de la propriété `MessageNumber` varie pour chaque message (cela peut être utilisé pour déterminer les abonnements qui le reçoivent, comme indiqué dans la section [Création d’un abonnement](#create-a-subscription)) :

```php
for($i = 0; $i < 5; $i++){
    // Create message.
    $message = new BrokeredMessage();
    $message->setBody("my message ".$i);

    // Set custom property.
    $message->setProperty("MessageNumber", $i);

    // Send message.
    $serviceBusRestProxy->sendTopicMessage("mytopic", $message);
}
```

Les rubriques Service Bus prennent en charge une taille de message maximale de 256 Ko dans le [niveau Standard](service-bus-premium-messaging.md) et de 1 Mo dans le [niveau Premium](service-bus-premium-messaging.md). L’en-tête, qui comprend les propriétés d’application standard et personnalisées, peut avoir une taille maximale de 64 Ko. Si une rubrique n'est pas limitée par le nombre de messages qu'elle peut contenir, elle l'est en revanche par la taille totale des messages qu'elle contient. La taille maximale de la rubrique est de 5 Go. Pour plus d’informations sur les quotas, consultez [Quotas Service Bus][Service Bus quotas].

## <a name="receive-messages-from-a-subscription"></a>Réception des messages d’un abonnement
Le meilleur moyen pour recevoir les messages d’un abonnement est d’utiliser une méthode `ServiceBusRestProxy->receiveSubscriptionMessage`. Les messages peuvent être reçus dans deux modes différents : [*ReceiveAndDelete* et *PeekLock*](/dotnet/api/microsoft.servicebus.messaging.receivemode). **PeekLock** est la valeur par défaut.

Lorsque le mode [ReceiveAndDelete](/dotnet/api/microsoft.servicebus.messaging.receivemode) est utilisé, la réception est une opération unique : lorsque Service Bus reçoit une demande de lecture pour un message figurant dans un abonnement, il marque le message comme étant consommé et le renvoie à l’application. Le mode [ReceiveAndDelete](/dotnet/api/microsoft.servicebus.messaging.receivemode) * est le modèle le plus simple et le mieux adapté aux scénarios dans lesquels une application peut tolérer le non-traitement d’un message après un échec. Pour mieux comprendre, imaginez un scénario dans lequel le consommateur émet la demande de réception et subit un incident avant de la traiter. Étant donné que Service Bus a marqué le message comme étant consommé, quand l’application redémarre et recommence à consommer des messages, elle a manqué le message consommé avant l’incident.

En mode [PeekLock](/dotnet/api/microsoft.servicebus.messaging.receivemode) par défaut), la réception d’un message devient une opération en deux étapes, ce qui permet de prendre en charge les applications qui ne tolèrent pas les messages manquants. Lorsque Service Bus reçoit une demande, il recherche le prochain message à consommer, le verrouille pour empêcher d'autres consommateurs de le recevoir, puis le renvoie à l'application. Dès que l’application a terminé de traiter le message (ou qu’elle l’a stocké de manière sûre en vue d’un traitement ultérieur), elle effectue la deuxième étape du processus de réception en transmettant le message reçu à `ServiceBusRestProxy->deleteMessage`. Lorsque Service Bus voit l’appel `deleteMessage`, il marque le message comme consommé et le supprime de la file d’attente.

L’exemple ci-dessous montre comment recevoir et traiter un message avec le mode [PeekLock](/dotnet/api/microsoft.servicebus.messaging.receivemode) (mode par défaut). 

```php
require_once 'vendor/autoload.php';

use WindowsAzure\Common\ServicesBuilder;
use WindowsAzure\Common\ServiceException;
use WindowsAzure\ServiceBus\Models\ReceiveMessageOptions;

// Create Service Bus REST proxy.
$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);

try {
    // Set receive mode to PeekLock (default is ReceiveAndDelete)
    $options = new ReceiveMessageOptions();
    $options->setPeekLock();

    // Get message.
    $message = $serviceBusRestProxy->receiveSubscriptionMessage("mytopic", "mysubscription", $options);

    echo "Body: ".$message->getBody()."<br />";
    echo "MessageID: ".$message->getMessageId()."<br />";

    /*---------------------------
        Process message here.
    ----------------------------*/

    // Delete message. Not necessary if peek lock is not set.
    echo "Deleting message...<br />";
    $serviceBusRestProxy->deleteMessage($message);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://docs.microsoft.com/rest/api/storageservices/Common-REST-API-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

## <a name="how-to-handle-application-crashes-and-unreadable-messages"></a>Gestion des blocages d’application et des messages illisibles
Service Bus intègre des fonctionnalités destinées à faciliter la récupération à la suite d’erreurs survenues dans votre application ou de difficultés à traiter un message. Si une application de réception ne parvient pas à traiter le message pour une raison quelconque, elle peut appeler la méthode `unlockMessage` sur le message reçu (au lieu de la méthode `deleteMessage`). Service Bus déverrouille alors le message dans la file d’attente et le rend à nouveau disponible en réception, pour la même application consommatrice ou pour une autre.

De même, il faut savoir qu’un message verrouillé dans une file d’attente est assorti d’un délai d’expiration et que si l’application ne parvient pas à traiter le message dans le temps imparti (par exemple, si l’application subit un incident), Service Bus déverrouille le message automatiquement et le rend à nouveau disponible en réception.

Si l’application se bloque après le traitement du message mais avant l’émission de la demande `deleteMessage`, le message est à nouveau transmis à l’application lorsqu’elle redémarre. Ce type de traitement est souvent appelé *au moins une fois*. Chaque message est traité au moins une fois, mais dans certaines circonstances, un même message peut être remis une nouvelle fois. Si le scénario ne peut pas tolérer le traitement en double, les développeurs d'application doivent ajouter une logique supplémentaire aux applications pour traiter la remise de messages en double, Cette logique est généralement obtenue grâce à la méthode `getMessageId` du message, qui reste constante entre chaque nouvelle tentative de remise.

## <a name="delete-topics-and-subscriptions"></a>Suppression de rubriques et d'abonnements
Pour supprimer une rubrique ou un abonnement, utilisez les méthodes `ServiceBusRestProxy->deleteTopic` ou `ServiceBusRestProxy->deleteSubscripton` respectivement. La suppression d’une rubrique a également pour effet de supprimer les abonnements inscrits dans la rubrique.

L’exemple suivant montre comment supprimer une rubrique nommée `mytopic` et ses abonnements inscrits.

```php
require_once 'vendor/autoload.php';

use WindowsAzure\ServiceBus\ServiceBusService;
use WindowsAzure\ServiceBus\ServiceBusSettings;
use WindowsAzure\Common\ServiceException;

// Create Service Bus REST proxy.
$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);

try {
    // Delete topic.
    $serviceBusRestProxy->deleteTopic("mytopic");
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here: 
    // https://docs.microsoft.com/rest/api/storageservices/Common-REST-API-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

La méthode `deleteSubscription` permet de supprimer un abonnement de façon indépendante :

```php
$serviceBusRestProxy->deleteSubscription("mytopic", "mysubscription");
```

> [!NOTE]
> Vous pouvez gérer les ressources Service Bus à l'aide de [Service Bus Explorer](https://github.com/paolosalvatori/ServiceBusExplorer/). Service Bus Explorer permet aux utilisateurs de se connecter à un espace de noms Service Bus et de gérer les entités de messagerie en toute simplicité. L’outil fournit des fonctionnalités avancées telles que la fonction d’importation/exportation ou la possibilité de tester une rubrique, des files d’attente, des abonnements, des services de relais, des hubs de notification et des hubs d’événements. 

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations, consultez [Files d’attente, rubriques et abonnements][Queues, topics, and subscriptions].

[BrokeredMessage]: /dotnet/api/microsoft.servicebus.messaging.brokeredmessage
[Queues, topics, and subscriptions]: service-bus-queues-topics-subscriptions.md
[sqlfilter]: /dotnet/api/microsoft.servicebus.messaging.sqlfilter
[require-once]: https://php.net/require_once
[Service Bus quotas]: service-bus-quotas.md