---
title: 'Tutoriel : Envoyer un e-mail avec Logic Apps'
description: Découvrez comment appeler des processus métier à partir de votre application App Service. Envoyez des e-mails, postez des tweets, publiez sur Facebook, ajoutez des membres à des listes de diffusion, et faites bien d’autres actions.
ms.topic: tutorial
ms.date: 04/08/2020
ms.custom: devx-track-csharp, mvc
ms.openlocfilehash: 8072a941cd89290af3e25cc63c4fccccce705df9
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95014660"
---
# <a name="tutorial-send-email-and-invoke-other-business-processes-from-app-service"></a>Tutoriel : Envoyer un e-mail et appeler d’autres processus métier à partir d’App Service

Dans ce tutoriel, vous allez apprendre à intégrer votre application App Service à vos processus métier. Cette intégration est courante dans les scénarios d’application web suivants :

- Envoyer un e-mail de confirmation d’une transaction
- Ajouter un utilisateur à un groupe Facebook
- Se connecter à des systèmes tiers comme SAP, Salesforce, etc.
- Échanger des messages B2B standard

Dans ce tutoriel, vous envoyez des e-mails Gmail à partir de votre application App Service en utilisant [Azure Logic Apps](../logic-apps/logic-apps-overview.md). Il existe d’autres méthodes possibles pour envoyer des e-mails à partir d’une application web, notamment la configuration SMTP fournie par votre framework de langage. Toutefois, Logic Apps apporte beaucoup plus de puissance à votre application App Service sans pour cela rendre votre code plus complexe. Logic Apps fournit une interface de configuration simple qui convient aux intégrations métier les plus courantes, et votre application peut les appeler à tout moment par une requête HTTP.

## <a name="prerequisite"></a>Configuration requise

Déployez une application avec le framework de langage de votre choix dans App Service. Pour suivre un tutoriel sur le déploiement d’un exemple d’application, voir ci-dessous :

### <a name="aspnet"></a>[ASP.NET](#tab/dotnet)

[Tutoriel : Créer une application ASP.NET dans Azure avec SQL Database](app-service-web-tutorial-dotnet-sqldatabase.md)

### <a name="aspnet-core"></a>[ASP.NET Core](#tab/dotnetcore)

[Tutoriel : Créer une application ASP.NET Core et SQL Database dans Azure App Service](tutorial-dotnetcore-sqldb-app.md)

### <a name="nodejs"></a>[Node.JS](#tab/node)

[Tutoriel : Créer une application Node.js et MongoDB dans Azure](tutorial-nodejs-mongodb-app.md)

### <a name="php"></a>[PHP](#tab/php)

[Tutoriel : Créer une application PHP et MySQL dans Azure](tutorial-php-mysql-app.md)

### <a name="python"></a>[Python](#tab/python)

[Tutoriel : Exécuter une application web Python (Django) avec PostgreSQL dans Azure App Service](tutorial-python-postgresql-app.md)

### <a name="ruby"></a>[Ruby](#tab/ruby)

[Générer une application Ruby et Postgres dans Azure App Service sur Linux](tutorial-ruby-postgres-app.md)

---

## <a name="create-the-logic-app"></a>Créer l’application logique

1. Dans le [Portail Azure](https://portal.azure.com), créez une application logique vide en suivant les instructions données dans [Créer votre application logique](../logic-apps/quickstart-create-first-logic-app-workflow.md#create-your-logic-app). Lorsque vous voyez la fenêtre **Concepteur Logic Apps** s’afficher, revenez dans ce tutoriel.
1. Dans la page de démarrage du concepteur Logic Apps, sélectionnez **Lors de la réception d’une demande HTTP** sous **Démarrer avec un déclencheur courant**.

    ![Capture d’écran montrant la page de démarrage du Concepteur Logic Apps avec Lors de la réception d’une demande HTTP mis en évidence.](./media/tutorial-send-email/receive-http-request.png)
1. Dans la boîte de dialogue **Lors de la réception d’une demande HTTP**, sélectionnez **Utiliser l’exemple de charge utile pour générer le schéma**.

    ![Capture d’écran montrant la boîte de dialogue Lors de la réception d’une demande HTTP et l’option Utiliser l’exemple de charge utile pour générer le schéma sélectionnée. ](./media/tutorial-send-email/generate-schema-with-payload.png)

1. Copiez l’exemple JSON suivant dans la zone de texte et sélectionnez **Terminé**.

    ```json
    {
        "task": "<description>",
        "due": "<date>",
        "email": "<email-address>"
    }
    ```

    Le schéma est maintenant généré pour les données demandées souhaitées. Dans la pratique, vous pouvez simplement capturer les données demandées réelles générées par votre code d’application et laisser Azure générer le schéma JSON pour vous. 
1. En haut du concepteur Logic Apps, sélectionnez **Enregistrer**. 

    Vous voyez maintenant l’URL de votre déclencheur de requête HTTP. Sélectionnez l’icône de copie pour copier cette URL en vue d’une utilisation ultérieure.

    ![Capture d’écran montrant l’icône de copie pour copier l’URL de votre déclencheur de requête HTTP.](./media/tutorial-send-email/http-request-url.png)

    Cette définition de requête HTTP est un déclencheur des actions que vous souhaitez faire dans cette application logique, qu’il s’agisse de Gmail ou d’une autre. Plus tard, vous appellerez cette URL dans votre application App Service. Pour plus d’informations sur le déclencheur de requête, consultez les [informations de référence sur les requêtes/réponses HTTP](../connectors/connectors-native-reqres.md).

1. En bas du concepteur, cliquez sur **Nouvelle étape**, tapez **Gmail** dans la zone de recherche des actions, puis recherchez et sélectionnez **Envoyer un e-mail (V2)** .
    
    > [!TIP]
    > Vous pouvez rechercher d’autres types d’intégrations comme SendGrid, MailChimp, Microsoft 365 et SalesForce. Pour plus d’informations, consultez la [documentation Logic Apps](../logic-apps/index.yml).

1. Dans la boîte de dialogue **Gmail**, sélectionnez **Connexion**, puis connectez-vous au compte Gmail à utiliser pour envoyer l’e-mail.

    ![Capture d’écran qui montre la boîte de dialogue Gmail que vous utilisez pour vous connecter au compte Gmail à partir duquel vous souhaitez envoyer un e-mail.](./media/tutorial-send-email/gmail-sign-in.png)

1. Une fois que vous êtes connecté, cliquez dans la zone de texte **À**. La boîte de dialogue de contenu dynamique s’ouvre automatiquement.

1. À côté de l’action **Lors de la réception d’une demande HTTP**, sélectionnez **Voir plus**.

    ![Capture d’écran qui montre le bouton Voir plus en regard de Lors de la réception d’une demande HTTP.](./media/tutorial-send-email/expand-dynamic-content.png)

    Vous voyez maintenant les trois propriétés des exemples de données JSON que vous avez utilisées précédemment. Dans cette étape, vous utilisez ces propriétés de la requête HTTP pour créer un e-mail.
1. Étant donné que vous cherchez à sélectionner la valeur du champ **À**, choisissez la propriété **email**. Si vous le souhaitez, désactivez la boîte de dialogue de contenu dynamique en cliquant sur **Ajouter du contenu dynamique**.

    ![Capture d’écran montrant l’option email et l’option Ajouter du contenu dynamique mise en évidence.](./media/tutorial-send-email/hide-dynamic-content.png)

1. Dans la liste déroulante **Ajouter un nouveau paramètre**, sélectionnez **Objet** et **Corps**.

1. Cliquez dans la zone de texte **Objet** et, de la même façon, choisissez la propriété **task**. En laissant le curseur dans la zone **Objet**, tapez *created* (créé). 

1. Cliquez dans la zone **Objet** et, de la même façon, choisissez la propriété **due**. Déplacez le curseur à gauche de **due** et tapez *This work item is due on* (Cet élément de travail est dû le).

    > [!TIP]
    > Si vous souhaitez modifier le contenu HTML directement dans le corps de l’e-mail, sélectionnez **Mode Code** en haut de la fenêtre Concepteur Logic Apps. Veillez simplement à conserver le code de contenu dynamique (par exemple, `@{triggerBody()?['due']}`)
    >
    > ![Capture d’écran montrant le mode code permettant d’afficher le contenu HTML directement dans le corps de l’e-mail.](./media/tutorial-send-email/edit-rich-html-email.png) 

1. Ensuite, ajoutez une réponse HTTP asynchrone au déclencheur HTTP. Entre le déclencheur HTTP et l’action Gmail, cliquez sur le signe **+** , puis sélectionnez **Ajouter une branche parallèle**.

    ![Capture d’écran qui montre le signe + et l’option Ajouter une branche parallèle mise en évidence.](./media/tutorial-send-email/add-http-response.png)

1. Dans la zone de recherche, recherchez **réponse**, puis sélectionnez l’action **Réponse**.

    ![Capture d’écran montrant le volet de recherche et l’action Réponse mise en évidence.](./media/tutorial-send-email/choose-response-action.png)

    Par défaut, l’action Réponse envoie une réponse HTTP 200. Cela est suffisant dans le cadre de ce tutoriel. Pour plus d’informations, consultez les [informations de référence sur les requêtes/réponses HTTP](../connectors/connectors-native-reqres.md).

1. En haut du concepteur Logic Apps, sélectionnez à nouveau **Enregistrer**. 

## <a name="add-http-request-code-to-app"></a>Ajouter un code de requête HTTP à l’application

Assurez-vous d’avoir précédemment copié l’URL du déclencheur de requête HTTP. Du fait que l’URL contient des informations sensibles, il est recommandé de ne pas la placer directement dans le code. Avec App Service, vous pouvez à la place la référencer en tant que variable d’environnement, en configurant les paramètres de l’application. 

Dans [Cloud Shell](https://shell.azure.com), créez le paramètre d’application à l’aide de la commande suivante (remplacez *\<app-name>* , *\<resource-group-name>* et *\<logic-app-url>* ) :

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings LOGIC_APP_URL="<your-logic-app-url>"
```

Dans votre code, effectuez une requête HTTP POST standard sur l’URL dans l’un des langages clients HTTP disponibles pour votre framework de langage, en respectant la configuration suivante :

- Le corps de la requête contient le même format JSON que celui que vous avez fourni à votre application logique :

    ```json
    {
        "task": "<description>",
        "due": "<date>",
        "email": "<email-address>"
    }
    ```

- La requête contient l’en-tête `Content-Type: application/json`. 
- Pour optimiser les performances, envoyez la requête de manière asynchrone si possible.

Pour voir un exemple, cliquez sur l’onglet du framework/langage de votre choix ci-dessous.

### <a name="aspnet"></a>[ASP.NET](#tab/dotnet)

Dans ASP.NET, vous pouvez envoyer la requête HTTP POST avec la classe [System.Net.Http.HttpClient](/dotnet/api/system.net.http.httpclient). Par exemple :

```csharp
// requires using System.Net.Http;
var client = new HttpClient();
// requires using System.Text.Json;
var jsonData = JsonSerializer.Serialize(new
{
    email = "a-valid@emailaddress.com",
    due = "4/1/2020",
    task = "My new task!"
});

HttpResponseMessage result = await client.PostAsync(
    // requires using System.Configuration;
    ConfigurationManager.AppSettings["LOGIC_APP_URL"],
    new StringContent(jsonData, Encoding.UTF8, "application/json"));
    
var statusCode = result.StatusCode.ToString();
```

Si vous testez ce code sur l’exemple d’application du [Tutoriel : Créer une application ASP.NET dans Azure avec SQL Database](app-service-web-tutorial-dotnet-sqldatabase.md), vous pouvez l’utiliser pour envoyer une confirmation par e-mail dans l’[action Créer](https://github.com/Azure-Samples/dotnet-sqldb-tutorial/blob/master/DotNetAppSqlDb/Controllers/TodosController.cs#L52-L63), après l’ajout de l’élément `Todo`. Pour utiliser le code asynchrone ci-dessus, convertissez l’action Créer en type asynchrone.

### <a name="aspnet-core"></a>[ASP.NET Core](#tab/dotnetcore)

Dans ASP.NET Core, vous pouvez envoyer la requête HTTP POST avec la classe [System.Net.Http.HttpClient](/dotnet/api/system.net.http.httpclient). Par exemple :

```csharp
// requires using System.Net.Http;
var client = new HttpClient();
// requires using System.Text.Json;
var jsonData = JsonSerializer.Serialize(new
{
    email = "a-valid@emailaddress.com",
    due = "4/1/2020",
    task = "My new task!"
});

HttpResponseMessage result = await client.PostAsync(
    // Requires DI configuration to access app settings. See https://docs.microsoft.com/azure/app-service/configure-language-dotnetcore#access-environment-variables
    _configuration["LOGIC_APP_URL"],
    new StringContent(jsonData, Encoding.UTF8, "application/json"));
    
var statusCode = result.StatusCode.ToString();
```

> [!NOTE]
> Ce code a été écrit dans un souci de simplifier la démonstration. Dans la pratique, n’instanciez pas un objet `HttpClient` pour chaque requête. Suivez les instructions fournies dans [Utiliser IHttpClientFactory pour implémenter des requêtes HTTP résilientes](/dotnet/architecture/microservices/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests).

Si vous testez ce code sur l’exemple d’application du [Tutoriel : Créer une application ASP.NET Core et SQL Database dans Azure App Service](tutorial-dotnetcore-sqldb-app.md), vous pouvez l’utiliser pour envoyer une confirmation par e-mail dans l’[action create](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial/blob/master/Controllers/TodosController.cs#L56-L65), après l’ajout de l’élément `Todo`.

### <a name="nodejs"></a>[Node.JS](#tab/node)

Dans Node.js, vous pouvez facilement envoyer la requête HTTP POST avec un package npm comme [axios](https://www.npmjs.com/package/axios). Par exemple :

```javascript
// Requires npm install --save axios
const axios = require('axios');

var jsonData = {
        email: "a-valid@emailaddress.com",
        due: "4/1/2020",
        task: "My new task!"
};

(async function(data) {
    try {
        const response = await axios.post(process.env.LOGIC_APP_URL, jsonData);
        console.log(response.status);
    } catch (error) {
        console.log(error);
    }
})(jsonData);

```

Si vous testez ce code sur l’exemple d’application du [Tutoriel : Créer une application Node.js et MongoDB dans Azure](tutorial-nodejs-mongodb-app.md), vous pouvez l’utiliser pour envoyer une confirmation par e-mail dans la [fonction create](https://github.com/Azure-Samples/meanjs/blob/master/modules/articles/server/controllers/articles.server.controller.js#L14-L27), après que [l’article a été enregistré avec succès](https://github.com/Azure-Samples/meanjs/blob/master/modules/articles/server/controllers/articles.server.controller.js#L24).

### <a name="php"></a>[PHP](#tab/php)

Dans PHP, vous pouvez envoyer facilement la requête HTTP POST avec [Guzzle](http://docs.guzzlephp.org/en/stable/index.html). Par exemple :

```php
// Requires composer require guzzlehttp/guzzle:~6.0
use GuzzleHttp\Client;
...
$client = new Client();
$options = [
    'json' => [ 
        'email' => 'a-valid@emailaddress.com',
        'due' => '4/1/2020',
        'task' => "My new task!"
    ]
];

$promise = $client-> postAsync(getenv($LOGIC_APP_URL), $options)->then( 
    function ($response) {
        return $response->getStatusCode();
    }, function ($exception) {
        return $exception->getResponse();
    }
);

$response = $promise->wait();
// Requires Laravel to run Log::info(). Check the documentation of your preferred framework for logging instructions.
Log::info(print_r($response, TRUE));
```

Si vous testez ce code sur l’exemple d’application du [Tutoriel : Créer une application PHP et MySQL dans Azure](tutorial-php-mysql-app.md), vous pouvez l’utiliser pour envoyer une confirmation par e-mail dans la [fonction Route::post](https://github.com/Azure-Samples/laravel-tasks/blob/master/routes/web.php#L30-L48), juste avant l’instruction return.

### <a name="python"></a>[Python](#tab/python)

Dans Python, vous pouvez envoyer facilement la requête HTTP POST avec [requests](https://pypi.org/project/requests/). Par exemple :

```python
# Requires pip install requests && pip freeze > requirements.txt
import requests
...
payload = {
    "email": "a-valid@emailaddress.com",
    "due": "4/1/2020",
    "task": "My new task!"
}
response = requests.post("https://prod-112.westeurope.logic.azure.com:443/workfl$
print(response.status_code)
```
<!-- ```python
# Requires pip install aiohttp && pip freeze > requirements.txt
import aiohttp
...
payload = {
        'email': 'a-valid@emailaddress.com',
        'due': '4/1/2020',
        'task': 'My new task!'
}
async with aiohttp.post('http://httpbin.org/post', data=json.dump(payload)) as resp:
    print(await resp.status())
``` -->

Si vous testez ce code sur l’exemple d’application du [Tutoriel : Exécuter une application web Python (Django) avec PostgreSQL dans Azure App Service](tutorial-python-postgresql-app.md), vous pouvez l’utiliser pour envoyer une confirmation par e-mail dans la [fonction Route::post](https://github.com/Azure-Samples/laravel-tasks/blob/master/routes/web.php#L30-L48), juste avant l’instruction return.

### <a name="ruby"></a>[Ruby](#tab/ruby)

Dans Ruby, vous pouvez envoyer facilement la requête HTTP POST avec JSONClient. Par exemple :

```ruby
clnt = JSONClient.new
body = { 
    'email' => 'a-valid@emailaddress.com',
    'due' => '4/1/2020',
    'task' => "My new task!"
}

connection = clnt.post_async(ENV['LOGIC_APP_URL'], body)
```

Si vous testez ce code sur l’exemple d’application du tutoriel [Générer une application Ruby et Postgres dans Azure App Service sur Linux](tutorial-ruby-postgres-app.md), vous pouvez l’utiliser pour envoyer une confirmation par e-mail dans l’action [create](https://github.com/Azure-Samples/rubyrails-tasks/blob/master/app/controllers/tasks_controller.rb#L26-L38), [après que @task.save a été exécuté avec succès](https://github.com/Azure-Samples/rubyrails-tasks/blob/master/app/controllers/tasks_controller.rb#L30).

---

## <a name="more-resources"></a>Plus de ressources

[Tutoriel : Héberger une API RESTful avec CORS dans Azure App Service](app-service-web-tutorial-rest-api.md)  
[Informations de référence sur les requêtes/réponses HTTP pour Logic Apps](../connectors/connectors-native-reqres.md)  
[Démarrage rapide : Créer votre premier workflow à l’aide d’Azure Logic Apps - Portail Azure](../logic-apps/quickstart-create-first-logic-app-workflow.md)