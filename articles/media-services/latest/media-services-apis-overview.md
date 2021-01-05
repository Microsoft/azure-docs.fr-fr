---
title: Développer avec les API v3
titleSuffix: Azure Media Services
description: En savoir plus sur les règles qui s’appliquent aux entités et API lors du développement avec Media Services v3.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: conceptual
ms.date: 10/23/2020
ms.author: inhenkel
ms.custom: seodec18
ms.openlocfilehash: 416fb9fc4ce0622a710f2c119942edc4986ddd06
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92790574"
---
# <a name="develop-with-media-services-v3-apis"></a>Développer avec les API Media Services v3

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

En tant que développeur, vous pouvez utiliser [l’API REST](/rest/api/media/) ou les bibliothèques clientes de Media Services qui vous permettent d’interagir avec l’API REST afin de créer, gérer et mettre à jour facilement les workflows multimédias personnalisés. L’API [Media Services v3](https://aka.ms/ams-v3-rest-sdk) s’appuie sur la spécification OpenAPI (anciennement appelée Swagger).

Cet article décrit les règles qui s’appliquent aux entités et API lors du développement avec Media Services v3.

## <a name="accessing-the-azure-media-services-api"></a>Accéder à l'API Azure Media Services

Pour être autorisé à accéder aux ressources Media Services et à l’API Media Services, vous devez tout d’abord être authentifié. Media Services prend en charge l’[authentification avec Azure Active Directory (Azure AD)](../../active-directory/fundamentals/active-directory-whatis.md). Parmi les options d’authentification courantes figurent les suivantes :
 
* **Authentification d’un principal de service** : Utilisée pour authentifier un service (applications web, applications de fonction, applications logiques, API et microservices, par exemple). Les applications qui utilisent généralement cette méthode d’authentification sont des applications qui exécutent des services démon, des services de niveau intermédiaire ou des travaux planifiés, Par exemple, pour les applications web, un niveau intermédiaire devrait toujours se connecter à Media Services avec un principal de service.
* **Authentification utilisateur** : Utilisée pour authentifier une personne qui utilise l’application pour interagir avec les ressources Media Services. L’application interactive invite tout d’abord l’utilisateur à entrer ses informations d’identification. Par exemple, une application de console de gestion peut être utilisée par les utilisateurs autorisés pour contrôler les travaux d’encodage ou de streaming en direct.

L’API Media Services implique que l’utilisateur ou l'application à l'origine des requêtes API REST ait accès à la ressource de compte Media Services et utilise un rôle **Contributeur** ou **Propriétaire**. L’API est accessible avec le rôle **Lecteur** , mais seules les opérations **Get** ou **List** sont disponibles. Pour en savoir plus, consultez [Contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour les comptes Media Services](rbac-overview.md).

Au lieu de créer un principal de service, envisagez d’utiliser des identités gérées pour permettre aux ressources Azure d'accéder à l’API Media Services via Azure Resource Manager. Pour en savoir plus sur les identités managées pour les ressources Azure, consultez [Que sont les identités managées pour les ressources Azure ?](../../active-directory/managed-identities-azure-resources/overview.md)

### <a name="azure-ad-service-principal"></a>Principal du service Azure AD

L’application Azure AD et le principal du service doivent se trouver dans le même locataire. Après avoir créé l’application, attribuez à l'application le rôle **Contributeur** ou **Propriétaire** pour accéder au compte Media Services.

Si vous ne savez pas si vous disposez des autorisations pour créer une application Azure AD, consultez [Autorisations requises](../../active-directory/develop/howto-create-service-principal-portal.md#permissions-required-for-registering-an-app).

Dans la figure suivante, les nombres représentent le flux des requêtes dans l’ordre chronologique :

![Authentification d’application de niveau intermédiaire avec AAD à partir d’une API web](./media/use-aad-auth-to-access-ams-api/media-services-principal-service-aad-app1.png)

1. Une application de niveau intermédiaire nécessite un jeton d’accès Azure AD qui possède les paramètres suivants :  

   * Point de terminaison de locataire Azure AD.
   * URI de ressource Media Services.
   * URI de ressource pour REST Media Services.
   * Valeurs de l’application Azure AD : ID client et secret client.

   Pour obtenir toutes les valeurs nécessaires, consultez [Accéder à l’API Azure Media Services](./access-api-howto.md).

2. Le jeton d’accès Azure AD est envoyé au niveau intermédiaire.
4. Le niveau intermédiaire envoie une requête à l’API REST Azure Media avec le jeton Azure AD.
5. Le niveau intermédiaire récupère les données de Media Services.

### <a name="samples"></a>Exemples

Les exemples suivants montrent comment se connecter à un principal de service Azure AD :

* [Se connecter avec REST](media-rest-apis-with-postman.md)  
* [Se connecter avec Java](configure-connect-java-howto.md)
* [Se connecter avec .NET](configure-connect-dotnet-howto.md)
* [Se connecter avec Node.js](configure-connect-nodejs-howto.md)
* [Se connecter avec Python](configure-connect-python-howto.md)

## <a name="naming-conventions"></a>Conventions d’affectation de noms

Les noms de ressources Azure Media Services v3 (par exemple Assets, Jobs, Transforms) sont sujets à des restrictions d’appellation par Azure Resource Manager. Conformément à Azure Resource Manager, les noms des ressources sont toujours uniques. Ainsi, vous pouvez utiliser n’importe quelle chaîne d’identificateur unique (par exemple des GUID) pour les noms de vos ressources.

Les noms de ressources Media Services ne peuvent pas contenir : '<', '>', '%', '&', ':', '&#92;', '?', '/', '*', '+', '.', le caractère de citation unique ou tout caractère de commande. Tous les autres caractères sont autorisés. La longueur maximale d’un nom de ressources est de 260 caractères.

Pour en savoir plus sur l’affectation de noms avec Azure Resource Manager, consultez : [Exigences d’affectation des noms](https://github.com/Azure/azure-resource-manager-rpc/blob/master/v1.0/resource-api-reference.md#arguments-for-crud-on-resource) et [Convention d’affectation de noms](/azure/cloud-adoption-framework/ready/azure-best-practices/naming-and-tagging).

### <a name="names-of-filesblobs-within-an-asset"></a>Noms des fichiers/objets blob dans une ressource

Les noms des fichiers/objets blob au sein d’une ressource doivent respecter les [exigences en matière de nom d’objet blob](/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata) et de [nom NTFS](/windows/win32/fileio/naming-a-file). Ces exigences se justifient par le fait que les fichiers peuvent être copiés du stockage d’objets blob vers un disque NTFS local à des fins de traitement.

## <a name="long-running-operations"></a>Opérations de longue durée

Les opérations marquées par `x-ms-long-running-operation` dans les [fichiers swagger](https://github.com/Azure/azure-rest-api-specs/blob/master/specification/mediaservices/resource-manager/Microsoft.Media/stable/2018-07-01/streamingservice.json) Azure Media Services exécutent des opérations de longue durée. 

Pour plus d’informations sur le suivi des opérations asynchrones Azure, consultez [Opérations asynchrones](../../azure-resource-manager/management/async-operations.md).

Media Services propose les opérations de longue durée suivantes :

* [Créer des événements en direct](/rest/api/media/liveevents/create)
* [Mettre à jour des événements en direct](/rest/api/media/liveevents/update)
* [Supprimer l’événement en direct](/rest/api/media/liveevents/delete)
* [Démarrer l’événement en direct](/rest/api/media/liveevents/start)
* [Arrêter l’événement en direct](/rest/api/media/liveevents/stop)

  Utilisez le paramètre `removeOutputsOnStop` pour supprimer toutes les sorties en temps réel associées lors de l’arrêt de l’événement.  
* [Réinitialiser l’événement en direct](/rest/api/media/liveevents/reset)
* [Créer une sortie en temps réel](/rest/api/media/liveevents/create)
* [Supprimer la sortie en temps réel](/rest/api/media/liveevents/delete)
* [Créer un StreamingEndpoint](/rest/api/media/streamingendpoints/create)
* [Mettre à jour un StreamingEndpoint](/rest/api/media/streamingendpoints/update)
* [Supprimer un StreamingEndpoint](/rest/api/media/streamingendpoints/delete)
* [Démarrer un StreamingEndpoint](/rest/api/media/streamingendpoints/start)
* [Arrêter un StreamingEndpoint](/rest/api/media/streamingendpoints/stop)
* [Mettre à l’échelle un StreamingEndpoint](/rest/api/media/streamingendpoints/scale)

En cas de soumission réussie d’une longue opération, vous recevez un message « 201 créé » et devez interroger la fin de l’opération à l’aide de l’ID d’opération retourné.

L’article [Suivre les opérations asynchrones Azure](../../azure-resource-manager/management/async-operations.md) explique en détail comment suivre l’état des opérations asynchrones Azure à l’aide des valeurs retournées dans la réponse.

Une seule opération de longue durée est prise en charge pour un événement en direct donné ou l’une de ses sorties en temps réel associées. Une fois démarrée, une opération de longue durée doit se terminer avant de commencer une opération de longue durée ultérieure sur le même LiveEvent ou sur les sorties en temps réel associées. Pour les événements en direct avec plusieurs sorties en temps réel, vous devez attendre la fin de l’exécution d’une opération de longue durée sur une sortie en temps réel avant de déclencher une opération de longue durée sur une autre sortie en temps réel.

## <a name="sdks"></a>Kits SDK

> [!NOTE]
> Les kits SDK Azure Media Services v3 ne sont pas garantis thread-safe. Lorsque vous développez une application multithread, vous devez ajouter votre propre logique de synchronisation de thread pour protéger le client, ou utiliser un objet AzureMediaServicesClient différent pour chaque thread. Vous devez également faire attention aux problèmes liés au multithreading provoqués par les objets facultatifs qui sont fournis au client par votre code (comme une instance HttpClient dans .NET, par exemple).

|Kit SDK|Informations de référence|
|---|---|
|[Kit de développement logiciel (SDK) .NET](https://aka.ms/ams-v3-dotnet-sdk)|[Ref de .NET](/dotnet/api/overview/azure/mediaservices/management)|
|[Kit SDK Java](https://aka.ms/ams-v3-java-sdk)|[Ref de Java](/java/api/overview/azure/mediaservices/management)|
|[Kit de développement logiciel (SDK) Python](https://aka.ms/ams-v3-python-sdk)|[Ref de Python](/python/api/overview/azure/mediaservices/management)|
|[Kit de développement logiciel (SDK) Node.js](https://aka.ms/ams-v3-nodejs-sdk) |[Ref de Node.js ](/javascript/api/overview/azure/mediaservices/management)| 
|[Kit de développement logiciel (SDK) Go](https://aka.ms/ams-v3-go-sdk) |[Ref de Go](https://aka.ms/ams-v3-go-ref)|
|[Kit de développement logiciel (SDK) Ruby](https://aka.ms/ams-v3-ruby-sdk)||

### <a name="see-also"></a>Voir aussi

- [Kit de développement logiciel (SDK) .NET EventGrid qui inclut les événements Media Services](https://www.nuget.org/packages/Microsoft.Azure.EventGrid/)
- [Définitions d’événements Media Services](https://github.com/Azure/azure-rest-api-specs/blob/master/specification/eventgrid/data-plane/Microsoft.Media/stable/2018-01-01/MediaServices.json)

## <a name="azure-media-services-explorer"></a>Azure Media Services Explorer

[Azure Media Services Explorer](https://github.com/Azure/Azure-Media-Services-Explorer) (AMSE) est un outil disponible pour les clients Windows qui souhaitent en savoir plus sur Media Services. AMSE est une application Winforms/C# qui charge, télécharge, encode, diffuse en continu du contenu VOD et en direct avec Media Services. L’outil AMSE est destiné aux clients qui souhaitent tester Media Services sans écrire de code. Le code AMSE est fourni en tant que ressource pour les clients qui souhaitent développer avec Media Services.

AMSE est un projet Open Source, son support est assuré par la Communauté (les problèmes peuvent être signalés sur https://github.com/Azure/Azure-Media-Services-Explorer/issues). Ce projet a adopté le [Code de conduite Open Source de Microsoft](https://opensource.microsoft.com/codeofconduct/). Pour plus d’informations, consultez les [Questions fréquentes (FAQ) sur le code de conduite](https://opensource.microsoft.com/codeofconduct/faq/) ou envoyez vos questions ou vos commentaires à opencode@microsoft.com.

## <a name="filtering-ordering-paging-of-media-services-entities"></a>Filtrage, classement et pagination d’entités Media Services

Consultez [Filtrage, tri et pagination des entités Azure Media Services](entities-overview.md).

## <a name="ask-questions-give-feedback-get-updates"></a>Poser des questions, envoyer des commentaires, obtenir des mises à jour

Découvrez l’article [Communauté Azure Media Services](media-services-community.md) pour découvrir les différentes façons dont vous pouvez poser des questions, faire des commentaires et obtenir des mises à jour sur Media Services.

## <a name="see-also"></a>Voir aussi

Pour obtenir toutes les valeurs nécessaires, consultez [Accéder à l’API Azure Media Services](./access-api-howto.md).

## <a name="next-steps"></a>Étapes suivantes

* [Se connecter à Media Services avec Java](configure-connect-java-howto.md)
* [Se connecter à Media Services avec .NET](configure-connect-dotnet-howto.md)
* [Se connecter à Media Services avec Node.js](configure-connect-nodejs-howto.md)
* [Se connecter à Media Services avec Python](configure-connect-python-howto.md)