---
title: Exposer vos fonctions avec OpenAPI à l’aide de Gestion des API Azure
description: Créez une définition OpenAPI permettant aux autres applications et services d’appeler votre fonction dans Azure.
ms.topic: tutorial
ms.date: 04/21/2020
ms.reviewer: sunayv
ms.custom: devx-track-csharp, mvc, cc996988-fb4f-47, references_regions
ms.openlocfilehash: 9083ff7d8f65c68ce8d173973a4eda650ac355aa
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88212916"
---
# <a name="create-an-openapi-definition-for-a-serverless-api-using-azure-api-management"></a>Créer une définition OpenAPI pour une API serverless à l’aide de Gestion des API Azure

Les API REST sont souvent décrites à l’aide d’une définition OpenAPI. Cette définition contient des informations sur les opérations qui sont disponibles dans une API et sur la façon dont les données de demande et de réponse de l’API doivent être structurées.

Dans ce didacticiel, vous allez créer une fonction qui détermine si la réparation d’urgence d’une éolienne est rentable. Ensuite, vous allez créer une définition OpenAPI pour l’application de fonction en utilisant [Gestion des API Azure](../api-management/api-management-key-concepts.md) afin que la fonction puisse être appelée à partir d’autres applications et services.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer une fonction dans Azure
> * Générer une définition OpenAPI à l’aide de Gestion des API Azure
> * Tester la définition en appelant la fonction
> * Télécharger la définition OpenAPI

## <a name="create-a-function-app"></a>Créer une application de fonction

Vous devez disposer d’une Function App pour héberger l’exécution de vos fonctions. Une application de fonctions vous permet de regrouper des fonctions en une unité logique pour faciliter la gestion, le déploiement et le partage des ressources.

[!INCLUDE [Create function app Azure portal](../../includes/functions-create-function-app-portal.md)]

## <a name="create-the-function"></a>Création de la fonction

Ce didacticiel utilise une fonction déclenchée via HTTP qui accepte deux paramètres :

* Durée estimée pour réparer une éolienne réparer, exprimée en heures.
* La capacité de l’éolienne, exprimée en kilowatts. 

La fonction calcule ensuite le coût de la réparation et les revenus engendrés par 24 heures de fonctionnement de l’éolienne. Pour créer une fonction déclenchée via HTTP dans le [portail Azure](https://portal.azure.com) :

1. Dans votre application de fonction, sélectionnez **Fonctions** dans le menu de gauche, puis **Ajouter** dans le menu supérieur.

1. Dans la fenêtre **Nouvelle fonction**, sélectionnez **Déclencheur http**.

1. Pour **Nouvelle fonction**, entrez `TurbineRepair`. 

1. Choisissez **Fonction** dans la liste déroulante **[Niveau d’autorisation](functions-bindings-http-webhook-trigger.md#http-auth)** , puis sélectionnez **Créer une fonction**.

    :::image type="content" source="media/functions-openapi-definition/select-http-trigger-openapi.png" alt-text="Créer une fonction HTTP pour OpenAPI&quot;:::

1. Sélectionnez **Code + test**, puis sélectionnez **run.csx** dans la liste déroulante. Remplacez le contenu du fichier de script C# run.csx par le code suivant, puis choisissez **Enregistrer** :

    ```csharp
    #r &quot;Newtonsoft.Json"
    
    using System.Net;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Primitives;
    using Newtonsoft.Json;
    
    const double revenuePerkW = 0.12;
    const double technicianCost = 250;
    const double turbineCost = 100;
    
    public static async Task<IActionResult> Run(HttpRequest req, ILogger log)
    {
        // Get query strings if they exist
        int tempVal;
        int? hours = Int32.TryParse(req.Query["hours"], out tempVal) ? tempVal : (int?)null;
        int? capacity = Int32.TryParse(req.Query["capacity"], out tempVal) ? tempVal : (int?)null;
    
        // Get request body
        string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
        dynamic data = JsonConvert.DeserializeObject(requestBody);
    
        // Use request body if a query was not sent
        capacity = capacity ?? data?.capacity;
        hours = hours ?? data?.hours;
    
        // Return bad request if capacity or hours are not passed in
        if (capacity == null || hours == null){
            return new BadRequestObjectResult("Please pass capacity and hours on the query string or in the request body");
        }
        // Formulas to calculate revenue and cost
        double? revenueOpportunity = capacity * revenuePerkW * 24;  
        double? costToFix = (hours * technicianCost) +  turbineCost;
        string repairTurbine;
    
        if (revenueOpportunity > costToFix){
            repairTurbine = "Yes";
        }
        else {
            repairTurbine = "No";
        };
    
        return (ActionResult)new OkObjectResult(new{
            message = repairTurbine,
            revenueOpportunity = "$"+ revenueOpportunity,
            costToFix = "$"+ costToFix
        });
    }
    ```

    Ce code de fonction retourne un message `Yes` ou `No` pour indiquer si une réparation d’urgence est rentable. Elle retourne également l’opportunité de revenu que représente la turbine et le coût de la réparation.

1. Pour tester la fonction, sélectionnez **Test**, puis l’onglet **Entrée**, saisissez l’entrée suivante pour le **Corps**, puis sélectionnez **Exécuter** :

    ```json
    {
    "hours": "6",
    "capacity": "2500"
    }
    ```

    :::image type="content" source="media/functions-openapi-definition/test-function.png" alt-text="Créer une fonction HTTP pour OpenAPI&quot;:::

1. Sélectionnez **Code + test**, puis sélectionnez **run.csx** dans la liste déroulante. Remplacez le contenu du fichier de script C# run.csx par le code suivant, puis choisissez **Enregistrer** :

    ```csharp
    #r &quot;Newtonsoft.Json":::

    La sortie suivante est retournée dans l’onglet **Sortie** :

    ```json
    {"message":"Yes","revenueOpportunity":"$7200","costToFix":"$1600"}
    ```

Vous disposez maintenant d’une fonction qui détermine la rentabilité des réparations d’urgence. Ensuite, vous générez une définition OpenAPI pour l’application de fonction.

## <a name="generate-the-openapi-definition"></a>Générer la définition OpenAPI

Pour générer la définition OpenAPI :

1. Sélectionnez l’application de fonction, choisissez **Gestion des API** dans le menu de gauche, puis sélectionnez **Créer nouveau** sous **Gestion des API**.

    :::image type="content" source="media/functions-openapi-definition/select-all-settings-openapi.png" alt-text="Créer une fonction HTTP pour OpenAPI&quot;:::

1. Sélectionnez **Code + test**, puis sélectionnez **run.csx** dans la liste déroulante. Remplacez le contenu du fichier de script C# run.csx par le code suivant, puis choisissez **Enregistrer** :

    ```csharp
    #r &quot;Newtonsoft.Json":::


1. Utilisez les paramètres de Gestion des API de la manière spécifiée dans le tableau suivant :

    | Paramètre      | Valeur suggérée  | Description                                        |
    | ------------ |  ------- | -------------------------------------------------- |
    | **Nom** | Nom globalement unique | Un nom est généré sur la base du nom de votre application de fonction. |
    | **Abonnement** | Votre abonnement | Abonnement sous lequel cette nouvelle ressource est créée. |  
    | **[Groupe de ressources](../azure-resource-manager/management/overview.md)** |  myResourceGroup | Même ressource que votre application de fonction, qui doit être définie pour vous. |
    | **Lieu** | USA Ouest | Choisissez l’emplacement USA Ouest. |
    | **Nom de l’organisation** | Contoso | Nom de l’organisation utilisé dans le portail des développeurs et pour les notifications par e-mail. |
    | **E-mail de l’administrateur** | votre e-mail | E-mail ayant reçu les notifications système de la Gestion des API. |
    | **Niveau tarifaire** | Consommation | Le niveau Consommation n’est pas disponible dans toutes les régions. Pour des informations tarifaires complètes, voir la [page de tarification de la Gestion des API](https://azure.microsoft.com/pricing/details/api-management/). |

    ![Créer un service de Gestion des API](media/functions-openapi-definition/new-apim-service-openapi.png)

1. Choisissez **Créer** pour créer l’instance de Gestion des API, ce qui peut prendre plusieurs minutes.

1. Une fois qu’Azure a créé l’instance, l’option **Activer Application Insights** est disponible sur la page. Sélectionnez-la pour envoyer les journaux au même emplacement que l’application de fonction, puis sélectionnez **API de liaison**.

1. La fenêtre **Importer Azure Functions** s’ouvre avec la fonction **TurbineRepair** en surbrillance. Cliquez sur **Sélectionner** pour continuer.

    ![Importer Azure Functions dans Gestion des API](media/functions-openapi-definition/import-function-openapi.png)

1. Dans la page **Créer à partir de Function App**, acceptez les valeurs par défaut, puis sélectionnez **Créer**.

    :::image type="content" source="media/functions-openapi-definition/create-function-openapi.png" alt-text="Créer une fonction HTTP pour OpenAPI&quot;:::

1. Sélectionnez **Code + test**, puis sélectionnez **run.csx** dans la liste déroulante. Remplacez le contenu du fichier de script C# run.csx par le code suivant, puis choisissez **Enregistrer** :

    ```csharp
    #r &quot;Newtonsoft.Json":::

    Azure crée l’API pour la fonction.

## <a name="test-the-api"></a>Tester l’API

Avant d’utiliser la définition OpenAPI, vous devez vérifier que l’API fonctionne.

1. Sur la page de votre application de fonction, sélectionnez **Gestion des API**, sélectionnez l’onglet **Test**, puis **POST TurbineRepair**. 

1. Entrez le code suivant dans le **Corps de la demande** :

    ```json
    {
    "hours": "6",
    "capacity": "2500"
    }
    ```

1. Sélectionnez **Envoyer**, puis affichez la **réponse HTTP**.

    :::image type="content" source="media/functions-openapi-definition/test-function-api-openapi.png" alt-text="Créer une fonction HTTP pour OpenAPI&quot;:::

1. Sélectionnez **Code + test**, puis sélectionnez **run.csx** dans la liste déroulante. Remplacez le contenu du fichier de script C# run.csx par le code suivant, puis choisissez **Enregistrer** :

    ```csharp
    #r &quot;Newtonsoft.Json":::

## <a name="download-the-openapi-definition"></a>Télécharger la définition OpenAPI

Si votre API fonctionne comme prévu, vous pouvez télécharger la définition OpenAPI.

1. Sélectionnez **Télécharger la définition OpenAPI** en haut de la page.
   
   ![Télécharger la définition OpenAPI](media/functions-openapi-definition/download-definition.png)

2. Enregistrez le fichier JSON téléchargé, puis ouvrez-le. Vérifiez la définition.

[!INCLUDE [clean-up-section-portal](../../includes/clean-up-section-portal.md)]

## <a name="next-steps"></a>Étapes suivantes

Vous avez utilisé l’intégration Gestion des API pour générer une définition OpenAPI de vos fonctions. Vous pouvez maintenant modifier la définition à l’aide de la Gestion des API dans le portail. Vous pouvez aussi [en savoir plus sur la Gestion des API](../api-management/api-management-key-concepts.md).

> [!div class="nextstepaction"]
> [Modifier la définition OpenAPI dans la Gestion des API](../api-management/edit-api.md)
