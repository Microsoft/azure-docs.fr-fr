---
title: Stocker les événements Azure Media Services dans Azure Log Analytics
titleSuffix: Azure Media Services
description: Découvrez comment stocker des événements de Azure Media Services dans Azure Log Analytics.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: tutorial
ms.date: 08/24/2020
ms.author: inhenkel
ms.openlocfilehash: cc3060c9253b23b97089ea35625aceb26737baba
ms.sourcegitcommit: 2c586a0fbec6968205f3dc2af20e89e01f1b74b5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92016993"
---
# <a name="tutorial-store-azure-media-services-events-in-azure-log-analytics"></a>Tutoriel : Stocker les événements Azure Media Services dans Azure Log Analytics

## <a name="azure-media-services-events"></a>Événements Azure Media Services

Azure Media Services V3 émet des événements sur [Azure Event Grid](media-services-event-schemas.md). Vous pouvez vous abonner à des événements de nombreuses façons et les stocker dans des magasins de données. Dans ce didacticiel, vous allez vous abonner aux événements Media Services à l’aide d’un [Flux d’application logique](https://azure.microsoft.com/services/logic-apps/). L’application logique est déclenchée pour chaque événement et stocke le corps de l’événement dans Azure Log Analytics. Une fois que les événements se trouvent dans Azure Log Analytics, vous pouvez utiliser d’autres services Azure pour créer un tableau de bord, suivre et générer des alertes sur ces événements. Toutefois, nous ne couvrirons pas cela dans ce didacticiel.

> [!NOTE]
> Il pourrait être utile d’être déjà familiarisé avec l’utilisation de FFmpeg comme encodeur local.  Si ce n’est pas le cas, ne vous inquiétez pas. La ligne de commande et les instructions pour diffuser une vidéo sont incluses ci-dessous.

Vous apprendrez à :

> [!div class="checklist"]
> * Créer un flux d’application logique sans code
> * S’abonner à des rubriques d’événements Azure Media Services
> * Analyser les événements et les stocker dans Azure Log Analytics
> * Interroger des événements à partir d’Azure Log Analytics

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

> * Un [abonnement Azure](how-to-set-azure-subscription.md)
> * Un compte [Media Services](create-account-howto.md) et un groupe de ressources.
> * Une installation de [FFmpeg](https://ffmpeg.org/download.html) pour votre système d’exploitation.
> * Un espace de travail [Log Analytics](../../azure-monitor/learn/quick-create-workspace.md)

## <a name="subscribe-to-a-media-services-event-with-logic-app"></a>S’abonner à un événement Media Services avec une application logique

1. Dans le portail Azure, si vous ne l’avez pas déjà fait, créez un espace de travail [Log Analytics](../../azure-monitor/learn/quick-create-workspace.md). Vous aurez besoin de l’ID de l’espace de travail et de l’une des clés, donc gardez cette fenêtre de navigateur ouverte. Ensuite, ouvrez le portail dans un autre onglet ou une autre fenêtre.

1. Accédez à votre compte Azure Media Services et sélectionnez **Événements**. Cette opération affiche toutes les méthodes permettant de s’abonner à des événements Azure Media Services.
    > [!div class="mx-imgBorder"]
    > ![Portail Azure Media Services](media/tutorial-events-log-analytics/select-events-01a.png)

1. Sélectionnez l’**icône Logic Apps** pour créer une application logique. Le concepteur d’application logique s’ouvre ; vous pouvez y créer le flux pour capturer les événements et les envoyer à Log Analytics. 
    > [!div class="mx-imgBorder"]
    > ![Créer une application logique](media/tutorial-events-log-analytics/select-logic-app-02.png)

1. Sélectionnez l’**icône +** , sélectionnez le locataire que vous souhaitez utiliser, puis sélectionnez Se connecter. Une invite de connexion Microsoft s’affiche.
    > [!div class="mx-imgBorder"]
    > ![Connectez-vous à Azure Event Grid](media/tutorial-events-log-analytics/select-event-add-grid-03.png)
![Sélectionnez le locataire](media/tutorial-events-log-analytics/select-tenant-03a.png)

1. Sélectionnez **Continuer** pour vous abonner aux événements Media Services.
    > [!div class="mx-imgBorder"]
    > ![Connecté à Azure Event Grid](media/tutorial-events-log-analytics/select-continue-04.png)

1. Dans la liste **Type de ressource**, recherchez « Microsoft.Media.MediaServices ».
    > [!div class="mx-imgBorder"]
    >![Événements de ressource Azure Media Services](media/tutorial-events-log-analytics/locate-azure-media-services-events-05.png)

1. Sélectionnez l’**élément Type d’événement**. Vous verrez une liste de tous les événements générés par Azure Media Services. Vous pouvez sélectionner les événements que vous souhaitez suivre. Vous pouvez ajouter plusieurs types d’événements. (Par la suite, vous allez apporter une petite modification au flux de l’application logique pour stocker chaque type d’événement dans un journal Log Analytics séparé et propager le nom du type d’événement vers le nom du journal Log Analytics de manière dynamique.)
    > [!div class="mx-imgBorder"]
    > ![Type d’événement Azure Media Services](media/tutorial-events-log-analytics/select-azure-media-services-event-type-06.png)

1. Sélectionnez **Enregistrer sous**.

1. Donnez un nom à votre application logique.  Le groupe de ressources est sélectionné par défaut. Laissez les autres paramètres tels qu’ils sont, puis sélectionnez **Créer**.  Vous êtes redirigé vers l’écran d’accueil Azure.
    > [!div class="mx-imgBorder"]
    > ![Interface de dénomination d’application logique](media/tutorial-events-log-analytics/give-logic-app-name-06a.png)

## <a name="create-an-action"></a>Créer une action

Maintenant que vous êtes abonné aux événements de votre choix, créez une action.

1. Si le portail vous renvoie à l’écran d’accueil, revenez à l’application logique que vous venez de créer en recherchant le nom de l’application dans Toutes les ressources.

1. Sélectionnez votre application, puis **Concepteur d’application logique**. Le volet du concepteur s’ouvre.

1. Sélectionnez **+ Nouvelle étape**.

1. Comme vous souhaitez envoyer les événements au service Azure Log Analytics, recherchez « Azure Log Analytics », puis sélectionnez « Azure Log Analytics Data Collector ».
    > [!div class="mx-imgBorder"]
    > ![Azure Log Analytics Data Collector](media/tutorial-events-log-analytics/select-azure-log-analytics-data-collector-07.png)

1. Pour vous connecter à l’espace de travail Log Analytics, vous avez besoin de l’ID de l’espace de travail et d’une clé d’agent. Ouvrez le portail Azure dans un nouvel onglet ou une nouvelle fenêtre, et accédez à l’espace de travail Log Analytics que vous avez créé avant le début de ce didacticiel.
    > [!div class="mx-imgBorder"]
    > ![ID d’espace de travail Azure Log Analytics](media/tutorial-events-log-analytics/log-analytics-workspace-id-08.png)

1. Dans le menu de gauche, recherchez **Gestion des agents** et sélectionnez-le. Cela vous montrera les clés d’agent qui ont été générées.
    > [!div class="mx-imgBorder"]
    > ![Gestion des agents dans Azure Log Analytics](media/tutorial-events-log-analytics/select-agents-management-09.png)

1. Copiez l’*ID de l’espace de travail*.
    > [!div class="mx-imgBorder"]
    > ![Copier l’ID de l’espace de travail](media/tutorial-events-log-analytics/copy-workspace-id.png)

1. Dans l’autre onglet ou fenêtre du navigateur, sous Azure Log Analytics Data Collector, sélectionnez **Envoyer des données**, attribuez un nom à votre connexion, puis collez l’*ID d’espace de travail* dans le champ **ID d’espace de travail**.

1. Revenez à l’onglet ou à la fenêtre du navigateur de l’espace de travail et copiez la *Clé d’espace de travail*.
    > [!div class="mx-imgBorder"]
    > ![Clé primaire de gestion des agents](media/tutorial-events-log-analytics/agents-management-primary-key-10.png)

1. Dans l’autre onglet ou fenêtre du navigateur, collez la *Clé de l’espace de travail* dans le champ **Clé de l'espace de travail**.

1. Sélectionnez **Create** (Créer). Vous allez maintenant créer le corps de la requête JSON et le nom du journal personnalisé.

1. Sélectionnez le champ **Corps de la requête JSON**.  Un lien vers **Ajouter du contenu dynamique** s’affiche.

1. Sélectionnez **Ajouter du contenu dynamique**, puis **Rubrique**.

1. Procédez de même pour **Nom de journal personnalisé**.
    > [!div class="mx-imgBorder"]
    > ![Rubrique sélectionnée](media/tutorial-events-log-analytics/topic-selected.png)

1. Sélectionnez le **Mode Code** de l’application logique. Recherchez les entrées et les lignes de type journal.
    > [!div class="mx-imgBorder"]
    > ![Mode Code de deux lignes](media/tutorial-events-log-analytics/code-view-two-lines.png)

1. Modifiez la valeur `body` de `"@triggerBody()?['topic']"` à `"@{triggerBody()}"`. Cela permet d’analyser la totalité du message sur Log Analytics.

1. Changez `Log-Type` de `"@triggerBody()?['topic']"` en `"@replace(triggerBody()?['eventType'],'.','')"`. (Cela remplacera « . », car le point n’est pas autorisé dans les noms de journaux Log Analytics.)
    > [!div class="mx-imgBorder"]
    > ![JSON de l’application logique après modification](media/tutorial-events-log-analytics/changed-lines.png)

1. Sélectionnez **Enregistrer**.

1. Pour vérifier, sélectionnez **Concepteur d’application logique**.
    > [!div class="mx-imgBorder"]
    > ![Vérifier les étapes Corps et Fonction](media/tutorial-events-log-analytics/verify-changes-to-json.png)

1. Lorsque vous examinez toutes les ressources du groupe de ressources, il y aura une application logique et deux connecteurs d’API d’application logique répertoriés, un pour les événements et l’autre pour Log Analytics. Pour plus d’informations sur les rubriques système Event Grid, consultez [Rubriques système Event Grid](../../event-grid/system-topics.md).
    > [!div class="mx-imgBorder"]
    > ![Afficher toutes les nouvelles ressources dans un groupe de ressources](media/tutorial-events-log-analytics/contoso-rg-listing.png)

## <a name="test"></a>Test

Pour tester le fonctionnement réel, créez un événement en direct dans Azure Media Services. Créez un événement en direct RTMP et utilisez ffmpeg pour envoyer un flux « en direct » basé sur un exemple de fichier .mp4. Une fois l’événement créé, récupérez l’URL de réception RTMP.

1. À partir de votre compte Media Services, sélectionnez **Streaming en direct**.
    > [!div class="mx-imgBorder"]
    > ![Créer un événement en direct Azure Media Services](media/tutorial-events-log-analytics/live-event.png)

1. Sélectionnez **Ajouter un événement en direct**.

1. Entrez un nom dans le champ **Nom de l’événement en direct**. (Le champ **Description** est facultatif.)

1. Sélectionnez l’encodage cloud **Standard**.

1. Sélectionnez **720p par défaut** pour l’encodage prédéfini.

1. Sélectionnez le protocole d’entrée **RTMP**.

1. Sélectionnez **Oui** pour l’URL d’entrée permanente.

1. Sélectionnez **Oui** pour démarrer l’événement lorsque l’événement est créé.

    > [!WARNING]
    > La facturation démarre si vous sélectionnez Oui.  Si vous souhaitez attendre jusqu’à *juste avant* de commencer la diffusion avec FFmpeg avant de démarrer le flux, sélectionnez **Non** et n’oubliez pas de démarrer votre événement en direct en temps voulu.

    > [!div class="mx-imgBorder"]
    > ![Paramètres des événements en direct](media/tutorial-events-log-analytics/live-event-settings.png)

1. Cochez la case **J'ai tous les droits nécessaires pour utiliser le contenu ou le fichier...** .

1. Sélectionnez **Revoir + créer**.

1. Passez en revue vos paramètres, puis sélectionnez **Créer**.  La liste des événements en direct s’affiche et l’URL de réception de l’événement en direct s’affiche.

1. Copiez l’**URL de réception** dans votre Presse-papiers.

1. Sélectionnez **événement en direct** dans la liste pour afficher la vue Producteur.

### <a name="stream-with-ffmpeg-cli"></a>Diffuser avec l’interface CLI FFmpeg

1. Utilisez la ligne de commande suivante.

    ```AzureCLI
    ffmpeg -i <localpathtovideo> -map 0 -c:v libx264 -c:a copy -f flv <ingestURL>/mystream
    ```

1. Remplacez `<ingestURL>` par l’URL de réception que vous avez copiée dans le presse-papiers.
1. Remplacez `<localpathtovideo>` par le chemin d’accès local du fichier que vous souhaitez diffuser à partir de FFmpeg.
1. Ajoutez un nom unique à la fin, par exemple `mystream`.
1. Ajustez la ligne de commande pour refléter votre fichier source de test et toute autre variable système.
1. Exécutez la commande. Au bout de quelques secondes, le lecteur en « Vue Producteur » devrait démarrer la diffusion en continu. (Actualisez le lecteur si la vidéo ne s’affiche pas automatiquement.)

    > [!div class="mx-imgBorder"]
    > ![Vérifier la bonne réception vidéo dans le lecteur Aperçu Producteur](media/tutorial-events-log-analytics/live-event-producer-view.png)

## <a name="verify-the-events"></a>Vérifier les événements

Avec le stream en direct, Azure Media Services émet différents événements qui déclenchent le flux de l’application logique. Pour vérifier cela, accédez à l’application logique et déterminez si des déclencheurs sont activés par les événements à partir de Media Services.

1. Accédez à la page Vue d’ensemble de l’application logique, qui répertorie les tâches qui se sont terminées avec succès.
    > [!div class="mx-imgBorder"]
    > ![Vérifier la réussite de l’exécution de la tâche dans l’application logique](media/tutorial-events-log-analytics/run-history.png)

1. Sélectionnez une tâche réussie. Les détails de la tâche au cours de l’exécution s’affichent.
1. Sélectionnez **Envoyer des données** pour les développer. Dans ce cas, l’événement `MicrosoftMediaLiveEventEncoderConnected` indique qu’il a été capturé, avec le corps analysé. C’est ce qui est envoyé à l’espace de travail Azure Log Analytics.
    > [!div class="mx-imgBorder"]
    > ![Voir les données envoyées](media/tutorial-events-log-analytics/verify-send-data.png)

## <a name="verify-the-logs"></a>Vérifier les journaux

1. Accédez à l’espace de travail Log Analytics que vous avez créé précédemment.

1. Sélectionnez **Journaux d’activité**.
1. Fermez la fenêtre Exemples de requêtes.
1. Il y aura une liste des journaux personnalisés. Sélectionnez la flèche vers le bas pour la développer. Vous verrez le nom de l’événement `MicrosoftMediaLiveEventEncoderConnected`.
1. Sélectionnez le nom de l’événement pour le développer.
1. Lorsque vous sélectionnez l’icône « œil », un aperçu du résultat de la requête s’affiche.
1. Sélectionnez **Afficher dans l'éditeur de requête**, puis sélectionnez l’élément sous l’entrée **TimeGenerated UTC** pour la développer et afficher les données brutes.

![Voir la sortie détaillée des événements dans Log Analytics](media/tutorial-events-log-analytics/raw-data.png)

## <a name="delete-resources"></a>Supprimer des ressources

Si vous ne souhaitez pas continuer à utiliser les ressources que vous avez créées au cours de ce didacticiel, veillez à supprimer toutes les ressources du groupe de ressources, sans quoi vous continuerez à être facturé.

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez créer des requêtes différentes et les enregistrer. Celles-ci peuvent être ajoutées au [tableau de bord Azure](../../azure-monitor/learn/tutorial-logs-dashboards.md).