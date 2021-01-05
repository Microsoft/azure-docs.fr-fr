---
title: Configuration des stratégies de remise de ressources à l’aide de l’API REST Media Services | Microsoft Docs
description: Cette rubrique montre comment configurer différentes stratégies de remise de ressources à l’aide de l’API REST Media Services.
services: media-services
documentationcenter: ''
author: Juliako
manager: cfowler
editor: ''
ms.assetid: 5cb9d32a-e68b-4585-aa82-58dded0691d0
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/20/2019
ms.author: juliako
ms.custom: devx-track-csharp
ms.openlocfilehash: 49fc67bcd45d0c364f47276a9b47676a65f6d837
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89268670"
---
# <a name="configuring-asset-delivery-policies"></a>Configuration des stratégies de remise de ressources

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

[!INCLUDE [media-services-selector-asset-delivery-policy](../../../includes/media-services-selector-asset-delivery-policy.md)]

Si vous envisagez la remise de ressources chiffrées dynamiquement, l'une des étapes du workflow de remise de contenu Media Services consiste à configurer les stratégies de remise pour les ressources. La stratégie de remise de ressources indique à Media Services comment vous souhaitez distribuer vos ressources : dans quel protocole de diffusion en continu votre ressource doit être empaquetée dynamiquement (par exemple, MPEG DASH, HLS, Smooth Streaming ou tous), si vous souhaitez chiffrer dynamiquement votre ressource ou non et comment (chiffrement commun ou d’enveloppe).

Cette rubrique explique pourquoi et comment créer et configurer des stratégies de livraison d’éléments multimédias.

> [!NOTE]
> Une fois votre compte AMS créé, un point de terminaison de diffusion continue **par défaut** est ajouté à l’état **Arrêté**. Pour démarrer la diffusion en continu de votre contenu et tirer parti de l’empaquetage et du chiffrement dynamiques, le point de terminaison de streaming à partir duquel vous souhaitez diffuser du contenu doit se trouver à l’état **En cours d’exécution**. 
>
> De plus, pour pouvoir utiliser l’empaquetage et le chiffrement dynamiques, votre ressource doit contenir un ensemble de MP4 à débit adaptatif ou des fichiers de diffusion en continu lisse à débit adaptatif.

Vous pouvez appliquer des stratégies différentes à la même ressource. Par exemple, vous pouvez appliquer le chiffrement PlayReady pour la diffusion en continu lisse et AES en enveloppe pour MPEG DASH et TLS. Tous les protocoles qui ne sont pas définis dans une stratégie de remise (par exemple, en cas d’ajout d’une stratégie unique qui spécifie uniquement TLS comme protocole) seront bloqués de la diffusion en continu. Cela ne s’applique toutefois pas si vous n’avez défini aucune stratégie de remise de ressources. Tous les protocoles seront alors autorisés.

Si vous souhaitez remettre une ressource à chiffrement de stockage, vous devez configurer la stratégie de remise de la ressource. Avant de pouvoir diffuser votre ressource en continu, le serveur de diffusion supprime le chiffrement de stockage et transmet en continu votre contenu à l’aide de la stratégie de remise spécifiée. Par exemple, pour remettre votre ressource chiffrée avec la clé de chiffrement d'enveloppe AES (Advanced Encryption Standard), définissez le type de stratégie sur **DynamicEnvelopeEncryption**. Pour supprimer le chiffrement de stockage et diffuser la ressource en clair, définissez le type de stratégie sur **NoDynamicEncryption**. Vous trouverez des exemples qui montrent comment configurer ces types de stratégie ci-dessous.

Selon la configuration de la stratégie de remise de ressources, vous pourrez empaqueter dynamiquement, chiffrer dynamiquement et diffuser les protocoles de diffusion en continu suivants : Smooth Streaming, HLS et MPEG DASH.

La liste suivante présente les formats utilisés pour diffuser en continu Smooth, HLS et DASH.

Smooth Streaming :

{nom du point de terminaison de diffusion en continu-nom du compte media services}.streaming.mediaservices.windows.net/{ID_de_localisateur}/{nom_de_fichier}.ISM/Manifest

HLS :

{nom du point de terminaison de diffusion en continu-nom du compte media services}.streaming.mediaservices.windows.net/{ID_de_localisateur}/{nom_de_fichier}.ISM/Manifest(format=m3u8-aapl)

MPEG DASH

{nom du point de terminaison de diffusion en continu-nom du compte media services}.streaming.mediaservices.windows.net/{ID_de_localisateur}/{nom_de_fichier}.ISM/Manifest(format=mpd-time-csf)


Pour savoir comment publier une ressource et générer une URL de diffusion en continu, consultez [Générer une URL de diffusion en continu](media-services-deliver-streaming-content.md).

## <a name="considerations"></a>Considérations
* Vous ne pouvez pas supprimer une stratégie AssetDeliveryPolicy associée à un élément multimédia alors qu’un localisateur (de diffusion en continuer) OnDemand existe pour cet élément. Il est recommandé de retirer la stratégie de l’élément multimédia avant de la supprimer.
* Il est impossible de créer un localisateur de diffusion en continu sur un élément multimédia chiffré de stockage quand aucune stratégie de distribution d’éléments multimédias n’est définie.  Si l’élément multimédia n’est pas chiffré dans le stockage, le système vous permet de créer un localisateur et de diffuser en continu l’élément multimédia en clair sans stratégie de distribution d’éléments multimédias.
* Vous pouvez avoir plusieurs stratégies de distribution d’éléments multimédias associées à un même élément multimédia, mais vous ne pouvez spécifier qu’une seule façon de traiter un AssetDeliveryProtocol donné.  Cela signifie que si vous essayez de lier deux stratégies de distribution qui spécifient le protocole AssetDeliveryProtocol.SmoothStreaming, cela va générer une erreur, car le système ne sait pas laquelle appliquer lorsqu’un client émet une demande Smooth Streaming.
* Si vous avez un élément multimédia avec un localisateur de diffusion en continu existant, vous ne pouvez pas lier une nouvelle stratégie à l’élément multimédia. supprimer le lien d’une stratégie existante de l’élément multimédia ou mettre à jour une stratégie de distribution associée à l’élément multimédia.  Vous devez d’abord supprimer le localisateur de diffusion en continu, ajuster les stratégies, puis recréer le localisateur de diffusion en continu.  Vous pouvez utiliser le même ID de localisateur (locatorId) quand vous recréez le localisateur de diffusion en continu. Vous devez cependant vérifier que cela ne crée pas de problèmes pour les clients, car le contenu peut être mis en cache par l’origine ou un CDN en aval.

> [!NOTE]
> 
> Lors de l’accès aux entités dans Media Services, vous devez définir les valeurs et les champs d’en-tête spécifiques dans vos requêtes HTTP. Pour plus d'informations, consultez [Installation pour le développement REST API de Media Services](media-services-rest-how-to-use.md).

## <a name="connect-to-media-services"></a>Connexion à Media Services

Pour savoir comment vous connecter à l’API AMS, consultez [Accéder à l’API Azure Media Services avec l’authentification Azure AD](media-services-use-aad-auth-to-access-ams-api.md). 

## <a name="clear-asset-delivery-policy"></a>Stratégie de remise de ressources
### <a name="create-asset-delivery-policy"></a><a id="create_asset_delivery_policy"></a>Création d’une stratégie de remise d’éléments multimédias
La requête HTTP suivante permet de créer une stratégie de remise d’éléments multimédias qui précise de ne pas appliquer de chiffrement dynamique et de fournir le flux avec l’un des protocoles suivants : MPEG DASH, HLS et Smooth Streaming. 

Pour plus d'informations sur les valeurs que vous pouvez spécifier au moment de la création d'une AssetDeliveryPolicy, consultez la section [Types utilisés au moment de la définition d'AssetDeliveryPolicy](#types) .   

Demande :

```console
POST https://media.windows.net/api/AssetDeliveryPolicies HTTP/1.1
Content-Type: application/json
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN> 
x-ms-version: 2.19
x-ms-client-request-id: 4651882c-d7ad-4d5e-86ab-f07f47dcb41e
Host: media.windows.net

{"Name":"Clear Policy",
"AssetDeliveryProtocol":7,
"AssetDeliveryPolicyType":2,
"AssetDeliveryConfiguration":null}
```

Réponse :

```output
HTTP/1.1 201 Created
Cache-Control: no-cache
Content-Length: 363
Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
Location: https://media.windows.net/api/AssetDeliveryPolicies('nb%3Aadpid%3AUUID%3A92b0f6ba-3c9f-49b6-a5fa-2a8703b04ecd')
Server: Microsoft-IIS/8.5
x-ms-client-request-id: 4651882c-d7ad-4d5e-86ab-f07f47dcb41e
request-id: 6aedbf93-4bc2-4586-8845-fd45590136af
x-ms-request-id: 6aedbf93-4bc2-4586-8845-fd45590136af
X-Content-Type-Options: nosniff
DataServiceVersion: 3.0;
X-Powered-By: ASP.NET
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Sun, 08 Feb 2015 06:21:27 GMT

{"odata.metadata":"https://media.windows.net/api/$metadata#AssetDeliveryPolicies/@Element",
"Id":"nb:adpid:UUID:92b0f6ba-3c9f-49b6-a5fa-2a8703b04ecd",
"Name":"Clear Policy",
"AssetDeliveryProtocol":7,
"AssetDeliveryPolicyType":2,
"AssetDeliveryConfiguration":null,
"Created":"2015-02-08T06:21:27.6908329Z",
"LastModified":"2015-02-08T06:21:27.6908329Z"}
```

### <a name="link-asset-with-asset-delivery-policy"></a><a id="link_asset_with_asset_delivery_policy"></a>Liaison d’un élément multimédia à la stratégie de remise d’élément multimédia
La demande HTTP suivante lie la ressource spécifiée à la stratégie de remise de ressources.

Demande :

```console
POST https://media.windows.net/api/Assets('nb%3Acid%3AUUID%3A86933344-9539-4d0c-be7d-f842458693e0')/$links/DeliveryPolicies HTTP/1.1
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
Content-Type: application/json
Authorization: Bearer <ENCODED JWT TOKEN> 
x-ms-version: 2.19
x-ms-client-request-id: 56d2763f-6e72-419d-ba3c-685f6db97e81
Host: media.windows.net

{"uri":"https://media.windows.net/api/AssetDeliveryPolicies('nb%3Aadpid%3AUUID%3A92b0f6ba-3c9f-49b6-a5fa-2a8703b04ecd')"}
```

Réponse :

```output
HTTP/1.1 204 No Content
```


## <a name="dynamicenvelopeencryption-asset-delivery-policy"></a>Stratégie de remise de ressources DynamicEnvelopeEncryption
### <a name="create-content-key-of-the-envelopeencryption-type-and-link-it-to-the-asset"></a>Création de la clé de contenu de type EnvelopeEncryption et liaison à la ressource
Lorsque vous spécifiez la stratégie de remise DynamicEnvelopeEncryption, vous devez veiller à lier votre ressource à une clé de contenu de type EnvelopeEncryption. Pour plus d’informations, consultez la page [Création d’une clé de contenu](media-services-rest-create-contentkey.md)).

### <a name="get-delivery-url"></a><a id="get_delivery_url"></a>Obtention de l’URL de remise
Obtenez l’URL de remise pour la méthode de remise spécifiée de la clé de contenu créée à l’étape précédente. Un client utilise l’URL retournée pour demander une clé AES ou une licence PlayReady afin de lire le contenu protégé.

Spécifiez le type d’URL à obtenir dans le corps de la demande HTTP. Si vous protégez votre contenu avec PlayReady, demandez une URL d’acquisition de licence PlayReady Media Services, en utilisant 1 pour keyDeliveryType : {"keyDeliveryType":1}. Si vous protégez votre contenu avec le chiffrement d’enveloppe, demandez une URL d’acquisition de clé en spécifiant 2 pour keyDeliveryType :{"keyDeliveryType":2}.

Demande :

```console
POST https://media.windows.net/api/ContentKeys('nb:kid:UUID:dc88f996-2859-4cf7-a279-c52a9d6b2f04')/GetKeyDeliveryUrl HTTP/1.1
Content-Type: application/json
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN> 
x-ms-version: 2.19
x-ms-client-request-id: 569d4b7c-a446-4edc-b77c-9fb686083dd8
Host: media.windows.net
Content-Length: 21

{"keyDeliveryType":2}
```

Réponse :

```output
HTTP/1.1 200 OK
Cache-Control: no-cache
Content-Length: 198
Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
Server: Microsoft-IIS/8.5
x-ms-client-request-id: 569d4b7c-a446-4edc-b77c-9fb686083dd8
request-id: d26f65d2-fe65-4136-8fcf-31545be68377
x-ms-request-id: d26f65d2-fe65-4136-8fcf-31545be68377
X-Content-Type-Options: nosniff
DataServiceVersion: 3.0;
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Sun, 08 Feb 2015 21:42:30 GMT

{"odata.metadata":"media.windows.net/api/$metadata#Edm.String","value":"https://amsaccount1.keydelivery.mediaservices.windows.net/?KID=dc88f996-2859-4cf7-a279-c52a9d6b2f04"}
```


### <a name="create-asset-delivery-policy"></a>Création d’une stratégie de remise d’éléments multimédias
La demande HTTP suivante crée la stratégie **AssetDeliveryPolicy** configurée pour appliquer le chiffrement dynamique en enveloppe (**DynamicEnvelopeEncryption**) au protocole **HLS** (dans cet exemple, les autres protocoles ne pourront pas être diffusés en continu). 

Pour plus d'informations sur les valeurs que vous pouvez spécifier au moment de la création d'une AssetDeliveryPolicy, consultez la section [Types utilisés au moment de la définition d'AssetDeliveryPolicy](#types) .   

Demande :

```console
POST https://media.windows.net/api/AssetDeliveryPolicies HTTP/1.1
Content-Type: application/json
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
User-Agent: Microsoft ADO.NET Data Services
Authorization: Bearer <ENCODED JWT TOKEN> 
x-ms-version: 2.19
x-ms-client-request-id: fff319f6-71dd-4f6c-af27-b675c0066fa7
Host: media.windows.net

{"Name":"AssetDeliveryPolicy","AssetDeliveryProtocol":4,"AssetDeliveryPolicyType":3,"AssetDeliveryConfiguration":"[{\"Key\":2,\"Value\":\"https:\\/\\/amsaccount1.keydelivery.mediaservices.windows.net\\/\"}]"}
```


Réponse :

```output
HTTP/1.1 201 Created
Cache-Control: no-cache
Content-Length: 460
Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
Location: media.windows.net/api/AssetDeliveryPolicies('nb%3Aadpid%3AUUID%3Aec9b994e-672c-4a5b-8490-a464eeb7964b')
Server: Microsoft-IIS/8.5
x-ms-client-request-id: fff319f6-71dd-4f6c-af27-b675c0066fa7
request-id: c2a1ac0e-9644-474f-b38f-b9541c3a7c5f
x-ms-request-id: c2a1ac0e-9644-474f-b38f-b9541c3a7c5f
X-Content-Type-Options: nosniff
DataServiceVersion: 3.0;
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Mon, 09 Feb 2015 05:24:38 GMT

{"odata.metadata":"media.windows.net/api/$metadata#AssetDeliveryPolicies/@Element","Id":"nb:adpid:UUID:ec9b994e-672c-4a5b-8490-a464eeb7964b","Name":"AssetDeliveryPolicy","AssetDeliveryProtocol":4,"AssetDeliveryPolicyType":3,"AssetDeliveryConfiguration":"[{\"Key\":2,\"Value\":\"https:\\/\\/amsaccount1.keydelivery.mediaservices.windows.net\\/\"}]","Created":"2015-02-09T05:24:38.9167436Z","LastModified":"2015-02-09T05:24:38.9167436Z"}
```


### <a name="link-asset-with-asset-delivery-policy"></a>Liaison d’un élément multimédia à la stratégie de remise d’élément multimédia
Consultez la rubrique [Liaison d’un élément multimédia à la stratégie de remise d’élément multimédia](#link_asset_with_asset_delivery_policy)

## <a name="dynamiccommonencryption-asset-delivery-policy"></a>Stratégie de remise de ressources DynamicCommonEncryption
### <a name="create-content-key-of-the-commonencryption-type-and-link-it-to-the-asset"></a>Création de la clé de contenu de type CommonEncryption et liaison à la ressource
Lorsque vous spécifiez la stratégie de remise DynamicCommonEncryption, vous devez veiller à lier votre ressource à une clé de contenu de type CommonEncryption. Pour plus d’informations, consultez la page [Création d’une clé de contenu](media-services-rest-create-contentkey.md)).

### <a name="get-delivery-url"></a>Obtention de l’URL de remise
Obtenez l’URL de remise pour la méthode de remise PlayReady de la clé de contenu créée à l’étape précédente. Un client utilise l’URL retournée pour demander une licence PlayReady afin de lire le contenu protégé. Pour plus d’informations, consultez la rubrique [Obtention de l’URL de remise](#get_delivery_url).

### <a name="create-asset-delivery-policy"></a>Création d’une stratégie de remise d’éléments multimédias
La demande HTTP suivante crée la stratégie **AssetDeliveryPolicy** configurée pour appliquer le chiffrement dynamique commun (**DynamicCommonEncryption**) au protocole **Smooth Streaming** (dans cet exemple, les autres protocoles ne pourront pas être diffusés en continu). 

Pour plus d'informations sur les valeurs que vous pouvez spécifier au moment de la création d'une AssetDeliveryPolicy, consultez la section [Types utilisés au moment de la définition d'AssetDeliveryPolicy](#types) .   

Demande :

```console
POST https://media.windows.net/api/AssetDeliveryPolicies HTTP/1.1
Content-Type: application/json
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
User-Agent: Microsoft ADO.NET Data Services
Authorization: Bearer <ENCODED JWT TOKEN> 
x-ms-version: 2.19
x-ms-client-request-id: fff319f6-71dd-4f6c-af27-b675c0066fa7
Host: media.windows.net

{"Name":"AssetDeliveryPolicy","AssetDeliveryProtocol":1,"AssetDeliveryPolicyType":4,"AssetDeliveryConfiguration":"[{\"Key\":2,\"Value\":\"https:\\/\\/amsaccount1.keydelivery.mediaservices.windows.net\/PlayReady\/"}]"}
```


Si vous souhaitez protéger votre contenu à l’aide de Widevine DRM, mettez à jour les valeurs AssetDeliveryConfiguration pour utiliser WidevineLicenseAcquisitionUrl (dont la valeur est 7) et indiquez l’URL d’un service de remise de licence. Vous pouvez utiliser les partenaires AMS suivants pour vous aider à fournir des licences Widevine : [Axinom](https://www.axinom.com), [EZDRM](https://ezdrm.com/), [castLabs](https://castlabs.com/company/partners/azure/).

Par exemple : 

```console
{"Name":"AssetDeliveryPolicy","AssetDeliveryProtocol":2,"AssetDeliveryPolicyType":4,"AssetDeliveryConfiguration":"[{\"Key\":7,\"Value\":\"https:\\/\\/example.net\/WidevineLicenseAcquisition\/"}]"}
```

> [!NOTE]
> Lors du chiffrement avec Widevine, vous ne pourriez effectuer une remise qu’avec DASH. Veillez à spécifier DASH (2) dans le protocole de remise de ressources.
> 
> 

### <a name="link-asset-with-asset-delivery-policy"></a>Liaison d’un élément multimédia à la stratégie de remise d’élément multimédia
Consultez la rubrique [Liaison d’un élément multimédia à la stratégie de remise d’élément multimédia](#link_asset_with_asset_delivery_policy)

## <a name="types-used-when-defining-assetdeliverypolicy"></a><a id="types"></a>Types utilisés durant la définition de AssetDeliveryPolicy

### <a name="assetdeliveryprotocol"></a>AssetDeliveryProtocol

La valeur d’énumération suivante décrit les valeurs que vous pouvez définir pour le protocole de remise de ressources.

```csharp
[Flags]
public enum AssetDeliveryProtocol
{
    /// <summary>
    /// No protocols.
    /// </summary>
    None = 0x0,

    /// <summary>
    /// Smooth streaming protocol.
    /// </summary>
    SmoothStreaming = 0x1,

    /// <summary>
    /// MPEG Dynamic Adaptive Streaming over HTTP (DASH)
    /// </summary>
    Dash = 0x2,

    /// <summary>
    /// Apple HTTP Live Streaming protocol.
    /// </summary>
    HLS = 0x4,

    ProgressiveDownload = 0x10, 

    /// <summary>
    /// Include all protocols.
    /// </summary>
    All = 0xFFFF
}
```

### <a name="assetdeliverypolicytype"></a>AssetDeliveryPolicyType

La valeur d’énumération suivante décrit les valeurs que vous pouvez définir pour le type de protocole de remise de ressources.  

```csharp
public enum AssetDeliveryPolicyType
{
    /// <summary>
    /// Delivery Policy Type not set.  An invalid value.
    /// </summary>
    None,

    /// <summary>
    /// The Asset should not be delivered via this AssetDeliveryProtocol. 
    /// </summary>
    Blocked, 

    /// <summary>
    /// Do not apply dynamic encryption to the asset.
    /// </summary>
    /// 
    NoDynamicEncryption,  

    /// <summary>
    /// Apply Dynamic Envelope encryption.
    /// </summary>
    DynamicEnvelopeEncryption,

    /// <summary>
    /// Apply Dynamic Common encryption.
    /// </summary>
    DynamicCommonEncryption
    }
```

### <a name="contentkeydeliverytype"></a>ContentKeyDeliveryType

La valeur d’énumération suivante décrit les valeurs que vous pouvez utiliser pour configurer la méthode de remise de la clé de contenu au client.
    
```csharp
public enum ContentKeyDeliveryType
{
    /// <summary>
    /// None.
    ///
    </summary>
    None = 0,

    /// <summary>
    /// Use PlayReady License acquisition protocol
    ///
    </summary>
    PlayReadyLicense = 1,

    /// <summary>
    /// Use MPEG Baseline HTTP key protocol.
    ///
    </summary>
    BaselineHttp = 2,

    /// <summary>
    /// Use Widevine License acquisition protocol
    ///
    </summary>
    Widevine = 3

}
```


### <a name="assetdeliverypolicyconfigurationkey"></a>AssetDeliveryPolicyConfigurationKey

La valeur d’énumération suivante décrit les valeurs que vous pouvez définir pour configurer les clés utilisées pour obtenir une configuration spécifique pour une stratégie de remise de ressources.

```csharp
public enum AssetDeliveryPolicyConfigurationKey
{
    /// <summary>
    /// No policies.
    /// </summary>
    None,

    /// <summary>
    /// Exact Envelope key URL.
    /// </summary>
    EnvelopeKeyAcquisitionUrl,

    /// <summary>
    /// Base key url that will have KID=<Guid> appended for Envelope.
    /// </summary>
    EnvelopeBaseKeyAcquisitionUrl,

    /// <summary>
    /// The initialization vector to use for envelope encryption in Base64 format.
    /// </summary>
    EnvelopeEncryptionIVAsBase64,

    /// <summary>
    /// The PlayReady License Acquisition Url to use for common encryption.
    /// </summary>
    PlayReadyLicenseAcquisitionUrl,

    /// <summary>
    /// The PlayReady Custom Attributes to add to the PlayReady Content Header
    /// </summary>
    PlayReadyCustomAttributes,

    /// <summary>
    /// The initialization vector to use for envelope encryption.
    /// </summary>
    EnvelopeEncryptionIV,

    /// <summary>
    /// Widevine DRM acquisition url
    /// </summary>
    WidevineLicenseAcquisitionUrl
}
```

## <a name="additional-notes"></a>Remarques supplémentaires

* Widevine est un service fourni par Google Inc. soumis aux conditions de service et à la politique de confidentialité de Google, Inc.

## <a name="media-services-learning-paths"></a>Parcours d’apprentissage de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

