---
title: Publier du contenu Azure Media Services à l’aide de REST
description: Apprenez à créer un localisateur utilisé pour générer une URL de diffusion en continu. Le code utilise l’API REST.
author: Juliako
manager: femila
editor: ''
services: media-services
documentationcenter: ''
ms.assetid: ff332c30-30c6-4ed1-99d0-5fffd25d4f23
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/20/2019
ms.author: juliako
ms.openlocfilehash: 438a816ca28b9414557e64591d9ece2dbb173370
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89256869"
---
# <a name="publish-azure-media-services-content-using-rest"></a>Publier du contenu Azure Media Services à l’aide de REST

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!div class="op_single_selector"]
> * [.NET](media-services-deliver-streaming-content.md)
> * [REST](media-services-rest-deliver-streaming-content.md)
> * [Portail](media-services-portal-publish.md)
> 
> 

Vous pouvez diffuser un MP4 à débit adaptatif défini par la création d’un localisateur de diffusion en continu à la demande et la création d’une URL de diffusion en continu. L’article [Encoder un actif multimédia à l’aide de Media Encoder Standard dans le portail Azure](media-services-rest-encode-asset.md) indique comment encoder un ensemble de fichiers MP4 à débit adaptatif. Si votre contenu est chiffré, configurez la stratégie de livraison d’éléments multimédias (comme décrit dans [cet](media-services-rest-configure-asset-delivery-policy.md) article) avant de créer un localisateur. 

Vous pouvez également utiliser un localisateur de diffusion en continu à la demande pour créer des URL qui pointent vers les fichiers MP4 pouvant être téléchargés progressivement.  

Cet article montre comment créer un localisateur de diffusion en continu à la demande pour publier votre élément multimédia et créer des URL de diffusion en continu Smooth, MPEG DASH et HLS. Il explique également comment créer des URL de téléchargement progressif.

La section [suivante](#types) indique les types d’énumération dont les valeurs sont utilisées dans les appels REST.   

> [!NOTE]
> Lors de l’accès aux entités dans Media Services, vous devez définir les valeurs et les champs d’en-tête spécifiques dans vos requêtes HTTP. Pour plus d'informations, consultez [Installation pour le développement REST API de Media Services](media-services-rest-how-to-use.md).
> 

## <a name="connect-to-media-services"></a>Connexion à Media Services

Pour savoir comment vous connecter à l’API AMS, consultez [Accéder à l’API Azure Media Services avec l’authentification Azure AD](media-services-use-aad-auth-to-access-ams-api.md). 

>[!NOTE]
>Une fois connecté à https://media.windows.net, vous recevez une réponse de redirection 301 spécifiant un autre URI Media Services. Vous devez faire d’autres appels au nouvel URI.

## <a name="create-an-ondemand-streaming-locator"></a>Création d’un localisateur de diffusion en continu à la demande
Pour créer le localisateur de diffusion en continu à la demande et obtenir les URL, vous devez effectuer les opérations suivantes :

1. Si le contenu est chiffré, définissez une stratégie d'accès.
2. Création d’un localisateur de diffusion en continu à la demande.
3. Si vous envisagez de diffuser en continu, obtenez le fichier manifeste de diffusion en continu (.ism) dans la ressource. 
   
   Si vous souhaitez télécharger progressivement, obtenez les noms des fichiers MP4 dans la ressource. 
4. Création d’URL vers le fichier manifeste ou les fichiers MP4. 
5. Vous ne pouvez pas créer de localisateur de diffusion en continu en utilisant une stratégie d’accès qui inclut des autorisations d’écriture ou de suppression.

### <a name="create-an-access-policy"></a>Définition d’une stratégie d’accès.

>[!NOTE]
>Un nombre limite de 1 000 000 a été défini pour les différentes stratégies AMS (par exemple, pour la stratégie de localisateur ou pour ContentKeyAuthorizationPolicy). Utilisez le même ID de stratégie si vous utilisez toujours les mêmes jours/autorisations d’accès, par exemple, les stratégies pour les localisateurs destinés à demeurer en place pendant une longue période (stratégies sans chargement). Pour plus d’informations, consultez [cet](media-services-dotnet-manage-entities.md#limit-access-policies) article.

Demande :

```console
POST https://media.windows.net/api/AccessPolicies HTTP/1.1
Content-Type: application/json
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN> 
x-ms-version: 2.19
x-ms-client-request-id: 6bcfd511-a561-448d-a022-a319a89ecffa
Host: media.windows.net
Content-Length: 68

{"Name":"access policy","DurationInMinutes":43200.0,"Permissions":1}
```

Réponse :

```console
HTTP/1.1 201 Created
Cache-Control: no-cache
Content-Length: 311
Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
Location: https:/media.windows.net/api/AccessPolicies('nb%3Apid%3AUUID%3A69c80d98-7830-407f-a9af-e25f4b0d3e5f')
Server: Microsoft-IIS/8.5
request-id: a877528a-bdb4-4414-9862-273f8e64f882
x-ms-request-id: a877528a-bdb4-4414-9862-273f8e64f882
x-ms-client-request-id: 6bcfd511-a561-448d-a022-a319a89ecffa
X-Content-Type-Options: nosniff
DataServiceVersion: 3.0;
X-Powered-By: ASP.NET
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Wed, 18 Feb 2015 06:52:09 GMT

{"odata.metadata":"https://media.windows.net/api/$metadata#AccessPolicies/@Element","Id":"nb:pid:UUID:69c80d98-7830-407f-a9af-e25f4b0d3e5f","Created":"2015-02-18T06:52:09.8862191Z","LastModified":"2015-02-18T06:52:09.8862191Z","Name":"access policy","DurationInMinutes":43200.0,"Permissions":1}
```

### <a name="create-an-ondemand-streaming-locator"></a>Création d’un localisateur de diffusion en continu à la demande
Créez le localisateur pour la ressource et la stratégie de ressource spécifiées.

Demande :

```console
POST https://media.windows.net/api/Locators HTTP/1.1
Content-Type: application/json
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN> 
x-ms-version: 2.19
x-ms-client-request-id: ac159492-9a0c-40c3-aacc-551b1b4c5f62
Host: media.windows.net
Content-Length: 181

{"AccessPolicyId":"nb:pid:UUID:1480030d-c481-430a-9687-535c6a5cb272","AssetId":"nb:cid:UUID:cc1e445d-1500-80bd-538e-f1e4b71b465e","StartTime":"2015-02-18T06:34:47.267872Z","Type":2}
```
Réponse :

```console
HTTP/1.1 201 Created
Cache-Control: no-cache
Content-Length: 637
Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
Location: https://media.windows.net/api/Locators('nb%3Alid%3AUUID%3Abe245661-2bbd-4fc6-b14f-9cf9a1492e5e')
Server: Microsoft-IIS/8.5
request-id: 5bd5864a-0afd-44c0-a67a-4044a2c9043b
x-ms-request-id: 5bd5864a-0afd-44c0-a67a-4044a2c9043b
x-ms-client-request-id: ac159492-9a0c-40c3-aacc-551b1b4c5f62
X-Content-Type-Options: nosniff
DataServiceVersion: 3.0;
X-Powered-By: ASP.NET
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Wed, 18 Feb 2015 06:58:37 GMT

{"odata.metadata":"https://media.windows.net/api/$metadata#Locators/@Element","Id":"nb:lid:UUID:be245661-2bbd-4fc6-b14f-9cf9a1492e5e","ExpirationDateTime":"2015-03-20T06:34:47.267872+00:00","Type":2,"Path":"https://amstest1.streaming.mediaservices.windows.net/be245661-2bbd-4fc6-b14f-9cf9a1492e5e/","BaseUri":"https://amstest1.streaming.mediaservices.windows.net","ContentAccessComponent":"be245661-2bbd-4fc6-b14f-9cf9a1492e5e","AccessPolicyId":"nb:pid:UUID:1480030d-c481-430a-9687-535c6a5cb272","AssetId":"nb:cid:UUID:cc1e445d-1500-80bd-538e-f1e4b71b465e","StartTime":"2015-02-18T06:34:47.267872+00:00","Name":null}
```

### <a name="build-streaming-urls"></a>Création d’URL de diffusion
Utilisez la valeur **Chemin d’accès** renvoyée après la création du localisateur pour générer les URL Smooth, HLS et MPEG DASH. 

Smooth Streaming : **Chemin d’accès** + nom du fichier manifeste + "/manifest"

exemple :

`https://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny.ism/manifest`

HLS : **Chemin d’accès** + nom du fichier manifeste "/manifest(format=m3u8-aapl)"

exemple :

`https://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny.ism/manifest(format=m3u8-aapl)`


DASH : **Chemin d’accès** + nom du fichier manifeste + "/manifest(format=mpd-time-csf)"

exemple :

`https://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny.ism/manifest(format=mpd-time-csf)`


### <a name="build-progressive-download-urls"></a>Génération d’URL de téléchargement progressif
Utilisez la valeur **Chemin d’accès** renvoyée après la création du localisateur pour générer l’URL de téléchargement progressif.   

URL : **Chemin d’accès** + nom du fichier de ressource mp4

exemple :

`https://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny_H264_650kbps_AAC_und_ch2_96kbps.mp4`

## <a name="enum-types"></a><a id="types"></a>Types Enum

```console
[Flags]
public enum AccessPermissions
{
    None = 0,
    Read = 1,
    Write = 2,
    Delete = 4,
    List = 8,
}

public enum LocatorType
{
    None = 0,
    Sas = 1,
    OnDemandOrigin = 2,
}
```

## <a name="media-services-learning-paths"></a>Parcours d’apprentissage de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="see-also"></a>Voir aussi
[Vue d’ensemble de l’API REST Media Services Operations](media-services-rest-how-to-use.md)

[Configurer une stratégie de distribution d’éléments multimédias](media-services-rest-configure-asset-delivery-policy.md)

