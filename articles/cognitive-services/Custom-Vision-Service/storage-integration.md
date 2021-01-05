---
title: Intégrer le stockage Azure pour les notifications et la sauvegarde de modèle
titleSuffix: Azure Cognitive Services
description: Découvrez comment intégrer le stockage Azure pour recevoir des notifications push lors de l’apprentissage ou de l’exportation de modèles Custom Vision. Vous pouvez également enregistrer une sauvegarde des modèles exportés.
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: custom-vision
ms.topic: how-to
ms.date: 09/11/2020
ms.author: pafarley
ms.openlocfilehash: f4d9cc4c02ab062c73e9dbd977d9ea9e6ccdb60d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90532949"
---
# <a name="integrate-azure-storage-for-notifications-and-backup"></a>Intégrer le stockage Azure pour les notifications et la sauvegarde

Vous pouvez intégrer votre projet Custom Vision à une file d’attente de stockage d’objets blob Azure pour recevoir des notifications push de l’activité d’apprentissage/d’exportation de projet et des copies de sauvegarde des modèles publiés. Cette fonctionnalité évite de devoir interroger continuellement les résultats du service lors de l’exécution d’opérations de longue durée. En effet, vous pouvez intégrer les notifications de la file d’attente de stockage à votre workflow.

Ce guide vous montre comment utiliser ces API REST avec cURL. Vous pouvez également utiliser un service de requête HTTP tel que Postman pour émettre les demandes.

> [!NOTE]
> Les notifications push dépendent du paramètre facultatif _notificationQueueUri_ de l’API **CreateProject**, et les sauvegardes de modèle impliquent que vous utilisiez également le paramètre facultatif _exportModelContainerUri_. Ce guide utilise ces deux paramètres pour l’ensemble complet des fonctionnalités.

## <a name="prerequisites"></a>Prérequis

- Ressource Custom Vision dans Azure. À défaut, accédez au portail Azure et [créez une ressource Custom Vision](https://portal.azure.com/?microsoft_azure_marketplace_ItemHideKey=microsoft_azure_cognitiveservices_customvision#create/Microsoft.CognitiveServicesCustomVision?azure-portal=true). Actuellement, cette fonctionnalité ne prend pas en charge la ressource Cognitive Service (dans une même clé).
- Compte Stockage Azure avec conteneur d’objets blob. Suivez les [exercices 1 du lab Azure Storage](https://github.com/Microsoft/computerscience/blob/master/Labs/Azure%20Services/Azure%20Storage/Azure%20Storage%20and%20Cognitive%20Services%20(MVC).md#Exercise1) si vous avez besoin d’aide pour cette étape.

## <a name="set-up-azure-storage-integration"></a>Configurer l’intégration du stockage Azure

Accédez à votre ressource de formation Custom Vision sur le portail Azure, sélectionnez la page **Identité** , puis activez l’identité managée affectée par le système.

Ensuite, accédez à votre ressource de stockage dans le portail Azue. Accédez à la page **Contrôle d’accès (IAM)** et ajoutez une attribution de rôle pour chaque fonctionnalité d’intégration :
* Sélectionnez votre ressource de formation Custom Vision et attribuez le rôle **Contributeur aux données Blob du stockage** si vous envisagez d’utiliser la fonctionnalité de sauvegarde de modèle. 
* Sélectionnez ensuite votre ressource de formation Custom Vision et attribuez le **Contributeur aux données en file d’attente du stockage** si vous envisagez d’utiliser la fonctionnalité de file d’attente de notification.

> [!div class="mx-imgBorder"]
> ![Page d’attribution de rôle au compte de stockage](./media/storage-integration/storage-access.png)

### <a name="get-integration-urls"></a>Obtenir les URL d'intégration

Vous obtenez ensuite les URL qui permettent à votre ressource Custom Vision d’accéder à ces points de terminaison.

Pour l’URL d’intégration de la file d’attente de notification, accédez à la page **Files d’attente** de votre compte de stockage, ajoutez une nouvelle file d’attente, puis enregistrez son URL dans un emplacement temporaire.

> [!div class="mx-imgBorder"]
> ![Page de file d’attente de stockage Azure](./media/storage-integration/queue-url.png) 

Pour l’URL d’intégration de la sauvegarde de modèle, accédez à la page **Conteneurs** de votre compte de stockage et créez un conteneur. Sélectionnez-le, puis accédez à la page **Propriétés**. Copiez l’URL dans un emplacement temporaire.
 
> [!div class="mx-imgBorder"]
> ![Page des propriétés du conteneur Stockage Azure](./media/storage-integration/container-url.png) 


## <a name="integrate-custom-vision-project"></a>Intégrer le projet Custom Vision

Maintenant que vous disposez des URL d’intégration, vous pouvez créer un projet Custom Vision intégrant les fonctionnalités Stockage Azure. Vous pouvez également mettre à jour un projet existant pour ajouter les fonctionnalités.

### <a name="create-new-project"></a>Création d’un projet

Lorsque vous appelez l’API [CreateProject](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.3/operations/5eb0bcc6548b571998fddeae), ajoutez les paramètres facultatifs _exportModelContainerUri_ et _notificationQueueUri_. Attribuez les valeurs d’URL obtenues dans la section précédente. 

```curl
curl -v -X POST "{endpoint}/customvision/v3.3/Training/projects?exportModelContainerUri={inputUri}&notificationQueueUri={inputUri}&name={inputName}"
-H "Training-key: {subscription key}"
```

Si vous recevez une réponse `200/OK`, cela signifie que les URL ont été configurées avec succès. Les valeurs d’URL doivent également s’afficher dans la réponse JSON :

```json
{
  "id": "00000000-0000-0000-0000-000000000000",
  "name": "string",
  "description": "string",
  "settings": {
    "domainId": "00000000-0000-0000-0000-000000000000",
    "classificationType": "Multiclass",
    "targetExportPlatforms": [
      "CoreML"
    ],
    "useNegativeSet": true,
    "detectionParameters": "string",
    "imageProcessingSettings": {
      "augmentationMethods": {}
},
"exportModelContainerUri": {url}
"notificationQueueUri": {url}
  },
  "created": "string",
  "lastModified": "string",
  "thumbnailUri": "string",
  "drModeEnabled": true,
  "status": "Succeeded"
}
```

### <a name="update-existing-project"></a>Mettre à jour un projet existant

Pour mettre à jour un projet existant avec l’intégration des fonctionnalités Stockage Azure, appelez l’API [UpdateProject](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.3/operations/5eb0bcc6548b571998fddeb1) à l’aide de l’ID du projet que vous souhaitez mettre à jour. 

```curl
curl -v -X PATCH "{endpoint}/customvision/v3.3/Training/projects/{projectId}"
-H "Content-Type: application/json"
-H "Training-key: {subscription key}"

--data-ascii "{body}" 
```

Définissez le corps de la demande (`body`) au format JSON suivant, en remplissant les valeurs appropriées pour _exportModelContainerUri_ et _notificationQueueUri_ :

```json
{
  "name": "string",
  "description": "string",
  "settings": {
    "domainId": "00000000-0000-0000-0000-000000000000",
    "classificationType": "Multiclass",
    "targetExportPlatforms": [
      "CoreML"
    ],
    "imageProcessingSettings": {
      "augmentationMethods": {}
},
"exportModelContainerUri": {url}
"notificationQueueUri": {url}

  },
  "status": "Succeeded"
}
```

Si vous recevez une réponse `200/OK`, cela signifie que les URL ont été configurées avec succès.

## <a name="verify-the-connection"></a>Vérifier la connexion 

Votre appel d’API de la section précédente doit avoir déjà déclenché de nouvelles informations dans votre compte Stockage Azure. 

Votre conteneur désigné doit contenir un objet blob de test dans un dossier **CustomVision-TestPermission**. Cet objet blob est temporaire.

Dans votre file d’attente de notification, une notification de test au format suivant doit s’afficher :
 
```json
{
"version": "1.0" ,
"type": "ConnectionTest",
"Content":
    {
    "projectId": "00000000-0000-0000-0000-000000000000"
    }
}
```

## <a name="get-event-notifications"></a>Recevoir des notifications d'événements

Lorsque vous êtes prêt, appelez l’API [TrainProject](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.3/operations/5eb0bcc7548b571998fddee1) sur votre projet pour effectuer une opération d’apprentissage classique.

Dans votre file d’attente de notification de stockage, vous recevrez une notification une fois l’opération terminée :

```json
{
"version": "1.0" ,
"type": "Training",
"Content":
    {
    "projectId": "00000000-0000-0000-0000-000000000000",
    "iterationId": "00000000-0000-0000-0000-000000000000",
    "trainingStatus": "TrainingCompleted"
    }
}
```

Le champ `"trainingStatus"` peut être `"TrainingCompleted"` ou `"TrainingFailed"`. Le champ `"iterationId"` correspond à l’ID du modèle formé.

## <a name="get-model-export-backups"></a>Obtenir des sauvegardes d’exportation de modèle

Lorsque vous êtes prêt, appelez l’API [ExportIteration](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.3/operations/5eb0bcc6548b571998fddece) pour exporter un modèle formé dans une plateforme spécifiée.

Dans votre conteneur de stockage désigné, une copie de sauvegarde du modèle exporté s’affiche. Le nom de l’objet blob se présentera comme suit :

```
{projectId} - {iterationId}.{platformType}
```

En outre, vous recevrez une notification dans votre file d’attente une fois l’exportation terminée. 

```json
{
"version": "1.0" ,
"type": "Export",
"Content":
    {
    "projectId": "00000000-0000-0000-0000-000000000000",
    "iterationId": "00000000-0000-0000-0000-000000000000",
    "exportStatus": "ExportCompleted",
    "modelUri": {url}
    }
}
```

Le champ `"exportStatus"` peut être `"ExportCompleted"` ou `"ExportFailed"`. Le champ `"modelUri"` contient l’URL du modèle de sauvegarde stocké dans votre conteneur, sous réserve que vous ayez intégré les notifications de la file d’attente au début. Dans le cas contraire, le champ `"modelUri"` affichera l’URL SAS de votre objet blob de modèle Custom Vision.

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide, vous avez appris à copier et déplacer un projet entre des ressources Custom Vision. Explorez à présent la documentation de référence sur les API pour voir ce que vous pouvez faire d’autre avec Custom Vision.
* [Documentation de référence sur l’API REST](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.3/operations/5eb0bcc6548b571998fddeb3)
