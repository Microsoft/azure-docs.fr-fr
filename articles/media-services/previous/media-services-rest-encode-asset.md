---
title: Encodage d’une ressource Azure à l’aide de Media Encoder Standard | Microsoft Docs
description: Découvrez comment utiliser Media Encoder Standard pour encoder un contenu multimédia sur Azure Media Services. Les exemples de code utilisent l’API REST.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.assetid: 2a7273c6-8a22-4f82-9bfe-4509ff32d4a4
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/20/2019
ms.author: juliako
ms.openlocfilehash: b0bff0c6b3ef7934188b548bd06c35d83b323fde
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91359594"
---
# <a name="how-to-encode-an-asset-by-using-media-encoder-standard"></a>Encodage d’une ressource à l’aide de Media Encoder Standard

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!div class="op_single_selector"]
> * [.NET](media-services-dotnet-encode-with-media-encoder-standard.md)
> * [REST](media-services-rest-encode-asset.md)
> * [Portail](media-services-portal-encode.md)
>
>

## <a name="overview"></a>Vue d’ensemble

Pour fournir une vidéo numérique sur Internet, vous devez compresser le contenu multimédia. Les fichiers vidéo numériques sont volumineux et peuvent être trop importants pour être fournis sur Internet ou pour que les appareils de vos clients les affichent correctement. L’encodage est le processus de compression audio et vidéo permettant à vos clients d’afficher votre contenu multimédia.

Les tâches d’encodage sont une des opérations de traitement les plus courantes dans Azure Media Services. Vous créez des tâches d’encodage pour convertir des fichiers multimédias d’un encodage à un autre. Lorsque vous les encodez, vous pouvez utiliser l’encodeur intégré de Media Services (Media Encoder Standard). Vous pouvez également utiliser un encodeur fourni par un partenaire Media Services. Les encodeurs tiers sont disponibles via Place de marché Azure. Vous pouvez spécifier les détails des tâches d’encodage à l’aide de chaînes de présélection définies pour votre encodeur ou en utilisant des fichiers de configuration prédéfinis. Pour voir les types de présélections disponibles, consultez [Présélections de tâches pour Media Encoder Standard](./media-services-mes-presets-overview.md).

Chaque travail peut comporter une ou plusieurs tâches, en fonction du type de traitement que vous souhaitez accomplir. Via l’API REST, vous pouvez créer des travaux et les tâches associées de deux manières :

* Des tâches peuvent être définies inline via la propriété de navigation de tâches sur les entités de travail.
* Utilisez le traitement par lots OData.

Nous vous recommandons de toujours encoder vos fichiers source sous forme de jeu de fichiers MP4 à débit adaptatif, puis de convertir ce jeu au format souhaité au moyen de [l’empaquetage dynamique](media-services-dynamic-packaging-overview.md).

Si votre ressource de sortie est stockée sous forme chiffrée, vous devez configurer une stratégie de remise de ressources. Pour plus d'informations, consultez [Configuration de la stratégie de remise de ressources](media-services-rest-configure-asset-delivery-policy.md).

## <a name="considerations"></a>Considérations

Lors de l’accès aux entités dans Media Services, vous devez définir les valeurs et les champs d’en-tête spécifiques dans vos requêtes HTTP. Pour plus d'informations, consultez [Installation pour le développement REST API de Media Services](media-services-rest-how-to-use.md).

Avant de référencer les processeurs multimédias, vérifiez que vous disposez de l’ID de processeur multimédia approprié. Pour plus d’informations, consultez la rubrique [Obtenir des processeurs multimédias](media-services-rest-get-media-processor.md).

## <a name="connect-to-media-services"></a>Connexion à Media Services

Pour savoir comment vous connecter à l’API AMS, consultez [Accéder à l’API Azure Media Services avec l’authentification Azure AD](media-services-use-aad-auth-to-access-ams-api.md). 

## <a name="create-a-job-with-a-single-encoding-task"></a>Création d’un travail avec une seule tâche d’encodage

> [!NOTE]
> Lorsque vous utilisez l’API REST de Media Services, les considérations suivantes s’appliquent :
>
> Lors de l’accès aux entités dans Media Services, vous devez définir les valeurs et les champs d’en-tête spécifiques dans vos requêtes HTTP. Pour plus d’informations, consultez [Installation pour le développement REST API de Media Services](media-services-rest-how-to-use.md).
>
> Lors de l’utilisation de JSON et de la spécification pour utiliser le mot clé **__metadata** dans la requête (par exemple, pour faire référence à un objet lié) vous devez définir l’en-tête **Accept** au [format JSON détaillé](https://www.odata.org/documentation/odata-version-3-0/json-verbose-format/) : Accept: application/json;odata=verbose.
>
>

L’exemple suivant montre comment créer et publier un projet avec une tâche visant à encoder une vidéo en une résolution et une qualité spécifiques. Quand vous encodez à l’aide de Media Encoder Standard, vous pouvez utiliser les présélections de configuration de tâche spécifiées [ici](./media-services-mes-presets-overview.md).

Demande :

```console
POST https://media.windows.net/API/Jobs HTTP/1.1
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
    Authorization: Bearer <ENCODED JWT TOKEN> 
    x-ms-client-request-id: 00000000-0000-0000-0000-000000000000
    Host: media.windows.net

{"Name" : "NewTestJob", "InputMediaAssets" : [{"__metadata" : {"uri" : "https://media.windows.net/api/Assets('nb%3Acid%3AUUID%3Aaab7f15b-3136-4ddf-9962-e9ecb28fb9d2')"}}],  "Tasks" : [{"Configuration" : "Adaptive Streaming", "MediaProcessorId" : "nb:mpid:UUID:ff4df607-d419-42f0-bc17-a481b1331e56",  "TaskBody" : "<?xml version=\"1.0\" encoding=\"utf-8\"?><taskBody><inputAsset>JobInputAsset(0)</inputAsset><outputAsset>JobOutputAsset(0)</outputAsset></taskBody>"}]}
```

Réponse :

```console
HTTP/1.1 201 Created

. . .
```

### <a name="set-the-output-assets-name"></a>Définir le nom de l’élément multimédia de sortie
L’exemple suivant montre comment définir l’attribut assetName :

```console
{ "TaskBody" : "<?xml version=\"1.0\" encoding=\"utf-8\"?><taskBody><inputAsset>JobInputAsset(0)</inputAsset><outputAsset assetName=\"CustomOutputAssetName\">JobOutputAsset(0)</outputAsset></taskBody>"}`
```

### <a name="considerations"></a>Considérations
* les propriétés TaskBody doivent utiliser un XML littéral pour définir le nombre de ressources d’entrée ou de sortie qui sont utilisées par la tâche. L’article de tâche contient la définition du schéma XML pour le XML.
* Dans la définition TaskBody, chaque valeur interne de `<inputAsset>` et `<outputAsset>` doit être définie en tant que JobInputAsset(valeur) ou JobOutputAsset(valeur).
* Une tâche peut comporter plusieurs ressources de sortie. Un JobOutputAsset(x) ne peut être utilisé qu’une fois en tant que résultat d’une tâche dans un travail.
* Vous pouvez spécifier JobInputAsset ou JobOutputAsset en tant que ressource d’entrée d’une tâche.
* Les tâches ne doivent pas former un cycle.
* Le paramètre de valeur que vous transmettez à JobInputAsset ou à JobOutputAsset représente la valeur d’index pour une ressource. Les ressources réelles sont définies dans les propriétés de navigation InputMediaAssets et OutputMediaAssets de la définition d’entité de travail.
* Étant donné que Media Services est basé sur OData v3, les ressources dans les collections de propriétés de navigation InputMediaAssets et OutputMediaAssets sont référencées par une paire nom-valeur « __metadata: uri ».
* InputMediaAssets mappe vers une ou plusieurs ressources que vous avez créées dans Media Services. Les OutputMediaAssets sont créés par le système. Ils ne font pas référence à une ressource existante.
* OutputMediaAssets peut être nommé à l’aide de l’attribut assetName. Si cet attribut n’est pas présent, le nom d’OutputMediaAsset est la valeur de texte interne de l’élément `<outputAsset>` avec le suffixe de la valeur du nom du travail ou de l’ID de travail (dans le cas où la propriété Name n’est pas définie). Par exemple, si vous affectez à assetName la valeur « Sample », la propriété de Nom d’OutputMediaAsset est définie sur « Sample ». Toutefois, si vous n’avez pas défini de valeur pour assetName, mais avez défini le nom du travail comme « NewJob », le nom d’OutputMediaAsset est « JobOutputAsset(value)_NewJob ».

## <a name="create-a-job-with-chained-tasks"></a>Création d’un travail avec des tâches chaînées
Dans de nombreux scénarios d’application, les développeurs souhaitent créer une série de tâches de traitement. Dans Media Services, vous pouvez créer une série de tâches chaînées. Chaque tâche effectue différentes étapes de traitement et peut utiliser différents processeurs multimédias. Les tâches chaînées peuvent transférer un élément multimédia d’une tâche à une autre, en effectuant une séquence linéaire de tâches sur la ressource. Toutefois, les tâches effectuées dans un travail ne le sont pas obligatoirement dans une séquence. Quand vous créez une tâche chaînée, les objets chaînés **ITask** sont créés dans un seul objet **IJob**.

> [!NOTE]
> Il existe actuellement une limite de 30 tâches par travail. Si vous devez chaîner plus de 30 tâches, créez plusieurs travaux pour contenir les tâches.
>
>

```console
POST https://media.windows.net/api/Jobs HTTP/1.1
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN> 
x-ms-client-request-id: 00000000-0000-0000-0000-000000000000

{  
   "Name":"NewTestJob",
   "InputMediaAssets":[  
      {  
         "__metadata":{  
            "uri":"https://testrest.cloudapp.net/api/Assets('nb%3Acid%3AUUID%3A910ffdc1-2e25-4b17-8a42-61ffd4b8914c')"
         }
      }
   ],
   "Tasks":[  
      {  
         "Configuration":"H264 Adaptive Bitrate MP4 Set 720p",
         "MediaProcessorId":"nb:mpid:UUID:ff4df607-d419-42f0-bc17-a481b1331e56",
         "TaskBody":"<?xml version=\"1.0\" encoding=\"utf-8\"?><taskBody><inputAsset>JobInputAsset(0)</inputAsset><outputAsset>JobOutputAsset(0)</outputAsset></taskBody>"
      },
      {  
         "Configuration":"H264 Smooth Streaming 720p",
         "MediaProcessorId":"nb:mpid:UUID:ff4df607-d419-42f0-bc17-a481b1331e56",
         "TaskBody":"<?xml version=\"1.0\" encoding=\"utf-16\"?><taskBody><inputAsset>JobOutputAsset(0)</inputAsset><outputAsset>JobOutputAsset(1)</outputAsset></taskBody>"
      }
   ]
}
```

### <a name="considerations"></a>Considérations
Pour activer le chaînage des tâches :

* un travail doit comporter au moins deux tâches.
* Il doit y avoir au moins une tâche dont l’entrée correspond à la sortie d’une autre tâche du travail.

## <a name="use-odata-batch-processing"></a>Utiliser le traitement par lots OData
L’exemple suivant illustre l’utilisation du traitement par lots OData pour créer un travail et des tâches. Pour plus d’informations sur le traitement par lots, consultez [Traitement par lots d’Open Data Protocol (OData)](https://www.odata.org/documentation/odata-version-3-0/batch-processing/).

```console
POST https://media.windows.net/api/$batch HTTP/1.1
DataServiceVersion: 1.0;NetFx
MaxDataServiceVersion: 3.0;NetFx
Content-Type: multipart/mixed; boundary=batch_a01a5ec4-ba0f-4536-84b5-66c5a5a6d34e
Accept: multipart/mixed
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN> 
x-ms-version: 2.19
x-ms-client-request-id: 00000000-0000-0000-0000-000000000000
Host: media.windows.net


--batch_a01a5ec4-ba0f-4536-84b5-66c5a5a6d34e
Content-Type: multipart/mixed; boundary=changeset_122fb0a4-cd80-4958-820f-346309967e4d

--changeset_122fb0a4-cd80-4958-820f-346309967e4d
Content-Type: application/http
Content-Transfer-Encoding: binary

POST https://media.windows.net/api/Jobs HTTP/1.1
Content-ID: 1
Content-Type: application/json
Accept: application/json
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN> 
x-ms-version: 2.19
x-ms-client-request-id: 00000000-0000-0000-0000-000000000000

{"Name" : "NewTestJob", "InputMediaAssets@odata.bind":["https://media.windows.net/api/Assets('nb%3Acid%3AUUID%3A2a22445d-1500-80c6-4b34-f1e5190d33c6')"]}

--changeset_122fb0a4-cd80-4958-820f-346309967e4d
Content-Type: application/http
Content-Transfer-Encoding: binary

POST https://media.windows.net/api/$1/Tasks HTTP/1.1
Content-ID: 2
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
Accept-Charset: UTF-8
Authorization: Bearer <ENCODED JWT TOKEN> 
x-ms-version: 2.19
x-ms-client-request-id: 00000000-0000-0000-0000-000000000000

{  
   "Configuration":"H264 Adaptive Bitrate MP4 Set 720p",
   "MediaProcessorId":"nb:mpid:UUID:ff4df607-d419-42f0-bc17-a481b1331e56",
   "TaskBody":"<?xml version=\"1.0\" encoding=\"utf-8\"?><taskBody><inputAsset>JobInputAsset(0)</inputAsset><outputAsset assetName=\"Custom output name\">JobOutputAsset(0)</outputAsset></taskBody>"
}

--changeset_122fb0a4-cd80-4958-820f-346309967e4d--
--batch_a01a5ec4-ba0f-4536-84b5-66c5a5a6d34e--
```


## <a name="create-a-job-by-using-a-jobtemplate"></a>Création d’un travail à l’aide d’un JobTemplate
Pendant le traitement de plusieurs ressources à l’aide d’un jeu commun de tâches, utilisez un JobTemplate pour spécifier les présélections de tâches par défaut ou pour définir l’ordre des tâches.

L’exemple suivant montre comment créer un JobTemplate avec un TaskTemplate défini en ligne. Le TaskTemplate utilise Media Encoder Standard comme MediaProcessor pour encoder le fichier multimédia. Toutefois, d’autres MediaProcessors peuvent être également utilisés.

```console
POST https://media.windows.net/API/JobTemplates HTTP/1.1
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN> 
Host: media.windows.net


{"Name" : "NewJobTemplate25", "JobTemplateBody" : "<?xml version=\"1.0\" encoding=\"utf-8\"?><jobTemplate><taskBody taskTemplateId=\"nb:ttid:UUID:071370A3-E63E-4E81-A099-AD66BCAC3789\"><inputAsset>JobInputAsset(0)</inputAsset><outputAsset>JobOutputAsset(0)</outputAsset></taskBody></jobTemplate>", "TaskTemplates" : [{"Id" : "nb:ttid:UUID:071370A3-E63E-4E81-A099-AD66BCAC3789", "Configuration" : "H264 Smooth Streaming 720p", "MediaProcessorId" : "nb:mpid:UUID:ff4df607-d419-42f0-bc17-a481b1331e56", "Name" : "SampleTaskTemplate2", "NumberofInputAssets" : 1, "NumberofOutputAssets" : 1}] }
```

> [!NOTE]
> Contrairement à d’autres entités Media Services, vous devez définir un nouvel identificateur GUID pour chaque TaskTemplate et le placer dans la propriété taskTemplateId et Id du corps de votre demande. Le schéma d’identification de contenu doit respecter le schéma décrit dans la rubrique Identifier les entités Azure Media Services. En outre, les JobTemplates ne peuvent pas être mis à jour. À la place, vous devez en créer un avec vos modifications mises à jour.
>
>

Si l’opération réussit, la réponse suivante est retournée :

```console
HTTP/1.1 201 Created

. . .
```

L’exemple suivant montre comment créer un travail faisant référence à un ID de JobTemplate :

```console
POST https://media.windows.net/API/Jobs HTTP/1.1
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN> 
Host: media.windows.net


{"Name" : "NewTestJob", "InputMediaAssets" : [{"__metadata" : {"uri" : "https://media.windows.net/api/Assets('nb%3Acid%3AUUID%3A3f1fe4a2-68f5-4190-9557-cd45beccef92')"}}], "TemplateId" : "nb:jtid:UUID:15e6e5e6-ac85-084e-9dc2-db3645fbf0aa"}
```

Si l’opération réussit, la réponse suivante est retournée :

```console
HTTP/1.1 201 Created

. . .
```


## <a name="advanced-encoding-features-to-explore"></a>Fonctionnalités d’encodage avancées à découvrir
* [Procédure de génération de miniatures](media-services-dotnet-generate-thumbnail-with-mes.md)
* [Génération de miniatures pendant l’encodage](media-services-dotnet-generate-thumbnail-with-mes.md#example-of-generating-a-thumbnail-while-encoding)
* [Rognage de vidéos pendant l’encodage](media-services-crop-video.md)
* [Personnalisation des présélections d’encodage](media-services-custom-mes-presets-with-dotnet.md)
* [Superposition ou filigrane d’une vidéo avec une image](media-services-advanced-encoding-with-mes.md#overlay)

## <a name="media-services-learning-paths"></a>Parcours d’apprentissage de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="next-steps"></a>Étapes suivantes
Maintenant que vous savez comment créer un travail d'encodage de ressource, consultez [Vérification de la progression d’une tâche avec Media Services](media-services-rest-check-job-progress.md).

## <a name="see-also"></a>Voir aussi
[Obtenir des processeurs multimédias](media-services-rest-get-media-processor.md)