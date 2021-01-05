---
title: Points à prendre en compte lors de l’utilisation de Video Indexer à grande échelle - Azure
titleSuffix: Azure Media Services
description: Cette rubrique explique les éléments à prendre en compte lors de l’utilisation de Video Indexer à grande échelle.
services: media-services
author: Juliako
manager: femila
ms.service: media-services
ms.subservice: video-indexer
ms.topic: how-to
ms.date: 11/13/2020
ms.author: juliako
ms.openlocfilehash: b955c0f494b757fd29c400194ef8b11314a89a03
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96483608"
---
# <a name="things-to-consider-when-using-video-indexer-at-scale"></a>Points à prendre en compte lors de l’utilisation de Video Indexer à grande échelle

Lorsque vous utilisez Azure Media Services Video indexer pour indexer des vidéos et que votre archive de vidéos augmente, envisagez une mise à l’échelle. 

Cet article apporte des réponses à des questions telles que :

* Dois-je prendre en considération des contraintes technologiques ?
* Existe-t-il un moyen intelligent et efficace pour y parvenir ?
* Puis-je empêcher les dépenses excédentaires au cours du processus ?

L’article propose six pratiques idéales concernant l’utilisation de Video Indexer à grande échelle.

## <a name="when-uploading-videos-consider-using-a-url-over-byte-array"></a>Lors du chargement de vidéos, envisagez d’utiliser une URL sur un tableau d’octets

Video Indexer vous donne la possibilité de charger des vidéos à partir d’une URL ou directement en envoyant le fichier sous la forme d’un tableau d’octets, cette dernière solution s’accompagne de certaines contraintes. Pour plus d’informations, consultez [considérations et limitations relatives au chargement](upload-index-videos.md#uploading-considerations-and-limitations)

Tout d’abord, le fichier présente des limitations en termes de taille. La taille du fichier de tableau d’octets est limitée à 2 Go par rapport à la limite de taille de chargement de 30 Go lors de l’utilisation de l’URL.

Prenez ensuite en compte certains des problèmes qui peuvent affecter vos performances et, par conséquent, la possibilité de mettre à l’échelle :

* L’envoi de fichiers en plusieurs parties signifie une dépendance élevée liée à votre réseau, 
* à la fiabilité des appareils, 
* à la connectivité, 
* à la vitesse de chargement, 
* ainsi que des paquets perdus sur le web.

:::image type="content" source="./media/considerations-when-use-at-scale/first-consideration.png" alt-text="Premier point à prendre en compte lors de l’utilisation de Video Indexer à grande échelle":::

Lorsque vous chargez des vidéos à partir d’une URL, il vous suffit de fournir un chemin d’accès à l’emplacement d’un fichier multimédia, et Video Indexer se charge du reste (consultez le champ `videoUrl` dans l’API [charger une vidéo](https://api-portal.videoindexer.ai/docs/services/Operations/operations/Upload-Video?&pattern=upload)).

> [!TIP]
> Utilisez le paramètre `videoUrl` facultatif de l’API de chargement de vidéos.

Pour voir un exemple de chargement de vidéos à partir d’une URL, consultez [cet exemple](upload-index-videos.md#code-sample). Vous pouvez également utiliser [AzCopy](../../storage/common/storage-use-azcopy-v10.md) pour obtenir un moyen rapide et fiable d’obtenir votre contenu sur un compte de stockage à partir duquel vous pouvez l’envoyer à Video Indexer à l’aide de l’[URL SAS](../../storage/common/storage-sas-overview.md).

## <a name="increase-media-reserved-units-if-needed"></a>Augmenter les unités réservées Multimédia si nécessaire

En général, lors de la phase de preuve de concept, lorsque vous commencez à utiliser Video Indexer, vous n’avez pas besoin de beaucoup de puissance de calcul. Lorsque vous commencez à utiliser une plus grande archive de vidéos que vous devez indexer et que vous souhaitez que le processus fonctionne à un rythme adapté à votre cas d’usage, vous devez mettre à l’échelle votre utilisation de Video Indexer. Par conséquent, vous devez envisager d’augmenter le nombre de ressources de calcul que vous utilisez si la puissance de calcul actuelle n’est pas suffisante.

Dans Azure Media Services, lorsque vous souhaitez augmenter la puissance de calcul et la parallélisation, vous devez prêter attention aux [unités réservées](../latest/concept-media-reserved-units.md) Multimédia. Les unités réservées déterminent les paramètres de vos tâches de traitement multimédia. Le nombre d’unités réservées affecte le nombre de tâches multimédia qui peuvent être traitées simultanément dans chaque compte. Leur type détermine la vitesse de traitement, et une vidéo peut nécessiter plus d’une unité réservée si son indexation est complexe. Lorsque vos unités réservées sont occupées, les nouvelles tâches sont conservées dans une file d’attente jusqu’à ce qu’une autre ressource soit disponible.

Pour fonctionner efficacement et éviter d’avoir des ressources qui restent partiellement inactives, Video Indexer offre un système de mise à l’échelle automatique qui éteint les unités réservées lorsque moins de traitement est nécessaire et les réactive en période d’affluence (jusqu’à la pleine utilisation de toutes vos unités réservées). Vous pouvez activer cette fonctionnalité en [activant la mise à l’échelle automatique](manage-account-connected-to-azure.md#autoscale-reserved-units) dans les paramètres du compte ou à l’aide de l’[API de mise à jour de Azure Media Services pour les comptes payants](https://api-portal.videoindexer.ai/docs/services/Operations/operations/Update-Paid-Account-Azure-Media-Services?&pattern=update).

:::image type="content" source="./media/considerations-when-use-at-scale/second-consideration.jpg" alt-text="Deuxième point à prendre en compte lors de l’utilisation de Video Indexer à grande échelle":::

:::image type="content" source="./media/considerations-when-use-at-scale/reserved-units.jpg" alt-text="Unités réservées AMS":::

Pour réduire la durée de l’indexation et le ralentissement du débit, nous vous recommandons de commencer par 10 unités réservées de type S3. Plus tard, si vous mettez à l’échelle pour prendre en charge davantage de contenu ou faire face à une concurrence plus élevée, et que vous avez besoin de plus de ressources pour y parvenir, vous pouvez [nous contacter à l’aide du système de support](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) (uniquement sur les comptes payants) pour demander d’affecter davantage d’unités réservées.

## <a name="respect-throttling"></a>Respect de la limitation

Video Indexer est conçu pour gérer l’indexation à grande échelle, et lorsque vous souhaitez en tirer pleinement parti, vous devez également connaître les capacités du système et concevoir votre intégration en conséquence. Vous ne souhaitez pas envoyer de demande de téléchargement pour un lot de vidéos uniquement pour découvrir que certains films n’ont pas été téléchargés et recevoir un code de réponse HTTP 429 (trop de requêtes). Cela peut se produire car vous avez envoyé plus de demandes que la [limite de films par minute que nous prenons en charge](upload-index-videos.md#uploading-considerations-and-limitations). Video Indexer ajoute un en-tête de `retry-after` dans la réponse HTTP, l’en-tête spécifie à quel moment vous devez réaliser votre prochaine tentative. Veillez à le respecter avant d’essayer votre prochaine requête.

:::image type="content" source="./media/considerations-when-use-at-scale/respect-throttling.jpg" alt-text="Concevez correctement votre intégration, respect de la limitation":::

## <a name="use-callback-url"></a>Utilisez l’URL de rappel

Nous vous recommandons, au lieu d’interroger l’état de votre requête dès que vous avez envoyé la demande de chargement, d’ajouter une [URL de rappel](upload-index-videos.md#callbackurl) et d’attendre la mise à jour de Video Indexer. Dès qu’il y a un changement d’état dans votre demande de chargement, vous recevez une notification POST vers l’URL que vous avez spécifiée.

Vous pouvez ajouter une URL de rappel comme l’un des paramètres de l’[API Charger la vidéo](https://api-portal.videoindexer.ai/docs/services/Operations/operations/Upload-Video?&pattern=upload). Consultez les exemples de code dans le [référentiel GitHub](https://github.com/Azure-Samples/media-services-video-indexer/tree/master/). 

Pour l’URL de rappel, vous pouvez également utiliser Azure Functions, une plateforme basée sur les événements serverless qui peut être déclenchée par HTTP et implémenter un flux suivant.

### <a name="callback-url-definition"></a>définition de l’URL de rappel

[!INCLUDE [callback url](./includes/callback-url.md)]

## <a name="use-the-right-indexing-parameters-for-you"></a>Utilisez les paramètres d’indexation appropriés pour vous

Lorsque vous prenez des décisions en rapport avec l’utilisation de Video Indexer à grande échelle, découvrez comment en tirer le meilleur parti avec les paramètres appropriés à vos besoins. Pensez à votre cas d’utilisation. En définissant des paramètres différents vous permettant de réaliser des économies et d’accélérer le processus d’indexation de vos vidéos.

Avant de charger et d’indexer votre vidéo, lisez cette brève [documentation](upload-index-videos.md), consultez [indexingPreset](upload-index-videos.md#indexingpreset) et [streamingPreset](upload-index-videos.md#streamingpreset) pour mieux comprendre quelles sont vos options.

Par exemple, ne définissez pas la présélection sur diffusion en continu si vous n’envisagez pas de regarder la vidéo ; n’indexez pas la vidéo si vous avez uniquement besoin d’aperçus audio.

## <a name="index-in-optimal-resolution-not-highest-resolution"></a>Index en résolution optimale, pas dans la résolution la plus élevée

Vous vous demandez peut-être la qualité vidéo dont vous avez besoin pour l’indexation de vos vidéos ? 

Dans de nombreux cas, il n’y a pratiquement aucune différence de performances d’indexation entre les vidéos HD (720P) et les vidéos 4K. Au bout du compte, vous obtiendrez quasiment les mêmes aperçus avec la même confiance. Plus la qualité du film que vous chargez est élevée, plus la taille du fichier est élevée, et cela entraîne une puissance de calcul supérieure et plus de temps nécessaire pour le chargement de la vidéo.

Par exemple, pour la fonctionnalité de détection de visages, une résolution plus élevée peut être utile en présence de nombreux visages plus petits mais importants dans le contexte. Toutefois, cela s’accompagne d’une augmentation quadratique du runtime et d’un risque accru de faux positifs.

Par conséquent, nous vous recommandons de vérifier que vous disposez des bons résultats pour votre cas d’usage et de le tester localement en premier lieu. Téléchargez la même vidéo en 720P et en 4K, puis comparez les insights que vous recevez.

## <a name="next-steps"></a>Étapes suivantes

[Examiner la sortie d’Azure Video Indexer générée par l’API](video-indexer-output-json-v2.md)