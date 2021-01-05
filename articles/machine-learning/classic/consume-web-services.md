---
title: 'ML Studio (classique) : Utiliser des services web – Azure'
description: Une fois qu’un service de machine learning a été déployé à partir d’Azure Machine Learning Studio (classique), le service web RESTful peut être utilisé soit en tant que service de requête-réponse en temps réel, soit en tant que service d’exécution par lot.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: how-to
author: likebupt
ms.author: keli19
ms.custom: seodec18, devx-track-python, devx-track-js, devx-track-csharp
ms.date: 05/29/2020
ms.openlocfilehash: eaf0131877e7a333fe2a6f157523da5ad7bcf07b
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95499997"
---
# <a name="how-to-consume-a-machine-learning-studio-classic-web-service"></a>Guide pratique pour utiliser un service web Machine Learning Studio (classique)

**S’APPLIQUE À :**  ![Cette coche signifie que cet article s’applique à Machine Learning Studio (classique).](../../../includes/media/aml-applies-to-skus/yes.png) Machine Learning Studio (classique)   ![Cette coche signifie que cet article s’applique à Azure Machine Learning.](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio)


Une fois que vous avez déployé un modèle prédictif Azure Machine Learning Studio (classique) en tant que service web, vous pouvez utiliser une API REST pour lui envoyer des données et obtenir des prédictions. Vous pouvez envoyer les données en temps réel ou par lot.

Vous trouverez des informations supplémentaires sur la création et le déploiement d’un service web Machine Learning à l’aide de Machine Learning Studio (classique) dans les pages suivantes :

* Pour accéder à un didacticiel sur la création d’une expérience dans Machine Learning Studio (classique), voir [Créer votre première expérience](create-experiment.md).
* Pour plus d’informations sur le déploiement d’un service web, consultez [Déployer un service web Machine Learning](deploy-a-machine-learning-web-service.md).
* Pour plus d’informations sur Machine Learning en général, consultez le [Centre de documentation Machine Learning](https://azure.microsoft.com/documentation/services/machine-learning/).



## <a name="overview"></a>Vue d’ensemble
Grâce au service web Microsoft Azure Machine Learning, une application externe peut communiquer avec le modèle de notation de workflow Machine Learning et ce, en temps réel. Un appel du service web Machine Learning renvoie les résultats d’une prédiction à une application externe. Pour créer cet appel, vous transmettez une clé API créée quand vous déployez une prédiction. Le service web Machine Learning est basé sur l’architecture REST, souvent choisie pour les projets de programmation web.

Microsoft Azure Machine Learning Studio (classique) propose deux types de service :

* Service de requête-réponse (Request-Response Service, RRS) : service hautement évolutif, à faible latence, qui constitue une interface pour les modèles sans état créés et déployés à partir de Machine Learning Studio (classique).
* Service d’exécution de lot (Batch Execution Service, BES) : service asynchrone qui effectue la notation d’un lot pour les enregistrements de données.

Pour plus d’informations sur les services web Machine Learning, consultez [Déployer un service web Machine Learning](deploy-a-machine-learning-web-service.md).

## <a name="get-an-authorization-key"></a>Obtenir une clé d’autorisation
Quand vous déployez votre expérimentation, les clés API sont générées pour le service web. Vous pouvez récupérer les clés de plusieurs emplacements.

### <a name="from-the-microsoft-azure-machine-learning-web-services-portal"></a>À partir du portail des services web Microsoft Azure Machine Learning
Connectez-vous au [portail des services web Microsoft Azure Machine Learning](https://services.azureml.net).

Pour récupérer la clé API pour un nouveau service web Machine Learning :

1. Dans le portail Services web Microsoft Azure Machine Learning, cliquez sur **Services web** dans le menu du haut.
2. Cliquez sur le service web pour lequel vous souhaitez récupérer la clé.
3. Cliquez sur **Consommer** dans le menu du haut.
4. Copiez et enregistrez la **clé primaire**.

Pour récupérer la clé API pour un service web Machine Learning classique :

1. Dans le portail Services web Microsoft Azure Machine Learning, cliquez sur **Services web classiques** dans le menu du haut.
2. Cliquez sur le service web que vous utilisez.
3. Cliquez sur le point de terminaison pour lequel vous souhaitez récupérer la clé.
4. Cliquez sur **Consommer** dans le menu du haut.
5. Copiez et enregistrez la **clé primaire**.

### <a name="classic-web-service"></a>Service web classique
 Vous pouvez également récupérer une clé pour un service web classique par le biais de Machine Learning Studio (classique).

#### <a name="machine-learning-studio-classic"></a>Machine Learning Studio (classique)
1. Dans Machine Learning Studio (cliquez), cliquez sur l’option **SERVICES WEB** à gauche.
2. Cliquez sur un service web. La **clé API** figure sur l’onglet **TABLEAU DE BORD**.

## <a name="connect-to-a-machine-learning-web-service"></a><a id="connect"></a>Se connecter à un service web Machine Learning
Vous pouvez vous connecter à un service web Machine Learning à l’aide de n’importe quel langage de programmation qui prend en charge les requêtes et réponses HTTP. Vous pouvez consulter des exemples en C#, Python et R sur l’une des pages d’aide du service web Machine Learning.

**Aide de l’API Machine Learning** L’aide de l’API Machine Learning est créée quand vous déployez un service web. Consultez le [Tutoriel 3 : Déployer un modèle de risque de crédit](tutorial-part3-credit-risk-deploy.md).
L’aide sur l’API Machine Learning contient des détails sur un service web de prédiction.

1. Cliquez sur le service web que vous utilisez.
2. Cliquez sur le point de terminaison pour lequel vous souhaitez afficher la page d’aide de l’API.
3. Cliquez sur **Consommer** dans le menu du haut.
4. Cliquez sur la **page d’aide de l’API** sous le point de terminaison Requête-réponse ou Exécution par lot.

**Pour afficher l’aide de l’API Machine Learning pour un nouveau service web**

Dans le [portail des services web Azure Machine Learning](https://services.azureml.net/) :

1. Cliquez sur **SERVICES WEB** dans le menu du haut.
2. Cliquez sur le service web pour lequel vous souhaitez récupérer la clé.

Cliquez sur **Utiliser le service web** pour obtenir les URI des services requête-réponse et d’exécution par lots, ainsi que des exemples de code en C#, R et Python.

Cliquez sur **API Swagger** pour obtenir une documentation basée sur Swagger pour les API appelées à partir de l’URI fourni.

### <a name="c-sample"></a>Exemple de code C#
Pour vous connecter à un service web Machine Learning, utilisez un **HttpClient** qui transmet l’élément ScoreData. ScoreData contient un FeatureVector, un vecteur à n dimensions des fonctionnalités numériques qui représente le ScoreData. Vous vous authentifiez auprès du service Machine Learning au moyen d’une clé API.

Pour vous connecter à un service web Machine Learning, le package NuGet **Microsoft.AspNet.WebApi.Client** doit être installé.

**Installer le package NuGet Microsoft.AspNet.WebApi.Client dans Microsoft Visual Studio**

1. Publier l’expérience Téléchargement d’un jeu de données depuis l’UCI : Service web du jeu de données de classe Adult 2.
2. Cliquez sur **Outils** > **Gestionnaire de package NuGet** > **Console du gestionnaire de package**.
3. Choisissez l'élément **Install-Package Microsoft.AspNet.WebApi.Client**.

**Pour exécuter l’exemple de code**

1. Publiez l’expérience « Exemple 1 : Téléchargement d’un jeu de données depuis l’UCI : jeu de données de classe Adult 2 », inclus dans la collection d’exemples Machine Learning.
2. Attribuez l’élément apiKey avec la clé à partir d’un service web. Voir **Générez une clé d’autorisation** ci-dessus.
3. Affectez l’élément serviceUri avec l’URI de requête.

**Voici à quoi ressemble une requête complète.**
```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Net.Http;
using System.Net.Http.Formatting;
using System.Net.Http.Headers;
using System.Text;
using System.Threading.Tasks;

namespace CallRequestResponseService
{
    class Program
    {
        static void Main(string[] args)
        {
            InvokeRequestResponseService().Wait();
        }

        static async Task InvokeRequestResponseService()
        {
            using (var client = new HttpClient())
            {
                var scoreRequest = new
                {
                    Inputs = new Dictionary<string, List<Dictionary<string, string>>> () {
                        {
                            "input1",
                            // Replace columns labels with those used in your dataset
                            new List<Dictionary<string, string>>(){new Dictionary<string, string>(){
                                    {
                                        "column1", "value1"
                                    },
                                    {
                                        "column2", "value2"
                                    },
                                    {
                                        "column3", "value3"
                                    }
                                }
                            }
                        },
                    },
                    GlobalParameters = new Dictionary<string, string>() {}
                };

                // Replace these values with your API key and URI found on https://services.azureml.net/
                const string apiKey = "<your-api-key>"; 
                const string apiUri = "<your-api-uri>";
                
                client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue( "Bearer", apiKey);
                client.BaseAddress = new Uri(apiUri);

                // WARNING: The 'await' statement below can result in a deadlock
                // if you are calling this code from the UI thread of an ASP.NET application.
                // One way to address this would be to call ConfigureAwait(false)
                // so that the execution does not attempt to resume on the original context.
                // For instance, replace code such as:
                //      result = await DoSomeTask()
                // with the following:
                //      result = await DoSomeTask().ConfigureAwait(false)

                HttpResponseMessage response = await client.PostAsJsonAsync("", scoreRequest);

                if (response.IsSuccessStatusCode)
                {
                    string result = await response.Content.ReadAsStringAsync();
                    Console.WriteLine("Result: {0}", result);
                }
                else
                {
                    Console.WriteLine(string.Format("The request failed with status code: {0}", response.StatusCode));

                    // Print the headers - they include the request ID and the timestamp,
                    // which are useful for debugging the failure
                    Console.WriteLine(response.Headers.ToString());

                    string responseContent = await response.Content.ReadAsStringAsync();
                    Console.WriteLine(responseContent);
                }
            }
        }
    }
}
```

### <a name="python-sample"></a>Exemple de code Python
Pour vous connecter à un service web Machine Learning, utilisez la bibliothèque **urllib2** pour Python 2.X et la bibliothèque **urllib.request** pour Python 3.X. Vous allez passer ScoreData, qui contient un FeatureVector, un vecteur à n dimensions des fonctionnalités numériques qui représente le ScoreData. Vous vous authentifiez auprès du service Machine Learning au moyen d’une clé API.

**Pour exécuter l’exemple de code**

1. Déployer l’exemple 1 : Téléchargement d’un jeu de données depuis l’UCI : jeu de données de classe Adult 2 », inclus dans la collection d’exemples Machine Learning.
2. Attribuez l’élément apiKey avec la clé à partir d’un service web. Voir la section **Obtenir une clé d’autorisation** au début de cet article.
3. Affectez l’élément serviceUri avec l’URI de requête.

**Voici à quoi ressemble une requête complète.**
```python
import urllib2 # urllib.request and urllib.error for Python 3.X
import json

data = {
    "Inputs": {
        "input1":
        [
            {
                'column1': "value1",   
                'column2': "value2",   
                'column3': "value3"
            }
        ],
    },
    "GlobalParameters":  {}
}

body = str.encode(json.dumps(data))

# Replace this with the URI and API Key for your web service
url = '<your-api-uri>'
api_key = '<your-api-key>'
headers = {'Content-Type':'application/json', 'Authorization':('Bearer '+ api_key)}

# "urllib.request.Request(url, body, headers)" for Python 3.X
req = urllib2.Request(url, body, headers)

try:
    # "urllib.request.urlopen(req)" for Python 3.X
    response = urllib2.urlopen(req)

    result = response.read()
    print(result)
# "urllib.error.HTTPError as error" for Python 3.X
except urllib2.HTTPError, error: 
    print("The request failed with status code: " + str(error.code))

    # Print the headers - they include the request ID and the timestamp, which are useful for debugging the failure
    print(error.info())
    print(json.loads(error.read())) 
```

### <a name="r-sample"></a>Exemple R

Pour vous connecter à un service web Machine Learning, utilisez les bibliothèques **RCurl** et **rjson** pour effectuer la requête et traiter la réponse JSON retournée. Vous allez passer ScoreData, qui contient un FeatureVector, un vecteur à n dimensions des fonctionnalités numériques qui représente le ScoreData. Vous vous authentifiez auprès du service Machine Learning au moyen d’une clé API.

**Voici à quoi ressemble une requête complète.**
```r
library("curl")
library("httr")
library("rjson")

requestFailed = function(response) {
    return (response$status_code >= 400)
}

printHttpResult = function(response, result) {
    if (requestFailed(response)) {
        print(paste("The request failed with status code:", response$status_code, sep=" "))
    
        # Print the headers - they include the request ID and the timestamp, which are useful for debugging the failure
        print(response$headers)
    }
    
    print("Result:") 
    print(fromJSON(result))  
}

req = list(
        Inputs = list( 
            "input1" = list(
                "ColumnNames" = list("Col1", "Col2", "Col3"),
                "Values" = list( list( "0", "value", "0" ),  list( "0", "value", "0" )  )
            )                ),
        GlobalParameters = setNames(fromJSON('{}'), character(0))
)

body = enc2utf8(toJSON(req))
api_key = "abc123" # Replace this with the API key for the web service
authz_hdr = paste('Bearer', api_key, sep=' ')

response = POST(url= "<your-api-uri>",
        add_headers("Content-Type" = "application/json", "Authorization" = authz_hdr),
        body = body)

result = content(response, type="text", encoding="UTF-8")

printHttpResult(response, result)
```

### <a name="javascript-sample"></a>Exemple JavaScript

Pour vous connecter à un service web Machine Learning, utilisez le package npm **request** dans votre projet. Vous allez également utiliser l’objet `JSON` pour mettre en forme votre entrée et analyser le résultat. Effectuez l’installation en utilisant `npm install request --save`, ou ajoutez `"request": "*"` à votre package.json sous `dependencies` et exécutez `npm install`.

**Voici à quoi ressemble une requête complète.**
```js
let req = require("request");

const uri = "<your-api-uri>";
const apiKey = "<your-api-key>";

let data = {
    "Inputs": {
        "input1":
        [
            {
                'column1': "value1",
                'column2': "value2",
                'column3': "value3"
            }
        ],
    },
    "GlobalParameters": {}
}

const options = {
    uri: uri,
    method: "POST",
    headers: {
        "Content-Type": "application/json",
        "Authorization": "Bearer " + apiKey,
    },
    body: JSON.stringify(data)
}

req(options, (err, res, body) => {
    if (!err && res.statusCode == 200) {
        console.log(body);
    } else {
        console.log("The request failed with status code: " + res.statusCode);
    }
});
```