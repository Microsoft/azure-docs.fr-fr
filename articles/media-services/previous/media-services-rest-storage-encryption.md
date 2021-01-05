---
title: Chiffrer votre contenu avec le chiffrement de stockage à l'aide de l'API REST AMS
description: Découvrez comment chiffrer votre contenu avec le chiffrement de stockage à l'aide des API REST AMS.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.assetid: a0a79f3d-76a1-4994-9202-59b91a2230e0
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/20/2019
ms.author: juliako
ms.custom: devx-track-csharp
ms.openlocfilehash: 8a3a51644f61d4a1e118798986f9c6fb6c52d0e5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89264162"
---
# <a name="encrypting-your-content-with-storage-encryption"></a>Chiffrer votre contenu avec le chiffrement de stockage

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> Pour suivre ce tutoriel, vous avez besoin d’un compte Azure. Pour plus d’informations, consultez [Essai gratuit Azure](https://azure.microsoft.com/pricing/free-trial/).   > Aucune nouvelle fonctionnalité ni fonction n’est ajoutée à Media Services v2. <br/>Découvrez la dernière version, [Media Services v3](../latest/index.yml). Consultez aussi [Conseils de migration de v2 vers v3](../latest/migrate-from-v2-to-v3.md).
>

Cet article donne une vue d'ensemble du chiffrement de stockage AMS et vous montre comment télécharger le contenu chiffré du stockage :

* Créez une clé de contenu.
* Créez une ressource. Définissez AssetCreationOption sur StorageEncryption lors de la création de la ressource.
  
     Les ressources chiffrées sont associées à des clés de contenu.
* Liez la clé de contenu à la ressource.  
* Définissez les paramètres liés au chiffrement sur les entités AssetFile.

## <a name="considerations"></a>Considérations 

Si vous souhaitez remettre une ressource à chiffrement de stockage, vous devez configurer la stratégie de remise de la ressource. Avant de pouvoir diffuser votre ressource en continu, le serveur de diffusion supprime le chiffrement de stockage et transmet en continu votre contenu à l’aide de la stratégie de remise spécifiée. Pour plus d'informations, consultez [Configuration des stratégies de distribution de ressources](media-services-rest-configure-asset-delivery-policy.md).

Lors de l’accès aux entités dans Media Services, vous devez définir les valeurs et les champs d’en-tête spécifiques dans vos requêtes HTTP. Pour plus d'informations, consultez [Installation pour le développement REST API de Media Services](media-services-rest-how-to-use.md). 

### <a name="storage-side-encryption"></a>Chiffrement côté stockage

|Option de chiffrement|Description|Media Services v2|Media Services v3|
|---|---|---|---|
|Chiffrement du stockage de Media Services|Chiffrement AES-256, clé gérée par Media Services|Pris en charge<sup>(1)</sup>|Non pris en charge<sup>(2)</sup>|
|[Storage Service Encryption pour les données au repos](../../storage/common/storage-service-encryption.md)|Chiffrement côté serveur proposé par le stockage Azure, clé gérée par Azure ou par un client|Prise en charge|Prise en charge|
|[Chiffrement de stockage côté client](../../storage/common/storage-client-side-encryption.md)|Chiffrement côté client proposé par le stockage Azure, clé gérée par un client dans un coffre de clés|Non pris en charge|Non pris en charge|

<sup>1</sup> Bien que Media Services prenne en charge la gestion de contenu en clair/sans aucune forme de chiffrement, ce n’est pas conseillé.

<sup>2</sup> Dans Media Services v3, le chiffrement de stockage (chiffrement AES-256) est uniquement pris en charge pour la compatibilité descendante lorsque vos ressources ont été créées avec Media Services v2. Cela signifie que la version v3 fonctionne avec les ressources chiffrées du stockage existant mais qu’elle n’autorisera pas de nouvelles créations.

## <a name="connect-to-media-services"></a>Connexion à Media Services

Pour savoir comment vous connecter à l’API AMS, consultez [Accéder à l’API Azure Media Services avec l’authentification Azure AD](media-services-use-aad-auth-to-access-ams-api.md). 

## <a name="storage-encryption-overview"></a>Vue d’ensemble du chiffrement du stockage
Le chiffrement de stockage AMS applique le mode de chiffrement **AES-CTR** à la totalité du fichier.  Le mode AES-CTR est un chiffrement par blocs qui permet de chiffrer des données de longueur arbitraire sans avoir besoin de remplissage. Il fonctionne en chiffrant un bloc de compteur avec l'algorithme AES, puis en appliquant l’opération XOR à la sortie d’AES avec les données à chiffrer ou déchiffrer.  Le bloc de compteur utilisé est construit en copiant la valeur InitializationVector sur les octets 0 à 7 de la valeur du compteur et les octets 8 à 15 de la valeur du compteur ont la valeur zéro. Dans le bloc de compteur de 16 octets, les octets 8 à 15 (c’est-à-dire les octets les moins significatifs) sont utilisés comme simple entier non signé de 64 bits, incrémenté de un pour chacun des blocs suivants de données traitées, et conservé dans l’ordre des octets du réseau. Si cet entier atteint la valeur maximale (0xFFFFFFFFFFFFFFFF), son incrémentation réinitialise le compteur de blocs à zéro (octets 8 à 15) sans affecter les autres 64 bits du compteur (c’est-à-dire les octets 0 à 7).   Pour maintenir la sécurité du mode de chiffrement AES-CTR, la valeur InitializationVector pour un identificateur de clé donné pour chaque clé de contenu doit être unique pour chaque fichier et les fichiers doivent avoir une longueur inférieure à 2^64 blocs.  Cette valeur unique permet de faire en sorte qu’aucune valeur de compteur ne soit jamais réutilisée avec une clé donnée. Pour plus d'informations sur le mode CTR, consultez [cette page wiki](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#CTR) (l'article wiki utilise le terme « Nonce » au lieu de « InitializationVector »).

Utilisez le **chiffrement du stockage** pour chiffrer votre contenu localement à l’aide du chiffrement AES 256 bits, puis chargez-le vers Azure Storage où il est stocké au repos sous forme chiffrée. Les éléments multimédias protégés par le chiffrement de stockage sont automatiquement déchiffrés et placés dans un système de fichiers chiffré avant d’être encodés, puis éventuellement rechiffrés avant d’être rechargés sous la forme d’un nouvel élément multimédia de sortie. Le principal cas d'utilisation du chiffrement de stockage concerne la sécurisation de fichiers multimédias d'entrée de haute qualité avec un chiffrement renforcé au repos sur le disque.

Pour fournir un élément multimédia avec chiffrement de stockage, vous devez configurer la stratégie de remise de l'élément multimédia afin que Media Services sache comment vous souhaitez remettre votre contenu. Pour que votre élément multimédia puisse être diffusé en continu, le serveur de diffusion supprime le chiffrement de stockage et diffuse votre contenu à l'aide de la stratégie de remise spécifiée (par exemple AES, chiffrement commun ou aucun chiffrement).

## <a name="create-contentkeys-used-for-encryption"></a>Créer des ContentKeys utilisées pour le chiffrement
Les ressources chiffrées sont associées à des clés de chiffrement du stockage. Créez la clé de contenu à utiliser pour le chiffrement avant de créer les fichiers de ressources. Cet article décrit la création d’une clé de contenu.

Voici les étapes générales pour la génération de clés de contenu que vous associez à des ressources devant être chiffrées. 

1. Pour le chiffrement du stockage, générez de façon aléatoire une clé AES de 32 octets. 
   
    La clé AES de 32 octets est la clé de contenu de votre ressource, ce qui signifie que tous les fichiers associés à cette ressource doivent utiliser la même clé de contenu lors du déchiffrement. 
2. Appelez les méthodes [GetProtectionKeyId](/rest/api/media/operations/rest-api-functions#getprotectionkeyid) et [GetProtectionKey](/rest/api/media/operations/rest-api-functions#getprotectionkey) pour obtenir le certificat X.509 approprié qui doit être utilisé pour chiffrer votre clé de contenu.
3. Chiffrez votre clé de contenu avec la clé publique du certificat X.509. 
   
   Le Kit de développement logiciel (SDK) Media Services pour .NET utilise RSA avec OAEP lorsque vous effectuez le chiffrement.  Vous trouverez un exemple .NET dans la [fonction EncryptSymmetricKeyData](https://github.com/Azure/azure-sdk-for-media-services/blob/dev/src/net/Client/Common/Common.FileEncryption/EncryptionUtils.cs).
4. Créez une valeur de somme de contrôle calculée à l'aide de l'identificateur de clé et de la clé de contenu. L’exemple .NET suivant calcule la somme de contrôle à l’aide de la partie GUID de l’identificateur de clé et de la clé de contenu en clair.

    ```csharp
    public static string CalculateChecksum(byte[] contentKey, Guid keyId)
    {
        const int ChecksumLength = 8;
        const int KeyIdLength = 16;

        byte[] encryptedKeyId = null;

        // Checksum is computed by AES-ECB encrypting the KID
        // with the content key.
        using (AesCryptoServiceProvider rijndael = new AesCryptoServiceProvider())
        {
            rijndael.Mode = CipherMode.ECB;
            rijndael.Key = contentKey;
            rijndael.Padding = PaddingMode.None;

            ICryptoTransform encryptor = rijndael.CreateEncryptor();
            encryptedKeyId = new byte[KeyIdLength];
            encryptor.TransformBlock(keyId.ToByteArray(), 0, KeyIdLength, encryptedKeyId, 0);
        }

        byte[] retVal = new byte[ChecksumLength];
        Array.Copy(encryptedKeyId, retVal, ChecksumLength);

        return Convert.ToBase64String(retVal);
    }
    ```

5. Créez la clé de contenu avec les valeurs **EncryptedContentKey** (convertie en chaîne codée en Base64), **ProtectionKeyId**, **ProtectionKeyType**, **ContentKeyType** et **Checksum** que vous avez obtenues lors des étapes précédentes.

    Pour le chiffrement du stockage, les propriétés suivantes doivent être incluses dans le corps de la demande.

    Propriété du corps de la demande    | Description
    ---|---
    Id | L'ID de ContentKey est généré en utilisant le format suivant : « nb:kid:UUID:\<NEW GUID> ».
    ContentKeyType | Le type de clé de contenu est un entier qui définit la clé. La valeur est 1 pour le format de chiffrement du stockage.
    EncryptedContentKey | Nous créons une valeur de clé de contenu qui est une valeur de 256 bits (32 octets). La clé est chiffrée à l’aide du certificat X.509 de chiffrement du stockage que nous récupérons à partir de Microsoft Azure Media Services en exécutant une requête HTTP GET pour les méthodes GetProtectionKeyId et GetProtectionKey. À titre d’exemple, consultez le code .NET suivant : la méthode **EncryptSymmetricKeyData** définie [ici](https://github.com/Azure/azure-sdk-for-media-services/blob/dev/src/net/Client/Common/Common.FileEncryption/EncryptionUtils.cs).
    ProtectionKeyId | Il s'agit de l'ID de la clé de protection du certificat de chiffrement de stockage X.509 qui a été utilisé pour chiffrer notre clé de contenu.
    ProtectionKeyType | Il s’agit du type de chiffrement de la clé de protection qui a été utilisé pour chiffrer la clé de contenu. Cette valeur est StorageEncryption(1) dans notre exemple.
    Somme de contrôle |La somme de contrôle calculée MD5 pour la clé de contenu. Elle est calculée en chiffrant l'ID de contenu avec la clé de contenu. L’exemple de code montre comment calculer la somme de contrôle.


### <a name="retrieve-the-protectionkeyid"></a>Récupération de ProtectionKeyId
L’exemple suivant montre comment récupérer ProtectionKeyId, une empreinte de certificat, pour le certificat que vous devez utiliser pour chiffrer votre clé de contenu. Effectuez cette étape pour vous assurer que vous possédez déjà le certificat approprié sur votre ordinateur.

Demande :

```console
GET https://media.windows.net/api/GetProtectionKeyId?contentKeyType=0 HTTP/1.1
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
User-Agent: Microsoft ADO.NET Data Services
Authorization: Bearer <ENCODED JWT TOKEN>
x-ms-version: 2.19
Host: media.windows.net
```

Réponse :

```console
HTTP/1.1 200 OK
Cache-Control: no-cache
Content-Length: 139
Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
Server: Microsoft-IIS/8.5
request-id: 2b6aa7a4-3a09-4b08-b581-26b55667f817
x-ms-request-id: 2b6aa7a4-3a09-4b08-b581-26b55667f817
X-Content-Type-Options: nosniff
DataServiceVersion: 3.0;
X-Powered-By: ASP.NET
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Wed, 04 Feb 2015 02:42:52 GMT

{"odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#Edm.String","value":"7D9BB04D9D0A4A24800CADBFEF232689E048F69C"}
```

### <a name="retrieve-the-protectionkey-for-the-protectionkeyid"></a>Récupération de ProtectionKey pour ProtectionKeyId
L’exemple suivant montre comment récupérer le certificat X.509 à l’aide de ProtectionKeyId, que vous avez reçue lors de l’étape précédente.

Demande :

```console
GET https://media.windows.net/api/GetProtectionKey?ProtectionKeyId='7D9BB04D9D0A4A24800CADBFEF232689E048F69C' HTTP/1.1
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
User-Agent: Microsoft ADO.NET Data Services
Authorization: Bearer <ENCODED JWT TOKEN> 
x-ms-version: 2.19
x-ms-client-request-id: 78d1247a-58d7-40e5-96cc-70ff0dfa7382
Host: media.windows.net
```

Réponse :

```console
HTTP/1.1 200 OK
Cache-Control: no-cache
Content-Length: 1227
Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
Server: Microsoft-IIS/8.5
x-ms-client-request-id: 78d1247a-58d7-40e5-96cc-70ff0dfa7382
request-id: 1523e8f3-8ed2-40fe-8a9a-5d81eb572cc8
x-ms-request-id: 1523e8f3-8ed2-40fe-8a9a-5d81eb572cc8
X-Content-Type-Options: nosniff
DataServiceVersion: 3.0;
X-Powered-By: ASP.NET
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Thu, 05 Feb 2015 07:52:30 GMT

{"odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#Edm.String",
"value":"MIIDSTCCAjGgAwIBAgIQqf92wku/HLJGCbMAU8GEnDANBgkqhkiG9w0BAQQFADAuMSwwKgYDVQQDEyN3YW1zYmx1cmVnMDAxZW5jcnlwdGFsbHNlY3JldHMtY2VydDAeFw0xMjA1MjkwNzAwMDBaFw0zMjA1MjkwNzAwMDBaMC4xLDAqBgNVBAMTI3dhbXNibHVyZWcwMDFlbmNyeXB0YWxsc2VjcmV0cy1jZXJ0MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAzR0SEbXefvUjb9wCUfkEiKtGQ5Gc328qFPrhMjSo+YHe0AVviZ9YaxPPb0m1AaaRV4dqWpST2+JtDhLOmGpWmmA60tbATJDdmRzKi2eYAyhhE76MgJgL3myCQLP42jDusWXWSMabui3/tMDQs+zfi1sJ4Ch/lm5EvksYsu6o8sCv29VRwxfDLJPBy2NlbV4GbWz5Qxp2tAmHoROnfaRhwp6WIbquk69tEtu2U50CpPN2goLAqx2PpXAqA+prxCZYGTHqfmFJEKtZHhizVBTFPGS3ncfnQC9QIEwFbPw6E5PO5yNaB68radWsp5uvDg33G1i8IT39GstMW6zaaG7cNQIDAQABo2MwYTBfBgNVHQEEWDBWgBCOGT2hPhsvQioZimw8M+jOoTAwLjEsMCoGA1UEAxMjd2Ftc2JsdXJlZzAwMWVuY3J5cHRhbGxzZWNyZXRzLWNlcnSCEKn/dsJLvxyyRgmzAFPBhJwwDQYJKoZIhvcNAQEEBQADggEBABcrQPma2ekNS3Wc5wGXL/aHyQaQRwFGymnUJ+VR8jVUZaC/U/f6lR98eTlwycjVwRL7D15BfClGEHw66QdHejaViJCjbEIJJ3p2c9fzBKhjLhzB3VVNiLIaH6RSI1bMPd2eddSCqhDIn3VBN605GcYXMzhYp+YA6g9+YMNeS1b+LxX3fqixMQIxSHOLFZ1G/H2xfNawv0VikH3djNui3EKT1w/8aRkUv/AAV0b3rYkP/jA1I0CPn0XFk7STYoiJ3gJoKq9EMXhit+Iwfz0sMkfhWG12/XO+TAWqsK1ZxEjuC9OzrY7pFnNxs4Mu4S8iinehduSpY+9mDd3dHynNwT4="}
```

### <a name="create-the-content-key"></a>Créez la clé de contenu
Après avoir récupéré le certificat X.509 et utilisé sa clé publique pour chiffrer votre clé de contenu, créez une entité **ContentKey** et définissez ses valeurs de propriété en conséquence.

Une des valeurs que vous devez définir lors de la création d’une clé de contenu est son type. Lorsque vous utilisez le chiffrement de stockage, la valeur doit être définie à « 1 ». 

L’exemple suivant montre comment créer une **ContentKey** avec un **ContentKeyType** défini pour le chiffrement du stockage (« 1 ») et le **ProtectionKeyType** avec la valeur « 0 » pour indiquer que l’ID de la clé de protection est l’empreinte du certificat X.509.  

Requête

```console
POST https://media.windows.net/api/ContentKeys HTTP/1.1
Content-Type: application/json
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
User-Agent: Microsoft ADO.NET Data Services
Authorization: Bearer <ENCODED JWT TOKEN>
x-ms-version: 2.19
Host: media.windows.net
{
"Name":"ContentKey",
"ProtectionKeyId":"7D9BB04D9D0A4A24800CADBFEF232689E048F69C", 
"ContentKeyType":"1", 
"ProtectionKeyType":"0",
"EncryptedContentKey":"your encrypted content key",
"Checksum":"calculated checksum"
}
```

Réponse :

```console
HTTP/1.1 201 Created
Cache-Control: no-cache
Content-Length: 777
Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
Location: https://media.windows.net/api/ContentKeys('nb%3Akid%3AUUID%3A9c8ea9c6-52bd-4232-8a43-8e43d8564a99')
Server: Microsoft-IIS/8.5
request-id: 76e85e0f-5cf1-44cb-b689-b3455888682c
x-ms-request-id: 76e85e0f-5cf1-44cb-b689-b3455888682c
X-Content-Type-Options: nosniff
DataServiceVersion: 3.0;
X-Powered-By: ASP.NET
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Wed, 04 Feb 2015 02:37:46 GMT

{"odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#ContentKeys/@Element",
"Id":"nb:kid:UUID:9c8ea9c6-52bd-4232-8a43-8e43d8564a99","Created":"2015-02-04T02:37:46.9684379Z",
"LastModified":"2015-02-04T02:37:46.9684379Z",
"ContentKeyType":1,
"EncryptedContentKey":"your encrypted content key",
"Name":"ContentKey",
"ProtectionKeyId":"7D9BB04D9D0A4A24800CADBFEF232689E048F69C",
"ProtectionKeyType":0,
"Checksum":"calculated checksum"}
```

## <a name="create-an-asset"></a>Créer une ressource
L’exemple suivant montre comment créer une ressource.

**Demande HTTP**

```console
POST https://media.windows.net/api/Assets HTTP/1.1
Content-Type: application/json
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN>
x-ms-version: 2.19
Host: media.windows.net

{"Name":"BigBuckBunny" "Options":1}
```

**Réponse HTTP**

Si l’opération réussit, la réponse suivante est retournée :

```console
HTP/1.1 201 Created
Cache-Control: no-cache
Content-Length: 452
Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
Location: https://wamsbayclus001rest-hs.cloudapp.net/api/Assets('nb%3Acid%3AUUID%3A9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1')
Server: Microsoft-IIS/8.5
x-ms-client-request-id: c59de965-bc89-4295-9a57-75d897e5221e
request-id: e98be122-ae09-473a-8072-0ccd234a0657
x-ms-request-id: e98be122-ae09-473a-8072-0ccd234a0657
X-Content-Type-Options: nosniff
DataServiceVersion: 3.0;
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Sun, 18 Jan 2015 22:06:40 GMT
{  
   "odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#Assets/@Element",
   "Id":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
   "State":0,
   "Created":"2015-01-18T22:06:40.6010903Z",
   "LastModified":"2015-01-18T22:06:40.6010903Z",
   "AlternateId":null,
   "Name":"BigBuckBunny.mp4",
   "Options":1,
   "Uri":"https://storagetestaccount001.blob.core.windows.net/asset-9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
   "StorageAccountName":"storagetestaccount001"
}
```

## <a name="associate-the-contentkey-with-an-asset"></a>Association de la ContentKey avec une ressource
Après avoir créé la ContentKey, associez-la à votre ressource à l’aide de l’opération $links, comme illustré dans l’exemple suivant :

Demande :

```console
POST https://media.windows.net/api/Assets('nb%3Acid%3AUUID%3Afbd7ce05-1087-401b-aaae-29f16383c801')/$links/ContentKeys HTTP/1.1
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
Content-Type: application/json
Authorization: Bearer <ENCODED JWT TOKEN>
x-ms-version: 2.19
Host: media.windows.net

{"uri":"https://wamsbayclus001rest-hs.cloudapp.net/api/ContentKeys('nb%3Akid%3AUUID%3A01e6ea36-2285-4562-91f1-82c45736047c')"}
```

Réponse :

```console
HTTP/1.1 204 No Content 
```

## <a name="create-an-assetfile"></a>Création d’un AssetFile
L’entité [AssetFile](/rest/api/media/operations/assetfile) représente un fichier audio ou vidéo stocké dans un conteneur d’objets blob. Un fichier de ressources est toujours associé à une ressource et une ressource peut contenir un ou plusieurs fichiers de ressources. La tâche de Media Services Encoder échoue si un objet de fichier de ressources n’est pas associé à un fichier numérique dans un conteneur d’objets blob.

L'instance **AssetFile** et le fichier multimédia réel sont deux objets distincts. L’instance AssetFile contient des métadonnées concernant le fichier multimédia, tandis que le fichier multimédia contient le contenu multimédia réel.

Après avoir téléchargé le fichier multimédia numérique dans un conteneur d’objets blob, vous utilisez la requête HTTP **MERGE** pour mettre à jour AssetFile avec des informations sur votre fichier multimédia (non illustré dans cet article). 

**Demande HTTP**

```console
POST https://media.windows.net/api/Files HTTP/1.1
Content-Type: application/json
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN>
x-ms-version: 2.19
Host: media.windows.net
Content-Length: 164

{  
   "IsEncrypted":"true",
   "EncryptionScheme" : "StorageEncryption", 
   "EncryptionVersion" : "1.0",       
   "EncryptionKeyId" : "nb:kid:UUID:32e6efaf-5fba-4538-b115-9d1cefe43510",
   "InitializationVector" : "397304628502661816</d:InitializationVector",
   "Options":0,
   "IsPrimary":"false",
   "MimeType":"video/mp4",
   "Name":"BigBuckBunny.mp4",
   "ParentAssetId":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1"
}
```

**Réponse HTTP**

```console
HTTP/1.1 201 Created
Cache-Control: no-cache
Content-Length: 535
Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
Location: https://wamsbayclus001rest-hs.cloudapp.net/api/Files('nb%3Acid%3AUUID%3Af13a0137-0a62-9d4c-b3b9-ca944b5142c5')
Server: Microsoft-IIS/8.5
request-id: 98a30e2d-f379-4495-988e-0b79edc9b80e
x-ms-request-id: 98a30e2d-f379-4495-988e-0b79edc9b80e
X-Content-Type-Options: nosniff
DataServiceVersion: 3.0;
X-Powered-By: ASP.NET
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Mon, 19 Jan 2015 00:34:07 GMT

{  
   "odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#Files/@Element",
   "Id":"nb:cid:UUID:f13a0137-0a62-9d4c-b3b9-ca944b5142c5",
   "Name":"BigBuckBunny.mp4",
   "ContentFileSize":"0",
   "ParentAssetId":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
   "EncryptionVersion": "1.0",
   "EncryptionScheme": "StorageEncryption",
   "IsEncrypted":true,
   "EncryptionKeyId":"nb:kid:UUID:32e6efaf-5fba-4538-b115-9d1cefe43510",
   "InitializationVector":"397304628502661816</d:InitializationVector",
   "IsPrimary":false,
   "LastModified":"2015-01-19T00:34:08.1934137Z",
   "Created":"2015-01-19T00:34:08.1934137Z",
   "MimeType":"video/mp4",
   "ContentChecksum":null
}
```
