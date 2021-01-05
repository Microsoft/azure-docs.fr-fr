---
title: Analyser des vidéos avec Media Services v3
titleSuffix: Azure Media Services
description: Découvrez comment analyser des vidéos à l’aide d’Azure Media Services.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: tutorial
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: seodec18
ms.openlocfilehash: 7bdc658ab5db9a3ffb27f3c155272f8928bbfb04
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89265862"
---
# <a name="tutorial-analyze-videos-with-media-services-v3"></a>Tutoriel : Analyser des vidéos avec Media Services v3

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

> [!NOTE]
> Ce tutoriel utilise les exemples du [SDK .NET](/dotnet/api/microsoft.azure.management.media.models.liveevent?view=azure-dotnet), mais les étapes générales sont les mêmes pour l’[API REST](/rest/api/media/liveevents), l’[interface CLI](/cli/azure/ams/live-event?view=azure-cli-latest) et d’autres [SDK](media-services-apis-overview.md#sdks) pris en charge.

Ce didacticiel vous montre comment analyser des vidéos avec Azure Media Services. Il existe plusieurs scénarios dans lesquels vous pouvez souhaiter obtenir des informations détaillées sur vos vidéos enregistrées ou vos contenus audio. Par exemple, pour obtenir une satisfaction plus élevée des clients, les organisations peuvent exécuter un traitement de reconnaissance vocale pour convertir les enregistrements du support client en un catalogue pouvant faire l’objet de recherches, avec des index et des tableaux de bord. Ensuite, elles peuvent obtenir des insights sur leur activité, par exemple, la liste des réclamations courantes avec leurs sources, et d’autres informations utiles.

Ce didacticiel vous explique les procédures suivantes :

> [!div class="checklist"]
> * Télécharger l’exemple d’application décrit dans la rubrique.
> * Examiner le code qui analyse la vidéo spécifiée.
> * Exécutez l'application.
> * Analyser la sortie.
> * Supprimer des ressources.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="compliance-privacy-and-security"></a>Conformité, confidentialité et sécurité
 
Il est important de vous rappeler que vous devez vous conformer à toutes les lois applicables dans le cadre de votre utilisation de Video Indexer, et que vous n’êtes pas autorisé à utiliser Video Indexer ou tout autre service Azure d’une façon qui porte atteinte aux droits d’autrui ou qui soit préjudiciable à autrui. Avant de charger des vidéos, y compris des données biométriques, vers le service Video Indexer à des fins de traitement et de stockage, Vous devez disposer de tous les droits appropriés sur les vidéos, y compris avoir obtenu le consentement de chaque personne apparaissant dans les vidéos. Pour en savoir plus sur la conformité, la confidentialité et la sécurité dans Video Indexer, consultez les [Conditions générales d’utilisation de Microsoft Cognitive Services](https://azure.microsoft.com/support/legal/cognitive-services-compliance-and-privacy/). Pour connaître les obligations de Microsoft en matière de confidentialité et de traitement de vos données, consultez la [Déclaration de confidentialité](https://privacy.microsoft.com/PrivacyStatement), les [Conditions d’utilisation des services en ligne](https://www.microsoft.com/licensing/product-licensing/products) (« OST ») et l’[Addenda au traitement des données](https://www.microsoftvolumelicensing.com/DocumentSearch.aspx?Mode=3&DocumentTypeId=67) (« DPA ») de Microsoft. Des informations complémentaires sur la confidentialité, notamment sur la conservation, la suppression et la destruction des données, sont disponibles dans l’OST et [ici](../video-indexer/faq.md). En utilisant Video Indexer, vous acceptez de vous conformer aux Conditions d’utilisation de Cognitive Services, à l’OST, au DPA et à la Déclaration de confidentialité.

## <a name="prerequisites"></a>Prérequis

- Si vous n’avez pas installé Visual Studio, installez [Visual Studio Community 2019](https://www.visualstudio.com/thank-you-downloading-visual-studio/?sku=Community&rel=15).
- [Créer un compte Media Services](./create-account-howto.md).<br/>Veillez à mémoriser les valeurs que vous avez utilisées pour le nom du groupe de ressources et le nom du compte Media Services.
- Suivez les étapes décrites dans [Accéder à l’API Azure Media Services avec Azure CLI](./access-api-howto.md) et enregistrez les informations d’identification. Vous en aurez besoin pour accéder à l’API.

## <a name="download-and-configure-the-sample"></a>Télécharger et configurer l’exemple

Clonez un référentiel GitHub qui contient l’exemple .NET sur votre machine à l’aide de la commande suivante :  

 ```bash
 git clone https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials.git
 ```

L’exemple se trouve dans le dossier [AnalyzeVideos](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/tree/master/AMSV3Tutorials/AnalyzeVideos).

Ouvrez [appsettings.json](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/AnalyzeVideos/appsettings.json) dans votre projet téléchargé. Remplacez les valeurs par les informations d’identification que vous avez obtenues en [accédant aux API](./access-api-howto.md).

## <a name="examine-the-code-that-analyzes-the-specified-video"></a>Examiner le code qui analyse la vidéo spécifiée

Cette section examine les fonctions définies dans le fichier [Program.cs](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/AnalyzeVideos/Program.cs) du projet *AnalyzeVideos*.

L’exemple effectue les actions suivantes :

1. Crée une **transformation** et un **travail** qui analyse votre vidéo.
2. Crée une **actif multimédia** d’entrée et charge la vidéo dans celui-ci. La ressource d’entrée est utilisée en tant qu’entrée du travail.
3. Créer une ressource de sortie qui stocke la sortie du travail.
4. Soumettre le travail.
5. Vérifier l’état du travail.
6. Télécharger les fichiers qui résultent de l’exécution du travail.

> [!NOTE]
> Lorsque vous utilisez des présélections pour l’analyseur vidéo ou audio, utilisez le Portail Azure pour paramétrer votre compte de sorte à ce qu’il dispose de 10 unités réservées Multimédia S3. Pour plus d’informations, consultez [Vue d’ensemble de la mise à l’échelle du traitement multimédia](media-reserved-units-cli-how-to.md).

### <a name="start-using-media-services-apis-with-net-sdk"></a>Commencer à utiliser les API Media Services avec le Kit de développement logiciel (SDK) .NET

Pour commencer à utiliser les API Media Services avec .NET, vous devez créer un objet **AzureMediaServicesClient**. Pour créer l’objet, vous devez fournir les informations d’identification nécessaires pour que le client puisse se connecter à Azure à l’aide d’Azure AD. Dans le code que vous avez cloné au début de l’article, la fonction **GetCredentialsAsync** crée l’objet ServiceClientCredentials basé sur les informations d’identification fournies dans le fichier de configuration local. 

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/AnalyzeVideos/Program.cs#CreateMediaServicesClient)]

### <a name="create-an-input-asset-and-upload-a-local-file-into-it"></a>Créer une ressource d’entrée et charger un fichier local dans celle-ci 

La fonction **CreateInputAsset** crée une nouvelle [ressource](/rest/api/media/assets) d’entrée et charge le fichier vidéo local spécifié dans celle-ci. Cette ressource est utilisée en tant qu’entrée de votre travail d’encodage. Dans Media Services v3, l’entrée d’un travail peut être une ressource ou bien le contenu que vous mettez à la disposition de votre compte Media Services via des URL HTTPS. Pour savoir comment encoder à partir d’une URL HTTPS, consultez [cet article](job-input-from-http-how-to.md).  

Dans Media Services v3, vous utilisez des API Stockage Azure pour charger des fichiers. L’extrait de code .NET suivant vous explique comment faire.

La fonction suivante effectue ces actions :

* Crée une ressource.
* Obtient une [URL SAS](../../storage/common/storage-sas-overview.md) accessible en écriture vers le [conteneur de stockage](../../storage/blobs/storage-quickstart-blobs-dotnet.md#upload-blobs-to-a-container) de la ressource.

    Si vous utilisez la fonction [ListContainerSas](/rest/api/media/assets/listcontainersas) de l’actif pour obtenir des URL SAS, notez que la fonction retourne plusieurs URL SAS dans la mesure où chaque compte de stockage a deux clés. Les deux clés d’un compte de stockage permettent d’effectuer une rotation transparente des clés (vous pouvez par exemple changer une clé pendant que vous utilisez l’autre, puis utiliser la nouvelle clé et déclencher la rotation de l’autre). La première URL SAS représente la clé de stockage 1 et la deuxième la clé de stockage 2.
* Charge le fichier dans le conteneur de stockage à l’aide de l’URL SAP.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/AnalyzeVideos/Program.cs#CreateInputAsset)]

### <a name="create-an-output-asset-to-store-the-result-of-the-job"></a>Créer une ressource de sortie pour stocker le résultat du travail

La [ressource](/rest/api/media/assets) de sortie stocke le résultat de votre travail. Le projet définit la fonction **DownloadResults** qui télécharge les résultats à partir de cette ressource de sortie dans le dossier « output », afin de voir ce que vous avez obtenu.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/AnalyzeVideos/Program.cs#CreateOutputAsset)]

### <a name="create-a-transform-and-a-job-that-analyzes-videos"></a>Créer une transformation et un travail qui analyse les vidéos

Lors de l’encodage ou du traitement de contenu dans Media Services, il est courant de configurer les paramètres de codage en tant que formule. Vous envoyez ensuite un **travail** pour appliquer cette formule à une vidéo. En envoyant de nouveaux travaux pour chaque nouvelle vidéo, vous appliquez cette formule à toutes les vidéos de votre bibliothèque. Dans Media Services, une formule est appelée **transformation**. Pour plus d’informations, consultez [Transformations et travaux](./transforms-jobs-concept.md). L’exemple décrit dans ce didacticiel définit une recette qui analyse la vidéo spécifiée.

#### <a name="transform"></a>Transformer

Lorsque vous créez une instance de [transformation](/rest/api/media/transforms), vous devez spécifier ce qu’elle doit produire comme sortie. **TransformOutput** est un paramètre obligatoire. Chaque objet **TransformOutput** contient un **préréglage**. Le **préréglage** décrit les instructions détaillées concernant les opérations de traitement vidéo et/ou audio qui doivent être utilisées pour générer l’objet **TransformOutput** souhaité. Dans cet exemple, le préréglage **VideoAnalyzerPreset** est utilisé et la langue (« fr-fr ») est passée à son constructeur (`new VideoAnalyzerPreset("en-US")`). Ce préréglage vous permet d’extraire plusieurs insights audio et vidéo à partir d’une vidéo. Vous pouvez utiliser le préréglage **AudioAnalyzerPreset** si vous avez besoin d’extraire plusieurs insights audio à partir d’une vidéo.

Quand vous créez une **transformation**, vérifiez d’abord s’il en existe déjà une à l’aide de la méthode **Get**, comme indiqué dans le code qui suit. Dans Media Services v3, les méthodes **Get** appliquées sur les entités renvoient **null** si l’entité n’existe pas (une vérification du nom respectant la casse).

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/AnalyzeVideos/Program.cs#EnsureTransformExists)]

#### <a name="job"></a>Travail

Comme indiqué ci-dessus, l’objet [Transformation](/rest/api/media/transforms) est la formule et un [travail](/rest/api/media/jobs) est la requête réelle envoyée à Media Services pour appliquer cette **transformation** à un contenu vidéo ou audio d’entrée donné. Le **travail** spécifie des informations telles que l’emplacement de la vidéo d’entrée et celui de la sortie. Vous pouvez spécifier l’emplacement de votre vidéo en utilisant : URL HTTPS, URL SAS ou ressources situées dans votre compte de service multimédia.

Dans cet exemple, l’entrée de la tâche est une vidéo locale.  

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/AnalyzeVideos/Program.cs#SubmitJob)]

### <a name="wait-for-the-job-to-complete"></a>Attendre la fin du travail

Ce travail prend un certain temps. Vous voulez être averti de la fin du travail. Il existe différentes options pour être prévenu de l’achèvement du [travail](/rest/api/media/jobs). L’option la plus simple (indiquée ici) est d’utiliser l’interrogation.

L’interrogation ne relève pas d'une pratique recommandée pour les applications de production en raison de la latence potentielle. L’interrogation peut être limitée si elle est utilisée de façon excessive sur un compte. À la place, les développeurs doivent utiliser Event Grid.

Event Grid est conçu pour une haute disponibilité, des performances cohérentes et une mise à l’échelle dynamique. Avec Event Grid, vos applications peuvent écouter les événements de presque tous les services Azure ou de toute source personnalisée, et y réagir. La gestion simple et réactive des événements basée sur HTTP vous aide à générer des solutions efficaces grâce au filtrage et au routage intelligents des événements. Pour plus d’informations, consultez [Router des événements vers un point de terminaison web personnalisé](job-state-events-cli-how-to.md).

Le **travail** passe généralement par les états suivants : **Planifié**, **En attente**,  **Traitement en cours**, **Terminé** (l’état final). Si le travail a rencontré une erreur, vous obtenez l’état **Erreur**. Si le travail est en cours d’annulation, vous obtenez **Annulation**, puis **Annulé** une fois l’opération terminée.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/AnalyzeVideos/Program.cs#WaitForJobToFinish)]

### <a name="job-error-codes"></a>Codes d’erreur des tâches

Consultez [Codes d’erreur](/rest/api/media/jobs/get#joberrorcode).

### <a name="download-the-result-of-the-job"></a>Télécharger le résultat du travail

La fonction suivante télécharge les résultats de la [ressource](/rest/api/media/assets) de sortie dans le dossier « sortie », afin que vous puissiez examiner les résultats du travail.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/AnalyzeVideos/Program.cs#DownloadResults)]

### <a name="clean-up-resource-in-your-media-services-account"></a>Supprimer les ressources de votre compte Media Services

En règle générale, vous devez supprimer tous les éléments à l’exception des objets que vous envisagez de réutiliser (habituellement, vous réutilisez les transformations et conservez les éléments StreamingLocators). Si vous voulez que votre compte soit propre après les expériences, supprimez les ressources dont vous n’avez plus besoin. Par exemple, le code suivant supprime le travail et la ressource de sortie :

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/AnalyzeVideos/Program.cs#CleanUp)]

## <a name="run-the-sample-app"></a>Exécution de l'exemple d'application

Appuyez sur Ctrl+F5 pour exécuter l’application *AnalyzeVideos*.

Lorsque nous exécutons le programme, le travail génère des miniatures pour chaque visage trouvé dans la vidéo. Il génère également le fichier insights.json.

## <a name="examine-the-output"></a>Analyser la sortie

Le fichier de sortie de l’analyse de vidéos est nommé insights.json. Ce fichier contient des insights sur votre vidéo. Vous trouverez une description des éléments trouvés au sein du fichier json dans l’article [Intelligence multimédia](./analyzing-video-audio-files-concept.md).

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous n’avez plus besoin des ressources de votre groupe de ressources, notamment les comptes de stockage et Media Services que vous avez créés pour ce didacticiel, supprimez le groupe de ressources créé précédemment.

Exécutez la commande CLI suivante :

```azurecli
az group delete --name amsResourceGroup
```

## <a name="multithreading"></a>Multithreading

Les kits de développement logiciel (SDK) Azure Media Services v3 ne sont pas thread-safe. Quand vous utilisez une application multithread, vous devez générer un nouvel objet AzureMediaServicesClient par thread.

## <a name="ask-questions-give-feedback-get-updates"></a>Poser des questions, envoyer des commentaires, obtenir des mises à jour

Découvrez l’article [Communauté Azure Media Services](media-services-community.md) pour découvrir les différentes façons dont vous pouvez poser des questions, faire des commentaires et obtenir des mises à jour sur Media Services.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Tutoriel : Charger, encoder et diffuser en continu des fichiers](stream-files-tutorial-with-api.md)
