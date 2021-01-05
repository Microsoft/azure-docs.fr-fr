---
title: Personnaliser un point de terminaison HTTP dans Azure Functions
description: Découvrez comment personnaliser un point de terminaison de déclencheur HTTP dans Azure Functions
author: mattchenderson
ms.topic: conceptual
ms.date: 04/27/2020
ms.author: mahender
ms.custom: mvc
ms.openlocfilehash: d683ef92c4e8d11e9defbed5454e5849211bf8f7
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92104748"
---
# <a name="customize-an-http-endpoint-in-azure-functions"></a>Personnaliser un point de terminaison HTTP dans Azure Functions

Dans cet article, vous allez découvrir comment Azure Functions vous permet de générer des API hautement évolutives. Azure Functions est fourni avec une collection de liaisons et de déclencheurs HTTP intégrés qui permettent de créer facilement un point de terminaison dans différents langages, dont Node.JS et C# et bien plus encore. Dans cet article, vous allez personnaliser un déclencheur HTTP pour gérer des actions spécifiques dans votre conception d’API. Vous allez également préparer le développement de votre API, l’intégration avec Proxys Azure Functions et la configuration d’API factices. Ces tâches s’effectuent sur l’environnement de calcul sans serveur de Functions. Vous n’avez donc pas à vous soucier de la mise à l’échelle des ressources et vous pouvez vous concentrer uniquement sur votre logique d’API.

## <a name="prerequisites"></a>Prérequis 

[!INCLUDE [Previous quickstart note](../../includes/functions-quickstart-previous-topics.md)]

La fonction résultante sera utilisée pour le reste de cet article.

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au [portail Azure](https://portal.azure.com) avec votre compte Azure.

## <a name="customize-your-http-function"></a>Personnaliser une fonction HTTP

Par défaut, la fonction de votre déclencheur par HTTP est configurée pour accepter n’importe quelle méthode HTTP. Vous pouvez également utiliser l’URL par défaut, `http://<yourapp>.azurewebsites.net/api/<funcname>?code=<functionkey>`. Dans cette section, vous allez modifier la fonction de façon à répondre uniquement aux demandes GET avec `/api/hello`. 

1. Accédez à votre fonction sur le Portail Azure. Dans le menu de gauche, sélectionnez **Integration** (Intégration), puis sous **Trigger** (Déclencheur), sélectionnez **HTTP (req)** .

    :::image type="content" source="./media/functions-create-serverless-api/customizing-http.png" alt-text="Personnalisation d’une fonction HTTP":::

1. Utilisez les paramètres de déclencheur HTTP spécifiés dans le tableau suivant.

    | Champ | Exemple de valeur | Description |
    |---|---|---|
    | Modèle d’itinéraire | /hello | Détermine l’itinéraire utilisé pour appeler cette fonction. |
    | Niveau d’autorisation | Anonyme | Facultatif : rend votre fonction accessible sans clé API |
    | Méthodes HTTP sélectionnées | GET | Autorise uniquement l’utilisation des méthodes HTTP sélectionnés pour appeler cette fonction. |

    Vous n’avez pas inclus le préfixe du chemin d’accès de base `/api` dans le modèle d’itinéraire, car il est géré par un paramètre global.

1. Sélectionnez **Enregistrer**.

Pour en savoir plus sur la personnalisation des fonctions HTTP, consultez [Liaisons HTTP Azure Functions](./functions-bindings-http-webhook.md).

### <a name="test-your-api"></a>Tester l’API

Ensuite, testez votre fonction pour observer son fonctionnement avec la nouvelle surface d’API.
1. Dans le menu de gauche de la page de fonction, sélectionnez **Code + Test**.

1. Dans le menu supérieur, sélectionnez **Get function URL** (Obtenir l’URL de fonction), puis copiez l’URL. Vérifiez que l’URL utilise maintenant le chemin d’accès `/api/hello`.
 
1. Copiez l’URL dans un nouvel onglet du navigateur ou dans le client REST de votre choix. 

   Les navigateurs utilisent GET par défaut.
 
1. Ajoutez des paramètres à la chaîne de requête dans votre URL. 

   Par exemple : `/api/hello/?name=John`.
 
1. Appuyez sur Entrée pour vérifier qu’elle fonctionne. La réponse « *Hello John* » doit s’afficher.

1. Vous pouvez également essayer d’appeler le point de terminaison avec une autre méthode HTTP pour vérifier que la fonction n’est pas exécutée. Pour cela, utilisez un client REST, par exemple cURL, Postman ou Fiddler.

## <a name="proxies-overview"></a>Vue d’ensemble des proxys

Dans la section suivante, vous ferez apparaître votre API par le biais d’un proxy. Azure Functions Proxies vous permet de transférer les requêtes vers d’autres ressources. Vous définissez un point de terminaison HTTP comme avec le déclencheur HTTP. Toutefois, au lieu d’écrire du code à exécuter lorsque ce point de terminaison est appelé, vous fournissez une URL vers une implémentation à distance. Cela vous permet de composer plusieurs sources d’API en une seule surface d’API, facile à utiliser par les clients. Cela est utile si vous souhaitez créer votre API en tant que microservice.

Un proxy peut pointer vers n’importe quelle ressource HTTP, notamment :
- Azure Functions 
- Applications API dans [Azure App Service](../app-service/overview.md)
- Conteneurs Docker dans [App Service sur Linux](../app-service/overview.md#app-service-on-linux)
- Toute autre API hébergée

Pour en savoir plus sur les proxys, consultez [Utilisation d’Azure Functions Proxies].

## <a name="create-your-first-proxy"></a>Créer un premier proxy

Dans cette section, vous allez créer un proxy qui sert de frontend à votre API globale. 

### <a name="setting-up-the-frontend-environment"></a>Configuration de l’environnement frontend

Répétez les étapes de la page [Créer une application de fonction](./functions-create-first-azure-function.md#create-a-function-app) pour créer une application de fonction dans laquelle vous allez créer votre proxy. L’URL de cette nouvelle application sert de frontend à notre API et l’application de fonction que vous avez modifiée précédemment sert de backend.

1. Accédez à votre nouvelle application de fonction frontend sur le portail.
1. Sélectionnez **Fonctionnalités de la plateforme** et choisissez **Paramètres de l’application**.
1. Faites défiler jusqu’à **Application settings** (Paramètres de l’application) où les paires clé/valeur sont stockées, puis créez un paramètre avec la clé `HELLO_HOST`. Donnez-lui comme valeur l’hôte de votre application de fonction backend, par exemple `<YourBackendApp>.azurewebsites.net`. Cette valeur fait partie de l’URL que vous avez copiée au moment de tester votre fonction HTTP. Vous ferez référence à ce paramètre plus tard dans la configuration.

    > [!NOTE] 
    > Les paramètres de l’application sont recommandés pour la configuration d’hôte afin d’éviter une dépendance à l’environnement codé en dur pour le proxy. Grâce à eux, vous pouvez déplacer la configuration du proxy d’un environnement à l’autre, et les paramètres de l’application propres à l’environnement s’appliqueront.

1. Sélectionnez **Enregistrer**.

### <a name="creating-a-proxy-on-the-frontend"></a>Créer un proxy sur le serveur frontal

1. Revenez à votre application de fonction frontend sur le portail.

1. Dans le menu de gauche, sélectionnez **Proxies**, puis sélectionnez **Add** (Ajouter). 

1. Sur la page **New Proxy** (Nouveau proxy), utilisez les paramètres du tableau suivant, puis sélectionnez **Create** (Créer).

    | Champ | Exemple de valeur | Description |
    |---|---|---|
    | Nom | HelloProxy | Nom convivial utilisé uniquement à des fins de gestion. |
    | Modèle d’itinéraire | /api/remotehello | Détermine l’itinéraire utilisé pour appeler ce proxy. |
    | URL principale | https://%HELLO_HOST%/api/hello | Spécifie le point de terminaison vers lequel la demande doit être redirigée via proxy. |

    
    :::image type="content" source="./media/functions-create-serverless-api/creating-proxy.png" alt-text="Personnalisation d’une fonction HTTP":::

    Azure Functions Proxies ne fournit pas de préfixe du chemin de base `/api` ; celui-ci doit être inclus dans le modèle d’itinéraire. La syntaxe `%HELLO_HOST%` fait référence au paramètre d’application que vous avez créé précédemment. L’URL résolue pointera vers votre fonction d’origine.

1. Essayez votre nouveau proxy en copiant l’URL du proxy et en le testant dans le navigateur ou avec le client HTTP de votre choix :
    - Pour une fonction anonyme, utilisez : `https://YOURPROXYAPP.azurewebsites.net/api/remotehello?name="Proxies"`
    - Pour une fonction utilisant l’autorisation : `https://YOURPROXYAPP.azurewebsites.net/api/remotehello?code=YOURCODE&name="Proxies"`

## <a name="create-a-mock-api"></a>Créer une API factice

Maintenant, utilisez un proxy pour créer une API factice pour votre solution. Ce proxy permet au développement client de progresser, sans que le serveur principal soit nécessairement implémenté dans sa totalité. Dans la suite du développement, vous pouvez créer une nouvelle application de fonction qui prenne en charge cette logique et y redirige votre proxy.

Pour créer cette API fictive, nous allons créer un nouveau proxy, cette fois en utilisant [l’Éditeur App Service](https://github.com/projectkudu/kudu/wiki/App-Service-Editor). Pour commencer, accédez à votre application de fonction sur le portail. Sélectionnez **Fonctionnalités de la plateforme**, puis, sous **Outils de développement**, recherchez **Éditeur App Service**. L’Éditeur App Service s’ouvre dans un nouvel onglet.

Sélectionnez `proxies.json` dans la barre de navigation gauche. Ce fichier stocke la configuration de tous vos proxys. Si vous utilisez l’une des [méthodes de déploiement de Functions](./functions-continuous-deployment.md), vous maintenez ce fichier dans le contrôle de code source. Pour en savoir plus sur ce fichier, consultez la page [Configuration avancée des proxys](./functions-proxies.md#advanced-configuration).

Si vous avez suivi toutes les étapes jusqu’à présent, votre proxies.json doit se présenter ainsi :

```json
{
    "$schema": "http://json.schemastore.org/proxies",
    "proxies": {
        "HelloProxy": {
            "matchCondition": {
                "route": "/api/remotehello"
            },
            "backendUri": "https://%HELLO_HOST%/api/hello"
        }
    }
}
```

Vous allez maintenant ajouter votre API factice. Remplacez votre fichier proxies.json par le code suivant :

```json
{
    "$schema": "http://json.schemastore.org/proxies",
    "proxies": {
        "HelloProxy": {
            "matchCondition": {
                "route": "/api/remotehello"
            },
            "backendUri": "https://%HELLO_HOST%/api/hello"
        },
        "GetUserByName" : {
            "matchCondition": {
                "methods": [ "GET" ],
                "route": "/api/users/{username}"
            },
            "responseOverrides": {
                "response.statusCode": "200",
                "response.headers.Content-Type" : "application/json",
                "response.body": {
                    "name": "{username}",
                    "description": "Awesome developer and master of serverless APIs",
                    "skills": [
                        "Serverless",
                        "APIs",
                        "Azure",
                        "Cloud"
                    ]
                }
            }
        }
    }
}
```

Ce code ajoute un nouveau proxy, `GetUserByName`, sans la propriété `backendUri`. Au lieu d’appeler une autre ressource, il modifie la réponse par défaut des proxys par substitution de réponse. Les substitutions de demandes et de réponses peuvent également être utilisées en association avec une URL principale. Cette technique est particulièrement utile pour la redirection via proxy vers un système hérité, où vous devrez peut-être modifier les en-têtes, interroger des paramètres, etc. Pour en savoir plus sur les substitutions de demandes et de réponses, consultez la page [Modifier les demandes et les réponses dans les proxys](./functions-proxies.md).

Testez votre API factice en appelant le point de terminaison `<YourProxyApp>.azurewebsites.net/api/users/{username}` à l’aide d’un navigateur ou du client REST de votre choix. Veillez à remplacer _{username}_ par une valeur de chaîne représentant un nom d’utilisateur.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à créer et à personnaliser une API sur Azure Functions. Vous avez également appris à réunir plusieurs API, y compris factices, en une surface d’API unifiée. Vous pouvez utiliser ces techniques pour construire des API plus complexes, qui s’exécutent sur le modèle de calcul sans serveur fourni par Azure Functions.

Les références suivantes peuvent être utiles pour développer davantage votre API :

- [Liaisons HTTP Azure Functions](./functions-bindings-http-webhook.md)
- [Utilisation d’Azure Functions Proxies]
- [Documenter une API Azure Functions (préversion)](./functions-openapi-definition.md)


[Create your first function]: ./functions-create-first-azure-function.md
[Utilisation d’Azure Functions Proxies]: ./functions-proxies.md