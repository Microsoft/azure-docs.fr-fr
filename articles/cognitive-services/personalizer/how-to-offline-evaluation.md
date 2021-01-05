---
title: Guide pratique pour effectuer une évaluation hors connexion - Personalizer
titleSuffix: Azure Cognitive Services
description: Cet article vous montre comment utiliser l’évaluation hors connexion pour mesurer l’efficacité de votre application et analyser votre boucle d’apprentissage.
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: how-to
ms.date: 02/20/2020
ms.openlocfilehash: a473085f9c94ca42a75d01b342d60cc33836b096
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88244837"
---
# <a name="analyze-your-learning-loop-with-an-offline-evaluation"></a>Analysez votre boucle d’apprentissage avec une évaluation hors connexion

Découvrez comment effectuer une évaluation hors connexion et comment comprendre les résultats.

Les évaluations hors connexion vous permettent de mesurer l’efficacité de Personalizer en comparaison du comportement par défaut de votre application, de trouver quelles caractéristiques contribuent le plus à la personnalisation, et de découvrir automatiquement de nouvelles valeurs de l’apprentissage automatique.

Pour plus d’informations, consultez [Évaluations hors connexion](concepts-offline-evaluation.md).

## <a name="prerequisites"></a>Conditions préalables requises

* Boucle Personalizer configurée
* La boucle Personalizer doit avoir une quantité de données représentative : pour fournir un ordre de grandeur, nous recommandons au moins 50 000 événements dans ses journaux pour obtenir des résultats d’évaluation significatifs. Si vous le souhaitez, vous pouvez aussi avoir des fichiers de _stratégie d’apprentissage_ préalablement exportés que vous pouvez comparer et tester dans la même évaluation.

## <a name="run-an-offline-evaluation"></a>Exécuter une évaluation hors connexion

1. Dans le [portail Azure](https://azure.microsoft.com/free/cognitive-services), recherchez votre ressource Personalizer.
1. Sur le portail Azure, accédez à la section **Évaluations**, puis sélectionnez l’option **Créer une évaluation**.
    ![Sur le portail Azure, accédez à la section **Évaluations**, puis sélectionnez l’option **Créer une évaluation**.](./media/offline-evaluation/create-new-offline-evaluation.png)
1. Configurez les valeurs suivantes :

    * Nom d’évaluation.
    * Dates de début et de fin : il s’agit de dates qui spécifient la plage de données à utiliser pour l’évaluation. Ces données doivent être présentes dans les journaux, tel que spécifié dans la valeur de [Conservation des données](how-to-settings.md).
    * Découverte d’optimisation définie sur **Oui**.

    > [!div class="mx-imgBorder"]
    > ![Choisir les paramètres d’évaluation hors connexion](./media/offline-evaluation/create-an-evaluation-form.png)

1. Démarrez l’Évaluation en sélectionnant **OK**.

## <a name="review-the-evaluation-results"></a>Examiner les résultats de l’évaluation

L’exécution des évaluations peut durer longtemps, selon la quantité de données à traiter, le nombre de stratégies d’apprentissage à comparer, et si une optimisation a été demandée.

Une fois l’opération terminée, vous pouvez sélectionner l’évaluation dans la liste des évaluations, puis **Comparer le score de votre application à d’autres paramètres d’apprentissage potentiels**. Sélectionnez cette fonctionnalité quand vous souhaitez comparer les performances de votre stratégie d’apprentissage actuelle à celles d’une nouvelle stratégie.

1. Passez en revue les performances des [stratégies d’apprentissage](concepts-offline-evaluation.md#discovering-the-optimized-learning-policy).

    > [!div class="mx-imgBorder"]
    > [![Examiner les résultats de l’évaluation](./media/offline-evaluation/evaluation-results.png)](./media/offline-evaluation/evaluation-results.png#lightbox)

1. Sélectionnez **Appliquer** pour appliquer la stratégie qui améliore le mieux le modèle pour vos données.

## <a name="next-steps"></a>Étapes suivantes

* Découvrez plus en détail le [fonctionnement des évaluations hors connexion](concepts-offline-evaluation.md).
