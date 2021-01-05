---
title: 'Tutoriel : Intégration à Power Virtual Agents – QnA Maker'
description: Dans ce tutoriel, améliorez la qualité de votre base de connaissances avec l’apprentissage actif. Examinez, acceptez ou refusez, ou ajoutez sans supprimer ou changer les questions existantes.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: tutorial
ms.date: 11/09/2020
ms.openlocfilehash: 3801bb44fed6bf24788957c41de77c89b3025ae8
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96351110"
---
# <a name="tutorial-add-your-knowledge-base-to-power-virtual-agents"></a>Tutoriel : Ajouter votre base de connaissances à Power Virtual Agents
Créez et étendez un bot [Power Virtual Agents](https://powervirtualagents.microsoft.com/) pour fournir des réponses à partir de votre base de connaissances.

Dans ce tutoriel, vous allez apprendre à :

<!-- green checkmark -->
> [!div class="checklist"]
> * Créer un bot Power Virtual Agents
> * Créer une rubrique de secours du système
> * Ajouter QnA Maker comme action à une rubrique sous forme de flux Power Automate
> * Créer une solution Power Automate
> * Ajouter un flux Power Automate à votre solution
> * Publier les agents Power Virtual Agents
> * Tester les agents Power Virtual Agents et recevoir une réponse de votre base de connaissances QnA Maker

## <a name="integrate-an-agent-with-a-knowledge-base"></a>Intégrer un agent à une base de connaissances

[Power Virtual Agents](https://powervirtualagents.microsoft.com/) permet aux équipes de créer des bots puissants à l’aide d’une interface graphique guidée sans code. Vous n’avez pas besoin de développeurs ni de scientifiques des données.

Dans Power Virtual Agents, vous créez un agent avec une série de rubriques (domaines) de façon à répondre à des questions d’utilisateurs en effectuant des actions. En l’absence de réponse, une rubrique de secours du système peut en retourner une.

Configurez l’agent de telle sorte qu’il envoie la question à votre base de connaissances dans une action de rubrique ou dans le chemin de rubrique de *secours du système*. Les deux utilisent une action pour se connecter à votre base de connaissances et retourner une réponse.

## <a name="power-automate-connects-to-generateanswer-action"></a>Power Automate se connecte à l’action `GenerateAnswer`

Pour connecter votre agent à votre base de connaissances, utilisez Power Automate pour créer l’action. Power Automate fournit un flux de processus, qui se connecte à l’API `GenerateAnswer` de QnA Maker.

Une fois le flux conçu et enregistré, il est accessible à partir d’une solution Power Automate. Utilisez cette solution en tant qu’action dans votre agent.

## <a name="connect-an-agent-to-your-knowledge-base"></a>Connecter un agent à votre base de connaissances

Voici une vue d’ensemble des étapes permettant de connecter un agent dans Power Virtual Agents à une base de connaissances dans QnA Maker.

* Sur le portail [QnA Maker](https://www.qnamaker.ai/) :
    * Créez et publiez votre base de connaissances.
    * Copiez les détails de votre base de connaissances, notamment l’ID, la clé du point de terminaison du runtime et l’hôte du point de terminaison du runtime.
* Sur le portail [Power Virtual Agents](https://powerva.microsoft.com/) :
    * Créez une rubrique d’agent.
    * Appelez une action (du flux Power Automate).
* Sur le portail [Power Automate](https://us.flow.microsoft.com/) :
    * Recherchez le modèle _Generate answer using QnA Maker_ (Générer une réponse à l’aide de QnA Maker).
    * Utilisez le modèle pour configurer le flux afin qu’il utilise le modèle [Générer une réponse de QnA Maker](/connectors/cognitiveservicesqnamaker/).
        * Informations sur la base de connaissances publiée QnA Maker :
            * ID de base de connaissances
            * Hôte du point de terminaison de ressource QnA Maker
            * Clé du point de terminaison de ressource QnA Maker
        * Entrée – Requête utilisateur
        * Sortie – Réponse de la base de connaissances
    * Créez une solution et ajoutez le flux, ou ajoutez un flux à une solution existante.
* Revenez à Power Virtual Agents :
    * sélectionnez la sortie de la solution sous forme de message pour une rubrique.

## <a name="create-and-publish-a-knowledge-base"></a>Créer et publier une base de connaissances

1. Suivez le guide de [démarrage rapide](../Quickstarts/create-publish-knowledge-base.md) pour créer une base de connaissances. N’effectuez pas la dernière section relative à la création d’un bot. Utilisez plutôt ce tutoriel pour créer un bot avec Power Virtual Agents.

    > [!div class="mx-imgBorder"]
    > ![Capture d’écran des paramètres de la base de connaissances publiée](../media/how-to-integrate-power-virtual-agent/published-knowledge-base-settings.png)

    Entrez les paramètres de la base de données que vous avez publiée et qui se trouvent dans la page **Settings** (Paramètres) du portail [QnA Maker](https://www.qnamaker.ai/). Vous aurez besoin de ces information à l’[étape Power Automate](#create-a-power-automate-flow-to-connect-to-your-knowledge-base) pour configurer votre connexion à QnA Maker `GenerateAnswer`.

1. Sur le portail QnA Maker, dans la page **Settings** (Paramètres), recherchez la clé du point de terminaison, l’hôte du point de terminaison et l’ID de la base de connaissances.

## <a name="create-an-agent-in-power-virtual-agents"></a>Créer un agent dans Power Virtual Agents

1. [Connectez-vous à Power Virtual Agents](https://go.microsoft.com/fwlink/?LinkId=2108000&clcid=0x409). Utilisez votre compte de messagerie scolaire ou professionnel.

1. S’il s’agit de votre premier bot, vous accédez à la page **Home** (Accueil) de l’agent. Si ce n’est pas votre premier bot, sélectionnez le bot dans la zone supérieure droite de la page, puis sélectionnez **+ New Bot** (Nouveau bot).

    > [!div class="mx-imgBorder"]
    > ![Capture d’écran de la page d’accueil de Power Virtual Agents](../media/how-to-integrate-power-virtual-agent/power-virtual-agent-home.png)

## <a name="topics-provided-in-the-bot"></a>Rubriques fournies dans le bot

L’agent de sert de la collection de rubriques pour répondre aux questions relevant de votre domaine. Dans ce tutoriel, l’agent propose de nombreuses rubriques, réparties en rubriques utilisateur et rubriques système.

Sélectionnez **Topics** (Rubriques) dans le volet de navigation gauche pour afficher les rubriques fournies par le bot.

> [!div class="mx-imgBorder"]
> ![Capture d’écran des rubriques fournies dans l’agent](../media/how-to-integrate-power-virtual-agent/power-virtual-agent-topics-provided.png)


## <a name="create-the-system-fallback-topic"></a>Créer la rubrique de secours du système

Bien que l’agent puisse se connecter à votre base de connaissances à partir de n’importe quelle rubrique, ce tutoriel utilise la rubrique *system fallback* (secours du système). La rubrique de secours est utilisée quand l’agent ne trouve pas de réponse. L’agent transmet le texte de l’utilisateur à l’API `GenerateAnswer` de QnA Maker, reçoit la réponse de votre base de connaissances et la présente à l’utilisateur sous forme de message.

1. Sur le portail [Power Virtual Agents](https://powerva.microsoft.com/#/), en haut à droite, sélectionnez **Settings** (Paramètres) (icône d’engrenage). Ensuite, sélectionnez **System Fallback** (Secours du système).

    > [!div class="mx-imgBorder"]
    > ![Capture d’écran de l’élément de menu Power Virtual Agents pour la rubrique de secours du système](../media/how-to-integrate-power-virtual-agent/power-virtual-agent-settings-system-fallback.png)

1. Sélectionnez **+ Add** (Ajouter) pour ajouter une rubrique de secours du système.

    > [!div class="mx-imgBorder"]
    > ![Capture d’écran de l’ajout d’une rubrique de secours.](../media/how-to-integrate-power-virtual-agent/power-virtual-agent-settings-add-fallback-topic.png)

1. Une fois la rubrique ajoutée, sélectionnez **Go to Fallback topic** (Accéder à la rubrique de secours) pour créer la rubrique de secours sur le canevas de création.

    > [!TIP]
    > Si vous avez besoin de retourner à la rubrique de secours, celle-ci se trouve dans la section **Topics** (Rubriques), dans la rubrique **System** (Système).

## <a name="use-the-authoring-canvas-to-add-an-action"></a>Utiliser le canevas de création pour ajouter une action

Utilisez le canevas de création Power Virtual Agents pour connecter la rubrique de secours à votre base de connaissances. La rubrique commence par le texte utilisateur non reconnu. Ajoutez une action qui transmet ce texte à QnA Maker, puis qui affiche la réponse sous forme de message. La dernière étape d’affichage de la réponse est traitée dans une [étape distincte](#add-your-solutions-flow-to-power-virtual-agents), plus loin dans ce tutoriel.

Cette section vise à créer le flux de conversation de la rubrique de secours.

1. La nouvelle action de secours peut déjà comporter des éléments de flux de conversation. Supprimez l’élément **Escalate** (Réaffecter) en sélectionnant le menu **Options**.

    :::image type="content" source="../media/how-to-integrate-power-virtual-agent/delete-escalate-action-using-option-menu.png" alt-text="Capture d’écran partielle du flux de conversation, avec l’option de suppression mise en surbrillance.":::

1. Sélectionnez le connecteur **+** pointant vers la zone **Message**, puis sélectionnez **Call an action** (Appeler une action).

    :::image type="content" source="../media/how-to-integrate-power-virtual-agent/create-new-item-call-an-action.png" alt-text="Capture d’écran partielle de l’option d’appel d’une action":::

1. Sélectionnez **Create a flow** (Créer un flux). Le processus vous amène au portail Power Automate.

    > [!div class="mx-imgBorder"]
    > ![Capture d’écran de l’option de création d’un flux](../media/how-to-integrate-power-virtual-agent/create-a-flow.png)


    Power Automate ouvre un nouveau modèle. Vous n’allez pas utiliser ce nouveau modèle.

    :::image type="content" source="../media/how-to-integrate-power-virtual-agent/power-automate-flow-initial-template.png" alt-text="Capture d’écran partielle de Power Automate avec le nouveau modèle de flux.":::

## <a name="create-a-power-automate-flow-to-connect-to-your-knowledge-base"></a>Créer un flux Power Automate à connecter à votre base de connaissances

> [!NOTE]
> Le modèle Power Automate ne prend pas en charge les points de terminaison QnA Maker managé (préversion). Pour ajouter une base de connaissances QnA Maker managé (préversion) à Power Automate, vous devez ignorer cette étape et y ajouter manuellement les points de terminaison. 

La procédure suivante crée un flux Power Automate qui effectue les opérations suivantes :
* Il intercepte le texte utilisateur entrant et l’envoie à QnA Maker.
* Il retourne la meilleure réponse à votre agent.

1. Dans **Power Automate**, sélectionnez **Modèles** dans le volet de navigation gauche. Si vous êtes invité à quitter la page du navigateur, acceptez la valeur Leave (Quitter).

1. Dans la page Modèles, recherchez le modèle **Generate answer using QnA Maker (Générer une réponse à l’aide de QnA Maker)** , puis sélectionnez le modèle. Ce modèle comporte toutes les étapes nécessaires pour appeler QnA Maker avec vos paramètres de base de connaissances et retourner la meilleure réponse.

1. Dans le nouvel écran du flux QnA Maker, sélectionnez **Continuer**.

    :::image type="content" source="../media/how-to-integrate-power-virtual-agent/power-automate-qna-flow-template-continue.png" alt-text="Capture d’écran partielle du flux du modèle QnA Maker, avec le bouton Continuer mis en surbrillance.":::

1. Activez la zone d’action **Generate Answer (Générer une réponse)** et renseignez vos paramètres QnA Maker provenant de la section précédente intitulée [Créer et publier une base de connaissances](#create-and-publish-a-knowledge-base). Votre **hôte de service** dans l’image suivante fait référence à votre **hôte** de base de connaissances et est au format `https://YOUR-RESOURCE-NAME.azurewebsites.net/qnamaker`.


    :::image type="content" source="../media/how-to-integrate-power-virtual-agent/power-virtual-agent-fill-in-generate-answer-settings.png" alt-text="Capture d’écran partielle du flux du modèle QnA Maker, avec Generate answer (Preview) mis en évidence.":::

1. Sélectionnez **Enregistrer** pour enregistrer le flux.

## <a name="create-a-solution-and-add-the-flow"></a>Créer une solution et ajouter le flux

Pour que l’agent puisse trouver le flux et s’y connecter, il convient d’inclure le flux dans une solution Power Automate.

1. Toujours sur le portail Power Automate, sélectionnez **Solutions** dans le volet de navigation à gauche.

1. Sélectionnez **+ New solution** (Nouvelle solution).

1. Entrez un nom d’affichage. La liste des solutions comprend chacune des solutions de votre organisation ou école. Choisissez une convention de nommage qui vous aide à filtrer uniquement vos solutions. Par exemple, vous pouvez faire précéder votre nom de solution de votre adresse e-mail : `jondoe-power-virtual-agent-qnamaker-fallback`.

1. Sélectionnez votre serveur de publication dans le liste de choix.

1. Acceptez les valeurs par défaut pour le nom et la version.

1. Sélectionnez **Create** (Créer) pour terminer le processus.

## <a name="add-your-flow-to-the-solution"></a>Ajouter votre flux à la solution

1. Dans la liste des solutions, sélectionnez la solution que vous venez de créer. Elle doit figurer en haut de la liste. Si ce n’est pas le cas, effectuez une recherche sur votre adresse e-mail, qui est comprise dans le nom de la solution.

1. Dans la solution, sélectionnez **+ Add existing** (Ajouter existant), puis sélectionnez **Flow** (Flux) dans la liste.

1. Recherchez votre flux dans la liste **Outside solutions (Solutions extérieures)** , puis sélectionnez **Add (Ajouter)** pour terminer le processus. Si les flux sont nombreux, examinez la colonne **Modified** (Modifié) pour trouver le flux le plus récent.

## <a name="add-your-solutions-flow-to-power-virtual-agents"></a>Ajouter le flux de votre solution à Power Virtual Agents

1. Revenez à l’onglet du navigateur où figure votre agent dans Power Virtual Agents. Le canevas de création doit encore être ouvert.

1. Pour insérer une nouvelle étape dans le flux, au-dessus de la zone d’action **Message**, sélectionnez le connecteur **+** . Ensuite, sélectionnez **Call an action** (Appeler une action).

1. Dans la fenêtre contextuelle **Flux**, sélectionnez le nouveau flux nommé **Generate answers using QnA Maker knowledge base** (Générer des réponses à l’aide de la base de connaissances QnA Maker). La nouvelle action apparaît dans le flux.

    :::image type="content" source="../media/how-to-integrate-power-virtual-agent/power-virtual-agent-flow-after-adding-action.png" alt-text="Capture d’écran partielle du canevas de conversation de la rubrique Power Virtual Agents, après ajout du flux QnA Maker.":::

1. Pour définir correctement la variable d’entrée sur l’action QnA Maker, sélectionnez **Select a variable (Sélectionner une variable)** , puis sélectionnez **bot.UnrecognizedTriggerPhrase**.

    :::image type="content" source="../media/how-to-integrate-power-virtual-agent/power-virtual-agent-selection-action-input.png" alt-text="Capture d’écran partielle du canevas de conversation de la rubrique Power Virtual Agents, avec sélection de la variable d’entrée.":::


1. Pour définir correctement la variable de sortie sur l’action QnA Maker, dans l’action **Message**, sélectionnez **UnrecognizedTriggerPhrase**, sélectionnez l’icône pour insérer la variable `{x}`, puis sélectionnez **FinalAnswer**.

1. Dans la barre d’outils contextuelle, sélectionnez **Save** (Enregistrer) pour enregistrer les détails du canevas de création pour la rubrique.

Voici à quoi ressemble le canevas final de l’agent.

> [!div class="mx-imgBorder"]
> ![Capture d’écran montrant le canevas final de l’agent avec les sections Trigger Phrases (Expressions de déclenchement), Action et Message.](../media/how-to-integrate-power-virtual-agent/power-virtual-agent-topic-authoring-canvas-full-flow.png)

## <a name="test-the-agent"></a>Tester l’agent

1. Dans le volet de test, activez **Track between topics** (Suivi entre les rubriques). Vous pouvez ainsi examiner la progression entre les rubriques, mais aussi dans une même rubrique.

1. Testez l’agent en entrant le texte utilisateur dans l’ordre suivant. Le canevas de création indique la bonne exécution des étapes par une coche verte.

    |Ordre des questions|Questions de test|Objectif|
    |--|--|--|
    |1|Hello|Commencer la conversation|
    |2|Store hours (Horaires du magasin)|Exemple de rubrique. Elle est configurée automatiquement sans aucune action supplémentaire de votre part.|
    |3|Oui|En réponse à `Did that answer your question?`|
    |4|Excellent|En réponse à `Please rate your experience.`|
    |5|Oui|En réponse à `Can I help with anything else?`|
    |6|Comment améliorer les performances de débit pour les prédictions de requête ?|Cette question déclenche une action de secours, qui envoie le texte à votre base de connaissances pour obtenir une réponse. La réponse est ensuite affichée. Les coches vertes pour les actions individuelles indiquent la réussite de chaque action.|

    :::image type="content" source="../media/how-to-integrate-power-virtual-agent/power-virtual-agent-test-tracked.png" alt-text="Capture d’écran du bot de conversation avec le canevas qui indique des coches vertes pour les actions réussies.":::

## <a name="publish-your-bot"></a>Publier votre bot

Pour rendre l’agent accessible à tous les membres de votre école ou organisation, vous devez le publier.

1. Dans le volet de navigation de gauche, sélectionnez **Publish** (Publier). Sélectionnez ensuite **Publish** dans la page.

1. Essayez votre bot sur le site web de démonstration (recherchez le lien sous **Publish**).

    Une nouvelle page web s’ouvre avec votre bot. Posez la même question de test au bot : `How can I improve the throughput performance for query predictions?`

    > [!div class="mx-imgBorder"]
    > ![Capture d’écran du canevas final de l’agent](../media/how-to-integrate-power-virtual-agent/demo-chat-bot.png)

## <a name="share-your-bot"></a>Partager votre bot

Pour partager le site web de démonstration, configurez-le en tant que canal.

1. Dans le volet de navigation de gauche, sélectionnez **Manage** (Gérer)  > **Channels** (Canaux).

1. Sélectionnez **Demo website** (Site web de démonstration) dans la liste des canaux.

1. Copiez le lien, puis sélectionnez **Save** (Enregistrer). Collez le lien du site web de démonstration dans un e-mail destiné aux membres de votre école ou organisation.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous en avez fini avec la base de connaissances, supprimez les ressources QnA Maker sur le portail Azure.

## <a name="next-step"></a>Étape suivante

[Obtenir une analyse de votre base de connaissances](../How-To/get-analytics-knowledge-base.md)

Pour en savoir plus :
* [Power Virtual Agents](/power-virtual-agents/)
* [Power Automate](/power-automate/)
* [Connecteur QnA Maker](https://us.flow.microsoft.com/connectors/shared_cognitiveservicesqnamaker/qna-maker/) et [paramètres du connecteur](/connectors/cognitiveservicesqnamaker/)