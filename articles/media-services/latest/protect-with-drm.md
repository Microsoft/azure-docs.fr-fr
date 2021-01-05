---
title: Chiffrement de gestion des droits numériques (DRM) et service de remise de licence Azure Media Services
titleSuffix: Azure Media Services
description: Découvrez comment utiliser le chiffrement dynamique DRM et le service de remise de licences pour remettre des flux chiffrés avec des licences Microsoft PlayReady, Google Widevine ou Apple FairPlay.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: seodec18
ms.openlocfilehash: abaa82d6f5f33a3dc29db50ae6d029dacd3f7c13
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89289355"
---
# <a name="tutorial-use-drm-dynamic-encryption-and-license-delivery-service"></a>Tutoriel : Utilisation du chiffrement dynamique DRM et du service de remise des licences

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

> [!NOTE]
> Ce tutoriel utilise les exemples du [SDK .NET](/dotnet/api/microsoft.azure.management.media.models.liveevent?view=azure-dotnet), mais les étapes générales sont les mêmes pour l’[API REST](/rest/api/media/liveevents), l’[interface CLI](/cli/azure/ams/live-event?view=azure-cli-latest) et d’autres [SDK](media-services-apis-overview.md#sdks) pris en charge.

Vous pouvez utiliser Azure Media Services pour diffuser vos flux chiffrés avec des licences Microsoft PlayReady, Google Widevine ou Apple FairPlay. Pour une explication détaillée, consultez [Protection du contenu avec chiffrement dynamique](content-protection-overview.md).

Media Services fournit également un service de remise de licences PlayReady, Widevine et FairPlay DRM. Lorsqu’un utilisateur demande un contenu protégé par le chiffrement DRM, l’application de lecteur demande une licence au service de licence Media Services. Si l’application de lecteur est autorisée, le service de licence Media Services remet une licence au lecteur. Une licence contient la clé de déchiffrement qui peut être utilisée par le lecteur client pour déchiffrer et diffuser le contenu.

Cet article est basé sur l’exemple [Chiffrement avec DRM](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithDRM).

L’exemple décrit dans cet article génère le résultat suivant :

![AMS avec contenu vidéo protégé par DRM dans le lecteur multimédia Azure](./media/protect-with-drm/ams_player.png)

Ce didacticiel vous explique les procédures suivantes :

> [!div class="checklist"]
> * Créer un encodage de transformation.
> * définir la clé de signature utilisée pour la vérification de votre jeton.
> * Définir les exigences sur la stratégie de clé de contenu.
> * Créer un localisateur de streaming avec la stratégie de streaming spécifiée.
> * Créer une URL utilisée pour la lecture de votre fichier.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

Les éléments suivants sont requis pour suivre le didacticiel :

* Consultez l’article [Présentation de la protection du contenu](content-protection-overview.md).
* Lisez la section intitulée [Concevoir un système de protection de contenu multi-DRM avec contrôle d’accès](design-multi-drm-system-with-access-control.md).
* Installez Visual Studio Code ou Visual Studio.
* Créez un nouveau compte Azure Media Services, comme décrit dans [ce démarrage rapide](./create-account-howto.md).
* Obtenir les informations d’identification nécessaires pour utiliser les API Media Services en respectant l’[accès aux API](./access-api-howto.md)
* Définissez les valeurs appropriées dans le fichier de configuration d’application (appsettings.json).

## <a name="download-code"></a>Télécharger le code

Clonez un dépôt GitHub qui contient l’exemple .NET complet abordé dans cet article sur votre machine avec la commande suivante :

 ```bash
 git clone https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials.git
 ```
 
L’exemple de « chiffrement avec DRM » se trouve dans le dossier [EncryptWithDRM](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithDRM).

> [!NOTE]
> L’exemple crée des ressources uniques chaque fois que vous exécutez l’application. En règle générale, vous réutilisez les ressources existantes telles que les transformations et les stratégies (si une ressource existante a des configurations requises).

## <a name="start-using-media-services-apis-with-net-sdk"></a>Commencer à utiliser les API Media Services avec le Kit de développement logiciel (SDK) .NET

Pour commencer à utiliser les API Media Services avec .NET, créez un objet **AzureMediaServicesClient**. Pour créer l’objet, vous devez fournir les informations d’identification nécessaires pour que le client puisse se connecter à Azure à l’aide d’Azure AD. Dans le code que vous avez cloné au début de l’article, la fonction **GetCredentialsAsync** crée l’objet ServiceClientCredentials basé sur les informations d’identification fournies dans le fichier de configuration local.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#CreateMediaServicesClient)]

## <a name="create-an-output-asset"></a>Créer une ressource de sortie  

La [ressource](assets-concept.md) de sortie stocke le résultat de votre travail d’encodage.  

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#CreateOutputAsset)]

## <a name="get-or-create-an-encoding-transform"></a>Obtenir ou créer un encodage de transformation

Lorsque vous créez une instance de [transformation](transforms-jobs-concept.md), vous devez spécifier ce qu’elle doit produire comme sortie. Le paramètre requis est un objet `transformOutput`, comme indiqué dans le code ci-dessous. Chaque objet TransformOutput contient un **préréglage**. Le préréglage décrit les instructions détaillées concernant les opérations de traitement vidéo et/ou audio qui doivent être utilisées pour générer l’objet TransformOutput souhaité. L’exemple décrit dans cet article utilise un préréglage appelé **AdaptiveStreaming**. Le préréglage encode la vidéo d’entrée dans une échelle des vitesses de transmission générée automatiquement (paires vitesse de transmission-résolution) basée sur la vitesse de transmission et la résolution de la sortie, et crée des fichiers MP4 ISO avec des fichiers audio AAC et des fichiers vidéo H.264 qui correspondent à chaque paire vitesse de transmission-résolution. 

Avant de créer une **transformation**, vous devez tout d’abord vérifier s’il en existe déjà une à l’aide de la méthode **Get**, comme indiqué dans le code suivant.  Dans Media Services v3, les méthodes **Get** appliquées sur les entités renvoient **null** si l’entité n’existe pas (une vérification du nom respectant la casse).

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#EnsureTransformExists)]

## <a name="submit-job"></a>Envoi du travail

Comme indiqué ci-dessus, l’objet **Transformation** est la formule et un [travail](transforms-jobs-concept.md) est la requête réelle envoyée à Media Services pour appliquer cette **transformation** à un contenu vidéo ou audio d’entrée donné. Le **travail** spécifie des informations telles que l’emplacement de la vidéo d’entrée et celui de la sortie.

Dans ce tutoriel, nous créons l’entrée du travail à partir d’un fichier qui est ingéré directement depuis une [URL HTTPS source](job-input-from-http-how-to.md).

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#SubmitJob)]

## <a name="wait-for-the-job-to-complete"></a>Attendre la fin du travail

Ce travail prend un certain temps. Vous voulez être averti de la fin du travail. L’exemple de code ci-dessous montre comment interroger le service pour connaître l’état du **travail**. L’interrogation ne relève pas d'une pratique recommandée pour les applications de production en raison de la latence potentielle. L’interrogation peut être limitée si elle est utilisée de façon excessive sur un compte. À la place, les développeurs doivent utiliser Event Grid. Consultez [Acheminer des événements Azure Media Services vers un point de terminaison personnalisé à l’aide de CLI](job-state-events-cli-how-to.md).

Le **travail** passe généralement par les états suivants : **Planifié**, **En attente**,  **Traitement en cours**, **Terminé** (l’état final). Si le travail a rencontré une erreur, vous obtenez l’état **Erreur**. Si le travail est en cours d’annulation, vous obtenez **Annulation en cours** et **Annulé** une fois l’opération terminée.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#WaitForJobToFinish)]

## <a name="create-a-content-key-policy"></a>Créer une stratégie de clé de contenu

Une clé de contenu fournit un accès sécurisé à vos éléments multimédias. Vous devez créer une [stratégie de clé de contenu](content-key-policy-concept.md) pour chiffrer votre contenu avec un DRM. La stratégie configure le mode de remise des clés de contenu aux clients. La clé de contenu est associée au localisateur de streaming. Media Services assure également le service de distribution des clés qui fournit des clés de chiffrement et des licences aux utilisateurs autorisés.

Vous devez définir la configuration requise (les restrictions) sur la **stratégie de clé de contenu** pour transmettre des clés avec la configuration spécifiée. Dans cet exemple, nous avons défini les configurations et les conditions requises suivantes :

* Configuration

    Les licences [PlayReady](playready-license-template-overview.md) et [Widevine](widevine-license-template-overview.md) sont configurées de sorte qu’elles peuvent être transmises par le service de remise des licences Media Services. Bien que cet exemple d’application ne configure pas la licence [FairPlay](fairplay-license-overview.md), il contient une méthode que vous pouvez utiliser pour configurer FairPlay. Vous pouvez ajouter une configuration FairPlay comme autre option.

* Restriction

    L’application définit une restriction de type jeton JWT (JSON Web Token) dans la stratégie.

Lorsqu’un lecteur demande un flux de données, Media Services utilise la clé spécifiée pour chiffrer dynamiquement votre contenu. Pour déchiffrer le flux de données, le lecteur demande la clé au service de remise de clé. Pour déterminer si l’utilisateur est autorisé à obtenir la clé, le service évalue la stratégie de clé de contenu que vous avez spécifiée pour la clé.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#GetOrCreateContentKeyPolicy)]

## <a name="create-a-streaming-locator"></a>Créer un localisateur de streaming

Une fois l’encodage terminé, et la stratégie de clé de contenu configurée, l’étape suivante consiste à mettre à la disposition des clients la vidéo dans la ressource de sortie pour qu’ils puissent la lire. Vous rendez la vidéo disponible en deux étapes :

1. Créez un [localisateur de streaming](streaming-locators-concept.md).
2. Générez les URL de streaming que les clients peuvent utiliser.

Le processus de création du **localisateur de streaming** est appelé publication. Par défaut, le **localisateur de streaming** devient valide immédiatement après les appels d’API. Il reste valide jusqu’à ce qu’il soit supprimé, sauf si vous configurez des heures de début et de fin (options facultatives).

Lors de la création d’un élément **Streaming Locator**, vous devez spécifier le nom `StreamingPolicyName` souhaité. Dans ce tutoriel, nous utilisons l’une des stratégies de streaming prédéfinies, qui indique à Azure Media Services comment publier le contenu en vue du streaming. Dans cet exemple, nous définissons l’élément StreamingLocator.StreamingPolicyName pour la stratégie «Predefined_MultiDrmCencStreaming». Les chiffrements PlayReady et Widevine sont appliqués, et la clé est remise au client de lecture sur la base des licences DRM configurées. Si vous voulez aussi chiffrer votre flux avec CBCS (FairPlay), utilisez l’élément «Predefined_MultiDrmStreaming».

> [!IMPORTANT]
> Quand vous utilisez une [stratégie de streaming](streaming-policy-concept.md) personnalisée, vous devez concevoir un ensemble limité de ces stratégies pour votre compte Media Services et les réutiliser pour vos éléments StreamingLocators chaque fois que les mêmes protocoles et options de chiffrement sont nécessaires. Votre compte Media Services a un quota en matière de nombre d’entrées de stratégie StreamingPolicy. Vous ne devez pas créer une stratégie StreamingPolicy pour chaque élément StreamingLocator.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#CreateStreamingLocator)]

## <a name="get-a-test-token"></a>Obtenir un test de jeton

Dans ce didacticiel, nous spécifions que la stratégie de clé de contenu a une restriction au niveau du jeton. La stratégie de restriction à jeton doit être accompagnée d’un jeton émis par un service d’émission de jeton de sécurité (STS). Media Services prend en charge les jetons aux formats [JWT](/previous-versions/azure/azure-services/gg185950(v=azure.100)#BKMK_3), et c’est ce qui est configuré dans cet exemple.

L’élément ContentKeyIdentifierClaim est utilisé dans ContentKeyPolicy, ce qui signifie que le jeton présenté au service de remise de clé doit contenir l’identificateur de la ContentKey. Dans l’exemple, nous ne spécifions pas de clé de contenu lors de la création de l’élément Streaming Locator. Le système en crée une de manière aléatoire pour nous. Pour générer le jeton de test, nous devons obtenir l’élément ContentKeyId à placer dans la revendication ContentKeyIdentifierClaim.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#GetToken)]

## <a name="build-a-streaming-url"></a>Créer une URL de diffusion en continu

Maintenant que l’élément [StreamingLocator](/rest/api/media/streaminglocators) a été créé, vous pouvez obtenir les URL de diffusion en continu. Pour générer une URL, vous devez concaténer le nom d’hôte [StreamingEndpoint](/rest/api/media/streamingendpoints) et le chemin d’accès du **localisateur de streaming**. Dans cet exemple, le **point de terminaison de streaming** *par défaut* est utilisé. Quand vous créez pour la première fois un compte Media Services, ce **point de terminaison de streaming** *par défaut* est dans l’état Arrêté. Vous devez donc appeler **Start**.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#GetMPEGStreamingUrl)]

Lorsque vous exécutez l’application, vous voyez l’écran ci-dessous :

![Protéger avec DRM](./media/protect-with-drm/playready_encrypted_url.png)

Vous pouvez ouvrir un navigateur et coller l’URL obtenue pour lancer la page de démonstration du lecteur multimédia Azure Media Player avec l’URL et le jeton que vous avez déjà renseignés.

## <a name="clean-up-resources-in-your-media-services-account"></a>Supprimer les ressources de votre compte Media Services

En règle générale, vous devez supprimer tous les éléments à l’exception des objets que vous envisagez de réutiliser (habituellement, vous réutilisez les transformations, les localisateurs de streaming, etc.). Si vous voulez que votre compte soit propre après les expériences, supprimez les ressources dont vous n’avez plus besoin. Par exemple, le code suivant supprime le travail, les ressources créées et la stratégie de clé de contenu :

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#CleanUp)]

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous n’avez plus besoin des ressources de votre groupe de ressources, notamment les comptes de stockage et Media Services que vous avez créés pour ce didacticiel, supprimez le groupe de ressources créé précédemment.

Exécutez la commande CLI suivante :

```azurecli
az group delete --name amsResourceGroup
```

## <a name="additional-notes"></a>Remarques supplémentaires

* Widevine est un service fourni par Google Inc. soumis aux conditions de service et à la politique de confidentialité de Google, Inc.

## <a name="ask-questions-give-feedback-get-updates"></a>Poser des questions, envoyer des commentaires, obtenir des mises à jour

Découvrez l’article [Communauté Azure Media Services](media-services-community.md) pour découvrir les différentes façons dont vous pouvez poser des questions, faire des commentaires et obtenir des mises à jour sur Media Services.

## <a name="next-steps"></a>Étapes suivantes

Consulter

> [!div class="nextstepaction"]
> [Protéger avec AES-128](protect-with-aes128.md)
