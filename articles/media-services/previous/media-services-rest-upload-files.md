---
title: Charger des fichiers dans un compte Azure Media Services à l’aide de REST | Microsoft Docs
description: Apprenez à obtenir du contenu multimédia dans Media Services en créant et en chargeant des ressources à l’aide de REST.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/20/2019
ms.author: juliako
ms.openlocfilehash: 49863bec4cbd367b6b309ef5a79e7287cb53ee5b
ms.sourcegitcommit: 4f4a2b16ff3a76e5d39e3fcf295bca19cff43540
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93042977"
---
# <a name="upload-files-into-a-media-services-account-using-rest"></a>Charger des fichiers dans un compte Media Services à l’aide de REST

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!div class="op_single_selector"]
> * [.NET](media-services-dotnet-upload-files.md)
> * [REST](media-services-rest-upload-files.md)
> * [Portail](media-services-portal-upload-files.md)
> 

Dans Media Services, vous téléchargez vos fichiers numériques dans une ressource. L’entité [Asset](/rest/api/media/operations/asset) peut contenir des fichiers vidéo et audio, des images, des collections de miniatures, des pistes textuelles et des légendes (et les métadonnées concernant ces fichiers).  Une fois les fichiers téléchargés dans la ressource, votre contenu est stocké en toute sécurité dans le cloud et peut faire l’objet d’un traitement et d’une diffusion en continu. 

Dans ce didacticiel, vous allez apprendre à charger un fichier et une autre opération associée :

> [!div class="checklist"]
> * Configurer Postman pour toutes les opérations de chargement
> * Connexion à Media Services 
> * Créer une stratégie d’accès avec autorisation d’écriture
> * Créer une ressource
> * Créer un localisateur SAS et créer l’URL de chargement
> * Charger un fichier vers le stockage d’objets blob à l’aide de l’URL de chargement
> * Créer des métadonnées dans l’élément multimédia pour le fichier multimédia que vous avez chargé

## <a name="prerequisites"></a>Prérequis

- Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) avant de commencer.
- [Créez un compte Azure Media Services avec le portail Azure](media-services-portal-create-account.md).
- Consultez l’article [Accéder à l’API Azure Media Services avec l’authentification Azure AD](media-services-use-aad-auth-to-access-ams-api.md).
- Pour plus d’informations, consultez également l’article [Utiliser l’authentification Azure AD pour accéder à l’API Azure Media Services avec .NET](./media-services-rest-connect-with-aad.md).
- Configurez **Postman** tel que décrit dans [Configurer Postman pour les appels d’API REST Media Services](media-rest-apis-with-postman.md).

## <a name="considerations"></a>Considérations

Les considérations suivantes s’appliquent lors de l’utilisation de l’API REST Media Services :
 
* Lors de l’accès aux entités à l’aide de l’API REST Media Services, vous devez définir les valeurs et les champs d’en-tête spécifiques dans vos requêtes HTTP. Pour plus d'informations, consultez [Installation pour le développement REST API de Media Services](media-services-rest-how-to-use.md). <br/>La collection Postman utilisée dans ce didacticiel se charge de la définition de tous les en-têtes nécessaires.
* Media Services utilise la valeur de la propriété IAssetFile.Name lors de la génération d’URL pour le contenu de streaming (par exemple, http://{AMSAccount}.origin.mediaservices.windows.net/{GUID}/{IAssetFile.Name}/streamingParameters). Pour cette raison, l’encodage par pourcentage n’est pas autorisé. La valeur de la propriété **Name** ne peut pas comporter les [caractères réservés à l’encodage en pourcentage suivants](https://en.wikipedia.org/wiki/Percent-encoding#Percent-encoding_reserved_characters) : !* '();:@&=+$,/?%#[]". En outre, il ne peut exister qu’un ’.’ pour l’extension de nom de fichier.
* La longueur du nom ne doit pas dépasser 260 caractères.
* Une limite est appliquée à la taille maximale de fichier prise en charge pour le traitement dans Media Services. Consultez [cet](media-services-quotas-and-limitations.md) article pour en savoir plus sur les limites de taille des fichiers.

## <a name="set-up-postman"></a>Configurer Postman

Pour savoir comment configurer Postman pour ce didacticiel, consultez [Configurer Postman](media-rest-apis-with-postman.md).

## <a name="connect-to-media-services"></a>Connexion à Media Services

1. Ajoutez des valeurs de connexion à votre environnement. 

    Certaines variables qui font partie de l’ [environnement](postman-environment.md) **MediaServices** doivent être définies manuellement avant de pouvoir commencer l’exécution d’opérations définies dans la [collection](postman-collection.md).

    Pour obtenir des valeurs pour les cinq premières variables, consultez [Accéder à l’API Azure Media Services avec l’authentification Azure AD](media-services-use-aad-auth-to-access-ams-api.md). 

    ![Capture d’écran montrant l’icône « Roue dentée » sélectionnée en haut à droite, et les cinq premières variables sélectionnées à partir de l’onglet « Environnements de gestion ».](./media/media-services-rest-upload-files/postman-import-env.png)
2. Spécifiez la valeur de la variable d’environnement **MediaFileName**.

    Spécifiez le nom du fichier multimédia que vous envisagez de charger. Dans cet exemple, vous allez charger le fichier BigBuckBunny.mp4. 
3. Examinez le fichier **AzureMediaServices.postman_environment.json**. Vous voyez que presque toutes les opérations dans la collection exécutent un script de « test ». Les scripts prennent certaines valeurs retournées par la réponse et définissent les variables d’environnement appropriées.

    Par exemple, la première opération obtient un jeton d’accès et le définit sur la variable d’environnement **AccessToken** qui est utilisée dans toutes les autres opérations.

    ```    
    "listen": "test",
    "script": {
        "type": "text/javascript",
        "exec": [
            "var json = JSON.parse(responseBody);",
            "postman.setEnvironmentVariable(\"AccessToken\", json.access_token);"
        ]
    }
    ```
4. À gauche de la fenêtre **Postman** , cliquez sur **1. Obtenir un jeton d’authentification AAD** -> **Obtenir un jeton Azure AD pour le principal du service**.

    La partie de l’URL est remplie avec la variable d’environnement **AzureADSTSEndpoint** (plus haut dans ce tutoriel, vous avez défini les valeurs des variables d’environnement prenant en charge la collection).

    ![Capture d’écran montrant la sélection de « 1. Obtenir un jeton d’authentification AAD - Obtenir un jeton Azure AD pour le principal du service » dans la fenêtre « Postman » et la sélection du bouton « Envoyer ».](./media/media-services-rest-upload-files/postment-get-token.png)

5. Appuyez sur **Envoyer**.

    Vous pouvez voir la réponse qui contient « access_token ». Le script de « test » prend cette valeur et définit la variable d’environnement **AccessToken** (comme décrit ci-dessus). Si vous examinez vos variables d’environnement, vous voyez que cette variable contient à présent la valeur de jeton d’accès (jeton du porteur) qui est utilisée dans le reste des opérations. 

    Si le jeton expire, effectuez à nouveau l’étape « Obtenir un jeton Azure AD pour le principal du service ». 

## <a name="create-an-access-policy-with-write-permission"></a>Créer une stratégie d’accès avec autorisation d’écriture

### <a name="overview"></a>Vue d’ensemble 

>[!NOTE]
>Un nombre limite de 1 000 000 a été défini pour les différentes stratégies AMS (par exemple, pour la stratégie de localisateur ou pour ContentKeyAuthorizationPolicy). Vous devez utiliser le même ID de stratégie si vous utilisez toujours les mêmes jours / autorisations d’accès, par exemple, les stratégies pour les localisateurs destinées à demeurer en place pendant une longue période (stratégies sans chargement). Pour plus d’informations, consultez [cet](media-services-dotnet-manage-entities.md#limit-access-policies) article.

Avant de télécharger des fichiers dans le stockage blob, définissez les droits de la stratégie d’accès pour l’écriture sur une ressource. Pour ce faire, utilisez POST avec une demande HTTP sur le jeu d’entités AccessPolicies. N’oubliez pas de définir une valeur DurationInMinutes après la création ou vous recevrez en réponse un message d’erreur interne de serveur 500. Pour plus d’informations sur AccessPolicies, consultez [AccessPolicy](/rest/api/media/operations/accesspolicy).

### <a name="create-an-access-policy"></a>Définition d’une stratégie d’accès.

1. Sélectionnez **AccessPolicy** -> **Créer la stratégie d’accès pour le chargement**.
2. Appuyez sur **Envoyer**.

    ![Capture d’écran montrant la sélection de « AccessPolicy - Créer la stratégie d’accès pour le chargement » dans le menu de gauche, et la sélection du bouton « Envoyer ».](./media/media-services-rest-upload-files/postman-access-policy.png)

    Le script de « test » obtient l’ID de stratégie d’accès et définit la variable d’environnement appropriée.

## <a name="create-an-asset"></a>Créer une ressource

### <a name="overview"></a>Vue d’ensemble

Une [ressource](/rest/api/media/operations/asset) est un conteneur pour plusieurs types ou ensembles d’objets dans Media Services, y compris des fichiers vidéo, audio, des images, des collections de miniatures, des pistes textuelles et des sous-titres. Dans l’API REST, la création d’une ressource nécessite d’envoyer une demande POST vers Media Services et de placer les informations de propriété concernant votre ressource dans le corps de la demande.

L’une des propriétés que vous pouvez ajouter lors de la création d’un élément multimédia est **Options**. Vous pouvez spécifier l’une des options de chiffrement suivantes : **Aucun** (valeur par défaut, aucun chiffrement n’est utilisé), **StorageEncrypted** (pour le contenu qui a été déjà chiffré avec le chiffrement du stockage côté client), **CommonEncryptionProtected** ou **EnvelopeEncryptionProtected**. Lorsque vous disposez d’un élément multimédia chiffré, vous devez configurer une stratégie de remise. Pour plus d'informations, consultez [Configuration des stratégies de distribution de ressources](media-services-rest-configure-asset-delivery-policy.md).

Si votre élément multimédia est chiffré, vous devez créer une **ContentKey** et la lier à votre élément multimédia, comme le décrit l’article : [Création de clés de contenu avec REST](media-services-rest-create-contentkey.md). Après avoir chargé les fichiers dans l’élément multimédia, vous devez mettre à jour les propriétés de chiffrement sur l’entité **AssetFile** avec les valeurs obtenues pendant le chiffrement **Asset**. Pour ce faire, utilisez la demande HTTP **MERGE** . 

Dans cet exemple, vous allez créer un élément multimédia déchiffré. 

### <a name="create-an-asset"></a>Créer une ressource

1. Sélectionnez **Éléments multimédias** -> **Créer un élément multimédia**.
2. Appuyez sur **Envoyer**.

    ![Capture d’écran montrant la sélection de « Éléments multimédias - Créer un élément multimédia » dans le menu « Collections », et le bouton « Envoyer » sélectionné.](./media/media-services-rest-upload-files/postman-create-asset.png)

    Le script de « test » obtient l’ID d’élément multimédia et définit la variable d’environnement appropriée.

## <a name="create-a-sas-locator-and-create-the-upload-url"></a>Créer un localisateur SAS et créer l’URL de chargement

### <a name="overview"></a>Vue d’ensemble

Après avoir défini AccessPolicy et Locator, le fichier réel est téléchargé vers un conteneur de stockage d’objets blob Microsoft Azure à l’aide des API REST Azure Storage. Vous devez télécharger les fichiers en tant qu’objets blob de blocs. Les objets blob de pages ne sont pas pris en charge par Azure Media Services.  

Pour plus d’informations sur l’utilisation d’objets blob de stockage Microsoft Azure, consultez [API REST du service BLOB](/rest/api/storageservices/blob-service-rest-api).

Pour recevoir l’URL de chargement réelle, créez un localisateur SAS (voir ci-dessous). Les localisateurs définissent l’heure de début et le type de point de terminaison de connexion pour les clients qui souhaitent accéder aux fichiers d’une ressource. Vous pouvez créer plusieurs entités de localisateurs pour une paire AccessPolicy et Asset donnée, afin de gérer les différentes demandes et besoins des clients. Chacun de ces localisateurs utilise la valeur StartTime et la valeur DurationInMinutes d’AccessPolicy pour déterminer la durée pendant laquelle une URL peut être utilisée. Pour plus d’informations, consultez la rubrique [Localisateur](/rest/api/media/operations/locator).

Une URL SAS a le format suivant :

`{https://myaccount.blob.core.windows.net}/{asset name}/{video file name}?{SAS signature}`

### <a name="considerations"></a>Considérations

Certaines considérations s’appliquent :

* Vous ne pouvez avoir plus de cinq localisateurs uniques associés à une ressource donnée. Pour plus d’informations, consultez la rubrique Localisateur.
* Si vous avez besoin de télécharger vos fichiers immédiatement, vous devez définir la valeur StartTime sur cinq minutes avant l’heure actuelle. Cela vient du fait qu’il peut exister un décalage horaire entre votre ordinateur client et Media Services. Votre valeur StartTime doit être au format DateHeure suivant : AAAA-MM-JJTHH:mm:ssZ (par exemple, « 2014-05-23T17:53:50Z »).    
* Il peut y avoir un délai de 30 à 40 secondes après la création d’un localisateur avant qu’il soit disponible.

### <a name="create-a-sas-locator"></a>Créer un localisateur SAS

1. Sélectionnez **Localisateur** -> **Créer un localisateur SAS**.
2. Appuyez sur **Envoyer**.

    Le script de « test » crée l’URL de chargement en fonction du nom de fichier spécifié et définit la variable d’environnement appropriée.

    ![Capture d’écran montrant la sélection de « Localisateur - Créer un localisateur SAS » dans le menu « Collections », et le bouton « Envoyer » sélectionné.](./media/media-services-rest-upload-files/postman-create-sas-locator.png)

## <a name="upload-a-file-to-blob-storage-using-the-upload-url"></a>Charger un fichier vers le stockage d’objets blob à l’aide de l’URL de chargement

### <a name="overview"></a>Vue d’ensemble

Maintenant que vous avez l’URL de chargement, vous devez écrire du code avec les API d’objet blob Azure directement pour charger votre fichier dans le conteneur SAS. Pour plus d’informations, consultez les articles suivants :

- [Utilisation de l’API REST Stockage Azure](../../storage/common/storage-rest-api-auth.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)
- [PUT Blob](/rest/api/storageservices/put-blob)
- [Chargement d’objets blob vers un stockage d’objets blob](/previous-versions/azure/storage/storage-use-azcopy#upload-blobs-to-blob-storage)

### <a name="upload-a-file-with-postman"></a>Charger un fichier avec Postman

Par exemple, nous utilisons Postman pour charger un petit fichier .mp4. Il peut y avoir une limite de taille de fichier lors du chargement binaire via Postman.

La demande de chargement ne fait pas partie de la collection **AzureMedia**. 

Créer et configurer une nouvelle demande :
1. Appuyez sur **+** pour créer un onglet de demande.
2. Sélectionnez l’opération **PUT** et collez **{{UploadURL}}** dans l’URL.
2. Ne modifiez pas l’onglet **Autorisation** (ne le définissez pas sur **Jetons du porteur** ).
3. Dans l’onglet **En-têtes** , spécifiez : **Clé**  : « x-ms-blob-type » et **Valeur**  : « BlockBlob ».
2. Sous l’onglet **Corps** , cliquez sur **Binaire**.
4. Sélectionnez le fichier portant le nom que vous avez spécifié dans la variable d’environnement **MediaFileName**.
5. Appuyez sur **Envoyer**.

    ![Capture d’écran montrant l’onglet « (UploadURL) » sélectionné.](./media/media-services-rest-upload-files/postman-upload-file.png)

##  <a name="create-a-metadata-in-the-asset"></a>Créer des métadonnées dans l’élément multimédia

Une fois que le fichier a été chargé, vous devez créer des métadonnées dans l’élément multimédia du fichier multimédia que vous avez chargé dans le stockage d’objets blob associé à votre élément multimédia.

1. Sélectionnez **AssetFiles** -> **CreateFileInfos**.
2. Appuyez sur **Envoyer**.

    ![Charger un fichier](./media/media-services-rest-upload-files/postman-create-file-info.png)

Le fichier doit être chargé et ses métadonnées définies.

## <a name="validate"></a>Valider

Pour valider que le fichier a été chargé avec succès, vous pouvez interroger [AssetFile](/rest/api/media/operations/assetfile) et comparer **ContentFileSize** (ou d’autres détails) avec ce que vous attendez dans le nouvel élément multimédia. 

Par exemple, l’opération **GET** suivante apporte des données de fichier pour votre fichier multimédia (dans le cas présent, le fichier BigBuckBunny.mp4). La requête utilise les [variables d’environnement](postman-environment.md) définies précédemment.

`{{RESTAPIEndpoint}}/Assets('{{LastAssetId}}')/Files`

La réponse contient la taille, le nom et d’autres informations.

```console
"Id": "nb:cid:UUID:69e72ede-2886-4f2a-8d36-80a59da09913",
"Name": "BigBuckBunny.mp4",
"ContentFileSize": "3186542",
"ParentAssetId": "nb:cid:UUID:0b8f3b04-72fb-4f38-8e7b-d7dd78888938",
```
  
## <a name="next-steps"></a>Étapes suivantes

Vous pouvez désormais encoder vos éléments multimédias téléchargés. Pour plus d'informations, consultez [Encode an asset using Media Encoder Standard with the Azure portal (Encoder un élément multimédia à l’aide de Media Encoder Standard avec le portail Azure)](media-services-portal-encode.md).

Vous pouvez également utiliser les fonctions Azure pour déclencher une tâche de codage à partir d’un fichier entrant dans le conteneur configuré. Pour plus d’informations, consultez [cet exemple](https://azure.microsoft.com/resources/samples/media-services-dotnet-functions-integration/ ).
