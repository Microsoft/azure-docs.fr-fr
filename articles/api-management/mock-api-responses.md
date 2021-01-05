---
title: Tutoriel - Simuler des réponses d’API dans Gestion des API - Portail Azure | Microsoft Docs
description: Dans ce tutoriel, vous utilisez Gestion des API pour définir une stratégie sur une API afin qu’elle retourne une réponse factice si le back-end n’est pas disponible pour envoyer des réponses réelles.
author: vladvino
ms.service: api-management
ms.custom: mvc
ms.topic: tutorial
ms.date: 09/30/2020
ms.author: apimpm
ms.openlocfilehash: 231ce9d946a2fb6650f25d90aaa423d1c95fb106
ms.sourcegitcommit: 50802bffd56155f3b01bfb4ed009b70045131750
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91930711"
---
# <a name="tutorial-mock-api-responses"></a>Tutoriel : Simuler des réponses de l’API

Les API back-end peuvent être importées dans une API de Gestion des API, ou bien créées et gérées manuellement. Les étapes décrites dans ce tutoriel vous montrent comment utiliser la Gestion des API pour créer une API vide et la gérer manuellement, puis définir une stratégie sur une API pour qu’elle retourne une réponse factice. Cette méthode permet aux développeurs de procéder à l’implémentation et au test de l’instance APIM même si le backend n’est pas en mesure d’envoyer des réponses réelles. 

La possibilité de simuler des réponses peut être utile dans différents scénarios :

+ Quand la façade de l’API est conçue avant que le backend ne soit implémenté. Ou bien, quand le backend est développé en parallèle.
+ Quand le backend est temporairement non opérationnel ou qu’il ne peut pas être mis à l’échelle.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer une API de test 
> * Ajouter une opération à l’API de test
> * Activer la simulation des réponses
> * Tester l’API simulée


:::image type="content" source="media/mock-api-responses/mock-api-responses01.png" alt-text="Réponse d’API simulée":::

## <a name="prerequisites"></a>Prérequis

+ Apprenez la [terminologie relative à Gestion des API Azure](api-management-terminology.md).
+ Comprendre le [concept des stratégies dans Gestion des API Azure](api-management-howto-policies.md).
+ Suivez ce guide de démarrage rapide : [Créer une instance du service Gestion des API Azure](get-started-create-service-instance.md).

## <a name="create-a-test-api"></a>Créer une API de test 

Cette section montre comment créer une API vide sans backend. 


1. Connectez-vous au portail Azure et accédez à votre instance de Gestion des API.
1. Sélectionnez **API** >  **+ Ajouter une API** > **API vide**.
1. Dans la fenêtre **Créer une d’API vierge**, sélectionnez **Complet**.
1. Entrez *Test API* (API de test) pour le **Nom d’affichage**.
1. Sélectionnez **Illimité** pour **Produits**.
1. Vérifiez que **Managé** est sélectionné dans **Passerelles**.
1. Sélectionnez **Create** (Créer).

    :::image type="content" source="media/mock-api-responses/03-mock-api-responses-01-create-test-api.png" alt-text="Réponse d’API simulée":::

## <a name="add-an-operation-to-the-test-api"></a>Ajouter une opération à l’API de test

Une API expose une ou plusieurs opérations. Dans cette section, ajoutez une opération à l’API vide que vous avez créée. Appeler l’opération une fois effectuées les étapes décrites dans cette section génère une erreur. Vous n’obtiendrez aucune erreur une fois que vous aurez effectué les étapes de la section [Activer la simulation des réponses](#enable-response-mocking).

1. Sélectionnez l’API que vous avez créée à l’étape précédente.
1. Sélectionnez **+ Ajouter une opération**.
1. Dans la fenêtre **Front-end**, entrez les valeurs suivantes.

     | Paramètre             | Valeur                             | Description                                                                                                                                                                                   |
    |---------------------|-----------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
    | **Nom complet**    | *Appel de test*                       | Le nom qui est affiché dans le [portail des développeurs](api-management-howto-developer-portal.md).                                                                                                                                       |
    | **URL** (verbe HTTP) | GET                               | Sélectionnez un des verbes HTTP prédéfinis.                                                                                                                                         |
    | **URL**             | */test*                           | Chemin d’URL de l’API.                                                                                                                                                                       |
    | **Description**     |                                   |  Description facultative de l’opération, utilisée pour fournir des informations dans le portail des développeurs aux développeurs utilisant cette API.                                                    |
    
1. Sélectionnez l’onglet **Réponses**, situé sous les champs URL, Nom d’affichage et Description. Entrez des valeurs sous cet onglet pour définir les codes d’état des réponses, les types de contenu, des exemples et les schémas.
1. Sélectionnez **+ Ajouter une réponse**, puis sélectionnez **200 OK** dans la liste.
1. Sous le titre **Représentations** à droite, sélectionnez **+ Ajouter la représentation**.
1. Entrez *application/json* dans la zone de recherche et sélectionnez le type de contenu **application/json**.
1. Dans la zone de texte **Exemple**, entrez `{ "sampleField" : "test" }`.
1. Sélectionnez **Enregistrer**.

:::image type="content" source="media/mock-api-responses/03-mock-api-responses-02-add-operation.png" alt-text="Réponse d’API simulée" border="false":::

Bien que ce ne soit pas obligatoire pour cet exemple, des paramètres supplémentaires pour une opération d’API peuvent être configurés sous d’autres onglets, notamment :


|Onglet      |Description  |
|---------|---------|
|**Requête**     |  Ajoutez des paramètres de requête. Outre un nom et une description, vous pouvez fournir des valeurs qui peuvent être affectées à un paramètre de requête. Une des valeurs peut être marquée comme valeur par défaut (facultatif).        |
|**Requête**     |  Définissez des types de contenu de demande, des exemples et des schémas.       |

## <a name="enable-response-mocking"></a>Activer la simulation des réponses

1. Sélectionnez l’API que vous avez créée dans [Créer une API de test](#create-a-test-api).
1. Sélectionnez l’opération de test que vous avez ajoutée.
1. Dans la fenêtre de droite, vérifiez que l’onglet **Conception** est sélectionné.
1. Dans la fenêtre **Traitement entrant**, sélectionnez **+ Ajouter une stratégie**.

    :::image type="content" source="media/mock-api-responses/03-mock-api-responses-03-enable-mocking.png" alt-text="Réponse d’API simulée" border="false":::

1. Sélectionnez **Simuler des réponses** dans la galerie.

    :::image type="content" source="media/mock-api-responses/mock-responses-policy-tile.png" alt-text="Réponse d’API simulée" border="false":::

1. Dans la zone de texte **réponse de Gestion des API:**, tapez **200 OK, application/json**. Cette sélection indique que votre API doit retourner l’exemple de réponse que vous avez défini dans la section précédente.

    :::image type="content" source="media/mock-api-responses/mock-api-responses-set-mocking.png" alt-text="Réponse d’API simulée":::

1. Sélectionnez **Enregistrer**.

    > [!TIP]
    > Une barre jaune accompagnée du texte **La simulation de réponse est activée** pour votre API indique que les réponses retournées par le service Gestion des API sont simulées par la [stratégie de simulation](api-management-advanced-policies.md#mock-response) et ne sont pas produites par le back-end.

## <a name="test-the-mocked-api"></a>Tester l’API simulée

1. Sélectionnez l’API que vous avez créée dans [Créer une API de test](#create-a-test-api).
1. Sélectionnez l’onglet **Test**.
1. Vérifiez que l’API **Appel de test** est sélectionnée. Sélectionnez **Envoyer** pour effectuer un appel de test.

   :::image type="content" source="media/mock-api-responses/03-mock-api-responses-04-test-mocking.png" alt-text="Réponse d’API simulée":::

1. La **réponse HTTP** affiche le JSON fourni comme exemple dans la première section du didacticiel.

    :::image type="content" source="media/mock-api-responses/mock-api-responses-test-response.png" alt-text="Réponse d’API simulée":::

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Créer une API de test
> * Ajouter une opération à l’API de test
> * Activer la simulation des réponses
> * Tester l’API simulée

Passez au tutoriel suivant :

> [!div class="nextstepaction"]
> [Transform and protect your API](transform-api.md) (Transformer et protéger votre API)
