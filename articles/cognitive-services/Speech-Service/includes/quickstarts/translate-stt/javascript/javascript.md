---
author: IEvangelist
ms.service: cognitive-services
ms.topic: include
ms.date: 04/03/2020
ms.author: trbye
ms.custom: devx-track-javascript
ms.openlocfilehash: a0eb507237044a1a28b75532ff1f782c1ad88dbb
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/29/2020
ms.locfileid: "87407714"
---
## <a name="prerequisites"></a>Prérequis

Avant de commencer :

> [!div class="checklist"]
> * <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesSpeechServices" target="_blank">Créer une ressource Azure Speech<span class="docon docon-navigate-external x-hidden-focus"></span></a>
> * [Configurer votre environnement de développement et créer un projet vide](../../../../quickstarts/setup-platform.md)

## <a name="create-a-new-website-folder"></a>Créer un dossier de site web

Créez un dossier vide. Si vous souhaitez héberger l’exemple sur un serveur web, vérifiez que le serveur web peut accéder au dossier.

## <a name="unpack-the-speech-sdk-for-javascript-into-that-folder"></a>Décompresser le SDK Speech pour JavaScript dans ce dossier

Téléchargez le SDK Speech en tant que [package .zip](https://aka.ms/csspeech/jsbrowserpackage), puis décompressez-le dans le dossier que vous venez de créer. Cela entraîne la décompression de cinq fichiers, notamment `microsoft.cognitiveservices.speech.sdk.bundle.js` et `microsoft.cognitiveservices.speech.sdk.bundle.js.map`.
Le second est facultatif et est utile pour déboguer le code du SDK.

## <a name="create-an-indexhtml-page"></a>Créer une page index.html

Créez un fichier nommé `index.html` dans le dossier, puis ouvrez-le dans un éditeur de texte.

1. Créez le squelette HTML suivant :

 [!code-html [SampleCode](~/samples-cognitive-services-speech-sdk/quickstart/javascript/browser/index-translate-stt.html)]

## <a name="create-the-token-source-optional"></a>Créer la source du jeton (facultatif)

Si vous souhaitez héberger la page web sur un serveur web, vous avez la possibilité de fournir une source de jeton pour votre application de démonstration.
De cette façon, votre clé d’abonnement ne quitte jamais le serveur et vous permet d’utiliser les fonctionnalités de reconnaissance vocale sans avoir à entrer de code d’autorisation.

Créez un nouveau fichier appelé `token.php`. Dans cet exemple, nous supposons que votre serveur web prend en charge le langage de script PHP avec curl activé. Entrez le code suivant :

```php
<?php
header('Access-Control-Allow-Origin: ' . $_SERVER['SERVER_NAME']);

// Replace with your own subscription key and service region (e.g., "westus").
$subscriptionKey = 'YourSubscriptionKey';
$region = 'YourServiceRegion';

$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, 'https://' . $region . '.api.cognitive.microsoft.com/sts/v1.0/issueToken');
curl_setopt($ch, CURLOPT_POST, 1);
curl_setopt($ch, CURLOPT_POSTFIELDS, '{}');
curl_setopt($ch, CURLOPT_HTTPHEADER, array('Content-Type: application/json', 'Ocp-Apim-Subscription-Key: ' . $subscriptionKey));
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
echo curl_exec($ch);
?>
```

> [!NOTE]
> Les jetons d’autorisation ont une durée de vie limitée.
> Cet exemple simplifié ne montre pas comment actualiser automatiquement les jetons d’autorisation. En tant qu’utilisateur, vous pouvez recharger manuellement la page ou appuyer sur F5 pour l’actualiser.

## <a name="build-and-run-the-sample-locally"></a>Générer et exécuter l’exemple localement

Pour lancer l’application, double-cliquez sur le fichier index.html ou ouvrez-le dans votre navigateur web de votre choix. Celui-ci présente une interface graphique simple, dans laquelle vous pouvez entrer votre clé d’abonnement et votre [région](../../../../regions.md), puis déclencher la transcription en texte de la voix entrée.

## <a name="build-and-run-the-sample-via-a-web-server"></a>Générer et exécuter l’exemple via un serveur web

Pour lancer l’application, ouvrez le navigateur web de votre choix, accédez à l’URL publique où est hébergé le dossier, entrez votre [région](../../../../regions.md), puis déclenchez la transcription en texte de la voix entrée. Vous obtiendrez un jeton de votre source de jeton, si cette option est configurée.

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [footer](./footer.md)]