---
title: Utiliser une application web – Personalizer
description: Personnalisez une application web .NET C# à l’aide d’une boucle Personalizer pour fournir le contenu approprié à un utilisateur en fonction d’actions (avec des fonctionnalités) et de caractéristiques contextuelles.
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: tutorial
ms.date: 06/10/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: c004887e3883ae711974b544510dff16a98d4ef9
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94363916"
---
# <a name="tutorial-add-personalizer-to-a-net-web-app"></a>Tutoriel : Ajouter Personalizer à une application web .NET

Personnalisez une application web .NET C# à l’aide d’une boucle Personalizer pour fournir le contenu approprié à un utilisateur en fonction d’actions (avec des fonctionnalités) et de caractéristiques contextuelles.

**Dans ce tutoriel, vous allez découvrir comment :**

<!-- green checkmark -->
> [!div class="checklist"]
> * Configurer la clé et le point de terminaison Personalizer
> * Collecter les fonctionnalités
> * Appeler des API de classement et de récompense
> * Afficher la meilleure action, désignée sous le nom _rewardActionId_



## <a name="select-the-best-content-for-a-web-app"></a>Sélectionner le meilleur contenu pour une application web

Une application web doit utiliser Personalizer lorsqu’il existe une liste d’ _actions_ (un certain type de contenu) sur la page web qui doit être personnalisée pour afficher un seul élément de tête (rewardActionId). Les listes d’actions comprennent par exemple des articles d’actualité, des emplacements de boutons et des choix de mots pour les noms de produits.

Vous envoyez la liste des actions, ainsi que les caractéristiques contextuelles, à la boucle Personalizer. Personalizer sélectionne la meilleure action, puis votre application web affiche cette action.

Dans ce didacticiel, les actions sont des types d’aliments :

* pasta (pâtes)
* ice cream (glace)
* juice (jus)
* salad (salade)
* popcorn
* coffee (café)
* soupe

Pour aider Personalizer à en savoir plus sur vos actions, envoyez à la fois des _actions avec fonctionnalités_ et des _caractéristiques contextuelles_ à chaque demande d’API de classement.

Une **fonctionnalité** du modèle est l’information sur l’action ou le contexte qui peut être agrégée (groupée) entre les membres de la base d’utilisateurs de votre application web. Une fonctionnalité _n’est pas_ spécifique à un individu (par exemple, un identificateur d’utilisateur) ni très précise (par exemple, une heure précise de la journée).

### <a name="actions-with-features"></a>Actions avec fonctionnalités

Chaque action (élément de contenu) possède des fonctionnalités qui permettent de distinguer le produit alimentaire.

Les fonctionnalités ne sont pas configurées dans le cadre de la configuration de la boucle sur le portail Azure. Au lieu de cela, elles sont envoyées sous la forme d’un objet JSON avec chaque appel d’API de classement. Les actions et leurs caractéristiques peuvent ainsi croître, changer et se réduire au fil du temps, ce qui permet à Personalizer de suivre les tendances.

```csharp
 /// <summary>
  /// Creates personalizer actions feature list.
  /// </summary>
  /// <returns>List of actions for personalizer.</returns>
  private IList<RankableAction> GetActions()
  {
      IList<RankableAction> actions = new List<RankableAction>
      {
          new RankableAction
          {
              Id = "pasta",
              Features =
              new List<object>() { new { taste = "savory", spiceLevel = "medium" }, new { nutritionLevel = 5, cuisine = "italian" } }
          },

          new RankableAction
          {
              Id = "ice cream",
              Features =
              new List<object>() { new { taste = "sweet", spiceLevel = "none" }, new { nutritionalLevel = 2 } }
          },

          new RankableAction
          {
              Id = "juice",
              Features =
              new List<object>() { new { taste = "sweet", spiceLevel = "none" }, new { nutritionLevel = 5 }, new { drink = true } }
          },

          new RankableAction
          {
              Id = "salad",
              Features =
              new List<object>() { new { taste = "sour", spiceLevel = "low" }, new { nutritionLevel = 8 } }
          },

          new RankableAction
          {
              Id = "popcorn",
              Features =
              new List<object>() { new { taste = "salty", spiceLevel = "none" }, new { nutritionLevel = 3 } }
          },

          new RankableAction
          {
              Id = "coffee",
              Features =
              new List<object>() { new { taste = "bitter", spiceLevel = "none" }, new { nutritionLevel = 3 }, new { drink = true } }
          },

          new RankableAction
          {
              Id = "soup",
              Features =
              new List<object>() { new { taste = "sour", spiceLevel = "high" }, new { nutritionLevel =  7} }
          }
      };

      return actions;
  }
```


## <a name="context-features"></a>Caractéristiques contextuelles

Les caractéristiques contextuelles aident Personalizer à comprendre le contexte des actions. Le contexte de cet exemple d’application comprend les éléments suivants :

* heure du jour : matin, après-midi, soir, nuit
* préférence de l’utilisateur pour le gout : salé, sucré, amer, acide ou umami
* contexte du navigateur : agent utilisateur, localisation géographique, référent

```csharp
/// <summary>
/// Get users time of the day context.
/// </summary>
/// <returns>Time of day feature selected by the user.</returns>
private string GetUsersTimeOfDay()
{
    Random rnd = new Random();
    string[] timeOfDayFeatures = new string[] { "morning", "noon", "afternoon", "evening", "night", "midnight" };
    int timeIndex = rnd.Next(timeOfDayFeatures.Length);
    return timeOfDayFeatures[timeIndex];
}

/// <summary>
/// Gets user food preference.
/// </summary>
/// <returns>Food taste feature selected by the user.</returns>
private string GetUsersTastePreference()
{
    Random rnd = new Random();
    string[] tasteFeatures = new string[] { "salty", "bitter", "sour", "savory", "sweet" };
    int tasteIndex = rnd.Next(tasteFeatures.Length);
    return tasteFeatures[tasteIndex];
}
```

## <a name="how-does-the-web-app-use-personalizer"></a>Comment l’application web utilise-t-elle Personalizer ?

L’application web utilise Personalizer pour sélectionner la meilleure action dans la liste des choix alimentaires. Pour ce faire, il envoie les informations suivantes avec chaque appel de l’API de classement :
* les **actions** avec leurs fonctionnalités telles que `taste` et `spiceLevel`
* les **caractéristiques contextuelles** telles que l’heure `time` de la journée, la préférence `taste` de l’utilisateur et les informations de l’agent utilisateur du navigateur, ainsi que les caractéristiques contextuelles
* les **actions à exclure** telles que le jus
* **eventId** , qui est différent pour chaque appel à l’API Rank.

## <a name="personalizer-model-features-in-a-web-app"></a>Fonctionnalités du modèle Personalizer dans une application web

Personalizer a besoin de fonctionnalités pour les actions (contenu) et du contexte actuel (utilisateur et environnement). Les fonctionnalités sont utilisées pour aligner des actions sur le contexte actuel du modèle. Le modèle est une représentation des connaissances passées de Personalizer sur les actions, le contexte et leurs fonctionnalités qui lui permettent de prendre des décisions éclairées.

Le modèle, y compris les fonctionnalités, est mis à jour selon une planification basée sur votre paramètre **Fréquence de mise à jour du modèle** dans le portail Azure.

> [!CAUTION]
> Les fonctionnalités de cette application sont destinées à illustrer les fonctionnalités et les valeurs des fonctionnalités, mais pas nécessairement les meilleures fonctionnalités à utiliser dans une application web.

### <a name="plan-for-features-and-their-values"></a>Planifier les fonctionnalités et leurs valeurs

Les fonctionnalités doivent être sélectionnées avec la même planification et la même conception que celles que vous appliqueriez à n’importe quel schéma ou modèle de votre architecture technique. Les valeurs des fonctionnalités peuvent être définies à l’aide d’une logique métier ou de systèmes tiers. Les valeurs des fonctionnalités ne doivent pas être si spécifiques qu’elles ne s’appliquent pas à un groupe ou une classe de fonctionnalités.

### <a name="generalize-feature-values"></a>Généraliser les valeurs des fonctionnalités

#### <a name="generalize-into-categories"></a>Généraliser en catégories

Cette application utilise `time` en tant que fonctionnalité, mais regroupe les heures en catégories telles que `morning`, `afternoon`, `evening` et `night`. C’est un exemple d’utilisation des informations de temps, mais pas d’une manière très spécifique comme`10:05:01 UTC+2`.

#### <a name="generalize-into-parts"></a>Généraliser en parties

Cette application utilise les fonctionnalités de requête HTTP du navigateur. Cela commence par une chaîne très spécifique avec toutes les données, par exemple :

```http
Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/530.99 (KHTML, like Gecko) Chrome/80.0.3900.140 Safari/537.36
```

La bibliothèque de classes **HttpRequestFeatures** généralise cette chaîne en un objet **userAgentInfo** avec des valeurs individuelles. Toute valeur trop spécifique est définie sur une chaîne vide. Lorsque les caractéristiques contextuelles de la demande sont envoyées, le format JSON est le suivant :

```JSON
{
  "httpRequestFeatures": {
    "_synthetic": false,
    "OUserAgent": {
      "_ua": "",
      "_DeviceBrand": "",
      "_DeviceFamily": "Other",
      "_DeviceIsSpider": false,
      "_DeviceModel": "",
      "_OSFamily": "Windows",
      "_OSMajor": "10",
      "DeviceType": "Desktop"
    }
  }
}
```


## <a name="using-sample-web-app"></a>Utilisation d’un exemple d’application web

L’exemple d’application web basée sur un navigateur (tout le code est fourni) nécessite l’installation des applications suivantes pour exécuter l’application.

Installez les logiciels suivants :

* [.NET Core 2.1](https://dotnet.microsoft.com/download/dotnet-core/2.1) : l’exemple de serveur back-end utilise .NET Core
* [Node.js](https://nodejs.org/) : le client/serveur frontal dépend de cette application
* [Visual Studio 2019](https://visualstudio.microsoft.com/vs/) ou [CLI .NET Core](/dotnet/core/tools/) : utilisez l’environnement de développement de Visual Studio 2019 ou CLI .NET Core pour générer et exécuter l’application

### <a name="set-up-the-sample"></a>Configurer l’exemple
1. Clonez le référentiel Azure Personalizer Samples.

    ```bash
    git clone https://github.com/Azure-Samples/cognitive-services-personalizer-samples.git
    ```

1. Accédez à _samples/HttpRequestFeatures_ pour ouvrir la solution, `HttpRequestFeaturesExample.sln`.

    Si nécessaire, autorisez Visual Studio à mettre à jour le package .NET pour Personalizer.

### <a name="set-up-azure-personalizer-service"></a>Configurer le service Azure Personalizer

1. [Créez une ressource Personalizer](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesPersonalizer) dans le portail Azure.

1. Dans le portail Azure, recherchez le `Endpoint` et `Key1` ou `Key2` (les deux fonctionnent) sous l’onglet **Clés et points de terminaison**. Il s’agit de votre `PersonalizerServiceEndpoint` et de votre `PersonalizerApiKey`.
1. Renseignez `PersonalizerServiceEndpoint` dans **appsettings.json**.
1. Configurez `PersonalizerApiKey` en tant que [secrets de l’application](/aspnet/core/security/app-secrets) de l’une des manières suivantes :

    * Si vous utilisez CLI .NET Core, vous pouvez utiliser la commande `dotnet user-secrets set "PersonalizerApiKey" "<API Key>"`.
    * Si vous utilisez Visual Studio, vous pouvez cliquer avec le bouton droit sur le projet et sélectionner l’option de menu **Gérer les secrets de l’utilisateur** pour configurer les clés Personalizer. Dans ce cas, Visual Studio ouvre un fichier `secrets.json` dans lequel vous pouvez ajouter les clés comme suit :

    ```JSON
    {
      "PersonalizerApiKey": "<your personalizer key here>",
    }
    ```

## <a name="run-the-sample"></a>Exécution de l'exemple

Créez et exécutez HttpRequestFeaturesExample de l’une des méthodes suivantes :

* Visual Studio 2019 : appuyez sur **F5**
* CLI .NET Core : `dotnet build` puis `dotnet run`

Via un navigateur web, vous pouvez envoyer une demande de classement et une demande de récompense et voir leurs réponses, ainsi que les fonctionnalités de requête HTTP extraites de votre environnement.

> [!div class="mx-imgBorder"]
> ![Capture d’écran montrant un exemple de la fonctionnalité de requête HTTP dans un navigateur web.](./media/tutorial-web-app/web-app-single-page.png)

## <a name="demonstrate-the-personalizer-loop"></a>Illustrer la boucle Personalizer

1. Sélectionnez le bouton **Générer une nouvelle demande de classement** pour créer un objet JSON pour l’appel de l’API de classement. Cela crée les actions (avec fonctionnalités) et les caractéristiques contextuelles et affiche les valeurs pour vous permettre de voir à quoi ressemble le JSON.

    Pour votre future application, la génération d’actions et de fonctionnalités peut se faire sur le client, sur le serveur, sur une combinaison des deux ou avec des appels vers d’autres services.

1. Sélectionnez **Envoyer une demande de classement** pour envoyer l’objet JSON au serveur. Le serveur appelle l’API de classement Personalizer. Le serveur reçoit la réponse et renvoie l’action la mieux classée au client pour affichage.

1. Définissez la valeur de récompense, puis sélectionnez le bouton **Envoyer une demande de récompense**. Si vous ne modifiez pas la valeur de récompense, l’application du client envoie toujours la valeur `1` à Personalizer.

    > [!div class="mx-imgBorder"]
    > ![Capture d’écran montrant la section de demande de récompense.](./media/tutorial-web-app/reward-score-api-call.png)

    Pour votre future application, la génération du score de récompense peut se faire après avoir recueilli des informations sur le comportement de l’utilisateur sur le client, ainsi que la logique métier sur le serveur.

## <a name="understand-the-sample-web-app"></a>Comprendre l’exemple d’application web

L’exemple d’application web dispose d’un serveur **.NET C#** , qui gère l’ensemble des fonctionnalités ainsi que l’envoi et la réception d’appels HTTP auprès de votre point de terminaison Personalizer.

L’exemple d’application web utilise une **application cliente frontale Knockout** pour capturer des fonctionnalités et traiter des actions de l’interface utilisateur telles que cliquer sur des boutons et envoyer des données au serveur .NET.

Les sections suivantes expliquent les parties du serveur et du client qu’un développeur doit comprendre afin d’utiliser Personalizer.

## <a name="rank-api-client-application-sends-context-to-server"></a>API de classement : l’application cliente envoie le contexte au serveur

L’application cliente collecte l’ _agent utilisateur_ du navigateur de l’utilisateur.

> [!div class="mx-imgBorder"]
> ![Build and run the HTTPRequestFeaturesExample project. A browser window opens to display the single page application.](./media/tutorial-web-app/user-agent.png)

## <a name="rank-api-server-application-calls-personalizer"></a>API de classement : l’application serveur appelle Personalizer

Il s’agit d’une application web .NET classique avec une application cliente, la majeure partie du code réutilisable vous est fournie. Tout code non spécifique à Personalizer est supprimé des extraits de code suivants pour vous permettre de vous concentrer sur le code spécifique à Personalizer.

### <a name="create-personalizer-client"></a>Créer le client Personalizer

Dans le fichier **Startup.cs** du serveur, le point de terminaison et la clé Personalizer sont utilisés pour créer le client Personalizer. L’application cliente n’a pas besoin de communiquer avec Personalizer dans cette application, elle se contente de compter sur le serveur pour effectuer ces appels de Kit de développement logiciel (SDK).

Le code de démarrage .NET du serveur web est le suivant :

```csharp
using Microsoft.Azure.CognitiveServices.Personalizer;
// ... other using statements removed for brevity

namespace HttpRequestFeaturesExample
{
    public class Startup
    {
        public Startup(IConfiguration configuration)
        {
            Configuration = configuration;
        }

        public IConfiguration Configuration { get; }

        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            string personalizerApiKey = Configuration.GetSection("PersonalizerApiKey").Value;
            string personalizerEndpoint = Configuration.GetSection("PersonalizerConfiguration:ServiceEndpoint").Value;
            if (string.IsNullOrEmpty(personalizerEndpoint) || string.IsNullOrEmpty(personalizerApiKey))
            {
                throw new ArgumentException("Missing Azure Personalizer endpoint and/or api key.");
            }
            services.AddSingleton(client =>
            {
                return new PersonalizerClient(new ApiKeyServiceClientCredentials(personalizerApiKey))
                {
                    Endpoint = personalizerEndpoint
                };
            });

            services.AddMvc();
        }

        // ... code removed for brevity
    }
}
```

### <a name="select-best-action"></a>Sélectionner la meilleure action

Dans le fichier **PersonalizerController.cs** du serveur, l’API de serveur **GenerateRank** résume la préparation pour appeler l’API de classement.

* Créer un `eventId` pour l’appel de classement
* Obtenir la liste des actions
* Obtenir la liste des fonctionnalités de l’utilisateur et créer des caractéristiques contextuelles
* Facultativement, définir les actions exclues
* Appeler l’API de classement, retourner les résultats au client

```csharp
/// <summary>
/// Creates a RankRequest with user time of day, HTTP request features,
/// and taste as the context and several different foods as the actions
/// </summary>
/// <returns>RankRequest with user info</returns>
[HttpGet("GenerateRank")]
public RankRequest GenerateRank()
{
    string eventId = Guid.NewGuid().ToString();

    // Get the actions list to choose from personalizer with their features.
    IList<RankableAction> actions = GetActions();

    // Get context information from the user.
    HttpRequestFeatures httpRequestFeatures = GetHttpRequestFeaturesFromRequest(Request);
    string timeOfDayFeature = GetUsersTimeOfDay();
    string tasteFeature = GetUsersTastePreference();

    // Create current context from user specified data.
    IList<object> currentContext = new List<object>() {
            new { time = timeOfDayFeature },
            new { taste = tasteFeature },
            new { httpRequestFeatures }
    };

    // Exclude an action for personalizer ranking. This action will be held at its current position.
    IList<string> excludeActions = new List<string> { "juice" };

    // Rank the actions
    return new RankRequest(actions, currentContext, excludeActions, eventId);
}
```

Le JSON envoyé à Personalizer, contenant à la fois des actions (avec fonctionnalités) et les caractéristiques du contexte actuel, ressemble à ceci :

```json
{
    "contextFeatures": [
        {
            "time": "morning"
        },
        {
            "taste": "savory"
        },
        {
            "httpRequestFeatures": {
                "_synthetic": false,
                "MRefer": {
                    "referer": "http://localhost:51840/"
                },
                "OUserAgent": {
                    "_ua": "",
                    "_DeviceBrand": "",
                    "_DeviceFamily": "Other",
                    "_DeviceIsSpider": false,
                    "_DeviceModel": "",
                    "_OSFamily": "Windows",
                    "_OSMajor": "10",
                    "DeviceType": "Desktop"
                }
            }
        }
    ],
    "actions": [
        {
            "id": "pasta",
            "features": [
                {
                    "taste": "savory",
                    "spiceLevel": "medium"
                },
                {
                    "nutritionLevel": 5,
                    "cuisine": "italian"
                }
            ]
        },
        {
            "id": "ice cream",
            "features": [
                {
                    "taste": "sweet",
                    "spiceLevel": "none"
                },
                {
                    "nutritionalLevel": 2
                }
            ]
        },
        {
            "id": "juice",
            "features": [
                {
                    "taste": "sweet",
                    "spiceLevel": "none"
                },
                {
                    "nutritionLevel": 5
                },
                {
                    "drink": true
                }
            ]
        },
        {
            "id": "salad",
            "features": [
                {
                    "taste": "sour",
                    "spiceLevel": "low"
                },
                {
                    "nutritionLevel": 8
                }
            ]
        },
        {
            "id": "popcorn",
            "features": [
                {
                    "taste": "salty",
                    "spiceLevel": "none"
                },
                {
                    "nutritionLevel": 3
                }
            ]
        },
        {
            "id": "coffee",
            "features": [
                {
                    "taste": "bitter",
                    "spiceLevel": "none"
                },
                {
                    "nutritionLevel": 3
                },
                {
                    "drink": true
                }
            ]
        },
        {
            "id": "soup",
            "features": [
                {
                    "taste": "sour",
                    "spiceLevel": "high"
                },
                {
                    "nutritionLevel": 7
                }
            ]
        }
    ],
    "excludedActions": [
        "juice"
    ],
    "eventId": "82ac52da-4077-4c7d-b14e-190530578e75",
    "deferActivation": null
}
```

### <a name="return-personalizer-rewardactionid-to-client"></a>Retourner le rewardActionId de Personalizer au client

L’API de classement retourne la meilleure action sélectionnée **rewardActionId** au serveur.

Affichez l’action retournée dans **rewardActionId**.

```json
{
    "ranking": [
        {
            "id": "popcorn",
            "probability": 0.833333254
        },
        {
            "id": "salad",
            "probability": 0.03333333
        },
        {
            "id": "juice",
            "probability": 0
        },
        {
            "id": "soup",
            "probability": 0.03333333
        },
        {
            "id": "coffee",
            "probability": 0.03333333
        },
        {
            "id": "pasta",
            "probability": 0.03333333
        },
        {
            "id": "ice cream",
            "probability": 0.03333333
        }
    ],
    "eventId": "82ac52da-4077-4c7d-b14e-190530578e75",
    "rewardActionId": "popcorn"
}
```

### <a name="client-displays-the-rewardactionid-action"></a>Le client affiche l’action rewardActionId

Dans ce didacticiel, la valeur `rewardActionId` s’affiche.

Dans votre future application, il peut s’agir d’un texte exact, d’un bouton ou d’une section de la page web mise en surbrillance. La liste est retournée pour toute post-analyse de scores, et non pour un classement du contenu. Seul le contenu de `rewardActionId` doit s’afficher.

## <a name="reward-api-collect-information-for-reward"></a>API de récompense : collecter des informations pour les récompenses

Le [score de récompense](concept-rewards.md) doit être soigneusement planifié, tout comme le sont les fonctionnalités. Le score de récompense doit généralement être une valeur comprise entre 0 et 1. La valeur _peut_ être calculée en partie dans l’application cliente, en fonction des comportements de l’utilisateur, et en partie sur le serveur, en fonction de la logique métier et des objectifs.

Si le serveur n’appelle pas l’API de récompense dans le temps imparti par la **durée d’attente de la récompense** configurée dans le portail Azure pour votre ressource Personalizer, la **récompense par défaut** (également configurée dans le portail Azure) est utilisée pour cet événement.

Dans cet exemple d’application, vous pouvez sélectionner une valeur pour voir l’impact de la récompense sur les sélections.

## <a name="additional-ways-to-learn-from-this-sample"></a>Autres moyens d’apprendre de cet exemple

L’exemple utilise plusieurs événements basés sur le temps configurés dans le portail Azure pour votre ressource Personalizer. Jouez avec ces valeurs, puis revenez à cet exemple d’application web pour observer l’impact des modifications sur les appels de classement et de récompense :

* Temps d’attente des récompenses
* Fréquence de mise à jour du modèle

Autres paramètres avec lesquels jouer :
* Récompense par défaut
* Pourcentage d’exploration


## <a name="clean-up-resources"></a>Nettoyer les ressources

Une fois ce didacticiel terminé, nettoyez les ressources suivantes :

* Supprimez votre exemple de répertoire de projet.
* Supprimez votre ressource Personalizer : pensez à une ressource Personalizer comme étant dédiée aux actions et au contexte ; ne réutilisez la ressource que si vous utilisez toujours les aliments comme domaine d’action.


## <a name="next-steps"></a>Étapes suivantes
* [Fonctionnement de Personalizer](how-personalizer-works.md)
* [Fonctionnalités](concepts-features.md) : découvrir les concepts relatifs à l’utilisation de fonctionnalités avec les actions et le contexte
* [Récompenses](concept-rewards.md) : en savoir plus sur le calcul des récompenses