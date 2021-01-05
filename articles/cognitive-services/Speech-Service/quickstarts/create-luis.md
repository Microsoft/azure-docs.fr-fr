---
title: 'Démarrage rapide : Créer une clé LUIS'
titleSuffix: Azure Cognitive Services
description: Dans ce guide de démarrage rapide, vous découvrez comment créer une application LUIS et obtenir une clé.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: tutorial
ms.date: 06/25/2020
ms.author: trbye
ms.openlocfilehash: f547b353091cc81816f0974fced8d4dc836d22b2
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95023818"
---
# <a name="quickstart-getting-a-luis-endpoint-key"></a>Démarrage rapide : Obtention d’une clé de point de terminaison LUIS

## <a name="prerequisites"></a>Prérequis

Veillez à disposer des éléments suivants avant de commencer ce tutoriel :

* Un compte LUIS. Vous pouvez en obtenir un gratuitement via le [portail LUIS](https://www.luis.ai/home).

## <a name="luis-and-speech"></a>LUIS et Speech

LUIS s’intègre au service Speech pour reconnaître les intentions à partir de la reconnaissance vocale. Vous n’avez besoin que de LUIS, aucun abonnement au service Speech n’est nécessaire.

LUIS utilise trois types de clés :

|Type de clé|Objectif|
|--------|-------|
|Création|Permet de créer et de modifier des applications LUIS programmatiquement|
|Starter|Permet de tester l’application LUIS en utilisant uniquement du texte|
|Point de terminaison |Autorise l’accès à une application LUIS particulière|

Pour ce tutoriel, vous avez besoin du type de clé du point de terminaison. Ce tutoriel utilise l’exemple d’application domotique LUIS Home Automation, que vous pouvez créer en suivant le guide de démarrage rapide [Utiliser une application domotique prédéfinie](../../luis/luis-get-started-create-app.md). Si vous avez créé votre propre application LUIS, vous pouvez aussi l’utiliser.

Lors de la création d’une application LUIS, une clé de démarrage est générée automatiquement par LUIS pour que vous puissiez tester l’application à l’aide de requêtes de texte. Cette clé n’active pas l’intégration du service Speech et ne fonctionne pas avec ce tutoriel. Créez une ressource LUIS dans le tableau de bord Azure et affectez-la à l’application LUIS. Vous pouvez utiliser le niveau d’abonnement gratuit pour ce tutoriel.

Après avoir créé la ressource LUIS dans le tableau de bord Azure, connectez-vous au [portail LUIS](https://www.luis.ai/home), choisissez votre application dans la page **My apps** (Mes applications) et basculez vers la page **Manage** (Gérer) de l’application. Enfin, sélectionnez **Keys and Endpoints** (Clés et points de terminaison) dans la barre latérale.

![Paramètres des clés et point de terminaison du portail LUIS](~/articles/cognitive-services/Speech-Service/media/sdk/luis-keys-endpoints-page.png)

Dans la page **Keys and Endpoint settings** (Paramètres des clés et du point de terminaison) :

1. Faites défiler vers le bas jusqu’à la section **Resources and Keys** (Ressources et clés) et des clés, puis sélectionnez **Assign resource** (Attribuer une ressource).
1. Dans la boîte de dialogue **Assign a key to your app** (Attribuer une clé à votre application), apportez les modifications suivantes :

   * Sous **Tenant** (Locataire), choisissez **Microsoft**.
   * Sous **Subscription Name** (Nom de l’abonnement), sélectionnez l’abonnement Azure qui contient la ressource LUIS à utiliser.
   * Sous **Key** (Clé), choisissez la ressource LUIS à utiliser avec l’application.

   Le nouvel abonnement apparaît dans le tableau, en bas de la page, au bout de quelques instants.

1. Sélectionnez l’icône en regard d’une clé pour la copier dans le Presse-papiers. (Vous pouvez utiliser l’une ou l’autre de ces clés.)

![Clés d’abonnement de l’appli LUIS](~/articles/cognitive-services/Speech-Service/media/sdk/luis-keys-assigned.png)


## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Reconnaître les intentions](~/articles/cognitive-services/Speech-Service/quickstarts/intent-recognition.md)