---
title: Vérifier l’instance de conteneur Extraction de phrases clés
titleSuffix: Azure Cognitive Services
description: Découvrez comment vérifier l’instance de conteneur Extraction de phrases clés.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: include
ms.date: 04/01/2020
ms.author: aahi
ms.openlocfilehash: 5dca4828a5c1127133461ddf9fc06099fc176b68
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96009915"
---
### <a name="verify-the-key-phrase-extraction-container-instance"></a>Vérifier l’instance de conteneur Extraction de phrases clés

1. Sélectionnez l’onglet **Vue d’ensemble** et copiez l’adresse IP.
1. Ouvrez un nouvel onglet de navigateur et entrez l’adresse IP. Par exemple, entrez `http://<IP-address>:5000 (http://55.55.55.55:5000`. La page d’accueil du conteneur s’affiche, vous informant que le conteneur est en cours d’exécution.

    ![Affichez la page d’accueil du conteneur pour vérifier qu’elle est en cours d’exécution](../media/how-tos/container-instance/swagger-docs-on-container.png)

1. Veuillez sélectionner le lien **Description de l’API de service** pour accéder à la page Swagger des conteneurs.

1. Choisissez une des API **POST** et sélectionnez **Essayer**. Les paramètres sont affichés, y compris cet exemple d’entrée :

    ```json
    {
      "documents": [
        {
          "id": "1",
          "text": "Hello world"
        },
        {
          "id": "2",
          "text": "Bonjour tout le monde"
        },
        {
          "id": "3",
          "text": "La carretera estaba atascada. Había mucho tráfico el día de ayer."
        },
        {
          "id": "4",
          "text": ":) :( :D"
        }
      ]
    }
    ```

1. Remplacez l’entrée par le contenu JSON suivant :

    ```json
    {
      "documents": [
        {
          "language": "en",
          "id": "7",
          "text": "I was fortunate to attend the KubeCon Conference in Barcelona, it is one of the best conferences I have ever attended. Great people, great sessions and I thoroughly enjoyed it!"
        }
      ]
    }
    ```

1. Définissez **showStats** sur `true`.

1. Sélectionnez **Exécuter** pour déterminer le sentiment du texte.

    Le modèle empaqueté dans le conteneur génère un score compris entre 0 et 1, où 0 est négatif et 1 est positif.

    La réponse JSON retournée inclut le sentiment pour l’entrée de texte mise à jour :

    ```json
    {
      "documents": [
        {
          "id": "7",
          "keyPhrases": [
            "Great people",
            "great sessions",
            "KubeCon Conference",
            "Barcelona",
            "best conferences"
          ],
          "statistics": {
            "charactersCount": 176,
            "transactionsCount": 1
          }
        }
      ],
      "errors": [],
      "statistics": {
        "documentsCount": 1,
        "validDocumentsCount": 1,
        "erroneousDocumentsCount": 0,
        "transactionsCount": 1
      }
    }
    ```

Nous pouvons maintenant corréler le document `id` des données JSON de la charge utile de réponse avec le document `id` de la charge utile de la requête d’origine. Le document résultant possède un tableau `keyPhrases` qui contient la liste des phrases clés extraites du document d’entrée correspondant. De plus, il existe différentes statistiques, telles que `characterCount` et `transactionCount`, pour chaque document résultant.