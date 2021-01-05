---
title: Utiliser l’éditeur Video Indexer pour créer des projets
titleSuffix: Azure Media Services
description: Cette rubrique montre comment utiliser l’éditeur Video Indexer pour créer des projets.
services: media-services
author: Juliako
manager: femila
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 04/02/2019
ms.author: juliako
ms.openlocfilehash: 28186a7dcced47a42b3249f1f74b13e969b41978
ms.sourcegitcommit: 4f4a2b16ff3a76e5d39e3fcf295bca19cff43540
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93041364"
---
# <a name="use-the-video-indexer-editor-to-create-projects"></a>Utiliser l’éditeur Video Indexer pour créer des projets

Le site web Video Indexer vous permet d’utiliser des informations détaillées sur vos vidéos pour : trouver le contenu multimédia adapté, rechercher les passages qui vous intéressent et utiliser les résultats pour créer un tout nouveau projet. Une fois créé, le projet peut être restitué et téléchargé depuis Video Indexer, puis utilisé dans vos propres applications d’édition ou dans vos flux de travail en aval.

Voici quelques scénarios pour lesquels l’utilisation de cette fonctionnalité peut être pratique : 

* Création de moments forts du film pour les bandes-annonces.
* Utilisation d’anciens clips vidéos dans des bulletins d’informations.
* Création de contenus plus courts pour les réseaux sociaux.

Cet article explique comment créer un projet à partir de zéro, mais également comment créer un projet à partir d’une vidéo de votre compte.

## <a name="create-new-project-and-manage-videos"></a>Créer un projet et gérer des vidéos

1. Accédez au site web [Video Indexer](https://www.videoindexer.ai/) et connectez-vous.
1. Sélectionnez l’onglet **Projets**. Si vous avez créé des projets avant, vous les verrez tous à cet endroit.
1. Cliquez sur **Créer un projet**.  

    ![Capture d’écran montrant la page « Video Indexer » avec l’onglet « Projets » sélectionné.](./media/video-indexer-view-edit/new-project.png)
1. Nommez votre projet en cliquant sur l’icône de crayon. Remplacez le texte « Projet sans titre » par le nom de votre projet, puis cliquez sur la coche.

    ![Nouveau projet](./media/video-indexer-view-edit/new-project3.png)
    
### <a name="add-videos-to-the-project"></a>Ajouter des vidéos au projet

> [!NOTE]
> Actuellement, les projets peuvent uniquement contenir des vidéos indexées dans la même langue. Une fois que vous avez sélectionné une vidéo dans une langue, vous ne pouvez pas ajouter les vidéos de votre compte qui se trouvent dans une autre langue.

1. Ajoutez les vidéos que vous souhaitez utiliser dans ce projet en sélectionnant **Ajouter des vidéos**.

    Toutes les vidéos de votre compte s’afficheront, ainsi qu’une zone de recherche indiquant « Search for text, keywords, or visual content » (Recherche de texte, de mots-clés ou de contenus visuels). Pour rechercher des vidéos qui ont une personne, une étiquette, une marque, un mot-clé ou une occurrence spécifiés dans la transcription et l’OCR.
    
    Par exemple, dans l’image ci-dessous, nous recherchons des vidéos qui mentionnent « GitHub ».
    
    ![Capture d’écran montrant la recherche de vidéos qui mentionnent GitHub avec deux résultats.](./media/video-indexer-view-edit/github.png)

    Vous pouvez filtrer davantage vos résultats en sélectionnant **Filtrer les résultats**. Vous pouvez filtrer pour afficher des vidéos avec une personne spécifique ou pour indiquer que vous voulez uniquement voir des résultats de vidéos dans une langue donnée ou ayant un certain propriétaire. <br/> Vous pouvez également spécifier l’étendue de votre requête. Par exemple, si vous voulez rechercher « GitHub » dans l’OCR, sélectionnez **Texte visuel**.

    ![Filtrer](./media/video-indexer-view-edit/visual-text.png)

    Vous pouvez superposer plusieurs filtres dans votre requête. Utilisez les boutons **+** / **-** pour ajouter/supprimer des filtres. Utilisez **Effacer les filtres** pour supprimer tous les filtres.
1. Pour ajouter des vidéos, sélectionnez-les, puis cliquez sur **Ajouter**.
1. Vous voyez maintenant toutes les vidéos que vous avez choisies. Voici les vidéos à partir desquelles vous allez sélectionner des clips pour votre projet.

    Vous pouvez réorganiser l’ordre des vidéos en les glissant-déposant ou en sélectionnant le bouton de menu de liste et en cliquant sur **Descendre** ou **Monter**. Dans le menu de la liste, vous serez également en mesure de supprimer la vidéo de ce projet. 

    ![Capture d’écran montrant Video Indexer avec un menu contextuel pour l’une des vidéos à supprimer, effacer la sélection ou descendre.](./media/video-indexer-view-edit/rearrange.png)
    
    Vous avez la possibilité d’ajouter plus de vidéos à ce projet à tout moment en sélectionnant **Ajouter des vidéos**. Vous pouvez également ajouter plusieurs occurrences de la même vidéo à votre projet. Vous souhaiterez peut-être le faire si vous voulez diffuser un clip d’une vidéo, puis un clip à partir d’un autre vidéo, pour finir par un autre clip de la première vidéo. 

### <a name="select-clips-to-use-in-your-project"></a>Sélectionner des clips à utiliser dans votre projet

Si vous cliquez sur la flèche vers le bas dans la partie droite de chaque vidéo, vous allez ouvrir les informations dans la vidéo basées sur les horodatages (clips de la vidéo). 

1. Sélectionnez **Afficher les informations** pour personnaliser les informations que vous souhaitez voir ou non. 

    ![Afficher les informations](./media/video-indexer-view-edit/insights.png)
1. Pour créer des requêtes pour des clips spécifiques, utilisez la zone de recherche qui indique «Search in transcript, visual text, people, and labels» (Rechercher dans la transcription, le texte visuel, les personnes et les étiquettes).
1. Ajoutez des filtres pour affiner les détails sur les scènes que vous cherchez en sélectionnant **Options de filtre**.

    ![Options de filtre](./media/video-indexer-view-edit/filter-options.png)

    Par exemple, vous voulez peut-être voir des clips où GitHub est mentionné pendant que Donovan Brown est à l’écran. Pour ce faire, vous devez ajouter un filtre « inclure » avec la valeur « Personnes » en tant que type d’information. Vous devez ensuite saisir « Donovan Brown » dans la zone de recherche du filtre.
    
    ![Capture d’écran montrant Video Indexer avec Personnes sélectionné pour un filtre d’inclusion.](./media/video-indexer-view-edit/include.png)
    
    Si vous souhaitez des clips où GitHub est mentionné pendant que Donovan Brown n’est _pas_ à l’écran, il vous suffit de remplacer le filtre « inclure » par un filtre « exclure » à l’aide de la liste déroulante. 

1. Ajoutez un clip à votre projet en sélectionnant le segment que vous souhaitez inclure. Vous pouvez désélectionner ce clip en cliquant une nouvelle fois sur le segment.
    
    Ajoutez tous les segments d’une vidéo en cliquant sur l’option de menu de liste en regard de la vidéo et en sélectionnant **Sélectionner tous les segments**. 

    ![Tout ajouter](./media/video-indexer-view-edit/add-all.png)

    Vous pouvez effacer toute votre sélection en sélectionnant Effacer la sélection.

> [!TIP]
> À mesure que vous sélectionnez et organiser vos clips, vous pouvez afficher un aperçu de la vidéo dans le lecteur sur le côté droit de la page. 

![Capture d’écran montrant Video Indexer avec l’aperçu d’une vidéo sur le côté droit de la fenêtre.](./media/video-indexer-view-edit/preview.png)

Pensez à enregistrer votre projet lorsque vous apportez des modifications en sélectionnant **Enregistrer le projet**. 

### <a name="render-and-download-the-project"></a>Restituer et télécharger le projet

> [!NOTE]
> Pour les comptes payants Video Indexer, la restitution de votre projet implique des coûts d’encodage. Les comptes d’essai gratuit Video Indexer sont limités à une restitution de 5 heures.

1. Une fois que vous avez terminé, assurez-vous que votre projet a été enregistré. Vous pouvez maintenant restituer ce projet. Sélectionnez **Rendu et téléchargement**. 

    ![Capture d’écran montrant Video Indexer avec l’option permettant d’afficher et de télécharger votre projet.](./media/video-indexer-view-edit/save.png)

    Une fenêtre contextuelle vous indiquera que Video Indexer va restituer un fichier, puis le lien de téléchargement sera envoyé à votre adresse e-mail. Sélectionnez Continuer. 
    
    Vous verrez également une notification en haut de la page indiquant que le projet est en cours de restitution. Une fois cette opération terminée, une nouvelle notification vous en informera. Cliquez sur la notification pour télécharger le projet. Le projet sera téléchargé au format mp4.

    ![Restitution terminée](./media/video-indexer-view-edit/rendering-done.png)

1. Vous pouvez accéder à des projets enregistrés depuis l’onglet **Projets**. 

    Si vous sélectionnez ce projet, vous voyez toutes les informations et la chronologie de ce projet. Si vous sélectionnez **Éditeur de vidéo** , vous pouvez continuer à apporter des modifications à ce projet. Les modifications incluent l’ajout ou la suppression de vidéos et de clips ou le renommage du projet.

    ![Éditeur de vidéo](./media/video-indexer-view-edit/video-editor.png)
     
## <a name="create-a-project-from-your-video"></a>Créer un projet à partir de votre vidéo

Vous pouvez créer un projet directement à partir d’une vidéo dans votre compte. 

1. Accédez à l’onglet **Bibliothèque** du site web de Video Indexer.
1. Ouvrez la vidéo que vous souhaitez utiliser pour créer votre projet. Sur la page des informations et de la chronologie, sélectionnez le bouton **Éditeur de vidéo**.

    Vous êtes redirigé vers la même page que celle permettant de créer un projet. Contrairement au nouveau projet, vous voyez les segments d’informations horodatés de la vidéo que vous aviez commencé à modifier précédemment.

## <a name="see-also"></a>Voir aussi

[Présentation de Video Indexer](video-indexer-overview.md)

