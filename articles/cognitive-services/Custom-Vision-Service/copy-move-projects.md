---
title: Copier et déplacer des projets Custom Vision
titleSuffix: Azure Cognitive Services
description: Découvrez comment utiliser les API ExportProject et ImportProject pour copier et déplacer vos projets Custom Vision.
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: custom-vision
ms.topic: how-to
ms.date: 09/08/2020
ms.author: pafarley
ms.openlocfilehash: 548d936538a909da10796d0377f119826582c420
ms.sourcegitcommit: 9706bee6962f673f14c2dc9366fde59012549649
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2020
ms.locfileid: "94616057"
---
# <a name="copy-and-move-your-custom-vision-projects"></a>Copier et déplacer vos projets Custom Vision

Une fois que vous avez créé et formé un projet Custom Vision, vous souhaiterez peut-être copier votre projet vers une autre ressource. Par exemple, vous souhaiterez peut-être déplacer un projet d’un environnement de développement vers un environnement de production ou sauvegarder un projet vers un compte dans une autre région Azure pour une sécurité accrue des données.

Les API **[ExportProject](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.3/operations/5eb0bcc6548b571998fddeb3)** et **[ImportProject](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.3/operations/5eb0bcc7548b571998fddee3)** rendent ce scénario possible en vous permettant de copier des projets d’un compte Custom Vision vers d’autres. Ce guide vous montre comment utiliser ces API REST avec cURL. Vous pouvez également utiliser un service de requête HTTP tel que Postman pour émettre les demandes.

## <a name="business-scenarios"></a>Scénarios d’entreprise

Si votre application ou votre entreprise dépend de l’utilisation d’un projet Custom Vision, nous vous recommandons de copier votre modèle vers un autre compte Custom Vision dans une autre région. Puis, en cas de panne régionale, vous pouvez accéder à votre projet dans la région où il a été copié.

##  <a name="prerequisites"></a>Prérequis

- Deux ressources Azure Custom Vision. Si vous n’en avez pas, accédez au Portail Azure et [créez une nouvelle ressource Custom Vision](https://portal.azure.com/?microsoft_azure_marketplace_ItemHideKey=microsoft_azure_cognitiveservices_customvision#create/Microsoft.CognitiveServicesCustomVision?azure-portal=true).
- Les clés de formation et les URL de point de terminaison de vos ressources Custom Vision. Ces valeurs se trouvent sous l’onglet **Vue d’ensemble** de la ressource sur le portail Azure.
- Un projet Custom Vision créé. Pour obtenir des instructions sur la façon de procéder, consultez [Créer un classifieur](./getting-started-build-a-classifier.md).

## <a name="process-overview"></a>Vue d’ensemble du processus

Le processus de copie d’un projet se compose des étapes suivantes :

1. Tout d’abord, vous récupérez l’ID du projet que vous souhaitez copier dans votre compte source.
1. Ensuite, vous appelez l’API **ExportProject** à l’aide de l’ID de projet et de la clé de formation de votre compte source. Vous obtiendrez une chaîne de jeton temporaire.
1. Ensuite, vous appelez l’API **ImportProject** à l’aide de la chaîne de jeton et de la clé de formation de votre compte cible. Le projet est alors répertorié sous votre compte cible.

## <a name="get-the-project-id"></a>Obtenir l'ID de projet

Tout d’abord, appelez **[GetProjects](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.3/operations/5eb0bcc6548b571998fddead)** pour afficher la liste de vos projets Custom Vision existants et leurs ID. Utilisez la clé de formation et le point de terminaison de votre compte source.

```curl
curl -v -X GET "{endpoint}/customvision/v3.3/Training/projects"
-H "Training-key: {training key}"
```

Vous obtiendrez une réponse `200\OK` avec une liste de projets et leurs métadonnées dans le corps. La valeur `"id"` est la chaîne à copier pour les étapes suivantes.

```json
[
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
      }
    },
    "created": "string",
    "lastModified": "string",
    "thumbnailUri": "string",
    "drModeEnabled": true,
    "status": "Succeeded"
  }
]
```

## <a name="export-the-project"></a>Exporter le projet

Appelez **[ExportProject](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.3/operations/5eb0bcc6548b571998fddeb3)** à l’aide de l’ID de projet et de la clé et du point de terminaison de formation sources.

```curl
curl -v -X GET "{endpoint}/customvision/v3.3/Training/projects/{projectId}/export"
-H "Training-key: {training key}"
```

Vous obtiendrez une réponse `200/OK` avec les métadonnées sur le projet exporté et une chaîne de référence `"token"`. Copiez la valeur du jeton.

```json
{
  "iterationCount": 0,
  "imageCount": 0,
  "tagCount": 0,
  "regionCount": 0,
  "estimatedImportTimeInMS": 0,
  "token": "string"
}
```

## <a name="import-the-project"></a>Importer le projet

Appelez **[ImportProject](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.3/operations/5eb0bcc7548b571998fddee3)** à l’aide de votre clé et de votre point de terminaison de formation cibles, ainsi que le jeton de référence. Vous pouvez également attribuer un nom à votre projet dans son nouveau compte.

```curl
curl -v -G -X POST "{endpoint}/customvision/v3.3/Training/projects/import"
--data-urlencode "token={token}" --data-urlencode "name={name}"
-H "Training-key: {training key}" -H "Content-Length: 0"
```

Vous obtiendrez une réponse `200/OK` avec les métadonnées relatives à votre projet nouvellement importé.

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
    }
  },
  "created": "string",
  "lastModified": "string",
  "thumbnailUri": "string",
  "drModeEnabled": true,
  "status": "Succeeded"
}
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide, vous avez appris à copier et déplacer un projet entre des ressources Custom Vision. Explorez à présent la documentation de référence sur les API pour voir ce que vous pouvez faire d’autre avec Custom Vision.
* [Documentation de référence sur l’API REST](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.3/operations/5eb0bcc6548b571998fddeb3)