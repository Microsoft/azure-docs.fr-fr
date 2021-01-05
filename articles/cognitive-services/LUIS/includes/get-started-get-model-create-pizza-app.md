---
title: Fichier include
description: Fichier include
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.date: 06/04/2020
ms.subservice: language-understanding
ms.topic: include
ms.custom: include file
ms.author: aahi
ms.reviewer: roy-har
ms.openlocfilehash: a021b7c5b26e1bca1e3f623ad762c2f3ee8d6020
ms.sourcegitcommit: 1140ff2b0424633e6e10797f6654359947038b8d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/30/2020
ms.locfileid: "97820565"
---
Créez l’application de pizza.

1. Sélectionnez [pizza-app-for-luis-v6.json](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/luis/apps/pizza-app-for-luis-v6.json) pour afficher la page GitHub du fichier `pizza-app-for-luis.json`.
1. Cliquez avec le bouton droit ou appuyez longuement sur le bouton **Raw** et sélectionnez **Enregistrer la cible sous** pour enregistrer le fichier `pizza-app-for-luis.json` sur votre ordinateur.
1. Connectez-vous au [portail LUIS](https://www.luis.ai).
1. Sélectionnez [My Apps](https://www.luis.ai/applications) (Mes applications).
1. Dans la page **My Apps**, sélectionnez **+ New app for conversation** (Nouvelle application pour la conversation).
1. Sélectionnez **Import as JSON** (Importer en tant que JSON).
1. Dans la boîte de dialogue **Import new app** (Importer une nouvelle application), sélectionnez le bouton **Choose File** (Choisir un fichier).
1. Sélectionnez le fichier `pizza-app-for-luis.json` que vous avez téléchargé, puis sélectionnez **Open** (Ouvrir).
1. Dans le champ **Name** (Nom) de la boîte de dialogue **Import new app**, entrez un nom pour votre application de pizza, puis sélectionnez le bouton **Done** (terminé).

L’application est importée.

Si apparaît une boîte de dialogue **How to create an effective LUIS app** (Comment créer une application LUIS efficace), fermez-la.

## <a name="train-and-publish-the-pizza-app"></a>Entraîner et publier l’application de pizza

Vous devez voir la page **Intents** (Intentions) avec une liste des intentions de l’application de pizza.

[!INCLUDE [How to train](howto-train.md)]

[!INCLUDE [How to publish](howto-publish.md)]

## <a name="add-an-authoring-resource-to-the-pizza-app"></a>Ajouter une ressource de création à l’application Pizza

1. Sélectionnez **MANAGE** (Gérer).
1. Sélectionnez **Azure Resources** (Ressources Azure).
1. Sélectionnez **Authoring Resource** (Ressource de création).
1. Sélectionnez **Change authoring resource** (Changer la ressource de création).

Si vous disposez d’une ressource de création, entrez le **nom du locataire**, le **nom de l’abonnement**, puis le **nom de la ressource LUIS** de votre ressource de création.

Si vous n’avez pas de ressource de création :

1. Sélectionnez **Create new resource** (Créer une ressource).
1. Entrez un **nom de locataire**, un **nom de ressource**, un **nom d’abonnement** et un **nom de groupe de ressources Azure**.

Votre application de pizza est maintenant prête à être utilisée.

## <a name="record-the-access-values-for-your-pizza-app"></a>Enregistrer les valeurs d’accès pour votre application Pizza

Pour utiliser votre nouvelle application de pizza, vous aurez besoin de son ID d’application, de sa clé de création et de son point de terminaison de création. Pour obtenir des prédictions, vous aurez besoin de votre propre point de terminaison de prédiction et de votre propre clé de prédiction.

Pour rechercher ces valeurs :

1. Dans la page **Intents** (Intentions), sélectionnez **MANAGE** (GÉRER).
1. Dans la page **Application Settings** (Paramètres de l’application), enregistrez l’**ID d’application**.
1. Sélectionnez **Azure Resources** (Ressources Azure).
1. Sélectionnez **Authoring Resource** (Ressource de création).
1. Sous les onglets **Ressource de création** et **Ressources de prédiction**, enregistrez la **Clé primaire**. Cette valeur est votre clé de création.
1. Enregistrez l’**URL du point de terminaison**. Cette valeur est votre point de terminaison de création.
