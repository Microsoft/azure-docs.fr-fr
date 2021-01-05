---
title: Codes d’erreur du Lecteur multimédia Azure
description: Document de référence sur les codes d'erreur du Lecteur multimédia Azure.
author: IngridAtMicrosoft
ms.author: inhenkel
ms.service: media-services
ms.topic: error-reference
ms.date: 04/20/2020
ms.openlocfilehash: 13d804ec39c3d7753d4ee04962a88d4451fb04d9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "81725902"
---
# <a name="error-codes"></a>Codes d’erreur #

Si la lecture ne peut pas démarrer ou s’arrête, un événement d’erreur est déclenché et la fonction `error()` renvoie un code et un message facultatif pour aider le développeur d'application à obtenir plus de détails. `error().message` ne correspond pas au message affiché pour l’utilisateur.  Le message affiché pour l'utilisateur est basé sur `error().code` bits 27-20, voir le tableau ci-dessous.

```javascript

    var myPlayer = amp('vid1');
    myPlayer.addEventListener('error', function() {
        var errorDetails = myPlayer.error();
        var code = errorDetails.code;
        var message = errorDetails.message;
    }
```

## <a name="error-codes-bits-31-28-4-bits"></a>Codes d’erreur, bits [31-28] (4 bits) ##

Décrire la zone de l’erreur.

- 0 - Inconnue
- 1 - AMP
- 2 - AzureHtml5JS
- 3 - FlashSS
- 4 - SilverlightSS
- 5 - Html5
- 6 - Html5FairPlayHLS

## <a name="error-codes-bits-27-0-28-bits"></a>Codes d’erreur, bits [27-0] (28 bits) ##

Décrire les détails de l’erreur, les bits 27-20 fournissent un niveau élevé, les bits 19-0 fournissent plus de détails s’ils sont disponibles.


| amp.errorCode.[nom] | Codes, bits [27-0] (28 bits) | Description |
|---|---:|---|
| **Plage d’erreurs MEDIA_ERR_ABORTED (0x0100000-0x01FFFFF)** | | |
| abortedErrUnknown | 0x0100000 | Erreur d’abandon générique |
| abortedErrNotImplemented | 0x0100001 | Erreur d’abandon, non implémentée |
| abortedErrHttpMixedContentBlocked | 0x0100002 | Erreur d’abandon, contenu mixte bloqué - se produit généralement lors du chargement d’un flux `http://` à partir d’une page `https://` |
| **Valeur de début des erreurs MEDIA_ERR_NETWORK (0x0200000-0x02FFFFF)** | | |
| networkErrUnknown | 0x0200000 | Erreur réseau générique |
| networkErrHttpBadUrlFormat | 0x0200190 | Réponse d’erreur HTTP 400 |
| networkErrHttpUserAuthRequired | 0x0200191 | Réponse d’erreur HTTP 401 |
| networkErrHttpUserForbidden | 0x0200193 | Réponse d’erreur HTTP 403 |
| networkErrHttpUrlNotFound | 0x0200194 | Réponse d’erreur HTTP 404 |
| networkErrHttpNotAllowed | 0x0200195 | Réponse d’erreur HTTP 405 |
| networkErrHttpGone | 0x020019A | Réponse d’erreur HTTP 410 |
| networkErrHttpPreconditionFailed | 0x020019C | Réponse d’erreur HTTP 412 |
| networkErrHttpInternalServerFailure | 0x02001F4 | Réponse d’erreur HTTP 500
| networkErrHttpBadGateway | 0x02001F6 | Réponse d’erreur HTTP 502 |
| networkErrHttpServiceUnavailable | 0x02001F7 | Réponse d’erreur HTTP 503 |
| networkErrHttpGatewayTimeout | 0x02001F8 | Réponse d’erreur HTTP 504 |
| networkErrTimeout | 0x0200258 | Erreur liée à une expiration du délai d'attente réseau
| networkErrErr | 0x0200259 | Réponse d’erreur de connexion réseau |
| **Erreurs MEDIA_ERR_DECODE (0x0300000-0x03FFFFF)** | | |
| decodeErrUnknown | 0x0300000 | Erreur de décodage générique |
| **Erreurs MEDIA_ERR_SRC_NOT_SUPPORTED (0x0400000-0x04FFFFF)** | | |
| srcErrUnknown | 0x0400000 | Erreur de source générique non prise en charge |
| srcErrParsePresentation | 0x0400001 | Erreur d'analyse de la présentation |
| srcErrParseSegment | 0x0400002 | Erreur d'analyse du segment |
| srcErrUnsupportedPresentation | 0x0400003 | Présentation non prise en charge |
| srcErrInvalidSegment | 0x0400004 | Segment non valide |
| **Valeur de début des erreurs MEDIA_ERR_ENCRYPTED (0x0500000-0x05FFFFF)** | | |
| encryptErrUnknown | 0x0500000 | Erreur chiffrée générique | 
| encryptErrDecrypterNotFound | 0x0500001 | Décodeur introuvable |
| encryptErrDecrypterInit | 0x0500002 | Erreur d'initialisation du décodeur |
| encryptErrDecrypterNotSupported | 0x0500003 | Décodeur non pris en charge |
| encryptErrKeyAcquire | 0x0500004 | Échec d’acquisition de la clé |
| encryptErrDecryption | 0x0500005 | Échec de déchiffrement du segment |
| encryptErrLicenseAcquire | 0x0500006 | Échec d'acquisition de la licence |
| **Valeur de début des erreurs SRC_PLAYER_MISMATCH (0x0600000-0x06FFFFF)** | | |
| srcPlayerMismatchUnknown | 0x0600000 | Générique, impossible pour le lecteur de lire la source |
| srcPlayerMismatchFlashNotInstalled | 0x0600001 |Plug-in Flash non installé, si la source peut lire le contenu. *OU* Flash 30 installé et lisant le contenu AES.  Si tel est le cas, essayez un autre navigateur. Flash 30 à partir du 7 juin. Pour plus de détails, voir [Problèmes connus](azure-media-player-known-issues.md). Remarque : Si 0x00600003, Flash et Silverlight non installés, s’ils sont spécifiés dans le techOrder.|
| srcPlayerMismatchSilverlightNotInstalled | 0x0600002 | Plug-in Silverlight non installé, si la source peut lire le contenu. Remarque : Si 0x00600003, Flash et Silverlight non installés, s’ils sont spécifiés dans le techOrder. |
| | 0x00600003 | Flash et Silverlight non installés, s’ils sont spécifiés dans le techOrder. |
| **Erreurs inconnues (0x0FF00000)** | | |
| errUnknown | 0xFF00000 | Erreurs inconnues |

## <a name="user-error-messages"></a>Messages d'erreur utilisateur ##

Le message utilisateur affiché est basé sur les bits 27-20 du code d’erreur.

- MEDIA_ERR_ABORTED (1) - « Vous avez abandonné la lecture de la vidéo ».
- MEDIA_ERR_NETWORK (2) - « Une erreur réseau a entraîné l'échec du téléchargement de la vidéo ».
- MEDIA_ERR_DECODE (3) - « La lecture de la vidéo a été abandonnée en raison d'un problème d'endommagement ou parce que la vidéo utilise des fonctionnalités que votre navigateur ne prend pas en charge ».
- MEDIA_ERR_SRC_NOT_SUPPORTED (4) - « La vidéo n'a pas pu être chargée parce que le serveur ou le réseau ont échoué ou parce que le format n'est pas pris en charge ».
- MEDIA_ERR_ENCRYPTED (5) - « La vidéo est chiffrée et nous n’avons pas les clés pour la déchiffrer ».
- SRC_PLAYER_MISMATCH (6) - « Aucune source compatible n’a été trouvée pour cette vidéo ».
- MEDIA_ERR_UNKNOWN (0xFF) - « Une erreur inconnue s’est produite ».

## <a name="examples"></a>Exemples ##

### <a name="0x10600001"></a>0x10600001 ##

« Aucune source compatible n’a été trouvée pour cette vidéo ». s’affiche pour l’utilisateur final.

Il n’existe pas de lecteur Tech capable de lire les sources demandées, mais si le plug-in Flash est installé, il est probable qu’une source soit lue.  

### <a name="0x20200194"></a>0x20200194 ###

« Une erreur réseau a entraîné l'échec du téléchargement de la vidéo ». s’affiche pour l’utilisateur final.

Échec de lecture de AzureHtml5JS à partir d’une réponse HTTP 404.

### <a name="categorizing-errors"></a>Classement des erreurs ###

```javascript
    if(myPlayer.error().code & amp.errorCode.abortedErrStart) {
        // MEDIA_ERR_ABORTED errors
    }
    else if(myPlayer.error().code & amp.errorCode.networkErrStart) {
        // MEDIA_ERR_NETWORK errors
    }
    else if(myPlayer.error().code & amp.errorCode.decodeErrStart) {
        // MEDIA_ERR_DECODE errors
    }
    else if(myPlayer.error().code & amp.errorCode.srcErrStart) {
        // MEDIA_ERR_SRC_NOT_SUPPORTED errors
    }
    else if(myPlayer.error().code & amp.errorCode.encryptErrStart) {
        // MEDIA_ERR_ENCRYPTED errors
    }
    else if(myPlayer.error().code & amp.errorCode.srcPlayerMismatchStart) {
        // SRC_PLAYER_MISMATCH errors
    }
    else {
        // unknown errors
    }
```

### <a name="catching-a-specific-error"></a>Interception d’une erreur spécifique ###

Le code suivant intercepte uniquement les erreurs 404 :

```javascript
    if(myPlayer.error().code & amp.errorCode.networkErrHttpUrlNotFound) {
        // all http 404 errors
    }
```

## <a name="next-steps"></a>Étapes suivantes ##

- [Démarrage rapide du Lecteur multimédia Azure](azure-media-player-quickstart.md)