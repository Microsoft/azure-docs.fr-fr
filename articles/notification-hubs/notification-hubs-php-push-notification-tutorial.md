---
title: Utilisation d’Azure Notification Hubs avec PHP
description: Découvrez comment utiliser Azure Notification Hubs à partir d’un serveur principal PHP.
services: notification-hubs
documentationcenter: ''
author: sethmanheim
manager: femila
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: php
ms.devlang: php
ms.topic: article
ms.date: 01/04/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 01/04/2019
ms.openlocfilehash: daebf7c6a5fc9056e16b77a40ee9f90db598749f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87076630"
---
# <a name="how-to-use-notification-hubs-from-php"></a>Utilisation de Notification Hubs à partir de PHP

[!INCLUDE [notification-hubs-backend-how-to-selector](../../includes/notification-hubs-backend-how-to-selector.md)]

Vous pouvez accéder à toutes les fonctionnalités de Notification Hubs à partir d’un serveur principal Java/PHP/Ruby en utilisant l’interface REST des hubs de notifications, comme décrit dans la rubrique MSDN [API REST de Notification Hubs](/previous-versions/azure/reference/dn223264(v=azure.100)).

Dans cette rubrique, nous vous montrons comment :

* créer un client REST pour les fonctionnalités de Notification Hubs en PHP ;
* suivre le tutoriel [Envoyer des notifications Push à des applications iOS avec Azure Notification Hubs](ios-sdk-get-started.md) pour la plateforme mobile de votre choix, en implémentant la partie back-end en PHP.

## <a name="client-interface"></a>Interface client

L’interface client principale peut fournir les mêmes méthodes que le [Kit de développement logiciel (SDK) .NET Notification Hubs](https://msdn.microsoft.com/library/jj933431.aspx), ce qui permet de traduire directement l’ensemble des tutoriels et des exemples actuellement disponibles sur ce site, enrichis par les contributions de la communauté Internet.

Tout le code est disponible dans l' [exemple de wrapper REST PHP].

Par exemple, pour créer un client :

```php
$hub = new NotificationHub("connection string", "hubname");
```

Pour envoyer une notification iOS native :

```php
$notification = new Notification("apple", '{"aps":{"alert": "Hello!"}}');
$hub->sendNotification($notification, null);
```

## <a name="implementation"></a>Implémentation

Si vous ne l’avez pas déjà fait, suivez le [tutoriel de démarrage] jusqu’à la dernière section où vous devez implémenter le back-end.
En outre, si vous le souhaitez, vous pouvez utiliser le code de l’[exemple de wrapper REST PHP] et accéder directement à la section du didacticiel [Suivi du didacticiel](#complete-tutorial).

Tous les détails de l'implémentation d'un wrapper REST complet se trouvent sur [MSDN](/previous-versions/azure/reference/dn530746(v=azure.100)). Dans cette section, nous allons décrire l’implémentation PHP des principales étapes requises pour accéder aux points de terminaison REST de Notification Hubs :

1. Analyse de la chaîne de connexion
2. Génération du jeton d'autorisation
3. Réalisation de l'appel HTTP

### <a name="parse-the-connection-string"></a>Analyse de la chaîne de connexion

Voici la classe principale implémentant le client, dont le constructeur analyse la chaîne de connexion :

```php
class NotificationHub {
    const API_VERSION = "?api-version=2013-10";

    private $endpoint;
    private $hubPath;
    private $sasKeyName;
    private $sasKeyValue;

    function __construct($connectionString, $hubPath) {
        $this->hubPath = $hubPath;

        $this->parseConnectionString($connectionString);
    }

    private function parseConnectionString($connectionString) {
        $parts = explode(";", $connectionString);
        if (sizeof($parts) != 3) {
            throw new Exception("Error parsing connection string: " . $connectionString);
        }

        foreach ($parts as $part) {
            if (strpos($part, "Endpoint") === 0) {
                $this->endpoint = "https" . substr($part, 11);
            } else if (strpos($part, "SharedAccessKeyName") === 0) {
                $this->sasKeyName = substr($part, 20);
            } else if (strpos($part, "SharedAccessKey") === 0) {
                $this->sasKeyValue = substr($part, 16);
            }
        }
    }
}
```

### <a name="create-a-security-token"></a>Créer un jeton de sécurité

Reportez-vous à la documentation Azure pour plus d’informations sur la façon de [créer un jeton de sécurité SAP](/previous-versions/azure/reference/dn495627(v=azure.100)#create-sas-security-token).

Ajoutez la méthode `generateSasToken` à la classe `NotificationHub` pour créer le jeton à partir de l’URI de la demande actuelle et des informations d’identification extraites de la chaîne de connexion.

```php
private function generateSasToken($uri) {
    $targetUri = strtolower(rawurlencode(strtolower($uri)));

    $expires = time();
    $expiresInMins = 60;
    $expires = $expires + $expiresInMins * 60;
    $toSign = $targetUri . "\n" . $expires;

    $signature = rawurlencode(base64_encode(hash_hmac('sha256', $toSign, $this->sasKeyValue, TRUE)));

    $token = "SharedAccessSignature sr=" . $targetUri . "&sig="
                . $signature . "&se=" . $expires . "&skn=" . $this->sasKeyName;

    return $token;
}
```

### <a name="send-a-notification"></a>Envoi d’une notification

Commençons par définir une classe représentant une notification.

```php
class Notification {
    public $format;
    public $payload;

    # array with keynames for headers
    # Note: Some headers are mandatory: Windows: X-WNS-Type, WindowsPhone: X-NotificationType
    # Note: For Apple you can set Expiry with header: ServiceBusNotification-ApnsExpiry in W3C DTF, YYYY-MM-DDThh:mmTZD (for example, 1997-07-16T19:20+01:00).
    public $headers;

    function __construct($format, $payload) {
        if (!in_array($format, ["template", "apple", "windows", "fcm", "windowsphone"])) {
            throw new Exception('Invalid format: ' . $format);
        }

        $this->format = $format;
        $this->payload = $payload;
    }
}
```

Cette classe est un conteneur pour un corps de notification natif, ou un ensemble de propriétés dans le cas d’un modèle de notification, et un ensemble d’en-têtes contenant le format (plateforme native ou modèle) et des propriétés propres à la plateforme (notamment la propriété d’expiration d’Apple et les en-têtes WNS).

Pour connaître toutes les options disponibles, consultez la [documentation sur les API REST de Notification Hubs](/previous-versions/azure/reference/dn495827(v=azure.100)) et les formats spécifiques des plateformes de notification.

Munis de cette classe, nous pouvons à présent écrire les méthodes d’envoi des notifications à l’intérieur de la classe `NotificationHub` :

```php
public function sendNotification($notification, $tagsOrTagExpression="") {
    if (is_array($tagsOrTagExpression)) {
        $tagExpression = implode(" || ", $tagsOrTagExpression);
    } else {
        $tagExpression = $tagsOrTagExpression;
    }

    # build uri
    $uri = $this->endpoint . $this->hubPath . "/messages" . NotificationHub::API_VERSION;
    $ch = curl_init($uri);

    if (in_array($notification->format, ["template", "apple", "fcm"])) {
        $contentType = "application/json";
    } else {
        $contentType = "application/xml";
    }

    $token = $this->generateSasToken($uri);

    $headers = [
        'Authorization: '.$token,
        'Content-Type: '.$contentType,
        'ServiceBusNotification-Format: '.$notification->format
    ];

    if ("" !== $tagExpression) {
        $headers[] = 'ServiceBusNotification-Tags: '.$tagExpression;
    }

    # add headers for other platforms
    if (is_array($notification->headers)) {
        $headers = array_merge($headers, $notification->headers);
    }

    curl_setopt_array($ch, array(
        CURLOPT_POST => TRUE,
        CURLOPT_RETURNTRANSFER => TRUE,
        CURLOPT_SSL_VERIFYPEER => FALSE,
        CURLOPT_HTTPHEADER => $headers,
        CURLOPT_POSTFIELDS => $notification->payload
    ));

    // Send the request
    $response = curl_exec($ch);

    // Check for errors
    if($response === FALSE){
        throw new Exception(curl_error($ch));
    }

    $info = curl_getinfo($ch);

    if ($info['http_code'] <> 201) {
        throw new Exception('Error sending notification: '. $info['http_code'] . ' msg: ' . $response);
    }
} 
```

Les méthodes ci-dessus envoient une demande POST HTTP au point de terminaison `/messages` de votre hub de notification, avec le corps et les en-têtes corrects pour envoyer la notification.

## <a name="complete-the-tutorial"></a><a name="complete-tutorial"></a>Suivi du didacticiel

Vous pouvez à présent suivre le tutoriel de prise en main en envoyant la notification à partir d’un serveur principal PHP.

Initialisez votre client Notification Hubs (remplacez la chaîne de connexion et le nom du hub comme indiqué dans le [tutoriel de démarrage]) :

```php
$hub = new NotificationHub("connection string", "hubname");
```

Ajoutez ensuite le code d'envoi en fonction de la plateforme mobile cible.

### <a name="windows-store-and-windows-phone-81-non-silverlight"></a>Windows Store et Windows Phone 8.1 (non-Silverlight)

```php
$toast = '<toast><visual><binding template="ToastText01"><text id="1">Hello from PHP!</text></binding></visual></toast>';
$notification = new Notification("windows", $toast);
$notification->headers[] = 'X-WNS-Type: wns/toast';
$hub->sendNotification($notification, null);
```

### <a name="ios"></a>iOS

```php
$alert = '{"aps":{"alert":"Hello from PHP!"}}';
$notification = new Notification("apple", $alert);
$hub->sendNotification($notification, null);
```

### <a name="android"></a>Android

```php
$message = '{"data":{"msg":"Hello from PHP!"}}';
$notification = new Notification("fcm", $message);
$hub->sendNotification($notification, null);
```

### <a name="windows-phone-80-and-81-silverlight"></a>Windows Phone 8.0 et 8.1 Silverlight

```php
$toast = '<?xml version="1.0" encoding="utf-8"?>' .
            '<wp:Notification xmlns:wp="WPNotification">' .
               '<wp:Toast>' .
                    '<wp:Text1>Hello from PHP!</wp:Text1>' .
               '</wp:Toast> ' .
            '</wp:Notification>';
$notification = new Notification("windowsphone", $toast);
$notification->headers[] = 'X-WindowsPhone-Target : toast';
$notification->headers[] = 'X-NotificationClass : 2';
$hub->sendNotification($notification, null);
```

### <a name="kindle-fire"></a>Kindle Fire

```php
$message = '{"data":{"msg":"Hello from PHP!"}}';
$notification = new Notification("adm", $message);
$hub->sendNotification($notification, null);
```

L'exécution de votre code PHP produit normalement une notification qui apparaît sur votre appareil cible.

## <a name="next-steps"></a>Étapes suivantes

Dans cette rubrique, nous vous avons montré comment créer un client REST Java simple pour Notification Hubs. À partir de là, vous pouvez :

* télécharger l'intégralité de l' [exemple de wrapper REST PHP], qui contient tout le code ci-dessus ;
* poursuivre l'apprentissage de la fonction de balisage dans le [didacticiel Nouvelles de dernière minute] ;
* vous familiariser avec l'envoi de notifications Push à des utilisateurs individuels dans le [didacticiel Envoi de notifications à des utilisateurs]

Pour plus d’informations, consultez également le [Centre pour développeurs PHP](https://azure.microsoft.com/develop/php/).

[exemple de wrapper REST PHP]: https://github.com/Azure/azure-notificationhubs-samples/tree/master/notificationhubs-rest-php
[Envoyer des notifications Push à des applications iOS avec Azure Notification Hubs](ios-sdk-get-started.md))
