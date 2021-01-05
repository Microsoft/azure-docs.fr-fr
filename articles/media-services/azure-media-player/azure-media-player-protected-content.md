---
title: Contenu protégé du Lecteur multimédia Azure
description: Le Lecteur multimédia Azure prend actuellement en charge le contenu chiffré d’enveloppe AES 128 bits et le contenu chiffré commun.
author: IngridAtMicrosoft
ms.author: inhenkel
ms.service: media-services
ms.topic: overview
ms.date: 04/20/2020
ms.custom: devx-track-js
ms.openlocfilehash: 83f144c06c23f3ab5507e3561be4a12350e20a42
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "91329684"
---
# <a name="protected-content"></a>Contenu protégé #

Le Lecteur multimédia Azure prend actuellement en charge le contenu chiffré d’enveloppe AES 128 bits et le contenu chiffré commun (via PlayReady et Widevine) ou le contenu chiffré par le biais de FairPlay. Afin de lire correctement le contenu protégé, vous devez indiquer l’élément `protectionInfo` au Lecteur multimédia Azure. Cette information existe au niveau de chaque source et peut être ajoutée directement dans la balise `<source>` via `data-setup`.  Vous pouvez également ajouter `protectionInfo` directement sous la forme d’un paramètre si vous définissez la source de manière dynamique.

`protectionInfo` accepte un objet JSON et comprend les éléments suivants :

- `type` : `AES` ou `PlayReady` ou `Widevine` ou `FairPlay`
- `certificateUrl` : il doit s’agir d’un lien direct vers votre certificat FairPlay hébergé

- `authenticationToken` : il s’agit d’un champ d’option pour ajouter un jeton d’authentification non codé

> [!IMPORTANT]
> L’objet **certificateUrl** n’est nécessaire que pour FairPlay DRM.***
>[!NOTE]
> L’objet techOrder par défaut a été modifié pour prendre en compte la nouvelle technologie `html5FairPlayHLS`, plus particulièrement pour lire le contenu FairPlay en mode natif sur les navigateurs qui le prennent en charge (Safari sur OSX 8+). Si vous avez un contenu FairPlay à lire **ET** que vous avez remplacé le techOrder par défaut par un autre qui est personnalisé dans votre application, vous devez ajouter cette nouvelle technologie à votre objet techOrder. Nous vous recommandons de l’inclure avant silverlightSS, pour que votre contenu ne soit pas lu via PlayReady.

## <a name="code-sample"></a>Exemple de code ##

```html
Ex:

    <video id="vid1" class="azuremediaplayer amp-default-skin">
        <source
            src="//example/path/to/myVideo.ism/manifest"
            type="application/vnd.ms-sstr+xml"
            data-setup='{"protectionInfo": [{"type": "AES", "authenticationToken": "Bearer=urn%3amicrosoft%3aazure%3amediaservices%3acontentkeyidentifier=8130520b-c116-45a9-824e-4a0082f3cb3c&Audience=urn%3atest&ExpiresOn=1450207516&Issuer=http%3a%2f%2ftestacs.com%2f&HMACSHA256=eV7HDgZ9msp9H9bnEPGN91sBdU7XsZ9OyB6VgFhKBAU%3d"}]}'
        />
    </video>
or

```javascript
    var myPlayer = amp("vid1", /* Options */);
    myPlayer.src([{
        src: "//example/path/to/myVideo.ism/manifest",
        type: "application/vnd.ms-sstr+xml",
        protectionInfo: [{
            type: "PlayReady",
            authenticationToken: "Bearer=urn%3amicrosoft%3aazure%3amediaservices%3acontentkeyidentifier=d5646e95-63ee-4fbe-ba4e-295c8d9502e0&Audience=urn%3atest&ExpiresOn=1450222961&Issuer=http%3a%2f%2ftestacs.com%2f&HMACSHA256=4Jop3kNJdzVI8L5IZLgFtPdImyE%2fHTRil0x%2bEikSdPs%3d"
        }] }, ]
    );
```

ou, avec plusieurs DRM

```javascript
    var myPlayer = amp("vid1", /* Options */);
    myPlayer.src([{
        src: "//example/path/to/myVideo.ism/manifest",
        type: "application/vnd.ms-sstr+xml",
        protectionInfo: [{
                type: "PlayReady",
                authenticationToken: "Bearer=urn%3amicrosoft%3aazure%3amediaservices%3acontentkeyidentifier=d5646e95-63ee-4fbe-ba4e-295c8d9502e0&Audience=urn%3atest&ExpiresOn=1450222961&Issuer=http%3a%2f%2ftestacs.com%2f&HMACSHA256=4Jop3kNJdzVI8L5IZLgFtPdImyE%2fHTRil0x%2bEikSdPs%3d"
            },
            {
                type: "Widevine",
                authenticationToken: "Bearer=urn%3amicrosoft%3aazure%3amediaservices%3acontentkeyidentifier=d5646e95-63ee-4fbe-ba4e-295c8d9502e0&Audience=urn%3atest&ExpiresOn=1450222961&Issuer=http%3a%2f%2ftestacs.com%2f&HMACSHA256=4Jop3kNJdzVI8L5IZLgFtPdImyE%2fHTRil0x%2bEikSdPs%3d"
            },
            {
                   type: "FairPlay",
                  certificateUrl: "//example/path/to/myFairplay.der",
                   authenticationToken: "Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1cm46bWljcm9zb2Z0OmF6dXJlOm1lZGlhc2VydmljZXM6Y29udGVudGtleWlkZW50aWZpZXIiOiIyMTI0M2Q2OC00Yjc4LTRlNzUtYTU5MS1jZWMzMDI0NDNhYWMiLCJpc3MiOiJodHRwOi8vY29udG9zbyIsImF1ZCI6InVybjp0ZXN0IiwiZXhwIjoxNDc0NTkyNDYzLCJuYmYiOjE0NzQ1ODg1NjN9.mE7UxgNhkieMMqtM_IiYQj-FK1KKIzB6lAptw4Mi67A"
        }] } ]
    );
```

> [!NOTE]
> Tous les navigateurs/toutes les plateformes ne sont pas capables de lire du contenu protégé. Pour plus d’informations sur les éléments pris en charge, consultez la section [Technologie de lecture](azure-media-player-playback-technology.md).
> [!IMPORTANT]
> Le jeton passé dans le Lecteur est destiné au contenu sécurisé, et utilisé uniquement pour les utilisateurs authentifiés. L’application est censée utiliser le protocole SSL ou une autre forme de mesure de sécurité. Par ailleurs, l’utilisateur final est supposé être approuvé pour que le jeton ne soit pas utilisé de façon malveillante. Si ce n’est pas le cas, veuillez impliquer vos experts en sécurité.

## <a name="next-steps"></a>Étapes suivantes ##

- [Démarrage rapide du Lecteur multimédia Azure](azure-media-player-quickstart.md)