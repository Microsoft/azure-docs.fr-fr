---
title: Utiliser Personalizer dans un chatbot - Personalizer
description: Personnalisez un chatbot .NET C# à l’aide d’une boucle Personalizer pour fournir le contenu approprié à un utilisateur en fonction d’actions (avec des caractéristiques) et de caractéristiques de contexte.
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: tutorial
ms.date: 07/17/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 7c4920eaa7a5619be37d38afd763e7be416d3124
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94565719"
---
# <a name="tutorial-use-personalizer-in-net-chat-bot"></a>Tutoriel : Utiliser Personalizer dans un chatbot .NET

Utilisez un chatbot .NET C# avec une boucle Personalizer pour fournir le contenu approprié à un utilisateur. Ce chatbot suggère un café ou un thé spécifique à un utilisateur. L’utilisateur peut accepter ou refuser cette suggestion. Cette action fournit à Personalizer des informations lui permettant de rendre la suggestion suivante plus appropriée.

**Dans ce tutoriel, vous allez découvrir comment :**

<!-- green checkmark -->
> [!div class="checklist"]
> * Configurer les ressources Azure
> * Configurer et exécuter un bot
> * Interagir avec un bot à l’aide de Bot Framework Emulator
> * Déterminer à quel moment et comment le bot utilise Personalizer


## <a name="how-does-the-chat-bot-work"></a>Fonctionnement du chatbot

Fondamentalement, un chatbot est un mécanisme de conversation bidirectionnelle avec un utilisateur. Ce chatbot spécifique utilise Personalizer pour sélectionner la meilleure action à proposer à l’utilisateur (café ou thé). Personalizer utilise l’apprentissage par renforcement pour effectuer cette sélection.

Le chatbot doit gérer des tours de conversation. Le chatbot utilise le [Bot Framework](https://github.com/microsoft/botframework-sdk) pour gérer l’architecture du bot et la conversation. Par ailleurs, il utilise le service cognitif LUIS ([Language Understanding](../LUIS/index.yml)) pour comprendre l’intention du langage naturel de l’utilisateur.

Le chatbot est un site web avec une route spécifique disponible pour répondre aux demandes : `http://localhost:3978/api/messages`. Vous pouvez utiliser l’émulateur de bot pour interagir visuellement avec le chatbot en cours d’exécution tout en développant un bot localement.

### <a name="user-interactions-with-the-bot"></a>Utiliser des interactions avec le bot

Il s’agit d’un chatbot simple qui vous permet d’entrer des requêtes de texte.

|Texte entré par l’utilisateur|Texte de réponse du bot|Description de l’action appliquée par le bot pour déterminer le texte de la réponse|
|--|--|--|
|Aucun texte entré - le bot commence la conversation.|`This is a simple chatbot example that illustrates how to use Personalizer. The bot learns what coffee or tea order is preferred by customers given some context information (such as weather, temperature, and day of the week) and information about the user.`<br>`To use the bot, just follow the prompts. To try out a new imaginary context, type “Reset” and a new one will be randomly generated.`<br>`Welcome to the coffee bot, please tell me if you want to see the menu or get a coffee or tea suggestion for today. Once I’ve given you a suggestion, you can reply with ‘like’ or ‘don’t like’. It’s Tuesday today and the weather is Snowy.`|Le bot commence la conversation avec un texte d’instructions et vous informe du contexte : `Tuesday`, `Snowy`.|
|`Show menu`|`Here is our menu: Coffee: Cappuccino Espresso Latte Macchiato Mocha Tea: GreenTea Rooibos`|Le bot détermine l’intention de la requête à l’aide de LUIS, puis affiche les choix de menu relatifs au café et au thé. Les caractéristiques des actions sont les suivantes. |
|`What do you suggest`|`How about Latte?`|Le bot détermine l’intention de la requête à l’aide de LUIS, puis appelle l’**API Rank** et affiche le meilleur choix sous forme de question : `How about {response.RewardActionId}?`. Il affiche également l’appel et la réponse JSON à des fins d’illustration.|
|`I like it`|`That’s great! I’ll keep learning your preferences over time.`<br>`Would you like to get a new suggestion or reset the simulated context to a new day?`|Le bot détermine l’intention de la requête à l’aide de LUIS, puis appelle l’**API Reward** avec la récompense `1`. Il affiche l’appel et la réponse JSON à des fins d’illustration.|
|`I don't like it`|`Oh well, maybe I’ll guess better next time.`<br>`Would you like to get a new suggestion or reset the simulated context to a new day?`|Le bot détermine l’intention de la requête à l’aide de LUIS, puis appelle l’**API Reward** avec la récompense `0`. Il affiche l’appel et la réponse JSON à des fins d’illustration.|
|`Reset`|Le bot retourne le texte d’instructions.|Le bot détermine l’intention de la requête à l’aide de LUIS, puis affiche le texte d’instructions et réinitialise le contexte.|


### <a name="personalizer-in-this-bot"></a>Personalizer dans ce bot

Ce chatbot utilise Personalizer pour sélectionner la meilleure action (café ou thé spécifique) en fonction d’une liste d’_actions_ (un type de contenu) et de caractéristiques de contexte.

Le bot envoie la liste des actions avec les caractéristiques de contexte à la boucle Personalizer. Personalizer retourne la meilleure action au bot, et le bot affiche l’action.

Dans ce tutoriel, les **actions** sont des types de café et de thé (« Coffee » et « Tea » dans cet exemple) :

|Coffee|Tea|
|--|--|
|Cappuccino<br>Espresso<br>Latte<br>Mocha|GreenTea<br>Rooibos|

**API Rank :** pour permettre à Personalizer d’apprendre en fonction de vos actions, le bot envoie les éléments suivants avec chaque requête à l’API Rank :

* Actions _avec caractéristiques_
* Caractéristiques contextuelles

Une **caractéristique** du modèle est une information sur l’action ou le contexte qui peut être agrégée (groupée) entre les membres de la base d’utilisateurs de votre chatbot. Une fonctionnalité _n’est pas_ spécifique à un individu (par exemple, un identificateur d’utilisateur) ni très précise (par exemple, une heure précise de la journée).

Les fonctionnalités sont utilisées pour aligner des actions sur le contexte actuel du modèle. Le modèle est une représentation des connaissances passées de Personalizer sur les actions, le contexte et leurs fonctionnalités qui lui permettent de prendre des décisions éclairées.

Le modèle, y compris les fonctionnalités, est mis à jour selon une planification basée sur votre paramètre **Fréquence de mise à jour du modèle** dans le portail Azure.

Les fonctionnalités doivent être sélectionnées avec la même planification et la même conception que celles que vous appliqueriez à n’importe quel schéma ou modèle de votre architecture technique. Les valeurs des fonctionnalités peuvent être définies à l’aide d’une logique métier ou de systèmes tiers.

> [!CAUTION]
> Les caractéristiques de cette application sont définies à des fins de démonstration. Elles ne sont pas nécessairement les plus appropriées pour une application web dans votre cas d’usage.

#### <a name="action-features"></a>Caractéristiques d’action

Chaque action (élément de contenu) possède des caractéristiques qui permettent de distinguer l’élément café ou thé.

Les fonctionnalités ne sont pas configurées dans le cadre de la configuration de la boucle sur le portail Azure. Au lieu de cela, elles sont envoyées sous la forme d’un objet JSON avec chaque appel d’API de classement. Les actions et leurs caractéristiques peuvent ainsi croître, changer et se réduire au fil du temps, ce qui permet à Personalizer de suivre les tendances.

Les caractéristiques du café et du thé sont les suivantes :

* Origine du grain de café, par exemple Kenya ou Brésil
* Le café ou le thé est-il bio ?
* Torréfaction légère ou foncée du café

Alors que le café a trois caractéristiques dans la liste précédente, le thé n’en a qu’une. Transmettez à Personalizer les caractéristiques pertinentes pour l’action uniquement. Ne transmettez pas de valeur vide pour une caractéristique si elle ne s’applique pas à l’action.

#### <a name="context-features"></a>Caractéristiques contextuelles

Les caractéristiques de contexte permettent à Personalizer de comprendre le contexte de l’environnement, ce qui inclut l’appareil d’affichage, l’utilisateur, l’emplacement et d’autres caractéristiques pertinentes pour votre cas d’usage.

Le contexte pour ce chatbot comprend les éléments suivants :

* Type de météo (neige, pluie, soleil)
* Jour de la semaine

La sélection de caractéristiques pour ce chatbot est présentée de façon aléatoire. Pour un bot réel, utilisez des données réelles pour vos caractéristiques de contexte.

### <a name="design-considerations-for-this-bot"></a>Considérations relatives à la conception du bot

Tenez compte de quelques remarques importantes concernant cette conversation :
* **Interaction avec le bot** : la conversation est très simple, car elle illustre le classement (Rank) et la récompense (Reward) pour un cas d’usage simple. Elle ne montre pas toutes les fonctionnalités du SDK Bot Framework et de l’émulateur.
* **Personalizer** : les caractéristiques sont sélectionnées de façon aléatoire pour simuler l’utilisation, ce qui ne doit pas être le cas dans un scénario de production avec Personalizer.
* **LUIS (Language Understanding)**  : les quelques exemples d’énoncés du modèle LUIS sont destinés uniquement à cet exemple. N’utilisez pas si peu d’exemples d’énoncés dans votre application LUIS de production.


## <a name="install-required-software"></a>Installer les logiciels nécessaires
- [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/). Le dépôt d’exemples téléchargeables contient des instructions qui vous seront utiles si vous préférez utiliser l’interface CLI .NET Core.
- [Microsoft Bot Framework Emulator](https://aka.ms/botframeworkemulator) est une application de bureau qui permet aux développeurs de bots de tester et déboguer leurs bots sur localhost ou en les exécutant à distance par le biais d’un tunnel.

## <a name="download-the-sample-code-of-the-chat-bot"></a>Télécharger l’exemple de code du chatbot

Le chatbot est disponible dans le dépôt d’exemples Personalizer. Clonez ou [téléchargez](https://github.com/Azure-Samples/cognitive-services-personalizer-samples/archive/master.zip) le dépôt, puis ouvrez l’exemple disponible dans le répertoire `/samples/ChatbotExample` avec Visual Studio 2019.

Pour cloner le dépôt, utilisez la commande Git suivante dans un interpréteur de commandes Bash (terminal).

```bash
git clone https://github.com/Azure-Samples/cognitive-services-personalizer-samples.git
```

## <a name="create-and-configure-personalizer-and-luis-resources"></a>Créer et configurer Personalizer et les ressources LUIS

### <a name="create-azure-resources"></a>Créer des ressources Azure

Pour utiliser ce chatbot, vous devez créer des ressources Azure pour Personalizer et LUIS (Language Understanding).

* [Créez des ressources LUIS](../luis/luis-how-to-azure-subscription.md#create-luis-resources-in-the-azure-portal). Sélectionnez **Les deux** à l’étape de création, car vous avez besoin de ressources de création et de prédiction.
* [Créez une ressource Personalizer](how-to-create-resource.md), puis copiez la clé et le point de terminaison à partir du portail Azure. Vous devrez définir ces valeurs dans le fichier `appsettings.json` du projet .NET.

### <a name="create-luis-app"></a>Créer une application LUIS

Si vous débutez avec LUIS, vous devez [vous connecter](https://www.luis.ai) et migrer immédiatement votre compte. Vous n’avez pas besoin de créer de ressources. Sélectionnez plutôt les ressources que vous avez créées à la section précédente de ce tutoriel.

1. Pour créer une application LUIS, dans le [portail LUIS](https://www.luis.ai), sélectionnez votre abonnement et votre ressource de création.
1. Ensuite, dans la même page, sélectionnez **+ New app for conversation**, puis **Import as JSON**.
1. Dans la boîte de dialogue contextuelle, sélectionnez **Choose file**, puis sélectionnez le fichier `/samples/ChatbotExample/CognitiveModels/coffeebot.json`. Entrez le nom `Personalizer Coffee bot`.
1. Sélectionnez le bouton **Train** dans la zone de navigation en haut à droite du portail LUIS.
1. Sélectionnez le bouton **Publish** afin de publier l’application à l’**emplacement de production** pour le runtime de prédiction.
1. Sélectionnez **Manage**, puis **Settings**. Copiez la valeur **App ID**. Vous devrez définir cette valeur dans le fichier `appsettings.json` du projet .NET.
1. Toujours dans la section **Manage**, sélectionnez **Azure Resources**. Les ressources associées s’affichent dans l’application.
1. Sélectionnez **Add prediction resource**. Dans la boîte de dialogue contextuelle, sélectionnez votre abonnement et la ressource de prédiction créée précédemment dans ce tutoriel, puis sélectionnez **Done**.
1. Copiez les valeurs **Primary key** et **Endpoint URL**. Vous devrez définir ces valeurs dans le fichier `appsettings.json` du projet .NET.

### <a name="configure-bot-with-appsettingsjson-file"></a>Configurer le bot avec le fichier appsettings.json

1. Ouvrez le fichier de solution du chatbot `ChatbotSamples.sln` dans Visual Studio 2019.
1. Ouvrez `appsettings.json`, situé dans le répertoire racine du projet.
1. Définissez les cinq paramètres copiés à la section précédente de ce tutoriel.

    ```json
    {
      "PersonalizerChatbot": {
        "LuisAppId": "",
        "LuisAPIKey": "",
        "LuisServiceEndpoint": "",
        "PersonalizerServiceEndpoint": "",
        "PersonalizerAPIKey": ""
      }
    }
    ```

### <a name="build-and-run-the-bot"></a>Générer et exécuter le bot

Après avoir configuré `appsettings.json`, vous êtes prêt à générer et exécuter le chatbot. Un navigateur s’ouvre alors, avec le site web `http://localhost:3978` en cours d’exécution.

:::image type="content" source="media/tutorial-chat-bot/chat-bot-web-site.png" alt-text="Capture d’écran du navigateur dans lequel s’affiche le site web du chatbot.":::

Conservez le site web en cours d’exécution. En effet, le tutoriel explique ce que fait le bot et vous pouvez ainsi interagir avec ce dernier.


## <a name="set-up-the-bot-emulator"></a>Configurer l’émulateur de bot

1. Ouvrez Bot Framework Emulator, puis sélectionnez **Open Bot** (Ouvrir le bot).

    :::image type="content" source="media/tutorial-chat-bot/bot-emulator-startup.png" alt-text="Capture d’écran de l’écran de démarrage de l’émulateur de bot":::


1. Configurez le bot en spécifiant l’URL de bot suivante sous **Bot URL**, puis sélectionnez **Connect** :

    `http://localhost:3978/api/messages`

    :::image type="content" source="media/tutorial-chat-bot/bot-emulator-open-bot-settings.png" alt-text="Capture d’écran des paramètres d’ouverture de bot dans l’émulateur de bot":::

    L’émulateur se connecte au chatbot et affiche le texte d’instructions avec les informations de journalisation et de débogage, utiles pour le développement local.

    :::image type="content" source="media/tutorial-chat-bot/bot-emulator-bot-conversation-first-turn.png" alt-text="Capture d’écran de l’émulateur de bot au premier tour de la conversation":::

## <a name="use-the-bot-in-the-bot-emulator"></a>Utiliser le bot dans l’émulateur de bot

1. Demandez l’affichage du menu en entrant `I would like to see the menu`. Le chatbot affiche les éléments.
1. Demandez au bot de suggérer un élément en entrant `Please suggest a drink for me.`. L’émulateur affiche la requête Rank et la réponse dans la fenêtre de conversation. Vous pouvez ainsi voir l’ensemble du code JSON. Le bot affiche alors une suggestion, par exemple `How about Latte?`.
1. Répondez que c’est ce que vous souhaitez : `I like it.`. Vous acceptez donc la sélection la mieux classée de Personalizer. L’émulateur affiche la requête Reward avec le score de récompense 1 et la réponse dans la fenêtre de conversation. Vous pouvez ainsi voir l’ensemble du code JSON. Le bot répond avec `That’s great! I’ll keep learning your preferences over time.` et `Would you like to get a new suggestion or reset the simulated context to a new day?`.

    Si vous répondez à la sélection avec `no`, le score de récompense 0 est envoyé à Personalizer.


## <a name="understand-the-net-code-using-personalizer"></a>Comprendre le code .NET avec Personalizer

La solution .NET est un chatbot simple basé sur le Bot Framework. Le code associé à Personalizer se trouve dans les dossiers suivants :
* `/samples/ChatbotExample/Bots`
    * `PersonalizerChatbot.cs` : fichier pour l’interaction entre le bot et Personalizer
* `/samples/ChatbotExample/ReinforcementLearning` : gère les actions et les caractéristiques pour le modèle Personalizer
* `/samples/ChatbotExample/Model` : fichiers pour les actions et les caractéristiques Personalizer et pour les intentions LUIS

### <a name="personalizerchatbotcs---working-with-personalizer"></a>Utilisation de PersonalizerChatbot.cs avec Personalizer

La classe `PersonalizerChatbot` est dérivée de la classe `Microsoft.Bot.Builder.ActivityHandler`. Elle a trois propriétés et méthodes permettant de gérer le flux de la conversation.

> [!CAUTION]
> Ne copiez pas le code de ce tutoriel. Utilisez l’exemple de code disponible dans le [dépôt d’exemples Personalizer](https://github.com/Azure-Samples/cognitive-services-personalizer-samples).

```csharp
public class PersonalizerChatbot : ActivityHandler
{

    private readonly LuisRecognizer _luisRecognizer;
    private readonly PersonalizerClient _personalizerClient;

    private readonly RLContextManager _rlFeaturesManager;

    public PersonalizerChatbot(LuisRecognizer luisRecognizer, RLContextManager rlContextManager, PersonalizerClient personalizerClient)
            {
                _luisRecognizer = luisRecognizer;
                _rlFeaturesManager = rlContextManager;
                _personalizerClient = personalizerClient;
            }
    }

    public override async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
    {
        await base.OnTurnAsync(turnContext, cancellationToken);

        if (turnContext.Activity.Type == ActivityTypes.Message)
        {
            // Check LUIS model
            var recognizerResult = await _luisRecognizer.RecognizeAsync(turnContext, cancellationToken);
            var topIntent = recognizerResult?.GetTopScoringIntent();
            if (topIntent != null && topIntent.HasValue && topIntent.Value.intent != "None")
            {
                Intents intent = (Intents)Enum.Parse(typeof(Intents), topIntent.Value.intent);
                switch (intent)
                {
                    case Intents.ShowMenu:
                        await turnContext.SendActivityAsync($"Here is our menu: \n Coffee: {CoffeesMethods.DisplayCoffees()}\n Tea: {TeaMethods.DisplayTeas()}", cancellationToken: cancellationToken);
                        break;
                    case Intents.ChooseRank:
                        // Here we generate the event ID for this Rank.
                        var response = await ChooseRankAsync(turnContext, _rlFeaturesManager.GenerateEventId(), cancellationToken);
                        _rlFeaturesManager.CurrentPreference = response.Ranking;
                        await turnContext.SendActivityAsync($"How about {response.RewardActionId}?", cancellationToken: cancellationToken);
                        break;
                    case Intents.RewardLike:
                        if (!string.IsNullOrEmpty(_rlFeaturesManager.CurrentEventId))
                        {
                            await RewardAsync(turnContext, _rlFeaturesManager.CurrentEventId, 1, cancellationToken);
                            await turnContext.SendActivityAsync($"That's great! I'll keep learning your preferences over time.", cancellationToken: cancellationToken);
                            await SendByebyeMessageAsync(turnContext, cancellationToken);
                        }
                        else
                        {
                            await turnContext.SendActivityAsync($"Not sure what you like. Did you ask for a suggestion?", cancellationToken: cancellationToken);
                        }

                        break;
                    case Intents.RewardDislike:
                        if (!string.IsNullOrEmpty(_rlFeaturesManager.CurrentEventId))
                        {
                            await RewardAsync(turnContext, _rlFeaturesManager.CurrentEventId, 0, cancellationToken);
                            await turnContext.SendActivityAsync($"Oh well, maybe I'll guess better next time.", cancellationToken: cancellationToken);
                            await SendByebyeMessageAsync(turnContext, cancellationToken);
                        }
                        else
                        {
                            await turnContext.SendActivityAsync($"Not sure what you dislike. Did you ask for a suggestion?", cancellationToken: cancellationToken);
                        }

                        break;
                    case Intents.Reset:
                        _rlFeaturesManager.GenerateRLFeatures();
                        await SendResetMessageAsync(turnContext, cancellationToken);
                        break;
                    default:
                        break;
                }
            }
            else
            {
                var msg = @"Could not match your message with any of the following LUIS intents:
                        'ShowMenu'
                        'ChooseRank'
                        'RewardLike'
                        'RewardDislike'.
                        Try typing 'Show me the menu','What do you suggest','I like it','I don't like it'.";
                await turnContext.SendActivityAsync(msg);
            }
        }
        else if (turnContext.Activity.Type == ActivityTypes.ConversationUpdate)
        {
            // Generate a new weekday and weather condition
            // These will act as the context features when we call rank with Personalizer
            _rlFeaturesManager.GenerateRLFeatures();

            // Send a welcome message to the user and tell them what actions they may perform to use this bot
            await SendWelcomeMessageAsync(turnContext, cancellationToken);
        }
        else
        {
            await turnContext.SendActivityAsync($"{turnContext.Activity.Type} event detected", cancellationToken: cancellationToken);
        }
    }

    // code removed for brevity, full sample code available for download
    private async Task SendWelcomeMessageAsync(ITurnContext turnContext, CancellationToken cancellationToken)
    private async Task SendResetMessageAsync(ITurnContext turnContext, CancellationToken cancellationToken)
    private async Task SendByebyeMessageAsync(ITurnContext turnContext, CancellationToken cancellationToken)
    private async Task<RankResponse> ChooseRankAsync(ITurnContext turnContext, string eventId, CancellationToken cancellationToken)
    private async Task RewardAsync(ITurnContext turnContext, string eventId, double reward, CancellationToken cancellationToken)
}
```

Les méthodes ayant le préfixe `Send` gèrent la conversation avec le bot et LUIS. Les méthodes `ChooseRankAsync` et `RewardAsync` interagissent avec Personalizer.

#### <a name="calling-rank-api-and-display-results"></a>Appel de l’API Rank et affichage des résultats

La méthode `ChooseRankAsync` crée les données JSON à envoyer à l’API Rank de Personalizer en recueillant les actions avec les caractéristiques et les caractéristiques de contexte.

```csharp
private async Task<RankResponse> ChooseRankAsync(ITurnContext turnContext, string eventId, CancellationToken cancellationToken)
{
    IList<object> contextFeature = new List<object>
    {
        new { weather = _rlFeaturesManager.RLFeatures.Weather.ToString() },
        new { dayofweek = _rlFeaturesManager.RLFeatures.DayOfWeek.ToString() },
    };

    Random rand = new Random(DateTime.UtcNow.Millisecond);
    IList<RankableAction> actions = new List<RankableAction>();
    var coffees = Enum.GetValues(typeof(Coffees));
    var beansOrigin = Enum.GetValues(typeof(CoffeeBeansOrigin));
    var organic = Enum.GetValues(typeof(Organic));
    var roast = Enum.GetValues(typeof(CoffeeRoast));
    var teas = Enum.GetValues(typeof(Teas));

    foreach (var coffee in coffees)
    {
        actions.Add(new RankableAction
        {
            Id = coffee.ToString(),
            Features =
            new List<object>()
            {
                new { BeansOrigin = beansOrigin.GetValue(rand.Next(0, beansOrigin.Length)).ToString() },
                new { Organic = organic.GetValue(rand.Next(0, organic.Length)).ToString() },
                new { Roast = roast.GetValue(rand.Next(0, roast.Length)).ToString() },
            },
        });
    }

    foreach (var tea in teas)
    {
        actions.Add(new RankableAction
        {
            Id = tea.ToString(),
            Features =
            new List<object>()
            {
                new { Organic = organic.GetValue(rand.Next(0, organic.Length)).ToString() },
            },
        });
    }

    // Sending a rank request to Personalizer
    // Here we are asking Personalizer to decide which drink the user is most likely to want
    // based on the current context features (weather, day of the week generated in RLContextManager)
    // and the features of the drinks themselves
    var request = new RankRequest(actions, contextFeature, null, eventId);
    await turnContext.SendActivityAsync(
        "===== DEBUG MESSAGE CALL TO RANK =====\n" +
        "This is what is getting sent to Rank:\n" +
        $"{JsonConvert.SerializeObject(request, Formatting.Indented)}\n",
        cancellationToken: cancellationToken);
    var response = await _personalizerClient.RankAsync(request, cancellationToken);
    await turnContext.SendActivityAsync(
        $"===== DEBUG MESSAGE RETURN FROM RANK =====\n" +
        "This is what Rank returned:\n" +
        $"{JsonConvert.SerializeObject(response, Formatting.Indented)}\n",
        cancellationToken: cancellationToken);
    return response;
}
```

#### <a name="calling-reward-api-and-display-results"></a>Appel de l’API Reward et affichage des résultats

La méthode `RewardAsync` crée les données JSON à envoyer à l’API Reward de Personalizer en déterminant le score. Le score est déterminé à partir de l’intention LUIS identifiée dans le texte de l’utilisateur et envoyé à partir de la méthode `OnTurnAsync`.

```csharp
private async Task RewardAsync(ITurnContext turnContext, string eventId, double reward, CancellationToken cancellationToken)
{
    await turnContext.SendActivityAsync(
        "===== DEBUG MESSAGE CALL REWARD =====\n" +
        "Calling Reward:\n" +
        $"eventId = {eventId}, reward = {reward}\n",
        cancellationToken: cancellationToken);

    // Sending a reward request to Personalizer
    // Here we are responding to the drink ranking Personalizer provided us
    // If the user liked the highest ranked drink, we give a high reward (1)
    // If they did not, we give a low reward (0)
    await _personalizerClient.RewardAsync(eventId, new RewardRequest(reward), cancellationToken);
}
```

## <a name="design-considerations-for-a-bot"></a>Considérations relatives à la conception d’un bot

Cet exemple est destiné à illustrer une solution de bout en bout simple avec Personalizer dans un bot. Votre cas d’usage peut être plus complexe.

Si vous envisagez d’utiliser Personalizer avec un bot de production, prévoyez les aspects suivants :
* Accès en temps réel à Personalizer _chaque fois_ que vous avez besoin d’une sélection classée. L’API Rank ne peut pas être traitée par lot ni mise en cache.  L’appel de récompense peut être retardé ou déplacé vers un processus distinct. Si vous ne retournez pas de récompense dans le délai imparti, une valeur de récompense par défaut est définie pour l’événement.
* Calcul de la récompense basé sur le cas d’usage : cet exemple illustre deux récompenses (zéro et un) sans plage entre les deux et sans valeur négative pour un score. Il se peut que votre système nécessite un scoring plus précis.
* Canaux de bots : cet exemple utilise un canal unique. Si vous envisagez d’utiliser plusieurs canaux ou des variantes de bots dans un même canal, vous devrez peut-être les considérer dans le cadre des caractéristiques de contexte du modèle Personalizer.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Une fois ce didacticiel terminé, nettoyez les ressources suivantes :

* Supprimez votre exemple de répertoire de projet.
* Supprimez vos ressources Personalizer et LUIS dans le portail Azure.

## <a name="next-steps"></a>Étapes suivantes
* [Fonctionnement de Personalizer](how-personalizer-works.md)
* [Fonctionnalités](concepts-features.md) : découvrir les concepts relatifs à l’utilisation de fonctionnalités avec les actions et le contexte
* [Récompenses](concept-rewards.md) : en savoir plus sur le calcul des récompenses
