---
title: Prendre en main la diffusion de contenus à la demande avec REST | Microsoft Docs
description: Ce didacticiel vous présente les étapes d’implémentation d’une application de diffusion de contenu à la demande avec Azure Media Services au moyen de l’API REST.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.assetid: 88194b59-e479-43ac-b179-af4f295e3780
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/20/2019
ms.author: juliako
ms.openlocfilehash: ddad462658465c07624f078e20c224750c5180c9
ms.sourcegitcommit: 2c586a0fbec6968205f3dc2af20e89e01f1b74b5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92019475"
---
# <a name="get-started-with-delivering-content-on-demand-using-rest"></a>Prendre en main la diffusion de contenus à la demande avec REST

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)] 

> [!NOTE]
> Aucune nouvelle fonctionnalité ni fonction n’est ajoutée à Media Services v2. <br/>Découvrez la dernière version, [Media Services v3](../latest/index.yml). Consultez aussi [Conseils de migration de v2 vers v3](../latest/migrate-from-v2-to-v3.md).

Ce document de démarrage rapide vous guide à travers les étapes d’implémentation d’une application de diffusion de contenu vidéo à la demande (VoD) avec les API REST Azure Media Services (AMS).

Il présente le workflow Media Services de base et les objets et tâches de programmation les plus courants requis pour le développement Media Services. À la fin de ce didacticiel, vous êtes en mesure de lire en continu ou de télécharger de façon progressive l’exemple de fichier multimédia que vous avez chargé, encodé et téléchargé.

L’image suivante illustre certains des objets couramment utilisés lors du développement d’applications VoD par rapport au modèle OData de Media Services.

Cliquez sur l’image pour l’afficher en plein écran.  

[![Diagramme montrant certains des objets les plus couramment utilisés dans le modèle de données d’objet Azure Media Services pour le développement d’applications de vidéo à la demande.](./media/media-services-rest-get-started/media-services-overview-object-model-small.png)](./media/media-services-rest-get-started/media-services-overview-object-model.png#lightbox)

## <a name="prerequisites"></a>Prérequis
Les conditions préalables suivantes sont requises pour commencer à développer avec les API REST et Media Services.

* Un compte Azure. Pour plus d’informations, consultez [Essai gratuit Azure](https://azure.microsoft.com/pricing/free-trial/).
* Un compte Media Services. Pour créer un compte Media Services, consultez [Création d’un compte Media Services](media-services-portal-create-account.md).
* Présentation du développement avec l’API REST Media Services. Pour plus d’informations, consultez [Vue d’ensemble de l’API REST Media Services](media-services-rest-how-to-use.md).
* Une application de votre choix qui peut envoyer des demandes et réponses HTTP. Ce didacticiel utilise [Fiddler](https://www.telerik.com/download/fiddler).

Ce document de démarrage rapide présente les tâches suivantes.

1. Démarrer les points de terminaison de streaming (à l’aide du portail Azure).
2. Connexion à un compte Media Services à l’aide de l’API REST
3. Création d’une ressource et téléchargement d’un fichier vidéo à l’aide de l’API REST
4. Encodage du fichier source en un ensemble de fichiers MP4 à débit adaptatif avec l’API REST
5. Publication des éléments et obtention des URL de diffusion et de téléchargement progressif avec l’API REST
6. Lire votre contenu.

>[!NOTE]
>Un nombre limite de 1 000 000 a été défini pour les différentes stratégies AMS (par exemple, pour la stratégie de localisateur ou pour ContentKeyAuthorizationPolicy). Utilisez le même ID de stratégie si vous utilisez toujours les mêmes jours/autorisations d’accès, par exemple, les stratégies pour les localisateurs destinés à demeurer en place pendant une longue période (stratégies sans chargement). Pour plus d’informations, consultez [cet](media-services-dotnet-manage-entities.md#limit-access-policies) article.

Pour plus d’informations sur les entités REST AMS utilisées dans cet article, consultez [Informations de référence sur l’API REST d’azure Media Services](/rest/api/media/operations/azure-media-services-rest-api-reference). Consultez également [Concepts Azure Media Services](media-services-concepts.md).

>[!NOTE]
>Lors de l’accès aux entités dans Media Services, vous devez définir les valeurs et les champs d’en-tête spécifiques dans vos requêtes HTTP. Pour plus d'informations, consultez [Installation pour le développement REST API de Media Services](media-services-rest-how-to-use.md).

## <a name="start-streaming-endpoints-using-the-azure-portal"></a>Démarrer les points de terminaison de streaming à l’aide du portail Azure

Lorsque vous utilisez Azure Media Services, la diffusion de vidéos en continu à débit binaire adaptatif constitue l’un des scénarios les plus courants. Media Services assure l’empaquetage dynamique, qui vous permet de distribuer juste-à-temps un contenu encodé en MP4 à débit adaptatif dans un format de diffusion en continu pris en charge par Media Services (MPEG DASH, HLS, Smooth Streaming), sans qu’il vous soit nécessaire de stocker des versions pré-empaquetées de chacun de ces formats.

>[!NOTE]
>Une fois votre compte AMS créé, un point de terminaison de diffusion continue **par défaut** est ajouté à l’état **Arrêté**. Pour démarrer la diffusion en continu de votre contenu et tirer parti de l’empaquetage et du chiffrement dynamiques, le point de terminaison de streaming à partir duquel vous souhaitez diffuser du contenu doit se trouver à l’état **En cours d’exécution**.

Pour démarrer le point de terminaison de streaming, procédez comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Dans la fenêtre Paramètres, cliquez sur Points de terminaison de streaming.
3. Cliquez sur le point de terminaison de diffusion en continu par défaut.

    La fenêtre DEFAULT STREAMING ENDPOINT DETAILS (DÉTAILS DU POINT DE TERMINAISON DE STREAMING PAR DÉFAUT) s’affiche.

4. Cliquez sur l’icône de démarrage.
5. Cliquez sur le bouton Enregistrer pour enregistrer vos modifications.

## <a name="connect-to-the-media-services-account-with-rest-api"></a><a id="connect"></a>Connexion à un compte Media Services à l’aide de l’API REST

Pour savoir comment vous connecter à l’API AMS, consultez [Accéder à l’API Azure Media Services avec l’authentification Azure AD](media-services-use-aad-auth-to-access-ams-api.md). 

## <a name="create-a-new-asset-and-upload-a-video-file-with-rest-api"></a><a id="upload"></a>Création d’une ressource et téléchargement d’un fichier vidéo à l’aide de l’API REST

Dans Media Services, vous téléchargez vos fichiers numériques dans une ressource. L’entité **Asset** peut contenir des fichiers vidéo et audio, des images, des collections de miniatures, des pistes textuelles et des légendes (et les métadonnées concernant ces fichiers).  Une fois les fichiers téléchargés dans la ressource, votre contenu est stocké en toute sécurité dans le cloud et peut faire l’objet d’un traitement et d’une diffusion en continu.

Les options de création de ressources sont une des valeurs que vous devez fournir lors de la création d’une ressource. La propriété **Options** est une valeur d’énumération qui décrit les options de chiffrement permettant de créer une ressource. Une valeur valide est une des valeurs de la liste ci-dessous, et non une combinaison de valeurs de cette liste :

* **None** = **0** : aucun chiffrement. Quand vous utilisez cette option, votre contenu n'est pas protégé pendant le transit ou le repos dans le stockage.
    Si vous prévoyez de fournir un MP4 sous forme de téléchargement progressif, utilisez cette option.
* **StorageEncrypted** = **1** : permet de chiffrer votre contenu en clair localement en utilisant le chiffrement AES-256 bits, puis de le télécharger vers le Stockage Azure où il est chiffré pour le stockage, au repos. Les éléments multimédias protégés par le chiffrement de stockage sont automatiquement déchiffrés et placés dans un système de fichiers chiffré avant d’être encodés, puis éventuellement rechiffrés avant d’être rechargés sous la forme d’un nouvel élément multimédia de sortie. Le principal cas d'utilisation du chiffrement de stockage concerne la sécurisation de fichiers multimédias d'entrée de haute qualité avec un chiffrement renforcé au repos sur le disque.
* **CommonEncryptionProtected** = **2** : utilisez cette option quand vous chargez du contenu qui a déjà été chiffré et protégé avec la DRM Common Encryption ou PlayReady (par exemple Smooth Streaming protégé avec la DRM PlayReady).
* **EnvelopeEncryptionProtected** = **4** : utilisez cette option lorsque vous téléchargez un contenu au format TLS chiffré avec AES. Les fichiers doivent avoir été encodés et chiffrés par le gestionnaire de transformation Transform Manager.

### <a name="create-an-asset"></a>Créer une ressource
Une ressource est un conteneur pour plusieurs types ou ensembles d’objets dans Media Services, y compris des fichiers vidéo, audio, des images, des collections de miniatures, des pistes textuelles et des légendes. Dans l’API REST, la création d’une ressource nécessite d’envoyer une demande POST vers Media Services et de placer les informations de propriété concernant votre ressource dans le corps de la demande.

L’exemple suivant montre comment créer une ressource.

**Demande HTTP**

```console
POST https://wamsbayclus001rest-hs.cloudapp.net/api/Assets HTTP/1.1
Content-Type: application/json
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN>
x-ms-version: 2.19
x-ms-client-request-id: c59de965-bc89-4295-9a57-75d897e5221e
Host: wamsbayclus001rest-hs.cloudapp.net
Content-Length: 45

{"Name":"BigBuckBunny.mp4", "Options":"0"}
```

**Réponse HTTP**

Si l’opération réussit, l’élément suivant est retourné :

```console
HTTP/1.1 201 Created
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
X-Powered-By: ASP.NET
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Sun, 18 Jan 2015 22:06:40 GMT

{  
    odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#Assets/@Element",
   "Id":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
   "State":0,
   "Created":"2015-01-18T22:06:40.6010903Z",
   "LastModified":"2015-01-18T22:06:40.6010903Z",
   "AlternateId":null,
   "Name":"BigBuckBunny2.mp4",
   "Options":0,
   "Uri":"https://storagetestaccount001.blob.core.windows.net/asset-9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
   "StorageAccountName":"storagetestaccount001"
}
```

### <a name="create-an-assetfile"></a>Création d’un AssetFile
L’entité [AssetFile](/rest/api/media/operations/assetfile) représente un fichier audio ou vidéo stocké dans un conteneur d’objets blob. Un fichier de ressources est toujours associé à une ressource et une ressource peut contenir un ou plusieurs AssetFiles. La tâche de Media Services Encoder échoue si un objet de fichier de ressources n’est pas associé à un fichier numérique dans un conteneur d’objets blob.

Après avoir téléchargé le fichier de ressource numérique dans un conteneur d’objets blob, vous utilisez la requête HTTP **MERGE** pour mettre à jour AssetFile avec des informations sur votre fichier multimédia (comme indiqué ultérieurement dans cette rubrique).

**Demande HTTP**

```console
POST https://wamsbayclus001rest-hs.cloudapp.net/api/Files HTTP/1.1
Content-Type: application/json
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN>
x-ms-version: 2.19
Host: wamsbayclus001rest-hs.cloudapp.net
Content-Length: 164

{  
   "IsEncrypted":"false",
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
   "EncryptionVersion":null,
   "EncryptionScheme":null,
   "IsEncrypted":false,
   "EncryptionKeyId":null,
   "InitializationVector":null,
   "IsPrimary":false,
   "LastModified":"2015-01-19T00:34:08.1934137Z",
   "Created":"2015-01-19T00:34:08.1934137Z",
   "MimeType":"video/mp4",
   "ContentChecksum":null
}
```

### <a name="creating-the-accesspolicy-with-write-permission"></a>Création d’AccessPolicy avec autorisation d’écriture
Avant de télécharger des fichiers dans le stockage blob, définissez les droits de la stratégie d’accès pour l’écriture sur une ressource. Pour ce faire, utilisez POST avec une demande HTTP sur le jeu d’entités AccessPolicies. N’oubliez pas de définir une valeur DurationInMinutes après la création ou vous recevrez en réponse un message d’erreur interne de serveur 500. Pour plus d’informations sur AccessPolicies, consultez [AccessPolicy](/rest/api/media/operations/accesspolicy).

L’exemple suivant montre comment créer une stratégie AccessPolicy :

**Demande HTTP**

```console
POST https://wamsbayclus001rest-hs.cloudapp.net/api/AccessPolicies HTTP/1.1
Content-Type: application/json
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN>
x-ms-version: 2.19
Host: wamsbayclus001rest-hs.cloudapp.net
Content-Length: 74

{"Name":"NewUploadPolicy", "DurationInMinutes":"440", "Permissions":"2"}
```

**Réponse HTTP**

Si l’opération réussit, la réponse suivante est retournée :

```console
HTTP/1.1 201 Created
Cache-Control: no-cache
Content-Length: 312
Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
Location: https://wamsbayclus001rest-hs.cloudapp.net/api/AccessPolicies('nb%3Apid%3AUUID%3Abe0ac48d-af7d-4877-9d60-1805d68bffae')
Server: Microsoft-IIS/8.5
request-id: 74c74545-7e0a-4cd6-a440-c1c48074a970
x-ms-request-id: 74c74545-7e0a-4cd6-a440-c1c48074a970
X-Content-Type-Options: nosniff
DataServiceVersion: 3.0;
X-Powered-By: ASP.NET
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Sun, 18 Jan 2015 22:18:06 GMT

{  
   "odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#AccessPolicies/@Element",
   "Id":"nb:pid:UUID:be0ac48d-af7d-4877-9d60-1805d68bffae",
   "Created":"2015-01-18T22:18:06.6370575Z",
   "LastModified":"2015-01-18T22:18:06.6370575Z",
   "Name":"NewUploadPolicy",
   "DurationInMinutes":440.0,
   "Permissions":2
}
```

### <a name="get-the-upload-url"></a>Obtention de l’URL de téléchargement

Pour recevoir l’URL de téléchargement réelle, créez un localisateur SAS. Les localisateurs définissent l’heure de début et le type de point de terminaison de connexion pour les clients qui souhaitent accéder aux fichiers d’une ressource. Vous pouvez créer plusieurs entités de localisateurs pour une paire AccessPolicy et Asset donnée, afin de gérer les différentes demandes et besoins des clients. Chacun de ces localisateurs utilise la valeur StartTime et la valeur DurationInMinutes d’AccessPolicy pour déterminer la durée pendant laquelle une URL peut être utilisée. Pour plus d’informations, consultez la rubrique [Localisateur](/rest/api/media/operations/locator).

Une URL SAS a le format suivant :

`{https://myaccount.blob.core.windows.net}/{asset name}/{video file name}?{SAS signature}`

Certaines considérations s’appliquent :

* Vous ne pouvez avoir plus de cinq localisateurs uniques associés à une ressource donnée. 
* Si vous avez besoin de télécharger vos fichiers immédiatement, vous devez définir la valeur StartTime sur cinq minutes avant l’heure actuelle. Cela vient du fait qu’il peut exister un décalage horaire entre votre ordinateur client et Media Services. Votre valeur StartTime doit être au format DateHeure suivant : AAAA-MM-JJTHH:mm:ssZ (par exemple, « 2014-05-23T17:53:50Z »).    
* Il peut y avoir un délai de 30 à 40 secondes après la création d’un localisateur avant qu’il soit disponible. Ce problème s’applique aux localisateurs [d’URL SAS](../../storage/common/storage-sas-overview.md) et d’origine.

L’exemple suivant montre comment créer un localisateur d’URL SAS, tel que défini par la propriété Type dans le corps de la demande (« 1 » pour un localisateur SAS et « 2 » pour un localisateur d’origine à la demande). La propriété **Path** retournée contient l’URL que vous devez utiliser pour télécharger votre fichier.

**Demande HTTP**

```console
POST https://wamsbayclus001rest-hs.cloudapp.net/api/Locators HTTP/1.1
Content-Type: application/json
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN>
x-ms-version: 2.19
Host: wamsbayclus001rest-hs.cloudapp.net
Content-Length: 178

{  
   "AccessPolicyId":"nb:pid:UUID:be0ac48d-af7d-4877-9d60-1805d68bffae",
   "AssetId":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
   "StartTime":"2015-02-18T16:45:53",
   "Type":1
}
```

**Réponse HTTP**

Si l’opération réussit, la réponse suivante est retournée :

```console
HTTP/1.1 201 Created
Cache-Control: no-cache
Content-Length: 949
Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
Location: https://wamsbayclus001rest-hs.cloudapp.net/api/Locators('nb%3Alid%3AUUID%3Aaf57bdd8-6751-4e84-b403-f3c140444b54')
Server: Microsoft-IIS/8.5
request-id: 2adeb1f8-89c5-4cc8-aa4f-08cdfef33ae0
x-ms-request-id: 2adeb1f8-89c5-4cc8-aa4f-08cdfef33ae0
X-Content-Type-Options: nosniff
DataServiceVersion: 3.0;
X-Powered-By: ASP.NET
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Mon, 19 Jan 2015 03:01:29 GMT

{  
   "odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#Locators/@Element",
   "Id":"nb:lid:UUID:af57bdd8-6751-4e84-b403-f3c140444b54",
   "ExpirationDateTime":"2015-02-19T00:05:53",
   "Type":1,
   "Path":"https://storagetestaccount001.blob.core.windows.net/asset-f438649c-313c-46e2-8d68-7d2550288247?sv=2012-02-12&sr=c&si=af57bdd8-6751-4e84-b403-f3c140444b54&sig=fE4btwEfZtVQFeC0Wh3Kwks2OFPQfzl5qTMW5YytiuY%3D&st=2015-02-18T16%3A45%3A53Z&se=2015-02-19T00%3A05%3A53Z",
   "BaseUri":"https://storagetestaccount001.blob.core.windows.net/asset-f438649c-313c-46e2-8d68-7d2550288247",
   "ContentAccessComponent":"?sv=2012-02-12&sr=c&si=af57bdd8-6751-4e84-b403-f3c140444b54&sig=fE4btwEfZtVQFeC0Wh3Kwks2OFPQfzl5qTMW5YytiuY%3D&st=2015-02-18T16%3A45%3A53Z&se=2015-02-19T00%3A05%3A53Z",
   "AccessPolicyId":"nb:pid:UUID:be0ac48d-af7d-4877-9d60-1805d68bffae",
   "AssetId":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1",
   "StartTime":"2015-02-18T16:45:53",
   "Name":null
}
```

### <a name="upload-a-file-into-a-blob-storage-container"></a>Téléchargement d’un fichier dans un conteneur de stockage d’objets blob
Après avoir défini AccessPolicy et Locator, le fichier réel est téléchargé vers un conteneur de stockage d’objets blob Microsoft Azure à l’aide des API REST Azure Storage. Vous devez télécharger les fichiers en tant qu’objets blob de blocs. Les objets blob de pages ne sont pas pris en charge par Azure Media Services.  

> [!NOTE]
> Vous devez ajouter le nom du fichier à télécharger dans la valeur **Path** du localisateur reçue dans la section précédente. Par exemple : `https://storagetestaccount001.blob.core.windows.net/asset-e7b02da4-5a69-40e7-a8db-e8f4f697aac0/BigBuckBunny.mp4?`.
>
>

Pour plus d’informations sur l’utilisation d’objets blob de stockage Microsoft Azure, consultez [API REST du service BLOB](/rest/api/storageservices/blob-service-rest-api).

### <a name="update-the-assetfile"></a>Mise à jour d’AssetFile
Maintenant que vous avez téléchargé votre fichier, mettez à jour les informations de taille FileAsset (et autres). Par exemple :

```console
MERGE https://wamsbayclus001rest-hs.cloudapp.net/api/Files('nb%3Acid%3AUUID%3Af13a0137-0a62-9d4c-b3b9-ca944b5142c5') HTTP/1.1
Content-Type: application/json
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN>
x-ms-version: 2.19
Host: wamsbayclus001rest-hs.cloudapp.net

{  
   "ContentFileSize":"1186540",
   "Id":"nb:cid:UUID:f13a0137-0a62-9d4c-b3b9-ca944b5142c5",
   "MimeType":"video/mp4",
   "Name":"BigBuckBunny.mp4",
   "ParentAssetId":"nb:cid:UUID:9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1"
}
```

**Réponse HTTP**

Si l’opération réussit, l’élément suivant est retourné :

```console
HTTP/1.1 204 No Content
...
```

## <a name="delete-the-locator-and-accesspolicy"></a>Suppression du localisateur et d’AcessPolicy
**Demande HTTP**

```console
DELETE https://wamsbayclus001rest-hs.cloudapp.net/api/Locators('nb%3Alid%3AUUID%3Aaf57bdd8-6751-4e84-b403-f3c140444b54') HTTP/1.1
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN>
x-ms-version: 2.19
Host: wamsbayclus001rest-hs.cloudapp.net
```

**Réponse HTTP**

Si l’opération réussit, l’élément suivant est retourné :

```console
HTTP/1.1 204 No Content
...
```

**Demande HTTP**

```console
DELETE https://wamsbayclus001rest-hs.cloudapp.net/api/AccessPolicies('nb%3Apid%3AUUID%3Abe0ac48d-af7d-4877-9d60-1805d68bffae') HTTP/1.1
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN>
x-ms-version: 2.19
Host: wamsbayclus001rest-hs.cloudapp.net
```

**Réponse HTTP**

Si l’opération réussit, l’élément suivant est retourné :

```console
HTTP/1.1 204 No Content
...
```

## <a name="encode-the-source-file-into-a-set-of-adaptive-bitrate-mp4-files"></a><a id="encode"></a>Encoder le fichier source en un ensemble de fichiers MP4 à débit adaptatif

Après avoir reçu des éléments multimédias dans Media Services, vous pouvez encoder un média, modifier le format de ce dernier, lui appliquer un filigrane, etc. avant de le livrer à des clients. Afin de garantir des performances et une disponibilité optimales, ces activités sont planifiées et exécutées dans de nombreuses instances de rôle en arrière-plan. Ces activités s’appellent des travaux et chaque Travail se compose de tâches atomiques qui effectuent le travail à proprement parler sur le fichier de ressource (pour plus d’informations, consultez les descriptions de [Travail](/rest/api/media/operations/job) et [Tâche](/rest/api/media/operations/task)).

Comme mentionné précédemment, lorsque vous travaillez avec Azure Media Services, un des scénarios les plus courants est la diffusion de contenu à débit adaptatif à vos clients. Media Services peut empaqueter dynamiquement un ensemble de fichiers MP4 à débit adaptatif dans un des formats suivants : HTTP Live Streaming (HLS), Smooth Streaming, MPEG DASH.

La section suivante montre comment créer un travail contenant une tâche d’encodage. La tâche spécifie de transcoder le fichier mezzanine en un ensemble de MP4 à débit adaptatif à l’aide de **Media Encoder Standard**. La section indique également comment surveiller la progression du traitement du travail. Une fois le travail terminé, vous pourrez créer les localisateurs nécessaires pour accéder à vos ressources.

### <a name="get-a-media-processor"></a>Obtention d’un processeur multimédia
Dans Media Services, un processeur multimédia est un composant qui gère une tâche de traitement spécifique, telle que l’encodage, la conversion de format, le chiffrement ou le déchiffrement de contenu multimédia. Pour la tâche d’encodage indiquée dans ce didacticiel, nous allons utiliser Media Encoder Standard.

Le code suivant demande l’ID de l’encodeur.

**Demande HTTP**

```console
    GET https://wamsbayclus001rest-hs.cloudapp.net/api/MediaProcessors()?$filter=Name%20eq%20'Media%20Encoder%20Standard' HTTP/1.1
    DataServiceVersion: 1.0;NetFx
    MaxDataServiceVersion: 3.0;NetFx
    Accept: application/json
    Accept-Charset: UTF-8
    Authorization: Bearer <ENCODED JWT TOKEN>
    x-ms-version: 2.19
    Host: wamsbayclus001rest-hs.cloudapp.net
```

**Réponse HTTP**

```console
HTTP/1.1 200 OK
Cache-Control: no-cache
Content-Length: 273
Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
Server: Microsoft-IIS/8.5
request-id: 6beb04b4-55a7-480d-8aa8-e5c5d59ffa1f
x-ms-request-id: 6beb04b4-55a7-480d-8aa8-e5c5d59ffa1f
X-Content-Type-Options: nosniff
DataServiceVersion: 3.0;
X-Powered-By: ASP.NET
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Mon, 19 Jan 2015 07:54:09 GMT

{  
   "odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#MediaProcessors",
   "value":[  
      {  
         "Id":"nb:mpid:UUID:ff4df607-d419-42f0-bc17-a481b1331e56",
         "Description":"Media Encoder Standard",
         "Name":"Media Encoder Standard",
         "Sku":"",
         "Vendor":"Microsoft",
         "Version":"1.1"
      }
   ]
}
```

### <a name="create-a-job"></a>Créer un travail
Chaque travail peut comporter une ou plusieurs tâches, en fonction du type de traitement que vous souhaitez accomplir. Via l’API REST, vous pouvez créer des travaux et les tâches associées de deux manières : Des tâches peuvent être définies inline via la propriété de navigation de tâches sur les entités de travail, ou via le traitement par lots OData. Le Kit de développement logiciel (SDK) Media Services utilise le traitement par lots. Toutefois, pour une meilleure lisibilité des exemples de code dans cet article, les tâches sont définies inline. Pour plus d’informations sur le traitement par lots, consultez [Traitement par lots d’Open Data Protocol (OData)](https://www.odata.org/documentation/odata-version-3-0/batch-processing/).

L’exemple suivant montre comment créer et publier un projet avec une tâche visant à encoder une vidéo en une résolution et une qualité spécifiques. La section suivante de la documentation contient la liste de toutes les [présélections de tâches](./media-services-mes-presets-overview.md) prises en charge par Media Encoder Standard.  

**Demande HTTP**

```console
POST https://wamsbayclus001rest-hs.cloudapp.net/api/Jobs HTTP/1.1
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN>
x-ms-version: 2.19
Host: wamsbayclus001rest-hs.cloudapp.net
Content-Length: 482

{  
   "Name":"NewTestJob",
   "InputMediaAssets":[  
      {  
         "__metadata":{  
            "uri":"https://wamsbayclus001rest-hs.net/api/Assets('nb%3Acid%3AUUID%3A9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1')"
         }
      }
   ],
   "Tasks":[  
      {  
         "Configuration":"Adaptive Streaming",
         "MediaProcessorId":"nb:mpid:UUID:ff4df607-d419-42f0-bc17-a481b1331e56",
         "TaskBody":"<?xml version=\"1.0\" encoding=\"utf-8\"?><taskBody><inputAsset>JobInputAsset(0)</inputAsset>
            <outputAsset>JobOutputAsset(0)</outputAsset></taskBody>"
      }
   ]
}
```

**Réponse HTTP**

Si l’opération réussit, la réponse suivante est retournée :

```console
HTTP/1.1 201 Created
Cache-Control: no-cache
Content-Length: 1215
Content-Type: application/json;odata=verbose;charset=utf-8
Location: https://wamsbayclus001rest-hs.cloudapp.net/api/Jobs('nb%3Ajid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')
Server: Microsoft-IIS/8.5
request-id: 532ac1ec-a475-4dce-b2d5-7c8ce94ac87c
x-ms-request-id: 532ac1ec-a475-4dce-b2d5-7c8ce94ac87c
X-Content-Type-Options: nosniff
DataServiceVersion: 3.0;
X-Powered-By: ASP.NET
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Mon, 19 Jan 2015 08:04:35 GMT

{  
   "d":{  
      "__metadata":{  
         "id":"https://wamsbayclus001rest-hs.cloudapp.net/api/Jobs('nb%3Ajid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')",
         "uri":"https://wamsbayclus001rest-hs.cloudapp.net/api/Jobs('nb%3Ajid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')",
         "type":"Microsoft.Cloud.Media.Vod.Rest.Data.Models.Job"
      },
      "Tasks":{  
         "__deferred":{  
            "uri":"https://wamsbayclus001rest-hs.cloudapp.net/api/Jobs('nb%3Ajid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')/Tasks"
         }
      },
      "OutputMediaAssets":{  
         "__deferred":{  
            "uri":"https://wamsbayclus001rest-hs.cloudapp.net/api/Jobs('nb%3Ajid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')/OutputMediaAssets"
         }
      },
      "InputMediaAssets":{  
         "__deferred":{  
            "uri":"https://wamsbayclus001rest-hs.cloudapp.net/api/Jobs('nb%3Ajid%3AUUID%3A9bc8ff20-24fb-4fdb-9d7c-b04c7ee573a1')/InputMediaAssets"
         }
      },
      "Id":"nb:jid:UUID:71d2dd33-efdf-ec43-8ea1-136a110bd42c",
      "Name":"NewTestJob",
      "Created":"2015-01-19T08:04:34.3287057Z",
      "LastModified":"2015-01-19T08:04:34.3287057Z",
      "EndTime":null,
      "Priority":0,
      "RunningDuration":0,
      "StartTime":null,
      "State":0,
      "TemplateId":null,
      "JobNotificationSubscriptions":{  
         "__metadata":{  
            "type":"Collection(Microsoft.Cloud.Media.Vod.Rest.Data.Models.JobNotificationSubscription)"
         },
         "results":[  

         ]
      }
   }
}
```

Il convient de noter quelques points importants concernant les demandes de travail :

* les propriétés TaskBody DOIVENT utiliser un XML littéral pour définir le nombre de ressources d’entrée ou de sortie qui sont utilisées par la tâche. L’article Tâche contient la définition du schéma XML pour le XML.
* Dans la définition TaskBody, chaque valeur interne de `<inputAsset>` et `<outputAsset>` doit être définie en tant que JobInputAsset(valeur) ou JobOutputAsset(valeur).
* Une tâche peut comporter plusieurs ressources de sortie. Un JobOutputAsset(x) ne peut être utilisé qu’une fois en tant que résultat d’une tâche dans un travail.
* Vous pouvez spécifier JobInputAsset ou JobOutputAsset en tant que ressource d’entrée d’une tâche.
* Les tâches ne doivent pas former un cycle.
* Le paramètre de valeur que vous transmettez à JobInputAsset ou à JobOutputAsset représente la valeur d’index pour une ressource. Les ressources réelles sont définies dans les propriétés de navigation InputMediaAssets et OutputMediaAssets de la définition d’entité de travail.

> [!NOTE]
> Étant donné que Media Services est basé sur OData v3, les ressources dans les collections de propriétés de navigation InputMediaAssets et OutputMediaAssets sont référencées par une paire nom de valeur « __metadata : uri ».
>
>

* InputMediaAssets mappe vers une ou plusieurs ressources que vous avez créées dans Media Services. Les OutputMediaAssets sont créés par le système. Ils ne font pas référence à une ressource existante.
* OutputMediaAssets peut être nommé à l’aide de l’attribut assetName. Si cet attribut n’est pas présent, le nom d’OutputMediaAsset est la valeur de texte interne de l’élément `<outputAsset>` avec le suffixe de la valeur du nom du travail ou de l’ID de travail (dans le cas où la propriété Name n’est pas définie). Par exemple, si vous affectez à assetName la valeur « Sample », la propriété de Nom d’OutputMediaAsset est définie sur « Sample ». Toutefois, si vous n’avez pas défini de valeur pour assetName, mais avez défini le nom du travail comme « NewJob », le nom d’OutputMediaAsset est « JobOutputAsset(value)_NewJob ».

    L’exemple suivant montre comment définir l’attribut assetName :

    ```console
    "<?xml version=\"1.0\" encoding=\"utf-8\"?><taskBody><inputAsset>JobInputAsset(0)</inputAsset><outputAsset assetName=\"CustomOutputAssetName\">JobOutputAsset(0)</outputAsset></taskBody>"
    ```
* Pour activer le chaînage des tâches :

  * un travail doit comporter au moins deux tâches
  * Il doit y avoir au moins une tâche dont l’entrée correspond à la sortie d’une autre tâche du travail.

Pour plus d’informations, consultez la page [Création d’une tâche d’encodage avec l’API REST Media Services](media-services-rest-encode-asset.md).

### <a name="monitor-processing-progress"></a>Surveillance de la progression du traitement
Vous pouvez récupérer l’état du travail à l’aide de la propriété State, comme l’illustre l’exemple suivant :

**Demande HTTP**

```console
GET https://wamsbayclus001rest-hs.net/api/Jobs('nb%3Ajid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')/State HTTP/1.1
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN>
Host: wamsbayclus001rest-hs.net
Content-Length: 0
```

**Réponse HTTP**

Si l’opération réussit, la réponse suivante est retournée :

```console
HTTP/1.1 200 OK
Cache-Control: no-cache
Content-Length: 17
Content-Type: application/json;odata=verbose;charset=utf-8
Server: Microsoft-IIS/7.5
x-ms-request-id: 01324d81-18e2-4493-a3e5-c6186209f0c8
X-Content-Type-Options: nosniff
DataServiceVersion: 1.0;
X-AspNet-Version: 4.0.30319
X-Powered-By: ASP.NET
Date: Sun, 13 May 2012 05:16:53 GMT

{"d":{"State":2}}
```

### <a name="cancel-a-job"></a>Annulation d’une tâche
Media Services vous permet d’annuler les travaux en cours d’exécution via la fonction CancelJob. Cet appel renvoie un code d’erreur 400 si vous essayez d’annuler un Travail dont l’état est annulé, en cours d’annulation, erreur ou terminé.

L’exemple suivant montre comment appeler CancelJob.

**Demande HTTP**

```console
GET https://wamsbayclus001rest-hs.net/API/CancelJob?jobid='nb%3ajid%3aUUID%3a71d2dd33-efdf-ec43-8ea1-136a110bd42c' HTTP/1.1
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN>
Host: wamsbayclus001rest-hs.net
```

Si l’opération réussit, un code de réponse 204 est retourné, sans corps de message.

> [!NOTE]
> Vous devez encoder par l’URL l’ID de travail (normalement nb:jid:UUID: valeur) lorsque vous la transmettez en tant que paramètre à la fonction CancelJob.
>
>

### <a name="get-the-output-asset"></a>Obtention de la ressource de sortie
Le code suivant montre comment demander l’ID de la ressource de sortie

**Demande HTTP**

```console
GET https://wamsbayclus001rest-hs.cloudapp.net/api/Jobs('nb%3Ajid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')/OutputMediaAssets() HTTP/1.1
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Accept: application/json
Accept-Charset: UTF-8
User-Agent: Microsoft ADO.NET Data Services
Authorization: Bearer <ENCODED JWT TOKEN>
x-ms-version: 2.19
Host: wamsbayclus001rest-hs.cloudapp.net
```

**Réponse HTTP**

```console
HTTP/1.1 200 OK
Cache-Control: no-cache
Content-Length: 445
Content-Type: application/json;odata=minimalmetadata;streaming=true;charset=utf-8
Server: Microsoft-IIS/8.5
request-id: 73cd605d-066a-46f1-8358-f4bd25a9220a
x-ms-request-id: 73cd605d-066a-46f1-8358-f4bd25a9220a
X-Content-Type-Options: nosniff
DataServiceVersion: 3.0;
X-Powered-By: ASP.NET
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Mon, 19 Jan 2015 08:28:13 GMT

{  
   "odata.metadata":"https://wamsbayclus001rest-hs.cloudapp.net/api/$metadata#Assets",
   "value":[  
      {  
         "Id":"nb:cid:UUID:71d2dd33-efdf-ec43-8ea1-136a110bd42c",
         "State":0,
         "Created":"2015-01-19T07:52:15.603",
         "LastModified":"2015-01-19T07:52:15.603",
         "AlternateId":null,
         "Name":"Multibitrate MP4s",
         "Options":0,
         "Uri":"https://storagetestaccount001.blob.core.windows.net/asset-71d2dd33-efdf-ec43-8ea1-136a110bd42c",
         "StorageAccountName":"storagetestaccount001"
      }
   ]
}
```

## <a name="publish-the-asset-and-get-streaming-and-progressive-download-urls-with-rest-api"></a><a id="publish_get_urls"></a>Publication des éléments et obtention des URL de diffusion et de téléchargement progressif avec l’API REST

Pour diffuser en continu ou télécharger un élément multimédia, vous devez tout d'abord le « publier » en créant un localisateur. Les localisateurs assurent l’accès aux fichiers contenus dans l’élément multimédia. Media Services prend en charge deux types de localisateurs : les localisateurs OnDemandOrigin, utilisés pour diffuser du contenu multimédia (par exemple, MPEG DASH, HLS ou Smooth Streaming) et les localisateurs d'URL SAS (signature d'accès partagé), utilisés pour télécharger des fichiers multimédias. 

Une fois que vous avez créé les localisateurs, vous pouvez générer les URL utilisées pour transmettre en continu ou télécharger les fichiers.

>[!NOTE]
>Une fois votre compte AMS créé, un point de terminaison de diffusion continue **par défaut** est ajouté à l’état **Arrêté**. Pour démarrer la diffusion en continu de votre contenu et tirer parti de l’empaquetage et du chiffrement dynamiques, le point de terminaison de streaming à partir duquel vous souhaitez diffuser du contenu doit se trouver à l’état **En cours d’exécution**.

Les URL de diffusion en continu pour Smooth Streaming ont le format suivant :

```console
{streaming endpoint name-media services account name}.streaming.mediaservices.windows.net/{locator ID}/{filename}.ism/Manifest
```

Les URL de diffusion en continu pour HLS ont le format suivant :

```console
{streaming endpoint name-media services account name}.streaming.mediaservices.windows.net/{locator ID}/{filename}.ism/Manifest(format=m3u8-aapl)
```

Les URL de diffusion en continu pour MPEG DASH ont le format suivant :

```console
{streaming endpoint name-media services account name}.streaming.mediaservices.windows.net/{locator ID}/{filename}.ism/Manifest(format=mpd-time-csf)
```

Les URL SAS permettant de télécharger les fichiers ont le format suivant :

```console
{blob container name}/{asset name}/{file name}/{SAS signature}
```

Cette section montre comment effectuer les tâches suivantes, nécessaires pour « publier » vos ressources.  

* Création d’AccessPolicy avec autorisation de lecture
* Création d’une URL SAS pour le téléchargement de contenu
* Création d’une URL d’origine pour la diffusion en continu de contenu

### <a name="creating-the-accesspolicy-with-read-permission"></a>Création d’AccessPolicy avec autorisation de lecture
Avant de télécharger ou de diffuser en continu du contenu multimédia, il convient tout d’abord de définir une AccessPolicy avec des autorisations de lecture et de créer l’entité de localisateur appropriée, qui spécifie le type de mécanisme de remise que vous souhaitez activer pour vos clients. Pour plus d’informations sur les propriétés disponibles, consultez [Propriétés de l’entité AccessPolicy](/rest/api/media/operations/accesspolicy#accesspolicy_properties).

L’exemple suivant montre comment spécifier AccessPolicy pour les autorisations de lecture d’une ressource donnée.

```console
POST https://wamsbayclus001rest-hs.net/API/AccessPolicies HTTP/1.1
Content-Type: application/json
Accept: application/json
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN>
Host: wamsbayclus001rest-hs.net
Content-Length: 74
Expect: 100-continue

{"Name": "DownloadPolicy", "DurationInMinutes" : "300", "Permissions" : 1}
```

Si l’opération réussit, un code de succès 201 est renvoyé et décrit l’entité AccessPolicy que vous avez créée. Vous utilisez ensuite l’ID de l’AccessPolicy et l’ID de la ressource contenant le fichier que vous souhaitez remettre (par exemple, une ressource de sortie) pour créer l’entité de localisateur.

> [!NOTE]
> Ce flux de travail de base est le même que pour le téléchargement d’un fichier lors de la réception d’une ressource (comme expliqué précédemment dans cette rubrique). En outre, comme pour le téléchargement de fichiers, si vous (ou vos clients) devez accéder à vos fichiers immédiatement, définissez la valeur StartTime cinq minutes avant l’heure actuelle. Cela vient du fait qu’il peut exister un décalage horaire entre le client et Media Services. La valeur StartTime doit être au format DateHeure suivant : AAAA-MM-JJTHH:mm:ssZ (par exemple, « 2014-05-23T17:53:50Z »).
>
>

### <a name="creating-a-sas-url-for-downloading-content"></a>Création d’une URL SAS pour le téléchargement de contenu
Le code suivant montre comment obtenir une URL qui peut être utilisée pour télécharger une ressource créée et téléchargée précédemment. AccessPolicy dispose d’autorisations en lecture et le chemin d’accès du localisateur fait référence à une URL de téléchargement SAS.

```console
POST https://wamsbayclus001rest-hs.net/API/Locators HTTP/1.1
Content-Type: application/json
Accept: application/json
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN>
Host: wamsbayclus001rest-hs.net
Content-Length: 182
Expect: 100-continue

{"AccessPolicyId": "nb:pid:UUID:38c71dd0-44c5-4c5f-8418-08bb6fbf7bf8", "AssetId" : "nb:cid:UUID:71d2dd33-efdf-ec43-8ea1-136a110bd42c", "StartTime" : "2014-05-17T16:45:53", "Type":1}
```

Si l’opération réussit, la réponse suivante est retournée :

```console
HTTP/1.1 201 Created
Cache-Control: no-cache
Content-Length: 1150
Content-Type: application/json;odata=verbose;charset=utf-8
Location: https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A8e5a821d-2194-4d00-8884-adf979856874')
Server: Microsoft-IIS/7.5
x-ms-request-id: 8cfd8556-3064-416a-97f2-67d9e35f0911
X-Content-Type-Options: nosniff
DataServiceVersion: 1.0;
X-AspNet-Version: 4.0.30319
X-Powered-By: ASP.NET
Date: Mon, 14 May 2012 21:41:32 GMT

{  
   "d":{  
      "__metadata":{  
         "id":"https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A8e5a821d-2194-4d00-8884-adf979856874')",
         "uri":"https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A8e5a821d-2194-4d00-8884-adf979856874')",
         "type":"Microsoft.Cloud.Media.Vod.Rest.Data.Models.Locator"
      },
      "AccessPolicy":{  
         "__deferred":{  
            "uri":"https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A8e5a821d-2194-4d00-8884-adf979856874')/AccessPolicy"
         }
      },
      "Asset":{  
         "__deferred":{  
            "uri":"https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A71d2dd33-efdf-ec43-8ea1-136a110bd42c')/Asset"
         }
      },
      "Id":"nb:lid:UUID:8e5a821d-2194-4d00-8884-adf979856874",
      "ExpirationDateTime":"\/Date(1337049393000)\/",
      "Type":1,
      "Path":"https://storagetestaccount001.blob.core.windows.net/asset-71d2dd33-efdf-ec43-8ea1-136a110bd42c?st=2012-05-14T21%3A36%3A33Z&se=2012-05-15T02%3A36%3A33Z&sr=c&si=8e5a821d-2194-4d00-8884-adf979856874&sig=y75dViDpC5V8WutrXM%2B%2FGpR3uOtqmlISiNlHU1YUBOg%3D",
      "AccessPolicyId":"nb:pid:UUID:38c71dd0-44c5-4c5f-8418-08bb6fbf7bf8",
      "AssetId":"nb:cid:UUID:71d2dd33-efdf-ec43-8ea1-136a110bd42c",
      "StartTime":"\/Date(1337031393000)\/"
   }
}
```

La propriété **Path** retournée contient l’URL SAS.

> [!NOTE]
> Si vous téléchargez du contenu avec chiffrement de stockage, vous devez le déchiffrer manuellement avant de le restituer ou utiliser le processeur multimédia de déchiffrement de stockage dans une tâche de traitement pour obtenir une sortie de fichiers traités en clair vers un OutputAsset et le télécharger à partir de cette ressource. Pour plus d’informations sur le traitement, consultez la page Création d’une tâche d’encodage avec l’API REST Media Services En outre, les localisateurs d’URL SAS ne peuvent pas être mis à jour après leur création. Par exemple, vous ne pouvez pas réutiliser le même localisateur avec une valeur StartTime mise à jour. Cela est dû au mode de création des URL SAS. Si vous souhaitez accéder à une ressource pour la télécharger après l’expiration d’un localisateur, vous devez en créer un nouveau, avec une nouvelle valeur StartTime.
>
>

### <a name="download-files"></a>Télécharger les fichiers
Après avoir défini AccessPolicy et le localisateur, vous pouvez télécharger des fichiers à l’aide des API REST Azure Storage.  

> [!NOTE]
> Vous devez ajouter le nom de fichier du fichier à télécharger à la valeur **Path** du Localisateur, obtenue dans la section précédente. Par exemple, `https://storagetestaccount001.blob.core.windows.net/asset-e7b02da4-5a69-40e7-a8db-e8f4f697aac0/BigBuckBunny.mp4` ? . . .

Pour plus d’informations sur l’utilisation d’objets blob de stockage Microsoft Azure, consultez [API REST du service BLOB](/rest/api/storageservices/blob-service-rest-api).

Suite à la tâche de codage que vous avez exécutée antérieurement (encodage vers un jeu de MP4 adaptatifs), vous disposez de plusieurs fichiers MP4 que vous pouvez télécharger progressivement. Par exemple :    

* `https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_650kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

* `https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_400kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

* `https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_3400kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

* `https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_2250kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

* `https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_1500kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

* `https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_1000kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

* `https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

* `https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_AAC_und_ch2_56kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

### <a name="creating-a-streaming-url-for-streaming-content"></a>Création d’une URL pour la diffusion en continu de contenu
Le code suivant montre comment créer un localisateur d’URL de diffusion en continu  :

```console
POST https://wamsbayclus001rest-hs/API/Locators HTTP/1.1
Content-Type: application/json
Accept: application/json
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN>
Host: wamsbayclus001rest-hs
Content-Length: 182
Expect: 100-continue

{"AccessPolicyId": "nb:pid:UUID:38c71dd0-44c5-4c5f-8418-08bb6fbf7bf8", "AssetId" : "nb:cid:UUID:eb5540a2-116e-4d36-b084-7e9958f7f3c3", "StartTime" : "2014-05-17T16:45:53",, "Type":2}
```

Si l’opération réussit, la réponse suivante est retournée :

```console
HTTP/1.1 201 Created
Cache-Control: no-cache
Content-Length: 981
Content-Type: application/json;odata=verbose;charset=utf-8
Location: https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A52034bf6-dfae-4d83-aad3-3bd87dcb1a5d')
Server: Microsoft-IIS/7.5
x-ms-request-id: 2eac4158-fc78-4fa2-81ee-c9f582dc385f
X-Content-Type-Options: nosniff
DataServiceVersion: 1.0;
X-AspNet-Version: 4.0.30319
X-Powered-By: ASP.NET
Date: Mon, 14 May 2012 21:41:39 GMT

{  
   "d":{  
      "__metadata":{  
         "id":"https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A52034bf6-dfae-4d83-aad3-3bd87dcb1a5d')",
         "uri":"https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A52034bf6-dfae-4d83-aad3-3bd87dcb1a5d')",
         "type":"Microsoft.Cloud.Media.Vod.Rest.Data.Models.Locator"
      },
      "AccessPolicy":{  
         "__deferred":{  
            "uri":"https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A52034bf6-dfae-4d83-aad3-3bd87dcb1a5d')/AccessPolicy"
         }
      },
      "Asset":{  
         "__deferred":{  
            "uri":"https://wamsbayclus001rest-hs.net/api/Locators('nb%3Alid%3AUUID%3A52034bf6-dfae-4d83-aad3-3bd87dcb1a5d')/Asset"
         }
      },
      "Id":"nb:lid:UUID:52034bf6-dfae-4d83-aad3-3bd87dcb1a5d",
      "ExpirationDateTime":"\/Date(1337049395000)\/",
      "Type":2,
      "Path":"http://wamsbayclus001rest-hs.net/52034bf6-dfae-4d83-aad3-3bd87dcb1a5d/",
      "AccessPolicyId":"nb:pid:UUID:38c71dd0-44c5-4c5f-8418-08bb6fbf7bf8",
      "AssetId":"nb:cid:UUID:eb5540a2-116e-4d36-b084-7e9958f7f3c3",
      "StartTime":"\/Date(1337031395000)\/"
   }
}
```

Pour diffuser une URL d’origine de diffusion en continu lisse dans un lecteur multimédia de diffusion en continu, vous devez ajouter la propriété de chemin d’accès avec le nom du fichier manifeste de diffusion en continu lisse, suivie de « /manifest ».

`http://amstestaccount001.streaming.mediaservices.windows.net/ebf733c4-3e2e-4a68-b67b-cc5159d1d7f2/BigBuckBunny.ism/manifest`

Pour la diffusion en continu HLS, ajoutez (format=m3u8-aapl) après « /manifest ».

`http://amstestaccount001.streaming.mediaservices.windows.net/ebf733c4-3e2e-4a68-b67b-cc5159d1d7f2/BigBuckBunny.ism/manifest(format=m3u8-aapl)`

Pour la diffusion en continu MPEG DASH, ajoutez (format=mpd-time-csf) après « /manifest ».

`http://amstestaccount001.streaming.mediaservices.windows.net/ebf733c4-3e2e-4a68-b67b-cc5159d1d7f2/BigBuckBunny.ism/manifest(format=mpd-time-csf)`


## <a name="play-your-content"></a><a id="play"></a>Lecture de votre contenu
Pour tester votre vidéo, utilisez le [lecteur Azure Media Services](https://aka.ms/azuremediaplayer).

Pour tester le téléchargement progressif, collez l’URL dans un navigateur (par exemple, Internet Explorer, Chrome ou Safari).

## <a name="next-steps-media-services-learning-paths"></a>Étapes suivantes : Parcours d’apprentissage de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]