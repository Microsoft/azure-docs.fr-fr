---
title: Résoudre les problèmes relatifs au SDK Speech - Service Speech
titleSuffix: Azure Cognitive Services
description: Cet article fournit des informations pour vous aider à résoudre des problèmes que vous pourriez rencontrer lors de l’utilisation du Kit de développement logiciel (SDK) Speech.
services: cognitive-services
author: jhakulin
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 07/23/2019
ms.author: jhakulin
ms.openlocfilehash: 421b9adf4ae5d2c641484e646bea096716d46cca
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "74815415"
---
# <a name="troubleshoot-the-speech-sdk"></a>Résoudre les problèmes avec le Kit de développement logiciel (SDK) Speech

Cet article fournit des informations pour vous aider à résoudre des problèmes que vous pourriez rencontrer lors de l’utilisation du Kit de développement logiciel (SDK) Speech.

## <a name="error-websocket-upgrade-failed-with-an-authentication-error-403"></a>Erreur : La mise à niveau de WebSocket a échoué avec une erreur d’authentification (403)

Le point de terminaison pour votre service ou région est peut-être incorrect. Vérifiez l’URI pour vous assurer qu’il est correct.

Il y a peut-être également un problème avec votre clé d’abonnement ou jeton d’autorisation. Pour plus d'informations, consultez la section suivante.

## <a name="error-http-403-forbidden-or-http-401-unauthorized"></a>Erreur : HTTP 403 Refusé ou HTTP 401 Non autorisé

Cette erreur est souvent due à des problèmes d’authentification. Les demandes de connexion sans en-tête `Ocp-Apim-Subscription-Key` ou `Authorization` valide sont rejetées. Elles renvoient un code d’état 403 ou 401.

* Si vous utilisez une clé d’abonnement pour l’authentification, vous verrez peut-être l’erreur pour les raisons suivantes :

    - La clé d’abonnement est manquante ou non valide.
    - Vous avez dépassé le quota d’utilisation auquel votre abonnement vous donne droit.

* Si vous utilisez un jeton d’autorisation pour l’authentification, vous verrez peut-être l’erreur pour les raisons suivantes :

    - Le jeton d’autorisation n’est pas valide.
    - Le jeton d’autorisation a expiré.

### <a name="validate-your-subscription-key"></a>Valider votre clé d’abonnement

Vous pouvez vérifier que vous disposez d’une clé d’abonnement valide en exécutant l’une des commandes suivantes.

> [!NOTE]
> Remplacez `YOUR_SUBSCRIPTION_KEY` et `YOUR_REGION` par votre propre clé d’abonnement et la région associée.

* PowerShell

    ```Powershell
    $FetchTokenHeader = @{
      'Content-type'='application/x-www-form-urlencoded'
      'Content-Length'= '0'
      'Ocp-Apim-Subscription-Key' = 'YOUR_SUBSCRIPTION_KEY'
    }
    $OAuthToken = Invoke-RestMethod -Method POST -Uri https://YOUR_REGION.api.cognitive.microsoft.com/sts/v1.0/issueToken -Headers $FetchTokenHeader
    $OAuthToken
    ```

* cURL

    ```
    curl -v -X POST "https://YOUR_REGION.api.cognitive.microsoft.com/sts/v1.0/issueToken" -H "Ocp-Apim-Subscription-Key: YOUR_SUBSCRIPTION_KEY" -H "Content-type: application/x-www-form-urlencoded" -H "Content-Length: 0"
    ```

Si vous avez entré une clé d’abonnement valide, la commande renvoie un jeton d’autorisation, voire une erreur dans le cas contraire.

### <a name="validate-an-authorization-token"></a>Valider un jeton d’autorisation

Si vous utilisez un jeton d’autorisation pour l’authentification, exécutez l’une des commandes suivantes pour vérifier que le jeton d’autorisation est toujours valide. Les jetons sont valides pendant 10 minutes.

> [!NOTE]
> Remplacez `YOUR_AUDIO_FILE` par le chemin de votre fichier audio préenregistré. Remplacez `YOUR_ACCESS_TOKEN` par le jeton d’autorisation retourné à l’étape précédente. Remplacez `YOUR_REGION` par la région correcte.

* PowerShell

    ```Powershell
    $SpeechServiceURI =
    'https://YOUR_REGION.stt.speech.microsoft.com/speech/recognition/interactive/cognitiveservices/v1?language=en-US'

    # $OAuthToken is the authorization token returned by the token service.
    $RecoRequestHeader = @{
      'Authorization' = 'Bearer '+ $OAuthToken
      'Transfer-Encoding' = 'chunked'
      'Content-type' = 'audio/wav; codec=audio/pcm; samplerate=16000'
    }

    # Read audio into byte array.
    $audioBytes = [System.IO.File]::ReadAllBytes("YOUR_AUDIO_FILE")

    $RecoResponse = Invoke-RestMethod -Method POST -Uri $SpeechServiceURI -Headers $RecoRequestHeader -Body $audioBytes

    # Show the result.
    $RecoResponse
    ```

* cURL

    ```
    curl -v -X POST "https://YOUR_REGION.stt.speech.microsoft.com/speech/recognition/interactive/cognitiveservices/v1?language=en-US" -H "Authorization: Bearer YOUR_ACCESS_TOKEN" -H "Transfer-Encoding: chunked" -H "Content-type: audio/wav; codec=audio/pcm; samplerate=16000" --data-binary @YOUR_AUDIO_FILE
    ```

Si vous avez entré une jeton d'autorisation valide, la commande renvoie la transcription de votre fichier audio, voire une erreur dans le cas contraire.

---

## <a name="error-http-400-bad-request"></a>Erreur : HTTP 400 Requête incorrecte

Cette erreur est généralement due au fait que le corps de la requête contient des données audio non valides. Seul le format WAV est pris en charge. De même, vérifiez les en-têtes de la requête pour vous assurer que vous spécifiez des valeurs appropriées pour `Content-Type` et `Content-Length`.

## <a name="error-http-408-request-timeout"></a>Erreur : HTTP 408 Dépassement de délai de demande

L’erreur est très probablement due au fait qu’aucune donnée audio n’est envoyée au service. Cette erreur peut également résulter de problèmes de réseau.

## <a name="recognitionstatus-in-the-response-is-initialsilencetimeout"></a>« RecognitionStatus » dans la réponse est « InitialSilenceTimeout »

Ce problème est généralement dû à des données audio. Ce message d’erreur peut s’afficher pour les raisons suivantes :

* Il existe un long stretch de silence au début de l’enregistrement audio. Dans ce cas, le service arrête la reconnaissance après quelques secondes et renvoie `InitialSilenceTimeout`.

* Le contenu audio utilise un format de codec non pris en charge, ce qui fait que les données audio sont traitées comme s’il s’agissait d’un silence.

## <a name="next-steps"></a>Étapes suivantes

* [Examiner les notes de publication](releasenotes.md)
