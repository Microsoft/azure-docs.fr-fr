---
title: Stratégie d’apprentissage - Personalizer
description: Les paramètres d’apprentissage déterminent les *hyperparamètres* d'apprentissage des modèles. Deux modèles contenant des données identiques, mais ayant été entraînés à l’aide de paramètres d’apprentissage différents, se révéleront différents.
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: conceptual
ms.date: 02/20/2020
ms.openlocfilehash: 1d2038e3796df843736eb80f7e5645f4141c82f4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91253630"
---
# <a name="learning-policy-and-settings"></a>Stratégie et paramètres d’apprentissage

Les paramètres d’apprentissage déterminent les *hyperparamètres* d'apprentissage des modèles. Deux modèles contenant des données identiques, mais ayant été entraînés à l’aide de paramètres d’apprentissage différents, se révéleront différents.

La [stratégie et les paramètres d’apprentissage](how-to-settings.md#configure-rewards-for-the-feedback-loop) sont définis sur votre ressource Personalizer dans le portail Azure.

## <a name="import-and-export-learning-policies"></a>Importer et exporter les stratégies d’apprentissage

Vous pouvez importer et exporter des fichiers de stratégie d’apprentissage à partir du portail Azure. Utilisez cette méthode pour enregistrer les stratégies existantes, les tester, les remplacer et les archiver dans votre contrôle de code source en tant qu’artefacts, à des fins de références et d'audits ultérieurs.

Découvrez [comment](how-to-manage-model.md#import-a-new-learning-policy) importer et exporter une stratégie d’apprentissage dans le portail Azure pour votre ressource Personalizer.

## <a name="understand-learning-policy-settings"></a>Comprendre les paramètres des stratégies d’apprentissage

Les paramètres des stratégies d'apprentissage ne sont pas censés être modifiés. Pour les modifier, vous devez impérativement comprendre la manière dont ils affectent Personalizer. Si ce n'est pas le cas, vous risquez de provoquer des problèmes, notamment d’invalider des modèles Personalizer.

Personalizer utilise [vowpalwabbit](https://github.com/VowpalWabbit) pour entraîner et scorer les événements. Reportez-vous à la [documentation vowpalwabbit](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Command-line-arguments) sur la façon de modifier les paramètres d’apprentissage avec vowpalwabbit. Une fois que vous avez les bons arguments de ligne de commande, enregistrez la commande dans un fichier au format suivant (remplacez la valeur de la propriété d’arguments par la commande souhaitée) et chargez le fichier pour importer les paramètres d’apprentissage dans le volet **Paramètres de modèle et d'apprentissage**  du portail Azure pour votre ressource Personalizer.

Le fichier `.json` suivant est un exemple de stratégie d’apprentissage.

```json
{
  "name": "new learning settings",
  "arguments": " --cb_explore_adf --epsilon 0.2 --power_t 0 -l 0.001 --cb_type mtr -q ::"
}
```

## <a name="compare-learning-policies"></a>Comparer les stratégies d’apprentissage

Vous pouvez comparer les performances des différentes stratégies d’apprentissage par rapport à d’anciennes données de journaux Personalizer en procédant à des [évaluations hors connexion](concepts-offline-evaluation.md).

[Chargez vos propres stratégies d’apprentissage](how-to-manage-model.md) pour les comparer à la stratégie d’apprentissage actuelle.

## <a name="optimize-learning-policies"></a>Optimiser les stratégies d’apprentissage

Personalizer peut créer une stratégie d’apprentissage optimisée dans une [évaluation hors connexion](how-to-offline-evaluation.md). Une stratégie d’apprentissage optimisée dotée de meilleures récompenses dans une évaluation hors connexion, obtiendra de meilleurs résultats si elle est utilisée en ligne dans Personalizer.

Après avoir optimisé une stratégie d'apprentissage, vous pouvez l’appliquer directement à Personalizer pour lui permettre de remplacement immédiatement la stratégie actuelle. Vous pouvez également enregistrer la stratégie optimisée pour une évaluation supplémentaire, puis décider ultérieurement de la supprimer, de l’enregistrer ou de l’appliquer.

## <a name="next-steps"></a>Étapes suivantes

* Découvrez les [événements actifs et inactifs](concept-active-inactive-events.md).
