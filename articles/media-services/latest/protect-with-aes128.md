---
title: Chiffrer la vidéo avec AES-128
titleSuffix: Azure Media Services
description: Découvrez comment chiffrer des vidéos avec le chiffrement AES 128 bits et comment utiliser le service de distribution de clés dans Azure Media Services.
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
ms.openlocfilehash: 5347479d32dc9f4909483dc63891e8057fd7ff86
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89289322"
---
# <a name="tutorial-encrypt-video-with-aes-128-and-use-the-key-delivery-service"></a>Tutoriel : Chiffrer la vidéo avec AES-128 et utiliser le service de distribution de clés

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

> [!NOTE]
> Bien que ce didacticiel utilise des exemples [.NET SDK](/dotnet/api/microsoft.azure.management.media.models.liveevent?view=azure-dotnet), les étapes générales sont les mêmes pour [l’API REST](/rest/api/media/liveevents), [l’interface de ligne de commande](/cli/azure/ams/live-event?view=azure-cli-latest) et d’autres [kits de développement logiciel (SDK)](media-services-apis-overview.md#sdks) pris en charge.

Vous pouvez utiliser Media Services pour transmettre du contenu HTTP Live Streaming (HLS), MPEG-DASH et Smooth Streaming chiffré avec AES à l’aide de clés de chiffrement 128 bits. Media Services assure également le service de distribution des clés qui fournit des clés de chiffrement aux utilisateurs autorisés. Si vous souhaitez que Media Services chiffre dynamiquement votre vidéo, associez une clé de chiffrement au localisateur de streaming et configurez la stratégie de clé de contenu. Lorsqu’un lecteur demande un flux de données, Media Services utilise la clé spécifiée pour chiffrer dynamiquement votre contenu avec AES-128. Pour déchiffrer le flux de données, le lecteur demande la clé au service de remise de clé. Pour déterminer si l’utilisateur est autorisé à obtenir la clé, le service évalue la stratégie de clé de contenu que vous avez spécifiée pour la clé.

Vous pouvez chiffrer chaque ressource avec plusieurs types de chiffrement (AES-128, PlayReady, Widevine, FairPlay). Consultez les [protocoles de diffusion en continu et les types de chiffrement](content-protection-overview.md#streaming-protocols-and-encryption-types) pour identifier la meilleure combinaison. Consultez aussi [Protéger avec DRM](protect-with-drm.md).

La sortie de l’exemple de cet article inclut une URL menant au lecteur multimédia Azure, l'URL du manifeste et le jeton AES nécessaires pour lire le contenu. L’exemple définit l’expiration du jeton JSON Web Token (JWT) sur 1 heure. Vous pouvez ouvrir un navigateur et coller l’URL obtenue pour lancer la page de démonstration du lecteur multimédia Azure avec l’URL et le jeton déjà renseignés pour vous au format suivant : ```https://ampdemo.azureedge.net/?url= {dash Manifest URL} &aes=true&aestoken=Bearer%3D{ JWT Token here}```.

Ce didacticiel vous explique les procédures suivantes :

> [!div class="checklist"]
> * Téléchargez l'exemple [EncryptWithAES](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithAES) décrit dans l’article.
> * Commencez à utiliser les API Media Services avec le SDK .NET.
> * Créez une ressource de sortie.
> * Créer un encodage de transformation.
> * Soumettez un travail.
> * Attendez la fin du travail.
> * Créer une stratégie de clé de contenu
> * Configurez la stratégie pour utiliser la restriction par jeton JWT.
> * Créez un localisateur de streaming.
> * Configurez le localisateur de streaming pour chiffrer la vidéo avec AES (ClearKey).
> * Obtenez un test de jeton.
> * Créez une URL de diffusion en continu.
> * Supprimer des ressources.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

Les éléments suivants sont requis pour suivre le didacticiel.

* Consultez l’article [Présentation de la protection du contenu](content-protection-overview.md).
* Installez Visual Studio Code ou Visual Studio.
* [Créer un compte Media Services](./create-account-howto.md).
* Obtenez les informations d’identification nécessaires pour utiliser les API Media Services en respectant l’[accès aux API](./access-api-howto.md).

## <a name="download-code"></a>Télécharger le code

Clonez un dépôt GitHub qui contient l’exemple .NET complet abordé dans cet article sur votre machine avec la commande suivante :

 ```bash
 git clone https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials.git
 ```

L’exemple de chiffrement avec AES-128 se trouve dans le dossier [EncryptWithAES](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithAES).

> [!NOTE]
> L’exemple crée des ressources uniques chaque fois que vous exécutez l’application. En règle générale, vous réutilisez les ressources existantes telles que les transformations et les stratégies (si une ressource existante a des configurations requises).

## <a name="start-using-media-services-apis-with-net-sdk"></a>Commencer à utiliser les API Media Services avec le Kit de développement logiciel (SDK) .NET

Pour commencer à utiliser les API Media Services avec .NET, créez un objet **AzureMediaServicesClient**. Pour créer l’objet, vous devez fournir les informations d’identification nécessaires pour que le client puisse se connecter à Azure à l’aide d’Azure AD. Dans le code que vous avez cloné au début de l’article, la fonction **GetCredentialsAsync** crée l’objet ServiceClientCredentials basé sur les informations d’identification fournies dans le fichier de configuration local.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithAES/Program.cs#CreateMediaServicesClient)]

## <a name="create-an-output-asset"></a>Créer une ressource de sortie  

La [ressource](/rest/api/media/assets) de sortie stocke le résultat de votre travail d’encodage.  

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithAES/Program.cs#CreateOutputAsset)]

## <a name="get-or-create-an-encoding-transform"></a>Obtenir ou créer un encodage de transformation

Lorsque vous créez une instance de [transformation](/rest/api/media/transforms), vous devez spécifier ce qu’elle doit produire comme sortie. Le paramètre requis est un objet **TransformOutput**, comme indiqué dans le code ci-dessous. Chaque objet **TransformOutput** contient un **préréglage**. Le **préréglage** décrit les instructions détaillées concernant les opérations de traitement vidéo et/ou audio qui doivent être utilisées pour générer l’objet **TransformOutput** souhaité. L’exemple décrit dans cet article utilise un préréglage appelé **AdaptiveStreaming**. Le préréglage encode la vidéo d’entrée dans une échelle des vitesses de transmission générée automatiquement (paires vitesse de transmission-résolution) basée sur la vitesse de transmission et la résolution de la sortie, puis crée des fichiers MP4 ISO avec des fichiers audio AAC et des fichiers vidéo H.264 qui correspondent à chaque paire vitesse de transmission-résolution.

Avant de créer une [transformation](/rest/api/media/transforms), vérifiez tout d’abord s’il en existe déjà une à l’aide de la méthode **Get**, comme indiqué dans le code suivant. Dans Media Services v3, les méthodes **Get** appliquées sur les entités renvoient **null** si l’entité n’existe pas (une vérification du nom respectant la casse).

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithAES/Program.cs#EnsureTransformExists)]

## <a name="submit-job"></a>Envoi du travail

Comme indiqué ci-dessus, l’objet [Transformation](/rest/api/media/transforms) est la formule et un [travail](/rest/api/media/jobs) est la requête réelle envoyée à Media Services pour appliquer cette **transformation** à un contenu vidéo ou audio d’entrée donné. Le **travail** spécifie des informations telles que l’emplacement de la vidéo d’entrée et celui de la sortie.

Dans ce tutoriel, nous créons l’entrée du travail à partir d’un fichier qui est ingéré directement depuis une [URL HTTPS source](job-input-from-http-how-to.md).

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithAES/Program.cs#SubmitJob)]

## <a name="wait-for-the-job-to-complete"></a>Attendre la fin du travail

Ce travail prend un certain temps. Vous voulez être averti de la fin du travail. L’exemple de code ci-dessous montre comment interroger le service pour connaître l’état du [travail](/rest/api/media/jobs). L’interrogation ne relève pas d'une pratique recommandée pour les applications de production en raison de la latence potentielle. L’interrogation peut être limitée si elle est utilisée de façon excessive sur un compte. À la place, les développeurs doivent utiliser Event Grid. Pour plus d’informations, consultez [Router des événements vers un point de terminaison web personnalisé](job-state-events-cli-how-to.md).

Le **travail** passe généralement par les états suivants : **Planifié**, **En attente**,  **Traitement en cours**, **Terminé** (l’état final). Si le travail a rencontré une erreur, vous obtenez l’état **Erreur**. Si le travail est en cours d’annulation, vous obtenez **Annulation en cours** et **Annulé** une fois l’opération terminée.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithAES/Program.cs#WaitForJobToFinish)]

## <a name="create-a-content-key-policy"></a>Créer une stratégie de clé de contenu

Une clé de contenu fournit un accès sécurisé à vos éléments multimédias. Vous devez créer une **stratégie de clé de contenu** qui configure le mode de remise de la clé de contenu pour les clients finaux. La clé de contenu est associée au **localisateur de streaming**. Media Services assure également le service de distribution des clés qui fournit des clés de chiffrement aux utilisateurs autorisés.

Lorsqu’un lecteur demande un flux de données, Media Services utilise la clé spécifiée pour chiffrer dynamiquement votre contenu (dans ce cas, à l’aide du chiffrement AES). Pour déchiffrer le flux de données, le lecteur demande la clé au service de remise de clé. Pour déterminer si l’utilisateur est autorisé à obtenir la clé, le service évalue la stratégie de clé de contenu que vous avez spécifiée pour la clé.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithAES/Program.cs#GetOrCreateContentKeyPolicy)]

## <a name="create-a-streaming-locator"></a>Créer un localisateur de streaming

Une fois l’encodage terminé, et la stratégie de clé de contenu configurée, l’étape suivante consiste à mettre à la disposition des clients la vidéo dans la ressource de sortie pour qu’ils puissent la lire. Vous rendez la vidéo disponible en deux étapes :

1. Créez un [localisateur de streaming](/rest/api/media/streaminglocators).
2. Générez les URL de streaming que les clients peuvent utiliser.

Le processus de création du **localisateur de streaming** est appelé publication. Par défaut, le **localisateur de streaming** devient valide immédiatement après les appels d’API. Il reste valide jusqu’à ce qu’il soit supprimé, sauf si vous configurez des heures de début et de fin (options facultatives).

Lors de la création d’un élément [Streaming Locator](/rest/api/media/streaminglocators), vous devrez spécifier le nom **StreamingPolicyName** souhaité. Dans ce tutoriel, nous utilisons un des éléments PredefinedStreamingPolicies, ce qui indique à Azure Media Services comment publier le contenu en vue de la diffusion en continu. Dans cet exemple, le chiffrement AES Envelope est appliqué (ce chiffrement est également appelé chiffrement ClearKey, car la clé est envoyée au client de la lecture via HTTPS et pas une licence DRM).

> [!IMPORTANT]
> Lorsque vous utilisez une stratégie [StreamingPolicy](/rest/api/media/streamingpolicies) personnalisée, vous devez concevoir un ensemble limité de ces stratégies pour votre compte Media Services et les réutiliser pour vos éléments StreamingLocators chaque fois que les mêmes protocoles et options de chiffrement sont nécessaires. Votre compte Media Services a un quota en matière de nombre d’entrées de stratégie StreamingPolicy. Vous ne devez pas créer de stratégie de streaming pour chaque localisateur de streaming.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithAES/Program.cs#CreateStreamingLocator)]

## <a name="get-a-test-token"></a>Obtenir un test de jeton

Dans ce didacticiel, nous spécifions que la stratégie de clé de contenu a une restriction au niveau du jeton. La stratégie de restriction à jeton doit être accompagnée d’un jeton émis par un service d’émission de jeton de sécurité (STS). Media Services prend en charge les jetons aux formats [JWT](/previous-versions/azure/azure-services/gg185950(v=azure.100)#BKMK_3), et c’est ce qui est configuré dans cet exemple.

L’élément ContentKeyIdentifierClaim est utilisé dans la **Stratégie de clé de contenu**, ce qui signifie que le jeton présenté au service de remise de clé doit contenir l’identificateur de la clé de contenu. Dans l’exemple, nous ne spécifions pas de clé de contenu lors de la création du localisateur de streaming. Le système en crée une de manière aléatoire pour nous. Pour générer le jeton de test, nous devons obtenir l’élément ContentKeyId à placer dans la revendication ContentKeyIdentifierClaim.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithAES/Program.cs#GetToken)]

## <a name="build-a-dash-streaming-url"></a>Générer une URL de diffusion en continu DASH

Maintenant que le [localisateur de streaming](/rest/api/media/streaminglocators) a été créé, vous pouvez obtenir les URL de diffusion en continu. Pour générer une URL, vous devez concaténer le nom d’hôte [StreamingEndpoint](/rest/api/media/streamingendpoints) et le chemin d’accès du **localisateur de streaming**. Dans cet exemple, le **point de terminaison de streaming** *par défaut* est utilisé. Quand vous créez pour la première fois un compte Media Services, ce **point de terminaison de streaming** *par défaut* est dans l’état Arrêté. Vous devez donc appeler **Start**.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithAES/Program.cs#GetMPEGStreamingUrl)]

## <a name="clean-up-resources-in-your-media-services-account"></a>Supprimer les ressources de votre compte Media Services

En règle générale, vous devez supprimer tous les éléments à l’exception des objets que vous envisagez de réutiliser (habituellement, vous réutilisez les transformations, les localisateurs de streaming, etc.). Si vous voulez que votre compte soit propre après les expériences, supprimez les ressources dont vous n’avez plus besoin. Par exemple, le code suivant supprime le travail, les ressources créées et la stratégie de clé de contenu :

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithAES/Program.cs#CleanUp)]

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous n’avez plus besoin des ressources de votre groupe de ressources, notamment les comptes de stockage et Media Services que vous avez créés pour ce didacticiel, supprimez le groupe de ressources créé précédemment.

Exécutez la commande CLI suivante :

```azurecli
az group delete --name amsResourceGroup
```

## <a name="ask-questions-give-feedback-get-updates"></a>Poser des questions, envoyer des commentaires, obtenir des mises à jour

Découvrez l’article [Communauté Azure Media Services](media-services-community.md) pour découvrir les différentes façons dont vous pouvez poser des questions, faire des commentaires et obtenir des mises à jour sur Media Services.

## <a name="additional-notes"></a>Remarques supplémentaires

* Widevine est un service fourni par Google Inc. soumis aux conditions de service et à la politique de confidentialité de Google, Inc.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Protéger avec DRM](protect-with-drm.md)
