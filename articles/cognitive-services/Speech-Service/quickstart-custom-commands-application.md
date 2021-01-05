---
title: 'Démarrage rapide : Créer un assistant vocal à l’aide de commandes personnalisées'
titleSuffix: Azure Cognitive Services
description: Dans ce guide de démarrage rapide, vous créez et testez une application Commandes personnalisées de base à l’aide de Speech Studio.
services: cognitive-services
author: singhsaumya
manager: yetian
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 06/18/2020
ms.author: sausin
ms.custom: references_regions
ms.openlocfilehash: c5f8be87e7ee95e1deff12cce09204b5fe8a0bb7
ms.sourcegitcommit: b6267bc931ef1a4bd33d67ba76895e14b9d0c661
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/19/2020
ms.locfileid: "97694193"
---
# <a name="create-a-voice-assistant-using-custom-commands"></a>Créer un assistant vocal à l’aide de commandes personnalisées

Dans ce guide de démarrage rapide, vous créez et testez une application Commandes personnalisées de base à l’aide de Speech Studio. Vous pouvez également accéder à cette application depuis une application cliente Windows. Les **commandes personnalisées** facilitent la création d’applications de commandes vocales complètes, optimisées pour les expériences d’interaction de type « voice-first ». Elles offrent une expérience de création unifiée, un modèle d’hébergement automatique et une complexité relativement inférieure, vous permettant de vous concentrer sur la conception de la meilleure solution pour vos scénarios de commandes vocales.

## <a name="region-availability"></a>Disponibilité dans les régions
À ce stade, les commandes personnalisées prennent en charge les abonnements vocaux créés dans ces régions :
* USA Ouest
* USA Ouest 2
* USA Est
* USA Est 2
* Centre-USA Ouest
* Europe Nord
* Europe Ouest
* Asie Est
* Asie Sud-Est
* Inde centrale

## <a name="prerequisites"></a>Prérequis

> [!div class="checklist"]
> * <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesSpeechServices" target="_blank">Créez une ressource Azure Speech dans une région qui prend en charge les commandes personnalisées.<span class="docon docon-navigate-external x-hidden-focus"></span></a> Pour obtenir la liste des régions prises en charge, reportez-vous à la section **Disponibilité des régions** ci-dessus.
> * Téléchargez l’exemple de fichier json [Smart Room Lite](https://aka.ms/speech/cc-quickstart).
> * Téléchargez la dernière version du [client Assistant vocal Windows](https://aka.ms/speech/va-samples-wvac).

## <a name="go-to-the-speech-studio-for-custom-commands"></a>Accédez à Speech Studio pour les commandes personnalisées

1. Dans un navigateur web, accédez à [Speech Studio](https://speech.microsoft.com/).
1. Entrez vos informations d’identification pour vous connecter au portail.

   L'affichage par défaut est votre liste d'abonnements Speech.
   > [!NOTE]
   > Si vous ne voyez pas la page Sélectionner un abonnement, vous pouvez y accéder en choisissant « Ressources Speech » dans le menu Paramètres sur la barre supérieure.

1. Sélectionnez votre abonnement Speech, puis **Accéder à Studio**.
1. Sélectionnez **Commandes personnalisées**.

   L'affichage par défaut présente liste des applications Commandes personnalisées disponibles sous l'abonnement sélectionné.

## <a name="import-an-existing-application-as-a-new-custom-commands-project"></a>Importer une application existante en tant que nouveau projet Commandes personnalisées

1. Sélectionnez **Nouveau projet** pour créer un projet.

1. Dans la zone **Nom**, entrez le nom de projet `Smart-Room-Lite` (ou le nom de votre choix).
1. Dans la liste **Langue**, sélectionnez **Français - France**.
1. Sélectionnez **Parcourir les fichiers** puis, dans la fenêtre, sélectionnez le fichier **SmartRoomLite.json**.

    > [!div class="mx-imgBorder"]
    > ![Créer un projet](media/custom-commands/import-project.png)

1.  Dans la liste **Ressource de création LUIS**, sélectionnez une ressource de création. En l’absence de ressources de création valides, créez-en une en sélectionnant **Créer une ressource de création LUIS**.

    > [!div class="mx-imgBorder"]
    > ![Créer un groupe de ressources](media/custom-commands/create-new-luis-resource.png)
    
    
    1. Dans la zone **Nom de la ressource**, entrez le nom de la ressource.
    1. Dans la liste **Groupe de ressources**, sélectionnez un groupe de ressources.
    1. Dans la liste **Localisation**, sélectionnez une localisation.
    1. Dans la liste **Niveau tarifaire**, sélectionnez un niveau.
    
    
    > [!NOTE]
    > Vous pouvez créer des groupes de ressources en entrant le nom du groupe de ressources souhaité dans le champ « Groupe de ressources ». Sélectionnez **Créer** pour créer le groupe de ressources.


1. Puis sélectionnez **Créer** pour créer le projet.
1. Une fois le projet créé, sélectionnez-le.
Vous devez à présent voir une vue d’ensemble de votre nouvelle application Commandes personnalisées.

## <a name="try-out-some-voice-commands"></a>Tester quelques commandes vocales
1. Sélectionnez **Entraîner** en haut du volet droit.
1. Une fois l’apprentissage terminé, sélectionnez **Tester** et essayez les énoncés suivants :
    - Allume la télévision
    - Régler la température sur 80 degrés
    - Éteindre la télévision
    - La télévision
    - Définir une alarme pour 17h

## <a name="integrate-custom-commands-application-in-an-assistant"></a>Intégrer une application Commandes personnalisées dans un assistant
Pour accéder à cette application en dehors de Speech Studio, vous devez la publier. Pour publier une application, vous devrez configurer la ressource LUIS de prédiction.  

### <a name="update-prediction-luis-resource"></a>Mettre à jour la ressource LUIS de prédiction


1. Sélectionnez **Paramètres** dans le volet gauche, puis sélectionnez **Ressources LUIS** dans le volet central.
1. Sélectionnez une ressource de prédiction, ou créez-en une en sélectionnant **Créer une ressource**.
1. Sélectionnez **Enregistrer**.
    
    > [!div class="mx-imgBorder"]
    > ![Définir les ressources LUIS](media/custom-commands/set-luis-resources.png)

> [!NOTE]
> Comme la ressource de création ne prend en charge que 1 000 requêtes de point de terminaison de prédiction par mois, vous devez obligatoirement définir une ressource de prédiction LUIS avant de publier votre application Commandes personnalisées.

### <a name="publish-the-application"></a>Publication de l'application

Sélectionnez **Publier** en haut du volet. Une fois la publication terminée, une nouvelle fenêtre s’affiche. Notez l’**ID de l’application** et sa valeur **Clé de la ressource Speech**. Vous aurez besoin de ces deux valeurs pour accéder à l’application en dehors de Speech Studio.

Vous pouvez également obtenir ces valeurs en sélectionnant la section **Paramètres** > **Général**.

### <a name="access-application-from-client"></a>Accéder à une application à partir du client

Dans le cadre de cet article, nous utiliserons le client de l’assistant vocal Windows que vous avez téléchargé conformément aux conditions préalables. Décompressez le dossier.
1. Lancez **VoiceAssistantClient.exe**.
1. Créez un profil de publication et entrez une valeur pour **Profil de connexion**. Dans la section **Paramètres généraux**, entrez des valeurs pour **Clé d’abonnement** (la même valeur que la **Clé de ressource Speech** enregistrée lors de la publication de l’application), **Région de la clé d’abonnement** et **ID d’application Commandes personnalisées**.
    > [!div class="mx-imgBorder"]
    > ![Capture d’écran mettant en évidence la section Paramètres généraux pour la création d’un profil WVAC.](media/custom-commands/create-profile.png)
1. Sélectionnez **Save and Apply Profile** (Enregistrer et appliquer le profil).
1. Testez maintenant les entrées suivantes en mode vocal/texte
    > [!div class="mx-imgBorder"]
    > ![Création d’un profil WVAC](media/custom-commands/conversation.png)


> [!TIP]
> Vous pouvez cliquer sur les entrées du **journal d'activité** pour examiner les réponses brutes envoyées par le service Commandes personnalisées.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez utilisé une application existante. Ensuite, dans les [sections pratiques](how-to-custom-commands-create-application-with-simple-commands.md), vous apprendrez à concevoir, développer, déboguer, tester et intégrer une application Commandes personnalisées à partir de zéro.
