---
title: 'Tutoriel : Déboguer des API dans la Gestion des API Azure à l’aide du suivi des demandes'
description: Suivez les étapes de ce tutoriel pour apprendre à suivre et à inspecter les étapes du traitement des demandes dans la Gestion des API Azure.
services: api-management
documentationcenter: ''
author: vladvino
editor: ''
ms.service: api-management
ms.topic: tutorial
ms.date: 10/30/2020
ms.author: apimpm
ms.openlocfilehash: 531e346569b85ababc382f997fd7764a92b3d05f
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94542312"
---
# <a name="tutorial-debug-your-apis-using-request-tracing"></a>Tutoriel : Déboguer vos API à l’aide du suivi des demandes

Ce tutoriel explique comment inspecter (suivre) le traitement des demandes dans la Gestion des API Azure pour vous aider à déboguer et à dépanner votre API. 

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Suivre un exemple d’appel
> * Passer en revue les étapes du traitement des demandes

:::image type="content" source="media/api-management-howto-api-inspector/api-inspector-001.png" alt-text="Inspecteur d’API":::

## <a name="prerequisites"></a>Prérequis

+ Apprenez la [terminologie relative à Gestion des API Azure](api-management-terminology.md).
+ Suivez le guide de démarrage rapide suivant : [Créer une instance du service Gestion des API Azure](get-started-create-service-instance.md).
+ Effectuez toutes les étapes du tutoriel suivant : [Importer et publier votre première API](import-and-publish.md).

## <a name="verify-allow-tracing-setting"></a>Vérifier le paramètre Autoriser le suivi 

Le paramètre **Autoriser le suivi** pour l’abonnement utilisé pour votre API doit être activé. Si vous utilisez l’abonnement avec accès complet intégré, il est activé par défaut. Pour effectuer la vérification dans le portail, accédez à votre instance de Gestion des API et sélectionnez **Abonnements**.

   :::image type="content" source="media/api-management-howto-api-inspector/allow-tracing.png" alt-text="Autoriser le suivi pour l’abonnement":::

## <a name="trace-a-call"></a>Suivre un appel

1. Connectez-vous au [portail Azure](https://portal.azure.com) et accédez à votre instance de Gestion des API.
1. Sélectionnez **API**.
1. Sélectionnez **API de conférence de démonstration** dans votre liste d’API.
1. Sélectionnez l’onglet **Test**.
1. Sélectionnez l’opération **GetSpeakers**.
1. Vérifiez que l’en-tête de requête HTTP comprend **Ocp-Apim-Trace: True** et une valeur valide pour **Ocp-Apim-Subscription-Key**. Si ce n’est pas le cas, sélectionnez **+ Ajouter un en-tête** pour ajouter l’en-tête.
1. Sélectionnez **Envoyer** pour effectuer un appel d’API.

  :::image type="content" source="media/api-management-howto-api-inspector/06-debug-your-apis-01-trace-call.png" alt-text="Configurer le suivi d’API":::

> [!TIP]
> Si la valeur de **Ocp-Apim-Subscription-Key** n’est pas automatiquement renseignée dans la requête HTTP, vous pouvez la récupérer dans le portail. Sélectionnez **Abonnements**, puis ouvrez le menu contextuel ( **...** ) pour votre abonnement. Sélectionnez **Afficher/masquer les clés**. Vous pouvez également regénérer les clés si nécessaire. Ajoutez ensuite une clé à l’en-tête.

## <a name="review-trace-information"></a>Examiner les informations de suivi

1. Une fois l’appel terminé, accédez à l’onglet **Trace** dans la **réponse HTTP**.
1. Sélectionnez les liens suivants pour accéder aux informations de suivi détaillées : **Entrant**, **Principal**, **Sortant**.

     :::image type="content" source="media/api-management-howto-api-inspector/response-trace.png" alt-text="Vérifier le suivi de la réponse":::

    * **Entrant** : affiche la demande d’origine reçue par le service Gestion des API de l’appelant et les stratégies appliquées à la demande. Par exemple, si vous avez ajouté des stratégies dans [Tutoriel : Transformer et protéger votre API](transform-api.md), elles s’affichent ici.

    * **Principal** : affiche les demandes envoyées par le service Gestion des API au backend d’API et la réponse qu’il a reçue.

    * **Sortant** : affiche toutes les stratégies appliquées à la réponse avant son envoi à l’appelant.

    > [!TIP]
    > Chaque étape indique également le temps écoulé depuis la réception de la demande par le service Gestion des API.

1. Sous l’onglet **Message**, l’en-tête **ocp-apim-trace-location** affiche l’emplacement des données de suivi dans le stockage d’objets blob Azure. Si nécessaire, accédez à cet emplacement pour récupérer le suivi.

     :::image type="content" source="media/api-management-howto-api-inspector/response-message.png" alt-text="Emplacement du suivi dans le stockage Azure":::
## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Suivre un exemple d’appel
> * Passer en revue les étapes du traitement des demandes

Passez au tutoriel suivant :

> [!div class="nextstepaction"]
> [Utiliser des révisions](api-management-get-started-revise-api.md)
