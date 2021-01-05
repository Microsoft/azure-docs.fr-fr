---
title: Vue d’ensemble de l’API REST Media Services Operations | Microsoft Docs
description: L’API REST Media Services Operations est utilisée pour créer des tâches, des actifs multimédias, des canaux live et d’autres ressources dans un compte Media Services. Cet article fournit une vue d’ensemble de l’API REST Azure Media Services v2.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.assetid: a5f1c5e7-ec52-4e26-9a44-d9ea699f68d9
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 03/20/2019
ms.author: juliako
ms.reviewer: johndeu
ms.openlocfilehash: 84e94a431efdc84ff6896de416bd222120784899
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89264281"
---
# <a name="media-services-operations-rest-api-overview"></a>Vue d’ensemble de l’API REST Media Services Operations

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> Aucune nouvelle fonctionnalité ni fonction n’est ajoutée à Media Services v2. <br/>Découvrez la dernière version, [Media Services v3](../latest/index.yml). Consultez aussi [Conseils de migration de v2 vers v3](../latest/migrate-from-v2-to-v3.md).

L’API **REST Media Services Operations** est utilisée pour créer des tâches, des éléments multimédias, des canaux directs et d’autres ressources dans un compte Media Services. Pour plus d’informations, consultez [Référence de l’API REST Media Services Operations](/rest/api/media/operations/azure-media-services-rest-api-reference).

Media Services fournit une API REST qui accepte les deux formats JSON et XML atom+pub. L’API REST Media Services nécessite des en-têtes HTTP spécifiques, que chaque client doit envoyer lors de la connexion à Media Services, ainsi qu’un ensemble d’en-têtes facultatifs. Les sections suivantes décrivent les en-têtes et verbes HTTP que vous pouvez utiliser lors de la création de requêtes et de la réception des réponses à partir de Media Services.

L’authentification à l’API REST Media Services est effectuée par le biais de l’authentification Azure Active Directory qui est décrite dans l’article [Utiliser l’authentification Azure AD pour accéder à l’API Azure Media Services avec REST](media-services-rest-connect-with-aad.md)

## <a name="considerations"></a>Considérations

Les considérations suivantes s'appliquent lors de l'utilisation de REST.

* Lors de l'interrogation des entités, il existe une limite de 1 000 entités retournées simultanément car l'API REST v2 publique limite les résultats des requêtes à 1 000 résultats. Vous devez utiliser **Skip** et **Take** (.NET)/ **top** (REST) comme décrit dans [cet exemple .NET](media-services-dotnet-manage-entities.md#enumerating-through-large-collections-of-entities) et cet exemple [d’API REST](media-services-rest-manage-entities.md#enumerating-through-large-collections-of-entities). 
* Lors de l’utilisation de JSON pour la spécification de l’utilisation du mot clé **__metadata** dans la requête (par exemple pour faire référence à un objet lié), vous DEVEZ définir l’en-tête **Accept** au [format JSON Verbose](https://www.odata.org/documentation/odata-version-3-0/json-verbose-format/) (consultez l’exemple suivant). OData ne comprend pas la propriété **metadata** dans la demande, sauf si vous la définissez sur « verbose ».  

    ```console
    POST https://media.windows.net/API/Jobs HTTP/1.1
    Content-Type: application/json;odata=verbose
    Accept: application/json;odata=verbose
    DataServiceVersion: 3.0
    MaxDataServiceVersion: 3.0
    x-ms-version: 2.19
    Authorization: Bearer <ENCODED JWT TOKEN> 
    Host: media.windows.net

    {
        "Name" : "NewTestJob", 
        "InputMediaAssets" : 
            [{"__metadata" : {"uri" : "https://media.windows.net/api/Assets('nb%3Acid%3AUUID%3Aba5356eb-30ff-4dc6-9e5a-41e4223540e7')"}}]
    . . . 
   ```

## <a name="standard-http-request-headers-supported-by-media-services"></a>En-têtes de requête HTTP standard pris en charge par Media Services
Pour chaque appel dans Media Services, il existe un ensemble d’en-têtes requis, que vous devez inclure dans votre requête, et un ensemble d’en-têtes facultatifs que vous pouvez également inclure. Le tableau ci-dessous répertorie les en-têtes nécessaires :

| En-tête | Type | Valeur |
| --- | --- | --- |
| Autorisation |Support |Le support est le seul mécanisme d’autorisation accepté. La valeur doit également inclure le jeton d’accès fourni par Azure Active Directory. |
| x-ms-version |Decimal |2.17 (ou version plus récente)|
| DataServiceVersion |Decimal |3.0 |
| MaxDataServiceVersion |Decimal |3.0 |

> [!NOTE]
> Étant donné que Media Services utilise OData pour exposer ses API REST, les en-têtes DataServiceVersion et MaxDataServiceVersion doivent être inclus dans toutes les requêtes. Toutefois, s’ils ne le sont pas, Media Services suppose que la valeur DataServiceVersion utilisée est 3.0.
> 
> 

Voici un ensemble d’en-têtes facultatifs :

| En-tête | Type | Valeur |
| --- | --- | --- |
| Date |Date RFC 1123 |Horodatage de la demande |
| Acceptation |Type de contenu |Type de contenu demandé pour la réponse, comme :<p> -application/json;odata=verbose<p> - application/atom+xml<p> Les réponses peuvent avoir un type de contenu différent, par exemple une extraction d’objets blob, dans laquelle une réponse correcte contient le flux d’objets blob en tant que charge utile. |
| Accept-Encoding |Gzip, deflate |Codage GZIP et DEFLATE, le cas échéant. Remarque : pour les ressources volumineuses, Media Services peut ignorer cet en-tête et retourner des données non compressées. |
| Accept-Language |« en », « es » et ainsi de suite. |Spécifie la langue préférée pour la réponse. |
| Accept-Charset |Type de jeu de caractères comme « UTF-8 » |La valeur par défaut est UTF-8. |
| X-HTTP-Method |Méthode HTTP |Permet aux clients ou pare-feu ne prenant pas en charge les méthodes HTTP comme PUT ou DELETE d’utiliser ces méthodes, acheminées via un appel GET. |
| Content-Type |Type de contenu |Le type de contenu du corps de la requête dans les demandes PUT ou POST. |
| client-request-id |String |Une valeur définie par l’appelant qui identifie la requête donnée. Si spécifiée, cette valeur sera incluse dans le message de réponse comme une méthode de mappage de la requête. <p><p>**Important**<p>Les valeurs doivent être limitées à 2096 b (2k). |

## <a name="standard-http-response-headers-supported-by-media-services"></a>En-têtes de réponse HTTP standard pris en charge par Media Services
Voici un ensemble d’en-têtes pouvant être renvoyés, en fonction de la ressource demandée et de l’action à entreprendre.

| En-tête | Type | Valeur |
| --- | --- | --- |
| request-id |String |Un identificateur unique pour l’opération actuelle, généré par le service. |
| client-request-id |String |Un identificateur spécifié par l’appelant dans la requête d’origine, le cas échéant. |
| Date |Date RFC 1123 |Les date et heure auxquelles la requête a été traitée. |
| Content-Type |Variable |Le type de contenu du corps de la réponse. |
| Content-Encoding |Variable |Gzip ou deflate, le cas échéant. |

## <a name="standard-http-verbs-supported-by-media-services"></a>Verbes HTTP standard pris en charge par Media Services
Voici une liste complète des verbes HTTP pouvant être utilisés lors de requêtes HTTP :

| Verbe | Description |
| --- | --- |
| GET |Retourne la valeur actuelle d’un objet. |
| POST |Crée un objet, selon les données fournies, ou envoie une commande. |
| PUT |Remplace un objet ou crée un objet nommé (le cas échéant). |
| Suppression |Supprime un objet. |
| MERGE |Met à jour un objet existant avec des modifications de propriété nommées. |
| HEAD |Retourne les métadonnées d’un objet d’une réponse GET. |

## <a name="discover-and-browse-the-media-services-entity-model"></a>Découvrir et parcourir le modèle d’entité Media Services
Pour rendre les entités Media Services plus détectables, l’opération $metadata peut être utilisée. Elle vous permet de récupérer l’ensemble des types d’entité, des propriétés d’entité, des associations, des fonctions, des actions valides, etc. En ajoutant l’opération $metadata à la fin du point de terminaison de votre API REST Media Services, vous pouvez accéder à ce service de découverte.

 /api/$metadata.

Vous devez ajouter « ?api-version=2.x » à la fin de l’URI si vous souhaitez afficher les métadonnées dans un navigateur ou n’incluez pas l’en-tête x-ms-version dans votre requête.

## <a name="authenticate-with-media-services-rest-using-azure-active-directory"></a>S’authentifier sur REST Media Services en utilisant Azure Active Directory

L’authentification à l’API REST s’effectue par le biais d’Azure Active Directory (AAD).
Pour plus d’informations sur les précisions à obtenir concernant l’authentification exigée de votre compte Media Services à partir du portail Azure, consultez [Accéder à l’API Azure Media Services avec l’authentification Azure AD](media-services-use-aad-auth-to-access-ams-api.md).

Pour obtenir des précisions sur l’écriture du code de connexion à l’API REST en utilisant l’authentification Azure AD, consultez l’article [Utiliser l’authentification Azure AD pour accéder à l’API Azure Media Services avec REST](media-services-rest-connect-with-aad.md).

## <a name="next-steps"></a>Étapes suivantes
Pour savoir comment utiliser l’authentification Azure AD avec l’API REST Media Services, consultez [Utiliser l’authentification Azure AD pour accéder à l’API Azure Media Services avec REST](media-services-rest-connect-with-aad.md).

## <a name="media-services-learning-paths"></a>Parcours d’apprentissage de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
