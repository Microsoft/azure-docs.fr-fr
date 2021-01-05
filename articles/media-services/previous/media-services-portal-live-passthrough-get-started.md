---
title: Streaming en direct avec des encodeurs locaux à l’aide du portail Azure | Microsoft Docs
description: Ce didacticiel vous guide tout au long des étapes de création d’un canal configuré pour une livraison directe.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.assetid: 6f4acd95-cc64-4dd9-9e2d-8734707de326
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 04/01/2019
ms.author: juliako
ms.openlocfilehash: 707c12cac6bbceee925c4710eff29482f687d47f
ms.sourcegitcommit: 4f4a2b16ff3a76e5d39e3fcf295bca19cff43540
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93040724"
---
# <a name="perform-live-streaming-with-on-premises-encoders-using-azure-portal"></a>Effectuer un streaming en direct avec des encodeurs locaux à l’aide du portail Azure

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!div class="op_single_selector"]
> * [Portail](media-services-portal-live-passthrough-get-started.md)
> * [.NET](media-services-dotnet-live-encode-with-onpremises-encoders.md)
> * [REST](/rest/api/media/operations/channel)
> 
> 

> [!NOTE]
> Aucune nouvelle fonctionnalité ni fonction n’est ajoutée à Media Services v2. <br/>Découvrez la dernière version, [Media Services v3](../latest/index.yml). Consultez aussi [Conseils de migration de v2 vers v3](../latest/migrate-from-v2-to-v3.md).

Ce didacticiel vous guide tout au long des étapes d’utilisation du portail Azure afin de créer un **canal** configuré pour une livraison directe. 

## <a name="prerequisites"></a>Prérequis
Les éléments suivants sont requis pour suivre le didacticiel :

* Un compte Azure. Pour plus d’informations, consultez [Essai gratuit Azure](https://azure.microsoft.com/pricing/free-trial/). 
* Un compte Media Services. Pour créer un compte Media Services, consultez [Création d’un compte Media Services](media-services-portal-create-account.md).
* Une webcam. Par exemple, un [encodeur Telestream Wirecast](media-services-configure-wirecast-live-encoder.md). 

Il est vivement recommandé de consulter les articles suivants :

* [Prise en charge RTMP et encodeurs dynamiques dans Azure Media Services.](https://azure.microsoft.com/blog/2014/09/18/azure-media-services-rtmp-support-and-live-encoders/)
* [Vue d’ensemble de la vidéo en flux continu à l’aide d’Azure Media Services](media-services-manage-channels-overview.md)
* [Streaming en direct avec des encodeurs en local qui créent des flux multidébits](media-services-live-streaming-with-onprem-encoders.md)

## <a name="common-live-streaming-scenario"></a><a id="scenario"></a>Scénario courant de streaming en direct

Les étapes suivantes décrivent les tâches impliquées dans la création d’applications courantes de diffusion en continu qui utilisent des canaux configurés pour une livraison directe. Ce didacticiel explique comment créer et gérer un canal direct et des événements en direct.

> [!NOTE]
> Assurez-vous que le point de terminaison à partir duquel vous souhaitez diffuser du contenu se trouve dans l’état **En cours d’exécution**. 
    
1. Connectez une caméra vidéo à un ordinateur. <br/>Pour obtenir des idées de configuration, consultez [Simple and portable event video gear setup]( https://link.medium.com/KNTtiN6IeT).
1. Lancez et configurez un encodeur dynamique local qui produit un flux à débit binaire multiple au format MP4 fragmenté ou RTMP. Pour plus d’informations, voir [Prise en charge RTMP et encodeurs dynamiques dans Azure Media Services](https://go.microsoft.com/fwlink/?LinkId=532824).<br/>Consultez également ce billet de blog : [Live streaming production with OBS](https://link.medium.com/ttuwHpaJeT).
   
    Cette étape peut également être effectuée après la création du canal.
1. Créez et démarrez un canal direct.
1. Récupérez l’URL de réception du canal. 
   
    L’URL de réception est utilisée par l’encodeur dynamique pour envoyer le flux au canal.
1. Récupérez l’URL d’aperçu du canal. 
   
    Utilisez cette URL pour vérifier que votre canal reçoit correctement le flux dynamique.
1. Créez un événement/programme en direct. 
   
    Lors de l’utilisation du portail Azure, la création d’un événement en direct a également pour effet de créer un élément multimédia. 

1. Démarrez l’événement ou le programme dès que vous êtes prêt à lancer la diffusion en continu et l’archivage.
1. Un signal peut éventuellement être envoyé à l’encodeur dynamique pour qu’il démarre une publicité. La publicité est insérée dans le flux de sortie.
1. Arrêtez l’événement ou le programme chaque fois que vous voulez arrêter la diffusion et archiver l’événement.
1. Supprimez l’événement ou le programme (et éventuellement la ressource).     

> [!IMPORTANT]
> Pour en savoir plus sur les considérations et concepts liés au streaming en direct avec des encodeurs en local et des canaux directs, consultez [Streaming en direct avec des encodeurs locaux qui créent des flux multidébits](media-services-live-streaming-with-onprem-encoders.md).
> 
> 

## <a name="to-view-notifications-and-errors"></a>Pour afficher les erreurs et notifications
Si vous souhaitez afficher les notifications et les erreurs produites par le portail Azure, cliquez sur l’icône Notification.

![Notifications](./media/media-services-portal-passthrough-get-started/media-services-notifications.png)

## <a name="create-and-start-pass-through-channels-and-events"></a>Créer et démarrer des canaux directs et des événements
Un canal est associé à des événements/programmes vous permettant de contrôler la publication et le stockage des segments dans un flux dynamique. Les canaux gèrent des événements. 

Vous pouvez spécifier le nombre d’heures pendant lesquelles vous souhaitez conserver le contenu enregistré pour le programme en définissant la durée de la **fenêtre d’archivage** . Cette valeur peut être comprise entre 5 minutes et 25 heures. La durée de la fenêtre d’archivage détermine également la plage maximale de temps dans laquelle les clients peuvent effectuer des recherches en arrière à partir de la position dynamique actuelle. Les événements peuvent durer davantage que le laps de temps spécifié, mais le contenu qui se situe en dehors de la longueur de fenêtre est ignoré en permanence. La valeur de cette propriété détermine également la longueur maximale que les manifestes de client peuvent atteindre.

Chaque événement est associé à un élément multimédia. Pour publier l’événement, vous devez créer un localisateur OnDemand pour l’élément multimédia associé. Le fait de posséder ce localisateur vous permet de générer une URL de diffusion en continu que vous pourrez fournir à vos clients.

Un canal prend en charge jusqu’à trois événements exécutés simultanément, ce qui rend possible la création de plusieurs archives du même flux entrant. Cela vous permet de publier et d’archiver différentes parties d’un événement en fonction des besoins. Par exemple, imaginez que vous devez archiver 6 heures d’un programme, mais diffuser uniquement les 10 dernières minutes. Pour ce faire, vous devez créer deux programmes exécutés simultanément. Un programme est configuré pour archiver 6 heures de l’événement, mais il n’est pas publié. L’autre programme est configuré pour archiver pendant 10 minutes et il est publié.

Vous ne devez pas réutiliser d’événements en direct existants. Créez et lancez plutôt un nouvel événement pour chaque événement.

Démarrez l’événement dès que vous êtes prêt à lancer la diffusion en continu et l’archivage. Arrêtez le programme chaque fois que vous voulez arrêter la diffusion et archiver l’événement. 

Pour supprimer du contenu archivé, arrêtez et supprimez l’événement, puis supprimez l’élément multimédia associé. Un élément multimédia ne peut pas être supprimé s’il est utilisé par un événement ; vous devez d’abord supprimer l’événement. 

Même après l’arrêt et la suppression de l’événement, les utilisateurs pourront lire votre contenu archivé en tant que vidéo à la demande tant que vous n’aurez pas supprimé l’élément multimédia.

Si vous souhaitez conserver le contenu archivé sans qu’il soit disponible pour la diffusion, supprimez le localisateur de diffusion en continu.

### <a name="to-use-the-portal-to-create-a-channel"></a>Pour créer un canal à l’aide du portail
Cette section explique comment utiliser l’option **Création rapide** pour créer un canal direct.

Pour plus d’informations sur les canaux directs, consultez [Streaming en direct avec des encodeurs locaux qui créent des flux multidébits](media-services-live-streaming-with-onprem-encoders.md).

1. Dans le [portail Azure](https://portal.azure.com/), sélectionnez votre compte Azure Media Services.
2. Dans la fenêtre **Paramètres** , cliquez sur **Diffusion en continu**. 
   
    ![Prise en main](./media/media-services-portal-passthrough-get-started/media-services-getting-started.png)
   
    La fenêtre **Diffusion en continu** s’affiche.
3. Cliquez sur **Création rapide** pour créer un canal direct avec le protocole de réception RTMP.
   
    La fenêtre **Créer un nouveau canal** s’affiche.
4. Nommez le nouveau canal et cliquez sur **Créer**. 
   
    Vous obtenez un canal direct avec le protocole de réception RTMP.

## <a name="create-events"></a>Créer des événements
1. Sélectionnez un canal auquel vous souhaitez ajouter un événement.
2. Appuyez sur le bouton **Événement réel** .

![Événement](./media/media-services-portal-passthrough-get-started/media-services-create-events.png)

## <a name="get-ingest-urls"></a>Obtenir les URL de réception
Lorsque le canal est créé, vous pouvez obtenir des URL de réception que vous devez fournir à l’encodeur dynamique. L’encodeur utilise ces URL pour entrer un flux dynamique.

![Capture d’écran montrant la page « Streaming en direct » avec un canal sélectionné et le volet du canal affiché.](./media/media-services-portal-passthrough-get-started/media-services-channel-created.png)

## <a name="watch-the-event"></a>Regarder l’événement
Pour visionner l’événement, cliquez sur **Watch** (Visionner) dans le portail Azure ou copiez l’URL de diffusion en continu et utilisez le lecteur de votre choix. 

![Date de création](./media/media-services-portal-passthrough-get-started/media-services-default-event.png)

Une fois arrêté, l’événement en direct est automatiquement converti en contenu à la demande.

## <a name="clean-up"></a>Nettoyer
Pour plus d’informations sur les canaux directs, consultez [Streaming en direct avec des encodeurs locaux qui créent des flux multidébits](media-services-live-streaming-with-onprem-encoders.md).

* Un canal peut être arrêté uniquement lorsque tous les événements/programmes du canal ont été arrêtés.  Une fois le canal arrêté, aucun frais n’est encouru. Lorsque vous devez le redémarrer, il possède la même URL de réception. Vous n’avez donc pas besoin de reconfigurer votre encodeur.
* Un canal peut être supprimé uniquement lorsque tous les événements/programmes du canal ont été supprimés.

## <a name="view-archived-content"></a>Afficher le contenu archivé
Même après l’arrêt et la suppression de l’événement, les utilisateurs pourront lire votre contenu archivé en tant que vidéo à la demande tant que vous n’aurez pas supprimé l’élément multimédia. Un élément multimédia ne peut pas être supprimé s’il est utilisé par un événement ; vous devez d’abord supprimer l’événement. 

Pour gérer vos éléments multimédias, cliquez sur **Paramètre** puis sur **Éléments multimédias**.

![Éléments multimédias](./media/media-services-portal-passthrough-get-started/media-services-assets.png)

## <a name="next-step"></a>Étape suivante
Consultez les parcours d’apprentissage de Media Services.

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
