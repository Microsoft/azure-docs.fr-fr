---
title: Appel téléphonique à partir de Twilio (PHP) | Microsoft Docs
description: Découvrez comment passer un appel téléphonique et envoyer un SMS avec le service d'API Twilio sur Azure. Les exemples concernent une applications PHP.
documentationcenter: php
services: ''
author: georgewallace
ms.assetid: 44e35adc-be06-4700-beee-8c9e2c20c540
ms.service: multiple
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: PHP
ms.topic: article
ms.date: 11/25/2014
ms.author: gwallace
ms.openlocfilehash: ef0642b26898249d78f5103e8a8f80198887bed9
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95542522"
---
# <a name="how-to-make-a-phone-call-using-twilio-in-a-php-application-on-azure"></a>Exécution d'un appel téléphonique à l'aide de Twilio dans une application PHP sur Azure
L'exemple qui suit montre comment utiliser Twilio pour passer un appel depuis une page web PHP hébergée sur Azure. L’application finale demande à l'utilisateur les valeurs de l’appel téléphonique, comme illustré dans la capture d’écran qui suit.

![Formulaire d'appel Azure avec Twilio et PHP][twilio_php]

Pour utiliser le code de cette rubrique, vous devrez effectuer les opérations suivantes :

1. Obtenir un compte Twilio et un jeton d’authentification auprès de la [console Twilio][twilio_console]. Pour bien démarrer avec Twilio, évaluez les tarifs sur le site [https://www.twilio.com/pricing][twilio_pricing]. Vous pouvez demander un compte d'évaluation gratuit sous [https://www.twilio.com/try-twilio][try_twilio].
2. Obtenez la [bibliothèque Twilio pour PHP](https://github.com/twilio/twilio-php) ou installez-la sous forme de package PEAR. Pour plus d’informations, consultez le fichier [Lisezmoi](https://github.com/twilio/twilio-php/blob/master/README.md).
3. Installer le kit SDK Azure pour PHP. 
<!-- For an overview of the SDK and instructions on installing it, see [Set up the Azure SDK for PHP](./app-service/quickstart-php.md) -->

## <a name="create-a-web-form-for-making-a-call"></a>Création d'un formulaire web pour passer un appel
Le code HTML qui suit présente la conception d'une page web (**callform.html**) qui extrait les données des utilisateurs pour passer un appel :

```html
<!DOCTYPE html>
<html>
<head>
  <title>Automated call form</title>
</head>
<body>
  <h1>Automated Call Form</h1>
  <p>Fill in all fields and click <b>Make this call</b>.</p>
  <form action="makecall.php" method="post">
    <table>
      <tr>
        <td>To:</td>
        <td><input name="callTo" size="50" type="text" value=""></td>
      </tr>
      <tr>
        <td>From:</td>
        <td><input name="callFrom" size="50" type="text" value=""></td>
      </tr>
      <tr>
        <td>Call message:</td>
        <td><input name="callText" size="100" type="text" value="Hello. This is the call text. Good bye."></td>
      </tr>
      <tr>
        <td colspan="2"><input type="submit" value="Make this call"></td>
      </tr>
    </table>
  </form><br>
</body>
</html>
```

## <a name="create-the-code-to-make-the-call"></a>Création du code pour passer l'appel
Le code qui suit montre comment créer **makecall.php** qui est appelé quand l’utilisateur envoie le formulaire affiché par **callform.html**. Le code présenté ici crée le message d'appel et passe l'appel. Veillez également à utiliser vos compte Twilio et jeton d’authentification à partir de la [console Twilio][twilio_console] plutôt que les valeurs d’espace réservé affectées à **$sid** et **$token** dans le code ci-dessous.

```html
<html>
<head><title>Making call...</title></head>
<body>
<p>Your call is being made.</p>

<?php
require_once 'path/to/vendor/autoload.php';

$sid   = "your_account_sid";
$token = "your_authentication_token";

$from_number = $_POST['callFrom']; // Calls must be made from a registered Twilio number.
$to_number   = $_POST['callTo'];
$message     = $_POST['callText'];

$client = new Twilio\Rest\Client($sid, $token);

$call = $client->calls->create(
            $to_number,
            $from_number,
            array('url' => https://twimlets.com/message?Message=' . urlencode($message))
        );

echo "Call status: " . $call->status . "<br />";
echo "URI resource: " . $call->uri . "<br />";
?>
</body>
</html>
```

En plus de passer l’appel, **makecall.php** affiche des métadonnées sur l’appel, comme illustré dans l’image ci-dessous. Pour plus d’informations sur l’appel de métadonnées, consultez [https://www.twilio.com/docs/api/rest/call#instance-properties][twilio_call_properties].

![Réponse d'appel Azure avec Twilio et PHP][twilio_php_response]

## <a name="run-the-application"></a>Exécution de l'application
L’étape suivante consiste à [déployer votre application sur Azure Web Apps avec Git](app-service/quickstart-php.md) (bien que toutes les informations fournies ne soient pas utiles). 

## <a name="next-steps"></a>Étapes suivantes
Ce code vous est fourni afin de vous présenter les fonctions de base de l'utilisation de Twilio dans PHP sur Azure. Avant d’effectuer le déploiement de production sur Azure, vous pouvez ajouter d’autres fonctionnalités telles que la gestion des erreurs. Par exemple :

* Au lieu d'utiliser un formulaire web, vous pouvez utiliser des objets blob de stockage Azure ou SQL Database pour stocker les numéros de téléphone et le texte des appels. Pour plus d’informations sur l’utilisation d’objets blob de stockage Azure dans PHP, consultez la page [Utilisation du stockage Azure dans les applications PHP][howto_blob_storage_php]. Pour plus d’informations sur l’utilisation de la base de données SQL dans PHP, consultez la page [Utilisation de la base de données SQL avec les applications PHP][howto_sql_azure_php].
* Le code **makecall.php** utilise une URL Twilio ([https://twimlets.com/message][twimlet_message_url]) pour fournir une réponse TwiML (Twilio Markup Language) qui précise à Twilio comment traiter l’appel. Par exemple, le code TwiML renvoyé peut contenir un verbe `<Say>` qui provoque la lecture audible du texte pour le destinataire de l'appel. Au lieu d’utiliser l’URL fournie par Twilio, vous pouvez concevoir votre propre service pour répondre à la demande de Twilio. Pour plus d’informations, consultez la page [Utilisation de Twilio pour les fonctionnalités vocales et de SMS dans PHP][howto_twilio_voice_sms_php]. Des informations supplémentaires sur TwiML sont disponibles à l’adresse [https://www.twilio.com/docs/api/twiml][twiml]. Des informations supplémentaires sur `<Say>` et sur d’autres verbes Twilio sont disponibles à l’adresse [https://www.twilio.com/docs/api/twiml/say][twilio_say].
* Lisez les instructions de sécurité Twilio disponibles à l’adresse [https://www.twilio.com/docs/security][twilio_docs_security].

Pour obtenir des informations supplémentaires sur Twilio, consultez [https://www.twilio.com/docs][twilio_docs].

## <a name="see-also"></a>Voir aussi
* [Utilisation de Twilio pour les fonctionnalités vocales et de SMS dans PHP](partner-twilio-php-how-to-use-voice-sms.md)

[twilio_console]: https://www.twilio.com/console
[twilio_pricing]: https://www.twilio.com/pricing
[try_twilio]: https://www.twilio.com/try-twilio
[twilio_api]: https://www.twilio.com/docs/api
[verify_phone]: https://www.twilio.com/console/phone-numbers/verified
[twimlet_message_url]: https://twimlets.com/message
[twiml]: https://www.twilio.com/docs/api/twiml
[twilio_api_service]: https://api.twilio.com
[build_php_azure_app]: http://azurephp.interoperabilitybridges.com/articles/build-and-deploy-a-windows-azure-php-application
[howto_twilio_voice_sms_php]: partner-twilio-php-how-to-use-voice-sms.md
[howto_blob_storage_php]: ./storage/blobs/storage-quickstart-blobs-php.md
[howto_sql_azure_php]: ./azure-sql/database/connect-query-content-reference-guide.md
[twilio_call_properties]: https://www.twilio.com/docs/api/rest/call#instance-properties
[twilio_docs_security]: https://www.twilio.com/docs/security
[twilio_docs]: https://www.twilio.com/docs
[twilio_say]: https://www.twilio.com/docs/api/twiml/say
[ssl_validation]: http://readthedocs.org/docs/twilio-php/en/latest/usage/rest.html
[twilio_php]: ./media/partner-twilio-php-make-phone-call/WA_TwilioPHPCallForm.jpg
[twilio_php_response]: ./media/partner-twilio-php-make-phone-call/WA_TwilioPHPMakeCall.jpg
[twilio_php_github]: https://github.com/twilio/twilio-php