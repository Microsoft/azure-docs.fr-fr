---
title: 'ML Studio (classique) : réentraîner un service web classique – Azure'
description: Apprenez à réentraîner un modèle et à mettre à jour un service web classique pour utiliser le modèle réentraîné dans Azure Machine Learning Studio (classique).
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: how-to
author: peterclu
ms.author: peterlu
ms.custom: seodec18, previous-ms.author=yahajiza, previous-author=YasinMSFT, devx-track-csharp
ms.date: 02/14/2019
ms.openlocfilehash: 158541d34568b7ea02ea82dbfe90f5801824716f
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93325793"
---
# <a name="retrain-and-deploy-a-classic-studio-classic-web-service"></a>Réentraîner et déployer un service web Studio (classique) classique

**S’APPLIQUE À :**  ![Coche verte.](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (classique)   ![X indiquant non. ](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio)


Réentraîner des modèles de Machine Learning est une façon de s’assurer qu’ils restent précis et qu’ils reposent sur les données les plus pertinentes. Cet article vous montre comment réentraîner un service web Studio (classique) classique. Pour obtenir un guide sur la façon de réentraîner un nouveau service web Studio (classique), [consultez cet article pratique.](retrain-machine-learning-model.md)

## <a name="prerequisites"></a>Prérequis

Cet article suppose que vous avez déjà une expérience de réentraînement et une expérience prédictive. Ces étapes sont expliquées dans [Réentraîner et déployer un modèle Machine Learning.](./retrain-machine-learning-model.md) Toutefois, au lieu de déployer votre modèle Machine Learning comme un nouveau service web, vous allez déployer votre expérience prédictive comme un service web classique.
     
## <a name="add-a-new-endpoint"></a>Ajouter un point de terminaison

Le service web prédictif que vous avez déployé contient un point de terminaison de notation par défaut qui est synchronisé avec le modèle formé pour les expériences de formation et de notation d’origine. Pour mettre à jour votre service web avec un nouveau modèle formé, vous devez créer un point de terminaison de notation.

Pour ajouter un point de terminaison à un service web, deux options s’offrent à vous :

* Par programmation
* Utilisation du portail de services web Azure

> [!NOTE]
> Veillez à ajouter le point de terminaison au service web prédictif et non au service web d’apprentissage. Si vous avez correctement déployé à la fois un service web prédictif et un service web d’apprentissage, vous devez voir deux services web distincts répertoriés. Le service web prédictif doit se terminer par « [exp. prédictive] ».
>

### <a name="programmatically-add-an-endpoint"></a>Ajouter un point de terminaison par programmation

Vous pouvez ajouter des points de terminaison de notation à l’aide de l’exemple de code fourni dans ce [dépôt GitHub](https://github.com/hning86/azuremlps#add-amlwebserviceendpoint).

### <a name="use-the-azure-web-services-portal-to-add-an-endpoint"></a>Utiliser le portail de services web Azure pour ajouter un point de terminaison

1. Dans la colonne de navigation de gauche de Machine Learning Studio (classique), cliquez sur Services web.
1. En bas du tableau de bord de services web, cliquez sur **Gérer les points de terminaison (préversion)** .
1. Cliquez sur **Add**.
1. Tapez un nom et une description pour le point de terminaison. Sélectionnez le niveau de journalisation et activez les exemples de données si nécessaire. Pour plus d’informations sur la journalisation, consultez [Activation de la journalisation pour les services web de Machine Learning](web-services-logging.md).

## <a name="update-the-added-endpoints-trained-model"></a>Mettre à jour le modèle entraîné du point de terminaison ajouté

### <a name="retrieve-patch-url"></a>Récupérer l’URL PATCH

Suivez ces étapes pour obtenir l’URL PATCH correcte à l’aide du portail web :

1. Connectez-vous au portail des [services web Azure Machine Learning](https://services.azureml.net/).
1. Cliquez sur **Services web** ou **Services web classiques** en haut de la page.
1. Cliquez sur le service web d’évaluation que vous utilisez (si vous n’avez pas modifié le nom par défaut du service web, il doit se terminer par « [Scoring Exp.] »).
1. Cliquez sur **+NOUVEAU**.
1. Après l’ajout du point de terminaison, cliquez sur le nom du point de terminaison.
1. Sous **l’URL d’application de correctifs** , cliquez sur **Aide de l’API** pour ouvrir la page d’aide d’application de correctifs.

> [!NOTE]
> Si vous avez ajouté le point de terminaison au service web de formation plutôt qu'au service web prédictif, lorsque vous cliquerez sur le lien **Mettre à jour la ressource** , vous recevrez un message d'erreur : signalant que cette fonctionnalité n'est pas prise en charge ou disponible dans ce contexte. Ce service web n’a aucune ressource actualisable. Veuillez nous excuser pour ce désagrément. Nous travaillons actuellement à l’amélioration de ce flux de travail.
>

La page d’aide d’application de correctifs contient l’URL d’application de correctifs que vous devez utiliser et fournit un exemple de code que vous pouvez utiliser pour l’appeler.

![URL d’application de correctifs.](./media/retrain-classic/ml-help-page-patch-url.png)

### <a name="update-the-endpoint"></a>Mettre à jour le point de terminaison

Vous pouvez maintenant utiliser le modèle entraîné pour mettre à jour le point de terminaison de notation que vous avez créé précédemment.

L’exemple de code suivant montre comment utiliser les éléments *BaseLocation* , *RelativeLocation* , *SasBlobToken* et l’URL d’application de correctifs pour mettre à jour le point de terminaison.

```csharp
private async Task OverwriteModel()
{
    var resourceLocations = new
    {
        Resources = new[]
        {
            new
            {
                Name = "Census Model [trained model]",
                Location = new AzureBlobDataReference()
                {
                    BaseLocation = "https://esintussouthsus.blob.core.windows.net/",
                    RelativeLocation = "your endpoint relative location", //from the output, for example: "experimentoutput/8946abfd-79d6-4438-89a9-3e5d109183/8946abfd-79d6-4438-89a9-3e5d109183.ilearner"
                    SasBlobToken = "your endpoint SAS blob token" //from the output, for example: "?sv=2013-08-15&sr=c&sig=37lTTfngRwxCcf94%3D&st=2015-01-30T22%3A53%3A06Z&se=2015-01-31T22%3A58%3A06Z&sp=rl"
                }
            }
        }
    };

    using (var client = new HttpClient())
    {
        client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", apiKey);

        using (var request = new HttpRequestMessage(new HttpMethod("PATCH"), endpointUrl))
        {
            request.Content = new StringContent(JsonConvert.SerializeObject(resourceLocations), System.Text.Encoding.UTF8, "application/json");
            HttpResponseMessage response = await client.SendAsync(request);

            if (!response.IsSuccessStatusCode)
            {
                await WriteFailedResponse(response);
            }

            // Do what you want with a successful response here.
        }
    }
}
```

L’ *apiKey* et l’ *endpointUrl* pour l’appel sont figurent sur le tableau de bord du point de terminaison.

La valeur du paramètre *Name* dans *Ressources* doit correspondre au nom de ressource du modèle formé enregistré dans l’expérience prédictive. Pour obtenir le nom de la ressource :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Dans le menu de gauche, cliquez sur **Machine Learning**.
1. Sous Nom, cliquez sur votre espace de travail, puis sur **Services web**.
1. Sous Nom, cliquez sur **Modèle de recensement [exp. prédictive]**.
1. Cliquez sur le nouveau point de terminaison que vous avez ajouté.
1. Dans le tableau de bord du point de terminaison, cliquez sur **Mettre à jour les ressources**.
1. Dans la page Documentation de l’API Mettre à jour la ressource du service web, vous trouverez le **Nom de la ressource** sous **Ressources actualisables**.

Si votre jeton SAS expire avant la fin de la mise à jour du point de terminaison, vous devez effectuer une opération GET avec l’ID du travail pour obtenir un nouveau jeton.

Lorsque le code a été exécuté avec succès, le nouveau point de terminaison doit commencer à utiliser le modèle de nouveau entraîné après environ 30 secondes.

## <a name="next-steps"></a>Étapes suivantes

Pour savoir comment gérer les services web ou effectuer le suivi de plusieurs exécutions d’expériences, voir les articles suivants :

* [Explorer le portail Services web](manage-new-webservice.md)
* [Gérer des itérations d'expériences](manage-experiment-iterations.md)