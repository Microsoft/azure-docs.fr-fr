---
title: Gestion des entités Media Services avec REST | Microsoft Docs
description: Cet article explique comment gérer des entités Media Services avec l’API REST.
author: juliako
manager: femila
editor: ''
services: media-services
documentationcenter: ''
ms.assetid: 95262a32-0f2a-4286-b9e2-1a1ca6399b5b
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/20/2019
ms.author: juliako
ms.openlocfilehash: 95124f101b1f14a70dabedd7d44077c5c1b6e99c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89264230"
---
# <a name="managing-media-services-entities-with-rest"></a>Gestion des entités Media Services avec REST

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!div class="op_single_selector"]
> * [REST](media-services-rest-manage-entities.md)
> * [.NET](media-services-dotnet-manage-entities.md)
> 
> 

Microsoft Azure Media Services est un service basé sur REST conçu autour d’OData v3. Vous pouvez ajouter, interroger, mettre à jour et supprimer des entités de la même façon que vous pouvez le faire dans tout autre service OData. Toute exception sera indiquée le cas échéant. Pour plus d’informations sur OData, consultez la [documentation Open Data Protocol](https://www.odata.org/documentation/).

Cette rubrique vous montre comment gérer les entités Azure Media Services avec REST.

>[!NOTE]
> À compter du 1er avril 2017, les enregistrements de travaux dans votre compte de plus de 90 jours seront automatiquement supprimés, ainsi que leurs enregistrements de tâches associés, même si le nombre total d’enregistrements est inférieur au quota maximum. Par exemple, le 1er avril 2017, tout enregistrement de travail dans votre compte antérieur au 31 décembre 2016 sera automatiquement supprimé. Si vous devez archiver les informations sur le travail/la tâche, vous pouvez utiliser le code décrit dans cette rubrique.

## <a name="considerations"></a>Considérations  

Lors de l’accès aux entités dans Media Services, vous devez définir les valeurs et les champs d’en-tête spécifiques dans vos requêtes HTTP. Pour plus d'informations, consultez [Installation pour le développement REST API de Media Services](media-services-rest-how-to-use.md).

## <a name="connect-to-media-services"></a>Connexion à Media Services

Pour savoir comment vous connecter à l’API AMS, consultez [Accéder à l’API Azure Media Services avec l’authentification Azure AD](media-services-use-aad-auth-to-access-ams-api.md). 

## <a name="adding-entities"></a>Ajout d’entités
Chaque entité de Media Services est ajoutée à un jeu d'entités, tel que Assets, via une demande HTTP POST.

L’exemple suivant montre comment créer une stratégie AccessPolicy :

```console
POST https://media.windows.net/API/AccessPolicies HTTP/1.1
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN> 
Host: media.windows.net
Content-Length: 74
Expect: 100-continue

{"Name": "DownloadPolicy", "DurationInMinutes" : "300", "Permissions" : 1}
```

## <a name="querying-entities"></a>Exécution d’une requête sur les entités
La procédure d’interrogation et d’énumération des entités est simple et ne nécessite qu’une requête HTTP GET et les opérations OData facultatives.
L’exemple suivant montre la récupération d’une liste de toutes les entités MediaProcessor.

```console
GET https://media.windows.net/API/MediaProcessors HTTP/1.1
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN> 
Host: media.windows.net
```

Vous pouvez également récupérer une entité spécifique ou tous les jeux d'entités associés à une entité spécifique, comme dans les exemples suivants :

```console
GET https://media.windows.net/API/JobTemplates('nb:jtid:UUID:e81192f5-576f-b247-b781-70a790c20e7c') HTTP/1.1
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN> 
Host: media.windows.net

GET https://media.windows.net/API/JobTemplates('nb:jtid:UUID:e81192f5-576f-b247-b781-70a790c20e7c')/TaskTemplates HTTP/1.1
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN> 
Host: media.windows.net
```

L’exemple suivant renvoie uniquement la propriété State de toutes les tâches.

```console
GET https://media.windows.net/API/Jobs?$select=State HTTP/1.1
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN> 
Host: media.windows.net
```

L'exemple suivant retourne l'ensemble des JobTemplates avec le nom « SampleTemplate ».

```console
GET https://media.windows.net/API/JobTemplates?$filter=startswith(Name,%20'SampleTemplate') HTTP/1.1
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN> 
Host: media.windows.net
```

> [!NOTE]
> L’opération $expand n’est pas prise en charge dans Media Services, tout comme les méthodes LINQ non prises en charge décrites dans les considérations sur LINQ (WCF Data Services).
> 
> 

## <a name="enumerating-through-large-collections-of-entities"></a>Énumérer les grandes collections d'entités
Lors de l'interrogation des entités, il existe une limite de 1 000 entités retournées simultanément car l'API REST v2 publique limite les résultats des requêtes à 1 000 résultats. Utilisez **skip** et **top** pour énumérer les grandes collections d’entités. 

L’exemple suivant montre comment utiliser **skip** et **top** pour ignorer les 2 000 premiers emplois et obtenir les 1 000 emplois suivants.  

```console
GET https://media.windows.net/api/Jobs()?$skip=2000&$top=1000 HTTP/1.1
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN>
Host: media.windows.net
```

## <a name="updating-entities"></a>Mise à jour des entités
Selon le type d’entité et l’état dans lequel elle se trouve, vous pouvez mettre à jour les propriétés de cette entité via une requête HTTP PATCH, PUT ou MERGE. Pour plus d’informations sur ces opérations, consultez [PATCH/PUT/MERGE](/openspecs/windows_protocols/ms-odata/59d5abd3-7b12-490a-a0e2-9d9324b91893).

L’exemple de code suivant montre comment mettre à jour la propriété Name sur une entité Asset.

```console
MERGE https://media.windows.net/API/Assets('nb:cid:UUID:80782407-3f87-4e60-a43e-5e4454232f60') HTTP/1.1
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN>
Host: media.windows.net
Content-Length: 21
Expect: 100-continue

{"Name" : "NewName" }
```

## <a name="deleting-entities"></a>Suppression des entités
Les entités peuvent être supprimées dans Media Services à l’aide d’une requête HTTP DELETE. En fonction de l’entité, l’ordre dans lequel vous supprimez les entités peut être important. Par exemple, les entités telles que Assets nécessitent que vous annuliez (ou supprimiez) tous les localisateurs qui font référence à cet Asset particulier avant de supprimer l’Asset.

L’exemple suivant montre comment supprimer un localisateur qui était utilisé pour charger un fichier dans le stockage d’objets blob.

```console
DELETE https://media.windows.net/API/Locators('nb:lid:UUID:76dcc8e8-4230-463d-97b0-ce25c41b5c8d') HTTP/1.1
Content-Type: application/json;odata=verbose
Accept: application/json;odata=verbose
DataServiceVersion: 3.0
MaxDataServiceVersion: 3.0
x-ms-version: 2.19
Authorization: Bearer <ENCODED JWT TOKEN>
Host: media.windows.net
Content-Length: 0
```

## <a name="media-services-learning-paths"></a>Parcours d’apprentissage de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
