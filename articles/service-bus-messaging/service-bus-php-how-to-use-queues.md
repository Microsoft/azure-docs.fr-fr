---
title: Guide pratique pour utiliser les files d’attente Azure Service Bus avec PHP
description: Dans ce tutoriel, vous allez apprendre à créer des applications PHP afin d’envoyer des messages à une file d’attente Service Bus.
services: service-bus-messaging
ms.devlang: PHP
ms.topic: quickstart
ms.date: 06/23/2020
ms.openlocfilehash: a7e0d1fa321f1b7c1295b5a640fe78b46adf1c72
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "85341115"
---
# <a name="quickstart-how-to-use-service-bus-queues-with-php"></a>Démarrage rapide : Utilisation des files d’attente Service Bus avec PHP
[!INCLUDE [service-bus-selector-queues](../../includes/service-bus-selector-queues.md)]

Dans ce tutoriel, vous allez apprendre à créer des applications PHP afin d’envoyer des messages à une file d’attente Service Bus. 

## <a name="prerequisites"></a>Prérequis
1. Un abonnement Azure. Pour suivre ce tutoriel, vous avez besoin d’un compte Azure. Vous pouvez [activer les avantages de votre abonnement MSDN](https://azure.microsoft.com/pricing/member-offers/credit-for-visual-studio-subscribers/?WT.mc_id=A85619ABF) ou [vous inscrire pour un compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A85619ABF).
2. Si vous n’avez pas de file d’attente à utiliser, suivez les étapes de l’article [Utiliser le portail Azure pour créer une file d’attente Service Bus](service-bus-quickstart-portal.md) pour créer une file d’attente.
    1. Consultez la **vue d’ensemble** rapide des **files d’attente** Service Bus. 
    2. Créez un **espace de noms** Service Bus. 
    3. Obtenez la **chaîne de connexion**. 

        > [!NOTE]
        > Dans ce tutoriel, vous allez créer un **file d’attente** dans l’espace de noms Service Bus à l’aide du PHP. 
3. [Kit de développement logiciel (SDK) Azure pour PHP](https://github.com/Azure/azure-sdk-for-php)

## <a name="create-a-php-application"></a>Création d'une application PHP
Le référencement de classes issues du [Kit de développement logiciel (SDK) Azure pour PHP](https://github.com/Azure/azure-sdk-for-php) dans votre code constitue la seule exigence pour créer une application PHP qui accède au service blob Azure. Vous pouvez utiliser tous les outils de développement pour créer votre application, ou Bloc-notes.

> [!NOTE]
> [L’extension OpenSSL](https://php.net/openssl) doit également être installée et activée dans votre installation PHP.

Dans ce guide, vous allez utiliser les fonctionnalités du service qui peuvent être appelées dans une application PHP en local, ou dans le code s’exécutant dans un rôle web, un rôle de travail ou un site web Azure.

## <a name="get-the-azure-client-libraries"></a>Obtention des bibliothèques clientes Azure
[!INCLUDE [get-client-libraries](../../includes/get-client-libraries.md)]

## <a name="configure-your-application-to-use-service-bus"></a>Configuration de votre application pour l’utilisation de Service Bus
Pour utiliser des API de file d'attente Service Bus, procédez comme suit :

1. Référencez le fichier de chargeur automatique à l’aide de l’instruction [require_once][require_once].
2. référencer toute classe que vous êtes susceptible d'utiliser.

L’exemple suivant montre comment inclure le fichier du chargeur automatique et référencer la classe `ServicesBuilder`.

> [!NOTE]
> Cet exemple et d'autres exemples de cet article partent du principe que vous avez installé les bibliothèques clientes PHP pour Azure via Composer. Si vous avez installé les bibliothèques manuellement ou en tant que package PEAR, vous devez référencer le fichier de chargeur automatique **WindowsAzure.php**.
> 
> 

```php
require_once 'vendor/autoload.php';
use WindowsAzure\Common\ServicesBuilder;
```

Dans les exemples ci-dessous, l'instruction `require_once` s'affichera toujours, mais seules les classes nécessaires aux besoins de l'exemple à exécuter sont référencées.

## <a name="set-up-a-service-bus-connection"></a>Configuration d’une connexion Service Bus
Pour instancier un client Service Bus, vous devez disposer au préalable d'une chaîne de connexion valide au format suivant :

```
Endpoint=[yourEndpoint];SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=[Primary Key]
```

où `Endpoint` est généralement au format `[yourNamespace].servicebus.windows.net`.

Pour créer un client de service Azure, vous devez utiliser la classe `ServicesBuilder`. Vous pouvez :

* Lui passer directement la chaîne de connexion.
* utiliser **CloudConfigurationManager (CCM)** pour vérifier plusieurs sources externes pour la chaîne de connexion :
  * par défaut une source externe est prise en charge : variables d'environnement
  * Vous pouvez ajouter de nouvelles sources en étendant la classe `ConnectionStringSource`.

Dans les exemples ci-dessous, la chaîne de connexion est passée directement.

```php
require_once 'vendor/autoload.php';

use WindowsAzure\Common\ServicesBuilder;

$connectionString = "Endpoint=[yourEndpoint];SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=[Primary Key]";

$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);
```

## <a name="create-a-queue"></a>Créer une file d’attente
Vous pouvez effectuer des opérations de gestion pour les files d’attente Service Bus en utilisant la classe `ServiceBusRestProxy`. Un objet `ServiceBusRestProxy` est construit via la méthode d’usine `ServicesBuilder::createServiceBusService` avec une chaîne de connexion appropriée qui encapsule les autorisations de jeton pour le gérer.

L’exemple suivant montre comment instancier un `ServiceBusRestProxy` et appeler `ServiceBusRestProxy->createQueue` pour créer une file d’attente nommée `myqueue` dans un espace de noms de service `MySBNamespace` :

```php
require_once 'vendor/autoload.php';

use WindowsAzure\Common\ServicesBuilder;
use WindowsAzure\Common\ServiceException;
use WindowsAzure\ServiceBus\Models\QueueInfo;

// Create Service Bus REST proxy.
$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);

try    {
    $queueInfo = new QueueInfo("myqueue");

    // Create queue.
    $serviceBusRestProxy->createQueue($queueInfo);
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
> vous pouvez utiliser la méthode `listQueues` sur les objets `ServiceBusRestProxy` pour vérifier s’il existe déjà une file d’attente d’un nom déterminé dans un espace de noms.
> 
> 

## <a name="send-messages-to-a-queue"></a>Envoi de messages à une file d'attente
Pour envoyer un message à une file d’attente Service Bus, votre application appelle la méthode `ServiceBusRestProxy->sendQueueMessage`. Le code suivant montre comment envoyer un message à la file d’attente `myqueue` créée plus haut dans l’espace de noms de service `MySBNamespace`.

```php
require_once 'vendor/autoload.php';

use WindowsAzure\Common\ServicesBuilder;
use WindowsAzure\Common\ServiceException;
use WindowsAzure\ServiceBus\Models\BrokeredMessage;

// Create Service Bus REST proxy.
$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);

try    {
    // Create message.
    $message = new BrokeredMessage();
    $message->setBody("my message");

    // Send message.
    $serviceBusRestProxy->sendQueueMessage("myqueue", $message);
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

Les messages envoyés aux files d’attente Service Bus (et reçus de celles-ci) sont des instances de la classe [BrokeredMessage][BrokeredMessage]. Les objets [BrokeredMessage][BrokeredMessage] possèdent un ensemble de méthodes et propriétés standard qui stockent des propriétés personnalisées propres à une application ainsi qu’un corps de données d’application arbitraires.

Les files d’attente Service Bus prennent en charge une taille de message maximale de 256 Ko dans le [niveau Standard](service-bus-premium-messaging.md) et de 1 Mo dans le [niveau Premium](service-bus-premium-messaging.md). L’en-tête, qui comprend les propriétés d’application standard et personnalisées, peut avoir une taille maximale de 64 Ko. Si une file d'attente n'est pas limitée par le nombre de messages qu'elle peut contenir, elle l'est en revanche par la taille totale des messages qu'elle contient. La taille maximale de la file d'attente est de 5 Go.

## <a name="receive-messages-from-a-queue"></a>Réception des messages d'une file d'attente

Le moyen le plus simple de recevoir les messages d’une file d’attente est d’utiliser une méthode `ServiceBusRestProxy->receiveQueueMessage`. Les messages peuvent être reçus dans deux modes différents : [*ReceiveAndDelete*](/dotnet/api/microsoft.servicebus.messaging.receivemode) et [*PeekLock*](/dotnet/api/microsoft.servicebus.messaging.receivemode#Microsoft_ServiceBus_Messaging_ReceiveMode_PeekLock). **PeekLock** est la valeur par défaut.

Lorsque le mode [ReceiveAndDelete](/dotnet/api/microsoft.servicebus.messaging.receivemode) est utilisé, la réception est une opération unique : quand Service Bus reçoit une demande de lecture d’un message figurant dans une file d’attente, il marque le message comme consommé et le renvoie à l’application. Le mode [ReceiveAndDelete](/dotnet/api/microsoft.servicebus.messaging.receivemode) est le modèle le plus simple et le mieux adapté aux scénarios dans lesquels une application est capable de tolérer le non-traitement d’un message en cas d’échec. Pour mieux comprendre, imaginez un scénario dans lequel le consommateur émet la demande de réception et subit un incident avant de la traiter. Comme Service Bus a marqué le message comme étant consommé, lorsque l’application redémarre et recommence à consommer des messages, elle manque le message consommé avant l’incident.

En mode [PeekLock](/dotnet/api/microsoft.servicebus.messaging.receivemode#Microsoft_ServiceBus_Messaging_ReceiveMode_PeekLock) par défaut), la réception d’un message devient une opération en deux étapes, ce qui permet de prendre en charge les applications qui ne tolèrent pas les messages manquants. Lorsque Service Bus reçoit une demande, il recherche le message suivant à consommer, le verrouille pour empêcher d'autres consommateurs de le recevoir, puis le renvoie à l'application. Dès que l’application a terminé de traiter le message (ou qu’elle l’a stocké de manière sûre en vue d’un traitement ultérieur), elle effectue la deuxième étape du processus de réception en transmettant le message reçu à `ServiceBusRestProxy->deleteMessage`. Lorsque Service Bus voit l’appel `deleteMessage`, il marque le message comme consommé et le supprime de la file d’attente.

L’exemple ci-dessous montre comment recevoir et traiter un message avec le mode [PeekLock](/dotnet/api/microsoft.servicebus.messaging.receivemode#Microsoft_ServiceBus_Messaging_ReceiveMode_PeekLock) (mode par défaut).

```php
require_once 'vendor/autoload.php';

use WindowsAzure\Common\ServicesBuilder;
use WindowsAzure\Common\ServiceException;
use WindowsAzure\ServiceBus\Models\ReceiveMessageOptions;

// Create Service Bus REST proxy.
$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);

try    {
    // Set the receive mode to PeekLock (default is ReceiveAndDelete).
    $options = new ReceiveMessageOptions();
    $options->setPeekLock();

    // Receive message.
    $message = $serviceBusRestProxy->receiveQueueMessage("myqueue", $options);
    echo "Body: ".$message->getBody()."<br />";
    echo "MessageID: ".$message->getMessageId()."<br />";

    /*---------------------------
        Process message here.
    ----------------------------*/

    // Delete message. Not necessary if peek lock is not set.
    echo "Message deleted.<br />";
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

De même, il faut savoir qu'un message verrouillé dans une file d'attente est assorti d'un délai d'expiration et que si l'application ne parvient pas à traiter le message dans le temps imparti (par exemple, si l'application subit un incident), Service Bus déverrouille le message automatiquement et le rend à nouveau disponible en réception.

Si l’application se bloque après le traitement du message mais avant l’émission de la demande `deleteMessage`, le message est à nouveau transmis à l’application lorsqu’elle redémarre. Ce traitement est souvent appelé *Au moins une fois*. Chaque message est traité au moins une fois, mais dans certaines circonstances, un même message peut être transmis une nouvelle fois. Si le scénario ne peut pas tolérer le traitement en double,l'ajout d'une logique supplémentaire à vos applications pour traiter la remise de messages en double est recommandé. Ceci, grâce à la méthode `getMessageId` du message, qui reste constante entre les tentatives de transmission.

> [!NOTE]
> Vous pouvez gérer les ressources Service Bus à l'aide de [Service Bus Explorer](https://github.com/paolosalvatori/ServiceBusExplorer/). Service Bus Explorer permet aux utilisateurs de se connecter à un espace de noms Service Bus et de gérer les entités de messagerie en toute simplicité. L’outil fournit des fonctionnalités avancées telles que la fonction d’importation/exportation ou la possibilité de tester une rubrique, des files d’attente, des abonnements, des services de relais, des hubs de notification et des hubs d’événements. 

## <a name="next-steps"></a>Étapes suivantes
Les principes de base des files d’attente Service Bus étant appris, consultez [Files d’attente, rubriques et abonnements][Queues, topics, and subscriptions] pour plus d’informations.

Pour plus d’informations, visitez aussi le [Centre pour développeurs PHP](https://azure.microsoft.com/develop/php/).

[BrokeredMessage]: /dotnet/api/microsoft.servicebus.messaging.brokeredmessage
[Queues, topics, and subscriptions]: service-bus-queues-topics-subscriptions.md
[require_once]: https://php.net/require_once


