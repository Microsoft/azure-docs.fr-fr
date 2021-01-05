---
title: Télécharger, encoder et diffuser avec Media Services v3
titleSuffix: Azure Media Services
description: Didacticiel illustrant le chargement d’un fichier, l’encodage d'une vidéo et la diffusion de contenu avec Azure Media Services v3.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: tutorial
ms.custom: mvc
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: eedbb63f4928c0397150b40a47fdc7c3e87d1991
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89256765"
---
# <a name="tutorial-upload-encode-and-stream-videos-with-media-services-v3"></a>Tutoriel : Télécharger, encoder et diffuser des vidéos avec Media Services v3

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

> [!NOTE]
> Bien que ce didacticiel utilise des exemples [.NET SDK](/dotnet/api/microsoft.azure.management.media.models.liveevent?view=azure-dotnet), les étapes générales sont les mêmes pour [l’API REST](/rest/api/media/liveevents), [l’interface de ligne de commande](/cli/azure/ams/live-event?view=azure-cli-latest) et d’autres [kits de développement logiciel (SDK)](media-services-apis-overview.md#sdks) pris en charge.

Azure Media Services vous permet d’encoder vos fichiers multimédias dans des formats lus sur un large choix de navigateurs et d’appareils. Par exemple, vous pouvez streamer votre contenu au format HLS ou MPEG DASH d’Apple. Avant la diffusion en continu, vous devez encoder votre fichier multimédia numérique haute qualité. Pour obtenir de l’aide sur l’encodage, consultez [Concept d’encodage](encoding-concept.md). Ce didacticiel charge un fichier vidéo local et encode le fichier chargé. Vous pouvez également encoder du contenu que vous mettez à disposition via une URL HTTPS. Pour plus d’informations, consultez [Créer une entrée de travail à partir d’une URL HTTP(s)](job-input-from-http-how-to.md).

![Lire une vidéo avec le lecteur multimédia Azure](./media/stream-files-tutorial-with-api/final-video.png)

Ce didacticiel vous explique les procédures suivantes :

> [!div class="checklist"]
> * Télécharger l’exemple d’application décrit dans la rubrique.
> * Examiner le code qui charge, encode et diffuse en continu
> * Exécutez l'application.
> * Tester l’URL de diffusion en continu.
> * Supprimer des ressources.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

- Si vous ne disposez pas d’une installation Visual Studio, vous pouvez obtenir [Visual Studio Community 2019](https://www.visualstudio.com/thank-you-downloading-visual-studio/?sku=Community&rel=15).
- [Créer un compte Media Services](./create-account-howto.md).<br/>Veillez à mémoriser les valeurs que vous avez utilisées pour le nom du groupe de ressources et le nom du compte Media Services.
- Suivez les étapes décrites dans [Accéder à l’API Azure Media Services avec Azure CLI](./access-api-howto.md) et enregistrez les informations d’identification. Vous en aurez besoin pour accéder à l’API.

## <a name="download-and-set-up-the-sample"></a>Télécharger et configurer l'exemple

Clonez un référentiel GitHub contenant l’exemple .NET de diffusion en continu sur votre machine à l’aide de la commande suivante :  

 ```bash
 git clone https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials.git
 ```

L’exemple se trouve dans le dossier [UploadEncodeAndStreamFiles](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/tree/master/AMSV3Tutorials/UploadEncodeAndStreamFiles).

Ouvrez [appsettings.json](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/UploadEncodeAndStreamFiles/appsettings.json) dans votre projet téléchargé. Remplacez les valeurs par les informations d’identification que vous avez obtenues en [accédant aux API](./access-api-howto.md).

## <a name="examine-the-code-that-uploads-encodes-and-streams"></a>Examiner le code qui charge, encode et diffuse en continu

Cette section examine les fonctions définies dans le fichier [Program.cs](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs) du projet *UploadEncodeAndStreamFiles*.

L’exemple effectue les actions suivantes :

1. Crée une **transformation** (vérifie d’abord si la transformation spécifiée existe).
2. Crée un **actif multimédia** de sortie qui est utilisé comme sortie du **travail** d’encodage.
3. Crée un **actif multimédia** d’entrée et charge le fichier vidéo local spécifié dans celui-ci. La ressource d’entrée est utilisée en tant qu’entrée du travail.
4. Soumet le travail d’encodage à l’aide de l’entrée et de la sortie qui ont été créées.
5. Vérifier l’état du travail.
6. Crée un **localisateur de streaming**.
7. Crée des URL de diffusion en continu.

### <a name="start-using-media-services-apis-with-net-sdk"></a>Commencer à utiliser les API Media Services avec le Kit de développement logiciel (SDK) .NET

Pour commencer à utiliser les API Media Services avec .NET, vous devez créer un objet **AzureMediaServicesClient**. Pour créer l’objet, vous devez fournir les informations d’identification permettant au client de se connecter à Azure à l’aide d’Azure AD. Dans le code que vous avez cloné au début de l’article, la fonction **GetCredentialsAsync** crée l’objet ServiceClientCredentials basé sur les informations d’identification fournies dans le fichier de configuration local.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#CreateMediaServicesClient)]

### <a name="create-an-input-asset-and-upload-a-local-file-into-it"></a>Créer une ressource d’entrée et charger un fichier local dans celle-ci

La fonction **CreateInputAsset** crée une nouvelle [ressource](/rest/api/media/assets) d’entrée et charge le fichier vidéo local spécifié dans celle-ci. Cet **élément multimédia** est utilisé comme entrée de votre travail d’encodage. Dans Media Services v3, l’entrée d’un **travail** peut être un **élément multimédia** ou du contenu que vous mettez à la disposition de votre compte Media Services par le biais d’URL HTTPS. Pour savoir comment encoder à partir d’une URL HTTPS, consultez [cet article](job-input-from-http-how-to.md).

Dans Media Services v3, vous utilisez des API Stockage Azure pour charger des fichiers. L’extrait de code .NET suivant vous explique comment faire.

La fonction suivante effectue les actions ci-après :

* Crée un **élément multimédia**.
* Obtient une [URL SAP](../../storage/common/storage-sas-overview.md) accessible en écriture vers le [conteneur de stockage](../../storage/blobs/storage-quickstart-blobs-dotnet.md#upload-blobs-to-a-container) de l’élément multimédia.

    Si vous utilisez la fonction [ListContainerSas](/rest/api/media/assets/listcontainersas) de l’actif pour obtenir des URL SAS, notez que la fonction retourne plusieurs URL SAS dans la mesure où chaque compte de stockage a deux clés. Les deux clés d’un compte de stockage permettent d’effectuer une rotation transparente des clés (vous pouvez par exemple changer une clé pendant que vous utilisez l’autre, puis utiliser la nouvelle clé et déclencher la rotation de l’autre). La première URL SAS représente la clé de stockage 1 et la deuxième la clé de stockage 2.
* Charge le fichier dans le conteneur de stockage à l’aide de l’URL SAP.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#CreateInputAsset)]

### <a name="create-an-output-asset-to-store-the-result-of-a-job"></a>Créer une ressource de sortie pour stocker le résultat d’un travail

La [ressource](/rest/api/media/assets) de sortie stocke le résultat de votre travail d’encodage. Le projet définit la fonction **DownloadResults** qui télécharge les résultats à partir de cette ressource de sortie dans le dossier « output », afin de voir ce que vous avez obtenu.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#CreateOutputAsset)]

### <a name="create-a-transform-and-a-job-that-encodes-the-uploaded-file"></a>Créer une transformation et un travail qui encode le fichier chargé

Lors de l’encodage ou du traitement de contenu dans Media Services, il est courant de configurer les paramètres de codage en tant que formule. Vous envoyez ensuite un **travail** pour appliquer cette formule à une vidéo. En envoyant de nouveaux travaux pour chaque nouvelle vidéo, vous appliquez cette formule à toutes les vidéos de votre bibliothèque. Dans Media Services, une formule est appelée **transformation**. Pour plus d’informations, consultez [Transformations et travaux](./transforms-jobs-concept.md). L’exemple décrit dans ce didacticiel définit une formule qui encode la vidéo, afin de la diffuser en continu sur divers appareils iOS et Android.

#### <a name="transform"></a>Transformer

Lorsque vous créez une instance de [transformation](/rest/api/media/transforms), vous devez spécifier ce qu’elle doit produire comme sortie. Le paramètre requis est un objet **TransformOutput**, comme indiqué dans le code ci-dessous. Chaque objet **TransformOutput** contient un **préréglage**. Le **préréglage** décrit les instructions détaillées concernant les opérations de traitement vidéo et/ou audio qui doivent être utilisées pour générer l’objet **TransformOutput** souhaité. L’exemple décrit dans cet article utilise un préréglage appelé **AdaptiveStreaming**. Le préréglage encode la vidéo d’entrée dans une échelle des vitesses de transmission générée automatiquement (paires vitesse de transmission-résolution) basée sur la vitesse de transmission et la résolution de la sortie, et crée des fichiers MP4 ISO avec des fichiers audio AAC et des fichiers vidéo H.264 qui correspondent à chaque paire vitesse de transmission-résolution. Pour plus d’informations sur ce préréglage, consultez [Encode with an auto-generated bitrate ladder](autogen-bitrate-ladder.md) (Encoder avec une échelle des vitesses de transmission générée automatiquement).

Vous pouvez utiliser un préréglage EncoderNamedPreset intégré ou des préréglages personnalisés. Pour plus d’informations, consultez [How to customize encoder presets](customize-encoder-presets-how-to.md) (Procédure : personnaliser les présélections de l’encodeur).

Lorsque vous créez une [transformation](/rest/api/media/transforms), vous devez tout d’abord vérifier s’il en existe déjà une à l’aide de la méthode **Get**, comme indiqué dans le code qui suit. Dans Media Services v3, les méthodes **Get** appliquées sur les entités renvoient **null** si l’entité n’existe pas (une vérification du nom respectant la casse).

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#EnsureTransformExists)]

#### <a name="job"></a>Travail

Comme indiqué ci-dessus, l’objet [Transformation](/rest/api/media/transforms) est la formule et un [travail](/rest/api/media/jobs) est la requête réelle envoyée à Media Services pour appliquer cette **transformation** à un contenu vidéo ou audio d’entrée donné. Le **travail** spécifie des informations telles que l’emplacement de la vidéo d’entrée et celui de la sortie.

Dans cet exemple, la vidéo d’entrée a été chargée à partir de votre ordinateur local. Pour savoir comment encoder à partir d’une URL HTTPS, consultez [cet article](job-input-from-http-how-to.md).

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#SubmitJob)]

### <a name="wait-for-the-job-to-complete"></a>Attendre la fin du travail

Le travail prend du temps à se terminer et vous voulez être prévenu lorsque c’est le cas. L’exemple de code ci-dessous montre comment interroger le service pour connaître l’état du [travail](/rest/api/media/jobs). L’interrogation ne relève pas d'une pratique recommandée pour les applications de production en raison de la latence potentielle. L’interrogation peut être limitée si elle est utilisée de façon excessive sur un compte. À la place, les développeurs doivent utiliser Event Grid.

Event Grid est conçu pour une haute disponibilité, des performances cohérentes et une mise à l’échelle dynamique. Avec Event Grid, vos applications peuvent écouter les événements de presque tous les services Azure ou de toute source personnalisée, et y réagir. La gestion simple et réactive des événements basée sur HTTP vous aide à générer des solutions efficaces grâce au filtrage et au routage intelligents des événements.  Consultez [Acheminer des événements Azure Media Services vers un point de terminaison personnalisé à l’aide de CLI](job-state-events-cli-how-to.md).

Le **travail** passe généralement par les états suivants : **Planifié**, **En attente**,  **Traitement en cours**, **Terminé** (l’état final). Si le travail a rencontré une erreur, vous obtenez l’état **Erreur**. Si le travail est en cours d’annulation, vous obtenez **Annulation en cours** et **Annulé** une fois l’opération terminée.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#WaitForJobToFinish)]

### <a name="job-error-codes"></a>Codes d’erreur des tâches

Consultez [Codes d’erreur](/rest/api/media/jobs/get#joberrorcode).

### <a name="get-a-streaming-locator"></a>Obtenir un localisateur de streaming

Une fois l’encodage terminé, l’étape suivante consiste à mettre à la disposition des clients la vidéo dans la ressource de sortie pour qu’ils puissent la lire. Pour ce faire, vous suivez deux étapes : commencez par créer un [localisateur de streaming](/rest/api/media/streaminglocators), puis générez les URL de streaming que les clients peuvent utiliser.

Le processus de création d’un **localisateur de streaming** est appelée « publication ». Par défaut, le **localisateur de streaming** est valide immédiatement après avoir effectué les appels d’API et dure jusqu’à ce qu’il soit supprimé, sauf si vous configurez les heures de début et de fin facultatives.

Lors de la création d’un élément [StreamingLocator](/rest/api/media/streaminglocators), vous devez spécifier le nom **StreamingPolicyName** souhaité. Dans cet exemple, vous allez diffuser en continu du contenu en clair (ou non chiffré), la stratégie de diffusion en continu en clair prédéfinie (**PredefinedStreamingPolicy.ClearStreamingOnly**) est donc utilisée.

> [!IMPORTANT]
> Quand vous utilisez une [stratégie de streaming](/rest/api/media/streamingpolicies) personnalisée, vous devez concevoir un ensemble limité de ces stratégies pour votre compte Media Services et les réutiliser pour vos éléments StreamingLocators chaque fois que les mêmes protocoles et options de chiffrement sont nécessaires. Votre compte Media Services a un quota en matière de nombre d’entrées de stratégie de streaming. Vous ne devez pas créer de stratégie de streaming pour chaque localisateur de streaming.

Le code suivant suppose que vous appelez la fonction avec un locatorName unique.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#CreateStreamingLocator)]

Tandis que l’exemple de cette rubrique traite de streaming, vous pouvez utiliser le même appel pour créer un localisateur de streaming pour la diffusion de contenu vidéo par le biais d’un téléchargement progressif.

### <a name="get-streaming-urls"></a>Obtenir des URL de diffusion en continu

Maintenant que le [localisateur de streaming](/rest/api/media/streaminglocators) a été créé, vous pouvez obtenir les URL de streaming, comme indiqué dans **GetStreamingURLs**. Pour générer une URL, vous devez concaténer le nom d’hôte du [point de terminaison de streaming](/rest/api/media/streamingendpoints) et le chemin du **localisateur de streaming**. Dans cet exemple, le **point de terminaison de streaming** *par défaut* est utilisé. Quand vous créez pour la première fois un compte Media Services, ce **point de terminaison de streaming** *par défaut* est dans l’état Arrêté. Vous devez donc appeler **Start**.

> [!NOTE]
> Dans cette méthode, vous avez besoin de la valeur locatorName qui a été utilisé lors de la création du **localisateur de streaming** pour l’actif multimédia de sortie.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#GetStreamingURLs)]

### <a name="clean-up-resources-in-your-media-services-account"></a>Supprimer les ressources de votre compte Media Services

En règle générale, vous devez supprimer tous les éléments à l’exception des objets que vous envisagez de réutiliser (habituellement, vous réutilisez les transformations et conservez les éléments StreamingLocators, etc.). Si vous souhaitez que votre compte soit propre au terme de vos expériences, vous devez supprimer les ressources que vous n’envisagez pas de réutiliser. Par exemple, le code suivant supprime le travail, les ressources créées et la stratégie de clé de contenu :

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/UploadEncodeAndStreamFiles/Program.cs#CleanUp)]

## <a name="run-the-sample-app"></a>Exécution de l'exemple d'application

1. Appuyez sur Ctrl+F5 pour exécuter l’application *EncodeAndStreamFiles*.
2. Copiez l’une des URL de diffusion en continu à partir de la console.

Cet exemple affiche les URL qui peuvent être utilisées pour lire la vidéo à l’aide de différents protocoles :

![Exemple de sortie présentant des URL pour la diffusion en continu de vidéo Media Services](./media/stream-files-tutorial-with-api/output.png)

## <a name="test-the-streaming-url"></a>Tester l’URL de diffusion en continu

Pour tester la diffusion en continu, cet article utilise le lecteur multimédia Azure.

> [!NOTE]
> Si un lecteur est hébergé sur un site https, veillez à mettre à jour l’URL vers « https ».

1. Ouvrez un navigateur web et accédez à [https://aka.ms/azuremediaplayer/](https://aka.ms/azuremediaplayer/).
2. Dans le champ **URL :** , collez l’une des valeurs d’URL de diffusion en continu que vous avez obtenues lors de l’exécution de l’application.
3. Sélectionnez **Update Player (Mettre à jour le lecteur)** .

Le lecteur multimédia Azure peut être utilisé pour effectuer des tests, mais ne doit pas être utilisé dans un environnement de production.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous n’avez plus besoin des ressources de votre groupe de ressources, notamment les comptes de stockage et Media Services que vous avez créés pour ce didacticiel, supprimez le groupe de ressources créé précédemment.

Exécutez la commande CLI suivante :

```azurecli
az group delete --name amsResourceGroup
```

## <a name="multithreading"></a>Multithreading

Les kits de développement logiciel (SDK) Azure Media Services v3 ne sont pas thread-safe. Lorsque vous développez une application multithread, vous devez générer et utiliser un nouvel objet AzureMediaServicesClient par thread.

## <a name="ask-questions-give-feedback-get-updates"></a>Poser des questions, envoyer des commentaires, obtenir des mises à jour

Découvrez l’article [Communauté Azure Media Services](media-services-community.md) pour découvrir les différentes façons dont vous pouvez poser des questions, faire des commentaires et obtenir des mises à jour sur Media Services.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous savez comment charger, encoder et diffuser en continu votre vidéo, consultez l’article suivant : 

> [!div class="nextstepaction"]
> [Analyser des vidéos](analyze-videos-tutorial-with-api.md)
